# 05 — Monitoring & Observability Deep-Dive

> **Mục tiêu thảo luận:** Hiểu cơ chế sampling, multi-turn judge, trace structure đủ để build "AI Ops" function cho ngân hàng — phát hiện drift, abuse, regulatory risk trong production gần realtime.

---

## 5.1 Trace anatomy (MLflow 3)

### Cấu trúc
Theo `mlflow3/genai/tracing/tracing-101`:

```
Trace
├── TraceInfo
│   ├── trace_id, status, timing, tags
│   └── (in relational DB → fast query)
└── TraceData
    └── [Span, Span, Span, ...]    ◀── hierarchical, app workflow
                                        (in artifact storage → cheap & large)
```

### Span types thường gặp (suy từ docs autolog)
- **LLM call span** — provider, model, prompt, completion, tokens.
- **Tool call span** — tool name, args, return.
- **Retrieval span** — query, top-k, returned chunks.
- **Custom span** — user-defined (e.g., business rule check).

### OTel compatibility
Docs xác nhận: *"MLflow Traces are compatible with OpenTelemetry specifications"*. Có nghĩa:
- Span structure tương thích → export ra Datadog/New Relic/Tempo được.
- Trace context propagation chuẩn W3C.

> 🏦 **Banking insight:**
> Ngân hàng đã chuẩn hóa OTel ở core/microservice rồi sẽ **không cần thay APM stack**. Mọi span agent merge vào trace W3C của microservice gốc → end-to-end visibility: mobile app → API gateway → microservice → agent → tool → core banking.
>
> Pattern: agent emit OTel span với parent_id từ caller → join chung trace với upstream. SRE team thấy "latency 8s này do agent retrieve 6s" rõ ràng.

---

## 5.2 Production monitoring — cơ chế (MLflow 3)

### Workflow scheduled scorer
```
[Live traces stream] ──► [Sampler] ──► [Scheduled scorer] ──► [Feedback attached]
                            │
                            │ sample_rate ∈ [0.05, 1.0]
                            ▼
                        [Selected traces]
```

### Tham số quan trọng (docs production-monitoring)
- Max **20 scorers concurrent** / experiment.
- Custom scorer: chỉ `@scorer` decorator, define trong notebook, self-contained imports.
- **Warm-up 15-20 phút** trước kết quả đầu tiên.
- Sample rate khuyến nghị docs:
  - **1.0** cho critical safety check.
  - **0.05-0.2** cho judge đắt (complex prompt, big context).
  - **0.3-0.5** cho dev iteration.

### Math kinh tế sampling

```
Cost/day = QPS × 86400 × sample_rate × Σ(cost_per_scorer)
Coverage = sample_rate × QPS × 86400 ≈ assessed traces/day
```

**Ví dụ retail chatbot:**
- QPS 20, judges = safety (1.0) + groundedness (0.2) + brand_voice (0.1).
- Safety: 20 × 86400 × 1 × $0.002 ≈ $3,456/day.
- Groundedness: 20 × 86400 × 0.2 × $0.005 ≈ $1,728/day.
- Brand_voice: 20 × 86400 × 0.1 × $0.003 ≈ $518/day.
- **Total ≈ $5,700/day** — đáng cân nhắc; đàm phán với CFO.

> 🏦 **Banking insight:**
> 3 cấp scorer cho banking agent:
> - **Tier 1 — Safety + PII check + jailbreak detect:** sample = 1.0 (mọi request, KHÔNG miss).
> - **Tier 2 — Groundedness + correctness:** sample = 0.2 (đủ statistical signal).
> - **Tier 3 — Brand voice + regulatory disclosure:** sample = 0.05 hoặc batch nightly.

> ⚠️ **Tradeoff warm-up 15-20p:** Production monitoring KHÔNG phải realtime alert. Cho hard-realtime guardrail (block toxic response trước khi gửi user), phải dùng **AI Gateway synchronous guardrail** (xem deep-dive 06), KHÔNG dùng scheduled scorer.

---

## 5.3 Multi-turn judges — cơ chế

### Session grouping
- Default: traces gom theo session_id qua tag `mlflow.trace.session`.
- Hội thoại "complete" sau **5 phút inactivity** (default).

### Judge scope
- Đánh giá **cả cuộc hội thoại** thay vì 1 lượt.
- Patterns chấm: user frustration, conversation completeness.

### Ý nghĩa thiết kế
Đây là khác biệt lớn so với batch eval — chấm "đã giải quyết được nhu cầu user chưa", không chỉ "câu trả lời lẻ có hợp lý không".

> 🏦 **Banking insight (multi-turn signal):**
> Indicator thực dụng cho retail support bot:
> - **`user_frustration`** — user lặp lại câu hỏi 3 lần → escalate human agent.
> - **`conversation_completeness`** — user bỏ giữa chừng → flag UX issue.
> - **`unauthorized_topic_persistence`** — user dụ vượt scope lặp lại → block + alert security.
> - **`product_handoff_success`** — agent có chuyển sang sản phẩm đúng (loan, card, savings) không.

