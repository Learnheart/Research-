# 01 — Authoring Deep-Dive

> **Mục tiêu thảo luận:** Hiểu rõ Databricks "đóng khung" agent ra sao để mọi capability (eval/monitor/govern) gắn được vào. Đây là điểm có **lock-in mạnh nhất** nếu chọn full-stack Databricks; cần thiết kế khôn ngoan để giữ portability.

---

## 1.1 Bức tranh kỹ thuật: 3 layer authoring

```
┌───────────────────────────────────────────────────────────────────────┐
│  LAYER A — Low-code (Agent Bricks)                                    │
│  ┌──────────────────┐   ┌─────────────────────────────────────────┐   │
│  │ Knowledge        │   │ Supervisor Agent                        │   │
│  │ Assistant        │   │  - max 20 sub-agents, EN-only           │   │
│  │ - serverless     │   │  - subagents: Genie, KA, MS endpoint,   │   │
│  │ - UC required    │   │    UC fn, VS index, MCP, custom agent   │   │
│  │ - gte-large-en   │   │  - permission-aware delegation          │   │
│  └──────────────────┘   └─────────────────────────────────────────┘   │
├───────────────────────────────────────────────────────────────────────┤
│  LAYER B — Mid-code (AI Playground / Foundation API)                  │
│  Prompt engineering, model picking, prototype no-code                 │
├───────────────────────────────────────────────────────────────────────┤
│  LAYER C — Full-code (Agent Framework + MLflow)                       │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │ MLflow ResponsesAgent  (wrap any framework: LC, LI, OpenAI…)    │  │
│  │     │                                                           │  │
│  │     ├─► MLflow AgentServer (async FastAPI + tracing + routing)  │  │
│  │     ├─► Built-in Chat UI (streaming + markdown + history)       │  │
│  │     ├─► MCP integration (managed / external / custom)           │  │
│  │     ├─► Auth model: app (SP) or on-behalf-of-user (OBO)         │  │
│  │     └─► Deploy via DABs (databricks.yml)                        │  │
│  └─────────────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────────────┘
```

3 layer này **không loại trừ nhau**. Pattern thường gặp: prototype Layer B → product hóa Layer C, đồng thời dùng Layer A như tool con trong supervisor.

---

## 1.2 MLflow ResponsesAgent — interface trung tâm

### Vai trò
Là contract mà mọi agent custom phải tuân thủ để Databricks "nhận diện" và hook tracing/eval/monitor vào. Bỏ qua interface này → mất quyền lợi MLflow 3 native.

### Cấu trúc khái niệm (suy từ docs)
- **Input**: messages + optional `custom_inputs` (session info, tenant id, user role…).
- **Output**: text response + `custom_outputs` (citations, tool_calls trace, reasoning…) + streaming delta events (`output_text.delta`).
- **Streaming protocol**: delta events → completion event cuối cùng.

### Tại sao nó khác `predict()` thông thường?
- `predict()` chỉ là batch infer.
- ResponsesAgent có **streaming + multi-modal output + custom IO** → đủ cho UI realtime.
- Khi log model qua `mlflow.pyfunc.log_model()` hoặc `mlflow.langchain.log_model()`, signature ResponsesAgent giúp MLflow Trace tự attach.

> 🏦 **Banking insight:**
> `custom_outputs` là chỗ vàng để giữ **citation + reasoning + risk score** trả về client. Ví dụ credit memo agent: `response.text` là draft memo, `custom_outputs.citations` là list source docs (credit policy v3.2, customer financials Q1), `custom_outputs.reasoning_score` là probability đoán đúng. UI sẽ render structured panel cho RM review — đây là cơ chế đạt **explainability cho regulator**.

> ⚠️ **Tradeoff:** Bind agent vào ResponsesAgent = giảm portability sang vendor khác (LangSmith, LangServe…). Cách giảm rủi ro: viết business logic ở 1 lớp tách biệt, ResponsesAgent chỉ là *adapter* mỏng.

> ❓ **Open question:** Hỗ trợ ngôn ngữ ngoài Python? Hiện docs chỉ đề cập Python.

---

## 1.3 Framework interop matrix

Theo docs Agent Framework + Lifecycle:

| Framework | Hỗ trợ wrap ResponsesAgent | Auto-trace | Ghi chú |
|---|---|---|---|
| OpenAI Agents SDK | ✅ (template mặc định) | ✅ | Khuyến nghị cho start mới |
| LangChain | ✅ | ✅ (MLflow autolog 20+ framework) | `mlflow.langchain.log_model()` riêng |
| LlamaIndex | ✅ | ✅ | Đề cập trong author-agent docs |
| Custom Python class | ✅ | ✅ qua manual span | `mlflow.pyfunc.log_model()` |

> 🏦 **Banking insight:**
> Multi-team setup ở ngân hàng thường có team risk dùng LangChain (vì community), team retail dùng OpenAI SDK (vì latency), team core dùng custom Python (vì legacy). ResponsesAgent giúp **chuẩn hoá observability layer** mà không bắt team đổi framework. Trade-off: vẫn cần training nội bộ vì ResponsesAgent contract là cái team phải học chung.

---

## 1.4 MCP Protocol — tool layer chuẩn hóa

### 3 loại server (xác nhận từ docs)

