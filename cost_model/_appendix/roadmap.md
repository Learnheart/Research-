# 07. Strategic Roadmap — R&D → POC → Production

> **Executive 1-pager (đọc trước nếu thời gian giới hạn):**
> Đề xuất triển khai FinOps cho Agent Platform qua **3 phase / 12 tháng**, với 3 stage-gate quyết định invest tiếp.
>
> | Phase | Tháng | Mục tiêu cốt lõi | Gate quyết định | Investment ước lượng |
> |---|---|---|---|---|
> | **R&D — Foundation** | 1–3 | Lock taxonomy, tagging policy, baseline data, council charter, tooling stack v1 | **G1:** ≥ 95% prod requests có đủ 6 tag bắt buộc + 1 dashboard L1+L2 live | 1.5 FTE × 3 tháng + Databricks usage |
> | **POC — Pilot** | 4–6 | Apply taxonomy + allocation lên 2–3 use case; deploy budget + alert; showback report | **G2:** Cost per business outcome đo được cho 3 use case; forecast accuracy month-ahead < 25% | 2 FTE × 3 tháng + Langfuse OSS |
> | **Production — Scale & Govern** | 7–12 | Onboard tất cả prod use case; chargeback go-live; FinOps Council BAU | **G3:** ≥ 90% prod use case lên chargeback; ≥ 90% budget breach phát hiện trước; forecast MAPE ≤ 15% | 2 FTE BAU + reassess commercial FinOps platform |
>
> **Expected outcome 12 tháng:** Cost visibility theo 6 lớp; chargeback chính xác cho mọi use case; forecast scenario-based; pre-spend controls active; cơ chế governance bi-weekly. **Cảnh báo:** Phụ thuộc vào kỷ luật tagging — đây là risk #1.

---

## 1. Guiding Principles cho roadmap

| # | Principle | Implication |
|---|---|---|
| 1 | **Visibility trước Control trước Optimization** | Không cố optimize khi chưa thấy đủ cost; không enforce policy khi chưa có showback ổn định |
| 2 | **Showback trước Chargeback** | Chargeback chính xác cần ≥ 6 tháng baseline showback |
| 3 | **Adopt-first, Build-second** | Mọi capability đã có native (Databricks/Cloud) → adopt; chỉ build connector + business logic riêng |
| 4 | **MECE 6-layer taxonomy là vocabulary chung** | Mọi báo cáo, dashboard, alert phải map về 6 lớp; không taxonomy ad-hoc |
| 5 | **Tagging discipline là điều kiện sống còn** | Không có tag = không có allocation = không có FinOps |
| 6 | **Operate, đừng project** | FinOps Council bi-weekly là cơ chế vận hành, không phải audit project |

## 2. Phase 1 — R&D / Foundation (Tháng 1–3)

### 2.1. Mục tiêu

Đặt nền móng: lock vocabulary, lock policy, lock tooling stack, có baseline data để Phase 2 chạy được.

### 2.2. Workstream

| WS | Hoạt động chính | Output | Owner |
|---|---|---|---|
| **WS1: Taxonomy & Methodology** | Lock 6-layer cost taxonomy (`02_cost_taxonomy/`); driver-based forecast model (`04_forecasting/`) | Methodology pack approved | FinOps Lead + Platform Lead |
| **WS2: Tagging Policy** | Định nghĩa 6 tag bắt buộc + 4 khuyến nghị; quy trình enforce ở Gateway + CI/CD + IAM | Policy doc signed; technical spec | Platform Eng + Security |
| **WS3: Tooling Stack v1** | Set up Unity AI Gateway cost tracking; deploy Langfuse OSS self-hosted; OTel SDK trong agent framework | Tooling live; smoke test pass | Platform Eng |
| **WS4: Baseline Data Collection** | Thu thập 60–90 ngày cost data; clean + tag retrofit cho top use case hiện hữu | Baseline dataset; data quality report | Data Eng |
| **WS5: Governance Setup** | Charter FinOps Council; assign members; bi-weekly cadence; agenda template | Charter signed; 2 meeting đầu tiên đã chạy | Head of AI Platform |
| **WS6: Quick-win dashboard** | Build dashboard L1+L2 (model cost + compute cost) theo use_case_id; cập nhật daily | Dashboard go-live, ≥ 5 stakeholder dùng tuần đầu | Data + Platform |

### 2.3. KPI Phase 1

| KPI | Target |
|---|---|
| % prod request có đủ 6 tag bắt buộc | ≥ 95% |
| Số use case có cost dashboard | ≥ 5 |
| Số council meeting đã chạy | ≥ 4 |
| Số driver thu thập được trong forecast model | ≥ 5/6 |

### 2.4. Gate G1 (cuối tháng 3)

> Pass nếu: ≥ 95% prod requests có đủ tag + L1+L2 dashboard live + FinOps Council chạy đều + baseline dataset 60+ ngày. **Fail action:** Kéo dài R&D 1 tháng, không tiến POC khi tagging chưa đạt.

