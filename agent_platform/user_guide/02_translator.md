# The Translator — Translate documents while preserving layout

> Translate PDF, DOCX, PPTX, TXT files between any two languages without breaking the layout — layout, fonts, tables, and slides are all preserved.

---

## 1. Capabilities overview

**What does The Translator do?** It translates documents between any two languages and **returns a file in the same format**, with a near-original layout.

**When to use it**
- You have a report, proposal, or deck that needs translation but you don't want to redo the formatting.
- You need to quickly translate many slides (PPTX) or tables (DOCX).
- You're unsure of the source language — the agent auto-detects it.

**Input**
- 1 file: PDF, DOCX, PPTX, or TXT.
- Target language (default: Vietnamese).
- Source language (default: auto-detect).

**Output**
- A translated file in the same format as the input — **except PDF**, which is returned as DOCX.
- For TXT: text is shown directly in the web app with a Copy button.
- Real-time progress: number of chunks translated / total chunks + estimated time remaining.

---

## 2. Limits & prerequisites

| Item | Limit |
| --- | --- |
| Supported formats | PDF, DOCX, PPTX, TXT |
| File size | Up to **20 MB** |
| TXT | Truncated at **12,000 characters** — anything beyond is not translated |
| PDF | Output is **DOCX** (no PDF returned) |
| Client timeout | 5 minutes — very large files may be cut off at the UI layer |
| Authentication | SSO required |

---

## 3. How to use (step-by-step)

### Step 1 — Open The Translator
Open the **The Translator** tab in Agent Studio.

![Main interface](./images/translator-01-home.png)
*Figure 1: The left panel is the upload zone; the right panel shows results/progress.*

### Step 2 — Upload a file
Drag and drop the file into the left panel, or click to browse.

![Upload zone](./images/translator-02-upload.png)
*Figure 2: The drag-and-drop zone in the left panel.*

### Step 3 — Choose the target language
Open the "Translate to" dropdown and pick the language. Default is Vietnamese.

![Language picker](./images/translator-03-language.png)
*Figure 3: The target language dropdown.*

### Step 4 — Click "Translate to {language}"
The process has 4 stages:
1. **Upload** — file is sent to the Volume.
2. **Analyzing** — backend parses & validates the file.
3. **Translating** — parallel translation, showing `chunk X / Y` + estimated time.
4. **Building** — assembling the output file.

![Translation progress](./images/translator-04-progress.png)
*Figure 4: Progress bar and real-time status messages.*

### Step 5 — Download the result
- **PPTX / DOCX / PDF (→DOCX):** a "Download" button appears in the right panel.
- **TXT:** the translated text is shown inline with a Copy button.

![Download button](./images/translator-05-download.png)
*Figure 5: The button to download the result file.*

---

## 4. Walkthroughs (User Journeys)

### Journey 1 — Translate a customer deck from English to Vietnamese

**Context:** You received `Vendor_Pitch.pptx` (25 slides) from a partner and need to translate it for an executive review.

**Steps:**
1. Open the Translator → drag and drop `Vendor_Pitch.pptx`.
2. Pick the target language: **Vietnamese**.
3. Click "Translate to Vietnamese".
4. Watch the progress: `chunk 5 / 25 — ~3 minutes left`.
   ![PPTX translation](./images/translator-scn1-pptx.png)
   *Figure: Slide-by-slide progress.*
5. When complete, click Download → receive `Vendor_Pitch_VN.pptx`.

**What you'll notice:** text inside tables, grouped shapes, headers/footers are all translated while keeping the original font, color, and position.

---

### Journey 2 — Translate a long PDF contract

**Context:** A 40-page contract `Contract_2026.pdf` needs to be translated into English.

**Steps:**
1. Upload the PDF (~8 MB).
2. Pick the target language: **English**.
3. Click Translate. The agent internally converts PDF → DOCX, then translates.
4. Download `Contract_2026_translated.docx`.

**Warning:** The output is DOCX, not PDF. If you need a PDF at the end, use Word's Export-as-PDF after reviewing.

---

### Journey 3 — Translate a short TXT note

**Context:** You copied 2,000 characters of internal notes into `notes.txt` and need a quick English translation.

**Steps:**
1. Upload `notes.txt`.
2. Target language: **English**.
3. Click Translate — done in ~5 seconds (single round-trip).
4. The translation is displayed inline → click "Copy" to paste it elsewhere.

![Inline TXT result](./images/translator-scn3-txt.png)
*Figure: Translated text shown inline with a Copy button.*

---

## 5. Usage tips

- **Prefer PPTX/DOCX when possible** — formatting is preserved better than with PDF.
- **PDFs with scanned images** cannot be translated — the agent only reads the text layer. Run OCR first if needed.
- **Files >20MB**: split them into smaller pieces (by chapter / slide group) and translate each.
- **Specialized terminology**: no glossary support yet — review the translation to fix financial/legal terms after the fact.
- **TXT over 12,000 characters**: split into multiple files or paste into a DOCX (DOCX has no strict character cap).
- **If the browser times out after 5 minutes**: the backend keeps processing — wait a bit, then refresh the page to fetch the result.

---

## 6. Known limitations

- **PDF → DOCX**: there is no way to keep the output as PDF; you must re-export from Word if PDF is required.
- **TXT is truncated at 12,000 characters** — content beyond that is dropped.
- **No glossary / domain dictionary** — financial terminology may be inconsistent.
- **Maximum 5 concurrent workers** — very large PPTX files (50+ slides) are still processed in batches.
- **Scanned PDFs / images** cannot be translated (no built-in OCR).
- **No email delivery** — you must download and share the file yourself.
