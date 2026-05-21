# 02 — Tools & Retrieval

> **Vai trò trong lifecycle:** Pha **Develop** — cung cấp agent khả năng "tay chân + mắt": gọi API/function (tools) và truy xuất tri thức (retrieval).

Đây là lớp quyết định agent có chạy được trên dữ liệu thật của doanh nghiệp hay không. Databricks tách rõ 2 trục: **tools** (hành động) và **retrieval** (lấy context).

---

## 2.1 Tools — 3 cách tạo

### 2.1.1 Unity Catalog Functions
- Build custom tools dưới dạng SQL/Python function, có sẵn access control.
- Docs lưu ý: "Databricks recommends using MCP tools instead for most new use cases" — UC Functions vẫn hỗ trợ nhưng MCP là hướng mới.

### 2.1.2 MCP Servers (Model Context Protocol) — khuyến nghị
- **Managed MCP servers:** cung cấp quyền truy cập vào dữ liệu Databricks (Unity Catalog data, Vector Search…).
- **External MCP servers:** kết nối API bên thứ ba.
- **Custom MCP servers:** deploy như Databricks Apps.

### 2.1.3 Vector Search Tools
- Cho phép agent "query unstructured data" qua index.

**Built-in tool đặc biệt:** `system.ai.python_exec` để chạy dynamic Python code.

🔗 [Agent Tools docs](https://docs.databricks.com/aws/en/generative-ai/agent-framework/agent-tool)

**Adapt vào sản phẩm:**
- Tool "lookup customer by ID" qua UC function gọi Delta table.
- Tool gọi external API thanh toán / shipping qua external MCP server.
- Tool "run analytical Python" cho FP&A agent với `python_exec`.
- Tool RAG nội bộ qua Vector Search index gắn vào Unity Catalog.

---

## 2.2 Vector Search (unstructured retrieval)

Là backbone retrieval của Databricks, tích hợp sâu với Delta Lake và Unity Catalog.

- **Thuật toán:** HNSW (Hierarchical Navigable Small World), L2 distance.
- **Index types:**
  - **Delta Sync** — auto sync với source table. Có 2 chế độ: **Databricks-managed embeddings** (Databricks tự embed bằng model chỉ định) hoặc **self-managed embeddings** (vector tính trước).
  - **Direct Vector Access** — manual update qua REST.
  - **Full-text Search** (Beta, storage-optimized) — BM25 keyword, không cần embeddings.
- **Search modes:** vector similarity, **hybrid search** (vector + keyword qua Reciprocal Rank Fusion), filtering theo metadata, reranking.
- **Endpoint:**
  - Standard: ~320M vector (768 dim), high QPS.
  - Storage-optimized: >1B vector, indexing nhanh 10-20x, latency ~250ms.
- **Governance:** index xuất hiện trong Unity Catalog, ACL-controlled. Mã hóa AES-256 at rest, TLS 1.2+ in transit.

🔗 [Vector Search docs](https://docs.databricks.com/aws/en/generative-ai/vector-search)

**Adapt vào sản phẩm:**
- Semantic search trên contract / policy library — Delta Sync với managed embeddings để khỏi tự quản pipeline embedding.
- Product search với hybrid mode khi cần khớp cả semantic ("áo mùa đông") lẫn keyword (SKU, mã hàng).
- Sản phẩm volume cực lớn (knowledge graph hàng tỷ chunk): chọn storage-optimized.

---

## 2.3 Structured Retrieval & Genie

Docs phân biệt rõ retrieval cho **unstructured** (Vector Search) và **structured** (SQL/Delta tables, feature store).

- **Structured Retrieval:** SQL integration + online feature stores cho real-time data.
- **Genie Agents:** multi-agent system cho phép natural-language query trên structured data (tables) — có thể đóng vai trò sub-agent dưới Supervisor.

🔗 Có trong [GenAI Capabilities docs](https://docs.databricks.com/aws/en/agents/gen-ai-capabilities) và [RAG guide](https://docs.databricks.com/aws/en/generative-ai/retrieval-augmented-generation).

**Adapt vào sản phẩm:**
- Sales assistant trả lời "Top 10 khách hàng tháng trước" qua Genie space gắn lên Delta table.
- Customer 360 view: agent hỏi feature store online để lấy CLV, last login realtime.

---

## 2.4 RAG end-to-end pattern

Docs định nghĩa RAG là "LLMs + real-time data retrieval" theo 3 pha: retrieve → augment → generate.

- Hỗ trợ cả **unstructured** (PDF, docs, image, video) lẫn **structured** (DB tables, API data).
- Citation/source attribution → fact-checking.
- User-credential-based data access (kế thừa từ Unity Catalog).
- Tích hợp Delta Lake (pipeline), Vector Search (retrieval), Model Serving, Agent Evaluation, monitoring, AI Gateway — tất cả trong cùng platform.

🔗 [RAG Guide docs](https://docs.databricks.com/aws/en/generative-ai/retrieval-augmented-generation)

**Adapt vào sản phẩm:**
- "Internal Wikipedia chatbot": RAG trên Confluence + Drive đã đổ vào Delta + Vector Search.
- Compliance Q&A trên regulatory documents — citation rất quan trọng cho audit.
- Sales enablement assistant kết hợp product docs (Vector Search) + CRM data (structured retrieval).

---

## Kết nối các bước tiếp theo

- Sau khi gắn tools/retrieval → đánh giá xem retrieval có chính xác không bằng **Chunk Relevance / Context Sufficiency / Document Recall** judges (xem [03-evaluation](03-evaluation.md)).
- Tools/retrieval đều được Unity Catalog quản trị → xem [06-governance-security](06-governance-security.md).
