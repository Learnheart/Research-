# 07. Business Case & Risk — Overview

**Workstream**: 07_Business_Case_And_Risk
**File ID**: 07-00
**Version**: v0.1
**Status**: DRAFT
**Owner**: TBD (đề xuất 2 owner — 1 cho track Business Case, 1 cho track Risk)
**Reviewer**: CFO + CRO (dual review)
**Last updated**: 2026-05-21
**Classification**: Internal

> **Mục tiêu workstream**: Cung cấp **cơ sở định lượng** (TCO/ROI/build-vs-buy) và **cơ sở rủi ro** (data residency, model risk, security, compliance) đủ chặt để CFO + CRO ký commit/decline R&D budget.
>
> **Key questions**:
> - [FACT/ANALYSIS] TCO 3-year (R&D + POC + Pilot + 12-month Production limited) của Databricks vs 2 alternative?
> - [ANALYSIS] ROI scenarios base / upside / downside có khoảng IRR / payback nào? Sensitivity với pricing và adoption?
> - [ANALYSIS] Top 5 rủi ro nào (theo severity × likelihood) cần mitigation gate trong roadmap?
>
> **Audience chính**: CFO, CRO, CIO, Internal Audit, Head of Procurement.

---

## 1. Scope

### 1.1 In-scope

- **Track Business Case** (file 01–04): TCO, ROI scenarios, pricing analysis, build-vs-buy.
- **Track Risk** (file 05–08): data residency/sovereignty, model risk management, security threat model, compliance mapping.
- 3 ROI scenarios (base / upside / downside) với assumption table tường minh.
- Risk register tối thiểu 15 mục với severity × likelihood + mitigation owner.

### 1.2 Out-of-scope

- Audited financial statements / pricing negotiation (`../README.md §2`).
- Internal audit deliverables.
- Disaster recovery runbook (chỉ note ở threat model, không viết runbook).

## 2. Strategic context

- [ANALYSIS] Đây là workstream **dual-track** — CFO và CRO có ưu tiên khác nhau (cost vs prudence). Mọi recommendation phải pass cả 2 review.
- [ASSUMPTION] Public list price Databricks (DBU + Foundation Model API + Vector Search) trên website hợp lệ làm baseline. Nếu vendor quote chính thức khác ≥30% → re-baseline TCO. Impact: high (R-02 tracking).
- [FACT] Critical-path workstream 07 → 01 (Exec Summary).

## 3. Dàn ý file con

### 3.1 Track Business Case

| File | Mục tiêu (1–2 câu) | Owner | Trạng thái |
|---|---|---|---|
| `01_tco_model.md` | TCO 3-year breakdown: compute (DBU), inference token, vector search, governance, platform team FTE, training, support; per component theo workstream 03. | TBD | NOT_STARTED |
| `02_roi_scenarios.md` | 3 scenarios base/upside/downside, mỗi scenarios có assumption table, sensitivity 2-way (pricing × adoption), IRR + payback. | TBD | NOT_STARTED |
| `03_pricing_analysis.md` | Phân tích pricing Databricks (Compute, Serving, Vector Search, Genie, MFA add-ons); commit tier discount; pricing risk forward. | TBD | NOT_STARTED |
| `04_build_vs_buy.md` | Build OSS stack vs Buy Databricks vs Buy hyperscaler — comparison theo 5 trục: capex, opex, time-to-value, talent need, exit cost. | TBD | NOT_STARTED |

### 3.2 Track Risk

| File | Mục tiêu (1–2 câu) | Owner | Trạng thái |
|---|---|---|---|
| `05_data_residency_sovereignty.md` | Yêu cầu region (VN, ASEAN, EU, US, CN) — Databricks coverage; data export control; on-prem fallback option. | TBD | NOT_STARTED |
| `06_model_risk_management.md` | Mapping với Fed SR 11-7, ECB TRIM, SBV; model lifecycle (development, validation, monitoring); LLM-specific risk (hallucination, drift, bias). | TBD | NOT_STARTED |
| `07_security_threat_model.md` | STRIDE / LINDDUN-LLM threat model cho agent platform; prompt injection, data exfil, tool abuse; control mapping. | TBD | NOT_STARTED |
| `08_compliance_mapping.md` | Mapping bảng: AI Act × DORA × GDPR × SBV × PCI-DSS × ISO 27001/42001 × NIST AI RMF — control × Databricks feature. | TBD | NOT_STARTED |

## 4. Key questions phải trả lời (chi tiết)

### Business Case

1. DBU pricing 2026 cho serverless model serving — vendor quote và list price chênh bao nhiêu?
2. Foundation Model API: pay-per-token vs provisioned throughput, ngưỡng break-even ở token volume nào?
3. Vector Search cost: scale với số embedding hay số query, công thức cụ thể?
4. Platform team FTE cần bao nhiêu cho R&D / POC / Pilot / Production — refer team-formation phase từ workstream 06?
5. ROI base case: giả định adoption 20% RM trong 12 tháng → savings bao nhiêu USD?
6. Build OSS: tổng FTE 24 tháng = bao nhiêu vs Databricks managed cost?
7. Exit cost từ Databricks: bao nhiêu tháng + bao nhiêu rework để chuyển sang Bedrock/Azure?

