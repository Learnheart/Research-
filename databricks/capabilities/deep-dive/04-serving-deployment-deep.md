# 04 — Serving & Deployment Deep-Dive

> **Mục tiêu thảo luận:** Hiểu endpoint topology, auth, infra economics đủ để thiết kế production serving cho ngân hàng — nơi SLA 99.9%, audit log immutable, cost cap nghiêm ngặt là default.

---

## 4.1 3 con đường deploy

```
┌─────────────────────────────────────────────────────────────────────┐
│ A. agents.deploy()  →  Model Serving endpoint                       │
│    - Built-in: REST API, auth, inference table, review app          │
│    - 15-min provisioning, zero-downtime updates                     │
│    - Best fit: API-only, internal copilot, B2B endpoint             │
├─────────────────────────────────────────────────────────────────────┤
│ B. Databricks Apps  →  Full stack (UI + API + business logic)       │
│    - DABs declarative deploy                                        │
│    - IDE-friendly, git-native                                       │
│    - Best fit: customer-facing app cần UI riêng, SaaS multi-tenant  │
├─────────────────────────────────────────────────────────────────────┤
│ C. Hybrid: deploy() endpoint + Apps gọi tới                         │
│    - Endpoint reusable cho mobile / web / IVR                       │
│    - Apps là 1 client trong nhiều client                            │
└─────────────────────────────────────────────────────────────────────┘
```

### Khi nào chọn cái nào?

| Yếu tố | (A) Endpoint | (B) Apps | (C) Hybrid |
|---|---|---|---|
| Cần UI ngân hàng-custom | ❌ | ✅ | ✅ |
| Reuse cho nhiều client (web/mobile/IVR/contact center) | ✅ | ⚠️ | ✅ |
| Streaming chat | ✅ | ✅ | ✅ |
| Multi-tenant per-tenant config | ⚠️ | ✅ | ✅ |
| Quick internal POC | ✅ | ✅ | — |
| Heaviest investment | thấp | trung bình | cao |

> 🏦 **Banking insight:**
> Đa số ngân hàng sẽ rơi vào (C) Hybrid: endpoint là "AI backbone" shared, Apps là từng "AI face" theo channel. Lý do: Internet Banking, Mobile, Contact Center đều cần dùng cùng agent core nhưng UX khác nhau.

---

## 4.2 `agents.deploy()` — bên dưới capot

### Provisioning tự động (xác nhận docs)
1. Tạo Model Serving endpoint scalable.
2. Provision short-lived credentials cho UC resources (VS, UC fn).
3. Verify caller permission → no privilege escalation.
4. Setup Inference Tables (audit log).
5. Enable Review App.
6. Bật automated quality evaluation trên production traffic.

### Yêu cầu version (từ docs)
- MLflow 3.1.3+ recommended (or 2.13.1+).
- `databricks-agents` SDK 1.1.0+.
- Agent registered in UC.

### Zero-downtime update
Re-deploy cùng model name → version mới song song với version cũ → cutover dần.

> 🏦 **Banking insight:**
> Pattern release ngân hàng đề xuất:
> ```
> 1. agents.deploy() → version N+1 (background)
> 2. AI Gateway traffic split: 95% v_N, 5% v_(N+1)
> 3. Monitor 30 phút: scorer pass, latency p95, error rate
> 4. Tăng dần: 20%, 50%, 80%, 100%
> 5. Khi 100% → keep v_N standby 24h cho rollback
> 6. Sau 24h → archive v_N
> ```
> Tận dụng cả `deploy()` zero-downtime + AI Gateway traffic split để **canary deployment** không cần code custom.

> ⚠️ **Tradeoff 15-min provisioning:** Không phù hợp hot-fix scenario. Phải có rollback plan dùng AI Gateway traffic split (chuyển 100% về version cũ trong giây).

---

## 4.3 Foundation Models economics

### 3 mode pricing (xác nhận docs)

