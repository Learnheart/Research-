# The Resume Evaluator — Candidate CV evaluation

> Score CVs against a JD across 6 criteria, issue an INTERVIEW / CONDITIONAL / DO_NOT_INTERVIEW decision, and suggest interview questions. Supports batch evaluation up to 10 candidates with ranking.

---

## 1. Capabilities overview

**What does The Resume Evaluator do?** It helps HR / Hiring Managers **screen CVs consistently** with AI:
- Scores against 6 weighted criteria.
- Issues a decision: should you interview or not?
- Detects red flags, culture fit, and signals that the CV was AI-written.
- Suggests interview questions + reference-check prompts.
- Exports a per-candidate DOCX report.

**When to use it**
- You received 5–10 CVs for one role and need fast screening.
- You want objective evaluation to reduce bias.
- You need a short report for the hiring committee.

**Input**
- 1 or more CVs (PDF, DOCX, TXT) — up to 10 per batch.
- Job description (text, pasted or typed).

**Output**
- **Score card**: decision + overall score + confidence.
- **Radar chart** across 6 criteria.
- **Detailed breakdown**: red flags, culture fit, authenticity, interview questions, reference-check prompts.
- **DOCX report** downloadable per candidate.
- **Batch ranking**: if you evaluate several CVs, you get a ranking table + skill-gap analysis.

---

## 2. Limits & prerequisites

| Item | Limit |
| --- | --- |
| CV formats | PDF, DOCX, TXT |
| CVs per batch | Up to **10** candidates |
| CV & JD length | Each up to **50,000 characters** (truncated beyond that) |
| Batch processing | **Sequential**, not parallel — 10 CVs take significant time |
| Report | **DOCX** format (not PDF, despite the function name) |
| Cache | None — re-evaluating the same CV runs from scratch |
| Authentication | SSO required |

---

## 3. How to use (step-by-step)

### A. Single evaluation (1 CV)

#### Step 1 — Open The Resume Evaluator
![Main interface](./images/resume-01-home.png)
*Figure 1: JD on the left panel, CV upload on the right.*

#### Step 2 — Enter the Job Description
Paste the JD into the left box. You can include requirements and responsibilities.

#### Step 3 — Upload the CV
Drag and drop one file (PDF/DOCX/TXT).

#### Step 4 — Click "Evaluate"
The agent parses the CV and sends both CV + JD to the LLM with a dedicated evaluation prompt. Takes ~20–40 seconds.

#### Step 5 — Review the score card
- **Decision**: `INTERVIEW` / `CONDITIONAL` / `DO_NOT_INTERVIEW`.
- **Score**: 0–100.
- **Confidence**: High / Medium / Low.
- **Title match**: Direct / Adjacent / Off-title.

![Score card](./images/resume-02-scorecard.png)
*Figure 2: Headline score card with decision + score + confidence.*

#### Step 6 — Review the detailed breakdown
- **Radar chart across 6 criteria**.
- **Red flags** with severity (High / Medium / Low).
- **Culture fit assessment**.
- **Authenticity** (probability the CV was AI-written).
- **Suggested interview questions**.
- **Reference-check prompts**.

![Detailed breakdown](./images/resume-03-breakdown.png)
*Figure 3: Radar chart + the deeper evaluation blocks.*

#### Step 7 — Download Report
Click "Download Report" → grab the `.docx` report for that candidate.

### B. Batch evaluation (multiple CVs)

#### Step 1 — Paste the JD as before
#### Step 2 — Upload multiple CVs (up to 10)

![Batch upload](./images/resume-04-batch-upload.png)
*Figure 4: List of 10 CVs ready to evaluate.*

#### Step 3 — Click "Evaluate All"
The agent processes them **sequentially**. The progress bar shows: `Evaluating 4/10 — Nguyen Van A`.

#### Step 4 — View the ranking table
Candidates are ranked by overall score (descending), each with 1 key strength + 1 key concern.

![Ranking](./images/resume-05-ranking.png)
*Figure 5: Ranking table for 10 candidates.*

#### Step 5 — Skill-gap analysis
A block lists the JD requirements that **no** candidate satisfied — useful for deciding whether to widen the search or adjust the JD.

#### Step 6 — Download
- Individual reports (one DOCX per candidate).
- Or ZIP everything + a comparison summary.

