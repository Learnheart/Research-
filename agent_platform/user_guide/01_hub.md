# Hub — Trợ lý AI hợp nhất

> Một cửa chat duy nhất. Bạn mô tả việc cần làm — Hub tự chọn và gọi agent chuyên biệt phía sau.

---

## 1. Giới thiệu khả năng

**Hub là gì?** Là điểm vào (entry point) cho toàn bộ Agent Studio. Bạn không cần biết "Translator nằm ở đâu" hay "agent nào tóm tắt được file" — chỉ cần viết yêu cầu bằng ngôn ngữ tự nhiên, Hub sẽ điều phối.

**Khi nào dùng Hub?**
- Bạn không chắc agent nào phù hợp.
- Bạn muốn làm chuỗi việc liên tiếp (dịch → tóm tắt → tạo slide) trong cùng một cuộc hội thoại.
- Bạn cần trợ lý "nhớ" ngữ cảnh giữa các yêu cầu.

**Input điển hình**
- Tin nhắn text: `"Dịch báo cáo này sang tiếng Anh"`, `"Tạo presentation 10 slide về chiến lược AI 2026"`.
- File đính kèm (khi Hub yêu cầu): PDF, DOCX, PPTX, TXT.

**Output điển hình**
- Phản hồi text trả về trong khung chat.
- Hành động đi kèm: nút download file, mở canvas brainstorm, hiển thị slide preview…

**Các agent Hub có thể gọi (qua tool):**
Translator, Summarizer, Powerpoint-er, Vaulter, Brainstormer, AI Visionary.

---

## 2. Giới hạn & điều kiện sử dụng

| Mục | Giới hạn |
| --- | --- |
| File hỗ trợ | PDF, DOCX, PPTX, TXT |
| Kích thước file | Tối đa **20 MB** mỗi file |
| Đăng nhập | Bắt buộc SSO của tổ chức |
| Agent chưa kết nối | **Canvas Designer** và **Resume Evaluator** chưa có tool trong Hub — vui lòng mở agent trực tiếp |
| Streaming | Không — kết quả trả về một lần khi xử lý xong |

---

## 3. Cách sử dụng (Step-by-step)

### Bước 1 — Mở Hub
Truy cập tab **Hub** trong trang chủ Agent Studio.

![Trang chủ Hub](./images/hub-01-home.png)
*Hình 1: Giao diện chính Hub — khung chat ở giữa, lịch sử session ở sidebar trái.*

### Bước 2 — Gõ yêu cầu
Nhập việc bạn cần làm bằng ngôn ngữ tự nhiên, ví dụ: `"Tạo slide 10 trang về kế hoạch AI 2026 cho hội đồng quản trị"`.

![Nhập yêu cầu](./images/hub-02-prompt.png)
*Hình 2: Ô nhập tin nhắn ở dưới cùng. Nhấn Enter để gửi.*

### Bước 3 — Phản hồi của Hub
- **Trường hợp A — Cần file:** Hub sẽ trả lời `"Tôi cần bạn upload một file…"`. Một vùng kéo-thả xuất hiện ngay trong chat.
- **Trường hợp B — Không cần file:** Hub sẽ gọi agent phù hợp ngay (ví dụ Powerpoint-er) và bắt đầu xử lý.

![Yêu cầu upload file](./images/hub-03-file-request.png)
*Hình 3: Khi cần file, vùng kéo-thả tự xuất hiện trong khung chat.*

### Bước 4 — Upload file (nếu được yêu cầu)
Kéo-thả file vào vùng được đánh dấu, hoặc bấm để chọn từ máy.

### Bước 5 — Đợi xử lý
Hub có thể mất 10–60 giây vì phải gọi agent chuyên biệt phía sau. Khung chat hiển thị trạng thái xử lý.

### Bước 6 — Nhận kết quả
Kết quả hiển thị dưới dạng phản hồi text, kèm các nút hành động (Download, Mở canvas, v.v.).

![Kết quả + nút download](./images/hub-04-result.png)
*Hình 4: Hub trả về kết quả kèm nút Download để tải file.*

### Bước 7 — Tiếp tục hội thoại (tuỳ chọn)
Gõ yêu cầu kế tiếp — Hub vẫn nhớ file/ngữ cảnh trước đó. Ví dụ: `"Bây giờ tóm tắt cùng file đó"`.

---

## 4. Kịch bản minh hoạ (User Journeys)

