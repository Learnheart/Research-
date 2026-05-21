# Prompt Deep Research — Thị trường, Công nghệ, Cạnh tranh của Databricks AI Agent (EU/US · Trung Quốc · Việt Nam)

[CẬP NHẬT: 21/05/2026]

> **Tóm tắt 1 câu**: Một prompt lớn dùng để chạy deep research trên **Claude web (claude.ai)** với **web search + extended thinking**, đầu ra phục vụ workstream `02_Market_Landscape`, `03_Databricks_Technology`, `04_Competitive_Analysis` trong workspace `databricks/docs/`.

---

## 1. Mục đích sử dụng

- **Khi nào dùng**: Khi cần khởi tạo dữ liệu nền (baseline data) cho 3 workstream 02/03/04 trước khi consultant tự viết file con. Đầu ra deep research là **input thô** — phải qua bước review + bổ sung tag content + cross-check trước khi commit thành file research chính thức.
- **Khi nào KHÔNG dùng**: (1) khi cần số liệu vendor quote chính thức (cần Procurement) — xem R-02 ở `docs/WORKSPACE_TRACKING.md`; (2) khi nội dung là internal roadmap / interview SME (không lên public web); (3) khi cần phân tích sâu 1 use case Việt Nam cụ thể với industry contact (Q-06).
- **Liên quan**: prompt này là phiên bản mở rộng / có cấu trúc rõ ràng của `prompts/first_insight.md` (prompt khởi tạo).

### Citations

Không có citation — section này là metadata hướng dẫn nội bộ.

---

## 2. Tool target và setup

- **Tool**: Claude web tại <https://claude.ai>.
- **Model**: Claude 4.x (Opus hoặc Sonnet — Opus cho depth tốt hơn).
- **Setting bắt buộc bật trước khi dán prompt**:
  - **Web search** (Claude sẽ tự gọi search tool khi cần).
  - **Extended thinking** (cho phép Claude lập research plan + reasoning sâu trước khi trả lời).
- **Project context (optional nhưng khuyến nghị)**: Upload kèm 4 file vào Claude Project để Claude tham chiếu convention workspace:
  - `databricks/CLAUDE.md`
  - `databricks/docs/RULES.md`
  - `databricks/docs/02_Market_Landscape/_overview.md`
  - `databricks/docs/03_Databricks_Technology/_overview.md`
  - `databricks/docs/04_Competitive_Analysis/_overview.md`

### Citations

[1] Anthropic, "Claude Web Search and Extended Thinking", anthropic.com, n.d., <https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking> (accessed 21/05/2026).

---

## 3. Phạm vi đầu ra (mapping vào workstream)

| Section trong output deep research | Map vào file workspace |
|---|---|
| Phần A — Market Trending (A.1–A.4) | `docs/02_Market_Landscape/01_ai_agent_market_sizing.md`, `02_adoption_trends.md`, `03_regulatory_environment.md`, `04_demand_drivers_fsi.md` |
| Phần B — Databricks Technology (B.1–B.8) | `docs/03_Databricks_Technology/01_…` đến `07_…` + section so sánh tiến hoá G1/G2 |
| Phần C — Competitive (C.1–C.8) | `docs/04_Competitive_Analysis/01_…` đến `07_feature_matrix.md` |
| Phần D — Regional Deep Dive (D.1–D.3) | Đóng góp bổ trợ cho `docs/05_Banking_FSI_Applications/01–03_*_landscape.md` (regional context cho 3 workstream chính) |
| Phần E — Synthesis (E.1–E.5) | Input cho `docs/01_Executive_Summary/01_key_findings.md` (nhưng KHÔNG thay thế — chỉ feed data) |

### Citations

Không có citation — bảng mapping nội bộ.

---

## 4. Cách sử dụng (step-by-step)

