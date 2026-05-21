# CLAUDE.md — Quy tắc dự án R&D: Analyze Databricks

[CẬP NHẬT: 21/05/2026]

> **Phạm vi áp dụng**: Toàn bộ subtree `C:\Projects\Databricks\databricks\` — gồm `docs/` (research workspace) và `prompts/` (prompt seeds).
> File này được Claude Code tự động nạp khi làm việc trong subtree này. **Mọi assistant đóng góp nội dung phải tuân thủ.**
>
> **Quan hệ với các file rule khác**:
> - `docs/RULES.md` — convention chi tiết cho **research deliverable** (header template, tag `[FACT]/[ANALYSIS]/[ASSUMPTION]/[GAP]`, source tier T1–T5, DoD). File CLAUDE.md này là **lớp baseline** ở cấp project; `docs/RULES.md` mở rộng thêm cho file research.
> - `PROGRESS.md` — checklist phase + task của dự án.
> - `PLAYBOOK.md` — lệnh và workflow để cập nhật dự án.

---

## 1. Bốn quy tắc cốt lõi (bắt buộc)

### 1.1 Ngôn ngữ — tiếng Việt có dấu chuẩn 100%

- Mọi file `.md` được tạo trong dự án này phải dùng **tiếng Việt có dấu đầy đủ** (không dùng telex, vni, hoặc tiếng Việt không dấu).
- **Thuật ngữ kỹ thuật** giữ nguyên tiếng Anh khi không có bản dịch chuẩn (ví dụ: RAG, LLM, MLOps, Unity Catalog, Mosaic AI, Vector Search, embedding, fine-tuning, prompt injection).
- **Thuật ngữ nghiệp vụ** dùng tiếng Việt + tiếng Anh trong ngoặc khi dễ nhầm (ví dụ: "Hiểu khách hàng 360 (Customer 360)", "Phòng chống rửa tiền (AML)", "Quản trị rủi ro mô hình (Model Risk Management)").
- Code block, file path, command, URL giữ nguyên tiếng Anh (không Việt hoá).

### 1.2 Định dạng file — bắt buộc `.md` (Markdown)

- Mọi file **nội dung** (analysis, overview, report, tracking, playbook) phải là `.md`.
- Markdown flavor: CommonMark + Github-flavored (bảng, checkbox, code fence, link).
- File khác `.md` (ví dụ `.xlsx`, `.pptx`, `.png` diagram) **chỉ dùng** cho:
  - Dữ liệu định lượng nguồn gốc dạng số (Excel cho TCO/ROI model).
  - Đầu ra trình bày cuối cho C-Level (slide deck).
  - Hình ảnh / diagram đính kèm.
- Mọi file `.xlsx`/`.pptx`/`.png` **phải có** file `.md` companion mô tả nội dung và link tham chiếu.

### 1.3 Citation — bắt buộc ở cuối mỗi section

- Mọi **thông tin kỹ thuật, số liệu, dữ liệu thị trường, quy định pháp luật** trong file `.md` bắt buộc có nguồn dẫn chiếu.
- Cuối **mỗi section** (`## H2`) chứa thông tin loại này phải có block `### Citations` (hoặc `### Nguồn dẫn chiếu`) liệt kê đầy đủ.
- **Format citation chuẩn** (IEEE rút gọn):

  ```
  [N] <Author/Org>, "<Title>", <Publication/Platform>, <YYYY-MM-DD or "n.d.">,
      <Full URL> (accessed <YYYY-MM-DD>).
  ```

- **Ví dụ áp dụng** ở cuối 1 section nói về Mosaic AI:

  ```markdown
  ### Citations

  [1] Databricks, "Mosaic AI Agent Framework Overview", docs.databricks.com,
      n.d., https://docs.databricks.com/aws/en/agents/ (accessed 2026-05-21).
  [2] Gartner, "Magic Quadrant for Data Science and Machine Learning Platforms",
      gartner.com, 2025-06-20, ID G00785432 (accessed 2026-05-21 via subscription).
  ```

