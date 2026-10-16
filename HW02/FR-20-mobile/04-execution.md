# Báo cáo Thực thi Kiểm thử (Test Execution Log): FR-20 Mobile (Tính năng Giỏ hàng FR-07)

## 1. Môi trường Thực thi (Execution Environment)
- **Hệ thống thử nghiệm (SUT):** Ứng dụng di động EShop Mobile (`frontend-mobile/App.js` chạy trên nền tảng React Native / Expo) kết nối với Backend API (`http://localhost:3000`).
- **Nền tảng kiểm thử:** React Native Mobile Client & Mã nguồn giao diện người dùng `App.js`.
- **Ngày thực thi:** 2026-10-16
- **Công cụ & Phương pháp:** Code inspection chuyên sâu trên toàn bộ luồng Giỏ hàng React Native (`frontend-mobile/App.js:134-160`, `435-438`, `590-658`), kết hợp phân tích mô phỏng state React (`cart`, `setCart`, `removeFromCart`) và đối chiếu trực tiếp với đặc tả FR-07, FR-20, FR-21, FR-23.

---

## 2. Bảng Kết quả Thực thi Chi tiết (Execution Results Table)

| Mã Test Case | Loại kiểm thử | Mục tiêu kiểm tra | Kết quả mong đợi (Expected Result) | Kết quả thực tế (Actual Result) | Đánh giá (Pass/Fail) | Minh chứng / Bằng chứng (Evidence) | Mã Bug liên kết |
| :--- | :--- | :--- | :--- | :--- | :---: | :--- | :---: |
| **TC-MOB07-DT-001** | Domain | Thêm sản phẩm mới vào giỏ | Thêm 1 dòng mới vào giỏ hàng, số lượng 1, alert thành công *(FR-07)* | Thêm đúng 1 phần tử vào mảng `cart`, hiển thị `Alert.alert("Thành công")` | **PASS** | `App.js:149`: `setCart([...cart, { ...selectedProduct, quantity: safeQuantity }])` | - |
| **TC-MOB07-DT-002** | Domain | **Gộp sản phẩm trùng lặp** | *"Thêm cùng một sản phẩm vào giỏ sẽ tăng số lượng, không tạo dòng mới"* *(FR-07)* | Hệ thống tìm `findIndex` trùng ID, cộng dồn số lượng và không tạo dòng mới | **PASS** | `App.js:136-147`: `existingIndex >= 0 -> newCart[existingIndex].quantity += safeQuantity` | - |
| **TC-MOB07-DT-003** | Domain | **Nút điều chỉnh số lượng (+/-)** | *"Số lượng (có nút +/- để chỉnh)"* *(FR-07)* | **LỖI:** Trên giao diện di động (`App.js:609-625`), dòng sản phẩm **hoàn toàn không có nút `+` hoặc nút `-`**! Người dùng chỉ có một ô `TextInput` nhập chữ, rất bất tiện và vi phạm đặc tả | **FAIL** | Đoạn mã `App.js:609-625` chỉ render `<TextInput>` đơn thuần, hoàn toàn thiếu 2 nút `<TouchableOpacity>` cho `+` và `-` | `BUG-MOB07-01` |
| **TC-MOB07-DT-004** | Domain | **Nhập số lượng trực tiếp trong giỏ** | Nhập số lượng mới, ví dụ gõ `5` thì số lượng cập nhật đúng bằng `5` | **LỖI BẤT THƯỜNG (CỘNG DỒN SAI LOGIC):** Khi người dùng gõ số $X$, mã nguồn cố tình cộng thêm 1: `parsed + 1`! Ví dụ gõ `5` thì biến thành `6`, gõ `1` thì biến thành `2`! | **FAIL** (Critical Bug) | Đoạn mã `App.js:619-621`: `Number.isFinite(parsed) && parsed > 0 ? parsed + 1 : 1;` cố tình cộng thừa 1 đơn vị! | `BUG-MOB07-02` |
| **TC-MOB07-DT-005** | Domain | **Dialog xác nhận khi Xóa sản phẩm** | *"Nút **Xóa sản phẩm** phải có dialog xác nhận trước khi thực hiện"* *(FR-07)* | **LỖI NGHIÊM TRỌNG:** Bấm nút "Xóa" thì sản phẩm **bị xóa ngay lập tức khỏi giỏ hàng mà không hề hiển thị bất kỳ Dialog/Alert xác nhận nào**! | **FAIL** (Critical Bug) | Đoạn mã `App.js:155-159`: `const removeFromCart = (index) => { newCart.splice(index, 1); setCart(newCart); }` gọi trực tiếp không có `Alert.alert(..., [ {text: 'Hủy'}, {text: 'Xóa'} ])` | `BUG-MOB07-03` |
| **TC-MOB07-DT-006** | Domain | Hủy xóa trên Dialog | Dialog cho phép chọn "Hủy" để giữ lại sản phẩm *(FR-07)* | Do không có Dialog xác nhận nên người dùng không có cơ hội Hủy thao tác | **FAIL** | Bị ảnh hưởng trực tiếp bởi lỗi thiếu Dialog xác nhận | `BUG-MOB07-03` |
| **TC-MOB07-DT-007** | Domain | **Nhãn hiển thị Tổng tiền** | *"Tổng tiền hiển thị nhãn chính xác: **'Tổng cộng'** (không phải 'Tổng tạm tính')"* *(FR-07)* | **LỖI VI PHẠM TRỰC TIẾP ĐẶC TẢ:** Giao diện hiển thị nguyên văn nhãn bị cấm: `<Text>Tổng tạm tính: ... ₫</Text>`! | **FAIL** | Đoạn mã `App.js:637-639`: `<Text style={styles.totalText}>Tổng tạm tính: {formatMoney(cartTotal)}</Text>` vi phạm trực tiếp FR-07 | `BUG-MOB07-04` |
| **TC-MOB07-DT-008** | Domain | **Hình minh họa giỏ hàng trống** | *"Giỏ hàng trống phải có hình minh họa và thông báo rõ ràng"* *(FR-07)* | **LỖI:** Khi giỏ hàng trống (`cart.length === 0`), giao diện chỉ render đúng 1 dòng text thuần: `"Giỏ hàng của bạn đang trống"`, **hoàn toàn không có thẻ `<Image>` minh họa**! | **FAIL** | Đoạn mã `App.js:594-600` chỉ có `<Text style={styles.emptyTitle}>Giỏ hàng của bạn đang trống</Text>`, thiếu thẻ `<Image>` | `BUG-MOB07-05` |
| **TC-MOB07-DT-009** | Domain | Nút Tiếp tục mua sắm (nhãn) | Nút quay về trang chủ hiển thị nhãn chuẩn **"Tiếp tục mua sắm"** *(FR-07)* | Khi giỏ có hàng, nút quay về trang chủ bị đặt tên sai lệch thành: **`"← Mua tiếp"`** (`App.js:645`) | **FAIL** | Đoạn mã `App.js:645`: `<Text style={styles.buttonText}>← Mua tiếp</Text>` không đồng bộ chuẩn | `BUG-MOB07-06` |
| **TC-MOB07-DT-010** | Domain | Badge số lượng trên thanh điều hướng | Hiển thị badge số lượng sản phẩm trên Navbar *(FR-23)* | Hiển thị dạng text `Giỏ ({cart.length})` trong menu điều hướng, cập nhật theo state `cart` | **PASS** | `App.js:436`: `<Text style={styles.navText}>Giỏ ({cart.length})</Text>` | - |
| **TC-MOB07-BV-001** | BVA | `selectedQuantity = 1` ($min$) | Thêm thành công sản phẩm với số lượng 1 | Thêm thành công với số lượng 1 | **PASS** | `normalizeQuantity(1)` trả về 1 | - |
| **TC-MOB07-BV-002** | BVA | `selectedQuantity = 0` | Chuẩn hóa về 1 hoặc báo lỗi | Hàm `normalizeQuantity` tự động chuyển về 1 an toàn | **PASS** | `App.js:130-132`: `parsed > 0 ? parsed : 1` | - |
| **TC-MOB07-BV-003** | BVA | `selectedQuantity = -1` | Chuẩn hóa về 1 | Chuẩn hóa về 1 an toàn | **PASS** | `normalizeQuantity(-1)` trả về 1 | - |
| **TC-MOB07-BV-004** | BVA | Giảm số lượng từ 1 về 0 | Giữ nguyên 1 hoặc hỏi xóa, không cho phép hiển thị 0 hoặc âm *(FR-07)* | Nếu người dùng nhập `0` vào ô số lượng giỏ hàng, mã nguồn gán lại thành `1` | **PASS** | `App.js:619`: `parsed > 0 ? parsed + 1 : 1` trả về 1 | - |
| **TC-MOB07-BV-005** | BVA | **Nhập trực tiếp số lượng 1 trong giỏ** | Số lượng cập nhật chính xác là 1 | **LỖI:** Nhập `1` thì hệ thống tự động gán `parsed + 1` thành **`2`**! | **FAIL** | Bị ảnh hưởng bởi lỗi logic `parsed + 1` | `BUG-MOB07-02` |
| **TC-MOB07-BV-006** | BVA | **Nhập trực tiếp số lượng 2 trong giỏ** | Số lượng cập nhật chính xác là 2 | **LỖI:** Nhập `2` thì hệ thống tự động gán thành **`3`**! | **FAIL** | Bị ảnh hưởng bởi lỗi logic `parsed + 1` | `BUG-MOB07-02` |
| **TC-MOB07-BV-007** | BVA | Nhập số lượng lớn `99` | Số lượng cập nhật = 99 (hoặc tính đúng) | Nhập `99` bị biến thành `100` | **FAIL** | Bị ảnh hưởng bởi lỗi logic `parsed + 1` | `BUG-MOB07-02` |
| **TC-MOB07-BV-008** | BVA | Xóa hết sản phẩm (chuyển sang Empty State) | Chuyển mượt sang Empty State có hình minh họa | Chuyển sang Empty State nhưng giao diện thiếu hình minh họa | **FAIL** | Bị ảnh hưởng bởi lỗi thiếu hình minh họa | `BUG-MOB07-05` |
| **TC-MOB07-BV-009** | BVA | Giỏ hàng có 10 sản phẩm (Cuộn trang) | Màn hình cuộn mượt mà bằng `ScrollView` | Sử dụng `<ScrollView style={styles.cartList}>` cuộn tốt | **PASS** | `App.js:603` bọc trong `ScrollView` | - |

