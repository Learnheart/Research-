# 05. Per-Service Forecasting — Business Unit of Value & user-level attribution

[CẬP NHẬT: 22/05/2026]

> **Insight cốt lõi:** Forecast cost theo service phải bắt đầu từ **Business Unit of Value (BUV)** mà người dùng cảm nhận được — `1 doc`, `1.000 chars`, `1 slide` — không phải `tokens`. Token là đơn vị **kỹ thuật** (kế toán nội bộ), BUV là đơn vị **kinh doanh** (chargeback + forecast). Service multi-step như PowerPoint generate phải tách thêm `slide_type` vì variance deck-level quá lớn để forecast được.

---

## ⚡ 30-sec card

```
forecast[service, t] = active_users[service, t]
                     × BUV_per_user[service, t]
                     × cost_per_BUV[service, t]
                     + amortized_overhead

BUV chọn theo service:
  summarizer    → 1 document        (split theo doc_length_bucket: S/M/L)
  translator    → 1.000 source chars (split theo language_pair)
  powerpointer  → 1 slide           (split theo slide_type: text/image/chart)
```

**Cần biết:**

| Câu hỏi | Đi tới |
|---|---|
| Cách tag user + service ở Gateway | §1.1 |
| Schema cost ledger để query per-user | §1.2 |
| Cách chọn BUV cho service mới | §2 |
| Công thức cost-per-BUV chi tiết 3 service | §3.1–§3.3 |
| Quy trình refresh forecast | §4 |
| Pitfall thường gặp | §5 |

---

## §1. Cost per user — instrumentation & query

### 1.1. Bắt buộc 3 lớp tag tại AI Gateway

| Layer | Field | Nguồn dữ liệu | Vai trò |
|---|---|---|---|
| **Identity** | `user_id` / `caller_principal` | OAuth/SSO header tại Gateway; IAM principal trên Bedrock | Chargeback per user |
| **Service** | `service_id` ∈ {`summarizer`, `translator`, `powerpointer`} | Header `X-Service-ID` enforced ở Gateway (HTTP 400 nếu thiếu) | Forecast per service |
| **Business** | `cost_center`, `business_unit`, `use_case_id` | Header / metadata (xem [[_appendix/allocation]] §3) | Allocation về P&L |

> **Quy tắc:** Request không đủ 3 lớp tag → Gateway reject. Không có tag = không có attribution = không có forecast [1].

### 1.2. Schema cost ledger (row = 1 invocation)

```sql
-- Delta table cost_ledger, partition by dt
trace_id            STRING   -- join với OTel orchestration trace
dt                  DATE
user_id             STRING
service_id          STRING   -- summarizer | translator | powerpointer
agent_version       STRING
model_id            STRING

-- L1 raw counts (đa modality, KHÔNG pre-convert sang $)
tokens_in           BIGINT
tokens_out          BIGINT
cached_tokens       BIGINT
image_in_count      INT
image_out_count     INT
audio_out_chars     INT

-- L3 orchestration
tool_calls          INT
retry_count         INT

-- BUV — Business Unit of Value per service
buv_unit            STRING   -- 'doc' | '1k_chars' | 'slide'
buv_qty             DOUBLE   -- 1 | 7.5 (= 7500 chars) | 12 (slides)
buv_subtype         STRING   -- doc_length_bucket | lang_pair | slide_type

-- Cost (computed downstream từ rate card)
cost_l1_usd         DOUBLE
cost_l3_usd         DOUBLE
cost_total_usd      DOUBLE

outcome_status      STRING   -- success | partial | fail
```

**Source-of-truth:** `system.serving.endpoint_usage` (L1 raw counts) join `system.serving.payload` theo `trace_id` (BUV fields trong payload metadata) [2][3]. Pattern này tương đương "GenAI Cost Supervisor Agent" của Capital One — 20 Unity Catalog SQL functions trace mọi invocation [4].

### 1.3. Query cost per user × service × period

