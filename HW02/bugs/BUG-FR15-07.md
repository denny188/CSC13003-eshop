# [BUG-FR15-07] Hệ thống cho phép nhập tên sản phẩm dài vượt quá 255 ký tự

- **Feature (FR):** `FR-15: Quản lý Sản phẩm (Product CRUD)`
- **Môi trường (Environment):** Backend REST API & CSDL SQLite
- **Mức độ nghiêm trọng (Severity):** **Medium** (Vi phạm ràng buộc độ dài tối đa của đặc tả)
- **Mã Test Case liên quan:** `TC-FR15-BV-005`

---

### 1. Mô tả lỗi (Description)
Đặc tả FR-15 quy định: *"Tên sản phẩm: bắt buộc, tối đa 255 ký tự."* Tuy nhiên, cả ở tầng Form Client và API Backend đều không có giới hạn độ dài (ví dụ không có thuộc tính `maxLength="255"` và backend không kiểm tra `name.length <= 255`). CSDL SQLite sử dụng kiểu dữ liệu `TEXT` không tự động cắt ngắn chuỗi, dẫn tới việc sản phẩm có tên dài 256 ký tự hoặc hàng nghìn ký tự vẫn được tạo thành công, có nguy cơ gây vỡ layout giao diện (UI layout breaking) hoặc tràn bộ nhớ.

### 2. Các bước tái hiện (Steps to Reproduce)
1. Gửi request tạo sản phẩm với chuỗi tên dài đúng 256 ký tự (ví dụ 256 ký tự `'A'`):
   ```bash
   node -e "fetch('http://localhost:3000/api/products', {
     method: 'POST',
     headers: {'Content-Type': 'application/json'},
     body: JSON.stringify({name: 'A'.repeat(256), price: 100000, category_id: 1})
   }).then(r => r.json()).then(console.log)"
   ```
2. Quan sát kết quả phản hồi.

### 3. Kết quả mong đợi (Expected Result)
- Theo đặc tả **FR-15**: Tên sản phẩm tối đa 255 ký tự.
- Hệ thống phải từ chối và trả về HTTP 400 Bad Request: `Tên sản phẩm không được vượt quá 255 ký tự`.

### 4. Kết quả thực tế (Actual Result)
- Server trả về HTTP 200 OK:
  ```json
  {"message": "Product created", "id": 9}
  ```
- Sản phẩm được lưu đầy đủ 256 ký tự vào CSDL.

### 5. Đề xuất khắc phục (Recommendation)
Thêm kiểm tra độ dài tối đa tại `server.js`:
```javascript
if (name.length > 255) {
  return res.status(400).json({ error: "Tên sản phẩm tối đa 255 ký tự." });
}
```
Và trên input Form Web Admin thêm `maxLength={255}`.

---
- **GitHub Issue Link:** [Issue #17](https://github.com/denny188/CSC13003-eshop/issues/17)