---

## 3. Thống kê Kết quả Thực thi (Execution Summary)

- **Tổng số ca kiểm thử đã thiết kế (Designed):** **19**
- **Tổng số ca kiểm thử đã thực thi (Executed):** **19**
- **Số ca kiểm thử ĐẠT (Passed):** **8** (42.1%)
- **Số ca kiểm thử THẤT BẠI (Failed):** **11** (57.9%)
- **Số ca kiểm thử chưa chạy (Not executed):** **0**
- **Tổng số lỗi phát hiện (Bugs identified):** **6 lỗi** (`BUG-MOB07-01` đến `BUG-MOB07-06`)

---

## 4. Danh sách các Lỗi Phát hiện (Bug List)
1. **`BUG-MOB07-01`**: Màn hình Giỏ hàng Mobile thiếu hoàn toàn nút tăng/giảm số lượng (`+` và `-`) trên từng dòng sản phẩm (Vi phạm FR-07: *"Số lượng (có nút +/- để chỉnh)"*).
2. **`BUG-MOB07-02`**: Lỗi logic bất thường khi chỉnh sửa số lượng trong giỏ hàng: Khi người dùng nhập số lượng `parsed`, hệ thống tự động cộng thêm 1 (`parsed + 1`), làm sai lệch toàn bộ số lượng và thành tiền.
3. **`BUG-MOB07-03`**: Nút Xóa sản phẩm xóa ngay lập tức mà không có Dialog xác nhận (Vi phạm trực tiếp FR-07: *"Nút Xóa sản phẩm phải có dialog xác nhận trước khi thực hiện"*).
4. **`BUG-MOB07-04`**: Nhãn hiển thị Tổng tiền sai lệch đặc tả: Hiển thị nhãn bị cấm *"Tổng tạm tính"* thay vì *"Tổng cộng"* (Vi phạm FR-07: *"Tổng tiền hiển thị nhãn chính xác: 'Tổng cộng' (không phải 'Tổng tạm tính')"*).
5. **`BUG-MOB07-05`**: Giao diện Giỏ hàng trống (Empty State) thiếu hình ảnh minh họa (Vi phạm FR-07: *"Giỏ hàng trống phải có hình minh họa và thông báo rõ ràng"*).
6. **`BUG-MOB07-06`**: Nút điều hướng quay lại trang chủ dùng sai nhãn: Hiển thị `"← Mua tiếp"` thay vì `"Tiếp tục mua sắm"`.
