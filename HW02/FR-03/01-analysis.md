# Phân tích Yêu cầu Kiểm thử: FR-03 (Quên mật khẩu & Đặt lại mật khẩu)

## 1. Trích dẫn Yêu cầu từ Đặc tả (Spec Excerpts)

### 1.1 Đặc tả chức năng (README.md - FR-03 & FR-01)
- **FR-03: Quên mật khẩu & Đặt lại mật khẩu (2 bước):**
  - **Bước 1 — Lấy mã OTP:**
    - Người dùng nhập địa chỉ Email đã đăng ký.
    - Hệ thống sinh mã OTP **6 chữ số ngẫu nhiên** và gửi qua Email (trong môi trường demo: hiển thị trực tiếp trên màn hình).
    - Giao diện phải hiển thị **chỉ báo bước (Step Indicator)** — ví dụ: "Bước 1 / 2".
    - Có nút **Quay lại đăng nhập**.
  - **Bước 2 — Đặt lại mật khẩu:**
    - Người dùng nhập OTP, Mật khẩu mới, và **Xác nhận mật khẩu mới**.
    - Mật khẩu mới phải tuân thủ điều kiện như FR-01.
    - Hai trường mật khẩu phải khớp nhau.
    - OTP chỉ hợp lệ cho email đã yêu cầu, không thể dùng cho email khác.
- **Ràng buộc mật khẩu (FR-01):**
  - Mật khẩu tối thiểu 8 ký tự.
  - Phải có ít nhất 1 chữ hoa, 1 chữ thường, 1 chữ số và 1 ký tự đặc biệt trong tập hợp: `@`, `$`, `!`, `%`, `*`, `?`, `&`.

### 1.2 Yêu cầu Giao diện & Form (README.md - FR-21, FR-22)
- Trường Email phải dùng `type="email"`.
- Trường Mật khẩu phải dùng `type="password"`.
- Các trường bắt buộc phải có ký hiệu `*` bên cạnh nhãn.
- Thông báo lỗi phải xuất hiện **trên** nút submit, không phải bên dưới.
- Form có từ 2 bước trở lên phải có Step Indicator rõ ràng.

### 1.3 Yêu cầu Bảo mật (README.md - SEC-07, SEC-01)
- **SEC-07:** OTP đặt lại mật khẩu phải đủ entropy (tối thiểu 6 chữ số), có thời hạn và vô hiệu hóa sau khi dùng.
- **SEC-01:** Mật khẩu không được lưu dưới dạng plaintext.

### 1.4 Đặc tả API (api_specification.md)
- `POST /api/forgot-password`: Body `{"email": "..."}` -> 200 OK `{"message": "Mã đặt lại mật khẩu đã được tạo", "resetToken": "123456"}`
- `POST /api/reset-password`: Body `{"email": "...", "resetToken": "...", "newPassword": "..."}` -> 200 OK `{"message": "Password reset successfully"}`

---

## 2. Bảng phân tích Biến đầu vào (Input Variables)

| Tên biến | Vị trí (UI / API) | Kiểu dữ liệu | Ràng buộc giá trị / Định dạng | Bắt buộc? | Nguồn đặc tả (Spec Ref) |
| :--- | :--- | :--- | :--- | :---: | :--- |
| `email` (Bước 1) | UI Input / API Body (`POST /api/forgot-password`) | Chuỗi ký tự (String) | Định dạng email hợp lệ (`user@domain.com`); phải tồn tại trong CSDL | Có | FR-03 Bước 1, FR-01, FR-22 |
| `email` (Bước 2) | Context / API Body (`POST /api/reset-password`) | Chuỗi ký tự (String) | Khớp đúng email đã gửi yêu cầu OTP ở Bước 1 | Có | FR-03 Bước 2 |
| `resetToken` (OTP) | UI Input / API Body (`POST /api/reset-password`) | Chuỗi chữ số (String) | Đúng 6 chữ số ngẫu nhiên (`0-9`); chưa hết hạn; chưa từng được sử dụng trước đó | Có | FR-03 Bước 1 & 2, SEC-07, api_spec 1.3 |
| `newPassword` | UI Input / API Body (`POST /api/reset-password`) | Chuỗi ký tự (String) | Độ dài $\ge 8$ ký tự; có ít nhất 1 chữ hoa, 1 chữ thường, 1 chữ số, 1 ký tự đặc biệt thuộc `[@$!%*?&]` | Có | FR-03 Bước 2, FR-01, FR-22 |
| `confirmNewPassword` | UI Input | Chuỗi ký tự (String) | Giá trị phải trùng khớp tuyệt đối với `newPassword` | Có | FR-03 Bước 2, FR-01 |

