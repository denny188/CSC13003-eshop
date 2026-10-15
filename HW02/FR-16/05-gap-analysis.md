# Phân tích Khoảng trống AI (AI Gap Analysis): FR-16 (Import Sản phẩm từ CSV)

Báo cáo phân tích và phản biện năng lực AI trong quá trình kiểm thử tính năng Import sản phẩm từ file CSV (FR-16), đáp ứng chuẩn đầu ra Bloom-AI cấp độ G9.3 (Phân tích & Đánh giá).

---

## 1. Các Ca Kiểm thử mà AI bỏ sót ban đầu (Missed Test Cases)

1. **Kiểm tra cơ chế phân tách chuỗi CSV theo chuẩn RFC 4180 tại tầng Client:**
   - *Mô tả:* Khi người dùng nhập mô tả sản phẩm chứa dấu phẩy được bọc trong dấu ngoặc kép `""` (ví dụ: `"Laptop màn hình 15.6 inch, RAM 16GB, SSD 512GB"`), AI ban đầu chỉ giả định định dạng CSV hợp lệ nói chung mà không trực tiếp kiểm tra xem bộ parser viết bằng tay ở frontend có bị lỗi chia cột sai hay không.
   - *Đánh giá:* Nếu không kiểm tra kỹ mã nguồn giao diện `App.jsx`, người kiểm thử sẽ không nhận ra việc tách chuỗi bằng `line.split(",")` gây đảo lộn vị trí dữ liệu của toàn bộ các cột phía sau.
2. **Kiểm tra trạng thái CSDL thực tế khi phát hiện dòng lỗi ở các vị trí khác nhau (Database Partial Insert Check):**
   - *Mô tả:* Kiểm thử một batch gồm nhiều dòng, trong đó cố tình đặt dòng lỗi ở vị trí cuối cùng ($pos = N$), sau đó truy vấn trực tiếp CSDL SQLite để đếm số lượng dòng được thêm mới.
   - *Đánh giá:* AI thường có xu hướng chỉ nhìn vào phản hồi HTTP 200 hoặc mảng `errors` trả về từ API. Việc truy vấn trực tiếp CSDL giúp chứng minh rõ ràng lỗi vi phạm tính nguyên tử giao dịch (Atomicity) khi các dòng hợp lệ trước đó đã bị commit vĩnh viễn vào CSDL.

---

## 2. Các Lỗi (Bugs) mà AI có thể đã bỏ sót nếu chỉ nhìn ở một tầng

1. **Lỗi Trình phân tích CSV Client-side không tuân thủ RFC 4180 (`BUG-FR16-01`):**
   - *Phân tích:* Nếu chỉ kiểm thử độc lập ở tầng Backend API (gửi payload JSON đã được format sẵn), API sẽ nhận đúng các trường và chèn vào CSDL thành công. Chỉ khi kiểm thử tích hợp từ file CSV trên Web Admin, ta mới phát hiện phía client tự động parse CSV bằng `line.split(",")` khiến dữ liệu gửi lên API bị méo mó hoàn toàn.
2. **Lỗi Vi phạm Quy tắc Giao dịch Nguyên tử All-or-Nothing (`BUG-FR16-04`):**
   - *Phân tích:* API trả về thông báo rất dễ gây hiểu nhầm: `Import hoàn tất: 2/3 sản phẩm được thêm` và kèm mảng lỗi `errors: ['Hàng 3: ...']`. Nếu chỉ kiểm tra status code HTTP 200 và thông báo có hiển thị lỗi, người kiểm thử có thể lầm tưởng tính năng đã xử lý đúng nghiệp vụ báo cáo. Tuy nhiên, đối chiếu với đặc tả FR-16: *"Nếu có lỗi ở bất kỳ dòng nào, toàn bộ import phải được rollback (giao dịch nguyên tử — all-or-nothing)"*, việc CSDL lưu lại 2 sản phẩm đầu là một lỗi vi phạm nghiêm trọng về tính toàn vẹn dữ liệu.
3. **Lỗ hổng Phân quyền tại Admin API (`BUG-FR16-05`):**
   - *Phân tích:* Do API có middleware `authenticateToken`, nếu kiểm thử bằng tài khoản vãng lai (không token), hệ thống từ chối với HTTP 401. Nhưng nếu không thử nghiệm với Token của tài khoản `user` bình thường, ta sẽ bỏ sót việc API này hoàn toàn thiếu bước kiểm tra vai trò Quản trị viên (`role === 'admin'`).

---

## 3. Lý do AI bỏ sót hoặc có hạn chế (Why the AI Missed It)

1. **Chất lượng câu lệnh (Prompt Quality):**
   - Các câu lệnh kiểm thử chung chung thường chỉ yêu cầu "kiểm tra upload file CSV" mà không nhắc nhở AI phải đối chiếu từng câu chữ trong đặc tả, đặc biệt là cụm từ *"giao dịch nguyên tử — all-or-nothing"* và *"RFC 4180"*.
2. **Hạn chế cố hữu của công cụ AI (Tool Limitations):**
   - AI có xu hướng mặc định rằng khi API trả về HTTP 200 kèm danh sách lỗi thì đó là hành vi "thông minh" (xử lý chịu lỗi từng phần), dẫn đến việc đánh giá nhầm kết quả nếu không có sự đối chiếu khắt khe với yêu cầu nghiệp vụ bắt buộc rollback.
3. **Sự chia cắt giữa Client và Server (Architecture Disconnect):**
   - Hệ thống không tải file trực tiếp lên server (multipart/form-data) mà lại đọc file trên trình duyệt rồi gửi mảng JSON lên backend. Sự phân chia kiến trúc này khiến việc kiểm thử đơn lẻ từng tầng dễ bỏ lọt lỗi chuyển đổi dữ liệu.

---

## 4. Những điều chỉnh của Sinh viên sau khi Review (What I Changed After Review)

- **Bổ sung ca kiểm thử cú pháp RFC 4180:** Yêu cầu AI đưa trường mô tả chứa dấu phẩy trong cặp nháy kép vào bộ test case (`TC-FR16-DT-002`) để kiểm tra bộ parser của Frontend.
- **Thiết kế ca kiểm thử All-or-Nothing có đo lường CSDL:** Bổ sung ca kiểm thử `TC-FR16-DT-009` và `TC-FR16-BV-006` với cơ chế đếm số lượng bản ghi trong SQLite trước và sau khi gửi batch có lỗi, từ đó vạch trần việc thiếu `BEGIN TRANSACTION / ROLLBACK`.
- **Kiểm tra phân quyền đa tầng:** Yêu cầu gửi request với cả token User thường và token Admin để phát hiện lỗ hổng `BUG-FR16-05`.
