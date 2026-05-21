# The Powerpoint-er — Tạo slide thương hiệu công ty

> Mô tả chủ đề → AI sinh outline → preview HTML → tải file `.pptx`. Pipeline 3 bước có chỉnh sửa nội dung và AI edit bằng ngôn ngữ tự nhiên.

---

## 1. Giới thiệu khả năng

**The Powerpoint-er làm gì?** Sinh nhanh một bộ slide thương hiệu công ty từ mô tả ngắn, không cần kỹ năng thiết kế.

**Khi nào dùng?**
- Bạn cần bản nháp slide trong 5–10 phút trước cuộc họp.
- Bạn muốn slide có nhận diện thương hiệu (màu đỏ, logo, footer) mà không phải tự dựng template.
- Bạn cần biến một outline có sẵn thành slide chỉn chu.

**Input**
- Chủ đề (topic).
- Đối tượng nghe (audience).
- Số slide mong muốn (5–15).
- Tuỳ chọn: bật/tắt khung template thương hiệu.

**Output**
- File `.pptx` 13.33" × 7.5" (16:9) tải về máy.
- Outline JSON xem trước trên web trước khi build.

**Cấu trúc slide hỗ trợ:**
- **Cover** — bìa với background thương hiệu.
- **Section** — slide ngăn cách kiểu divider.
- **Content** — 3 biến thể: `data` (KPI/số liệu), `concept` (nguyên lý), `process` (timeline/workflow).
- **Summary** — slide kết với takeaways.

---

## 2. Giới hạn & điều kiện sử dụng

| Mục | Giới hạn |
| --- | --- |
| Số slide | **5–15** mỗi bộ |
| Topic | Phải vượt qua input guardrail (Haiku phân loại) |
| Output guardrail | Áp dụng — nội dung không phù hợp sẽ bị chặn |
| **Khả năng edit sau khi tải** | **PPTX xuất ra là image-based — KHÔNG edit được text/shape trong PowerPoint** |
| Timeout build | 5 phút (toàn bộ bộ slide), 60 giây/slide khi regenerate |
| Đăng nhập | SSO bắt buộc |

> ⚠️ **Lưu ý quan trọng:** Mỗi slide được render thành ảnh PNG rồi nhúng vào PPTX. Khi mở trong PowerPoint, bạn chỉ thấy ảnh — không sửa được text trực tiếp. Mọi chỉnh sửa cần thực hiện **trên web trước khi tải**.

---

## 3. Cách sử dụng (Step-by-step)

Quy trình 3 bước: **Brief → Outline → Preview**.

### Bước 1 — Brief

- Nhập **topic**: ví dụ `"Chiến lược AI 2026"`.
- Nhập **audience**: ví dụ `"Hội đồng quản trị"`.
- Chọn **số slide**: 5–15.
- Bật/tắt **Use template** (thường để bật).

![Form Brief](./images/powerpointer-01-brief.png)
*Hình 1: Form nhập chủ đề, audience, số slide.*

### Bước 2 — Outline

- Bấm "Generate Outline" → AI sinh outline JSON: title + bullet cho từng slide.
- Bạn có thể:
  - Sửa tiêu đề/bullet trực tiếp.
  - Thêm / xoá / đổi thứ tự slide.
  - Đổi loại slide (cover, section, content, summary).
  - Đổi variant của content (data / concept / process).
  - Bấm "Preview slide này" để render thử từng slide.

![Outline editor](./images/powerpointer-02-outline.png)
*Hình 2: Outline có thể chỉnh trực tiếp trước khi build.*

### Bước 3 — Preview

- Bấm "Build" → tất cả slide được render song song thành HTML.
- Grid hiển thị tất cả slide ở dạng thumbnail.
- Hai lựa chọn edit cho mỗi slide:
  - **Rich Editor** (visual): in đậm, nghiêng, gạch dưới, đổi màu, đổi cỡ chữ, căn lề, bullet.
  - **AI Edit** (ngôn ngữ tự nhiên): gõ `"Đổi sang dạng timeline 4 mốc"` → AI viết lại slide.
- **Regenerate**: sinh lại slide với cùng outline nhưng version khác.
- **Presentation Mode**: xem fullscreen, phím mũi tên để chuyển slide.

![Preview grid + AI edit](./images/powerpointer-03-preview.png)
*Hình 3: Grid preview slide, modal edit gồm tab Rich Editor và AI Edit.*

### Bước 4 — Export

- Bấm "Export to PPTX".
- Frontend capture từng slide thành PNG, gói lại thành `.pptx`.
- Link/URL trong slide vẫn click được (overlay hyperlink).

![Export PPTX](./images/powerpointer-04-export.png)
*Hình 4: Nút export — file tải về máy ngay.*

---

## 4. Kịch bản minh hoạ (User Journeys)

