# RULES — Quy ước làm việc

**Workspace**: Databricks AI Agent Platform — Strategic Research
**Version**: v0.1-draft
**Last updated**: 2026-05-21
**Owner**: Lead Consultant
**Classification**: Internal

Tài liệu này là **contract làm việc chung** cho toàn workspace. Mọi thành viên đóng góp nội dung phải đọc trước khi viết. Khi xung đột với hướng dẫn cá nhân, RULES này thắng. Mọi đề xuất sửa đổi phải qua change log (§12).

---

## 1. Naming convention

### 1.1 Folder (Level 2 — workstream)
- Format: `NN_PascalCase_With_Underscore/`, `NN` = 2 chữ số (01–09), tăng dần theo thứ tự đọc.
- Không dấu cách, không dấu tiếng Việt, không ký tự đặc biệt ngoài `_`.
- **Không** tạo subfolder bên trong workstream — workspace có **đúng 2 levels** (root + workstream).
- Hợp lệ: `03_Databricks_Technology/`, `07_Risk_Compliance_Security/`.
- Không hợp lệ: `03-databricks-tech/`, `03_Databricks Technology/`, `03_Databricks/Technology/`.

### 1.2 File markdown trong workstream folder
- Format: `NN_snake_case_topic.md`, `NN` = 2 chữ số thứ tự đọc trong workstream (01, 02, ...).
- **Bắt buộc** 1 file `_overview.md` (không có `NN`, prefix `_` để pin lên đầu danh sách) trong mỗi workstream — chứa scope, dàn ý, key questions.
- Hợp lệ: `01_mosaic_ai_agent_framework.md`, `_overview.md`.
- Không hợp lệ: `mosaic-ai-agent-framework.md`, `1_mosaic_ai.md`, `Mosaic_AI.md`.

### 1.3 Section heading trong file
- `# H1`: **1 lần duy nhất** — tiêu đề file.
- `## H2`: section chính, đánh số `## 1. ...`, `## 2. ...`.
- `### H3`: subsection, không đánh số (dùng tiêu đề mô tả).
- `#### H4`: hạn chế dùng; nếu cần >3 levels, cân nhắc tách file.

