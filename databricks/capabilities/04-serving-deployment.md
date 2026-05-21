# 04 — Serving & Deployment

> **Vai trò trong lifecycle:** Pha **Deploy** — đưa agent từ notebook lên hạ tầng production có scale, auth, version control.

Databricks tách 2 con đường deploy: **Model Serving endpoint** (truyền thống) và **Databricks Apps** (mới hơn, được docs khuyến nghị cho hầu hết use case mới).

---

## 4.1 Deploy qua Model Serving — `agents.deploy()`

Hàm `deploy()` trong Agent Framework Python API tự build infra production.

- "Deployment creates a serving endpoint with built-in scalability, monitoring, and collaboration tools."
- **Tự động provision:**
  - Scalable REST API endpoint
  - Secure authentication credentials (short-lived)
  - Monitoring + inference tables cho audit log
  - **Review App** cho stakeholder feedback
  - **Automated quality evaluation** trên production traffic
- **Auth model:** auto-provision short-lived credentials cho Databricks-managed resources (Vector Search, UC functions); kiểm tra permission của caller.
- **Yêu cầu (theo docs):** MLflow 3.1.3+ (khuyến nghị) hoặc 2.13.1+; `databricks-agents` SDK 1.1.0+; agent đã register vào Unity Catalog.
- **Time:** có thể mất tới 15 phút.
- **Zero-downtime updates:** gọi `deploy()` lại với cùng model name → tạo version mới, version cũ vẫn serve trong lúc rollout.

🔗 [Deploy Agent docs](https://docs.databricks.com/aws/en/generative-ai/agent-framework/deploy-agent)

**Adapt vào sản phẩm:**
- Internal copilot cho nhân sự nội bộ — Model Serving endpoint đủ, không cần custom UI.
- B2B API: expose agent endpoint cho partner qua OpenAI-compatible REST.

---

## 4.2 Deploy qua Databricks Apps (full code control)

Docs khuyến nghị: "for new use cases, Databricks recommends deploying agents on Databricks Apps for full control over agent code, server configuration, and deployment workflow."

- Bundle Agent Framework template gồm REST API + chat UI + streaming.
- **Deploy declarative** qua Databricks Asset Bundles (DABs) — file `databricks.yml` + CLI.
- IDE-based, git-friendly workflow.

🔗 [Author Agent docs](https://docs.databricks.com/aws/en/generative-ai/agent-framework/author-agent)

**Adapt vào sản phẩm:**
- Customer-facing agent cần UI brand-custom, history, login flow → Databricks Apps.
- Sản phẩm SaaS cần CI/CD git-ops chuẩn cho agent code.

---

## 4.3 Logging & Registering qua Unity Catalog

Tiền đề bắt buộc trước khi deploy.

- **Log:** dùng MLflow "Models from Code" — agent code lưu dạng Python file, environment lưu thành package deps.
  - LangChain: `mlflow.langchain.log_model()`
  - PyFunc (custom class): `mlflow.pyfunc.log_model()`
- **Register:** `mlflow.register_model(model_uri=…, name=…)` đẩy agent thành asset trong Unity Catalog.
- **Resource binding:** khi log, khai báo `resources` param để hệ thống biết auth nào cần (Vector Search index, serving endpoint…).
- **Validate trước deploy:** `mlflow.models.predict()`.

🔗 [Log Agent docs](https://docs.databricks.com/aws/en/generative-ai/agent-framework/log-agent)

**Adapt vào sản phẩm:**
- Quy chuẩn release: tất cả agent phải pass `mlflow.models.predict()` smoke test trong CI trước khi deploy.

---

## 4.4 Foundation Models & Serving Options

Mosaic AI Model Serving host model theo 3 cấu hình:

| Mode | Khi dùng |
|---|---|
| **Pay-per-token** | Experimentation, traffic biến động |
| **Provisioned throughput** | Production workloads, cần SLA latency |
| **External model routing** | Gọi qua OpenAI / Anthropic / vendor khác qua 1 layer |

🔗 [Query LLMs docs](https://docs.databricks.com/aws/en/agents/query-llms)
🔗 [Foundation Models docs](https://docs.databricks.com/aws/en/machine-learning/model-serving/foundation-model-overview)

**Adapt vào sản phẩm:**
- POC dùng pay-per-token; khi go-live và biết QPS → switch provisioned throughput để fix cost.
- Vendor risk: external model routing cho phép "đổi nhà cung cấp model" mà không sửa code client.

---

## 4.5 Query Interfaces

- **Databricks OpenAI Client** — khuyến nghị cho project mới, native streaming.
- **OpenAI-compatible REST API** — language-agnostic.
- **`ai_query` SQL function** — batch inference qua SQL, gồm cả task-specific functions (`ai_classify`, `ai_extract`).

🔗 [Query LLMs docs](https://docs.databricks.com/aws/en/agents/query-llms)

**Adapt vào sản phẩm:**
- Backend Node.js / Go → REST API.
- Data team xử lý batch 10 triệu document → `ai_query` trong SQL warehouse, không phải viết app.

---

## Kết nối các bước tiếp theo

- Endpoint deployed sẽ phát trace → Production Monitoring chấm scorer realtime ([05-monitoring-observability](05-monitoring-observability.md)).
- Mọi endpoint chạy qua **AI Gateway** để hưởng rate-limit / guardrail / audit ([06-governance-security](06-governance-security.md)).
