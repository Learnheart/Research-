# Phân tích kiến trúc Agentic AI — Tổng hợp từ TCB Platform & TÀI Studio

> **Nguồn dữ liệu:**
> - `architecture.md` — Bản phê duyệt nền tảng *TCB Agentic AI Platform* (DAB0, cấp chiến lược)
> - `tai_architecture.md` — Bản thiết kế tổng thể *TÀI Studio* (cấp triển khai)
> - `tai_list_agent.md` — Đặc tả 9 agent cụ thể trong TÀI Studio
>
> **Phạm vi:** So sánh kiến trúc 2 sản phẩm, rút ra (i) các cấu phần bắt buộc khi xây dựng một sản phẩm agentic, (ii) các design pattern đang được áp dụng.

---

## 1. Định vị 2 sản phẩm

Hai sản phẩm nằm ở 2 lớp trừu tượng khác nhau nhưng cùng giải quyết bài toán *Agentic AI cho doanh nghiệp tài chính (TCB)*. So sánh không phải là *A vs B* mà là *vision vs implementation*.

| Tiêu chí | TCB Agentic AI Platform | TÀI Studio |
| --- | --- | --- |
| **Bản chất** | Reference architecture / Platform blueprint cho toàn bank | Super-app sản phẩm cụ thể (1 hub + 9 agent chuyên biệt) |
| **Cấp độ tài liệu** | DAB0 — xin chủ trương đầu tư | Architecture-of-record + per-agent design docs |
| **Đối tượng** | CIO, kiến trúc sư, security, compliance | Engineering team, end-user trong bank |
| **Use case** | 3 use case khung: CX10 Post-Call, Customer 360, BA GenAI | 9 use case thực tế: dịch, tóm tắt, PPTX, vault, brainstorm, vision, canvas, resume + hub TÀI |
| **Tech stack** | LangChain / LangGraph / LangFlow / LangFuse (đề xuất) + AWS Bedrock | LangGraph + LangChain + Databricks Apps + Lakebase + Unity Catalog Volumes + Claude (Opus/Sonnet/Haiku) |
| **Mức độ trưởng thành** | Maturity Level 1-2 (target) | Đang vận hành, đa số agent ở Level 1 (recommend + human approve) |
| **Phạm vi bao phủ** | Đủ 7 layer (Input → Foundation) + security stack đầy đủ | Đủ phần Agent + Orchestration + Storage + một phần Security; thiếu A2A protocol, vector store, DLP |

> **Quan sát mấu chốt:** TÀI Studio là một **instance hợp lệ** của TCB Platform — nó hiện thực hóa 5/6 capability (C01–C05) mà platform đề xuất, nhưng *chưa* triển khai C06 (Security Controls) ở mức enterprise (chưa có Trellix DLP, Bedrock Guardrails, prompt validation tập trung — chỉ dựa vào Databricks Apps proxy auth + guardrail-as-classifier ở Powerpoint-er).

---

## 2. So sánh kiến trúc theo 7 layer của TCB Platform

TCB Platform đề xuất 7 layer (mục 4.2.2 — `architecture.md`). Bảng dưới đối chiếu từng layer với hiện trạng TÀI Studio để chỉ ra *điều gì đã có, điều gì còn thiếu*.

