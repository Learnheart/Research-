# List of agents

> Hub super agent

## Table of Contents

- [Agent: Hub (Super Agent)](#agent-tài-super-agent)
- [Translator Agent: The Translator](#translator-agent-the-translator)
- [Summarizer Agent: The Summarizer](#summarizer-agent-the-summarizer)
- [Powerpointer: The Powerpoint-er](#powerpointer-the-powerpoint-er)
- [Vaulter: The Vaulter](#vaulter-the-vaulter)
- [Brainstormer: The Brainstormer](#brainstormer-the-brainstormer)
- [Visionary: The AI Visionary](#visionary-the-ai-visionary)
- [Canvas Designer: The Canvas Designer](#canvas-designer-the-canvas-designer)
- [Resume Evaluator: The Resume Evaluator](#resume-evaluator-the-resume-evaluator)

---

## Agent: Hub (Super Agent)

### Agent Overview

| Field | Value |
| --- | --- |
| **Name** | Hub |
| **ID** | `hub` |
| **Databricks App** | `agent-hub` |
| **Primary Responsibility** | Unified AI assistant — orchestrates all specialist agents via LangGraph ReAct tool calling |
| **Business Purpose** | Single chat interface where users can accomplish any task without knowing which agent to use |
| **When Invoked** | User sends a message in Hub Chat (hub frontend component) |

### Input & Output

**Chat Input**

```json
{"message": "Translate this file to English", "session_id": "uuid-or-null"}
```

**Chat Output**

```json
{"reply": "Translation complete...", "session_id": "uuid", "status": "ok", "interrupt": null, "ui_actions": [{"type": "file_download", "props": {...}}]}
```

**Interrupt Output (when file needed)**

```json
{"reply": "I need you to upload a file...", "session_id": "uuid", "status": "interrupted", "interrupt": {"type": "file_upload", "purpose": "to translate to English", "accepted_types": ["pdf","docx"]}}
```

### 3. Core Logic

#### User Flow

1. User switches to "Hub" tab in the hub (or accesses Hub Chat directly)
2. User types a message (e.g., "Translate my report to English")
3. **If file needed:** Hub responds "I need you to upload a file" → file upload UI appears → user uploads file → Hub resumes automatically
4. **If no file needed** (e.g., "Create a presentation about AI"): Hub directly calls the appropriate agent tool
5. **Tool execution:** Hub calls specialist agent API (e.g., Translator, PPTX-er) behind the scenes
6. **Result:** Hub presents the result with UI actions (download button, brainstorm canvas, etc.)
7. User can continue the conversation — Hub remembers context and can chain multiple tools
8. **Example multi-step:** "Translate this file" → uploads → "Now summarize it" → Hub calls Summarizer with the same file

#### LangGraph ReAct Graph

```
load_preferences → agent ⇄ tools → update_preferences → END
```

**Agent Node**

- Load system prompt with user preferences
- Compact messages (summarize older messages via Haiku)
- Invoke Sonnet with `bind_tools(ALL_TOOLS)`
- If response has `tool_calls` → route to tools node
- If no `tool_calls` → route to `update_preferences` → END

**Tool Execution**

- `ToolNode(ALL_TOOLS)` executes the called tool
- After tool execution, control returns to agent node for next decision
- This creates a ReAct loop: agent → tool → agent → tool → ... → agent → done

#### Human-in-the-Loop (File Upload)

1. Agent calls `request_file_upload` tool
2. Tool calls `interrupt()` — pauses the graph
3. Frontend detects `status: "interrupted"` → shows file upload UI
4. User uploads file → frontend calls `POST /api/hub/resume` with `volume_path`
5. Graph resumes with the `volume_path` as the tool result
6. Agent continues with the file path (e.g., calls `translate_document`)

#### Download Proxy

Hub provides `GET /api/download/{agent}/{file_id}` to proxy file downloads from agent apps using SP auth (browsers don't have SP tokens).

### 4. Internal APIs / Functions

#### Tools (`tools.py`)

| Tool | Target Agent | Purpose |
| --- | --- | --- |
| `request_file_upload` (interrupt) | — | Pause graph, request file from user |
| `translate_document` | Translator | Translate uploaded file (SSE) |
| `summarize_documents` | Summarizer | Summarize uploaded files |
| `generate_presentation` | PPTX-er | Generate branded PPTX |
| `upload_to_vault` | Vaulter | Add document to knowledge vault |
| `query_vault` | Vaulter | Query knowledge vault |
| `brainstorm` | Brainstormer | Structured brainstorming |
| `ask_visionary` | Visionary | AI vision Q&A |

### Dependencies

| Type | Details |
| --- | --- |
| **Other Agents** | Translator, Summarizer, PPTX-er, Vaulter, Brainstormer, Visionary (via HTTP) |
| **Shared Modules** | `llm.py`, `user_context.py`, `user_preferences.py`, `compact_messages.py` |
| **External Libraries** | langgraph, langchain-core, databricks-langchain (AsyncCheckpointSaver), httpx |

### Error Handling & Edge Cases

| Scenario | Handling |
| --- | --- |
| Empty message | HTTP 400 |
| Agent URL not configured | `ValueError` raised by `_agent_url()` |
| Agent API returns error | httpx raises; caught by graph, returned as error |
| LLM fails | HTTP 500 with error detail |
| Interrupt detection | Checks `state.next` and `task.interrupts` after invoke |

### Known Limitations

- No tools for Canvas Designer or Resume Evaluator (newly added agents)
- Higher latency due to multi-hop: Hub → Agent API → LLM
- No streaming — entire response returned at once
- Session state grows unbounded (mitigated by message compaction)

### Configuration

| Variable | Purpose |
| --- | --- |
| `DATABRICKS_LLM_ENDPOINT` | Sonnet (for agent reasoning) |
| `LAKEBASE_AUTOSCALING_PROJECT` | Lakebase for checkpoints + preferences |
| `LAKEBASE_AUTOSCALING_BRANCH` | production |
| `TRANSLATOR_URL` / `SUMMARIZER_URL` / `PPTX_URL` / `VAULTER_URL` / `BRAINSTORMER_URL` / `VISIONARY_URL` | Agent base URLs |
| `MLFLOW_EXPERIMENT_ID` | 1811265038828691 |

### Observability

| Type | Details |
| --- | --- |
| **Logs** | `hub` and `hub.tools` loggers |
| **Traces** | MLflow autolog (experiment: 1811265038828691) |

---

## Translator Agent: The Translator

### Agent Overview

| Field | Value |
| --- | --- |
| **Name** | The Translator |
| **ID** | `the-translator` |
| **Databricks App** | `agent-the-translator` |
| **Primary Responsibility** | Translate documents between languages while preserving original formatting |
| **Business Purpose** | Enable employees to translate financial reports, presentations, and documents without losing layout |
| **When Invoked** | User uploads a file and selects target language; or Hub calls `translate_document` tool |

### Input & Output

**Input**

```json
{"volume_path": "/Volumes/.../file.pdf", "target_language": "Vietnamese", "source_language": "auto"}
```

**Output (SSE stream)**

- `event: plan` → `{"segments": N, "chunks": M, "estimated_seconds": S}`
- `event: progress` → `{"chunk": i, "total": M}` (per completed chunk)
- `event: result` → `TranslateResponse` (translated_text, file_id, detected_language, etc.)
- `event: error` → `{"detail": "error message"}`

**Side Effects**

- Translated file saved to Unity Catalog Volume
- File download available via `GET /api/translate/download/{file_id}`

### Core Logic

#### User Flow

1. User opens The Translator page
2. User drops/selects a file (PDF, DOCX, PPTX, or TXT) in the left panel
3. User selects target language from dropdown (default: Vietnamese). Source language is auto-detected.
4. User clicks "Translate to {language}" button
5. **Upload phase:** File is base64-encoded and uploaded to Volume via `/api/files/upload`
6. **Analyzing phase:** Frontend sends `POST /api/translate` with the `volume_path`; backend reads and validates the file
7. **Translating phase:** SSE stream delivers real-time progress (chunk X of Y). UI shows progress bar + estimated time remaining.
8. **Building phase:** Backend reassembles the translated file preserving original formatting
9. **Result:** For structured files (PPTX/DOCX/PDF) → Download button appears. For TXT → translated text displayed inline with Copy button.
10. User clicks Download to get the translated file

#### Flow by File Type

**PPTX:**

1. Extract segments at run-level (shapes, tables, groups) → `_extract_pptx_segments()`
2. Group by slide → `_group_pptx_by_slide()`
3. Parallel translation (ThreadPool, max 5 workers) — Haiku for <50 chars, Sonnet otherwise
4. Apply translations back preserving whitespace/formatting → `_apply_pptx_translations()`
5. Save rebuilt PPTX to Volume

**DOCX:**

1. Extract segments at run-level (paragraphs, tables, headers/footers) → `_extract_docx_segments()`
2. Group into sections of 50 segments → `_group_docx_sections()`
3. Parallel translation (same strategy as PPTX)
4. Apply translations back → `_apply_docx_translations()`
5. Save rebuilt DOCX to Volume

**PDF:**

1. Convert PDF → DOCX via pdf2docx
2. Extract unique text slots (deduplicated)
3. Join with `⟪SEP⟫` delimiter, chunk at 8K chars
4. Sequential LLM calls (flat prompt preserving delimiters)
5. Split translated text back, map to document paragraphs
6. Save as DOCX (not PDF) to Volume

**TXT:**

1. Parse file content, truncate at 12,000 chars
2. Single LLM call with flat system prompt
3. Save translated text as `.txt` to Volume

#### LLM Strategy (Hybrid)

- **Haiku** (`fast=True`): Segments with <50 total chars (titles, labels, short table cells)
- **Sonnet** (`fast=False`): Everything else

### Internal APIs / Functions

| Function | Purpose | Key Parameters |
| --- | --- | --- |
| `translate()` | Main endpoint — validates, downloads file, routes by extension | `TranslateRequest` body |
| `_translate_streaming()` | Generator yielding SSE events | `ext`, `content`, `filename`, `target_language` |
| `_translate_chunk()` | Translate one segment group via LLM | chunk dict, `target_language`, `retries=2`, `fast` |
| `_extract_pptx_segments()` | Extract text at run-level from PPTX | content bytes |
| `_apply_pptx_translations()` | Write translations back to PPTX preserving format | `Presentation`, translations dict |
| `_extract_docx_segments()` | Extract text at run-level from DOCX | content bytes |
| `_apply_docx_translations()` | Write translations back to DOCX | `Document`, translations dict |
| `_is_simple()` | Check if segment group is simple enough for Haiku | group dict |
| `_group_pptx_by_slide()` | Group segments by slide index | segments, num_slides |
| `_group_docx_sections()` | Group segments into batches of 50 | segments, section_size |
| `download_translation()` | Serve translated file for download | file_id, ext |

### Dependencies

| Type | Details |
| --- | --- |
| **Other Agents** | None (standalone) |
| **Shared Modules** | `llm.py`, `llm_utils.py`, `file_store.py`, `file_parser.py`, `file_validator.py`, `user_context.py`, `upload_router.py`, `schemas.py` |
| **External Libraries** | python-pptx, python-docx, pdf2docx, langchain-core |

### Error Handling & Edge Cases

| Scenario | Handling |
| --- | --- |
| LLM returns invalid JSON | Retry up to 2 times; raise HTTP 502 after exhaustion |
| Empty file | Returns SSE error event: "File appears to be empty" |
| Unsupported file type | HTTP 400 via `validate_upload()` |
| File too large (>20MB) | HTTP 400 via `validate_upload()` |
| Magic bytes mismatch | HTTP 400 via `validate_upload()` |
| Request timeout (5 min) | Frontend aborts; server continues processing |
| PDF with no extractable text | Returns SSE error: "PDF appears to be empty" |

### Known Limitations

- PDF output is DOCX (not PDF) — pdf2docx conversion is one-way
- TXT files truncated at 12,000 chars
- No domain-specific terminology handling
- Parallel workers limited to 5 (large PPTX with 50+ slides still sequential in batches)

### Configuration

**Environment Variables**

| Variable | Purpose |
| --- | --- |
| `DATABRICKS_LLM_ENDPOINT` | Sonnet endpoint |
| `DATABRICKS_LLM_ENDPOINT_FAST` | Haiku endpoint (optional, defaults to `databricks-claude-haiku-4-5`) |
| `DATABRICKS_HTTP_PATH` | SQL Warehouse for `ai_parse_document` |
| `VOLUME_PATH` | File storage volume |
| `MLFLOW_EXPERIMENT_ID` | 1811265038828270 |

**Hard-coded Constants**

| Constant | Value | Purpose |
| --- | --- | --- |
| `_MAX_FLAT_CHARS` | 12,000 | Max chars for TXT translation |
| `_MAX_PARALLEL` | 5 | Concurrent LLM calls |
| `_HAIKU_CHAR_THRESHOLD` | 50 | Below this → use Haiku |
| `_SECS_PER_GROUP` | 12 | Estimated seconds per translation chunk |

### Observability

| Type | Details |
| --- | --- |
| **Logs** | `agent_studio.translate` logger — logs file size, segment counts, parse results |
| **Metrics** | None |
| **Traces** | MLflow autolog via `mlflow.langchain.autolog()` (experiment ID: 1811265038828270) |

---

## Summarizer Agent: The Summarizer

### Agent Overview

| Field | Value |
| --- | --- |
| **Name** | The Summarizer |
| **ID** | `the-summarizer` |
| **Databricks App** | `agent-the-summarizer` |
| **Primary Responsibility** | Summarize uploaded documents into executive summaries with key insights |
| **Business Purpose** | Help employees quickly digest long reports, meeting transcripts, and documents |
| **When Invoked** | User uploads one or more files; or Hub calls `summarize_documents` tool |

### Input & Output

**Input**

```json
{"volume_paths": ["/Volumes/.../file1.pdf", "/Volumes/.../file2.docx"]}
```

**Output**

```json
{"summary": "Executive summary text...", "key_points": ["Point 1", "Point 2"], "filenames": ["file1.pdf"], "truncated": false}
```

**Side Effects**

- None (no file generation — returns text only)

### Core Logic

#### User Flow

1. User opens The Summarizer page
2. User drops/selects one or more files (PDF, DOCX, PPTX, TXT) into the upload zone
3. User clicks "Summarize" button
4. Files are uploaded to Volume (base64 → `/api/files/upload`)
5. Frontend sends `POST /api/summarize` with all `volume_paths`
6. **Loading state:** Spinner shown while backend parses files and calls LLM
7. **Result displayed:** Executive summary paragraph + bullet-point key insights appear in the right panel
8. User can copy the summary or upload additional files for a combined summary

#### Technical Flow

1. Validate all volume paths
2. For each file: download from Volume → validate (ext, size, magic) → parse via `parse_file()`
3. Concatenate all parsed text with `--- filename ---` separators
4. Truncate combined text at 100,000 chars if needed
5. Send to Sonnet with structured system prompt requesting JSON output
6. Parse LLM response → extract `summary` and `key_points`
7. Return `SummarizeResponse`

#### System Prompt Strategy

- **Role:** "senior analyst who distills complex documents into clear, actionable intelligence"
- **Output:** executive summary (3-5 sentences) + key insights (5-8 bullet points)
- **Guidelines:** prioritize decisions, metrics, deadlines, risks; synthesize multiple documents; no filler words

### Internal APIs / Functions

| Function | Purpose |
| --- | --- |
| `summarize()` | Main endpoint — parses files, calls LLM, returns structured summary |

### Dependencies

| Type | Details |
| --- | --- |
| **Other Agents** | None (standalone) |
| **Shared Modules** | `llm.py`, `llm_utils.py`, `file_parser.py`, `file_validator.py`, `file_store.py`, `user_context.py`, `upload_router.py`, `schemas.py` |
| **External Libraries** | langchain-core |

### Error Handling & Edge Cases

| Scenario | Handling |
| --- | --- |
| No files provided | HTTP 400 |
| All files empty | HTTP 400: "All uploaded files appear to be empty" |
| LLM returns invalid JSON | Falls back to raw text as summary, empty `key_points` |
| File parse error | HTTP 400 with error detail |
| Content exceeds 100K chars | Truncated; `truncated=true` in response |

### Configuration

| Variable | Purpose |
| --- | --- |
| `DATABRICKS_LLM_ENDPOINT` | Sonnet endpoint |
| `DATABRICKS_HTTP_PATH` | SQL Warehouse for document parsing |
| `VOLUME_PATH` | File storage volume |
| `MLFLOW_EXPERIMENT_ID` | 1811265038828695 |

**Constants**

| Constant | Value |
| --- | --- |
| `_MAX_CHARS` | 100,000 |
| `_ALLOWED` | pdf, docx, pptx, txt |

### Observability

| Type | Details |
| --- | --- |
| **Logs** | `agent_studio.summarize` — logs file count, parse sizes, total chars |
| **Traces** | MLflow autolog (experiment: 1811265038828695) |

---

## Powerpointer: The Powerpoint-er

### Agent Overview

> *Last updated: 2026-05-12. Source of truth for architecture, APIs, and design decisions.*

| Field | Value |
| --- | --- |
| **Name** | The Powerpoint-er |
| **ID** | `the-powerpoint-er` |
| **Databricks App** | `agent-the-powerpoint-er` |
| **Primary Responsibility** | Generate branded PPTX presentations from text descriptions |
| **Business Purpose** | Enable employees to create professional slide decks instantly without design skills |
| **When Invoked** | User provides topic / audience / slide count via web UI |

### Input & Output

**Input** (`POST /api/v1/presentation/outline` or `/generate`):

```json
{
  "topic": "AI Strategy 2025",
  "audience": "Executive Board",
  "num_slides": 10,
  "use_template": true
}
```

**Output** (from `/generate` or after `/build`):

```json
{
  "slides": [
    {
      "type": "cover",
      "title": "AI Strategy 2025",
      "bullets": ["For Executive Board"],
      "variant": null,
      "sources": []
    },
    {
      "type": "content",
      "title": "Market Landscape",
      "bullets": ["..."],
      "variant": "data",
      "html": "<html>...</html>"
    }
  ]
}
```

**Side Effects:** PPTX assembled and downloaded client-side via `pptxgenjs` (no server-side file storage in current version).

### User Flow (3-Step UI)

```
BRIEF → OUTLINE → PREVIEW
```

**Step 1 — Brief:**

- User enters topic, audience, optional context, number of slides (5–15)
- Toggle `use_template` (corporate-branded frame vs. free-form HTML)

**Step 2 — Outline:**

- Calls `/outline` → LLM generates JSON slide skeleton
- User sees editable outline (titles, bullets, slide type)
- Can preview individual slides on demand (each calls `/regenerate-slide`)
- Can add, delete, reorder, or regenerate slides

**Step 3 — Preview:**

- Calls `/build` in parallel for all unrendered slides
- Grid view of all slides as rendered HTML (sandboxed iframes)
- Edit each slide via Rich Editor (inline formatting) or AI Edit bar (natural language instructions)
- Presentation Mode: fullscreen slideshow, arrow/space/Esc navigation
- Export: client-side PNG capture → PPTX assembly via `pptxgenjs`, with clickable hyperlinks preserved

### API Endpoints

All routes: `POST`, prefix `/api/v1/`.

| Endpoint | Request Body | Response | Purpose |
| --- | --- | --- | --- |
| `/presentation/outline` | `PptxGenerateRequest` | `{slides: OutlineSlide[]}` | LLM generates slide skeleton (no HTML) |
| `/presentation/build` | `{slides, topic, use_template, use_web_search}` | `{slides: Slide[]}` | Render all slides to HTML in parallel |
| `/presentation/regenerate-slide` | `{slide, topic, index, use_template, use_web_search}` | `{html: string}` | Re-render a single slide |
| `/presentation/ai-edit-slide` | `{slide, instruction}` | `{html: string}` | Apply natural-language edit to existing slide HTML |
| `/presentation/generate` | `PptxGenerateRequest` | `{slides: Slide[]}` | One-shot: outline + build (no review step) |
| `/api/health` | — | `{status, service, env}` | Health check, returns `APP_ENV` |

### Core Logic

#### Rendering Pipeline

```
/outline or /generate
   └─ LLM(SYSTEM_OUTLINE) → JSON slide array
       └─ parse_llm_json() → OutlineSlide[]

/build or /regenerate-slide  (per slide, via asyncio.gather for /build)
   ├─ type == cover    → cover_html()    [Python, no LLM]
   ├─ type == section  → section_html()  [Python, no LLM]
   ├─ type == summary  → summary_html()  [Python, no LLM]
   └─ type == content
        ├─ use_template=True  → LLM(SYSTEM_HTML_BRAND_FRAGMENT_{VARIANT})
        │                       → inner HTML fragment (1213×580 px)
        │                       → content_frame() wraps with title/logo/footer
        └─ use_template=False → LLM(SYSTEM_HTML_FREE) → full self-contained HTML (1333×750 px)
              └─ on LLM failure → fallback_html()

/ai-edit-slide
   └─ strip_data_urls(html)  [saves ~70K tokens]
       └─ LLM(SYSTEM_AI_EDIT) + stripped HTML
           └─ parse_llm_json() → new HTML
               └─ restore_data_urls(new_html)
```

#### LLM Model Selection

| Model | When Used |
| --- | --- |
| **Sonnet** (default) | All outline and content rendering |
| **Haiku** (`fast=True`) | Guardrails classification (input/output safety) |
| **Opus** (`opus=True`) | Available, not used in default flow |
| **GPT-5.5** (`gpt=True`) | Only when `use_web_search=True` (currently disabled in UI) |
| **Gemini 3 Flash** (`gemini=True`) | Available, not used in default flow |

> `use_web_search` is hardcoded to `false` in the UI. Backend still supports it if called directly.

#### Guardrails

Every `/outline` and `/generate` call runs:

- `check_input_guardrails()` — Haiku classifies user topic as safe/unsafe before processing
- `check_output_guardrails()` — Haiku classifies LLM output before returning to client

#### JSON Parsing (`parse_llm_json`)

4-strategy fallback:

1. Direct `json.loads()`
2. Brace/bracket matching extraction
3. Fix trailing commas → retry
4. Regex extraction

### Slide Types & Variants

#### Slide Types

| Type | Description | Rendered By |
| --- | --- | --- |
| `cover` | Title page with `bg_cover.png` background | Python (`cover_html`) |
| `section` | Red gradient divider | Python (`section_html`) |
| `summary` | Closing slide with bulleted takeaways | Python (`summary_html`) |
| `content` | Main content slide — must specify variant | LLM + optional frame |

#### Content Variants (content slides only)

| Variant | Purpose | LLM Prompt |
| --- | --- | --- |
| `data` | KPIs, metrics, charts, comparisons | `SYSTEM_HTML_BRAND_FRAGMENT_DATA` |
| `concept` | Principles, frameworks, prose-forward | `SYSTEM_HTML_BRAND_FRAGMENT_CONCEPT` |
| `process` | Timelines, workflows, step sequences | `SYSTEM_HTML_BRAND_FRAGMENT_PROCESS` |

> Default variant is `data` for backward compatibility.

#### Content Frame (`use_template=True`)

When rendering a content slide with `use_template=True`, the LLM produces an inner HTML fragment (1213×580 px), and `content_frame()` wraps it with:

| Element | Position | Detail |
| --- | --- | --- |
| Diamond background | Full slide | `bg_content.png` |
| Title | Top-left | Red (#ED1B24), bold |
| Corporate Logo | Top-right | Fixed asset |
| Citation footer | Bottom-left | Source links from `slide.sources` |
| Page number | Bottom-right | Slide index |

### Frontend — Rich Editor & Export

**RichEditor**

- Inline contenteditable on leaf text nodes only
- Formatting toolbar: Bold / Italic / Underline / Strikethrough
- Font size picker, color palette (10 brand colors)
- Text alignment, list formatting
- Undo / Redo

**EditModal**

- **Tab 1:** Rich Editor (visual)
- **Tab 2:** AI Edit bar (natural language instruction → `/ai-edit-slide`)
- **Regenerate** button → `/regenerate-slide`

**Export (Client-Side)**

- Capture each slide iframe as PNG via `html-to-image`
- Assemble with `pptxgenjs` (13.33" × 7.5", 100 px/in)
- Extract all `<a href>` from slide HTML → overlay invisible hyperlink text boxes
- Download as `.pptx`

### Configuration

#### Environment Variables

| Variable | Purpose |
| --- | --- |
| `DATABRICKS_WORKSPACE_ID` | Injected by platform; used in CORS allowlist |
| `DATABRICKS_LLM_ENDPOINT` | Sonnet serving endpoint (default LLM) |
| `DATABRICKS_LLM_ENDPOINT_FAST` | Haiku endpoint (guardrails) |
| `DATABRICKS_LLM_ENDPOINT_OPUS` | Opus endpoint |
| `DATABRICKS_LLM_ENDPOINT_GPT` | GPT-5.5 endpoint (`web_search`) |
| `DATABRICKS_LLM_ENDPOINT_GEMINI` | Gemini 3 Flash endpoint |
| `DATABRICKS_HOST` | Databricks workspace URL |
| `DATABRICKS_TOKEN` | API token (local dev + GPT/Gemini clients) |
| `VOLUME_PATH` | UC volume path (reserved for file storage) |
| `MLFLOW_EXPERIMENT_ID` | 1811265038828694 |
| `MLFLOW_ARTIFACT_URI` | MLflow artifact volume path |
| `MLFLOW_TRACKING_URI` | `"databricks"` |
| `APP_ENV` | `"local"` / `"dev"` / `"prod"` — controls TLS and auth mode |

#### Design Tokens (Brand)

| Token | Value | Purpose |
| --- | --- | --- |
| Red | `#ED1B24` | Primary brand color |
| Dark Red | `#7A0B0F` | Dark accent |
| Light Pink | `#FBD2D4` | Soft background |
| Coral | `#F3797D` | Secondary highlight |
| Navy | `#1B1464` | Secondary color |
| Sky Tint | `#DAEBEC` | Subtle background |
| Body Text | `#404040` | Default text |
| Caption | `#767171` | Secondary text |
| Card BG | `#F2F2F2` | Card/box background |
| Canvas | 1333 × 750 px | 16:9, 100 px/in → 13.33" × 7.5" PPTX |

### Dependencies

**Backend**

- FastAPI, Uvicorn
- LangChain (`langchain-core`, `databricks-langchain`, `langchain-openai`)
- Databricks SDK (`databricks.sdk`)
- MLflow (`mlflow.langchain.autolog`)
- OpenAI SDK (raw client for GPT/Gemini endpoints)
- Pydantic (schemas)

**Frontend**

- React 18.3 + TypeScript 5.4
- Vite 5.2 + Tailwind CSS 4.0
- `pptxgenjs` 3.12 — client-side PPTX assembly
- `html-to-image` 1.11 — slide PNG capture

### Directory Structure

```
the-powerpoint-er/
├── app.yaml.tpl                  deployment config template
├── AGENT.md                      registry entry (index only)
├── docs/
│   └── agent-overview.md         ← this file
├── backend/
│   ├── main.py                   FastAPI app, CORS, health endpoint
│   ├── pptx/
│   │   ├── pipeline.py           render_slide_html() dispatcher
│   │   ├── tokens.py             brand colors & canvas constants
│   │   ├── utils.py              HTML escaping, base64 data URL helpers
│   │   ├── prompts/
│   │   │   ├── outline.py        SYSTEM_OUTLINE prompt
│   │   │   ├── html.py           SYSTEM_HTML_* prompts (free + 3 variants)
│   │   │   └── ai_edit.py        SYSTEM_AI_EDIT prompt
│   │   └── renderers/
│   │       ├── cover.py          cover_html()
│   │       ├── section.py        section_html()
│   │       ├── summary.py        summary_html()
│   │       ├── content_frame.py  content_frame() wrapper
│   │       └── fallback.py       fallback_html()
│   ├── routers/
│   │   └── pptx.py               5 POST endpoints
│   ├── services -> ../../../shared/backend/services
│   └── models   -> ../../../shared/backend/models
└── frontend/
    └── src/
        ├── PptxPage.tsx          main UI (Brief → Outline → Preview)
        ├── api/
        │   ├── client.ts         5 API methods with typed bodies + timeouts
        │   └── request.ts        fetch wrapper
        └── components/
            ├── GovernanceBanner.tsx
            └── AgentIcons.tsx
```

### Shared Modules (via symlinks)

| Module | Purpose |
| --- | --- |
| `models/schemas.py` | `PptxGenerateRequest`, slide types, response schemas |
| `services/llm.py` | Model selection (`get_llm`), client caching, multi-model support |
| `services/llm_utils.py` | `parse_llm_json()` — 4-strategy JSON parser |
| `services/guardrails.py` | Input/output safety classification via Haiku |
| `services/user_context.py` | User identity from request headers |
| `services/file_store.py` | UC volume file I/O (reserved) |

### Error Handling

| Scenario | Handling |
| --- | --- |
| LLM returns invalid JSON | `parse_llm_json` tries 4 strategies; raises `ValueError` on all failures → HTTP 502 |
| LLM fails per-slide during build | try/except per slide → `fallback_html()` (plain bullets, no crash) |
| Input classified as unsafe | HTTP 400 from guardrails before LLM call |
| Output classified as unsafe | HTTP 502 from guardrails after LLM call |
| No slides provided to `/build` | HTTP 400 |
| Slide render timeout | Client-side: 60s (`/regenerate-slide`), 300s (`/build`, `/generate`), 90s (`/ai-edit-slide`) |

### Observability

| Type | Details |
| --- | --- |
| **Logs** | `the-powerpoint-er` logger |
| **Traces** | MLflow autolog, experiment ID 1811265038828694 |
| **Artifacts** | Disabled (avoids S3 ConnectionResetError in Databricks Apps) |
| **Health** | `GET /api/health` → `{status, service, env}` |

### Current Limitations

Exported PPTX is **image-based and not editable after download**: each slide is rendered to PNG (via `html-to-image`) and embedded into the `.pptx` file using `pptxgenjs`. As a result, when the file is opened in PowerPoint / Keynote / Google Slides, users cannot edit text, shapes, or layout — only replace the image or overlay new content on top.

- **Mitigation:** all editing operations (text, formatting, AI edit, regenerate) are available on the web UI before export.
- Investigating a proper solution to export PPTX with native editable shapes and text objects.

---

## Vaulter: The Vaulter

>
> **Agent: The Vaulter**

### Agent: The Vaulter

### Agent Overview

| Field | Value |
| --- | --- |
| **Name** | The Vaulter |
| **ID** | `the-vaulter` |
| **Databricks App** | `agent-the-vaulter` |
| **Primary Responsibility** | Build a knowledge graph from documents and enable natural-language queries |
| **Business Purpose** | Create a personal knowledge base from uploaded documents; answer questions using stored knowledge |
| **When Invoked** | User uploads documents to vault; queries the vault; or Hub calls `upload_to_vault`/`query_vault` tools |

### Input & Output

**Upload Input**

```json
{"volume_path": "/Volumes/.../document.pdf"}
```

**Upload Output**

```json
{"document_id": 1, "filename": "doc.pdf", "nodes_extracted": 12, "edges_extracted": 8}
```

**Query Input**

```json
{"question": "Who is responsible for project X?"}
```

**Query Output**

```json
{"answer": "Based on the documents, John Smith is responsible for..."}
```

**Side Effects**

- Documents stored in Lakebase `agent_studio.documents` table
- Graph nodes stored in `agent_studio.nodes` table
- Graph edges stored in `agent_studio.edges` table

### Core Logic

#### User Flow

1. User opens The Vaulter page
2. **Upload mode:** User drops/selects a document (PDF, DOCX, TXT) into the upload zone
3. User clicks "Upload to Vault" → file is uploaded to Volume, then processed
4. **Processing:** Backend parses document, extracts entities/relationships via LLM
5. **Result:** Shows "X nodes, Y edges extracted" confirmation
6. **Graph view:** User sees an interactive knowledge graph visualization (nodes + edges)
7. **Query mode:** User types a question in the query box (e.g., "Who manages project Alpha?")
8. User clicks "Ask" → backend searches stored documents and answers using context
9. **Answer displayed:** LLM-generated answer based only on vault content

#### Upload Flow (Technical)

1. Download file from Volume → validate → parse text via `parse_file()`
2. Store document content in Lakebase (`documents` table)
3. Call `extract_graph()` — LLM extracts entities and relationships as JSON
4. Upsert nodes and edges into Lakebase (deduplicated via `ON CONFLICT DO NOTHING`)

#### Query Flow (Technical)

1. Load all document content for the user (last 5 documents, concatenated)
2. Truncate context at 10,000 chars
3. Send to Haiku (`fast=True`) with system prompt: "Answer using ONLY the provided context"
4. Return LLM answer

#### Graph Extraction

- System prompt instructs LLM to extract named entities and relationships
- Input truncated at 6,000 chars
- Returns `{"nodes": ["Entity1", ...], "edges": [{"source": "A", "target": "B", "relation": "..."}]}`
- On parse failure, returns empty graph (graceful degradation)

### Internal APIs / Functions

| Function | Purpose |
| --- | --- |
| `vault_upload()` | Parse document, extract graph, store in Lakebase |
| `vault_graph()` | Return full graph (nodes + edges) for user |
| `vault_query()` | Answer question using stored document context |
| `extract_graph()` | LLM-based entity/relationship extraction |
| `VaultStore.add_document()` | Store document text |
| `VaultStore.upsert_nodes()` | Batch insert nodes (`ON CONFLICT DO NOTHING`) |
| `VaultStore.upsert_edges()` | Batch insert edges (`ON CONFLICT DO NOTHING`) |
| `VaultStore.get_graph()` | Retrieve all nodes/edges for user |
| `VaultStore.get_all_content()` | Get last 5 documents' content |

### Dependencies

| Type | Details |
| --- | --- |
| **Other Agents** | None (standalone) |
| **Shared Modules** | `llm.py`, `llm_utils.py`, `file_parser.py`, `file_validator.py`, `file_store.py`, `user_context.py`, `upload_router.py`, `schemas.py` |
| **External Libraries** | `databricks-ai-bridge` (LakebaseClient), langchain-core |
| **Agent-specific** | `vault_store.py`, `graph_extractor.py` |

### Error Handling & Edge Cases

| Scenario | Handling |
| --- | --- |
| Empty vault (no documents) | Returns "Your vault is empty. Upload some documents first." |
| Graph extraction fails (invalid JSON) | Returns empty nodes/edges (graceful degradation) |
| Document parse error | HTTP 400 |
| Context exceeds 10K chars | Truncated with warning log |

### Known Limitations

- No vector search — queries load ALL document content (last 5 docs). Not scalable beyond ~50K chars total
- Graph extraction limited to 6,000 chars per document
- No document deletion API
- Graph is per-user (no sharing)

### Configuration

| Variable | Purpose |
| --- | --- |
| `DATABRICKS_LLM_ENDPOINT` | Sonnet (for graph extraction) |
| `DATABRICKS_HTTP_PATH` | SQL Warehouse for document parsing |
| `VOLUME_PATH` | File storage |
| `LAKEBASE_AUTOSCALING_PROJECT` | Lakebase project for graph storage |
| `LAKEBASE_AUTOSCALING_BRANCH` | Lakebase branch |
| `MLFLOW_EXPERIMENT_ID` | 1811265038828696 |

**Constants**

| Constant | Value | Purpose |
| --- | --- | --- |
| `_MAX_QUERY_CONTEXT` | 10,000 | Max chars for query context |
| `_MAX_EXTRACT_CHARS` | 6,000 | Max chars for graph extraction |

### Observability

| Type | Details |
| --- | --- |
| **Logs** | `agent_studio.vault` — logs upload/query operations |
| **Traces** | MLflow autolog (experiment: 1811265038828696) |

### Related content

---

## Brainstormer: The Brainstormer

>
> **Agent: The Brainstormer**

### Agent: The Brainstormer

### Agent Overview

| Field | Value |
| --- | --- |
| **Name** | The Brainstormer |
| **ID** | `the-brainstormer` |
| **Databricks App** | `agent-the-brainstormer` |
| **Primary Responsibility** | Structured brainstorming with live canvas and cross-session memory |
| **Business Purpose** | Help employees think through ideas, problems, and decisions with a Socratic AI partner |
| **When Invoked** | User starts a brainstorming session; or Hub calls `brainstorm` tool |

### Input & Output

**Input**

```json
{"message": "I need to improve customer onboarding", "session_id": null}
```

**Output**

```json
{"reply": "What would change if you solved this?", "canvas": {"problem": null, "options": [], "action_plan": []}, "session_id": "uuid", "completed": false}
```

**Side Effects**

- Session state persisted in Lakebase via LangGraph `AsyncCheckpointSaver`
- Session metadata stored in `agent_studio.brainstorm_index` table
- User preferences extracted and stored in `agent_studio.user_preferences` table

### Core Logic

#### User Flow

1. User opens The Brainstormer page
2. User sees chat panel (left) + live canvas panel (right, initially empty)
3. User types their brainstorming topic (e.g., "How to improve customer onboarding?")
4. **Step 1 — Understand (exchanges 1-3):** AI asks one clarifying question per message to understand the real goal. Canvas shows: `problem: null`
5. **Step 2 — Explore (exchanges 3-6):** AI maps 2-3 distinct approaches with trade-offs. Canvas updates: `problem: "...", options: ["Option A", "Option B"]`
6. **Step 3 — Converge (exchanges 6-8):** AI helps evaluate options against the stated goal. Canvas refines options.
7. **Step 4 — Wrap up (exchange 8+):** AI delivers a clear action plan. Canvas shows: `action_plan: ["Step 1", "Step 2", ...]`. Session marked as `completed: true`.
8. User can return later → sessions are listed in sidebar → clicking a session restores full conversation + canvas state
9. AI remembers user preferences (language, tone, past topics) across all sessions

#### LangGraph StateGraph

```
load_preferences → chat → update_preferences → END
```

#### Canvas System

After each reply, the LLM appends a `---CANVAS---` JSON block:

```json
{"problem": "one sentence or null", "options": ["option 1", "option 2"], "action_plan": ["step 1"]}
```

#### Memory Architecture

- **Short-term:** LangGraph `AsyncCheckpointSaver` (Lakebase) — full message history within a session
- **Long-term:** `UserPreferencesStore` — language, tone, topics across all sessions
- **Message compaction:** When conversation exceeds 10 messages, older messages are summarized via Haiku

#### Forced Wrap-up Rule

At the 8th user message, a system message is injected: *"You MUST wrap up now. Deliver the final action plan."*

### 4. Internal APIs / Functions

| Function | Purpose |
| --- | --- |
| `brainstorm_chat()` | Main endpoint — creates/resumes session, invokes graph |
| `list_sessions()` | List user's brainstorm sessions |
| `load_session()` | Load session with messages and canvas |
| `build_graph()` | Construct LangGraph StateGraph |
| `load_preferences()` | Node: load user prefs from Lakebase |
| `chat()` | Node: invoke LLM with system prompt + compacted messages |
| `update_preferences()` | Node: extract and save preferences from last user message |
| `_parse_canvas()` | Split reply from canvas JSON |
| `SessionIndex.create/update_canvas/list_sessions/get()` | Session metadata CRUD |

### Dependencies

| Type | Details |
| --- | --- |
| **Other Agents** | None (standalone) |
| **Shared Modules** | `llm.py`, `user_context.py`, `user_preferences.py`, `compact_messages.py` |
| **External Libraries** | langgraph, langchain-core, databricks-langchain (AsyncCheckpointSaver) |
| **Agent-specific** | `agent.py` (LangGraph graph), `session_index.py` (metadata store) |

### Error Handling & Edge Cases

| Scenario | Handling |
| --- | --- |
| Empty message | HTTP 400: "Message cannot be empty" |
| Session not found | HTTP 404 |
| LLM fails to emit canvas JSON | Previous canvas preserved (fallback) |
| Preference extraction fails | Returns existing preferences unchanged |
| Message compaction fails | Falls back to recent window only (no summary) |

### Configuration

| Variable | Purpose |
| --- | --- |
| `DATABRICKS_LLM_ENDPOINT` | Sonnet (for chat) |
| `LAKEBASE_AUTOSCALING_PROJECT` | Lakebase project for checkpoints + session index |
| `LAKEBASE_AUTOSCALING_BRANCH` | Lakebase branch |
| `MLFLOW_EXPERIMENT_ID` | 1811265038828693 |

### Observability

| Type | Details |
| --- | --- |
| **Logs** | `the-brainstormer` logger |
| **Traces** | MLflow autolog (experiment: 1811265038828693) |

### Related content

---

## Visionary: The AI Visionary

### Agent: The AI Visionary

### Agent Overview

| Field | Value |
| --- | --- |
| **Name** | The AI Visionary |
| **ID** | `the-ai-visionary` |
| **Databricks App** | `agent-the-ai-visionary` |
| **Primary Responsibility** | AI vision Q&A (proxy to Knowledge Agent endpoint) + PPTX report generation |
| **Business Purpose** | Answer questions about AI strategy/vision using a specialized Knowledge Agent; generate summary decks |
| **When Invoked** | User asks AI vision questions; requests a report deck; or Hub calls `ask_visionary` tool |

### Input & Output

**Chat Input/Output**

```js
// Input
{"messages": [{"role": "user", "content": "What is our AI roadmap for 2025?"}]}

// Output
{"reply": "Based on the AI strategy documents..."}
```

**Build Input/Output**

```js
// Input
{"slides": [{"title": "...", "bullets": [...]}], "topic": "AI Weekly Update"}

// Output
{"file_id": "uuid", "num_slides": 6}
```

### 3. Core Logic

#### User Flow

1. User opens The AI Visionary page
2. **Chat mode (default):** User types a question about AI strategy/vision (e.g., "What's our AI roadmap for 2025?")
3. AI responds with insights from the Knowledge Agent (backed by internal AI strategy documents)
4. User can continue the conversation — context is maintained across messages
5. **Report mode:** User clicks "Generate Report" or asks for a deck
6. User provides a prompt (e.g., "Create a weekly AI update deck")
7. **Outline generated:** LLM creates slide structure → user can review/edit
8. User clicks "Build" → PPTX is generated using template
9. **Download:** User downloads the branded `.pptx` file

#### Chat Flow (Proxy to Knowledge Agent)

1. Validate messages (last must be user, max 50 messages, max 30K chars per message)
2. Compact messages via `acompact_messages()` (Haiku) if conversation is long
3. Build payload: `{"input": condensed_messages}`
4. `POST` to Databricks serving endpoint (`VISIONARY_AGENT_ENDPOINT`)
5. Auth via WorkspaceClient SP token
6. Parse response: extract text from `output[].content[].text`
7. Return reply

#### Outline + Build Flow

- **Outline:** LLM generates JSON array of slides (title + bullets + notes)
- **Build:** Renders slides to PPTX using template
  - Cover slide: title in ALL CAPS
  - Content slides: title + bullet list
  - Simpler rendering than PPTX-er (no visual enrichment)

### Internal APIs / Functions

| Function | Purpose |
| --- | --- |
| `visionary_chat()` | Proxy chat to Knowledge Agent endpoint |
| `generate_outline()` | LLM generates slide outline |
| `build_pptx()` | Render slides to PPTX file |
| `download_visionary()` | Serve generated PPTX |
| `_get_endpoint_url()` | Construct KA endpoint URL from env vars |
| `_get_token()` | Get SP auth token |
| `_build_pptx()` | Core PPTX renderer (simpler than PPTX-er) |

### Dependencies

| Type | Details |
| --- | --- |
| **Other Agents** | None directly (proxies to external Knowledge Agent endpoint) |
| **Shared Modules** | `llm.py`, `llm_utils.py`, `file_store.py`, `user_context.py`, `compact_messages.py` |
| **External Libraries** | httpx (async HTTP client), python-pptx |
| **External Services** | Databricks serving endpoint (Knowledge Agent: `ka-a0573ee8-endpoint`) |

### Error Handling & Edge Cases

| Scenario | Handling |
| --- | --- |
| Last message not from user | HTTP 400 |
| Too many messages (>50) | HTTP 400: "Start a new session" |
| Message too long (>30K chars) | HTTP 400 |
| KA endpoint returns error | HTTP 502 |
| LLM returns invalid outline | HTTP 502 |
| File not found for download | HTTP 404 |

### Configuration

| Variable | Purpose |
| --- | --- |
| `DATABRICKS_LLM_ENDPOINT` | Sonnet (for outline generation) |
| `DATABRICKS_HOST` | Workspace host (for KA endpoint URL) |
| `VISIONARY_AGENT_ENDPOINT` | Knowledge Agent endpoint name (default: `ka-a0573ee8-endpoint`) |
| `VOLUME_PATH` | File storage |
| `MLFLOW_EXPERIMENT_ID` | 1811265038828692 |

### Observability

| Type | Details |
| --- | --- |
| **Logs** | `visionary` logger |
| **Traces** | MLflow autolog (experiment: 1811265038828692) |

---

## Canvas Designer: The Canvas Designer

### Agent: The Canvas Designer

### Agent Overview

| Field | Value |
| --- | --- |
| **Name** | The Canvas Designer |
| **ID** | `the-canvas-designer` |
| **Databricks App** | `agent-the-canvas-designer` |
| **Primary Responsibility** | Generate visual graphics, vector art, and formatted documents from text descriptions |
| **Business Purpose** | Create presentation visuals, infographics, and executive report designs without design tools |
| **When Invoked** | User describes desired visual; selects output format (PNG/JPG/SVG/DOCX) |

### Input & Output

**Input**

```json
{"description": "A futuristic robot with glowing eyes", "style": "creative", "output_format": "png", "width": 1200, "height": 1200}
```

**Output**

```json
{"file_id": "uuid", "filename": "canvas-abc12345.png", "format": "png", "width": 1200, "height": 1200}
```

**Side Effects**

- Generated file saved to Volume (`/canvas/{user}/{date}/canvas-{id}.{ext}`)

### Core Logic

#### User Flow

1. User opens The Canvas Designer page
2. User types a description of the desired visual (e.g., "A futuristic robot with glowing neon eyes")
3. User selects output format: **PNG** (default), **JPG**, **SVG**, or **DOCX**
4. User optionally adjusts dimensions (default: 1200×1200)
5. User clicks "Generate"
6. **Loading state:** Spinner while LLM generates design spec and renderer produces the image
7. **Preview:** Generated image/document is displayed in the result panel
8. User clicks "Download" to save the file
9. User can modify the description and regenerate

#### Output Format Routing

| Format | Method | Technology |
| --- | --- | --- |
| **PNG** | `generate_artistic_image()` | Pillow (PIL) — programmatic pixel art with glows, gradients |
| **JPG** | `generate_artistic_image()` | Same as PNG, saved as JPEG |
| **SVG** | `generate_svg()` | LLM generates raw SVG code |
| **DOCX** | `generate_docx()` | LLM generates JSON spec → python-docx renders |

#### PNG/JPG Generation (Programmatic Art)

- LLM (Opus for creative quality) generates a JSON specification describing visual components
- System prompt provides extensive component library and design principles
- Renderer processes the JSON spec:
  - Create RGBA image with gradient background + vignette
  - Render components: glowing shapes, bezier curves, stars, text with glow, charts
  - Apply multi-layer glow effects, soft lighting, particles
- Save to Volume

#### Color Palettes (5 built-in)

| Palette | Theme |
| --- | --- |
| `synthetic_optimism` | Cyan/pink on dark blue |
| `midnight_aurora` | Blue/purple on deep navy |
| `forest_tech` | Green on dark forest |
| `cosmic_dream` | Pink/blue on space black |
| `neon_noir` | Neon green/magenta on pure black |

### Internal APIs / Functions

| Function | Purpose |
| --- | --- |
| `generate_canvas()` | Main endpoint — routes by format |
| `download_canvas()` | Serve generated file (tries all extensions) |
| `generate_artistic_image()` | Pillow-based programmatic art renderer |
| `generate_svg()` | LLM → raw SVG code |
| `generate_docx()` | LLM → JSON spec → python-docx |
| `render_component()` | Dispatch component rendering by type |
| `draw_soft_glow()` / `draw_multi_layer_glow()` | Glow effects |
| `draw_glowing_line/arc/ellipse()` | Shape primitives with glow |
| `create_gradient_background()` | Gradient + vignette + noise |

### Dependencies

| Type | Details |
| --- | --- |
| **Other Agents** | None (standalone) |
| **Shared Modules** | `llm.py`, `file_store.py` (partial), `user_context.py`, `upload_router.py` |
| **External Libraries** | Pillow (PIL), python-docx |
| **LLM Model** | Uses `get_llm(opus=True)` for creative generation (Claude Opus 4.5) |

### Error Handling & Edge Cases

| Scenario | Handling |
| --- | --- |
| LLM returns invalid JSON for image spec | Falls back to default composition |
| LLM returns invalid SVG | Raw response returned as-is |
| LLM returns invalid DOCX spec | Falls back to simple document |
| File not found for download | HTTP 404 (tries all format extensions) |
| Download searches only current date | Limitation — files from previous days may not be found |

### Configuration

| Variable | Purpose |
| --- | --- |
| `DATABRICKS_LLM_ENDPOINT` | Sonnet (for SVG/DOCX) |
| `DATABRICKS_LLM_ENDPOINT_OPUS` | Opus (for artistic image generation) |
| `VOLUME_PATH` | File storage |
| `MLFLOW_EXPERIMENT_ID` | 1818065766679570 |

### Observability

| Type | Details |
| --- | --- |
| **Logs** | `agent_studio.canvas` logger |
| **Traces** | MLflow autolog (experiment: 1818065766679570) |

---

## Resume Evaluator: The Resume Evaluator

### Agent: The Resume Evaluator

### Agent Overview

| Field | Value |
| --- | --- |
| **Name** | The Resume Evaluator |
| **ID** | `the-resume-evaluator` |
| **Databricks App** | `agent-the-resume-evaluator` |
| **Primary Responsibility** | Evaluate resumes/CVs against job descriptions with structured scoring, batch ranking, and PDF reports |
| **Business Purpose** | Help HR teams screen candidates consistently with AI-powered evaluation, reducing bias and time |
| **When Invoked** | User uploads resume(s) + provides job description |

### Input & Output

**Single Evaluation Input**

```json
{"resume_path": "/Volumes/.../resume.pdf", "job_description": "Senior Data Engineer..."}
```

**Single Evaluation Output**

```json
{"decision": "INTERVIEW", "score": 82, "confidence": "High", "title_match": "Direct", "summary": "Strong candidate...", "scores": {...}, "red_flags": [...], "report_id": "file_id"}
```

**Batch Input**

```json
{"resume_paths": ["/Volumes/.../r1.pdf", "/Volumes/.../r2.pdf"], "job_description": "..."}
```

### 3. Core Logic

#### User Flow — Single Evaluation

1. User opens The Resume Evaluator page
2. User pastes or types the job description in the left panel
3. User drops/selects a resume file (PDF, DOCX, or TXT)
4. User clicks "Evaluate"
5. **Loading state:** Spinner while backend parses resume and calls LLM
6. **Result displayed:** Score card with decision (`INTERVIEW` / `CONDITIONAL` / `DO_NOT_INTERVIEW`), overall score, and confidence level
7. **Detailed breakdown shown:**
   - 6-category radar chart (Skills, Experience, Transferability, Achievement, Online Presence, Education)
   - Red flags with severity indicators
   - Culture fit assessment
   - Authenticity assessment (AI-generation detection)
   - Suggested interview questions
   - Reference check prompts
8. User clicks "Download Report" → receives DOCX evaluation report

#### User Flow — Batch Evaluation

1. User pastes job description
2. User uploads multiple resumes (up to 10)
3. User clicks "Evaluate All"
4. **Progress:** Each candidate evaluated sequentially with progress indicator
5. **Rankings displayed:** Candidates sorted by score with key strength/concern per candidate
6. **Skill gap analysis:** Shows which requirements are unmet across the candidate pool
7. User can download individual reports or a ZIP with all reports + comparison summary

#### Evaluation Flow (Technical)

1. Parse resume from Volume path via `parse_file()` (supports PDF, DOCX, TXT)
2. Truncate resume + JD at 50,000 chars each
3. Send to Sonnet with comprehensive system prompt
4. LLM returns structured JSON with 6-category scoring + assessments
5. Parse response → build `EvaluateResponse` with all sub-assessments
6. Generate individual DOCX report → save to Volume

#### Scoring System (6 Categories)

| Category | Weight | Description |
| --- | --- | --- |
| Skills Match | 25% | Technical requirements alignment |
| Experience Relevance | 20% | Role history fit (recency-weighted) |
| Achievement Quality | 20% | Quantified impact with metrics |
| Education Fit | 15% | Credentials alignment |
| Transferability | 10% | Off-title potential |
| Online Presence | 10% | LinkedIn, GitHub, portfolio |

#### Decision Matrix

| Score | Red Flags | Decision |
| --- | --- | --- |
| ≥75 | None/Low | `INTERVIEW` |
| ≥75 | High | `CONDITIONAL` |
| 60-74 | None/Low | `INTERVIEW` (promising) |
| 60-74 | Medium+ | `CONDITIONAL` |
| <60 | Any | `DO_NOT_INTERVIEW` |

#### Advanced Assessments

- **Transferability:** Conceptual overlap (40%) + Problem pattern (35%) + Tool adjacency (25%)
- **Culture Fit:** Communication style, ownership, collaboration, growth mindset, environment fit
- **Authenticity:** AI-generation likelihood, specificity, buzzword density, internal consistency
- **Reference Check:** Priority references, verification questions, behavioral questions, watch-for items

### Internal APIs / Functions

| Function | Purpose |
| --- | --- |
| `evaluate_resume()` | Single resume evaluation endpoint |
| `batch_evaluate()` | Batch evaluation with ranking |
| `download_report()` | Serve generated report file |
| `evaluate_single_resume()` | Core evaluation logic |
| `parse_resume_from_path()` | Download + parse resume from Volume |
| `parse_llm_response()` | Extract JSON from LLM response |
| `build_evaluate_response()` | Construct typed response from raw dict |
| `generate_individual_pdf()` | Generate DOCX report for one candidate |

### Dependencies

| Type | Details |
| --- | --- |
| **Other Agents** | None (standalone) |
| **Shared Modules** | `llm.py`, `file_parser.py`, `file_validator.py`, `file_store.py`, `user_context.py`, `upload_router.py` |
| **External Libraries** | python-docx (for report generation), langchain-core |

### Error Handling & Edge Cases

| Scenario | Handling |
| --- | --- |
| Empty resume | HTTP 400: "Resume appears to be empty" |
| Batch > 10 candidates | HTTP 400: "Maximum 10 candidates per batch" |
| Individual evaluation fails in batch | Logged; returns `score=0`, `decision=DO_NOT_INTERVIEW` |
| LLM returns invalid JSON | HTTP 500: "Failed to parse evaluation response" |
| Report generation fails | Logged; evaluation still returned (`report_id = null`) |

### Known Limitations

- Max 10 candidates per batch (sequential processing — no parallelism)
- Resume text truncated at 50,000 chars
- No caching — same resume re-evaluated each time
- Report format is DOCX only (not PDF despite function name)

### Configuration

| Variable | Purpose |
| --- | --- |
| `DATABRICKS_LLM_ENDPOINT` | Sonnet (for evaluation) |
| `DATABRICKS_HTTP_PATH` | SQL Warehouse for document parsing |
| `VOLUME_PATH` | File storage |
| `MLFLOW_EXPERIMENT_ID` | 1818065766679571 |

**Constants**

| Constant | Value |
| --- | --- |
| `_MAX_CHARS` | 50,000 |
| `_ALLOWED` | pdf, docx, txt |

### Observability

| Type | Details |
| --- | --- |
| **Logs** | `agent_studio.evaluate` logger — logs parse sizes, evaluation progress |
| **Traces** | MLflow autolog (experiment: 1818065766679571) |
