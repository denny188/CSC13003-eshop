# [BUG-FR03-05] Backend API không kiểm tra độ mạnh mật khẩu khi đặt lại mật khẩu

- **Feature (FR):** `FR-03: Đặt lại mật khẩu (Bước 2)` & `FR-01: Ràng buộc Mật khẩu`
- **Môi trường (Environment):** Backend API (`POST /api/reset-password`)
- **Mức độ nghiêm trọng (Severity):** **High** (Lỗ hổng Broken Authentication - Bỏ qua chính sách mật khẩu)
- **Mã Test Case liên quan:** `TC-FR03-DT-017`, `TC-FR03-BV-001`

---

### 1. Mô tả lỗi (Description)
Endpoint `POST /api/reset-password` hoàn toàn không có bất kỳ logic kiểm tra độ dài hay độ phức tạp của `newPassword`. Kẻ tấn công hoặc người dùng có thể gửi trực tiếp một chuỗi mật khẩu cực ngắn hoặc cực yếu (ví dụ: `"123"` hoặc `"abc"`) qua API và hệ thống vẫn cập nhật thành công với mã phản hồi 200 OK.

### 2. Các bước tái hiện (Steps to Reproduce)
1. Tạo mã reset token hợp lệ cho `test@eshop.com`.
2. Gửi request đặt lại mật khẩu với mật khẩu mới chỉ có 3 ký tự `"123"`:
   ```bash
   node -e "fetch('http://localhost:3000/api/reset-password', {
     method: 'POST',
     headers: {'Content-Type': 'application/json'},
     body: JSON.stringify({email: 'test@eshop.com', resetToken: '<valid_token>', newPassword: '123'})
   }).then(r => r.json()).then(console.log)"
   ```
3. Quan sát phản hồi từ server.

### 3. Kết quả mong đợi (Expected Result)
- Theo đặc tả **FR-03 / Bước 2**: *"Mật khẩu mới phải tuân thủ điều kiện như FR-01."*
- Theo **FR-01**: Mật khẩu tối thiểu 8 ký tự, có ít nhất 1 chữ hoa, 1 chữ thường, 1 chữ số và 1 ký tự đặc biệt trong `[@$!%*?&]`.
- Backend phải kiểm tra và từ chối với HTTP 400 Bad Request kèm thông báo lỗi phù hợp khi mật khẩu không đủ độ mạnh.

### 4. Kết quả thực tế (Actual Result)
- Server trả về HTTP 200 OK:
  ```json
  {"message":"Password reset successfully"}
  ```
- CSDL cập nhật mật khẩu thành `"123"`.

### 5. Đề xuất khắc phục (Recommendation)
Thêm bước validate mật khẩu phía server trước khi thực thi truy vấn UPDATE:
```javascript
const passwordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/;
if (!passwordRegex.test(newPassword)) {
  return res.status(400).json({ error: "Mật khẩu không thỏa mãn chính sách bảo mật" });
}
```
