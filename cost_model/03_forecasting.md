# 03. Forecasting — Driver-based + Scenario Modeling

> **Insight cốt lõi:** Forecast AI bằng **point-estimate là sai phương pháp** — usage tăng phi tuyến, vendor pricing thay đổi 2–4 lần/năm, retry/reflection có cost-tail dày. Phương pháp đúng: **driver-based + 3 scenarios + rolling refresh hàng tháng**.

---

## ⚡ 30-sec card

```
forecast_cost[t] = active_users[t]
                 × invocations_per_user[t]
                 × avg_units_per_invocation[t][modality]
                 × $_per_unit[t][modality, model]   # blended từ rate card
                 × retry_multiplier[t]
                 + non_token_overhead[t]            # L2..L6
```

**3 scenarios — chọn theo mục đích:**

| Mục đích | Scenario | Tại sao |
|---|---|---|
| Board budget | **Base + buffer 20%** | Cân giữa likely và safety |
| CFO stress test | **Aggressive** | Worst case cash reserve |
| ROI baseline | **Conservative** | Show ROI dương cả case xấu |
| GPU capacity | **Aggressive** | Tránh stockout |

**Top sensitivity (lớn → nhỏ):**

```
1. retry_multiplier        ±30–60%  ← Engineering kiểm soát được
2. avg_tokens_per_inv      ±20%     ← Engineering kiểm soát được
3. invocations_per_user    ±20%     ← Business adoption
4. active_users            ±20%     ← Business adoption
5. model_mix (frontier %)  ±15–25%  ← Procurement/Strategy
6. $_per_unit              ±20%     ← Procurement/Strategy
```

→ Engineering attention: **driver 1–2 trước**.

---

## §1. Vì sao point-estimate fail

| Hiện tượng | Hệ quả lên forecast |
|---|---|
| **Adoption hockey-stick** — pilot OK → BU khác xin onboard | Linear forecast underestimate 3–10× |
| **Model swap mid-quarter** — release mới (Sonnet → Opus) giảm latency 30% nhưng cost +50% | Variance ±40% |
| **Silent retry storm** — fail downstream → token ×2–5 mà user traffic không đổi | Forecast `users × tokens` sai 2–5× |
| **Prompt drift** — team thêm vài câu system prompt → tokens/call +20% | Sai vài % nhưng compound |
| **Vendor price change** — frontier model giảm 30–50%/năm | Lock giá cũ → overestimate |
| **Modality shift** — chuyển từ text-only sang multimodal | Driver model thiếu term → underestimate |

*Nguồn: Drivetrain; Galileo; FinOps Foundation 2025.*

---

## §2. Driver-based Forecast Model

### 2.1. Công thức (multimodal-aware)

```
forecast_cost[t] = (
    active_users[t]                       # D1 — adoption
  × invocations_per_user[t]               # D2 — engagement
  × Σ_m ( avg_units[m, t] × $_per_unit[m, model, t] )  # D3+D4 — usage × price per modality
  × retry_multiplier[t]                   # D5 — quality/reliability
) + non_token_overhead[t]                 # D6 — L2..L6
```

### 2.2. Driver definitions

| # | Driver | Định nghĩa | Cách thu | Refresh |
|---|---|---|---|---|
| D1 | `active_users[t]` | DAU/MAU thực dùng agent | AI Gateway audit log | Tuần |
| D2 | `invocations_per_user[t]` | Avg/user/period | `system.serving.endpoint_usage` group by user | Tuần |
| D3 | `avg_units[m, t]` | Trung bình theo modality (tokens / images / audio sec / ...) | Endpoint usage + OTel multimodal fields | Tuần |
| D4 | `$_per_unit[m, model, t]` | Blended rate theo model mix × modality | [[01_rate_card]] × model mix % | Tháng / khi vendor đổi giá |
| D5 | `retry_multiplier[t]` | `total_inference_calls / user_facing_requests` | OTel traces | Tuần |
| D6 | `non_token_overhead[t]` | L2+L3+L4+L5+L6 trong [[02_cost_formula]] | Allocation engine | Tháng |

> **Khác biệt vs v1:** D3+D4 tách thành **per-modality**, không phải scalar `$_per_token`. Cho phép forecast use case multimodal (image-in/audio-out, RAG, vision agent) chính xác.

### 2.3. D5 — retry_multiplier (driver biến động mạnh nhất)

| Trạng thái | Retry mult |
|---|---|
| Workflow chưa tối ưu | 2–5× |
| Có guardrail + caching | 1.1–1.3× |
| Có circuit breaker + idempotency | 1.05–1.1× |

→ Đây là driver **bị bỏ qua nhiều nhất** nhưng impact ±30–60% (xem §4).

---

## §3. Three-Scenario Framework

