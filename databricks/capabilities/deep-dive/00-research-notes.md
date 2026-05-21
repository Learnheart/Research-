# 00 — Research Notes & Glossary

> File này là **starter pack** cho người mới tham gia thảo luận: methodology, glossary, các câu hỏi mở còn treo (open questions) cần verify thêm.

---

## 0.1 Methodology

**Cách crawl docs (gross outline):**
1. Bắt đầu từ `docs.databricks.com/aws/en/agents/` index page.
2. Theo link sub-pages thuộc các domain: `agent-framework`, `agent-evaluation`, `agent-bricks`, `vector-search`, `ai-gateway`, `mlflow3/genai`, `data-governance/unity-catalog`.
3. Cross-check khi 1 capability xuất hiện ở nhiều page — chọn page **tham chiếu kỹ thuật** làm canonical (ví dụ Vector Search internals → trang `vector-search`, judge mechanics → `llm-judge-reference`).
4. Đánh dấu (Beta) nếu docs label, đánh dấu *"không tìm thấy trong docs"* nếu thiếu chi tiết cần thiết.

**Cách sàng tin:**
- Ưu tiên trang `mlflow3/genai` hơn `agent-evaluation` (MLflow 2) cho design mới.
- Bỏ qua phần "marketing-style" — chỉ trích con số, tham số, cấu hình.
- Khi 2 trang đưa ra số khác nhau, chọn trang gần nhất tới capability đó về mặt URL (ví dụ rate limit theo `ai-gateway/rate-limits-beta` chứ không phải overview).

---

## 0.2 Glossary chuyên ngành

### Agent / GenAI

| Thuật ngữ | Định nghĩa (theo docs) |
|---|---|
| **Agent** | Hệ thống dùng LLM ra quyết định gọi tool / retrieve / phản hồi user. |
| **Agent Bricks** | Thư viện agent low-code do Databricks cung cấp (Knowledge Assistant, Supervisor). |
| **Agent Framework** | SDK + Python API (`databricks-agents`) để author / log / deploy agent. |
| **ResponsesAgent** | Interface MLflow chuẩn để wrap agent — cho phép Databricks hook tracing/eval/monitoring. |
| **ChatAgent** | Interface cũ hơn của MLflow (đề cập trong context MLflow tracing). |
| **MCP** | Model Context Protocol — chuẩn open-source kết nối agent tới tool/resource/prompt. |
| **Supervisor Agent** | Pattern multi-agent có 1 orchestrator điều phối sub-agent. |
| **Knowledge Assistant** | Agent low-code chuyên Q&A trên documents qua Instructed Retriever. |
| **Genie Space** | Agent natural language → SQL trên structured data. |

### Retrieval / Search

| Thuật ngữ | Định nghĩa |
|---|---|
| **HNSW** | Hierarchical Navigable Small World — graph-based ANN algorithm. |
| **L2 distance** | Khoảng cách Euclid; Vector Search default. Cosine sim cần normalize trước. |
| **Relevance score** | `1 / (1 + distance²)`, kết quả trong [0, 1]. |
| **Delta Sync Index** | Index tự đồng bộ với source Delta table; có 2 mode: managed embedding / self-managed embedding. |
| **Direct Vector Access** | Index ghi/cập nhật thủ công qua REST API. |
| **Full-text Search Index** | (Beta, storage-optimized only) BM25 keyword, không cần embeddings. |
| **RRF** | Reciprocal Rank Fusion — kết hợp vector rank + keyword rank trong hybrid search. |
| **rrf_param** | Tham số RRF; default `60` (theo literature). |
| **BM25 (Okapi)** | Probabilistic ranking function cho keyword search. |

### Evaluation / Monitoring

| Thuật ngữ | Định nghĩa |
|---|---|
| **Scorer** | Unified eval component, return pass/fail / value, attach feedback vào trace. |
| **LLM Judge** | Scorer dùng LLM để chấm (vs code-based scorer chấm deterministic). |
| **Trace** | Execution flow của 1 request, gồm TraceInfo + TraceData (collection of spans). |
| **Span** | 1 step trong trace (LLM call, tool call, retrieval). |
| **Assessment** | Feedback hoặc Expectation gắn vào trace (MLflow 3 data model). |
| **Feedback** | Đánh giá *actual* output (rating + comment). |
| **Expectation** | Ground truth — *desired* outcome. |
| **Labeling session** | SME ngồi label batch trace để build eval dataset. |
| **Review App** | UI Databricks-provided cho human review chat. |
| **Multi-turn judge** | Judge đánh giá nguyên cuộc hội thoại; default group theo session 5 phút inactivity. |

### Serving / Governance