1. Mở <https://claude.ai>, tạo conversation mới.
2. Bật **Extended thinking** và **Web search** trong settings của conversation.
3. (Optional) Upload 5 file workspace ở §2 vào Project context.
4. **Copy nguyên si block prompt ở §5** (giữa hai dấu ``` ```) và dán vào ô chat.
5. Trước khi Claude bắt đầu search, Claude phải trả lời với **research plan** (~30–40 search query + sources priority + potential conflicts cần cross-check). User xem và xác nhận / điều chỉnh.
6. Sau khi confirm plan → Claude thực thi web search và viết output theo cấu trúc Phần A–E.
7. Output dài ~15.000–20.000 từ. Có thể Claude phải chia thành 2–3 message — yêu cầu "tiếp tục" để Claude hoàn thành.
8. **Sau khi nhận output**:
   - Lưu raw output thành `databricks/prompts/output/<date>_deep_research_raw.md` (tạo folder `output/` nếu chưa có).
   - Chạy validation checklist ở §7.
   - Tách section A/B/C/D về file workstream tương ứng (xem mapping §3).
   - Bổ sung header research template (xem `PLAYBOOK.md §6.1`) cho mỗi file.
   - Bổ sung tag `[FACT]/[ANALYSIS]/[ASSUMPTION]/[GAP]` nếu Claude bỏ sót.

### Citations

Không có citation — workflow nội bộ.

---

## 5. PROMPT (copy nguyên si block dưới đây vào Claude web)

