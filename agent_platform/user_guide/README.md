# Agent Studio — User Guide

> End-user documentation for the 9 agents in Agent Studio. This guide is written for web app users — no technical knowledge required.

---

## What do you need to do? → Pick the right agent

| You want to… | Best agent | Typical input | Typical output |
| --- | --- | --- | --- |
| Have a single chat that picks the right tool for you | **Hub** | Text question/request + file (if needed) | Text response + result document |
| Translate a document while preserving its format | **The Translator** | PDF / DOCX / PPTX / TXT file | Translated file (same format) |
| Summarize one or many long documents | **The Summarizer** | 1–N files | 3–5 sentence summary + 5–8 key points |
| Generate a PowerPoint deck quickly | **The Powerpoint-er** | Topic + audience + slide count | Branded `.pptx` file |
| Save documents into a personal knowledge base for Q&A | **The Vaulter** | Documents (uploaded over time) | Knowledge graph + Q&A |
| Brainstorm ideas / solutions | **The Brainstormer** | A problem/topic you want to think through | Canvas: problem → options → action plan |
| Ask questions about the internal AI strategy | **The AI Visionary** | Questions about AI direction | Response from the Knowledge Agent + PPTX report |
| Generate designed images / SVG / DOCX | **The Canvas Designer** | Description of the desired visual | PNG / JPG / SVG / DOCX file |
| Evaluate candidate CVs against a JD | **The Resume Evaluator** | CV + Job Description | 6-criteria scorecard + interview recommendations |

---

## Detailed guides

1. [Hub — The unified AI assistant](./01_hub.md)
2. [The Translator — Document translation](./02_translator.md)
3. [The Summarizer — Document summarization](./03_summarizer.md)
4. [The Powerpoint-er — Slide generation](./04_powerpointer.md)
5. [The Vaulter — Personal knowledge base](./05_vaulter.md)
6. [The Brainstormer — Brainstorming partner](./06_brainstormer.md)
7. [The AI Visionary — AI strategy Q&A](./07_visionary.md)
8. [The Canvas Designer — Image & graphics generation](./08_canvas_designer.md)
9. [The Resume Evaluator — CV evaluation](./09_resume_evaluator.md)

---

## Structure of each guide

Every agent guide follows the same 6-section structure:

1. **Capabilities overview** — What the agent does, when to use it, Input/Output.
2. **Limits & prerequisites** — File types, size limits, access, prerequisites.
3. **How to use (step-by-step)** — Order of actions in the web app.
4. **Walkthroughs (User Journeys)** — 2–3 end-to-end scenarios with placeholders for screenshots.
5. **Usage tips** — Tips for getting better results.
6. **Known limitations** — What the agent cannot do (yet) or does not do well.

---

## Image conventions

- Screenshots are stored under `./images/<agent>-<seq>-<description>.png`.
- A short italic caption appears below each image.
- When the UX team produces screenshots, simply drop the file into the correct path — markdown will render automatically.

## Access & support

- All agents authenticate via the **organization SSO** — no separate account is required.
- Each agent has its own URL (Databricks App). Hub is the recommended entry point for new users.
- For technical issues or feature requests, contact the Agent Studio team.

---

*This document describes Agent Studio at the time of release. When an agent receives a major update (new tool, UI change), the corresponding file will be revised.*
