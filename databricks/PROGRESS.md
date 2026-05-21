# PROGRESS — Analyze Databricks R&D

[CẬP NHẬT: 21/05/2026]

> **Version**: v0.2 — bổ sung Phase 0.5 (Deep Research input).

> **Single source of truth** cho tiến độ phase / task của toàn dự án.
> Mọi assistant đóng góp **phải** cập nhật checkbox tương ứng ngay sau khi hoàn thành 1 task.
> Đối với file research trong `docs/`, đây là **layer tóm tắt** — chi tiết status/owner/reviewer ở `docs/WORKSPACE_TRACKING.md`.

**Quy ước checkbox**:
- `[ ]` chưa làm hoặc đang làm dở (in_progress)
- `[x]` đã hoàn thành (DONE)
- `[~]` đang IN_REVIEW (chờ reviewer ký)
- `[!]` BLOCKED (xem cột ghi chú)
- `[-]` đã huỷ / không còn relevant

---

## Phase 0 — Khởi tạo workspace (Setup)

- [x] Lưu prompt khởi tạo tại `prompts/first_insight.md`
- [x] Tạo `docs/README.md` (workspace overview, audience, structure 2-level)
- [x] Tạo `docs/RULES.md` (Big4 standard: header, tag, citation tier T1–T5, DoD)
- [x] Tạo `docs/WORKSPACE_TRACKING.md` (status, open questions, risks, dependencies)
- [x] Tạo 8 folder workstream + 8 file `_overview.md` (DRAFT v0.1)
- [x] Tạo `CLAUDE.md` (rule project-level)
- [x] Tạo `PROGRESS.md` (file này)
- [x] Tạo `PLAYBOOK.md` (lệnh + workflow)
- [ ] Sponsor (CIO) approve 3 root file `docs/{README, RULES, WORKSPACE_TRACKING}.md` (DRAFT → APPROVED)
- [ ] Sponsor approve 8 file `_overview.md` (DRAFT → APPROVED)
- [ ] Chốt 3 open question critical-path:
  - [ ] Q-01 — Phạm vi POC: 1 hay 3 use case song song
  - [ ] Q-04 — Budget envelope R&D giai đoạn 1
  - [ ] Q-07 — Định nghĩa "AI Agent Platform nội bộ": ai là user

## Phase 0.5 — Chuẩn bị input deep research (cầu nối từ Setup sang Research)

- [x] Tạo prompt `prompts/deep_research_market_tech_competitive.md` (cover 3 trục Market/Tech/Competitive × 3 region EU-US/CN/VN, dùng Claude web + extended thinking)
- [ ] Chạy prompt trên Claude web, confirm research plan
- [ ] Nhận output deep research, lưu raw tại `prompts/output/<date>_deep_research_raw.md`
- [ ] Chạy validation checklist (xem §7 của prompt file) — random check ≥10 citation, đếm tỷ lệ T1+T2
- [ ] Tách section A/B/C/D về file workstream tương ứng (mapping ở §3 của prompt file)
- [ ] (Optional) Chạy variant prompt Vietnam drill-down nếu Phần D.3 shallow
- [ ] (Optional) Chạy variant prompt Pricing & TCO nếu cần input cho workstream 07

## Phase 1 — Nghiên cứu nền tảng (Workstream 02–08)

> Critical path theo `docs/WORKSPACE_TRACKING.md §1`: 03 → 04 → 06 → 07 → 01.
> Workstream 02, 05, 08 chạy song song được, không block.

### Workstream 02 — Market Landscape

- [ ] `docs/02_Market_Landscape/01_ai_agent_market_sizing.md`
- [ ] `docs/02_Market_Landscape/02_adoption_trends.md`
- [ ] `docs/02_Market_Landscape/03_regulatory_environment.md`
- [ ] `docs/02_Market_Landscape/04_demand_drivers_fsi.md`

### Workstream 03 — Databricks Technology [P0 — critical path]

- [ ] `docs/03_Databricks_Technology/01_mosaic_ai_agent_framework.md`
- [ ] `docs/03_Databricks_Technology/02_unity_catalog_governance.md`
- [ ] `docs/03_Databricks_Technology/03_model_serving_inference.md`
- [ ] `docs/03_Databricks_Technology/04_vector_search_rag.md`
- [ ] `docs/03_Databricks_Technology/05_agent_evaluation_mlflow.md`
- [ ] `docs/03_Databricks_Technology/06_lakehouse_foundation.md`
- [ ] `docs/03_Databricks_Technology/07_genie_assistant_apps.md`

### Workstream 04 — Competitive Analysis [P0 — critical path]

- [ ] `docs/04_Competitive_Analysis/01_vs_snowflake_cortex.md`
- [ ] `docs/04_Competitive_Analysis/02_vs_aws_bedrock_sagemaker.md`
- [ ] `docs/04_Competitive_Analysis/03_vs_azure_ai_foundry.md`
- [ ] `docs/04_Competitive_Analysis/04_vs_gcp_vertex.md`
- [ ] `docs/04_Competitive_Analysis/05_vs_openai_anthropic_native.md`
- [ ] `docs/04_Competitive_Analysis/06_vs_oss_langchain_llamaindex.md`
- [ ] `docs/04_Competitive_Analysis/07_feature_matrix.md` (đầu ra ăn vào Executive Summary)

### Workstream 05 — Banking / FSI Applications