### Kịch bản 1 — Dịch nhanh báo cáo tài chính

**Bối cảnh:** Bạn nhận `Q3_Report.pdf` từ đối tác và cần bản tiếng Anh để gửi cho giám đốc khu vực.

**Bước thực hiện:**
1. Mở Hub, gõ: `"Dịch file này sang tiếng Anh"`.
2. Hub trả lời: `"Tôi cần bạn upload một file để dịch."`
   ![Hub yêu cầu file](./images/hub-scn1-upload-prompt.png)
   *Hình: Vùng upload xuất hiện sau yêu cầu.*
3. Kéo-thả `Q3_Report.pdf` vào.
4. Hub gọi Translator (mất ~45 giây với PDF 10 trang), hiển thị tiến độ.
5. Hub trả về: `"Đã dịch xong. Bạn tải file ở đây:"` + nút Download.

**Kết quả:** Tải về `Q3_Report_translated.docx` (PDF được chuyển sang DOCX — xem [The Translator](./02_translator.md)).

---

### Kịch bản 2 — Chuỗi nhiều bước trong một cuộc hội thoại

**Bối cảnh:** Bạn có biên bản họp `Meeting_2026-05-15.docx` muốn (1) tóm tắt nhanh, (2) tạo slide review.

**Bước thực hiện:**
1. Mở Hub, gõ: `"Tóm tắt file biên bản họp này"` → upload file → nhận tóm tắt 5 câu + 7 key points.
2. Gõ tiếp (cùng session): `"Tạo slide 8 trang trình bày các quyết định chính cho team"`.
3. Hub dùng tóm tắt vừa rồi làm nguyên liệu, gọi Powerpoint-er, trả về preview slide.
4. Bạn duyệt outline → nhấn "Build" → tải `.pptx`.

![Chuỗi 2 tool trong 1 session](./images/hub-scn2-chained.png)
*Hình: Hub kết hợp Summarizer rồi Powerpoint-er trong cùng cuộc hội thoại.*

---

### Kịch bản 3 — Yêu cầu trực tiếp không cần file

**Bối cảnh:** Bạn muốn nhanh có bản nháp PowerPoint về chủ đề mới.

**Bước thực hiện:**
1. Mở Hub, gõ: `"Tạo presentation 10 slide về 'Ứng dụng AI Agent trong vận hành ngân hàng' cho ban điều hành"`.
2. Hub gọi thẳng Powerpoint-er (không cần file đầu vào), trả về outline.
3. Review outline, gõ: `"Đổi slide 5 sang dạng process timeline"` — Hub gọi tool ai-edit-slide.
4. Khi hài lòng → Build → tải PPTX.

---

## 5. Tips sử dụng

- **Mô tả rõ "động từ + đối tượng"**: `"Dịch file này sang tiếng Anh"` rõ hơn `"Cái này cần xử lý"`.
- **Chỉ định ngôn ngữ/định dạng đầu ra**: `"Tóm tắt bằng tiếng Việt, không quá 5 bullet"`.
- **Tận dụng ngữ cảnh trong session**: sau khi upload file, không cần upload lại — chỉ cần nói `"Bây giờ làm X với file đó"`.
- **Nếu cần Canvas Designer hoặc Resume Evaluator**: mở trực tiếp trang agent, vì Hub chưa kết nối.
- **Khi yêu cầu phức tạp, chia thành các bước nhỏ**: thay vì `"Dịch, tóm tắt, rồi tạo slide"` trong một lệnh, hãy gửi 3 lệnh liên tiếp để theo dõi từng kết quả.

---

## 6. Hạn chế đã biết

- **Độ trễ cao hơn**: Hub gọi agent phía sau qua HTTP → mỗi lệnh chậm hơn ~2–5 giây so với gọi agent trực tiếp.
- **Không streaming**: không thấy text được sinh dần — chờ trọn vẹn kết quả.
- **Chưa kết nối Canvas Designer & Resume Evaluator**: phải vào trang agent riêng.
- **Session state tăng dần**: cuộc hội thoại càng dài, Hub càng phải nén lịch sử — chất lượng nhớ giảm sau ~30 lượt trao đổi. Khuyến nghị bắt đầu session mới khi đổi chủ đề lớn.
- **Không hoàn tác**: nếu yêu cầu sai, gõ yêu cầu mới — Hub không có nút "undo" cho hành động đã chạy.
