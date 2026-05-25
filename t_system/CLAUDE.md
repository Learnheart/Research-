# CLAUDE.md — Workspace t_system/
[CẬP NHẬT: 25/05/2026]

> Phạm vi: phân tích & so sánh kiến trúc Agentic AI (blueprint vs implementation) trong subtree `t_system/`.
> Source T1: `bankwide_agentic_platform/agent_platform.md` (blueprint), `agent_platform/agent_studio_*.md` (implementation of record), `agentic/Agent_analytics.md` (use case riêng).

## 1. Quy tắc cốt lõi
- **Ngôn ngữ**: tiếng Việt có dấu; thuật ngữ kỹ thuật giữ EN (LangGraph, ReAct, MCP, A2A, RAG, HITL).
- **Format**: `.md` CommonMark; tag `[CẬP NHẬT: DD/MM/YYYY]` ngay dưới H1.
- **Citation nội bộ**: mọi claim kiến trúc phải kèm `(file.md § section)` — không khẳng định chay.
- **Versioning**: sửa nội dung → cập nhật tag ngày + 1 dòng change log cuối file.

## 2. Convention cho doc phân tích kiến trúc
- **Bảng > prose**: so sánh / gap / pattern dùng bảng (Capability | Blueprint | Implementation | Gap).
- **Pattern phân 3 nhóm**: Architectural / Agent / Implementation; mỗi pattern có cột "Minh chứng cụ thể".
- **Gap rõ ràng**: ✅ có / ◐ một phần / ❌ thiếu; kèm khuyến nghị Must / Should / Could.
- **Tách vision vs implementation**: không trộn — 2 cột/bảng riêng, kết luận ở section cuối.

## 3. Cấm
- Marketing language ("revolutionary", "best-in-class"); claim không trỏ source; trùng nội dung giữa các doc.