- [ ] `docs/05_Banking_FSI_Applications/01_eu_us_landscape.md`
- [ ] `docs/05_Banking_FSI_Applications/02_china_landscape.md`
- [ ] `docs/05_Banking_FSI_Applications/03_vietnam_landscape.md`
- [ ] `docs/05_Banking_FSI_Applications/04_use_cases_catalog.md`
- [ ] `docs/05_Banking_FSI_Applications/05_regulatory_banking.md`

### Workstream 06 — Strategic Roadmap [P0 — critical path]

- [ ] `docs/06_Strategic_Roadmap/01_rd_phase.md`
- [ ] `docs/06_Strategic_Roadmap/02_poc_phase.md`
- [ ] `docs/06_Strategic_Roadmap/03_pilot_production_phase.md`
- [ ] `docs/06_Strategic_Roadmap/04_coverage_matrix.md` (đối chiếu roadmap sản phẩm hiện hành)
- [ ] `docs/06_Strategic_Roadmap/05_milestones_timeline.md`

### Workstream 07 — Business Case & Risk [P0 — critical path, dual-track CFO + CRO]

Track Business Case (CFO):
- [ ] `docs/07_Business_Case_And_Risk/01_tco_model.md`
- [ ] `docs/07_Business_Case_And_Risk/02_roi_scenarios.md` (≥3 scenarios base/upside/downside)
- [ ] `docs/07_Business_Case_And_Risk/03_pricing_analysis.md`
- [ ] `docs/07_Business_Case_And_Risk/04_build_vs_buy.md`

Track Risk (CRO):
- [ ] `docs/07_Business_Case_And_Risk/05_data_residency_sovereignty.md`
- [ ] `docs/07_Business_Case_And_Risk/06_model_risk_management.md`
- [ ] `docs/07_Business_Case_And_Risk/07_security_threat_model.md`
- [ ] `docs/07_Business_Case_And_Risk/08_compliance_mapping.md`

### Workstream 08 — References & Appendix

- [ ] `docs/08_References_Appendix/01_primary_sources.md`
- [ ] `docs/08_References_Appendix/02_secondary_sources.md`
- [ ] `docs/08_References_Appendix/03_glossary.md`
- [ ] `docs/08_References_Appendix/04_interview_notes.md`

## Phase 2 — Tổng hợp Executive Summary (Workstream 01)

> Chỉ được start khi ≥3 trong 4 workstream upstream (03, 04, 06, 07) ở status IN_REVIEW.

- [ ] `docs/01_Executive_Summary/01_key_findings.md` (5–7 finding, mỗi finding có evidence pointer)
- [ ] `docs/01_Executive_Summary/02_recommendations.md` (go / no-go / conditional-go + alternatives + rationale)
- [ ] `docs/01_Executive_Summary/03_decision_framework.md` (5–7 câu hỏi C-Level stress-test recommendation)
- [ ] Cross-check coverage matrix với `docs/06_Strategic_Roadmap/04_coverage_matrix.md`
- [ ] CIO ký off (lead reviewer)
- [ ] CFO ký off (business case)
- [ ] CRO ký off (risk)

## Phase 3 — Decision Gate

- [ ] Re-snapshot Databricks docs (target: 2026-08-15) trước final review
- [ ] Tổ chức C-Level review session (CIO + CFO + CRO)
- [ ] Capture feedback từ review session vào `docs/WORKSPACE_TRACKING.md §3`
- [ ] Quyết định Go / No-Go / Conditional-Go R&D budget
- [ ] Bump workspace version v0.x → v1.0 APPROVED
- [ ] Document quyết định trong `docs/01_Executive_Summary/` change log

## Phase 4 — Hand-off sang R&D execution (nếu Go)

> Phase này nằm ngoài scope của workspace nghiên cứu nhưng để hình dung roadmap end-to-end.

- [ ] Brief team R&D dựa trên `docs/06_Strategic_Roadmap/01_rd_phase.md`
- [ ] Setup environment Databricks workspace (Terraform / IaC)
- [ ] Bắt đầu use case POC ưu tiên (theo Q-01 chốt ở Phase 0)
- [ ] Review hậu POC vào `docs/06_Strategic_Roadmap/02_poc_phase.md`

---

## Risks & Blockers đang mở

> Đồng bộ với `docs/WORKSPACE_TRACKING.md §4`. Mọi blocker phải có owner + due.

- [!] R-02 — Vendor quote Databricks chưa có (chỉ public list price) — owner: Head of Procurement — due: 2026-07-31. Block `07/01_tco_model.md`, `07/03_pricing_analysis.md`.
- [!] R-04 — Roadmap sản phẩm hiện hành chưa được share đầy đủ — owner: Lead — due: 2026-06-15. Block `06/04_coverage_matrix.md`.
- [!] R-01 — Thiếu primary data về Vietnam banking AI adoption — owner: TBD — due: 2026-06-30. Block `05/03_vietnam_landscape.md`.

---

## Change log

| Ngày | Phiên bản | Thay đổi | Tác giả |
|---|---|---|---|
| 21/05/2026 | v0.1 | Khởi tạo `PROGRESS.md` project-level — chia 4 phase (Setup, Research, Exec Synthesis, Decision Gate) + Phase 4 hand-off. Sync với `docs/WORKSPACE_TRACKING.md` snapshot 21/05/2026. | Lead Consultant |
| 21/05/2026 | v0.2 | Bổ sung Phase 0.5 — chuẩn bị input deep research; tick task tạo `prompts/deep_research_market_tech_competitive.md`. | Lead Consultant |

### Citations

Không có citation — file này là tracking checkbox, không chứa thông tin kỹ thuật/thị trường/quy định. Toàn bộ references gốc nằm trong file workstream tương ứng.