```
┌────────────────────────────────────────────────────────────────┐
│ Managed MCP (Databricks-hosted)                                │
│  - Vector Search, Genie Spaces, SQL, UC functions              │
│  - Permission qua UC                                           │
│  - Pricing theo compute backend (serverless / VS / SQL)        │
├────────────────────────────────────────────────────────────────┤
│ External MCP                                                   │
│  - Third-party servers qua Databricks-managed proxy            │
│  - Managed OAuth → secure auth                                 │
│  - Use case: gọi Stripe, SAP, Bloomberg, Murex…                │
├────────────────────────────────────────────────────────────────┤
│ Custom MCP                                                     │
│  - Host as Databricks App                                      │
│  - Full code control                                           │
│  - Pricing: Databricks Apps                                    │
└────────────────────────────────────────────────────────────────┘
        │
        ▼
  AI Gateway dashboard (MCPs tab) — central catalog + control
```

### Vì sao MCP > UC Function thô?
Docs ghi rõ "Databricks recommends using MCP tools instead for most new use cases". Lý do (suy từ thiết kế MCP):
- MCP có discovery protocol (agent biết tool nào available, schema gì).
- Managed OAuth tránh tự code auth cho mỗi tool.
- Cùng catalog trong AI Gateway → audit + permission tập trung.

> 🏦 **Banking insight:**
> Map MCP vào kiến trúc ngân hàng:
> - **Managed MCP** → tool gọi UC table (customer master, account balance, transaction history) — hợp cho retail customer service agent.
> - **External MCP** → tool gọi external KYC vendor (Refinitiv, World-Check), payment switch (NAPAS), credit bureau (CIC). Managed OAuth = không phải tự code OAuth dance.
> - **Custom MCP** → tool wrap core banking API legacy (T24, Flexcube, Symbols) — Custom MCP host trên Databricks App, public ra cho mọi agent dùng chung.

> ⚠️ **Tradeoff:** Mỗi MCP server thêm 1 hop network → latency tăng. Tool latency-critical (gọi core banking lấy balance để trả khách hàng <500ms) cần benchmark; nếu chậm thì giữ inline UC function thay vì MCP.

---

## 1.5 Authentication model — 2 mode

### App auth (Service Principal)
- Agent chạy dưới identity của SP.
- Mọi user gọi agent đều "thấy" cùng data SP có quyền.
- Đơn giản, predictable cost, nhưng **không multi-tenant về data**.

### User auth (on-behalf-of-user)
- Agent kế thừa quyền end-user.
- UC ACL filter tự động ở mọi query.
- Phức tạp hơn, cần token propagate.

> 🏦 **Banking insight:**
> - **Customer-facing chatbot (consumer side):** App auth. SP có quyền đọc public KB + customer master với row-filter theo `customer_id`. User context truyền qua `custom_inputs.customer_id` rồi filter ở UC function.
> - **Internal RM/teller copilot:** User auth. RM A không được xem khách hàng của RM B → UC row filter theo RM ID. OBO ensure không leak.
> - **Treasury / capital markets agent:** User auth bắt buộc + thêm chinese-wall via UC tag — research analyst không xem trading book và ngược lại.

> ⚠️ **Tradeoff OBO:** Mỗi request token user expire → cần refresh logic. Latency cộng dồn ở mỗi hop tool. Nên cache permission decision trong span của trace.

---

## 1.6 Databricks Asset Bundles (DABs) — IaC cho agent

### Vì sao docs khuyến nghị?
- `databricks.yml` chứa: agent code path, dependencies, endpoint config, permission, resource binding (VS index, UC fn).
- 1 lệnh CLI deploy → reproducible.
- Git-friendly → merge PR review prompt + code + config cùng nhau.

> 🏦 **Banking insight:**
> Ngân hàng có 3 môi trường (dev/uat/prod) bắt buộc; DABs giải bài toán "agent dev xong, ai deploy lên prod thế nào". Pattern đề xuất:
> - 1 repo per agent.
> - 1 `databricks.yml` với targets `dev/uat/prod` parameterize SP, endpoint name, VS index.
> - CI pipeline: `databricks bundle deploy --target uat` → test → `--target prod`.
> - Approval gate trước prod: regression eval PASS + risk officer sign-off.

> ❓ **Open question:** DABs có hỗ trợ rollback 1-command không? Hay phải redeploy version cũ?

---

## 1.7 So sánh nhanh: chọn layer authoring nào?

| Use case banking | Layer khuyến nghị | Lý do |
|---|---|---|
| Internal policy Q&A (HR, Compliance KB) | A — Knowledge Assistant | Low effort, citation built-in, SME có thể tự maintain |
| Multi-domain bank-wide assistant | A — Supervisor + KA sub-agents | Orchestration sẵn, permission ACL native |
| Customer service chat | C — Agent Framework | Cần custom UI, tích hợp core banking, auth phức tạp |
| Credit memo automation | C — Agent Framework | Workflow nhiều bước, maker-checker, structured output |
| AML alert triage | C — Agent Framework + Supervisor | Cần routing alert type → specialist sub-agent |
| Quick exec dashboard Q&A | A — Genie + Supervisor | NL → SQL trên Delta, zero custom code |
| RM advisory copilot | C — full Agent Framework + MCP | Tool richness (Bloomberg, CRM, KB) cần MCP |

---

## 1.8 Câu hỏi thảo luận cho team meeting

1. **Lock-in:** Ta accept lock-in ResponsesAgent đến mức nào? Có cần lớp abstraction trên (`our.Agent`) bọc thêm không?
2. **Auth strategy:** Mặc định OBO hay App auth? Có use case nào BẮT BUỘC OBO cho compliance (Basel, MAS, SBV)?
3. **MCP catalog:** Ai chủ trì danh mục MCP server nội bộ ngân hàng? Trung tâm AI? Mỗi BU tự quản?
4. **DABs CI/CD:** Tích hợp với pipeline Jenkins/GitLab hiện tại thế nào? Có thay được legacy deployment process không?
5. **Supervisor limit 20:** Nếu vượt (50+ tool nội bộ) → hierarchical supervisor hay multi-supervisor + router custom?