| Thuật ngữ | Định nghĩa |
|---|---|
| **Model Serving** | Hạ tầng serve model + agent dưới REST endpoint. |
| **Pay-per-token** | Pricing endpoint thiên về experimentation, traffic biến động. |
| **Provisioned throughput** | Endpoint dedicate compute, fixed cost, prod SLA. |
| **External model routing** | Endpoint proxy tới OpenAI/Anthropic/vendor. |
| **AI Gateway** | (Beta) Control plane tập trung cho LLM/agent traffic. |
| **Inference Table** | Delta table log request/response endpoint qua AI Gateway. |
| **Unity Catalog** | Unified governance layer cho data + AI assets. |
| **DABs** | Databricks Asset Bundles — IaC declarative cho agent / job / app. |
| **Service Principal** | Identity service-to-service (vs PAT của user). |
| **On-behalf-of-user auth** | Agent gọi tài nguyên dưới quyền của end-user. |

---

## 0.3 Open questions (cần verify thêm)

> Các câu hỏi sau **không tìm thấy answer rõ ràng trong docs** đã crawl. Liệt kê để team prioritize verification trước khi commit kiến trúc.

### Authoring
- **Q1:** ResponsesAgent có hỗ trợ ngôn ngữ khác Python không? Docs nói "wrap with MLflow's ResponsesAgent" nhưng không nói có client SDK ngoài Python.
- **Q2:** Supervisor Agent giới hạn 20 sub-agent — có config override không? Hay phải hierarchical (supervisor-of-supervisors)?
- **Q3:** Knowledge Assistant chỉ English — có roadmap multilingual không, hay cần custom Agent Framework cho VN?

### Tools / Retrieval
- **Q4:** `system.ai.python_exec` sandbox: có internet egress không? Có timeout? Có thể call UC function khác không?
- **Q5:** Vector Search Delta Sync: latency end-to-end từ row insert → index update trong worst case? (Docs nói "auto sync" nhưng không nói SLA cụ thể.)
- **Q6:** Embedding model `databricks-gte-large-en` có hỗ trợ tiếng Việt không? Cần test thực nghiệm.

### Evaluation
- **Q7:** LLM judge model: docs cho phép specify `<provider>:/<model-name>`. Có cho phép self-hosted model làm judge không?
- **Q8:** Judge có thể dùng để chấm trace có PII không, hay LLM judge có log lại payload? (Quan trọng cho data residency.)

### Serving / Gateway
- **Q9:** AI Gateway guardrails (PII detection, content safety) chính xác là sub-feature nào? Docs chính chưa mô tả enforcement point + latency overhead.
- **Q10:** Traffic split 5 destinations max — có hỗ trợ "shadow traffic" (gọi 2 model parallel, chỉ trả 1) không?

### Monitoring
- **Q11:** Production monitoring 15-20p warm-up: có thể giảm bằng sample rate cao + scorer ít không? Hay là constant?
- **Q12:** Inference Tables "best effort delivery" — có metric đo loss rate không? Critical cho audit ngân hàng.

### Governance
- **Q13:** Unity Catalog tag PII có tự propagate sang vector index không?
- **Q14:** Multi-region replication cho UC + Vector Search trong DR plan?

---

## 0.4 Sources crawled (tham chiếu nhanh)

Sắp xếp theo nhóm capability để mở lại khi cần.

### Authoring & Framework
- `agents/agent-system-design-patterns`
- `agents/gen-ai-capabilities`
- `agents/agents-dev-lifecycle`
- `generative-ai/agent-framework/author-agent`
- `generative-ai/agent-framework/log-agent`
- `generative-ai/agent-framework/deploy-agent`
- `generative-ai/agent-framework/agent-tool`
- `generative-ai/agent-bricks/knowledge-assistant`
- `generative-ai/agent-bricks/multi-agent-supervisor`

### Retrieval & Tools
- `generative-ai/vector-search`
- `generative-ai/retrieval-augmented-generation`
- `generative-ai/mcp/`

### Evaluation
- `generative-ai/agent-evaluation/`
- `generative-ai/agent-evaluation/llm-judge-reference`
- `generative-ai/agent-evaluation/llm-judge-metrics`
- `generative-ai/agent-evaluation/synthesize-evaluation-set`
- `mlflow3/genai/eval-monitor/concepts/scorers`
- `mlflow3/genai/eval-monitor/production-monitoring`
- `mlflow3/genai/human-feedback/`
- `mlflow3/genai/tracing/tracing-101`

### Gateway & Governance
- `ai-gateway/inference-tables-beta`
- `ai-gateway/rate-limits-beta`
- `ai-gateway/configure-traffic-splitting-beta`
- `ai-gateway/usage-tracking-beta`
- `data-governance/unity-catalog/`
- `data-governance/unity-catalog/ai-governance`

### Query
- `agents/query-llms`
- `agents/gen-ai-challenges`

---

## 0.5 Convention cho deep-dive notes

- Khi viện dẫn số liệu → đặt link inline ngay tại đó.
- Khi đưa solution direction cho ngân hàng → đặt trong block `> 🏦 Banking insight:`
- Khi flag tradeoff / risk → đặt trong block `> ⚠️ Tradeoff:`
- Khi flag open question → đặt trong block `> ❓ Open question:`
