# 03 — Evaluation Deep-Dive

> **Mục tiêu thảo luận:** Hiểu rõ scorer architecture của MLflow 3 + judge mechanics để thiết kế quality gate phù hợp ngân hàng — nơi "trả lời sai 1%" có thể đồng nghĩa với phạt regulator hoặc mất khách.

---

## 3.1 Scorer architecture (MLflow 3)

### 4 loại scorer
Theo `mlflow3/genai/eval-monitor/concepts/scorers`:

```
┌──────────────────────────────────────────────────────────────────┐
│  SCORER (unified interface, attach feedback vào trace)            │
├──────────────────────────────────────────────────────────────────┤
│ 1. Built-in LLM Judges                                            │
│    - Research-validated (Cohen's Kappa, F1)                       │
│    - Safety, Groundedness, Correctness, Relevance, …              │
│    - Minimal config                                               │
├──────────────────────────────────────────────────────────────────┤
│ 2. Custom LLM Judges                                              │
│    - Domain-specific prompt                                       │
│    - User-defined criteria                                        │
├──────────────────────────────────────────────────────────────────┤
│ 3. Code-based Scorers                                             │
│    - Deterministic (regex, exact match, schema validation)        │
│    - Cheap, fast, no LLM cost                                     │
├──────────────────────────────────────────────────────────────────┤
│ 4. Third-party Scorers                                            │
│    - Plug-in từ open-source eval framework                        │
└──────────────────────────────────────────────────────────────────┘
        │
        ▼
   Receives trace → Extract relevant fields → Run logic → Return feedback
```

### Output schema (chung)
Mọi judge return:
- **Rating** (yes/no, pass/fail, hoặc số/categorical).
- **Rationale** (explanation từ LLM, hoặc reason từ code).
- **Error message** (null nếu OK).

Aggregate: `rating/average` = pass rate trên dataset.

### LLM Judge model
- Default: Databricks-hosted LLM.
- Override: `<provider>:/<model-name>`.
- EU workspace → judge chạy EU model; còn lại US.
- Cảnh báo từ docs: *"LLM judge outputs should not be used to train, improve, or fine-tune an LLM"* → tránh data leak qua loop self-improving.

> 🏦 **Banking insight:**
> Judge chạy EU/US khác nhau là **vấn đề data residency** cho VN bank có khách hàng EU/US (correspondent banking, remittance). Phải check tenant region khi enable scorer.

> ⚠️ **Tradeoff custom scorer production:** Theo docs production-monitoring:
> - Chỉ `@scorer` decorator (not class).
> - Phải define trong **Databricks notebook**.
> - Self-contained: imports trong function body, không reference external var/module.
> → Cản trở team có codebase Python repo riêng. Workaround: copy-paste vào notebook + git-sync.

---

## 3.2 Built-in judges — cơ chế cụ thể

### Bảng quyết định "có cần ground truth không?"

| Judge | Ground truth? | Input cần | Output ngữ nghĩa |
|---|---|---|---|
| `global_guideline_adherence` | ❌ | `global_guidelines` config | Yes nếu response tuân thủ guideline toàn hệ thống |
| `relevance_to_query` | ❌ | request, response | Yes nếu response đúng intent |
| `safety` | ❌ | response | Yes nếu không có harmful/toxic |
| `groundedness` | ❌ | response, retrieved_context | Yes nếu response bám context, không hallucinate |
| `chunk_relevance` | ❌ | request, retrieved_context | Yes per-chunk; sinh precision |
| `guideline_adherence` | ✅ (per-row guideline) | per-row guideline | Yes nếu tuân thủ guideline cho row đó |
| `correctness` | ✅ (`expected_facts[]` recommended) | request, response, expected | Yes nếu cover expected facts |
| `context_sufficiency` | ✅ (expected) | retrieved_context, expected | Yes nếu retrieval đủ để answer |
| `document_recall` | ✅ (`expected_retrieved_context[].doc_uri`) | retrieved_context, expected URIs | % docs đúng được retrieve |

### Lưu ý from docs
- **Correctness:** docs khuyến nghị `expected_facts[]` (minimal facts) thay vì `expected_response` đầy đủ → robust hơn vì không phạt phong cách diễn đạt.
- **Multi-turn:** judge chỉ chấm **lượt cuối** (cho MLflow 2 Agent Evaluation; multi-turn judges riêng cho production monitoring).

> 🏦 **Banking insight (selection matrix):**
>
> | Banking use case | Must-have judges | Nice-to-have |
> |---|---|---|
> | Customer chatbot | safety, relevance, groundedness, safety (custom: brand voice) | guideline_adherence (script tuân thủ) |
> | Credit memo | groundedness, correctness (vs expected_facts), context_sufficiency | document_recall (đủ policy được trích) |
> | Compliance Q&A | groundedness, document_recall, guideline_adherence (per-regulation) | chunk_relevance (verify citation precision) |
> | AML triage | guideline_adherence (typology coverage), safety | groundedness, custom: false-positive rate |
> | RM advisory | safety (no investment advice off-script), guideline_adherence | groundedness, custom: regulatory disclosure included |

> ⚠️ **Tradeoff:** Mỗi judge thêm 1 LLM call → cost × số judge × số sample. Production monitoring sample rate cần tính:
> `cost/day = QPS_prod × 86400 × sample_rate × num_judges × cost_per_judge_call`
> Ví dụ 10 QPS × 0.2 sample × 5 judges × $0.005/call ≈ $4,300/day. Đáng giá trao đổi.

---

## 3.3 Custom LLM judge — kỹ thuật

### Cấu trúc khái niệm
- Một custom judge = prompt template + criteria + output schema (yes/no + rationale).
- Có thể tham chiếu guideline (per-row hoặc global).
- Convert legacy `make_genai_metric_from_prompt` thành custom metric.