```sql
-- Cost & unit economics per user × service × month
SELECT
  user_id,
  service_id,
  date_trunc('month', dt)             AS month,
  count(*)                            AS invocations,
  sum(buv_qty)                        AS buv_total,
  sum(cost_total_usd)                 AS cost_usd,
  sum(cost_total_usd) / NULLIF(sum(buv_qty), 0)
                                      AS cost_per_buv,
  percentile_approx(cost_total_usd, 0.95)
                                      AS p95_invocation_cost,
  sum(retry_count) / count(*)         AS avg_retry_rate
FROM cost_ledger
WHERE dt >= current_date - INTERVAL 30 DAYS
GROUP BY 1, 2, 3
ORDER BY cost_usd DESC;
```

### Citations

- [1] FinOps Foundation. *How to Build a Generative AI Cost and Usage Tracker.* finops.org/wg/how-to-build-a-generative-ai-cost-and-usage-tracker/ (truy cập 22/05/2026)
- [2] Databricks Docs. *Monitor usage for Unity AI Gateway endpoints.* docs.databricks.com/aws/en/ai-gateway/usage-tracking-beta (truy cập 22/05/2026)
- [3] Databricks Blog. *Tracing for Generative AI on Databricks.* databricks.com/blog/tracing-generative-ai-databricks (truy cập 22/05/2026)
- [4] Capital One Software Blog. *Building a GenAI cost supervisor agent in Databricks.* capitalone.com/software/blog/databricks-genai-cost-supervisor-agent/ (truy cập 22/05/2026)

---

## §2. Business Unit of Value — chọn đơn vị đo cho từng service

### 2.1. Quy tắc chọn BUV

1. **User-perceivable** — đơn vị mà người dùng nghĩ tới (1 doc, 1 deck, 1.000 chữ dịch). Không dùng `tokens` vì user không quan tâm.
2. **Variance thấp trong cùng BUV** — coefficient of variation (CV) của `cost_per_BUV` trong cùng service nên < 30%. CV cao → chia nhỏ thêm sub-dimension.
3. **Đếm được tại boundary** — observable ở input hoặc output, không cần peek vào mid-pipeline.
4. **Tự nhiên cho chargeback** — Finance đọc "100 docs × $0.04 = $4" dễ hơn "32M tokens × $... = $4".

### 2.2. Bảng BUV per service

| Service | BUV | Sub-dimension bắt buộc track | CV ước lượng | Lý do sub-dim |
|---|---|---|---:|---|
| **Summarizer** | `1 document summarized` | `doc_length_bucket` ∈ {S < 2K, M 2K–20K, L > 20K} tokens | ~ 25% | L documents dùng map-reduce → cost không tuyến tính với length |
| **Translator** | `1.000 source characters` | `language_pair` (en↔vi, vi↔ja, ...) | ~ 20% | `output_ratio` khác 30–80% giữa pair |
| **PowerPointer** | `1 slide generated` | `slide_type` ∈ {title, text_only, bullet_chart, image, complex} | ~ 15% | Image gen slide cost ×10–50 slide text-only |

### 2.3. Vì sao deck-level không forecast được cho PowerPointer

| Deck | Slides | Image gen | Chart | Cost ước lượng |
|---|---:|---:|---:|---:|
| Memo nội bộ | 5 | 0 | 0 | ~ $0.04 |
| Sales pitch | 20 | 8 | 3 | ~ $1.50 |
| Annual report | 60 | 15 | 20 | ~ $5.80 |

> Deck CV ~ 150% · Slide CV ~ 40% · Slide × slide_type CV ~ 15%. Forecast chính xác đòi hỏi đo ở mức 3.

### Citations

- [5] FinOps Foundation. *Cost Estimation of AI Workloads.* finops.org/wg/cost-estimation-of-ai-workloads/ (truy cập 22/05/2026)
- [6] Drivetrain. *Unit economics for AI SaaS companies: A survival guide for CFOs.* drivetrain.ai/post/unit-economics-of-ai-saas-companies-cfo-guide (truy cập 22/05/2026)
- [7] Acropolium. *AI agent unit economics: TCO, ROI, payback.* acropolium.com/blog/ai-agent-unit-economics/ (truy cập 22/05/2026)

