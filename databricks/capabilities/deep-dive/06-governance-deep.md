# 06 — Governance & AI Gateway Deep-Dive

> **Mục tiêu thảo luận:** Hiểu control plane mà Databricks dựng quanh LLM/agent traffic, đối chiếu với yêu cầu ngân hàng (PCI-DSS, ISO 27001, SBV, GDPR cho khách hàng EU, OFAC). Đây là layer có giá trị **defensive** cao nhất với CISO/risk officer.

---

## 6.1 Unity Catalog — 3 trục cho AI assets

### Assets được manage (docs)
- **Models** (registered ML model in UC).
- **Functions** (UC functions làm tool / data transformation).
- **Connections** (HTTP connections để gọi external API + MCP server).
- **Vector indexes** (xuất hiện như securable object).

### Cơ chế chung
- Access control: privileges + attribute-based policies.
- **Automatic lineage** — track data flow.
- **Audit logging** — mọi activity được log.
- **Data classification** — tag sensitive info.

### Lineage cho AI — pattern thực tế
```
[Source table] (PII tagged)
    │
    │ Delta Sync
    ▼
[Vector Search index] ──► (inherits PII tag? — open question)
    │
    │ Retrieval
    ▼
[Agent run] ──► [Response]
    │
    ▼
[Inference Table]
```

> 🏦 **Banking insight:**
> Lineage end-to-end là yêu cầu của SBV thanh tra "show me data journey". UC native lineage giảm effort manual.
>
> Tuy nhiên cần verify:
> - Tag PII có propagate qua Vector Search index không.
> - Tag có truyền vào response và inference table không.
>
> Nếu không tự động → policy thủ công: bất kỳ source PII tag → vector index PHẢI tag PII.

> ❓ **Open question:** PII tag propagation qua VS index — verify implementation.

---

## 6.2 AI Gateway — control plane internals

### 6.2.1 Endpoints quản trị
Theo docs index `ai-gateway/`:
- **Configure endpoints** — central endpoint config.
- **Query endpoints** — OpenAI-compatible query interface.
- **Rate limits** — RPM/TPM enforcement.
- **Inference tables** — request/response log.
- **Usage tracking** — `system.ai_gateway.usage` Delta table.
- **Cost observability** — cost dashboards.
- **Traffic splitting** — weighted route giữa destinations.
- **Coding agent integration** — special-purpose support.
- **MCPs** — central MCP catalog.

### 6.2.2 Rate limit — chi tiết

3 cấp (xác nhận docs):
1. **Endpoint-level** — global cap.
2. **User-level (default)** — baseline cho tất cả user.
3. **Custom** — per user / SP / group.

Metric: **QPM** (Queries per Minute) hoặc **TPM** (Tokens per Minute).

Enforcement hierarchy:
- Individual override group.
- Group override default.
- User trong nhiều group → cap = OR (chỉ bị limit nếu vượt MỌI group).

Response khi vượt: HTTP **429**.

Limits: max **20 rate limits** / endpoint, max **5 group-specific** / endpoint.

> 🏦 **Banking insight (rate limit strategy):**
>
> | Layer | RPM | TPM | Lý do |
> |---|---|---|---|
> | Endpoint global | 6,000 | 5M | Bảo vệ infra |
> | Default user | 30 | 30K | Default safe |
> | Group "RM-corporate" | 120 | 200K | RM cần burst nhiều query |
> | Group "customer-mobile" | 12 | 10K | Customer dùng vừa phải, bảo vệ cost |
> | SP "batch-ai-query" | 1,200 | 5M | Batch job high TPM, low RPM |
>
> Đảm bảo 1 user/SP rogue không drain quota làm sập agent cho cả ngân hàng.

> ⚠️ **Tradeoff:** "Concurrent requests aren't validated beforehand — system records usage after a response is sent". Tức là agent có thể **burst** ngắn vượt rate, sau đó throttle. Phù hợp với latency, nhưng banking cần biết để tính capacity peak hour.

### 6.2.3 Traffic splitting — chi tiết

Capabilities (docs):
- Max **5 destinations** / endpoint.
- Random routing theo percentage.
- Tổng phải = 100%.
- Fallbacks độc lập với split (sau khi pick primary, retry theo fallback list).
- Observability qua `system.ai_gateway.usage.routing_information`.

Limitations:
- Không split trên fallback.
- Không "shadow traffic" mode trong docs (gọi parallel nhiều destination, trả 1).

> 🏦 **Banking insight (canary):**
> Pattern canary cho ngân hàng:
> ```
> Day 1-3:  Primary v_N (95%), v_(N+1) (5%)   ← monitor metrics
> Day 4-5:  v_N (80%), v_(N+1) (20%)
> Day 6-7:  v_N (50%), v_(N+1) (50%)
> Day 8-9:  v_N (20%), v_(N+1) (80%)
> Day 10+:  v_(N+1) (100%), v_N standby
> ```
> Khi metric scorer xấu ở v_(N+1) → snap về 100% v_N trong giây (rollback).

