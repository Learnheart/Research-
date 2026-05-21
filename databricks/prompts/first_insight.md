Đóng vai trò là Chuyên gia Nghiên cứu Chiến lược và Tư vấn công nghệ cấp cao (chuẩn Big4). 
Hãy đọc và phân tích những tài liệu của databricks về các khả năng cung cấp phục vụ cho việc xây dựng AI Agent để làm rõ techstack và capabilities của họ

Trước khi quyết định dồn nguồn lực R&D, tôi cần một view tổng quan về tính khả thi và cấu trúc dự án để báo cáo sơ bộ cho CIO/CFO.

HÃY THỰC HIỆN 3 NHIỆM VỤ SAU:

1. Phân tích Bức tranh Toàn cảnh (Tech Overview):
- Đánh giá tổng quan về thị trường và công nghệ lõi của Databricks trong lĩnh vực AI Agent.
- Phân tích tính ứng dụng thực tế trong mảng ngân hàng tại các nhóm quốc gia: Âu Mỹ, Trung Quốc, và đặc biệt là thị trường Việt Nam
- So sánh sự tiến hóa công nghệ qua 2 giai đoạn: [GIAI_ĐOẠN_1 - VD: Trước năm 2026] và [GIAI_ĐOẠN_2 - VD: Từ 2026 trở lại đây]. Đồng thời so sánh ưu nhược điểm với các đối tác cạnh tranh

2. Khởi tạo Cấu trúc Dự án (Big4 Standard Folder Tree):
- Tạo cây thư mục chuẩn hóa tư vấn để chứa các tài liệu nghiên cứu tiếp theo.
- Cấu trúc phải chia sâu đúng 2 level (Level 1: Nhóm nội dung chính, Level 2: Các thư mục con và file chi tiết).
- TẠO NGAY: Trong mỗi thư mục, bắt buộc có một file `_overview.md` tóm tắt nội dung sẽ được triển khai bên trong.

3. Phác thảo Strategic Roadmap:
- Dựa trên overview, đưa ra lộ trình (roadmap) triển khai cấp cao (High-level) từ R&D -> POC -> Production để tôi có cơ sở trình bày với C-Level và đánh giá độ phủ sóng của nhà cung cấp này cho việc phát triển sản phẩm hiện tại 

Các thông tin được rút ra đều phải có reference đầy đủ, có thể tham khảo tại đường dẫn: https://docs.databricks.com/aws/en/agents/


Đóng vai trò là AI Project Manager & Lead Architect. Chúng ta sẽ cần xây dựng lại cấu trúc cho bài nghiên cứu

HÃY THỰC HIỆN CHÍNH XÁC CÁC BƯỚC SAU MỘT CÁCH TUẦN TỰ:

1. TẠO FILE RULES (`CLAUDE.md` hoặc `RULES.md`):
- Bắt buộc sử dụng tiếng Việt có dấu chuẩn 100% trong mọi file được tạo ra.
- Mọi file nội dung phải là định dạng `.md`.
- Mọi thông tin kỹ thuật, số liệu, thị trường BẮT BUỘC phải có [Citations/Links] ở cuối mỗi section.
- Đánh dấu phiên bản [CẬP NHẬT: DD/MM/YYYY] ở đầu mỗi file.

2. TẠO WORKSPACE TRACKING (`PROGRESS.md` & `PLAYBOOK.md`):
- `PROGRESS.md`: Chứa checklist các phase cần làm. Đánh dấu [ ] cho task chưa làm và [x] cho task hoàn thành.
- `PLAYBOOK.md`: Ghi chép lại các lệnh và workflow để update dự án.

Dựa trên cấu trúc thư mục vừa tạo cho dự án nghiên cứu. Hãy đóng vai trò là Tech Lead và Business Analyst:

1. Viết nội dung draft tổng quan (Overview) cho các file trong thư mục `02_Market` và `03_technology`.
2. TẠO DEEP RESEARCH PROMPTS: Sinh ra một danh sách 20 câu hỏi nghiên cứu chuyên sâu (Deep Research Prompts) chia làm 4 nhóm:
   - Công nghệ Lõi (VD: Các mô hình OCR SOTA hiện nay, tối ưu hóa suy luận).
   - Kiến trúc Hạ tầng (VD: Thiết kế multi-tenant, cô lập dữ liệu cho các tier bảo mật cao).
   - Bảo mật & Compliance (VD: Tiêu chuẩn mã hóa, che giấu dữ liệu nhạy cảm PII).
   - Thị trường & Đối thủ.
3. Lọc ra "TOP 10 PRIORITY PROMPTS" quan trọng nhất và lưu vào file `Phase_1_TOP_10_PRIORITY_PROMPTS.md`. 
Yêu cầu mỗi prompt trong top 10 phải có lệnh rõ ràng: "Đầu ra phải là file .md và có đầy đủ link trích dẫn".

- tôi cần một prompt lớn để phục vụ cho việc deep research bằng claude web. OUtput là 1 prompt trong 1 file .md để trong folder databricks\prompts
- phạm vi địa lý nên xét ở cả 3 khu vực : âu mỹ, trung quốc, và việt nam.
- bao gồm cả phân tích cạnh tranh (workstream 04) và market