# The Brainstormer — Đối tác brainstorming có canvas

> AI hỏi câu hỏi Socratic theo 4 bước. Canvas trực quan cập nhật theo thời gian thực: vấn đề → các lựa chọn → action plan.

---

## 1. Giới thiệu khả năng

**The Brainstormer làm gì?** Đóng vai một đối tác brainstorming. Thay vì đưa ra câu trả lời ngay, AI hỏi các câu Socratic để bạn tự đào sâu — và đồng bộ một **canvas trực quan** (vấn đề / lựa chọn / kế hoạch) theo từng lượt trò chuyện.

**Khi nào dùng?**
- Bạn có một vấn đề chưa rõ ràng và muốn "nghĩ thấu" thay vì xin câu trả lời.
- Bạn muốn so sánh 2–3 hướng đi với trade-off rõ.
- Bạn cần một action plan có cấu trúc cho cuối phiên.

**Input**
- Tin nhắn text mô tả vấn đề ban đầu.
- Tiếp tục trả lời các câu hỏi của AI trong session.

**Output**
- Phản hồi text trong chat.
- **Canvas** bên phải, cập nhật mỗi lượt:
  - `problem`: vấn đề thực sự (1 câu).
  - `options`: 2–3 lựa chọn kèm trade-off.
  - `action_plan`: các bước cụ thể (xuất hiện ở cuối).
- Session lưu lại → có thể quay lại tiếp tục.

---

## 2. Giới hạn & điều kiện sử dụng

| Mục | Giới hạn |
| --- | --- |
| Cấu trúc phiên | 4 bước cứng: Understand → Explore → Converge → Wrap up |
| Số lượt trao đổi | AI sẽ **buộc kết thúc và đưa action plan** ở **lượt thứ 8** |
| Nén lịch sử | Trên 10 message, lượt cũ được tóm tắt bằng Haiku |
| Preferences | AI nhớ ngôn ngữ / tone / chủ đề **xuyên session** |
| Đăng nhập | SSO bắt buộc |

---

## 3. Cách sử dụng (Step-by-step)

### Bước 1 — Mở The Brainstormer
Giao diện 2 panel: chat bên trái, canvas bên phải.

![Giao diện chính](./images/brainstormer-01-home.png)
*Hình 1: Chat trái + canvas phải. Canvas ban đầu trống.*

### Bước 2 — Gõ vấn đề
Ví dụ: `"Tôi muốn cải thiện trải nghiệm onboarding cho khách hàng mới"`.

### Bước 3 — Trả lời câu hỏi của AI (Step 1: Understand)
**Lượt 1–3:** AI hỏi để hiểu mục tiêu thật:
- *"What would change if you solved this?"*
- *"Who feels this problem most acutely?"*

Canvas hiện tại: `problem: null` (chưa khoá).

![Bước Understand](./images/brainstormer-02-understand.png)
*Hình 2: AI đặt câu hỏi Socratic, canvas đang trống.*

### Bước 4 — Khám phá lựa chọn (Step 2: Explore)
**Lượt 3–6:** AI đưa ra 2–3 hướng tiếp cận với trade-off.

Canvas cập nhật:
```
problem: "Tỉ lệ khách hàng abandon trong tuần đầu cao do quy trình KYC dài"
options:
  - "Số hoá KYC qua eKYC video"
  - "Concierge onboarding cho khách VIP"
  - "Self-service checklist đơn giản"
```

![Canvas có options](./images/brainstormer-03-explore.png)
*Hình 3: Canvas cập nhật với problem + 3 options.*

### Bước 5 — Hội tụ (Step 3: Converge)
**Lượt 6–8:** AI giúp đánh giá option vs mục tiêu — bạn loại bớt và refine.

### Bước 6 — Kết thúc với action plan (Step 4: Wrap up)
**Lượt 8 trở đi:** AI bắt buộc đưa action plan cụ thể.

Canvas hoàn chỉnh:
```
problem: "..."
options: ["...", "..."]
action_plan:
  - "Tuần 1: Map quy trình KYC hiện tại"
  - "Tuần 2: PoC eKYC với vendor X"
  - "Tuần 3: Test với 50 user"
```

Session được đánh dấu `completed: true`.

![Action plan hoàn chỉnh](./images/brainstormer-04-actionplan.png)
*Hình 4: Canvas cuối phiên có problem, options, action_plan.*

### Bước 7 — Lưu / Tiếp tục sau

- Session tự lưu — đóng tab không mất.
- Sidebar trái liệt kê các session cũ → bấm để khôi phục toàn bộ chat + canvas.

![Sidebar session](./images/brainstormer-05-sessions.png)
*Hình 5: Lịch sử session để mở lại.*

