# 03. Allocation Framework — Tagging, Chargeback, Unit Economics

> **Kết luận đầu (Pyramid):**
> Không thể có chargeback / showback chính xác nếu **không có tagging discipline được enforce ngay tại điểm tạo resource và điểm gọi API**. Khung dưới đây định nghĩa: (a) **tagging taxonomy** 6 thẻ bắt buộc + 4 khuyến nghị; (b) **mô hình allocation 3 tầng** (direct / shared / overhead); (c) **unit economics** với 3 cấp số (per-invocation, per-use-case, per-business-outcome); (d) lựa chọn **showback trước, chargeback sau** — phù hợp với mức độ trưởng thành hiện tại của tổ chức.

## 1. Showback vs Chargeback — Chọn cái nào, khi nào

| Tiêu chí | Showback | Chargeback |
|---|---|---|
| **Bản chất** | Hiển thị cost cho team, không bill | Bill thật vào P&L team |
| **Mục đích chính** | Tăng visibility, tạo awareness | Buộc accountability, đẩy hành vi tối ưu |
| **Yêu cầu maturity** | Trung — chỉ cần tagging + dashboard | Cao — cần Finance buy-in + dispute process |
| **Rủi ro** | Có thể bị bỏ qua nếu không có incentive | Chargeback sai gây dispute, lost trust |
| **Phù hợp giai đoạn** | **R&D + POC** | **Production sau 6–12 tháng baseline showback** |
| **Trigger chuyển sang chargeback** | Khi cost > 0.5% OpEx của business unit (rule of thumb) HOẶC khi tagging accuracy >95% qua 2 quý liên tiếp | — |

*Nguồn: Logiciel.io "Showback or Chargeback"; Portkey "FinOps chargeback for GenAI"; Finout "GenAI Cost Allocation Guide".*

**Recommendation cho platform của chúng ta:** Bắt đầu **Showback** trong 6–9 tháng đầu, chuyển dần sang **Chargeback** cho các use case đã ổn định (xem `07_roadmap/`).

## 2. Allocation Model — 3 tầng cost pool

Một invocation tạo ra cost thuộc 3 nhóm khác nhau. Phải allocate riêng từng nhóm.

| Tầng | Bản chất | Ví dụ | Cách allocate |
|---|---|---|---|
| **Direct cost** | Có thể attribute trực tiếp tới 1 use case / 1 invocation | Token cost của 1 request, vector DB query của RAG step | Theo `use_case_id` / `agent_id` tag — 100% phân bổ |
| **Shared cost** | Dùng chung cho nhiều use case, chia theo driver | Provisioned throughput GPU endpoint phục vụ N agents, vector DB cluster | Theo driver tỷ lệ (% tokens consumed, % queries) — cập nhật rolling 30 ngày |
| **Overhead** | Platform-wide, không attribute được tới use case | Platform team FTE, FinOps tooling license, security/compliance | Allocate theo công thức chuẩn: **flat % của direct + shared** HOẶC **tỷ lệ usage** |

> **Quy tắc:** Cost report cho team luôn break ra 3 dòng (Direct / Shared / Overhead) để team hiểu cái nào họ optimize được, cái nào không.

## 3. Tagging Taxonomy

### 3.1. Tag bắt buộc (enforced — request bị reject nếu thiếu)

| Tag | Giá trị mẫu | Vai trò |
|---|---|---|
| `cost_center` | `CC-1023` (Retail Banking) | Finance attribution, chargeback target |
| `business_unit` | `retail` / `wholesale` / `risk` / `ops` / `it` | Bucket báo cáo cấp BU |
| `use_case_id` | `UC-RM-001` (RM call summary) | Đơn vị attribute nhỏ nhất về business |
| `agent_id` | `agent-rm-summary-v2` | Đơn vị attribute kỹ thuật |
| `environment` | `dev` / `staging` / `prod` | Phân biệt cost thử nghiệm vs prod |
| `owner_email` | `team-lead@org` | Người chịu trách nhiệm tối ưu |

### 3.2. Tag khuyến nghị

| Tag | Vai trò |
|---|---|
| `model_tier` | (frontier / mid / small) — cho model-tiering analytics |
| `data_classification` | (public / internal / confidential / restricted) — cho compliance allocation |
| `experiment_id` | A/B test attribution |
| `customer_segment` | Per-customer-segment unit economics (nếu áp dụng được) |

### 3.3. Cơ chế enforce (Databricks-first)

| Layer | Enforcement |
|---|---|
| **IAM / Provisioning** | Resource bị reject nếu thiếu tag bắt buộc (Cloud-native policy: AWS SCP / Azure Policy / Databricks Unity Catalog policies) |
| **AI Gateway** | Request bị reject (HTTP 400) nếu header / metadata thiếu `use_case_id` + `cost_center` |
| **CI/CD** | Pipeline check tags trước khi deploy agent |
| **Monthly audit** | Tag drift report; team có resource untagged > 3 ngày → escalate |

