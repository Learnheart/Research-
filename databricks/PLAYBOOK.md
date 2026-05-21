# PLAYBOOK — Lệnh và workflow chuẩn

[CẬP NHẬT: 21/05/2026]

> File này tập hợp **lệnh shell** và **workflow chuẩn** để mọi assistant đóng góp dùng đúng cách cập nhật dự án.
> Mọi lệnh trong file này được test trên Windows PowerShell 5.1 (xem `CLAUDE.md`).
> Khi thêm command mới → bắt buộc test trước + thêm 1 dòng vào `## Change log` cuối file.

---

## 1. Quy ước nhanh

| Mục | Quy ước |
|---|---|
| Working directory | `C:\Projects\Databricks\databricks\` |
| Encoding khi tạo file `.md` | UTF-8 không BOM |
| Line ending | LF (Unix-style) — Markdown render tốt cross-platform |
| Date format trong tag `[CẬP NHẬT]` | `DD/MM/YYYY` (ví dụ `21/05/2026`) |
| Date format trong header research file | `YYYY-MM-DD` (ví dụ `2026-05-21` — theo `docs/RULES.md §2`) |
| Citation format | IEEE rút gọn (xem `CLAUDE.md §1.3`) |

---

## 2. Workflow A — Bắt đầu 1 file research mới

> Áp dụng cho file `.md` mới trong `docs/<workstream>/`.

**Bước 1**: Đọc 2 file rule liên quan để nắm convention:

```powershell
Get-Content -Path "docs\RULES.md" -TotalCount 100
Get-Content -Path "docs\<workstream>\_overview.md" -TotalCount 60
```

Trong workflow Claude Code, dùng tool `Read` (không Bash) — ví dụ:

```
Read docs/RULES.md           (xem §2 header template + §3 tag + §4 citation)
Read docs/<workstream>/_overview.md   (xem dàn ý + scope + DoD)
```

**Bước 2**: Tạo file theo template (xem `## 6 — Templates` ở cuối file này).

**Bước 3**: Sau khi viết xong section đầu tiên → tự self-check 4 rule cốt lõi:

- [ ] Tag `[CẬP NHẬT: DD/MM/YYYY]` ở đầu file
- [ ] Header research đầy đủ (workstream, file ID, version, status, owner, reviewer, key questions, audience)
- [ ] Mọi đoạn nội dung có tag `[FACT]/[ANALYSIS]/[ASSUMPTION]/[GAP]`
- [ ] Mỗi `[FACT]` có inline citation `[N]` + section `### Citations` ở cuối

**Bước 4**: Cập nhật **3 chỗ**:

1. Tăng tag `[CẬP NHẬT]` ngày hiện tại.
2. Tick checkbox tương ứng trong `PROGRESS.md` (dùng workflow B).
3. Cập nhật bảng `docs/WORKSPACE_TRACKING.md §2.2` (status DRAFT/IN_REVIEW + last update).

---

## 3. Workflow B — Cập nhật `PROGRESS.md` (tick checkbox)

Dùng tool `Edit`, **không** Bash sed. Ví dụ tick task "tạo file `03/01_mosaic_ai_agent_framework.md`":

```
Edit PROGRESS.md
  old_string: "- [ ] `docs/03_Databricks_Technology/01_mosaic_ai_agent_framework.md`"
  new_string: "- [~] `docs/03_Databricks_Technology/01_mosaic_ai_agent_framework.md` (IN_REVIEW 22/05/2026)"
```

**Quy ước trạng thái**:
- `[ ]` → `[~]` khi đổi sang IN_REVIEW (chờ reviewer ký)
- `[~]` → `[x]` khi APPROVED
- `[ ]` → `[!]` khi BLOCKED (kèm ghi chú lý do + owner + due trong cùng dòng)

Sau khi tick → cập nhật `[CẬP NHẬT: DD/MM/YYYY]` ở đầu `PROGRESS.md` thành ngày hôm nay.

---

## 4. Workflow C — Cập nhật `docs/WORKSPACE_TRACKING.md`

3 bảng cần cập nhật tuỳ trường hợp:

| Trường hợp | Bảng cập nhật | Field |
|---|---|---|
| File con đổi status | `§2.2` | Status, Owner, Last update, Notes |
| Workstream đạt 100% | `§1` | % + Last update của workstream |
| Có open question mới | `§3` | Thêm dòng Q-NN |
| Có risk mới | `§4` | Thêm dòng R-NN + severity |
| Đổi `Workspace version` | Header file + `§7 Change log` | Bump version (v0.x → v0.x+1 hoặc v1.0) |

**Lưu ý**: `docs/WORKSPACE_TRACKING.md` dùng `**Last updated**: YYYY-MM-DD` ở header (không phải DD/MM/YYYY) — đây là convention `docs/RULES.md`, không xung đột với CLAUDE.md.

---

## 5. Workflow D — Thêm citation mới

**Bước 1**: Thêm inline citation `[N]` ngay sau câu chứa claim:

```markdown
[FACT] Mosaic AI Agent Framework đạt General Availability tại Data + AI Summit Q2-2024 [1].
```

**Bước 2**: Thêm entry vào section `### Citations` của section đó:

```markdown
### Citations

[1] Databricks, "Mosaic AI Agent Framework Overview", docs.databricks.com,
    n.d., https://docs.databricks.com/aws/en/agents/ (accessed 21/05/2026).
```

**Bước 3 (research file)**: Nếu source là T1 (primary) hoặc T2 (reputable analyst) → cũng phải có entry trong `docs/08_References_Appendix/01_primary_sources.md` hoặc `02_secondary_sources.md` (master index, deduplicated).