---

## 3. Phase 2 — POC / Pilot (Tháng 4–6)

### 3.1. Mục tiêu

Validate end-to-end methodology trên 2–3 use case thật. Đo được cost per business outcome. Deploy pre-spend controls.

### 3.2. Use case ứng viên (selection criteria)

| Tiêu chí | Trọng số |
|---|---|
| Volume đủ lớn để có signal (≥ 10K invocations/tháng) | High |
| Business outcome định nghĩa được rõ (đơn vị đo được) | High |
| BU owner sẵn sàng participate trong council | High |
| Đa dạng pattern (1 RAG-heavy, 1 multi-agent, 1 single-shot) | Medium |
| Đại diện cho ≥ 2 BU khác nhau | Medium |

### 3.3. Workstream

| WS | Hoạt động chính | Output |
|---|---|---|
| **WS1: Pilot use case onboarding** | Apply 6-layer taxonomy + tagging + unit economics cho 2–3 pilot | Pilot agents instrumented end-to-end |
| **WS2: Allocation engine v1** | Build allocation engine (Direct/Shared/Overhead); chạy month-end report cho pilot | Allocation report tháng cuối Phase 2 |
| **WS3: Forecasting v1** | Calibrate driver model với data Phase 1; chạy 3 scenarios; CFO review | Quarterly forecast pack |
| **WS4: Pre-spend controls** | Cấu hình budget per-use-case; rate limit; policy guardrails ở Gateway | Active controls; first alert dispatched |
| **WS5: Anomaly detection v1** | Static threshold + rolling baseline; alert ở P2/P3 | First-month alert log; tuning notes |
| **WS6: Showback report** | Monthly showback per use-case + per BU; 360° feedback từ BU lead | Report v1; feedback log |

### 3.4. KPI Phase 2

| KPI | Target |
|---|---|
| Cost per business outcome đo được cho pilot | ≥ 3 use case |
| Forecast MAPE month-ahead (pilot) | < 25% |
| % alert P1/P2 với RCA root cause trong SLA | ≥ 80% |
| Showback report on-time delivery | 100% |
| BU lead chấp nhận showback (no major dispute) | ≥ 2/3 pilot |

### 3.5. Gate G2 (cuối tháng 6)

> Pass nếu: 3 use case có cost per outcome + forecast MAPE < 25% + anomaly engine bắt được ≥ 1 incident thật + BU lead nghiệm thu showback. **Fail action:** Mở rộng POC thêm 1 quý trước khi go production.

---

## 4. Phase 3 — Production / Scale (Tháng 7–12)

### 4.1. Mục tiêu

Onboard toàn bộ prod use case; chuyển từ Showback → Chargeback cho use case ổn định; FinOps Council BAU; tái đánh giá tooling commercial.

### 4.2. Workstream

| WS | Hoạt động chính | Output |
|---|---|---|
| **WS1: Mass onboarding** | Onboard 100% prod use case lên taxonomy + tagging | All-use-case dashboard |
| **WS2: Chargeback go-live** | Chuyển use case đạt criteria sang chargeback (≥ 6 tháng showback + tagging accuracy > 95%) | Chargeback ledger entries trong ERP |
| **WS3: Forecast maturity** | Rolling 12-month forecast; quarterly stress test; scenario-based capacity planning | Forecast accuracy MAPE ≤ 15% |
| **WS4: Anomaly v2** | ML-based anomaly (Lakehouse Monitoring); reduce false positive | False-positive rate ≤ 10% |
| **WS5: Optimization rituals** | Quarterly "cost trap hunt" (retry storm, prompt drift, model tier audit) | Quarterly optimization report; saved $ tracked |
| **WS6: Commercial FinOps reassessment** | Reassess Finout/Apptio nếu spend > $1M/quý hoặc > 30 prod use case | Buy/Build/Defer decision documented |

### 4.3. KPI Phase 3

| KPI | Target end of Year 1 |
|---|---|
| % prod use case có chargeback active | ≥ 90% |
| % budget breach phát hiện trước (alert trước 100%) | ≥ 90% |
| Forecast accuracy MAPE month-ahead | ≤ 15% |
| Forecast accuracy MAPE quarter-ahead | ≤ 25% |
| Tag compliance (audit pass rate) | ≥ 98% |
| Quarterly savings từ optimization rituals | ≥ 10% of optimizable spend [cần verify baseline] |
| FinOps Council on-time meeting compliance | ≥ 95% |

### 4.4. Gate G3 (cuối tháng 12)

> Pass nếu: ≥ 90% prod chargeback + forecast MAPE month-ahead ≤ 15% + tag compliance ≥ 98% + ≥ 1 commercial tooling decision documented (adopt or defer). **Output:** Year-2 plan với scale capability + ROI summary cho Board.

---

## 5. Dependencies & Pre-requisites

