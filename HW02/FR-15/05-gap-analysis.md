# Phân tích Khoảng trống AI (AI Gap Analysis): FR-15 (Quản lý Sản phẩm)

> **Lưu ý:** Báo cáo này do **sinh viên trực tiếp sở hữu, đánh giá và hoàn thiện** nhằm đáp ứng mục tiêu chuẩn đầu ra Bloom-AI cấp độ G9.3 (Phân tích & Phản biện).

---

## 1. Các Ca Kiểm thử mà AI bỏ sót ban đầu (Missed Test Cases)
*(Đánh giá xem trong quá trình AI phân tích chức năng CRUD sản phẩm ở Step 2 & Step 3 có bỏ qua khía cạnh nào không?)*

- **Ca kiểm thử 1 (Gợi ý):** Kiểm tra tính hợp lệ của định dạng URL hình ảnh (`imageUrl` format validation): Nhập chuỗi không phải URL (ví dụ: `ftp://...`, chuỗi văn bản linh tinh, hoặc đường dẫn file nội bộ) xem hệ thống có kiểm tra định dạng và hiển thị ảnh bị lỗi không.
- **Ca kiểm thử 2 (Gợi ý):** Kiểm tra xóa sản phẩm khi đang có trong Giỏ hàng hoặc Đơn hàng (Integrity & Constraint check): Xóa sản phẩm $P$ rồi kiểm tra xem giỏ hàng của User hoặc đơn hàng lịch sử có bị crash hay mất dữ liệu không.
- **Ca kiểm thử bổ sung của sinh viên:**
  - *[Sinh viên bổ sung thêm các case phát hiện được]*

---

## 2. Các Lỗi (Bugs) mà AI có thể đã bỏ sót nếu chỉ nhìn ở một tầng
*(Đánh giá các khuyết tật mà AI có thể bị khuất tầm nhìn nếu không kiểm thử đa tầng)*

- **Lỗi bỏ sót tiềm ẩn (Gợi ý):**
  - **Lỗi Frontend đột biến mảng `fakeMassUpdatedProducts` (`BUG-FR15-05`):** Nếu AI chỉ chạy API test qua cURL/Postman mà không đọc mã nguồn hoặc test trực tiếp trên Web Admin, lỗi này sẽ hoàn toàn bị bỏ lọt! Bởi vì trên API backend chỉ cập nhật đúng 1 bản ghi `UPDATE products SET ... WHERE id = ?`, nhưng chính code JavaScript phía Frontend React lại cố tình `products.map(p => ({ ...p, name: productForm.name }))` làm hỏng giao diện Admin.
  - **Lỗi toàn bộ API CRUD không có bảo vệ xác thực (`BUG-FR15-06`):** Nếu chỉ thao tác trên giao diện Admin (nơi đã đăng nhập sẵn), người kiểm thử sẽ lầm tưởng các API này an toàn, mà không nhận ra các route này hoàn toàn là public API, cho phép bất kỳ ai gửi request sửa/xóa cơ sở dữ liệu.
- **Lỗi bổ sung của sinh viên:**
  - *[Sinh viên ghi nhận thêm]*

---

## 3. Lý do AI bỏ sót hoặc có hạn chế (Why the AI Missed It)

1. **Chất lượng câu lệnh (Prompt Quality):**
   - Nếu prompt không nhắc đến việc soi chiếu giữa hành vi State của Client React và Database Backend, AI sẽ có xu hướng chỉ tập trung vào tầng API mà bỏ quên các lỗi logic giao diện (UI state side effects).
2. **Hạn chế của công cụ AI (Tool Limitations):**
   - AI chỉ là mô hình ngôn ngữ tĩnh. Nếu không có khả năng đọc sâu từng dòng mã nguồn Frontend (`App.jsx`) và đồng thời kiểm tra backend (`server.js`), AI không thể tự động phát hiện được sự không đồng nhất giữa dữ liệu thực tế trong SQLite và dữ liệu hiển thị trên state React.
3. **Độ phức tạp và bẫy lỗi có chủ đích (Seeded Bugs):**
   - SUT có các lỗi được cài cắm tinh vi (như `fakeMassUpdatedProducts`), đòi hỏi người kiểm thử phải bám sát câu chữ của đặc tả (*"Khi Sửa một sản phẩm, chỉ sản phẩm đó bị thay đổi — các sản phẩm khác giữ nguyên"*) để kiểm tra chéo các sản phẩm không liên quan.

---

## 4. Những điều chỉnh của Sinh viên sau khi Review (What I Changed After Review)
*(Sinh viên tóm tắt các điểm bản thân đã can thiệp, phản biện hoặc điều chỉnh kết quả của AI)*

- Nhắc nhở AI chú trọng kiểm tra thuộc tính Update Isolation giữa các sản phẩm.
- Chỉ đạo kiểm tra các route API trực tiếp mà không kèm header Authorization để phát hiện triệt để lỗ hổng thiếu middleware xác thực.
- *[Sinh viên bổ sung thêm các nhận xét/chỉnh sửa của bản thân]*
