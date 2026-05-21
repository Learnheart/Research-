# The Summarizer — Tóm tắt tài liệu

> Nén báo cáo dài, biên bản họp, hợp đồng thành 3–5 câu executive summary + 5–8 key insight có thể hành động.

---

## 1. Giới thiệu khả năng

**The Summarizer làm gì?** Nhận một hoặc nhiều tài liệu, đọc toàn bộ và trả về:
- Một đoạn tóm tắt cấp lãnh đạo (3–5 câu).
- Danh sách 5–8 ý chính dạng bullet (ưu tiên: quyết định, số liệu, deadline, rủi ro).

**Khi nào dùng?**
- Bạn cần "nắm gọn" trong 2 phút trước cuộc họp.
- Bạn có nhiều file cùng chủ đề và muốn một bản tổng hợp duy nhất.
- Bạn muốn rút ý chính từ biên bản họp / báo cáo / nghiên cứu thị trường.

**Input**
- 1 hoặc nhiều file: PDF, DOCX, PPTX, TXT.

**Output**
- `summary`: đoạn văn 3–5 câu.
- `key_points`: 5–8 bullet ý chính.
- `truncated`: cờ báo nếu nội dung gốc bị cắt do quá dài.

---

## 2. Giới hạn & điều kiện sử dụng

| Mục | Giới hạn |
| --- | --- |
| Định dạng | PDF, DOCX, PPTX, TXT |
| Kích thước mỗi file | Tối đa **20 MB** |
| Tổng văn bản đầu vào | **100.000 ký tự** — vượt sẽ bị cắt và đánh dấu `truncated: true` |
| Số file cùng lúc | Không giới hạn cứng, nhưng tổng phải nằm trong 100K ký tự |
| Đăng nhập | SSO bắt buộc |

---

## 3. Cách sử dụng (Step-by-step)

### Bước 1 — Mở The Summarizer
Vào tab **The Summarizer**.

![Giao diện Summarizer](./images/summarizer-01-home.png)
*Hình 1: Vùng upload bên trái, kết quả bên phải.*

### Bước 2 — Upload một hoặc nhiều file
Kéo-thả nhiều file cùng lúc, hoặc bấm chọn nhiều lần.

![Upload nhiều file](./images/summarizer-02-multi-upload.png)
*Hình 2: Danh sách file đã thêm — có thể xoá từng file trước khi tóm tắt.*

### Bước 3 — Bấm "Summarize"
Agent đọc tất cả file, ghép nội dung, gửi tới LLM.

### Bước 4 — Đợi kết quả
Khoảng 10–30 giây tuỳ tổng kích thước.

### Bước 5 — Xem kết quả
- **Executive Summary** ở trên: 3–5 câu.
- **Key Insights** ở dưới: 5–8 bullet.

![Kết quả tóm tắt](./images/summarizer-03-result.png)
*Hình 3: Kết quả tóm tắt + bullet ý chính.*

### Bước 6 — Copy hoặc thêm file
- Bấm Copy để dán vào email/chat.
- Hoặc thêm file mới → bấm Summarize lại để có bản tổng hợp mở rộng.

---

## 4. Kịch bản minh hoạ (User Journeys)

### Kịch bản 1 — Tóm tắt biên bản họp dài 30 trang

**Bối cảnh:** `Board_Meeting_May.docx` 30 trang. Cần gửi tóm tắt cho team không tham dự.

**Bước thực hiện:**
1. Mở Summarizer → kéo `Board_Meeting_May.docx`.
2. Bấm Summarize → đợi ~15 giây.
3. Nhận:
   - **Summary:** *"Cuộc họp tập trung vào ngân sách Q3, quyết định hoãn dự án X đến Q4, và phê duyệt tuyển 5 vị trí mới cho team Data…"*
   - **Key insights:** 7 bullet — mỗi cái có số liệu cụ thể, người chịu trách nhiệm, deadline.
4. Copy → dán vào email cho team.

![Tóm tắt biên bản](./images/summarizer-scn1-meeting.png)
*Hình: Kết quả có executive summary + bullet ưu tiên quyết định và deadline.*

---

### Kịch bản 2 — Tổng hợp nhiều báo cáo thành 1 bản tóm tắt duy nhất

**Bối cảnh:** Bạn có 4 báo cáo nghiên cứu thị trường từ 4 hãng khác nhau, muốn ý chính tổng hợp.

**Bước thực hiện:**
1. Upload cả 4 file: `Gartner.pdf`, `Forrester.pdf`, `IDC.pdf`, `McKinsey.docx`.
2. Bấm Summarize.
3. Agent ghép tất cả (nối nhau bằng separator `--- filename ---`) và tóm tắt thống nhất.
4. Kết quả: 1 executive summary so sánh quan điểm 4 hãng, 8 bullet so sánh điểm chung và khác biệt.

**Lưu ý:** Tổng 4 file gần 120K ký tự → agent báo `truncated: true`. Phần cuối file `McKinsey.docx` có thể không được tóm tắt — xem tip phần 5.

---

### Kịch bản 3 — Tóm tắt slide bán hàng đối tác

**Bối cảnh:** `Partner_Deck.pptx` 40 slide, bạn cần biết partner đó đang đề xuất gì trước khi vào họp.

**Bước thực hiện:**
1. Upload PPTX.
2. Summarize.
3. Nhận: 5 câu tóm tắt giải pháp đối tác đề xuất + bullet về pricing, timeline, risks.

---

## 5. Tips sử dụng

- **Hỏi cụ thể bằng cách thêm context file**: nếu bạn upload kèm 1 file `prompt.txt` ghi `"Focus on financial risks and Q4 deadlines"`, agent sẽ ưu tiên các phần đó.
- **Tránh tổng quá 100K ký tự**: nếu nhiều file lớn, ưu tiên loại bỏ phụ lục / annexes trước khi upload.
- **Báo `truncated: true`** = nội dung bị cắt — chia nhỏ và tóm tắt từng phần, rồi tổng hợp lại bằng cách upload các bản tóm tắt vừa rồi.
- **Kết quả không "sáng tạo nội dung mới"** — nếu kết quả thiếu một điểm bạn nhớ, có thể điểm đó nằm ở phần đã bị truncate.
- **Tóm tắt biên bản họp** sẽ trả về tốt nếu file có cấu trúc rõ (tiêu đề, người phát biểu, action items).

---

## 6. Hạn chế đã biết

- **Cắt cứng ở 100.000 ký tự** tổng — không có cách nâng giới hạn từ giao diện.
- **Không sinh file** — kết quả chỉ là text trên web; phải copy thủ công.
- **Không lưu lịch sử** — đóng tab là mất kết quả; lưu trước khi rời trang.
- **Không phân biệt nguồn**: nếu upload nhiều file, summary không nói rõ insight nào từ file nào (trừ khi key point bạn yêu cầu rõ điều đó).
- **Không có chế độ "summary theo độ dài"** — luôn là 3–5 câu + 5–8 bullet. Cần ngắn hơn/dài hơn → tự rút gọn hoặc dùng Brainstormer/Hub để mở rộng.
- **Không xử lý ảnh trong tài liệu** — biểu đồ/đồ thị bị bỏ qua, chỉ đọc text.