| # | Layer (TCB) | TCB Platform — Capabilities đề xuất | TÀI Studio — Hiện trạng triển khai | Gap |
| --- | --- | --- | --- | --- |
| 1 | **Input Layer** | Multi-source data lake; CRM/ERP qua MCP; prompt từ Agent Builder | File upload (PDF/DOCX/PPTX/TXT) → base64 → Unity Catalog Volume; user prompt qua React UI | Không có MCP enterprise; không kết nối CRM/ERP trực tiếp |
| 2 | **Agent Orchestration** | Orchestrator agent phân rã goal thành subtask | TÀI Super Agent (LangGraph ReAct + 8 tool calls); centralised orchestration, không có A2A | Đúng pattern nhưng *không có A2A protocol*; agent-to-agent chỉ thông qua TÀI |
| 3 | **AI Agent Layer** | Hỗ trợ 9 pattern: Planning, Reflective, Task-Oriented, Hierarchical, Coordinating, Context-Aware, Self-Learning, HITL, RAG | Đã có: Task-Oriented (8 specialist), Coordinating (TÀI), Context-Aware (Brainstormer + user_preferences), HITL (interrupt/resume), một phần RAG (Vaulter, Visionary qua KA endpoint) | Thiếu: Reflective, Self-Learning thực sự (Vaulter chỉ là KB không có vòng feedback) |
| 4 | **Data Storage / Retrieval** | Unified storage; vector DB + knowledge graph cho RAG | Lakebase (PostgreSQL) cho checkpoint + preferences + brainstorm index + vault graph; UC Volumes cho file; **không có vector store** | Vaulter dùng "load toàn bộ document" thay vì vector search → giới hạn ~50K chars (`tai_architecture.md` § Bottlenecks) |
| 5 | **Output Layer** | Custom output format; knowledge update; enriched data | Output đa dạng: text (Summarizer), file (Translator, PPTX-er, Canvas, Resume), JSON cấu trúc (Brainstormer canvas), SSE stream (Translator); UI actions embedded qua `---UI---` delimiter | Đầy đủ |
| 6 | **Service Layer** | Multi-channel delivery; recommendation; adaptive response | FastAPI REST + SSE stream + iframe embed + TÀI download proxy (SP token); single channel (web) | Chưa multi-channel (chưa có chat tích hợp Salesforce/Email) |
| 7 | **Foundation** | Security controls, No-code/Low-code builder, A2A + MCP protocol | LangChain/LangGraph (code-first, không có no-code); Service Principal auth; CORS; guardrail-as-classifier (chỉ Powerpoint-er); không A2A/MCP | Lớn nhất: **không có No-Code Builder + không có Security Stack tập trung + không có A2A/MCP** |

---

## 3. Các cấu phần chính cần có khi triển khai sản phẩm agentic

Tổng hợp từ 2 nguồn, một sản phẩm agentic tối thiểu cần **9 cấu phần** sau. Mỗi cấu phần được phân loại theo *mức độ bắt buộc* (Must / Should / Could) dựa trên bằng chứng triển khai trong TÀI Studio.

### 3.1. Cấu phần cốt lõi (Must-have)

| # | Cấu phần | Vai trò | Minh chứng (TÀI Studio) | Lý do bắt buộc |
| --- | --- | --- | --- | --- |
| 1 | **Orchestration Engine** | Quản lý vòng reasoning (plan → act → observe), điều phối tool, giữ state | LangGraph `StateGraph` (Brainstormer) + `ReAct + ToolNode` (TÀI) (`tai_list_agent.md` § TÀI Core Logic) | Không có engine này, agent chỉ là LLM call đơn lẻ — không thể đa bước, không state |
| 2 | **LLM Gateway / Model Router** | Truy cập đa model + chọn model theo cost/task | `llm.py` cache `ChatDatabricks` theo endpoint; routing Opus (creative) / Sonnet (default) / Haiku (short, guardrail) (`tai_architecture.md` § LLM Model Decision Tree) | Single model = single point of failure + cost không tối ưu |
| 3 | **Tool/Function layer** | Cho phép agent gọi action ra ngoài LLM (API, file, DB) | TÀI tools: `translate_document`, `summarize_documents`, `generate_presentation`... (`tai_list_agent.md` § TÀI Tools) | Bản chất "agentic" = LLM + tool calling; không có tool thì chỉ là chatbot |
| 4 | **State / Memory Management** | Giữ context giữa turn và giữa session | LangGraph `AsyncCheckpointSaver` (Lakebase) cho short-term; `UserPreferencesStore` cho long-term; `compact_messages.py` (summarize-then-trim ở mốc 10 message) (`tai_architecture.md` § Shared Abstractions) | Multi-turn workflow yêu cầu memory; context window có giới hạn → cần compaction |
| 5 | **File / Artifact Storage** | Lưu input/output của agent (file, knowledge, report) | Unity Catalog Volumes với hierarchy `/{agent}/{user}/{date}/{uuid}.{ext}` (`tai_architecture.md` § Data Flow) | Mọi use case enterprise đều liên quan tới file (BRD, CV, slide, transcript) |

### 3.2. Cấu phần nâng cao (Should-have)

