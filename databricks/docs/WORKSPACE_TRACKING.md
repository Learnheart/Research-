# Workspace Tracking — Databricks AI Agent Platform Research

**Workspace version**: v0.3-draft
**Last updated**: 2026-05-21
**Tracking owner**: Lead Consultant
**Update cadence**: sau mỗi đơn vị công việc hoàn thành (DoD §7.1) — tối thiểu 1 lần/ngày làm việc.

> File này là **single source of truth** cho tiến độ workspace. Mọi thay đổi status, owner, due của file/workstream **phải** phản ánh tại đây.

---

## 1. Trạng thái workstream

**Status legend**: `NOT_STARTED` · `DRAFT` · `IN_REVIEW` · `REVISION` · `APPROVED` · `BLOCKED`
**Priority legend**: `P0` critical-path · `P1` quan trọng · `P2` nice-to-have

| # | Workstream | Status | Priority | Owner | Reviewer | Due | % | Last update |
|---|---|---|---|---|---|---|---|---|
| 00 | Workspace setup (3 root files + 8 _overview) | DRAFT | P0 | Lead | CIO/CFO | 2026-05-21 | 100% | 2026-05-21 |
| 01 | Executive Summary | NOT_STARTED | P0 | TBD | CIO | TBD | 0% | – |
| 02 | Market Landscape | NOT_STARTED | P1 | TBD | Head of Strategy | TBD | 0% | – |
| 03 | Databricks Technology | NOT_STARTED | P0 | TBD | Head of Data/AI | TBD | 0% | – |
| 04 | Competitive Analysis | NOT_STARTED | P0 | TBD | Head of Data/AI | TBD | 0% | – |
| 05 | Banking/FSI Applications | NOT_STARTED | P1 | TBD | Head of Digital Banking | TBD | 0% | – |
| 06 | Strategic Roadmap | NOT_STARTED | P0 | TBD | CIO | TBD | 0% | – |
| 07 | Business Case & Risk | NOT_STARTED | P0 | TBD | CFO + CRO (dual review) | TBD | 0% | – |
| 08 | References & Appendix | NOT_STARTED | P2 | TBD | Lead | TBD | 0% | – |

**Critical path** (must be APPROVED trước khi `01_Executive_Summary` finalize): 03 → 04 → 06 → 07 → 01.

---

## 2. Tiến độ theo file

> Bảng này sẽ được expand sau khi cấu trúc workspace được approve.

### 2.1 Root files

| File | Status | Owner | Reviewer | Notes |
|---|---|---|---|---|
| README.md | DRAFT | Lead | CIO | Chờ approve cấu trúc |
| WORKSPACE_TRACKING.md | DRAFT | Lead | CIO | – |
| RULES.md | DRAFT | Lead | CIO | – |

### 2.2 Workstream files (placeholder — fill sau approve)

