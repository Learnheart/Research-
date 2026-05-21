# 04. Competitive Analysis — Overview

**Workstream**: 04_Competitive_Analysis
**File ID**: 04-00
**Version**: v0.1
**Status**: DRAFT
**Owner**: TBD
**Reviewer**: Head of Data/AI
**Last updated**: 2026-05-21
**Classification**: Internal

> **Mục tiêu workstream**: So sánh Databricks AI Agent stack với 6 nhóm đối thủ trên **cùng tiêu chí, cùng năm, cùng version**, đầu ra là một **feature matrix** + **positioning narrative** dùng cho Executive Summary và Roadmap.
>
> **Key questions**:
> - [ANALYSIS] Databricks **thắng** ở capability nào, **thua** ở capability nào — bằng chứng cụ thể?
> - [ANALYSIS] Decision tree: khi nào nên chọn Databricks, khi nào chọn hyperscaler, khi nào chọn OSS / native LLM provider?
> - [ANALYSIS] Mức độ **interoperability** giữa các stack — có thể mix-and-match (vd dùng Databricks UC + AWS Bedrock model) không?
>
> **Audience chính**: CIO, Head of Data/AI, Architecture review board.

---

## 1. Scope

### 1.1 In-scope

- **6 nhóm đối thủ**: Snowflake Cortex; AWS Bedrock + SageMaker; Azure AI Foundry; GCP Vertex AI Agent Builder; native LLM providers (OpenAI Assistants / Anthropic Claude with tools); OSS stack (LangChain + LlamaIndex + self-managed infra).
- So sánh trên **8 trục**: agent framework, model availability, governance, retrieval/RAG, evaluation, MLOps, security/compliance, pricing/TCO indicator.
- **Decision tree** cho 4 use case archetype phổ biến trong banking (RAG Q&A, RM copilot, fraud co-pilot, ops automation).

### 1.2 Out-of-scope

- Multi-cloud failover architecture chi tiết (đẩy sang `../06_Strategic_Roadmap/`).
- Vendor negotiation, pricing đàm phán (`../README.md §2`).
- So sánh consumer-facing chatbot SaaS (ChatGPT Enterprise, Copilot consumer, etc.).

## 2. Strategic context

- [ANALYSIS] Tránh "marketing comparison" — mỗi vendor sẽ có self-published comparison chart, **không** dùng làm primary source.
- [ANALYSIS] Same-version, same-date là **bắt buộc** (`../RULES.md §5`): nếu so Databricks v2026-05 với Snowflake v2024-11 → invalid.
- [ASSUMPTION] Giả định 2 trong 3 hyperscaler đều khả dụng tại region target. Nếu Azure / AWS bị giới hạn region tại VN → decision tree đổi. Impact: high.

## 3. Dàn ý file con

| File | Mục tiêu (1–2 câu) | Owner | Trạng thái |
|---|---|---|---|
| `_overview.md` | (file này) Scope, methodology, dàn ý 6 file đối thủ + 1 matrix. | TBD | DRAFT |
| `01_vs_snowflake_cortex.md` | So sánh Databricks vs Snowflake Cortex (Cortex AI, Cortex Agents, Native Apps). | TBD | NOT_STARTED |
| `02_vs_aws_bedrock_sagemaker.md` | So sánh với AWS Bedrock + Bedrock Agents + SageMaker (Studio, JumpStart, Clarify). | TBD | NOT_STARTED |
| `03_vs_azure_ai_foundry.md` | So sánh với Azure AI Foundry / Studio + Azure OpenAI + Azure ML. | TBD | NOT_STARTED |
| `04_vs_gcp_vertex.md` | So sánh với GCP Vertex AI Agent Builder + Vertex Model Garden + Search & Conversation. | TBD | NOT_STARTED |
| `05_vs_openai_anthropic_native.md` | So sánh với native LLM provider stack: OpenAI Assistants API, Anthropic Claude tool use, Mistral / Cohere; ưu nhược kiến trúc thuần API. | TBD | NOT_STARTED |
| `06_vs_oss_langchain_llamaindex.md` | So sánh với OSS stack (LangChain, LlamaIndex, LangGraph + self-managed infra); cost + flexibility + maintenance burden. | TBD | NOT_STARTED |
| `07_feature_matrix.md` | Matrix tổng hợp 8 trục × 7 vendor (Databricks + 6 đối thủ). Đầu ra ăn vào Exec Summary. | TBD | NOT_STARTED |

## 4. Methodology bắt buộc

1. **Same date**: snapshot 2026-05-21 (đồng bộ với workstream 03).
2. **Same version**: ghi rõ product version + GA/preview cho mỗi vendor.
3. **Same source-tier preference**: vendor docs (T1) + ≥1 analyst T2 + ≥1 third-party benchmark T2/T3 nếu có.
4. **Same evaluation rubric**: 8 trục dùng chung scoring 0–3 + evidence pointer; cấm "✓ / ✗" trần.
5. **Disclosure**: nếu file có data sponsored bởi vendor (whitepaper "X vs Y") → ghi `**Disclosure**:` ở header.

