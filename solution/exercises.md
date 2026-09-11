# K4 — Ngày 1: Bài Tập & Phản Ánh
## Khám Phá LLM API | Phiếu Thực Hành

**Thời lượng:** 4 tiếng
**Cách làm:** Trả lời từng câu ngay sau khi hoàn thành block tương ứng —
đừng để dồn hết về cuối buổi. Thay dòng `*Câu trả lời của bạn*` bằng câu
trả lời thật (chấm tự động sẽ đếm số câu đã trả lời).

---

## Block 1 — API Cơ Bản (trả lời sau Checkpoint 1)

### Câu 1.1 — Độ nhạy của temperature
Gọi `call_openai` với temperature 0.0, 0.5, 1.0 và 1.5 dùng prompt
**"Hãy kể cho tôi một sự thật thú vị về Việt Nam."**

**Bạn nhận thấy quy luật gì qua bốn phản hồi?** (2–3 câu)
> Temperature càng cao phản hồi càng dài và trang trí nhiều hơn (nhiều markdown/bold, liệt kê theo mục), còn temperature 0.0 cho câu trả lời ngắn, cấu trúc đơn giản và lặp lại ổn định giữa các lần gọi. Trên endpoint Gemini em dùng, temperature 1.5 còn đổi hẳn cách mở bài (giật tít cảm thán) trong khi 0.0 mở bài trung tính, đúng xu hướng low-temp = nhất quán, high-temp = đa dạng.

### Câu 1.2 — Chọn temperature cho sản phẩm
**Bạn sẽ đặt temperature bao nhiêu cho chatbot hỗ trợ khách hàng, và tại sao?**
> Em đặt temperature khoảng 0.1–0.3. Chatbot CSKH cần trả lời nhất quán, đúng chính sách và dễ kiểm thử hồi quy; temperature thấp giảm nguy cơ bịa thông tin và giữ giọng điệu ổn định, trong khi tính sáng tạo không phải ưu tiên của kênh này.

### Câu 1.3 — Đánh đổi chi phí
Kịch bản: 10.000 người dùng hoạt động mỗi ngày, mỗi người gọi API 3 lần,
mỗi lần trung bình ~350 token đầu ra.

**Ước tính GPT-4o đắt hơn GPT-4o-mini bao nhiêu lần cho workload này? Nêu một
trường hợp GPT-4o xứng đáng với chi phí và một trường hợp nên dùng mini:**
> Workload = 10.000 × 3 × 350 = 10,5 triệu output token/ngày. Theo bảng giá lab: GPT-4o tốn 10.500 × 0,010 = ~105 USD/ngày (~3.150 USD/tháng), mini tốn 10.500 × 0,0006 = ~6,3 USD/ngày (~189 USD/tháng), tức GPT-4o đắt gấp ~16,7 lần. Em dùng GPT-4o cho tác vụ suy luận phức tạp (tư vấn pháp lý, chẩn đoán sự cố khó) và dùng mini cho phân loại, tóm tắt, FAQ lặp lại. Thực đo ở Block 1 cũng cho thấy mini nhanh hơn hẳn (0,86s so với 3,66s) nên còn lợi cả về latency.

---

## Block 2 — System Prompt & Token (trả lời sau Checkpoint 2)

### Câu 2.1 — Sức mạnh của persona
Gọi `chat_with_system_prompt` hai lần với cùng câu hỏi
**"Giải thích blockchain là gì?"** nhưng hai system prompt khác nhau:
- "Bạn là giáo viên tiểu học, giải thích thật đơn giản cho trẻ 8 tuổi."
- "Bạn là chuyên gia tài chính, trả lời chuyên sâu bằng thuật ngữ kỹ thuật."

**Hai phản hồi khác nhau như thế nào (độ dài, từ vựng, ví dụ)? System prompt
ảnh hưởng đến hành vi model ra sao?** (3–4 câu)
> Persona giáo viên tiểu học mở bài bằng "Chào con!" và hứa giải thích đơn giản, câu ngắn, ví dụ đời thường; persona chuyên gia tài chính mở bài bằng thuật ngữ "Settlement Finality", hợp đồng thông minh, câu dài và dày khái niệm. Cùng một câu hỏi nhưng từ vựng, độ dài và giọng điệu khác hẳn, chứng tỏ system prompt đóng vai trò "chỉ thị đạo diễn" chi phối toàn bộ phản hồi phía sau.

### Câu 2.2 — tiktoken vs đếm từ
Chọn một đoạn văn tiếng Việt ~100 từ. So sánh số token theo `count_tokens`
(tiktoken) với ước lượng `số từ / 0.75` mà Part 1 đã dùng.

