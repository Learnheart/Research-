# Tài studio architecture

---

## Part 1 — Overall System Analysis

### High-Level Architecture

#### System Purpose

TÀI Studio is a super app platform that bundles 10 specialized AI agents on Databricks Apps. Each agent solves a specific workplace task — translation, summarization, presentation generation, knowledge management, brainstorming, AI vision analysis, visual design, and resume evaluation. A unified orchestrator agent (TÀI) can invoke any specialist via LangGraph tool calling.

#### How Agents Interact

**Pattern 1 — Standalone (Direct Access)**

Each specialist agent operates independently as a standalone Databricks App (FastAPI + React). Users access them via iframe in the hub or directly via their own URL. No inter-agent communication occurs.

**Pattern 2 — Orchestrator (TÀI Super Agent)**

The TÀI agent acts as a centralized orchestrator using a LangGraph ReAct pattern with 8 tools. Each tool wraps an HTTP call to a specialist agent's API. The LLM decides which agent to invoke based on conversation context. Auth uses the app's service principal token.

```
User → TÀI Chat → LangGraph ReAct Agent
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
  translate_doc    summarize_docs    generate_pptx   ...
        │                 │                 │
        ▼                 ▼                 ▼
  Translator API   Summarizer API     PPTX-er API
```

**Pattern 3 — Human-in-the-Loop (Interrupt/Resume)**

When TÀI needs a file from the user, it uses LangGraph's `interrupt()` to pause the graph, renders a file upload UI in the frontend, then resumes after receiving the file path.

#### Control Flow Model

- Centralized orchestration via TÀI (LangGraph ReAct loop)
- Request-response for standalone agent access (REST API)
- SSE streaming for long-running tasks (Translator)
- Checkpoint-based state for multi-turn conversations (Brainstormer, TÀI)

#### Data Flow

- **File uploads:** Frontend → base64 POST → Volume storage → volume_path returned
- **LLM calls:** Agent backend → ChatDatabricks (FMAPI) → Claude Sonnet/Haiku/Opus
- **File outputs:** Agent generates file → saves to Volume → returns file_id → frontend downloads via GET
- **State persistence:** LangGraph checkpoints → Lakebase (PostgreSQL-compatible)
- **User preferences:** Extracted by LLM → stored in Lakebase → loaded on next session

---

### Technology Stack

| Layer | Technology | Details |
| --- | --- | --- |
| Languages | Python 3.10+ (backend), TypeScript (frontend) | |
| Backend Framework | FastAPI | Each agent is a standalone FastAPI app |
| Frontend Framework | React + Vite + Tailwind CSS | SPA served by FastAPI static files |
| AI/LLM | Claude Opus 4.5, Sonnet 4.5, Haiku 4.5 | Via Databricks Foundation Model API (ChatDatabricks) |
| Agent Framework | LangGraph + LangChain | StateGraph (Brainstormer), ReAct with ToolNode (TÀI) |
| File Storage | Unity Catalog Volumes | Hierarchical: `/{agent}/{user}/{date}/{uuid}.{ext}` |
| State Storage | Lakebase (PostgreSQL) | Chat checkpoints, session index, user preferences, vault graph |
| Document Parsing | `ai_parse_document` (Databricks SQL) | For PDF/DOCX; PPTX parsed locally via python-pptx |
| Tracing | MLflow | `mlflow.langchain.autolog()`, one experiment per agent |
| Deployment | Databricks Apps | `deploy.sh` → workspace upload → `databricks apps deploy` |
| Auth | Service Principal (SP) | SP token for inter-agent API calls; X-Forwarded-Email for user identity |
| Image Generation | Pillow (PIL) | Canvas Designer renders programmatic art |
| Document Generation | python-pptx, python-docx, pdf2docx | PPTX/DOCX creation and manipulation |

---

### LLM Model Decision Tree

| Model | Use Case | Cost |
| --- | --- | --- |
| Opus 4.5 | Artistic/creative generation (Canvas Designer) | ~5/25 per M tokens |
| Sonnet 4.5 | Multi-turn chat, creative generation, complex extraction, translation (≥50 chars), summarization | ~3/15 per M tokens |
| Haiku 4.5 | Short content translation (<50 chars), Q&A over context, preference extraction, message compaction | ~1/5 per M tokens |

---

### Prompting Style