```text
# ROLE

Bạn là Senior Strategy Consultant cấp cao (chuẩn Big4) chuyên đánh giá AI/Data platforms cho ngành Banking & Financial Services. Bạn được engage bởi CIO + CFO của một ngân hàng top-10 Việt Nam để nghiên cứu Databricks làm nền tảng AI Agent Platform nội bộ.

# OBJECTIVE

Thực hiện DEEP RESEARCH tổng hợp về cách thị trường đang sử dụng Databricks cho AI Agent, gồm 3 trục chính:

- Trục 1 — MARKET TRENDING: quy mô thị trường + tốc độ tăng trưởng + adoption pattern + regulatory driver.
- Trục 2 — TECHNOLOGY: capability stack của Databricks cho AI Agent + GA/preview status từng component + tiến hoá Giai đoạn 1 (trước 2026) vs Giai đoạn 2 (từ 2026).
- Trục 3 — COMPETITIVE: vị thế Databricks so với 6 nhóm đối thủ — Snowflake Cortex, AWS Bedrock+SageMaker, Azure AI Foundry, GCP Vertex AI Agent Builder, native LLM provider (OpenAI/Anthropic), OSS stack (LangChain/LlamaIndex).

Khu vực địa lý: 3 nhóm — Âu Mỹ (EU + UK + US), Trung Quốc, Việt Nam (deep dive vì là region target).

# CONTEXT

- Sponsor: CIO + CFO của ngân hàng FSI tại Việt Nam.
- Mục tiêu cuối cùng: quyết định commit hay decline R&D budget xây AI Agent Platform nội bộ trên Databricks.
- Snapshot date: 21/05/2026. Mọi feature release sau date này phải flag rõ là "ngoài snapshot".
- Output sẽ được tách và feed vào báo cáo feasibility chuẩn Big4 trình C-Level.
- Tài liệu sponsor chính của Databricks: <https://docs.databricks.com/aws/en/agents/> — bám làm starting point cho Phần B (Technology), nhưng PHẢI cross-check với nguồn độc lập (analyst, benchmark, third-party blog có data) vì là vendor docs có thiên kiến tích cực.

# RESEARCH APPROACH — THỰC HIỆN TRƯỚC KHI VIẾT OUTPUT

Bước 0 (BẮT BUỘC trước khi web search): Lập RESEARCH PLAN và trả lời lại tôi để confirm. Plan gồm:

1. List ~30–40 search query cần chạy, group theo trục × region (ví dụ: "Databricks Mosaic AI Agent Framework GA date 2024 2025", "Snowflake Cortex Agents vs Databricks comparison 2025", "Vietnam SBV GenAI banking guidance 2025", "ICBC LLM agent production case study", ...).
2. List ~10–15 nguồn primary (T1) cần truy cập: vendor docs, regulator sites, peer-reviewed papers, central bank documents.
3. Identify ~5–10 potential conflict cần cross-check (vendor claim vs analyst claim, ví dụ: "Databricks claim multi-cloud feature parity — analyst có verify không?").
4. Note phần nào có thể không tìm được data (đặc biệt Vietnam) → đề xuất proxy method (ví dụ: dùng số SE Asia rồi adjust theo GDP/banking AUM).

Sau khi tôi confirm plan → bắt đầu thực thi web search và viết output theo cấu trúc Phần A–E ở dưới.

# RESEARCH METHODOLOGY (BẮT BUỘC)

## Phân hạng nguồn (Source Tier)

- T1 (primary): vendor official docs, SEC filings, central bank documents (SBV/NHNN, ECB, Fed, PBOC), peer-reviewed papers, công bố chính thức của doanh nghiệp (annual report, investor day, earnings call).
- T2 (reputable analyst): Gartner, Forrester, IDC, McKinsey, BCG, Bain.
- T3 (reputable media): WSJ, FT, Reuters, The Information, IEEE Spectrum, MIT Technology Review.
- T4 (vendor blog / marketing / analyst sponsored): Databricks blog, Snowflake blog, AWS blog, conference keynote — chỉ làm điểm khởi đầu, PHẢI tag rõ "vendor view".
- T5 (forum / anecdotal): Reddit, Hacker News, social media — CHỈ làm hint khám phá, KHÔNG dùng làm evidence cho claim trọng yếu.

Quy tắc:
- Mọi claim trọng yếu (ảnh hưởng recommendation cho CIO/CFO) phải có ≥1 nguồn T1 hoặc T2.
- Nếu chỉ có nguồn T4/T5 cho 1 claim → claim đó BẮT BUỘC tag [ASSUMPTION] hoặc [GAP], không được tag [FACT].
- Khi cite vendor docs cho competitor: ghi rõ là "vendor official statement", không claim là fact độc lập.

## Content tag (bắt buộc gắn cho từng đoạn)

Mọi đoạn nội dung phải có đúng 1 trong 4 tag ở đầu đoạn:

- [FACT] thông tin có thể kiểm chứng từ nguồn ngoài — bắt buộc inline citation [N].
- [ANALYSIS] diễn giải / suy luận dựa trên facts — trỏ đến [FACT] đang phân tích.
- [ASSUMPTION] giả định khi data không đủ — ghi rõ điều kiện đúng + impact nếu sai.
- [GAP] thiếu data, cần collect thêm — ghi rõ data nào cần, từ nguồn nào, đề xuất ai owner + due date.

## Trung lập

- CẤM ngôn ngữ marketing: "revolutionary", "best-in-class", "leading", "industry-first", "game-changer", "cutting-edge". Thay bằng mô tả định lượng có nguồn.
- Khi mô tả Databricks: cân bằng strengths + weaknesses + open questions.
- Khi so sánh đối thủ: dùng cùng tiêu chí, cùng năm tham chiếu, cùng đơn vị đo, cùng version sản phẩm.
- Khi vendor có công bố mâu thuẫn với T2/T3 → trình bày cả hai phía + [ANALYSIS] nêu khả năng xác minh.

## Citation format (IEEE rút gọn)

Inline: dùng [N] ngay sau câu/cụm chứa claim. Một câu có thể có nhiều: [1][2][3].

Cuối mỗi Phần (A/B/C/D/E) có section "### Citations" liệt kê đầy đủ theo format:

[N] <Author/Org>, "<Title>", <Publication/Platform>, <YYYY-MM-DD or "n.d.">, <Full URL> (accessed 21/05/2026).

Ví dụ:
[1] Databricks, "Mosaic AI Agent Framework Overview", docs.databricks.com, n.d., https://docs.databricks.com/aws/en/agents/ (accessed 21/05/2026).
[2] Gartner, "Magic Quadrant for Data Science and Machine Learning Platforms", gartner.com, 2025-06-20, ID G00785432 (accessed 21/05/2026 via subscription).

# OUTPUT STRUCTURE (BẮT BUỘC)

Viết output theo cấu trúc dưới đây, với mỗi Phần có section "### Citations" ở cuối.

---

# Báo cáo Deep Research — Databricks AI Agent Platform: Thị trường, Công nghệ, Cạnh tranh

[CẬP NHẬT: 21/05/2026]

## Phần A — MARKET TRENDING

### A.1 Quy mô thị trường AI Agent enterprise

- TAM / SAM / SOM 2026–2028, CAGR.
- Methodology comparison giữa Gartner / IDC / McKinsey / Forrester — convergence vs divergence.
- Breakdown vertical: FSI chiếm % bao nhiêu của tổng AI Agent spend.
- Breakdown region: EU vs US vs APAC vs CN vs SE Asia.
- Lưu ý: nếu nguồn khác nhau cho number chênh ≥3x → trình bày range và chú thích methodology của từng nguồn.

### A.2 Adoption pattern + production case

- % enterprise có ≥1 AI Agent production (không phải POC).
- Top 5 use case được production hoá nhiều nhất.
- Time-to-production median từ POC → prod.
- Failure rate của POC AI agent (industry average).
- Bài học pattern thành công vs thất bại.

### A.3 Regulatory landscape (so sánh 3 region)

- EU: AI Act (classification tier cho banking agent), DORA (operational resilience), GDPR.
- US: Fed SR 11-7 (model risk), OCC bulletin AI, NIST AI RMF.
- China: CAC Generative AI Measures (hiệu lực 2023-08-15), PBOC fintech guidance, ràng buộc cross-border data.
- Vietnam: SBV thông tư PCI / QĐ 2345/QĐ-NHNN về an toàn giao dịch điện tử, Luật An ninh mạng 2018, NĐ 13/2023 về bảo vệ dữ liệu cá nhân — có guidance riêng cho GenAI banking chưa?

### A.4 Demand drivers FSI

- Operating cost pressure (cost-to-income ratio) bank EU/US/CN/VN 3 năm gần đây — correlation với AI investment.
- Customer experience expectation gap.
- Compliance burden (số giờ FTE / năm cho AML/KYC/reporting).
- Pressure từ fintech challenger và digital-only bank.

### Citations (Phần A)

[N] ...

## Phần B — DATABRICKS TECHNOLOGY

### B.1 Mosaic AI Agent Framework

- Anatomy: tool calling, multi-step orchestration, state management, tracing, agent-as-code.
- GA / preview status từng feature tại snapshot date.
- Multi-agent collaboration (planner / worker pattern) — có hay không?
- Limitation đã được vendor công bố hoặc cộng đồng xác nhận.

### B.2 Unity Catalog cho agent governance

- Data + model + tool + function governance hợp nhất.
- ABAC (attribute-based access control): hỗ trợ row/column level cho agent context?
- Lineage cho agent step.
- Exit path nếu rời Databricks.

### B.3 Model Serving + Foundation Model APIs

- Provisioned throughput vs pay-per-token: ngưỡng break-even.
- Model nào host tại region nào (AWS / Azure / GCP × region).
- SLA cam kết (uptime, latency P50/P99).
- Foundation model: Claude, GPT, Llama, Mistral, Gemini — coverage.

### B.4 Vector Search + RAG

- Delta-sync vs direct vector search.
- Embedding model options trong region.
- Hybrid search (semantic + keyword) — có hỗ trợ?
- Latency Delta → Vector sync.

### B.5 Agent Evaluation + MLflow

- Agent Evaluation framework, LLM-as-judge, ground truth dataset.
- MLflow tracking, model registry, A/B testing, regression detection.
- Integration với external registry (SageMaker MR, Azure ML).

### B.6 Lakehouse foundation

- Delta + Unity Catalog + Workflow là baseline data plane cho agent.
- Data freshness pattern: streaming vs batch ingestion.
- Hybrid pattern: agent on Databricks + data lakehouse on-prem (Iceberg / open table).

### B.7 Genie + Databricks Apps

- Genie (NL → SQL), AI/BI dashboard.
- Databricks Apps — front-end pattern cho deliver agent đến business user.
- Hỗ trợ tiếng Việt và domain banking đặc thù tới đâu.

### B.8 So sánh tiến hoá Giai đoạn 1 (trước 2026) vs Giai đoạn 2 (từ 2026)

Bảng so sánh ≥6 trục:

| Trục so sánh | Giai đoạn 1 (trước 2026) | Giai đoạn 2 (từ 2026) | Tác động |
|---|---|---|---|
| Agent abstraction | (vd: LangChain on Databricks + MLflow flavors) | (vd: Mosaic AI Agent Framework native) | (vd: giảm time-to-prod 30–50%) |
| Model hosting | (vd: endpoint-per-model manual) | (vd: Foundation Model APIs + pay-per-token + provisioned throughput) | … |
| Governance | … | … | … |
| Retrieval | … | … | … |
| Evaluation | … | … | … |
| Front-end | … | … | … |

### Citations (Phần B)

[N] ...

## Phần C — COMPETITIVE

### C.1 vs Snowflake Cortex (Cortex AI + Cortex Agents + Native Apps)

Strengths Snowflake | Weaknesses Snowflake | Win/loss vs Databricks

### C.2 vs AWS Bedrock + Bedrock Agents + SageMaker

### C.3 vs Azure AI Foundry (Studio + Azure OpenAI + Azure ML)

### C.4 vs GCP Vertex AI Agent Builder + Gemini

### C.5 vs Native LLM stack (OpenAI Assistants API, Anthropic Claude tool use, Cohere, Mistral)

### C.6 vs OSS stack (LangChain + LlamaIndex + LangGraph + self-managed)

### C.7 FEATURE MATRIX TỔNG HỢP

8 trục × 7 vendor (Databricks + 6 đối thủ), mỗi cell score 0–3 + evidence pointer [N].

8 trục: (1) Agent framework, (2) Model availability, (3) Governance, (4) Retrieval/RAG, (5) Evaluation, (6) MLOps maturity, (7) Security/Compliance, (8) Pricing/TCO indicator.

### C.8 Decision tree — khi nào chọn Databricks vs alternative

Cho 4 use case archetype banking: (a) Customer-facing RAG, (b) RM/employee copilot, (c) Fraud co-pilot, (d) Compliance / AML assistant.

### Citations (Phần C)

[N] ...

## Phần D — REGIONAL DEEP DIVE

### D.1 EU/US

- Adoption Databricks tại bank EU lớn: HSBC, ING, BBVA, Erste, Deutsche Bank, BNP Paribas, Santander — case nào production, evidence từ investor day / annual report.
- Adoption tại bank US: JPMC (IndexGPT/LLM Suite), BoA (Erica + GenAI), Goldman, Wells Fargo, Citi, Capital One.
- Use case production specific (chứ không phải POC marketing).
- Regulatory framing áp dụng: AI Act + DORA + Fed SR 11-7 + OCC.

### D.2 Trung Quốc

- Databricks footprint tại CN (có region không? bank CN có dùng không?).
- Bank lớn CN (ICBC, CCB, BoC, ABC, Ping An, WeBank, MyBank, Ant Group) — dùng foundation model nội địa (Baichuan, GLM, Qwen, ERNIE, DeepSeek) hay vendor foreign?
- Tech stack architecture so với western counterpart.
- CAC Generative AI Measures + PBOC: ràng buộc cross-border data, model approval.

### D.3 Việt Nam (DEEP DIVE — region target)

- Bank top-10 VN: VCB, BIDV, VietinBank, Techcombank, MB, ACB, TPBank, VPBank, SHB, HDBank.
- Stage hiện tại: chatbot truyền thống vs GenAI assistant vs agentic agent — evidence từ annual report / investor day / news.
- AI initiative đã công bố công khai.
- Databricks footprint tại VN/SE Asia: có region không (Singapore? Jakarta?)? Bank VN nào đã engage Databricks?
- SBV/NHNN framework: thông tư PCI, QĐ 2345/QĐ-NHNN, AML guidance — có guidance riêng GenAI/AI Agent banking chưa? Nếu chưa, fallback dùng framework nào?
- Gap với EU baseline + đề xuất roadmap khả thi cho ngân hàng VN.

### Citations (Phần D)

[N] ...

## Phần E — SYNTHESIS

### E.1 Top 5 finding cho CIO (focus technology + competitive position)

### E.2 Top 5 finding cho CFO (focus market size + adoption ROI)

### E.3 Top 5 risk + mitigation đề xuất

### E.4 Open questions cần follow-up

Mỗi question kèm đề xuất: ai có thể trả lời (vendor briefing? Procurement? SBV consultation? industry SME?).

### E.5 GAPs — data không tìm được trên public web

List rõ ràng từng GAP + proposed method để fill (ví dụ: "Vendor quote chính thức Databricks — owner: Head of Procurement — due: 2026-07-31").

### Citations (Phần E)

[N] ...

# DEFINITION OF DONE

Trước khi kết thúc output, tự self-check:

- [ ] Mọi đoạn nội dung có đúng 1 tag [FACT]/[ANALYSIS]/[ASSUMPTION]/[GAP] ở đầu.
- [ ] Mọi [FACT] có inline citation [N] và entry trong section "### Citations" tương ứng.
- [ ] ≥60% claim trọng yếu (ảnh hưởng recommendation) có nguồn T1 hoặc T2.
- [ ] Mỗi region D.1/D.2/D.3 có ≥3 case study production (có evidence).
- [ ] Feature matrix C.7 đầy đủ 8 trục × 7 vendor, mỗi cell có score + evidence.
- [ ] Bảng so sánh G1 vs G2 ở B.8 có ≥6 trục.
- [ ] ≥5 open question (E.4) và ≥5 gap (E.5) có proposed-method.
- [ ] Không có ngôn ngữ marketing.
- [ ] Snapshot date ghi rõ ở đầu output.
- [ ] Mọi URL có "accessed 21/05/2026".

# CONSTRAINTS (NHẮC LẠI)

- Ngôn ngữ: tiếng Việt có dấu chuẩn 100%. Thuật ngữ kỹ thuật giữ tiếng Anh (RAG, LLM, MLOps, Unity Catalog, Mosaic AI, Vector Search, embedding, fine-tuning, prompt injection, foundation model).
- Thuật ngữ nghiệp vụ: tiếng Việt + tiếng Anh trong ngoặc khi dễ nhầm.
- KHÔNG bịa số liệu — thà ghi [GAP] còn hơn quote sai.
- KHÔNG dùng vendor self-comparison page làm primary source.
- KHÔNG dùng marketing buzzword.
- Snapshot 21/05/2026 — flag rõ feature release sau date này.

# OUTPUT LENGTH

Target 15.000–20.000 từ. Đây là báo cáo comprehensive, không phải executive summary. Nếu output dài hơn message limit của Claude, dừng ở section logic và chờ tôi gõ "tiếp tục" để bạn hoàn thành phần còn lại.

# START

Bước 0: Trả lời tôi với RESEARCH PLAN (~30–40 query + ~10–15 source priority + ~5–10 conflict cần cross-check + GAP có khả năng). Đợi tôi confirm xong mới chạy web search.
```

