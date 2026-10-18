# [BUG-FR15-03] Ô nhập giá tiền trên Form Web Admin thiếu thuộc tính bắt buộc (required)

- **Feature (FR):** `FR-15: Quản lý Sản phẩm` & `FR-22: Form Requirements`
- **Môi trường (Environment):** Frontend Web Admin (`src/App.jsx:500-508`)
- **Mức độ nghiêm trọng (Severity):** **Medium** (Thiếu validation ở form người dùng)
- **Mã Test Case liên quan:** `TC-FR15-DT-007`

---

### 1. Mô tả lỗi (Description)
Tại form Quản lý Sản phẩm trên Web Admin, trường nhập Tên sản phẩm có khai báo `required`. Tuy nhiên, trường nhập Giá tiền (`price`) lại bị bỏ quên thuộc tính `required`. Do đó, nếu người dùng chỉ nhập Tên mà để trống ô Giá tiền, form vẫn cho phép bấm nút "Lưu sản phẩm" và submit giá trị rỗng/undefined lên backend.

### 2. Các bước tái hiện (Steps to Reproduce)
1. Mở trang Web Admin tại `http://localhost:5174`.
2. Chuyển sang tab "Sản phẩm".
3. Nhập tên sản phẩm: `"Test Không Giá"`, nhưng để trống ô "Giá tiền".
4. Bấm nút "Lưu sản phẩm".
5. Quan sát: Trình duyệt không ngăn chặn mà gửi ngay request tạo sản phẩm.

### 3. Kết quả mong đợi (Expected Result)
- Theo đặc tả **FR-15**: *"Giá: bắt buộc, phải là số dương (> 0)."*
- Theo **FR-22**: Tất cả các trường bắt buộc phải có validation và ký hiệu `*`.
- Trình duyệt phải ngăn submit và bật tooltip yêu cầu nhập trường Giá tiền.

### 4. Kết quả thực tế (Actual Result)
- Mã nguồn tại `frontend-admin/src/App.jsx:500-508`:
  ```jsx
  <input
    placeholder="Giá tiền"
    type="number"
    className="border p-2 rounded"
    value={productForm.price}
    onChange={(e) =>
      setProductForm({ ...productForm, price: e.target.value })
    }
  />
  ```
  Hoàn toàn không có thuộc tính `required` hay `min="1"`.

### 5. Đề xuất khắc phục (Recommendation)
Thêm `required` và `min="1"` vào thẻ input:
```jsx
<input
  placeholder="Giá tiền"
  type="number"
  min="1"
  required
  className="border p-2 rounded"
  ...
/>
```

---
- **GitHub Issue Link:** [Issue #13](https://github.com/denny188/CSC13003-eshop/issues/13)
