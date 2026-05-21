# 02. Market Landscape — Overview

**Workstream**: 02_Market_Landscape
**File ID**: 02-00
**Version**: v0.1
**Status**: DRAFT
**Owner**: TBD
**Reviewer**: Head of Strategy
**Last updated**: 2026-05-21
**Classification**: Internal

> **Mục tiêu workstream**: Mô tả định lượng và định tính bức tranh thị trường **AI Agent** (toàn cầu + FSI) trong 24 tháng tới, làm cơ sở narrative cho Executive Summary và làm denominator cho TCO/ROI scenarios.
>
> **Key questions**:
> - [FACT] Quy mô thị trường AI Agent (TAM/SAM/SOM) toàn cầu và banking/FSI cụ thể là bao nhiêu?
> - [ANALYSIS] Adoption curve đang ở giai đoạn nào (innovator / early adopter / early majority)? Bằng chứng nào?
> - [FACT] Driver pháp lý nào đang **đẩy** hoặc **kìm** adoption tại 3 nhóm thị trường (EU/US, Trung Quốc, Việt Nam)?
>
> **Audience chính**: CIO, Head of Digital Banking, Head of Strategy.

---

## 1. Scope

### 1.1 In-scope

- Market sizing AI Agent **chuyên cho enterprise** (loại trừ consumer chatbot).
- Adoption curve: tỷ lệ enterprise có ≥1 AI Agent **production** (không phải POC).
- Driver pháp lý: AI Act (EU), Generative AI Measures (CN), SBV/NHNN guidance (VN), Fed SR 11-7 model risk (US).
- Demand driver chuyên cho FSI: pressure từ operating cost, customer experience, compliance load.

### 1.2 Out-of-scope

- Market sizing GenAI tổng thể (đã có nhiều báo cáo công khai, không add value).
- Geographic detail ngoài 3 nhóm (Latam, MENA, India) — note `[GAP]` nếu cần sau.
- Consumer-facing chatbot, agent trong gaming / entertainment.

## 2. Strategic context

- [ANALYSIS] Số liệu thị trường là **input cho narrative**, không phải kết luận. Tránh dùng size để biện minh đầu tư — cần kết hợp với 03/04/05/07.
- [ASSUMPTION] Giả định analyst report Gartner/IDC/Forrester về "Agentic AI" hợp lệ về phương pháp luận. Nếu sai → cross-check với báo cáo McKinsey/BCG. Impact: medium.

## 3. Dàn ý file con

| File | Mục tiêu (1–2 câu) | Owner | Trạng thái |
|---|---|---|---|
| `_overview.md` | (file này) Scope + dàn ý + key questions cho Market Landscape. | TBD | DRAFT |
| `01_ai_agent_market_sizing.md` | TAM/SAM/SOM 2026–2028 với 3+ nguồn (Gartner, IDC, McKinsey); methodology comparison; SOM cho banking/FSI. | TBD | NOT_STARTED |
| `02_adoption_trends.md` | Adoption curve, % enterprise có agent in production, top use case, time-to-production median, failure rate of POC. | TBD | NOT_STARTED |
| `03_regulatory_environment.md` | Mapping AI Act (EU), Generative AI Measures (CN), SBV/NHNN (VN), Fed/OCC (US); timeline hiệu lực; impact lên agent deployment. | TBD | NOT_STARTED |
| `04_demand_drivers_fsi.md` | Tại sao FSI là vertical "ăn dày" nhất AI Agent: cost-to-serve, compliance burden, customer expectation gap; bằng chứng định lượng. | TBD | NOT_STARTED |

## 4. Key questions phải trả lời (chi tiết)

