# 06. Strategic Roadmap — Overview

**Workstream**: 06_Strategic_Roadmap
**File ID**: 06-00
**Version**: v0.1
**Status**: DRAFT
**Owner**: TBD
**Reviewer**: CIO (lead) · Head of Data/AI · CFO (cho phần budget envelope)
**Last updated**: 2026-05-21
**Classification**: Internal

> **Mục tiêu workstream**: Phác thảo lộ trình triển khai cấp cao **R&D → POC → Pilot → Production** trong 18–24 tháng, kèm **coverage matrix** đối chiếu với roadmap sản phẩm hiện hành — đầu ra là tài liệu nhập cho Executive Summary.
>
> **Key questions**:
> - [ANALYSIS] Pha R&D nên đầu tư bao nhiêu (% tổng budget) trước khi commit POC?
> - [ANALYSIS] Use case POC nào có ROI rõ nhất + risk thấp nhất để de-risk decision?
> - [ANALYSIS] Coverage matrix: phần nào của roadmap sản phẩm hiện hành được Databricks **cover đầy đủ / cover một phần / không cover**?
>
> **Audience chính**: CIO, CFO, Head of Data/AI.

---

## 1. Scope

### 1.1 In-scope

- 4 phase: **R&D** (foundation, team, governance), **POC** (1–3 use case), **Pilot** (limited production, 1 BU), **Production** (multi-BU, multi-region).
- Gating criteria pass/fail cho mỗi phase chuyển tiếp.
- **Coverage matrix** Databricks capability × product roadmap line item.
- Timeline + milestone + dependency, không sa vào task-level.
- Resourcing **mức envelope** (FTE range), không headcount cụ thể.

### 1.2 Out-of-scope

- Project plan cấp task (Jira / MS Project) — workspace này là strategic, không operational.
- Change management plan chi tiết (`../README.md §2`).
- Vendor contract terms (`../07_Business_Case_And_Risk/03_pricing_analysis.md`).

## 2. Strategic context

- [ANALYSIS] Roadmap là **kết quả tổng hợp** của 02 (timing thị trường) + 03 (capability) + 04 (vendor choice) + 05 (use case fit) + 07 (budget/risk). Không thể draft trước khi 4 workstream upstream ở `IN_REVIEW`.
- [ASSUMPTION] Giả định budget envelope giai đoạn 1 (R&D 6 tháng) ≤ USD 1M; pilot/production sẽ re-baseline. Nếu sai (budget < 500K) → POC bị giới hạn về 1 use case duy nhất. Impact: high.
- [FACT] Critical-path workstream 06 → 01 (Exec Summary) tại `../WORKSPACE_TRACKING.md §1`.

## 3. Dàn ý file con

| File | Mục tiêu (1–2 câu) | Owner | Trạng thái |
|---|---|---|---|
| `_overview.md` | (file này) Phase model, gating, dàn ý 5 file. | TBD | DRAFT |
| `01_rd_phase.md` | R&D phase 0–6 tháng: team formation, platform setup, governance baseline, learning agenda, exit criteria sang POC. | TBD | NOT_STARTED |
| `02_poc_phase.md` | POC 6–12 tháng: 1–3 use case ưu tiên (lấy từ workstream 05), KPI pass/fail, exit criteria sang Pilot. | TBD | NOT_STARTED |
| `03_pilot_production_phase.md` | Pilot (12–18 tháng) → Production (18–24+ tháng): scaling pattern, SRE/MLOps maturity, multi-BU rollout. | TBD | NOT_STARTED |
| `04_coverage_matrix.md` | Matrix capability Databricks × line item product roadmap hiện tại; phân loại Full/Partial/None + alternative. | TBD | NOT_STARTED |
| `05_milestones_timeline.md` | Gantt cấp cao, milestone list, dependency map, key decision point cho C-Level. | TBD | NOT_STARTED |

## 4. Phase model — định nghĩa thống nhất

| Phase | Thời lượng điển hình | Mục tiêu | Kết thúc bằng |
|---|---|---|---|
| R&D | 0–6 tháng | Team + platform baseline + governance + learning | Capability proven trên 1 sandbox use case + go/no-go POC |
| POC | 6–12 tháng | 1–3 use case end-to-end, có user thật (limited) | KPI pass/fail; decision đầu tư Pilot |
| Pilot | 12–18 tháng | 1 use case lên production limited (1 BU, 1 region) | SLA met 3 tháng liên tiếp; security + compliance signed off |
| Production | 18–24+ tháng | Scale multi-BU, multi-region, multi-use-case | Platform self-service cho team |

