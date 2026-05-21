# The Vaulter — Personal knowledge base

> Upload documents → AI extracts entities and relations into a knowledge graph → ask questions grounded in your own vault.

---

## 1. Capabilities overview

**What does The Vaulter do?** It turns your collection of documents into a **personal knowledge base**:
- Extracts entities (people, projects, companies, products…) and the relations between them.
- Visualizes them as an interactive knowledge graph.
- Lets you ask natural-language questions — answers are grounded **only** in the documents you've uploaded.

**When to use it**
- You have dozens of project reports and documents and need quick lookups (`"Who runs project X?"`).
- You want to build a "memory" for your own work without re-typing content.
- You need Q&A with a clear source — the LLM won't make things up; if the info isn't in the vault, the agent says it doesn't know.

**Input**
- **Upload mode:** 1 document at a time (PDF, DOCX, TXT).
- **Query mode:** a text question.

**Output**
- **Upload:** confirmation `X nodes, Y edges extracted`.
- **Graph view:** an interactive knowledge graph.
- **Query:** a text answer drawn from the vault.

---

## 2. Limits & prerequisites

| Item | Limit |
| --- | --- |
| Upload formats | PDF, DOCX, TXT |
| File size | Up to **20 MB** |
| Graph extraction | Up to **6,000 characters** from the start of each upload |
| Query context | Only the **5 most recent documents**, up to **10,000 characters** combined |
| Scope | Each user has their own vault — **not shared** across users |
| Delete documents | No delete API yet — accidentally uploaded files will remain in the vault |
| Authentication | SSO required |

---

## 3. How to use (step-by-step)

The Vaulter has 2 main modes — **Upload** and **Query** — plus a **Graph view** for exploration.

### A. Upload mode

#### Step 1 — Open The Vaulter and select the Upload tab
![Upload tab](./images/vaulter-01-upload-tab.png)
*Figure 1: The main interface, with Upload and Query tabs.*

#### Step 2 — Drag and drop the file
Pick a single file (PDF/DOCX/TXT).

#### Step 3 — Click "Upload to Vault"
The backend will:
1. Parse text from the file.
2. Store the content in the vault.
3. Call the LLM to extract entities + relations (capped at the first 6,000 characters).
4. Update the graph (de-duplicating).

![Processing](./images/vaulter-02-uploading.png)
*Figure 2: Upload and extraction progress.*

#### Step 4 — See the result
Message: `"Document_X.pdf added — 12 nodes, 8 edges extracted"`.

### B. Graph view

After uploading, switch to the **Graph** tab to visualize.

![Knowledge graph](./images/vaulter-03-graph.png)
*Figure 3: Interactive knowledge graph — nodes are entities, edges are relations.*

- Hover a node → see details.
- Drag to reposition.
- Zoom in/out to explore clusters.

### C. Query mode

#### Step 1 — Switch to the Query tab
#### Step 2 — Type your question
Examples: `"Who owns the Alpha project?"`, `"What risks does project X have?"`, `"When does project Beta kick off?"`.

![Query box](./images/vaulter-04-query.png)
*Figure 4: Question input + Ask button.*

#### Step 3 — Click "Ask"
The agent reads the 5 most recent documents and answers strictly from that content.

#### Step 4 — Read the answer
![Query result](./images/vaulter-05-answer.png)
*Figure 5: Answer grounded in vault context — if nothing is found, the agent says "I don't have that information."*

---

## 4. Walkthroughs (User Journeys)

### Journey 1 — Build a vault for a long-running project

**Context:** You manage the "AI Transformation" project, running for 6 months. You have ~10 documents: kickoff deck, weekly status, risk register, tech specs, vendor proposal.

**Steps:**
1. Week 1 — upload the first 4 files (kickoff, scope, vendor proposal, plan).
2. Open Graph: you see nodes like `Alpha`, `Vendor X`, `Q3 2026 Milestone`, `John (PM)`, `Privacy Risk`…
3. Week 4 — upload 2 more status reports.
4. End of the month — you're asked `"What risks does the Alpha project face right now?"`.
5. Open the Query tab → ask → the agent answers using the risk register + status reports.

![Vault with 10 documents](./images/vaulter-scn1-rich-graph.png)
*Figure: Graph after 6 weeks — multiple topical clusters.*

---

### Journey 2 — Quick lookup before a meeting

**Context:** 5 minutes before a meeting, your boss asks `"What are Vendor X's commitments in the contract?"`.

**Steps:**
1. Open the Vaulter → Query tab.
2. Type: `"Vendor X commitments in contract?"`.
3. The agent reads the recent files mentioning "Vendor X" within the 5 most-recent documents → returns the list of commitments.
4. Copy the answer, walk into the meeting.

**Warning:** If the contract file was uploaded long ago (position 6+ by recency), it is **not** in the query context — re-upload it, or upload a summary.

---

### Journey 3 — Find connections across projects

**Context:** You suspect projects A and B overlap on stakeholders/tech stack.

**Steps:**
1. Open the Graph view.
2. Find the `Project A` and `Project B` nodes.
3. Inspect shared edges — e.g., both link to `John (Engineering Lead)`, both use `Databricks`, both have `Data Privacy Risk`.
4. Confirm with a question: `"What do projects A and B have in common?"` → the agent synthesizes a text answer.

---

## 5. Usage tips

- **Upload in priority order**: the vault only uses the 5 most recent files for query — re-upload older files if you need Q&A on them.
- **Smaller files → more accurate graph**: only the first 6,000 characters drive extraction; split long documents into parts (executive summary, scope, risks).
- **Use consistent entity names**: if one file says `"Mr. John Smith"` and another says `"John S."`, the graph creates two separate nodes. Normalize names before uploading if you need a clean graph.
- **Make questions specific**: `"Who manages Project Alpha?"` is better than `"Tell me about Alpha"`.
- **The vault is not a search engine**: for exact lookups, use Ctrl-F in the source file. The vault is best for open-ended questions.
- **Privacy**: the vault is **per-user** — it is not shared with teammates. Don't upload documents expecting others to query them.

---

## 6. Known limitations

- **Only the first 6,000 characters** of each file are used for extraction — the rest is not in the knowledge graph.
- **Only the 5 most recent documents** + a 10K-character context cap are used per query — a vault with 100 files still queries only the latest 5.
- **No vector search**: there is no similarity retrieval — content is concatenated into context as-is.
- **No delete**: no delete API yet. Accidental uploads stay forever (they only get "pushed out of the top 5" by newer uploads).
- **Does not scale beyond ~50K total characters**: a very large vault doesn't improve results because of the context cap.
- **Graph extraction may miss things**: the LLM occasionally misses minor entities — no 100% guarantee.
- **Per-user**: no sharing, no team-wide vault.
