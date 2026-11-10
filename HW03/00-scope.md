# HW03 — Xác định Phạm vi Kiểm thử Giao diện & Độ khả dụng (Scope Selection)

- **Sinh viên thực hiện:** Nguyễn Lê Hữu Điền (`23120233`)
- **Email sinh viên:** `23120233@student.hcmus.edu.vn`
- **Nhánh làm việc:** `hw03-23120233`
- **Hệ thống thử nghiệm (SUT):** EShop (`http://localhost:5173` — Frontend Web, `http://localhost:3000` — Backend API)

---

## 1. Phạm vi Kiểm thử Đã Chọn (§5 Scope Selection)

### 1.1. Phạm vi Task 1 — Các màn hình kiểm thử GUI Checklist
Để đảm bảo độ bao phủ sâu rộng (>40 checklist items) qua cả 4 khía cạnh giao diện (`IA-01` → `IA-04`), phạm vi kiểm thử tập trung vào cụm màn hình **Xác thực, Tài khoản & Lịch sử đơn hàng** cùng thanh điều hướng chung:

| Mã màn hình | Tên màn hình | Đường dẫn (Route) | Thành phần mã nguồn (Component) | Yêu cầu đặc tả liên quan |
| :---: | :--- | :---: | :--- | :---: |
| **S1** | **Đăng ký tài khoản** *(Account Registration)* | `/register` | `frontend-web/src/pages/Register.jsx` | `FR-01`, `FR-21`, `FR-22`, `FR-23`, `FR-24` |
| **S2** | **Đăng nhập & Quên mật khẩu** *(Login & Forgot Password)* | `/login`<br>`/forgot-password` | `frontend-web/src/pages/Login.jsx`<br>`frontend-web/src/pages/ForgotPassword.jsx` | `FR-02`, `FR-03`, `FR-21`, `FR-22`, `FR-23`, `FR-24` |
| **S3** | **Hồ sơ cá nhân & Lịch sử đơn hàng** *(Profile & Order History)* | `/profile` | `frontend-web/src/pages/Profile.jsx` | `FR-04`, `FR-11`, `FR-21`, `FR-22`, `FR-23`, `FR-24` |
| **Common** | **Thanh điều hướng & Bố cục chung** *(Header / Navbar & Layout)* | Toàn trang | `frontend-web/src/App.jsx` | `FR-21`, `FR-23`, `FR-24` |

### 1.2. Phạm vi Task 2 — Luồng Đánh giá Độ khả dụng (Usability Evaluation Flow)
- **Luồng nghiệp vụ End-to-End:**
  `Tạo tài khoản mới (FR-01) → Đăng nhập (FR-02) → Cập nhật hồ sơ cá nhân: Họ tên, Số điện thoại, Địa chỉ giao hàng (FR-04) → Kiểm tra lịch sử & trạng thái đơn hàng (FR-11 / FR-24)`
- **Phương pháp & Công cụ đo lường:**
  - Thực hiện đánh giá có điều phối (*Moderated Usability Testing*) với **1 phiên chạy thử (Pilot)** và **7 người tham gia thật** ngoài lớp học.
  - Sử dụng thang đo chuẩn **SUS (System Usability Scale — 10 câu)** kết hợp bộ câu hỏi mở khảo sát 4 khía cạnh: *Clarity* (Độ rõ ràng), *Error Recovery* (Khả năng phục hồi sau lỗi), *Speed* (Tốc độ thao tác), và *Trust* (Độ tin cậy).

### 1.3. Phạm vi Task 3 — Kiểm thử Đa trình duyệt / Đa nền tảng (Cross-Browser / Cross-Platform)
- Thực thi bộ GUI Checklist trên **3 nền tảng** thông qua **BrowserStack / LambdaTest**:
  1. **Google Chrome** (Desktop)
  2. **Mozilla Firefox** (Desktop)
  3. **Apple Safari** (macOS)

---

## 2. Ánh xạ 4 Khía cạnh Giao diện (IA-01 → IA-04) với Đặc tả Hệ thống

| Mã IA | Khía cạnh Giao diện (Interface Aspect) | Tham chiếu Spec (`README.md`) | Trọng tâm Kiểm thử trên các màn hình S1, S2, S3 |
| :---: | :--- | :---: | :--- |
| **IA-01** | **General UI standards** *(Tiêu chuẩn giao diện chung)* | `FR-21` | Nhất quán ngôn ngữ tiếng Việt, màu sắc nút hành động (Submit xanh dương `#2563eb`, Hủy/Xóa đỏ `#ef4444`), định dạng tiền tệ `₫` có dấu phân cách hàng nghìn, duy nhất 1 thẻ `<h1>` mỗi trang, thứ tự nhấn phím `Tab` từ trên xuống dưới, độ tương phản màu sắc (WCAG AA), bố cục Responsive/Dark mode/RTL. |
| **IA-02** | **Forms** *(Biểu mẫu nhập liệu)* | `FR-22`, `FR-01`..`04` | Dấu `*` bên cạnh nhãn trường bắt buộc, `type="email"` cho ô Email, `type="password"` che ký tự cho ô Mật khẩu, thông báo lỗi hiển thị **phía trên** nút Submit, chỉ báo bước (**Step Indicator**) trên form 2 bước (`/forgot-password`), trường Xác nhận mật khẩu, liên kết `<label htmlFor>` và `<input id>`. |
| **IA-03** | **Navigation** *(Điều hướng)* | `FR-23` | Highlight trang hiện tại trên thanh Navbar, hiển thị badge số lượng sản phẩm cạnh link `"Giỏ hàng"`, nhãn nút đăng xuất là `"Đăng xuất"` (không dùng `"Thoát"`), có Breadcrumb trên các trang con, có nút `"Quay lại đăng nhập"` ở Bước 1 Quên mật khẩu, chỉ báo tiêu điểm `:focus-visible`. |
| **IA-04** | **Feedback / state** *(Phản hồi & Trạng thái)* | `FR-24`, `FR-10`, `FR-11` | Phản hồi trực quan (inline message/toast thay vì `alert()` chặn luồng), hộp thoại xác nhận trước khi hủy đơn hàng, trạng thái trống (**Empty State**) của lịch sử đơn hàng phải có icon/hình minh họa kèm thông điệp thân thiện, badge trạng thái đơn hàng dịch tiếng Việt và phân biệt màu sắc, khóa nút Submit khi đang xử lý. |
