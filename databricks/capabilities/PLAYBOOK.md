# PLAYBOOK — Databricks Capabilities Workspace

> Cách update / mở rộng báo cáo trong folder `databricks/capabilities/`.

## 1. Cấu trúc workspace

```
databricks/capabilities/
├── README.md                    # Tóm tắt tổng quan + 3 câu hỏi mục tiêu
├── PROGRESS.md                  # Checklist phase
├── PLAYBOOK.md                  # File này
├── 01-authoring.md              # Authoring layer
├── 02-tools-retrieval.md        # Tools & retrieval
├── 03-evaluation.md             # Evaluation
├── 04-serving-deployment.md     # Serving & deploy
├── 05-monitoring-observability.md
├── 06-governance-security.md
└── 07-when-to-choose.md         # Decision criteria
```

## 2. Quy tắc nội dung

- **Single source of truth**: chỉ trích từ `docs.databricks.com/aws/en/agents/` và sub-pages liên kết trực tiếp.
- **Mỗi capability cần**:
  1. Tên (giữ thuật ngữ English: Agent Framework, Vector Search, Unity Catalog, …)
  2. Vấn đề trong agent lifecycle mà nó giải quyết
  3. Bullet points những gì docs nói (không thêm code)
  4. Link `docs.databricks.com/...` tương ứng
  5. Mục **"Adapt vào sản phẩm"** — gợi ý use case doanh nghiệp
- Nếu docs không nói → ghi "không tìm thấy trong docs".
- Vietnamese narrative, giữ English term khi cần (Unity Catalog, Mosaic AI, MLflow…).

## 3. Workflow update

### 3.1 Khi Databricks ra capability mới

```powershell
# 1. Fetch trang mới (qua WebFetch tool trong session Claude)
# 2. Xác định nhóm capability phù hợp (01-07)
# 3. Cập nhật file capability tương ứng:
#    - Add bullet mới dưới đúng heading
#    - Add link docs
#    - Add "Adapt vào sản phẩm" nếu có insight mới
# 4. Update README.md nếu thay đổi câu trả lời cho 3 câu hỏi mục tiêu
# 5. Tick checklist trong PROGRESS.md
```

### 3.2 Khi cần re-verify nguồn

```powershell
# Mở file capability bất kỳ → grep link docs.databricks.com
# Dùng WebFetch để xác nhận docs vẫn còn / nội dung khớp
# Nếu docs đã thay đổi → update bullet + ghi chú date check
```

### 3.3 Khi cần thêm nhóm capability mới

```powershell
# Tạo file 08-<group>.md theo template
# Thêm dòng vào README.md mục "Cấu trúc báo cáo"
# Thêm checklist trong PROGRESS.md Phase 2
```

## 4. Convention

- File names: `NN-kebab-case.md` (NN = 01..09)
- Headings: H1 cho tiêu đề file, H2 cho từng capability con, H3 cho sub-section
- Mỗi link docs đặt inline `[Authoring](https://docs.databricks.com/...)` ngay sau câu mô tả
- Adaptation section luôn đặt ở cuối mỗi capability, prefix `**Adapt vào sản phẩm:**`

## 5. Ranh giới (theo yêu cầu chủ project)

- READ/WRITE chỉ trong `databricks/capabilities/`
- KHÔNG sửa file ngoài folder này
- KHÔNG đưa code samples, API signatures, step-by-step tutorials
- KHÔNG bổ sung kiến thức ngoài docs

## 6. Health check trước khi commit phiên bản mới

- [ ] Mọi capability có ít nhất 1 link docs
- [ ] Mọi adaptation suggestion bám sát capability docs đề cập (không bịa)
- [ ] README.md trả lời được 3 câu hỏi mục tiêu
- [ ] PROGRESS.md đã tick các task hoàn thành
