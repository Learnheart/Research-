# 02 — Tools & Retrieval Deep-Dive

> **Mục tiêu thảo luận:** Hiểu cơ chế Vector Search ở mức đủ debug retrieval failure + thiết kế chunking/embedding pipeline production-grade cho data ngân hàng (PII, đa ngôn ngữ, document phức tạp).

---

## 2.1 HNSW + L2 — chọn ANN engine

### Bản chất kỹ thuật
- **HNSW** (Hierarchical Navigable Small World): multi-layer graph; top layer thưa, layer dưới dày → search log-scale.
- **L2 (Euclidean)**: `dist = sqrt(Σ(a_i - b_i)²)`. Tốc độ tính nhanh, hardware-friendly.
- **Cosine sim**: docs ghi *"For cosine similarity, you must normalize embeddings beforehand"*. Tức là Vector Search **không** tự normalize — ngầm hiểu pipeline embedding phải xuất unit vector nếu muốn cosine.

### Relevance score
Công thức docs: `score = 1 / (1 + distance²)`.
- distance = 0 → score = 1.
- distance = 1 → score = 0.5.
- distance = ∞ → score → 0.
- Khoảng score [0, 1] thuận tiện cho threshold filtering.

> 🏦 **Banking insight:**
> Khi build "compliance Q&A" agent trên SBV circulars, threshold thấp (ví dụ score > 0.4) sẽ trả về nhiều chunk → nguy cơ hallucinate. Threshold cao (>0.7) lại miss nội dung diễn đạt khác. Solution: **đa giai đoạn**:
> 1. Retrieve top-k=20 với threshold thấp.
> 2. Rerank bằng cross-encoder (Databricks chưa có dedicated, có thể dùng LLM judge custom).
> 3. Trả top-5 sau rerank cho LLM generation.

> ⚠️ **Tradeoff HNSW:** không có "delete-in-place" cheap. Khi cập nhật document policy → cần Delta Sync để index tự rebuild incremental, không phải DIY.

---

## 2.2 Hybrid Search (RRF) — vũ khí thực dụng

### Cơ chế (xác nhận docs)
- Vector search độc lập → ranked list A.
- BM25 keyword search độc lập → ranked list B.
- RRF: `rrf_score(d) = Σ 1/(rrf_param + rank_i(d))`, default `rrf_param = 60`.
- Normalize max = 1.
- Trả top-k theo `rrf_score`.

### Tại sao hybrid quan trọng cho ngân hàng?
Query banking thường mix:
- **Semantic intent**: "lãi suất cho doanh nghiệp vừa và nhỏ" (cần embedding hiểu khái niệm SME).
- **Exact tokens**: "Thông tư 39/2016", "QĐ-NHNN", mã sản phẩm "VPB-LH-005".
- Vector-only miss exact token; BM25-only miss paraphrase. Hybrid handle cả 2.

### Số giới hạn từ docs
- Hybrid/full-text query: tối đa **200 results**.
- ANN: tối đa **10,000 results**.

> 🏦 **Banking insight:**
> Hybrid là default cho mọi retrieval trên data ngân hàng. Đặc biệt:
> - **Product catalog search:** SKU/mã sản phẩm phải match exact → hybrid + BM25 weight cao.
> - **Regulation/policy KB:** số văn bản (TT 39/2016, NĐ 13/2023) match BM25; điều khoản match semantic.

> ⚠️ **Tradeoff RRF default 60:** không tunable từ docs trang index. Nếu RRF default không cân bằng cho domain ngân hàng VN (ngôn ngữ ít token, BM25 score thấp), kết quả có thể nghiêng vector. Cần thử nghiệm.

> ❓ **Open question:** `rrf_param` có config được không? Docs chỉ nói "literature-based default".

---

## 2.3 Index types — chọn đúng từ ngày 1