| # | Cấu phần | Vai trò | Minh chứng | Khi cần |
| --- | --- | --- | --- | --- |
| 6 | **Knowledge Retrieval (RAG)** | Lấy thông tin từ knowledge base để giảm hallucination | Vaulter dùng knowledge graph trên Lakebase; Visionary proxy tới Knowledge Agent endpoint | Khi agent cần "biết" thông tin nội bộ doanh nghiệp |
| 7 | **Human-in-the-Loop interface** | Tạm dừng agent để chờ input/approval từ người dùng | LangGraph `interrupt()` trong TÀI khi cần file (`tai_list_agent.md` § Human-in-the-Loop) | Khi agent ở maturity Level 1-2 (human approve trước khi action) — đúng nguyên tắc TCB |
| 8 | **Observability (Tracing + Logging)** | Debug, audit, đánh giá hiệu năng + chất lượng output | `mlflow.langchain.autolog()` mỗi agent có 1 experiment riêng; logger per-agent (`tai_architecture.md` § Tracing) | Bắt buộc khi production; cũng là yêu cầu của TCB NFR-3 (Prompt Auditing) |

### 3.3. Cấu phần governance (Could-have ngay, Must-have khi scale)

| # | Cấu phần | Vai trò | Hiện trạng TÀI Studio | Khi cần |
| --- | --- | --- | --- | --- |
| 9 | **Security & Guardrails** | Input/output safety, DLP, PII masking, prompt validation | Chỉ Powerpoint-er có `check_input_guardrails()` + `check_output_guardrails()` (Haiku classifier); auth dựa vào Databricks Apps proxy + SP token; *chưa* có DLP/Trellix/Bedrock Guardrails (`tai_architecture.md` § Security Concerns) | Khi mở rộng cho end-user thực, dữ liệu nhạy cảm, hoặc compliance (đúng yêu cầu DAB0 § 4.7) |

> **Quan trọng:** TÀI Studio đang ở giai đoạn nội bộ → có thể accept gap về security. Nhưng để scale ra production theo target của TCB Platform, cần bổ sung *cả 3 lớp checkpoint* mà DAB0 mô tả (Trellix DLP endpoint → AI Backend → Bedrock Guardrail API → LLM).

### 3.4. Cấu phần hỗ trợ vận hành

| # | Cấu phần | Vai trò | Minh chứng |
| --- | --- | --- | --- |
| 10 | **No-Code/Low-Code Builder** | Cho user non-technical xây agent (capability C05 của TCB) | **Không có** trong TÀI Studio — agent build bằng code Python | LangFlow là đề xuất của TCB cho gap này |
| 11 | **Deployment automation** | Đóng gói + deploy agent | `deploy.sh`: temp dir → merge shared modules → Vite build → `databricks apps deploy` (`tai_architecture.md` § Deploy Process) | Bắt buộc cho multi-agent platform |
| 12 | **Shared abstractions / SDK** | DRY giữa các agent | 11 shared module: `llm.py`, `file_store.py`, `file_parser.py`, `user_context.py`, `compact_messages.py`... (`tai_architecture.md` § Shared Abstractions) | Khi có >2 agent — nếu không, copy-paste code → bug nhân lên |

---

## 4. Design Pattern sử dụng

### 4.1. Pattern kiến trúc (Architectural)

| Pattern | Mô tả | Áp dụng trong TÀI Studio | Tham chiếu TCB |
| --- | --- | --- | --- |
| **Centralized Orchestrator** | Một super-agent điều phối nhiều specialist; specialist không gọi lẫn nhau | TÀI là điểm vào duy nhất; 8 tool wrap HTTP call tới specialist API | "Orchestrator agent breaks down request" (TCB § 4.2.2) |
| **ReAct Loop** | `Reason → Act (tool) → Observe → Reason...` | TÀI: `load_preferences → agent ⇄ tools → update_preferences → END` | Implicit trong "Planning Agent" pattern (TCB § 5.3) |
| **StateGraph (Stateful Workflow)** | Workflow dạng đồ thị có state, hỗ trợ cycle + checkpoint | Brainstormer: 4-step Understand → Explore → Converge → Wrap-up | "Stateful multi-agent workflow orchestration with cyclical operations" (LangGraph review, TCB § 4.4) |
| **Standalone + Hub (Hybrid Access)** | Mỗi agent vừa độc lập (URL riêng + iframe) vừa được orchestrator gọi | Hub embed iframe; TÀI gọi qua REST | Chưa mô tả tường minh trong TCB |
| **Hexagonal / Shared-kernel** | Mỗi agent có core riêng + module shared dùng chung | 11 shared module qua symlink (`models/`, `services/`) | Áp dụng nguyên tắc "Extendable" (TCB Principle 03) |

### 4.2. Pattern agent (theo phân loại của TCB § 5.3)

