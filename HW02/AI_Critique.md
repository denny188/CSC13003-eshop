# Báo cáo Phê bình Năng lực AI (AI Critique Report)

- **Sinh viên thực hiện:** Nguyễn Lê Hữu Điền (MSSV: 23120233)
- **Hệ thống thử nghiệm:** EShop SUT (Backend Node.js/SQLite, Frontend Web React, Web Admin, Frontend Mobile React Native)
- **Công cụ AI sử dụng:** Antigravity (Gemini 3.8 Flash)
- **Độ dài bài luận:** ~275 từ (Đáp ứng chuẩn quy định 200–300 từ của đề bài)

---

### Bài luận Phê bình & Đánh giá Năng lực AI (AI Critique)

Trong quá trình thực hiện bài tập kiểm thử toàn diện trên hệ thống EShop, công cụ AI đã bộc lộ những hạn chế rõ rệt về tính thiên kiến và sự phụ thuộc vào bề mặt văn bản. Điểm sai sót lớn nhất của AI xuất phát từ **thiên kiến chức năng đơn tầng (Functional Bias)**: AI có xu hướng mặc định rằng khi một API phản hồi HTTP 200 kèm thông báo thành công hoặc mảng lỗi thì hệ thống đã vận hành đúng. Điển hình tại tính năng FR-16 (Import CSV), AI suýt bỏ qua lỗi vi phạm nguyên tắc giao dịch nguyên tử All-or-Nothing (`BUG-FR16-04`) khi thấy API trả về danh sách lỗi từng dòng, trong khi thực tế dữ liệu dở dang đã bị commit vĩnh viễn vào SQLite. Tương tự tại FR-15 và Mobile FR-07, các bẫy lỗi cài cắm tinh vi như đột biến state hàng loạt (`fakeMassUpdatedProducts` - `BUG-FR15-05`) hay tự động cộng dồn số lượng (`parsed + 1` - `BUG-MOB07-02`) hoàn toàn qua mặt được AI nếu không có sự thanh tra trực tiếp vào mã nguồn component và tương tác người dùng.

AI thất bại trong việc phát hiện các vấn đề trên là do mô hình ngôn ngữ lớn chỉ suy luận trên văn bản tĩnh, thiếu khả năng tự giác kết nối đa tầng giữa phản hồi mạng, trạng thái giao diện và tính toàn vẹn của cơ sở dữ liệu. 

Nguyên tắc cốt lõi tôi đúc kết được khi cộng tác với AI là: **AI chỉ là một "trợ lý có kỷ luật" (Disciplined Assistant), tuyệt đối không phải là một chiếc hộp đen tự động hóa (Black Box)**. Người kỹ sư kiểm thử bắt buộc phải giữ vai trò chủ đạo: trực tiếp định hướng chiến lược phân tích, áp dụng khắt khe các kỹ thuật kiểm thử hình thức (Domain Testing, BVA), và chủ động truy vấn chéo dữ liệu để phát hiện những sai lệch mà AI bỏ sót.
