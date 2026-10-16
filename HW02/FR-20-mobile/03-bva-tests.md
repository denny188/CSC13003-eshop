# Báo cáo Phân tích Giá trị Biên (Boundary Value Analysis - BVA): FR-20 Mobile (Tính năng Giỏ hàng FR-07)

## 1. Phương pháp & Nguyên tắc Áp dụng
1. Áp dụng kỹ thuật **Boundary Value Analysis (BVA)** cho các biến số học có thứ tự và các giá trị số lượng trong giỏ hàng trên ứng dụng di động React Native:
   - **Số lượng sản phẩm khi thêm từ trang chi tiết (`selectedQuantity` boundaries):** Ràng buộc đặc tả là *"số nguyên dương, tối thiểu là 1"* (FR-06, FR-07).
     - Cận dưới: $min - 1 = 0$ (hoặc số âm $-1$).
     - Cận dưới hợp lệ: $min = 1$.
     - Cận lân cận: $min + 1 = 2$.
     - Cận trên lớn: $quantity = 99$, $100$ (hoặc giá trị rất lớn để kiểm tra tràn giao diện).
   - **Số lượng sản phẩm khi chỉnh sửa trong giỏ hàng (`item.quantity` boundaries):**
     - Cận dưới: Giảm về $1$ ($min$), bấm giảm tiếp về $0$ hoặc nhập $0$ trực tiếp.
     - Kiểm tra hành vi khi nhập trực tiếp các giá trị số học đặc biệt ($0$, $1$, $2$, $9999$).
   - **Số lượng mặt hàng trong giỏ (`cart.length` distinct items boundaries):**
     - $0$ mặt hàng: Giỏ hàng trống hoàn toàn (Cận dưới).
     - $1$ mặt hàng: Giỏ có đúng 1 sản phẩm.
     - $2$ mặt hàng: Giỏ có 2 sản phẩm khác nhau.
     - Nhiều mặt hàng (Scroll boundary: 10 mặt hàng) để kiểm tra thanh cuộn `ScrollView` trên màn hình di động.
   - **Tổng tiền giỏ hàng (`cartTotal` boundaries):**
     - Cận 0 ₫ (giỏ trống).
     - Cận giá trị nhỏ nhất ($1$ ₫ hoặc đơn giá nhỏ nhất).
     - Cận giá trị tiền tệ lớn (kiểm tra tràn dòng hiển thị text trên màn hình điện thoại).
2. Tránh trùng lặp với Step 2, tập trung sâu vào các điểm chuyển trạng thái biên (Boundary transitions).

---

## 2. Bảng Phân tích Các Điểm Biên (Boundary Identification)

### 2.1 Biên Số lượng thêm vào giỏ (`selectedQuantity` & `item.quantity`)
- **Cận dưới ($min = 1$):**
  - $quantity = 0$: Vi phạm ràng buộc số nguyên dương $\ge 1$.
  - $quantity = -1$: Số âm, không hợp lệ.
  - $quantity = 1$: Số lượng nguyên dương nhỏ nhất hợp lệ.
  - $quantity = 2$: Điểm biên lân cận $min + 1$.
- **Cận trên (Stress / Layout Boundary):**
  - $quantity = 99$: Số lượng hai chữ số phổ biến.
  - $quantity = 9999$: Số lượng rất lớn, kiểm tra tính toán tổng tiền và bố cục hiển thị trên màn hình nhỏ.

---

### 2.2 Biên Số lượng dòng sản phẩm trong giỏ (`cart.length`)
- $length = 0$: Giỏ hàng trống (Kích hoạt giao diện Empty State).
- $length = 1$: Giỏ hàng bắt đầu có dữ liệu (Chuyển từ Empty State sang List State).
- $length = 2$: Danh sách có từ 2 sản phẩm trở lên.
- $length = 10$: Kiểm tra khả năng cuộn trang (`ScrollView`) và cố định nút thanh toán / tổng tiền ở dưới.

---

### 2.3 Biên Điều chỉnh giảm số lượng từ 1 xuống 0
- Khi số lượng đang ở mức $min = 1$, bấm nút giảm (`-`):
  - Hệ thống phải giữ nguyên mức $1$ (chặn không cho giảm tiếp), hoặc hiển thị hộp thoại hỏi xóa sản phẩm. Tuyệt đối không cho phép số lượng chuyển thành $0$ hoặc số âm.

---

## 3. Bảng Ca Kiểm thử Giá trị Biên (BVA Test Cases)

