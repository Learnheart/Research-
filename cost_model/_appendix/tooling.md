# 06. Tooling Assessment — Build vs Buy + Vendor Comparison

> **Kết luận đầu (Pyramid):**
> Cho stack Databricks-first + Agentic AI nội bộ, **stack đề xuất là combo 3 layer**: (1) **Databricks Unity AI Gateway** làm cost-attribution backbone (native, không thay được); (2) **Langfuse OSS self-hosted** làm orchestration-layer observability (cover gap L3 mà Gateway chưa cover sâu); (3) **Databricks Lakehouse Monitoring + custom dashboard** thay vì FinOps platform commercial trong 12 tháng đầu — đến khi quy mô đủ để justify Finout/Apptio. **Build vs Buy**: Build chỉ ở layer connector/orchestrator-instrumentation; Buy/Adopt ở phần còn lại.

## 1. Build vs Buy Framework

| Capability | Recommend | Lý do |
|---|---|---|
| Cost data ingestion từ Databricks system tables | **Adopt** (native, không build) | Databricks expose sẵn `system.serving.*`, `system.billing.*` |
| Cost data từ external provider (OpenAI, Anthropic) khi đi direct | **Build connector** (light) | Provider API + tag normalization; effort thấp |
| Tagging enforcement | **Adopt + light Build** | Adopt: Unity Catalog + Cloud Policy; Build: CI/CD lint, request-time validator |
| LLM trace observability (L1) | **Adopt** (Databricks tracing + AI Gateway) | Native; không có lý do build |
| Orchestration trace (L3 — tool calls, retries, reflection) | **Adopt OSS** (Langfuse / OpenLLMetry) + light Build | Native chưa cover sâu; OSS đủ; OTel chuẩn để tránh lock-in |
| Cost dashboard | **Build trên Databricks SQL** | Linh hoạt; tận dụng skill SQL có sẵn; tránh license commercial sớm |
| Chargeback ledger | **Build** (light, trong Databricks) | Logic riêng theo tagging taxonomy của chúng ta |
| Anomaly detection | **Adopt** (Lakehouse Monitoring) Phase 1; có thể **Build** thêm Phase 2 | Bắt đầu native, tránh over-engineer |
| FinOps platform (cross-cloud, executive view) | **Defer** 12 tháng | Quy mô chưa đủ justify commercial; reassess khi ≥ $1M/quý AI spend [cần verify ngưỡng] |
| ROI / business outcome tracking | **Build** | Logic riêng cho từng use case; không vendor làm hộ |

## 2. So sánh Vendor — Cost Attribution & Observability cho LLM/Agent

### 2.1. Native cloud / platform

| Tool | Strengths | Weaknesses | Cost model | Phù hợp cho |
|---|---|---|---|---|
| **Databricks Unity AI Gateway** | Cost tracking per user/model/tag native; system tables; budget alerts; gateway cho external models; service policies (rate limit, guardrails); coding-agent governance | Đang phát triển; một số capability ở Beta (usage-tracking-beta); ecosystem ngoài Databricks vẫn cần connector | Inclued trong Databricks usage; serving DBU | **Core — bắt buộc với Databricks-first** |
| **AWS Bedrock cost allocation** | Application Inference Profiles; IAM principal-based allocation (CUR 2.0); tích hợp Cost Explorer + Budgets + Anomaly Detection | Chỉ Bedrock; nếu agent dùng external model phải tag riêng | Native trong AWS billing | Cần khi đi AWS Bedrock |
| **Azure OpenAI + Azure FinOps Hubs** | Resource tags; Azure Cost Management; FinOps reference architecture | Granularity per-call hạn chế hơn Databricks/Bedrock | Native Azure billing | Cần khi đi Azure OpenAI |

### 2.2. LLM Observability — OSS / Cloud

| Tool | Cost tracking strength | Pricing | Self-host | Phù hợp |
|---|---|---|---|---|
| **Langfuse** | Unit-based pricing (traces + observations + scores); cost per trace rõ; multi-framework | Hobby: 50K events free; Enterprise self-host | ✅ | **Recommended** cho L3 gap |
| **LangSmith** | Trace-level cost; integration LangChain tốt; OTel support từ 2025-03 | Seat-based; 5K traces free | Limited | Phù hợp nếu locked-in LangChain ecosystem |
| **Helicone** | Proxy-based, đổi base URL là xong; cost dashboard nhanh | 10K req/tháng free | ✅ | Phù hợp cho POC nhanh; weakness: thêm network hop |
| **OpenLLMetry** | Pure OpenTelemetry — plug vào Datadog/New Relic/Grafana | OSS | ✅ | Chỉ nên dùng nếu đã có observability backbone OTel-based |
| **Laminar** | Agent-focused observability, evaluation | Hybrid | ✅ | Strong cho evaluation, không phải core cost mgmt |

*Nguồn: ZenML "10 Best LLM Monitoring Tools to Use in 2025"; Firecrawl "Best LLM Observability Tools in 2026"; DigitalApplied "Observability Stack TCO".*

### 2.3. FinOps platform (commercial, cross-cloud)

