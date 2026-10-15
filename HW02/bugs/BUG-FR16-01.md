# [BUG-FR16-01] Bộ phân tích CSV phía Client không tuân thủ chuẩn RFC 4180 làm xô lệch cấu trúc cột

- **Feature (FR):** `FR-16: Import Sản phẩm từ CSV`
- **Môi trường (Environment):** Frontend Web Admin (`src/App.jsx:367-377`)
- **Mức độ nghiêm trọng (Severity):** **Critical** (Lỗi hỏng dữ liệu - Vi phạm đặc tả chuẩn RFC 4180)
- **Mã Test Case liên quan:** `TC-FR16-DT-002`

---

### 1. Mô tả lỗi (Description)
Đặc tả FR-16 yêu cầu: *"Hỗ trợ các trường có chứa dấu phẩy nếu được bọc trong dấu nháy kép (RFC 4180)"*. Tuy nhiên, trong mã nguồn React của Web Admin, hàm xử lý đọc file CSV chỉ sử dụng phương thức `line.split(",")` thô sơ của JavaScript. Khi trường `description` (hoặc bất kỳ trường nào) chứa dấu phẩy được bọc trong dấu nháy kép (ví dụ: `"RAM 16GB, SSD 512GB"`), câu lệnh `split(",")` sẽ chia tách chuỗi này thành nhiều phần tử, làm lệch toàn bộ các cột tiếp theo (`imageUrl` bị nhận giá trị của nửa sau mô tả, `category_id` bị nhận giá trị của `imageUrl`...).

### 2. Các bước tái hiện (Steps to Reproduce)
1. Tạo một file `products.csv` với nội dung có trường description chứa dấu phẩy:
   ```csv
   name,price,description,imageUrl,category_id
   Laptop Dell,15000000,"RAM 16GB, SSD 512GB",http://img.com/dell.jpg,1
   ```
2. Đăng nhập trang Web Admin tại `http://localhost:5174`, vào tab "Sản phẩm", phần "Import từ CSV".
3. Chọn file `products.csv` vừa tạo và quan sát bảng "Xem trước".

### 3. Kết quả mong đợi (Expected Result)
- Theo chuẩn **RFC 4180**:
  - `name`: `Laptop Dell`
  - `price`: `15000000`
  - `description`: `RAM 16GB, SSD 512GB`
  - `imageUrl`: `http://img.com/dell.jpg`
  - `category_id`: `1`

### 4. Kết quả thực tế (Actual Result)
- Bảng dữ liệu và đối tượng gửi lên server bị xô lệch hoàn toàn:
  - `name`: `Laptop Dell`
  - `price`: `15000000`
  - `description`: `"RAM 16GB`
  - `imageUrl`: `SSD 512GB"`
  - `category_id`: `http://img.com/dell.jpg`
- Đoạn mã lỗi tại `frontend-admin/src/App.jsx:367-377`:
  ```javascript
  const lines = text.trim().split("\n");
  const headers = lines[0].split(",").map((h) => h.trim());

  const rows = lines.slice(1).map((line) => {
    const values = line.split(",").map((v) => v.trim()); // LỖI: split(',') cắt vụn chuỗi có dấu phẩy trong ngoặc kép
    const obj = {};
    headers.forEach((h, i) => {
      obj[h] = values[i] || "";
    });
    return obj;
  });
  ```

### 5. Đề xuất khắc phục (Recommendation)
Sử dụng thư viện phân tích CSV chuẩn RFC 4180 (như `PapaParse`) hoặc biểu thức chính quy (Regex) hỗ trợ nháy kép:
```javascript
import Papa from "papaparse";

Papa.parse(file, {
  header: true,
  skipEmptyLines: true,
  complete: function (results) {
    setImportPreview(results.data);
  },
});
```
