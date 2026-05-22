# Cost Model cho Agent Platform — Master Index

> **Mục tiêu:** FinOps cho Agentic AI nội bộ, bao phủ 3 bài toán cốt lõi:
> (1) **Cost accuracy & allocation** tới từng agent / use case / team,
> (2) **Forecasting** scenario-based,
> (3) **Monitoring & governance** chủ động (pre-spend controls).

---

## ⚡ 30-sec navigation

| Bạn cần | Đi tới | Một dòng |
|---|---|---|
| Tra **đơn giá** của 1 model | [`01_rate_card.md`](01_rate_card.md) | Bảng giá per-modality × per-provider; refresh tháng |
| Hiểu **cost của 1 invocation** | [`02_cost_formula.md`](02_cost_formula.md) | 6 lớp MECE + công thức multimodal-ready |
| **Dự báo** spend tháng/quý/năm | [`03_forecasting.md`](03_forecasting.md) | Driver-based + 3 scenarios + sensitivity |
| **Cảnh báo** budget / anomaly | [`04_monitoring.md`](04_monitoring.md) | 3-tier (pre / real-time / post-spend) + Council |
| **Forecast per service** (cost per user × service) | [`05_per_service_forecasting.md`](05_per_service_forecasting.md) | BUV chuẩn hoá (doc / 1k chars / slide) + công thức 3 service |
| Context phụ (market / allocation / tooling / roadmap) | [`_appendix/`](_appendix/) | Reference — không phải đường găng |

---

## Công thức một dòng (link giữa các file)

```
cost = Σ_modality units[m] × price[m]    × (1 + retry_mult)   ──┐
                  └──[01_rate_card]──┘                          │
     + non_token_overhead (L2..L6)                              │── [02_cost_formula]
                  └──────────[02_cost_formula]──────────────────┘
                                       │
                  forecast = drivers × cost                  ── [03_forecasting]
                                       │
                  alert    = actual vs forecast / budget     ── [04_monitoring]
```

---

## Cách đọc

| Audience | Ưu tiên |
|---|---|
| **Engineering / Platform team** | `02_cost_formula` → `01_rate_card` → `04_monitoring` |
| **Finance / FP&A** | `03_forecasting` → `_appendix/allocation` → `04_monitoring` §4 |
| **Stakeholder C-Level** | `_appendix/roadmap` §0 (Executive 1-pager) |
| **Lần đầu tiếp cận** | README → `02_cost_formula` 30-sec card → tuỳ nhu cầu |

---

## Cấu trúc thư mục

```
cost_model/
├── README.md                   ← bạn đang đọc
├── 01_rate_card.md             ← đơn giá per-modality × provider
├── 02_cost_formula.md          ← taxonomy 6 lớp + công thức tổng
├── 03_forecasting.md           ← driver-based + 3 scenarios
├── 04_monitoring.md            ← alerts + budget enforcement + Council
├── 05_per_service_forecasting.md ← BUV per service + cost per user
└── _appendix/
    ├── market_landscape.md     ← industry benchmark, FinOps Foundation framework
    ├── allocation.md           ← tagging taxonomy, chargeback vs showback, unit econ
    ├── tooling.md              ← build vs buy, vendor comparison, stack đề xuất
    └── roadmap.md              ← R&D → POC → Prod (12 tháng) + C-Level talking points
```

---

## Pyramid Summary (insight cấp 1)

| Insight | Ý nghĩa với platform của chúng ta |
|---|---|
| **63% enterprise đã track AI spend (2025), tăng từ 31% (2024)** — FinOps Foundation | Cost management cho AI là tiên quyết để scale |
| **Token cost chỉ chiếm 30–50% tổng chi phí vận hành agent** — Galileo, Acropolium | Mọi cost model dừng ở tokens sẽ thiếu 50–70% bức tranh |
| **Silent retries có thể nhân token usage 2–5×** — Agents Arcade | Observability ở orchestration layer là bắt buộc |
| **TTS/video dominate cost trong task multimodal** ([[01_rate_card]] §6) | Modality budget policy = pre-spend control hiệu quả nhất cho audio use case |
| **Chỉ 14.2% tổ chức đạt FinOps "Run" maturity** — FinOps Foundation 2025 | Window dẫn trước vẫn rộng |

---

## Quy ước

- Tiếng Việt; thuật ngữ kỹ thuật giữ tiếng Anh.
- Nguồn trích `(Tên tổ chức, năm)`; link đầy đủ ở mục **Nguồn** cuối mỗi file.
- Nội dung chưa chắc đánh dấu **[cần verify]**.
- **Bảng > prose** — ưu tiên scannable.
- Mỗi file top-level mở bằng **⚡ 30-sec card** trước nội dung chi tiết.
- Link giữa file dùng `[[name]]` (Obsidian-style) để dễ navigate.

---

## Trạng thái

| File | Trạng thái | Ghi chú |
|---|---|---|
| `01_rate_card.md` | ✅ v1 (2026-05-21) — populated với giá list-price | Finance cần verify trước chargeback |
| `02_cost_formula.md` | ✅ v1 — multimodal-ready | Validate với platform team Phase 1 |
| `03_forecasting.md` | ✅ v1 — methodology | Calibrate với baseline data Phase 2 |
| `04_monitoring.md` | ✅ v1 — operating model | Council charter cần executive endorse |
| `05_per_service_forecasting.md` | ✅ v1 (2026-05-22) — BUV framework | Cần calibrate sub-dim weights với data thật (Phase 4) |
| `_appendix/*` | ✅ v1 — research baseline | Refresh quarterly |