- **Structured JSON output:** All agents instruct the LLM to return raw JSON only (no markdown fences)
- **System + Human message pattern:** SystemMessage with detailed instructions, HumanMessage with content
- **Canvas delimiter:** Brainstormer uses `---CANVAS---` delimiter to separate conversational reply from structured canvas data
- **UI delimiter:** TÀI uses `---UI---` delimiter to embed frontend component instructions

---

### Deployment & Runtime

#### Environment

- **Cloud:** Databricks Apps on AWS (`*.aws.databricksapps.com`)
- **Runtime:** uvicorn serving FastAPI on port 8000
- **Frontend:** Pre-built Vite SPA served as static files by FastAPI

#### Deploy Process (`deploy.sh`)

1. Create clean temp dir `/tmp/deploy-<name>`
2. Copy agent backend
3. Merge shared backend modules (no-clobber — agent-specific files preserved)
4. Build frontend (`npm run build`) if present
5. Copy `app.yaml` and `requirements.txt`
6. Upload to Databricks workspace
7. `databricks apps deploy <app-name>`

#### Config / Secrets

- **Environment variables** in `app.yaml` per agent (LLM endpoints, Volume paths, Lakebase config, agent URLs)
- **No `.env` files** — all config via Databricks Apps env vars
- **Service Principal auth** — WorkspaceClient auto-authenticates; SP token used for inter-agent calls
- **CORS** — each agent allows the hub origin + localhost for dev

---

## Part 3 — Cross-Cutting Analysis

### Agent Interaction Matrix

| Caller → | Translator | Summarizer | PPTX-er | Vaulter | Brainstormer | Visionary | Canvas | Resume |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **TÀI** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✗ | ✗ |
| **Hub (iframe)** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Visionary** | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ (proxies to KA endpoint) | ✗ | ✗ |

- TÀI does not yet have tools for Canvas Designer or Resume Evaluator (newly added agents)
- No agent calls another agent directly — only TÀI orchestrates
- The Hub embeds all agents via iframe (no API calls, just URL embedding)

---

### Shared Abstractions

| Module | Purpose | Used By |
| --- | --- | --- |
| `llm.py` | Cached ChatDatabricks client (Opus/Sonnet/Haiku) + lazy MLflow init | All agents |
| `llm_utils.py` | Strip markdown fences, parse JSON from LLM output | Translator, Summarizer, PPTX-er, Vaulter, Visionary, Canvas, Resume |
| `file_store.py` | Volume-based file save/load with `/{agent}/{user}/{date}/` hierarchy | Translator, Summarizer, PPTX-er, Vaulter, Visionary, Canvas, Resume |
| `file_parser.py` | Document parsing via `ai_parse_document` (SQL) or local python-pptx | Translator, Summarizer, Vaulter, Resume |
| `file_validator.py` | Upload validation: extension, size (≤20MB), magic bytes | All file-accepting agents |
| `user_context.py` | Extract user email from X-Forwarded-Email header | All agents |
| `user_preferences.py` | Lakebase-backed cross-session preferences (language, tone, topics) | Brainstormer, TÀI |
| `compact_messages.py` | Summarize-then-trim message compaction to prevent context overflow | Brainstormer, TÀI, Visionary |
| `chat_store.py` | Lakebase-backed conversation storage with compaction on load | Available but not directly used (agents use LangGraph checkpoints) |
| `upload_router.py` | Shared FastAPI router for `/api/files/upload` and `/api/files/resolve` | All file-accepting agents |
| `schemas.py` | Shared Pydantic models (TranslateRequest/Response, SummarizeRequest/Response, etc.) | Translator, Summarizer, Vaulter, PPTX-er |

---

### Bottlenecks & Risks

| Risk | Severity | Details |
| --- | --- | --- |
| Single LLM endpoint | **High** | All agents share `databricks-claude-sonnet-4-5`. If the endpoint is throttled or down, all agents fail simultaneously |
| No retry on LLM calls | Medium | Most agents do not retry failed LLM calls (except Translator which retries 2x for structured translation). A transient LLM error fails the entire request |
| Synchronous file parsing | Medium | `ai_parse_document` uses a SQL Warehouse connection. Long-running queries block the FastAPI thread (mitigated by `asyncio.to_thread` in some agents) |
| No rate limiting | Medium | No request rate limiting on any agent API. A single user could exhaust LLM quota |
| Shared Lakebase schema | Medium | All agents use `tai_studio` schema in Lakebase. Schema migrations are manual (`init_db()` with `CREATE TABLE IF NOT EXISTS`) |
| Canvas Designer pixel-level rendering | Low | `generate_artistic_image` iterates pixel-by-pixel for gradients/glows. Large images (>2000px) will be slow |
| Translator parallel workers | Low | `_MAX_PARALLEL = 5` concurrent LLM calls per translation. Large PPTX files with many slides may still be slow |