### Citations

Không có citation — đây là prompt template, các citation sẽ được Claude web tự generate khi chạy.

---

## 6. Output expected structure

Sau khi Claude web hoàn thành, output phải có 5 phần chính (A–E) với tổng độ dài 15.000–20.000 từ:

| Phần | Sections | Độ dài ước tính | File workspace nhận |
|---|---|---|---|
| A. Market Trending | A.1–A.4 | ~2.500–3.500 từ | `docs/02_Market_Landscape/0X_*.md` |
| B. Databricks Technology | B.1–B.8 | ~4.000–5.000 từ | `docs/03_Databricks_Technology/0X_*.md` |
| C. Competitive | C.1–C.8 | ~4.000–5.000 từ | `docs/04_Competitive_Analysis/0X_*.md` |
| D. Regional Deep Dive | D.1–D.3 | ~3.000–4.000 từ | `docs/05_Banking_FSI_Applications/01–03_*.md` (bổ trợ) |
| E. Synthesis | E.1–E.5 | ~1.500–2.500 từ | input cho `docs/01_Executive_Summary/01_key_findings.md` |

Mỗi Phần đều có `### Citations` ở cuối, format IEEE rút gọn.

### Citations

Không có citation — bảng cấu trúc output.

---

## 7. Validation checklist sau khi nhận output

