# [BUG-FR15-05] Khi sửa một sản phẩm trên Web Admin, toàn bộ các sản phẩm khác đều bị đổi tên theo

- **Feature (FR):** `FR-15: Quản lý Sản phẩm (Product CRUD)`
- **Môi trường (Environment):** Frontend Web Admin (`src/App.jsx:110-114`)
- **Mức độ nghiêm trọng (Severity):** **Critical** (Lỗi hủy hoại tính toàn vẹn dữ liệu hiển thị - Vi phạm trực tiếp quy tắc Update Isolation)
- **Mã Test Case liên quan:** `TC-FR15-DT-011`

---

### 1. Mô tả lỗi (Description)
Đặc tả FR-15 nêu rõ quy tắc bất biến: *"Khi Sửa một sản phẩm, chỉ sản phẩm đó bị thay đổi — các sản phẩm khác giữ nguyên."* Tuy nhiên, trong mã nguồn React của Web Admin, hàm xử lý submit khi cập nhật sản phẩm (`handleProductSubmit`) lại cố tình chạy một hàm duyệt mảng (`fakeMassUpdatedProducts = products.map(...)`) và gán tên của tất cả mọi sản phẩm trong danh sách thành tên của sản phẩm vừa sửa. Kết quả là trên giao diện bảng sản phẩm, toàn bộ danh mục sản phẩm của cửa hàng đều bị đổi tên đồng loạt!

### 2. Các bước tái hiện (Steps to Reproduce)
1. Mở trang Web Admin tại `http://localhost:5174`, vào tab "Sản phẩm".
2. Bấm nút "Sửa" tại sản phẩm đầu tiên (ví dụ đang có tên là "iPhone 15").
3. Đổi tên sản phẩm thành "Bàn phím cơ AKKO" và bấm "Lưu sản phẩm".
4. Quan sát bảng danh sách sản phẩm hiển thị ngay sau đó.

### 3. Kết quả mong đợi (Expected Result)
- Theo đặc tả **FR-15**: Chỉ sản phẩm có ID được chọn mới bị thay đổi tên; các sản phẩm còn lại phải giữ nguyên 100% tên và giá ban đầu.

### 4. Kết quả thực tế (Actual Result)
- Toàn bộ các dòng sản phẩm trong bảng (từ sản phẩm 1 đến $N$) đều hiển thị chung tên "Bàn phím cơ AKKO"!
- Đoạn mã lỗi được cài cắm tại `frontend-admin/src/App.jsx:108-114`:
  ```javascript
  if (productForm.id) {
    await axios.put(`${API_URL}/products/${productForm.id}`, productForm);
    const fakeMassUpdatedProducts = products.map((p) => ({
      ...p,
      name: productForm.name,
    }));
    setProducts(fakeMassUpdatedProducts);
    alert("Cập nhật thành công!");
  }
  ```

### 5. Đề xuất khắc phục (Recommendation)
Gọi lại hàm `fetchData()` để lấy danh sách chính xác từ CSDL, hoặc chỉ cập nhật đúng phần tử có ID trùng khớp:
```javascript
if (productForm.id) {
  await axios.put(`${API_URL}/products/${productForm.id}`, productForm);
  fetchData(); // Tải lại danh sách chuẩn từ backend
  alert("Cập nhật thành công!");
}
```

---
- **GitHub Issue Link:** [Issue #15](https://github.com/denny188/CSC13003-eshop/issues/15)
