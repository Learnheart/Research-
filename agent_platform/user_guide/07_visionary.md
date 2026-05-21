# The AI Visionary — Hỏi đáp về chiến lược AI nội bộ

> Cửa hỏi đáp về định hướng AI của tổ chức (dựa trên Knowledge Agent chuyên dụng) + tạo nhanh báo cáo cập nhật dạng PPTX.

---

## 1. Giới thiệu khả năng

**The AI Visionary làm gì?** Trả lời câu hỏi về **chiến lược / tầm nhìn AI** của tổ chức bằng cách proxy tới một **Knowledge Agent** chuyên dụng (đã được nuôi trên tài liệu chiến lược nội bộ), đồng thời cho phép tạo slide báo cáo tóm tắt.

**Khi nào dùng?**
- Bạn cần tra cứu nhanh "AI roadmap 2026 của công ty là gì?".
- Bạn muốn refresh kiến thức về định hướng AI trước khi present cho khách hàng / cấp trên.
- Bạn cần một bản update deck nhanh về AI strategy gửi cho team.

**Input**
- **Chat mode**: câu hỏi text (có thể nối tiếp nhiều lượt).
- **Report mode**: prompt mô tả deck cần tạo.

**Output**
- **Chat**: phản hồi text dựa trên tài liệu nội bộ.
- **Report**: file `.pptx` đơn giản (cover ALL CAPS + content slide với bullet).

---

## 2. Giới hạn & điều kiện sử dụng

| Mục | Giới hạn |
| --- | --- |
| Số message mỗi session | Tối đa **50** |
| Độ dài mỗi message | Tối đa **30.000 ký tự** |
| Tài liệu nguồn | **Cố định** — do team owner Knowledge Agent quản lý, user không upload thêm được |
| Đăng nhập | SSO bắt buộc |
| Style report deck | Đơn giản (cover + content) — **đơn giản hơn Powerpoint-er** |

> ℹ️ Visionary **không tự thu nạp tài liệu của bạn**. Nếu bạn cần Q&A trên file mình upload, dùng [Vaulter](./05_vaulter.md). Visionary chỉ trả lời theo nội dung Knowledge Agent đã được nuôi sẵn.

---

## 3. Cách sử dụng (Step-by-step)

### A. Chế độ Chat (mặc định)

#### Bước 1 — Mở The AI Visionary
![Giao diện chat](./images/visionary-01-chat.png)
*Hình 1: Khung chat chính, ô nhập tin nhắn ở dưới.*

#### Bước 2 — Gõ câu hỏi
Ví dụ: `"What is our AI roadmap for 2026?"`, `"Chiến lược AI ở Wholesale Banking là gì?"`, `"Use case Generative AI ưu tiên năm nay?"`.

#### Bước 3 — Nhận trả lời
Visionary proxy yêu cầu tới Knowledge Agent endpoint, parse kết quả, hiển thị reply.

![Trả lời từ KA](./images/visionary-02-answer.png)
*Hình 2: Phản hồi text với insight từ Knowledge Agent.*

#### Bước 4 — Hỏi tiếp
Visionary lưu context trong session — bạn có thể follow-up `"Và milestone Q3 là gì?"`.

### B. Chế độ Report

#### Bước 1 — Bấm "Generate Report"
![Chuyển sang Report mode](./images/visionary-03-report-mode.png)
*Hình 3: Nút Generate Report ở thanh công cụ.*

#### Bước 2 — Mô tả deck cần tạo
Ví dụ: `"Tạo deck 6 slide về cập nhật AI Strategy cho ban giám đốc"`.

#### Bước 3 — Review outline
LLM sinh outline JSON: title + bullet + notes cho từng slide.

#### Bước 4 — Bấm "Build"
Render slide bằng python-pptx (không phức tạp như Powerpoint-er — cover ALL CAPS + content bullet list).

#### Bước 5 — Tải PPTX
![Download deck](./images/visionary-04-download.png)
*Hình 4: File .pptx tải về máy.*

---

## 4. Kịch bản minh hoạ (User Journeys)