Trước khi copy output về file workspace, **bắt buộc** check:

- [ ] Output có đầy đủ 5 phần A/B/C/D/E.
- [ ] Mỗi phần có section `### Citations` ở cuối.
- [ ] Mọi đoạn có tag `[FACT]/[ANALYSIS]/[ASSUMPTION]/[GAP]` ở đầu — nếu thiếu → bổ sung thủ công hoặc yêu cầu Claude tag lại.
- [ ] Mọi `[FACT]` có inline citation `[N]` trỏ đúng entry.
- [ ] Random check ≥10 citation: URL truy cập được + nội dung khớp claim (chống hallucination).
- [ ] ≥60% claim trọng yếu có T1 hoặc T2 — đếm tỷ lệ thủ công cho random sample 30 citation.
- [ ] Phần D.3 (Việt Nam) có ≥3 case study với evidence — đây là phần Claude dễ bịa nhất → check kỹ.
- [ ] Feature matrix C.7 đầy đủ 8 trục × 7 vendor — không có cell trống không có [GAP].
- [ ] So sánh G1 vs G2 ở B.8 có ≥6 trục.
- [ ] Không có ngôn ngữ marketing ("revolutionary", "best-in-class", "leading", ...).
- [ ] Số liệu Việt Nam: cross-check ít nhất 2 nguồn (vì Claude dễ extrapolate sai từ SE Asia).

