# Databricks AI Agent Capabilities — Tóm tắt High-Level

> **Phạm vi:** Tóm tắt các capabilities Databricks cung cấp để build, evaluate, deploy, monitor AI agents.
> **Nguồn:** chỉ từ [`docs.databricks.com/aws/en/agents/`](https://docs.databricks.com/aws/en/agents/) và các sub-pages liên kết trực tiếp.
> **Đối tượng:** người ra quyết định kỹ thuật (CIO/CTO/Tech Lead/Architect) muốn đánh giá Databricks cho use case agent mà không cần mở docs gốc.
> **Ngày crawl:** 2026-05-21.

---

## TL;DR — 3 câu hỏi mục tiêu

### Q1. Databricks cung cấp những thành phần cốt lõi nào?

Docs xác nhận 6 nhóm capability lồng ghép thành một **lifecycle khép kín**:

| Nhóm | Thành phần chính |
|---|---|
| **Authoring** | AI Playground, Agent Bricks (Knowledge Assistant, Supervisor), Agent Framework + MLflow ResponsesAgent, Databricks Apps |
| **Tools & Retrieval** | Unity Catalog Functions, MCP servers (managed/external/custom), Vector Search (Delta Sync, hybrid, full-text), Genie, structured retrieval, RAG pattern |
| **Evaluation** | Mosaic AI Agent Evaluation, 10 built-in LLM judges, Review App, labeling sessions, MLflow Tracing, AI Insights |
| **Serving & Deployment** | `agents.deploy()` Model Serving endpoint, Databricks Apps, Unity Catalog registration, Foundation Model APIs (pay-per-token / provisioned throughput / external) |
| **Monitoring** | Production Monitoring trên live traces, multi-turn judges, MLflow Tracing, OpenTelemetry export, SQL alerts, Inference Tables |
| **Governance / LLMOps** | Unity Catalog (ACL + lineage + audit), AI Gateway (rate limit, traffic split, guardrails), Prompt Registry, MLOps Stacks |

### Q2. Mỗi nhóm giải quyết vấn đề gì trong lifecycle?

```
DEVELOP  ──► EVALUATE ──► DEPLOY  ──► MONITOR  ──► (vòng lặp)
   │              │           │           │
   │  Authoring   │ Mosaic AI │ Model     │ Production
   │  Agent       │ Agent     │ Serving / │ Monitoring
   │  Framework   │ Eval +    │ Databricks│ + AI
   │  Knowledge   │ Judges +  │ Apps +    │ Insights +
   │  Asst,       │ Review    │ Unity     │ Inference
   │  Supervisor  │ App       │ Catalog   │ Tables
   │              │           │ register  │
   │
   └─► Tools & Retrieval (Vector Search, MCP, UC functions) — input layer cho Develop
   └─► Unity Catalog + AI Gateway — cross-cutting governance qua mọi pha
```

Docs ghi rõ vòng feedback "production failure → labeling session → evaluation dataset → regression → re-deploy zero-downtime" là **first-class workflow**, không phải tự lắp.

### Q3. Khi nào nên chọn Databricks?

**Fit rõ khi:** data đã ở Databricks/Unity Catalog · cần audit + governance chặt · cần đa dạng pattern agent (simple → multi-agent) · coi quality monitoring là release gate · cần multi-provider model strategy · có SME-in-the-loop workflow.

**Cân nhắc khi:** data chủ yếu ở ngoài hệ sinh thái · cần >20 sub-agent / multi-lingual Supervisor · cần real-time monitoring (<1 phút) · chưa bật serverless/Unity Catalog.

→ Checklist chi tiết tại **[07-when-to-choose.md](07-when-to-choose.md)**.

---

## Cấu trúc báo cáo

| File | Nội dung | Phase lifecycle |
|---|---|---|
| **[01-authoring.md](01-authoring.md)** | AI Playground, Knowledge Assistant, Supervisor Agent, Agent Framework + ResponsesAgent, 4 system design patterns | Develop |
| **[02-tools-retrieval.md](02-tools-retrieval.md)** | UC Functions, MCP servers, Vector Search (index types, hybrid, scaling), Genie, structured retrieval, RAG | Develop |
| **[03-evaluation.md](03-evaluation.md)** | Agent Evaluation, 10 built-in judges, Review App, MLflow Tracing, AI Insights, prompt optimization | Evaluate |
| **[04-serving-deployment.md](04-serving-deployment.md)** | `agents.deploy()`, Databricks Apps, log/register qua UC, Foundation Models, query interfaces | Deploy |
| **[05-monitoring-observability.md](05-monitoring-observability.md)** | Production Monitoring scorers, multi-turn judges, MLflow Tracing, OTel export, dashboards & alerts | Monitor |
| **[06-governance-security.md](06-governance-security.md)** | Unity Catalog, AI Gateway, LLMOps Stack, security baseline, 3 GenAI challenges → solutions | Cross-cutting |
| **[07-when-to-choose.md](07-when-to-choose.md)** | Tiêu chí fit / không fit, checklist quyết định 1 trang | Decision |

**Deep-dive (technical + banking focus):**

| File | Tập trung |
|---|---|
| **[deep-dive/README.md](deep-dive/README.md)** | Index riêng cho lớp deep-dive, hướng dẫn dùng cho buổi tech share |
| **[deep-dive/00-research-notes.md](deep-dive/00-research-notes.md)** | Methodology, glossary, open questions |
| **[deep-dive/01-authoring-deep.md](deep-dive/01-authoring-deep.md)** | ResponsesAgent contract, MCP protocol, DABs, framework interop |
| **[deep-dive/02-tools-retrieval-deep.md](deep-dive/02-tools-retrieval-deep.md)** | HNSW + RRF math, chunking, embedding pipelines, index governance |
| **[deep-dive/03-evaluation-deep.md](deep-dive/03-evaluation-deep.md)** | Judge mechanics, scorer architecture, synthetic eval, validity |
| **[deep-dive/04-serving-deployment-deep.md](deep-dive/04-serving-deployment-deep.md)** | Endpoint topology, auth flows, Inference Tables, throughput economics |
| **[deep-dive/05-monitoring-deep.md](deep-dive/05-monitoring-deep.md)** | Sampling, multi-turn judges, OTel, alerting |
| **[deep-dive/06-governance-deep.md](deep-dive/06-governance-deep.md)** | UC AI assets, AI Gateway control plane, audit, multi-tenant |
| **[deep-dive/07-banking-blueprints.md](deep-dive/07-banking-blueprints.md)** | 8 reference architecture cho banking products |

Tracking:
- **[PROGRESS.md](PROGRESS.md)** — checklist các phase của báo cáo.
- **[PLAYBOOK.md](PLAYBOOK.md)** — quy trình update workspace.

---

## Bản đồ capability theo lifecycle

```
┌─────────────────────────── GOVERNANCE LAYER (cross-cutting) ───────────────────────────┐
│  Unity Catalog (ACL, lineage, audit)   ·   AI Gateway (rate limit, guardrails, split)  │
│  Prompt Registry   ·   MLOps Stacks    ·   OpenTelemetry export                        │
└─────────────────────────────────────────────────────────────────────────────────────────┘
           ▲                       ▲                       ▲                       ▲
  ┌────────┴────────┐    ┌─────────┴────────┐    ┌─────────┴─────────┐    ┌─────────┴──────────┐
  │   DEVELOP        │    │   EVALUATE        │    │   DEPLOY           │    │   MONITOR           │
  │                  │    │                   │    │                    │    │                     │
  │ AI Playground    │    │ Agent Evaluation  │    │ agents.deploy()    │    │ Production Monitor  │
  │ Agent Bricks:    │    │ 10 LLM Judges     │    │ Databricks Apps    │    │ Multi-turn Judges   │
  │   - Knowledge    │    │ Review App        │    │ Model Serving      │    │ MLflow Tracing      │
  │   - Supervisor   │    │ Labeling Sessions │    │ Foundation Models  │    │ AI Insights         │
  │ Agent Framework  │    │ MLflow Tracing    │    │ Unity Catalog      │    │ Inference Tables    │
  │ ResponsesAgent   │    │ AI Insights       │    │   registration     │    │ SQL Alerts          │
  └──────────────────┘    └───────────────────┘    └────────────────────┘    └─────────────────────┘
           ▲
  ┌────────┴───────────────────────────┐
  │  TOOLS & RETRIEVAL (input layer)    │
  │  UC Functions · MCP · Vector Search │
  │  Genie · Structured Retrieval · RAG │
  └─────────────────────────────────────┘
```

---

## Nguyên tắc biên soạn

- ✅ Chỉ trích từ docs Databricks chính thức.
- ✅ Mọi capability có link docs tương ứng.
- ✅ Mỗi nhóm có mục **"Adapt vào sản phẩm"** với hướng triển khai thực tế (gợi ý, không phải tutorial).
- ✅ Bullet ngắn, không code, không API signature, không step-by-step.
- ⚠️ Bất kỳ chỗ nào docs không nói rõ → đánh dấu **"không tìm thấy trong docs"**.
- ❌ Không so sánh trực tiếp với platform khác (out-of-scope theo brief).

---

## 2 lớp tài liệu — đọc theo persona

| Persona | Đọc trước | Mục tiêu |
|---|---|---|
| **CIO / CTO / Head of Tech** | README + 07-when-to-choose | Quyết định platform fit |
| **Solution Architect / Tech Lead** | README → 01-06 → deep-dive/00 → deep-dive/07 blueprints | Thiết kế hệ thống cho product cụ thể |
| **ML Engineer / Data Engineer** | deep-dive/01, 02, 03 | Build agent + retrieval + eval |
| **SRE / Platform / Observability** | deep-dive/04, 05 | Vận hành endpoint + monitor |
| **Security / Compliance / Risk** | deep-dive/06 + Inference Tables sections | Govern + audit + multi-tenant |
| **Product Manager / Solution Designer** | README + deep-dive/07 blueprints | Map use case ngân hàng → capability |

## Đề xuất bước tiếp theo (cho người đọc)

1. Đọc **README → 07-when-to-choose** để kết luận fit hay không.
2. Nếu fit → đọc theo lifecycle: **01 → 02 → 03 → 04 → 05 → 06**.
3. Đào sâu kỹ thuật + ý tưởng sản phẩm: vào **[deep-dive/README.md](deep-dive/README.md)**.
4. Khi cần verify chi tiết technical, click link docs trong từng file (không có file nào tóm tắt thay cho docs gốc về implementation details).
5. Khi Databricks ra capability mới → theo workflow trong [PLAYBOOK.md](PLAYBOOK.md) để cập nhật báo cáo.
