# The Resume Evaluator — Đánh giá CV ứng viên

> Đánh giá CV vs JD theo 6 tiêu chí, đưa quyết định INTERVIEW / CONDITIONAL / DO_NOT_INTERVIEW, kèm câu hỏi phỏng vấn gợi ý. Hỗ trợ batch tới 10 ứng viên với ranking.

---

## 1. Giới thiệu khả năng

**The Resume Evaluator làm gì?** Giúp HR / Hiring Manager **screen CV nhất quán** bằng AI:
- Chấm điểm theo 6 tiêu chí có trọng số.
- Đưa quyết định: nên phỏng vấn hay không.
- Phát hiện red flag, fit văn hoá, dấu hiệu CV viết bằng AI.
- Đề xuất câu hỏi phỏng vấn + reference check.
- Export DOCX báo cáo từng ứng viên.

**Khi nào dùng?**
- Bạn nhận 5–10 CV cho một vị trí, cần screen nhanh.
- Bạn muốn đánh giá khách quan, giảm bias.
- Bạn cần báo cáo ngắn cho hiring committee.

**Input**
- 1 hoặc nhiều CV (PDF, DOCX, TXT) — tối đa 10 / lần.
- Job description (text, paste hoặc gõ).

**Output**
- **Score card**: decision + overall score + confidence.
- **Radar chart** 6 tiêu chí.
- **Detailed breakdown**: red flags, culture fit, authenticity, interview questions, reference check prompts.
- **Báo cáo DOCX** tải về cho từng ứng viên.
- **Batch ranking**: nếu chấm nhiều CV, có bảng xếp hạng + skill gap analysis.

---

## 2. Giới hạn & điều kiện sử dụng

| Mục | Giới hạn |
| --- | --- |
| Định dạng CV | PDF, DOCX, TXT |
| CV mỗi batch | Tối đa **10** ứng viên |
| Độ dài CV & JD | Mỗi cái tối đa **50.000 ký tự** (truncate nếu vượt) |
| Xử lý batch | **Tuần tự**, không song song — 10 CV mất nhiều thời gian |
| Báo cáo | Format **DOCX** (không phải PDF, dù tên hàm có "pdf") |
| Cache | Không có — cùng CV chấm lại sẽ chạy lại từ đầu |
| Đăng nhập | SSO bắt buộc |

---

## 3. Cách sử dụng (Step-by-step)

### A. Đánh giá đơn lẻ (1 CV)

#### Bước 1 — Mở The Resume Evaluator
![Giao diện chính](./images/resume-01-home.png)
*Hình 1: JD ở panel trái, upload CV ở panel phải.*

#### Bước 2 — Nhập Job Description
Paste JD vào ô bên trái. Có thể paste cả phần requirement + responsibilities.

#### Bước 3 — Upload CV
Kéo-thả 1 file (PDF/DOCX/TXT).

#### Bước 4 — Bấm "Evaluate"
Agent parse CV, gửi cả CV + JD vào LLM với prompt evaluation chuyên dụng. Mất ~20–40 giây.

#### Bước 5 — Xem score card
- **Decision**: `INTERVIEW` / `CONDITIONAL` / `DO_NOT_INTERVIEW`.
- **Score**: 0–100.
- **Confidence**: High / Medium / Low.
- **Title match**: Direct / Adjacent / Off-title.

![Score card](./images/resume-02-scorecard.png)
*Hình 2: Score card chính với decision + score + confidence.*

#### Bước 6 — Xem breakdown chi tiết
- **Radar chart 6 tiêu chí**.
- **Red flags** với severity (High / Medium / Low).
- **Culture fit assessment**.
- **Authenticity** (xác suất CV viết bằng AI).
- **Suggested interview questions**.
- **Reference check prompts**.

![Detailed breakdown](./images/resume-03-breakdown.png)
*Hình 3: Radar chart + các block đánh giá nâng cao.*

#### Bước 7 — Download Report
Bấm "Download Report" → tải `.docx` báo cáo cho ứng viên này.

### B. Đánh giá batch (nhiều CV)

#### Bước 1 — Paste JD như trên
#### Bước 2 — Upload nhiều CV (tối đa 10)

![Batch upload](./images/resume-04-batch-upload.png)
*Hình 4: Danh sách 10 CV sẵn sàng.*

#### Bước 3 — Bấm "Evaluate All"
Agent chấm **tuần tự**. Progress bar hiển thị: `Đang chấm 4/10 — Nguyễn Văn A`.

#### Bước 4 — Xem bảng ranking
Ứng viên sắp xếp theo overall score giảm dần, kèm 1 strength + 1 concern key cho mỗi người.

![Ranking](./images/resume-05-ranking.png)
*Hình 5: Bảng xếp hạng 10 ứng viên.*

#### Bước 5 — Skill gap analysis
Block hiển thị các requirement của JD **không** ứng viên nào đáp ứng được — gợi ý cần mở rộng tuyển dụng hay điều chỉnh JD.

#### Bước 6 — Download
- Từng báo cáo riêng (mỗi ứng viên 1 DOCX).
- Hoặc ZIP toàn bộ + comparison summary.

