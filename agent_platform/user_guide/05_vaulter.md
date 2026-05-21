# The Vaulter — Kho tri thức cá nhân

> Upload tài liệu → AI rút thực thể & quan hệ thành knowledge graph → hỏi đáp dựa trên kho của bạn.

---

## 1. Giới thiệu khả năng

**The Vaulter làm gì?** Biến tập tài liệu của bạn thành một **kho tri thức cá nhân**:
- Trích xuất thực thể (người, dự án, công ty, sản phẩm…) và quan hệ giữa chúng.
- Hiển thị knowledge graph trực quan.
- Cho phép hỏi đáp bằng ngôn ngữ tự nhiên — câu trả lời **chỉ dựa trên** tài liệu bạn đã upload.

**Khi nào dùng?**
- Bạn có chục báo cáo, tài liệu dự án và muốn tra cứu nhanh (`"Ai phụ trách dự án X?"`).
- Bạn muốn xây "memory" cho công việc của riêng mình mà không nhập lại nội dung.
- Bạn cần Q&A có nguồn gốc rõ ràng (LLM không bịa — nếu thông tin không có trong vault, agent sẽ trả lời "không biết").

**Input**
- **Upload mode:** 1 tài liệu (PDF, DOCX, TXT) mỗi lần.
- **Query mode:** câu hỏi dạng text.

**Output**
- **Upload:** xác nhận `X nodes, Y edges extracted`.
- **Graph view:** knowledge graph tương tác.
- **Query:** câu trả lời text dựa trên nội dung vault.

---

## 2. Giới hạn & điều kiện sử dụng

| Mục | Giới hạn |
| --- | --- |
| Định dạng upload | PDF, DOCX, TXT |
| Kích thước file | Tối đa **20 MB** |
| Trích xuất graph | Tối đa **6.000 ký tự** đầu file mỗi lần upload |
| Context khi query | Chỉ dùng **5 tài liệu gần nhất**, tối đa **10.000 ký tự** ghép lại |
| Phạm vi | Mỗi user một vault riêng — **không share** giữa các user |
| Xoá tài liệu | Chưa có API xoá — tài liệu upload nhầm sẽ tồn tại trong vault |
| Đăng nhập | SSO bắt buộc |

---

## 3. Cách sử dụng (Step-by-step)

The Vaulter có 2 chế độ chính: **Upload** và **Query**, cùng một **Graph view** để khám phá.

### A. Chế độ Upload

#### Bước 1 — Mở The Vaulter, chọn tab Upload
![Tab Upload](./images/vaulter-01-upload-tab.png)
*Hình 1: Giao diện chính, hai tab Upload và Query.*

#### Bước 2 — Kéo-thả file
Chọn 1 file (PDF/DOCX/TXT).

#### Bước 3 — Bấm "Upload to Vault"
Backend sẽ:
1. Parse text từ file.
2. Lưu nội dung vào kho.
3. Gọi LLM trích xuất thực thể + quan hệ (giới hạn 6.000 ký tự đầu).
4. Cập nhật graph (loại bỏ trùng lặp).

![Đang xử lý](./images/vaulter-02-uploading.png)
*Hình 2: Tiến độ upload và extract.*

#### Bước 4 — Xem kết quả
Thông báo: `"Document_X.pdf added — 12 nodes, 8 edges extracted"`.

### B. Chế độ Graph view

Sau khi upload, chuyển sang tab **Graph** để xem trực quan.

![Knowledge graph](./images/vaulter-03-graph.png)
*Hình 3: Knowledge graph tương tác — nodes là thực thể, edges là quan hệ.*

- Hover node → xem thông tin chi tiết.
- Kéo thả để bố trí lại.
- Zoom in/out để khám phá cluster.

### C. Chế độ Query

#### Bước 1 — Chuyển sang tab Query
#### Bước 2 — Gõ câu hỏi
Ví dụ: `"Ai chịu trách nhiệm dự án Alpha?"`, `"Dự án X có những risk gì?"`, `"Khi nào dự án Beta kick-off?"`.

![Query box](./images/vaulter-04-query.png)
*Hình 4: Ô câu hỏi, nút Ask.*

#### Bước 3 — Bấm "Ask"
Agent đọc 5 tài liệu gần nhất, trả lời chỉ dựa trên nội dung đó.

#### Bước 4 — Đọc câu trả lời
![Kết quả query](./images/vaulter-05-answer.png)
*Hình 5: Trả lời dựa trên context vault — nếu không tìm thấy, agent nói "I don't have that information."*

