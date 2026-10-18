# [BUG-FR16-02] Web Admin không giới hạn định dạng file upload (.csv) và thiếu xác thực header chuẩn

- **Feature (FR):** `FR-16: Import Sản phẩm từ CSV`
- **Môi trường (Environment):** Frontend Web Admin (`src/App.jsx:356-384`)
- **Mức độ nghiêm trọng (Severity):** **Medium** (Lỗi thiếu ràng buộc giao diện & kiểm tra định dạng file)
- **Mã Test Case liên quan:** `TC-FR16-DT-003`, `TC-FR16-DT-004`

---

### 1. Mô tả lỗi (Description)
Đặc tả FR-16 quy định:
- *"Đuôi file phải là `.csv`."*
- *"Dòng đầu tiên là header: `name,price,description,imageUrl,category_id`."*

Tuy nhiên, thẻ `<input type="file" />` trên giao diện Web Admin không hề có thuộc tính `accept=".csv"`. Người dùng có thể chọn bất kỳ định dạng file nào như `.txt`, `.json`, `.png`, `.xlsx` để tải lên. Khi chọn file sai, hệ thống cố gắng đọc text và parse dòng đầu tiên làm header, gây ra lỗi hiển thị hoặc gửi dữ liệu vô nghĩa lên server mà không có thông báo lỗi từ chối file.

### 2. Các bước tái hiện (Steps to Reproduce)
1. Mở trang Web Admin tại `http://localhost:5174`, vào tab "Sản phẩm", phần "Import từ CSV".
2. Bấm nút "Choose File" / "Chọn tệp".
3. Cửa sổ mở file cho phép chọn mọi loại file (All Files `*.*`).
4. Chọn một file bất kỳ không phải CSV (ví dụ `package.json` hoặc `report.txt`).

### 3. Kết quả mong đợi (Expected Result)
- Khung chọn file chỉ hiển thị và cho phép chọn file có đuôi `.csv`.
- Nếu người dùng cố tình tải file có nội dung/header không đúng cấu trúc `name,price,description,imageUrl,category_id`, giao diện hiển thị thông báo lỗi rõ ràng và từ chối xử lý.

### 4. Kết quả thực tế (Actual Result)
- Thẻ input tại `frontend-admin/src/App.jsx:356-384`:
  ```jsx
  <input
    type="file"
    onChange={(e) => { ... }}
    className="border p-2 rounded bg-white flex-1 text-sm"
  />
  ```
  Hoàn toàn thiếu `accept=".csv"`, không có bước kiểm tra tên file kết thúc bằng `.csv`, và không xác thực danh sách header bắt buộc.

### 5. Đề xuất khắc phục (Recommendation)
1. Thêm thuộc tính `accept=".csv"` vào thẻ `<input type="file">`.
2. Kiểm tra phần mở rộng file và xác thực header bắt buộc trước khi hiển thị xem trước:
```javascript
if (!file.name.endsWith(".csv")) {
  alert("Vui lòng chỉ tải lên file định dạng .csv");
  return;
}
const requiredHeaders = ["name", "price", "category_id"];
const hasAllHeaders = requiredHeaders.every((h) => headers.includes(h));
if (!hasAllHeaders) {
  alert("File CSV không đúng cấu trúc header chuẩn.");
  return;
}
```

---
- **GitHub Issue Link:** [Issue #19](https://github.com/denny188/CSC13003-eshop/issues/19)