| Dependency | Mức độ critical | Status (giả định) |
|---|---|---|
| Databricks Unity Catalog + Unity AI Gateway active | **Blocker** | Cần confirm với Platform team |
| Finance buy-in cho FinOps Council charter | **Blocker** | Cần executive sponsor |
| Cost center mapping (HR → IT systems) | High | Phụ thuộc HR/Finance |
| Tagging enforcement ở IAM + Gateway | **Blocker** | Phải có trước Phase 2 |
| Headcount: 1.5–2 FTE FinOps + 0.5–1 FTE Data Eng | High | Cần approve |
| Existing observability backbone (nếu có) | Medium | Tận dụng nếu có Datadog/Grafana |

## 6. Risk Register (top 5)

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Tagging discipline thấp ở team adoption | High | Critical | Enforce at Gateway (HTTP 400 nếu thiếu tag) + CI/CD lint + quarterly audit |
| BU pushback chargeback (dispute storm) | Medium | High | Showback ≥ 6 tháng trước; SLA dispute rõ; FinOps Council mediation |
| Forecast accuracy thấp do data nhiễu Phase 1–2 | High | Medium | Scenario thay vì point estimate; cập nhật rolling |
| Vendor pricing thay đổi đột biến | Medium | High | Multi-provider; provisioned throughput negotiation; quarterly vendor review |
| Headcount không đủ → BAU bị bỏ rơi | Medium | Critical | Hire trước Phase 2; document mọi process; tránh single-person dependency |

## 7. Investment Summary (12 tháng — ước lượng, [cần verify])

| Hạng mục | Year-1 (USD) | Ghi chú |
|---|---|---|
| Headcount FinOps + Platform Eng allocated | $300K–500K | 2–3 FTE blend rate |
| Databricks usage incremental (instrumentation, dashboards) | $30K–60K | Marginal trên hợp đồng đã có |
| Langfuse OSS self-host (compute + ops) | $10K–20K | Hạ tầng có sẵn |
| Build effort (allocation engine, dashboards, OTel SDK) | đã include trong FTE | One-time |
| Training + certification (FinOps for AI) | $5K–10K | FinOps Foundation cert |
| Commercial tooling | $0 Year-1 | Defer; reassess cuối Year-1 |
| **Total Year-1** | **~$345K–590K** | Chủ yếu là FTE; tooling marginal |

> So với mức spend AI dự kiến (cần baseline cụ thể), FinOps program nhằm tiết kiệm 10–25% spend → payback < 12 tháng với scale dự kiến.

## 8. C-Level Talking Points (cho meeting trình Board)

1. **Problem statement:** AI spend đang tăng phi tuyến nhưng visibility, allocation, forecast đều ad-hoc. Risk: thất thoát budget + thiếu accountability cross-BU.
2. **Industry benchmark:** 63% Fortune 500 đã track AI spend; chỉ 14.2% đạt "Run" maturity → window dẫn trước vẫn rộng. Databricks/AWS/Azure đều đã ra capability native trong 12 tháng qua.
3. **Đề xuất:** 12-tháng program qua 3 phase với 3 stage-gate. Adopt-first, Build-second. Showback trước, Chargeback sau.
4. **Investment:** ~$345K–590K Year-1 (chủ yếu FTE allocation); commercial tooling defer.
5. **Expected outcome:** Full visibility 6-layer; chargeback ≥ 90% prod; forecast MAPE ≤ 15%; pre-spend controls active; FinOps Council BAU.
6. **Risk #1:** Tagging discipline — đã có mitigation rõ ràng (enforce ở gateway + CI/CD).
7. **Đề nghị:** Approve G1–G3 gating; sponsor FinOps Council charter; hire 1.5–2 FTE FinOps.

## Nguồn

- FinOps Foundation. *State of FinOps 2025.* data.finops.org/2025-report/
- FinOps Foundation. *2025 FinOps Framework.* finops.org/insights/2025-finops-framework/
- FinOps Foundation. *FinOps for AI Overview.* finops.org/wg/finops-for-ai-overview/
- Microsoft Tech Community. *Managing the cost of AI: Leveraging the FinOps Framework.* techcommunity.microsoft.com/blog/finopsblog/4381666
- Databricks Blog. *Introducing AI spend controls with Unity AI Gateway.* databricks.com/blog/introducing-ai-spend-controls-unity-ai-gateway
- Capital One Software Blog. *Building a GenAI cost supervisor agent in Databricks.* capitalone.com/software/blog/databricks-genai-cost-supervisor-agent/
- AWS Blog. *Manage AI costs with Amazon Bedrock Projects.* aws.amazon.com/blogs/machine-learning/manage-ai-costs-with-amazon-bedrock-projects/
- Apptio. *FinOps for AI: Enabling the Next Wave of Cloud Innovation.* apptio.com/blog/finops-for-ai-enabling-the-next-wave-of-cloud-innovation/
- Finout. *FinOps in the Age of AI: A CPO's Guide.* finout.io/blog/finops-in-the-age-of-ai-a-cpos-guide-to-llm-workflows-rag-ai-agents-and-agentic-systems
