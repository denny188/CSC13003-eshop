# [BUG-FR10-03] Lỗ hổng Broken Access Control: API cập nhật trạng thái đơn hàng của Admin cho phép người dùng thường gọi

- **Feature (FR):** `FR-10: Trạng thái Đơn hàng` & `SEC-03: Kiểm tra phân quyền Admin`
- **Môi trường (Environment):** Backend REST API (`PUT /api/admin/orders/:id/status`)
- **Mức độ nghiêm trọng (Severity):** **Critical** (Lỗ hổng leo quyền - Broken Access Control / Privilege Escalation)
- **Mã Test Case liên quan:** `TC-FR10-DT-021`

---

### 1. Mô tả lỗi (Description)
Endpoint `PUT /api/admin/orders/:id/status` là API quản trị dành riêng cho Admin nhằm điều phối và cập nhật trạng thái đơn hàng trên toàn hệ thống. Tuy nhiên, endpoint này chỉ áp dụng middleware xác thực đăng nhập chung (`authenticateToken`) mà **hoàn toàn không kiểm tra vai trò người dùng (`req.user.role === 'admin'`)**. Do đó, bất kỳ tài khoản khách hàng thông thường nào (`role = 'user'`) cũng có thể gửi request để can thiệp và thay đổi trạng thái của mọi đơn hàng trong hệ thống (như tự chuyển đơn hàng của mình hoặc người khác sang `confirmed`, `shipping`, `delivered`).

### 2. Các bước tái hiện (Steps to Reproduce)
1. Đăng nhập với tài khoản người dùng bình thường (`test@eshop.com` / `Test1234!`) để lấy JWT token.
2. Gửi request PUT tới API quản trị của Admin:
   ```bash
   curl -X PUT http://localhost:3000/api/admin/orders/1/status \
     -H "Authorization: Bearer <USER_JWT_TOKEN>" \
     -H "Content-Type: application/json" \
     -d '{"status": "confirmed"}'
   ```
3. Quan sát kết quả trả về từ server.

### 3. Kết quả mong đợi (Expected Result)
- Theo yêu cầu bảo mật **SEC-03**: *"API Admin phải kiểm tra `role = 'admin'` trong Token, không chỉ kiểm tra sự tồn tại của Token."*
- Theo **api_specification.md (Mục 6)**: *"Tất cả API dưới đây yêu cầu Authorization: Bearer <token> và tài khoản phải có quyền Admin."*
- Server phải kiểm tra và từ chối truy cập với mã lỗi **HTTP 403 Forbidden** (hoặc 401).

### 4. Kết quả thực tế (Actual Result)
- Server chấp nhận token người dùng thường và trả về HTTP 200 OK:
  ```json
  {"message": "Order status updated"}
  ```
- Khách hàng thường thực hiện thành công quyền của quản trị viên.
- Nguyên nhân trong `backend/server.js:525`:
  ```javascript
  app.put("/api/admin/orders/:id/status", authenticateToken, (req, res) => { ... })
  ```
  Middleware chỉ kiểm tra tính hợp lệ của chữ ký JWT, không có middleware `requireAdmin` để kiểm tra thuộc tính `role`.

### 5. Đề xuất khắc phục (Recommendation)
Xây dựng middleware kiểm tra quyền Admin và áp dụng cho tất cả các route `/api/admin/*`:
```javascript
const requireAdmin = (req, res, next) => {
  if (req.user && req.user.role === 'admin') {
    next();
  } else {
    res.status(403).json({ error: "Yêu cầu quyền Quản trị viên (Forbidden)" });
  }
};

app.put("/api/admin/orders/:id/status", authenticateToken, requireAdmin, (req, res) => {
  // Logic cập nhật
});
```

---
- **GitHub Issue Link:** [Issue #10](https://github.com/denny188/CSC13003-eshop/issues/10)