---

## 4. Kịch bản minh hoạ (User Journeys)

### Kịch bản 1 — Screen nhanh 1 CV trước phone interview

**Bối cảnh:** Recruiter gửi CV `Nguyen_Van_A.pdf` cho vị trí Senior Data Engineer. Bạn muốn 2 phút screen trước khi gọi.

**Bước thực hiện:**
1. Mở Resume Evaluator → paste JD `"Senior Data Engineer — 5+ years Spark/Databricks, Python..."`.
2. Upload `Nguyen_Van_A.pdf`.
3. Bấm Evaluate → đợi 30 giây.
4. Nhận: **Decision: INTERVIEW**, Score: 82, Confidence: High, Title match: Direct.
5. Radar: Skills 85%, Experience 90%, Achievement 75%, Transferability 70%, Online 60%, Education 80%.
6. Red flags: 1 medium — `"Job hopping — 3 jobs in last 3 years"`.
7. Suggested questions: 5 câu, copy 2 câu vào script phone screen.
8. Gọi điện với context chuẩn bị sẵn.

---

### Kịch bản 2 — Batch screen 8 ứng viên

**Bối cảnh:** 8 CV cho vị trí Product Manager. Cần shortlist 3 cho on-site.

**Bước thực hiện:**
1. Paste JD.
2. Upload cả 8 CV.
3. Bấm Evaluate All → đợi ~5 phút (tuần tự).
4. Ranking trả về:
   - #1: 88 — INTERVIEW (Direct match, strong product cases)
   - #2: 81 — INTERVIEW (Adjacent — fintech background)
   - #3: 76 — INTERVIEW
   - #4–6: 60–72 — CONDITIONAL
   - #7–8: <60 — DO_NOT_INTERVIEW
5. Skill gap analysis: `"None of the candidates have B2B SaaS experience"` → cân nhắc mở rộng JD hoặc tìm pool khác.
6. Download ZIP báo cáo top 3.
7. Gửi cho hiring committee cùng comparison summary.

---

### Kịch bản 3 — Phát hiện CV viết bằng AI

**Bối cảnh:** CV có wording quá hoàn hảo, bạn nghi viết bằng ChatGPT.

**Bước thực hiện:**
1. Evaluate CV.
2. Vào block **Authenticity Assessment**:
   - AI-generation likelihood: **High (78%)**.
   - Specificity: Low — `"Phần lớn achievements là phrase chung chung, ít số liệu cụ thể"`.
   - Buzzword density: High.
   - Internal consistency: dates không khớp.
3. Quyết định: vẫn phỏng vấn nhưng dùng câu hỏi gợi ý focus vào "tell me about a specific time..." để verify.

---

## 5. Tips sử dụng

- **JD càng rõ, đánh giá càng đúng**: paste full JD (requirements + nice-to-have + responsibilities), không chỉ tiêu đề.
- **6 tiêu chí có trọng số cố định** (Skills 25%, Experience 20%, Achievement 20%, Education 15%, Transferability 10%, Online Presence 10%) — không tuỳ biến từ UI.
- **Score < 60 = DO_NOT_INTERVIEW**: nhưng đừng auto-reject — đọc breakdown để chắc agent không miss context (ví dụ CV ngắn vì junior, không phải vì thiếu).
- **High AI-generation likelihood KHÔNG = từ chối ngay** — chỉ là tín hiệu cần verify trong phỏng vấn.
- **Transferability score cao** với title không trực tiếp đáng để xem xét — agent đánh giá conceptual overlap.
- **Batch processing là tuần tự** — 10 CV có thể mất ~5 phút. Không refresh trang.
- **Báo cáo DOCX có thể edit** trong Word — có thể paste comment HR / recruiter trước khi gửi hiring committee.
- **Re-evaluate cùng CV sẽ ra điểm gần như nhau** nhưng không hoàn toàn deterministic — nếu cần consistency tuyệt đối, lưu báo cáo lần đầu thay vì chạy lại.

---

## 6. Hạn chế đã biết

- **Tối đa 10 CV / batch, tuần tự**: batch lớn (50+ CV) phải chia thành nhiều lần.
- **CV bị cắt ở 50.000 ký tự**: CV bình thường không vấn đề, nhưng portfolio CV của senior có thể bị truncate.
- **Báo cáo là DOCX, không phải PDF**: phải Export-as-PDF từ Word nếu cần PDF.
- **Không cache**: cùng CV chấm lại tốn LLM call mới.
- **Không tính toán bias** theo cách chính thức: agent cố giảm bias nhưng không có audit / metric chính thức về fairness.
- **Không hỗ trợ video resume / portfolio website**: chỉ đọc text trong file CV.
- **Trọng số 6 tiêu chí cố định**: không tuỳ chỉnh theo vai trò (vd. vai trò sales nên weight Achievement cao hơn — hiện chưa có).
- **Score có thể trôi nhẹ giữa các lần chạy** (LLM không 100% deterministic) — không dùng làm tiêu chí pháp lý.
- **Không tích hợp ATS** — phải tự upload thủ công.