**Hai con số chênh nhau bao nhiêu phần trăm? Vì sao tiếng Việt thường tốn
nhiều token hơn tiếng Anh cùng độ dài?**
> Đoạn văn 123 từ của em: `count_tokens` trả 132 (model Gemini không có trong tiktoken nên rơi về fallback `len(text)//4`), còn ước lượng Part 1 `số từ/0.75` cho 164 — chênh ~19,5%. Tiếng Việt thường tốn nhiều token hơn vì bộ mã hóa BPE được huấn luyện chủ yếu trên tiếng Anh: chữ có dấu và từ đa âm tiết bị chẻ thành nhiều mảnh byte/token nhỏ, nên cùng một ý thì tiếng Việt "đắt token" hơn.

---

## Block 3 — Streaming & Độ Bền (trả lời sau Checkpoint 3)

### Câu 3.1 — Trải nghiệm người dùng với streaming
**Streaming quan trọng nhất trong trường hợp nào, và khi nào thì
non-streaming lại phù hợp hơn?** (1 đoạn văn)
> Streaming quan trọng nhất ở chatbot hội thoại thời gian thực vì nó giảm Time to First Token: người dùng thấy chữ hiện dần sau vài trăm ms thay vì chờ cả câu trả lời dài, đúng bài học latency và trải nghiệm trong lab. Ngược lại, non-streaming phù hợp hơn cho tác vụ batch, API nền hoặc khi cần JSON có cấu trúc hoàn chỉnh để parse, vì lúc đó không có người đọc trực tiếp và code cần toàn vẹn response trước khi xử lý.

### Câu 3.2 — Vì sao backoff theo cấp số nhân?
**So với delay cố định (ví dụ luôn chờ 1 giây), exponential backoff có lợi
thế gì khi API bị quá tải? Điều gì xảy ra nếu hàng nghìn client cùng retry
với delay cố định giống nhau?**
> Exponential backoff giãn các lần thử lại ra (0,1s → 0,2s → 0,4s...) nên giảm áp lực lên server đang nghẽn và cho nó thời gian hồi phục, đồng thời tăng xác suất lần thử sau rơi vào lúc server đã khỏe. Nếu hàng nghìn client cùng retry với delay cố định giống nhau, chúng sẽ đồng loạt ập lại theo nhịp (thundering herd) và đánh sập server lần nữa; thực tế em cũng gặp 429 trên free-tier Gemini và phải chờ ~44s, đúng trường hợp backoff phát huy tác dụng.

---

## Block 4 — Mini-Project (trả lời sau Checkpoint 4)

### Câu 4.1 — Thiết kế persona
**Bạn chọn persona gì cho trợ lý của mình? Viết lại system prompt đó và giải
thích 1–2 lựa chọn từ ngữ quan trọng trong prompt (ví dụ: vì sao yêu cầu
"trả lời ngắn gọn", vì sao chỉ định ngôn ngữ...):**
> Persona của em: "Bạn là trợ giảng thân thiện của khóa AI, trả lời ngắn gọn bằng tiếng Việt, giải thích kèm đúng một ví dụ thực tế." Em chọn "ngắn gọn" để chặn câu trả lời lan man và tiết kiệm token output (output đắt gấp 4 lần input), và chỉ định "bằng tiếng Việt" vì model đa ngữ mặc định hay trả lẫn tiếng Anh như em đã thấy ở Block 1.

### Câu 4.2 — Hạn chế & cải thiện
**Trợ lý của bạn hiện có hạn chế lớn nhất là gì (ví dụ: history chỉ 3 lượt,
không có bộ nhớ dài hạn, không kiểm duyệt nội dung...)? Đề xuất một cải
thiện cụ thể và mô tả ngắn cách triển khai:**
> Hạn chế lớn nhất là history chỉ giữ 3 lượt gần nhất nên trợ lý mất trí nhớ dài hạn (ví dụ quên tên người dùng đã nói ở lượt 1) và không có kiểm duyệt nội dung. Cải thiện cụ thể: thêm bộ nhớ tóm tắt — sau mỗi lượt, gọi model tóm tắt history đã bị cắt thành 2–3 câu lưu trong biến `summary` rồi chèn vào sau system prompt, vừa giữ ngữ cảnh dài vừa không tăng token input vô hạn.

---

## Danh Sách Kiểm Tra Nộp Bài

- [ ] `python grade.py` — xem điểm tự động, mục tiêu ≥ 75/100
- [ ] Cả 4 checkpoint pytest đều pass
- [ ] Tất cả 9 câu trong file này đã được trả lời
- [ ] Đã copy bài làm vào folder `solution/`, push lên fork và dán link trên trang bài Lab ở VLearn trước 23:59 ngày 11/09/2026
