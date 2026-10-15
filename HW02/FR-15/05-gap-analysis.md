# Phân tích Khoảng trống AI (AI Gap Analysis): FR-15 (Quản lý Sản phẩm)

Báo cáo phân tích và phản biện năng lực AI trong quá trình kiểm thử tính năng CRUD sản phẩm FR-15, đáp ứng chuẩn đầu ra Bloom-AI cấp độ G9.3 (Phân tích & Đánh giá).

---

## 1. Các Ca Kiểm thử mà AI bỏ sót ban đầu (Missed Test Cases)

1. **Kiểm tra định dạng và tính hợp lệ của đường dẫn ảnh (`imageUrl` format validation):**
   - *Mô tả:* Nhập chuỗi ký tự không phải URL (ví dụ: chuỗi văn bản thuần túy, giao thức không an toàn, hoặc đường dẫn file hệ điều hành) để xem hệ thống có kiểm tra định dạng URL trước khi lưu và hiển thị hình ảnh có bị vỡ giao diện hay không.
   - *Đánh giá:* AI ban đầu chỉ tập trung vào 3 biến bắt buộc trong đặc tả (`name`, `price`, `category_id`) mà xem nhẹ các trường tùy chọn như `imageUrl` và `description`.
2. **Kiểm tra ràng buộc toàn vẹn khi xóa sản phẩm (Referential Integrity Check):**
   - *Mô tả:* Thực hiện xóa một sản phẩm đang nằm trong giỏ hàng của người dùng hoặc đã có trong chi tiết của một đơn hàng lịch sử, kiểm tra xem CSDL có bị lỗi xung đột khóa ngoại hoặc gây crash trang người dùng hay không.

---

## 2. Các Lỗi (Bugs) mà AI có thể đã bỏ sót nếu chỉ nhìn ở một tầng

1. **Lỗi đột biến State hàng loạt trên giao diện Web Admin (`BUG-FR15-05`):**
   - *Phân tích:* Đây là lỗi nguy hiểm nhất trong SUT. Nếu người kiểm thử chỉ chạy kiểm thử API bằng cURL/Postman hoặc viết test script tự động trên backend, API `PUT /api/products/:id` hoạt động hoàn toàn chính xác (chỉ cập nhật đúng 1 dòng trong SQLite). Tuy nhiên, trên giao diện Web Admin, đoạn mã React cố tình sử dụng `fakeMassUpdatedProducts = products.map(...)` khiến toàn bộ danh sách sản phẩm hiển thị trên màn hình đều bị đổi tên theo! Lỗi này hoàn toàn bị bỏ lọt nếu chỉ kiểm thử đơn tầng ở backend.
2. **Lỗi các API CRUD sản phẩm hoàn toàn mở cho công chúng (`BUG-FR15-06`):**
   - *Phân tích:* Khi kiểm thử thủ công trên Web Admin, do người dùng đã đăng nhập tài khoản Admin từ trước nên các thao tác Thêm/Sửa/Xóa đều diễn ra trơn tru. Chỉ khi bóc tách kiểm thử độc lập ở tầng API và cố tình loại bỏ Header `Authorization`, ta mới phát hiện toàn bộ các route quản trị sản phẩm không hề có bất kỳ lớp bảo vệ nào.

---

## 3. Lý do AI bỏ sót hoặc có hạn chế (Why the AI Missed It)

1. **Chất lượng câu lệnh (Prompt Quality):**
   - Nếu câu lệnh không yêu cầu kiểm tra chéo tác động phụ giữa Client State và Database State, AI thường sẽ xem nhẹ việc kiểm tra giao diện sau khi API trả về kết quả thành công.
2. **Hạn chế cố hữu của công cụ AI (Tool Limitations):**
   - AI LLM không tự giác chạy đồng thời cả giao diện Frontend và backend API trừ khi được người hướng dẫn định hình phương pháp kiểm thử toàn diện (End-to-End Multilayer Testing).
3. **Đặc thù của các bẫy lỗi cài cắm (Seeded Bugs):**
   - Những bẫy lỗi như `fakeMassUpdatedProducts` được thiết kế có chủ đích nhằm đánh lừa các công cụ kiểm thử tự động chỉ kiểm tra mã phản hồi HTTP 200 mà không kiểm tra trạng thái của các đối tượng xung quanh.

---

## 4. Những điều chỉnh của Sinh viên sau khi Review (What I Changed After Review)

- **Nhấn mạnh yêu cầu Update Isolation:** Trực tiếp yêu cầu AI thiết kế ca kiểm thử `TC-FR15-DT-011` để kiểm tra thông tin của các sản phẩm lân cận khi sửa một sản phẩm bất kỳ, từ đó phát hiện thành công lỗi `BUG-FR15-05`.
- **Kiểm tra tính an toàn của API CRUD:** Yêu cầu gửi request không kèm token và request với token người dùng thường để phát hiện lỗ hổng thiếu middleware xác thực `BUG-FR15-06`.