[ANALYSIS] Mỗi phase có gating criteria độc lập — không bị mặc định "thành công tự động chuyển phase".

## 5. Key questions phải trả lời (chi tiết)

1. POC scope: 1 use case (deep) hay 3 use case (broad) — tradeoff de-risk vs learning velocity? (= Q-01)
2. Use case POC nào: RM copilot, fraud co-pilot, RAG knowledge base hay AML investigation assistant?
3. Pilot region: VN-only hay multi-region để test data residency từ đầu? (= Q-03)
4. Foundation model: 1 vendor (Anthropic Claude qua Bedrock/Databricks) hay multi (kèm Llama on-prem fallback)?
5. Team formation: build internal vs hire consulting partner vs hybrid — phasing thế nào?
6. Governance: Unity Catalog policy framework cần ready ở phase nào — pre-R&D hay sau?
7. Exit criteria POC → Pilot: KPI ngưỡng nào (accuracy, latency, user satisfaction, complaint, cost-per-request)?
8. Coverage matrix: roadmap sản phẩm hiện tại có bao nhiêu line item AI-related, Databricks cover được bao nhiêu %?

## 6. Cross-references (dependencies)

| Liên quan tới | Mục đích |
|---|---|
| ← `../03_Databricks_Technology/` | Capability set vào coverage matrix |
| ← `../04_Competitive_Analysis/07_feature_matrix.md` | Decision tree alternative cho line item "None" coverage |
| ← `../05_Banking_FSI_Applications/04_use_cases_catalog.md` | Use case priority vào `02_poc_phase.md` |
| ← `../07_Business_Case_And_Risk/02_roi_scenarios.md` | Budget envelope + ROI gate vào gating criteria |
| → `../01_Executive_Summary/02_recommendations.md` | Recommendation phasing + budget |
| → `../01_Executive_Summary/03_decision_framework.md` | Gating criteria là input cho framework |
| Internal | Roadmap sản phẩm hiện hành (cần sponsor approve access — xem R-04 tracking) |

## 7. Sources & evidence plan

- Internal: roadmap sản phẩm hiện tại (Confidential — cần approve access).
- External: Databricks reference architecture, McKinsey "Building the AI-Bank of the Future" (T2), BCG "AI at Scale".
- Best practice phasing: Google "MLOps Maturity Model", Microsoft "AI Maturity Model".
- Case study scaling: JPMorgan AI investor day, BBVA digital transformation reports.

[GAP] Roadmap sản phẩm nội bộ chưa được chia sẻ đầy đủ (= R-04). Owner: Lead. Due: 2026-06-15.

## 8. Open questions từ tracking (subset)

- Q-01 (POC scope) — block `02_poc_phase.md`.
- Q-04 (budget envelope) — block `01_rd_phase.md` và `02_poc_phase.md`.
- Q-07 ("AI Agent Platform" là gì) — block tất cả 5 file (định nghĩa user).

## 9. Risks & GAPs

| ID | Risk | Severity | Mitigation |
|---|---|---|---|
| R-Road-01 (= R-04 tracking) | Roadmap sản phẩm chưa share đầy đủ → coverage matrix khó hoàn chỉnh | H | Sponsor approve access; nếu không kịp → ship phiên bản v1.0 với note `[GAP]` rõ phần thiếu |
| R-Road-02 | Phase model bị "ép" để fit budget thay vì fit feasibility | M | Gating criteria độc lập với budget; CFO + CIO đồng review |
| R-Road-03 | Vendor release cadence ngắn (~2 tuần) làm roadmap obsolete | M | Roadmap v1.0 có note "review cadence quý"; viết section "assumptions to revisit" |

## 10. Definition of Done — workstream level

- [ ] `_overview.md` APPROVED.
- [ ] 5 file con APPROVED.
- [ ] Phase model + gating criteria thống nhất giữa 5 file (không mâu thuẫn).
- [ ] Coverage matrix đầy đủ, ≥80% line item roadmap được phân loại Full/Partial/None.
- [ ] Timeline có ≥5 milestone với owner + due quy về RACI.
- [ ] CIO + CFO ký off `02_recommendations.md` ở workstream 01.
- [ ] `../WORKSPACE_TRACKING.md` cập nhật status.

## 11. Change log

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-05-21 | v0.1 | Khởi tạo overview Strategic Roadmap | Lead Consultant |