---

## 4. Kịch bản minh hoạ (User Journeys)

### Kịch bản 1 — Xây vault cho dự án dài hơi

**Bối cảnh:** Bạn quản lý dự án "AI Transformation" đang chạy 6 tháng. Có ~10 tài liệu: kickoff deck, weekly status, risk register, tech specs, vendor proposal.

**Bước thực hiện:**
1. Tuần 1 — upload 4 file đầu (kickoff, scope, vendor proposal, plan).
2. Mở Graph: thấy các node `Alpha`, `Vendor X`, `Q3 2026 Milestone`, `John (PM)`, `Privacy Risk`…
3. Tuần 4 — upload thêm 2 file status report.
4. Cuối tháng — bạn được hỏi `"Dự án Alpha hiện đang gặp risk gì?"`.
5. Vào tab Query → gõ câu hỏi → agent trả lời dựa trên risk register + status report.

![Vault với 10 tài liệu](./images/vaulter-scn1-rich-graph.png)
*Hình: Graph sau 6 tuần — nhiều cluster theo chủ đề.*

---

### Kịch bản 2 — Tra nhanh trước cuộc họp

**Bối cảnh:** 5 phút trước họp, sếp hỏi `"Vendor X có những commitment gì trong contract?"`.

**Bước thực hiện:**
1. Mở Vaulter → tab Query.
2. Gõ: `"Vendor X commitment in contract?"`.
3. Agent đọc các file đã upload có chứa "Vendor X" trong 5 tài liệu gần nhất → trả về danh sách commitment.
4. Copy câu trả lời, vào họp.

**Cảnh báo:** Nếu file contract bạn cần đã upload từ lâu (vị trí 6 trở đi tính theo thời gian), nó **không** nằm trong context query — cần re-upload hoặc upload file tóm tắt.

---

### Kịch bản 3 — Khám phá kết nối giữa các dự án

**Bối cảnh:** Bạn nghi 2 dự án A và B có overlap về stakeholder/tech stack.

**Bước thực hiện:**
1. Mở Graph view.
2. Tìm node `Project A` và `Project B`.
3. Quan sát các edge chia sẻ — ví dụ cùng kết nối tới `John (Engineering Lead)`, cùng dùng `Databricks`, cùng có `Data Privacy Risk`.
4. Đặt câu hỏi xác nhận: `"What do projects A and B have in common?"` → agent tổng hợp text.

---

## 5. Tips sử dụng

- **Upload theo thứ tự ưu tiên**: vault chỉ dùng 5 file gần nhất cho query — nếu cần Q&A trên file cũ, re-upload nó.
- **File nhỏ hơn → graph chính xác hơn**: chỉ 6.000 ký tự đầu được dùng để trích xuất; tài liệu dài nên tách thành phần (executive summary, scope, risks).
- **Tên thực thể nhất quán**: nếu trong file gọi là `"Mr. John Smith"`, file khác gọi `"John S."`, graph sẽ tạo 2 node khác nhau. Chuẩn hoá tên trước khi upload nếu cần graph gọn.
- **Câu hỏi nên cụ thể**: `"Who manages Project Alpha?"` tốt hơn `"Tell me about Alpha"`.
- **Vault không phải search engine**: nếu cần tra cứu chính xác, dùng Ctrl-F trong file gốc. Vault tốt cho câu hỏi mở.
- **Privacy**: vault là **per-user** — không share với đồng nghiệp. Đừng upload tài liệu cần share rộng và mong người khác query được.

---

## 6. Hạn chế đã biết

- **Chỉ 6.000 ký tự đầu file** được dùng để trích xuất graph — phần sau không vào knowledge graph.
- **Chỉ 5 tài liệu gần nhất** + tối đa 10K ký tự context cho query — vault có 100 file vẫn chỉ query trên 5 file mới nhất.
- **Không có vector search**: không "tìm" theo độ tương đồng — chỉ ghép thẳng nội dung vào context.
- **Không xoá file**: chưa có API delete. Upload nhầm sẽ tồn tại mãi (chỉ bị "đẩy ra khỏi top 5" khi upload thêm).
- **Không scale beyond ~50K ký tự tổng**: vault quá lớn không cải thiện kết quả vì cap context.
- **Graph extraction có thể bỏ sót**: LLM đôi khi không nhận diện được thực thể phụ — không đảm bảo 100%.
- **Per-user**: không share, không có vault chung của team.
