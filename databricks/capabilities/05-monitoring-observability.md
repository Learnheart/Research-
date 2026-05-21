# 05 — Monitoring & Observability

> **Vai trò trong lifecycle:** Pha **Monitor** — đảm bảo chất lượng, cost, latency của agent trong production, không chỉ tại thời điểm release.

Mosaic AI gắn evaluation và monitoring vào cùng một bộ công cụ (cùng scorers chạy được ở cả 2 môi trường), giảm khoảng cách dev↔prod.

---

## 5.1 Production Monitoring trên Live Traces

Docs mô tả là "automatically run MLflow 3 scorers on traces from your GenAI apps to continuously assess quality" — schedule scorer chạy với MLflow experiment.

- **Automated assessment:** built-in + custom scorers, gồm cả **multi-turn judges** đánh giá nguyên cuộc hội thoại (user frustration, conversation completeness).
- **Configurable sampling:** sample rate 0.05–1.0 để cân bằng cost vs coverage.
- **Consistent evaluation:** cùng scorer dev và prod — kết quả gắn vào từng trace.
- **Multi-turn support:** đánh giá pattern hội thoại.
- **Judge types:**
  - Built-in LLM judges (Safety, Guidelines…)
  - Custom prompt-based judges
  - Code-based scorer functions
  - Multi-turn conversation judges
- **Constraints (docs liệt kê):** tối đa 20 concurrent scorers / experiment; custom scorer phải khai báo trong Databricks notebook; function self-contained với inline imports; 15-20 phút "warm-up" trước khi có kết quả.

🔗 [Production Monitoring docs](https://docs.databricks.com/aws/en/generative-ai/agent-evaluation/monitoring)

**Adapt vào sản phẩm:**
- Sample 10% production trace để chấm Groundedness + Safety realtime, đảm bảo không bị "drift" sau khi đổi prompt.
- Multi-turn judge cho retention metric: nếu hội thoại bị "user frustration" trên ngưỡng → alert.
- Convert failure traces thành labeled set cho next iteration → vòng feedback dev↔prod ngắn.

---

## 5.2 MLflow Tracing

- Observability ghi lại từng step của agent execution (LLM call, tool call, retrieval).
- Tương thích "across 20+ open-source frameworks" — LangChain, LlamaIndex, OpenAI SDK…
- Auto-attach trace cho mọi endpoint deploy qua `agents.deploy()`.
- Trace là input chính cho Production Monitoring scorers + Review App + AI Insights.

🔗 Trong [Agent Development Lifecycle docs](https://docs.databricks.com/aws/en/agents/agents-dev-lifecycle).

**Adapt vào sản phẩm:**
- Khi customer complain, support copy paste request ID → mở trace ngay, không cần "tái hiện".
- Debug spike latency: trace thể hiện step nào ăn nhiều token / time.

---

## 5.3 OpenTelemetry Integration

Docs (trong [gen-ai-capabilities](https://docs.databricks.com/aws/en/agents/gen-ai-capabilities)) đề cập: "OpenTelemetry Integration: Export traces to third-party monitoring tools".

**Adapt vào sản phẩm:**
- Tổ chức đã chuẩn OTel (Datadog, New Relic, Honeycomb…) — vẫn dùng được monitoring stack hiện hành, không bị vendor-lock vào MLflow UI.

---

## 5.4 Dashboards & SQL Alerts

- Build dashboards và **SQL quality alerts** trên trace tables.
- Tận dụng Inference Tables (do `agents.deploy()` tự tạo) như một bảng audit log query được bằng SQL.

🔗 Trong [Agent Development Lifecycle docs](https://docs.databricks.com/aws/en/agents/agents-dev-lifecycle).

**Adapt vào sản phẩm:**
- Dashboard "agent health" cho CIO: groundedness 7 ngày, top failure mode, cost / 1000 request.
- Alert pager khi safety score < 99% hoặc latency p95 > 5s.

---

## 5.5 Feedback Loop từ Production

Theo Lifecycle docs, vòng feedback chuẩn gồm:

1. End-user gửi feedback (trên Review App hoặc app sản phẩm).
2. Production failures → convert thành new evaluation data.
3. AI Insights phân tích trace, gợi ý root cause.
4. Engineer fix → regression run → deploy version mới (zero-downtime).

**Adapt vào sản phẩm:**
- Thiết kế "👍/👎 + lý do" trong UI từ ngày đầu — feedback signal là tài nguyên đắt nhất, không có nó monitoring chỉ là half-loop.
- KPI ops: thời gian từ "fail trace xuất hiện" → "regression test cover case đó" → "deploy fix" — đo và cải thiện.

---

## Kết nối các bước tiếp theo

- Mọi log/trace/feedback đều chảy vào Unity Catalog → governance & audit ([06-governance-security](06-governance-security.md)).
- Trace là nguyên liệu cho vòng evaluation tiếp theo ([03-evaluation](03-evaluation.md)).
