# The Brainstormer — Brainstorming partner with a live canvas

> The AI asks Socratic questions across 4 stages. A visual canvas updates in real time: problem → options → action plan.

---

## 1. Capabilities overview

**What does The Brainstormer do?** It plays the role of a thinking partner. Instead of handing you an answer, the AI asks Socratic questions so you can dig in yourself — while a **live canvas** (problem / options / action plan) is updated turn by turn.

**When to use it**
- You have a fuzzy problem and want to "think it through" rather than ask for the answer.
- You want to compare 2–3 directions with clear trade-offs.
- You need a structured action plan by the end of the session.

**Input**
- A text message describing the initial problem.
- Your responses to the AI's follow-up questions during the session.

**Output**
- Text replies in the chat.
- A **canvas** on the right, updated each turn:
  - `problem`: the real problem statement (one sentence).
  - `options`: 2–3 options with trade-offs.
  - `action_plan`: concrete steps (appears at the end).
- Sessions are saved → you can return and continue later.

---

## 2. Limits & prerequisites

| Item | Limit |
| --- | --- |
| Session structure | 4 fixed stages: Understand → Explore → Converge → Wrap up |
| Turn count | The AI **forces a wrap-up and an action plan** on **turn 8** |
| History compaction | Beyond 10 messages, older turns are summarized by Haiku |
| Preferences | The AI remembers language / tone / topic **across sessions** |
| Authentication | SSO required |

---

## 3. How to use (step-by-step)

### Step 1 — Open The Brainstormer
A 2-panel UI: chat on the left, canvas on the right.

![Main interface](./images/brainstormer-01-home.png)
*Figure 1: Chat left + canvas right. The canvas starts empty.*

### Step 2 — State the problem
Example: `"I want to improve the onboarding experience for new customers"`.

### Step 3 — Answer the AI's questions (Stage 1: Understand)
**Turns 1–3:** the AI asks questions to surface the real goal:
- *"What would change if you solved this?"*
- *"Who feels this problem most acutely?"*

Current canvas: `problem: null` (not yet locked in).

![Understand stage](./images/brainstormer-02-understand.png)
*Figure 2: The AI asks Socratic questions while the canvas remains empty.*

### Step 4 — Explore options (Stage 2: Explore)
**Turns 3–6:** the AI proposes 2–3 directions with trade-offs.

The canvas updates:
```
problem: "High abandonment in week 1 due to a lengthy KYC flow"
options:
  - "Digitize KYC via video eKYC"
  - "Concierge onboarding for VIP customers"
  - "Lightweight self-service checklist"
```

![Canvas with options](./images/brainstormer-03-explore.png)
*Figure 3: Canvas updated with the problem + 3 options.*

### Step 5 — Converge (Stage 3: Converge)
**Turns 6–8:** the AI helps evaluate options against the goal — you eliminate some and refine the rest.

### Step 6 — Wrap up with an action plan (Stage 4: Wrap up)
**Turn 8 onward:** the AI commits to a concrete action plan.

The completed canvas:
```
problem: "..."
options: ["...", "..."]
action_plan:
  - "Week 1: Map the current KYC flow"
  - "Week 2: PoC eKYC with vendor X"
  - "Week 3: Test with 50 users"
```

The session is marked `completed: true`.

![Completed action plan](./images/brainstormer-04-actionplan.png)
*Figure 4: Final canvas with problem, options, and action_plan.*

### Step 7 — Save / resume later

- Sessions auto-save — closing the tab doesn't lose anything.
- The left sidebar lists past sessions → click to restore the full chat + canvas.

![Session sidebar](./images/brainstormer-05-sessions.png)
*Figure 5: Session history for reopening later.*

---

## 4. Walkthroughs (User Journeys)

### Journey 1 — Tackle a vague problem

**Context:** You feel "the team is slowing down" but aren't sure why.

**Steps:**
1. Open the Brainstormer → `"My team is slowing down and I want to improve it but don't know where to start"`.
2. AI asks: *"Slow compared to what baseline? Any concrete metric?"* → you think and answer `"velocity is down 30% vs. last quarter"`.
3. AI follows up about suspected causes → you say `"maybe morale, or tech debt"`.
4. Turns 4–5: the AI proposes 3 directions — fix morale (1:1 + offsite), reduce tech debt (one dedicated sprint), split the team into 2 smaller squads.
5. Turns 6–7: the AI helps evaluate them along cost/impact.
6. Turn 8: the AI produces the action plan: week 1 run a morale survey + tech-debt audit → decide the approach in week 2.

---

### Journey 2 — Compare two clear options

**Context:** You're torn between `"build in-house vs. buy SaaS"` for an analytics tool.

**Steps:**
1. Enter the Brainstormer → describe both options and the budget.
2. AI asks: *"Is the primary goal time-to-market or long-term total cost?"*.
3. You answer → the AI broadens the option set (adding `"build a wrapper around SaaS"`).
4. Converge: the AI maps cost / time / risk / strategic fit for each option.
5. Action plan: 1-week SaaS PoC running in parallel with an internal team capability assessment.

---

### Journey 3 — Resume an older session

**Context:** Last week you brainstormed "AI Adoption strategy" but didn't finish. You want to continue today.

**Steps:**
1. Open the Brainstormer → pick session `"AI Adoption — May 14"` from the sidebar.
2. The full chat + canvas are restored.
3. Type: `"I checked with the Sales team — they support option 2. Let's continue from there"`.
4. The AI remembers context, doesn't re-ask from scratch, and moves on to refining the action plan.

---

## 5. Usage tips

- **Start with the problem, not the solution**: `"I want to improve onboarding"` works better than `"Should I use eKYC?"` — the AI will push back if you open with a solution.
- **Actually answer the AI's questions**: it isn't stalling — short one-line answers are fine as long as they are specific.
- **Don't rush past the Understand stage**: the first 2–3 turns determine the quality of the options later.
- **Watch the canvas**: if `problem: null` is still there after 3 turns, the AI thinks the problem isn't clear yet — don't force it to skip ahead.
- **Lean on cross-session memory**: the AI remembers you prefer Vietnamese responses / a formal tone / fintech topics — you don't need to repeat preferences each session.
- **Want to wrap up early?**: type `"Give me the action plan now"` at any turn after turn 5 — the AI will synthesize using what it has.

---

## 6. Known limitations

- **Forced wrap-up at turn 8**: if the problem really needs more than 8 turns, the AI will force an action plan despite incomplete convergence. Workaround: open a fresh session for the next stage.
- **Canvas is not free-form**: only 3 fixed fields (problem / options / action_plan) — no matrices, mindmaps, or comparison tables.
- **No document reference**: the Brainstormer does not read files. If you need data context, use [Vaulter](./05_vaulter.md) alongside.
- **Sessions are not shareable**: only the owner can see them — no collaborative brainstorming.
- **Compaction can lose nuance**: beyond 10 messages, history is summarized — subtle details may be lost when you resume after many turns.
- **No file export**: the action plan only lives on the canvas. Copy it manually into a doc/sheet to track.