Nếu phát hiện hallucination citation → re-prompt với hint cụ thể: "Citation [N] cho claim X không tìm thấy nguồn — verify lại hoặc đổi sang [GAP]".

### Citations

Không có citation — checklist validation nội bộ.

---

## 8. Known limitations

- **Vendor quote chính thức**: Claude web KHÔNG truy cập được pricing đàm phán riêng. Phải đợi Procurement (R-02 ở `docs/WORKSPACE_TRACKING.md`).
- **Internal SME interview**: thông tin từ industry contact không lên public web → phải làm riêng (Q-06).
- **Roadmap sản phẩm internal**: không có trên web (R-04). Cần sponsor approve access.
- **Region availability mới nhất của Databricks**: status page Databricks có thể bị cache → cross-check với Sales rep nếu cần.
- **Số liệu Việt Nam**: public web thường thiếu, Claude có thể extrapolate từ SE Asia → [GAP] cần xử lý riêng.
- **Foundation model availability per region**: thay đổi nhanh (mỗi 2–4 tuần) — snapshot 21/05/2026 sẽ obsolete sau ~3 tháng. Cần re-snapshot 2026-08 trước final review (R-05).

### Citations

Không có citation — section ghi rõ giới hạn tự nhiên của tool.

---

## 9. Variant prompts (đề xuất khi cần drill-down sau)