---

## 3. Danh mục Hành vi & Kết quả đầu ra cần kiểm tra (Expected Outputs & Behaviors)

1. **Bước 1 (Yêu cầu OTP):**
   - Khi email hợp lệ và đã đăng ký: Hệ thống sinh OTP 6 chữ số ngẫu nhiên; hiển thị Step Indicator "Bước 1/2"; có nút "Quay lại đăng nhập"; chuyển sang Bước 2 hoặc thông báo kèm mã OTP demo.
   - Khi email rỗng hoặc sai định dạng: Báo lỗi định dạng email trên client; không gửi request.
   - Khi email chưa đăng ký: Báo lỗi tài khoản không tồn tại; không tạo mã OTP.
2. **Bước 2 (Đặt lại mật khẩu):**
   - Khi OTP chính xác (6 số, đúng email yêu cầu) và Mật khẩu mới hợp lệ + Xác nhận mật khẩu khớp: Hệ thống cập nhật mật khẩu mới (được mã hóa theo SEC-01); vô hiệu hóa OTP (không dùng lại được theo SEC-07); hiển thị thông báo thành công; chuyển hướng về trang Đăng nhập.
   - Khi OTP không khớp hoặc OTP của email khác: Báo lỗi "Mã OTP không đúng hoặc không hợp lệ".
   - Khi OTP đã qua sử dụng hoặc hết hạn: Từ chối đặt lại mật khẩu (SEC-07).
   - Khi Mật khẩu mới không thỏa mãn chính sách độ mạnh (FR-01): Báo lỗi yêu cầu mật khẩu mạnh, chặn submit.
   - Khi Mật khẩu mới và Xác nhận mật khẩu không khớp nhau: Báo lỗi không khớp trên UI; không gửi request lên backend.
   - Giao diện Step 2 phải hiển thị Step Indicator "Bước 2/2" và nút Quay lại (FR-03, FR-22).

---

## 4. Các điểm mơ hồ / Khe hở đặc tả cần xác nhận (Ambiguities & Spec Gaps)

1. **Độ dài và định dạng OTP:**
   - Spec `FR-03` ghi rõ OTP ngẫu nhiên 6 chữ số (`SEC-07` ghi tối thiểu 6 chữ số).
   - *Cần xác nhận:* OTP chỉ chứa ký số `0-9` hay cho phép cả chữ cái? (Mặc định hiểu: 6 chữ số `0-9`).
2. **Thời hạn hiệu lực của OTP (Expiration time):**
   - Spec `SEC-07` quy định "có thời hạn" nhưng không nêu rõ bao nhiêu phút (5 phút hay 15 phút?).
3. **Trường `confirmNewPassword` ở mức API:**
   - Trong `api_specification.md`, endpoint `POST /api/reset-password` chỉ nhận `newPassword` (việc kiểm tra khớp mật khẩu diễn ra ở Frontend UI hay Backend có kiểm tra trường này không?).
4. **Giới hạn số lần thử nhập sai OTP (Brute-force limit):**
   - Spec không nêu rõ sau bao nhiêu lần nhập sai OTP thì khóa mã OTP đó hoặc hủy phiên reset password.
