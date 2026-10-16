# Phân tích Yêu cầu Kiểm thử: FR-20 Mobile (Tính năng Giỏ hàng - FR-07 trên Mobile App)

## 1. Trích dẫn Yêu cầu từ Đặc tả (Spec Excerpts)

### 1.1 Đặc tả chức năng phân hệ Mobile (README.md - FR-20)
- **FR-20: Tính năng Mobile (React Native):**
  - *"Đầy đủ các chức năng: Xem sản phẩm, Đăng nhập, Đăng xuất, Đăng ký, Giỏ hàng, Thanh toán, Hồ sơ, Lịch sử đơn hàng."*
  - *"Chức năng Hủy đơn hàng tuân theo đúng State Machine ở FR-10 (chỉ được hủy khi `pending` hoặc `confirmed`)."*
  - Theo phạm vi được chọn: Kiểm thử tính năng **Giỏ hàng (Shopping Cart)** trên nền tảng ứng dụng di động React Native theo các quy chuẩn nghiệp vụ của **FR-07** kết hợp các tiêu chuẩn giao diện và phản hồi di động (**FR-21, FR-23, FR-24**).

### 1.2 Đặc tả nghiệp vụ Giỏ hàng (README.md - FR-07)
- **Hiển thị giỏ hàng:**
  - Danh sách sản phẩm bao gồm các thông tin: **Sản phẩm**, **Đơn giá**, **Số lượng** (có nút `+` / `-` để điều chỉnh), **Thành tiền**, và **Thao tác** (Xóa).
- **Quy tắc gộp sản phẩm trùng lặp (Merge identical items):**
  - *"Thêm cùng một sản phẩm vào giỏ sẽ tăng số lượng, không tạo dòng mới."*
- **Quy tắc xác nhận khi xóa sản phẩm:**
  - *"Nút **Xóa sản phẩm** phải có dialog xác nhận trước khi thực hiện."*
- **Điều hướng tiếp tục mua sắm:**
  - *"Có nút **Tiếp tục mua sắm** để quay về trang chủ."*
- **Nhãn hiển thị tổng tiền:**
  - *"Tổng tiền hiển thị nhãn chính xác: **'Tổng cộng'** (không phải 'Tổng tạm tính')."*
- **Trạng thái giỏ hàng trống (Empty State):**
  - *"Giỏ hàng trống phải có hình minh họa và thông báo rõ ràng."*

### 1.3 Tiêu chuẩn Giao diện & Điều hướng áp dụng cho Mobile (FR-21, FR-23, FR-24)
- **FR-21: Tiêu chuẩn giao diện chung:**
  - Nhất quán ngôn ngữ tiếng Việt.
  - Định dạng tiền tệ: Luôn dùng ký hiệu `₫` với định dạng phân cách hàng nghìn.
- **FR-23: Navigation & Badge:**
  - Link/Tab "Giỏ hàng" trên thanh điều hướng phải hiển thị **badge số lượng** sản phẩm trong giỏ (tổng số lượng sản phẩm hoặc số mặt hàng).
- **FR-24: Phản hồi hành động (Feedback & Toast):**
  - Khi thêm vào giỏ hàng hoặc thao tác số lượng phải có phản hồi trực quan, thông báo rõ ràng, không gây giật lag hoặc treo ứng dụng.

---

## 2. Bảng Phân tích Biến Đầu vào (Input Variables)

| Tên biến | Vị trí (UI / Screen) | Kiểu dữ liệu | Ràng buộc giá trị / Định dạng | Bắt buộc? | Nguồn đặc tả (Spec Ref) |
| :--- | :--- | :--- | :--- | :---: | :--- |
| `selectedProduct` | Màn hình Home / Product Detail | Object (`id`, `name`, `price`, `imageUrl`) | Phải là sản phẩm tồn tại hợp lệ với `id > 0`, `price > 0` | Có | FR-07, FR-20 |
| `selectedQuantity` | Ô nhập số lượng (Product Detail) | Số nguyên (Integer) | Bắt buộc là số nguyên dương $\ge 1$; mặc định = 1 | Có | FR-06, FR-07 |
| `item.quantity` (trong giỏ) | Màn hình Giỏ hàng (`CartScreen`) | Số nguyên (Integer) | Cho phép điều chỉnh tăng/giảm bằng nút `+`/`-` hoặc nhập trực tiếp; giá trị $\ge 1$ | Có | FR-07 |
| `itemIndex` / `productId` | Nút "Xóa" trên từng dòng sản phẩm | Số nguyên (Integer / Index) | Xác định chính xác vị trí/ID của sản phẩm cần xóa | Có (khi Xóa) | FR-07 |
| `confirmDeleteAction` | Dialog/Alert xác nhận xóa | Boolean (Xác nhận / Hủy) | Bắt buộc có hộp thoại xác nhận trước khi thực hiện xóa sản phẩm khỏi giỏ | Có | FR-07 |
| `navigationTarget` | Nút "Tiếp tục mua sắm" / "Mua tiếp" | Navigation Action | Chuyển màn hình về trang chủ (`home`) | Có | FR-07 |
| `cartTotal` | Nhãn & Giá trị tổng tiền | Number / Currency String | Tự động tính toán = $\sum (price \times quantity)$; nhãn bắt buộc: **"Tổng cộng"** | Có | FR-07 |