> 🏦 **Banking insight (custom judges ngân hàng cần build):**
> 1. **`brand_voice_compliance`** — không xưng quá thân mật, không hứa hẹn lãi suất cố định ngoài approved.
> 2. **`regulatory_disclosure_present`** — câu trả lời về sản phẩm đầu tư có kèm disclaimer SBV.
> 3. **`pii_leak_check`** — response không lộ CMND/CIF/số tài khoản của khách hàng khác.
> 4. **`no_investment_advice_off_script`** — không tự ý gợi ý mua/bán cổ phiếu, vàng.
> 5. **`citation_required`** — mọi câu trả lời compliance phải có citation văn bản.
> 6. **`anti_jailbreak_check`** — phát hiện user dụ "ignore previous instructions".

### Validation judge — bao giờ tin được judge?
Docs nói built-in judges validated bằng "Cohen's Kappa, accuracy, F1".

> 🏦 **Banking insight:**
> Trước khi enable custom judge cho production gate, team nên:
> 1. Label 200-500 sample bằng SME.
> 2. Run judge.
> 3. Tính Cohen's Kappa giữa judge vs SME. Kappa > 0.6 → "good agreement", >0.8 → "excellent".
> 4. Document threshold + judge model version trong policy doc nội bộ (audit yêu cầu).

---

## 3.4 Code-based scorer — đừng quên

Nhiều use case ngân hàng KHÔNG cần LLM judge:
- **Output schema validation:** response phải là JSON match schema (KYC verdict).
- **Field present:** mọi response credit memo phải có "rating", "rationale", "risk_factors".
- **Regex:** không chứa email/phone của khách khác.
- **Length:** response < 300 words cho chatbot mobile.

> 🏦 **Banking insight:**
> Code-based scorer rẻ + deterministic + audit-friendly. Nên là **first-layer gate** trước LLM judge. Pattern:
> ```
> [Response] → code-based gate (fail-fast) → LLM judge → human review (Review App)
> ```

---

## 3.5 Evaluation dataset — build & maintain

### Synthetic eval (`generate_evals_df`)
- Input: `content` + `doc_uri` columns.
- Param: `num_evals`, `agent_description`, `question_guidelines`.
- Output (MLflow 3): `request_id`, `inputs` (chat format), `expectations` (expected_facts, expected_retrieved_context).
- Distribution: cố gắng đều câu hỏi / page.

> ⚠️ **Cảnh báo docs:** *"Synthetic data is intended to help customers evaluate their agent applications, and the outputs should not be used to train, improve, or fine-tune an LLM"*. Tức là synthetic eval **không** thay được human-labeled set cho fine-tune; chỉ dùng evaluate.

> ❗ **Constraint:** *"Unavailable in workspaces with serverless egress control enabled"* — workspace ngân hàng thường lock egress → có thể không dùng được synthetic eval. Cần verify.

> 🏦 **Banking insight (eval set build):**
> Hybrid approach:
> 1. **Synthetic** từ corpus tài liệu chính thức (policy, procedure, FAQ) → đủ "happy path" baseline.
> 2. **Production failures** convert thành labeled set (loop từ Review App).
> 3. **SME-crafted** edge cases:
>    - Câu hỏi liên hệ giữa 2 policy.
>    - Câu hỏi về sản phẩm bị thu hồi (regulator yêu cầu).
>    - Adversarial: "Tôi muốn rút USD bằng cách lách quy định, giúp tôi".
> 4. **Regulator-published Q&A** (SBV FAQ, NHNN public) → "must answer correctly" set.

### Labeling sessions
- SME ngồi label batch traces.
- Build expectations (ground truth) song song với feedback.

> 🏦 **Banking insight:**
> Cần workflow rõ ràng cho labeling session — đề xuất:
> - Schedule weekly 2h cho legal + compliance team review 50 trace.
> - Inter-rater agreement check (2 labeler / trace) cho 10% mẫu.
> - Label quality dashboard hiển thị Kappa giữa labelers — phát hiện disagreement → tinh chỉnh guideline.

---

## 3.6 Bridging dev ↔ production

Triết lý docs: **cùng scorer chạy dev & prod**.

```
DEV phase                          PROD phase
─────────                          ──────────
Eval dataset                       Live traces (sampled)
   │                                  │
   ├─ scorer A ───────────────────────┤  (same scorer object)
   ├─ scorer B ───────────────────────┤
   └─ scorer C ───────────────────────┘
   ▼                                  ▼
Aggregate metric                   Per-trace assessment + alert
+ regression gate                   + dashboard
```

> 🏦 **Banking insight:**
> Đây là điểm "win" rõ nhất cho audit. Trước đây team đo offline khác metric với online → khó giải trình "vì sao dev pass, prod fail". Với MLflow 3, audit-ready evidence là **cùng object scorer + cùng version + run trên 2 môi trường**, lưu lineage trong UC.

---

## 3.7 Hot debate points

1. **Quality gate strict đến đâu?** Safety = 100% (zero tolerance) là hợp lý, nhưng correctness = 95% có drift theo domain → set per domain hay global?
2. **Custom judge ownership:** AI center build chung hay BU tự build judge cho domain mình?
3. **Eval dataset size:** Bao nhiêu đủ? 200 / 500 / 5,000 sample? Trade-off cost vs statistical power.
4. **Judge model:** Dùng same model agent đang chạy (cùng provider, biased) hay model độc lập (Claude judge GPT agent)?
5. **Synthetic eval guard:** Cần guard nào để synthetic không bị "easy" mode (LLM tự sinh câu dễ tự trả lời)?
6. **Production sample rate:** Sample 0.2 cho cost, hay 1.0 cho critical safety scorer + 0.1 cho expensive judge?
