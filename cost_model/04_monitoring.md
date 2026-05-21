# 04. Monitoring & Governance — Pre-spend Controls, không phải Reactive Reporting

> **Insight cốt lõi:** Monitoring kiểu báo cáo cuối tháng đã là **quá muộn** — một retry storm hoặc reflection loop có thể đốt budget tháng trong vài giờ. Cần **3 tầng**: pre-spend → real-time → post-spend, gắn với FinOps Council bi-weekly.

---

## ⚡ 30-sec card

```
┌─────────────────────────────────────────────────────────────┐
│  TẦNG 1 — PRE-SPEND (Preventive)                            │
│  Budget hard-limit  |  Policy guardrail  |  Rate limit      │
│  → Chặn trước khi xảy ra @ AI Gateway                       │
├─────────────────────────────────────────────────────────────┤
│  TẦNG 2 — REAL-TIME (Detective)                             │
│  Streaming metrics → 3σ alert (5–15 phút)                   │
│  → Bắt anomaly khi đang xảy ra                              │
├─────────────────────────────────────────────────────────────┤
│  TẦNG 3 — POST-SPEND (Reactive)                             │
│  Daily/weekly report + chargeback ledger                    │
│  → Forensic + accountability + RCA                          │
└─────────────────────────────────────────────────────────────┘
```

**Alert P1/P2 phải bắt được:** budget burn > 200% pacing • cost spike > 10σ • retry rate > 50% • cost-per-outcome anomaly per use case.

**Cần biết:**

| Câu hỏi | Đi tới |
|---|---|
| Budget tier nào, ngưỡng nào | §2.1 |
| Policy guardrails ở Gateway | §2.2 |
| Metric & threshold real-time | §3.1–§3.2 |
| Approach anomaly detection | §3.3 |
| FinOps Council operating model | §4 |
| Phase artifact (charter, runbook, ...) | §5 |

---

## §1. Mô hình 3 tầng kiểm soát

| Tầng | Mục đích | Cơ chế | Vendor support |
|---|---|---|---|
| **1. Pre-spend** | Chặn over-spend trước khi xảy ra | Budget hard-limit, rate-limit per user/use-case, model-tier policy, prompt size limit | Databricks Unity AI Gateway service policies; AWS Bedrock Guardrails; Azure quota |
| **2. Real-time** | Phát hiện anomaly trong 5–15 phút | Streaming metrics → threshold alert; ML anomaly trên cost-per-inv, retry rate, token-per-call | Databricks Lakehouse Monitoring; AWS Cost Anomaly Detection; Langfuse/LangSmith alerts |
| **3. Post-spend** | Forensic, RCA, accountability | Daily/weekly cost report; chargeback ledger; FinOps Council review | Databricks dashboards; Apptio; Finout |

---

## §2. Pre-spend (Tầng 1)

### 2.1. Budget enforcement

| Loại | Scope | Hành vi khi vượt |
|---|---|---|
| Per-user | Dev cá nhân (coding agent) | 80% email alert · 100% throttle · 120% block + manager approval |
| Per-use-case | UC-RM-001, UC-BA-002 … | 80% owner alert · 100% throttle non-prod · 120% block + BU exec approval |
| Per-BU | Retail / Wholesale / Risk / … | 90% BU lead alert · 100% freeze new use case onboarding |
| Platform-wide | Cả Agent Platform | 95% exec alert · 100% all-hands cost review |

*Reference: Databricks "AI spend controls with Unity AI Gateway" (2025); AWS Bedrock proactive cost management.*

### 2.2. Policy guardrails (chặn ngay tại Gateway)

| Policy | Mục tiêu | Ví dụ rule |
|---|---|---|
| Model-tier | Tránh frontier cho task đơn giản | Dev env → cấm `gpt-5-frontier`; phải dùng mid-tier để escalate |
| Prompt size | Tránh prompt explosion | Max 32K context cho prod; 128K cần justification |
| Reflection depth | Tránh runaway | Max 3 levels; max 5 self-critique calls |
| Rate limit | Tránh runaway loop / abuse | 1000 req/min prod; 100 req/min dev |
| PII | Compliance + cost | Reject request có PII chưa redact |
| External provider routing | Chặn data egress | `data_classification = restricted` → chỉ on-prem hoặc Bedrock private |
| **Modality budget** (NEW) | TTS/video cost dominance | Cap audio_out_chars/inv = 5000; video_out_sec/inv = 30 |

> **Modality budget** là policy mới so với v1 — vì task multimodal như image-in/audio-out, **TTS dominate ≥ 90% cost** (xem [[01_rate_card]] §6). Cap audio_out_chars là pre-spend control hiệu quả nhất cho audio use case.

---

## §3. Real-time (Tầng 2)

### 3.1. Metric stack tối thiểu