| Mã Test Case | Biến & Điểm biên kiểm tra | Loại biên (Valid / Invalid) | Dữ liệu kiểm thử (Test Data) | Các bước thực hiện | Kết quả mong đợi (Expected Result) | Độ ưu tiên | Trạng thái | Ghi chú & Đối chiếu Step 2 |
| :--- | :--- | :---: | :--- | :--- | :--- | :---: | :---: | :--- |
| **TC-MOB07-BV-001** | `selectedQuantity` = 1 ($min$) | Valid Boundary | `quantity`: `1` | 1. Mở chi tiết sản phẩm<br>2. Giữ nguyên số lượng là 1<br>3. Bấm Thêm vào giỏ | Thêm thành công sản phẩm với số lượng 1 *(FR-06, FR-07)* | High | Not executed | Cận dưới hợp lệ khi thêm giỏ hàng |
| **TC-MOB07-BV-002** | `selectedQuantity` = 0 ($min - 1$) | Invalid Boundary | `quantity`: `0` | 1. Nhập số lượng = 0 tại trang chi tiết<br>2. Bấm Thêm vào giỏ | Hệ thống tự chuẩn hóa về 1 hoặc báo lỗi số lượng phải từ 1 trở lên *(FR-06, FR-07)* | High | Not executed | Cận dưới không hợp lệ |
| **TC-MOB07-BV-003** | `selectedQuantity` = -1 | Invalid Boundary | `quantity`: `-1` | 1. Nhập số lượng = -1 tại trang chi tiết<br>2. Bấm Thêm vào giỏ | Hệ thống từ chối hoặc chuẩn hóa về 1, không cho phép số âm *(FR-06)* | Medium | Not executed | Số lượng âm |
| **TC-MOB07-BV-004** | `item.quantity` = 1 -> Giảm xuống 0 | Boundary Transition | `quantity` đang = 1 | 1. Tại màn hình Giỏ hàng, sản phẩm đang có số lượng = 1<br>2. Thao tác giảm số lượng (bấm nút giảm hoặc nhập 0) | Hệ thống giữ nguyên số lượng là 1 hoặc hiển thị dialog xác nhận xóa, không cho phép hiển thị số lượng 0 hoặc âm *(FR-07)* | Critical | Not executed | Điểm biên chuyển trạng thái nhạy cảm |
| **TC-MOB07-BV-005** | Nhập `item.quantity` = 1 trực tiếp trong giỏ | Valid Boundary | Nhập `1` vào ô số lượng giỏ hàng | 1. Chọn ô số lượng trong giỏ hàng<br>2. Xóa và gõ số `1` | Số lượng cập nhật chính xác là 1, thành tiền = đơn giá *(FR-07)* | Critical | Not executed | Kiểm tra xử lý chuỗi nhập liệu tại ô text input |
| **TC-MOB07-BV-006** | Nhập `item.quantity` = 2 trực tiếp trong giỏ | Valid Boundary | Nhập `2` vào ô số lượng giỏ hàng | 1. Chọn ô số lượng trong giỏ hàng<br>2. Gõ số `2` | Số lượng cập nhật chính xác là 2, thành tiền $= 2 \times price$ *(FR-07)* | Critical | Not executed | Điểm biên lân cận $min + 1$ |
| **TC-MOB07-BV-007** | Nhập `item.quantity` = 99 (Số lượng lớn) | Stress Boundary | Nhập `99` | 1. Nhập số lượng = 99<br>2. Quan sát thành tiền và tổng cộng | Tính toán chính xác $= 99 \times price$, giao diện hiển thị gọn gàng, không bị vỡ layout trên mobile *(FR-21)* | Medium | Not executed | Kiểm tra layout số lượng lớn |
| **TC-MOB07-BV-008** | `cart.length` chuyển từ 1 về 0 (Xóa hết sản phẩm) | State Transition Boundary | Giỏ hàng còn đúng 1 sản phẩm cuối cùng | 1. Thực hiện xóa sản phẩm duy nhất trong giỏ hàng<br>2. Xác nhận xóa trên dialog | Màn hình lập tức chuyển đổi mượt mà sang trạng thái Giỏ hàng trống (Empty State) với đầy đủ hình minh họa và nút tiếp tục mua sắm *(FR-07, FR-24)* | High | Not executed | Chuyển đổi trạng thái giao diện |
| **TC-MOB07-BV-009** | `cart.length` = 10 (Danh sách dài cần cuộn trang) | Layout Boundary | Thêm 10 sản phẩm khác nhau vào giỏ hàng | 1. Thêm 10 sản phẩm vào giỏ<br>2. Mở màn hình giỏ hàng và cuộn trang | Danh sách cuộn mượt mà bằng `ScrollView`, các phần tử tổng tiền và nút bấm thanh toán không bị che khuất hoặc tràn màn hình *(FR-20, FR-21)* | Medium | Not executed | Kiểm tra tràn màn hình cảm ứng |

---

## 4. Tổng kết Độ phủ Giá trị Biên (BVA Coverage Summary)

- **Tổng số ca kiểm thử BVA mới bổ sung:** **9 test cases** (`TC-MOB07-BV-001` đến `TC-MOB07-BV-009`).
- **Phân bố kiểm tra biên:**
  - Biên số lượng thêm vào giỏ ($quantity = 0, -1, 1$): 3 test cases.
  - Biên chuyển trạng thái số lượng từ 1 về 0: 1 test case.
  - Biên nhập liệu trực tiếp tại ô số lượng trong giỏ ($quantity = 1, 2, 99$): 3 test cases.
  - Biên số lượng mặt hàng và chuyển trạng thái giỏ hàng ($length = 0, 1 \rightarrow 0, 10$): 2 test cases.
- Kết hợp với 10 test cases của Domain Testing (Step 2), tổng số ca kiểm thử thiết kế cho FR-20 Mobile (Giỏ hàng FR-07) là **19 test cases**.
