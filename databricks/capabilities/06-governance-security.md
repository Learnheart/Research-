# 06 — Governance, Security & LLMOps

> **Vai trò trong lifecycle:** Cắt ngang **toàn bộ lifecycle** — đảm bảo data, model, agent, prompt, trace đều được quản trị tập trung, có audit, có ACL.

Docs gọi đây là 1 trong 3 thách thức cốt lõi của GenAI ("unified governance, data privacy, and security for data and AI assets") — và là điểm khác biệt lớn nhất của Databricks so với "lắp ráp 5 SaaS".

---

## 6.1 Unity Catalog — backbone governance

- Unified governance layer "built into Databricks".
- **AI assets được quản trị:** models, functions, vector indexes, model registry, AI traffic (qua AI Gateway).
- **Cơ chế:**
  - Access control bằng privileges + attribute-based policies.
  - **Automatic lineage** — thấy dòng data đi từ raw → embedding → vector index → agent response.
  - **Audit logging** đầy đủ cho data + system activity.
  - Data classification cho thông tin nhạy cảm.
- "Enforces access control when you query a table, tracks lineage as data moves, logs activity for auditing" — áp dụng đồng nhất cho AI asset.
- Workspaces tạo sau 2023-11-08 mặc định bật Unity Catalog.

🔗 [Unity Catalog docs](https://docs.databricks.com/aws/en/data-governance/unity-catalog/)

**Adapt vào sản phẩm:**
- "Một mặt phẳng" để security team review quyền cho agent: thấy được agent X đọc table Y nào, qua tool Z nào.
- Lineage hỗ trợ compliance: khi auditor hỏi "câu trả lời này dựa trên data nào?" → trace + lineage ra ngay.

---

## 6.2 Unity AI Gateway — control plane cho LLM/Agent traffic

Docs mô tả là "central AI governance layer for agents, LLM endpoints, MCP servers, and coding agents".

### Capabilities được docs xác nhận
- **Access control & permission** cho LLM/agent endpoints.
- **Capacity management** across providers.
- **Rate limiting** để kiểm soát consumption.
- **Traffic splitting** giữa nhiều model backend (cơ sở cho A/B / canary).
- **Guardrails** (docs page title đề cập, chi tiết ngoài scope page index).
- **Observability:**
  - Unified dashboard usage analysis.
  - Cost tracking qua billable usage system tables.
  - Inference table logging cho request/response audit.
  - Real-time usage monitoring.
- **Scope:** LLM endpoints, coding agents, MCP servers, serving endpoints.
- **Status (tại thời điểm crawl docs):** Beta, miễn phí; account admin bật qua workspace previews.

🔗 [AI Gateway docs](https://docs.databricks.com/aws/en/ai-gateway/index)

> **Note:** PII detection, content guardrails specifics, A/B testing chưa được nêu chi tiết trong trang index — **không tìm thấy đầy đủ trong docs sample đã crawl**; cần đọc sub-pages của AI Gateway để confirm.

**Adapt vào sản phẩm:**
- Đặt rate limit theo team/dept để tránh "1 nhóm xài hết quota".
- Traffic split để rollout model mới: 5% traffic sang Claude 4.x trong 1 tuần, monitor metric trước khi 100%.
- Inference table phục vụ audit ngân hàng — mọi request/response lưu nguyên dạng.

---

## 6.3 LLMOps Stack

Theo [GenAI Capabilities docs](https://docs.databricks.com/aws/en/agents/gen-ai-capabilities):

- **Prompt Registry:** version control + experiment tracking qua MLflow.
- **MLOps Stacks:** infrastructure-as-code deployment automation.
- **OpenTelemetry Integration:** export trace ra third-party monitoring.

**Adapt vào sản phẩm:**
- Prompt là "code", không phải "config trong DB" — bắt buộc qua Registry để rollback dễ.
- MLOps Stacks giúp scale từ 1 agent → 50 agent qua IaC, tránh "click-ops".

---

## 6.4 Security baseline trong stack

- **Vector Search:** AES-256 at rest, TLS 1.2+ in transit, ACL qua Unity Catalog.
- **Model Serving deploy_agent:** auto-provision short-lived credentials, verify permission của caller trước khi gọi tool.
- **User authorization (on-behalf-of-user)** trong Agent Framework: agent kế thừa quyền của end-user thay vì chạy bằng service principal — quan trọng cho RAG trên data nhạy cảm.

🔗 [Agent Framework — Author Agent docs](https://docs.databricks.com/aws/en/generative-ai/agent-framework/author-agent)
🔗 [Vector Search docs](https://docs.databricks.com/aws/en/generative-ai/vector-search)

**Adapt vào sản phẩm:**
- Internal HR agent: bắt buộc on-behalf-of-user, không để nhân viên cấp dưới đọc được data của cấp trên qua agent.
- Multi-tenant SaaS: dùng service principal auth + tenant filter ở UC level.

---

## 6.5 Liên quan tới 3 challenges GenAI theo Databricks

Docs nói rõ 3 challenges + lời giải:

| Challenge | Databricks giải bằng |
|---|---|
| **Governance** (data leakage, compliance, uncontrolled cost) | Unity Catalog, AI Gateway, AI Security & Governance Frameworks |
| **Quality** (unpredictable outputs) | MLflow Evaluation, Tracing, human feedback tools |
| **Control** (multi-provider, customization, privacy) | Foundation Model APIs, Model Serving, Data Intelligence |

🔗 [GenAI Challenges docs](https://docs.databricks.com/aws/en/agents/gen-ai-challenges)

**Adapt vào sản phẩm:** Khi pitch CIO/CISO, dùng bảng này làm gốc — "không phải Databricks thay LLM, mà là Databricks bọc 3 lớp này quanh LLM bạn chọn".

---

## Kết nối các bước tiếp theo

- Sau khi nắm governance baseline → đối chiếu với checklist "when to choose Databricks" ([07-when-to-choose](07-when-to-choose.md)).
