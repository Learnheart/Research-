# PROGRESS — Databricks AI Agent Capabilities Summary

> Workspace: `databricks/capabilities/`
> Mục tiêu: Bản tóm tắt high-level về capabilities của Databricks cho việc build AI agents, dựa hoàn toàn vào docs chính thức `docs.databricks.com/aws/en/agents/`.

## Phase 0 — Setup
- [x] Tạo `PROGRESS.md` và `PLAYBOOK.md`
- [x] Crawl trang index `docs.databricks.com/aws/en/agents/`
- [x] Lập danh sách sub-pages cần đọc

## Phase 1 — Crawl docs
- [x] Concepts: `agent-system-design-patterns`, `gen-ai-capabilities`, `gen-ai-challenges`
- [x] Lifecycle: `agents-dev-lifecycle`
- [x] Authoring: `agent-framework/author-agent`, `agent-bricks/*`
- [x] Tools & retrieval: RAG guide, Vector Search, Unity Catalog tools
- [x] Evaluation: MLflow Agent Evaluation, scorers, judges
- [x] Serving & deployment: Model Serving cho agents, deploy-agent
- [x] Monitoring & governance: Production monitoring, Unity Catalog, AI Gateway

## Phase 2 — Viết capability files
- [x] `01-authoring.md` — Agent Framework, Agent Bricks, Playground
- [x] `02-tools-retrieval.md` — Vector Search, Unity Catalog Functions, MCP
- [x] `03-evaluation.md` — Agent Evaluation, judges, scorers, review app
- [x] `04-serving-deployment.md` — Model Serving, deploy_agent, AI Gateway
- [x] `05-monitoring-observability.md` — Lakehouse Monitoring for GenAI, traces, feedback
- [x] `06-governance-security.md` — Unity Catalog, ACL, PII, lineage
- [x] `07-when-to-choose.md` — Tiêu chí chọn Databricks vs nền tảng khác

## Phase 3 — Tổng hợp
- [x] `README.md` — summary toàn bộ, 3 câu hỏi mục tiêu
- [x] Soát lại link docs trong từng file
- [x] Verify mọi claim có nguồn trong docs (không suy đoán)

## Phase 4 — Deep-dive technical + banking insights
- [x] Crawl trang technical-depth: ResponsesAgent / ChatAgent, Vector Search internals, AI Gateway (inference tables, rate limits, traffic split, usage), MLflow 3 scorers/tracing/monitoring, MCP servers, judge reference, synthesize eval sets, human feedback
- [x] `deep-dive/00-research-notes.md` — methodology, scope, glossary
- [x] `deep-dive/01-authoring-deep.md` — ResponsesAgent vs ChatAgent, MCP protocol, DABs
- [x] `deep-dive/02-tools-retrieval-deep.md` — HNSW, RRF, chunking, embedding internals
- [x] `deep-dive/03-evaluation-deep.md` — judge math, scorer architecture, synthetic eval
- [x] `deep-dive/04-serving-deployment-deep.md` — endpoint topology, auth, Inference Tables schema
- [x] `deep-dive/05-monitoring-deep.md` — sampling, multi-turn judges, OTel
- [x] `deep-dive/06-governance-deep.md` — UC AI assets, AI Gateway control plane internals
- [x] `deep-dive/07-banking-blueprints.md` — reference architecture cho 8 use case ngân hàng
- [x] `deep-dive/README.md` — index riêng cho deep-dive

## Out-of-scope (theo yêu cầu)
- Không viết code samples, API signatures, step-by-step tutorials (deep-dive cho phép pseudo-schema và sơ đồ kiến trúc, KHÔNG snippet)
- Không upsert thông tin ngoài folder `databricks/capabilities/`
- Không dùng blog hay third-party sources
