# 01 — Authoring & Agent Development

> **Vai trò trong lifecycle:** Pha **Develop** — chuyển ý tưởng / yêu cầu nghiệp vụ thành agent chạy được, từ no-code đến full-code.

Databricks cung cấp một dải lựa chọn authoring đi từ "drag-and-drop / config" đến "wrap framework bất kỳ bằng custom code", giúp đội ngũ chọn được mức độ kiểm soát phù hợp với độ phức tạp use case.

---

## 1.1 AI Playground (no-code prototyping)

- UI để "query available GenAI models and agents", thử prompt, chỉnh temperature và so sánh model.
- Là điểm khởi đầu được khuyến nghị trong [Agent Development Lifecycle](https://docs.databricks.com/aws/en/agents/agents-dev-lifecycle) trước khi viết bất kỳ dòng code nào.
- Hỗ trợ prompt engineering nhanh, kiểm tra Foundation Model trước khi quyết định framework.

🔗 [AI Playground docs](https://docs.databricks.com/aws/en/large-language-models/ai-playground)

**Adapt vào sản phẩm:** Dùng cho giai đoạn POC khi PM/BA cần "cảm" xem một use case có khả thi với LLM hay không, trước khi giao cho engineer build. Phù hợp cho discovery workshop với business stakeholders.

---

## 1.2 Agent Bricks — Knowledge Assistant (low-code, document Q&A)

Agent Bricks là dòng sản phẩm cấu hình thay vì viết code. **Knowledge Assistant** là agent built-in chuyên trả lời câu hỏi dựa trên tài liệu, sử dụng Instructed Retriever (theo docs không phải RAG truyền thống).

- Quy trình 3 bước: **Configure** (đặt tên, mô tả, gắn knowledge sources từ Volumes, tables hoặc Vector Search indexes) → **Test** (chat trực tiếp, xem reasoning + citations) → **Improve** (thêm labeled questions và guidelines từ SME).
- Response có citation pointing về document gốc.
- Có thể query qua AI Playground, REST/Python API hoặc Databricks SDK.
- Yêu cầu: serverless compute, Unity Catalog, embedding model `databricks-gte-large-en`.

🔗 [Knowledge Assistant docs](https://docs.databricks.com/aws/en/generative-ai/agent-bricks/knowledge-assistant)

**Adapt vào sản phẩm:**
- Chatbot trả lời từ policy / HR handbook nội bộ.
- Customer support agent từ knowledge base / FAQ.
- Q&A trên product documentation cho khách hàng.

---

## 1.3 Agent Bricks — Supervisor Agent (multi-agent orchestration)

Agent điều phối nhiều sub-agent / tool chuyên biệt, quản lý phân quyền và "improving coordination quality based on natural language feedback".

- **Subagents hỗ trợ:** Genie Spaces, published dashboards, Knowledge Assistant endpoints, model serving endpoints, Unity Catalog functions, Vector Search indexes, external/custom MCP servers, custom agents.
- **Access control:** end user chỉ tương tác với subagent họ có quyền; supervisor tự redirect khi thiếu permission.
- **Giới hạn (theo docs):** tối đa 20 agent / supervisor, hiện chỉ English.

🔗 [Supervisor Agent docs](https://docs.databricks.com/aws/en/generative-ai/agent-bricks/multi-agent-supervisor)

**Adapt vào sản phẩm:**
- Market analysis assistant kết hợp research reports + usage data + dashboards.
- Internal IT/HR assistant: routing giữa policy KB, ticket creation tool và account lookup function.
- Omni-channel customer service: 1 supervisor + nhiều sub-agent (billing, returns, product info).

---

## 1.4 Agent Framework + MLflow ResponsesAgent (full-code)

Khi cần control hoàn toàn về code, server, deployment workflow. Cách tiếp cận "build an AI agent and deploy it using Databricks Apps".

- **MLflow ResponsesAgent interface:** Databricks recommend wrap agent bằng interface này để hưởng "robust logging, tracing, evaluation, and monitoring" — tương thích với mọi framework third-party.
- **Framework hỗ trợ:** template mặc định là OpenAI Agents SDK; cũng hỗ trợ LangChain, LlamaIndex.
- **MLflow AgentServer:** async FastAPI server với built-in tracing/observability, request routing và error handling.
- **Built-in Chat UI:** template tự bundle REST API + chat UI có streaming, markdown rendering, persistent chat history.
- **Advanced:** streaming delta events, `custom_inputs`/`custom_outputs`, MCP integration, choice giữa app authorization (service principal) và user authorization (on-behalf-of-user).
- **Deployment:** dùng Databricks Asset Bundles (DABs) với `databricks.yml` cho declarative config.

🔗 [Agent Framework — Author Agent docs](https://docs.databricks.com/aws/en/generative-ai/agent-framework/author-agent)

**Adapt vào sản phẩm:**
- Agent core của ngân hàng / bảo hiểm yêu cầu custom business logic, audit trail riêng.
- Agent tích hợp với hệ thống cũ qua adapter Python mà low-code không bọc được.
- Sản phẩm SaaS multi-tenant cần control deployment topology, scaling, IDE-based git workflow.

---

## 1.5 Agent System Design Patterns (chọn mức độ phức tạp)

Docs chia thành 4 mức, gợi ý "Start simple, increase complexity when needed":

| Pattern | Đặc điểm docs | Use case |
|---|---|---|
| **LLM + Prompt** | "Very simple, easy to create", customization tối thiểu | Prototype, Q&A chung |
| **Deterministic Chain** | "Highest predictability and auditability", latency thấp | Quy trình tĩnh, RAG cơ bản |
| **Single-Agent System** | "Often the sweet spot for enterprise use cases" | Truy vấn đa dạng cùng domain, quyết định động |
| **Multi-Agent System** | "Highly modular, scales to big domains" | Nhiều domain chuyên biệt, supervisor điều phối |

🔗 [Agent System Design Patterns docs](https://docs.databricks.com/aws/en/agents/agent-system-design-patterns)

**Adapt vào sản phẩm:** Dùng bảng này như "rubric kiến trúc" trong solution review — đẩy ngược team tránh over-engineer multi-agent khi single-agent đủ.

---

## Kết nối các bước tiếp theo

- Sau khi authoring → log + register vào Unity Catalog (xem [04-serving-deployment](04-serving-deployment.md)).
- Trước khi prompt finalize → đánh giá bằng MLflow Agent Evaluation ([03-evaluation](03-evaluation.md)).
- Khi cần kết nối data / hành động → thêm tools (xem [02-tools-retrieval](02-tools-retrieval.md)).