---

## 3. Danh mục Hành vi & Kết quả Đầu ra Cần Kiểm tra (Expected Outputs & Behaviors)

1. **Thêm sản phẩm vào giỏ (Add to Cart):**
   - Thêm sản phẩm mới: Tạo dòng sản phẩm mới trong giỏ với số lượng tương ứng.
   - Thêm sản phẩm đã tồn tại trong giỏ: Tăng số lượng của sản phẩm đó, **không được tạo thêm dòng mới**.
   - Badge số lượng trên thanh điều hướng cập nhật chính xác.
2. **Hiển thị & Bố cục Giỏ hàng (Cart UI & Layout):**
   - Hiển thị đầy đủ: Tên sản phẩm, Đơn giá (định dạng `₫`), Ô/Nút điều chỉnh số lượng, Thành tiền từng dòng ($price \times quantity$), và Nút Xóa.
   - Nhãn hiển thị tổng tiền bắt buộc phải là **"Tổng cộng"** (nghiêm cấm hiển thị *"Tổng tạm tính"*).
3. **Cơ chế điều chỉnh số lượng (Quantity Controls):**
   - Phải có nút tăng (`+`) và giảm (`-`) số lượng trực quan, dễ thao tác trên màn hình cảm ứng di động.
   - Khi giảm số lượng về cận dưới $1$, không cho phép giảm xuống $0$ (hoặc phải hỏi xác nhận xóa).
   - Khi người dùng nhập số lượng mới (ví dụ nhập 2), giá trị số lượng phải được cập nhật đúng bằng 2 (không bị tự động cộng thêm hoặc tính sai).
4. **Quy tắc Xóa sản phẩm khỏi giỏ (Remove from Cart with Confirmation):**
   - Khi bấm nút "Xóa", hệ thống **bắt buộc phải hiển thị hộp thoại xác nhận (Alert / Modal Dialog)** hỏi người dùng: *"Bạn có chắc chắn muốn xóa sản phẩm này khỏi giỏ hàng?"*.
   - Chỉ khi người dùng chọn "Đồng ý" / "Xóa" thì sản phẩm mới bị gỡ bỏ khỏi giỏ hàng.
   - Nếu người dùng bấm "Hủy", sản phẩm vẫn giữ nguyên trong giỏ.
5. **Trạng thái Giỏ hàng trống (Empty Cart State):**
   - Khi giỏ hàng không có sản phẩm nào: Phải hiển thị **hình minh họa** rõ ràng và thông điệp thông báo giỏ hàng trống.
   - Có nút / liên kết **"Tiếp tục mua sắm"** dẫn người dùng quay về trang chủ.
6. **Điều hướng Tiếp tục mua sắm (Continue Shopping Navigation):**
   - Cả khi giỏ hàng trống và khi giỏ hàng đang có sản phẩm, đều phải có nút **"Tiếp tục mua sắm"** (nhãn đúng quy chuẩn, không phải viết tắt hoặc sai nghĩa như *"← Mua tiếp"*).

---

## 4. Các Điểm Mơ hồ & Khe hở Đặc tả Cần Lưu ý (Ambiguities & Spec Gaps)

1. **Cơ chế lưu trữ giỏ hàng (Local Cart vs Server-side Sync):**
   - Phân hệ Mobile lưu trữ giỏ hàng trong Local State (hoặc AsyncStorage) hay đồng bộ trực tiếp với CSDL qua API giỏ hàng người dùng?
2. **Số lượng tối đa trên một sản phẩm (Max Quantity Per Item):**
   - Đặc tả chỉ quy định số lượng tối thiểu là $1$, chưa quy định giới hạn số lượng tối đa hoặc kiểm tra tồn kho (stock inventory) khi người dùng tăng số lượng lên quá lớn.
3. **Hiển thị badge số lượng:**
   - Badge trên thanh điều hướng thể hiện tổng số dòng sản phẩm (distinct items: `cart.length`) hay tổng số lượng đơn vị sản phẩm ($\sum item.quantity$)?