**Bước 4**: Nếu thuật ngữ mới xuất hiện → bổ sung vào `docs/08_References_Appendix/03_glossary.md`.

---

## 6. Templates

### 6.1 Template file research mới (đặt trong `docs/<workstream>/`)

```markdown
# <Tên file đầy đủ dạng câu>

[CẬP NHẬT: DD/MM/YYYY]

**Workstream**: NN_Workstream_Name
**File ID**: NN-MM
**Version**: v0.1
**Status**: DRAFT
**Owner**: <Họ tên>
**Reviewer**: <Họ tên>
**Last updated**: YYYY-MM-DD
**Classification**: Internal

> **Mục tiêu file**: 1–2 câu trả lời câu hỏi gì.
>
> **Key questions**:
> - Câu hỏi 1
> - Câu hỏi 2
>
> **Audience chính**: <ai sẽ đọc>

---

## 1. <Section title>

[FACT] <Câu nội dung có claim> [1].

[ANALYSIS] <Diễn giải dựa trên fact trên> [1].

[ASSUMPTION] <Giả định>. Điều kiện đúng: <...>. Impact nếu sai: <...>.

[GAP] <Cái còn thiếu>. Owner: <ai>. Due: DD/MM/YYYY.

### Citations

[1] <Author/Org>, "<Title>", <Publication/Platform>, <YYYY-MM-DD or n.d.>,
    <Full URL> (accessed DD/MM/YYYY).

## 2. <Section tiếp>

...

### Citations

[2] ...

---

## Change log

| Ngày | Phiên bản | Thay đổi | Tác giả |
|---|---|---|---|
| DD/MM/YYYY | v0.1 | Khởi tạo file | <name> |
```

### 6.2 Template overview file `_overview.md` (đã có sẵn 8 file mẫu)

Tham khảo bất kỳ file `_overview.md` nào trong `docs/0X_*/` đã tạo ở v0.3-draft. Cấu trúc chuẩn:

1. Header per `docs/RULES.md §2`
2. Mục tiêu workstream + Key questions + Audience (block `> ... `)
3. `## 1. Scope` (in/out)
4. `## 2. Strategic context`
5. `## 3. Dàn ý file con` (bảng)
6. `## 4. Key questions phải trả lời (chi tiết)`
7. `## 5. Cross-references (dependencies)`
8. `## 6. Sources & evidence plan`
9. `## 7. Open questions từ tracking (subset)`
10. `## 8. Risks & GAPs`
11. `## 9. Definition of Done — workstream level`
12. `## 10. Change log`

---

## 7. Common commands

| Mục đích | Lệnh PowerShell |
|---|---|
| Liệt kê tất cả `.md` trong workspace | `Get-ChildItem -Path . -Recurse -Filter "*.md" \| Select-Object FullName, Length, LastWriteTime` |
| Đếm tổng số file `.md` | `(Get-ChildItem -Path . -Recurse -Filter "*.md").Count` |
| Tìm file chưa có tag `[CẬP NHẬT]` | `Get-ChildItem -Recurse -Filter "*.md" \| Where-Object { -not (Select-String -Path $_.FullName -Pattern "\[CẬP NHẬT:" -Quiet) }` |
| Tìm file chứa `[FACT]` không kèm citation `[N]` | (cần script — đề xuất thêm sau khi research bắt đầu) |
| Tạo folder workstream con (nếu có) | `New-Item -ItemType Directory -Path "docs\<workstream>\<sub>" -Force` (lưu ý: `docs/RULES.md §1.1` cấm subfolder trong workstream — chỉ dùng cho exception) |

> Trong Claude Code: ưu tiên `Glob` và `Grep` thay cho lệnh PowerShell tương ứng (xem rule "Using your tools" của Claude Code).

---

## 8. Pre-flight check trước khi submit PR / handoff

- [ ] Tag `[CẬP NHẬT]` ngày hôm nay ở đầu mọi file đã sửa.
- [ ] `PROGRESS.md` đã tick checkbox cho task hoàn thành.
- [ ] `docs/WORKSPACE_TRACKING.md` đã cập nhật bảng `§2.2` (status, last update).
- [ ] Mọi `[FACT]` trong file mới có citation `[N]` + section `### Citations`.
- [ ] Không file `.md` nào còn `[CẦN NGUỒN — ...]` không có owner + due.
- [ ] Spell-check tiếng Việt (đặc biệt từ có dấu hỏi/ngã/nặng — accent thường sai khi paste).
- [ ] Cross-reference link (`../03_Databricks_Technology/...`) đã verify không gãy.

---

## 9. Khi gặp xung đột rule

Thứ tự ưu tiên khi rule mâu thuẫn:

1. `CLAUDE.md` (project-level) — baseline 4 rule cốt lõi.
2. `docs/RULES.md` (research-deliverable level) — convention chi tiết Big4.
3. Hướng dẫn ad-hoc trong prompt user — có thể override 1+2 nếu user nêu rõ.

Nếu sau khi đối chiếu vẫn không rõ → **hỏi user 1 câu xác nhận** trước khi viết, theo đúng tinh thần `docs/RULES.md §9` (giải quyết ambiguity sớm).

---

## Change log

| Ngày | Phiên bản | Thay đổi | Tác giả |
|---|---|---|---|
| 21/05/2026 | v0.1 | Khởi tạo `PLAYBOOK.md` — 4 workflow chính (A: new research file, B: tick `PROGRESS.md`, C: cập nhật tracking, D: thêm citation) + 2 template + pre-flight check | Lead Consultant |

### Citations

Không có citation — file này là playbook nội bộ, không chứa thông tin kỹ thuật/thị trường/quy định.
