# CLAUDE.md — Quy tắc tác giả cho Cost Model Research

[CẬP NHẬT: 21/05/2026]

> **Mục đích:** File này định nghĩa các quy tắc bắt buộc cho mọi thao tác tạo / sửa file trong thư mục `cost_model/` và các thư mục con. Claude Code tự động đọc CLAUDE.md khi được khởi tạo trong thư mục này. Mọi vi phạm = file không đạt review.

---

## §1. Quy tắc ngôn ngữ

| # | Quy tắc | Bắt buộc |
|---|---|:---:|
| 1.1 | **Tiếng Việt có dấu chuẩn 100%** — không viết không dấu, không pha tiếng Anh kiểu "lifecycle" trong văn bản thông thường | ✅ |
| 1.2 | Thuật ngữ kỹ thuật giữ nguyên tiếng Anh: `tokens`, `retry`, `RAG`, `embedding`, `chargeback`, `showback`, `multimodal`, `provisioned throughput` — **không dịch** | ✅ |
| 1.3 | Tên cột bảng, key JSON / YAML, identifier code giữ tiếng Anh | ✅ |
| 1.4 | Comment trong code block được phép tiếng Anh ngắn gọn | ⚪ |
| 1.5 | Số liệu, đơn vị tiền tệ ghi rõ: `$0.0019` không phải `0.0019` ; `15.00 USD/1M tokens` không phải `15` | ✅ |

### Citations

- Bộ Giáo dục & Đào tạo Việt Nam. *Quy tắc chính tả tiếng Việt.* (tham chiếu chung)
- Anthropic. *Claude Code — CLAUDE.md auto-loading.* docs.anthropic.com/en/docs/claude-code (truy cập 21/05/2026)

---

## §2. Quy tắc định dạng file

| # | Quy tắc | Áp dụng cho |
|---|---|---|
| 2.1 | **Mọi file nội dung là `.md`** (Markdown). Không dùng `.txt`, `.docx`, `.rst`, `.adoc` | Tất cả file nghiên cứu |
| 2.2 | File data có thể là `.yaml`, `.csv`, `.json` — **phải đi kèm file `.md` mô tả** | Chỉ data artifacts (vd. `rate_card.yaml`) |
| 2.3 | **Đầu mỗi file `.md` phải có dòng** `[CẬP NHẬT: DD/MM/YYYY]` ngay sau heading H1 | Tất cả file `.md` |
| 2.4 | Encoding UTF-8 (không BOM); line-ending LF (Unix) | Tất cả |
| 2.5 | Heading: H1 chỉ dùng 1 lần cho title; H2 (`##`) dùng cho section chính; H3 (`###`) cho sub-section | Tất cả |
| 2.6 | Code block đa ngôn ngữ có ngôn ngữ ngay sau ```` ``` ```` (vd. ```` ```python ````, ```` ```yaml ```` ) | Tất cả |

### Citations

- CommonMark Spec. *CommonMark 0.31.* commonmark.org/spec (truy cập 21/05/2026)
- GitHub Docs. *Basic writing and formatting syntax.* docs.github.com/en/get-started/writing-on-github (truy cập 21/05/2026)

---

## §3. Quy tắc citations (BẮT BUỘC NGHIÊM)

| # | Quy tắc |
|---|---|
| 3.1 | **Mọi section chứa số liệu / dữ liệu thị trường / tuyên bố kỹ thuật BẮT BUỘC có khối `### Citations` ở cuối section** |
| 3.2 | Format chuẩn: `- <Tên tổ chức / tác giả>. *<Title>.* <URL hoặc DOI> (truy cập DD/MM/YYYY)` |
| 3.3 | Trong văn bản, tham chiếu bằng `[N]` hoặc inline `(Tên, năm)` để không gián đoạn flow |
| 3.4 | Nếu sự kiện chưa có nguồn xác đáng → đánh dấu **`[cần verify]`** ngay tại điểm sử dụng và không tính là "đã có citation" |
| 3.5 | Nguồn dạng "tin đồn", LinkedIn post chưa verify, ChatGPT output → **không chấp nhận** |
| 3.6 | Citation trùng nhiều section: vẫn lặp lại ở từng section, không "tham chiếu lên trên" |
| 3.7 | File `.md` không có citations ở section có số liệu = **không đạt review** |

### Ví dụ ĐÚNG

```markdown
## §2. Cost trap phổ biến

Silent retries có thể nhân token usage lên 2–5× với agentic workflow chưa tối ưu [1].
Reflection loop không bounded có thể đẩy cost lên 10–100× cho corner case [2].

### Citations

- [1] Agents Arcade. *Cost Modeling for Agentic Systems Before You Go to Production.* agentsarcade.com/blog/cost-modeling-agentic-systems-production (truy cập 21/05/2026)
- [2] Galileo. *The Hidden Costs of Agentic AI.* galileo.ai/blog/hidden-cost-of-agentic-ai (truy cập 21/05/2026)
```

### Ví dụ SAI

