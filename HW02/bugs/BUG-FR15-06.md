# [BUG-FR15-06] Các API Thêm/Sửa/Xóa sản phẩm (POST/PUT/DELETE /api/products) hoàn toàn mở, không có kiểm tra xác thực hoặc phân quyền Admin

- **Feature (FR):** `FR-15: Quản lý Sản phẩm (Admin)` & `SEC-02, SEC-03: Yêu cầu Bảo mật`
- **Môi trường (Environment):** Backend REST API (`backend/server.js:167-196`)
- **Mức độ nghiêm trọng (Severity):** **Critical** (Lỗ hổng Broken Authentication & Access Control - Mở toang dữ liệu sản phẩm cho công chúng)
- **Mã Test Case liên quan:** `TC-FR15-DT-013`, `TC-FR15-DT-014`, `TC-FR15-DT-015`, `TC-FR15-DT-016`

---

### 1. Mô tả lỗi (Description)
Theo tài liệu kiến trúc và đặc tả `api_specification.md (Mục 3.3)`, các hành động Thêm, Sửa, Xóa sản phẩm là chức năng quản trị ("Dành cho Admin"). Tuy nhiên, trong mã nguồn `server.js`, các route:
- `POST /api/products`
- `PUT /api/products/:id`
- `DELETE /api/products/:id`

hoàn toàn **không được gắn bất kỳ middleware xác thực nào** (không có `authenticateToken` và không có kiểm tra `role === 'admin'`). Bất kỳ ai trên internet (kể cả khách vãng lai không có tài khoản) đều có thể gửi request để thêm sản phẩm rác, thay đổi giá thành 1đ, hoặc xóa sạch toàn bộ sản phẩm của cửa hàng.

### 2. Các bước tái hiện (Steps to Reproduce)
1. Mở terminal và gửi request DELETE sản phẩm mà không truyền bất kỳ Token nào:
   ```bash
   curl -X DELETE http://localhost:3000/api/products/1
   ```
2. Gửi request POST tạo sản phẩm mà không có Token:
   ```bash
   curl -X POST http://localhost:3000/api/products \
     -H "Content-Type: application/json" \
     -d '{"name":"Hacked Product","price":1000}'
   ```
3. Quan sát phản hồi.

### 3. Kết quả mong đợi (Expected Result)
- Theo yêu cầu bảo mật **SEC-02**: *"Các API có tính bảo mật phải yêu cầu JWT Token hợp lệ."*
- Theo **SEC-03**: *"API Admin phải kiểm tra role = 'admin' trong Token..."*
- Server phải chặn ngay từ đầu và trả về HTTP 401 Unauthorized hoặc 403 Forbidden.

### 4. Kết quả thực tế (Actual Result)
- Server thực thi câu lệnh xóa/thêm thành công và trả về HTTP 200 OK:
  ```json
  {"message": "Product deleted"}
  {"message": "Product created", "id": 11}
  ```
- Đoạn mã trong `backend/server.js:167, 179, 191`:
  ```javascript
  app.post("/api/products", (req, res) => { ... });
  app.put("/api/products/:id", (req, res) => { ... });
  app.delete("/api/products/:id", (req, res) => { ... });
  ```

### 5. Đề xuất khắc phục (Recommendation)
Thêm middleware xác thực và kiểm tra vai trò Admin cho cả 3 route:
```javascript
app.post("/api/products", authenticateToken, requireAdmin, (req, res) => { ... });
app.put("/api/products/:id", authenticateToken, requireAdmin, (req, res) => { ... });
app.delete("/api/products/:id", authenticateToken, requireAdmin, (req, res) => { ... });
```