| Mode | Đặc điểm | Use case |
|---|---|---|
| **Pay-per-token** | Pay khi gọi, scale-to-zero | POC, dev, low-traffic agent |
| **Provisioned throughput** | Dedicated compute, fixed cost / hour | Production có SLA, traffic dự đoán được |
| **External model routing** | Proxy tới OpenAI/Anthropic/vendor | Best-of-breed model, vendor risk diversification |

### Khi nào switch?

Quy luật ngón tay cái (suy luận, không có công thức từ docs):
- QPS < 0.1 → pay-per-token rẻ hơn.
- QPS > 1 sustained → provisioned throughput rẻ hơn, predictable.
- QPS spike-heavy (ví dụ chiến dịch marketing): pay-per-token (scale-to-zero) hoặc mix.

> 🏦 **Banking insight (cost cap):**
> - **Internal copilot RM (50 RM, 8h/ngày, ~100 query/day):** pay-per-token.
> - **Public chatbot Internet Banking:** provisioned throughput vì peak hour predictable.
> - **External model routing:** dùng cho fallback (primary Databricks-hosted, fallback Claude) hoặc benchmark.

> ⚠️ **Vendor risk:** External routing giúp **không lock-in 1 LLM provider**. Mọi prompt được route qua AI Gateway → có thể đổi vendor mà không sửa client code. Quan trọng cho compliance & business continuity ngân hàng.

---

## 4.4 Query interfaces — 3 con đường

| Interface | Khi dùng | Tradeoff |
|---|---|---|
| **Databricks OpenAI Client** | Project Python mới, native streaming | Lock-in vendor SDK |
| **OpenAI-compatible REST** | Java/Go/.NET backend, mobile | Language-agnostic, streaming qua SSE |
| **`ai_query` SQL** | Batch (10M+ docs), tích hợp với SQL warehouse | Không streaming, không tool-use phức tạp |

> 🏦 **Banking insight:**
> - **Mobile/Web app:** REST (Go/Node backend gọi qua AI Gateway). Streaming SSE cho UX.
> - **Core Java microservice:** REST.
> - **Batch processing transactions (categorize, summarize):** `ai_query` chạy trong SQL warehouse, scale-to-zero, cost low.

---

## 4.5 Inference Tables — audit gold mine

### Schema (từ `ai-gateway/inference-tables-beta`)

```
inference_table (Delta in Unity Catalog)
├── request_id            (string, PK)
├── destination_id        (string, model/endpoint version)
├── requester             (string, user/SP identity)
├── event_time            (timestamp)
├── latency_ms            (long)
├── time_to_first_byte_ms (long)         <- streaming UX metric
├── request               (string, JSON raw payload)
├── response              (string, JSON raw payload)
├── status_code           (int, HTTP)
├── sampling_fraction     (double, 0..1)
└── logging_error_codes   (array<string>)
```

### Setup requirements (docs)
- UC enabled workspace.
- External storage catalog (KHÔNG dùng default storage).
- `CREATE TABLE` + `USE` permission.
- Endpoint creator: `Can Manage` permission.

### Limitations (docs)
- Payload > 10 MiB không log.
- "Best effort delivery" — log đến trong phút, **không guarantee**.
- Private endpoint storage không support.
- Error response (401/403/429/500) **có thể không log**.

> 🏦 **Banking insight:**
> - Inference Tables = audit log built-in → nguyên liệu cho **SBV inspector** khi yêu cầu "show me 90 days of customer interactions".
> - **Retention:** Delta table → cấu hình Delta vacuum + time travel theo policy ngân hàng (thường 5-7 năm).
> - **Schema augment:** join `request_id` với app log để gắn `customer_id`, `channel`, `session_id` → audit chiều sâu hơn schema mặc định.