```markdown
## §2. Cost trap phổ biến

Silent retries có thể nhân token usage lên 2–5×.   ← thiếu citation
Token cost khoảng 30–50% tổng cost.                 ← thiếu citation
```

### Citations

- FinOps Foundation. *Cost Estimation of AI Workloads.* finops.org/wg/cost-estimation-of-ai-workloads/ (truy cập 21/05/2026)
- ACM. *ACM Reference Format.* acm.org/publications/authors/reference-formatting (tham chiếu phong cách)

---

## §4. Quy ước tổ chức nội dung

| # | Quy ước |
|---|---|
| 4.1 | **Pyramid principle:** Kết luận / insight cốt lõi luôn ở đầu file dưới dạng block quote `>` |
| 4.2 | Đầu mỗi file top-level (4 file chính) có **⚡ 30-sec card** — gồm: 1 sơ đồ ASCII hoặc 1 bảng + 1 công thức cốt lõi + 1 bảng navigation |
| 4.3 | **Bảng > prose** — ưu tiên dùng bảng so sánh thay vì bullet list dài (≥ 4 mục) |
| 4.4 | Link nội bộ dùng `[[name]]` (Obsidian-style); `name` là filename không kèm `.md` (vd. `[[01_rate_card]]`) |
| 4.5 | Heading section dùng `## §N. <Tên>` để dễ tham chiếu chéo (vd. "xem [[02_cost_formula]] §3.1") |
| 4.6 | Đánh dấu chưa chắc: `**[cần verify]**` hoặc `**[ước lượng]**` |
| 4.7 | Số đo đơn vị tiền tệ: `$0.0019` ; số đo phần trăm: `30–50%` (dùng em-dash `–` cho range, không dùng `-`) |

### Citations

- Minto, Barbara. *The Pyramid Principle: Logic in Writing and Thinking.* (1987)
- Tufte, Edward. *The Visual Display of Quantitative Information.* (1983)

---

## §5. Quy ước version & changelog

| # | Quy ước |
|---|---|
| 5.1 | Dòng `[CẬP NHẬT: DD/MM/YYYY]` ở đầu file là **single source of truth** cho version của file đó |
| 5.2 | Mỗi sửa đổi đáng kể (đổi cấu trúc, cập nhật giá, thêm/xoá section) → cập nhật date stamp + ghi 1 dòng vào `PROGRESS.md` |
| 5.3 | Tracking diff chi tiết dùng git; **không** duy trì changelog inline ở mỗi file trừ khi file > 500 dòng |
| 5.4 | Khi file vượt v3 (cấu trúc đổi 3 lần) → cân nhắc tách thành nhiều file con |

### Citations

- Conventional Commits. *Conventional Commits 1.0.0.* conventionalcommits.org (truy cập 21/05/2026)

---

## §6. Quy ước peer review

| # | Quy ước |
|---|---|
| 6.1 | Mọi file mới hoặc edit lớn → ghi vào `PROGRESS.md` để track |
| 6.2 | File chưa qua review giữ marker `[draft]` trong dòng `[CẬP NHẬT: ... | draft]` |
| 6.3 | Reviewer phải check 4 điểm: (a) đủ citations theo §3; (b) ngôn ngữ chuẩn theo §1; (c) date stamp đúng theo §2.3; (d) link `[[name]]` hợp lệ |
| 6.4 | Audit auto theo `PLAYBOOK.md` §4 |

### Citations

- (không có claim cần citation trong section này)

---

## §7. Cấm

| # | Cấm |
|---|---|
| 7.1 | ❌ Không tạo file nội dung nghiên cứu ngoài `cost_model/` — giữ self-contained |
| 7.2 | ❌ Không xoá hard file đã review pass; chỉ archive vào `_archive/<YYYY-MM>/` |
| 7.3 | ❌ Không viết số liệu mà không nguồn (vd. "khoảng 30%" → phải có citation) |
| 7.4 | ❌ Không hardcode tỷ giá / số liệu ở nhiều file — gốc 1 chỗ ([[01_rate_card]]), file khác link đến |
| 7.5 | ❌ Không pha tiếng Anh trong văn bản thông thường ngoài thuật ngữ kỹ thuật (§1.2) |
| 7.6 | ❌ Không dùng emoji trừ khi đã có sẵn trong file gốc (vd. ⚡ trong "30-sec card") |
| 7.7 | ❌ Không sửa `CLAUDE.md` này mà không update `PROGRESS.md` Phase 2 |

### Citations

- (không có claim cần citation trong section này)

---

## §8. Workflow tham chiếu

Các quy tắc trên triển khai qua workflow ở `PLAYBOOK.md`:

- §1 Cập nhật date stamp → [[PLAYBOOK]] §1
- §3 Thêm citation → [[PLAYBOOK]] §5 (template)
- §6 Audit compliance → [[PLAYBOOK]] §4

---

## Liên kết

- Checklist phase + task: [[PROGRESS]]
- Commands & workflow: [[PLAYBOOK]]
- Master index: [[README]]