Sau khi chạy prompt chính ở §5 và nhận output, có thể chạy 2 prompt phụ ngắn hơn để bổ sung depth:

- **Variant 1 — Vietnam deep dive**: chỉ Phần D.3, target 5.000 từ, focus 10 bank top + SBV framework + use case khả thi. Khi nào dùng: nếu Phần D.3 ở prompt chính bị shallow.
- **Variant 2 — Pricing & TCO**: chỉ Phần C.7 trục "Pricing/TCO indicator", target 3.000 từ, list price chính thức 7 vendor + công thức tính + ngưỡng break-even. Khi nào dùng: input cho `docs/07_Business_Case_And_Risk/03_pricing_analysis.md` (workstream 07 — tuy không trong scope prompt chính nhưng overlap data).

Hai variant này sẽ được tạo thành file riêng `prompts/deep_research_vietnam_drill_down.md` và `prompts/deep_research_pricing_drill_down.md` khi user yêu cầu.

### Citations

Không có citation — đề xuất nội bộ.

---

## 10. Change log

| Ngày | Phiên bản | Thay đổi | Tác giả |
|---|---|---|---|
| 21/05/2026 | v0.1 | Khởi tạo prompt deep research — cover 3 trục (Market/Technology/Competitive) × 3 region (EU-US/CN/VN) cho Claude web + extended thinking. Đầu ra map vào workstream 02/03/04 (+ bổ trợ 05). | Lead Consultant |

### Citations

Tham chiếu prompt khởi tạo gốc: `databricks/prompts/first_insight.md`. Reference rule: `databricks/CLAUDE.md`, `databricks/docs/RULES.md`. Reference workstream overview: `databricks/docs/02_Market_Landscape/_overview.md`, `03_Databricks_Technology/_overview.md`, `04_Competitive_Analysis/_overview.md`.
