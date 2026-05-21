# 08. References & Appendix — Overview

**Workstream**: 08_References_Appendix
**File ID**: 08-00
**Version**: v0.1
**Status**: DRAFT
**Owner**: TBD
**Reviewer**: Lead Consultant (kiểm tra coverage citation)
**Last updated**: 2026-05-21
**Classification**: Internal

> **Mục tiêu workstream**: Là **single source of truth** cho mọi nguồn dẫn chiếu, từ vựng, và interview note của workspace — đảm bảo workstream 01–07 không bị "citation drift" và mọi `[FACT]` có đường truy ngược.
>
> **Key questions**:
> - [FACT] Mọi citation `[N]` trong file workstream 01–07 có tương ứng entry T1/T2/T3 đầy đủ tại đây không?
> - [ANALYSIS] Tỷ lệ T1/T2 trên tổng citation có đủ chuẩn Big4 (≥60% T1+T2 cho claim trọng yếu)?
>
> **Audience chính**: Lead Consultant, reviewer (kiểm tra evidence), audit downstream.

---

## 1. Scope

### 1.1 In-scope

- Tập hợp citation **toàn workspace** (không phải citation file-level — file-level vẫn ở cuối từng file theo RULES §4).
- Glossary thuật ngữ kỹ thuật + nghiệp vụ + viết tắt.
- Interview note (nếu có industry contact / vendor briefing / internal SME).
- Hình ảnh, diagram (lưu tại đây, file workstream reference bằng relative path — `../RULES.md §1.4`).

### 1.2 Out-of-scope

- Tạo nội dung phân tích mới (workstream này chỉ aggregate).
- Tạo template trình bày C-Level (đó là phần Exec Summary / output channel).

## 2. Strategic context

- [ANALYSIS] Big4 audit chuẩn yêu cầu mọi claim có evidence truy được trong ≤30 giây. File `01_primary_sources.md` là chốt cuối — nếu reviewer mất hơn 30 giây tìm nguồn, evidence không đạt.
- [ASSUMPTION] Giả định reviewer C-Level chỉ đọc Executive Summary, không drill xuống Appendix. Internal Audit và CRO sẽ drill. Impact: medium — phải tối ưu cho 2 audience khác nhau.

## 3. Dàn ý file con

| File | Mục tiêu (1–2 câu) | Owner | Trạng thái |
|---|---|---|---|
| `_overview.md` | (file này) Scope, citation policy, dàn ý. | TBD | DRAFT |
| `01_primary_sources.md` | Master list nguồn T1 (vendor official docs, regulator, peer-reviewed, SEC, NHNN) — đầu vào của mọi citation `[N]` trọng yếu. | TBD | NOT_STARTED |
| `02_secondary_sources.md` | Master list T2 (Gartner, Forrester, IDC, McKinsey, BCG, Bain), T3 (WSJ, FT, Reuters, The Information, IEEE), với note phương pháp + thiên kiến. | TBD | NOT_STARTED |
| `03_glossary.md` | Thuật ngữ kỹ thuật (RAG, LLM, MLOps, vector store, Mosaic AI, Unity Catalog…) + nghiệp vụ (KYC, AML, RM, CIF, AUM, DORA, SR 11-7, SBV…), tiếng Việt – tiếng Anh. | TBD | NOT_STARTED |
| `04_interview_notes.md` | Note phỏng vấn (vendor briefing, industry contact, SME internal); mỗi note có metadata (date, attendee, attribution policy). | TBD | NOT_STARTED |

## 4. Citation policy (cross-link tới RULES.md)

Tóm tắt nhanh từ `../RULES.md §4`:

- Format IEEE rút gọn: `[N] Author/Org, "Title", Publication/Platform, YYYY-MM-DD, URL (accessed YYYY-MM-DD).`
- 5 Tier: T1 primary, T2 reputable analyst, T3 reputable media, T4 vendor blog/marketing, T5 forum.
- Mỗi claim trọng yếu cần ≥1 T1 hoặc T2.
- T4/T5 chỉ làm exploratory; nếu là evidence duy nhất → claim phải tag `[ASSUMPTION]` hoặc `[GAP]`.
- Disclosure bắt buộc khi cite sponsored content.

### 4.1 Convention dành riêng workspace này

- Citation `[N]` trong file workstream **trỏ tới reference list cuối file đó** (file-local).
- Khi cùng 1 source dùng ở ≥2 file → vẫn re-cite ở file mới, **không** chéo file (chống citation drift).
- `01_primary_sources.md` / `02_secondary_sources.md` là **master index** (deduplicated), không phải reference list của workstream 01–07.

### 4.2 Convention cho Databricks docs

[FACT] Tài liệu sponsor: <https://docs.databricks.com/aws/en/agents/> — cập nhật liên tục. Format chuẩn:

```
[N] Databricks, "<page title>", docs.databricks.com,
    <publication date if shown, else "n.d.">,
    <full URL> (accessed YYYY-MM-DD).
```

Nếu page không có publication date → ghi `n.d.` + bắt buộc kèm accessed date.

## 5. Glossary policy

- Bổ sung **mỗi khi** gặp thuật ngữ mới trong workstream 01–07 (xem `../RULES.md §9`).
- Format: `Thuật ngữ (viết tắt) — định nghĩa 1–2 câu. Context dùng trong workspace.`
- Tách 3 nhóm: Technical / Business / Regulatory.

## 6. Interview notes policy

- Mỗi interview ghi metadata: date, attendee, organization, attribution policy (`on-the-record` / `Chatham House` / `confidential`).
- Quote trực tiếp **chỉ** khi `on-the-record`.
- Internal SME interview: anonymize danh tính nếu non-attribution.

## 7. Cross-references (dependencies)

| Liên quan tới | Mục đích |
|---|---|
| ← Tất cả workstream 01–07 | Mỗi file submit phải đồng bộ citation về 08 |
| → `../README.md §8` | Coverage citation là một trong các DoD workspace-level |
| → `../RULES.md §4`, `§9` | Policy authoritative ở đó, file này chỉ áp dụng |

## 8. Risks & GAPs

| ID | Risk | Severity | Mitigation |
|---|---|---|---|
| R-Ref-01 | Citation drift giữa file con và master index | M | Mỗi đợt close workstream, chạy cross-check thủ công (sau này có thể automate) |
| R-Ref-02 | Vendor docs URL có thể đổi sau release → broken link | L | Bắt buộc accessed date; nếu URL chết → archive bằng Wayback Machine |
| R-Ref-03 | Interview note có PII / confidential bị leak | H | Classification `Confidential` cho file 04; access list rõ ràng (`../RULES.md §10`) |

## 9. Definition of Done — workstream level

- [ ] `_overview.md` APPROVED.
- [ ] 4 file con APPROVED.
- [ ] Mọi `[FACT]` ở workstream 01–07 có tương ứng entry trong 01/02 (cross-check).
- [ ] Glossary phủ ≥95% thuật ngữ xuất hiện trong workspace.
- [ ] Interview notes có metadata đầy đủ + classification phù hợp.
- [ ] Tỷ lệ T1+T2 ≥60% trên tổng claim trọng yếu (định nghĩa "trọng yếu" = claim ảnh hưởng recommendation cuối).
- [ ] `../WORKSPACE_TRACKING.md` cập nhật status.

## 10. Change log

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-05-21 | v0.1 | Khởi tạo overview References & Appendix; định nghĩa citation/glossary/interview policy ở mức workstream | Lead Consultant |
