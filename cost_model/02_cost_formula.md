# 02. Cost Formula — Taxonomy 6 lớp + công thức multimodal-ready

> **Insight cốt lõi:** Token cost chỉ là **30–50%** tổng cost vận hành một agent (Galileo, Acropolium 2025). Mọi cost model dừng ở "tokens × giá" sẽ bỏ sót 50–70% chi phí thực.

---

## ⚡ 30-sec card

```
┌──────────────────────────────────────────────────────────────┐
│  L6. HIDDEN     (oversight, remediation, opportunity)        │
│  L5. OPS        (logging, security, FinOps tools, headcount) │
│  L4. DATA       (vector DB, embeddings, KB, egress)          │
│  L3. ORCH       (agent framework, tool calls, retries)       │
│  L2. COMPUTE    (GPU, CPU serving, network, storage)         │
│  L1. MODEL      (multi-modality units × rate card)           │
└──────────────────────────────────────────────────────────────┘
```

**Công thức tổng (multimodal-ready):**

```
cost_invocation =
  Σ_modality ( units[m] × price[m] × (1 + retry_mult) )      # L1
+ Σ_cached  ( cached_tokens × price_cached )                  # L1
+ (gpu_hours × price_gpu) / invocations_per_hour              # L2 amortized
+ Σ tool_calls × avg_tool_cost                                # L3
+ Σ rag_queries × avg_rag_cost + embedding_units × price_emb  # L4
+ ops_overhead_per_invocation                                 # L5 amortized
+ P(error) × cost_per_remediation                             # L6 probabilistic

modality m ∈ { text_in, text_out, image_in, image_out,
               audio_in_sec, audio_out_char, video_sec,
               embedding, finetune_token }

retry_mult: 2–5× (unguarded) → 1.1–1.3× (caching + guardrail)
```

**Cần biết:**

| Câu hỏi | Đi tới |
|---|---|
| 6 lớp gồm gì + driver | §1 |
| Đơn giá cụ thể | [[01_rate_card]] |
| Công thức chi tiết | §3 |
| Cost trap phổ biến | §4 |
| Field instrument bắt buộc | §5 |
| Source-of-truth dữ liệu | §6 |

---

## §1. Bảng phân tầng 6 lớp (MECE)

| # | Lớp | Cost drivers chính | Unit | Nguồn billing | Visibility |
|---|---|---|---|---|---|
| **L1** | Model | input/output tokens (mọi modality), cached tokens, finetune tokens, function-call overhead | $/1M tokens, $/image, $/min audio, $/char, $/sec video | `system.serving.endpoint_usage`; provider invoices | **Cao** |
| **L2** | Compute | GPU-hours (provisioned), CPU serving, autoscale overhead, egress, model artifact storage | $/GPU-hour, $/DBU, $/GB egress | `system.billing.usage`; cloud CUR | **Cao** |
| **L3** | Orchestration | Tool calls, retries (silent + explicit), reflection loops, planner/critic, multi-agent coord | calls/req, retries/req, planner_tokens/req | OTel traces; `system.serving.payload`; Langfuse | **Thấp–Trung** |
| **L4** | Data | Vector DB storage + read amp, embedding gen, doc ingestion, RAG query, egress sang model | $/GB-month, $/embed token, $/query | Vector Search billing; provider logs | **Trung** |
| **L5** | Ops | Logging/observability tools, security scanning, FinOps platform license, platform team FTE | $/month, FTE allocated | Procurement + HR allocation | **Trung** |
| **L6** | Hidden | Error remediation (HITL), false-positive review, retry-due-to-bad-output, user wait opportunity cost, audit overhead | $/error, hours/incident | Workforce + ticketing join | **Rất thấp** |

> **Note multimodal:** L1 không chỉ là tokens. Một invocation image-in/audio-out kích hoạt ≥ 3 đơn vị khác nhau (token + char + (đôi khi) image_count). Rate card ở [[01_rate_card]] tách rõ.

---

## §2. Multimodal — quy đổi đơn vị

Khi 1 invocation chạy qua **nhiều modality**, không thể gộp về 1 unit duy nhất. Cách track:

```
invocation_L1_cost = (
    text_in_tokens     × price[text_in, model_A]
  + text_out_tokens    × price[text_out, model_A]
  + image_in_count     × price[image_in, model_A]      # hoặc image_in_tokens
  + image_out_count    × price[image_out, model_B]
  + audio_in_minutes   × price[audio_in_sec, model_C]
  + audio_out_chars    × price[audio_out_char, model_D]
  + video_in_seconds   × price[video_sec, model_E]
  + embedding_tokens   × price[embedding, model_F]
)
```

| Cân nhắc khi instrument | Lý do |
|---|---|
| Lưu **raw count theo từng modality** (không pre-convert sang $) | Giá đổi liên tục; preserve count để recompute lịch sử |
| Lưu **`model_id` cùng count** | Cùng 1 modality, model khác giá khác (Sonnet vs Haiku) |
| Lưu `provider_call_id` (vendor-side reference) | Reconcile với invoice cuối tháng |
| Token-converted modality (image-on-Anthropic) | Lưu **cả raw image count VÀ converted_tokens** để biết quy đổi |

---

## §3. Công thức cost — chi tiết theo tầng

### 3.1. L1 — Model

```
L1 = Σ_m units[m] × price[m] × (1 + retry_mult)  +  cached_tokens × price_cached
```

- `retry_mult` lấy từ OTel: `total_inference_calls / user_facing_requests`
- `price[m]` tra từ [[01_rate_card]] §1–§4
- Cached pricing: §5 của rate card

### 3.2. L2 — Compute (amortized)

```
L2_per_inv = (gpu_hours × $_gpu + cpu_serving_hours × $_cpu + egress_gb × $_egress) / N_invocations
```

- Áp dụng cho provisioned throughput; pay-per-token endpoint **đã include L2 trong L1**
- Amortize theo invocations trong cùng window (rolling 1-hour bucket)

### 3.3. L3 — Orchestration

```
L3 = Σ tool_calls × avg_tool_cost  +  Σ planner_critic_calls × avg_planner_token_cost
```

- `avg_tool_cost`: bao gồm cost của external API (search, code-exec, etc.) — track riêng per tool
- Retry đã bao gồm trong `retry_mult` ở L1 — đừng đếm 2 lần

### 3.4. L4 — Data

```
L4 = (vector_storage_gb × $_storage/12) + (vector_queries × $_per_query) +
     (embedding_tokens × $_per_embed_token) + (egress_to_model_gb × $_egress)
```

### 3.5. L5 — Ops (allocated)

```
L5_per_inv = (monthly_tooling_$ + monthly_FTE_allocated_$) / monthly_invocations
```

- Recompute monthly; bucket theo `use_case_id`
- Bao gồm: Langfuse self-host compute, FinOps tooling, security scan, platform FTE %

### 3.6. L6 — Hidden (probabilistic)

```
L6 = P(error) × cost_per_remediation
   + P(bad_output_caught_by_HITL) × HITL_review_minutes × $_per_minute
   + opportunity_cost (estimate hàng quý)
```

- `P(error)` lấy từ outcome_status field
- Update estimate hàng quý từ workforce data

### 3.7. Tổng

```
cost_invocation = L1 + L2_amort + L3 + L4 + L5_amort + L6
```

→ Aggregate lên `cost_per_use_case`, `cost_per_BU`, `cost_per_outcome` xem [[_appendix/allocation]] §4.

---

## §4. Cost traps phổ biến (có nguồn)

| Trap | Cơ chế | Impact | Nguồn | Detection |
|---|---|---|---|---|
| **Silent retries** | Network/rate-limit/tool fail → auto-retry, full inference run mỗi lần | Token ×2–5 không tăng traffic | Agents Arcade 2025 | Alert `retry_rate > 25%` ([[04_monitoring]] §3) |
| **Reflection loop unbounded** | Self-critique không max-depth → token explode ở corner case | ×10–100 cho request lỗi | Galileo | Policy guardrail: max reflection = 3 |
| **RAG amplification** | 10-step chain × query/embedding mỗi step → embed ×10, vector read ×10 | Có thể vượt LLM cost | Subhojyoti Singha, Medium 2025 | Track `rag_queries / invocation`; baseline + alert |
| **Over-provisioned GPU** | Provisioned throughput đặt theo peak, không autoscale | 30–50% wasted | Cogent 2025 | Utilization < 60% trong 14 ngày → flag |
| **Over-logging payload** | Lưu full req/resp mọi call → storage tăng + PII risk | Storage ×2–10 | Vantage | Sample rate policy ở L5 |
| **Wrong-tier model** | Frontier model cho task đơn giản | 5–50× differential | Clarifai | Model-tier policy ở Gateway ([[04_monitoring]] §2.2) |
| **TTS dominance ở multimodal** | Chuỗi image-in→text→audio-out, audio-out chiếm ≥90% cost | Optimize sai chỗ | (xem [[01_rate_card]] §6) | Track cost-per-modality breakdown |
| **Forgotten finetune** | Orphan finetune lab vẫn billing | Range rộng | FinOps Foundation 2025 | Quarterly audit |

