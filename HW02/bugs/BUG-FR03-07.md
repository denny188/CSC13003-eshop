# [BUG-FR03-07] Giao diện Bước 2 thiếu hoàn toàn trường "Xác nhận mật khẩu mới"

- **Feature (FR):** `FR-03: Đặt lại mật khẩu (Bước 2)` & `FR-01: Yêu cầu Form`
- **Môi trường (Environment):** Frontend Web (`src/pages/ForgotPassword.jsx`)
- **Mức độ nghiêm trọng (Severity):** **Medium** (Thiếu trường dữ liệu bắt buộc theo đặc tả nghiệp vụ)
- **Mã Test Case liên quan:** `TC-FR03-DT-023`, `TC-FR03-DT-024`

---

### 1. Mô tả lỗi (Description)
Theo đặc tả, ở Bước 2 khi đặt lại mật khẩu, người dùng bắt buộc phải nhập Mật khẩu mới và Xác nhận mật khẩu mới để hệ thống kiểm tra sự trùng khớp trước khi cập nhật. Tuy nhiên trên giao diện form Bước 2, chỉ có một ô nhập duy nhất là "Mật khẩu mới", hoàn toàn không có ô "Xác nhận mật khẩu mới".

### 2. Các bước tái hiện (Steps to Reproduce)
1. Mở trang Đặt lại mật khẩu Bước 2 trên giao diện Web.
2. Quan sát các trường nhập liệu trong form:
   - Trường 1: Mã OTP (4 số)
   - Trường 2: Mật khẩu mới
   - Nút: Đặt lại mật khẩu & Nút Quay lại.
3. Hoàn toàn không tìm thấy trường "Xác nhận mật khẩu mới".

### 3. Kết quả mong đợi (Expected Result)
- Theo đặc tả **FR-03 / Bước 2**:
  - *"Người dùng nhập OTP, Mật khẩu mới, và **Xác nhận mật khẩu mới**."*
  - *"Hai trường mật khẩu phải khớp nhau."*
- Form phải có ô nhập "Xác nhận mật khẩu mới" (`confirmNewPassword`) và kiểm tra `newPassword === confirmNewPassword`.

### 4. Kết quả thực tế (Actual Result)
- Mã nguồn `ForgotPassword.jsx:63-97` hoàn toàn bỏ qua ô nhập xác nhận mật khẩu và logic kiểm tra so khớp.

### 5. Đề xuất khắc phục (Recommendation)
Bổ sung state `confirmPassword`, thêm ô nhập `type="password"` cho "Xác nhận mật khẩu mới" và thêm câu lệnh kiểm tra:
```javascript
if (newPassword !== confirmPassword) {
  alert("Mật khẩu xác nhận không khớp!");
  return;
}
```