> ⚠️ **CRITICAL Banking:** "Best effort delivery" + "error 4xx/5xx có thể không log" KHÔNG đủ cho audit nghiêm. Mitigation:
> - **Dual logging:** app gateway tự log mỗi request vào Kafka → Delta riêng (mirror).
> - Treat inference table như **analytical store**, không phải single source of truth.
> - Document policy: nếu inference table miss, system of record là app log.

> ❓ **Open question:** Có metric đo log loss rate? Cần verify trước khi rely cho compliance.

---

## 4.6 Auth flow — 2 mode chi tiết

### App auth (Service Principal)
```
[Client] ─► [Endpoint] ─► [Agent run as SP] ─► [UC resource: SP permission]
                                          └─► Filter by custom_inputs.customer_id
```
- 1 token SP → mọi request.
- Phải filter row-level trong UC function bằng input.

### On-behalf-of-user (OBO)
```
[Client] ─► [Auth header: user token] ─► [Endpoint propagates] ─► [Agent runs as user]
                                                              └─► UC: user-level ACL
```
- Token user truyền end-to-end.
- UC enforce ACL tự động → row/column hiding.

> 🏦 **Banking insight (recommended pattern):**
>
> | Scenario | Auth mode | Lý do |
> |---|---|---|
> | Mobile customer chatbot | App auth + customer_id filter | User chưa có Databricks identity; quản qua app session |
> | Internal RM copilot | OBO | Mỗi RM chỉ thấy portfolio mình; UC ACL native |
> | Corporate banking RM | OBO + UC tag chinese-wall | Risk team không thấy deal trading; UC tag enforce |
> | Public knowledge bot | App auth, read-only SP | Không có data nhạy cảm |

### Token-passing complexity
OBO requires:
- Client (mobile app) lấy Databricks token cho user (hoặc dùng OAuth flow).
- Pass token qua header.
- Endpoint propagate xuống tool calls.

> ❓ **Open question:** Token refresh latency budget bao nhiêu? Có thể cache permission decision per session?

---

## 4.7 Logging & registering — pipeline thực tế

```
[Notebook / IDE]
   │
   │  mlflow.langchain.log_model(...) or pyfunc.log_model(...)
   ▼
[MLflow Tracking Server]
   │  resources=[VectorSearchIndex, ServingEndpoint, UCFunction]
   ▼
[MLflow Model artifact] ── logged_agent_info.model_uri
   │
   │  mlflow.register_model(model_uri, name="catalog.schema.agent_v2")
   ▼
[Unity Catalog Model Registry] ── registered, versioned
   │
   │  agents.deploy(name, version)
   ▼
[Model Serving endpoint + Inference Tables + Review App + auto-eval]
```

### Critical: khai báo `resources`
Khi log, phải declare resources agent sẽ chạm → MLflow gen credentials đúng scope khi deploy.

> 🏦 **Banking insight:**
> Đây là điểm kiểm soát quan trọng. Code review checklist:
> - [ ] `resources` chỉ chứa exactly UC objects agent cần.
> - [ ] Không có "wildcard" hay "all-access".
> - [ ] SP của endpoint không có quyền vượt resources declare.
>
> Audit team có thể grep `resources=[...]` trong git để biết agent nào chạm data nào → lineage manual nhưng minh bạch.

---

## 4.8 Hot debate points

1. **Endpoint vs Apps:** Mặc định pattern hybrid hay tách bạch theo BU?
2. **Provisioned throughput sizing:** Right-size như thế nào ban đầu? Có công cụ load test agent từ Databricks không?
3. **Inference table reliability:** Có chấp nhận "best effort" hay đầu tư dual-logging từ ngày 1?
4. **OBO complexity vs App auth simplicity:** Convention default là gì cho dự án mới?
5. **Multi-region DR:** Endpoint backup region thế nào? Có cross-region replication cho VS index + UC model registry không?
6. **Cost showback:** Endpoint tag → usage table → showback BU. Quy trình ai chịu trách nhiệm tag chuẩn?
