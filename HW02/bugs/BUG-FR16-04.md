# [BUG-FR16-04] Hệ thống vi phạm nguyên tắc giao dịch nguyên tử (All-or-Nothing Rollback), gây rò rỉ dữ liệu dở dang khi có lỗi

- **Feature (FR):** `FR-16: Import Sản phẩm từ CSV`
- **Môi trường (Environment):** Backend API (`backend/server.js:199-241`)
- **Mức độ nghiêm trọng (Severity):** **Critical** (Lỗi tính toàn vẹn giao dịch - Vi phạm trực tiếp yêu cầu bắt buộc của đặc tả)
- **Mã Test Case liên quan:** `TC-FR16-DT-009`, `TC-FR16-DT-010`, `TC-FR16-BV-005`, `TC-FR16-BV-006`

---

### 1. Mô tả lỗi (Description)
Đặc tả FR-16 nêu rõ yêu cầu bắt buộc:
- *"Nếu có lỗi ở bất kỳ dòng nào, toàn bộ import phải được **rollback** (giao dịch nguyên tử — all-or-nothing)."*

Tuy nhiên, trong backend `server.js`, hàm import sử dụng vòng lặp `rows.forEach()` để chạy câu lệnh `INSERT INTO products` độc lập từng dòng mà **hoàn toàn không mở một Transaction (`BEGIN TRANSACTION / COMMIT / ROLLBACK`)**. Khi một file có nhiều dòng và xuất hiện lỗi ở một dòng bất kỳ (ví dụ dòng thứ 3 rỗng tên), hệ thống vẫn âm thầm commit các dòng hợp lệ trước đó (dòng 1, 2) vào CSDL SQLite! Việc chèn dữ liệu dở dang (partial insert) này phá vỡ hoàn toàn nguyên tắc giao dịch nguyên tử được quy định trong đặc tả.

### 2. Các bước tái hiện (Steps to Reproduce)
1. Ghi nhận số lượng sản phẩm hiện có trong CSDL (ví dụ $N = 10$).
2. Chuẩn bị file CSV gồm 3 dòng:
   - Dòng 1: Sản phẩm A (Hợp lệ, giá 100,000)
   - Dòng 2: Sản phẩm B (Hợp lệ, giá 200,000)
   - Dòng 3: Dòng lỗi (Tên để trống `""`, giá 300,000)
3. Thực hiện tải file lên và bấm Import trên Web Admin.
4. Kiểm tra lại số lượng sản phẩm trong CSDL.

### 3. Kết quả mong đợi (Expected Result)
- Theo đặc tả **FR-16**: Vì dòng thứ 3 bị lỗi, hệ thống phải **hủy bỏ toàn bộ giao dịch (Rollback 100%)**. CSDL vẫn phải giữ nguyên $N = 10$ sản phẩm ban đầu, không có sản phẩm A hay B nào được thêm vào. Báo cáo hiển thị rõ lỗi ở dòng 3.

### 4. Kết quả thực tế (Actual Result)
- Số lượng sản phẩm trong CSDL tăng lên thành $12$! Sản phẩm A và B đã bị lưu vĩnh viễn vào CSDL.
- Backend phản hồi HTTP 200:
  ```json
  {
    "message": "Import hoàn tất: 2/3 sản phẩm được thêm",
    "inserted": 2,
    "errors": [
      "Hàng 4: Thiếu tên sản phẩm"
    ]
  }
  ```
- Đoạn mã lỗi tại `backend/server.js:209-232`:
  ```javascript
  const stmt = db.prepare(
    "INSERT INTO products (name, price, description, imageUrl, category_id) VALUES (?, ?, ?, ?, ?)",
  );

  rows.forEach((row, index) => {
    if (!row.name) {
      errors.push(`Hàng ${index + 2}: Thiếu tên sản phẩm`);
      return; // Chỉ bỏ qua dòng này mà KHÔNG rollback các dòng trước!
    }
    stmt.run(...); // Insert trực tiếp vào DB từng dòng
  });
  ```

### 5. Đề xuất khắc phục (Recommendation)
1. Thực hiện validation toàn bộ dữ liệu trước khi chèn vào CSDL:
```javascript
const validationErrors = [];
rows.forEach((row, index) => {
  if (!row.name) validationErrors.push(`Hàng ${index + 2}: Thiếu tên sản phẩm`);
  if (!row.price || Number(row.price) <= 0) validationErrors.push(`Hàng ${index + 2}: Giá phải lớn hơn 0`);
});

// Nếu có bất kỳ lỗi nào, dừng ngay và rollback
if (validationErrors.length > 0) {
  return res.status(400).json({
    error: "Import thất bại do có lỗi dữ liệu. Toàn bộ thao tác đã được rollback.",
    errors: validationErrors,
    inserted: 0
  });
}
```
2. Bao bọc toàn bộ thao tác thêm mới bằng Transaction của SQLite (`BEGIN TRANSACTION` và `ROLLBACK` khi có lỗi xảy ra).
