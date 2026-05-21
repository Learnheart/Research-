# 03. Databricks Technology — Overview

**Workstream**: 03_Databricks_Technology
**File ID**: 03-00
**Version**: v0.1
**Status**: DRAFT
**Owner**: TBD
**Reviewer**: Head of Data/AI
**Last updated**: 2026-05-21
**Classification**: Internal

> **Mục tiêu workstream**: Mô tả **chính xác và trung lập** các component lõi của Databricks phục vụ xây AI Agent — kèm version, GA/preview status, dependency, và open questions kỹ thuật — để workstream 04 (competitive) và 07 (TCO/risk) làm việc trên một feature set thống nhất.
>
> **Key questions**:
> - [FACT] Components nào đã **GA** vs vẫn **preview / private preview** tại snapshot date?
> - [ANALYSIS] Mosaic AI Agent Framework là một **framework đầy đủ** hay vẫn là tập hợp các primitive cần "đóng gói" thêm?
> - [ANALYSIS] Mức độ **vendor lock-in** ở mỗi component (Unity Catalog, MLflow, Model Serving, Vector Search) — exit path nào khả thi?
>
> **Audience chính**: Head of Data/AI, CIO, Architecture review board.

---

## 1. Scope

### 1.1 In-scope

- **7 component lõi**: Mosaic AI Agent Framework, Unity Catalog (governance), Model Serving (inference), Vector Search, Agent Evaluation + MLflow, Lakehouse foundation (Delta + Workflow), Genie / Databricks Apps.
- Mỗi component: capability, dependency, version, GA/preview, integration surface, limitation đã được vendor công bố hoặc cộng đồng xác nhận.
- **So sánh tiến hoá 2 giai đoạn**: trước 2026 (Mosaic ML acquisition + pre-Agent-Framework primitive) vs từ 2026 (Agent Framework + Genie GA + Lakehouse Apps).
- Architecture reference cho 2–3 deployment patterns điển hình (single-region, multi-region, hybrid với on-prem data).

### 1.2 Out-of-scope

- Setup hạ tầng chi tiết (Terraform, network) — `../README.md §2`.
- Production code sample (chỉ pseudo-code minh hoạ).
- So sánh giá với competitor (đẩy sang `../07_Business_Case_And_Risk/03_pricing_analysis.md`).

## 2. Strategic context

- [FACT] Tài liệu nguồn chính: <https://docs.databricks.com/aws/en/agents/> — vendor official, **T1 nhưng có thiên kiến tích cực** (`../RULES.md §4.3`).
- [ANALYSIS] Vì là vendor docs, mọi claim ảnh hưởng recommendation phải cross-check bằng ≥1 nguồn T2/T3 độc lập (benchmark, analyst report, third-party blog có data).
- [ASSUMPTION] Snapshot công nghệ tại **2026-05-21**. Vendor release cadence Databricks ~mỗi 2 tuần — sẽ re-snapshot vào 2026-08 trước khi finalize Exec Summary. Impact nếu skip: medium (feature mới có thể đảo competitive comparison).

## 3. Dàn ý file con

| File | Mục tiêu (1–2 câu) | Owner | Trạng thái |
|---|---|---|---|
| `_overview.md` | (file này) Scope, dàn ý, key questions, snapshot policy. | TBD | DRAFT |
| `01_mosaic_ai_agent_framework.md` | Anatomy của Agent Framework: tool calling, multi-step orchestration, state management, tracing; GA/preview status từng feature. | TBD | NOT_STARTED |
| `02_unity_catalog_governance.md` | UC dùng cho data + model + tool + function governance; policy enforcement; lineage; ABAC; exit path. | TBD | NOT_STARTED |
| `03_model_serving_inference.md` | Provisioned throughput vs pay-per-token endpoint, foundation model API, batch inference, autoscaling, SLA cam kết. | TBD | NOT_STARTED |
| `04_vector_search_rag.md` | Vector Search architecture (Delta-sync vs direct), embedding model options, hybrid search, retrieval quality metric. | TBD | NOT_STARTED |
| `05_agent_evaluation_mlflow.md` | Agent Evaluation framework, LLM-as-judge, ground truth dataset, MLflow tracking, A/B testing, regression detection. | TBD | NOT_STARTED |
| `06_lakehouse_foundation.md` | Delta + Unity Catalog + Workflow là baseline data plane cho agent; data freshness pattern; streaming vs batch ingestion. | TBD | NOT_STARTED |
| `07_genie_assistant_apps.md` | Genie (NL→SQL), Databricks Apps, AI/BI dashboard — front-end options cho deliver agent đến business user. | TBD | NOT_STARTED |

## 4. Key questions phải trả lời (chi tiết)

