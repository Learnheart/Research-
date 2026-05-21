# Prompt: Tóm tắt khả năng Databricks cho việc build AI Agent

## Vai trò
Bạn là technical researcher tổng hợp các capabilities mà Databricks cung cấp để hỗ trợ xây dựng AI agents.

## Nguồn
- **Bắt buộc**: https://docs.databricks.com/aws/en/agents/ và các trang con liên kết trực tiếp trong cùng docs
- KHÔNG dùng blog, nguồn thứ ba, hay training data nếu mâu thuẫn với docs chính thức

## Mục tiêu
Bản tóm tắt **high-level** trả lời được:
1. Databricks cung cấp những thành phần cốt lõi nào cho việc build agent?
2. Mỗi thành phần giải quyết vấn đề gì trong agent lifecycle (develop → evaluate → deploy → monitor)?
3. Khi nào nên chọn Databricks thay vì nền tảng khác?

## Yêu cầu output
- Phân nhóm theo capability (ví dụ: authoring, tools & retrieval, evaluation, governance, serving, monitoring — tự quyết định cách nhóm phù hợp với docs)
- KHÔNG đi sâu vào code samples, API signatures, hay step-by-step tutorials
- Có link tới mục docs tương ứng cho mỗi capability
- Ngôn ngữ tiếng Việt (giữ thuật ngữ tiếng Anh khi cần)
- Đưa ra các hướng có thể adapt vào sản phẩm thực tế của các doanh nghiệp với mỗi capabilities
- Luôn phải có một file README.md để summerize lại nội dung của toàn bộ báo cáo
- TẠO WORKSPACE TRACKING (`PROGRESS.md` & `PLAYBOOK.md`):
    - `PROGRESS.md`: Chứa checklist các phase cần làm. Đánh dấu [ ] cho task chưa làm và [x] cho task hoàn thành.
    - `PLAYBOOK.md`: Ghi chép lại các lệnh và workflow để update dự án.

## Ràng buộc
- Chỉ liệt kê những gì docs thực sự đề cập; không suy đoán hay bổ sung kiến thức ngoài
- Nếu thông tin không có trong docs, ghi rõ "không tìm thấy trong docs"
- Tự quyết định cách triển khai (thứ tự đọc, cách crawl các trang con, cách phân nhóm) — không hỏi lại trước khi bắt đầu
- Việc read và write dữ liệu cho phần này được phép toàn quyền trong folder databricks\capabilities. Không được phép upsert những thông tin trong các file khác


## Tiêu chí thành công
Người đọc sau khi xem tóm tắt có thể quyết định được Databricks có phù hợp với use case của họ hay không mà không cần mở docs gốc.