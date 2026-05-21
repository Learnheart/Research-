# The Powerpoint-er — Generate branded slides

> Describe a topic → AI drafts the outline → preview as HTML → download the `.pptx`. A 3-step pipeline with content editing and natural-language AI edits.

---

## 1. Capabilities overview

**What does The Powerpoint-er do?** It quickly produces a branded slide deck from a short description — no design skills required.

**When to use it**
- You need a draft deck in 5–10 minutes before a meeting.
- You want slides that follow corporate branding (red palette, logo, footer) without building a template.
- You want to turn an existing outline into a polished deck.

**Input**
- Topic.
- Audience.
- Desired slide count (5–15).
- Optional: enable/disable the branded template frame.

**Output**
- A `.pptx` file, 13.33" × 7.5" (16:9), downloaded to your machine.
- An outline JSON you can preview on the web before building.

**Supported slide types:**
- **Cover** — title slide with branded background.
- **Section** — divider-style section break.
- **Content** — 3 variants: `data` (KPIs/numbers), `concept` (principles), `process` (timeline/workflow).
- **Summary** — closing slide with takeaways.

---

## 2. Limits & prerequisites

| Item | Limit |
| --- | --- |
| Slide count | **5–15** per deck |
| Topic | Must pass the input guardrail (Haiku-based classifier) |
| Output guardrail | Applied — inappropriate content is blocked |
| **Post-download editability** | **The exported PPTX is image-based — text/shapes CANNOT be edited in PowerPoint** |
| Build timeout | 5 minutes (full deck), 60 seconds/slide when regenerating |
| Authentication | SSO required |

> ⚠️ **Important note:** Each slide is rendered to a PNG and embedded in the PPTX. In PowerPoint, you'll only see images — text cannot be edited directly. All edits must be done **on the web before downloading**.

---

## 3. How to use (step-by-step)

The 3-step flow: **Brief → Outline → Preview**.

### Step 1 — Brief

- Enter the **topic**: e.g., `"AI strategy 2026"`.
- Enter the **audience**: e.g., `"Board of Directors"`.
- Pick the **slide count**: 5–15.
- Toggle **Use template** (typically left on).

![Brief form](./images/powerpointer-01-brief.png)
*Figure 1: Form for entering topic, audience, slide count.*

### Step 2 — Outline

- Click "Generate Outline" → AI produces an outline JSON: title + bullets per slide.
- You can:
  - Edit titles/bullets in place.
  - Add / remove / reorder slides.
  - Change the slide type (cover, section, content, summary).
  - Switch the content variant (data / concept / process).
  - Click "Preview this slide" to render any individual slide.

![Outline editor](./images/powerpointer-02-outline.png)
*Figure 2: The outline can be edited directly before building.*

### Step 3 — Preview

- Click "Build" → all slides render in parallel as HTML.
- A grid shows every slide as a thumbnail.
- Two editing modes per slide:
  - **Rich Editor** (visual): bold, italic, underline, color, font size, alignment, bullets.
  - **AI Edit** (natural language): type `"Convert to a 4-step timeline"` → the AI rewrites the slide.
- **Regenerate**: rebuild the slide from the same outline but a different version.
- **Presentation Mode**: full-screen view, use arrow keys to navigate.

![Preview grid + AI edit](./images/powerpointer-03-preview.png)
*Figure 3: Slide preview grid; the edit modal has Rich Editor and AI Edit tabs.*

### Step 4 — Export

- Click "Export to PPTX".
- The frontend captures each slide as PNG and packages everything into a `.pptx`.
- Links/URLs inside slides remain clickable (hyperlink overlay).

![Export PPTX](./images/powerpointer-04-export.png)
*Figure 4: Export button — the file downloads to your machine immediately.*

---

## 4. Walkthroughs (User Journeys)

### Journey 1 — A 10-slide strategy draft in 10 minutes

**Context:** Your boss asks at 2pm `"Do you have anything to present on AI strategy 2026?"`. The meeting is at 3pm.

**Steps:**
1. Open the Powerpoint-er.
2. Brief:
   - Topic: `"AI strategy 2026 — applications in banking operations"`.
   - Audience: `"Executive board"`.
   - Slide count: `10`.
   - Use template: ✅
3. Click Generate Outline → ~20 seconds.
   ![Strategy deck outline](./images/powerpointer-scn1-outline.png)
   *Figure: 10-slide outline — cover, 2 sections, 6 content slides (mix of data/concept/process), 1 summary.*
4. Review the outline, switch slide 5 from `concept` to `data` (you want hard KPIs) → regenerate just slide 5.
5. Click Build → ~2 minutes for the full deck.
6. Preview, use AI Edit on the cover: `"Change the title to 'AI 2026: From Pilot to Production'"`.
7. Export → download `AI_strategy_2026.pptx`.

---

### Journey 2 — Build slides from an existing outline

**Context:** You already have an 8-slide outline written by hand and want AI to do the visuals quickly.

**Steps:**
1. Brief with a generic topic (the AI's outline will be overwritten by yours).
2. Generate Outline → edit each slide to match yours:
   - Slide 1: cover — `"Q1 Performance Review"`
   - Slide 2: section — `"Highlights"`
   - Slide 3: content/data — `"Revenue grew 15% QoQ"` + KPI bullets
   - ...
3. Build → preview.
4. Use AI Edit on any slide that isn't quite right — e.g., `"Add a comparison column with last year's Q4"`.
5. Export.

---

### Journey 3 — Restyle an entire presentation

**Context:** You've built 12 slides but the deck feels too data-heavy — you want more concept-style slides.

**Steps:**
1. Go back to Outline → switch slides 4, 6, 9 from `data` to `concept`.
2. Regenerate just those 3 slides (no need to rebuild everything).
3. Re-preview — the content has been rewritten in a concept/prose style.
4. Export.

---

## 5. Usage tips

- **The more detailed the brief, the better the outline**: include context like `"company is a Vietnamese commercial bank, target audience knows AI basics"` right in the topic.
- **Variants matter**:
  - `data` for KPIs, charts, numerical comparisons.
  - `concept` for principles, frameworks, prose.
  - `process` for timelines, workflows, step-by-step.
- **Use short AI Edit commands**: `"Convert to a 4-quadrant layout"`, `"Add icons"`, `"Drop this slide"` — clearer than long, complex sentences.
- **Edit everything on the web BEFORE exporting** — the PPTX export is not editable.
- **Need to edit natively in PowerPoint?** Not possible today — this is a known limitation; the team is exploring solutions.
- **Regenerate vs. Edit**: regenerate rebuilds from scratch (loses custom content); edit fine-tunes. Be careful regenerating slides you've hand-tuned.
- **Presentation Mode** in the web app can replace PowerPoint for live demos — exporting is not always necessary.

---

## 6. Known limitations

- **PPTX is image-based**: text/shapes are not editable after download. The trade-off is the rich web editor.
- **Slide count is capped at 5–15**: larger decks must be built in multiple passes (and merged manually).
- **The guardrail may block sensitive topics**: if you're rejected at the outline step (HTTP 400), try rephrasing the topic.
- **Web search disabled in UI**: the backend supports `use_web_search`, but the UI does not expose it — slides rely on the LLM's internal knowledge, not real-time data.
- **No custom templates yet**: every deck uses the fixed branded template — no user-uploaded template support.
- **Slides fall back on LLM errors**: if a slide hits an LLM failure, the agent uses `fallback_html()` (plain text + bullets, no visuals) — regenerate that slide.
- **No custom fonts/languages**: fonts follow the predefined brand tokens.
