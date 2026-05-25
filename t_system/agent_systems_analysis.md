# Phân tích Agent Systems trong `t_system/`
[CẬP NHẬT: 25/05/2026]

> **Phạm vi**: Phân tích 3 folder trong `t_system/` dưới góc nhìn AI Engineer chuyên agent systems. Mỗi folder = 1 hệ thống độc lập (1 blueprint + 2 implementation).
> **Source T1**: `bankwide_agentic_platform/agent_platform.md`, `agent_platform/agent_studio_*.md`, `agentic/Agent_analytics.md`.
> **Convention**: tuân thủ `t_system/CLAUDE.md` — bảng > prose, citation inline `(file § section)`, gap ✅/◐/❌, recommendation Must/Should/Could.

## Mục lục

- [§A. `agent_platform/` — Agent Studio (Implementation of Record)](#a-agent_platform--agent-studio-implementation-of-record)
- [§B. `agentic/` — AI Analytics Agent (Use case riêng)](#b-agentic--ai-analytics-agent-use-case-riêng)
- [§C. `bankwide_agentic_platform/` — Blueprint vision](#c-bankwide_agentic_platform--blueprint-vision)
- [§D. So sánh chéo & Kết luận](#d-so-sánh-chéo--kết-luận)

---

# §A. `agent_platform/` — Agent Studio (Implementation of Record)

Source: `agent_studio_architecture.md` (225 dòng) + `agent_studio_agents.md` (1,450 dòng).

## A.1. Tổng quan

| Hạng mục | Giá trị | Citation |
| --- | --- | --- |
| **Mục đích** | Super app bundling 9 AI agent chuyên biệt trên Databricks Apps; mỗi agent giải 1 task workplace (translate, summarize, PPTX, vault, brainstorm, vision, canvas, resume) | (architecture.md § Part 1 → System Purpose) |
| **Use case chính** | Employee productivity — không cần biết agent nào, gõ message vào Hub là xong | (agents.md § Hub → Business Purpose) |
| **Framework** | **LangGraph + LangChain** (StateGraph cho Brainstormer, ReAct + ToolNode cho Hub); **FastAPI** backend; **React + Vite + Tailwind** frontend | (architecture.md § Technology Stack) |
| **LLM Provider** | Claude Opus 4.5 / Sonnet 4.5 / Haiku 4.5 qua **Databricks Foundation Model API** (`ChatDatabricks`) | (architecture.md § Technology Stack) |
| **Vector DB** | ❌ Không dùng — Vaulter chỉ load 5 doc gần nhất, không có embedding | (agents.md § Vaulter → Known Limitations) |
| **State storage** | **Lakebase (PostgreSQL-compatible)** — checkpoint, session index, user preferences, vault graph | (architecture.md § Technology Stack) |
| **Kiến trúc** | **Hierarchical hub-and-spoke**: 1 Hub orchestrator + 8 specialist agents, không có A2A trực tiếp giữa specialist | (architecture.md § Patterns 1-2) |

### Sơ đồ kiến trúc tổng thể

```
                    ┌─────────────────────────────────┐
User ──► Hub Chat ──►   LangGraph ReAct (Sonnet)     │
                    │   bind_tools(ALL_TOOLS)         │
                    └────────────┬────────────────────┘
                                 │
       ┌────────┬────────┬───────┼────────┬───────────┬──────────┐
       ▼        ▼        ▼       ▼        ▼           ▼          ▼
  Translator Summarizer PPTX-er Vaulter Brainstormer Visionary  (Canvas /
   (HTTP +     (HTTP)   (HTTP)  (HTTP)   (HTTP)      (proxy KA)  Resume:
    SSE)                                                          chỉ iframe
                                                                  chưa có
                                                                  Hub tool)
       │                                  │
       ▼                                  ▼
  Volume + LLM                       Lakebase
  parallel chunks                    (checkpoints +
                                      preferences)
```

(architecture.md § Patterns 1-3; agents.md § Hub → LangGraph ReAct Graph)

---

## A.2. Core Components

### A.2.1. Agent loop

**3 pattern cùng tồn tại** trong cùng codebase:

| Pattern | Agent dùng | Mô tả |
| --- | --- | --- |
| **ReAct loop** | Hub | `agent ⇄ tools → update_preferences → END`; LLM quyết định gọi tool nào dựa trên `tool_calls` trong response (agents.md § Hub → LangGraph ReAct Graph) |
| **StateGraph 4-step** | Brainstormer | `load_preferences → chat → update_preferences → END` + **forced wrap-up** ở message thứ 8 (agents.md § Brainstormer → LangGraph StateGraph + Forced Wrap-up Rule) |
| **Sequential pipeline** | Translator, Summarizer, PPTX-er, Resume Evaluator, Visionary, Canvas | Không có agent loop — đơn giản parse → LLM call → format response. Đây là "agent" theo nghĩa lỏng (single-shot inference) |

**Code minh hoạ** — Hub ReAct loop dispatch logic (mô tả ở agents.md § Hub → Agent Node):

```python
# Pseudo từ agents.md mô tả
def agent_node(state):
    messages = compact_messages(state.messages)  # Haiku-based summarization
    response = sonnet.bind_tools(ALL_TOOLS).invoke(messages)
    if response.tool_calls:
        return "tools"          # → ToolNode
    else:
        return "update_preferences"  # → END
```

### A.2.2. Tools / Functions

**Hub có 8 tool** (agents.md § Hub → Tools):

| Tool | Wraps | Pattern |
| --- | --- | --- |
| `request_file_upload` | `interrupt()` của LangGraph | **HITL** — pause graph chờ file user upload |
| `translate_document` | HTTP `POST → Translator API` (SSE) | Đồng bộ blocking |
| `summarize_documents` | HTTP → Summarizer | — |
| `generate_presentation` | HTTP → PPTX-er | — |
| `upload_to_vault` / `query_vault` | HTTP → Vaulter | — |
| `brainstorm` | HTTP → Brainstormer | — |
| `ask_visionary` | HTTP → Visionary (proxy KA) | — |

> ❌ **Gap**: Canvas Designer và Resume Evaluator chưa có Hub tool — chỉ truy cập qua iframe direct URL (architecture.md § Agent Interaction Matrix).

### A.2.3. Memory

| Loại | Cơ chế | Agent |
| --- | --- | --- |
| **Short-term (in-conversation)** | LangGraph `AsyncCheckpointSaver` → Lakebase | Hub, Brainstormer, Visionary |
| **Mid-term (message compaction)** | Summarize-then-trim: tóm tắt qua Haiku khi >10 messages, giữ 6 message gần nhất | `compact_messages.py` shared (architecture.md § Shared Abstractions) |
| **Long-term (cross-session)** | `UserPreferencesStore` — language, tone, topics extract bằng LLM, lưu Lakebase | Brainstormer, Hub |
| **Episodic (knowledge graph)** | Lakebase tables `nodes` / `edges` + `documents` | Vaulter (agents.md § Vaulter) |
| **Vector embeddings** | ❌ **KHÔNG có** — Vaulter load all docs as context, truncate 10K chars | (agents.md § Vaulter → Known Limitations) |

### A.2.4. Prompts

**3 kỹ thuật prompt engineering** (architecture.md § Prompting Style):

1. **Structured JSON output** — yêu cầu LLM trả raw JSON, không markdown fence. Shared util `parse_llm_json()` có **4-strategy fallback** (direct `json.loads` → brace matching → fix trailing commas → regex) (agents.md § PPTX-er → JSON Parsing).
2. **System + Human message pattern** — SystemMessage chi tiết, HumanMessage gọn.
3. **Delimiter pattern**:
   - `---CANVAS---` (Brainstormer): tách conversational reply khỏi structured canvas
   - `---UI---` (Hub): embed frontend component instructions
4. **Guardrails 2 chiều** (PPTX-er) — Haiku classify input safe/unsafe trước LLM, output safe/unsafe sau LLM (agents.md § PPTX-er → Guardrails).

### A.2.5. State management

```
User Request → FastAPI handler
            → LangGraph state (TypedDict: messages, preferences, ...)
            → AsyncCheckpointSaver (Lakebase) sau mỗi node
            → Resume từ checkpoint khi user gửi message tiếp
```

State transfer giữa Hub ↔ Agent specialist qua **HTTP body + SP token header** — không share state object, chỉ pass `volume_path` (file location) (agents.md § Hub → Tool Execution).

---

## A.3. Control Flow

### A.3.1. Entry point & execution pipeline

```
[Frontend React]
   │ POST /api/hub/chat { message, session_id }
   ▼
[FastAPI Hub backend]
   │
   ▼
[LangGraph.invoke(state, checkpoint_id=session_id)]
   │
   ├─► load_preferences (Lakebase read)
   │
   ├─► agent_node ─┬─► [no tool_calls] → update_preferences → END
   │               │
   │               └─► [tool_calls] → ToolNode
   │                                    │
   │                                    ├─► request_file_upload → interrupt() → 202 Interrupted
   │                                    │                              │
   │                                    │      [Frontend uploads]      │
   │                                    │      POST /api/hub/resume────┘ (graph resume)
   │                                    │
   │                                    └─► HTTP call to specialist agent
   │                                          (httpx + SP token header)
   │                                          → tool result back to agent_node
   │
   └─► update_preferences (Lakebase write) → END
   
→ Return { reply, session_id, ui_actions, interrupt? }
```

(agents.md § Hub → User Flow + LangGraph ReAct Graph + HITL)

### A.3.2. Decision logic

| Khi nào | Quyết định |
| --- | --- |
| Gọi LLM | Mỗi turn user message → 1 call Sonnet (`agent_node`); message compaction trigger khi >10 msg |
| Gọi tool | LLM trả `tool_calls` array → ToolNode dispatch theo tool name |
| HITL pause | LLM gọi `request_file_upload` → `interrupt()` raise; frontend nhận `status: "interrupted"` |
| Dừng | LLM trả message không có `tool_calls` → route `update_preferences` → END |
| Force end (Brainstormer) | Message thứ 8 → inject system message: "You MUST wrap up now" (agents.md § Brainstormer → Forced Wrap-up Rule) |

### A.3.3. Error handling / Retry / Fallback

| Cơ chế | Áp dụng | Citation |
| --- | --- | --- |
| Retry LLM call | ◐ **Chỉ Translator** retry 2x cho structured translation; các agent khác **không retry** | (architecture.md § Bottlenecks: No retry on LLM calls) |
| Fallback prompt | ✅ PPTX-er: nếu LLM fail per-slide → `fallback_html()` (plain bullets) | (agents.md § PPTX-er → Error Handling) |
| Fallback parse | ✅ `parse_llm_json` 4 strategy; Vaulter trả empty graph khi parse fail | (agents.md § Vaulter → Error Handling) |
| Guardrails | ✅ PPTX-er: Haiku classify safe/unsafe input + output → HTTP 400/502 | (agents.md § PPTX-er) |
| Validation upload | ✅ `file_validator.py` shared: extension + size ≤20MB + magic bytes | (architecture.md § Shared Abstractions) |

---

## A.4. Đánh giá

### Điểm mạnh

1. **Shared abstractions tốt** — 11 module dùng chung (`llm.py`, `compact_messages.py`, `file_store.py`, `user_context.py`, `upload_router.py`, `schemas.py`) giảm copy-paste (architecture.md § Shared Abstractions).
2. **Hybrid LLM strategy** — Haiku cho task ngắn/cheap (translate <50 chars, preference extract, compaction), Sonnet default, Opus cho creative (Canvas) → cost-optimized (architecture.md § LLM Model Decision Tree).
3. **HITL elegant** — `interrupt()` + `Command(resume=...)` của LangGraph giải bài toán "agent cần file từ user" cleanly (agents.md § Hub → HITL).
4. **Message compaction** — summarize-then-trim ở 10 msg, giữ 6 msg → tránh context overflow mà không mất context cũ.
5. **Observability nhẹ nhưng đủ** — MLflow autolog mỗi agent có experiment ID riêng (architecture.md § Deployment).

### Điểm yếu

| Risk | Mức độ | Chi tiết |
| --- | --- | --- |
| **Single LLM endpoint** | 🔴 High | Tất cả agent share `databricks-claude-sonnet-4-5` — endpoint throttle/down → toàn bộ system fail (architecture.md § Bottlenecks) |
| **No retry trên hầu hết LLM call** | 🟠 Medium | Transient LLM error fail luôn request (chỉ Translator retry) |
| **No rate limiting** | 🟠 Medium | Single user có thể exhaust LLM quota |
| **No vector search (Vaulter)** | 🟠 Medium | Load all docs as context — không scale beyond ~50K chars |
| **Hub thiếu 2 tool** | 🟡 Low | Canvas + Resume chưa accessible qua Hub orchestration |
| **SQL injection bề mặt** | 🟡 Low | `file_parser.py` interpolate volume_path vào SQL string (mitigated bằng regex `[A-Za-z0-9/_.-@]+` nhưng không parameterized) (architecture.md § Security Concerns) |
| **CORS allow localhost** | 🟡 Low | Production nên bỏ `http://localhost:5173` |
| **No streaming response** | 🟡 Low | Hub trả entire response 1 lần, UX kém với multi-tool chain |

### Scalability & Observability

- **Scalability**: ◐ Mỗi agent độc lập deploy được, nhưng **single-instance** — chưa load balance; batch resume eval sequential, max 10 candidate (agents.md § Resume → Known Limitations).
- **Observability**: ✅ MLflow autolog + structured logger per agent; ❌ không có distributed tracing cross-agent (không correlate Hub call → Translator call).
- **Security boundary**: ◐ X-Forwarded-Email auth chỉ valid khi qua Databricks Apps proxy — bypass proxy = no auth (architecture.md § Security Concerns).

### Đề xuất cải tiến cụ thể

**Must**:
- **(M1)** Thêm fallback LLM endpoint config — nếu Sonnet primary throttled → switch sang secondary (Bedrock/Azure mirror).
- **(M2)** Retry với exponential backoff cho **mọi** LLM call (tối thiểu 2 lần), không chỉ Translator.
- **(M3)** Thêm Hub tool cho Canvas Designer + Resume Evaluator (architecture.md § Improvement Suggestions → AI/Agent Design).

**Should**:
- **(S1)** Migrate Vaulter sang **vector embedding + similarity search** (Databricks Vector Search) thay vì truncate 10K chars.
- **(S2)** Dùng LangChain `with_structured_output()` thay JSON-in-prompt → loại bỏ `parse_llm_json` 4-strategy fragility.
- **(S3)** Streaming token-by-token cho Hub + Brainstormer (SSE đã có sẵn ở Translator — replicate pattern).
- **(S4)** Shared API gateway: centralize auth + rate limit + CORS thay vì duplicate ở mỗi agent.

**Could**:
- **(C1)** Distributed tracing với MLflow trace ID propagate qua header `X-Trace-Id` → correlate Hub → Specialist calls.
- **(C2)** Tool result caching trong session — Hub gọi cùng tool với cùng args → reuse result.
- **(C3)** Canvas Designer chuyển từ pixel-by-pixel sang numpy/Cairo (architecture.md § Performance: Canvas pixel rendering Poor).

---

# §B. `agentic/` — AI Analytics Agent (Use case riêng)

Source: `Agent_analytics.md` (705 dòng) + `A2A_MCP.jpg`, `Agent_reasoning_graph.png`.

## B.1. Tổng quan

| Hạng mục | Giá trị | Citation |
| --- | --- | --- |
| **Mục đích** | AI analytics agent cho banking domain — CASA, Credit Card, Customer 360, AUM/TRV analysis cho Techcombank | (Agent_analytics.md § Introduction + Supervisor Prompt v2) |
| **Use case chính** | Trả lời câu hỏi BI dạng "Why does X decrease?" / "AUM analysis for PnP HH" qua multi-agent BI synthesis | (Agent_analytics.md § Routing Strategy Examples) |
| **Framework** | **LangGraph raw nodes** (prototype) → đang **migrate sang A2A + MCP** (proposal) | (Agent_analytics.md § Introduction) |
| **LLM Provider** | Không khai báo cụ thể trong doc — context "TalkZone" platform của Techcombank | (Agent_analytics.md § Final Answer Prompt) |
| **Vector DB** | ❌ Không đề cập — pattern dựa vào Genie SQL agent, không RAG |  |
| **Kiến trúc** | **Supervisor + Multi-agent specialist** (CASA_Expert, CCB_Expert, AE_Expert, TD_Expert, customer360_agent) với **parallel executor** | (Agent_analytics.md § v2 SYSTEM_PROMPT + PARALLEL EXECUTION PLANNING) |

> Tài liệu này không phải implementation chi tiết — là **knowledge notes + prompt design + 2 phiên bản supervisor prompt** (v1 + v2). Có proposal kiến trúc nhưng chưa có code path.

### Sơ đồ kiến trúc (mô tả)

```
User Question
     │
     ▼
┌────────────────────────────────────────────┐
│ Supervisor Agent (LLM, Techcombank-aware)  │
│  - Phân tích: simple / complex / cross-prod│
│  - Quyết định should_plan_research T/F     │
└──────┬─────────────────────────────────────┘
       │
   ┌───┴───┐
   │       │
[simple]  [complex]
   │       │
   ▼       ▼
Single   ParallelExecutor
Genie    │
 Agent   ├─► CASA_Expert (Genie SQL)
         ├─► AE_Expert
         ├─► TD_Expert  
         ├─► CD_Expert
         ├─► Bond_Expert
         ├─► Fund_Expert
         └─► customer360_agent
                │
                ▼
       Aggregated results
                │
                ▼
       Final Answer Prompt (forced format)
       - General Trends + Chart
       - Root Cause Analysis
       - Summary 1-phrase
```

(Agent_analytics.md § v2 → ROUTING STRATEGY + AUM Analysis Example)

---

## B.2. Core Components

### B.2.1. Agent loop

**Pattern**: **Supervisor → Planner → Parallel Executor → Synthesizer** (Agent_analytics.md § v2 SYSTEM_PROMPT).

Khác Agent Studio (Hub-and-spoke ReAct), đây là **Plan-and-Execute** với parallel branch:

```
loop {
    supervisor_decision = supervisor_llm(original_q, utilized_agents, count)
    if supervisor_decision.next_node == "FINISH":
        break
    if supervisor_decision.should_plan_research:
        research_plan = [
            {"query": q, "target_agent": a, "reasoning": r}
            for q, a, r in supervisor_decision.queries
        ]
        results = parallel_execute(research_plan)  # asyncio.gather equiv
    else:
        results = single_genie_call(supervisor_decision.next_node)
    utilized_agents.append(...)
    count += 1
}
final_answer = final_answer_llm(context=all_results)
```

### B.2.2. Tools / Sub-tasks

**Triết lý cốt lõi** (Agent_analytics.md § Best practice for building AI Agent using LLM):

> "Never use LLM to reason on-the-fly" — chia task thành sub-task có **structured JSON output**, để LLM chỉ làm summarize/rewording.
> "Never use LLM to do calculation or determine trend" — dùng Python + Statistical test với p-value.

**5 sub-task standardized** với schema JSON cụ thể (Agent_analytics.md § Example output at each step):

| Sub-task | Output schema |
| --- | --- |
| Trend detection | `{trend_direction, change_value, change_pct}` |
| Root cause analysis | `{internal_factors[], external_factors[], customer_behavior[]}` |
| Sector contribution | `{segments: {VIP, Mass}, sectors: {Retail, SME}}` |
| Recommendation | `{short_term_action, long_term_action, priority}` |
| Confidence | `{confidence_score, explanation}` |

### B.2.3. Memory

| Loại | Trạng thái |
| --- | --- |
| Short-term | ◐ `utilized_agents` list + `count` truyền qua mỗi iteration của supervisor — minimalist state |
| Long-term | ❌ Không đề cập |
| Episodic | ❌ Không đề cập |
| **Temporal context** | ✅ Supervisor được inject "temporal context" để transform "current year" → "2025", "recent" → "12 2025" (Agent_analytics.md § Temporal Context Handling) |

### B.2.4. Prompts

**Đây là phần đậm đặc nhất của file** — có **2 phiên bản** supervisor prompt + final answer prompt:

**Version 1** (Agent_analytics.md § Supervisor Prompt v1):
- 5-LAYER framework: Domain → Segmentation → Metrics → Time → Routing
- Routing rule: customer context first → product performance second
- Output: `{rephrase_question, next_node, reasoning, confidence}`

**Version 2** (Agent_analytics.md § SYSTEM_PROMPT v2):
- Thêm `should_plan_research: bool` + `research_plan` schema
- Thêm `parallel_executor` cho complex multi-product (AUM = CASA + AE + TD + CD + Bond + Fund)
- **Genie follow-up handling**: nếu Genie hỏi clarification → supervisor PHẢI trả lời và route lại Genie, không FINISH

**Final Answer Prompt** — forced output format cho Data Analysis question (Agent_analytics.md § final_answer_prompt v2):

```
1. General Trends:
   (Chart of X metric)
   "X of [group] from [period] dropped to [amount] in [month]..."

2. Root Cause Analysis:
   "[N]K customers ([%]) with >=[threshold] X accounted for [%] X 
    of total book on [date]. During [period], drop concentrated in
    this group, accounting for ~[amount] X drop"

3. Summary (1 phrase):
   "[Dominant group] [dropped/increased] X during [period], 
    drove the [up/down] trend"
```

### B.2.5. Domain knowledge embedded in prompt

Đây là **strength chính** của thiết kế này — supervisor prompt nhúng **toàn bộ ontology nghiệp vụ ngân hàng** vào system message (Agent_analytics.md § LAYER 1-4):

- **Domains**: RBG (CASA, CCB, TD, CD, Bond, Fund, Loan, Banca) vs CIBG (FX, OTT, SCF, H2H)
- **Customer tiers**: PRIVATE > PRIORITY > INSPIRE > NON TIER
- **Activity status**: MTB > TRULY ACTIVE > ACTIVE > INACTIVE
- **Economic segments**: AFF > MAF > EMAF > MAS
- **Occupation**: SA / HH / BO-TL / Other
- **Metrics formula**: AUM = CASA + AE + TD + CD + Bond + Fund (avg 3M); TRV = AUM + Lending; TOI = NII + NFI + FX Sales + Trading + Other

→ LLM không cần "biết" ngân hàng, chỉ cần **route đúng theo ontology đã cho**.

### B.2.6. Reasoning process for Root Cause Analysis

**Pattern đặc biệt** — không để LLM tự "reason" mà **scripted reasoning** (Agent_analytics.md § 2a For CASA & AUM Trend):

1. Thử **sequential / parallel multiple dimensions** (1D, 2D, 3D combinations) để tìm nhóm khách hàng lớn nhất giải thích trend
2. Dùng **Genie để tìm threshold** ứng với percentile >75% / >80% của distribution (không pick threshold tùy ý)
3. Tính 6 chỉ số: (a) count nhóm, (b) % trên total, (c) % proportion ở thời điểm bắt đầu, (d) threshold value, (e) value change, (f) % của all change

**CashFlow Analysis** dùng pattern khác — **correlation-based** thay percentile-based:

1. Tìm label tích lũy đến 80% trong mỗi Dim (Pareto)
2. Tính **correlation** giữa CashFlow In và từng Label
3. Label có correlation ≥ 90% với CashFlow pattern → đó là explanation

---

## B.3. Control Flow

### B.3.1. Entry point & pipeline

```
User: "Why does AUM for PnP HH drop in H1 2025?"
   │
   ▼
[Supervisor v2]
   - Parse: question type = "Why...drop" = Analytical/Diagnostic
   - Domain: RBG → CASA + Customer360
   - Complexity: COMPLEX (AUM needs CASA + AE + TD + CD + Bond + Fund)
   - Set should_plan_research = True
   - research_plan = [
       {q: "CASA EOP balance for PnP HH 2025 H1", target: CASA_Expert},
       {q: "AE portfolio for PnP HH 2025 H1", target: AE_Expert},
       {q: "TD performance for PnP HH 2025 H1", target: TD_Expert},
       ...
     ]
   │
   ▼
[ParallelExecutor]
   - asyncio.gather across all expert agents
   - Mỗi Genie agent: NL → SQL → data
   │
   ▼
[Aggregator]
   - Merge results
   - Compute AUM = sum(products)
   │
   ▼
[Final Answer Prompt]
   - Force format: 1. General Trends + 2. Root Cause + 3. Summary
   - LLM chỉ rewording (calculation done in Python)
```

### B.3.2. Decision logic

| Khi nào | Quyết định |
| --- | --- |
| **Route single agent** | Question 1 domain, simple performance query |
| **Route parallel_executor** | Multi-product calc (AUM), cross-product comparison, comprehensive analysis |
| **Route FINISH** | Sufficient info từ previous iterations, hoặc question out-of-scope |
| **Handle Genie clarification** | **CRITICAL** — nếu Genie hỏi `?` → supervisor TRẢ LỜI thay user và route lại Genie, không FINISH (Agent_analytics.md § CRITICAL: Handling Genie Follow-up Questions) |
| **Bias** | When in doubt → bias toward single Genie (`should_plan_research=False`) |

### B.3.3. Error handling / Retry / Fallback

| Vấn đề | Cách xử lý trong doc |
| --- | --- |
| Infinite loop routing | "Avoid infinite loops — don't route to same agent repeatedly" (rule trong prompt, không có code guard) |
| LLM trả invalid JSON | ❌ Không đề cập |
| Genie agent fail | ❌ Không đề cập |
| Truncation data | ✅ "Note data limitations if truncation affects analysis" (rule prompt) |

---

## B.4. Đánh giá

### Điểm mạnh

1. **Triết lý thiết kế đúng đắn** — "Never use LLM to reason on-the-fly" → break task thành sub-task có schema JSON, LLM chỉ rewording, calculation/trend ở Python. Đây là pattern matured nhất cho production BI.
2. **Ontology nhúng sâu** — toàn bộ ngữ pháp nghiệp vụ Techcombank được encode trong supervisor prompt → LLM không cần fine-tune.
3. **Scripted reasoning cho root cause** — percentile + correlation-based, không phải LLM tự nghĩ ra → deterministic và reproducible.
4. **Forced output format** — giảm variance giữa các lần chạy, dễ test/eval.
5. **Parallel executor** — xử lý multi-product AUM với latency = max(branch) thay vì sum.

### Điểm yếu

| Risk | Mức độ | Chi tiết |
| --- | --- | --- |
| **Document-only, no implementation** | 🔴 High | Đây là knowledge notes + prompt design, chưa có code path / repo / tests minh chứng |
| **Migration LangGraph → A2A+MCP chưa có blueprint chi tiết** | 🟠 Medium | Doc nói "we propose adopting A2A + MCP" nhưng chỉ ở mức rationale, chưa có schema cụ thể (Agent_analytics.md § Introduction) |
| **Prompt v1 vs v2 chưa rõ adoption status** | 🟠 Medium | Có 2 phiên bản, không nói version nào đang chạy production |
| **Loop guard chỉ là rule trong prompt** | 🟠 Medium | "Don't route to same agent repeatedly" — không có code-level check (count limit) |
| **Test cases sơ sài** | 🟡 Low | Test 1 + Test 2 ở cuối doc — Expected Answer trống, chỉ có Question + Answer thực tế (Agent_analytics.md § Test 1, Test 2) |
| **Confidence score chưa có method** | 🟡 Low | Schema output có `confidence_score: 0.85` nhưng không nói cách tính |

### Scalability & Observability

- **Scalability**: ✅ Parallel executor cho multi-product → tốt; ❌ chưa có concurrency limit / rate limit / queue
- **Observability**: ❌ Không đề cập MLflow / LangFuse / tracing
- **Bảo mật**: ❌ Không bàn — supervisor prompt expose toàn bộ ontology nghiệp vụ → prompt injection có thể leak Techcombank metric formula (low risk vì là kiến thức công khai mức nghiệp vụ)

### Đề xuất cải tiến cụ thể

**Must**:
- **(M1)** Implement code-level loop guard (`max_iterations=5`) ngoài rule trong prompt.
- **(M2)** Pin xuống 1 version supervisor prompt làm "of record"; deprecate phiên bản còn lại.
- **(M3)** Bổ sung handling khi Genie agent timeout / fail (currently undefined).

**Should**:
- **(S1)** Migrate sang **A2A + MCP** với schema OpenAPI/JSON Schema cụ thể (theo proposal trong doc).
- **(S2)** Tách **ontology** (tier, segment, metric formula) ra **MCP resource** thay vì hard-code prompt — dễ update khi business thay đổi.
- **(S3)** Implement confidence_score bằng Bayesian / sample variance trên Genie SQL results, không để LLM tự gán.
- **(S4)** Thêm Pydantic schema validation cho mỗi sub-task output — fail-fast khi LLM trả JSON sai shape.

**Could**:
- **(C1)** Tracing: integrate LangFuse hoặc MLflow để trace Supervisor → Parallel → Aggregator chain.
- **(C2)** Cache Genie SQL results trong session (cùng query → reuse).
- **(C3)** Eval harness: dataset 50-100 question + expected answer → regression test khi update prompt.

---

# §C. `bankwide_agentic_platform/` — Blueprint vision

Source: `agent_platform.md` (486 dòng) + 3 ảnh (`AI_Agent_caps.jpg`, `Agent_components.jpg`, ảnh schema).

## C.1. Tổng quan

| Hạng mục | Giá trị | Citation |
| --- | --- | --- |
| **Bản chất tài liệu** | **Endorsement / approval document** — đề xuất adopt Agentic AI Platform cho ngân hàng | (agent_platform.md § 1 Introduction → In-scope) |
| **Mục đích** | Justify WHY (pain points) + WHAT (capabilities) + HOW (target architecture) ở level chiến lược | (agent_platform.md § Table of Contents) |
| **Use case scope** | 3 use case: (1) Post-Call Workflow automation, (2) Customer 360 Assistant, (3) BA GenAI Assistant | (agent_platform.md § 5.1) |
| **Framework đề xuất** | **LangChain + LangGraph + LangFlow + LangFuse** (open-source stack) | (agent_platform.md § 4.4 + 4.5) |
| **LLM hosting** | **AWS Bedrock Serverless** + Bedrock Marketplace → SageMaker EC2 cho non-available models | (agent_platform.md § 4.7.2 Compliance) |
| **Vector DB** | AWS Bedrock KB (OpenSearch / S3 Vector) + MongoDB Vector DB | (agent_platform.md § 4.5 Tech Stack item 9) |
| **Maturity target** | Level 1-2 (current Level 0) — HITL decision-making, không full autonomy | (agent_platform.md § 4.2.3) |

> Đây **không phải** kiến trúc chi tiết của 1 hệ thống đang chạy — là **blueprint endorsement** cho hệ thống tương lai. Detailed Design Document được referenced nhưng không attached.

## C.2. Core Components (Blueprint level)

### C.2.1. Targeted Capabilities (6 capability — C01 → C06)

(agent_platform.md § 3.2)

| ID | Capability | Mô tả ngắn |
| --- | --- | --- |
| **C01** | Autonomous Decision-Making | Reason → plan → execute với LLM; **bắt đầu ở low autonomy** |
| **C02** | A2A + A2S Collaboration | **A2A** cho agent-agent, **MCP** cho agent-service |
| **C03** | Popular AI Agent Patterns | Reflection, Hierarchical, Coordinating, HITL, Context-aware, Self-learning, RAG |
| **C04** | Context & Memory Management | Interaction history + personalization (LangChain memory) |
| **C05** | No-Code/Low-Code Builder | Access self-built + 3rd party models |
| **C06** | Standardized Security Controls | DLP, Data Poisoning Prevention, Guardrails, prompt validation, logging, audit |

### C.2.2. 7-Layer Architecture (agent_platform.md § 4.2.2)

```
┌──────────────────────────────────────────────────────────┐
│ Output Layer  — custom format + knowledge update         │
├──────────────────────────────────────────────────────────┤
│ Service Layer — multi-channel + recommendations          │
├──────────────────────────────────────────────────────────┤
│ AI Agent Layer — 7 patterns (RAG, Coordinating, HITL,    │
│                  Planning, Context-Aware, Self-Learning) │
├──────────────────────────────────────────────────────────┤
│ Agent Orchestration Layer — Orchestrator break down goal │
├──────────────────────────────────────────────────────────┤
│ Data Storage / Retrieval — Vector + Knowledge Graph + RAG│
├──────────────────────────────────────────────────────────┤
│ Input Layer — Data Lake + Enterprise apps (CRM, ERP) +   │
│               User prompts                               │
├──────────────────────────────────────────────────────────┤
│ Foundation Layer — Security + No-Code Builder + A2A/MCP  │
└──────────────────────────────────────────────────────────┘
```

### C.2.3. Solution comparison (agent_platform.md § 4.4)

5 framework được khảo sát; **shortlist 1 option**:

| Framework | Verdict |
| --- | --- |
| **LangChain/LangGraph/LangFlow/LangFuse** | ✅ **Selected** — full customization, MIT license, open-source, có no-code builder (LangFlow) |
| AWS Bedrock Agents | ◐ Tie-in AWS, low re-usability |
| AgentForce (Salesforce) | ◐ Tốt nhưng heavy Salesforce ecosystem |
| CrewAI | ◐ Strong but lacking observability + UI builder |
| Microsoft AutoGen | ◐ Not production-ready, rapid prototyping focus |

### C.2.4. Security architecture (agent_platform.md § 4.7)

**Data flow 7 checkpoint**:

```
User prompt + file
   ↓
Chat WebApp (Authenticated)
   ↓
[CP1] Trellix DLP Scan (endpoint)
   ↓
AI Application Backend
   ↓
[CP2] GuardRail API (content + denied topics + PII filter + grounding)
   ↓
Routing + Audit + API
   ↓
LLM (OpenAI / Gemini / LLaMa / Bedrock)
   ↓
Response to User
```

**Layer security**:
- Endpoint: Trellix DLP block clipboard/email/cloud/USB/print
- System: AWS Bedrock Guardrails (content filter, denied topics, word filter, PII filter, grounding check, prompt validation)
- User portal: Auth + input validation + sensitive data extraction + logging + encryption + OWASP top 10
- Lang* security: Tool restriction, flow path restriction, memory mgmt, executing constraints, sanitization

## C.3. Control Flow (Blueprint — no implementation detail)

Doc reference đến **Detailed Design Document → Deployment View** cho chi tiết (agent_platform.md § 4.6). Phần này chỉ outline:

- **Orchestrator pattern**: agent orchestrator break down request → sub-agents
- **HITL**: critical decisions → human approval (Maturity Level 1-2)
- **RAG-Based**: agent retrieves từ corporate KB + external sources
- **Coordinating**: agents request clarification từ users

Không có code path / sequence diagram cụ thể.

## C.4. Đánh giá

### Điểm mạnh

1. **Justification chỉn chu** — 5 business pain point map 1-1 với required capability + 2 solution option (AI Agent vs traditional) + recommendation.
2. **Maturity level conservative** — start Level 1-2 (HITL) thay vì full autonomy → build trust before scaling autonomy (agent_platform.md § 4.2.3).
3. **Security-first** — 4 security layer (endpoint Trellix → Guardrail → Portal → Lang*) với compliance EC2 vs Bedrock tradeoff analysis.
4. **Framework comparison có cơ sở** — 5 framework × multi-criteria → defensible shortlist.
5. **NFR có capacity number** — UC1+UC2: 800 users, 1.75M calls/year, 1.17M meetings/year — cụ thể, không hand-wave.

### Điểm yếu

| Risk | Mức độ | Chi tiết |
| --- | --- | --- |
| **Blueprint, no implementation reference** | 🔴 High | Không link đến Detailed Design Document → khó verify feasibility (agent_platform.md § 4.6) |
| **Tech stack table có gap đánh số (item 2, 4, 5, 7 thiếu)** | 🟠 Medium | List "1, 3, 6, 8, 9" — items 2, 4, 5, 7 không có → có item bị remove? (agent_platform.md § 4.5) |
| **Out-of-scope rộng** | 🟠 Medium | Loại bỏ "Data Processing, Model dev, Model training, OPS alignment, High-autonomy agents, Operating model, Steering committee approval" — còn lại scope chưa rõ ai owns |
| **Maturity Level 1-2 không có rubric** | 🟡 Low | "Level 1-2" — không định nghĩa cụ thể level nào là 1 vs 2 |
| **UC3 NFR không nhất quán** | 🟡 Low | "500 transactions/day" (2025) → "600 transactions/**month**" (2026F) — đơn vị nhảy day → month (agent_platform.md § 5.2) |
| **Section 4.7.2 trùng số** | 🟡 Low | "4.7.2. System security" + "4.7.2. Compliance" — cùng số (typo) |

### Đánh giá so với implementation (`agent_platform/`)

| Capability blueprint (§3.2) | Implementation (Agent Studio) | Trạng thái |
| --- | --- | --- |
| C01 Autonomous Decision-Making | ✅ Hub ReAct + Brainstormer StateGraph | ✅ |
| C02 A2A + MCP | ❌ Hub gọi specialist qua HTTP REST, không A2A protocol; ❌ không có MCP server | ❌ |
| C03 Patterns (Reflection, Hierarchical, etc.) | ✅ Hierarchical (Hub-spoke), ✅ HITL (`interrupt()`), ✅ RAG-lite (Vaulter), ❌ Reflection, ❌ Self-learning | ◐ |
| C04 Context & Memory | ✅ Checkpoint + UserPreferences + compaction | ✅ |
| C05 No-Code Builder | ❌ Không có — mọi agent là code Python | ❌ |
| C06 Security Controls | ◐ Có Guardrails (Haiku classification ở PPTX-er) + file validation; ❌ chưa có DLP, ❌ chưa có comprehensive audit | ◐ |
| Tech: LangChain/LangGraph | ✅ Dùng cả 2 | ✅ |
| Tech: LangFlow (no-code) | ❌ | ❌ |
| Tech: LangFuse (tracing) | ◐ Dùng MLflow thay | ◐ |
| LLM: Bedrock | ❌ Dùng Databricks Foundation Model API (Claude qua Databricks) | ❌ |
| Vector DB: Bedrock KB / Mongo Vector | ❌ Không dùng vector — Vaulter dùng Lakebase + truncate | ❌ |

> **Kết luận §C vs §A**: Implementation Agent Studio hiện thực **subset** của blueprint, với 2 lệch lớn: (a) **không** dùng A2A/MCP, (b) **không** dùng Bedrock/Mongo Vector — thay bằng Databricks-native stack (FMAPI + Lakebase).

### Đề xuất cải tiến cho blueprint

**Must**:
- **(M1)** Bổ sung **Detailed Design Document** reference link/path — hiện chỉ "Refer to ... Detailed Design Document" mà không có URL/path (agent_platform.md § 4.6, 5.4, 5.5).
- **(M2)** Fix tech stack table — restore items 2, 4, 5, 7 hoặc renumber.
- **(M3)** Define Maturity Level rubric: Level 1 = ? vs Level 2 = ? (criteria cụ thể).

**Should**:
- **(S1)** Sync blueprint với implementation reality: nếu Databricks-native stack được chọn → cập nhật §4.5 thay vì giữ LangChain/Bedrock pure.
- **(S2)** Add **Section 4.6 Target Architecture diagram** — hiện chỉ có placeholder "Reference Deployment View".
- **(S3)** Fix UC3 NFR unit inconsistency (day vs month).

**Could**:
- **(C1)** Bổ sung **Cost model** — Bedrock Serverless cost projection cho 1.75M calls + 1.17M meetings/year.
- **(C2)** Bổ sung **success metrics** — KPI nào đo Maturity Level đạt đến Level 1, Level 2.

---

# §D. So sánh chéo & Kết luận

## D.1. Capability matrix (Blueprint vs Implementation vs Use case riêng)

| Capability | Blueprint (§C) | Agent Studio (§A) | Analytics Agent (§B) |
| --- | --- | --- | --- |
| **Orchestration pattern** | Generic orchestrator | Hub-spoke ReAct (1 hub + 8 spec) | Supervisor + Parallel Plan-Execute |
| **A2A protocol** | ✅ Đề xuất | ❌ HTTP REST thuần | ◐ Đề xuất migrate sang A2A+MCP |
| **MCP protocol** | ✅ Đề xuất | ❌ | ◐ Đề xuất |
| **LLM hosting** | AWS Bedrock | Databricks FMAPI (Claude) | N/A (Genie + TalkZone) |
| **Framework** | LangChain + LangGraph + LangFlow + LangFuse | LangGraph + LangChain | LangGraph raw nodes |
| **No-code builder** | ✅ LangFlow | ❌ | ❌ |
| **Vector DB / RAG** | Bedrock KB + Mongo Vector | ❌ (truncate-based) | ❌ |
| **HITL** | ✅ Maturity L1-2 | ✅ `interrupt()` cho file upload | ◐ Genie clarification (text-only) |
| **Memory** | ✅ LangChain memory | ✅ Checkpoint + Preferences + Compaction | ◐ Minimal (utilized_agents list) |
| **Guardrails** | ✅ Bedrock Guardrails | ◐ Chỉ PPTX-er có Haiku classification | ❌ Không đề cập |
| **DLP** | ✅ Trellix endpoint + Guardrail PII | ❌ Chưa có | ❌ |
| **Observability** | ✅ LangFuse | ◐ MLflow autolog | ❌ |
| **Self-learning** | ✅ Đề xuất pattern | ❌ | ❌ |
| **Reflection pattern** | ✅ Đề xuất | ❌ | ◐ Confidence score nhưng không reflect |
| **Domain ontology embedded** | ❌ Generic | ❌ Generic workplace task | ✅ Techcombank banking ontology nhúng prompt |
| **Structured sub-task JSON** | ❌ Không đề cập cụ thể | ◐ Per-agent JSON output | ✅ 5 sub-task standardized schema |
| **Calculation in Python (not LLM)** | ❌ | ◐ | ✅ "Never use LLM for calculation" |

## D.2. 3 Pattern phân nhóm (theo `t_system/CLAUDE.md` convention)

### Architectural Patterns

| Pattern | Blueprint | Agent Studio | Analytics |
| --- | --- | --- | --- |
| Hub-and-spoke orchestration | ◐ | ✅ Hub + 8 spec | ❌ |
| Supervisor + parallel executor | ◐ | ❌ | ✅ |
| 7-Layer architecture | ✅ | ◐ partial | ❌ |
| Microservices per agent | ✅ Đề xuất | ✅ Databricks Apps per agent | ❌ |

### Agent Patterns

| Pattern | Blueprint | Agent Studio | Analytics |
| --- | --- | --- | --- |
| ReAct (Reason+Act loop) | ✅ | ✅ Hub | ❌ |
| Plan-and-Execute | ✅ Planning Agent | ◐ Brainstormer 4-step | ✅ Supervisor v2 |
| Reflexion / Self-critique | ✅ Reflective Agent | ❌ | ❌ |
| HITL | ✅ | ✅ `interrupt()` | ◐ Genie clarification |
| RAG-Based | ✅ | ◐ Vaulter (no vector) | ❌ |
| Hierarchical | ✅ | ✅ Hub orchestrator | ✅ Supervisor → Genie |
| Coordinating | ✅ | ✅ Hub | ✅ Supervisor |
| Context-aware | ✅ | ✅ UserPreferences | ✅ Temporal context |
| Self-learning | ✅ | ❌ | ❌ |

### Implementation Patterns

| Pattern | Blueprint | Agent Studio | Analytics |
| --- | --- | --- | --- |
| Structured JSON output (no markdown fence) | ❌ | ✅ Mọi agent | ✅ 5 sub-task schema |
| Multi-strategy JSON parse fallback | ❌ | ✅ `parse_llm_json` 4-strategy | ❌ |
| Message compaction (summarize+trim) | ◐ Context mgmt | ✅ Haiku-based, 10→6 | ❌ |
| Hybrid LLM (Haiku/Sonnet/Opus by task) | ◐ Model selection | ✅ Cost-optimized | ❌ |
| Force output template | ❌ | ❌ | ✅ "1. General Trends / 2. Root Cause / 3. Summary" |
| Calculation in Python (not LLM) | ❌ | ◐ Implicit | ✅ Explicit principle |
| Domain ontology in system prompt | ❌ | ❌ | ✅ Techcombank 5-layer ontology |
| Per-agent MLflow experiment | ❌ | ✅ | ❌ |
| Service Principal token cross-agent | ❌ | ✅ Hub → spec | ❌ |
| Checkpoint-based state | ✅ | ✅ Lakebase | ❌ |
| Volume-based file storage hierarchy | ❌ | ✅ `/{agent}/{user}/{date}/` | ❌ |
| `interrupt()` + `Command(resume)` HITL | ✅ | ✅ Hub file upload | ❌ |

## D.3. Gap chính & Recommendation tổng

### Gap nghiêm trọng (Must)

| Gap | Vị trí | Recommendation |
| --- | --- | --- |
| **Blueprint thiếu Detailed Design Doc link** | §C | Bổ sung path / URL hoặc inline diagram |
| **Implementation lệch khỏi Blueprint ở 2 dimension lớn (LLM provider, A2A/MCP)** | §A vs §C | Quyết định: (a) update Blueprint reflect Databricks-native reality, hoặc (b) plan migration roadmap → Bedrock/A2A |
| **Analytics doc là blueprint, no code reference** | §B | Cần repo / tests / implementation snapshot để verify pattern proposal |
| **Single LLM endpoint trong Agent Studio** | §A | Multi-endpoint failover (Bedrock secondary + Databricks primary, hoặc Anthropic API mirror) |
| **No retry across LLM calls (chỉ Translator)** | §A | Wrap toàn bộ `llm.invoke` bằng retry decorator |

### Gap nên fix (Should)

| Gap | Vị trí | Recommendation |
| --- | --- | --- |
| **Vector search missing (Vaulter)** | §A | Migrate sang Databricks Vector Search |
| **No A2A standardization** | §A + §B | Adopt MCP server pattern theo Anthropic spec; agent thành MCP server endpoint |
| **Confidence score không có method** | §B | Bayesian / sample variance trên Genie results |
| **DLP + Guardrails comprehensive** | §A vs §C | Implement Bedrock Guardrails / equivalent ở mọi agent (hiện chỉ PPTX-er) |
| **No streaming Hub** | §A | SSE pattern từ Translator → replicate cho Hub |

### Gap có thể defer (Could)

| Gap | Vị trí | Recommendation |
| --- | --- | --- |
| **Reflection pattern** | §C blueprint mention nhưng §A không có | Implement self-critique cho high-stakes output (Resume Evaluator, BA assistant) |
| **Self-learning** | §C blueprint | Out-of-scope cho exploration phase — defer |
| **No-code builder (LangFlow)** | §C nhưng §A code-only | Defer trừ khi business demand non-tech agent authoring |
| **Distributed tracing cross-agent** | §A | MLflow trace ID propagate via header |

## D.4. Kết luận

| Folder | Bản chất | Maturity | Khuyến nghị tiếp theo |
| --- | --- | --- | --- |
| **`bankwide_agentic_platform/`** | Strategic blueprint endorsement | Vision / Pre-implementation | Đồng bộ với reality của Agent Studio; bổ sung Detailed Design Document |
| **`agent_platform/`** | Production implementation 9 agents | Working v1 với 7 risk medium+ | Fix Must items M1-M3; plan migration cho A2A/MCP nếu blueprint giữ định hướng đó |
| **`agentic/`** | Knowledge notes + prompt design cho banking BI | Prompt-design + proposal | Implement repo + tests; pin prompt version; A2A+MCP migration với schema cụ thể |

**Insight cốt lõi cross-folder**:

1. **Blueprint (§C) idealizes** AWS Bedrock + A2A/MCP + LangFlow stack; **Implementation (§A) pragmatically** dùng Databricks-native (FMAPI + Lakebase) bỏ qua A2A. Cần align hoặc justify divergence.
2. **§B (Analytics)** có **triết lý design** mature nhất ("never use LLM for reasoning/calculation, only for rewording") nhưng **thiếu implementation evidence** — đây là pattern đáng adopt rộng cho cả §A.
3. **3 folder cùng đề cập HITL + Memory + Hierarchical orchestration** → consensus mạnh; nên đầu tư shared abstraction tier cho 3 capability này.
4. **Vector RAG là gap nhất quán** — blueprint đề xuất, implementation 1 (§A) bypass bằng truncate, implementation 2 (§B) không dùng. Quyết định: hoặc chấp nhận no-vector design (Databricks Vector Search có thể bypass), hoặc systematic adopt.

---

## Change log

- **25/05/2026** — Initial analysis covering 3 folder trong `t_system/`; 4-part structure (Tổng quan / Core Components / Control Flow / Đánh giá) × 3 folder + cross-cut so sánh.
