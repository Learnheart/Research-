# The Summarizer — Document summarization

> Condense long reports, meeting notes, and contracts into a 3–5 sentence executive summary plus 5–8 actionable key insights.

---

## 1. Capabilities overview

**What does The Summarizer do?** It accepts one or more documents, reads them in full, and returns:
- An executive-level summary (3–5 sentences).
- A list of 5–8 key bullet points (prioritizing decisions, numbers, deadlines, risks).

**When to use it**
- You need to "get up to speed" in 2 minutes before a meeting.
- You have several files on the same topic and want one consolidated summary.
- You want to extract the main points from meeting minutes / reports / market research.

**Input**
- 1 or more files: PDF, DOCX, PPTX, TXT.

**Output**
- `summary`: a 3–5 sentence paragraph.
- `key_points`: 5–8 bullet points.
- `truncated`: a flag indicating that the source content was cut because it was too long.

---

## 2. Limits & prerequisites

| Item | Limit |
| --- | --- |
| Formats | PDF, DOCX, PPTX, TXT |
| Size per file | Up to **20 MB** |
| Total input text | **100,000 characters** — anything beyond is cut and flagged as `truncated: true` |
| Files at once | No hard cap, but the combined text must fit within 100K characters |
| Authentication | SSO required |

---

## 3. How to use (step-by-step)

### Step 1 — Open The Summarizer
Go to the **The Summarizer** tab.

![Summarizer interface](./images/summarizer-01-home.png)
*Figure 1: Upload zone on the left, results on the right.*

### Step 2 — Upload one or more files
Drag and drop multiple files at once, or click to add files one by one.

![Multi-file upload](./images/summarizer-02-multi-upload.png)
*Figure 2: The list of added files — each one can be removed before summarizing.*

### Step 3 — Click "Summarize"
The agent reads all the files, concatenates the content, and sends it to the LLM.

### Step 4 — Wait for the result
Roughly 10–30 seconds depending on total size.

### Step 5 — Review the result
- **Executive Summary** at the top: 3–5 sentences.
- **Key Insights** below: 5–8 bullets.

![Summary result](./images/summarizer-03-result.png)
*Figure 3: Executive summary + key bullet points.*

### Step 6 — Copy or add more files
- Click Copy to paste into email/chat.
- Or add a new file → click Summarize again for an expanded combined summary.

---

## 4. Walkthroughs (User Journeys)

### Journey 1 — Summarize a 30-page meeting minute

**Context:** `Board_Meeting_May.docx` is 30 pages long. You need to send a recap to teammates who didn't attend.

**Steps:**
1. Open the Summarizer → drag in `Board_Meeting_May.docx`.
2. Click Summarize → wait ~15 seconds.
3. You receive:
   - **Summary:** *"The meeting focused on the Q3 budget, the decision to postpone project X to Q4, and approval for hiring 5 new positions for the Data team…"*
   - **Key insights:** 7 bullets — each with specific numbers, owners, and deadlines.
4. Copy → paste into an email to the team.

![Meeting minutes summary](./images/summarizer-scn1-meeting.png)
*Figure: Result with an executive summary and bullets prioritizing decisions and deadlines.*

---

### Journey 2 — Consolidate multiple reports into one summary

**Context:** You have 4 market research reports from 4 different firms and want a consolidated set of key takeaways.

**Steps:**
1. Upload all 4 files: `Gartner.pdf`, `Forrester.pdf`, `IDC.pdf`, `McKinsey.docx`.
2. Click Summarize.
3. The agent concatenates them (joined by a `--- filename ---` separator) and produces a unified summary.
4. Result: a single executive summary comparing the four firms' views, and 8 bullets contrasting agreements and disagreements.

**Note:** Combined, the 4 files are close to 120K characters → the agent flags `truncated: true`. The tail of `McKinsey.docx` may not be summarized — see the tip in section 5.

---

### Journey 3 — Summarize a partner sales deck

**Context:** `Partner_Deck.pptx` has 40 slides and you want to know the partner's pitch before the meeting.

**Steps:**
1. Upload the PPTX.
2. Summarize.
3. You get: 5 sentences summarizing the partner's proposed solution + bullets on pricing, timeline, and risks.

---

## 5. Usage tips

- **Direct the focus by adding a context file**: if you upload a `prompt.txt` containing `"Focus on financial risks and Q4 deadlines"`, the agent will prioritize those areas.
- **Avoid going over 100K characters**: if many files are large, drop appendices/annexes before uploading.
- **`truncated: true`** = content was cut — break it down and summarize in parts, then upload those partial summaries for a final consolidated pass.
- **The agent does not "invent" content** — if a point you remember is missing, it may live in the truncated section.
- **Meeting minute summaries** come out well if the file has clear structure (titles, speakers, action items).

---

## 6. Known limitations

- **Hard cap at 100,000 characters** combined — there is no UI control to raise it.
- **No file output** — the result is web-only text; you must copy it manually.
- **No history is saved** — closing the tab loses the result; save it before leaving.
- **No source attribution**: when multiple files are uploaded, the summary does not say which insight came from which file (unless you explicitly ask).
- **No "summary length" mode** — it's always 3–5 sentences + 5–8 bullets. For shorter/longer, edit manually or use Brainstormer/Hub to expand.
- **Images in documents are not processed** — charts/diagrams are ignored; only text is read.
