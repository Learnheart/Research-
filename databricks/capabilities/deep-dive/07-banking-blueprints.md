# 07 — Banking Solution Blueprints

> **Mục tiêu thảo luận:** 8 reference architecture cho banking products dùng Databricks capabilities. Mỗi blueprint trả lời: WHY, WHAT components, HOW data flows, RISK/MITIGATION, KPI.
>
> **Lưu ý:** Mọi blueprint dưới đây map vào capability **đã được docs xác nhận**; chỗ nào suy luận sẽ ghi rõ. Không phải SOW — là *direction* để team discuss.

---

## Mục lục blueprint

1. [Customer Service Chatbot (retail)](#bp1)
2. [Internal Knowledge Assistant (policy / HR / IT helpdesk)](#bp2)
3. [Credit Memo Drafting Assistant](#bp3)
4. [AML Alert Triage Co-pilot](#bp4)
5. [Compliance Q&A (regulatory)](#bp5)
6. [RM Advisory Co-pilot (wealth / corporate)](#bp6)
7. [Treasury Markets Research Synthesizer](#bp7)
8. [Operations Ticket Triage](#bp8)

Cuối file: **bảng map cross-blueprint × capability**.

---

<a id="bp1"></a>
## BP1 — Customer Service Chatbot (Retail)

### Pain point
Contact center cost cao, peak hour tắc, FAQ 80% nhưng vẫn tốn người. Customer want 24/7 instant answer.

### Architecture

```
[Mobile / Web App / IVR]
        │
        ▼
[API Gateway (existing) ──► [AI Gateway Endpoint]
        │                          │
        │   (input guardrail)      │ traffic split: 95% v_N, 5% v_(N+1)
        ▼                          ▼
[Agent Framework + ResponsesAgent]
   │
   ├─► Tool: VS index ── "retail_product_kb"
   │      (Delta Sync, multilingual embed VN/EN)
   ├─► Tool: UC function "lookup_account_balance"
   │      (filter by customer_id from custom_inputs)
   ├─► Tool: MCP managed "genie_retail" (NL→SQL for analytics)
   └─► Tool: MCP external "ticketing_system" (escalate to human)
        │
        ▼ output
   custom_outputs: {citations, confidence, intent_class}
```

### Capability mapping
- **Authoring:** Agent Framework (full code) + Databricks Apps (UI custom brand) — *Hybrid pattern*.
- **Retrieval:** Vector Search Delta Sync, hybrid mode, threshold 0.6, top-k 5 sau rerank.
- **Tools:** UC function (balance lookup), Managed MCP (Genie), External MCP (ticket).
- **Auth:** App auth + `custom_inputs.customer_id` filter (customer chưa có UC identity).
- **Eval:** safety, brand_voice, no_advice_off_script, citation_required, multi-turn `user_frustration`.
- **Serving:** Provisioned throughput (peak hour predictable), AI Gateway traffic split.
- **Monitor:** Inference Tables + scorer sample 1.0 safety, 0.2 groundedness.
- **Governance:** UC ACL on customer table; AI Gateway rate limit per session.

### Quality gate (release)
- Safety judge ≥ 99.9%.
- Groundedness ≥ 90%.
- No PII leak in 1000 sample.
- Multi-turn frustration < 5%.

### Risk & Mitigation
| Risk | Mitigation |
|---|---|
| PII leak qua chat (vô tình lộ số khác) | Output guardrail synchronous (pii_check scorer) |
| Promised lãi suất sai | guideline_adherence per-product + structured response template |
| Jailbreak ("ignore all instructions") | Input guardrail (deterministic keyword + small LLM check) |
| Cost overrun peak hour | Provisioned throughput + AI Gateway TPM cap |

### KPI
- Containment rate (% session không escalate người).
- CSAT post-chat.
- Cost / contained session.
- Time-to-first-byte (UX critical).

---

<a id="bp2"></a>
## BP2 — Internal Knowledge Assistant (HR / IT / Compliance KB)

### Pain point
Nhân viên không tìm được policy mới nhất; HR/IT/Legal trả lời cùng câu hỏi nhiều lần.

### Architecture

```
[Internal portal / Teams / Slack]
        │
        ▼
[AI Gateway Endpoint with OBO auth]
        │
        ▼
[Agent Bricks — Supervisor]
        │
        ├─► Knowledge Assistant "hr_policies"
        ├─► Knowledge Assistant "it_procedures"
        ├─► Knowledge Assistant "compliance_kb"
        └─► UC function "lookup_employee_ticket_history"
```

### Capability mapping
- **Authoring:** Agent Bricks (Supervisor + Knowledge Assistants) — low-code.
- **Auth:** OBO — employee chỉ thấy info được phép (HR officer thấy comp data, dev thấy IT only).
- **Eval:** correctness vs expected_facts, document_recall, citation_required.
- **Monitor:** scheduled scorer weekly.

### Constraints
- Knowledge Assistant **English-only** → cho tài liệu tiếng Việt phải dùng Agent Framework custom + VS index multilingual. Hoặc dịch policy sang EN nhưng mất nuance pháp lý.

> ⚠️ **Banking insight:** Với ngân hàng VN, BP2 nên build qua Agent Framework custom (Layer C) chứ không Knowledge Assistant nếu cần Việt ngữ. Đầu tư cao hơn nhưng đúng nhu cầu.

### KPI
- Self-service rate (giảm ticket cho HR/IT).
- Time saved / employee.
- Citation accuracy (audit từ legal team).

---

<a id="bp3"></a>
## BP3 — Credit Memo Drafting Assistant

### Pain point
Credit analyst tốn 4-8h drafting memo. Inconsistent format. Junior analyst miss critical risk factor.

### Architecture

```
[Credit workbench (custom Databricks App)]
        │
        ▼
[Agent Framework — multi-step ResponsesAgent]
        │
        ├─► Tool: VS index "credit_policy" (version controlled)
        ├─► Tool: VS index "industry_research" (Bloomberg/Reuters via External MCP)
        ├─► Tool: UC function "get_customer_financials" (3yr P&L, BS)
        ├─► Tool: UC function "get_credit_bureau_score" (CIC via MCP)
        ├─► Tool: UC function "get_collateral_valuation"
        └─► Tool: `system.ai.python_exec` (compute ratios: DSCR, LTV, leverage)
        │
        ▼ structured output
   custom_outputs: {
     memo_draft: str,
     financial_ratios: {dscr, ltv, leverage, current_ratio},
     risk_factors: [string],
     citations: [{policy_section, chunk_id, doc_uri}],
     recommendation: enum {approve, approve_with_conditions, reject},
     confidence: float,
   }
```

### Capability mapping
- **Authoring:** Agent Framework (heavy custom). Multi-tool orchestration.
- **Tools:** UC functions (financials, bureau), Vector Search (policy, research), python_exec (ratios), External MCP (CIC bureau).
- **Auth:** OBO — credit officer level access.
- **Eval:** correctness vs expected_facts (gold memos labeled by senior credit officers), context_sufficiency, custom `risk_factor_coverage` (đã list đủ ngành / leverage / market chưa).
- **Workflow:** Maker-checker — agent draft, human senior approve.
- **Inference Tables:** lưu full audit trail (input financials, policy version, output memo, signer).

### Risk & Mitigation
| Risk | Mitigation |
|---|---|
| Agent miss risk factor → bad loan | custom scorer `risk_factor_coverage` + senior approval gate |
| Policy version mismatch | Pin policy VS index version per agent version; log version in custom_outputs |
| Ratio calc error | Code-based scorer validate ratio range |
| Hallucinate fact about customer | Groundedness scorer 1.0 sample, threshold ≥ 95% |

### KPI
- Draft time reduction (target 50%+).
- Senior officer override rate (proxy quality).
- NPL correlation with agent-confidence score (model validation).

> 🏦 **Banking insight:** BP3 nhạy cảm regulatory nhất. Mỗi response phải có **policy version**, **financial snapshot date**, **agent version**, **human approver ID** — chuẩn cho audit Basel/ICAAP.

---

<a id="bp4"></a>
## BP4 — AML Alert Triage Co-pilot

### Pain point
AML team nhận hàng ngàn alert / ngày. 95-99% false positive. Analyst burn out, true positive bị miss.

### Architecture

```
[AML platform (Actimize / Fircosoft / custom) ── alert event]
        │
        ▼
[Kafka topic: aml_alerts]
        │
        ▼
[Streaming job ──► Agent endpoint per alert]
        │
        ▼
[Agent Framework — Supervisor pattern]
        │
        ├─► Sub-agent "typology_classifier"
        │     (rule + LLM classify: structuring/layering/PEP/sanction)
        ├─► Sub-agent "transaction_summarizer"
        │     (NL summary of 30/60/90 day pattern)
        ├─► Sub-agent "kyc_lookup"
        │     (customer profile, recent KYC update)
        ├─► Sub-agent "sanction_check"
        │     (External MCP → OFAC, UN, EU lists)
        └─► Sub-agent "case_history_lookup"
             (past alerts same customer)
        │
        ▼ structured triage output
   {
     triage_level: enum {auto_clear, review, escalate_str},
     typology: string,
     evidence: [{event, time, amount, citation}],
     recommended_action: [],
     confidence: float
   }
```

### Capability mapping
- **Authoring:** Agent Framework Supervisor pattern.
- **Tools:** UC tables (transactions, KYC), External MCP (sanction list), VS index (typology KB).
- **Auth:** App auth (service principal for batch); analyst luôn approve final triage.
- **Eval:**
  - false_positive_rate (so với human ground truth).
  - typology_correctness.
  - Critical: false_negative cho high-risk typology = **zero tolerance**.
- **Monitor:** Inference Tables MUST be reliable → dual-log to Kafka.
- **Governance:** Output triage là **suggestion**, không decision — STR filing vẫn cần human.

### Risk & Mitigation
| Risk | Mitigation |
|---|---|
| Miss real money laundering (false negative) | Confidence threshold cao; mọi case nghi ngờ → human |
| Bias trong typology classify | Eval set balanced across ethnicity/geo/industry; bias scorer |
| Sanction list outdated | External MCP refresh real-time, version log per call |
| Regulator audit "explainability" | Citation per evidence + reasoning trong custom_outputs |

### KPI
- Analyst time saved per alert.
- FP reduction rate.
- FN rate (caught by audit) — guard rail tuyệt đối.

> 🏦 **Banking insight:** AML là use case nhạy cảm bậc nhất; agent **never auto-close**. Vai trò là *prioritize* và *summarize*, không *decide*.

---

<a id="bp5"></a>
## BP5 — Compliance Q&A (Regulatory Knowledge)

### Pain point
Hàng nghìn circular từ SBV/MoF/SSC + nội quy nội bộ. Compliance officer + frontline cần tra cứu nhanh, chính xác đến điều khoản.

### Architecture

```
[Compliance portal / Slack bot]
        │
        ▼
[Agent Framework + custom RAG]
        │
        ├─► Tool: VS index "regulation_vn" (TT, NĐ, QĐ, structure-aware chunked)
        ├─► Tool: VS index "internal_policy" (policy theo BU)
        ├─► Tool: UC function "regulation_version_check"
        │     (lấy version effective tại date hỏi)
        └─► Tool: External MCP "law_database" (cập nhật regulation mới)
        │
        ▼
   custom_outputs: {
     answer: str,
     citations: [{document, article, clause, effective_date, doc_uri}],
     confidence: float,
     disclaimer: "Tư vấn này không thay thế ý kiến chính thức compliance officer"
   }
```

### Capability mapping
- **Authoring:** Agent Framework, structure-aware chunking (Điều, Khoản).
- **Retrieval:** Hybrid search (vector + BM25 cho số văn bản).
- **Eval:** groundedness 1.0 sample, document_recall, citation_required, custom `effective_date_check` (regulation version đúng thời điểm).
- **Auth:** OBO compliance team + frontline (filter theo BU).
- **Inference Tables:** retention 7 năm (regulatory compliance).

### Risk & Mitigation
| Risk | Mitigation |
|---|---|
| Trả regulation bị thay thế bằng version mới | UC version control + `effective_date_check` scorer |
| Trả nhầm điều khoản (Điều 8 vs Điều 18) | Structure-aware chunking + citation precise; verify on labeled set |
| Tóm tắt sai ý regulation | Groundedness scorer + disclaimer luôn kèm response |

### KPI
- % câu trả lời có citation đúng.
- Time saved cho compliance team.
- Frontline self-service rate.

---

<a id="bp6"></a>
## BP6 — RM Advisory Co-pilot (Wealth / Corporate)

### Pain point
RM phục vụ 50-200 client; cần synthesize portfolio + market + product nhanh trước meeting.

### Architecture

```
[RM workstation (web app)]
        │
        ▼
[Agent Framework — full RM copilot]
        │
        ├─► VS index "product_catalog"
        ├─► VS index "research_reports"
        ├─► UC function "get_portfolio" (filter by RM ownership)
        ├─► UC function "get_customer_360" (CRM)
        ├─► UC function "compute_portfolio_metrics"
        ├─► MCP external "market_data" (Bloomberg/Reuters)
        └─► MCP external "compliance_check" (suitability rules)
        │
        ▼
   custom_outputs: {
     client_brief: str,
     portfolio_summary: {...},
     opportunities: [{product, rationale, suitability_check}],
     risks: [...],
     citations: [...]
   }
```

### Capability mapping
- **Authoring:** Agent Framework, multi-step planning.
- **Auth:** **OBO bắt buộc** — chinese wall + RM only sees own clients.
- **Eval:** safety (no off-script advice), guideline_adherence (suitability), brand_voice.
- **Special:** Suitability check là HARD GATE — agent không suggest product mà client không pass suitability.

### Risk & Mitigation
| Risk | Mitigation |
|---|---|
| Misselling (suggest unsuitable product) | Suitability tool blocking + suitability scorer 1.0 sample |
| Insider info leak (research has MNPI?) | Research VS index không index MNPI; UC tag enforce |
| Personal opinion advice | Brand voice scorer + structured template |

### KPI
- Meeting prep time saved.
- Cross-sell conversion (suitable products only).
- Compliance flag rate (low = healthy).

---

<a id="bp7"></a>
## BP7 — Treasury Markets Research Synthesizer

### Pain point
Treasury/dealer phải đọc dozens of research / news / data daily; cần synthesis ngắn trước market open.

### Architecture

```
[Schedule trigger 5 AM daily]
        │
        ▼
[Databricks Workflow ──► Agent batch run]
        │
        ├─► VS index "research_internal"
        ├─► External MCP "bloomberg_news"
        ├─► External MCP "reuters_news"
        ├─► UC function "get_yield_curve_snapshot"
        ├─► UC function "get_fx_overnight"
        └─► UC function "get_overnight_funding_rate"
        │
        ▼
   Daily brief sent to:
   - Email dealing room
   - Internal portal (with full citations)
   - Inference Tables (audit)
```

### Capability mapping
- **Authoring:** Agent Framework (batch mode).
- **Eval:** Daily SME review (head of dealer signs off first 2 weeks); pass rate tracked.
- **Auth:** SP for batch; output gated by dealer head review.
- **Monitor:** Track factual error rate via post-day SME labeling.

### Risk & Mitigation
| Risk | Mitigation |
|---|---|
| Outdated data | Tool refresh timestamp check trong custom_outputs |
| Speculative claim | Groundedness; no_advice scorer for "buy/sell" verb detection |
| Source attribution miss | Citation_required mandatory |

### KPI
- Dealer engagement (open rate, time spent).
- Factual accuracy from SME audit weekly.
- Time saved / dealer.

> 🏦 **Banking insight:** Treasury brief có thể là use case ROI rõ ràng nhất — vài USD/day inference cost, đổi lấy 30 phút × N dealer × $/h.

---

<a id="bp8"></a>
## BP8 — Operations Ticket Triage

### Pain point
Operations / Back-office nhận ticket exception (settlement fail, payment hold, KYC missing). Manual route, SLA breach.

### Architecture

```
[Ticketing system → Kafka new_ticket]
        │
        ▼
[Agent Framework — classify + suggest action]
        │
        ├─► VS index "sop_kb" (Standard Operating Procedure)
        ├─► UC function "get_ticket_history"
        ├─► UC function "lookup_related_transactions"
        └─► UC function "check_resolution_pattern"
        │
        ▼
   {
     ticket_category: enum,
     priority: enum,
     assigned_team: string,
     suggested_resolution_steps: [...],
     similar_past_tickets: [...],
     auto_resolvable: bool
   }
```

### Capability mapping
- **Authoring:** Agent Framework + Genie sub-agent cho transaction lookup.
- **Eval:** correctness on category, ROI on time-saved.
- **Auth:** SP cho batch; ops team approve auto-resolution.

### KPI
- Auto-resolution rate (% ticket close without human).
- SLA breach reduction.
- First-time-right routing rate.

---

## Bảng map cross-blueprint × capability

| Capability | BP1 | BP2 | BP3 | BP4 | BP5 | BP6 | BP7 | BP8 |
|---|---|---|---|---|---|---|---|---|
| Agent Framework (full code) | ✅ | ⚠️ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Knowledge Assistant | — | ✅ (EN) | — | — | — | — | — | — |
| Supervisor | — | ✅ | — | ✅ | — | — | — | — |
| Vector Search Delta Sync | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Hybrid search | ✅ | ✅ | ✅ | ✅ | ✅✅ | ✅ | ✅ | ✅ |
| UC Functions | ✅ | ✅ | ✅✅ | ✅✅ | ✅ | ✅✅ | ✅ | ✅✅ |
| MCP managed | ✅ | — | — | — | — | — | — | ✅ |
| MCP external | ✅ | — | ✅ | ✅✅ | ✅ | ✅✅ | ✅✅ | — |
| `python_exec` | — | — | ✅ | — | — | ✅ | ✅ | — |
| App auth | ✅ | — | — | ✅ | — | — | ✅ | ✅ |
| OBO auth | — | ✅ | ✅✅ | — | ✅ | ✅✅ | — | — |
| Provisioned throughput | ✅ | — | — | ✅ | — | — | — | ✅ |
| Pay-per-token | — | ✅ | ✅ | — | ✅ | ✅ | ✅ | — |
| Traffic split (canary) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Inference Tables | ✅ | ✅ | ✅✅ | ✅✅ | ✅✅ | ✅✅ | ✅ | ✅ |
| Synchronous guardrail | ✅✅ | ⚠️ | ✅ | — | ✅ | ✅✅ | — | — |
| Production monitor scorer | ✅ | ✅ | ✅✅ | ✅✅ | ✅✅ | ✅ | ✅ | ✅ |
| Review App (SME label) | ✅ | ✅ | ✅✅ | ✅✅ | ✅✅ | ✅ | ✅ | ✅ |
| Multi-turn judge | ✅✅ | ✅ | — | — | — | — | — | — |
| AI Gateway rate limit per group | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Dual logging (Kafka mirror) | ✅ | — | ✅ | ✅✅ | ✅ | ✅ | — | ✅ |
| UC catalog tách audit | ✅ | — | ✅ | ✅✅ | ✅✅ | ✅ | — | — |

Legend: ✅ = dùng, ✅✅ = critical, ⚠️ = caveat (xem detail).

---

## Câu hỏi tổng cho discussion

1. **Ưu tiên build trước:** BP nào ROI nhanh nhất? (gợi ý: BP7, BP1, BP8 là quick win; BP3, BP4 là high-value high-risk).
2. **Common platform vs per-product:** Có nên build "AI center" shared (VS, MCP, monitoring) làm platform, BU tự build agent trên đó?
3. **Maker-checker integration:** Quy trình human-approve nằm trong Databricks Apps hay ticketing system hiện hữu?
4. **Vendor risk diversification:** Mỗi BP có nên multi-LLM ready (Anthropic primary, OpenAI fallback) qua AI Gateway?
5. **PII strategy:** Mỗi BP nên redact trước embed hay ACL filter retrieval-time?
6. **Eval set ownership:** Ai duy trì eval set cho BP3 (credit), BP4 (AML), BP5 (compliance)? Quy trình labeling weekly?
7. **Cost showback:** BP nào BU pay, BP nào AI center subsidize?
8. **Rollout sequencing:** BP nào pilot trên BU/branch hẹp trước? Criteria gì để mở rộng?
