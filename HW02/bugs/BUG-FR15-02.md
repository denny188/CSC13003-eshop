# [BUG-FR15-02] Hệ thống cho phép nhập giá tiền bằng 0 hoặc số âm

- **Feature (FR):** `FR-15: Quản lý Sản phẩm (Product CRUD)`
- **Môi trường (Environment):** Backend REST API (`POST /api/products` & `PUT /api/products/:id`)
- **Mức độ nghiêm trọng (Severity):** **High** (Lỗ hổng logic tài chính - Cho phép bán hàng giá 0đ hoặc nhận tiền âm)
- **Mã Test Case liên quan:** `TC-FR15-DT-005`, `TC-FR15-DT-006`, `TC-FR15-BV-008`

---

### 1. Mô tả lỗi (Description)
Đặc tả FR-15 quy định rõ ràng rằng giá sản phẩm bắt buộc phải là một số dương ($price > 0$). Tuy nhiên, cả ở tầng API Backend và Form nhập liệu phía Web Admin đều không có câu lệnh kiểm tra điều kiện này. Khi người dùng nhập `price = 0` hoặc `price = -50000`, hệ thống vẫn chấp nhận và lưu thẳng vào CSDL.

### 2. Các bước tái hiện (Steps to Reproduce)
1. Gửi request tạo sản phẩm với giá âm:
   ```bash
   curl -X POST http://localhost:3000/api/products \
     -H "Content-Type: application/json" \
     -d '{"name": "Sản phẩm Giá Âm", "price": -50000, "category_id": 1}'
   ```
2. Gửi request tạo sản phẩm với giá bằng 0:
   ```bash
   curl -X POST http://localhost:3000/api/products \
     -H "Content-Type: application/json" \
     -d '{"name": "Sản phẩm Giá 0", "price": 0, "category_id": 1}'
   ```
3. Quan sát kết quả.

### 3. Kết quả mong đợi (Expected Result)
- Theo đặc tả **FR-15**: *"Giá: bắt buộc, phải là số **dương** (> 0)."*
- Hệ thống phải từ chối mọi giá trị $price \le 0$ và trả về HTTP 400 Bad Request.

### 4. Kết quả thực tế (Actual Result)
- Cả hai request đều trả về HTTP 200 OK:
  ```json
  {"message": "Product created", "id": 6}
  {"message": "Product created", "id": 7}
  ```
- CSDL SQLite lưu trữ trực tiếp các sản phẩm có giá 0đ và -50.000đ.

### 5. Đề xuất khắc phục (Recommendation)
Thêm kiểm tra giá trị số dương trong `backend/server.js`:
```javascript
const numPrice = Number(price);
if (isNaN(numPrice) || numPrice <= 0) {
  return res.status(400).json({ error: "Giá sản phẩm bắt buộc phải là số dương (> 0)." });
}
```
Đồng thời trên form Web Admin thêm thuộc tính `min="1"` vào thẻ `<input type="number">`.
