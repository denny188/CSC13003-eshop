# [BUG-MOB07-03] Nút Xóa sản phẩm khỏi giỏ hàng thực hiện xóa tức thì mà không có Dialog xác nhận

- **Feature (FR):** `FR-20: Tính năng Mobile (kế thừa FR-07: Giỏ hàng)`
- **Môi trường (Environment):** Frontend Mobile React Native (`frontend-mobile/App.js:155-159`, `630-632`)
- **Mức độ nghiêm trọng (Severity):** **Critical** (Lỗi vi phạm quy tắc an toàn dữ liệu người dùng - Bỏ qua yêu cầu bắt buộc của đặc tả)
- **Mã Test Case liên quan:** `TC-MOB07-DT-005`, `TC-MOB07-DT-006`

---

### 1. Mô tả lỗi (Description)
Đặc tả FR-07 nêu rõ yêu cầu bắt buộc:
- *"Nút **Xóa sản phẩm** phải có dialog xác nhận trước khi thực hiện."*

Trên giao diện di động, khi người dùng bấm vào nút "Xóa", hàm `removeFromCart(index)` được gọi ngay lập tức mà **hoàn toàn không bật bất kỳ hộp thoại xác nhận nào (`Alert.alert`)**. Do nút "Xóa" nằm sát cạnh danh sách sản phẩm trên màn hình cảm ứng điện thoại, việc thiếu hộp thoại xác nhận khiến người dùng rất dễ bấm nhầm và bị mất sản phẩm trong giỏ hàng mà không thể khôi phục.

### 2. Các bước tái hiện (Steps to Reproduce)
1. Thêm một vài sản phẩm vào giỏ hàng.
2. Mở màn hình Giỏ hàng.
3. Chạm vào chữ "Xóa" màu đỏ tại một sản phẩm bất kỳ.
4. Quan sát phản hồi của ứng dụng.

### 3. Kết quả mong đợi (Expected Result)
- Ứng dụng phải hiển thị hộp thoại `Alert` với tiêu đề và nội dung xác nhận, ví dụ:
  - Tiêu đề: *"Xác nhận xóa"*
  - Nội dung: *"Bạn có chắc chắn muốn xóa sản phẩm này khỏi giỏ hàng?"*
  - Hai lựa chọn: *"Hủy"* (giữ lại sản phẩm) và *"Xóa"* (xác nhận xóa).

### 4. Kết quả thực tế (Actual Result)
- Sản phẩm biến mất tức thì khỏi giỏ hàng mà không có bất kỳ thông báo hay câu hỏi xác nhận nào!
- Mã nguồn tại `frontend-mobile/App.js:155-159`:
  ```javascript
  const removeFromCart = (index) => {
    const newCart = [...cart];
    newCart.splice(index, 1);
    setCart(newCart);
  };
  ```
  Và tại giao diện render `App.js:630-632`:
  ```jsx
  <TouchableOpacity onPress={() => removeFromCart(index)}>
    <Text style={styles.deleteText}>Xóa</Text>
  </TouchableOpacity>
  ```

### 5. Đề xuất khắc phục (Recommendation)
Sử dụng `Alert.alert()` của React Native để hỏi xác nhận người dùng trước khi gọi lệnh splice mảng:
```javascript
const removeFromCart = (index) => {
  Alert.alert(
    "Xác nhận xóa",
    "Bạn có chắc chắn muốn xóa sản phẩm này khỏi giỏ hàng?",
    [
      { text: "Hủy", style: "cancel" },
      {
        text: "Xóa",
        style: "destructive",
        onPress: () => {
          const newCart = [...cart];
          newCart.splice(index, 1);
          setCart(newCart);
        },
      },
    ]
  );
};
```