| Pattern TCB | Định nghĩa | TÀI Studio — Hiện trạng |
| --- | --- | --- |
| **Planning Agent** | Agent tự lập + thực thi kế hoạch đa bước | ✅ TÀI ReAct loop tự chain nhiều tool: "Translate → Now summarize it" |
| **Task-Oriented Agent** | Agent chuyên 1 task | ✅ 8/8 specialist (Translator, Summarizer, PPTX-er, Vaulter, Brainstormer, Visionary, Canvas, Resume) |
| **Coordinating Agent** | Agent phối hợp + giao task | ✅ TÀI (qua tool calling) |
| **Hierarchical Agent** | Agent cấp cao quản lý agent cấp thấp | ◐ Một phần — TÀI là cấp cao, specialist là cấp thấp, nhưng specialist không phân quyền tiếp |
| **Context-Aware Agent** | Điều chỉnh hành vi theo context | ✅ Brainstormer (load_preferences node); TÀI (user_preferences cross-session) |
| **Human-in-the-Loop** | Bán tự động, người duyệt quyết định | ✅ `interrupt()` cho file upload; ✅ Canvas/PPTX cho phép edit trước export |
| **RAG-Based Agent** | Truy xuất KB để augment | ◐ Vaulter (knowledge graph, không vector); Visionary (proxy KA endpoint) |
| **Reflective Agent** | Tự đánh giá output để cải thiện | ❌ Chưa có — đề xuất ở § 4 Improvement Suggestions |
| **Self-Learning** | Học từ feedback | ❌ Chưa có |

### 4.3. Pattern triển khai (Implementation)

| Pattern | Mô tả | Minh chứng cụ thể |
| --- | --- | --- |
| **Hybrid LLM Tiering** | Định tuyến request theo độ phức tạp + cost | Translator: Haiku cho segment <50 chars, Sonnet cho >50 chars (`_HAIKU_CHAR_THRESHOLD = 50`); Canvas Designer dùng Opus cho creative; Guardrails dùng Haiku |
| **Message Compaction** | Summarize-then-trim khi vượt ngưỡng | `compact_messages.py` — trigger ở 10 messages, giữ last 6, summary các message cũ qua Haiku |
| **Checkpoint Pattern** | Lưu state để resume sau interrupt/restart | `AsyncCheckpointSaver` (Lakebase) trong Brainstormer + TÀI |
| **Interrupt-Resume (HITL)** | Pause graph → render UI → resume với input mới | TÀI `request_file_upload` tool → `interrupt()` → frontend POST `/api/tai/resume` |
| **Guardrail-as-Classifier** | LLM rẻ phân loại input/output là safe/unsafe trước khi gọi LLM chính | Powerpoint-er `check_input_guardrails()` + `check_output_guardrails()` với Haiku |
| **Structured Output via Delimiter** | LLM trả về reply + JSON tách bằng delimiter | Brainstormer `---CANVAS---`; TÀI `---UI---` |
| **Tool Result Wrapping** | Mỗi tool = wrap HTTP call tới microservice + SP auth | TÀI tools wrap REST call tới agent API; token lấy từ `WorkspaceClient._token()` |
| **SSE Streaming for Long-Running Task** | Trả progress qua Server-Sent Events | Translator: `event: plan` → `event: progress` × N → `event: result` |
| **Parallel Worker Pool with Cap** | ThreadPoolExecutor cap số worker để cân bằng throughput vs LLM quota | Translator `_MAX_PARALLEL = 5` |
| **Multi-Strategy JSON Parsing** | Nhiều chiến lược fallback khi LLM trả JSON sai | Powerpoint-er `parse_llm_json()`: direct → brace match → fix trailing comma → regex |
| **Graceful Degradation** | Lỗi không block — fallback về output đơn giản hơn | Powerpoint-er per-slide try/except → `fallback_html()`; Vaulter graph extraction fail → empty graph |
| **Lazy Initialization** | Init resource expensive khi cần | MLflow init lazy ở first `get_llm()` call để tránh startup crash |

---

## 5. Khoảng cách giữa Vision (TCB) và Implementation (TÀI)

