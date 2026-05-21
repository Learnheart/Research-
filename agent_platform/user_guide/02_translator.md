# The Translator — Dịch tài liệu giữ nguyên format

> Dịch PDF, DOCX, PPTX, TXT giữa các ngôn ngữ mà không phá vỡ layout — bố cục, font, bảng biểu, slide đều được giữ lại.

---

## 1. Giới thiệu khả năng

**The Translator làm gì?** Dịch tài liệu giữa hai ngôn ngữ bất kỳ và **trả về file cùng định dạng** với bố cục gần như nguyên gốc.

**Khi nào dùng?**
- Bạn có báo cáo, đề xuất, slide cần bản dịch nhưng không muốn mất công định dạng lại.
- Bạn cần dịch nhanh nhiều slide (PPTX) hoặc bảng biểu (DOCX).
- Ngôn ngữ nguồn không chắc — agent tự nhận diện.

**Input**
- 1 file: PDF, DOCX, PPTX hoặc TXT.
- Ngôn ngữ đích (mặc định: Tiếng Việt).
- Ngôn ngữ nguồn (mặc định: tự động phát hiện).

**Output**
- File đã dịch, cùng định dạng đầu vào — **trừ PDF**, sẽ trả về DOCX.
- Với TXT: text hiển thị thẳng trên web + nút Copy.
- Tiến độ realtime: số chunk đã dịch / tổng chunk + ước lượng thời gian còn lại.

---

## 2. Giới hạn & điều kiện sử dụng

| Mục | Giới hạn |
| --- | --- |
| Định dạng hỗ trợ | PDF, DOCX, PPTX, TXT |
| Kích thước file | Tối đa **20 MB** |
| TXT | Truncate ở **12.000 ký tự** — phần vượt sẽ không được dịch |
| PDF | Output là **DOCX** (không trả lại PDF) |
| Timeout client | 5 phút — file rất lớn có thể bị ngắt phía giao diện |
| Đăng nhập | SSO bắt buộc |

---

## 3. Cách sử dụng (Step-by-step)

### Bước 1 — Mở The Translator
Vào tab **The Translator** trong Agent Studio.

![Giao diện chính](./images/translator-01-home.png)
*Hình 1: Panel trái là vùng upload, panel phải hiển thị kết quả/tiến độ.*

### Bước 2 — Upload file
Kéo-thả file vào ô bên trái, hoặc bấm để chọn.

![Vùng upload](./images/translator-02-upload.png)
*Hình 2: Vùng kéo-thả file ở panel trái.*

### Bước 3 — Chọn ngôn ngữ đích
Mở dropdown "Translate to" và chọn ngôn ngữ. Mặc định là Tiếng Việt.

![Chọn ngôn ngữ](./images/translator-03-language.png)
*Hình 3: Dropdown ngôn ngữ đích.*

### Bước 4 — Bấm "Translate to {ngôn ngữ}"
Quá trình gồm 4 giai đoạn:
1. **Upload** — file lên Volume.
2. **Analyzing** — backend đọc & validate.
3. **Translating** — dịch song song, hiển thị `chunk X / Y` + ước lượng thời gian.
4. **Building** — ghép file kết quả.

![Tiến độ dịch](./images/translator-04-progress.png)
*Hình 4: Progress bar và thông báo realtime.*

### Bước 5 — Tải kết quả
- **PPTX / DOCX / PDF (→DOCX):** nút "Download" xuất hiện ở panel phải.
- **TXT:** text dịch hiển thị trực tiếp, kèm nút Copy.

![Nút download](./images/translator-05-download.png)
*Hình 5: Nút tải file kết quả.*

---

## 4. Kịch bản minh hoạ (User Journeys)

### Kịch bản 1 — Dịch slide khách hàng từ tiếng Anh sang tiếng Việt

**Bối cảnh:** Bạn nhận `Vendor_Pitch.pptx` 25 slide từ đối tác cần dịch để trình ban giám đốc.

**Bước thực hiện:**
1. Mở Translator → kéo-thả `Vendor_Pitch.pptx`.
2. Chọn ngôn ngữ đích: **Vietnamese**.
3. Bấm "Translate to Vietnamese".
4. Quan sát tiến độ: `chunk 5 / 25 — còn ~3 phút`.
   ![Dịch PPTX](./images/translator-scn1-pptx.png)
   *Hình: Slide-by-slide progress.*
5. Sau khi hoàn tất, bấm Download → nhận `Vendor_Pitch_VN.pptx`.

**Lưu ý quan sát được:** chữ trong table, group shape, header/footer đều được dịch và giữ font/màu/vị trí.

---

### Kịch bản 2 — Dịch hợp đồng PDF dài

**Bối cảnh:** Hợp đồng `Contract_2026.pdf` 40 trang cần bản tiếng Anh.

**Bước thực hiện:**
1. Upload PDF (~8 MB).
2. Chọn ngôn ngữ đích: **English**.
3. Bấm Translate. Agent chuyển PDF → DOCX nội bộ, sau đó dịch.
4. Tải về `Contract_2026_translated.docx`.

**Cảnh báo:** Output là DOCX chứ không phải PDF. Nếu cần PDF cuối cùng, hãy Export-as-PDF từ Word sau khi review.

---

### Kịch bản 3 — Dịch ghi chú ngắn dạng TXT

**Bối cảnh:** Bạn copy 2.000 ký tự ghi chú nội bộ vào `notes.txt`, cần dịch nhanh sang tiếng Anh.

**Bước thực hiện:**
1. Upload `notes.txt`.
2. Ngôn ngữ đích: **English**.
3. Bấm Translate — hoàn tất trong ~5 giây (vì gọi một lượt duy nhất).
4. Text dịch hiển thị thẳng trên web → bấm "Copy" để dán vào nơi khác.

![Kết quả TXT inline](./images/translator-scn3-txt.png)
*Hình: Text dịch hiển thị inline + nút Copy.*

---

## 5. Tips sử dụng

- **Dùng PPTX/DOCX nếu có thể** — chất lượng giữ format tốt hơn PDF.
- **PDF có hình quét (scan)** sẽ không dịch được — agent chỉ đọc text layer. Chạy OCR trước nếu cần.
- **File >20MB**: tách nhỏ ra (theo chương / theo nhóm slide) rồi dịch từng phần.
- **Thuật ngữ chuyên ngành**: hiện chưa có glossary — review sau khi dịch để chỉnh thuật ngữ tài chính/pháp lý chính xác.
- **TXT vượt 12.000 ký tự**: chia thành nhiều file hoặc dán vào DOCX (DOCX không có giới hạn ký tự nghiêm ngặt).
- **Nếu trình duyệt timeout sau 5 phút**: backend vẫn xử lý tiếp — chờ một lúc rồi refresh trang để lấy kết quả.

---

## 6. Hạn chế đã biết

- **PDF → DOCX**: không có cách giữ output PDF; phải export lại từ Word nếu bắt buộc PDF.
- **TXT bị cắt ở 12.000 ký tự** — phần sau bị bỏ.
- **Không có glossary / từ điển chuyên ngành** — thuật ngữ tài chính có thể không nhất quán.
- **Tối đa 5 worker song song** — file PPTX rất lớn (50+ slide) vẫn xử lý theo lô.
- **PDF scan / ảnh** không dịch được (không có OCR built-in).
- **Không tự gửi email** — phải tự tải về và chia sẻ.