| Workstream | File | Status | Owner | Last update | Notes |
|---|---|---|---|---|---|
| 01 | _overview.md | DRAFT | Lead | 2026-05-21 | Scope + dàn ý 3 file con + DoD |
| 01 | 01_key_findings.md | NOT_STARTED | TBD | – | – |
| 01 | 02_recommendations.md | NOT_STARTED | TBD | – | – |
| 01 | 03_decision_framework.md | NOT_STARTED | TBD | – | – |
| 02 | _overview.md | DRAFT | Lead | 2026-05-21 | Scope + dàn ý 4 file con + DoD |
| 02 | 01_ai_agent_market_sizing.md | NOT_STARTED | TBD | – | – |
| 02 | 02_adoption_trends.md | NOT_STARTED | TBD | – | – |
| 02 | 03_regulatory_environment.md | NOT_STARTED | TBD | – | – |
| 02 | 04_demand_drivers_fsi.md | NOT_STARTED | TBD | – | – |
| 03 | _overview.md | DRAFT | Lead | 2026-05-21 | Scope + dàn ý 7 file con + so sánh G1/G2 + DoD |
| 03 | 01_mosaic_ai_agent_framework.md | NOT_STARTED | TBD | – | – |
| 03 | 02_unity_catalog_governance.md | NOT_STARTED | TBD | – | – |
| 03 | 03_model_serving_inference.md | NOT_STARTED | TBD | – | – |
| 03 | 04_vector_search_rag.md | NOT_STARTED | TBD | – | – |
| 03 | 05_agent_evaluation_mlflow.md | NOT_STARTED | TBD | – | – |
| 03 | 06_lakehouse_foundation.md | NOT_STARTED | TBD | – | – |
| 03 | 07_genie_assistant_apps.md | NOT_STARTED | TBD | – | – |
| 04 | _overview.md | DRAFT | Lead | 2026-05-21 | Scope + methodology + dàn ý 7 file (6 vendor + matrix) |
| 04 | 01_vs_snowflake_cortex.md | NOT_STARTED | TBD | – | – |
| 04 | 02_vs_aws_bedrock_sagemaker.md | NOT_STARTED | TBD | – | – |
| 04 | 03_vs_azure_ai_foundry.md | NOT_STARTED | TBD | – | – |
| 04 | 04_vs_gcp_vertex.md | NOT_STARTED | TBD | – | – |
| 04 | 05_vs_openai_anthropic_native.md | NOT_STARTED | TBD | – | – |
| 04 | 06_vs_oss_langchain_llamaindex.md | NOT_STARTED | TBD | – | – |
| 04 | 07_feature_matrix.md | NOT_STARTED | TBD | – | – |
| 05 | _overview.md | DRAFT | Lead | 2026-05-21 | Scope + dàn ý 5 file (3 region + catalog + reg) |
| 05 | 01_eu_us_landscape.md | NOT_STARTED | TBD | – | – |
| 05 | 02_china_landscape.md | NOT_STARTED | TBD | – | – |
| 05 | 03_vietnam_landscape.md | NOT_STARTED | TBD | – | – |
| 05 | 04_use_cases_catalog.md | NOT_STARTED | TBD | – | – |
| 05 | 05_regulatory_banking.md | NOT_STARTED | TBD | – | – |
| 06 | _overview.md | DRAFT | Lead | 2026-05-21 | Phase model + gating + dàn ý 5 file |
| 06 | 01_rd_phase.md | NOT_STARTED | TBD | – | – |
| 06 | 02_poc_phase.md | NOT_STARTED | TBD | – | – |
| 06 | 03_pilot_production_phase.md | NOT_STARTED | TBD | – | – |
| 06 | 04_coverage_matrix.md | NOT_STARTED | TBD | – | – |
| 06 | 05_milestones_timeline.md | NOT_STARTED | TBD | – | – |
| 07 | _overview.md | DRAFT | Lead | 2026-05-21 | Dual-track (BC + Risk) — dàn ý 8 file |
| 07 | 01_tco_model.md | NOT_STARTED | TBD | – | Business Case track |
| 07 | 02_roi_scenarios.md | NOT_STARTED | TBD | – | Business Case track |
| 07 | 03_pricing_analysis.md | NOT_STARTED | TBD | – | Business Case track |
| 07 | 04_build_vs_buy.md | NOT_STARTED | TBD | – | Business Case track |
| 07 | 05_data_residency_sovereignty.md | NOT_STARTED | TBD | – | Risk track |
| 07 | 06_model_risk_management.md | NOT_STARTED | TBD | – | Risk track |
| 07 | 07_security_threat_model.md | NOT_STARTED | TBD | – | Risk track |
| 07 | 08_compliance_mapping.md | NOT_STARTED | TBD | – | Risk track |
| 08 | _overview.md | DRAFT | Lead | 2026-05-21 | Citation/glossary/interview policy + dàn ý 4 file |
| 08 | 01_primary_sources.md | NOT_STARTED | TBD | – | – |
| 08 | 02_secondary_sources.md | NOT_STARTED | TBD | – | – |
| 08 | 03_glossary.md | NOT_STARTED | TBD | – | – |
| 08 | 04_interview_notes.md | NOT_STARTED | TBD | – | – |

---

## 3. Open Questions / Decisions cần chốt