### 1.4 Bảng, hình, code block
- Bảng: phải có header row + alignment.
- Hình: nếu insert ảnh, đặt tại `09_References_Appendix/` (hoặc note `[GAP] cần diagram`). Reference bằng relative path.
- Code block: chỉ định ngôn ngữ (` ```python `, ` ```sql `, ` ```yaml `).

---

## 2. File header template (bắt buộc)

Mọi file `.md` trong workstream folder (trừ root files `README.md`, `WORKSPACE_TRACKING.md`, `RULES.md`) phải mở đầu bằng block dưới đây:

```
# <Tên file đầy đủ, dạng câu>

**Workstream**: NN_Workstream_Name
**File ID**: NN-MM (NN = workstream, MM = file order)
**Version**: v0.X
**Status**: DRAFT | IN_REVIEW | REVISION | APPROVED
**Owner**: <Họ tên>
**Reviewer**: <Họ tên>
**Last updated**: YYYY-MM-DD
**Classification**: Internal

> **Mục tiêu file**: 1–2 câu mô tả file trả lời câu hỏi gì.
>
> **Key questions**:
> - Câu hỏi 1 file này phải trả lời
> - Câu hỏi 2
> - Câu hỏi 3
>
> **Audience chính**: <ai sẽ đọc file này, từ bảng audience trong README §3>
```

---

## 3. Content classification tags (bắt buộc)

Mọi đoạn nội dung phải được gán **đúng 1** trong 4 tag dưới đây, đặt ở **đầu câu/đoạn**:

| Tag | Khi nào dùng | Yêu cầu kèm theo |
|---|---|---|
| `[FACT]` | Thông tin có thể kiểm chứng từ nguồn bên ngoài | **Bắt buộc citation** `[N]` |
| `[ANALYSIS]` | Diễn giải, suy luận của consultant dựa trên facts | Trỏ đến `[FACT]` đang phân tích bằng citation |
| `[ASSUMPTION]` | Giả định khi data không đủ nhưng cần phải kết luận | Ghi rõ **điều kiện để assumption đúng** + **impact nếu sai** |
| `[GAP]` | Thiếu data, cần collect thêm | Ghi rõ: data nào cần, từ nguồn nào, ai owner, due date |

**Ví dụ áp dụng**:

```markdown
[FACT] Databricks giới thiệu Mosaic AI Agent Framework tại Data + AI Summit 2024 [1].

[ANALYSIS] Việc tích hợp Unity Catalog vào agent governance giúp giảm time-to-compliance
so với stacks tự build, vì policy enforcement xảy ra tại data plane thay vì application
plane [1][2].

[ASSUMPTION] Giả định pricing serverless model serving không tăng quá 20% trong 12 tháng tới.
Nếu sai (tăng >50%) → TCO scenario base case sai lệch ~15%, cần re-run mô hình.

[GAP] Chưa có public case study về production deployment Mosaic AI Agent tại ngân hàng
Việt Nam. Cần khảo sát qua industry contacts. Owner: Lead. Due: 2026-06-15.
```

**Cấm**:
- Viết đoạn không có tag.
- Viết `[FACT]` không kèm citation.
- Trộn nhiều tag trong cùng 1 câu — tách thành nhiều câu.

---

## 4. Citation format

### 4.1 Inline citation
- Dùng số thứ tự `[N]` ngay sau câu/cụm từ chứa claim.
- Một câu có thể có nhiều citation: `[1][2][3]`.
- Mọi `[N]` phải có entry tương ứng trong section **References** ở cuối file **hoặc** trong `09_References_Appendix/01_primary_sources.md` (kèm dẫn chiếu).

### 4.2 Reference entry — IEEE rút gọn

Format chuẩn:
```
[N] Author/Org, "Title", Publication/Platform, YYYY-MM-DD, URL (accessed YYYY-MM-DD).
```

Ví dụ:
```
[1] Databricks, "Mosaic AI Agent Framework Overview", docs.databricks.com,
    2025-03-15, https://docs.databricks.com/aws/en/agents/ (accessed 2026-05-21).

[2] Gartner, "Magic Quadrant for Data Science and Machine Learning Platforms",
    Gartner Research, 2025-06-20, ID G00785432 (accessed 2026-05-21 via subscription).

[3] State Bank of Vietnam, "Quyết định 2345/QĐ-NHNN về an toàn giao dịch điện tử",
    sbv.gov.vn, 2023-12-18, https://sbv.gov.vn/... (accessed 2026-05-21).
```

### 4.3 Phân hạng nguồn (Source Tier)

| Tier | Loại nguồn | Khi nào dùng | Caveat |
|---|---|---|---|
| **T1** | Primary: vendor official docs, SEC filings, central bank documents, peer-reviewed papers, công bố chính thức | Ưu tiên cao nhất | Vendor docs vẫn có thiên kiến tích cực — cross-check với T2 |
| **T2** | Reputable analysts: Gartner, Forrester, IDC, McKinsey, BCG, Bain | Đo lường thị trường, vị thế cạnh tranh | Phương pháp luận đôi khi opaque; ghi rõ ngày báo cáo |
| **T3** | Tech media uy tín: WSJ, FT, Reuters, The Information, IEEE Spectrum, MIT Tech Review | Sự kiện, timeline, quote stakeholder | Bias biên tập có thể có |
| **T4** | Vendor blog, marketing, analyst sponsored content, conference keynote | Chỉ làm điểm khởi đầu | **Bắt buộc đánh dấu rõ "vendor view"** |
| **T5** | Forum, Reddit, anecdotal, social media | Chỉ làm hint khám phá | **Không** dùng làm evidence cho claim trọng yếu |

**Quy tắc**:
- Mỗi claim trọng yếu (ảnh hưởng recommendation) phải có **ít nhất 1 nguồn T1 hoặc T2**.
- Nếu chỉ có T4/T5 → claim đó **bắt buộc** tag `[ASSUMPTION]` hoặc `[GAP]`.
- Khi cite vendor docs cho competitor, ghi rõ là vendor official statement.

---

## 5. Trung lập & xung đột lợi ích

- **Cấm ngôn ngữ marketing**: "revolutionary", "best-in-class", "leading", "industry-first", "game-changer". Thay bằng **mô tả định lượng có nguồn** ("đạt X% accuracy theo benchmark Y [N]").
- Khi mô tả Databricks: cân bằng **strengths + weaknesses + open questions**.
- Khi so sánh đối thủ: dùng **cùng tiêu chí, cùng năm tham chiếu, cùng đơn vị đo**, cùng version (vendor product version).
- Khi vendor có công bố mâu thuẫn với T2/T3 → trình bày **cả hai phía** + `[ANALYSIS]` nêu khả năng xác minh.
- Disclosure: nếu file có nội dung dựa trên engagement có conflict of interest (analyst sponsored, vendor-funded research) → ghi rõ ở header `**Disclosure**:`.

---

## 6. Versioning

- File version theo SemVer rút gọn: `v<major>.<minor>`.
  - `v0.x` = DRAFT, chưa approved.
  - Lên `v1.0` khi APPROVED lần đầu.
  - Major bump (`v1.0` → `v2.0`) khi thay đổi **kết luận** hoặc **cấu trúc** file.
  - Minor bump (`v1.0` → `v1.1`) khi bổ sung data, sửa lỗi, refresh citation.
- Mỗi lần thay đổi nội dung → cập nhật `**Last updated**:` ở header + entry trong section **Change log** cuối file.

**Change log template ở cuối mỗi file**:
```markdown
## Change log

| Date | Version | Change | Author |
|---|---|---|---|
| YYYY-MM-DD | v0.1 | Khởi tạo dàn ý | <name> |
| YYYY-MM-DD | v0.2 | Bổ sung section 3, citation T1 cho 5 claims | <name> |
| YYYY-MM-DD | v1.0 | Reviewer approve, gắn status APPROVED | <name> |
```

---

## 7. Definition of Done (DoD)

### 7.1 Cho 1 file workstream

Trước khi đổi status từ `DRAFT` → `IN_REVIEW`:
- [ ] Header đầy đủ (§2): version, status, owner, reviewer, last updated, key questions, audience.
- [ ] `_overview.md` của workstream đã được approve **trước**.
- [ ] **Mọi đoạn** được tag `[FACT]` / `[ANALYSIS]` / `[ASSUMPTION]` / `[GAP]` (§3).
- [ ] **Mọi `[FACT]`** có ít nhất 1 citation, ưu tiên T1/T2 (§4).
- [ ] **Mọi claim trọng yếu** ảnh hưởng recommendation có ≥1 nguồn T1 hoặc T2.
- [ ] **Mọi `[GAP]`** có owner + due để fill.
- [ ] **Mọi `[ASSUMPTION]`** có condition + impact-if-wrong.
- [ ] Section **References** ở cuối file đầy đủ, format IEEE.
- [ ] Section **Change log** ở cuối file đã cập nhật.
- [ ] Đọc lại file: ngôn ngữ trung lập (§5), không marketing wording.
- [ ] Cross-reference (§8) đã check, không link gãy.
- [ ] `WORKSPACE_TRACKING.md` cập nhật status + last update.

Để đổi `IN_REVIEW` → `APPROVED`:
- [ ] Ít nhất 1 reviewer comment + owner đã xử lý hoặc accept/reject có lý do.
- [ ] Bump version (v0.x → v1.0 nếu approve lần đầu).
- [ ] `WORKSPACE_TRACKING.md` cập nhật %, last update.

### 7.2 Cho 1 workstream
- [ ] `_overview.md` APPROVED.
- [ ] Tất cả file con APPROVED.
- [ ] Cross-reference với các workstream liên quan đã check theo bảng dependency (`WORKSPACE_TRACKING.md §5`).
- [ ] Findings của workstream đã sync vào `01_Executive_Summary/`.

### 7.3 Cho toàn workspace
- Xem `README.md §8`.

---

## 8. Cross-reference

- Link đến file khác trong workspace: **relative path** từ workspace root.
  - Hợp lệ: `[link](../03_Databricks_Technology/01_mosaic_ai_agent_framework.md)`.
  - Hợp lệ trong cùng workstream: `[link](./02_unity_catalog_governance.md)`.
- Khi nhắc số liệu **đã có ở file khác**: **trích lại nguồn gốc**, không re-cite chéo, để tránh "citation drift".
- Khi cập nhật 1 file khiến số liệu thay đổi → check `WORKSPACE_TRACKING.md §5 dependencies` để sync các file phụ thuộc.

---

## 9. Bảng từ vựng & viết tắt

- Mọi viết tắt **dùng lần đầu** trong 1 file phải spell-out: "Retrieval-Augmented Generation (RAG)".
- **Thuật ngữ kỹ thuật**: giữ tiếng Anh (RAG, LLM, MLOps, Lakehouse, Unity Catalog, vector store, fine-tuning, embedding).
- **Thuật ngữ nghiệp vụ**: tiếng Việt + tiếng Anh trong ngoặc nếu có khả năng gây nhầm ("Hiểu khách hàng 360 (Customer 360)", "Phòng chống rửa tiền (AML)").
- Glossary tập trung tại `09_References_Appendix/03_glossary.md` — bổ sung mỗi khi gặp thuật ngữ mới.

---

## 10. Bảo mật & chia sẻ

- Classification mặc định: `Internal`.
- Đổi sang `Confidential` hoặc `Restricted` ở header nếu nội dung chứa:
  - Pricing vendor đang đàm phán
  - Danh sách khách hàng / partner
  - Roadmap sản phẩm chưa công bố
  - IP / patent đang chuẩn bị
  - Audit findings / regulator correspondence
- File `Confidential` / `Restricted` phải có dòng `**Access list**:` ở header.
- **Không** paste trực tiếp tài liệu bản quyền bên thứ ba; trích dẫn theo fair use + citation.
- **Không** commit PII (CIF, CMND/CCCD, số tài khoản, thông tin định danh khách hàng / nhân viên).
- Khi cần redacted: dùng `[REDACTED — <reason>]`.

---

## 11. Workflow

```
NOT_STARTED ──pick──> DRAFT ──self-check DoD §7.1──> IN_REVIEW
                                                          │
                                                     reviewer
                                                          │
                                          ┌──── REVISION ─┴─── APPROVED ──┐
                                          │                                │
                                          └────────── back to DRAFT        v
                                                                       (frozen until
                                                                        major change)
```

1. Pick workstream từ `WORKSPACE_TRACKING.md` (status `NOT_STARTED`, hoặc bạn là owner).
2. Update status → `DRAFT`, ghi tên owner.
3. Đọc `_overview.md` của workstream → confirm scope, key questions.
4. Viết file con theo dàn ý.
5. Self-check DoD §7.1 → đổi status `IN_REVIEW`, ping reviewer.
6. Reviewer comment trực tiếp trong file dạng block:
   ```
   > **REVIEWER COMMENT (<name>, YYYY-MM-DD)**: <nội dung comment>
   ```
   Owner xử lý → reply ngay dưới với `> **OWNER REPLY**: ...`.
7. Tất cả comment resolved → reviewer chuyển status `APPROVED`, bump version, owner cập nhật tracking.

---

## 12. Change log của RULES này

| Date | Version | Change | Author |
|---|---|---|---|
| 2026-05-21 | v0.1-draft | Khởi tạo RULES, chờ approve cùng workspace structure | Lead Consultant |
