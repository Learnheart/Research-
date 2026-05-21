# The Canvas Designer — Tạo hình ảnh & đồ hoạ từ mô tả

> Sinh visual (PNG/JPG), SVG, hoặc DOCX có thiết kế chỉ bằng một câu mô tả. Ảnh được render programmatic — có glow, gradient, particles theo 5 palette built-in.

---

## 1. Giới thiệu khả năng

**The Canvas Designer làm gì?** Biến mô tả text thành **hình ảnh hoặc tài liệu có thiết kế**:
- PNG / JPG: ảnh nghệ thuật pixel art có hiệu ứng glow / gradient.
- SVG: code SVG có thể nhúng vào web hoặc slide.
- DOCX: tài liệu Word có cấu trúc đẹp (heading, bullet, format).

**Khi nào dùng?**
- Cần visual minh hoạ cho slide / report mà không có designer.
- Cần infographic, icon, hoặc cover image cho tài liệu.
- Cần một file Word có cấu trúc chuẩn từ outline.

**Input**
- Mô tả văn bản (description).
- Output format: **PNG / JPG / SVG / DOCX**.
- Style: `creative` (mặc định) hoặc các style khác tuỳ phiên bản.
- Kích thước (mặc định: 1200×1200 cho PNG/JPG/SVG).

**Output**
- File tương ứng tải về (lưu trên Volume).
- Preview hiển thị ngay trên web trước khi tải.

---

## 2. Giới hạn & điều kiện sử dụng

| Mục | Giới hạn |
| --- | --- |
| Định dạng output | PNG, JPG, SVG, DOCX |
| Kích thước mặc định | 1200×1200 (tuỳ chỉnh được) |
| LLM model | Opus cho PNG/JPG (chất lượng cao hơn), Sonnet cho SVG/DOCX |
| Lưu trữ | Lưu theo `/canvas/{user}/{date}/canvas-{id}.{ext}` |
| Tìm file cũ | Download chỉ tìm trong **ngày hôm nay** — file ngày khác có thể không tìm thấy |
| Đăng nhập | SSO bắt buộc |

---

## 3. Cách sử dụng (Step-by-step)

### Bước 1 — Mở The Canvas Designer
![Giao diện](./images/canvas-01-home.png)
*Hình 1: Panel mô tả bên trái, preview kết quả bên phải.*

### Bước 2 — Mô tả hình ảnh
Ví dụ: `"A futuristic robot with glowing neon eyes, cyberpunk style"`, `"Một biểu đồ tăng trưởng cách điệu hình rocket"`, `"Cover image cho deck về AI agent — phong cách tech, nhiều ánh sáng xanh"`.

![Nhập mô tả](./images/canvas-02-prompt.png)
*Hình 2: Ô mô tả + chọn format / kích thước.*

### Bước 3 — Chọn định dạng output
- **PNG** (mặc định): ảnh chất lượng cao, hỗ trợ transparency.
- **JPG**: ảnh nén nhẹ, không transparency.
- **SVG**: vector, scale không vỡ — phù hợp icon/logo.
- **DOCX**: tài liệu Word (mô tả khi đó nên là cấu trúc tài liệu, không phải hình).

### Bước 4 — (Tuỳ chọn) chỉnh kích thước
Mặc định 1200×1200. Tăng kích thước cho banner / poster.

### Bước 5 — Bấm "Generate"
- Với PNG/JPG: Opus sinh JSON spec → Pillow render. Mất ~15–30 giây.
- Với SVG: LLM trả raw SVG code (nhanh hơn).
- Với DOCX: LLM trả JSON spec → python-docx render.

![Loading state](./images/canvas-03-generating.png)
*Hình 3: Spinner trong khi generate.*

### Bước 6 — Preview kết quả
Hình hiển thị ở panel phải.

![Preview kết quả](./images/canvas-04-preview.png)
*Hình 4: Ảnh sinh ra hiển thị trực tiếp trên web.*

### Bước 7 — Tải về hoặc Regenerate
- Bấm **Download** → lưu file về máy.
- Sửa mô tả → bấm Generate lại nếu chưa ưng.

---

## 4. Kịch bản minh hoạ (User Journeys)

### Kịch bản 1 — Tạo cover image cho slide

**Bối cảnh:** Bạn cần ảnh cover cho deck `"AI Agent Platform 2026"`.