*Nguồn: AWS Bedrock IAM-based cost allocation (2026-04); Databricks Unity AI Gateway service policies.*

## 4. Unit Economics — 3 cấp số

Để C-Level dùng được, mỗi use case có 3 cấp số. Cấp 3 là cấp duy nhất ROI thật.

| Cấp | Metric | Công thức | Phù hợp cho |
|---|---|---|---|
| **1. Cost per invocation** | `total_cost_per_invocation` | Σ 6 lớp ở `02_cost_taxonomy/` | Engineer optimize agent |
| **2. Cost per use case-period** | `cost_per_use_case_month` = `Σ invocations × cost_per_invocation + amortized_shared + overhead` | Use-case owner báo cáo cho BU |
| **3. Cost per business outcome** | `cost_per_outcome` = `cost_per_use_case_month / successful_outcomes_per_month` | C-Level, FP&A — chỉ số duy nhất so với manual baseline |

### 4.1. Ví dụ cấp 3 (mô phỏng — cần thay bằng số thật khi có)

| Use Case | Cấp 3 metric | Giá trị mục tiêu | Manual baseline |
|---|---|---|---|
| RM call summary | $ per CRM record updated | < $0.15 [cần verify với số thật] | $2–5 (RM time × hourly rate) |
| BA — BRD draft | $ per BRD generated | < $3 [cần verify] | $200–500 (BA hours × hourly rate) |
| Customer engagement script | $ per personalized script | < $0.05 [cần verify] | $1–3 |

> **Quy tắc vàng:** Nếu cost per outcome > 20% manual baseline trong 3 tháng liên tiếp, escalate use case sang FinOps Council review (xem `05_monitoring_governance/`).

## 5. Quy trình month-end (Operating Model)

```
Day 1–3:  Auto-ingest cost data từ system.serving.* + system.billing.*
Day 3–5:  Auto-allocate Direct / Shared / Overhead theo rules
Day 5–7:  Tag drift audit; flag untagged > 1% spend
Day 7–10: Per-BU report drafted; owner reviews
Day 10:   Finance sign-off; chargeback entries posted (Production phase)
Day 11:   FinOps Council meeting (anomaly + cost outliers)
```

## 6. Dispute Process (cho Chargeback phase)

| Step | Hành động | SLA |
|---|---|---|
| 1 | BU submit dispute trong vòng 10 business days sau khi report ra | T+10 |
| 2 | Platform team replay trace; xác định nguyên nhân (tag sai / shared cost mis-allocate / cost trap) | T+15 |
| 3 | Quyết định: adjust chargeback / no-adjust + cải thiện rules | T+20 |
| 4 | Update allocation rules nếu cần | T+30 |

## 7. Liên kết với folder khác

- **L1–L6 cost layers:** xem `02_cost_taxonomy/_overview.md`
- **Forecast theo use case + BU:** xem `04_forecasting/_overview.md`
- **Anomaly detection trên tag-level spend:** xem `05_monitoring_governance/_overview.md`
- **Tooling support tagging policy:** xem `06_tooling_assessment/_overview.md`

## Nguồn

- Finout. *GenAI Cost Allocation: The Complete Guide.* finout.io/blog/genai-cost-allocation-the-complete-guide-for-engineering-and-finance-teams
- Portkey. *FinOps chargeback and how it can help GenAI platforms.* portkey.ai/blog/finops-chargeback-for-genai/
- Logiciel.io. *Showback or Chargeback: What's Working for Engineering Accountability.* logiciel.io/blog/showback-chargeback-engineering-accountability
- FinOps Foundation. *How to Build a Generative AI Cost and Usage Tracker.* finops.org/wg/how-to-build-a-generative-ai-cost-and-usage-tracker/
- AWS Blog. *Introducing granular cost attribution for Amazon Bedrock.* aws.amazon.com/blogs/machine-learning/introducing-granular-cost-attribution-for-amazon-bedrock/
- AWS Blog. *Amazon Bedrock now supports cost allocation by IAM user and role.* aws.amazon.com/about-aws/whats-new/2026/04/bedrock-iam-cost-allocation/
- nOps. *AI Cost Visibility: The Ultimate Guide.* nops.io/blog/ai-cost-visibility-the-ultimate-guide/
- Apptio. *FinOps for AI: Enabling the Next Wave of Cloud Innovation.* apptio.com/blog/finops-for-ai-enabling-the-next-wave-of-cloud-innovation/
- Drivetrain. *Unit economics for AI SaaS companies.* drivetrain.ai/post/unit-economics-of-ai-saas-companies-cfo-guide-for-managing-token-based-costs-and-margins
