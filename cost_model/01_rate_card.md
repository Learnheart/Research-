# 01. Rate Card — Bảng giá per-modality × per-provider

> **Snapshot:** 2026-05-21 · **Refresh cadence:** Tháng (hoặc trong vòng 14 ngày sau khi vendor thông báo đổi giá — xem [[03_forecasting]] §5)
> **⚠️ Disclaimer:** Giá list-price công khai, **chưa áp enterprise discount / committed-use / Bedrock-Azure markup**. Finance/Procurement phải verify trước khi đưa vào chargeback. Mỗi dòng có cột **Verify-by** trỏ tới nguồn chính thức.

---

## ⚡ 30-sec card

```
cost_invocation = Σ_modality  units[m] × price[m]  × (1 + retry_mult)
                + cached_tokens × price_cached     # L1 only
                + non_token_overhead               # L2..L6 — xem [[02_cost_formula]]

modality ∈ { text_in, text_out, image_in, image_out, audio_in_sec,
             audio_out_char, video_sec, embedding, finetune_token }
```

| Bạn cần biết | Đi tới |
|---|---|
| Text token (LLM thuần) | §1 |
| Image input / output | §2 |
| Audio (STT / TTS) | §3 |
| Video / Embedding / Finetune | §4 |
| Prompt caching mechanics | §5 |
| Ví dụ task multimodal | §6 |
| Cách verify + refresh | §7 |

---

## §1. Text — input / output tokens

Đơn vị: **USD per 1M tokens**. Cache-read & cache-write tách riêng (xem §5).

