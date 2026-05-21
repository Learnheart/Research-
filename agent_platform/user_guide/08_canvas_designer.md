# The Canvas Designer — Generate images & graphics from a description

> Produce visuals (PNG/JPG), SVG, or designed DOCX from a single description. Images are rendered programmatically — with glow, gradients, and particles across 5 built-in palettes.

---

## 1. Capabilities overview

**What does The Canvas Designer do?** It turns a text description into a **designed image or document**:
- PNG / JPG: stylized art with glow / gradient effects.
- SVG: SVG code you can embed in web pages or slides.
- DOCX: a Word document with structure and formatting (headings, bullets, layout).

**When to use it**
- You need a visual for a slide / report but have no designer.
- You need an infographic, icon, or cover image for a document.
- You need a well-structured Word file from an outline.

**Input**
- A text description.
- Output format: **PNG / JPG / SVG / DOCX**.
- Style: `creative` (default) or other styles depending on the release.
- Dimensions (default: 1200×1200 for PNG/JPG/SVG).

**Output**
- The corresponding file is downloaded (stored in the Volume).
- A preview is shown directly in the web app before download.

---

## 2. Limits & prerequisites

| Item | Limit |
| --- | --- |
| Output formats | PNG, JPG, SVG, DOCX |
| Default size | 1200×1200 (customizable) |
| LLM model | Opus for PNG/JPG (higher quality), Sonnet for SVG/DOCX |
| Storage | Saved to `/canvas/{user}/{date}/canvas-{id}.{ext}` |
| Finding older files | Download only searches **today's files** — files from other days may not be retrievable |
| Authentication | SSO required |

---

## 3. How to use (step-by-step)

### Step 1 — Open The Canvas Designer
![Interface](./images/canvas-01-home.png)
*Figure 1: Description panel on the left, result preview on the right.*

### Step 2 — Describe the image
Examples: `"A futuristic robot with glowing neon eyes, cyberpunk style"`, `"A stylized growth chart shaped like a rocket"`, `"Cover image for an AI agent deck — tech aesthetic, lots of blue light"`.

![Enter the description](./images/canvas-02-prompt.png)
*Figure 2: Description box + format/size pickers.*

### Step 3 — Pick the output format
- **PNG** (default): high-quality image with transparency support.
- **JPG**: lighter compressed image, no transparency.
- **SVG**: vector — scales without loss; great for icons/logos.
- **DOCX**: a Word document (your description should describe the document structure, not an image).

### Step 4 — (Optional) Adjust dimensions
Default is 1200×1200. Increase for banners / posters.

### Step 5 — Click "Generate"
- For PNG/JPG: Opus produces a JSON spec → Pillow renders it. Takes ~15–30 seconds.
- For SVG: the LLM returns raw SVG code (faster).
- For DOCX: the LLM returns a JSON spec → python-docx renders it.

![Loading state](./images/canvas-03-generating.png)
*Figure 3: Spinner during generation.*

### Step 6 — Preview the result
The image appears in the right panel.

![Result preview](./images/canvas-04-preview.png)
*Figure 4: The generated image is shown directly in the web app.*

### Step 7 — Download or regenerate
- Click **Download** → save the file to your machine.
- Adjust the description → click Generate again if you're not satisfied.

---

## 4. Walkthroughs (User Journeys)

### Journey 1 — Cover image for a slide

**Context:** You need a cover image for the deck `"AI Agent Platform 2026"`.

**Steps:**
1. Open the Canvas Designer.
2. Describe: `"Cover image — abstract AI agents connected as a network, glowing blue nodes on dark background, cyberpunk aesthetic"`.
3. Pick format: **PNG**, size 1920×1080 (16:9).
4. Generate → preview.
5. Not happy → add: `"... add subtle red accent lines, more depth"` → regenerate.
6. Download → insert into the deck's cover slide in PowerPoint.

![Cover image result](./images/canvas-scn1-cover.png)
*Figure: Network-node cover image with glow.*

---

### Journey 2 — Generate an SVG icon for a web app

**Context:** You need a `"shield with checkmark"` icon for a security dashboard.

**Steps:**
1. Describe: `"Minimalist shield icon with a green checkmark, flat design, clean lines"`.
2. Format: **SVG**, size 256×256.
3. Generate — the LLM returns raw SVG code, the web app renders it.
4. Download `.svg` → embed in your codebase.

**Warning:** LLM-generated SVG may not be optimal (complex paths / unminified). Run it through SVGO for production-grade output.

---

### Journey 3 — Generate a structured DOCX document

**Context:** You need a `"Project Charter"` template with a standard structure.

**Steps:**
1. Format: **DOCX**.
2. Describe: `"Project Charter document with sections: Project Name, Objectives, Scope (in/out), Stakeholders table, Milestones timeline, Risks & Mitigations, Approval signatures"`.
3. Generate → the LLM returns a JSON spec → python-docx renders it.
4. Download `.docx` → open in Word and fill in the real content.

---

## 5. Usage tips

- **More detail in the description → better results**: state the style (`"flat design"`, `"cyberpunk"`, `"corporate minimalist"`), the colors (`"red and navy"`), and the composition (`"centered composition"`).
- **Use the 5 built-in palettes to pick a tone**:
  - `synthetic_optimism` (cyan/pink/dark blue) — futuristic
  - `midnight_aurora` (blue/purple/navy) — premium
  - `forest_tech` (green/dark forest) — sustainability
  - `cosmic_dream` (pink/blue/black) — creative
  - `neon_noir` (neon green/magenta/black) — bold
- **PNG > JPG** for graphics that need transparency; pick **JPG** only if file size matters.
- **SVG is not for photorealism** — only good for icons, logos, geometric graphics.
- **DOCX is not "write the article for me"** — it only produces structure/formatting. Detailed content is up to you, or use the Powerpoint-er/Summarizer for text.
- **Download right away** — older files may not be findable after the same day.
- **Regenerate is full rebuild, not a tweak** — for a subtle style change, edit the description; for a completely different style, regenerate.

---

## 6. Known limitations

- **Not a DALL-E / Midjourney-style image generator**: images are rendered programmatically via Pillow from a JSON spec — great for abstract graphics, infographics, network diagrams, KPI visuals; **not** for realistic portraits, landscapes, or detailed real-world objects.
- **Limited palettes**: 5 built-in palettes — no fully custom color tuning from the user prompt yet.
- **LLM-written SVG** may be sub-optimal — not production-ready without SVGO.
- **Download only finds today's files** — older files may not be retrievable through the UI.
- **No version history**: regenerating overwrites the preview — earlier versions are not saved.
- **No batch / mass generation**: one file at a time.
- **No custom DOCX header/footer / page numbering**: only body content per the spec.
- **No official branded DOCX template**: copy-paste into the corporate template after downloading.
