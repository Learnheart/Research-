# Databricks AI Agent Platform — Strategic Research Workspace

**Workspace version**: v0.2-draft
**Last updated**: 2026-05-21
**Classification**: Internal

---

## 1. Bối cảnh & Mục tiêu

- **Sponsor**: CIO, CFO.
- **Mục tiêu chiến lược**: Đánh giá Databricks (Mosaic AI Agent Framework, Unity Catalog, Model Serving, Vector Search, MLflow, Genie) làm nền tảng xây dựng **AI Agent Platform nội bộ** cho banking/FSI.
- **Đầu ra của workspace**:
  1. Báo cáo sơ bộ **feasibility** trình CIO/CFO.
  2. **Strategic roadmap** R&D → POC → Pilot → Production với coverage matrix so với roadmap sản phẩm hiện hành.
  3. Cơ sở định lượng (TCO, ROI scenarios) để **commit/decline R&D budget**.
- **Nguồn gốc tài liệu chính**: <https://docs.databricks.com/aws/en/agents/> và các trang liên kết, bổ sung primary sources khác theo `RULES.md`.

## 2. Phạm vi (Scope)

| In-scope | Out-of-scope (giai đoạn này) |
|---|---|
| Tech overview Databricks cho AI Agent | Cài đặt hạ tầng chi tiết (Terraform, network) |
| Market landscape AI Agent + FSI | Vendor negotiation, pricing đàm phán |
| Competitive analysis với hyperscaler & OSS | Procurement & legal contracting |
| Banking applications EU/US – China – Vietnam | Implementation code samples production-grade |
| Roadmap R&D → Production | Detailed change management plan |
| TCO/ROI ở mức scenario | Audited financial statements |
| Risk, compliance, security mapping | Internal audit deliverables |

## 3. Audience & quan tâm chính

| Stakeholder | Quan tâm chính | Workstream ưu tiên |
|---|---|---|
| CIO | Tính khả thi kỹ thuật, time-to-value, vendor lock-in | 03, 04, 06 |
| CFO | TCO, ROI, build vs buy | 07, 06 |
| Chief Risk Officer | Model risk, data residency, compliance | 07, 05 |
| Head of Data/AI | Architecture, MLOps maturity, integration | 03, 04 |
| Head of Digital Banking | Use case fit, competitive position trong ngành | 02, 05 |
| Head of Procurement | Pricing model, contractual lock-in | 07 |

## 4. Cấu trúc Workspace (đúng 2 levels)

```
databricks/docs/
├── README.md                       ← bạn đang ở đây
├── WORKSPACE_TRACKING.md           ← bảng tiến độ + open questions
├── RULES.md                        ← quy ước làm việc (citation, classification, DoD)
├── 01_Executive_Summary/
├── 02_Market_Landscape/
├── 03_Databricks_Technology/
├── 04_Competitive_Analysis/
├── 05_Banking_FSI_Applications/
├── 06_Strategic_Roadmap/
├── 07_Business_Case_And_Risk/
└── 08_References_Appendix/
```

Mỗi folder có `_overview.md` (scope + dàn ý + key questions) và các file phân tích chi tiết được đánh số theo thứ tự đọc đề xuất.

## 5. Workstream — Mục tiêu một dòng

| # | Workstream | Mục tiêu |
|---|---|---|
| 01 | Executive Summary | Tóm tắt key findings, recommendation, decision framework cho C-Level |
| 02 | Market Landscape | Quy mô thị trường AI Agent, adoption trends, regulatory drivers |
| 03 | Databricks Technology | Phân tích sâu các component lõi của Databricks cho AI Agent |
| 04 | Competitive Analysis | So sánh với Snowflake Cortex, AWS Bedrock/SageMaker, Azure AI Foundry, GCP Vertex, native LLM provider, OSS stacks |
| 05 | Banking/FSI Applications | Case study & use case EU/US, China, Vietnam — quy định ngành |
| 06 | Strategic Roadmap | R&D → POC → Pilot → Production + coverage matrix so với roadmap hiện tại |
| 07 | Business Case & Risk | TCO/ROI/build-vs-buy + model risk, data sovereignty, threat modeling, compliance mapping |
| 08 | References & Appendix | Primary/secondary sources, glossary, interview notes |

## 6. Bắt đầu từ đâu (How to navigate)

1. Đọc `RULES.md` để hiểu quy ước (naming, citation, classification, Definition of Done).
2. Mở `WORKSPACE_TRACKING.md` để xem trạng thái và workstream được giao.
3. Vào folder workstream, đọc `_overview.md` **trước khi** viết các file con.
4. Sau mỗi đơn vị công việc → cập nhật `WORKSPACE_TRACKING.md`.

## 7. Quy ước nhanh (chi tiết trong `RULES.md`)

- Mọi claim phải có **citation** theo format IEEE rút gọn; ưu tiên primary source (Tier 1).
- Phân loại nội dung: `[FACT]` / `[ANALYSIS]` / `[ASSUMPTION]` / `[GAP]`.
- **Trung lập**: không tô hồng/bôi đen vendor. Thiếu data → ghi `[GAP]`, không bịa.
- Ngôn ngữ: **tiếng Việt nghiệp vụ**; thuật ngữ kỹ thuật giữ nguyên tiếng Anh.

## 8. Definition of Done — workspace level

Workspace coi là sẵn sàng trình C-Level khi:
- [ ] `01_Executive_Summary/` đầy đủ và `APPROVED`.
- [ ] `06_Strategic_Roadmap/04_coverage_matrix.md` đối chiếu xong với roadmap sản phẩm hiện hành.
- [ ] `07_Business_Case_And_Risk/` có ít nhất 3 ROI scenarios (base/upside/downside) với assumptions tường minh + risk register hoàn chỉnh.
- [ ] `08_References_Appendix/01_primary_sources.md` chứa toàn bộ T1 citation.
- [ ] Mọi `[GAP]` ở các workstream 01–07 có owner + due.
- [ ] CIO + CFO + CRO ký off `01_Executive_Summary/02_recommendations.md`.

## 9. Change log

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-05-21 | v0.1-draft | Khởi tạo workspace, draft 3 root files chờ approve | Lead Consultant |
| 2026-05-21 | v0.2-draft | Merge workstream 07 (Risk) + 08 (Financial) → `07_Business_Case_And_Risk`; renumber References → 08 | Lead Consultant |
