# [BUG-FR03-04] Mật khẩu người dùng bị lưu trữ dưới dạng bản rõ (Plaintext) sau khi đặt lại mật khẩu

- **Feature (FR):** `FR-03: Đặt lại mật khẩu (Bước 2)` & `SEC-01: Yêu cầu Bảo mật`
- **Môi trường (Environment):** Backend API & CSDL SQLite (`backend/database.sqlite`)
- **Mức độ nghiêm trọng (Severity):** **Critical** (Lỗ hổng bảo mật nghiêm trọng - Rò rỉ thông tin xác thực)
- **Mã Test Case liên quan:** `TC-FR03-DT-008`, `TC-FR03-BV-002`

---

### 1. Mô tả lỗi (Description)
Khi người dùng hoàn tất đặt lại mật khẩu thành công qua endpoint `POST /api/reset-password`, Backend cập nhật trường `password` trong bảng `users` bằng chính chuỗi mật khẩu thô mà người dùng gửi lên. Mật khẩu không hề được băm (hash) bằng bất kỳ thuật toán an toàn nào (như bcrypt hay argon2).

### 2. Các bước tái hiện (Steps to Reproduce)
1. Gửi request đặt lại mật khẩu hợp lệ:
   ```bash
   node -e "fetch('http://localhost:3000/api/reset-password', {
     method: 'POST',
     headers: {'Content-Type': 'application/json'},
     body: JSON.stringify({email: 'test@eshop.com', resetToken: '<valid_token>', newPassword: 'P@ssword123'})
   })"
   ```
2. Mở CSDL SQLite `backend/database.sqlite` và kiểm tra bảng `users`:
   ```sql
   SELECT id, email, password FROM users WHERE email = 'test@eshop.com';
   ```
3. Quan sát giá trị cột `password`.

### 3. Kết quả mong đợi (Expected Result)
- Theo yêu cầu bảo mật **SEC-01**: *"Mật khẩu **không** được lưu dưới dạng plaintext."*
- Cột `password` trong CSDL phải là chuỗi băm (hash string) an toàn (ví dụ chuẩn bcrypt `$2b$10$...`).

### 4. Kết quả thực tế (Actual Result)
- Mật khẩu lưu nguyên văn chuỗi thô:
  ```json
  [
    {
      "id": 2,
      "email": "test@eshop.com",
      "password": "P@ssword123"
    }
  ]
  ```
- Đoạn mã trong `backend/server.js:89-91`:
  ```javascript
  db.run(
    "UPDATE users SET password = ?, reset_token = NULL WHERE email = ? AND reset_token = ?",
    [newPassword, email, resetToken],
    ...
  );
  ```

### 5. Đề xuất khắc phục (Recommendation)
Sử dụng thư viện `bcrypt` hoặc hàm băm có salt trước khi lưu vào CSDL:
```javascript
const bcrypt = require('bcrypt');
const hashedPassword = await bcrypt.hash(newPassword, 10);
db.run("UPDATE users SET password = ?, reset_token = NULL ...", [hashedPassword, ...]);
```