| Metric | Computation | Trigger threshold |
|---|---|---|
| `cost_per_minute_by_use_case` | Σ cost rolling 5-min | > 3σ vs baseline 30 ngày |
| `retry_rate_by_agent` | retries / inv rolling 15-min | > 25% (baseline ~10%) |
| `avg_units_per_invocation[modality]` | rolling 1-hour, per modality | > 1.5× baseline tuần trước |
| `error_rate` | failed / total rolling 15-min | > 5% |
| `budget_burn_rate` | (spend / period_elapsed) / (budget / period_total) | > 1.2 |
| `model_mix_drift` | % req frontier model | > 1.5× baseline |
| `cache_hit_rate` | hits / total (semantic cache) | < 0.5× baseline |
| `audio_out_chars_per_inv` (NEW) | rolling 1-hour, audio use case | > 1.5× baseline → flag TTS bloat |
| `image_in_count_per_inv` (NEW) | rolling 1-hour, vision use case | > 1.5× baseline |

### 3.2. Alert hierarchy

| Severity | Trigger | Recipient | SLA |
|---|---|---|---|
| **P1 — Critical** | Budget burn > 200% pacing HOẶC cost spike > 10σ | On-call + BU lead + FinOps + Exec | 15 phút |
| **P2 — High** | Anomaly cost/use-case > 5σ; retry rate > 50% | Use case owner + Platform on-call | 1 giờ |
| **P3 — Medium** | Cost drift +20% WoW; tag accuracy < 95%; forecast variance > 25% | Owner + FinOps | 1 ngày |
| **P4 — Info** | Forecast variance > 15%; vendor pricing change ([[01_rate_card]] §7) | FinOps mailbox | 1 tuần |

### 3.3. Anomaly detection approach

| Approach | Pros | Cons | Phù hợp |
|---|---|---|---|
| **Static threshold** (3σ cost/min) | Đơn giản, deploy nhanh | Sai khi seasonality cao | Phase 1 POC |
| **Rolling baseline** (compare 4 tuần cùng kỳ) | Bắt seasonality | Cần ≥ 60 ngày data | Phase 2 |
| **ML-based** (Prophet, Lakehouse Monitoring) | Bắt trend + seasonality | Phải maintain model; false-positive cao đầu | Phase 3 |

*Reference: AWS Cost Anomaly Detection; Databricks Lakehouse Monitoring; Capital One Slingshot.*

---

## §4. FinOps Council — Operating Model

### 4.1. Charter

> **Mission:** Đảm bảo Agent Platform spend được phân bổ, dự báo, kiểm soát theo policy đã thông qua, và mọi anomaly được xử lý trong SLA.

### 4.2. Membership

| Role | Đại diện | Tần suất |
|---|---|---|
| Chair | Head of AI Platform | Bi-weekly |
| FinOps Lead | FinOps Practitioner | Bi-weekly |
| Finance | FP&A AI cost analyst | Bi-weekly |
| Engineering | Platform engineering lead | Bi-weekly |
| BU reps | 1 mỗi major BU | Monthly rotate |
| Security / Compliance | InfoSec rep | Monthly hoặc ad-hoc |

### 4.3. Agenda (bi-weekly, 60 phút)

| Phần | Time | Output |
|---|---|---|
| Cost report period trước (variance vs forecast) | 10' | Re-baseline forecast? |
| Anomaly review (P1/P2) | 15' | RCA + action item + owner |
| Top spenders / Top growers | 10' | Watch list period tới |
| Policy / Tagging compliance | 10' | Action item enforce |
| Use case onboarding pipeline | 10' | Approve / hold dựa budget headroom |
| Vendor / Procurement updates | 5' | Decisions log |

---

## §5. Governance Artifacts (kế hoạch sản xuất)

| Artifact | Phase | Owner |
|---|---|---|
| Budget policy document (per-tier limits) | R&D | FinOps |
| Tagging enforcement policy (tech impl) | R&D | Platform Eng |
| Alert runbook (P1/P2 response steps) | POC | Platform Eng + FinOps |
| Anomaly model spec | POC | Data Science |
| FinOps Council charter (signed) | R&D | Head of AI Platform |
| Quarterly cost governance scorecard | Production | FinOps |
| **Modality budget policy** (NEW — TTS/video cap) | R&D | Platform Eng |

---

## Liên kết

- Đo gì (taxonomy + công thức): [[02_cost_formula]]
- Đơn giá để compute threshold: [[01_rate_card]]
- Forecast làm baseline cho anomaly: [[03_forecasting]]
- Tagging điều kiện tiên quyết budget per-tag: [[_appendix/allocation]]
- Tooling stack support monitoring: [[_appendix/tooling]]
- Phase rollout governance: [[_appendix/roadmap]]

## Nguồn

- Databricks Blog. *Introducing AI spend controls with Unity AI Gateway.*
- Databricks Blog. *What's new in Unity AI Gateway: service policies, guardrails, observability, and cost controls.*
- Databricks Blog. *Expanding agent governance with Unity AI Gateway.*
- AWS Blog. *Build a proactive AI cost management system for Amazon Bedrock — Part 2.*
- Clarifai. *AI Cost Controls: Budgets, Throttling & Model Tiering.*
- Capital One Software Blog. *Building a GenAI cost supervisor agent in Databricks.*
- Galileo. *A Guide to AI Agent Cost Optimization With Observability.*
- nOps. *AI Cost Visibility: The Ultimate Guide.*
- FinOps Foundation. *State of FinOps 2025.*
