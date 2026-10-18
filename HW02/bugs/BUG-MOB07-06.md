# [BUG-MOB07-06] Nút điều hướng quay về trang chủ bị đặt sai nhãn thành '← Mua tiếp' thay vì 'Tiếp tục mua sắm'

- **Feature (FR):** `FR-20: Tính năng Mobile (kế thừa FR-07: Giỏ hàng)`
- **Môi trường (Environment):** Frontend Mobile React Native (`frontend-mobile/App.js:644-646`)
- **Mức độ nghiêm trọng (Severity):** **Low** (Lỗi không nhất quán thuật ngữ giao diện)
- **Mã Test Case liên quan:** `TC-MOB07-DT-009`

---

### 1. Mô tả lỗi (Description)
Đặc tả FR-07 quy định:
- *"Có nút **Tiếp tục mua sắm** để quay về trang chủ."*

Tại màn hình Giỏ hàng khi đang có sản phẩm, nút hành động bên cạnh nút "Tiến hành thanh toán" lại được đặt tên là: `<Text style={styles.buttonText}>← Mua tiếp</Text>`. Việc dùng từ viết tắt mang tính khẩu ngữ này không đồng bộ với nhãn chuẩn "Tiếp tục mua sắm" được quy định trong đặc tả và không nhất quán với nhãn của trạng thái giỏ hàng trống (`App.js:598`).

### 2. Các bước tái hiện (Steps to Reproduce)
1. Thêm một sản phẩm vào giỏ hàng trên ứng dụng Mobile.
2. Mở màn hình Giỏ hàng.
3. Quan sát hàng nút bấm hành động ở cuối màn hình.

### 3. Kết quả mong đợi (Expected Result)
- Nút bấm bên trái có nhãn chính xác: **"Tiếp tục mua sắm"**.

### 4. Kết quả thực tế (Actual Result)
- Nút bấm hiển thị nhãn là: **"← Mua tiếp"**.
- Đoạn mã tại `frontend-mobile/App.js:641-646`:
  ```jsx
  <TouchableOpacity
    style={[styles.grayButton, styles.actionFlex]}
    onPress={goHome}
  >
    <Text style={styles.buttonText}>← Mua tiếp</Text>
  </TouchableOpacity>
  ```

### 5. Đề xuất khắc phục (Recommendation)
Sửa nhãn text thành:
```jsx
<Text style={styles.buttonText}>Tiếp tục mua sắm</Text>
```

---
- **GitHub Issue Link:** [Issue #28](https://github.com/denny188/CSC13003-eshop/issues/28)
