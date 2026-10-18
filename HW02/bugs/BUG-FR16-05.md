# [BUG-FR16-05] API Import sản phẩm (/api/admin/import-products) thiếu kiểm tra vai trò Quản trị viên (role === 'admin')

- **Feature (FR):** `FR-16: Import Sản phẩm từ CSV`
- **Môi trường (Environment):** Backend API (`backend/server.js:199`)
- **Mức độ nghiêm trọng (Severity):** **High** (Lỗ hổng Phân quyền & Kiểm soát truy cập - Broken Access Control)
- **Mã Test Case liên quan:** `TC-FR16-DT-012`

---

### 1. Mô tả lỗi (Description)
Đặc tả FR-12 và SEC-03 quy định:
- *"Phân hệ Admin chỉ dành cho tài khoản có `role = 'admin'`."*
- *"Tất cả các API Admin (`/api/admin/*`) đều phải yêu cầu: 1. Token JWT hợp lệ; 2. `role = 'admin'` trong Token."*

Tuy nhiên, tại endpoint `POST /api/admin/import-products`, lập trình viên chỉ gắn middleware `authenticateToken` chung chung mà không kiểm tra thuộc tính `req.user.role === 'admin'`. Do đó, bất kỳ tài khoản người dùng thông thường nào (ví dụ tài khoản `test@eshop.com` với `role = 'user'`) chỉ cần có JWT token hợp lệ là có thể gửi request import hàng loạt sản phẩm vào CSDL thành công.

### 2. Các bước tái hiện (Steps to Reproduce)
1. Đăng nhập với tài khoản khách hàng thông thường `test@eshop.com` / `Test1234!` qua API `POST /api/login`.
2. Lấy JWT Token trả về (có payload chứa `role: "user"`).
3. Gửi request `POST /api/admin/import-products` với Header:
   - `Authorization: Bearer <user_token>`
   - `Content-Type: application/json`
   - Body:
     ```json
     {
       "products": [
         { "name": "Sản phẩm do User thường thêm trái phép", "price": 999999 }
       ]
     }
     ```
4. Quan sát mã phản hồi HTTP và dữ liệu trong CSDL.

### 3. Kết quả mong đợi (Expected Result)
- Backend kiểm tra vai trò người dùng, phát hiện `req.user.role !== 'admin'` và trả về HTTP 403 Forbidden: `{"error": "Yêu cầu quyền Quản trị viên"}`. Thao tác import bị chặn hoàn toàn.

### 4. Kết quả thực tế (Actual Result)
- Backend trả về HTTP 200:
  ```json
  {
    "message": "Import hoàn tất: 1/1 sản phẩm được thêm",
    "inserted": 1,
    "errors": []
  }
  ```
- Sản phẩm được chèn thành công vào CSDL bởi một người dùng thông thường.
- Đoạn mã lỗi tại `backend/server.js:199`:
  ```javascript
  app.post("/api/admin/import-products", authenticateToken, (req, res) => {
    // LỖI: Hoàn toàn thiếu bước kiểm tra if (req.user.role !== "admin")
  ```

### 5. Đề xuất khắc phục (Recommendation)
Bổ sung middleware kiểm tra quyền Admin hoặc kiểm tra trực tiếp trong controller:
```javascript
function requireAdmin(req, res, next) {
  if (!req.user || req.user.role !== "admin") {
    return res.status(403).json({ error: "Yêu cầu quyền Quản trị viên (Admin)" });
  }
  next();
}

app.post("/api/admin/import-products", authenticateToken, requireAdmin, (req, res) => {
  // Logic xử lý import
});
```

---
- **GitHub Issue Link:** [Issue #22](https://github.com/denny188/CSC13003-eshop/issues/22)