### Kịch bản 1 — Bản nháp slide chiến lược 10 trang trong 10 phút

**Bối cảnh:** Sếp hỏi `"Có gì để present về AI strategy 2026 không?"` lúc 14h, họp lúc 15h.

**Bước thực hiện:**
1. Mở Powerpoint-er.
2. Brief:
   - Topic: `"Chiến lược AI 2026 — ứng dụng cho hoạt động ngân hàng"`.
   - Audience: `"Ban điều hành"`.
   - Số slide: `10`.
   - Use template: ✅
3. Bấm Generate Outline → ~20 giây.
   ![Outline cho strategy deck](./images/powerpointer-scn1-outline.png)
   *Hình: Outline 10 slide — cover, 2 section, 6 content (mix data/concept/process), 1 summary.*
4. Review outline, đổi slide 5 từ `concept` sang `data` (vì muốn KPI cụ thể) → bấm regenerate riêng slide 5.
5. Bấm Build → ~2 phút cho toàn bộ.
6. Preview, dùng AI Edit cho slide bìa: `"Đổi title thành 'AI 2026: From Pilot to Production'"`.
7. Export → tải `chien_luoc_AI_2026.pptx`.

---

### Kịch bản 2 — Sinh slide từ outline đã có

**Bối cảnh:** Bạn đã có outline 8 slide viết tay, muốn AI dựng visual nhanh.

**Bước thực hiện:**
1. Brief với topic chung chung (vì AI sẽ ghi đè bằng outline của bạn).
2. Generate Outline → sửa từng slide thành đúng nội dung của bạn:
   - Slide 1: cover — `"Q1 Performance Review"`
   - Slide 2: section — `"Highlights"`
   - Slide 3: content/data — `"Revenue grew 15% QoQ"` + bullet số liệu
   - ...
3. Build → preview.
4. AI Edit từng slide nếu chưa ưng — ví dụ: `"Thêm 1 cột so sánh với Q4 năm trước"`.
5. Export.

---

### Kịch bản 3 — Đổi style toàn bộ presentation

**Bối cảnh:** Bạn xây xong 12 slide nhưng cảm thấy quá nặng số liệu, muốn nhiều concept hơn.

**Bước thực hiện:**
1. Quay lại Outline → đổi variant slide 4, 6, 9 từ `data` sang `concept`.
2. Bấm regenerate cho 3 slide đó (không cần build lại toàn bộ).
3. Preview lại — nội dung được viết lại theo phong cách concept/prose.
4. Export.

---

## 5. Tips sử dụng

- **Brief càng chi tiết → outline càng tốt**: thêm context như `"company is a Vietnamese commercial bank, target audience knows AI basics"` ngay trong topic.
- **Variant matters**: 
  - `data` cho KPI, biểu đồ, so sánh con số.
  - `concept` cho nguyên lý, framework, văn bản dài.
  - `process` cho timeline, workflow, các bước.
- **AI Edit dùng câu lệnh ngắn**: `"Đổi sang dạng 4 quadrant"`, `"Thêm icon"`, `"Bỏ slide này"` — rõ ràng hơn câu dài phức tạp.
- **Edit toàn bộ trên web TRƯỚC khi export** — vì PPTX export không edit được.
- **Cần edit native trong PowerPoint?** Hiện tại không có cách — đây là hạn chế đã biết, team đang tìm giải pháp.
- **Regenerate vs Edit**: regenerate là sinh lại hoàn toàn (mất nội dung custom), edit là tinh chỉnh. Cẩn thận khi regenerate slide đã chỉnh tay.
- **Presentation Mode** trên web có thể dùng thay PowerPoint để demo ngay — không nhất thiết phải export.

---

## 6. Hạn chế đã biết

- **PPTX là image-based**: không edit được text/shape sau khi tải. Bù lại là toàn bộ editor trên web.
- **Số slide bị giới hạn 5–15**: bộ slide lớn hơn cần chia làm nhiều lần build (gộp thủ công).
- **Guardrail có thể chặn topic nhạy cảm**: nếu bị từ chối ở bước outline (HTTP 400), thử rephrase topic.
- **Web search disabled trong UI**: backend hỗ trợ `use_web_search` nhưng UI hiện không bật — slide dựa vào kiến thức nội tại của LLM, không có dữ liệu realtime.
- **Không bookmark template tự chỉnh**: tất cả slide dùng chung template thương hiệu cố định — chưa cho upload template riêng.
- **Slide bị fallback khi LLM lỗi**: nếu một slide gặp lỗi LLM, agent dùng `fallback_html()` (text + bullet đơn giản, không có visual) — cần regenerate slide đó.
- **Không hỗ trợ font/ngôn ngữ tuỳ chỉnh**: font dùng theo brand token đã định sẵn.