1. Tốc độ tăng trưởng CAGR 2026–2030 của AI Agent enterprise market — Gartner / IDC / McKinsey có hội tụ không?
2. Banking/FSI chiếm bao nhiêu % spend AI Agent toàn cầu? Tỷ lệ này có tăng / giảm so với 2024?
3. Use case nào được production hóa nhiều nhất (RAG Q&A, RM copilot, fraud co-pilot, compliance assistant)?
4. EU AI Act phân loại agent banking ở risk tier nào? Yêu cầu kỹ thuật cụ thể gì?
5. Trung Quốc: Generative AI Measures (有效 2023-08-15) áp dụng cho on-prem agent hay chỉ public service? Ảnh hưởng cross-border data?
6. Việt Nam: SBV có guidance riêng cho GenAI / AI Agent trong banking không? Hay đang dùng general framework an toàn giao dịch điện tử?
7. Operating cost pressure (cost-to-serve / cost-to-income ratio) của bank EU/US/CN/VN trong 3 năm gần đây — corellation với AI investment?

## 5. Cross-references (dependencies)

| Liên quan tới | Mục đích |
|---|---|
| → `../01_Executive_Summary/01_key_findings.md` | Cung cấp market context cho narrative |
| → `../05_Banking_FSI_Applications/` (3 region) | Cung cấp regulatory baseline để discuss use case |
| → `../07_Business_Case_And_Risk/02_roi_scenarios.md` | Cung cấp denominator (market size) cho SOM-based ROI |
| ← `../05_Banking_FSI_Applications/04_use_cases_catalog.md` | Use case catalog bổ sung cho `02_adoption_trends.md` |

## 6. Sources & evidence plan

| Loại nguồn | Cụ thể | Tier |
|---|---|---|
| Vendor / analyst report | Gartner "Hype Cycle for AI 2025", IDC FutureScape AI, Forrester Wave AI Platforms | T2 |
| Consulting firm | McKinsey "The State of AI", BCG "Build for the Future", Bain Tech Report | T2 |
| Regulator | EU AI Act consolidated text, CAC Generative AI Measures, SBV thông tư, OCC bulletin SR 11-7 | T1 |
| Industry survey | MIT/Sloan Management Review survey, Stanford AI Index | T2 |
| Vendor blog | Databricks blog, Snowflake blog (chỉ làm gợi ý) | T4 — phải tag rõ |

[GAP] Số liệu market cho **Vietnam riêng** thường không có. Đề xuất proxy: SE Asia + Vietnam-specific multiplier theo GDP/banking AUM. Owner: TBD. Due: 2026-06-10.

## 7. Open questions từ tracking (subset)

- Q-06 (case study Vietnam: tự khảo sát hay thuê analyst) — ảnh hưởng `01_ai_agent_market_sizing.md` và `02_adoption_trends.md` cho region VN.

## 8. Risks & GAPs

| ID | Risk | Severity | Mitigation |
|---|---|---|---|
| R-Mkt-01 | Analyst report khác nhau cho size chênh ≥3x (Gartner vs IDC vs vendor) | H | Trình **range** không phải single number; chú thích methodology của mỗi nguồn |
| R-Mkt-02 | Số liệu Việt Nam thiếu, dễ over-extrapolate từ SE Asia | M | Đánh dấu `[ASSUMPTION]` rõ ràng, không quote như `[FACT]` |
| R-Mkt-03 (= R-03 ở tracking) | China regulatory thay đổi nhanh | M | Snapshot date + version; cross-check 2 T2 sources |

## 9. Definition of Done — workstream level

- [ ] `_overview.md` APPROVED.
- [ ] 4 file con APPROVED.
- [ ] Mọi market size có ≥3 nguồn T1/T2 hội tụ hoặc note rõ phân kỳ.
- [ ] Mọi regulation có ngày hiệu lực + URL văn bản gốc.
- [ ] Số liệu Vietnam có đề xuất proxy method nếu primary data không đủ.
- [ ] `../WORKSPACE_TRACKING.md` cập nhật status.

## 10. Change log

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-05-21 | v0.1 | Khởi tạo overview Market Landscape | Lead Consultant |
