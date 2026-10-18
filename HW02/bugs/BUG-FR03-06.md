# [BUG-FR03-06] Biểu thức chính quy (Regex) kiểm tra mật khẩu ở Client sai: bắt buộc khoảng trắng thay vì ký tự đặc biệt

- **Feature (FR):** `FR-03: Đặt lại mật khẩu (Bước 2)` & `FR-01: Ràng buộc Mật khẩu`
- **Môi trường (Environment):** Frontend Web (`src/pages/ForgotPassword.jsx`)
- **Mức độ nghiêm trọng (Severity):** **High** (Chặn người dùng nhập mật khẩu hợp lệ và chấp nhận mật khẩu chứa khoảng trắng)
- **Mã Test Case liên quan:** `TC-FR03-DT-021`, `TC-FR03-DT-022`

---

### 1. Mô tả lỗi (Description)
Hàm kiểm tra mật khẩu phía Frontend Web sử dụng một regex có tên là `flawedStrongPasswordRegex`. Regex này thay vì kiểm tra ký tự đặc biệt trong tập `[@$!%*?&]` thì lại kiểm tra `(?=.*\s)` (bắt buộc phải có ký tự khoảng trắng / khoảng trắng). Hậu quả là người dùng nhập mật khẩu mạnh hợp lệ theo spec (ví dụ: `P@ssword123`) sẽ bị báo lỗi "Mật khẩu quá yếu", trong khi nếu nhập mật khẩu có khoảng trắng như `Pass word1` thì lại được chấp nhận!

### 2. Các bước tái hiện (Steps to Reproduce)
1. Mở trang Đặt lại mật khẩu Bước 2 trên Web.
2. Nhập mật khẩu hợp lệ theo đặc tả: `P@ssword123`.
3. Bấm "Đặt lại mật khẩu".
4. Quan sát: Hộp thoại alert bật lên báo lỗi *"Mật khẩu quá yếu! Phải dài tối thiểu 8 ký tự, gồm chữ hoa, chữ thường, số và KÝ TỰ ĐẶC BIỆT."* dù người dùng đã nhập đúng ký tự `@`.

### 3. Kết quả mong đợi (Expected Result)
- Theo đặc tả **FR-01**: Mật khẩu có ký tự đặc biệt thuộc tập `[@$!%*?&]`.
- Mật khẩu `P@ssword123` phải được chấp nhận và không báo lỗi.

### 4. Kết quả thực tế (Actual Result)
- Đoạn mã tại `ForgotPassword.jsx:26-27`:
  ```javascript
  const flawedStrongPasswordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*\s)[A-Za-z\d\s]{8,}$/;
  if (!flawedStrongPasswordRegex.test(newPassword)) { ... }
  ```
- Regex đòi hỏi `\s` (khoảng trắng) và chỉ cho phép ký tự trong `[A-Za-z\d\s]`, loại trừ hoàn toàn các ký tự `@$!%*?&`.

### 5. Đề xuất khắc phục (Recommendation)
Sửa lại regex đúng theo yêu cầu FR-01:
```javascript
const strongPasswordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/;
```

---
- **GitHub Issue Link:** [Issue #6](https://github.com/denny188/CSC13003-eshop/issues/6)
