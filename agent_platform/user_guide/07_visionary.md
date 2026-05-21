# The AI Visionary — Q&A on the internal AI strategy

> A Q&A portal on the organization's AI direction (powered by a dedicated Knowledge Agent) + a fast way to produce update decks in PPTX.

---

## 1. Capabilities overview

**What does The AI Visionary do?** It answers questions about the organization's **AI strategy / vision** by proxying to a dedicated **Knowledge Agent** (trained on internal strategy documents), and it can also generate summary report decks.

**When to use it**
- You need a quick lookup like "What is our 2026 AI roadmap?".
- You want to refresh your knowledge of the AI direction before presenting to a client / executive.
- You need a quick update deck on AI strategy for your team.

**Input**
- **Chat mode**: text question (can continue across multiple turns).
- **Report mode**: a prompt describing the deck you want.

**Output**
- **Chat**: text response based on internal documents.
- **Report**: a simple `.pptx` file (ALL CAPS cover + bullet content slides).

---

## 2. Limits & prerequisites

| Item | Limit |
| --- | --- |
| Messages per session | Up to **50** |
| Length per message | Up to **30,000 characters** |
| Source documents | **Fixed** — managed by the Knowledge Agent owner team; users cannot upload more |
| Authentication | SSO required |
| Report deck style | Simple (cover + content) — **simpler than the Powerpoint-er** |

> ℹ️ The Visionary **does not ingest your own documents**. If you need Q&A on files you upload, use [Vaulter](./05_vaulter.md). The Visionary only answers based on what the Knowledge Agent has already been trained on.

---

## 3. How to use (step-by-step)

### A. Chat mode (default)

#### Step 1 — Open The AI Visionary
![Chat interface](./images/visionary-01-chat.png)
*Figure 1: Main chat window, input field at the bottom.*

#### Step 2 — Type your question
Examples: `"What is our AI roadmap for 2026?"`, `"What is the AI strategy in Wholesale Banking?"`, `"Which Generative AI use cases are prioritized this year?"`.

#### Step 3 — Receive the answer
The Visionary proxies the request to the Knowledge Agent endpoint, parses the result, and displays the reply.

![Answer from KA](./images/visionary-02-answer.png)
*Figure 2: Text response with insights from the Knowledge Agent.*

#### Step 4 — Ask follow-ups
The Visionary preserves session context — you can follow up with `"And what's the Q3 milestone?"`.

### B. Report mode

#### Step 1 — Click "Generate Report"
![Switch to Report mode](./images/visionary-03-report-mode.png)
*Figure 3: The Generate Report button in the toolbar.*

#### Step 2 — Describe the deck
Example: `"Generate a 6-slide deck updating the AI Strategy for the executive board"`.

#### Step 3 — Review the outline
The LLM produces an outline JSON: title + bullets + notes per slide.

#### Step 4 — Click "Build"
Slides render via python-pptx (simpler than the Powerpoint-er — ALL CAPS cover + bullet content).

#### Step 5 — Download the PPTX
![Download the deck](./images/visionary-04-download.png)
*Figure 4: The .pptx downloads to your machine.*

---

## 4. Walkthroughs (User Journeys)

### Journey 1 — Quick AI roadmap lookup before a presentation

**Context:** You're about to present to a client `"Introducing the bank's AI strategy"`. You need a fast refresher on the main points.

**Steps:**
1. Open the Visionary → `"Summarize the 2026 AI roadmap across the 3 main pillars"`.
2. Get a reply with the 3 pillars + prioritized use cases.
3. Follow up: `"Which GenAI use cases are prioritized in Q3?"` → reply.
4. Follow up: `"Which team is receiving the largest 2026 AI budget allocation?"`.
5. Copy the relevant snippets into your own slides.

---

### Journey 2 — Weekly team update deck

**Context:** Every week you send an "AI Initiatives This Week" deck to a team of 20.

**Steps:**
1. Open the Visionary → click Generate Report.
2. Prompt: `"Generate an 8-slide weekly AI update deck: 1 cover, 1 agenda, 5 highlights (one per slide), 1 next steps"`.
3. Review the outline → tweak titles if needed.
4. Click Build → download `weekly_AI_update.pptx`.
5. Open in PowerPoint and edit if needed — **Visionary's PPTX export is editable** (unlike the Powerpoint-er's).

![Weekly update deck](./images/visionary-scn2-weekly.png)
*Figure: Simple deck with a cover + bullet content slides.*

---

### Journey 3 — Multi-turn Q&A to learn deeply

**Context:** You've just joined and need to learn the AI strategy quickly.

**Steps:**
1. Turn 1: `"Which initiatives exist in the internal AI strategy?"` → a list reply.
2. Turn 2: `"Initiative #3 — can you go deeper?"` → a detailed reply.
3. Turn 3: `"Who is the sponsor of that initiative?"` → a reply.
4. Turn 4: `"Is there a source document I can read?"` → reply may include a link.

---

## 5. Usage tips

- **Visionary ≠ Vaulter**: the Visionary answers from a **fixed corpus of internal strategy documents** (managed by a team). The Vaulter answers from **files you upload yourself**.
- **English or Vietnamese questions both work**, but the Knowledge Agent may answer more accurately in English depending on how the corpus was prepared.
- **Open a new session when switching topics**: 50 messages is the ceiling — start a fresh session to stay lean.
- **Long input warning**: pasting a block of >30K characters (~6,000 words) into chat will be rejected. Summarize before pasting.
- **Report decks are simple** — fine for internal updates. For a visual client-ready deck, use [the Powerpoint-er](./04_powerpointer.md).
- **Citations**: ask `"What document is this from?"` — the Knowledge Agent may return the source document name if configured.

---

## 6. Known limitations

- **Fixed source**: you cannot add/edit documents — contact the Knowledge Agent team.
- **No streaming**: wait for the full reply.
- **50-message / 30K-character ceiling** — good for Q&A, not for batch querying.
- **Report mode is simple**: no rich branded template, no rich editor, no per-slide AI Edit.
- **No rich outline editing like the Powerpoint-er's**: only cover + content slides.
- **Depends on the Knowledge Agent endpoint**: if the endpoint is down, the Visionary returns HTTP 502 — you have to wait for recovery.
