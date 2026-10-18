# [BUG-MOB07-01] Màn hình Giỏ hàng trên Mobile thiếu các nút bấm tăng/giảm (+/-) số lượng trên từng dòng sản phẩm

- **Feature (FR):** `FR-20: Tính năng Mobile (kế thừa FR-07: Giỏ hàng)`
- **Môi trường (Environment):** Frontend Mobile React Native (`frontend-mobile/App.js:609-625`)
- **Mức độ nghiêm trọng (Severity):** **Medium** (Lỗi thiếu thành phần giao diện theo đặc tả - Ảnh hưởng nghiêm trọng đến trải nghiệm di động)
- **Mã Test Case liên quan:** `TC-MOB07-DT-003`

---

### 1. Mô tả lỗi (Description)
Đặc tả FR-07 quy định rõ cấu trúc hiển thị của giỏ hàng:
- *"Hiển thị danh sách sản phẩm với các cột: Sản phẩm, Đơn giá, **Số lượng (có nút +/- để chỉnh)**, Thành tiền, Thao tác."*

Trên giao diện ứng dụng di động React Native, mỗi dòng sản phẩm trong giỏ chỉ hiển thị một ô `TextInput` để nhập số bàn phím, **hoàn toàn không có hai nút bấm tăng (`+`) và giảm (`-`)**. Trên thiết bị di động với màn hình cảm ứng, việc bắt người dùng phải chạm vào ô nhỏ, bật bàn phím ảo rồi xóa/gõ số thay vì chạm nhanh vào nút `+`/`-` gây khó khăn lớn cho thao tác và vi phạm trực tiếp yêu cầu của đặc tả.

### 2. Các bước tái hiện (Steps to Reproduce)
1. Mở ứng dụng di động EShop Mobile trên máy ảo hoặc thiết bị thật.
2. Thêm một sản phẩm bất kỳ vào giỏ hàng.
3. Chạm vào mục "Giỏ" trên thanh điều hướng để mở màn hình Giỏ Hàng.
4. Quan sát dòng hiển thị sản phẩm tại phần "Số lượng".

### 3. Kết quả mong đợi (Expected Result)
- Bên cạnh ô hiển thị số lượng phải có hai nút bấm:
  - Nút `-`: Giảm số lượng đi 1 đơn vị.
  - Nút `+`: Tăng số lượng lên 1 đơn vị.

### 4. Kết quả thực tế (Actual Result)
- Giao diện chỉ có dòng text "Số lượng:" và ô `<TextInput>`:
  ```jsx
  <View style={styles.inlineRow}>
    <Text>Số lượng:</Text>
    <TextInput
      style={styles.cartQuantityInput}
      keyboardType="numeric"
      value={String(item.quantity ?? "")}
      onChangeText={(text) => { ... }}
    />
  </View>
  ```
  Hoàn toàn không có bất kỳ nút bấm `<TouchableOpacity>` nào cho thao tác `+` hoặc `-`.

### 5. Đề xuất khắc phục (Recommendation)
Bổ sung hai nút bấm `+` và `-` bao quanh ô số lượng và gắn hàm xử lý tương ứng:
```jsx
<View style={styles.quantityControls}>
  <TouchableOpacity
    style={styles.qtyBtn}
    onPress={() => updateQuantity(index, item.quantity - 1)}
  >
    <Text style={styles.qtyBtnText}>-</Text>
  </TouchableOpacity>
  <TextInput
    style={styles.cartQuantityInput}
    keyboardType="numeric"
    value={String(item.quantity ?? "")}
    onChangeText={(text) => handleQuantityChange(index, text)}
  />
  <TouchableOpacity
    style={styles.qtyBtn}
    onPress={() => updateQuantity(index, item.quantity + 1)}
  >
    <Text style={styles.qtyBtnText}>+</Text>
  </TouchableOpacity>
</View>
```

---
- **GitHub Issue Link:** [Issue #23](https://github.com/denny188/CSC13003-eshop/issues/23)
