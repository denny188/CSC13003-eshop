# [BUG-FR03-01] Mã OTP chỉ sinh ra 4 chữ số thay vì 6 chữ số theo quy định đặc tả

- **Feature (FR):** `FR-03: Quên mật khẩu & Đặt lại mật khẩu (Bước 1)`
- **Môi trường (Environment):** Backend API (`http://localhost:3000`) & Web Client (`http://localhost:5173`)
- **Mức độ nghiêm trọng (Severity):** **High** (Entropy OTP quá thấp, vi phạm quy định bảo mật)
- **Mã Test Case liên quan:** `TC-FR03-DT-001`, `TC-FR03-DT-010`

---

### 1. Mô tả lỗi (Description)
Khi người dùng gửi yêu cầu lấy mã OTP đặt lại mật khẩu tại endpoint `POST /api/forgot-password` (hoặc form Bước 1 trên Web), hệ thống sinh ra mã OTP chỉ gồm **4 chữ số** (từ 1000 đến 9999). Trên giao diện Web cũng hiển thị rõ nhãn yêu cầu: *"Mã OTP (4 số)"*.

### 2. Các bước tái hiện (Steps to Reproduce)
1. Gửi request API lấy OTP:
   ```bash
   curl -X POST http://localhost:3000/api/forgot-password \
     -H "Content-Type: application/json" \
     -d '{"email":"test@eshop.com"}'
   ```
2. Quan sát trường `resetToken` trong phản hồi JSON trả về.
3. Đồng thời mở giao diện Quên mật khẩu trên trình duyệt tại `http://localhost:5173/forgot-password`, nhập email và bấm "Lấy mã OTP".
4. Quan sát mã hiển thị và nhãn của trường nhập OTP ở Bước 2.

### 3. Kết quả mong đợi (Expected Result)
- Theo đặc tả **FR-03 / Bước 1**: *"Hệ thống sinh mã OTP 6 chữ số ngẫu nhiên..."*
- Theo yêu cầu bảo mật **SEC-07**: *"OTP đặt lại mật khẩu phải đủ entropy (tối thiểu 6 chữ số)..."*
- Giá trị `resetToken` sinh ra phải gồm đúng 6 chữ số ngẫu nhiên (`000000` - `999999`).

### 4. Kết quả thực tế (Actual Result)
- Backend API sinh mã 4 chữ số:
  ```json
  {"message":"Mã đặt lại mật khẩu đã được tạo","resetToken":"3742"}
  ```
- Mã nguồn backend (`server.js:72`):
  ```javascript
  const resetToken = Math.floor(1000 + Math.random() * 9000).toString();
  ```
- Giao diện Web hiển thị label: *"Mã OTP (4 số)"*.

### 5. Đề xuất khắc phục (Recommendation)
Sửa công thức sinh OTP thành chuỗi ngẫu nhiên 6 chữ số:
```javascript
const resetToken = Math.floor(100000 + Math.random() * 900000).toString();
```
Đồng thời cập nhật lại label và placeholder trên giao diện Web thành "Mã OTP (6 số)".

---
- **GitHub Issue Link:** [Issue #1](https://github.com/denny188/CSC13003-eshop/issues/1)
