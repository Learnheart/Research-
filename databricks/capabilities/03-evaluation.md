# 03 — Evaluation & Quality Measurement

> **Vai trò trong lifecycle:** Pha **Evaluate** — đo lường chất lượng agent một cách hệ thống, cả offline (dev) lẫn online (production).

Đây là lớp Databricks nhấn mạnh đặc biệt: docs liệt kê **Quality** là 1 trong 3 thách thức GenAI chính ("GenAI models produce unpredictable outputs, making it difficult to define and measure quality").

---

## 3.1 Mosaic AI Agent Evaluation (MLflow-based)

Built on MLflow (docs lưu ý: MLflow 2 còn chạy được, nhưng "Databricks recommends migrating to MLflow 3").

- **LLM Judges & Scoring:** AI-powered judges chấm yes/no kèm rationale.
- **Evaluation Datasets:** benchmark set chứa representative questions ± ground-truth answers.
- **Review App:** UI thu thập "feedback about the quality of an AI application from human reviewers" (SME, stakeholders).
- **Output 2 dạng:**
  1. Per-row assessments — chất lượng / cost / latency từng request.
  2. Aggregate metrics — tổng hợp toàn bộ run.
- **Bridge dev ↔ production:** cùng scorer chạy cho offline labels và online unlabeled traces.

🔗 [Agent Evaluation docs](https://docs.databricks.com/aws/en/generative-ai/agent-evaluation/)

---

## 3.2 Built-in LLM Judges

Docs liệt kê các judge sẵn có, phân theo mục đích:

| Judge | Đo gì |
|---|---|
| **Correctness** | So sánh response với `expected_facts` / `expected_response` — phát hiện factual error |
| **Relevance to Query** | Response có địa chỉ trực tiếp câu hỏi user, không lạc đề |
| **Groundedness** | Response có bám retrieved context, không hallucinate |
| **Safety** | Không chứa harmful / offensive / toxic content |
| **Guideline Adherence** | Tuân thủ guideline được khai báo (global hoặc per-row) |
| **Context Sufficiency** | Retrieved documents có đủ thông tin để answer |
| **Chunk Relevance** | Từng retrieved chunk có liên quan — sinh precision metrics |
| **Document Recall** | Tỉ lệ ground-truth relevant docs thực sự retrieve được |

Mỗi judge cho ra **rating + rationale + error message**. Aggregate metric = % pass.

🔗 [LLM Judge Reference docs](https://docs.databricks.com/aws/en/generative-ai/agent-evaluation/llm-judge-reference)

**Adapt vào sản phẩm:**
- Đặt SLA chất lượng: "Groundedness ≥ 90%, Safety = 100%, Correctness ≥ 80%" làm release gate.
- Cho team data ngân hàng: Groundedness + Chunk Relevance là 2 metric must-have trước khi agent thấy production traffic.
- Custom judge cho "tone of voice" thương hiệu (ví dụ "không xưng anh/chị với customer Gen Z").

---

## 3.3 Review App & Labeling Sessions

- Stakeholders/SME review traces qua chat UI; attach feedback trực tiếp vào trace.
- Prototype có thể deploy nội bộ qua Review App Chat UI để beta tester dùng.
- Feedback → **labeling sessions** → **evaluation datasets** mới.

🔗 Trong [Agent Development Lifecycle docs](https://docs.databricks.com/aws/en/agents/agents-dev-lifecycle).

**Adapt vào sản phẩm:**
- Quy trình "SME-in-the-loop": pháp chế review answer của legal assistant trước khi mở rộng.
- Convert "production failures" thành test cases — biến lỗi sản xuất thành regression guard tự động.

---

## 3.4 MLflow Tracing & AI Insights

- **MLflow Tracing:** observability "across 20+ open-source frameworks" — ghi nhận từng step của agent (LLM call, tool call, retrieval).
- **MLflow AI Insights:** "analyze traces and point to likely causes" của low-quality response — debug systematic.
- Custom judges/scorers cho failure modes cụ thể.

🔗 Trong [Agent Development Lifecycle docs](https://docs.databricks.com/aws/en/agents/agents-dev-lifecycle).

**Adapt vào sản phẩm:**
- Khi user complain "agent trả lời sai", PM có trace + judge rationale → root cause trong phút, không cần ngồi đoán.
- Build dashboard "Top 10 failure modes tuần này" để team biết fix gì trước.

---

## 3.5 Regression Testing & Prompt Optimization

- Regression run trên evaluation dataset mỗi khi đổi prompt / model / tool config.
- Hỗ trợ **prompt optimization**: manual qua AI Playground, hoặc data-driven qua **MLflow + DSPy**.
- **Prompt Registry:** version control prompt theo MLflow experiments.

🔗 [Agent Development Lifecycle docs](https://docs.databricks.com/aws/en/agents/agents-dev-lifecycle)

**Adapt vào sản phẩm:**
- "Prompt PR" workflow: mỗi đổi prompt phải pass regression set với threshold cố định mới merge.
- Phòng marketing test multiple prompt variants — track A/B của prompt như sản phẩm experiment.

---

## Kết nối các bước tiếp theo

- Sau evaluation → deploy qua Model Serving / Apps (xem [04-serving-deployment](04-serving-deployment.md)).
- Cùng scorer chạy trên live traffic → xem [05-monitoring-observability](05-monitoring-observability.md).