## 5. Key questions phải trả lời (chi tiết)

1. **Snowflake Cortex** (Q3-2024 GA Agents) — coverage agent abstraction so với Mosaic AI Agent Framework, đặc biệt tool calling + multi-step?
2. **AWS Bedrock Agents** + Knowledge Bases — có gì Databricks chưa có (multi-agent collaboration GA?), và ngược lại (Unity Catalog tương đương)?
3. **Azure AI Foundry** — Microsoft đặt Foundry là layer thống nhất, mức độ integration với Microsoft Fabric vs Databricks Lakehouse?
4. **GCP Vertex AI Agent Builder** + Gemini multimodal — strength về search/conversation, yếu về data platform integration?
5. **Native LLM provider** (OpenAI/Anthropic) — TCO ưu thế ở quy mô nhỏ, mất ưu thế từ ngưỡng nào (token volume / month)?
6. **OSS stack** — flexibility cao nhất, nhưng maintenance burden bao nhiêu FTE? Khi nào break-even?
7. Mức độ **interop**: dùng Databricks làm data + UC, gọi model qua Bedrock/Azure OpenAI — đã production-ready?
8. **Exit cost**: từ Databricks chuyển sang AWS/Azure mất bao nhiêu tháng và mức rework code/data nào?

## 6. Cross-references (dependencies)

| Liên quan tới | Mục đích |
|---|---|
| ← `../03_Databricks_Technology/` (toàn bộ) | Lấy feature set chuẩn của Databricks để so |
| → `../06_Strategic_Roadmap/04_coverage_matrix.md` | Output decision tree đi vào roadmap |
| → `../07_Business_Case_And_Risk/04_build_vs_buy.md` | Output OSS comparison đi vào build-vs-buy |
| → `../01_Executive_Summary/01_key_findings.md` | Output feature matrix là appendix bắt buộc |

## 7. Sources & evidence plan

| Vendor | Primary source (T1) | Analyst (T2) | Independent (T2/T3) |
|---|---|---|---|
| Snowflake Cortex | docs.snowflake.com/en/guides-overview-ai-features | Gartner MQ DSML, Forrester Wave | dbt-snowflake comparison blog (cẩn trọng bias) |
| AWS Bedrock + SageMaker | docs.aws.amazon.com/bedrock/, sagemaker | Gartner, Forrester | re:Invent technical session recap |
| Azure AI Foundry | learn.microsoft.com/azure/ai-studio | Gartner, Forrester | MS Build session recap |
| GCP Vertex AI Agent Builder | cloud.google.com/vertex-ai/docs/agent-builder | Gartner, Forrester | Google I/O session recap |
| OpenAI / Anthropic | platform.openai.com, docs.anthropic.com | – | Independent eval (LMSys, HELM) |
| OSS (LangChain / LlamaIndex) | langchain.com/docs, llamaindex.ai/docs | – | GitHub star/issue trend, contributor count |

[GAP] Một số sản phẩm (Bedrock Agent multi-agent, Vertex Agent Builder) **đổi feature set rất nhanh** trong 6 tháng gần đây. Owner: TBD. Due: re-snapshot 2026-08.

## 8. Risks & GAPs

| ID | Risk | Severity | Mitigation |
|---|---|---|---|
| R-Comp-01 | Vendor self-comparison bias dẫn dắt findings | H | Cấm dùng vendor "vs" page làm primary; mỗi claim cần T1 hai phía + T2 độc lập |
| R-Comp-02 | Feature parity thay đổi giữa snapshot và Exec Summary review | M | Bắt buộc re-snapshot 2026-08; flag delta trong change log từng file |
| R-Comp-03 | OSS comparison khó định lượng maintenance burden | M | Dùng ≥3 reference (industry survey, Hugging Face stat, McKinsey ML survey) |

## 9. Definition of Done — workstream level

- [ ] `_overview.md` APPROVED.
- [ ] 7 file con APPROVED (6 vendor + 1 matrix).
- [ ] Feature matrix 8 trục × 7 vendor, mỗi cell có score 0–3 + ≥1 evidence pointer.
- [ ] Decision tree cho 4 use case archetype.
- [ ] Mọi snapshot có cùng date + cùng version note.
- [ ] Cross-check với `../03_Databricks_Technology/` — không có claim Databricks nào mâu thuẫn 2 workstream.
- [ ] `../WORKSPACE_TRACKING.md` cập nhật status.

## 10. Change log

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-05-21 | v0.1 | Khởi tạo overview Competitive Analysis | Lead Consultant |