---

## §3. Công thức cost-per-BUV & forecast — chi tiết 3 service

Khung chung (từ [[02_cost_formula]] §2 + [[03_forecasting]] §2):

```
cost_per_BUV[service] = Σ_layer (units × $_per_unit) × (1 + retry_mult)

forecast[service, t]  = active_users[service, t]
                      × BUV_per_user[service, t]
                      × E[cost_per_BUV[service, t] | sub_dim]
                      + amortized_overhead
```

### 3.1. Summarizer — BUV = 1 document

**Công thức:**

```
cost_per_doc =
    ( doc_tokens_in   × $_in[model]
    + doc_tokens_in   × summary_ratio × $_out[model]      # summary_ratio ~ 0.05–0.15
    + planning_tokens × $_out[model]                      # map-reduce chunking
    ) × (1 + retry_mult)

forecast_summarizer[month] =
    active_users × docs_per_user_per_month
                 × E[cost_per_doc | doc_length_bucket]
```

**Ví dụ numeric** (Sonnet 4.6: $3.00/M input, $15.00/M output [[01_rate_card]] §1):

| Bucket | Tokens in | Summary ratio | Retry mult | Cost per doc |
|---|---:|---:|---:|---:|
| S (< 2K) | 1.500 | 0,10 | 1,15 | $0.0078 |
| M (2K–20K) | 8.000 | 0,10 | 1,20 | $0.0432 |
| L (> 20K, map-reduce) | 50.000 + 20.000 planning | 0,10 | 1,30 | $0.3315 |

**Driver phải refresh hàng tuần:**
- `docs_per_user_per_month` — tổng + breakdown theo bucket.
- `summary_ratio` empirical (trung bình thực tế, không assumption).
- `retry_mult` — theo §5.

### 3.2. Translator — BUV = 1.000 source characters

**Công thức:**

```
cost_per_1k_chars =
    ( src_chars × $_in_per_char[model]
    + src_chars × output_ratio[lang_pair] × $_out_per_char[model]
    ) × (1 + retry_mult)
```

**`output_ratio` empirical theo cặp ngôn ngữ** (cần verify với data thật):

| Language pair | output_ratio | Lý do |
|---|---:|---|
| en → vi | ~ 1,30 | Tiếng Việt có khoảng trắng nhiều hơn, dấu |
| vi → en | ~ 0,80 | Anh ngắn gọn hơn |
| en → ja | ~ 0,70 | Ký tự kanji "đậm" thông tin |
| ja → en | ~ 1,50 | Bung ra |
| en → zh-CN | ~ 0,65 | Hán tự đậm |

**Ví dụ numeric** (Haiku 4.5: $1.00/M input, $5.00/M output; quy đổi ~ 4 char/token):

- 1.000 src chars ≈ 250 input tokens
- en → vi: 1.300 output chars ≈ 325 output tokens
- `cost = (250 × $1.00 + 325 × $5.00) × 1,10 / 1M = $0.0021/1k chars`

**Forecast:**

```
forecast_translator[month] =
    active_users × chars_per_user_per_month / 1000
                 × E[cost_per_1k_chars | lang_pair]
```

### 3.3. PowerPointer — BUV = 1 slide × slide_type

Service multi-step, cost không tuyến tính với slide count:

**Công thức:**

```
cost_per_slide[type] =
    planning_share                                       # tokens lập kế hoạch deck ÷ N_slides
  + text_tokens[type]    × $_out
  + ( has_image[type]   ? image_gen_cost          : 0 ) # gpt-image-1 medium ~ $0.042/img
  + ( has_chart[type]   ? chart_extra_tokens × $_out : 0 )
  ) × (1 + retry_mult)

forecast_powerpointer[month] =
    active_users
  × decks_per_user_per_month
  × E[slides_per_deck]
  × Σ_type ( weight[type] × cost_per_slide[type] )
```