| Capability TCB | Trạng thái TÀI Studio | Khuyến nghị bổ sung |
| --- | --- | --- |
| **C01 Autonomous Decision-Making** | ✅ Có (ReAct loop) | Maintain Level 1-2; tránh autonomous Level 3+ cho banking |
| **C02 A2A + MCP** | ❌ Không A2A; ❌ Không MCP | Cần khi tích hợp với agent của hệ thống khác (CRM, BPM); LangFlow đã hỗ trợ MCP server |
| **C03 Popular Patterns** | ✅ 6/9 pattern | Thiếu Reflective, Self-Learning — cần khi muốn agent cải thiện theo thời gian |
| **C04 Context & Memory** | ✅ Đầy đủ (short + long term + compaction) | Có thể thêm vector memory cho semantic recall |
| **C05 No-Code Builder** | ❌ Không có | Bổ sung LangFlow như TCB đề xuất khi mở rộng cho BA/non-tech user |
| **C06 Security Controls** | ◐ Một phần (guardrail-as-classifier ở 1 agent) | Tích hợp Trellix DLP (endpoint), Bedrock Guardrails (system), prompt validation tập trung, audit log → đúng theo data flow của DAB0 § 4.7.1 |

---

## 6. Kết luận

### 6.1. Cấu phần tối thiểu cho một sản phẩm agentic

Một sản phẩm agentic enterprise-grade phải có ít nhất **9 cấu phần** xếp theo 3 tầng:

```
┌─────────────────────────────────────────────────────────┐
│  TẦNG GOVERNANCE   │  Security/Guardrails   Observability │
├─────────────────────────────────────────────────────────┤
│  TẦNG CORE         │  Orchestration   LLM Gateway        │
│                    │  Tool Layer      State/Memory        │
│                    │  Knowledge (RAG) HITL Interface      │
├─────────────────────────────────────────────────────────┤
│  TẦNG INFRA        │  File Storage    Deployment          │
│                    │  Shared SDK                          │
└─────────────────────────────────────────────────────────┘
```

Trong đó **6 cấu phần CORE là bắt buộc** ngay từ MVP (Orchestration, LLM Gateway, Tool, Memory, Storage, Observability), **3 cấu phần GOVERNANCE** có thể defer tới giai đoạn scale nhưng phải có roadmap rõ ràng.

### 6.2. Design pattern then chốt

3 pattern *bắt buộc* để có "agentic" thực sự (không chỉ là chatbot):

1. **ReAct Loop + Tool Calling** — vòng reason-act-observe với tool external; là DNA của agentic
2. **Stateful Orchestration** — workflow đa bước có state, checkpoint, cycle (LangGraph StateGraph)
3. **Human-in-the-Loop với Interrupt-Resume** — bắt buộc cho banking (Level 1-2 maturity)

4 pattern *làm sản phẩm production-ready*:

4. **Hybrid LLM Tiering** — cost vs quality balance (Haiku/Sonnet/Opus)
5. **Message Compaction** — vượt context window
6. **Guardrail-as-Classifier** — input/output safety check rẻ và nhanh
7. **Graceful Degradation** — agent fail không làm crash workflow

### 6.3. Bài học rút ra từ so sánh

- **Blueprint không thay được implementation**: TCB Platform đề xuất 6 capability, TÀI Studio cho thấy C01-C05 hiện thực được bằng LangGraph + Databricks Apps, nhưng C06 (Security Controls) cần stack riêng (Trellix + Bedrock Guardrails) mà code-level guardrail không thay thế được.
- **Centralized orchestration > A2A** ở giai đoạn đầu: TÀI chọn 1 orchestrator + N tool (đơn giản, debug được) thay vì agent-to-agent mesh — phù hợp với rủi ro cao của banking.
- **Shared abstractions là điều kiện sống còn khi >3 agent**: 11 module dùng chung trong TÀI Studio chứng minh việc không có chúng → bug nhân với N.
- **No-Code Builder là gap chiến lược**: TÀI Studio dev-only → muốn democratize cho BA/non-tech như mục tiêu DAB0, cần LangFlow hoặc tương đương.

---

> **Định nghĩa hoàn thành (DoD) cho document này:**
> - ✅ Phân tích kiến trúc 2 sản phẩm theo 7 layer của TCB Platform
> - ✅ Liệt kê 9+3 cấu phần với phân loại Must/Should/Could
> - ✅ Phân loại design pattern theo 3 nhóm (Architectural / Agent / Implementation) với bằng chứng cụ thể
> - ✅ Đối chiếu gap giữa TCB vision và TÀI implementation
> - ✅ Trích dẫn rõ nguồn (`architecture.md`, `tai_architecture.md`, `tai_list_agent.md`)
