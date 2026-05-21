# Cẩm nang FinOps cho Agentic AI

### Phương pháp luận tính cost, forecasting và monitoring cho nền tảng Agent / Foundation Model nội bộ

[CẬP NHẬT: 21/05/2026]

---

> **Tóm lược (Pyramid):**
> Token cost chỉ chiếm 30–50% tổng chi phí vận hành một agent ở production [1][2]. Mọi cost model dừng ở "tokens × giá" sẽ bỏ sót 50–70% bức tranh thực — phần còn lại nằm ở orchestration overhead, retries, vector DB ops, và hidden cost (error remediation, human oversight). Cẩm nang này cung cấp **phương pháp luận chuẩn hoá** để: (1) **đo cost chính xác** theo 6 lớp MECE và đa modality; (2) **dự báo** scenario-based với 6 driver; (3) **giám sát** pre-spend qua 3 tầng kiểm soát. Đối tượng: Engineering Platform, FinOps, Finance/FP&A, và C-Level cần ROI rõ cho AI initiatives.

---

## MỤC LỤC

- [Tóm tắt điều hành (Executive Summary)](#tóm-tắt-điều-hành)
- [Hướng dẫn đọc](#hướng-dẫn-đọc)
- [Phần I — Bối cảnh thị trường & FinOps cho AI](#phần-i--bối-cảnh-thị-trường--finops-cho-ai)
  - I.1. Bối cảnh & Drivers thúc đẩy
  - I.2. Framework chuẩn ngành — FinOps Foundation 2025
  - I.3. Case study Fortune 500
  - I.4. Vendor & Partner Landscape
  - I.5. Khoảng trống ngành (Gap Analysis)
- [Phần II — Taxonomy cost & Công thức multimodal](#phần-ii--taxonomy-cost--công-thức-multimodal)
  - II.1. Khung 6 lớp (MECE)
  - II.2. Công thức cost tổng quát
  - II.3. Quy đổi đơn vị đa modality
  - II.4. Cost traps phổ biến
  - II.5. Instrumentation tối thiểu
  - II.6. Source-of-truth dữ liệu
- [Phần III — Rate Card per-modality](#phần-iii--rate-card-per-modality)
  - III.1. Text — input / output tokens
  - III.2. Image — input (vision) & output (generation)
  - III.3. Audio — STT & TTS
  - III.4. Video / Embedding / Finetune
  - III.5. Prompt caching mechanics
  - III.6. Ví dụ task multimodal — image-in / audio-out
  - III.7. Cách verify + refresh
- [Phần IV — Allocation & Unit Economics](#phần-iv--allocation--unit-economics)
  - IV.1. Showback vs Chargeback
  - IV.2. Allocation Model 3 tầng cost pool
  - IV.3. Tagging Taxonomy
  - IV.4. Unit Economics 3 cấp số
  - IV.5. Quy trình month-end
  - IV.6. Dispute process
- [Phần V — Forecasting (Driver + 3 Scenarios)](#phần-v--forecasting-driver--3-scenarios)
  - V.1. Vì sao point-estimate fail
  - V.2. Driver-based Forecast Model
  - V.3. Three-Scenario Framework
  - V.4. Sensitivity Analysis
  - V.5. Operating Cadence
  - V.6. KPI cho forecast
- [Phần VI — Monitoring & Governance](#phần-vi--monitoring--governance)
  - VI.1. Mô hình 3 tầng kiểm soát
  - VI.2. Pre-spend Controls
  - VI.3. Real-time Monitoring
  - VI.4. FinOps Council Operating Model
  - VI.5. Governance Artifacts
- [Phần VII — Tooling Assessment](#phần-vii--tooling-assessment)
  - VII.1. Build vs Buy Framework
  - VII.2. So sánh vendor
  - VII.3. Stack đề xuất
  - VII.4. TCO 12 tháng
  - VII.5. Trigger tái đánh giá
  - VII.6. Risk & Mitigation
- [Phần VIII — Implementation Roadmap (R&D → POC → Prod)](#phần-viii--implementation-roadmap-rd--poc--prod)
  - VIII.1. Phase 1 — R&D / Foundation (Tháng 1–3)
  - VIII.2. Phase 2 — POC / Pilot (Tháng 4–6)
  - VIII.3. Phase 3 — Production / Scale (Tháng 7–12)
  - VIII.4. Dependencies & Pre-requisites
  - VIII.5. Risk Register
  - VIII.6. Investment Summary
  - VIII.7. C-Level Talking Points
- [Phụ lục A — Bảng thuật ngữ (Glossary)](#phụ-lục-a--bảng-thuật-ngữ-glossary)
- [Phụ lục B — Bibliography tổng hợp](#phụ-lục-b--bibliography-tổng-hợp)
- [Phụ lục C — Lịch sử bản ghi](#phụ-lục-c--lịch-sử-bản-ghi)

---

## Tóm tắt điều hành

**Vấn đề.** AI spend đang tăng phi tuyến trong khi visibility, allocation và forecast vẫn ad-hoc. Risk: thất thoát budget + thiếu accountability cross-BU. 63% enterprise đã track AI spend (so với 31% năm trước), nhưng chỉ 14.2% đạt FinOps "Run" maturity [3][4].

**Benchmark ngành.** Databricks, AWS và Azure đều đã ra mắt capability native cho cost-attribution LLM/Agent workload trong 12 tháng qua. JPMorgan vận hành 450+ AI use cases trên budget công nghệ $18B/năm với LLM Suite + governance layer [5][6]. Capital One xây "GenAI Cost Supervisor Agent" trên Databricks với 20 Unity Catalog SQL functions [7].

**Đề xuất.** Triển khai chương trình 12 tháng qua 3 phase với 3 stage-gate:

| Phase | Tháng | Mục tiêu | Investment ước lượng |
|---|---|---|---|
| **R&D — Foundation** | 1–3 | Lock taxonomy, tagging policy, baseline data, tooling stack v1 | 1,5 FTE × 3 tháng |
| **POC — Pilot** | 4–6 | Apply methodology lên 2–3 use case; deploy budget + alert; showback report | 2 FTE × 3 tháng |
| **Production — Scale** | 7–12 | 100% prod use case lên chargeback; FinOps Council BAU | 2 FTE BAU |

**Outcome 12 tháng:**
- Cost visibility theo 6 lớp MECE cho mọi invocation;
- Chargeback chính xác cho ≥ 90% prod use case;
- Forecast MAPE month-ahead ≤ 15%;
- Pre-spend controls active (budget + policy guardrails ở AI Gateway);
- FinOps Council bi-weekly là cơ chế vận hành thường trực.

**Đầu tư.** ~$345K–590K Year-1 (chủ yếu FTE allocation); commercial tooling defer 12 tháng [ước lượng — cần verify với baseline cụ thể của tổ chức].

**Insight cốt lõi cần truyền đạt cho stakeholder:**

| Insight | Nguồn | Ý nghĩa |
|---|---|---|
| Token cost chỉ chiếm 30–50% TCO vận hành agent | Galileo, Acropolium [1][2] | Mọi cost model dừng ở tokens sẽ thiếu 50–70% bức tranh |
| Silent retries có thể nhân token usage 2–5× | Agents Arcade [8] | Observability ở orchestration layer là bắt buộc |
| Reflection loop không bounded có thể ×10–100 trong corner case | Galileo [1] | Policy guardrail (max reflection depth) là pre-spend control rẻ nhất |
| TTS / video dominate cost trong task multimodal (≥ 90% chuỗi) | (tính toán Phần III.6) | Modality budget policy là đòn bẩy lớn nhất cho audio/video use case |
| Vendor pricing frontier model giảm 30–50%/năm | Drivetrain [9] | Forecast lock giá cũ → overestimate; phải refresh tháng |

### Citations

- [1] Galileo. *The Hidden Costs of Agentic AI.* galileo.ai/blog/hidden-cost-of-agentic-ai (truy cập 21/05/2026)
- [2] Acropolium. *AI agent unit economics: TCO, ROI, payback.* acropolium.com/blog/ai-agent-unit-economics/ (truy cập 21/05/2026)
- [3] FinOps Foundation. *State of FinOps 2025 Report.* data.finops.org/2025-report/ (truy cập 21/05/2026)
- [4] FinOps Foundation. *FinOps for AI Overview.* finops.org/wg/finops-for-ai-overview/ (truy cập 21/05/2026)
- [5] artificialintelligence-news.com. *JPMorgan Chase AI strategy: US$18B bet paying off.* 2025
- [6] CNBC. *JPMorgan Chase's blueprint to become the world's first fully AI-powered megabank.* 30/09/2025
- [7] Capital One Software Blog. *Building a GenAI cost supervisor agent in Databricks.* capitalone.com/software/blog/databricks-genai-cost-supervisor-agent/ (truy cập 21/05/2026)
- [8] Agents Arcade. *Cost Modeling for Agentic Systems Before You Go to Production.* agentsarcade.com/blog/cost-modeling-agentic-systems-production (truy cập 21/05/2026)
- [9] Drivetrain. *Unit economics for AI SaaS companies: A survival guide for CFOs.* drivetrain.ai/post/unit-economics-of-ai-saas-companies-cfo-guide (truy cập 21/05/2026)

---

## Hướng dẫn đọc

| Vai trò | Lộ trình đề xuất |
|---|---|
| **C-Level / Board** | [Tóm tắt điều hành](#tóm-tắt-điều-hành) → Phần VIII.7 *C-Level Talking Points* |
| **CFO / FP&A** | [Tóm tắt điều hành](#tóm-tắt-điều-hành) → Phần V *Forecasting* → Phần IV *Allocation & Unit Economics* |
| **Head of AI Platform** | Phần II *Taxonomy* → Phần VI *Monitoring* → Phần VIII *Roadmap* |
| **Engineering / Platform team** | Phần II → Phần III *Rate Card* → Phần VI → Phần VII *Tooling* |
| **FinOps practitioner** | Toàn bộ, theo thứ tự |
| **Lần đầu tiếp cận** | [Tóm tắt điều hành](#tóm-tắt-điều-hành) → Phần I *Bối cảnh* → tuỳ vai trò |

**Quy ước trong cẩm nang:**

- Tiếng Việt có dấu chuẩn 100%; thuật ngữ kỹ thuật giữ tiếng Anh (tokens, retry, RAG, embedding, chargeback, ...).
- Số liệu chưa xác thực đánh dấu `[cần verify]`; ước lượng đánh dấu `[ước lượng]`.
- Citations dạng `[N]` tham chiếu khối *Citations* cuối mỗi section.
- Bibliography tổng hợp ở Phụ lục B.

---

## Phần I — Bối cảnh thị trường & FinOps cho AI

> **Kết luận đầu (Pyramid):** FinOps cho AI đã chuyển từ giai đoạn "experimental" sang "core operating discipline" trong 12 tháng qua. 63% enterprise đã track AI spend (so với 31% năm trước), FinOps Foundation đã ban hành certification riêng cho FinOps for AI, và 3 hyperscaler (AWS / Azure / Databricks) đều đã ra mắt native cost-attribution capability cho LLM/Agent workload trong 2024–2025. Tuy nhiên, **chỉ 14,2% organizations đạt "Run" maturity** — cửa sổ cơ hội để dẫn trước vẫn còn rộng [3][4].

### I.1. Bối cảnh & Drivers thúc đẩy

| Driver | Mô tả | Bằng chứng |
|---|---|---|
| **Bùng nổ AI spend** | "Managing AI/ML spend" tăng +4 hạng, "unit economics" tăng +5 hạng trong priority list của FinOps practitioners 2025 | FinOps Foundation, State of FinOps 2025 [3] |
| **Pricing model phức tạp** | Pay-per-token + pay-per-call + provisioned throughput + cached tokens + reserved capacity → khó dự báo | Vantage [10] |
| **Agentic workflow opacity** | Một user request kích hoạt 5–6 micro-calls ẩn (search, RAG, tool use, reflection) | Gravitee [11] |
| **Regulator & Board pressure** | Board yêu cầu ROI rõ ràng cho AI initiatives; CFO cần unit economics | FinOps Foundation 2025, Apptio [12] |
| **Vendor ecosystem chín muồi** | Databricks Unity AI Gateway, AWS Bedrock cost allocation, Azure FinOps blueprint đều ra mắt 2024–2025 | Databricks Blog 2025, AWS Blog 2025–2026 [13][14] |

#### Citations

- [3] FinOps Foundation. *State of FinOps 2025 Report.* data.finops.org/2025-report/ (truy cập 21/05/2026)
- [10] Vantage. *AI Cost Considerations Every Engineer Should Know.* vantage.sh/blog/ai-llm-pricing-dimensions (truy cập 21/05/2026)
- [11] Gravitee. *How to Control the Hidden Costs of Generative AI.* gravitee.io/blog/how-to-control-the-hidden-costs-of-generative-ai (truy cập 21/05/2026)
- [12] Apptio. *FinOps for AI: Enabling the Next Wave of Cloud Innovation.* apptio.com/blog/finops-for-ai-enabling-the-next-wave-of-cloud-innovation/ (truy cập 21/05/2026)
- [13] Databricks Blog. *Introducing AI spend controls with Unity AI Gateway.* databricks.com/blog/introducing-ai-spend-controls-unity-ai-gateway (truy cập 21/05/2026)
- [14] AWS Blog. *Track Amazon Bedrock Costs by Caller Identity with IAM Principal-Based Cost Allocation.* aws.amazon.com/about-aws/whats-new/2026/04/bedrock-iam-cost-allocation/ (truy cập 21/05/2026)

### I.2. Framework chuẩn ngành — FinOps Foundation 2025

FinOps Foundation 2025 framework tái cấu trúc gồm **3 trụ cột**: Domains (6 lĩnh vực), Capabilities (năng lực thực thi), **Scopes** (phạm vi áp dụng — mới). **Scope thứ 4 — AI** (tokens, models, GPU compute) chính thức là first-class citizen.

| Domain | Capability nổi bật liên quan AI |
|---|---|
| **Understand Usage & Cost** | Cost attribution theo agent / use case / IAM principal |
| **Quantify Business Value** | Unit economics cho AI; cost-per-business-outcome |
| **Manage Anomalies** | Anomaly detection trên token spike, retry storm |
| **Manage Commitments** | Provisioned throughput vs on-demand trade-off |
| **Plan & Forecast** | Driver-based forecasting cho AI workload |
| **Optimize Rate / Usage** | Model tiering, prompt compression, caching |

#### Citations

- [15] FinOps Foundation. *2025 FinOps Framework.* finops.org/insights/2025-finops-framework/ (truy cập 21/05/2026)

### I.3. Case study Fortune 500

| Tổ chức | Bài toán | Cách tiếp cận | Kết quả / Insight | Nguồn |
|---|---|---|---|---|
| **JPMorgan Chase** | Quản lý 450+ AI use cases trong production trên budget $18B công nghệ/năm | LLM Suite nội bộ + governance layer + deploy "agentic AI" cho complex multistep | Ước tính $1,5B value/năm; consumer banking ops giảm ≥ 10% headcount; COiN tiết kiệm 360K labor hours/năm | [5][6] |
| **Capital One (Slingshot)** | Visibility/optimization cho Databricks GenAI spend | Xây "GenAI Cost Supervisor Agent" trên Databricks, chạy 20 Unity Catalog SQL functions, mọi agent interaction traced trong `system.serving.endpoint_usage` | Mọi user có "personal GenAI cost analyst"; cost supervisor tự tracking cost của chính nó (recursive observability) | [7] |
| **Microsoft** | Quản lý cost cho Copilot ecosystem + Azure OpenAI tenancy | FinOps framework adoption + Azure Cost Management với AI tags; FinOps Hubs reference architecture | Public blueprint cho enterprise customer; áp dụng nội bộ trước khi ra ngoài | [16] |
| **Databricks** | Govern coding-agent sprawl (Cursor, Codex CLI, Gemini CLI dùng nội bộ) | Route mọi request qua Unity AI Gateway → one invoice, one dashboard, one set of rate limits | Single pane of glass cho coding-agent ROI tracking | [17] |
| **Apptio / IBM** | Chargeback cho GenAI spend tới business unit | Apptio TBM + cost-allocation tags ở model/inference layer | Reference patterns trong "FinOps for AI: Enabling the Next Wave" | [12] |

> **[cần verify]** Số liệu chi tiết cost-per-agent ở JPMorgan và Capital One chưa được công bố public; số trên là ROI / labor hour, không phải cost của chính agent platform.

#### Citations

- [5] artificialintelligence-news.com. *JPMorgan Chase AI strategy: US$18B bet paying off.* 2025
- [6] CNBC. *JPMorgan Chase's blueprint to become the world's first fully AI-powered megabank.* 30/09/2025
- [7] Capital One Software Blog. *Building a GenAI cost supervisor agent in Databricks.* capitalone.com/software/blog/databricks-genai-cost-supervisor-agent/ (truy cập 21/05/2026)
- [16] Microsoft Tech Community. *Managing the cost of AI: Leveraging the FinOps Framework.* techcommunity.microsoft.com/blog/finopsblog/4381666 (truy cập 21/05/2026)
- [17] Databricks Blog. *Governing Coding Agent Sprawl with Unity AI Gateway.* databricks.com/blog/governing-coding-agent-sprawl-databricks-ai-gateway (truy cập 21/05/2026)
- [12] Apptio. *FinOps for AI: Enabling the Next Wave of Cloud Innovation.* apptio.com/blog/finops-for-ai-enabling-the-next-wave-of-cloud-innovation/ (truy cập 21/05/2026)

### I.4. Vendor & Partner Landscape

**Native capabilities (hyperscaler):**

| Hyperscaler | Capability | Granularity | Trạng thái |
|---|---|---|---|
| **Databricks Unity AI Gateway** | Cost tracking + budget alerts theo user / use case / model / provider / tag; `system.serving.*` tables | Per-request, per-token, per-user, per-tag | GA + beta (usage-tracking-beta) [18] |
| **AWS Bedrock** | Application Inference Profiles + cost allocation tags + IAM principal-based allocation trong CUR 2.0 | Per-IAM principal, per-tag, per-project | GA (04/2026) [14] |
| **Azure OpenAI** | Azure Cost Management + resource tags + Azure FinOps Hubs | Per-resource, per-tag, per-deployment | GA [19] |

**Third-party tooling (Observability + FinOps):**

| Loại | Tool | Điểm mạnh | Điểm yếu | Free tier |
|---|---|---|---|---|
| LLM Observability — Cloud | **LangSmith** | Trace-level cost; tích hợp sâu LangChain; OTel support từ 03/2025 | Seat-based pricing; vendor lock | 5K traces/tháng |
| LLM Observability — OSS Cloud | **Langfuse** | Unit-based pricing; self-host được; multi-framework | UI ít polish hơn LangSmith ở enterprise | 50K events/tháng |
| LLM Observability — Proxy | **Helicone** | Đổi base URL là xong; cost dashboard nhanh | Proxy → thêm network hop | 10K requests/tháng |
| Observability — OTel native | **OpenLLMetry** | Pure OpenTelemetry; plug vào Datadog/New Relic/Grafana | Không có dashboard riêng | OSS |
| FinOps platform | **Finout / nOps / Vantage / Apptio** | Cross-cloud cost view, chargeback engine, anomaly detection | Chưa cover sâu agent/orchestration layer | Trial |
| Partner trên Databricks | **Capital One Slingshot** | Cost allocation + optimization riêng Databricks | Hẹp Databricks ecosystem | Commercial |

#### Citations

- [14] AWS Blog. *Track Amazon Bedrock Costs by Caller Identity with IAM Principal-Based Cost Allocation.* aws.amazon.com/about-aws/whats-new/2026/04/bedrock-iam-cost-allocation/ (truy cập 21/05/2026)
- [18] Databricks Docs. *Monitor usage for Unity AI Gateway endpoints.* docs.databricks.com/aws/en/ai-gateway/usage-tracking-beta (truy cập 21/05/2026)
- [19] Microsoft Learn. *Azure FinOps toolkit.* learn.microsoft.com Azure FinOps (truy cập 21/05/2026)
- [20] ZenML Blog. *10 Best LLM Monitoring Tools to Use in 2025.* zenml.io/blog/best-llm-monitoring-tools (truy cập 21/05/2026)

### I.5. Khoảng trống ngành (Gap Analysis)

| Gap | Hàm ý cho platform nội bộ |
|---|---|
| **Orchestration-layer cost vẫn opaque** với hầu hết tooling — chỉ trace LLM call, không trace tool call / retry / reflection | Phải tự instrument ở orchestrator (LangChain / LangGraph / custom) bằng OTel hoặc tương đương |
| **Cost-per-business-outcome** (cost per resolved ticket, per BRD generated) **chưa có chuẩn** | Định nghĩa "business unit of value" cho từng use case là việc của tổ chức, không vendor làm hộ |
| **Forecasting AI workload chính xác là khó** — usage tăng phi tuyến khi user adopt | Phải sống chung với scenario modeling; không kỳ vọng forecast accuracy > 80% trong 6 tháng đầu |
| **Tagging discipline thiếu** — chỉ 14,2% tổ chức "Run" maturity | Tagging policy + enforcement phải làm sớm; chậm là phải migrate đau đớn |

#### Citations

- [4] FinOps Foundation. *FinOps for AI Overview.* finops.org/wg/finops-for-ai-overview/ (truy cập 21/05/2026)
- [21] Portkey.ai. *The State of AI FinOps 2025.* portkey.ai/blog/the-state-of-ai-finops-2025-key-insights-from-finops-foundations-latest-report/ (truy cập 21/05/2026)

---

## Phần II — Taxonomy cost & Công thức multimodal

> **Insight cốt lõi:** Token cost chỉ là 30–50% tổng cost vận hành một agent ở production [1][2]. Khung 6 lớp dưới đây MECE-fy chi phí — mỗi lớp có driver định lượng được, mapping được tới billing source.

### II.1. Khung 6 lớp (MECE)

```
┌─────────────────────────────────────────────────────────────┐
│  L6. HIDDEN COST  (oversight, remediation, opportunity)     │
├─────────────────────────────────────────────────────────────┤
│  L5. OPS  (logging, security, FinOps tooling, headcount)    │
├─────────────────────────────────────────────────────────────┤
│  L4. DATA  (vector DB, embeddings, knowledge base, egress)  │
├─────────────────────────────────────────────────────────────┤
│  L3. ORCHESTRATION  (agent framework, tool calls, retries)  │
├─────────────────────────────────────────────────────────────┤
│  L2. COMPUTE  (GPU, CPU serving, networking, storage)       │
├─────────────────────────────────────────────────────────────┤
│  L1. MODEL  (multi-modality units × rate card)              │
└─────────────────────────────────────────────────────────────┘
```

| # | Lớp | Cost drivers chính | Unit | Nguồn billing | Visibility |
|---|---|---|---|---|---|
| **L1** | Model | input/output tokens (mọi modality), cached tokens, finetune tokens, function-call overhead | $/1M tokens, $/image, $/min audio, $/char, $/sec video | `system.serving.endpoint_usage`; provider invoices | **Cao** |
| **L2** | Compute | GPU-hours (provisioned), CPU serving, autoscale overhead, egress, model artifact storage | $/GPU-hour, $/DBU, $/GB egress | `system.billing.usage`; cloud CUR | **Cao** |
| **L3** | Orchestration | Tool calls, retries (silent + explicit), reflection loops, planner/critic, multi-agent coord | calls/req, retries/req, planner_tokens/req | OTel traces; `system.serving.payload`; Langfuse | **Thấp–Trung** |
| **L4** | Data | Vector DB storage + read amp, embedding gen, doc ingestion, RAG query, egress sang model | $/GB-month, $/embed token, $/query | Vector Search billing; provider logs | **Trung** |
| **L5** | Ops | Logging/observability tools, security scanning, FinOps platform license, platform team FTE | $/month, FTE allocated | Procurement + HR allocation | **Trung** |
| **L6** | Hidden | Error remediation (HITL), false-positive review, retry-due-to-bad-output, user wait opportunity cost, audit overhead | $/error, hours/incident | Workforce + ticketing join | **Rất thấp** |

#### Citations

- [1] Galileo. *The Hidden Costs of Agentic AI.* galileo.ai/blog/hidden-cost-of-agentic-ai (truy cập 21/05/2026)
- [2] Acropolium. *AI agent unit economics: TCO, ROI, payback.* acropolium.com/blog/ai-agent-unit-economics/ (truy cập 21/05/2026)
- [22] FinOps Foundation. *Cost Estimation of AI Workloads.* finops.org/wg/cost-estimation-of-ai-workloads/ (truy cập 21/05/2026)

### II.2. Công thức cost tổng quát

**Multimodal-ready formula** cho một agent invocation:

```
cost_invocation =
    Σ_modality ( units[m] × price[m] × (1 + retry_mult) )       # L1 (mọi modality)
  + Σ          ( cached_tokens × price_cached )                  # L1 caching
  + (gpu_hours × price_gpu) / invocations_per_hour               # L2 amortized
  + Σ tool_calls × avg_tool_cost                                 # L3
  + Σ rag_queries × avg_rag_cost + embedding_units × price_emb   # L4
  + ops_overhead_per_invocation                                  # L5 amortized
  + P(error) × cost_per_remediation                              # L6 probabilistic

modality m ∈ { text_in, text_out, image_in, image_out,
               audio_in_sec, audio_out_char, video_sec,
               embedding, finetune_token }

retry_mult: 2–5× (unguarded) → 1.1–1.3× (caching + guardrail) [8]
```

**Lưu ý phương pháp luận:**
- Trong giai đoạn pay-per-token endpoint, L2 đã include trong L1 → không double-count.
- L3 retry không đếm 2 lần: chỉ ghi nhận qua `retry_mult`, không cộng lại ở L1.
- L6 là probabilistic — không attribute tới invocation đơn lẻ mà allocate qua statistical model.

#### Citations

- [2] Acropolium. *AI agent unit economics: TCO, ROI, payback.* acropolium.com/blog/ai-agent-unit-economics/ (truy cập 21/05/2026)
- [8] Agents Arcade. *Cost Modeling for Agentic Systems Before You Go to Production.* agentsarcade.com/blog/cost-modeling-agentic-systems-production (truy cập 21/05/2026)
- [22] FinOps Foundation. *Cost Estimation of AI Workloads.* finops.org/wg/cost-estimation-of-ai-workloads/ (truy cập 21/05/2026)

### II.3. Quy đổi đơn vị đa modality

Khi 1 invocation chạy qua **nhiều modality**, không thể gộp về 1 unit duy nhất:

```
invocation_L1_cost = (
    text_in_tokens     × price[text_in, model_A]
  + text_out_tokens    × price[text_out, model_A]
  + image_in_count     × price[image_in, model_A]      # hoặc token-converted
  + image_out_count    × price[image_out, model_B]
  + audio_in_minutes   × price[audio_in_sec, model_C]
  + audio_out_chars    × price[audio_out_char, model_D]
  + video_in_seconds   × price[video_sec, model_E]
  + embedding_tokens   × price[embedding, model_F]
)
```

**Quy tắc instrumentation:**

| Quy tắc | Lý do |
|---|---|
| Lưu **raw count theo từng modality** (không pre-convert sang $) | Giá đổi liên tục; preserve count để recompute lịch sử |
| Lưu **`model_id` cùng count** | Cùng 1 modality, model khác giá khác (vd. Sonnet vs Haiku) |
| Lưu `provider_call_id` (vendor reference) | Reconcile với invoice cuối tháng |
| Token-converted modality (vd. image-on-Anthropic) | Lưu **cả raw image count VÀ converted_tokens** để biết quy đổi |

#### Citations

- [23] Anthropic. *Vision pricing documentation.* docs.anthropic.com/en/docs/build-with-claude/vision (truy cập 21/05/2026)
- [24] OpenAI. *Vision guide.* platform.openai.com/docs/guides/vision (truy cập 21/05/2026)

### II.4. Cost traps phổ biến

| Trap | Cơ chế | Impact | Nguồn | Detection |
|---|---|---|---|---|
| **Silent retries** | Network/rate-limit/tool fail → auto-retry, full inference mỗi lần | Token ×2–5 không tăng traffic | [8] | Alert `retry_rate > 25%` (VI.3) |
| **Reflection loop unbounded** | Self-critique không max-depth → token explode | ×10–100 cho corner case | [1] | Policy guardrail max reflection = 3 |
| **RAG amplification** | 10-step chain × query/embed mỗi step → ×10 | Có thể vượt LLM cost | [25] | Track `rag_queries/inv`; baseline + alert |
| **Over-provisioned GPU** | Provisioned throughput đặt theo peak, không autoscale | 30–50% wasted | [26] | Utilization < 60% trong 14 ngày → flag |
| **Over-logging payload** | Lưu full req/resp mọi call | Storage ×2–10 + PII risk | [10] | Sample rate policy ở L5 |
| **Wrong-tier model** | Frontier model cho task đơn giản | 5–50× differential | [27] | Model-tier policy ở Gateway (VI.2) |
| **TTS dominance ở multimodal** | Chuỗi image-in→text→audio-out: audio-out ≥ 90% cost | Optimize sai chỗ | (Phần III.6) | Track cost-per-modality breakdown |
| **Forgotten finetune** | Orphan finetune lab vẫn billing | Range rộng | [3] | Quarterly audit |

#### Citations

- [1] Galileo. *The Hidden Costs of Agentic AI.* galileo.ai/blog/hidden-cost-of-agentic-ai (truy cập 21/05/2026)
- [3] FinOps Foundation. *State of FinOps 2025 Report.* data.finops.org/2025-report/ (truy cập 21/05/2026)
- [8] Agents Arcade. *Cost Modeling for Agentic Systems Before You Go to Production.* agentsarcade.com/blog/cost-modeling-agentic-systems-production (truy cập 21/05/2026)
- [10] Vantage. *AI Cost Considerations Every Engineer Should Know.* vantage.sh/blog/ai-llm-pricing-dimensions (truy cập 21/05/2026)
- [25] Subhojyoti Singha. *The Hidden Cost of Embeddings.* Medium, 2025
- [26] Cogent. *AI-Native FinOps: Controlling GPU and LLM Cloud Costs.* cogentinfo.com/resources/ai-native-finops-controlling-gpu-and-llm-cloud-costs (truy cập 21/05/2026)
- [27] Clarifai. *AI Cost Controls: Budgets, Throttling & Model Tiering.* clarifai.com/blog/ai-cost-controls (truy cập 21/05/2026)

### II.5. Instrumentation tối thiểu

Mỗi agent invocation phải emit (qua OpenTelemetry hoặc tương đương):

| Attribute | Bắt buộc | Mục đích | Lớp |
|---|:---:|---|:---:|
| `trace_id` | ✅ | Join cross-system | All |
| `agent_id` / `agent_version` | ✅ | Attribution | All |
| `use_case_id` | ✅ | Allocation tới BU | All |
| `user_id` / `caller_principal` | ✅ | Chargeback per user | All |
| `cost_center_tag` | ✅ | Finance attribution | All |
| `model_id` / `provider` | ✅ | Per-model cost | L1 |
| `tokens_in` / `tokens_out` / `cached_tokens` | ✅ | Text cost | L1 |
| `image_in_count` / `image_in_tokens_converted` | ✅ (nếu vision) | Image cost | L1 |
| `image_out_count` / `quality_tier` | ✅ (nếu gen) | Image-gen cost | L1 |
| `audio_in_seconds` / `audio_out_chars` | ✅ (nếu audio) | STT/TTS cost | L1 |
| `video_in_seconds` / `video_out_seconds` | ✅ (nếu video) | Video cost | L1 |
| `embedding_tokens` / `embedding_model` | ✅ (nếu RAG) | Embedding cost | L1/L4 |
| `tool_name` / `tool_calls_count` | ✅ | Orchestration | L3 |
| `retry_count` / `retry_reason` | ✅ | Cost trap detection | L3 |
| `rag_queries_count` / `vectors_read` | ⚪ recommend | Data cost | L4 |
| `outcome_status` (success/partial/fail) | ⚪ recommend | Unit econ + L6 | All |

#### Citations

- [28] OpenTelemetry. *Semantic Conventions for Generative AI.* opentelemetry.io/docs/specs/semconv/gen-ai/ (truy cập 21/05/2026)
- [29] Databricks Blog. *Tracing for Generative AI on Databricks.* databricks.com/blog/tracing-generative-ai-databricks (truy cập 21/05/2026)

### II.6. Source-of-truth dữ liệu (Databricks-first)

| Lớp | Source chính | Source phụ |
|---|---|---|
| L1 Model | `system.serving.endpoint_usage` | Provider invoices nếu đi direct (OpenAI/Anthropic) |
| L2 Compute | `system.billing.usage` (DBU) | Cloud CUR (AWS/Azure) |
| L3 Orchestration | OTel traces → Langfuse → join L1 theo `trace_id` | Custom span attributes |
| L4 Data | `system.billing.usage` Vector Search; Pinecone/Weaviate invoices | Embedding API logs |
| L5 Ops | Procurement; HR cost center allocation | Tooling invoices export |
| L6 Hidden | Workforce + ticketing join theo `trace_id` | Manual quarterly update |

#### Citations

- [18] Databricks Docs. *Monitor usage for Unity AI Gateway endpoints.* docs.databricks.com/aws/en/ai-gateway/usage-tracking-beta (truy cập 21/05/2026)
- [30] Databricks Docs. *System tables billable usage.* docs.databricks.com/aws/en/admin/system-tables/billing (truy cập 21/05/2026)

---

## Phần III — Rate Card per-modality

> **Snapshot:** 21/05/2026 · **Refresh cadence:** Tháng (hoặc trong 14 ngày sau khi vendor thông báo đổi giá — Phần V.5).
> **⚠️ Disclaimer:** Giá list-price công khai, **chưa áp enterprise discount / committed-use / Bedrock-Azure markup**. Finance/Procurement phải verify trước khi đưa vào chargeback. Mỗi dòng có cột **Verify-by** trỏ tới nguồn chính thức.

### III.1. Text — input / output tokens

Đơn vị: **USD per 1M tokens**. Cache-read & cache-write tách riêng (xem III.5).

| Provider | Model | Input | Output | Cache-read | Notes | Verify-by |
|---|---|---:|---:|---:|---|---|
| Anthropic | Claude Opus 4.x | 15.00 | 75.00 | 1.50 | Frontier reasoning | anthropic.com/pricing |
| Anthropic | Claude Sonnet 4.5/4.6 | 3.00 | 15.00 | 0.30 | Default tier | anthropic.com/pricing |
| Anthropic | Claude Haiku 4.5 | 1.00 | 5.00 | 0.10 | Small/cheap tier | anthropic.com/pricing |
| OpenAI | GPT-5 | 1.25 | 10.00 | 0.13 | Frontier | openai.com/pricing |
| OpenAI | GPT-5-mini | 0.25 | 2.00 | 0.025 | Mid tier | openai.com/pricing |
| OpenAI | GPT-4o | 2.50 | 10.00 | 1.25 | Multimodal native | openai.com/pricing |
| OpenAI | GPT-4o-mini | 0.15 | 0.60 | 0.075 | Cheap multimodal | openai.com/pricing |
| Google | Gemini 2.5 Pro | 1.25 / 2.50 | 10.00 / 15.00 | 0.31 | Hai bậc giá ≤ 200K / > 200K ctx | ai.google.dev/pricing |
| Google | Gemini 2.5 Flash | 0.30 | 2.50 | 0.075 | Default tier | ai.google.dev/pricing |
| Google | Gemini 2.5 Flash-Lite | 0.10 | 0.40 | 0.025 | Cheap tier | ai.google.dev/pricing |
| AWS Bedrock | Nova Pro | 0.80 | 3.20 | 0.20 | AWS-native | aws.amazon.com/bedrock/pricing |
| AWS Bedrock | Nova Lite | 0.06 | 0.24 | 0.015 | AWS-native cheap | aws.amazon.com/bedrock/pricing |
| AWS Bedrock | Claude/* via Bedrock | = Anthropic | = Anthropic | = Anthropic | Bedrock markup ~ 0% với Claude | aws.amazon.com/bedrock/pricing |
| Databricks | Pay-per-token Llama-3.3-70B | ~ 1.00 | ~ 3.00 | n/a | Trả bằng DBU; tỷ giá DBU theo workspace tier | docs.databricks.com/foundation-model-apis |
| Databricks | Pay-per-token Claude / GPT (via AI Gateway) | = provider | = provider | = provider | Pass-through; markup tính ở DBU serving | docs.databricks.com/ai-gateway |
| Databricks | Provisioned throughput (own model) | — | — | — | Tính theo `$/DBU-hour × hours` thay vì tokens | docs.databricks.com/model-serving |

> **Bedrock / Azure markup:** Bedrock pass-through ~ 0% với Claude, **Azure OpenAI ~ = OpenAI list** nhưng có committed-use discount riêng. Verify mỗi quarter.

#### Citations

- [31] Anthropic. *Pricing.* anthropic.com/pricing (truy cập 21/05/2026)
- [32] OpenAI. *API Pricing.* openai.com/api/pricing (truy cập 21/05/2026)
- [33] Google. *Gemini API Pricing.* ai.google.dev/pricing (truy cập 21/05/2026)
- [34] AWS. *Amazon Bedrock Pricing.* aws.amazon.com/bedrock/pricing (truy cập 21/05/2026)
- [35] Databricks. *Foundation Model APIs.* docs.databricks.com/aws/en/machine-learning/foundation-model-apis/ (truy cập 21/05/2026)

### III.2. Image — input (vision) & output (generation)

**Image INPUT (vision understanding) — mỗi vendor đo khác đơn vị:**

| Provider | Model | Đơn vị tính | Quy đổi gần đúng | Verify-by |
|---|---|---|---|---|
| Anthropic | Claude (any 4.x) | Token-converted: `tokens ≈ (W × H) / 750` | 1MP image ≈ 1334 tokens → ≈ $0.004 (Sonnet) | docs.anthropic.com/vision |
| OpenAI | GPT-4o / GPT-5 vision | Tile-based: 85 base + 170 × tiles | 1MP "high detail" ≈ 765 input tokens → ≈ $0.002 (GPT-4o) | platform.openai.com/docs/guides/vision |
| Google | Gemini 2.5 | Token-converted: 258 tokens/image (≤ 384×384) | ≈ $0.0003 (Flash) per image | ai.google.dev/vision |
| AWS Bedrock | Nova Pro vision | Image counted as input tokens; ~ 1280 tokens/image | ≈ $0.001 per image | aws.amazon.com/bedrock/pricing |
| OpenAI | gpt-image-1 (input cho edit) | $10 per 1M image tokens | — | openai.com/pricing |

**Image OUTPUT (generation) — đơn vị USD per image:**

| Provider | Model | Resolution / Quality | $/image | Verify-by |
|---|---|---|---:|---|
| OpenAI | gpt-image-1 | low / 1024² | 0.011 | openai.com/pricing |
| OpenAI | gpt-image-1 | medium / 1024² | 0.042 | openai.com/pricing |
| OpenAI | gpt-image-1 | high / 1024² | 0.167 | openai.com/pricing |
| OpenAI | DALL·E 3 | standard 1024² | 0.040 | openai.com/pricing |
| OpenAI | DALL·E 3 | HD 1024² | 0.080 | openai.com/pricing |
| Google | Imagen 4 Standard | 1024² | 0.040 | ai.google.dev/pricing |
| Google | Imagen 4 Ultra | 1024² | 0.060 | ai.google.dev/pricing |
| AWS Bedrock | Nova Canvas | 1024² standard | 0.040 | aws.amazon.com/bedrock/pricing |
| Stability AI | SD3.5 Large (via Bedrock) | 1024² | 0.065 | aws.amazon.com/bedrock/pricing |

#### Citations

- [23] Anthropic. *Vision pricing documentation.* docs.anthropic.com/en/docs/build-with-claude/vision (truy cập 21/05/2026)
- [24] OpenAI. *Vision guide.* platform.openai.com/docs/guides/vision (truy cập 21/05/2026)
- [36] Google. *Gemini vision pricing.* ai.google.dev/gemini-api/docs/vision (truy cập 21/05/2026)

### III.3. Audio — STT & TTS

**Audio INPUT — Speech-to-Text:**

| Provider | Model | Đơn vị | Giá | Verify-by |
|---|---|---|---:|---|
| OpenAI | Whisper-1 | $/minute audio | 0.006 | openai.com/pricing |
| OpenAI | gpt-4o-transcribe | $/minute audio | 0.006 | openai.com/pricing |
| OpenAI | GPT-4o realtime audio (input) | $/1M audio tokens | 40.00 | openai.com/pricing |
| Google | Gemini audio input | $/1M audio tokens | 1.00 (Flash) / 3.00 (Pro) | ai.google.dev/pricing |
| Deepgram | Nova-3 | $/minute audio | 0.0043 | deepgram.com/pricing |
| AssemblyAI | Universal-2 | $/hour audio | 0.27 (≈ $0.0045/min) | assemblyai.com/pricing |
| AWS | Transcribe (standard) | $/minute audio | 0.024 | aws.amazon.com/transcribe/pricing |

**Audio OUTPUT — Text-to-Speech (đơn vị khác nhau — cost trap phổ biến):**

| Provider | Model | Đơn vị | Giá | Notes | Verify-by |
|---|---|---|---:|---|---|
| OpenAI | tts-1 | $/1M characters | 15.00 | Standard quality | openai.com/pricing |
| OpenAI | tts-1-hd | $/1M characters | 30.00 | HD quality | openai.com/pricing |
| OpenAI | GPT-4o realtime audio (output) | $/1M audio tokens | 80.00 | Streaming voice | openai.com/pricing |
| ElevenLabs | Multilingual v2 | $/1K characters | ~ 0.30 (subscription-blended) | PAYG cao hơn | elevenlabs.io/pricing |
| Google | Gemini 2.5 native audio (output) | $/1M audio tokens | 20.00 | — | ai.google.dev/pricing |
| AWS | Polly Neural | $/1M characters | 16.00 | — | aws.amazon.com/polly/pricing |
| AWS | Polly Generative | $/1M characters | 30.00 | — | aws.amazon.com/polly/pricing |
| Azure | TTS Neural | $/1M characters | 16.00 | — | azure.microsoft.com/speech-services |

> **⚠️ Cẩn thận đơn vị:** ElevenLabs niêm yết per-1K characters (×1000 để so OpenAI), một số vendor per-1M characters. Allocation engine phải normalize tất cả về **$/1M characters**.

#### Citations

- [32] OpenAI. *API Pricing.* openai.com/api/pricing (truy cập 21/05/2026)
- [37] ElevenLabs. *Pricing.* elevenlabs.io/pricing (truy cập 21/05/2026)
- [38] Deepgram. *Pricing.* deepgram.com/pricing (truy cập 21/05/2026)
- [39] AssemblyAI. *Pricing.* assemblyai.com/pricing (truy cập 21/05/2026)
- [40] AWS. *Amazon Polly Pricing.* aws.amazon.com/polly/pricing (truy cập 21/05/2026)

### III.4. Video / Embedding / Finetune

**Video:**

| Provider | Model | Đơn vị | Giá | Verify-by |
|---|---|---|---:|---|
| Google | Veo 3 | $/second generated | 0.50 | ai.google.dev/pricing |
| Google | Veo 2 | $/second generated | 0.35 | ai.google.dev/pricing |
| OpenAI | Sora | $/second generated | 0.50–1.00 (theo resolution) | openai.com/pricing |
| Google | Gemini video INPUT | $/second video | ~ 0.001 (Flash) / 0.007 (Pro) | ai.google.dev/pricing |
| Twelve Labs | Marengo (video understanding) | $/minute indexed | varies | twelvelabs.io/pricing |

**Embeddings:**

| Provider | Model | Đơn vị | Giá | Verify-by |
|---|---|---|---:|---|
| OpenAI | text-embedding-3-large | $/1M tokens | 0.13 | openai.com/pricing |
| OpenAI | text-embedding-3-small | $/1M tokens | 0.02 | openai.com/pricing |
| Google | gemini-embedding-001 | $/1M tokens | 0.15 | ai.google.dev/pricing |
| Anthropic | (no native; dùng Voyage AI) | $/1M tokens | 0.05–0.12 | voyageai.com/pricing |
| Cohere | embed-v4 | $/1M tokens | 0.12 | cohere.com/pricing |
| AWS | Titan Embeddings v2 | $/1M tokens | 0.02 | aws.amazon.com/bedrock/pricing |
| Databricks | Vector Search storage | $/GB-month | varies (DBU-based) | docs.databricks.com/vector-search |

**Finetune:**

| Provider | Model | Train | Inference markup | Verify-by |
|---|---|---|---|---|
| OpenAI | GPT-4o finetune | $25 / 1M training tokens | 2× base inference | openai.com/pricing |
| OpenAI | GPT-4o-mini finetune | $3 / 1M training tokens | 2× base | openai.com/pricing |
| Anthropic | Claude (on Bedrock) | $/hour custom training | varies | aws.amazon.com/bedrock/pricing |
| Google | Gemini Flash tuning | Free training, pay inference | inference = base × tier | ai.google.dev/pricing |
| Databricks | Foundation Model Fine-tuning | $/M training tokens (DBU) | depends on model | docs.databricks.com/foundation-model-training |

#### Citations

- [33] Google. *Gemini API Pricing.* ai.google.dev/pricing (truy cập 21/05/2026)
- [41] Voyage AI. *Pricing.* voyageai.com/pricing (truy cập 21/05/2026)
- [42] Cohere. *Pricing.* cohere.com/pricing (truy cập 21/05/2026)
- [43] Twelve Labs. *Pricing.* twelvelabs.io/pricing (truy cập 21/05/2026)

### III.5. Prompt caching mechanics

Caching giảm cost lớn (5–10×) **nếu** repeat patterns đủ dài:

| Provider | Cơ chế | Write cost | Read cost | TTL | Min size để cache |
|---|---|---|---|---|---|
| Anthropic | Explicit `cache_control` block | +25% base input | 10% base input | 5 phút (default) / 1 giờ (extended) | 1024 tokens (Sonnet/Opus), 2048 (Haiku) |
| OpenAI | Auto (prefix-match) | 0 (cùng giá base) | 50% base input (GPT-4o), 10% (GPT-5) | ~ 5–10 phút | 1024 tokens |
| Google | Explicit context cache | Storage fee $1/1M tokens/hour | 25% base input | User-controlled | 4096 tokens |
| AWS Bedrock | Pass-through Anthropic / Nova | = provider | = provider | = provider | = provider |

> **Hàm ý:** OpenAI auto-cache "miễn phí write" nhưng savings thấp hơn (50% vs 90%). Anthropic explicit cache cần engineering, nhưng savings cao hơn cho long system prompt + RAG context.

#### Citations

- [44] Anthropic. *Prompt caching documentation.* docs.anthropic.com/en/docs/build-with-claude/prompt-caching (truy cập 21/05/2026)
- [45] OpenAI. *Prompt caching documentation.* platform.openai.com/docs/guides/prompt-caching (truy cập 21/05/2026)
- [46] Google. *Context caching guide.* ai.google.dev/gemini-api/docs/caching (truy cập 21/05/2026)

### III.6. Ví dụ task multimodal — image-in / audio-out

**Use case:** Agent nhận ảnh sản phẩm (1 ảnh 1MP) → phân tích → trả mô tả bằng giọng nói TTS (300 từ ≈ 1500 ký tự).

**Stack option A — OpenAI all-in-one:**

| Bước | Đơn vị tính | Lượng | Đơn giá | Cost |
|---|---|---|---|---|
| Vision input (GPT-4o) | tokens | 765 | $2.50/M | $0.0019 |
| Text output (mô tả) | tokens | 200 | $10.00/M | $0.0020 |
| TTS (tts-1-hd) | characters | 1500 | $30.00/M | $0.0450 |
| **Total** | | | | **~ $0.049** |

**Stack option B — Multi-vendor optimal cost:**

| Bước | Provider / Model | Lượng | Đơn giá | Cost |
|---|---|---|---|---|
| Vision input | Gemini 2.5 Flash | 258 tokens | $0.30/M | $0.00008 |
| Text output | Gemini 2.5 Flash | 200 tokens | $2.50/M | $0.0005 |
| TTS | ElevenLabs (subscription blended) | 1500 chars | $0.30/1K | $0.45 |
| **Total** | | | | **~ $0.45** |

**Insight rút ra:**

1. **TTS thường là bước đắt nhất** trong chuỗi multimodal — dominate ≥ 90% cost. Khi optimize, target TTS trước (tier-selection / caching audio fragments / Polly Neural thay vì Generative).
2. **Naive comparison "$/image" hoặc "$/character" cross-vendor có thể sai.** ElevenLabs có giọng tốt hơn → có thể justify cost; nhưng nếu use case không cần voice quality cao, Polly Neural ($16/1M) rẻ hơn ElevenLabs (~ $300/1M PAYG) gần 20×.
3. **Cấu trúc rate card phải tách per-modality riêng** — không thể blend thành "$ per request" trung bình mà giữ được signal.

**Công thức tổng quát:**

```python
cost_image_in_audio_out = (
    image_in_tokens   × price_per_token[model]
  + text_out_tokens   × price_per_output_token[model]
  + tts_characters    × price_per_char[tts_model]
) × (1 + retry_multiplier)
+ L2..L6_overhead   # xem Phần II.2
```

#### Citations

- [31] Anthropic. *Pricing.* anthropic.com/pricing (truy cập 21/05/2026)
- [32] OpenAI. *API Pricing.* openai.com/api/pricing (truy cập 21/05/2026)
- [37] ElevenLabs. *Pricing.* elevenlabs.io/pricing (truy cập 21/05/2026)

### III.7. Cách verify + refresh

| Khi | Hành động | Owner |
|---|---|---|
| Hàng tháng | Scrape pricing pages top-5 provider; diff với rate card; commit changes với date-stamp | FinOps + Platform |
| Vendor đổi giá | Update trong 14 ngày (Phần V.5) | FinOps |
| Trước chargeback go-live | Cross-check với invoice 2 tháng gần nhất; flag delta > 5% | Finance |
| Hàng quý | Recompute blended `$_per_unit` theo model mix thực tế cho forecast | FinOps |

**Field cần track khi update:**

```yaml
- provider: <Anthropic/OpenAI/Google/AWS/Azure/Databricks/...>
  model: <model-id>
  modality: <text_in|text_out|image_in|image_out|audio_in_sec|audio_out_char|video_sec|embedding|finetune>
  unit: <usd_per_M_tokens|usd_per_image|usd_per_minute|usd_per_M_characters|usd_per_second>
  price: <number>
  effective_date: YYYY-MM-DD
  source_url: <link>
  notes: <free text>
```

#### Citations

- [3] FinOps Foundation. *State of FinOps 2025 Report.* data.finops.org/2025-report/ (truy cập 21/05/2026)

---

## Phần IV — Allocation & Unit Economics

> **Kết luận đầu:** Không thể có chargeback / showback chính xác nếu không có **tagging discipline được enforce ngay tại điểm tạo resource và điểm gọi API**. Bắt đầu **Showback 6–9 tháng**, chuyển dần sang **Chargeback** cho use case đã ổn định.

### IV.1. Showback vs Chargeback

| Tiêu chí | Showback | Chargeback |
|---|---|---|
| **Bản chất** | Hiển thị cost cho team, không bill | Bill thật vào P&L team |
| **Mục đích chính** | Tăng visibility, tạo awareness | Buộc accountability, đẩy hành vi tối ưu |
| **Yêu cầu maturity** | Trung — chỉ cần tagging + dashboard | Cao — cần Finance buy-in + dispute process |
| **Rủi ro** | Có thể bị bỏ qua nếu không có incentive | Chargeback sai gây dispute, lost trust |
| **Phù hợp giai đoạn** | **R&D + POC** | **Production sau 6–12 tháng baseline showback** |
| **Trigger chuyển sang chargeback** | Cost > 0,5% OpEx của BU (rule of thumb) HOẶC tagging accuracy > 95% qua 2 quý liên tiếp | — |

#### Citations

- [47] Logiciel.io. *Showback or Chargeback: What's Working for Engineering Accountability.* logiciel.io/blog/showback-chargeback-engineering-accountability (truy cập 21/05/2026)
- [48] Portkey. *FinOps chargeback for GenAI platforms.* portkey.ai/blog/finops-chargeback-for-genai/ (truy cập 21/05/2026)
- [49] Finout. *GenAI Cost Allocation: The Complete Guide.* finout.io/blog/genai-cost-allocation-the-complete-guide (truy cập 21/05/2026)

### IV.2. Allocation Model — 3 tầng cost pool

| Tầng | Bản chất | Ví dụ | Cách allocate |
|---|---|---|---|
| **Direct cost** | Attribute trực tiếp tới 1 use case / 1 invocation | Token cost của 1 request, vector DB query của RAG step | Theo `use_case_id` / `agent_id` tag — 100% phân bổ |
| **Shared cost** | Dùng chung cho nhiều use case, chia theo driver | Provisioned throughput GPU phục vụ N agents, vector DB cluster | Theo driver tỷ lệ (% tokens consumed, % queries) — cập nhật rolling 30 ngày |
| **Overhead** | Platform-wide, không attribute được tới use case | Platform team FTE, FinOps tooling license, security/compliance | Allocate theo công thức chuẩn: **flat % của direct + shared** HOẶC **tỷ lệ usage** |

> **Quy tắc:** Cost report cho team luôn break ra 3 dòng (Direct / Shared / Overhead) để team hiểu cái nào họ optimize được, cái nào không.

#### Citations

- [50] FinOps Foundation. *How to Build a Generative AI Cost and Usage Tracker.* finops.org/wg/how-to-build-a-generative-ai-cost-and-usage-tracker/ (truy cập 21/05/2026)

### IV.3. Tagging Taxonomy

**Tag bắt buộc (enforced — request bị reject nếu thiếu):**

| Tag | Giá trị mẫu | Vai trò |
|---|---|---|
| `cost_center` | `CC-1023` (Retail Banking) | Finance attribution, chargeback target |
| `business_unit` | `retail` / `wholesale` / `risk` / `ops` / `it` | Bucket báo cáo cấp BU |
| `use_case_id` | `UC-RM-001` (RM call summary) | Đơn vị attribute nhỏ nhất về business |
| `agent_id` | `agent-rm-summary-v2` | Đơn vị attribute kỹ thuật |
| `environment` | `dev` / `staging` / `prod` | Phân biệt cost thử nghiệm vs prod |
| `owner_email` | `team-lead@org` | Người chịu trách nhiệm tối ưu |

**Tag khuyến nghị:**

| Tag | Vai trò |
|---|---|
| `model_tier` | (frontier / mid / small) — model-tiering analytics |
| `data_classification` | (public / internal / confidential / restricted) — compliance allocation |
| `experiment_id` | A/B test attribution |
| `customer_segment` | Per-customer-segment unit economics (nếu áp dụng) |

**Cơ chế enforce (Databricks-first):**

| Layer | Enforcement |
|---|---|
| **IAM / Provisioning** | Resource bị reject nếu thiếu tag (AWS SCP / Azure Policy / Databricks Unity Catalog) |
| **AI Gateway** | Request bị reject (HTTP 400) nếu header/metadata thiếu `use_case_id` + `cost_center` |
| **CI/CD** | Pipeline check tags trước khi deploy agent |
| **Monthly audit** | Tag drift report; team có resource untagged > 3 ngày → escalate |

#### Citations

- [14] AWS Blog. *Track Amazon Bedrock Costs by Caller Identity with IAM Principal-Based Cost Allocation.* aws.amazon.com/about-aws/whats-new/2026/04/bedrock-iam-cost-allocation/ (truy cập 21/05/2026)
- [51] AWS Blog. *Amazon Bedrock now supports cost allocation by IAM user and role.* aws.amazon.com/blogs/machine-learning/introducing-granular-cost-attribution-for-amazon-bedrock/ (truy cập 21/05/2026)

### IV.4. Unit Economics — 3 cấp số

| Cấp | Metric | Công thức | Phù hợp cho |
|---|---|---|---|
| **1. Cost per invocation** | `total_cost_per_invocation` | Σ 6 lớp ở Phần II.2 | Engineer optimize agent |
| **2. Cost per use case-period** | `cost_per_use_case_month` = `Σ invocations × cost_per_invocation + amortized_shared + overhead` | Use-case owner báo cáo cho BU |
| **3. Cost per business outcome** | `cost_per_outcome` = `cost_per_use_case_month / successful_outcomes_per_month` | C-Level, FP&A — chỉ số duy nhất so với manual baseline |

**Ví dụ cấp 3 (mô phỏng):**

| Use Case | Cấp 3 metric | Giá trị mục tiêu | Manual baseline |
|---|---|---|---|
| RM call summary | $ per CRM record updated | < $0.15 [cần verify] | $2–5 (RM time × hourly rate) |
| BA — BRD draft | $ per BRD generated | < $3 [cần verify] | $200–500 (BA hours × hourly rate) |
| Customer engagement script | $ per personalized script | < $0.05 [cần verify] | $1–3 |

> **Quy tắc vàng:** Nếu cost per outcome > 20% manual baseline trong 3 tháng liên tiếp, escalate use case sang FinOps Council review (Phần VI.4).

#### Citations

- [9] Drivetrain. *Unit economics for AI SaaS companies.* drivetrain.ai/post/unit-economics-of-ai-saas-companies-cfo-guide (truy cập 21/05/2026)
- [2] Acropolium. *AI agent unit economics: TCO, ROI, payback.* acropolium.com/blog/ai-agent-unit-economics/ (truy cập 21/05/2026)

### IV.5. Quy trình month-end

```
Day 1–3:  Auto-ingest cost data từ system.serving.* + system.billing.*
Day 3–5:  Auto-allocate Direct / Shared / Overhead theo rules
Day 5–7:  Tag drift audit; flag untagged > 1% spend
Day 7–10: Per-BU report drafted; owner reviews
Day 10:   Finance sign-off; chargeback entries posted (Production phase)
Day 11:   FinOps Council meeting (anomaly + cost outliers)
```

### IV.6. Dispute process (cho Chargeback phase)

| Step | Hành động | SLA |
|---|---|---|
| 1 | BU submit dispute trong vòng 10 business days sau khi report ra | T+10 |
| 2 | Platform team replay trace; xác định nguyên nhân (tag sai / shared cost mis-allocate / cost trap) | T+15 |
| 3 | Quyết định: adjust chargeback / no-adjust + cải thiện rules | T+20 |
| 4 | Update allocation rules nếu cần | T+30 |

#### Citations

- [52] nOps. *AI Cost Visibility: The Ultimate Guide.* nops.io/blog/ai-cost-visibility-the-ultimate-guide/ (truy cập 21/05/2026)

---

## Phần V — Forecasting (Driver + 3 Scenarios)

> **Kết luận đầu:** Forecast AI bằng **point-estimate là sai phương pháp** — usage tăng phi tuyến, vendor pricing thay đổi 2–4 lần/năm, retry/reflection có cost-tail dày. Phương pháp đúng: **driver-based + 3 scenarios + rolling refresh hàng tháng**.

### V.1. Vì sao point-estimate fail

| Hiện tượng | Hệ quả lên forecast |
|---|---|
| **Adoption hockey-stick** — pilot OK → BU khác xin onboard | Linear forecast underestimate 3–10× |
| **Model swap mid-quarter** — release mới (Sonnet → Opus) giảm latency 30% nhưng cost +50% | Variance ±40% |
| **Silent retry storm** — fail downstream → token ×2–5 mà user traffic không đổi | Forecast `users × tokens` sai 2–5× |
| **Prompt drift** — team thêm vài câu system prompt → tokens/call +20% | Sai vài % nhưng compound |
| **Vendor price change** — frontier model giảm 30–50%/năm | Lock giá cũ → overestimate |
| **Modality shift** — chuyển từ text-only sang multimodal | Driver model thiếu term → underestimate |

#### Citations

- [9] Drivetrain. *Unit economics for AI SaaS companies.* drivetrain.ai/post/unit-economics-of-ai-saas-companies-cfo-guide (truy cập 21/05/2026)
- [1] Galileo. *The Hidden Costs of Agentic AI.* galileo.ai/blog/hidden-cost-of-agentic-ai (truy cập 21/05/2026)
- [53] FinOps Foundation. *Effect of Optimization on AI Forecasting.* finops.org/wg/effect-of-optimization-on-ai-forecasting/ (truy cập 21/05/2026)

### V.2. Driver-based Forecast Model

**Công thức (multimodal-aware):**

```
forecast_cost[t] = (
    active_users[t]                       # D1 — adoption
  × invocations_per_user[t]               # D2 — engagement
  × Σ_m ( avg_units[m, t] × $_per_unit[m, model, t] )  # D3+D4 — usage × price per modality
  × retry_multiplier[t]                   # D5 — quality/reliability
) + non_token_overhead[t]                 # D6 — L2..L6
```

**Driver definitions:**

| # | Driver | Định nghĩa | Cách thu | Refresh |
|---|---|---|---|---|
| D1 | `active_users[t]` | DAU/MAU thực dùng agent | AI Gateway audit log | Tuần |
| D2 | `invocations_per_user[t]` | Avg/user/period | `system.serving.endpoint_usage` group by user | Tuần |
| D3 | `avg_units[m, t]` | Trung bình theo modality (tokens / images / audio sec / ...) | Endpoint usage + OTel multimodal fields | Tuần |
| D4 | `$_per_unit[m, model, t]` | Blended rate theo model mix × modality | Rate card (Phần III) × model mix % | Tháng / khi vendor đổi giá |
| D5 | `retry_multiplier[t]` | `total_inference_calls / user_facing_requests` | OTel traces | Tuần |
| D6 | `non_token_overhead[t]` | L2+L3+L4+L5+L6 trong Phần II.2 | Allocation engine | Tháng |

**D5 — retry_multiplier (biến động mạnh nhất):**

| Trạng thái | Retry mult |
|---|---|
| Workflow chưa tối ưu | 2–5× |
| Có guardrail + caching | 1.1–1.3× |
| Có circuit breaker + idempotency | 1.05–1.1× |

> **Lưu ý:** D5 là driver **bị bỏ qua nhiều nhất** nhưng impact ±30–60% (xem V.4).

#### Citations

- [8] Agents Arcade. *Cost Modeling for Agentic Systems Before You Go to Production.* agentsarcade.com/blog/cost-modeling-agentic-systems-production (truy cập 21/05/2026)
- [9] Drivetrain. *Unit economics for AI SaaS companies.* drivetrain.ai/post/unit-economics-of-ai-saas-companies-cfo-guide (truy cập 21/05/2026)
- [54] Keito. *Why AI Agent Costs Are Unpredictable.* keito.ai/blog/ai-agent-costs-unpredictable-consumption/ (truy cập 21/05/2026)

### V.3. Three-Scenario Framework

| Tham số | Conservative | **Base** | Aggressive |
|---|---|---|---|
| Adoption rate (`active_users` growth) | +5%/tháng | +15%/tháng | +35%/tháng |
| `invocations_per_user` growth | +0% | +10%/quý | +25%/quý |
| `avg_tokens_per_invocation` | stable | +5%/quý (prompt drift) | +15%/quý (longer chains) |
| `avg_audio_seconds_per_inv` (nếu audio) | stable | +5%/quý | +15%/quý |
| `$_per_unit` (text) | –10%/năm (vendor cut) | –20%/năm | –30%/năm |
| `$_per_unit` (TTS / image) | stable | –5%/năm | –10%/năm |
| `retry_multiplier` | 1,3 → 1,2 (improve) | 1,8 stable | 2,5 → 3,0 (drift) |
| Model mix shift tới higher tier | 0% | +10%/năm | +30%/năm |

> **Multimodal note:** Vendor pricing **text giảm nhanh hơn image/audio**. Khi mix shift về multimodal use case, blended `$_per_unit` không giảm tuyến tính như "30% vendor cut" mà phụ thuộc weight của từng modality.

**Khi nào dùng scenario nào:**

| Mục đích | Scenario chính | Use |
|---|---|---|
| Budget approval năm sau | Base + buffer 20% | Board |
| Worst-case planning | Aggressive | CFO stress test |
| Investment / ROI baseline | Conservative | Show ROI vẫn dương |
| Capacity planning (GPU provisioned) | Aggressive | Tránh stockout |

#### Citations

- [55] Ridgeway Financial Services. *GPU Cost Forecasting, AI Unit Economics, and Infrastructure Strategy.* ridgewayfs.com/gpu-cost-forecasting-ai-unit-economics-infrastructure-strategy/ (truy cập 21/05/2026)

### V.4. Sensitivity Analysis

Bài tập ±20% mỗi driver, hold others constant, agentic workflow điển hình:

| Rank | Driver | Impact (±20%) | Action ưu tiên |
|---|---|---|---|
| 1 | `retry_multiplier` | ±30–60% | Guardrail + reflection-depth limit + caching (Phần VI.2) |
| 2 | `avg_units_per_invocation` | ±20% | Prompt compression; context discipline; modality optimization (TTS chunking, image downsampling) |
| 3 | `invocations_per_user` | ±20% | Rate limiting; batch where possible |
| 4 | `active_users` | ±20% | Phased rollout |
| 5 | Model mix (frontier %) | ±15–25% | Model-tier policy (Phần VI.2) |
| 6 | `$_per_unit` | ±20% | Multi-provider; provisioned throughput cho high-volume |

> **Hàm ý:** Engineering attention vào **driver 1–2 trước** (retry + units) — 2 driver platform team kiểm soát trực tiếp. Driver 3–4 do business adoption, 5–6 do procurement/strategy.

### V.5. Operating Cadence

| Cadence | Hoạt động | Owner |
|---|---|---|
| **Hàng tuần** | Refresh D1, D2, D3, D5 từ system tables; flag deviation > 15% vs forecast | Platform |
| **Hàng tháng** | Re-baseline forecast; cập nhật model mix + vendor pricing (Phần III); rolling 12-month | FinOps + Platform |
| **Hàng quý** | Scenario reassessment; CFO stress test; capacity planning quý tới | FinOps Council |
| **Hàng năm** | Strategic budget; multi-year forecast; vendor commitment decisions (provisioned throughput) | Board / Exec |

> Vendor pricing update phải xong **trong 14 ngày** sau khi vendor announce — không để stale price làm sai forecast.

### V.6. KPI cho forecast process

| KPI | Target sau 12 tháng |
|---|---|
| **Forecast accuracy** (MAPE month-ahead) | ≤ 15% [cần verify baseline] |
| **Forecast accuracy** (MAPE quarter-ahead) | ≤ 25% |
| **% use case có driver-level forecast** | 100% prod use case |
| **% budget breach phát hiện trước** | ≥ 90% |
| **Refresh cadence compliance** | ≥ 95% on-time |

**Cảnh báo phương pháp luận:**

1. **Không extrapolate khi data < 90 ngày** — variance quá lớn, dùng analog (use case tương tự) thay vì regression.
2. **Forecast chỉ tốt khi tagging tốt** — pre-condition là tagging discipline (Phần IV.3).
3. **Update vendor pricing trong 14 ngày** — xem V.5.
4. **Tracking forecast accuracy là KPI Platform team**, không phải nice-to-have.
5. **Multimodal cần track per-modality unit count riêng** — đừng pre-blend thành "token equivalent" làm mất thông tin recompute.

#### Citations

- [53] FinOps Foundation. *Effect of Optimization on AI Forecasting.* finops.org/wg/effect-of-optimization-on-ai-forecasting/ (truy cập 21/05/2026)
- [22] FinOps Foundation. *Cost Estimation of AI Workloads.* finops.org/wg/cost-estimation-of-ai-workloads/ (truy cập 21/05/2026)

---

## Phần VI — Monitoring & Governance

> **Kết luận đầu:** Monitoring kiểu báo cáo cuối tháng đã là **quá muộn** — một retry storm hoặc reflection loop có thể đốt budget tháng trong vài giờ. Cần **3 tầng**: pre-spend → real-time → post-spend, gắn với FinOps Council bi-weekly.

### VI.1. Mô hình 3 tầng kiểm soát

```
┌─────────────────────────────────────────────────────────────┐
│  TẦNG 1 — PRE-SPEND (Preventive)                            │
│  Budget hard-limit  |  Policy guardrail  |  Rate limit      │
│  → Chặn trước khi xảy ra @ AI Gateway                       │
├─────────────────────────────────────────────────────────────┤
│  TẦNG 2 — REAL-TIME (Detective)                             │
│  Streaming metrics → 3σ alert (5–15 phút)                   │
│  → Bắt anomaly khi đang xảy ra                              │
├─────────────────────────────────────────────────────────────┤
│  TẦNG 3 — POST-SPEND (Reactive)                             │
│  Daily/weekly report + chargeback ledger                    │
│  → Forensic + accountability + RCA                          │
└─────────────────────────────────────────────────────────────┘
```

| Tầng | Mục đích | Cơ chế | Vendor support |
|---|---|---|---|
| **1. Pre-spend** | Chặn over-spend trước khi xảy ra | Budget hard-limit, rate-limit per user/use-case, model-tier policy, prompt size limit | Databricks Unity AI Gateway service policies; AWS Bedrock Guardrails; Azure quota |
| **2. Real-time** | Phát hiện anomaly trong 5–15 phút | Streaming metrics → threshold alert; ML anomaly trên cost-per-inv, retry rate, token-per-call | Databricks Lakehouse Monitoring; AWS Cost Anomaly Detection; Langfuse/LangSmith alerts |
| **3. Post-spend** | Forensic, RCA, accountability | Daily/weekly cost report; chargeback ledger; FinOps Council review | Databricks dashboards; Apptio; Finout |

#### Citations

- [13] Databricks Blog. *Introducing AI spend controls with Unity AI Gateway.* databricks.com/blog/introducing-ai-spend-controls-unity-ai-gateway (truy cập 21/05/2026)
- [56] AWS Blog. *Build a proactive AI cost management system for Amazon Bedrock — Part 2.* aws.amazon.com/blogs/machine-learning/ (truy cập 21/05/2026)

### VI.2. Pre-spend Controls

**Budget enforcement:**

| Loại | Scope | Hành vi khi vượt |
|---|---|---|
| Per-user | Dev cá nhân (coding agent) | 80% email alert · 100% throttle · 120% block + manager approval |
| Per-use-case | UC-RM-001, UC-BA-002, ... | 80% owner alert · 100% throttle non-prod · 120% block + BU exec approval |
| Per-BU | Retail / Wholesale / Risk / ... | 90% BU lead alert · 100% freeze new use case onboarding |
| Platform-wide | Cả Agent Platform | 95% exec alert · 100% all-hands cost review |

**Policy guardrails (chặn ngay tại Gateway):**

| Policy | Mục tiêu | Ví dụ rule |
|---|---|---|
| Model-tier | Tránh frontier cho task đơn giản | Dev env → cấm `gpt-5-frontier`; phải dùng mid-tier để escalate |
| Prompt size | Tránh prompt explosion | Max 32K context cho prod; 128K cần justification |
| Reflection depth | Tránh runaway | Max 3 levels; max 5 self-critique calls |
| Rate limit | Tránh runaway loop / abuse | 1000 req/min prod; 100 req/min dev |
| PII | Compliance + cost | Reject request có PII chưa redact |
| External provider routing | Chặn data egress | `data_classification = restricted` → chỉ on-prem hoặc Bedrock private |
| **Modality budget** | TTS / video cost dominance | Cap `audio_out_chars/inv` = 5000; `video_out_sec/inv` = 30 |

> **Modality budget** là pre-spend control quan trọng nhất cho audio/video use case — TTS dominate ≥ 90% cost trong chuỗi multimodal (Phần III.6).

#### Citations

- [27] Clarifai. *AI Cost Controls: Budgets, Throttling & Model Tiering.* clarifai.com/blog/ai-cost-controls (truy cập 21/05/2026)
- [57] Databricks Blog. *What's new in Unity AI Gateway: service policies, guardrails, observability, and cost controls.* databricks.com/blog/whats-new-unity-ai-gateway (truy cập 21/05/2026)

### VI.3. Real-time Monitoring

**Metric stack tối thiểu:**

| Metric | Computation | Trigger threshold |
|---|---|---|
| `cost_per_minute_by_use_case` | Σ cost rolling 5-min | > 3σ vs baseline 30 ngày |
| `retry_rate_by_agent` | retries / inv rolling 15-min | > 25% (baseline ~ 10%) |
| `avg_units_per_invocation[modality]` | rolling 1-hour, per modality | > 1,5× baseline tuần trước |
| `error_rate` | failed / total rolling 15-min | > 5% |
| `budget_burn_rate` | (spend / period_elapsed) / (budget / period_total) | > 1,2 |
| `model_mix_drift` | % req frontier model | > 1,5× baseline |
| `cache_hit_rate` | hits / total (semantic cache) | < 0,5× baseline |
| `audio_out_chars_per_inv` | rolling 1-hour, audio use case | > 1,5× baseline → flag TTS bloat |
| `image_in_count_per_inv` | rolling 1-hour, vision use case | > 1,5× baseline |

**Alert hierarchy:**

| Severity | Trigger | Recipient | SLA |
|---|---|---|---|
| **P1 — Critical** | Budget burn > 200% pacing HOẶC cost spike > 10σ | On-call + BU lead + FinOps + Exec | 15 phút |
| **P2 — High** | Anomaly cost/use-case > 5σ; retry rate > 50% | Use case owner + Platform on-call | 1 giờ |
| **P3 — Medium** | Cost drift +20% WoW; tag accuracy < 95%; forecast variance > 25% | Owner + FinOps | 1 ngày |
| **P4 — Info** | Forecast variance > 15%; vendor pricing change (Phần III.7) | FinOps mailbox | 1 tuần |

**Anomaly detection approach:**

| Approach | Pros | Cons | Phù hợp |
|---|---|---|---|
| **Static threshold** (3σ cost/min) | Đơn giản, deploy nhanh | Sai khi seasonality cao | Phase 1 POC |
| **Rolling baseline** (compare 4 tuần cùng kỳ) | Bắt seasonality | Cần ≥ 60 ngày data | Phase 2 |
| **ML-based** (Prophet, Lakehouse Monitoring) | Bắt trend + seasonality | Phải maintain model; false-positive cao đầu | Phase 3 |

#### Citations

- [58] Capital One Software Blog. *Building a GenAI cost supervisor agent in Databricks.* capitalone.com/software/blog/databricks-genai-cost-supervisor-agent/ (truy cập 21/05/2026)
- [59] Galileo. *A Guide to AI Agent Cost Optimization With Observability.* galileo.ai/blog/ai-agent-cost-optimization-observability (truy cập 21/05/2026)

### VI.4. FinOps Council Operating Model

**Charter:**

> **Mission:** Đảm bảo Agent Platform spend được phân bổ, dự báo, kiểm soát theo policy đã thông qua, và mọi anomaly được xử lý trong SLA.

**Membership:**

| Role | Đại diện | Tần suất |
|---|---|---|
| Chair | Head of AI Platform | Bi-weekly |
| FinOps Lead | FinOps Practitioner | Bi-weekly |
| Finance | FP&A AI cost analyst | Bi-weekly |
| Engineering | Platform engineering lead | Bi-weekly |
| BU reps | 1 mỗi major BU | Monthly rotate |
| Security / Compliance | InfoSec rep | Monthly hoặc ad-hoc |

**Agenda (bi-weekly, 60 phút):**

| Phần | Time | Output |
|---|---|---|
| Cost report period trước (variance vs forecast) | 10' | Re-baseline forecast? |
| Anomaly review (P1/P2) | 15' | RCA + action item + owner |
| Top spenders / Top growers | 10' | Watch list period tới |
| Policy / Tagging compliance | 10' | Action item enforce |
| Use case onboarding pipeline | 10' | Approve / hold dựa budget headroom |
| Vendor / Procurement updates | 5' | Decisions log |

### VI.5. Governance Artifacts

| Artifact | Phase | Owner |
|---|---|---|
| Budget policy document (per-tier limits) | R&D | FinOps |
| Tagging enforcement policy (tech impl) | R&D | Platform Eng |
| Alert runbook (P1/P2 response steps) | POC | Platform Eng + FinOps |
| Anomaly model spec | POC | Data Science |
| FinOps Council charter (signed) | R&D | Head of AI Platform |
| Quarterly cost governance scorecard | Production | FinOps |
| Modality budget policy (TTS / video cap) | R&D | Platform Eng |

#### Citations

- [60] Databricks Blog. *Expanding agent governance with Unity AI Gateway.* databricks.com/blog/ai-gateway-governance-layer-agentic-ai (truy cập 21/05/2026)

---

## Phần VII — Tooling Assessment

> **Kết luận đầu:** Cho stack Databricks-first + Agentic AI nội bộ, **stack đề xuất là combo 3 layer**: (1) **Databricks Unity AI Gateway** làm cost-attribution backbone (native, không thay được); (2) **Langfuse OSS self-hosted** làm orchestration-layer observability (cover gap L3 mà Gateway chưa cover sâu); (3) **Databricks Lakehouse Monitoring + custom dashboard** thay vì FinOps platform commercial trong 12 tháng đầu — đến khi quy mô đủ để justify Finout/Apptio.

### VII.1. Build vs Buy Framework

| Capability | Recommend | Lý do |
|---|---|---|
| Cost data ingestion từ Databricks system tables | **Adopt** (native, không build) | Databricks expose sẵn `system.serving.*`, `system.billing.*` |
| Cost data từ external provider (OpenAI, Anthropic) khi đi direct | **Build connector** (light) | Provider API + tag normalization; effort thấp |
| Tagging enforcement | **Adopt + light Build** | Adopt: Unity Catalog + Cloud Policy; Build: CI/CD lint, request-time validator |
| LLM trace observability (L1) | **Adopt** (Databricks tracing + AI Gateway) | Native; không có lý do build |
| Orchestration trace (L3 — tool calls, retries, reflection) | **Adopt OSS** (Langfuse / OpenLLMetry) + light Build | Native chưa cover sâu; OSS đủ; OTel chuẩn để tránh lock-in |
| Cost dashboard | **Build trên Databricks SQL** | Linh hoạt; tận dụng SQL skill có sẵn; tránh license commercial sớm |
| Chargeback ledger | **Build** (light, trong Databricks) | Logic riêng theo tagging taxonomy |
| Anomaly detection | **Adopt** (Lakehouse Monitoring) Phase 1; có thể **Build** thêm Phase 2 | Bắt đầu native, tránh over-engineer |
| FinOps platform (cross-cloud, executive view) | **Defer** 12 tháng | Reassess khi ≥ $1M/quý AI spend [cần verify ngưỡng] |
| ROI / business outcome tracking | **Build** | Logic riêng cho từng use case |

### VII.2. So sánh vendor

**Native cloud / platform:**

| Tool | Strengths | Weaknesses | Cost model | Phù hợp |
|---|---|---|---|---|
| **Databricks Unity AI Gateway** | Cost tracking per user/model/tag native; system tables; budget alerts; gateway cho external models; service policies; coding-agent governance | Đang phát triển; một số capability ở Beta; ecosystem ngoài Databricks vẫn cần connector | Included trong Databricks usage; serving DBU | **Core — bắt buộc với Databricks-first** |
| **AWS Bedrock cost allocation** | Application Inference Profiles; IAM principal-based allocation (CUR 2.0); tích hợp Cost Explorer + Budgets + Anomaly Detection | Chỉ Bedrock; external model phải tag riêng | Native trong AWS billing | Cần khi đi AWS Bedrock |
| **Azure OpenAI + Azure FinOps Hubs** | Resource tags; Azure Cost Management; FinOps reference architecture | Granularity per-call hạn chế hơn Databricks/Bedrock | Native Azure billing | Cần khi đi Azure OpenAI |

**LLM Observability — OSS / Cloud:**

| Tool | Cost tracking strength | Pricing | Self-host | Phù hợp |
|---|---|---|---|---|
| **Langfuse** | Unit-based pricing; cost per trace rõ; multi-framework | Hobby: 50K events free; Enterprise self-host | ✅ | **Recommended** cho L3 gap |
| **LangSmith** | Trace-level cost; integration LangChain tốt; OTel support từ 03/2025 | Seat-based; 5K traces free | Limited | Phù hợp nếu locked-in LangChain |
| **Helicone** | Proxy-based; cost dashboard nhanh | 10K req/tháng free | ✅ | POC nhanh; weakness: network hop |
| **OpenLLMetry** | Pure OpenTelemetry — plug vào Datadog/New Relic/Grafana | OSS | ✅ | Chỉ nên dùng nếu đã có OTel backbone |
| **Laminar** | Agent-focused observability, evaluation | Hybrid | ✅ | Strong cho evaluation |

**FinOps platform (commercial, cross-cloud):**

| Tool | Strengths | Weaknesses | Khi nên reassess |
|---|---|---|---|
| **Finout** | GenAI cost allocation; Bedrock-aware; megabill view; chargeback engine | Chưa sâu ở orchestration layer; license fee đáng kể | Spend vượt $1M/quý [cần verify] và cần unified multi-cloud |
| **Apptio** | TBM + chargeback enterprise; AI cost roadmap | Heavyweight, deployment lâu | Khi đã có Apptio cho IT cost mgmt |
| **nOps** | Cloud cost optimization; AI visibility; commitment management | Cloud-cost-first, AI là feature thêm | Khi quá tải native tooling |
| **Vantage** | Multi-cloud cost view; UX tốt; AI pricing dimensions tốt | Chưa đủ depth cho chargeback complex | Exec visibility nhanh |
| **Capital One Slingshot** | Sâu Databricks; có GenAI Cost Supervisor agent reference | Hẹp Databricks-only; phải mua | Khi quy mô Databricks lớn |

#### Citations

- [20] ZenML Blog. *10 Best LLM Monitoring Tools to Use in 2025.* zenml.io/blog/best-llm-monitoring-tools (truy cập 21/05/2026)
- [61] Firecrawl. *Best LLM Observability Tools in 2026.* firecrawl.dev/blog/best-llm-observability-tools (truy cập 21/05/2026)
- [62] DigitalApplied. *Observability Stack TCO: LangSmith vs LangFuse vs Helicone.* digitalapplied.com/blog/observability-stack-tco-calculator-langsmith-langfuse-helicone (truy cập 21/05/2026)

### VII.3. Stack đề xuất (cho Phase 1–2)

```
┌──────────────────────────────────────────────────────────────────┐
│  EXECUTIVE / FINANCE LAYER                                       │
│  - Custom dashboards (Databricks SQL + AI/BI)                    │
│  - Monthly chargeback report (export → ERP)                      │
└──────────────────────────────────────────────────────────────────┘
              ▲                              ▲
              │                              │
┌─────────────────────────────────┐   ┌──────────────────────────────┐
│  COST & FINOPS LOGIC LAYER       │   │  ML / ANOMALY DETECTION     │
│  - Allocation engine (Build,     │   │  - Databricks Lakehouse     │
│    Databricks notebooks + jobs)  │   │    Monitoring               │
│  - Tagging policy enforcer       │   │  - Custom alert rules       │
│  - Chargeback ledger              │   │                             │
└─────────────────────────────────┘   └──────────────────────────────┘
              ▲                                       ▲
              │                                       │
┌──────────────────────────────────────────────────────────────────┐
│  OBSERVABILITY LAYER                                             │
│  - Databricks Unity AI Gateway (L1 cost + budget + policy)       │
│  - Langfuse OSS self-hosted (L3 orchestration trace)             │
│  - OTel instrumentation in agent framework                       │
└──────────────────────────────────────────────────────────────────┘
              ▲
              │
┌──────────────────────────────────────────────────────────────────┐
│  DATA SOURCES                                                    │
│  - system.serving.endpoint_usage  (L1)                           │
│  - system.billing.usage           (L2)                           │
│  - Langfuse traces                (L3, L4)                       │
│  - HR / Procurement systems       (L5)                           │
│  - Ticketing / Workforce data     (L6)                           │
└──────────────────────────────────────────────────────────────────┘
```

### VII.4. TCO 12 tháng đầu (rough estimate)

> **[cần verify]** Số dưới là điểm xuất phát thảo luận, không phải báo giá. Cần RFQ chính thức trước khi sign-off.

| Hạng mục | Build / Buy | Year-1 cost (USD, ước lượng) | Ghi chú |
|---|---|---|---|
| Databricks Unity AI Gateway | Native (đã có) | $0 incremental | Đã có trong contract Databricks |
| Langfuse OSS self-hosted | Adopt | $5K–15K (compute + ops) | Chạy trên hạ tầng có sẵn |
| OTel instrumentation lib | Build | 0,5 FTE × 3 tháng ≈ $30K | Một lần |
| Allocation engine | Build | 1 FTE × 4 tháng ≈ $80K | Một lần, sau maintenance |
| Custom dashboards | Build | 0,5 FTE × 2 tháng ≈ $20K | Một lần |
| Lakehouse Monitoring | Native | Included | DBU consumption |
| FinOps platform commercial | Defer | $0 | Reassess sau 12 tháng |
| **Total Year-1 (Build phase + ops)** | | **~ $135K–145K** | Phần lớn là FTE, không phải license |

### VII.5. Trigger tái đánh giá

| Trigger | Hành động |
|---|---|
| Spend AI > $1M/quý ổn định 2 quý | Reassess FinOps commercial platform (Finout/Apptio) |
| Multi-cloud (Databricks + native AWS + native Azure) | Cân nhắc tool unify cross-cloud |
| Số use case > 30 active prod | Cân nhắc commercial cho governance scale |
| Đội FinOps < 2 FTE và workload tăng | Mua commercial thay vì hire |
| Audit / regulator yêu cầu reporting chuẩn TBM | Cân nhắc Apptio |

### VII.6. Risk & Mitigation

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Databricks AI Gateway feature gap (đang phát triển nhanh) | Medium | Medium | Theo Beta release notes; quarterly capability review |
| Langfuse self-host operational burden | Medium | Low–Medium | Định nghĩa SLA nội bộ; pin version; có upgrade window |
| Tagging compliance trượt khi user tăng | High | High | Enforcement tự động ở Gateway + CI/CD; quarterly audit |
| Vendor pricing tăng đột biến | Medium | High | Multi-provider; provisioned throughput negotiation |
| Build cost vượt estimate | Medium | Medium | Time-box mỗi component; ưu tiên adopt over build khi possible |

#### Citations

- [63] Softcery. *9 AI Observability Platforms Compared.* softcery.com/lab/top-8-observability-platforms-for-ai-agents-in-2025 (truy cập 21/05/2026)
- [64] Finout. *The Hidden Superpower of Bedrock Cost Allocation — and Its Limits.* finout.io/blog/the-hidden-superpower-of-bedrock-cost-allocation-and-its-limits (truy cập 21/05/2026)
- [65] Laminar. *Langfuse Alternatives 2026.* laminar.sh/article/langfuse-alternatives-2026 (truy cập 21/05/2026)

---

## Phần VIII — Implementation Roadmap (R&D → POC → Prod)

> **Executive 1-pager (đọc trước nếu thời gian giới hạn):**
> Đề xuất triển khai FinOps cho Agent Platform qua **3 phase / 12 tháng**, với 3 stage-gate quyết định invest tiếp.
>
> | Phase | Tháng | Mục tiêu cốt lõi | Gate quyết định | Investment ước lượng |
> |---|---|---|---|---|
> | **R&D — Foundation** | 1–3 | Lock taxonomy, tagging policy, baseline data, council charter, tooling stack v1 | **G1:** ≥ 95% prod requests có đủ 6 tag bắt buộc + 1 dashboard L1+L2 live | 1,5 FTE × 3 tháng + Databricks usage |
> | **POC — Pilot** | 4–6 | Apply taxonomy + allocation lên 2–3 use case; deploy budget + alert; showback report | **G2:** Cost per business outcome đo được cho 3 use case; forecast MAPE month-ahead < 25% | 2 FTE × 3 tháng + Langfuse OSS |
> | **Production — Scale & Govern** | 7–12 | Onboard tất cả prod use case; chargeback go-live; FinOps Council BAU | **G3:** ≥ 90% prod use case lên chargeback; ≥ 90% budget breach phát hiện trước; forecast MAPE ≤ 15% | 2 FTE BAU + reassess commercial FinOps platform |
>
> **Expected outcome 12 tháng:** Cost visibility theo 6 lớp; chargeback chính xác cho mọi use case; forecast scenario-based; pre-spend controls active; cơ chế governance bi-weekly. **Cảnh báo:** Phụ thuộc kỷ luật tagging — đây là risk #1.

**Guiding principles:**

| # | Principle | Implication |
|---|---|---|
| 1 | **Visibility trước Control trước Optimization** | Không cố optimize khi chưa thấy đủ cost; không enforce policy khi chưa có showback ổn định |
| 2 | **Showback trước Chargeback** | Chargeback chính xác cần ≥ 6 tháng baseline showback |
| 3 | **Adopt-first, Build-second** | Mọi capability đã có native → adopt; chỉ build connector + business logic riêng |
| 4 | **MECE 6-layer taxonomy là vocabulary chung** | Mọi báo cáo, dashboard, alert phải map về 6 lớp; không taxonomy ad-hoc |
| 5 | **Tagging discipline là điều kiện sống còn** | Không có tag = không có allocation = không có FinOps |
| 6 | **Operate, đừng project** | FinOps Council bi-weekly là cơ chế vận hành, không phải audit project |

### VIII.1. Phase 1 — R&D / Foundation (Tháng 1–3)

**Mục tiêu:** Đặt nền móng — lock vocabulary, lock policy, lock tooling stack, có baseline data để Phase 2 chạy được.

**Workstream:**

| WS | Hoạt động chính | Output | Owner |
|---|---|---|---|
| **WS1: Taxonomy & Methodology** | Lock 6-layer cost taxonomy (Phần II); driver-based forecast (Phần V) | Methodology pack approved | FinOps Lead + Platform Lead |
| **WS2: Tagging Policy** | Định nghĩa 6 tag bắt buộc + 4 khuyến nghị; quy trình enforce ở Gateway + CI/CD + IAM | Policy doc signed; technical spec | Platform Eng + Security |
| **WS3: Tooling Stack v1** | Set up Unity AI Gateway cost tracking; deploy Langfuse OSS; OTel SDK trong agent framework | Tooling live; smoke test pass | Platform Eng |
| **WS4: Baseline Data Collection** | Thu thập 60–90 ngày cost data; clean + tag retrofit cho top use case hiện hữu | Baseline dataset; data quality report | Data Eng |
| **WS5: Governance Setup** | Charter FinOps Council; assign members; bi-weekly cadence; agenda template | Charter signed; 2 meeting đã chạy | Head of AI Platform |
| **WS6: Quick-win dashboard** | Build dashboard L1+L2 theo use_case_id; cập nhật daily | Dashboard go-live, ≥ 5 stakeholder dùng tuần đầu | Data + Platform |

**KPI Phase 1:**

| KPI | Target |
|---|---|
| % prod request có đủ 6 tag bắt buộc | ≥ 95% |
| Số use case có cost dashboard | ≥ 5 |
| Số council meeting đã chạy | ≥ 4 |
| Số driver thu thập được trong forecast model | ≥ 5/6 |

**Gate G1 (cuối tháng 3):**

> Pass nếu: ≥ 95% prod requests có đủ tag + L1+L2 dashboard live + FinOps Council chạy đều + baseline dataset 60+ ngày. **Fail action:** Kéo dài R&D 1 tháng, không tiến POC khi tagging chưa đạt.

### VIII.2. Phase 2 — POC / Pilot (Tháng 4–6)

**Mục tiêu:** Validate end-to-end methodology trên 2–3 use case thật. Đo được cost per business outcome. Deploy pre-spend controls.

**Use case ứng viên (selection criteria):**

| Tiêu chí | Trọng số |
|---|---|
| Volume đủ lớn để có signal (≥ 10K invocations/tháng) | High |
| Business outcome định nghĩa được rõ (đơn vị đo được) | High |
| BU owner sẵn sàng participate trong council | High |
| Đa dạng pattern (1 RAG-heavy, 1 multi-agent, 1 single-shot) | Medium |
| Đại diện cho ≥ 2 BU khác nhau | Medium |

**Workstream:**

| WS | Hoạt động chính | Output |
|---|---|---|
| **WS1: Pilot use case onboarding** | Apply 6-layer taxonomy + tagging + unit economics cho 2–3 pilot | Pilot agents instrumented end-to-end |
| **WS2: Allocation engine v1** | Build allocation engine (Direct/Shared/Overhead); chạy month-end report cho pilot | Allocation report tháng cuối Phase 2 |
| **WS3: Forecasting v1** | Calibrate driver model với data Phase 1; chạy 3 scenarios; CFO review | Quarterly forecast pack |
| **WS4: Pre-spend controls** | Cấu hình budget per-use-case; rate limit; policy guardrails ở Gateway | Active controls; first alert dispatched |
| **WS5: Anomaly detection v1** | Static threshold + rolling baseline; alert ở P2/P3 | First-month alert log; tuning notes |
| **WS6: Showback report** | Monthly showback per use-case + per BU; 360° feedback từ BU lead | Report v1; feedback log |

**KPI Phase 2:**

| KPI | Target |
|---|---|
| Cost per business outcome đo được cho pilot | ≥ 3 use case |
| Forecast MAPE month-ahead (pilot) | < 25% |
| % alert P1/P2 với RCA root cause trong SLA | ≥ 80% |
| Showback report on-time delivery | 100% |
| BU lead chấp nhận showback (no major dispute) | ≥ 2/3 pilot |

**Gate G2 (cuối tháng 6):**

> Pass nếu: 3 use case có cost per outcome + forecast MAPE < 25% + anomaly engine bắt được ≥ 1 incident thật + BU lead nghiệm thu showback. **Fail action:** Mở rộng POC thêm 1 quý trước khi go production.

### VIII.3. Phase 3 — Production / Scale (Tháng 7–12)

**Mục tiêu:** Onboard toàn bộ prod use case; chuyển từ Showback → Chargeback cho use case ổn định; FinOps Council BAU; tái đánh giá tooling commercial.

**Workstream:**

| WS | Hoạt động chính | Output |
|---|---|---|
| **WS1: Mass onboarding** | Onboard 100% prod use case lên taxonomy + tagging | All-use-case dashboard |
| **WS2: Chargeback go-live** | Chuyển use case đạt criteria sang chargeback (≥ 6 tháng showback + tagging accuracy > 95%) | Chargeback ledger entries trong ERP |
| **WS3: Forecast maturity** | Rolling 12-month forecast; quarterly stress test; scenario-based capacity planning | Forecast accuracy MAPE ≤ 15% |
| **WS4: Anomaly v2** | ML-based anomaly (Lakehouse Monitoring); reduce false positive | False-positive rate ≤ 10% |
| **WS5: Optimization rituals** | Quarterly "cost trap hunt" (retry storm, prompt drift, model tier audit) | Quarterly optimization report; saved $ tracked |
| **WS6: Commercial FinOps reassessment** | Reassess Finout/Apptio nếu spend > $1M/quý hoặc > 30 prod use case | Buy/Build/Defer decision documented |

**KPI Phase 3:**

| KPI | Target end of Year 1 |
|---|---|
| % prod use case có chargeback active | ≥ 90% |
| % budget breach phát hiện trước | ≥ 90% |
| Forecast accuracy MAPE month-ahead | ≤ 15% |
| Forecast accuracy MAPE quarter-ahead | ≤ 25% |
| Tag compliance (audit pass rate) | ≥ 98% |
| Quarterly savings từ optimization rituals | ≥ 10% of optimizable spend [cần verify baseline] |
| FinOps Council on-time meeting compliance | ≥ 95% |

**Gate G3 (cuối tháng 12):**

> Pass nếu: ≥ 90% prod chargeback + forecast MAPE month-ahead ≤ 15% + tag compliance ≥ 98% + ≥ 1 commercial tooling decision documented (adopt or defer). **Output:** Year-2 plan với scale capability + ROI summary cho Board.

### VIII.4. Dependencies & Pre-requisites

| Dependency | Mức độ critical | Status (giả định) |
|---|---|---|
| Databricks Unity Catalog + Unity AI Gateway active | **Blocker** | Cần confirm với Platform team |
| Finance buy-in cho FinOps Council charter | **Blocker** | Cần executive sponsor |
| Cost center mapping (HR → IT systems) | High | Phụ thuộc HR/Finance |
| Tagging enforcement ở IAM + Gateway | **Blocker** | Phải có trước Phase 2 |
| Headcount: 1,5–2 FTE FinOps + 0,5–1 FTE Data Eng | High | Cần approve |
| Existing observability backbone (nếu có) | Medium | Tận dụng nếu có Datadog/Grafana |

### VIII.5. Risk Register

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Tagging discipline thấp ở team adoption | High | Critical | Enforce at Gateway (HTTP 400 nếu thiếu tag) + CI/CD lint + quarterly audit |
| BU pushback chargeback (dispute storm) | Medium | High | Showback ≥ 6 tháng trước; SLA dispute rõ; FinOps Council mediation |
| Forecast accuracy thấp do data nhiễu Phase 1–2 | High | Medium | Scenario thay vì point estimate; cập nhật rolling |
| Vendor pricing thay đổi đột biến | Medium | High | Multi-provider; provisioned throughput negotiation; quarterly vendor review |
| Headcount không đủ → BAU bỏ rơi | Medium | Critical | Hire trước Phase 2; document mọi process; tránh single-person dependency |

### VIII.6. Investment Summary

> **[cần verify]** — ước lượng cho thảo luận, không phải báo giá.

| Hạng mục | Year-1 (USD) | Ghi chú |
|---|---|---|
| Headcount FinOps + Platform Eng allocated | $300K–500K | 2–3 FTE blend rate |
| Databricks usage incremental (instrumentation, dashboards) | $30K–60K | Marginal trên hợp đồng đã có |
| Langfuse OSS self-host (compute + ops) | $10K–20K | Hạ tầng có sẵn |
| Build effort (allocation engine, dashboards, OTel SDK) | đã include trong FTE | One-time |
| Training + certification (FinOps for AI) | $5K–10K | FinOps Foundation cert |
| Commercial tooling | $0 Year-1 | Defer; reassess cuối Year-1 |
| **Total Year-1** | **~ $345K–590K** | Chủ yếu FTE; tooling marginal |

> So với mức spend AI dự kiến (cần baseline cụ thể), FinOps program nhằm tiết kiệm 10–25% spend → payback < 12 tháng với scale dự kiến.

### VIII.7. C-Level Talking Points

1. **Problem statement:** AI spend đang tăng phi tuyến nhưng visibility, allocation, forecast đều ad-hoc. Risk: thất thoát budget + thiếu accountability cross-BU.
2. **Industry benchmark:** 63% Fortune 500 đã track AI spend; chỉ 14,2% đạt "Run" maturity → window dẫn trước vẫn rộng. Databricks/AWS/Azure đều đã ra capability native trong 12 tháng qua.
3. **Đề xuất:** 12-tháng program qua 3 phase với 3 stage-gate. Adopt-first, Build-second. Showback trước, Chargeback sau.
4. **Investment:** ~ $345K–590K Year-1 (chủ yếu FTE allocation); commercial tooling defer.
5. **Expected outcome:** Full visibility 6-layer; chargeback ≥ 90% prod; forecast MAPE ≤ 15%; pre-spend controls active; FinOps Council BAU.
6. **Risk #1:** Tagging discipline — đã có mitigation rõ ràng (enforce ở gateway + CI/CD).
7. **Đề nghị:** Approve G1–G3 gating; sponsor FinOps Council charter; hire 1,5–2 FTE FinOps.

#### Citations

- [3] FinOps Foundation. *State of FinOps 2025 Report.* data.finops.org/2025-report/ (truy cập 21/05/2026)
- [15] FinOps Foundation. *2025 FinOps Framework.* finops.org/insights/2025-finops-framework/ (truy cập 21/05/2026)
- [66] Finout. *FinOps in the Age of AI: A CPO's Guide.* finout.io/blog/finops-in-the-age-of-ai-a-cpos-guide-to-llm-workflows-rag-ai-agents-and-agentic-systems (truy cập 21/05/2026)

---

## Phụ lục A — Bảng thuật ngữ (Glossary)

| Thuật ngữ | Định nghĩa |
|---|---|
| **Agentic AI** | Hệ thống AI tự chủ thực hiện chuỗi tác vụ multi-step (planning, tool use, reflection), khác với single-shot inference |
| **Allocation** | Phân bổ chi phí tới đơn vị attribute (use case / BU / cost center) |
| **Cached tokens** | Tokens được lưu cache (prompt caching) → giá rẻ hơn input tokens |
| **Chargeback** | Bill thật cost vào P&L của team / BU |
| **Cost trap** | Hành vi gây bloat cost không tăng giá trị (retry storm, reflection loop, ...) |
| **CUR** | Cost and Usage Report — file billing chi tiết của AWS |
| **DBU** | Databricks Unit — đơn vị tính usage trên Databricks |
| **Driver-based forecast** | Forecast dựa trên cấu phần định lượng (users × invocations × tokens × price × retry) thay vì extrapolate trend |
| **Embedding** | Vector đại diện cho text/image/audio dùng cho semantic search |
| **FinOps** | FinOps = Financial Operations cho Cloud (mở rộng sang AI) — discipline kết hợp Finance + Engineering + Business |
| **Frontier model** | Model lớn nhất / mới nhất / đắt nhất của một provider (vd. Claude Opus, GPT-5, Gemini Ultra) |
| **HITL** | Human-in-the-loop — quy trình có người review/intervene |
| **MAPE** | Mean Absolute Percentage Error — chỉ số đo độ chính xác forecast |
| **MECE** | Mutually Exclusive, Collectively Exhaustive — nguyên tắc phân loại không trùng, không thiếu |
| **Multimodal** | Model nhận / sinh được nhiều loại data (text + image + audio + video) |
| **OTel** | OpenTelemetry — chuẩn mở cho observability (traces, metrics, logs) |
| **PAYG** | Pay-As-You-Go — pricing model trả theo usage thực |
| **Provisioned throughput** | Reserve capacity (GPU) với giá cố định $/giờ, dùng cho high-volume |
| **RAG** | Retrieval-Augmented Generation — kết hợp vector search với LLM |
| **Reflection loop** | Agent tự critique output của mình và lặp lại → token bloat nếu unbounded |
| **Retry multiplier** | Tỷ lệ `total_inference_calls / user_facing_requests` — đo overhead do retry |
| **Showback** | Hiển thị cost cho team nhưng không bill (đối lập với chargeback) |
| **STT** | Speech-to-Text |
| **TBM** | Technology Business Management — framework chuẩn cho IT cost mgmt |
| **TCO** | Total Cost of Ownership |
| **TTS** | Text-to-Speech |
| **Unit economics** | Cost per đơn vị business (per invocation / per outcome / per ticket resolved) |

---

## Phụ lục B — Bibliography tổng hợp

> Tham chiếu `[N]` trong văn bản trỏ đến mục tương ứng dưới đây. Citations cũng được lặp lại ở cuối mỗi section để tiện đối chiếu khi đọc rời.

1. Galileo. *The Hidden Costs of Agentic AI.* galileo.ai/blog/hidden-cost-of-agentic-ai (truy cập 21/05/2026)
2. Acropolium. *AI agent unit economics: TCO, ROI, payback.* acropolium.com/blog/ai-agent-unit-economics/ (truy cập 21/05/2026)
3. FinOps Foundation. *State of FinOps 2025 Report.* data.finops.org/2025-report/ (truy cập 21/05/2026)
4. FinOps Foundation. *FinOps for AI Overview.* finops.org/wg/finops-for-ai-overview/ (truy cập 21/05/2026)
5. artificialintelligence-news.com. *JPMorgan Chase AI strategy: US$18B bet paying off.* 2025
6. CNBC. *JPMorgan Chase's blueprint to become the world's first fully AI-powered megabank.* 30/09/2025
7. Capital One Software Blog. *Building a GenAI cost supervisor agent in Databricks.* capitalone.com/software/blog/databricks-genai-cost-supervisor-agent/
8. Agents Arcade. *Cost Modeling for Agentic Systems Before You Go to Production.* agentsarcade.com/blog/cost-modeling-agentic-systems-production
9. Drivetrain. *Unit economics for AI SaaS companies: A survival guide for CFOs.* drivetrain.ai/post/unit-economics-of-ai-saas-companies-cfo-guide
10. Vantage. *AI Cost Considerations Every Engineer Should Know.* vantage.sh/blog/ai-llm-pricing-dimensions
11. Gravitee. *How to Control the Hidden Costs of Generative AI.* gravitee.io/blog/how-to-control-the-hidden-costs-of-generative-ai
12. Apptio. *FinOps for AI: Enabling the Next Wave of Cloud Innovation.* apptio.com/blog/finops-for-ai-enabling-the-next-wave-of-cloud-innovation/
13. Databricks Blog. *Introducing AI spend controls with Unity AI Gateway.* databricks.com/blog/introducing-ai-spend-controls-unity-ai-gateway
14. AWS Blog. *Track Amazon Bedrock Costs by Caller Identity with IAM Principal-Based Cost Allocation.* aws.amazon.com/about-aws/whats-new/2026/04/bedrock-iam-cost-allocation/
15. FinOps Foundation. *2025 FinOps Framework.* finops.org/insights/2025-finops-framework/
16. Microsoft Tech Community. *Managing the cost of AI: Leveraging the FinOps Framework.* techcommunity.microsoft.com/blog/finopsblog/4381666
17. Databricks Blog. *Governing Coding Agent Sprawl with Unity AI Gateway.* databricks.com/blog/governing-coding-agent-sprawl-databricks-ai-gateway
18. Databricks Docs. *Monitor usage for Unity AI Gateway endpoints.* docs.databricks.com/aws/en/ai-gateway/usage-tracking-beta
19. Microsoft Learn. *Azure FinOps toolkit.* learn.microsoft.com Azure FinOps
20. ZenML Blog. *10 Best LLM Monitoring Tools to Use in 2025.* zenml.io/blog/best-llm-monitoring-tools
21. Portkey.ai. *The State of AI FinOps 2025.* portkey.ai/blog/the-state-of-ai-finops-2025-key-insights-from-finops-foundations-latest-report/
22. FinOps Foundation. *Cost Estimation of AI Workloads.* finops.org/wg/cost-estimation-of-ai-workloads/
23. Anthropic. *Vision pricing documentation.* docs.anthropic.com/en/docs/build-with-claude/vision
24. OpenAI. *Vision guide.* platform.openai.com/docs/guides/vision
25. Subhojyoti Singha. *The Hidden Cost of Embeddings.* Medium, 2025
26. Cogent. *AI-Native FinOps: Controlling GPU and LLM Cloud Costs.* cogentinfo.com/resources/ai-native-finops-controlling-gpu-and-llm-cloud-costs
27. Clarifai. *AI Cost Controls: Budgets, Throttling & Model Tiering.* clarifai.com/blog/ai-cost-controls
28. OpenTelemetry. *Semantic Conventions for Generative AI.* opentelemetry.io/docs/specs/semconv/gen-ai/
29. Databricks Blog. *Tracing for Generative AI on Databricks.* databricks.com/blog/tracing-generative-ai-databricks
30. Databricks Docs. *System tables billable usage.* docs.databricks.com/aws/en/admin/system-tables/billing
31. Anthropic. *Pricing.* anthropic.com/pricing
32. OpenAI. *API Pricing.* openai.com/api/pricing
33. Google. *Gemini API Pricing.* ai.google.dev/pricing
34. AWS. *Amazon Bedrock Pricing.* aws.amazon.com/bedrock/pricing
35. Databricks. *Foundation Model APIs.* docs.databricks.com/aws/en/machine-learning/foundation-model-apis/
36. Google. *Gemini vision pricing.* ai.google.dev/gemini-api/docs/vision
37. ElevenLabs. *Pricing.* elevenlabs.io/pricing
38. Deepgram. *Pricing.* deepgram.com/pricing
39. AssemblyAI. *Pricing.* assemblyai.com/pricing
40. AWS. *Amazon Polly Pricing.* aws.amazon.com/polly/pricing
41. Voyage AI. *Pricing.* voyageai.com/pricing
42. Cohere. *Pricing.* cohere.com/pricing
43. Twelve Labs. *Pricing.* twelvelabs.io/pricing
44. Anthropic. *Prompt caching documentation.* docs.anthropic.com/en/docs/build-with-claude/prompt-caching
45. OpenAI. *Prompt caching documentation.* platform.openai.com/docs/guides/prompt-caching
46. Google. *Context caching guide.* ai.google.dev/gemini-api/docs/caching
47. Logiciel.io. *Showback or Chargeback: What's Working for Engineering Accountability.* logiciel.io/blog/showback-chargeback-engineering-accountability
48. Portkey. *FinOps chargeback for GenAI platforms.* portkey.ai/blog/finops-chargeback-for-genai/
49. Finout. *GenAI Cost Allocation: The Complete Guide.* finout.io/blog/genai-cost-allocation-the-complete-guide
50. FinOps Foundation. *How to Build a Generative AI Cost and Usage Tracker.* finops.org/wg/how-to-build-a-generative-ai-cost-and-usage-tracker/
51. AWS Blog. *Introducing granular cost attribution for Amazon Bedrock.* aws.amazon.com/blogs/machine-learning/introducing-granular-cost-attribution-for-amazon-bedrock/
52. nOps. *AI Cost Visibility: The Ultimate Guide.* nops.io/blog/ai-cost-visibility-the-ultimate-guide/
53. FinOps Foundation. *Effect of Optimization on AI Forecasting.* finops.org/wg/effect-of-optimization-on-ai-forecasting/
54. Keito. *Why AI Agent Costs Are Unpredictable.* keito.ai/blog/ai-agent-costs-unpredictable-consumption/
55. Ridgeway Financial Services. *GPU Cost Forecasting, AI Unit Economics, and Infrastructure Strategy.* ridgewayfs.com/gpu-cost-forecasting-ai-unit-economics-infrastructure-strategy/
56. AWS Blog. *Build a proactive AI cost management system for Amazon Bedrock — Part 2.* aws.amazon.com/blogs/machine-learning/
57. Databricks Blog. *What's new in Unity AI Gateway: service policies, guardrails, observability, and cost controls.* databricks.com/blog/whats-new-unity-ai-gateway
58. Capital One Software Blog. *Building a GenAI cost supervisor agent in Databricks.* capitalone.com/software/blog/databricks-genai-cost-supervisor-agent/
59. Galileo. *A Guide to AI Agent Cost Optimization With Observability.* galileo.ai/blog/ai-agent-cost-optimization-observability
60. Databricks Blog. *Expanding agent governance with Unity AI Gateway.* databricks.com/blog/ai-gateway-governance-layer-agentic-ai
61. Firecrawl. *Best LLM Observability Tools in 2026.* firecrawl.dev/blog/best-llm-observability-tools
62. DigitalApplied. *Observability Stack TCO: LangSmith vs LangFuse vs Helicone.* digitalapplied.com/blog/observability-stack-tco-calculator-langsmith-langfuse-helicone
63. Softcery. *9 AI Observability Platforms Compared.* softcery.com/lab/top-8-observability-platforms-for-ai-agents-in-2025
64. Finout. *The Hidden Superpower of Bedrock Cost Allocation — and Its Limits.* finout.io/blog/the-hidden-superpower-of-bedrock-cost-allocation-and-its-limits
65. Laminar. *Langfuse Alternatives 2026.* laminar.sh/article/langfuse-alternatives-2026
66. Finout. *FinOps in the Age of AI: A CPO's Guide.* finout.io/blog/finops-in-the-age-of-ai-a-cpos-guide-to-llm-workflows-rag-ai-agents-and-agentic-systems

---

## Phụ lục C — Lịch sử bản ghi

| Phiên bản | Ngày | Nội dung thay đổi chính | Tác giả |
|---|---|---|---|
| v1.0 | 21/05/2026 | Consolidate từ 5 file core + 4 file appendix thành cẩm nang tổng hợp; thêm Glossary; thêm Bibliography đánh số tham chiếu | Cost Model Research Team |
| v0.3 | 21/05/2026 | Refactor cấu trúc folder cũ (7 folder × `_overview.md`) → 4 file flat + `_appendix/`; thêm `01_rate_card.md` với giá thật cho 5 provider × 9 modality; generalize công thức cost sang multimodal | — |
| v0.2 | 21/05/2026 | Setup governance: `CLAUDE.md` quy tắc tác giả, `PROGRESS.md` checklist, `PLAYBOOK.md` workflow | — |
| v0.1 | 20/05/2026 | Research baseline 7 chủ đề ban đầu | — |

---

**Kết thúc cẩm nang.**

> Cẩm nang này là deliverable Phase 0–1 của chương trình FinOps cho Agent Platform. Để tham gia review hoặc đóng góp cập nhật, xem `PLAYBOOK.md` §5 (template tạo file mới) và `PROGRESS.md` (backlog hiện tại).
