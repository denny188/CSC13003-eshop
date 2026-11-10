# HW03 — Xác định Phạm vi Kiểm thử Giao diện & Độ khả dụng (Scope Selection)

- **Sinh viên thực hiện:** Nguyễn Lê Hữu Điền (`23120233`)
- **Email sinh viên:** `23120233@student.hcmus.edu.vn`
- **Nhánh làm việc:** `hw03-23120233`
- **Hệ thống thử nghiệm (SUT):** EShop (`http://localhost:5173` — Frontend Web, `http://localhost:3000` — Backend API)

---

## 1. Đối chiếu Phân chia Phạm vi Không trùng lặp trong Nhóm (§5 Scope Selection)

Theo quy định tại Mục 5 của đề bài: *"Within each group, ensure that your selection is not duplicated among the members of the group: no two members may choose the same primary screen for the GUI checklist or the same usability flow."*

| Thành viên | Phạm vi Task 1 (Màn hình chính cho GUI Checklist) | Phạm vi Task 2 (Luồng Usability End-to-End) | Trạng thái |
| :--- | :--- | :--- | :---: |
| **Xuân Thu** | - Chi tiết sản phẩm (`FR-06`)<br>- Giỏ hàng (`FR-07`)<br>- Thanh toán (`FR-08`) | **Luồng mua hàng:**<br>`Search → Product Detail → Cart → Coupon → Checkout` | Đã đăng ký trước |
| **Nguyễn Lê Hữu Điền (`23120233`)** | - **S1:** Đăng ký tài khoản (`FR-01` — `/register`)<br>- **S2:** Đăng nhập & Quên mật khẩu (`FR-02`, `FR-03` — `/login`, `/forgot-password`)<br>- **S3:** Hồ sơ cá nhân & Lịch sử đơn hàng (`FR-04`, `FR-11` — `/profile`) | **Luồng quản lý tài khoản & đơn hàng:**<br>`Tạo tài khoản mới (FR-01) → Đăng nhập (FR-02) → Cập nhật hồ sơ cá nhân (FR-04) → Kiểm tra lịch sử & trạng thái đơn hàng (FR-11)` | **Tách biệt 100%** |

---

## 2. Ánh xạ 4 Khía cạnh Giao diện (IA-01 → IA-04) với Đặc tả Hệ thống

| Mã IA | Khía cạnh Giao diện (Interface Aspect) | Tham chiếu Spec (`README.md`) | Trọng tâm Kiểm thử trên các màn hình S1, S2, S3 |
| :---: | :--- | :---: | :--- |
| **IA-01** | **General UI standards** *(Tiêu chuẩn giao diện chung)* | `FR-21` | Nhất quán ngôn ngữ tiếng Việt, màu sắc nút hành động (Submit xanh dương `#2563eb`, Hủy/Xóa đỏ `#ef4444`), định dạng tiền tệ `₫` có dấu phân cách hàng nghìn, duy nhất 1 thẻ `<h1>` mỗi trang, thứ tự nhấn phím `Tab` từ trên xuống dưới. |
| **IA-02** | **Forms** *(Biểu mẫu nhập liệu)* | `FR-22`, `FR-01`..`04` | Dấu `*` bên cạnh nhãn trường bắt buộc, `type="email"` cho ô Email, `type="password"` che ký tự cho ô Mật khẩu, thông báo lỗi hiển thị **phía trên** nút Submit, chỉ báo bước (**Step Indicator**) trên form 2 bước (`/forgot-password`), trường Xác nhận mật khẩu. |
| **IA-03** | **Navigation** *(Điều hướng)* | `FR-23` | Highlight trang hiện tại trên thanh Navbar, hiển thị badge số lượng sản phẩm cạnh link `"Giỏ hàng"`, nhãn nút đăng xuất là `"Đăng xuất"` (không dùng `"Thoát"`), có Breadcrumb trên các trang con, có nút `"Quay lại đăng nhập"` ở Bước 1 Quên mật khẩu. |
| **IA-04** | **Feedback / state** *(Phản hồi & Trạng thái)* | `FR-24`, `FR-10`, `FR-11` | Phản hồi trực quan (inline message/toast thay vì `alert()` chặn luồng), hộp thoại xác nhận trước khi hủy đơn hàng, trạng thái trống (**Empty State**) của lịch sử đơn hàng phải có icon/hình minh họa kèm thông điệp thân thiện, badge trạng thái đơn hàng dịch tiếng Việt và phân biệt màu sắc. |