> ❓ **Open question:** 5 phút inactivity có config được không? Banking có session retail dài hơn (mobile app session, IVR call).

---

## 5.4 Dashboards & SQL alerts

### Cơ chế (suy từ docs Lifecycle + Inference Tables + system tables)
- Inference Tables + system tables (`system.ai_gateway.usage`) là Delta → SQL trên Databricks SQL.
- Dashboard built-in cho AI Gateway, hoặc custom Lakeview.
- Databricks SQL Alerts → trigger Slack/Email/PagerDuty khi metric vượt threshold.

### Pattern alert ngân hàng

| Alert | Source | Threshold | Action |
|---|---|---|---|
| Safety judge fail rate > 0.1% trong 15 phút | scorer + inference table | rolling window | PagerDuty AI Ops |
| PII leak detected | custom scorer | 1 event | Page security + freeze endpoint |
| Latency p95 > SLA | inference table latency_ms | 5 min window | SRE channel |
| Cost spike > $X/hour | usage table | hourly | Finance + AI center |
| Endpoint 5xx > 1% | inference table status_code | 5 min | SRE |
| Drift in retrieve@5 score | scorer | weekly delta | Data + ML team |
| Multi-turn user_frustration spike | multi-turn judge | daily | UX team |
| Unauthorized topic attempts > N | safety scorer | daily | Security review |

> 🏦 **Banking insight:**
> Dashboard "AI Ops" cần expose cho CIO weekly + Risk monthly. Layout đề xuất:
> - **Topline:** Total traces, % pass safety, % pass groundedness, total cost, p95 latency.
> - **Per-agent:** rank theo failure rate.
> - **Drill-down:** Click vào agent → top 10 trace fail, link MLflow Trace UI.
> - **Trend:** 30 days, breakdown by channel (mobile/web/IVR).

---

## 5.5 OpenTelemetry export — không lock-in

Docs (`gen-ai-capabilities`): *"OpenTelemetry Integration: Export traces to third-party monitoring tools"*.

### Lợi ích thực tế
- Ngân hàng đã Datadog/Splunk/Dynatrace: trace agent đổ vào cùng stack → SRE 1 view.
- Compliance team có SIEM (Splunk Enterprise Security, IBM QRadar) → forward selected event qua OTel collector → security operations center.

> 🏦 **Banking insight:**
> Pattern hybrid log:
> - **Inference Tables** lưu trong UC → analytics, ad-hoc query.
> - **OTel export** ra Splunk → real-time monitoring + SIEM correlation.
> - **MLflow Trace UI** cho dev debug.
>
> Không cần chọn 1, dùng cả 3 cho mục đích khác nhau.

---

## 5.6 Feedback loop — vòng cải tiến

```
[Production trace] ─► [Scorer fail] ─► [Trace identified low-quality]
                                            │
            ┌───────────────────────────────┼──────────┐
            ▼                               ▼          ▼
  [Auto add to eval set]      [SME label in Review App]   [Customer feedback (👍/👎)]
                                     │
                                     ▼
                          [Updated eval dataset]
                                     │
                                     ▼
                          [Regression run]
                                     │
                                     ▼
                       [New agent version deploy]
```

> 🏦 **Banking insight:**
> KPI ngân hàng cho AI Ops:
> - **MTTR cho AI bug:** thời gian từ "fail detected" → "fix deployed".
> - **% production failure converted to eval set:** càng cao càng tự học hỏi.
> - **Eval set growth rate:** dataset phải lớn dần để cover edge case mới.
> - **Regression test latency:** đo bao lâu test full eval set khi đổi prompt.

---

## 5.7 Inference table loss & dual-logging strategy

Như đã đề cập deep-dive 04: docs ghi *"best effort delivery"*. Đối với banking:

### Mitigation patterns

**Pattern A — Dual sink (recommended for compliance use case):**
```
[Endpoint] ──► [AI Gateway Inference Table] (analytics, eval, dev)
        │
        └──► [App backend log] ──► [Kafka] ──► [Delta] (source of truth audit)
```

**Pattern B — Reconciliation job:**
- Nightly job compare app log vs inference table.
- Alert nếu mismatch > threshold (1%).
- Document log loss rate cho audit.

> 🏦 **Banking insight:**
> Inference Tables tốt cho 90% analytics, NHƯNG audit/compliance log không nên rely 100%. Đầu tư dual log từ ngày 1 nếu deploy cho regulated workload.

---

## 5.8 Hot debate points

1. **Sample rate strategy:** Mặc định 1.0 hay 0.2 cho safety? Có budget ceiling tổng cho monitoring không?
2. **Multi-turn session window:** 5 phút đủ cho IVR (call 15-20 phút) không?
3. **OTel + Splunk dual sink:** Đầu tư từ ngày 1 hay đợi pain point?
4. **AI Ops team org:** AI center own dashboard hay SRE own?
5. **Convert failure → eval:** Tự động hóa được không? Hay phải SME approve mỗi case?
6. **Drift detection:** Embed-level drift (chunk distribution thay đổi) có monitor được không qua MLflow? Hay phải custom?