### Risk

8. Databricks có region tại VN/SE Asia không? Nếu không, dùng region SGP/JPN — data có vi phạm SBV/NĐ 13 không?
9. Model validation theo SR 11-7 áp dụng cho LLM agent (non-deterministic) thế nào? Tiền lệ gần đây?
10. Prompt injection: vendor có guardrail (Lakehouse Monitoring, AI Gateway) đủ chưa? Cần wrap thêm gì?
11. DORA (EU) yêu cầu cụ thể về **vendor concentration risk** với critical ICT provider — Databricks có nằm trong scope?
12. PII handling: agent log có chứa PII không, retention policy thế nào, ai access?

## 5. Cross-references (dependencies)

| Liên quan tới | Mục đích |
|---|---|
| ← `../03_Databricks_Technology/` | Component cost breakdown + region availability + security feature |
| ← `../04_Competitive_Analysis/07_feature_matrix.md` | Cơ sở build-vs-buy alternative |
| ← `../05_Banking_FSI_Applications/05_regulatory_banking.md` | Regulatory baseline cho compliance mapping |
| ← `../06_Strategic_Roadmap/` | Phase + FTE + timeline cho TCO/ROI |
| → `../01_Executive_Summary/01_key_findings.md` | Top 3 finding kinh tế + top 5 risk |
| → `../01_Executive_Summary/02_recommendations.md` | Conditional-go gate dựa trên TCO/ROI threshold |

## 6. Sources & evidence plan

| Track | Nguồn | Tier |
|---|---|---|
| Business Case | Databricks pricing page, vendor quote, AWS Bedrock pricing, Azure OpenAI pricing, OSS reference TCO (Mistral self-host blog) | T1 / T4 |
| Business Case | Gartner TCO model DSML, Forrester Total Economic Impact studies | T2 |
| Risk | Fed SR 11-7, ECB TRIM, SBV thông tư, EU AI Act, EU DORA RTS, NIST AI RMF, ISO 42001:2023, OWASP LLM Top 10 | T1 |
| Risk | MITRE ATLAS, Microsoft AI security best practice, Anthropic responsible scaling policy | T2 |
| Risk | Lakera, HiddenLayer, Robust Intelligence (vendor — chỉ làm hint) | T4 |

[GAP] Databricks region availability tại VN/SE Asia tại thời điểm planning. Owner: TBD. Due: 2026-06-10.
[GAP] Vendor quote chính thức (R-02 tracking). Owner: Head of Procurement. Due: 2026-07-31.

## 7. Open questions từ tracking (subset)

- Q-02 (Cloud target & region) — block `01_tco_model.md` và `05_data_residency_sovereignty.md`.
- Q-03 (region compliance VN) — block `05_data_residency_sovereignty.md`.
- Q-04 (budget envelope) — block `02_roi_scenarios.md` và `04_build_vs_buy.md`.

## 8. Risks & GAPs (workstream-level)

| ID | Risk | Severity | Mitigation |
|---|---|---|---|
| R-BC-01 (= R-02 tracking) | Public list price outdated, vendor quote chưa có | H | TCO v0.x có note "list price baseline", lock số ở v1.0 sau quote |
| R-BC-02 | ROI assumption phụ thuộc adoption — over-optimistic dễ skew recommendation | H | Downside scenario phải có adoption ≤10% để stress-test |
| R-Risk-01 | Model risk framework cho LLM (non-deterministic) chưa có precedent rõ ở VN | M | Adapt SR 11-7 + ECB TRIM, document cẩn thận `[ASSUMPTION]` |
| R-Risk-02 | DORA scope vendor concentration với Databricks chưa rõ status | M | Track classification "critical ICT provider" + tiền lệ ECB |
| R-Risk-03 | OWASP LLM Top 10 update nhanh, threat model lỗi thời | L | Snapshot date + section "assumptions to revisit" |

## 9. Definition of Done — workstream level

- [ ] `_overview.md` APPROVED.
- [ ] 8 file con APPROVED (4 BC + 4 Risk).
- [ ] TCO model có Excel/Markdown source-of-truth + sensitivity matrix.
- [ ] ROI ≥3 scenarios với assumption table tường minh + downside có adoption ≤10%.
- [ ] Risk register ≥15 mục, mỗi mục có severity × likelihood + owner + due.
- [ ] Compliance matrix ≥6 framework × ≥10 control.
- [ ] CFO + CRO co-sign (`../README.md §8`).
- [ ] `../WORKSPACE_TRACKING.md` cập nhật status.

## 10. Change log

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-05-21 | v0.1 | Khởi tạo overview Business Case & Risk (dual-track BC + Risk) | Lead Consultant |