---

## 4. Kịch bản minh hoạ (User Journeys)

### Kịch bản 1 — Giải quyết vấn đề mơ hồ

**Bối cảnh:** Bạn cảm thấy "team đang chậm" nhưng không rõ nguyên nhân.

**Bước thực hiện:**
1. Mở Brainstormer → `"Team tôi đang chậm, tôi cần improve nhưng chưa biết bắt đầu từ đâu"`.
2. AI hỏi: *"Chậm so với baseline nào? Có metric cụ thể không?"* → bạn suy nghĩ và trả lời `"velocity giảm 30% so với quý trước"`.
3. AI hỏi tiếp về nguyên nhân nghi ngờ → bạn nói `"có thể do morale, hoặc do tech debt"`.
4. Lượt 4–5: AI đề xuất 3 hướng — fix morale (1-1 + offsite), giảm tech debt (1 sprint dành riêng), tách team thành 2 squad nhỏ.
5. Lượt 6–7: AI giúp đánh giá theo cost/impact.
6. Lượt 8: AI đưa action plan: tuần 1 chạy survey morale + audit tech debt → quyết định approach ở tuần 2.

---

### Kịch bản 2 — So sánh 2 lựa chọn rõ ràng

**Bối cảnh:** Bạn đang phân vân `"Build in-house vs buy SaaS"` cho công cụ phân tích.

**Bước thực hiện:**
1. Vào Brainstormer → mô tả 2 option và budget.
2. AI hỏi: *"Mục tiêu chính là tốc độ ra mắt hay tổng cost dài hạn?"*.
3. Bạn trả lời → AI mở rộng: thêm option lai (`"build wrapper around SaaS"`).
4. Converge: AI map ra cost / time / risk / strategic fit cho mỗi option.
5. Action plan: 1 tuần PoC SaaS, song song đánh giá nội lực team build.

---

### Kịch bản 3 — Quay lại tiếp tục session cũ

**Bối cảnh:** Tuần trước bạn brainstorm về "AI Adoption strategy", chưa xong. Hôm nay muốn tiếp.

**Bước thực hiện:**
1. Mở Brainstormer → sidebar chọn session `"AI Adoption — May 14"`.
2. Toàn bộ chat + canvas khôi phục.
3. Gõ tiếp: `"Tôi đã hỏi team Sales, họ ủng hộ option 2. Mình tiếp tục từ đó nhé"`.
4. AI nhớ context, không hỏi lại từ đầu, chuyển sang refine action plan.

---

## 5. Tips sử dụng

- **Khởi đầu bằng vấn đề, không bằng giải pháp**: `"Tôi muốn improve onboarding"` tốt hơn `"Tôi nên dùng eKYC không?"` — AI sẽ ép bạn nghĩ lại nếu bắt đầu bằng solution.
- **Trả lời câu hỏi của AI thật**: AI không hỏi để câu giờ — câu trả lời ngắn 1 dòng cũng được, miễn là cụ thể.
- **Đừng vội rush qua step Understand**: 2–3 lượt đầu quyết định chất lượng option phía sau.
- **Quan sát canvas**: nếu canvas vẫn `problem: null` sau 3 lượt, AI thấy bạn chưa rõ vấn đề — đừng cố ép sang Explore.
- **Tận dụng cross-session memory**: AI nhớ bạn thích trả lời tiếng Việt / tone trang trọng / hay nghĩ về fintech — không cần lặp lại preferences mỗi session.
- **Khi muốn kết thúc sớm**: gõ `"Cho tôi action plan ngay"` ở bất kỳ lượt nào sau lượt 5 — AI sẽ tổng hợp với những gì đã có.

---

## 6. Hạn chế đã biết

- **Buộc kết thúc ở lượt 8**: nếu vấn đề phức tạp cần > 8 lượt, AI sẽ ép action plan dù chưa đủ converge. Cách bù: tạo session mới cho phần sau.
- **Canvas không free-form**: chỉ có 3 trường cố định (problem / options / action_plan) — không hỗ trợ matrix, mindmap, hay bảng so sánh chi tiết.
- **Không tham chiếu tài liệu**: brainstormer không đọc file. Nếu cần context dữ liệu, dùng [Vaulter](./05_vaulter.md) song song.
- **Một session không share được**: chỉ owner xem được — không có collaborative brainstorming.
- **Compaction có thể mất nuance**: trên 10 message, lịch sử cũ bị tóm tắt — chi tiết tinh tế có thể mất khi resume sau nhiều lượt.
- **Không export ra file**: action plan chỉ tồn tại trong canvas. Copy thủ công vào doc/sheet để theo dõi.
