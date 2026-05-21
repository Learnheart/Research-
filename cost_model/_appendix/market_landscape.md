# 01. Market Landscape — FinOps cho Agentic AI ở Enterprise

> **Kết luận đầu (Pyramid Principle):**
> FinOps cho AI đã chuyển từ giai đoạn "experimental" sang "core operating discipline" trong 12 tháng qua. **63% enterprise đã track AI spend (so với 31% năm trước)**, FinOps Foundation đã ban hành certification riêng cho FinOps for AI, và 3 hyperscaler (AWS / Azure / Databricks) đều đã ra mắt native cost-attribution capability cho LLM/Agent workload trong 2024–2025. Tuy nhiên, **chỉ 14.2% organizations đạt "Run" maturity** — cửa sổ cơ hội để dẫn trước vẫn còn rộng.

## 1. Bối cảnh & Drivers thúc đẩy

| Driver | Mô tả | Bằng chứng |
|---|---|---|
| **Bùng nổ AI spend** | "Managing AI/ML spend" tăng +4 hạng, "unit economics" tăng +5 hạng trong priority list của FinOps practitioners 2025 | FinOps Foundation, State of FinOps 2025 |
| **Pricing model phức tạp** | Pay-per-token + pay-per-call + provisioned throughput + cached tokens + reserved capacity → khó dự báo | Vantage, "AI Cost Considerations" |
| **Agentic workflow opacity** | Một user request kích hoạt 5–6 micro-calls ẩn (search, RAG, tool use, reflection) | Gravitee, "Hidden Costs of GenAI" |
| **Regulator & Board pressure** | Board yêu cầu ROI rõ ràng cho AI initiatives; CFO cần unit economics | FinOps Foundation 2025, Apptio |
| **Vendor ecosystem chín muồi** | Databricks Unity AI Gateway, AWS Bedrock cost allocation, Azure FinOps blueprint đều ra mắt cost-attribution capability 2024–2025 | Databricks Blog 2025, AWS Blog 2025–2026 |

## 2. Framework chuẩn ngành: FinOps Foundation 2025

FinOps Foundation 2025 framework tái cấu trúc gồm **3 trụ cột**: Domains (6 lĩnh vực), Capabilities (năng lực thực thi), **Scopes** (phạm vi áp dụng — mới). **Scope thứ 4 — AI** (tokens, models, GPU compute) chính thức là first-class citizen.

| Domain | Capability nổi bật liên quan AI |
|---|---|
| **Understand Usage & Cost** | Cost attribution theo agent / use case / IAM principal |
| **Quantify Business Value** | Unit economics cho AI; cost-per-business-outcome |
| **Manage Anomalies** | Anomaly detection trên token spike, retry storm |
| **Manage Commitments** | Provisioned throughput vs on-demand trade-off |
| **Plan & Forecast** | Driver-based forecasting cho AI workload |
| **Optimize Rate / Usage** | Model tiering, prompt compression, caching |