| Tham số | Conservative | **Base** | Aggressive |
|---|---|---|---|
| Adoption rate (`active_users` growth) | +5%/tháng | +15%/tháng | +35%/tháng |
| `invocations_per_user` growth | +0% | +10%/quý | +25%/quý |
| `avg_tokens_per_invocation` | stable | +5%/quý (prompt drift) | +15%/quý (longer chains) |
| `avg_audio_seconds_per_inv` (nếu audio) | stable | +5%/quý | +15%/quý |
| `$_per_unit` (text) | -10%/năm (vendor cut) | -20%/năm | -30%/năm |
| `$_per_unit` (TTS/image) | stable | -5%/năm | -10%/năm |
| `retry_multiplier` | 1.3 → 1.2 (improve) | 1.8 stable | 2.5 → 3.0 (drift) |
| Model mix shift tới higher tier | 0% | +10%/năm | +30%/năm |

> **Multimodal note:** Vendor pricing **text giảm nhanh hơn image/audio**. Khi mix shift về multimodal use case, blended `$_per_unit` không giảm tuyến tính như "30% vendor cut" mà phụ thuộc weight của từng modality.

### 3.1. Khi nào dùng scenario nào

| Mục đích | Scenario chính | Use |
|---|---|---|
| Budget approval năm sau | Base + buffer 20% | Board |
| Worst-case planning | Aggressive | CFO stress test |
| Investment / ROI baseline | Conservative | Show ROI vẫn dương |
| Capacity planning (GPU provisioned) | Aggressive | Tránh stockout |

*Nguồn: Drivetrain; Ridgeway FS.*

---

## §4. Sensitivity Analysis

Bài tập ±20% mỗi driver, hold others constant, agentic workflow điển hình:

| Rank | Driver | Impact (±20%) | Action ưu tiên |
|---|---|---|---|
| 1 | `retry_multiplier` | ±30–60% | Guardrail + reflection-depth limit + caching ([[04_monitoring]] §2.2) |
| 2 | `avg_units_per_invocation` | ±20% | Prompt compression; context discipline; modality optimization (TTS chunking, image downsampling) |
| 3 | `invocations_per_user` | ±20% | Rate limiting; batch where possible |
| 4 | `active_users` | ±20% | Phased rollout |
| 5 | Model mix (frontier %) | ±15–25% | Model-tier policy ([[04_monitoring]] §2.2) |
| 6 | `$_per_unit` | ±20% | Multi-provider; provisioned throughput cho high-volume |

> **Hàm ý:** Engineering attention vào **driver 1–2 trước** (retry + units). Đây là 2 driver platform team kiểm soát trực tiếp. Driver 3–4 do business, 5–6 do procurement/strategy.

---

## §5. Operating Cadence

| Cadence | Hoạt động | Owner |
|---|---|---|
| **Hàng tuần** | Refresh D1, D2, D3, D5 từ system tables; flag deviation > 15% vs forecast | Platform |
| **Hàng tháng** | Re-baseline forecast; cập nhật model mix + vendor pricing ([[01_rate_card]]); rolling 12-month | FinOps + Platform |
| **Hàng quý** | Scenario reassessment; CFO stress test; capacity planning quý tới | FinOps Council |
| **Hàng năm** | Strategic budget; multi-year forecast; vendor commitment decisions (provisioned throughput) | Board / Exec |

> Vendor pricing update phải xong **trong 14 ngày** sau khi vendor thông báo — không để stale price làm sai forecast.

---

## §6. KPI cho forecast process

| KPI | Target sau 12 tháng |
|---|---|
| **Forecast accuracy** (MAPE month-ahead) | ≤ 15% [cần verify baseline] |
| **Forecast accuracy** (MAPE quarter-ahead) | ≤ 25% |
| **% use case có driver-level forecast** | 100% prod use case |
| **% budget breach phát hiện trước** | ≥ 90% |
| **Refresh cadence compliance** | ≥ 95% on-time |

---

## §7. Cảnh báo phương pháp luận

1. **Không extrapolate khi data < 90 ngày** — variance quá lớn, dùng analog (use case tương tự) thay vì regression.
2. **Forecast chỉ tốt khi tagging tốt** — pre-condition là tagging discipline ([[_appendix/allocation]] §3).
3. **Update vendor pricing trong 14 ngày** — xem §5.
4. **Tracking forecast accuracy là KPI Platform team**, không phải nice-to-have.
5. **Multimodal cần track per-modality unit count riêng** — đừng pre-blend thành "token equivalent" làm mất thông tin recompute.

---

## Liên kết

- Đơn giá feed vào `$_per_unit`: [[01_rate_card]]
- Công thức cost cho `non_token_overhead`: [[02_cost_formula]] §3
- Anomaly alert khi actual vs forecast > 25%: [[04_monitoring]] §3 (P3)
- Tagging để có `active_users` per use case: [[_appendix/allocation]] §3
- Roadmap Phase 2 calibrate forecast model: [[_appendix/roadmap]]

## Nguồn

- FinOps Foundation. *Effect of Optimization on AI Forecasting.*
- FinOps Foundation. *Cost Estimation of AI Workloads.*
- Drivetrain. *Unit economics for AI SaaS companies.*
- Ridgeway Financial Services. *GPU Cost Forecasting, AI Unit Economics, and Infrastructure Strategy.*
- Acropolium. *AI agent unit economics: TCO, ROI, payback.*
- Agents Arcade. *Cost Modeling for Agentic Systems Before You Go to Production.*
- Galileo. *The Hidden Costs of Agentic AI.*
- Keito. *Why AI Agent Costs Are Unpredictable.*
