# [BUG-FR15-01] Backend API không kiểm tra bắt buộc tên sản phẩm (cho phép tạo tên rỗng hoặc toàn khoảng trắng)

- **Feature (FR):** `FR-15: Quản lý Sản phẩm (Product CRUD)`
- **Môi trường (Environment):** Backend REST API (`POST /api/products` & `PUT /api/products/:id`)
- **Mức độ nghiêm trọng (Severity):** **Medium** (Thiếu ràng buộc tính toàn vẹn dữ liệu ở tầng backend)
- **Mã Test Case liên quan:** `TC-FR15-DT-002`, `TC-FR15-DT-003`

---

### 1. Mô tả lỗi (Description)
Theo đặc tả nghiệp vụ FR-15, tên sản phẩm là trường dữ liệu bắt buộc. Tuy nhiên, endpoint `POST /api/products` và `PUT /api/products/:id` trên backend hoàn toàn không kiểm tra sự tồn tại hoặc độ dài của trường `name`. Nếu client gửi request với `name: ""` hoặc chuỗi chỉ chứa khoảng trắng `"   "`, hệ thống vẫn lưu vào CSDL SQLite và trả về HTTP 200 OK.

### 2. Các bước tái hiện (Steps to Reproduce)
1. Gửi request tạo sản phẩm mới với tên để trống:
   ```bash
   curl -X POST http://localhost:3000/api/products \
     -H "Content-Type: application/json" \
     -d '{"name": "", "price": 100000, "category_id": 1}'
   ```
2. Quan sát phản hồi JSON và kiểm tra CSDL SQLite.

### 3. Kết quả mong đợi (Expected Result)
- Theo đặc tả **FR-15**: *"Tên sản phẩm: bắt buộc, tối đa 255 ký tự."*
- Backend phải từ chối và trả về HTTP 400 Bad Request kèm thông báo: `Tên sản phẩm không được để trống`.

### 4. Kết quả thực tế (Actual Result)
- Server trả về HTTP 200 OK:
  ```json
  {"message": "Product created", "id": 8}
  ```
- CSDL tạo một sản phẩm có cột `name` rỗng.

### 5. Đề xuất khắc phục (Recommendation)
Thêm kiểm tra tính hợp lệ của `name` trước khi thực thi truy vấn:
```javascript
if (!name || name.trim().length === 0) {
  return res.status(400).json({ error: "Tên sản phẩm là bắt buộc và không được để trống." });
}
```
