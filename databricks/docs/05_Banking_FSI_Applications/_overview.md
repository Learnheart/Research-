# 05. Banking / FSI Applications — Overview

**Workstream**: 05_Banking_FSI_Applications
**File ID**: 05-00
**Version**: v0.1
**Status**: DRAFT
**Owner**: TBD
**Reviewer**: Head of Digital Banking · CRO (cho phần regulatory)
**Last updated**: 2026-05-21
**Classification**: Internal

> **Mục tiêu workstream**: Tổng hợp **bằng chứng triển khai thật** của AI Agent trong banking/FSI tại 3 nhóm thị trường (Âu Mỹ, Trung Quốc, Việt Nam), kèm khung quy định, cùng catalog use case có khả năng nhân bản cho nội bộ.
>
> **Key questions**:
> - [FACT] Use case AI Agent nào đã **lên production** trong banking ở mỗi region (không phải POC, không phải proof of concept marketing)?
> - [ANALYSIS] Use case nào của EU/US có thể **port nguyên trạng** sang Việt Nam, use case nào cần điều chỉnh do quy định / hành vi khách hàng?
> - [FACT] Khung pháp lý của SBV / NHNN, ECB, OCC, PBOC áp dụng cho AI Agent banking — yêu cầu cụ thể nào?
>
> **Audience chính**: Head of Digital Banking, Head of Risk/Compliance, CIO.

---

## 1. Scope

### 1.1 In-scope

- **3 region**: Âu Mỹ (EU + UK + US), Trung Quốc, Việt Nam (deep dive — vì là target market).
- Use case catalog: 8–15 use case quy về 4 archetype (customer-facing, RM/employee copilot, ops automation, risk/compliance assistant).
- Mapping quy định: AI Act, GDPR, EU DORA, Fed SR 11-7, OCC bulletin, PBOC, CAC Generative AI Measures, SBV thông tư PCI/giao dịch điện tử.
- Vendor / consulting case study (có nguồn) cho mỗi use case.

### 1.2 Out-of-scope

- Cài đặt kỹ thuật chi tiết của use case (đẩy sang `../03_Databricks_Technology/` cho capability + `../06_Strategic_Roadmap/` cho phasing).
- TCO chi tiết per use case (đẩy sang `../07_Business_Case_And_Risk/01_tco_model.md`).
- Khu vực ngoài 3 nhóm (Latam, MENA, India, ASEAN ngoài VN) — note `[GAP]`.

## 2. Strategic context

- [ANALYSIS] Đây là workstream **dễ rơi vào quảng cáo vendor** nhất vì vendor + consulting firm xuất bản nhiều case study chỉ phục vụ marketing. Phải bám tiêu chí Tier (`../RULES.md §4.3`).
- [FACT] Tài liệu sponsor: <https://docs.databricks.com/aws/en/agents/> không phải nguồn cho region/regulatory — chỉ cho capability mapping vào use case.
- [ASSUMPTION] Giả định trong 24 tháng tới, SBV/NHNN sẽ ban hành guidance dedicated cho GenAI banking. Nếu không → use case Việt Nam phải tự ánh xạ vào thông tư chung; impact: medium (recommendation timeline có thể trượt 6 tháng).

## 3. Dàn ý file con

| File | Mục tiêu (1–2 câu) | Owner | Trạng thái |
|---|---|---|---|
| `_overview.md` | (file này) Scope, methodology, dàn ý 5 file con. | TBD | DRAFT |
| `01_eu_us_landscape.md` | Adoption + case study production tại JPMC, BoA, Goldman, HSBC, ING, BBVA, Erste; framing AI Act + DORA + Fed SR 11-7. | TBD | NOT_STARTED |
| `02_china_landscape.md` | ICBC, CCB, Ping An, WeBank, MyBank; framing CAC Generative AI Measures + PBOC guidance; ràng buộc cross-border data. | TBD | NOT_STARTED |
| `03_vietnam_landscape.md` | Adoption tại VCB, BIDV, Techcombank, MB, TPBank, VPBank; framing SBV thông tư PCI/giao dịch điện tử/AML; gap với EU/US baseline. | TBD | NOT_STARTED |
| `04_use_cases_catalog.md` | 8–15 use case quy về 4 archetype, cho mỗi use case có: mô tả, capability cần (link đến 03), region đã production, regulatory note. | TBD | NOT_STARTED |
| `05_regulatory_banking.md` | Mapping bảng quy định × use case archetype × control yêu cầu (data, model, human-in-loop, audit trail). | TBD | NOT_STARTED |

## 4. Key questions phải trả lời (chi tiết)