| Index type | Khi dùng | Tradeoff |
|---|---|---|
| **Delta Sync + Databricks-managed embedding** | Source là Delta table, embedding model trong Databricks (gte-large-en) đủ tốt | Đơn giản nhất; lock-in vào model Databricks-hosted |
| **Delta Sync + self-managed embedding** | Cần embedding model riêng (multilingual, domain-specific) | Phải tự maintain embedding pipeline |
| **Direct Vector Access** | Data đến từ source ngoài Delta (stream từ Kafka, snapshot legacy) | Phải tự gọi REST update; không auto sync |
| **Full-text Search (Beta, storage-optimized)** | Keyword-heavy, không cần embed (code search, SKU lookup) | Beta; chỉ trên storage-optimized endpoint |

### Số liệu giới hạn (từ docs Vector Search)

| Tham số | Giới hạn |
|---|---|
| Max embedding dimensions | **4096** |
| Max columns / index | **50** |
| Max metadata fields / index | **50** |
| Max row size | **100 KB** (Delta Sync) |
| Max ANN result | **10,000** |
| Max hybrid/full-text result | **200** |
| Standard endpoint capacity | ~320M vectors @ 768 dim |
| Storage-optimized capacity | >1B vectors, ~250ms latency penalty |
| Indexing speed (storage-opt) | 10-20× faster than standard |

> 🏦 **Banking insight (chọn index):**
> - **Customer-doc KB (KYC docs, statement scan):** Delta Sync + self-managed embedding (multilingual cho VN). Volume 10-100M docs → standard endpoint đủ.
> - **Transaction text search (description SQL, narration):** Full-text Search Beta để khỏi embed hàng tỷ row.
> - **Realtime fraud signal store:** Direct Vector Access nếu signal đến qua Kafka (Delta Sync có latency phút, không đủ cho fraud).

> ⚠️ **Row size 100KB Delta Sync:** Document scan PDF có thể vượt nếu chunk to. Phải chunk + base64 đính kèm ra UC Volume riêng, index chỉ giữ chunk text + URI.

---

## 2.4 Chunking & embedding pipeline (suy từ docs RAG)

Docs RAG nói "Databricks supports unstructured (PDF, docs, images, videos) and structured data" nhưng **không bàn chunking strategy cụ thể** — phần này là solution direction.

### Pipeline reference

```
[Source]                [Ingestion]              [Transform]            [Index]            [Serve]
 PDF, Word, ──► Auto Loader / ──► Parser ──► Chunker ──► Embed ──► Vector ──► MCP ──► Agent
 Confluence,    Workflow            (PyMuPDF,  (semantic   model    Search    tool
 Drive,         (Delta Live          Tika,      / token-                              
 Sharepoint,    Tables)              Unstruct.) based)                                
 Database,      
 Mainframe
```

### Lựa chọn chunker (out-of-docs, solution direction)
- **Token-based** (e.g., 512 token + 50 overlap): đơn giản, predictable.
- **Semantic** (split theo similarity giữa câu): bảo vệ ngữ nghĩa, chunk bị dài/ngắn không đều.
- **Document-structure-aware** (split theo H1/H2/section): chuẩn cho contract / policy có structure rõ → giữ provenance section.

> 🏦 **Banking insight:**
> - **Credit policy / Procedure doc:** structure-aware (split theo điều khoản) → citation chính xác đến Điều X, Khoản Y.
> - **Customer email / chat history:** semantic chunking (đoạn hội thoại gắn liền).
> - **Statement / transaction CSV:** không embed text mà dùng structured retrieval (SQL), tránh waste embedding budget.

> ⚠️ **PII leak qua embedding:** Embedding một string chứa CMND/CIF không lộ ngược (one-way), nhưng *retrieved chunk* trả ra LLM lại có thể chứa raw. Phải **redact PII trước khi embed** hoặc tag chunk → filter ở retrieval time.