**Bảng cost-per-slide-type** (giả định Sonnet 4.6 cho text + `gpt-image-1` medium 1024² cho image; weights dựa trên historical mix — cần calibrate với data thật):

| `slide_type` | Weight (%) | Text tokens out | Image gen | Chart | Cost / slide |
|---|---:|---:|:---:|:---:|---:|
| `title` | 5% | 80 | – | – | $0.0012 |
| `text_only` | 50% | 250 | – | – | $0.0038 |
| `bullet_chart` | 25% | 350 | – | ✅ | $0.0080 |
| `image` | 15% | 200 | ✅ | – | $0.0450 |
| `complex` | 5% | 500 | ✅ | ✅ | $0.0550 |

**Blended $/slide:**

```
= 0,05 × $0.0012 + 0,50 × $0.0038 + 0,25 × $0.0080 + 0,15 × $0.0450 + 0,05 × $0.0550
= ~ $0.0141 / slide
```

**Ví dụ forecast tháng:**

```
100 active users × 4 decks/user/tháng × 15 slides/deck × $0.0141
= ~ $84.6 / tháng
```

**Insight:** `image` slides chỉ chiếm 15% nhưng > 50% tổng cost. **Pre-spend control hiệu quả nhất** là `max_images_per_deck` cap ở Gateway (analog với "modality budget" cho TTS trong audio use case — xem [[04_monitoring]] §2.2) [8].

### Citations

- [8] Galileo. *The Hidden Costs of Agentic AI.* galileo.ai/blog/hidden-cost-of-agentic-ai (truy cập 22/05/2026)
- [9] Agents Arcade. *Cost Modeling for Agentic Systems Before You Go to Production.* agentsarcade.com/blog/cost-modeling-agentic-systems-production (truy cập 22/05/2026)
- [10] OpenAI. *Image generation pricing.* openai.com/api/pricing (truy cập 22/05/2026)

---

## §4. Quy trình lên forecast — cadence

| # | Bước | Cadence | Hoạt động | Output |
|---|---|---|---|---|
| 1 | Thu thập driver | Tuần | Refresh từ `system.serving.endpoint_usage`: `active_users`, `BUV_per_user`, `cost_per_BUV` cho từng service × sub-dim | Driver table |
| 2 | Phân khúc user | Tháng | Cluster user theo intensity (power / regular / occasional) — tránh blended mean che lấp long-tail | User segment table |
| 3 | Calibrate cost-per-BUV | Tháng | Recompute từ rolling 30 ngày; flag drift > 20% so với baseline trước | Updated `cost_per_BUV` per sub-dim |
| 4 | Run 3 scenarios | Tháng | Conservative / Base / Aggressive cho `active_users` growth + `BUV_per_user` growth (xem [[03_forecasting]] §3) | Forecast pack 3 scenario per service |
| 5 | Refresh vendor price | ≤ 14 ngày sau khi vendor đổi giá | Update [[01_rate_card]] → cost_per_BUV → forecast | Re-baseline |

**KPI mục tiêu:**

| KPI | Target sau 6 tháng | Target sau 12 tháng |
|---|---|---|
| MAPE month-ahead per service | ≤ 25% | ≤ 15% |
| % service có forecast theo BUV granular | 100% prod service | 100% prod service |
| % budget breach phát hiện trước (vs reactive) | ≥ 80% | ≥ 90% |
| Drift detection latency | < 1 tuần | < 3 ngày |

### Citations

- [11] FinOps Foundation. *Effect of Optimization on AI Forecasting.* finops.org/wg/effect-of-optimization-on-ai-forecasting/ (truy cập 22/05/2026)

---

## §5. Pitfall thường gặp

