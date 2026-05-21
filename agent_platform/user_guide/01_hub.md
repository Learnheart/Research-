# Hub — The unified AI assistant

> One single chat window. Describe what you need — Hub picks and calls the right specialized agent behind the scenes.

---

## 1. Capabilities overview

**What is Hub?** It is the entry point for the entire Agent Studio. You don't need to know "where the Translator lives" or "which agent can summarize files" — just write your request in natural language, and Hub will orchestrate.

**When to use Hub**
- You're not sure which agent fits.
- You want to chain tasks (translate → summarize → build slides) inside the same conversation.
- You need an assistant that "remembers" context across requests.

**Typical input**
- Text messages: `"Translate this report into English"`, `"Create a 10-slide presentation about AI strategy 2026"`.
- File attachments (when Hub asks): PDF, DOCX, PPTX, TXT.

**Typical output**
- A text response inside the chat window.
- Associated actions: download buttons, opening a brainstorming canvas, displaying a slide preview, etc.

**Agents Hub can call (via tools):**
Translator, Summarizer, Powerpoint-er, Vaulter, Brainstormer, AI Visionary.

---

## 2. Limits & prerequisites

| Item | Limit |
| --- | --- |
| Supported files | PDF, DOCX, PPTX, TXT |
| File size | Up to **20 MB** per file |
| Authentication | Organization SSO required |
| Agents not yet connected | **Canvas Designer** and **Resume Evaluator** are not exposed as Hub tools — please open those agents directly |
| Streaming | Not supported — the result is returned in one shot when processing completes |

---

## 3. How to use (step-by-step)

### Step 1 — Open Hub
Go to the **Hub** tab on the Agent Studio home page.

![Hub home screen](./images/hub-01-home.png)
*Figure 1: Hub's main interface — chat window in the middle, session history in the left sidebar.*

### Step 2 — Type your request
Enter what you need in natural language, for example: `"Build a 10-slide presentation about the 2026 AI plan for the Board of Directors"`.

![Entering a request](./images/hub-02-prompt.png)
*Figure 2: The message input at the bottom. Press Enter to send.*

### Step 3 — Hub responds
- **Case A — File needed:** Hub replies `"I need you to upload a file…"`. A drag-and-drop zone appears inside the chat.
- **Case B — No file needed:** Hub calls the right agent right away (e.g. the Powerpoint-er) and starts processing.

![File upload request](./images/hub-03-file-request.png)
*Figure 3: When a file is needed, a drag-and-drop zone shows up inside the chat.*

### Step 4 — Upload the file (if requested)
Drag and drop the file into the highlighted zone, or click to pick it from your computer.

### Step 5 — Wait for processing
Hub may take 10–60 seconds because it calls a specialized agent behind the scenes. The chat shows the processing status.

### Step 6 — Receive the result
The result appears as a text response, accompanied by action buttons (Download, Open canvas, etc.).

![Result with download button](./images/hub-04-result.png)
*Figure 4: Hub returns the result along with a Download button to grab the file.*

### Step 7 — Continue the conversation (optional)
Type your next request — Hub still remembers the previous file/context. Example: `"Now summarize that same file"`.

---

## 4. Walkthroughs (User Journeys)

### Journey 1 — Quickly translate a financial report

**Context:** You received `Q3_Report.pdf` from a partner and need an English version to send to the regional director.

**Steps:**
1. Open Hub, type: `"Translate this file into English"`.
2. Hub responds: `"I need you to upload a file to translate."`
   ![Hub asking for a file](./images/hub-scn1-upload-prompt.png)
   *Figure: The upload zone appears after the request.*
3. Drag and drop `Q3_Report.pdf`.
4. Hub calls the Translator (~45 seconds for a 10-page PDF), showing progress.
5. Hub returns: `"Done translating. Download the file here:"` + a Download button.

**Outcome:** Download `Q3_Report_translated.docx` (PDF is converted to DOCX — see [The Translator](./02_translator.md)).

---

### Journey 2 — Multi-step chain inside a single conversation

**Context:** You have a meeting minute `Meeting_2026-05-15.docx` and want to (1) summarize it quickly, then (2) build a review deck.

**Steps:**
1. Open Hub, type: `"Summarize these meeting minutes"` → upload the file → receive a 5-sentence summary + 7 key points.
2. Continue in the same session: `"Build an 8-slide deck presenting the key decisions for the team"`.
3. Hub uses the summary you just got as input, calls the Powerpoint-er, and returns the slide preview.
4. Review the outline → click "Build" → download the `.pptx`.

![Two tools chained in one session](./images/hub-scn2-chained.png)
*Figure: Hub combines the Summarizer and the Powerpoint-er in the same conversation.*

---

### Journey 3 — Direct request without a file

**Context:** You want a quick PowerPoint draft on a new topic.

**Steps:**
1. Open Hub, type: `"Create a 10-slide presentation on 'AI Agent applications in banking operations' for executive leadership"`.
2. Hub calls the Powerpoint-er directly (no input file needed) and returns an outline.
3. Review the outline, type: `"Convert slide 5 into a process timeline"` — Hub invokes the ai-edit-slide tool.
4. When satisfied → Build → download the PPTX.

---

## 5. Usage tips

- **Use clear "verb + object" descriptions**: `"Translate this file into English"` is clearer than `"Process this for me"`.
- **Specify the output language/format**: `"Summarize in Vietnamese, no more than 5 bullets"`.
- **Leverage in-session context**: once a file is uploaded, you don't need to re-upload — just say `"Now do X with that file"`.
- **If you need Canvas Designer or Resume Evaluator**: open the agent page directly, since Hub is not connected to them yet.
- **For complex requests, split them into smaller steps**: instead of `"Translate, summarize, then build slides"` in one command, send three sequential commands so you can review each result.

---

## 6. Known limitations

- **Higher latency**: Hub calls downstream agents over HTTP → each command is ~2–5 seconds slower than calling the agent directly.
- **No streaming**: you won't see text appear gradually — you must wait for the full result.
- **Canvas Designer & Resume Evaluator not yet connected**: you must use their standalone pages.
- **Session state grows**: the longer the conversation, the more Hub has to compress history — memory quality degrades after ~30 turns. We recommend starting a new session when switching to a major new topic.
- **No undo**: if a request is wrong, send a new one — Hub has no "undo" button for actions already executed.