### Embedding model — `databricks-gte-large-en` only?
- Knowledge Assistant **yêu cầu** `databricks-gte-large-en` (English).
- Vector Search **độc lập**, có thể plug embedding model khác qua Foundation Model APIs hoặc self-managed.

> 🏦 **Banking insight (VN context):**
> `gte-large-en` chưa optimize cho tiếng Việt. Cho doc tiếng Việt:
> - Option A: multilingual embedding (BGE-M3, E5-multilingual) qua self-managed → giảm fit với Knowledge Assistant.
> - Option B: dịch sang EN qua LLM batch trước embed → mất nuance pháp lý VN, tốn token.
> - Option C: fine-tune embedding với data VN (cần infra training riêng) → cao đầu tư.
> → Khuyến nghị POC A trước, đo recall@k trên test set thực.

---

## 2.5 Tool architecture: chấp nhận 3 trục

```
                    ┌──────────────────────────┐
                    │       Agent (LLM)         │
                    └──────────────┬───────────┘
                                   │ tool_calls
                ┌──────────────────┼──────────────────┐
                ▼                  ▼                  ▼
        ┌────────────┐     ┌─────────────┐    ┌──────────────┐
        │ UC Function│     │ MCP Tool    │    │ Vector Search│
        │ (SQL/Py)   │     │ (Managed/   │    │ tool (índex) │
        │            │     │ Ext/Custom) │    │              │
        └────────────┘     └─────────────┘    └──────────────┘
                │                 │                  │
                ▼                 ▼                  ▼
        ┌──────────────────────────────────────────────────────┐
        │            Unity Catalog ACL + Lineage              │
        └──────────────────────────────────────────────────────┘
```

### `system.ai.python_exec` — sandbox dynamic code
Docs đề cập tool built-in này cho agent. Mang nhiều giá trị nhưng cần cẩn trọng:

> 🏦 **Banking insight:**
> Use case: "analytical Python" cho FP&A agent (compute moving avg, var, ratio). NHƯNG:
> - Không cho phép code chạm core banking API (chỉ cho tính toán local trong sandbox).
> - Cần check sandbox capabilities: có internet egress? Có truy cập UC volume? (Open question #4 ở file `00-research-notes`).

> ⚠️ **Tradeoff python_exec:** Rủi ro prompt injection bảo người dùng dụ LLM viết code lộ data. Mitigation: dùng AI Gateway guardrail (input filter), log mọi exec qua trace.

---

## 2.6 Genie & structured retrieval

Genie biến NL → SQL trên Delta tables/UC. Trong Supervisor Agent, Genie có thể là 1 sub-agent.

> 🏦 **Banking insight:**
> Genie phù hợp **exec/analyst Q&A** ("top 10 corporate khách hàng tăng dư nợ tháng qua") nhưng **không thay được manual SQL cho dashboard chính thức**. Lý do:
> - Genie có thể sinh SQL khác nhau cho cùng câu hỏi → không reproducible cho report regulator.
> - Audit yêu cầu fix SQL cho metric quan trọng (NPL, LCR…).
>
> Best practice: dùng Genie cho exploratory analysis, snapshot SQL Genie ra fix-form cho production dashboard.

---

## 2.7 Hot debate points

1. **Embedding model strategy:** Đa ngôn ngữ → tự host hay multilingual managed?
2. **Chunking ownership:** Trung tâm AI thiết kế hay BU tự chunk?
3. **Hybrid threshold tuning:** Mỗi domain (retail / corporate / treasury) cần threshold khác — quản lý thế nào không sprawl?
4. **PII redaction pipeline:** Embed-time (an toàn nhưng mất info) vs retrieval-time filter (giữ info nhưng phụ thuộc ACL)?
5. **MCP vs UC Function:** Khi nào migrate, khi nào giữ UC Function legacy?
6. **Direct Vector Access:** Có case nào ngân hàng thực sự cần realtime đến mức bỏ Delta Sync? Fraud có thể là điển hình — đáng đào sâu.
