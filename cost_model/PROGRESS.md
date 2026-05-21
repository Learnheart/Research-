# PROGRESS — Cost Model Research

[CẬP NHẬT: 21/05/2026]

> **Mục đích:** Checklist tracking tất cả phase + task của bài nghiên cứu Cost Model cho Agent Platform. Đánh dấu `[ ]` cho task chưa làm, `[x]` cho task hoàn thành. Mỗi task hoàn thành ghi rõ ngày DD/MM/YYYY.

---

## ⚡ Tổng quan trạng thái

| Phase | Mục tiêu | Trạng thái | Hoàn thành |
|---|---|---|---|
| **Phase 0** | Initial research baseline | ✅ DONE | 20/05/2026 |
| **Phase 1** | Simplify cấu trúc & multimodal extension | ✅ DONE | 21/05/2026 |
| **Phase 2** | Governance setup (rules + tracking) | 🔄 IN PROGRESS | — |
| **Phase 3** | Rate card validation (Finance) | ⏳ PENDING | — |
| **Phase 4** | Methodology calibration với baseline data | ⏳ PENDING | — |
| **Phase 5** | Monitoring deployment | ⏳ PENDING | — |
| **Phase 6** | Chargeback go-live | ⏳ PENDING | — |
| **Phase 7** | FinOps Council BAU | ⏳ PENDING | — |

---

## Phase 0 — Initial baseline ✅

- [x] Research baseline 7 chủ đề (market, taxonomy, allocation, forecast, monitoring, tooling, roadmap) — 20/05/2026
- [x] Tạo 7 folder `01_..._07_` với `_overview.md` mỗi folder — 20/05/2026

---

## Phase 1 — Simplify & multimodal extend ✅

- [x] Đơn giản hoá từ 7 folder → 4 file flat + `_appendix/` — 21/05/2026
- [x] Tạo `01_rate_card.md` với giá thật cho 5 provider × 9 modality — 21/05/2026
- [x] Generalize công thức cost từ token-only → multimodal-ready (Σ per modality) — 21/05/2026
- [x] Thêm ⚡ 30-sec card ở 4 file top-level — 21/05/2026
- [x] Refactor `04_forecasting` → `03_forecasting.md` với per-modality blended rate — 21/05/2026
- [x] Refactor `05_monitoring` → `04_monitoring.md` với modality budget policy — 21/05/2026
- [x] Move 4 file phụ vào `_appendix/` (market, allocation, tooling, roadmap) — 21/05/2026
- [x] Xoá 7 folder legacy `01_..._07_` — 21/05/2026

---

## Phase 2 — Governance setup (đang làm) 🔄

### Hạ tầng quy tắc

- [x] Tạo `CLAUDE.md` — 8 section quy tắc tác giả — 21/05/2026
- [x] Tạo `PROGRESS.md` — checklist này — 21/05/2026
- [x] Tạo `PLAYBOOK.md` — commands & workflow — 21/05/2026

### Compliance retrofit cho file hiện có

- [ ] Audit `README.md` — bổ sung `[CẬP NHẬT: DD/MM/YYYY]` + citations cuối section
- [ ] Audit `01_rate_card.md` — citations từng section thay vì gộp cuối file
- [ ] Audit `02_cost_formula.md` — citations từng section
- [ ] Audit `03_forecasting.md` — citations từng section
- [ ] Audit `04_monitoring.md` — citations từng section
- [ ] Audit `_appendix/market_landscape.md` — compliance toàn bộ
- [ ] Audit `_appendix/allocation.md` — compliance toàn bộ
- [ ] Audit `_appendix/tooling.md` — compliance toàn bộ
- [ ] Audit `_appendix/roadmap.md` — compliance toàn bộ

### Version control

- [ ] `git init` trong `cost_model/` (xem [[PLAYBOOK]] §6)
- [ ] Tạo `.gitignore` (loại trừ tạm thời, `.DS_Store`, `*.bak`)
- [ ] Initial commit baseline v1
- [ ] Branching strategy documented (main + feature branches)

---

## Phase 3 — Rate card validation (cần Finance) ⏳