### Kịch bản 1 — Tra cứu nhanh AI roadmap trước buổi present

**Bối cảnh:** Bạn sắp present với client `"giới thiệu AI strategy của ngân hàng"`. Cần refresh nhanh các điểm chính.

**Bước thực hiện:**
1. Mở Visionary → `"Hãy tóm tắt AI roadmap 2026 trên 3 trụ cột chính"`.
2. Nhận reply với 3 trụ cột + use case ưu tiên.
3. Hỏi tiếp: `"Use case GenAI ưu tiên Q3 là gì?"` → reply.
4. Hỏi tiếp: `"Ngân sách AI 2026 đang allocate cho team nào nhiều nhất?"`.
5. Copy các đoạn cần thiết, làm note vào slide riêng.

---

### Kịch bản 2 — Tạo weekly update deck cho team

**Bối cảnh:** Mỗi tuần bạn gửi update deck "AI Initiatives This Week" cho team 20 người.

**Bước thực hiện:**
1. Mở Visionary → bấm Generate Report.
2. Prompt: `"Tạo weekly AI update deck 8 slide: 1 cover, 1 agenda, 5 highlights (1 highlight/slide), 1 next steps"`.
3. Review outline → chỉnh tiêu đề nếu cần.
4. Bấm Build → tải `weekly_AI_update.pptx`.
5. Mở trong PowerPoint, edit nếu cần — **file Visionary export là PPTX có thể edit** (khác Powerpoint-er).

![Weekly update deck](./images/visionary-scn2-weekly.png)
*Hình: Deck đơn giản với cover + bullet content.*

---

### Kịch bản 3 — Multi-turn Q&A để hiểu sâu

**Bối cảnh:** Bạn mới join, cần học nhanh về AI strategy.

**Bước thực hiện:**
1. Lượt 1: `"AI strategy nội bộ có những initiative nào?"` → reply liệt kê.
2. Lượt 2: `"Initiative number 3 — chi tiết hơn được không?"` → reply chi tiết.
3. Lượt 3: `"Ai là sponsor của initiative đó?"` → reply.
4. Lượt 4: `"Có document gốc tôi có thể đọc thêm không?"` → reply có thể có link.

---

## 5. Tips sử dụng

- **Visionary ≠ Vaulter**: Visionary trả lời dựa trên kho tài liệu **chiến lược nội bộ** (cố định, do team quản lý). Vaulter trả lời dựa trên **file bạn tự upload**.
- **Câu hỏi tiếng Anh hay tiếng Việt đều được**, nhưng Knowledge Agent có thể trả lời chính xác hơn với tiếng Anh tuỳ theo cách nuôi nguồn.
- **Bắt đầu session mới khi đổi chủ đề lớn**: > 50 message sẽ bị từ chối — vào session mới gọn hơn.
- **Cảnh báo dài**: nếu paste vào ô chat một block > 30K ký tự (~6.000 từ) sẽ bị reject. Tóm tắt trước khi paste.
- **Report deck đơn giản** — phù hợp update internal. Nếu cần deck visual đẹp cho client, dùng [Powerpoint-er](./04_powerpointer.md).
- **Citation**: hỏi `"Nguồn cho thông tin này là tài liệu nào?"` — Knowledge Agent có thể trả về tên tài liệu gốc nếu được cấu hình.

---

## 6. Hạn chế đã biết

- **Nguồn cố định**: bạn không thêm/sửa tài liệu được — phải liên hệ team Knowledge Agent.
- **Không streaming**: chờ trọn câu trả lời.
- **Giới hạn 50 message / 30K ký tự** — phù hợp Q&A, không phù hợp truy vấn dạng batch.
- **Report mode đơn giản**: không có template thương hiệu phong phú, không có rich editor, không có AI Edit từng slide.
- **Không chỉnh sửa được outline phong phú như Powerpoint-er**: chỉ có cover + content slide.
- **Phụ thuộc Knowledge Agent endpoint**: nếu endpoint downtime, Visionary trả về HTTP 502 — phải đợi recover.
