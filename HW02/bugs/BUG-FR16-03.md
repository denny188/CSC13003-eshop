# [BUG-FR16-03] API Import sản phẩm thiếu validation nghiệp vụ từng dòng (chấp nhận giá âm, giá = 0, tên vượt quá 255 ký tự)

- **Feature (FR):** `FR-16: Import Sản phẩm từ CSV`
- **Môi trường (Environment):** Backend API (`backend/server.js:213-232`)
- **Mức độ nghiêm trọng (Severity):** **High** (Lỗi toàn vẹn dữ liệu - Thiếu validation đầu vào)
- **Mã Test Case liên quan:** `TC-FR16-DT-006`, `TC-FR16-DT-007`, `TC-FR16-BV-008`, `TC-FR16-BV-011`

---

### 1. Mô tả lỗi (Description)
Đặc tả FR-16 quy định:
- *"Validation trước khi import: `name` không được rỗng; `price` phải là số dương (> 0)."*
- Kế thừa từ FR-15: Tên sản phẩm tối đa 255 ký tự.

Tuy nhiên, trong backend `server.js`, hàm xử lý API `POST /api/admin/import-products` chỉ có duy nhất một câu lệnh kiểm tra `if (!row.name)` để loại bỏ dòng rỗng tên. Hệ thống hoàn toàn không kiểm tra giá tiền (`price`), cho phép nhập giá bằng 0, số âm (ví dụ: `-50000`), chuỗi chữ không phải số, và không kiểm tra độ dài tối đa của `name`. Kết quả là các sản phẩm lỗi này vẫn được lưu thành công vào CSDL.

### 2. Các bước tái hiện (Steps to Reproduce)
1. Gửi request `POST /api/admin/import-products` kèm token Admin và body chứa dòng có giá âm và giá bằng 0:
   ```json
   {
     "products": [
       { "name": "Sản phẩm Giá Âm", "price": -50000, "category_id": 1 },
       { "name": "Sản phẩm Giá Không", "price": 0, "category_id": 1 }
     ]
   }
   ```
2. Kiểm tra phản hồi từ API và dữ liệu trong CSDL SQLite.

### 3. Kết quả mong đợi (Expected Result)
- Backend từ chối các dòng có `price <= 0` hoặc `isNaN(price)`, báo lỗi: *"Giá sản phẩm phải là số dương (> 0)"*.

### 4. Kết quả thực tế (Actual Result)
- Backend phản hồi HTTP 200:
  ```json
  {
    "message": "Import hoàn tất: 2/2 sản phẩm được thêm",
    "inserted": 2,
    "errors": []
  }
  ```
- CSDL lưu thành công 2 sản phẩm có giá âm và giá bằng 0.
- Mã nguồn tại `backend/server.js:213-232`:
  ```javascript
  rows.forEach((row, index) => {
    if (!row.name) {
      errors.push(`Hàng ${index + 2}: Thiếu tên sản phẩm`);
      return;
    }
    // LỖI: Hoàn toàn không có kiểm tra if (row.price <= 0 || isNaN(row.price))
    stmt.run(
      row.name,
      row.price,
      row.description || "",
      row.imageUrl || "",
      row.category_id || 1,
      ...
    );
  });
  ```

### 5. Đề xuất khắc phục (Recommendation)
Thêm kiểm tra đầy đủ các ràng buộc nghiệp vụ trước khi thực thi câu lệnh SQL:
```javascript
const priceNum = Number(row.price);
if (isNaN(priceNum) || priceNum <= 0) {
  errors.push(`Hàng ${index + 2}: Giá phải là số dương (> 0)`);
  return;
}
if (row.name.length > 255) {
  errors.push(`Hàng ${index + 2}: Tên sản phẩm không được vượt quá 255 ký tự`);
  return;
}
```
