# [BUG-MOB07-02] Nhập số lượng trong giỏ hàng bị tự động cộng thừa 1 đơn vị (parsed + 1), làm sai lệch toàn bộ thành tiền

- **Feature (FR):** `FR-20: Tính năng Mobile (kế thừa FR-07: Giỏ hàng)`
- **Môi trường (Environment):** Frontend Mobile React Native (`frontend-mobile/App.js:615-623`)
- **Mức độ nghiêm trọng (Severity):** **Critical** (Lỗi logic tính toán cốt lõi - Cài cắm cố ý làm sai lệch số lượng và tiền tệ)
- **Mã Test Case liên quan:** `TC-MOB07-DT-004`, `TC-MOB07-BV-005`, `TC-MOB07-BV-006`, `TC-MOB07-BV-007`

---

### 1. Mô tả lỗi (Description)
Khi người dùng nhập số lượng mong muốn vào ô `TextInput` trên dòng sản phẩm trong giỏ hàng (ví dụ: muốn mua 2 sản phẩm và gõ số `2`), mã nguồn React Native lại cố tình lấy giá trị vừa phân tích cộng thêm 1 đơn vị (`parsed + 1`). Hậu quả là người dùng gõ số 1 thì giỏ hàng nhận là 2; gõ số 2 thì nhận là 3; gõ số 5 thì biến thành 6! Điều này khiến số lượng hàng mua luôn bị tăng thừa ngoài ý muốn, làm sai lệch toàn bộ thành tiền và tổng giá trị đơn hàng.

### 2. Các bước tái hiện (Steps to Reproduce)
1. Thêm một sản phẩm có giá 100,000 ₫ vào giỏ hàng.
2. Mở màn hình Giỏ hàng.
3. Chạm vào ô nhập số lượng (hiện đang hiển thị số 1), xóa đi và gõ số `1`.
4. Chạm ra ngoài và quan sát số lượng cùng thành tiền hiển thị.

### 3. Kết quả mong đợi (Expected Result)
- Số lượng sản phẩm phải giữ nguyên là `1`.
- Thành tiền phải là: $1 \times 100,000 = 100,000$ ₫.

### 4. Kết quả thực tế (Actual Result)
- Ô số lượng lập tức nhảy lên thành **`2`**!
- Thành tiền bị tính thành $2 \times 100,000 = 200,000$ ₫.
- Nếu gõ tiếp số `5`, số lượng tự nhảy lên thành **`6`** và thành tiền là $600,000$ ₫.
- Đoạn mã lỗi được cài cắm tại `frontend-mobile/App.js:615-623`:
  ```javascript
  onChangeText={(text) => {
    const newCart = [...cart];
    const parsed = parseInt(text, 10);
    newCart[index].quantity =
      Number.isFinite(parsed) && parsed > 0
        ? parsed + 1 // LỖI NGHIÊM TRỌNG: Cố tình cộng thêm 1 đơn vị!
        : 1;
    setCart(newCart);
  }}
  ```

### 5. Đề xuất khắc phục (Recommendation)
Sửa lại biểu thức gán, gán trực tiếp giá trị `parsed` hợp lệ mà không cộng thêm:
```javascript
onChangeText={(text) => {
  const newCart = [...cart];
  const parsed = parseInt(text, 10);
  newCart[index].quantity =
    Number.isFinite(parsed) && parsed > 0 ? parsed : 1;
  setCart(newCart);
}}
```

---
- **GitHub Issue Link:** [Issue #24](https://github.com/denny188/CSC13003-eshop/issues/24)