> ❓ **Open question Q10:** Shadow traffic support? Quan trọng cho A/B model mới mà không gây impact production.

### 6.2.4 Usage tracking — `system.ai_gateway.usage`

Docs ghi: ~25 data points / request. Bao gồm:
- Request/response details (timestamps, latency, token counts).
- User (requester ID, IP, user-agent).
- Endpoint metadata (name, tags, destination model).
- Performance (latency percentiles, error rates, HTTP status).
- Cost attribution data.

### Tagging cho cost attribution

2 cách:
- **Request tag** — HTTP header per-request.
- **Endpoint tag** — config-time, apply tất cả.

> 🏦 **Banking insight (showback):**
> Cost showback cho BU:
> - Endpoint tag = `{"bu": "retail", "product": "card-chatbot"}`.
> - Request tag = `{"channel": "mobile", "customer_segment": "priority"}`.
> - Join `system.ai_gateway.usage` với cost rate table → bill BU theo % consumption.
> - Dashboard finance: chi phí AI / BU / sản phẩm / kênh.

---

## 6.3 Guardrails — khoảng mờ trong docs

Docs **đề cập** guardrails ở overview AI Gateway nhưng **không có sub-page chi tiết** trong index đã crawl (`ai-gateway/`). Theo nội dung tổng:
- "Guardrails configuration" được nhắc tới.
- PII detection, content safety **không** liệt kê chi tiết enforcement point.

> ❓ **Open question Q9:** Guardrails chi tiết — input/output filter, latency overhead, PII redaction strategy. Cần đào sub-page riêng (chưa fetch được trong crawl).

### Solution direction (out-of-docs)

Pattern guardrail "đủ dùng" cho banking trong khi chờ verify:

```
[User input]
    │
    ▼
┌─────────────────────────────────────┐
│ Layer 1 — Input guardrail            │
│  - Jailbreak detect (deterministic) │
│  - PII mask in prompt               │
│  - Topic check (allow-list)         │
└─────────────────────────────────────┘
    │
    ▼
[Agent execution]
    │
    ▼
┌─────────────────────────────────────┐
│ Layer 2 — Output guardrail           │
│  - PII leak check                   │
│  - No-advice check (investment)     │
│  - Brand voice                      │
│  - Citation required                │
└─────────────────────────────────────┘
    │
    ▼
[Response to user]
```

Implementation:
- Layer 1: code-based + small LLM judge synchronous (block trước khi gọi expensive LLM chính).
- Layer 2: code-based + custom LLM judge synchronous (block trước khi trả user).
- Async: production monitoring scorer cho post-hoc audit.

> 🏦 **Banking insight:**
> CISO ngân hàng sẽ KHÔNG chấp nhận "post-hoc detect" cho PII leak. Cần synchronous output guardrail. Nếu AI Gateway built-in chưa cover → tự build layer custom trong agent code (qua tool / middleware) trước khi gọi `return`.

---

## 6.4 Inference Tables — audit + security joint use

### Câu hỏi compliance ngân hàng yêu cầu Inference Tables giải đáp

| Câu hỏi audit | SQL query |
|---|---|
| Show me all requests by user X in last 7 days | `WHERE requester = 'X' AND event_time > ...` |
| Show all PII leak alerts | join với scorer feedback `WHERE feedback.judge = 'pii_check' AND rating = 'no'` |
| Show all responses about "loan rate" | `WHERE response LIKE '%lãi suất%'` |
| Slow request investigation | `ORDER BY latency_ms DESC LIMIT 100` |
| Cost per RM in Sep | `JOIN usage ON ... GROUP BY requester` |

> ⚠️ **CRITICAL banking risks:**
> - Inference Tables **chứa raw request/response** → tự thân là PII repository.
> - PHẢI áp UC ACL khắt khe: chỉ security/compliance được query.
> - Retention policy align với data retention chung của ngân hàng (5-7 năm).
> - Backup separately cho audit immutability.

### Setup permissions
- External storage catalog (KHÔNG default).
- Endpoint creator: "Can Manage".
- Riêng schema/table: ACL chặt cho security team.

---

## 6.5 Multi-tenant patterns

### Ngân hàng có 3 tầng tenancy thường gặp

1. **Internal tenants** (BU): retail, corporate, treasury, risk, compliance, HR.
2. **Customer segments**: mass, affluent, priority, corporate, SME.
3. **External** (nếu có open banking): partner fintech, white-label.

### UC structure đề xuất