---

## 4. Walkthroughs (User Journeys)

### Journey 1 — Quick screen of 1 CV before a phone interview

**Context:** A recruiter sent `Nguyen_Van_A.pdf` for a Senior Data Engineer role. You want a 2-minute screen before the call.

**Steps:**
1. Open the Resume Evaluator → paste the JD `"Senior Data Engineer — 5+ years Spark/Databricks, Python..."`.
2. Upload `Nguyen_Van_A.pdf`.
3. Click Evaluate → wait 30 seconds.
4. Result: **Decision: INTERVIEW**, Score: 82, Confidence: High, Title match: Direct.
5. Radar: Skills 85%, Experience 90%, Achievement 75%, Transferability 70%, Online 60%, Education 80%.
6. Red flags: 1 medium — `"Job hopping — 3 jobs in last 3 years"`.
7. Suggested questions: 5 prompts; copy 2 into your phone-screen script.
8. Make the call with the context already prepared.

---

### Journey 2 — Batch screen 8 candidates

**Context:** 8 CVs for a Product Manager role. You need to shortlist 3 for on-site interviews.

**Steps:**
1. Paste the JD.
2. Upload all 8 CVs.
3. Click Evaluate All → wait ~5 minutes (sequential).
4. The ranking:
   - #1: 88 — INTERVIEW (Direct match, strong product cases)
   - #2: 81 — INTERVIEW (Adjacent — fintech background)
   - #3: 76 — INTERVIEW
   - #4–6: 60–72 — CONDITIONAL
   - #7–8: <60 — DO_NOT_INTERVIEW
5. Skill-gap analysis: `"None of the candidates have B2B SaaS experience"` → consider widening the JD or searching another pool.
6. Download the top-3 ZIP report.
7. Send to the hiring committee with the comparison summary.

---

### Journey 3 — Detect an AI-written CV

**Context:** A CV reads too polished — you suspect ChatGPT was used.

**Steps:**
1. Evaluate the CV.
2. Open the **Authenticity Assessment** block:
   - AI-generation likelihood: **High (78%)**.
   - Specificity: Low — `"Most achievements are generic phrases with few concrete numbers"`.
   - Buzzword density: High.
   - Internal consistency: dates don't line up.
3. Decision: still interview, but use the suggested "tell me about a specific time..." questions to verify.

---

## 5. Usage tips

- **The clearer the JD, the more accurate the evaluation**: paste the full JD (requirements + nice-to-have + responsibilities), not just the title.
- **The 6 criteria use fixed weights** (Skills 25%, Experience 20%, Achievement 20%, Education 15%, Transferability 10%, Online Presence 10%) — no UI customization.
- **Score < 60 = DO_NOT_INTERVIEW**: but don't auto-reject — read the breakdown to make sure the agent didn't miss context (e.g., a short CV because the candidate is junior, not unqualified).
- **High AI-generation likelihood is NOT auto-reject** — it's only a signal to verify during the interview.
- **High Transferability with a non-direct title** is worth a look — the agent evaluates conceptual overlap.
- **Batch processing is sequential** — 10 CVs may take ~5 minutes. Don't refresh the page.
- **The DOCX report is editable** in Word — paste HR/recruiter comments before sending to the hiring committee.
- **Re-evaluating the same CV produces nearly identical scores** but not fully deterministic — for strict consistency, save the first report instead of re-running.

---

## 6. Known limitations

- **Up to 10 CVs per batch, sequential**: large batches (50+) must be split into multiple runs.
- **CVs are truncated at 50,000 characters**: normal CVs are fine, but a senior portfolio CV may be cut.
- **Reports are DOCX, not PDF**: use Word's Export-as-PDF if PDF is required.
- **No cache**: re-evaluating the same CV consumes new LLM calls.
- **No formal bias metric**: the agent tries to reduce bias but has no official audit / fairness metric.
- **No video resume / portfolio website support**: only text inside the CV file is read.
- **The 6-criteria weights are fixed**: not customizable per role (e.g., sales roles arguably should weight Achievement higher — not yet supported).
- **Scores may drift slightly across runs** (the LLM is not 100% deterministic) — do not use as a legal criterion.
- **No ATS integration** — upload manually.