| Provider | Model | Input | Output | Cache-read | Notes | Verify-by |
|---|---|---:|---:|---:|---|---|
| Anthropic | Claude Opus 4.x | 15.00 | 75.00 | 1.50 | Frontier reasoning | anthropic.com/pricing |
| Anthropic | Claude Sonnet 4.5/4.6 | 3.00 | 15.00 | 0.30 | Default tier | anthropic.com/pricing |
| Anthropic | Claude Haiku 4.5 | 1.00 | 5.00 | 0.10 | Small/cheap tier | anthropic.com/pricing |
| OpenAI | GPT-5 | 1.25 | 10.00 | 0.13 | Frontier | openai.com/pricing |
| OpenAI | GPT-5-mini | 0.25 | 2.00 | 0.025 | Mid tier | openai.com/pricing |
| OpenAI | GPT-4o | 2.50 | 10.00 | 1.25 | Multimodal native | openai.com/pricing |
| OpenAI | GPT-4o-mini | 0.15 | 0.60 | 0.075 | Cheap multimodal | openai.com/pricing |
| Google | Gemini 2.5 Pro | 1.25 / 2.50 | 10.00 / 15.00 | 0.31 | Hai bậc giá ≤200K / >200K ctx | ai.google.dev/pricing |
| Google | Gemini 2.5 Flash | 0.30 | 2.50 | 0.075 | Default tier | ai.google.dev/pricing |
| Google | Gemini 2.5 Flash-Lite | 0.10 | 0.40 | 0.025 | Cheap tier | ai.google.dev/pricing |
| AWS Bedrock | Nova Pro | 0.80 | 3.20 | 0.20 | AWS-native | aws.amazon.com/bedrock/pricing |
| AWS Bedrock | Nova Lite | 0.06 | 0.24 | 0.015 | AWS-native cheap | aws.amazon.com/bedrock/pricing |
| AWS Bedrock | Claude/* via Bedrock | = Anthropic | = Anthropic | = Anthropic | Bedrock markup ~0% on Claude | aws.amazon.com/bedrock/pricing |
| Databricks | Pay-per-token Llama-3.3-70B | ~1.00 | ~3.00 | n/a | Trả bằng DBU; tỷ giá DBU theo workspace tier | docs.databricks.com/aws/en/machine-learning/foundation-model-apis/ |
| Databricks | Pay-per-token Claude / GPT (via AI Gateway) | = provider | = provider | = provider | Pass-through; markup tính ở DBU serving | docs.databricks.com/aws/en/ai-gateway/ |
| Databricks | Provisioned throughput (own model) | — | — | — | Tính theo `$/DBU-hour × hours` thay vì tokens | docs.databricks.com/aws/en/machine-learning/model-serving/ |

> **Bedrock / Azure markup:** Bedrock pass-through ~0% với Claude, **Azure OpenAI ~= OpenAI list** nhưng có committed-use discount riêng. Verify mỗi quarter.

---

## §2. Image — input (vision) & output (generation)

### 2.1. Image INPUT (vision understanding)

Mỗi vendor đo khác đơn vị — không tự so sánh "$/image" cross-vendor:

| Provider | Model | Đơn vị tính | Quy đổi gần đúng | Verify-by |
|---|---|---|---|---|
| Anthropic | Claude (any 4.x) | Token-converted: `tokens ≈ (width × height) / 750` | 1MP image ≈ 1334 tokens → ≈ $0.004 (Sonnet) | docs.anthropic.com/.../vision |
| OpenAI | GPT-4o / GPT-5 vision | Tile-based: 85 base + 170 × tiles (low=1 tile, high tùy size) | 1MP "high detail" ≈ 765 input tokens → ≈ $0.002 (GPT-4o) | platform.openai.com/docs/guides/vision |
| Google | Gemini 2.5 | Token-converted: 258 tokens/image (≤384×384) hoặc theo tile | ≈ $0.0003 (Flash) per image | ai.google.dev/.../vision |
| AWS Bedrock | Nova Pro vision | Image counted as input tokens; ~1280 tokens/image | ≈ $0.001 per image | aws.amazon.com/bedrock/pricing |
| OpenAI | gpt-image-1 (input cho edit) | $10 per 1M image tokens | — | openai.com/pricing |

### 2.2. Image OUTPUT (generation)

Đơn vị: **USD per image** (không phải tokens).

| Provider | Model | Resolution / Quality | $/image | Verify-by |
|---|---|---|---:|---|
| OpenAI | gpt-image-1 | low / 1024² | 0.011 | openai.com/pricing |
| OpenAI | gpt-image-1 | medium / 1024² | 0.042 | openai.com/pricing |
| OpenAI | gpt-image-1 | high / 1024² | 0.167 | openai.com/pricing |
| OpenAI | DALL·E 3 | standard 1024² | 0.040 | openai.com/pricing |
| OpenAI | DALL·E 3 | HD 1024² | 0.080 | openai.com/pricing |
| Google | Imagen 4 Standard | 1024² | 0.040 | ai.google.dev/pricing |
| Google | Imagen 4 Ultra | 1024² | 0.060 | ai.google.dev/pricing |
| AWS Bedrock | Nova Canvas | 1024² standard | 0.040 | aws.amazon.com/bedrock/pricing |
| Stability AI | SD3.5 Large (via Bedrock) | 1024² | 0.065 | aws.amazon.com/bedrock/pricing |

---

## §3. Audio — input (STT) & output (TTS)

### 3.1. Audio INPUT — Speech-to-Text

| Provider | Model | Đơn vị | Giá | Verify-by |
|---|---|---|---:|---|
| OpenAI | Whisper-1 | $/minute audio | 0.006 | openai.com/pricing |
| OpenAI | gpt-4o-transcribe | $/minute audio | 0.006 | openai.com/pricing |
| OpenAI | GPT-4o realtime audio (input) | $/1M audio tokens | 40.00 | openai.com/pricing |
| Google | Gemini audio input | $/1M audio tokens | 1.00 (Flash) / 3.00 (Pro) | ai.google.dev/pricing |
| Deepgram | Nova-3 | $/minute audio | 0.0043 | deepgram.com/pricing |
| AssemblyAI | Universal-2 | $/hour audio | 0.27 (≈$0.0045/min) | assemblyai.com/pricing |
| AWS | Transcribe (standard) | $/minute audio | 0.024 | aws.amazon.com/transcribe/pricing |

### 3.2. Audio OUTPUT — Text-to-Speech

Đơn vị tính khác nhau giữa các vendor — đây là cost trap phổ biến nhất:

| Provider | Model | Đơn vị | Giá | Notes | Verify-by |
|---|---|---|---:|---|---|
| OpenAI | tts-1 | $/1M characters | 15.00 | Standard quality | openai.com/pricing |
| OpenAI | tts-1-hd | $/1M characters | 30.00 | HD quality | openai.com/pricing |
| OpenAI | GPT-4o realtime audio (output) | $/1M audio tokens | 80.00 | Streaming voice | openai.com/pricing |
| ElevenLabs | Multilingual v2 | $/1K characters | ~0.30 (subscription-blended) | Per-tier; pay-as-you-go cao hơn | elevenlabs.io/pricing |
| Google | Gemini 2.5 native audio (output) | $/1M audio tokens | 20.00 | — | ai.google.dev/pricing |
| AWS | Polly Neural | $/1M characters | 16.00 | — | aws.amazon.com/polly/pricing |
| AWS | Polly Generative | $/1M characters | 30.00 | — | aws.amazon.com/polly/pricing |
| Azure | TTS Neural | $/1M characters | 16.00 | — | azure.microsoft.com/.../speech-services |

> **⚠️ Cẩn thận đơn vị:** ElevenLabs niêm yết per-1K characters (×1000 để so với OpenAI), một số vendor thì per-1M characters. Khi build allocation engine, normalize tất cả về **$/1M characters**.

---

## §4. Video / Embedding / Finetune

### 4.1. Video

| Provider | Model | Đơn vị | Giá | Verify-by |
|---|---|---|---:|---|
| Google | Veo 3 | $/second generated | 0.50 | ai.google.dev/pricing |
| Google | Veo 2 | $/second generated | 0.35 | ai.google.dev/pricing |
| OpenAI | Sora | $/second generated | 0.50–1.00 (theo resolution) | openai.com/pricing |
| Google | Gemini video INPUT | $/second video | ~0.001 (Flash) / 0.007 (Pro) | ai.google.dev/pricing |
| Twelve Labs | Marengo (video understanding) | $/minute indexed | varies | twelvelabs.io/pricing |

### 4.2. Embeddings

| Provider | Model | Đơn vị | Giá | Verify-by |
|---|---|---|---:|---|
| OpenAI | text-embedding-3-large | $/1M tokens | 0.13 | openai.com/pricing |
| OpenAI | text-embedding-3-small | $/1M tokens | 0.02 | openai.com/pricing |
| Google | gemini-embedding-001 | $/1M tokens | 0.15 | ai.google.dev/pricing |
| Anthropic | (no native; dùng Voyage AI) | $/1M tokens | 0.05–0.12 | voyageai.com/pricing |
| Cohere | embed-v4 | $/1M tokens | 0.12 | cohere.com/pricing |
| AWS | Titan Embeddings v2 | $/1M tokens | 0.02 | aws.amazon.com/bedrock/pricing |
| Databricks | Vector Search storage | $/GB-month | varies (DBU-based) | docs.databricks.com/aws/en/generative-ai/vector-search.html |

### 4.3. Finetune

| Provider | Model | Train | Inference markup | Verify-by |
|---|---|---|---|---|
| OpenAI | GPT-4o finetune | $25 / 1M training tokens | 2× base inference | openai.com/pricing |
| OpenAI | GPT-4o-mini finetune | $3 / 1M training tokens | 2× base | openai.com/pricing |
| Anthropic | Claude (on Bedrock) | $/hour custom training | varies | aws.amazon.com/bedrock/pricing |
| Google | Gemini Flash tuning | Free training, pay inference | inference = base × tier | ai.google.dev/pricing |
| Databricks | Foundation Model Fine-tuning | $/M training tokens (DBU) | depends on model | docs.databricks.com/aws/en/large-language-models/foundation-model-training/ |

---

## §5. Prompt caching mechanics

Caching giảm cost lớn (5–10×) **nếu** repeat patterns đủ dài. Cơ chế khác nhau giữa vendor:

| Provider | Cơ chế | Write cost | Read cost | TTL | Min size để cache |
|---|---|---|---|---|---|
| Anthropic | Explicit `cache_control` block | +25% base input | 10% base input | 5 phút (default) / 1 giờ (extended) | 1024 tokens (Sonnet/Opus), 2048 (Haiku) |
| OpenAI | Auto (prefix-match) | 0 (cùng giá base) | 50% base input (GPT-4o), 10% (GPT-5) | ~5–10 phút | 1024 tokens |
| Google | Explicit context cache | Storage fee $1/1M tokens/hour | 25% base input | User-controlled | 4096 tokens |
| AWS Bedrock | Pass-through Anthropic / Nova | = provider | = provider | = provider | = provider |

> **Hàm ý:** OpenAI auto-cache "miễn phí write" nhưng savings thấp hơn (50% vs 90%). Anthropic explicit cache cần engineering, nhưng savings cao hơn cho long system prompt + RAG context.

---

## §6. Ví dụ — Task multimodal: image-in / audio-out

**Use case:** Agent nhận ảnh sản phẩm (1 ảnh 1MP) → phân tích → trả về mô tả bằng giọng nói TTS (300 từ ≈ 1500 ký tự).

### 6.1. Stack option A — OpenAI all-in-one

| Bước | Đơn vị tính | Lượng | Đơn giá | Cost |
|---|---|---|---|---|
| Vision input (GPT-4o) | tokens | 765 | $2.50/M | $0.0019 |
| Text output (mô tả) | tokens | 200 | $10.00/M | $0.0020 |
| TTS (tts-1-hd) | characters | 1500 | $30.00/M | $0.0450 |
| **Total** | | | | **~$0.049** |

### 6.2. Stack option B — Multi-vendor optimal cost

| Bước | Provider / Model | Lượng | Đơn giá | Cost |
|---|---|---|---|---|
| Vision input | Gemini 2.5 Flash | 258 tokens | $0.30/M | $0.00008 |
| Text output | Gemini 2.5 Flash | 200 tokens | $2.50/M | $0.0005 |
| TTS | ElevenLabs (subscription blended) | 1500 chars | $0.30/1K | $0.45 |
| **Total** | | | | **~$0.45** |

> **Insight:** TTS thường là **bước đắt nhất** trong chuỗi multimodal — dominate cost (≥90%). Khi optimize, target TTS trước (tier-selection / caching audio fragments / Polly Neural thay vì Generative).
>
> **Insight 2:** Naive comparison "$/image" hoặc "$/character" cross-vendor có thể sai. ElevenLabs có giọng tốt hơn → có thể justify cost; nhưng nếu use case không cần voice quality cao, Polly Neural ($16/1M) rẻ hơn ElevenLabs (~$300/1M PAYG) gần 20×.

### 6.3. Công thức tổng quát

```python
cost_image_in_audio_out = (
    image_in_tokens   × price_per_token[model]
  + text_out_tokens   × price_per_output_token[model]
  + tts_characters    × price_per_char[tts_model]
) × (1 + retry_multiplier)
+ L2..L6_overhead   # xem [[02_cost_formula]] §3
```

---

## §7. Cách verify + refresh

| Khi | Hành động | Owner |
|---|---|---|
| Hàng tháng | Scrape pricing pages của top-5 provider; diff với rate card; commit changes với date-stamp | FinOps + Platform |
| Vendor đổi giá | Update trong vòng 14 ngày (xem [[03_forecasting]] §5) | FinOps |
| Trước chargeback go-live | Cross-check với invoice 2 tháng gần nhất; flag delta > 5% | Finance |
| Hàng quý | Recompute blended `$_per_token` theo model mix thực tế cho forecast | FinOps |

### Field cần track khi update

```yaml
- provider: <Anthropic/OpenAI/Google/AWS/Azure/Databricks/...>
  model: <model-id>
  modality: <text_in|text_out|image_in|image_out|audio_in_sec|audio_out_char|video_sec|embedding|finetune>
  unit: <usd_per_M_tokens|usd_per_image|usd_per_minute|usd_per_M_characters|usd_per_second>
  price: <number>
  effective_date: YYYY-MM-DD
  source_url: <link>
  notes: <free text>
```

> **Đề xuất Phase 2 ([[_appendix/roadmap]]):** Move bảng này từ markdown sang `rate_card.yaml` + render markdown để CI/CD diff phát hiện stale.

---

## Liên kết

- Cách rate card chèn vào công thức tổng cost: [[02_cost_formula]] §3
- Cách driver `$_per_token` được blended theo model mix trong forecasting: [[03_forecasting]] §2
- Alert khi vendor đổi giá: [[04_monitoring]] §3 (P4)
- So sánh vendor & decision build/buy: [[_appendix/tooling]]

## Nguồn

- Anthropic. *Pricing.* anthropic.com/pricing
- OpenAI. *API Pricing.* openai.com/api/pricing
- Google. *Gemini API Pricing.* ai.google.dev/pricing
- AWS. *Amazon Bedrock Pricing.* aws.amazon.com/bedrock/pricing
- Azure. *Azure OpenAI Service Pricing.* azure.microsoft.com/.../pricing/details/cognitive-services/openai-service
- Databricks. *Model Serving Pricing.* databricks.com/product/pricing/foundation-model-serving
- ElevenLabs. *Pricing.* elevenlabs.io/pricing
- Deepgram. *Pricing.* deepgram.com/pricing
- AssemblyAI. *Pricing.* assemblyai.com/pricing