| ID | Question | Owner | Workstream impacted | Due | Status |
|---|---|---|---|---|---|
| Q-01 | Phạm vi POC: 1 use case ưu tiên hay 3 use case song song? | CIO | 06, 08 | TBD | OPEN |
| Q-02 | Cloud target: AWS / Azure / multi-cloud? Databricks region nào? | Head of Infra | 03, 07 | TBD | OPEN |
| Q-03 | Region compliance: chấp nhận US/EU region hay bắt buộc on-prem/VN region? | CRO | 07 | TBD | OPEN |
| Q-04 | Budget envelope R&D giai đoạn 1 (12 tháng)? | CFO | 06, 08 | TBD | OPEN |
| Q-05 | Có yêu cầu so sánh với GenAI stack đang dùng nội bộ (nếu có)? | Head of Data | 04 | TBD | OPEN |
| Q-06 | Phạm vi case study Vietnam: tự khảo sát hay thuê industry analyst? | Head of Strategy | 05 | TBD | OPEN |
| Q-07 | Định nghĩa "AI Agent Platform nội bộ": ai là user (nội bộ engineer / business user / khách hàng cuối)? | CIO | 01, 03, 05 | TBD | OPEN |
| Q-08 | Mức độ vendor lock-in chấp nhận được (Unity Catalog, MLflow tracking)? | Head of Data/AI | 03, 04 | TBD | OPEN |

---

## 4. Risks & Blockers

**Severity**: `L` low · `M` medium · `H` high · `C` critical

| ID | Risk / Blocker | Workstream | Severity | Mitigation | Owner | Status |
|---|---|---|---|---|---|---|
| R-01 | Thiếu primary data về Vietnam banking AI adoption | 05 | M | Khảo sát qua industry contacts + báo cáo SBV/NHNN | TBD | OPEN |
| R-02 | Databricks pricing chính xác cần vendor quote, public list price thường outdated | 08 | H | Yêu cầu vendor quote chính thức + benchmark public reports | TBD | OPEN |
| R-03 | China regulatory landscape thay đổi nhanh (Generative AI Measures, data export control) | 05, 07 | M | Cite version + ngày hiệu lực; cross-check với 2+ T2 sources | TBD | OPEN |
| R-04 | Roadmap sản phẩm hiện tại chưa được chia sẻ đầy đủ → coverage matrix khó hoàn chỉnh | 06 | H | Xin sponsor approve access tài liệu sản phẩm nội bộ | Lead | OPEN |
| R-05 | Mosaic AI feature set đang evolve nhanh, GA vs preview không rõ ràng | 03, 04 | M | Snapshot tại date cụ thể, mark `[FACT]` kèm version | TBD | OPEN |

---

## 5. Cross-workstream dependencies

| From | To | Loại dependency |
|---|---|---|
| 03 Tech | 04 Competitive | Feature matrix chuẩn để so sánh |
| 03 Tech | 07 Risk | Threat model dựa trên architecture |
| 03 Tech | 08 Financial | Component nào tính TCO |
| 04 Competitive | 06 Roadmap | Lựa chọn vendor ảnh hưởng phasing |
| 02 Market + 05 Industry | 01 Exec Summary | Cơ sở narrative |
| 07 Risk + 08 Financial | 01 Exec Summary | Cơ sở recommendation |
| 06 Roadmap | 01 Exec Summary | Coverage matrix là appendix bắt buộc |

---

## 6. Definition of Done — tóm tắt

Chi tiết: `RULES.md §7`. Tóm tắt nhanh:
- File header đầy đủ (version, status, owner, reviewer, last updated).
- Mọi đoạn được tag `[FACT]/[ANALYSIS]/[ASSUMPTION]/[GAP]`.
- Mọi `[FACT]` có citation T1/T2/T3.
- Mọi `[GAP]` có owner + due.
- Peer review xong → status `APPROVED`.
- Cập nhật tracking ngay sau khi đổi status.

---

## 7. Change log

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-05-21 | v0.1-draft | Khởi tạo workspace, draft 3 root files chờ approve | Lead Consultant |
| 2026-05-21 | v0.3-draft | Tạo 8 workstream folder + 8 file `_overview.md` (DRAFT); cập nhật bảng tiến độ §2.2; workstream 00 setup → 100% | Lead Consultant |
