# [BUG-FR15-04] Cho phép tạo sản phẩm với mã danh mục (category_id) không tồn tại hoặc rỗng

- **Feature (FR):** `FR-15: Quản lý Sản phẩm (Product CRUD)`
- **Môi trường (Environment):** Backend REST API (`POST /api/products`)
- **Mức độ nghiêm trọng (Severity):** **Medium** (Vi phạm ràng buộc khóa ngoại và tính nhất quán dữ liệu)
- **Mã Test Case liên quan:** `TC-FR15-DT-009`, `TC-FR15-DT-010`

---

### 1. Mô tả lỗi (Description)
Đặc tả FR-15 nêu rõ: *"Danh mục: bắt buộc, phải chọn từ danh sách có sẵn."* Tuy nhiên, trên Backend API, khi client gửi trường `category_id` là một ID không hề tồn tại trong bảng `categories` (ví dụ `category_id = 9999`) hoặc `null`, câu lệnh INSERT vẫn được thực thi thành công do SQLite không bật cờ kiểm tra ràng buộc khóa ngoại (`PRAGMA foreign_keys = ON;`) và backend không có câu lệnh kiểm tra sự tồn tại của danh mục trước khi thêm.

### 2. Các bước tái hiện (Steps to Reproduce)
1. Gửi request tạo sản phẩm với `category_id = 9999`:
   ```bash
   curl -X POST http://localhost:3000/api/products \
     -H "Content-Type: application/json" \
     -d '{"name": "SP Lạ", "price": 100000, "category_id": 9999}'
   ```
2. Quan sát kết quả.

### 3. Kết quả mong đợi (Expected Result)
- Theo đặc tả **FR-15**: Danh mục phải thuộc danh sách có sẵn.
- Hệ thống phải từ chối và trả về HTTP 400 Bad Request: `Danh mục không tồn tại`.

### 4. Kết quả thực tế (Actual Result)
- Server trả về HTTP 200 OK:
  ```json
  {"message": "Product created", "id": 10}
  ```
- Sản phẩm được lưu với danh mục mồ côi (orphaned foreign key).

### 5. Đề xuất khắc phục (Recommendation)
1. Kích hoạt ràng buộc khóa ngoại trong kết nối SQLite (`database.js`):
   ```javascript
   db.run("PRAGMA foreign_keys = ON;");
   ```
2. Kiểm tra `category_id` trong bảng `categories` trước khi insert hoặc bắt lỗi ngoại lệ khóa ngoại.
