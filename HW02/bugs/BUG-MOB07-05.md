# [BUG-MOB07-05] Màn hình Giỏ hàng trống (Empty State) thiếu hình ảnh minh họa

- **Feature (FR):** `FR-20: Tính năng Mobile (kế thừa FR-07: Giỏ hàng)`
- **Môi trường (Environment):** Frontend Mobile React Native (`frontend-mobile/App.js:594-601`)
- **Mức độ nghiêm trọng (Severity):** **Medium** (Lỗi thiếu thành phần thị giác đồ họa theo đặc tả - Ảnh hưởng tiêu chuẩn UI/UX)
- **Mã Test Case liên quan:** `TC-MOB07-DT-008`, `TC-MOB07-BV-008`

---

### 1. Mô tả lỗi (Description)
Đặc tả FR-07 quy định:
- *"Giỏ hàng trống phải có **hình minh họa** và thông báo rõ ràng."*

Trên ứng dụng di động React Native, khi giỏ hàng chưa có sản phẩm nào (`cart.length === 0`), giao diện hiển thị component `emptyBox` chỉ bao gồm duy nhất 1 thẻ `<Text>` hiển thị thông điệp "Giỏ hàng của bạn đang trống" và nút bấm quay về trang chủ. Màn hình hoàn toàn không có bất kỳ hình ảnh minh họa (`<Image>`) hoặc icon đồ họa nào miêu tả chiếc giỏ hàng trống, khiến giao diện trở nên đơn điệu và vi phạm yêu cầu của tài liệu đặc tả.

### 2. Các bước tái hiện (Steps to Reproduce)
1. Mở ứng dụng Mobile khi mới khởi động (chưa thêm sản phẩm nào vào giỏ).
2. Chạm vào mục "Giỏ" trên thanh điều hướng.
3. Quan sát giao diện màn hình giỏ hàng trống.

### 3. Kết quả mong đợi (Expected Result)
- Màn hình hiển thị đầy đủ:
  - Một hình ảnh hoặc vector minh họa (`<Image>`) giỏ hàng trống thân thiện, thẩm mỹ.
  - Thông báo: "Giỏ hàng của bạn đang trống".
  - Nút bấm: "Tiếp tục mua sắm".

### 4. Kết quả thực tế (Actual Result)
- Giao diện hoàn toàn thiếu hình ảnh minh họa:
  ```jsx
  {cart.length === 0 ? (
    <View style={styles.emptyBox}>
      {/* THIẾU THẺ <Image> MINH HỌA */}
      <Text style={styles.emptyTitle}>Giỏ hàng của bạn đang trống</Text>
      <TouchableOpacity onPress={goHome}>
        <Text style={styles.linkText}>Tiếp tục mua sắm</Text>
      </TouchableOpacity>
    </View>
  ) : ...}
  ```

### 5. Đề xuất khắc phục (Recommendation)
Thêm thẻ `<Image>` minh họa giỏ hàng trống vào component:
```jsx
<View style={styles.emptyBox}>
  <Image
    source={{ uri: "https://cdn-icons-png.flaticon.com/512/11329/11329060.png" }}
    style={styles.emptyCartImage}
    resizeMode="contain"
  />
  <Text style={styles.emptyTitle}>Giỏ hàng của bạn đang trống</Text>
  <TouchableOpacity style={styles.continueShoppingBtn} onPress={goHome}>
    <Text style={styles.continueShoppingText}>Tiếp tục mua sắm</Text>
  </TouchableOpacity>
</View>
```
