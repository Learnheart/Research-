# 01. Executive Summary — Overview

**Workstream**: 01_Executive_Summary
**File ID**: 01-00
**Version**: v0.1
**Status**: DRAFT
**Owner**: TBD
**Reviewer**: CIO (lead) · CFO + CRO (co-review)
**Last updated**: 2026-05-21
**Classification**: Internal

> **Mục tiêu workstream**: Cô đọng toàn bộ phát hiện từ workstream 02–07 thành tài liệu ≤10 trang cho C-Level, đủ cơ sở để **commit / decline / defer** R&D budget cho Databricks AI Agent Platform.
>
> **Key questions**:
> - [ANALYSIS] Databricks có đủ năng lực kỹ thuật + thương mại để làm nền tảng AI Agent nội bộ của ngân hàng không?
> - [ANALYSIS] Lộ trình R&D → POC → Production khả thi trong horizon 18–24 tháng với budget envelope nào?
> - [ANALYSIS] Build-on-Databricks vs Build-on-(competitor) vs Buy-managed-service — recommendation và rationale?
>
> **Audience chính**: CIO, CFO, CRO (xem `../README.md §3`).

---

## 1. Scope

### 1.1 In-scope

- Tóm tắt **top 5 findings** từ workstream 02–07, mỗi finding có 1 evidence pointer.
- **Recommendation** dạng go / no-go / conditional-go với điều kiện gates rõ ràng.
- **Decision framework** giúp C-Level tự kiểm tra logic recommendation.
- **Coverage matrix snapshot** (full version tại `../06_Strategic_Roadmap/04_coverage_matrix.md`).
- **Risk dashboard top 5** (full register tại `../07_Business_Case_And_Risk/`).

### 1.2 Out-of-scope

- Phân tích kỹ thuật chi tiết (đẩy sang workstream 03).
- Tài chính chi tiết (đẩy sang workstream 07).
- Implementation plan ở mức task (đẩy sang workstream 06 sub-files).

## 2. Strategic context

- [FACT] Workspace này phục vụ quyết định **R&D investment** chứ chưa phải procurement [README §1].
- [ANALYSIS] Vì là tài liệu chốt narrative, Executive Summary chỉ được finalize **sau khi** 03 → 04 → 06 → 07 đã ở status `APPROVED` (critical path tại `../WORKSPACE_TRACKING.md §1`).
- [ASSUMPTION] Giả định C-Level dành ≤30 phút để đọc + 60 phút briefing. Nếu sai (chỉ có 15 phút) → cần thêm 1 file `00_one_pager.md`. Impact: low.

## 3. Dàn ý file con

| File | Mục tiêu (1–2 câu) | Owner | Trạng thái |
|---|---|---|---|
| `_overview.md` | (file này) Định nghĩa scope, dàn ý, DoD cho workstream Exec Summary. | TBD | DRAFT |
| `01_key_findings.md` | Top 5–7 finding từ 02–07, mỗi finding 1 đoạn + evidence pointer. Không tự sinh data, chỉ trích. | TBD | NOT_STARTED |
| `02_recommendations.md` | Recommendation chính (go / no-go / conditional-go) + 2–3 alternatives bị loại bỏ + rationale. | TBD | NOT_STARTED |
| `03_decision_framework.md` | Khung 5–7 câu hỏi C-Level dùng để stress-test recommendation; kèm gating criteria cho mỗi giai đoạn R&D/POC/Pilot. | TBD | NOT_STARTED |

## 4. Key questions phải trả lời (chi tiết)

1. Databricks Mosaic AI Agent Framework đã GA hay vẫn preview tại thời điểm quyết định? Impact lên SLA cam kết được?
2. Vendor lock-in mức nào? Có exit path nào khả thi trong 24–36 tháng?
3. TCO 3-year vs 2 alternatives (Azure AI Foundry, OSS stack) chênh bao nhiêu? Sensitivity đến đâu thì đảo chiều?
4. Có rào cản compliance / data residency nào với SBV / NHNN, ECB, OCC làm Databricks không khả thi tại region đang xét?
5. POC định nghĩa thành công bằng metric gì? Threshold pass/fail?
6. Phần nào của roadmap sản phẩm hiện tại được Databricks **cover hoàn toàn / cover một phần / không cover**?
7. Nếu defer 6–12 tháng, chi phí cơ hội là gì (so với deploy ngay với stack alternative)?

## 5. Cross-references (dependencies)

| Nguồn input (workstream khác) | Mục đích sử dụng trong 01 |
|---|---|
| `../02_Market_Landscape/01_ai_agent_market_sizing.md` | Bối cảnh narrative — vì sao đầu tư bây giờ |
| `../03_Databricks_Technology/` (toàn bộ) | Cơ sở claim về capability + maturity |
| `../04_Competitive_Analysis/07_feature_matrix.md` | Cơ sở so sánh ngang |
| `../05_Banking_FSI_Applications/` (3 region) | Use case fit + regulatory fit |
| `../06_Strategic_Roadmap/04_coverage_matrix.md` | Đối chiếu roadmap sản phẩm hiện tại |
| `../07_Business_Case_And_Risk/02_roi_scenarios.md` | Cơ sở định lượng cho recommendation |
| `../07_Business_Case_And_Risk/` (risk register) | Top 5 risks vào executive summary |

[GAP] Chưa rõ liệu CIO muốn 1 file PDF tổng hợp hay 3 file riêng. Owner: Lead. Due: 2026-05-28.

## 6. Sources & evidence plan

- Workstream này **không** generate primary data — chỉ refer.
- Mọi `[FACT]` trong 01_key_findings.md phải có pointer dạng `[3.2.1]` tới file gốc + reference number tại file gốc.
- Trích lại nguyên văn số liệu, không paraphrase (tránh citation drift — `../RULES.md §8`).

## 7. Open questions từ tracking (subset)

Liên quan trực tiếp: **Q-01** (POC scope), **Q-04** (budget envelope), **Q-07** ("AI Agent Platform" là gì — định nghĩa user). Cả 3 phải chốt **trước khi** viết `02_recommendations.md`. Xem `../WORKSPACE_TRACKING.md §3`.

## 8. Risks & GAPs

| ID | Risk | Severity | Mitigation |
|---|---|---|---|
| R-Exec-01 | Recommendation phụ thuộc 03/04/06/07 chưa APPROVED — viết sớm sẽ phải rework | M | Chỉ start drafting khi ≥3 trong 4 workstream upstream ở `IN_REVIEW` |
| R-Exec-02 | Stakeholder mâu thuẫn (CFO ưu tiên cost, CRO ưu tiên risk, CIO ưu tiên speed) → recommendation bị challenge | H | Decision framework (`03_decision_framework.md`) đưa cả 3 trục, tránh đơn lý do |
| [GAP] | Chưa có template slide deck cho C-Level briefing | – | Owner: Lead. Due: trước final review |

## 9. Definition of Done — workstream level

- [ ] `_overview.md` APPROVED.
- [ ] 3 file con (`01`, `02`, `03`) đều APPROVED, mỗi file ≤4 trang.
- [ ] Mọi finding/recommendation có pointer tới evidence ở 02–07.
- [ ] CIO + CFO + CRO ký off `02_recommendations.md` (xem `../README.md §8`).
- [ ] Coverage matrix snapshot đã sync với `../06_Strategic_Roadmap/04_coverage_matrix.md` (cùng version).
- [ ] `../WORKSPACE_TRACKING.md` reflect status `APPROVED` cho cả 4 file.

## 10. Change log

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-05-21 | v0.1 | Khởi tạo overview, định nghĩa scope + dàn ý + DoD | Lead Consultant |