---

### Security Concerns

| Concern | Details |
| --- | --- |
| SQL injection in `file_parser.py` | Volume path is interpolated into SQL string. Mitigated by strict regex validation (`[A-Za-z0-9/_.-@]+`) but not parameterized |
| No authentication on standalone agents | Agents rely on Databricks Apps proxy for auth (X-Forwarded-Email). If accessed directly (bypassing proxy), no auth is enforced |
| Service Principal token exposure | TÀI's `_token()` function extracts SP token from WorkspaceClient. Token is passed in HTTP headers to agent APIs — standard pattern but sensitive |
| CORS allows localhost | All agents allow `http://localhost:5173` in CORS origins. Should be removed in production |
| No input sanitization on LLM prompts | User messages are passed directly to LLM system prompts. Prompt injection is possible (mitigated by LLM's own guardrails) |
| File upload size limit only | 20MB limit enforced, but no virus scanning or content inspection beyond magic bytes |

---

### Performance Considerations

| Area | Current State | Impact |
| --- | --- | --- |
| LLM caching | ChatDatabricks instances cached by endpoint name | Good — avoids re-initialization |
| MLflow init | Lazy (on first `get_llm()` call) | Good — prevents startup crashes |
| DB connections | `file_parser.py` caches SQL connection with reconnect logic | Good — avoids connection churn |
| Message compaction | Summarize-then-trim at 10 messages, keep last 6 | Good — prevents context overflow |
| Translator parallelism | ThreadPoolExecutor with max 5 workers | Good — balances throughput vs. LLM quota |
| Canvas pixel rendering | Per-pixel iteration in Python | Poor — should use numpy/vectorized operations for large images |
| No response caching | Every request hits the LLM | Could cache common translations/summaries |
| Synchronous LLM in some paths | Some agents use `asyncio.to_thread(llm.invoke)`, others call synchronously | Inconsistent — should standardize async pattern |

---

## Part 4 — Improvement Suggestions

### Architectural Improvements

- **Add a shared API gateway** — centralize auth, rate limiting, and logging instead of duplicating CORS/auth in each agent
- **Event-driven architecture** — for long-running tasks (translation, batch evaluation), use a job queue instead of synchronous HTTP
- **Health check aggregation** — the hub should check all agent health endpoints and display status

### Refactoring Opportunities

- **Standardize async pattern** — all LLM calls should use `asyncio.to_thread()` consistently
- **Extract common FastAPI boilerplate** — CORS, static file serving, health endpoint are copy-pasted across all agents. Create a shared `create_app()` factory
- **Unify PPTX generation** — both PPTX-er and Visionary have independent PPTX builders. Extract a shared PPTX rendering module
- **Canvas Designer optimization** — replace pixel-by-pixel rendering with numpy array operations or use Cairo/Skia for GPU-accelerated rendering

### Scalability Enhancements

- **Add TÀI tools for new agents** — Canvas Designer and Resume Evaluator are not yet accessible via TÀI
- **Implement request queuing** — for batch operations (Resume Evaluator batch, large translations), use background workers
- **LLM endpoint failover** — configure fallback endpoints if primary is throttled
- **Horizontal scaling** — each agent is independently deployable, but currently single-instance. Add load balancing for high-traffic agents

### AI/Agent Design Improvements

- **Streaming responses** — TÀI and Brainstormer should stream LLM responses token-by-token for better UX
- **RAG for Vaulter** — current implementation loads all documents as context. Should use vector embeddings + similarity search for scalable retrieval
- **Structured output** — use LangChain's `with_structured_output()` instead of JSON-in-prompt for more reliable parsing
- **Tool result caching** — TÀI could cache tool results within a session to avoid redundant agent calls
- **Multi-modal TÀI** — extend TÀI to handle image inputs directly (for Canvas Designer and Visionary integration)