1. JPMC "IndexGPT" / "LLM Suite" — actual production scope (FTE saved, accuracy, complaint rate) chứ không chỉ headline?
2. ICBC / Ping An có deploy LLM agent với foundation model nội địa (Baichuan, GLM, Qwen)? Architecture so với western counterpart?
3. Techcombank / MB / VPBank đang ở giai đoạn nào — chatbot truyền thống, GenAI assistant, hay agentic? Evidence?
4. Tỷ lệ use case customer-facing vs internal-facing — bài học: production agent thành công thường internal-first?
5. EU AI Act: agent cho credit decision / fraud detection vào high-risk tier, yêu cầu kỹ thuật cụ thể nào (conformity assessment, post-market monitoring)?
6. SBV: thông tư nào áp dụng cho AI in banking — hiện chỉ có thông tư an toàn giao dịch điện tử + AML, có guidance riêng GenAI chưa?
7. DORA (operational resilience EU, hiệu lực 2025-01) ảnh hưởng vendor concentration risk Databricks ra sao?
8. Cross-border data: Trung Quốc cấm xuất khẩu personal data → impact lên model serving region?
9. Customer expectation: ngân hàng VN có insight nào về acceptance của khách hàng với AI agent (so với chatbot truyền thống)?

## 5. Cross-references (dependencies)

| Liên quan tới | Mục đích |
|---|---|
| ← `../02_Market_Landscape/03_regulatory_environment.md` | Lấy regulatory framing cấp cao |
| → `../03_Databricks_Technology/` | Use case → capability mapping (verify Databricks có cover) |
| → `../06_Strategic_Roadmap/01_rd_phase.md` / `02_poc_phase.md` | Use case nào ưu tiên POC |
| → `../07_Business_Case_And_Risk/08_compliance_mapping.md` | Output regulatory mapping ăn vào compliance file |
| → `../01_Executive_Summary/01_key_findings.md` | Top 2–3 use case VN-ready |

## 6. Sources & evidence plan

| Region | Primary (T1) | Secondary (T2/T3) |
|---|---|---|
| EU | EU AI Act consolidated text, ECB supervisory expectations, EBA guidelines | Gartner banking practice, McKinsey "AI in Banking" |
| US | Fed SR 11-7, OCC Bulletin 2011-12 + AI updates, NIST AI RMF | Forrester, BCG banking |
| China | CAC Generative AI Measures 2023-08, PBOC fintech development plan | McKinsey China, China Banking Association |
| Việt Nam | SBV: thông tư PCI, QĐ 2345/QĐ-NHNN, Luật an ninh mạng 2018, NĐ 13/2023 PII | VNBA, VietinResearch, FiinResearch, McKinsey Vietnam |
| Bank case study | Investor day, annual report (T1), earnings call transcript | The Banker, FT Lex, Asian Banker awards |

[GAP] Case study Vietnam ít công bố công khai về AI agent production. Owner: TBD. Due: 2026-06-30. Cần khảo sát qua industry contacts (xem Q-06 tracking).

## 7. Open questions từ tracking (subset)

- Q-06 (case study VN tự khảo sát vs analyst) — block `03_vietnam_landscape.md`.
- Q-03 (region compliance VN — chấp nhận US/EU region hay bắt buộc on-prem) — ảnh hưởng feasibility các use case customer-facing.

## 8. Risks & GAPs

| ID | Risk | Severity | Mitigation |
|---|---|---|---|
| R-FSI-01 (= R-01 tracking) | Thiếu primary data về Vietnam banking AI adoption | M | Industry contacts + SBV/NHNN báo cáo; nếu không có → tag `[GAP]` rõ |
| R-FSI-02 (= R-03 tracking) | China regulatory thay đổi nhanh | M | Snapshot date + version + cross-check 2 T2 |
| R-FSI-03 | Bank case study thường marketing, thiếu metric khả tín | H | Loại bỏ claim không có evidence định lượng, hoặc dùng `[ASSUMPTION]` |
| [GAP] | Customer acceptance research cho AI banking VN — chưa có dataset công | – | Owner: TBD. Due: 2026-07-15. Đề xuất tự survey nhỏ qua kênh CX nội bộ |

## 9. Definition of Done — workstream level

- [ ] `_overview.md` APPROVED.
- [ ] 5 file con APPROVED.
- [ ] Mỗi region có ≥3 case study production (có evidence) và ≥1 framework quy định.
- [ ] Use case catalog có ≥10 use case, mỗi use case có capability mapping → file workstream 03.
- [ ] Regulatory mapping bảng đầy đủ × 4 region × 4 archetype.
- [ ] Cross-check VN section với `../07_Business_Case_And_Risk/05_data_residency_sovereignty.md`.
- [ ] `../WORKSPACE_TRACKING.md` cập nhật status.

## 10. Change log

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-05-21 | v0.1 | Khởi tạo overview Banking/FSI Applications | Lead Consultant |
