# 07 — Khi nào nên chọn Databricks cho AI Agent

> Tổng hợp các signal từ docs giúp quyết định Databricks **fit** hay **không fit** với use case.
> Chỉ dùng signal docs đề cập trực tiếp, không suy đoán so sánh với platform khác.

---

## 7.1 Databricks fit rõ khi…

### ✅ Data của bạn đã ở Databricks (hoặc sẽ về Databricks)
- Unity Catalog quản trị data + AI assets cùng một mặt phẳng → bỏ qua được lớp ETL + permission sync sang vendor agent ngoài.
- Vector Search **Delta Sync** auto đồng bộ với source table → pipeline embedding gần như "miễn phí".
- 🔗 [Unity Catalog](https://docs.databricks.com/aws/en/data-governance/unity-catalog/) · [Vector Search](https://docs.databricks.com/aws/en/generative-ai/vector-search)

### ✅ Cần governance / audit chặt
- Lineage tự động + audit log + inference tables (do `agents.deploy()` tự sinh).
- AI Gateway tập trung rate limit, guardrails, cost tracking.
- Phù hợp ngành tài chính, bảo hiểm, y tế, public sector.
- 🔗 [GenAI Challenges](https://docs.databricks.com/aws/en/agents/gen-ai-challenges) · [AI Gateway](https://docs.databricks.com/aws/en/ai-gateway/index)

### ✅ Cần đa dạng pattern agent (simple chain → multi-agent)
- Cùng platform support 4 mức: LLM+Prompt, Deterministic Chain, Single-Agent, Multi-Agent (Supervisor).
- Knowledge Assistant + Supervisor cho phép low-code; Agent Framework + Databricks Apps cho phép full-code.
- 🔗 [Agent System Design Patterns](https://docs.databricks.com/aws/en/agents/agent-system-design-patterns)

### ✅ Quality là release gate (không phải afterthought)
- Built-in 10 LLM judges (Groundedness, Safety, Correctness, Chunk Relevance, Document Recall…).
- Cùng scorer chạy được offline (dev) + online (prod) → loại bỏ gap "dev tốt, prod fail".
- 🔗 [Agent Evaluation](https://docs.databricks.com/aws/en/generative-ai/agent-evaluation/)

### ✅ Multi-provider model strategy
- Foundation Model APIs hỗ trợ multi provider; external model routing để gọi OpenAI/Anthropic qua 1 layer.
- Pay-per-token cho POC → provisioned throughput cho prod, không đổi code client.
- 🔗 [Query LLMs](https://docs.databricks.com/aws/en/agents/query-llms)

### ✅ Stakeholder feedback loop quan trọng (SME-in-the-loop)
- Review App + labeling sessions built-in.
- Production failure → evaluation data → regression test, vòng feedback rõ ràng.
- 🔗 [Agent Development Lifecycle](https://docs.databricks.com/aws/en/agents/agents-dev-lifecycle)

### ✅ Team có mix data engineer + ML engineer
- Lifecycle thiết kế quanh MLflow + Unity Catalog — chuỗi công cụ DE/ML đã quen.
- Databricks Apps + DABs cho engineer git-native workflow; AI Playground + Agent Bricks cho PM/analyst.

---

## 7.2 Databricks **có thể không phải** lựa chọn tối ưu khi…

> Lưu ý: phần này dựa trên các giới hạn docs đề cập trực tiếp, KHÔNG so sánh với vendor khác (out of scope theo yêu cầu).

### ⚠️ Use case agent phụ thuộc data nằm ngoài hệ sinh thái Databricks
- Nếu data chủ yếu ở SaaS / data warehouse khác mà không có nhu cầu đưa về Databricks → bạn sẽ phải build ingestion lớp riêng, làm mất lợi thế Delta Sync.

### ⚠️ Supervisor Agent ràng buộc giới hạn
- Docs ghi rõ: max **20 agent / supervisor**, và **chỉ English** (tại thời điểm tài liệu).
- Use case multi-lingual / multi-domain rộng cần xem xét custom orchestration thay vì Supervisor Bricks.
- 🔗 [Supervisor Agent](https://docs.databricks.com/aws/en/generative-ai/agent-bricks/multi-agent-supervisor)

### ⚠️ Knowledge Assistant ràng buộc hạ tầng
- Yêu cầu **serverless compute** + Unity Catalog + embedding model `databricks-gte-large-en` bật.
- Tổ chức chưa bật serverless / chưa migrate UC sẽ tốn pha onboarding trước khi dùng được.
- 🔗 [Knowledge Assistant](https://docs.databricks.com/aws/en/generative-ai/agent-bricks/knowledge-assistant)

### ⚠️ Production Monitoring giới hạn
- Max **20 concurrent scorers** / experiment.
- Custom scorer phải khai báo trong **Databricks notebook**, self-contained inline imports — kỷ luật code khác workflow Python repo thông thường.
- "15-20 phút warm-up" trước khi có kết quả monitoring đầu tiên — không phù hợp use case cần feedback theo giây.
- 🔗 [Production Monitoring](https://docs.databricks.com/aws/en/generative-ai/agent-evaluation/monitoring)

### ⚠️ MLflow 2 vs 3 — đang migration
- Agent Evaluation gốc trên MLflow 2; docs khuyến nghị "migrate to MLflow 3 for new implementations".
- Team build mới nên xác nhận version compatibility ngay từ ngày 1 để tránh tech-debt sớm.

---

## 7.3 Checklist quyết định 1 trang

| Câu hỏi | Có → fit | Không → cân nhắc |
|---|---|---|
| Data chính ở Databricks? | ✅ | ⚠️ |
| Audit/compliance là bắt buộc? | ✅ | ↔ neutral |
| Cần multi-provider model? | ✅ | ↔ neutral |
| Có SME workflow review/feedback? | ✅ | ↔ neutral |
| Multi-agent >20 sub-agent, multi-language? | ⚠️ | ✅ (single/<20) |
| Quality monitoring trên live traffic là yêu cầu? | ✅ | ↔ neutral |
| Sản phẩm SaaS multi-tenant lớn? | ↔ (cần thiết kế UC kỹ) | — |
| Cần real-time monitoring (<1 phút)? | ⚠️ (20p warm-up) | — |

> Bảng này tổng hợp signal từ docs đã crawl — dùng làm gốc cho discovery với khách / stakeholder, KHÔNG phải competitive comparison.