| Tool | Strengths | Weaknesses cho use case của chúng ta | Khi nên reassess |
|---|---|---|---|
| **Finout** | GenAI cost allocation; Bedrock-aware; megabill view; chargeback engine | Chưa sâu ở orchestration layer; license fee đáng kể | Khi cost vượt $1M/quý [cần verify] và cần unified view multi-cloud |
| **Apptio** | TBM + chargeback enterprise; AI cost roadmap | Heavyweight, deployment lâu; phù hợp tổ chức lớn đã dùng Apptio | Khi đã có Apptio cho IT cost mgmt |
| **nOps** | Cloud cost optimization; AI visibility; commitment management | Cloud-cost-first, AI là feature thêm | Khi quá tải native tooling |
| **Vantage** | Multi-cloud cost view; ngon UX; AI pricing dimensions tốt | Chưa đủ depth cho chargeback complex | Cho exec visibility nhanh |
| **Capital One Slingshot** | Sâu Databricks; có GenAI Cost Supervisor agent reference | Hẹp Databricks-only; phải mua | Khi quy mô Databricks lớn và cần optimization layer sâu |

## 3. Stack Đề xuất (cho Phase 1–2)

```
┌──────────────────────────────────────────────────────────────────┐
│  EXECUTIVE / FINANCE LAYER                                       │
│  - Custom dashboards (Databricks SQL + AI/BI)                    │
│  - Monthly chargeback report (export → ERP)                      │
└──────────────────────────────────────────────────────────────────┘
              ▲                              ▲
              │                              │
┌─────────────────────────────────┐   ┌──────────────────────────────┐
│  COST & FINOPS LOGIC LAYER       │   │  ML / ANOMALY DETECTION    │
│  - Allocation engine (Build,     │   │  - Databricks Lakehouse    │
│    Databricks notebooks + jobs)  │   │    Monitoring              │
│  - Tagging policy enforcer       │   │  - Custom alert rules      │
│  - Chargeback ledger              │   │                            │
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

## 4. Total Cost of Ownership — 12 tháng đầu (rough estimate)

> **[cần verify]** — Số dưới là điểm xuất phát để thảo luận, không phải báo giá. Cần RFQ chính thức trước khi sign-off.

| Hạng mục | Build / Buy | Year-1 cost (USD, ước lượng) | Ghi chú |
|---|---|---|---|
| Databricks Unity AI Gateway | Native (đã có) | $0 incremental | Đã có trong contract Databricks |
| Langfuse OSS self-hosted | Adopt | $5K–15K (compute + ops) | Chạy trên hạ tầng có sẵn |
| OTel instrumentation lib | Build | 0.5 FTE × 3 tháng ≈ $30K | Một lần |
| Allocation engine | Build | 1 FTE × 4 tháng ≈ $80K | Một lần, sau đó maintenance |
| Custom dashboards | Build | 0.5 FTE × 2 tháng ≈ $20K | Một lần |
| Lakehouse Monitoring | Native | Included | DBU consumption |
| FinOps platform commercial | Defer | $0 | Reassess sau 12 tháng |
| Total Year-1 (Build phase + ops) | | **~$135K–145K** | Phần lớn là FTE, không phải license |

## 5. Khi nào tái đánh giá

| Trigger | Hành động |
|---|---|
| Spend AI > $1M/quý ổn định 2 quý | Reassess FinOps commercial platform (Finout/Apptio) |
| Multi-cloud (Databricks + native AWS + native Azure) | Cân nhắc tool unify cross-cloud |
| Số use case > 30 active prod | Cân nhắc commercial cho governance scale |
| Đội FinOps < 2 FTE và workload tăng | Mua commercial thay vì hire |
| Audit / regulator yêu cầu reporting chuẩn TBM | Cân nhắc Apptio |

## 6. Risk & Mitigation

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Databricks AI Gateway feature gap (đang phát triển nhanh) | Medium | Medium | Theo Beta release notes; quarterly capability review |
| Langfuse self-host operational burden | Medium | Low–Medium | Định nghĩa SLA nội bộ; pin version; có upgrade window |
| Tagging compliance trượt khi user tăng | High | High | Enforcement tự động ở Gateway + CI/CD; quarterly audit |
| Vendor pricing tăng đột biến | Medium | High | Multi-provider; provisioned throughput negotiation |
| Build cost vượt estimate | Medium | Medium | Time-box mỗi component; ưu tiên adopt over build khi possible |

## Nguồn

- ZenML Blog. *10 Best LLM Monitoring Tools to Use in 2025.* zenml.io/blog/best-llm-monitoring-tools
- Firecrawl. *Best LLM Observability Tools in 2026.* firecrawl.dev/blog/best-llm-observability-tools
- Laminar. *Langfuse Alternatives 2026.* laminar.sh/article/langfuse-alternatives-2026
- DigitalApplied. *Observability Stack TCO: LangSmith vs LangFuse vs Helicone.* digitalapplied.com/blog/observability-stack-tco-calculator-langsmith-langfuse-helicone
- Softcery. *9 AI Observability Platforms Compared.* softcery.com/lab/top-8-observability-platforms-for-ai-agents-in-2025
- Databricks Blog. *Unity AI Gateway: A Single Place to Govern Your AI Estate.* databricks.com/product/artificial-intelligence/ai-gateway
- Databricks Blog. *Expanding agent governance with Unity AI Gateway.* databricks.com/blog/ai-gateway-governance-layer-agentic-ai
- AWS Blog. *Manage AI costs with Amazon Bedrock Projects.* aws.amazon.com/blogs/machine-learning/manage-ai-costs-with-amazon-bedrock-projects/
- Finout. *The Hidden Superpower of Bedrock Cost Allocation — and Its Limits.* finout.io/blog/the-hidden-superpower-of-bedrock-cost-allocation-and-its-limits
- Capital One Software. *Cost allocation.* capitalone.com/software/products/slingshot/cost-allocation/
- Apptio. *FinOps for AI: Enabling the Next Wave of Cloud Innovation.* apptio.com/blog/finops-for-ai-enabling-the-next-wave-of-cloud-innovation/