*Nguồn: FinOps Foundation, "Framework 2025" — finops.org/insights/2025-finops-framework/*

## 3. Case Study Fortune 500 (có nguồn)

| Tổ chức | Bài toán | Cách tiếp cận | Kết quả / Insight | Nguồn |
|---|---|---|---|---|
| **JPMorgan Chase** | Quản lý 450+ AI use cases trong production trên budget $18B công nghệ/năm | LLM Suite nội bộ + governance layer + deploy "agentic AI" cho complex multistep | Ước tính $1.5B value/năm; consumer banking ops giảm ≥10% headcount; COiN tiết kiệm 360K labor hours/năm | JPMorgan AI strategy report 2025 (artificialintelligence-news.com); CNBC 2025-09-30 |
| **Capital One (Slingshot)** | Visibility/optimization cho Databricks GenAI spend | Xây "GenAI Cost Supervisor Agent" trên Databricks, chạy 20 Unity Catalog SQL functions, mọi agent interaction traced trong `system.serving.endpoint_usage` | Mọi user có "personal GenAI cost analyst"; cost supervisor tự tracking cost của chính nó (recursive observability) | Capital One Software Blog 2025; capitalone.com/software |
| **Microsoft (internal & blueprint)** | Quản lý cost cho Copilot ecosystem + Azure OpenAI tenancy | FinOps framework adoption + Azure Cost Management with AI tags; FinOps Hubs reference architecture | Public blueprint cho enterprise customer; áp dụng nội bộ trước khi ra ngoài | Microsoft Tech Community, "Managing the cost of AI: Leveraging the FinOps Framework" 2025 |
| **Databricks (dogfooding)** | Govern coding-agent sprawl (Cursor, Codex CLI, Gemini CLI dùng nội bộ) | Route mọi request qua Unity AI Gateway → one invoice, one dashboard, one set of rate limits | Single pane of glass cho coding-agent ROI tracking | Databricks Blog, "Governing Coding Agent Sprawl" 2025 |
| **Apptio / IBM customer base** | Chargeback cho GenAI spend tới business unit | Apptio TBM + cost-allocation tags ở model/inference layer | Reference patterns được công bố trong "FinOps for AI: Enabling the Next Wave" | Apptio Blog 2025 |

> **[cần verify]** Số liệu chi tiết hơn về cost-per-agent ở JPMorgan và Capital One chưa được công bố public; số trên là ROI/labor hour, không phải cost của chính agent platform.

## 4. Vendor & Partner Landscape (Databricks / AWS / Azure ecosystem)

### 4.1. Native capabilities

| Hyperscaler | Capability | Granularity | Trạng thái | Nguồn |
|---|---|---|---|---|
| **Databricks Unity AI Gateway** | Cost tracking + budget alerts theo user, use case, model, provider, tag; system tables `system.serving.*` | Per-request, per-token, per-user, per-tag | GA + beta (usage-tracking-beta) | docs.databricks.com/aws/en/ai-gateway/usage-tracking-beta |
| **AWS Bedrock** | Application Inference Profiles + cost allocation tags + IAM principal-based allocation trong CUR 2.0 | Per-IAM principal, per-tag, per-project | GA (announced 2026-04) | aws.amazon.com/about-aws/whats-new/2026/04/bedrock-iam-cost-allocation/ |
| **Azure OpenAI** | Azure Cost Management + resource tags + Azure FinOps toolkit + FinOps Hubs | Per-resource, per-tag, per-deployment | GA | learn.microsoft.com Azure FinOps |

### 4.2. Third-party tooling

| Loại | Tool | Điểm mạnh | Điểm yếu | Free tier |
|---|---|---|---|---|
| LLM Observability — Cloud | **LangSmith** | Trace-level cost ($/run); tích hợp sâu LangChain; OTel support từ 2025-03 | Seat-based pricing; vendor lock với LangChain stack | 5K traces/tháng |
| LLM Observability — OSS Cloud | **Langfuse** | Unit-based pricing; self-host được; phù hợp multi-framework | UI ít polish hơn LangSmith ở mức enterprise | 50K events/tháng (Hobby) |
| LLM Observability — Proxy | **Helicone** | Đổi base URL là xong, không cần SDK; cost dashboard nhanh | Là proxy → thêm hop network, có thể ảnh hưởng latency | 10K requests/tháng |
| Observability — OTel native | **OpenLLMetry** | Pure OpenTelemetry; plug vào Datadog, New Relic, Grafana đã có | Không có dashboard riêng; phải tự build phía consumer | OSS |
| FinOps platform | **Finout / nOps / Vantage / Apptio** | Cross-cloud cost view, chargeback engine, anomaly detection trưởng thành | Chưa cover sâu agent/orchestration layer; cần dữ liệu source-of-truth | Trial |
| Partner trên Databricks | **Capital One Slingshot** | Cost allocation + optimization riêng cho Databricks; có GenAI Cost Supervisor pattern | Hẹp ở Databricks ecosystem | Commercial |

*Nguồn tổng hợp: ZenML, Firecrawl, Laminar, Softcery — comparison reports 2025–2026; tham khảo thêm finout.io blog "Hidden Superpower of Bedrock Cost Allocation".*

## 5. Khoảng trống ngành (Gap Analysis)

| Gap | Implication với platform của chúng ta |
|---|---|
| **Orchestration-layer cost vẫn opaque** với hầu hết tooling — chỉ trace LLM call, không trace tool call / retry / reflection | Phải tự instrument ở orchestrator (LangChain / LangGraph / custom) bằng OTel hoặc tương đương |
| **Cost-per-business-outcome** (cost per resolved ticket, per BRD generated) **chưa có chuẩn** | Định nghĩa rõ "business unit of value" cho từng use case là việc của chúng ta, không vendor làm hộ |
| **Forecasting AI workload chính xác là khó** — usage tăng phi tuyến khi user adopt | Phải sống chung với scenario modeling; không kỳ vọng forecast accuracy >80% trong 6 tháng đầu |
| **Tagging discipline thiếu** — chỉ 14.2% tổ chức "Run" maturity | Tagging policy + enforcement phải làm sớm; chậm là phải migrate đau đớn |

## 6. Hàm ý cho Agent Platform của chúng ta

1. **Không build cost model từ con số 0** — kế thừa FinOps Foundation 2025 framework + Databricks Unity AI Gateway capabilities.
2. **Multi-layer cost taxonomy là bắt buộc** — token cost-only sẽ thiếu 50–70% bức tranh (xem `02_cost_taxonomy/`).
3. **Tagging là điều kiện tiên quyết** — không có tagging discipline thì không có chargeback (xem `03_allocation_framework/`).
4. **Build vs Buy quyết định sớm** — Databricks-native + 1 OSS observability (Langfuse) là combo phổ biến nhất cho Databricks-first stack (xem `06_tooling_assessment/`).
5. **Scenario forecasting**, không point estimate — usage AI phi tuyến (xem `04_forecasting/`).

## Nguồn

- FinOps Foundation. *State of FinOps 2025 Report.* data.finops.org/2025-report/
- FinOps Foundation. *FinOps for AI Overview.* finops.org/wg/finops-for-ai-overview/
- FinOps Foundation. *2025 FinOps Framework.* finops.org/insights/2025-finops-framework/
- FinOps Foundation. *Cost Estimation of AI Workloads.* finops.org/wg/cost-estimation-of-ai-workloads/
- Microsoft Tech Community. *Managing the cost of AI: Leveraging the FinOps Framework.* techcommunity.microsoft.com/blog/finopsblog/4381666
- Databricks Blog. *Introducing AI spend controls with Unity AI Gateway.* databricks.com/blog/introducing-ai-spend-controls-unity-ai-gateway
- Databricks Blog. *Governing Coding Agent Sprawl with Unity AI Gateway.* databricks.com/blog/governing-coding-agent-sprawl-databricks-ai-gateway
- Databricks Docs. *Monitor usage for Unity AI Gateway endpoints.* docs.databricks.com/aws/en/ai-gateway/usage-tracking-beta
- AWS Blog. *Track, allocate, and manage your generative AI cost and usage with Amazon Bedrock.* aws.amazon.com/blogs/machine-learning/
- AWS Blog. *Track Amazon Bedrock Costs by Caller Identity with IAM Principal-Based Cost Allocation.* aws.amazon.com/blogs/aws-cloud-financial-management/
- Capital One Software Blog. *Building a GenAI cost supervisor agent in Databricks.* capitalone.com/software/blog/databricks-genai-cost-supervisor-agent/
- artificialintelligence-news.com. *JPMorgan Chase AI strategy: US$18B bet paying off.* 2025
- CNBC. *JPMorgan Chase's blueprint to become the world's first fully AI-powered megabank.* 2025-09-30
- Galileo. *The Hidden Costs of Agentic AI.* galileo.ai/blog/hidden-cost-of-agentic-ai
- Gravitee. *How to Control the Hidden Costs of Generative AI.* gravitee.io/blog/how-to-control-the-hidden-costs-of-generative-ai
- Vantage. *AI Cost Considerations Every Engineer Should Know.* vantage.sh/blog/ai-llm-pricing-dimensions
- Apptio. *FinOps for AI: Enabling the Next Wave of Cloud Innovation.* apptio.com/blog/finops-for-ai-enabling-the-next-wave-of-cloud-innovation/
- ZenML Blog. *10 Best LLM Monitoring Tools to Use in 2025.* zenml.io/blog/best-llm-monitoring-tools
- Portkey.ai. *The State of AI FinOps 2025.* portkey.ai/blog/the-state-of-ai-finops-2025-key-insights-from-finops-foundations-latest-report/
