# Phân tích Khoảng trống AI (AI Gap Analysis): FR-03 (Quên mật khẩu & Đặt lại mật khẩu)

Báo cáo phân tích và phản biện năng lực AI trong quá trình kiểm thử tính năng FR-03, đáp ứng chuẩn đầu ra Bloom-AI cấp độ G9.3 (Phân tích & Đánh giá).

---

## 1. Các Ca Kiểm thử mà AI bỏ sót ban đầu (Missed Test Cases)

1. **Kiểm tra cơ chế giới hạn tấn công vét cạn (Rate limiting / Brute-force OTP):**
   - *Mô tả:* Thử nghiệm gửi request nhập sai mã OTP liên tiếp nhiều lần (quá 5 lần) xem hệ thống có tự động hủy phiên, khóa tạm thời hoặc vô hiệu hóa token để chống đoán mò hay không.
   - *Đánh giá:* Do đặc tả FR-03 và SEC-07 không nêu rõ số lần thử tối đa cho OTP (khác với FR-02 có quy định khóa sau 3 lần), AI ban đầu chỉ tập trung vào kiểm tra định dạng và tính hợp lệ của token mà chưa thiết kế ca kiểm thử tần suất gửi OTP.
2. **Kiểm tra tính nhạy cảm ký tự hoa/thường (Case sensitivity) của Email:**
   - *Mô tả:* Yêu cầu OTP cho email viết hoa/thường hỗn hợp (ví dụ: `Test.User@Eshop.Com`) và xác minh hệ thống có chuẩn hóa (lowercase) trước khi so khớp trong CSDL hay không.
3. **Kiểm tra thời gian sống của Token (OTP Expiration / TTL):**
   - *Mô tả:* Thử dùng mã OTP sau một khoảng thời gian chờ (ví dụ 10-15 phút) để xác nhận yêu cầu "có thời hạn" trong SEC-07.

---

## 2. Các Lỗi (Bugs) mà AI có thể đã bỏ sót nếu chỉ nhìn ở một tầng

1. **Lỗi lưu mật khẩu dạng bản rõ trong CSDL SQLite (`BUG-FR03-04`):**
   - *Phân tích:* Nếu chỉ thực hiện kiểm thử trên giao diện Web (Black-box UI testing), sau khi đặt lại mật khẩu thành công và đăng nhập được bình thường, người kiểm thử hoặc AI sẽ đánh giá ca kiểm thử là **PASS**. Lỗi vi phạm nghiêm trọng SEC-01 này chỉ bị phát hiện khi truy vấn trực tiếp vào tầng lưu trữ CSDL SQLite (`SELECT password FROM users`).
2. **Lỗi Backend hoàn toàn bỏ qua việc xác thực mật khẩu mạnh (`BUG-FR03-05`):**
   - *Phân tích:* Giao diện Web có cài đặt regex chặn mật khẩu yếu (dù regex bị sai). Nếu chỉ kiểm thử qua form Web mà không kiểm thử độc lập ở tầng Backend API bằng cURL/Postman, AI sẽ không thể phát hiện ra việc Backend chấp nhận cả mật khẩu `"123"` với HTTP 200 OK.

---

## 3. Lý do AI bỏ sót hoặc có hạn chế (Why the AI Missed It)

1. **Chất lượng câu lệnh (Prompt Quality):**
   - Khi prompt ban đầu chỉ yêu cầu "sinh ca kiểm thử theo đặc tả chức năng", AI có xu hướng bám sát bề mặt văn bản mô tả quy trình, ít khi chủ động mở rộng sang các kịch bản kiểm thử phi chức năng chuyên sâu như Rate limiting, Timing attack hoặc Replay attack nếu không được người hướng dẫn định hướng cụ thể.
2. **Hạn chế cố hữu của công cụ AI (Tool Limitations):**
   - AI là mô hình ngôn ngữ lớn hoạt động trên văn bản, không có khả năng tự cảm nhận trạng thái thực tế của hệ thống. Nếu không được cấp quyền thực thi các câu lệnh thanh tra CSDL (Database Inspection) và API call thực tế, AI chỉ có thể dự đoán hành vi dựa trên giả định.
3. **Độ phức tạp và khoảng trống của bản đặc tả (Spec Ambiguity):**
   - Bản đặc tả không xác định rõ thời gian hết hạn cụ thể của OTP (bao nhiêu phút) và không nhắc tới trường `confirmNewPassword` trong body của API, dẫn tới việc AI phải đặt câu hỏi làm rõ thay vì có thể tự động sinh kiểm thử hoàn chỉnh ngay từ đầu.

---

## 4. Những điều chỉnh của Sinh viên sau khi Review (What I Changed After Review)

- **Chuẩn hóa quy trình thiết kế kiểm thử:** Yêu cầu AI áp dụng nghiêm ngặt nguyên tắc *single-fault* trong Domain Testing và bổ sung khảo sát các điểm biên số học (Leading zeros: `012345`) trong BVA để tránh lỗi ép kiểu dữ liệu chuỗi sang số.
- **Mở rộng phạm vi kiểm thử sang tầng CSDL:** Yêu cầu AI không chỉ dừng lại ở mã phản hồi HTTP 200 mà phải kiểm tra trực tiếp bảng `users` trong SQLite, qua đó phát hiện thành công lỗ hổng mật khẩu plaintext (`BUG-FR03-04`).
- **Phản biện regex kiểm tra mật khẩu:** Nhận diện và chỉ ra sự sai lệch trong mã nguồn regex của Frontend Web khi dùng `(?=.*\s)` (đòi hỏi dấu cách) thay vì kiểm tra ký tự đặc biệt theo đúng yêu cầu FR-01.