- [ ] Cross-check rate card với invoice 2 tháng gần nhất; flag delta > 5%
- [ ] Verify Bedrock / Azure markup so với list-price provider
- [ ] Document committed-use discount / enterprise pricing (nếu có)
- [ ] Convert `01_rate_card.md` → `rate_card.yaml` + render markdown để CI diff
- [ ] Setup scrape job hàng tháng từ pricing pages top-5 provider (xem [[PLAYBOOK]] §3)
- [ ] Define dispute resolution process cho price discrepancy

---

## Phase 4 — Methodology calibration với baseline data ⏳

- [ ] Thu thập 60–90 ngày cost data từ `system.serving.endpoint_usage`
- [ ] Calibrate `retry_multiplier` cho 2–3 use case pilot
- [ ] Calibrate `non_token_overhead` (L2..L6) từ allocation engine output
- [ ] Run scenario forecast (Conservative / Base / Aggressive) với data thật
- [ ] Đo MAPE month-ahead < 25% (target POC) cho 3 use case
- [ ] Validate công thức multimodal với use case có image/audio
- [ ] Document sensitivity ranking với data thực

---

## Phase 5 — Monitoring deployment ⏳

- [ ] Cấu hình budget policy ở Unity AI Gateway (per-user / per-UC / per-BU)
- [ ] Deploy alert P1–P4 với threshold ở [[04_monitoring]] §3
- [ ] Tạo Alert runbook (response steps cho P1/P2)
- [ ] Modality budget policy (cap `audio_out_chars`, `video_out_sec`)
- [ ] Anomaly detection v1 (static threshold) → v2 (rolling baseline) → v3 (ML)
- [ ] Test fire-drill cho P1 alert chain

---

## Phase 6 — Chargeback go-live ⏳

- [ ] Showback ổn định ≥ 6 tháng cho ≥ 3 use case
- [ ] Tag compliance ≥ 95% qua 2 quý liên tiếp
- [ ] Dispute process documented + SLA test (xem [[_appendix/allocation]] §6)
- [ ] Chargeback ledger entries → ERP integration
- [ ] Finance sign-off cho cost-center mapping

---

## Phase 7 — FinOps Council BAU ⏳

- [ ] Council charter signed bởi executive sponsor
- [ ] Bi-weekly meeting cadence ≥ 4 lần đã chạy
- [ ] Quarterly cost governance scorecard template
- [ ] Year-1 ROI report cho Board
- [ ] Re-assess commercial FinOps tooling (Finout / Apptio) khi spend > $1M/quý

---

## Backlog / Ý tưởng chưa schedule

- [ ] Cost-per-business-outcome benchmark cho 3 use case (RM summary, BRD draft, customer script)
- [ ] Build vs Buy reassessment khi spend > $1M/quý
- [ ] Multi-cloud unified view (nếu mở rộng AWS + Azure ngoài Databricks)
- [ ] Carbon cost overlay (FinOps for AI Green)
- [ ] Audit log retention policy cho compliance
- [ ] Tự động hoá rate card scrape (Phase 3 manual → Phase 4 automated)
- [ ] Integration với existing Apptio TBM (nếu org đã có)

---

## Risk register (top 5 — đồng bộ với [[_appendix/roadmap]] §6)

| # | Risk | Status mitigation |
|---|---|---|
| 1 | Tagging discipline thấp ở team adoption | Pending — phụ thuộc Phase 5 |
| 2 | BU pushback chargeback (dispute storm) | Pending — phụ thuộc Phase 6 |
| 3 | Forecast accuracy thấp do data nhiễu | Pending — phụ thuộc Phase 4 |
| 4 | Vendor pricing thay đổi đột biến | Mitigation: scrape monthly + alert (Phase 3) |
| 5 | Headcount không đủ → BAU bỏ rơi | Pending — phụ thuộc Phase 7 |

---

## Citations

- FinOps Foundation. *State of FinOps 2025.* data.finops.org/2025-report/ (truy cập 21/05/2026)
- FinOps Foundation. *FinOps for AI Overview.* finops.org/wg/finops-for-ai-overview/ (truy cập 21/05/2026)
- Atlassian. *How to write a project status report.* atlassian.com/work-management/project-management/project-status-report (tham chiếu phong cách checklist)

---

## Liên kết

- Quy tắc tác giả: [[CLAUDE]]
- Commands & workflow: [[PLAYBOOK]]
- Master index: [[README]]
- Roadmap chi tiết theo phase: [[_appendix/roadmap]]
