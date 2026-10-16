# Phân tích Khoảng trống AI (AI Gap Analysis): FR-20 Mobile (Tính năng Giỏ hàng FR-07)

Báo cáo phân tích và phản biện năng lực AI trong quá trình kiểm thử tính năng Giỏ hàng trên ứng dụng di động React Native (FR-20 / FR-07), đáp ứng chuẩn đầu ra Bloom-AI cấp độ G9.3 (Phân tích & Đánh giá).

---

## 1. Các Ca Kiểm thử mà AI bỏ sót ban đầu (Missed Test Cases)

1. **Kiểm tra hành vi nhập liệu trực tiếp tại ô số lượng (`onChangeText` handling in TextInput):**
   - *Mô tả:* Nhập trực tiếp các số cụ thể (ví dụ gõ số `1`, `2`, `5`) vào ô nhập số lượng của từng dòng sản phẩm trong giỏ hàng, sau đó kiểm tra xem giá trị lưu trong state `cart` có khớp chính xác với số vừa gõ hay không.
   - *Đánh giá:* AI thường giả định rằng các sự kiện `onChangeText` cơ bản chỉ đơn thuần gán giá trị người dùng nhập (`setCart(text)`). Nếu không trực tiếp soi vào logic xử lý chuỗi và ép kiểu `parseInt` bên trong component, AI sẽ bỏ lọt hoàn toàn bẫy lỗi cài cắm `parsed + 1` làm sai lệch số lượng.
2. **Kiểm tra sự hiện diện của thành phần minh họa thị giác (Visual Asset Check) trong Empty State:**
   - *Mô tả:* Khi giỏ hàng rỗng, kiểm tra xem giao diện có render thẻ hình ảnh `<Image>` minh họa giỏ hàng trống hay chỉ render thẻ `<Text>` thông báo đơn thuần.
   - *Đánh giá:* Các công cụ kiểm thử tự động hoặc AI kiểm thử chức năng thường chỉ kiểm tra "có thông báo giỏ hàng trống hay không" (kiểm tra text tồn tại) mà bỏ qua yêu cầu đặc tả về yếu tố đồ họa: *"Giỏ hàng trống phải có hình minh họa"*.

---

## 2. Các Lỗi (Bugs) mà AI có thể đã bỏ sót nếu chỉ nhìn lướt qua

1. **Bẫy lỗi cộng dồn số lượng sai logic (`BUG-MOB07-02`):**
   - *Phân tích:* Đây là lỗi tinh vi và nguy hiểm nhất trong mã nguồn React Native `App.js`. Đoạn mã `newCart[index].quantity = Number.isFinite(parsed) && parsed > 0 ? parsed + 1 : 1;` được cố ý cài cắm để gây sai lệch số lượng khi người dùng chỉnh sửa trong giỏ. Nếu kiểm thử viên chỉ bấm nút thêm ở trang chủ (hàm `addToCart` hoạt động đúng) mà không kiểm tra thao tác gõ số trong giỏ, lỗi này sẽ hoàn toàn bị bỏ sót.
2. **Lỗi xóa sản phẩm thiếu Dialog xác nhận (`BUG-MOB07-03`):**
   - *Phân tích:* Khi bấm nút "Xóa", sản phẩm biến mất khỏi giỏ hàng thành công. Nếu chỉ nhìn từ góc độ Happy Path (sản phẩm đã được xóa), AI sẽ đánh giá là "PASS". Nhưng đối chiếu với quy định an toàn người dùng trong FR-07: *"Nút Xóa sản phẩm phải có dialog xác nhận trước khi thực hiện"*, việc thiếu `Alert.alert()` để xác nhận là một lỗi vi phạm nghiêm trọng trải nghiệm người dùng trên thiết bị di động (nơi ngón tay rất dễ chạm nhầm).
3. **Lỗi sai lệch nhãn hiển thị Tổng tiền (`BUG-MOB07-04`):**
   - *Phân tích:* Người lập trình thường dùng cụm từ *"Tổng tạm tính"* như một thói quen. Tuy nhiên, đặc tả FR-07 đã ghi rõ ràng quy tắc loại trừ: *"Tổng tiền hiển thị nhãn chính xác: 'Tổng cộng' (không phải 'Tổng tạm tính')"*. AI nếu không đọc kỹ phần trong ngoặc đơn của đặc tả sẽ xem nhãn "Tổng tạm tính" là bình thường và bỏ qua lỗi này.

---

## 3. Lý do AI bỏ sót hoặc có hạn chế (Why the AI Missed It)

1. **Chất lượng câu lệnh (Prompt Quality):**
   - Nếu câu lệnh kiểm thử chỉ yêu cầu "kiểm tra chức năng giỏ hàng mobile" một cách chung chung, AI sẽ tập trung vào luồng CRUD cơ bản (thêm, hiển thị, xóa) mà không rà soát các ràng buộc chi tiết về UI/UX (nhãn text chính xác, hộp thoại xác nhận, nút +/-).
2. **Xu hướng đánh giá theo hành vi thay vì theo đặc tả (Functional Bias):**
   - AI thường coi việc xóa thành công hay hiển thị thông báo text là đã "đạt yêu cầu", trong khi đặc tả phần mềm đòi hỏi sự tuân thủ nghiêm ngặt về cả hình thức (phải có dialog, phải có hình ảnh minh họa).
3. **Mã nguồn React Native lồng ghép trong một file duy nhất:**
   - Việc toàn bộ ứng dụng di động nằm trọn trong một file `App.js` dài hơn 1300 dòng dễ khiến các công cụ AI phân tích bề mặt bị quá tải hoặc bỏ sót các đoạn code inline nằm sâu bên trong hàm render (`renderCart`).

---

## 4. Những điều chỉnh của Sinh viên sau khi Review (What I Changed After Review)

- **Soi chiếu trực tiếp logic `onChangeText`:** Trực tiếp yêu cầu AI thiết kế ca kiểm thử nhập số lượng cụ thể (`TC-MOB07-DT-004`, `TC-MOB07-BV-005`, `TC-MOB07-BV-006`) và kiểm tra từng dòng mã trong `App.js:615-623`, từ đó phát hiện trọn vẹn lỗi `BUG-MOB07-02` (`parsed + 1`).
- **Bắt buộc kiểm tra hộp thoại xác nhận khi xóa:** Yêu cầu AI kiểm tra hàm `removeFromCart` để xác minh có gọi `Alert.alert` hay không, ghi nhận thành công lỗi `BUG-MOB07-03`.
- **Rà soát từng chữ nhãn giao diện và tài nguyên hình ảnh:** Bắt buộc đối chiếu nhãn "Tổng cộng" vs "Tổng tạm tính" (`BUG-MOB07-04`), nút "Tiếp tục mua sắm" (`BUG-MOB07-06`), và thẻ `<Image>` trong Empty State (`BUG-MOB07-05`).