- **Inline citation**: dùng `[N]` ngay sau câu/cụm chứa claim, ví dụ: "Mosaic AI Agent Framework GA Q4-2024 [1]".
- File research thuộc `docs/` còn phải tuân thủ thêm **phân hạng nguồn T1–T5** ở `docs/RULES.md §4.3`.
- **Cấm** số liệu, fact, regulation, market data không có citation. Nếu chưa có nguồn → ghi `[CẦN NGUỒN — owner: <ai>, due: <ngày>]`.

### 1.4 Versioning — tag `[CẬP NHẬT: DD/MM/YYYY]` ở đầu file

- Dòng thứ 2 hoặc 3 của **mọi** file `.md` phải có tag:

  ```markdown
  [CẬP NHẬT: DD/MM/YYYY]
  ```

  (ngay dưới `# H1` tiêu đề file)

- Định dạng ngày: `DD/MM/YYYY` (ví dụ `21/05/2026`), **không** dùng `YYYY-MM-DD` ở dòng tag này (vì user quy định).
- Mỗi lần sửa nội dung nghiệp vụ → cập nhật ngày trong tag + bổ sung 1 dòng trong section `## Change log` ở cuối file.
- File trong `docs/` còn có header thêm theo template `docs/RULES.md §2` (workstream, file ID, version v0.x, status, owner, reviewer) — **không thay thế** mà **đứng cạnh** tag `[CẬP NHẬT]` này.

---

## 2. Tổ chức thư mục

```
databricks/
├── CLAUDE.md           ← bạn đang ở đây — rules project-level
├── PROGRESS.md         ← checklist phase + task
├── PLAYBOOK.md         ← lệnh và workflow
├── docs/               ← research workspace (Big4 standard)
│   ├── README.md
│   ├── RULES.md        ← convention chi tiết cho file research
│   ├── WORKSPACE_TRACKING.md
│   ├── 01_Executive_Summary/
│   ├── 02_Market_Landscape/
│   ├── 03_Databricks_Technology/
│   ├── 04_Competitive_Analysis/
│   ├── 05_Banking_FSI_Applications/
│   ├── 06_Strategic_Roadmap/
│   ├── 07_Business_Case_And_Risk/
│   └── 08_References_Appendix/
└── prompts/            ← prompt seed lưu trữ
    └── first_insight.md
```

**Quy tắc**: Không tạo file `.md` ngoài 3 location trên (`databricks/`, `databricks/docs/<workstream>/`, `databricks/prompts/`). Nếu cần location mới → cập nhật CLAUDE.md trước.

---

## 3. Workflow tóm tắt

1. Trước khi bắt đầu task mới → đọc `PROGRESS.md` để xác định phase + task hợp lý kế tiếp.
2. Đọc `PLAYBOOK.md` để dùng đúng lệnh và workflow chuẩn.
3. Khi tạo / sửa file `.md` trong `docs/<workstream>/`: tuân thủ thêm `docs/RULES.md` (header template + tag content + source tier + DoD).
4. Sau khi hoàn thành 1 task → cập nhật **3 chỗ**: tag `[CẬP NHẬT]` ở đầu file vừa sửa + checkbox tương ứng trong `PROGRESS.md` + (nếu là file research) bảng tiến độ trong `docs/WORKSPACE_TRACKING.md`.

---

## 4. Cấm và lưu ý

- **Cấm** tạo file `.md` mới không có tag `[CẬP NHẬT: DD/MM/YYYY]`.
- **Cấm** đoạn nội dung số liệu / quy định / thị trường không có citation.
- **Cấm** ngôn ngữ marketing ("revolutionary", "best-in-class", "leading", "game-changer") — bám rule trung lập ở `docs/RULES.md §5`.
- **Lưu ý**: User là Lead Consultant chuẩn Big4 cho audience C-Level (CIO/CFO/CRO). Output phải đủ rigor để stress-test.

---

## 5. Change log của file CLAUDE.md này

| Ngày | Phiên bản | Thay đổi | Tác giả |
|---|---|---|---|
| 21/05/2026 | v0.1 | Khởi tạo CLAUDE.md project-level — định nghĩa 4 quy tắc cốt lõi (ngôn ngữ, format, citation, versioning) | Lead Consultant |

### Citations

Không có citation — file này là rule nội bộ, không chứa thông tin kỹ thuật/thị trường/quy định.
