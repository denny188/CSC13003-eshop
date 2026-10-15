# Phân tích Khoảng trống AI (AI Gap Analysis): FR-03

> **Lưu ý quan trọng:** Theo quy định tại `SKILL.md` và hướng dẫn bài tập HW02, báo cáo này do **sinh viên trực tiếp sở hữu, đánh giá và hoàn thiện** nhằm thể hiện năng lực Bloom-AI mức G9.3 (Phân tích & Đánh giá). Dưới đây là khung sườn (scaffold) kèm gợi ý tham khảo.

---

## 1. Các Ca Kiểm thử mà AI bỏ sót ban đầu (Missed Test Cases)
*(Sinh viên đối chiếu xem trong quá trình AI sinh test case ở Step 2 và Step 3, AI có bỏ sót trường hợp nào không?)*

- **Ca kiểm thử 1 (Gợi ý):** Kiểm tra cơ chế Rate limiting / Brute-force: Thử nhập sai mã OTP liên tiếp nhiều lần (ví dụ quá 5 lần) xem hệ thống có khóa phiên hay hủy token không (đặc tả không ghi rõ, AI ban đầu chưa sinh ca kiểm tra này).
- **Ca kiểm thử 2 (Gợi ý):** Kiểm tra tính nhạy cảm hoa/thường (Case sensitivity) của địa chỉ email (ví dụ: yêu cầu OTP bằng `Test@Eshop.Com` và đăng nhập bằng `test@eshop.com`).
- **Ca kiểm thử bổ sung của sinh viên:**
  - *[Sinh viên tự điền bổ sung thêm các case phát hiện được]*

---

## 2. Các Lỗi (Bugs) mà AI có thể đã bỏ sót nếu chỉ nhìn ở một tầng
*(Đánh giá các khuyết tật mà AI có thể bị khuất tầm nhìn nếu không kiểm thử đa tầng Web + API + Database)*

- **Lỗi bỏ sót tiềm ẩn (Gợi ý):**
  - Nếu chỉ test trên giao diện Web mà không kiểm tra tầng Database qua SQLite, AI sẽ không thể biết mật khẩu bị lưu plaintext (`BUG-FR03-04`).
  - Nếu chỉ test trên giao diện Web mà không test trực tiếp API bằng cURL/Postman, AI sẽ không phát hiện được lỗ hổng Backend hoàn toàn không validate độ dài/độ phức tạp của mật khẩu (`BUG-FR03-05`).
- **Lỗi bổ sung của sinh viên phát hiện:**
  - *[Sinh viên ghi nhận thêm các lỗi khác nếu có]*

---

## 3. Lý do AI bỏ sót hoặc có hạn chế (Why the AI Missed It)
*(Phân tích nguyên nhân gốc rễ theo 3 khía cạnh)*

1. **Chất lượng câu lệnh (Prompt Quality):**
   - Ban đầu prompt nếu chỉ yêu cầu "sinh test case theo spec" thì AI sẽ có xu hướng bám sát bề mặt văn bản mô tả chức năng, ít chủ động mở rộng sang các góc khuất bảo mật chuyên sâu (như timing attack, brute-force OTP, replay attack).
2. **Hạn chế của công cụ AI (Tool Limitations):**
   - AI LLM là mô hình sinh ngôn ngữ, không có trạng thái thực thi trực tiếp nếu không được trang bị môi trường terminal/browser để chạy thực tế. Nếu không chạy câu lệnh SQLite thực tế, AI không thể "đoán" được mã nguồn lưu plaintext hay bcrypt.
3. **Độ phức tạp và sự thiếu sót của bản đặc tả (Feature & Spec Ambiguity):**
   - Bản đặc tả không nêu thời gian hết hạn cụ thể của OTP (bao nhiêu phút) và không nhắc tới trường `confirmNewPassword` trong API body, khiến AI phải đặt câu hỏi làm rõ ở Step 1 thay vì tự suy diễn.

---

## 4. Những điều chỉnh của Sinh viên sau khi Review (What I Changed After Review)
*(Sinh viên tóm tắt các điểm bản thân đã can thiệp, phản biện hoặc điều chỉnh kết quả của AI)*

- Chuẩn hóa lại các prompt chỉ thị để AI bám sát quy tắc *single-fault* và ma trận độ phủ.
- Yêu cầu AI kiểm tra trực tiếp CSDL SQLite để xác minh yêu cầu bảo mật SEC-01 (không đoán mò qua phản hồi API).
- *[Sinh viên bổ sung thêm các nhận xét/chỉnh sửa của bản thân]*