| Pitfall | Hậu quả | Phòng tránh |
|---|---|---|
| Dùng **mean `cost_per_BUV`** không tính p95 | Underestimate khi xuất hiện outlier (deck 200 slide, doc 500 trang, dịch 100K chars) | Forecast = mean × N + p95 buffer cho top 5% users |
| Không tách **sub-dimension** (`slide_type`, `lang_pair`, `doc_length_bucket`) | Variance ăn forecast 40–60% | Bắt buộc tag sub-dim tại Gateway từ ngày 1 |
| **`retry_mult` không track riêng theo service** | Service nhiều tool call (PPT) sẽ wrap variance vào service khác → forecast service kia sai oan | OTel span attribute `retry_count` per `service_id` |
| Forecast theo **deck/doc/translation_job** thay vì BUV granular | User behavior thay đổi (deck dài hơn) mà forecast không bắt được | Track distribution `BUV_per_invocation`, không chỉ mean |
| Bỏ qua **L2–L6 overhead** | Underestimate 30–50% — token cost chỉ là 30–50% TCO [8][7] | Allocate flat % overhead lên `cost_per_BUV` (xem [[02_cost_formula]] §2) |
| Lock model mix cũ khi vendor đổi giá | Forecast lệch khi tự động fail-over sang model rẻ hơn | Re-calibrate `cost_per_BUV` mỗi khi rate card update |
| Quên cached_tokens | Service có long system prompt (PowerPointer planning) overestimate 20–40% | Track `cached_tokens` riêng và áp giá cache-read |

### Citations

- [7] Acropolium. *AI agent unit economics: TCO, ROI, payback.* acropolium.com/blog/ai-agent-unit-economics/ (truy cập 22/05/2026)
- [8] Galileo. *The Hidden Costs of Agentic AI.* galileo.ai/blog/hidden-cost-of-agentic-ai (truy cập 22/05/2026)

---

## §6. Mở rộng cho service mới — checklist onboarding

Khi onboard service thứ N:

- [ ] Định nghĩa BUV theo §2.1 (user-perceivable, CV < 30%, đếm tại boundary).
- [ ] Xác định 1–2 sub-dimension bắt buộc track (cái gì gây variance trong cùng BUV).
- [ ] Đăng ký `service_id` ở Gateway; bật reject nếu thiếu header.
- [ ] Bổ sung cột `buv_unit`, `buv_qty`, `buv_subtype` vào event payload.
- [ ] Thu thập 30 ngày data trước khi forecast — không extrapolate trên < 30 ngày (xem [[03_forecasting]] §6).
- [ ] Tính `cost_per_BUV` baseline mean + p95 per sub-dim.
- [ ] Add service vào driver refresh job (§4 bước 1).
- [ ] Cấu hình budget per-service ở Gateway (xem [[04_monitoring]] §2).
- [ ] Add metric `cost_per_BUV[service]` vào dashboard P3 alert (drift > 20% WoW).

---

## Liên kết

- Đơn giá compute `$_per_unit` per modality: [[01_rate_card]]
- Khung 6 lớp + công thức gốc: [[02_cost_formula]]
- Driver-based forecasting framework gốc: [[03_forecasting]]
- Pre-spend control + alert threshold: [[04_monitoring]]
- Tagging taxonomy + chargeback flow: [[_appendix/allocation]]

## Nguồn (tổng hợp)

- Capital One Software Blog. *Building a GenAI cost supervisor agent in Databricks.*
- Databricks Docs. *Monitor usage for Unity AI Gateway endpoints — usage-tracking-beta.*
- Databricks Blog. *Tracing for Generative AI on Databricks.*
- FinOps Foundation. *How to Build a Generative AI Cost and Usage Tracker.*
- FinOps Foundation. *Cost Estimation of AI Workloads.*
- FinOps Foundation. *Effect of Optimization on AI Forecasting.*
- Drivetrain. *Unit economics for AI SaaS companies.*
- Acropolium. *AI agent unit economics: TCO, ROI, payback.*
- Galileo. *The Hidden Costs of Agentic AI.*
- Agents Arcade. *Cost Modeling for Agentic Systems Before You Go to Production.*
- OpenAI. *API Pricing (image generation).*