**Bước thực hiện:**
1. Mở Canvas Designer.
2. Mô tả: `"Cover image — abstract AI agents connected as a network, glowing blue nodes on dark background, cyberpunk aesthetic"`.
3. Chọn format: **PNG**, kích thước 1920×1080 (16:9).
4. Generate → preview.
5. Chưa ưng → bổ sung: `"... add subtle red accent lines, more depth"` → regenerate.
6. Download → chèn vào slide cover trong PowerPoint.

![Cover image kết quả](./images/canvas-scn1-cover.png)
*Hình: Cover image kiểu network nodes với glow.*

---

### Kịch bản 2 — Sinh icon SVG cho web app

**Bối cảnh:** Bạn cần một icon `"shield with checkmark"` cho dashboard security.

**Bước thực hiện:**
1. Mô tả: `"Minimalist shield icon with a green checkmark, flat design, clean lines"`.
2. Format: **SVG**, kích thước 256×256.
3. Generate — LLM trả raw SVG code, web hiển thị render trước.
4. Download `.svg` → nhúng vào codebase.

**Cảnh báo:** SVG do LLM viết có thể không tối ưu (path phức tạp / chưa minify). Chạy qua SVGO nếu cần production-grade.

---

### Kịch bản 3 — Sinh tài liệu DOCX có cấu trúc

**Bối cảnh:** Bạn cần một bản template `"Project Charter"` với cấu trúc chuẩn.

**Bước thực hiện:**
1. Format: **DOCX**.
2. Mô tả: `"Project Charter document with sections: Project Name, Objectives, Scope (in/out), Stakeholders table, Milestones timeline, Risks & Mitigations, Approval signatures"`.
3. Generate → LLM trả JSON spec → python-docx render.
4. Download `.docx` → mở Word, điền nội dung thực tế.

---

## 5. Tips sử dụng

- **Mô tả càng chi tiết, kết quả càng đúng ý**: nêu rõ style (`"flat design"`, `"cyberpunk"`, `"corporate minimalist"`), màu sắc (`"red and navy"`), bố cục (`"center composition"`).
- **Tham khảo 5 palette built-in** để chọn tone:
  - `synthetic_optimism` (cyan/pink/dark blue) — futuristic
  - `midnight_aurora` (blue/purple/navy) — premium
  - `forest_tech` (green/dark forest) — sustainability
  - `cosmic_dream` (pink/blue/black) — creative
  - `neon_noir` (neon green/magenta/black) — bold
- **PNG > JPG** cho graphics có nền trong suốt; **JPG** chỉ khi cần dung lượng nhỏ.
- **SVG** không phù hợp ảnh thực — chỉ tốt cho icon, logo, đồ hoạ hình học.
- **DOCX** không phải `"viết bài cho tôi"` — chỉ tạo cấu trúc/format. Nội dung chi tiết bạn tự điền, hoặc dùng Powerpoint-er/Summarizer cho text.
- **Tải về ngay** sau khi sinh — search file cũ chỉ tìm trong ngày hôm nay.
- **Regenerate là sinh lại hoàn toàn** — không phải tinh chỉnh. Cần style khác nhẹ, sửa mô tả; cần style hoàn toàn mới, regenerate.

---

## 6. Hạn chế đã biết

- **Không phải image generation kiểu DALL-E / Midjourney**: ảnh được render programmatic bằng Pillow theo spec JSON — phù hợp đồ hoạ trừu tượng, infographic, network diagram, KPI visual; **không** sinh được ảnh thực tế chân dung, phong cảnh, vật thể chi tiết.
- **Limited palettes**: 5 palette built-in — chưa tuỳ biến màu hoàn toàn từ user prompt.
- **SVG do LLM viết** có thể không tối ưu — không production-ready nếu không qua SVGO.
- **Download chỉ tìm file của ngày hôm nay** — file ngày trước có thể không truy lại được qua giao diện.
- **Không có version history**: regenerate ghi đè preview — chưa lưu các phiên bản trước.
- **Không có batch / mass generation**: mỗi lần 1 file.
- **DOCX không có header/footer / page number tuỳ biến**: chỉ sinh body content theo spec.
- **Không có brand template DOCX chính thức**: cần copy-paste vào template công ty sau khi tải.