---

## §5. Instrumentation tối thiểu (cho mọi invocation)

| Attribute | Bắt buộc | Mục đích | Lớp |
|---|:---:|---|:---:|
| `trace_id` | ✅ | Join cross-system | All |
| `agent_id` / `agent_version` | ✅ | Attribution | All |
| `use_case_id` | ✅ | Allocation tới BU | All |
| `user_id` / `caller_principal` | ✅ | Chargeback per user | All |
| `cost_center_tag` | ✅ | Finance attribution | All |
| `model_id` / `provider` | ✅ | Per-model cost | L1 |
| `tokens_in` / `tokens_out` / `cached_tokens` | ✅ | Text cost | L1 |
| `image_in_count` / `image_in_tokens_converted` | ✅ (nếu vision) | Image cost | L1 |
| `image_out_count` / `quality_tier` | ✅ (nếu gen) | Image-gen cost | L1 |
| `audio_in_seconds` / `audio_out_chars` | ✅ (nếu audio) | STT/TTS cost | L1 |
| `video_in_seconds` / `video_out_seconds` | ✅ (nếu video) | Video cost | L1 |
| `embedding_tokens` / `embedding_model` | ✅ (nếu RAG) | Embedding cost | L1/L4 |
| `tool_name` / `tool_calls_count` | ✅ | Orchestration | L3 |
| `retry_count` / `retry_reason` | ✅ | Cost trap detection | L3 |
| `rag_queries_count` / `vectors_read` | ⚪ recommend | Data cost | L4 |
| `outcome_status` (success/partial/fail) | ⚪ recommend | Unit econ + L6 | All |

> Multimodal-aware fields (image/audio/video) là **mới so với v1 taxonomy**. Mọi agent từ Phase 2 phải emit nếu có dùng modality tương ứng.

---

## §6. Source-of-truth dữ liệu (Databricks-first)

| Lớp | Source chính | Source phụ |
|---|---|---|
| L1 Model | `system.serving.endpoint_usage` | Provider invoices nếu đi direct (OpenAI/Anthropic) |
| L2 Compute | `system.billing.usage` (DBU) | Cloud CUR (AWS/Azure) |
| L3 Orchestration | OTel traces → Langfuse → join L1 theo `trace_id` | Custom span attributes |
| L4 Data | `system.billing.usage` Vector Search; Pinecone/Weaviate invoices | Embedding API logs |
| L5 Ops | Procurement; HR cost center allocation | Tooling invoices export |
| L6 Hidden | Workforce + ticketing join theo `trace_id` | Manual quarterly update |

---

## Liên kết

- Đơn giá cụ thể: [[01_rate_card]]
- Driver-based forecast dùng công thức này: [[03_forecasting]] §2
- Anomaly detection per cost trap: [[04_monitoring]] §3
- Allocation engine (Direct/Shared/Overhead): [[_appendix/allocation]]
- Tooling support instrumentation: [[_appendix/tooling]]

## Nguồn

- Acropolium. *AI agent unit economics: TCO, ROI, payback.*
- FinOps Foundation. *Cost Estimation of AI Workloads.*
- Galileo. *The Hidden Costs of Agentic AI.*
- Agents Arcade. *Cost Modeling for Agentic Systems Before You Go to Production.*
- Gravitee. *How to Control the Hidden Costs of Generative AI.*
- Vantage. *AI Cost Considerations Every Engineer Should Know.*
- Cogent. *AI-Native FinOps: Controlling GPU and LLM Cloud Costs.*
- Subhojyoti Singha. *The Hidden Cost of Embeddings.* Medium, 2025.
- Clarifai. *AI Cost Controls: Budgets, Throttling & Model Tiering.*
- Databricks Docs. *Monitor usage for Unity AI Gateway endpoints.*
- Finout. *Top 6 AI Cost Drivers and GenAI Cost Examples in 2026.*
