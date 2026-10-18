# [BUG-MOB07-04] Nhãn hiển thị Tổng tiền trong giỏ hàng hiển thị sai lệch thành 'Tổng tạm tính' thay vì 'Tổng cộng'

- **Feature (FR):** `FR-20: Tính năng Mobile (kế thừa FR-07: Giỏ hàng)`
- **Môi trường (Environment):** Frontend Mobile React Native (`frontend-mobile/App.js:637-639`)
- **Mức độ nghiêm trọng (Severity):** **Medium** (Lỗi vi phạm trực tiếp quy chuẩn đặt nhãn đặc tả nghiệp vụ)
- **Mã Test Case liên quan:** `TC-MOB07-DT-007`

---

### 1. Mô tả lỗi (Description)
Đặc tả FR-07 quy định quy tắc bất biến về nhãn tổng tiền:
- *"Tổng tiền hiển thị nhãn chính xác: **'Tổng cộng'** (không phải 'Tổng tạm tính')."*

Tuy nhiên, trên màn hình Giỏ hàng của ứng dụng di động React Native, dòng hiển thị tổng tiền lại sử dụng nguyên văn nhãn bị nghiêm cấm trong tài liệu: `<Text style={styles.totalText}>Tổng tạm tính: {formatMoney(cartTotal)}</Text>`. Điều này vi phạm trực tiếp yêu cầu chuẩn hóa giao diện giữa các phân hệ của hệ thống EShop.

### 2. Các bước tái hiện (Steps to Reproduce)
1. Thêm một sản phẩm vào giỏ hàng trên ứng dụng Mobile.
2. Chuyển sang màn hình Giỏ hàng.
3. Quan sát dòng hiển thị tổng số tiền ở phía dưới danh sách sản phẩm.

### 3. Kết quả mong đợi (Expected Result)
- Nhãn hiển thị đúng theo đặc tả: **"Tổng cộng: ... ₫"**.

### 4. Kết quả thực tế (Actual Result)
- Nhãn hiển thị là: **"Tổng tạm tính: ... ₫"**.
- Đoạn mã lỗi tại `frontend-mobile/App.js:637-639`:
  ```jsx
  <Text style={styles.totalText}>
    Tổng tạm tính: {formatMoney(cartTotal)}
  </Text>
  ```

### 5. Đề xuất khắc phục (Recommendation)
Sửa nhãn text thành "Tổng cộng":
```jsx
<Text style={styles.totalText}>
  Tổng cộng: {formatMoney(cartTotal)}
</Text>
```

---
- **GitHub Issue Link:** [Issue #26](https://github.com/denny188/CSC13003-eshop/issues/26)