```
catalog: bank_main
├── schema: retail_kb            (PII tagged, ACL: retail BU)
│   ├── vs_index_retail_policy
│   └── fn_lookup_retail_customer
├── schema: corporate_kb         (ACL: corporate BU)
│   ├── vs_index_corporate_deals
│   └── fn_lookup_corporate
├── schema: shared_kb            (cross BU readable)
│   ├── vs_index_circulars       (SBV / regulator)
│   └── fn_calculator_fx
└── schema: ai_registry          (model registry, scoped to AI center)
    ├── model.agent_retail_chatbot
    ├── model.agent_corporate_rm
    └── ...

inference_tables:
catalog: bank_audit
└── schema: ai_inference
    ├── inference_retail_chatbot   (ACL: security + compliance)
    └── inference_corporate_rm
```

> 🏦 **Banking insight:**
> Tách "main catalog" (working data) và "audit catalog" (immutable log) là pattern hay. Audit catalog chỉ append-only, không update/delete. Backup riêng.

---

## 6.6 Chinese-wall / Information Barrier

Trong investment bank, **Chinese wall** giữa research analyst và trading desk là bắt buộc.

### Implementation đề xuất qua UC + Agent

- **UC tag:** `wall = 'research'` vs `wall = 'trading'`.
- **Row filter:** UC row filter check user attribute.
- **Agent design:** 2 agent riêng, 2 endpoint riêng. KHÔNG share supervisor giữa 2 wall.
- **AI Gateway:** rate limit + group khác nhau, không cross.
- **Inference Tables:** schema riêng, ACL riêng.

> 🏦 **Banking insight:**
> Đừng cố "1 agent serve all" — chinese wall yêu cầu hạ tầng vật lý tách bạch về mặt logic. Pattern: namespace catalog + endpoint + SP per wall.

---

## 6.7 Data residency & cross-border

Issues từ docs:
- LLM judge: EU workspace → EU model, others → US model.
- Synthetic eval: "doesn't retain prompts for abuse monitoring in EU regions" → có thể retain ở US.

> 🏦 **Banking insight VN context:**
> - Khách hàng EU dùng dịch vụ ngân hàng VN: GDPR áp dụng cho data subject EU.
> - Cần workspace EU riêng cho data EU customer? Hoặc dùng external model routing tới EU LLM endpoint?
> - SBV chưa có yêu cầu data residency mạnh như EU, nhưng có "lưu trữ trên lãnh thổ VN" cho 1 số dữ liệu thanh toán (NĐ 53/2022).
> - Databricks region availability cho VN: cần check.

> ❓ **Open question:** Databricks có VN/SEA region cho workspace + Vector Search không?

---

## 6.8 LLMOps stack — Prompt Registry & MLOps Stacks

### Prompt Registry
- MLflow versioning + experiment tracking cho prompts.
- Prompt = first-class artifact, không phải "config dán DB".

> 🏦 **Banking insight:**
> Prompt nhạy cảm cho ngân hàng (chứa policy intent, brand voice, risk guidance). Quy trình:
> - Prompt PR trong git → review bởi domain expert (compliance, brand, legal).
> - Register Prompt Registry → version.
> - Agent load prompt by version → reproducible.
> - Rollback bằng cách trỏ về version cũ.

### MLOps Stacks
- IaC declarative cho pipeline, workflow, agent.
- Tái lập môi trường dev/uat/prod.

> 🏦 **Banking insight:**
> Kết hợp với DABs → reproducible deployment cho agent. SOX/SBV audit yêu cầu "deployment process documented & repeatable" → MLOps Stack đáp ứng natural.

---

## 6.9 3 GenAI challenges — Databricks framing

Docs khẳng định 3 challenges chính:

| Challenge | Databricks giải |
|---|---|
| Governance (leak, compliance, cost) | UC + AI Gateway + Security & Governance Frameworks |
| Quality (unpredictable) | MLflow Evaluation + Tracing + human feedback |
| Control (multi-provider, customization) | Foundation Model APIs + Model Serving + Data Intelligence |

> 🏦 **Banking insight:**
> Khi pitch CIO/CISO, dùng framing này làm "platform value proposition":
> 1. **Governance** — Tất cả AI traffic đi qua 1 control plane (AI Gateway) + 1 governance layer (UC). Không phải bolt-on 5 tool.
> 2. **Quality** — Quality không phải "tốt-xấu" cảm tính mà có scorer + judge + dataset version → measurable.
> 3. **Control** — Đổi model provider không sửa code; data ở UC không lock-in.

---

## 6.10 Hot debate points

1. **PII propagation:** Tự động hay manual policy?
2. **Guardrail synchronous:** Build trong agent code hay đợi AI Gateway sub-feature?
3. **Audit catalog tách biệt:** Đầu tư dual catalog từ ngày 1?
4. **Region strategy:** Multi-region cho EU customer? Đắt vs compliance benefit?
5. **Prompt as code:** Mỗi prompt change phải qua git PR — over-engineer cho POC, hợp lý cho prod. Threshold ở đâu?
6. **Cost showback:** Đầu tư tagging discipline ngay từ POC để khỏi rebuild?
