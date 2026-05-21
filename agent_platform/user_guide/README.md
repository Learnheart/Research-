# Agent Studio — User Guide

> Hướng dẫn sử dụng end-user cho 9 agent của Agent Studio. Tài liệu này dành cho người dùng trên web app — không yêu cầu kiến thức kỹ thuật.

---

## Bạn cần làm gì? → Chọn agent phù hợp

| Bạn muốn… | Agent phù hợp | Đầu vào điển hình | Đầu ra điển hình |
| --- | --- | --- | --- |
| Trò chuyện 1 cửa, để AI tự chọn công cụ | **Hub** | Câu hỏi/yêu cầu text + file (nếu cần) | Phản hồi text + tài liệu kết quả |
| Dịch tài liệu giữ nguyên format | **The Translator** | File PDF / DOCX / PPTX / TXT | File đã dịch (cùng định dạng) |
| Tóm tắt 1 hay nhiều tài liệu dài | **The Summarizer** | 1–N file | Tóm tắt 3–5 câu + 5–8 ý chính |
| Tạo slide PowerPoint nhanh | **The Powerpoint-er** | Chủ đề + audience + số slide | File `.pptx` thương hiệu công ty |
| Lưu tài liệu thành "kho tri thức" để hỏi đáp | **The Vaulter** | Tài liệu (upload dần) | Knowledge graph + Q&A |
| Brainstorm ý tưởng / giải pháp | **The Brainstormer** | Vấn đề / chủ đề muốn nghĩ thấu | Canvas: vấn đề → lựa chọn → action plan |
| Hỏi đáp về chiến lược AI nội bộ | **The AI Visionary** | Câu hỏi về định hướng AI | Phản hồi từ Knowledge Agent + báo cáo PPTX |
| Tạo hình ảnh / SVG / DOCX có thiết kế | **The Canvas Designer** | Mô tả hình ảnh mong muốn | File PNG / JPG / SVG / DOCX |
| Đánh giá CV ứng viên so với JD | **The Resume Evaluator** | CV + Job Description | Bảng điểm 6 tiêu chí + đề xuất phỏng vấn |

---

## Danh sách hướng dẫn chi tiết

1. [Hub — Trợ lý AI hợp nhất](./01_hub.md)
2. [The Translator — Dịch tài liệu](./02_translator.md)
3. [The Summarizer — Tóm tắt tài liệu](./03_summarizer.md)
4. [The Powerpoint-er — Tạo slide](./04_powerpointer.md)
5. [The Vaulter — Kho tri thức cá nhân](./05_vaulter.md)
6. [The Brainstormer — Đối tác brainstorming](./06_brainstormer.md)
7. [The AI Visionary — Hỏi đáp AI Strategy](./07_visionary.md)
8. [The Canvas Designer — Tạo hình ảnh & đồ hoạ](./08_canvas_designer.md)
9. [The Resume Evaluator — Đánh giá CV](./09_resume_evaluator.md)

---

## Cấu trúc một bài hướng dẫn

Mỗi file hướng dẫn agent đều theo cấu trúc 6 phần:

1. **Giới thiệu khả năng** — Agent làm gì, khi nào dùng, Input/Output.
2. **Giới hạn & điều kiện sử dụng** — File type, size, quyền truy cập, prerequisites.
3. **Cách sử dụng (Step-by-step)** — Trình tự thao tác trên web app.
4. **Kịch bản minh hoạ (User Journeys)** — 2–3 tình huống end-to-end + chỗ chèn ảnh.
5. **Tips sử dụng** — Mẹo để có kết quả tốt hơn.
6. **Hạn chế đã biết (Limitations)** — Điều agent chưa làm được hoặc làm chưa tốt.

---

## Quy ước hình ảnh

- Hình minh hoạ đặt trong `./images/<agent>-<seq>-<mô_tả>.png`.
- Caption mô tả ngắn bên dưới mỗi hình bằng chữ in nghiêng.
- Khi UX team chuẩn bị screenshot, chỉ cần đặt file vào đúng đường dẫn — markdown sẽ tự render.

## Truy cập & hỗ trợ

- Tất cả agent đăng nhập qua **SSO của tổ chức** — không cần tài khoản riêng.
- Mỗi agent có URL riêng (Databricks App). Hub là entry point khuyến nghị cho người mới.
- Vấn đề kỹ thuật hoặc đề xuất tính năng: liên hệ team Agent Studio.

---

*Tài liệu này mô tả Agent Studio tại thời điểm phát hành. Khi agent có cập nhật lớn (thêm tool, đổi UI), file tương ứng sẽ được revise.*