1. Agent Framework hỗ trợ **multi-agent collaboration** (planner / worker pattern) hay chỉ single-agent với tool?
2. Tool calling: schema, max nested depth, max parallelism, tool authorization (per-user vs per-agent)?
3. Foundation model: vendor cho phép Anthropic Claude, OpenAI GPT, Llama, Mistral — model nào host trong region nào? Pricing differ?
4. Unity Catalog ABAC: policy có hỗ trợ row-level / column-level cho agent context không? Limitation?
5. Vector Search: hỗ trợ embedding model nào trong region APAC / EU? Sync latency Delta→Vector ms / second / minute?
6. Agent Evaluation: ground truth dataset cần bao nhiêu sample minimum để LLM-as-judge có statistical power?
7. MLflow Tracking + Model Registry: tích hợp với external registry (SageMaker MR, Azure ML) hay khoá trong Databricks?
8. Hybrid deployment: agent run on Databricks nhưng data lakehouse on-prem (Iceberg / open table format) — pattern khả thi không?
9. Genie hỗ trợ Tiếng Việt và domain banking đặc thù tới đâu? Cần fine-tune semantic layer bao lâu?

## 5. So sánh tiến hoá công nghệ (Giai đoạn 1 vs 2)

> File `01_mosaic_ai_agent_framework.md` sẽ chi tiết hoá. Tóm tắt scope so sánh:

| Trục so sánh | Trước 2026 (G1) | Từ 2026 (G2) |
|---|---|---|
| Agent abstraction | Primitive: LangChain on Databricks, MLflow flavors | Mosaic AI Agent Framework (native) |
| Model hosting | Endpoint-per-model, manual | Foundation Model APIs + pay-per-token + provisioned throughput |
| Governance | Hive Metastore + ad-hoc IAM | Unity Catalog hợp nhất data + model + function |
| Retrieval | Self-managed vector DB (Pinecone, Weaviate) | Vector Search managed, Delta-sync |
| Evaluation | Custom code, ad-hoc | Agent Evaluation + LLM-as-judge, ground truth dataset chuẩn |
| Front-end | Notebook / external app | Databricks Apps + Genie + AI/BI |

(File 01–07 sẽ verify từng dòng + dẫn link doc + ngày GA.)

## 6. Cross-references (dependencies)

| Liên quan tới | Mục đích |
|---|---|
| → `../04_Competitive_Analysis/07_feature_matrix.md` | Cung cấp feature set chuẩn để so sánh ngang |
| → `../07_Business_Case_And_Risk/01_tco_model.md` | Cung cấp component breakdown cho TCO |
| → `../07_Business_Case_And_Risk/05_data_residency_sovereignty.md` | Cung cấp region availability cho phân tích residency |
| → `../07_Business_Case_And_Risk/07_security_threat_model.md` | Cung cấp architecture cho threat model |
| ← `../05_Banking_FSI_Applications/04_use_cases_catalog.md` | Use case yêu cầu capability nào — cross-validate |

## 7. Sources & evidence plan

| Nguồn | Tier | Cách dùng |
|---|---|---|
| docs.databricks.com/aws/en/agents/ và subpages | T1 (vendor) | Primary cho mỗi component, **phải kèm ngày access** |
| Databricks release notes (theo URL trên) | T1 | Source-of-truth cho GA / preview status |
| Databricks blog (Data + AI Summit recap, technical deep-dive) | T4 | Bổ sung context, **tag vendor view** |
| Gartner MQ DSML 2025, Forrester Wave AI Platforms 2025 | T2 | Vị thế thị trường, đánh giá maturity |
| Third-party benchmark (LMSys, MLPerf, Hugging Face leaderboard) | T2/T3 | Performance metric độc lập |
| Cộng đồng (HN, r/Databricks, dbt slack — chỉ làm hint) | T5 | **Không** dùng làm evidence cho recommendation |

## 8. Risks & GAPs

| ID | Risk | Severity | Mitigation |
|---|---|---|---|
| R-Tech-01 (= R-05 tracking) | Feature set evolve nhanh, GA/preview không rõ ràng | M | Snapshot date trong header mỗi file con; re-snapshot trước final review |
| R-Tech-02 | Vendor docs thiếu performance number cụ thể (latency P50/P99, throughput) | H | Note `[GAP]` + đề xuất benchmark trong POC (workstream 06) |
| R-Tech-03 | Multi-cloud claim của Databricks (AWS/Azure/GCP) — feature parity có thể không đầy đủ | M | Tách 3 cột so sánh trong feature matrix nếu cần |
| [GAP] | Chưa có data về Databricks region availability tại VN/SE Asia | – | Owner: TBD. Due: 2026-06-15. Hỏi vendor + check status page. |

## 9. Definition of Done — workstream level

- [ ] `_overview.md` APPROVED.
- [ ] 7 file con APPROVED.
- [ ] Mỗi component có table: **Feature × Status (GA/Preview/Private) × Region × Limitation**.
- [ ] Mỗi claim trọng yếu có ≥1 nguồn T1 + ≥1 nguồn T2/T3 độc lập (hoặc note `[GAP]`).
- [ ] So sánh G1 vs G2 có ít nhất 5 trục với evidence cho mỗi trục.
- [ ] Feature set tổng hợp đã sync với `../04_Competitive_Analysis/07_feature_matrix.md`.
- [ ] `../WORKSPACE_TRACKING.md` cập nhật status + last update.

## 10. Change log

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-05-21 | v0.1 | Khởi tạo overview Databricks Technology | Lead Consultant |
