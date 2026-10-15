# [BUG-FR03-02] Trường nhập Email dùng sai kiểu type="text" thay vì type="email"

- **Feature (FR):** `FR-03: Quên mật khẩu (Bước 1)` & `FR-22: Form Requirements`
- **Môi trường (Environment):** Frontend Web (`http://localhost:5173`)
- **Mức độ nghiêm trọng (Severity):** **Medium** (Thiếu validation chuẩn client-side)
- **Mã Test Case liên quan:** `TC-FR03-DT-001`, `TC-FR03-DT-004`, `TC-FR03-DT-005`

---

### 1. Mô tả lỗi (Description)
Tại trang Quên Mật Khẩu Bước 1, thẻ `<input>` dành cho trường Email được khai báo với thuộc tính `type="text"`. Điều này làm mất khả năng tự động kiểm tra định dạng email của trình duyệt (HTML5 email format validation), cho phép người dùng submit chuỗi bất kỳ (ví dụ không có `@` hoặc thiếu domain) lên server.

### 2. Các bước tái hiện (Steps to Reproduce)
1. Mở trang Quên mật khẩu trên trình duyệt Web.
2. Tại ô "Nhập Email của bạn", điền một chuỗi không hợp lệ như `testeshop.com`.
3. Bấm nút "Lấy mã OTP".
4. Quan sát: Trình duyệt không hề báo lỗi sai định dạng email mà gửi thẳng request lên server.

### 3. Kết quả mong đợi (Expected Result)
- Theo đặc tả **FR-22 / Form Requirements**: *"Trường Email phải dùng `type=\"email\"`."*
- Khi người dùng nhập chuỗi không đúng định dạng email, trình duyệt phải ngăn không cho submit và hiển thị tooltip cảnh báo định dạng hợp lệ.

### 4. Kết quả thực tế (Actual Result)
- Tại `ForgotPassword.jsx:49-55`:
  ```jsx
  <input
    type="text"
    value={email}
    onChange={(e) => setEmail(e.target.value)}
    className="w-full border p-2 rounded"
    required
  />
  ```
- Form cho phép submit chuỗi không phải email.

### 5. Đề xuất khắc phục (Recommendation)
Đổi thuộc tính `type="text"` thành `type="email"`.
