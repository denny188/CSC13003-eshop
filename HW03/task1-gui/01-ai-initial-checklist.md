# Task 1 (Bước 1.1) — Bảng GUI Checklist Ban Đầu Do AI Sinh (Initial AI-Generated Checklist)

- **Sinh viên thực hiện:** Nguyễn Lê Hữu Điền (`23120233`)
- **Công cụ AI hỗ trợ:** Claude / Gemini ( thông qua Agent Skill `hw03-gui-checklist`)
- **Phạm vi màn hình kiểm thử:**
  - **S1:** Màn hình Đăng ký (`/register` — `FR-01`)
  - **S2:** Màn hình Đăng nhập (`/login` — `FR-02`) & Quên mật khẩu (`/forgot-password` — `FR-03`)
  - **S3:** Màn hình Hồ sơ cá nhân & Lịch sử đơn hàng (`/profile` — `FR-04`, `FR-11`)
  - **Common:** Thanh điều hướng chung (`Header/Navbar` — `FR-23`)
- **Tổng số mục do AI sinh ban đầu:** **40 mục** (`CHK-001` → `CHK-040`), bao phủ đủ 4 khía cạnh `IA-01`, `IA-02`, `IA-03`, `IA-04`.

---

## 1. Nhóm IA-01: Tiêu chuẩn Giao diện Chung (General UI Standards — `FR-21`)

| Mã số | Khía cạnh | Màn hình | Tham chiếu Spec | Nội dung kiểm tra (Checklist Item) | Kết quả kỳ vọng (Expected UI Behavior) |
| :---: | :---: | :---: | :---: | :--- | :--- |
| `CHK-001` | `IA-01` | `/register` (S1) | `FR-21` | Tính nhất quán ngôn ngữ trên trang Đăng ký | Toàn bộ tiêu đề, nhãn trường, thông báo gợi ý và nút bấm trên trang `/register` đều hiển thị bằng tiếng Việt chuẩn. |
| `CHK-002` | `IA-01` | `/login` (S2) | `FR-21` | Tính nhất quán ngôn ngữ trên trang Đăng nhập | Toàn bộ nhãn trường (Email, Mật khẩu) và nút bấm gửi biểu mẫu trên trang `/login` phải dùng tiếng Việt (không trộn lẫn tiếng Anh như `Username` hay `Sign In`). |
| `CHK-003` | `IA-01` | `/register` (S1) | `FR-21` | Màu sắc nút hành động tích cực (Submit) trên trang Đăng ký | Nút chính `"Đăng Ký"` phải sử dụng màu xanh dương (`bg-blue-600`), không được dùng màu đỏ (`bg-red-500` vốn dành cho hành động nguy hiểm/hủy bỏ). |
| `CHK-004` | `IA-01` | `/login` (S2) | `FR-21` | Màu sắc nút hành động tích cực (Submit) trên trang Đăng nhập | Nút gửi biểu mẫu đăng nhập phải sử dụng màu xanh dương (`bg-blue-600`). |
| `CHK-005` | `IA-01` | `/forgot-password` (S2) | `FR-21` | Màu sắc nút hành động tích cực & nút phụ ở Bước 2 Quên mật khẩu | Nút chính `"Đặt lại mật khẩu"` phải dùng màu xanh dương (`bg-blue-600`), và nút phụ `"← Quay lại"` phải có màu sắc phân biệt (trung tính/xám) thay vì dùng cùng màu xanh lá (`bg-green-600`). |
| `CHK-006` | `IA-01` | `/profile` (S3) | `FR-21` | Màu sắc nút hành động tích cực (`Cập nhật`) và nút nguy hiểm (`Hủy đơn`) trên trang Hồ sơ | Nút `"Cập nhật"` hồ sơ dùng màu xanh dương (`bg-blue-600`); nút `"Hủy đơn"` trong bảng lịch sử đơn hàng dùng màu đỏ (`bg-red-500`). |
| `CHK-007` | `IA-01` | `/profile` (S3) | `FR-21`, `FR-11` | Định dạng tiền tệ tại cột Tổng tiền trong bảng Lịch sử đơn hàng | Giá trị tổng tiền của mỗi đơn hàng luôn hiển thị kèm ký hiệu `₫` và có dấu phân cách hàng nghìn rõ ràng (ví dụ: `350.000 ₫` hoặc `350,000 ₫`). |
| `CHK-008` | `IA-01` | `/register`, `/forgot-password`, `/profile` (S1, S2, S3) | `FR-21` | Quy chuẩn thẻ tiêu đề chính `<h1>` trên mỗi trang | Mỗi trang (`/register`, `/forgot-password`, `/profile`) phải có đúng 1 thẻ `<h1>` mô tả nội dung chính của trang (không dùng `<h2>` làm tiêu đề cấp cao nhất của trang). |
| `CHK-009` | `IA-01` | `/login` (S2) | `FR-21` | Nội dung và thẻ ngữ nghĩa của tiêu đề trang Đăng nhập | Trang `/login` phải có đúng 1 thẻ `<h1>` và văn bản tiêu đề phải là `"Đăng Nhập"` (không được hiển thị sai thành `"Đăng Ký"`). |
| `CHK-010` | `IA-01` | `/login` (S2) | `FR-21` | Thứ tự di chuyển tiêu điểm bàn phím (Tab Order) trên trang Đăng nhập | Khi nhấn phím `Tab`, tiêu điểm (focus) phải đi tuần tự từ trên xuống dưới: `Ô Email → Ô Mật khẩu → Link Quên mật khẩu → Nút Đăng nhập → Link Đăng ký ngay` (không bị nhảy cóc vào nút Submit trước tiên do gán sai `tabIndex={1}`). |

---

## 2. Nhóm IA-02: Biểu mẫu Nhập liệu (Forms — `FR-22`, `FR-01`..`04`)

| Mã số | Khía cạnh | Màn hình | Tham chiếu Spec | Nội dung kiểm tra (Checklist Item) | Kết quả kỳ vọng (Expected UI Behavior) |
| :---: | :---: | :---: | :---: | :--- | :--- |
| `CHK-011` | `IA-02` | `/register` (S1) | `FR-22` | Ký hiệu bắt buộc `*` bên cạnh nhãn các trường trên trang Đăng ký | Tất cả các trường bắt buộc (`Họ Tên`, `Email`, `Mật khẩu`, `Xác nhận mật khẩu`) phải hiển thị dấu `*` bên cạnh nhãn `<label>`. |
| `CHK-012` | `IA-02` | `/login` (S2) | `FR-22` | Ký hiệu bắt buộc `*` bên cạnh nhãn các trường trên trang Đăng nhập | Hai trường bắt buộc (`Email` và `Mật khẩu`) trên trang `/login` phải có dấu `*` bên cạnh nhãn. |
| `CHK-013` | `IA-02` | `/forgot-password` (S2) | `FR-22` | Ký hiệu bắt buộc `*` bên cạnh nhãn các trường trên trang Quên mật khẩu | Trường `Nhập Email của bạn` (Bước 1) và `Mã OTP`, `Mật khẩu mới`, `Xác nhận mật khẩu mới` (Bước 2) phải có dấu `*` bên cạnh nhãn. |
| `CHK-014` | `IA-02` | `/profile` (S3) | `FR-22` | Ký hiệu bắt buộc `*` bên cạnh nhãn trường bắt buộc trên trang Hồ sơ | Trường bắt buộc `Họ Tên` (`required`) trong biểu mẫu Hồ sơ cá nhân phải có dấu `*` bên cạnh nhãn. |
| `CHK-015` | `IA-02` | `/register`, `/login`, `/forgot-password` (S1, S2) | `FR-22`, `FR-02` | Thuộc tính HTML5 `type="email"` cho tất cả các ô nhập Email | Ô nhập Email ở cả 3 màn hình (`/register`, `/login`, `/forgot-password`) phải dùng `type="email"` để kích hoạt kiểm tra định dạng email chuẩn của trình duyệt (không dùng `type="text"`). |
| `CHK-016` | `IA-02` | `/login` (S2) | `FR-22` | Thuộc tính `type="password"` che giấu ký tự mật khẩu trên trang Đăng nhập | Ô nhập Mật khẩu trên trang `/login` phải dùng `type="password"` để ẩn ký tự nhập vào (không được để `type="text"` làm lộ mật khẩu dạng rõ). |
| `CHK-017` | `IA-02` | `/register`, `/forgot-password` (S1, S2) | `FR-22` | Thuộc tính `type="password"` trên trang Đăng ký và Quên mật khẩu | Ô nhập Mật khẩu trên `/register` và ô Mật khẩu mới trên `/forgot-password` sử dụng đúng `type="password"`. |
| `CHK-018` | `IA-02` | `/login` (S2) | `FR-22` | Vị trí hiển thị thông báo lỗi trên trang Đăng nhập | Khi đăng nhập thất bại, khung thông báo lỗi phải xuất hiện **phía trên** nút Submit, không được đặt bên dưới nút Submit / dưới đáy form. |
| `CHK-019` | `IA-02` | `/register` (S1) | `FR-22` | Vị trí hiển thị thông báo lỗi trên trang Đăng ký | Khung thông báo lỗi xác thực trên trang `/register` xuất hiện ngay phía trên nút Submit (hoặc phía trên vùng hành động gửi form), không đặt bên dưới nút Submit. |
| `CHK-020` | `IA-02` | `/forgot-password` (S2) | `FR-22`, `FR-03` | Chỉ báo bước (Step Indicator) trên biểu mẫu 2 bước Quên mật khẩu | Biểu mẫu Quên mật khẩu có 2 bước (Bước 1: Nhập Email lấy OTP, Bước 2: Nhập OTP & Đặt lại mật khẩu) bắt buộc phải hiển thị **Step Indicator** rõ ràng (ví dụ: `"Bước 1 / 2"`, `"Bước 2 / 2"`). |
| `CHK-021` | `IA-02` | `/register`, `/forgot-password` (S1, S2) | `FR-01`, `FR-03` | Sự hiện diện của trường "Xác nhận mật khẩu" | Biểu mẫu Đăng ký (`/register`) và Bước 2 Quên mật khẩu (`/forgot-password`) phải có trường nhập **Xác nhận mật khẩu** để đối chiếu khớp mật khẩu. |
| `CHK-022` | `IA-02` | `/profile` (S3) | `FR-04` | Trạng thái khóa chỉnh sửa (disabled/read-only) của trường Email trên trang Hồ sơ | Trường Email trên trang `/profile` phải ở trạng thái `disabled` với nền xám (`bg-gray-100`), không cho phép người dùng sửa đổi qua giao diện. |

---

## 3. Nhóm IA-03: Điều hướng (Navigation — `FR-23`, `FR-03`)

| Mã số | Khía cạnh | Màn hình | Tham chiếu Spec | Nội dung kiểm tra (Checklist Item) | Kết quả kỳ vọng (Expected UI Behavior) |
| :---: | :---: | :---: | :---: | :--- | :--- |
| `CHK-023` | `IA-03` | Navbar chung (S1, S2, S3) | `FR-23` | Đánh dấu nổi bật (Highlight) mục đang chọn trên thanh Navbar | Khi người dùng đang ở `/login`, `/register`, hoặc `/profile`, liên kết tương ứng trên thanh Navbar phải được highlight (đổi màu nền/chữ/gạch chân cố định) để báo hiệu vị trí hiện tại. |
| `CHK-024` | `IA-03` | Navbar chung (S1, S2, S3) | `FR-23` | Hiển thị Badge số lượng sản phẩm cạnh liên kết "Giỏ hàng" | Liên kết `"Giỏ hàng"` trên thanh Navbar phải hiển thị huy hiệu (badge) số lượng sản phẩm hiện có trong giỏ hàng. |
| `CHK-025` | `IA-03` | Navbar khi đã đăng nhập (S3) | `FR-23` | Nhãn chuẩn của nút Đăng xuất trên thanh Navbar | Nút đăng xuất trên thanh Navbar bắt buộc phải có nhãn là `"Đăng xuất"` (không được đặt nhãn là `"Thoát"`). |
| `CHK-026` | `IA-03` | `/register`, `/login`, `/forgot-password`, `/profile` | `FR-23` | Hiển thị thanh điều hướng phân cấp (Breadcrumb) ở các trang con | Các trang con ngoài trang chủ phải có thanh **Breadcrumb** (ví dụ: `Trang chủ > Hồ sơ cá nhân`) để người dùng nhận biết cấp độ trang và quay lại nhanh. |
| `CHK-027` | `IA-03` | `/forgot-password` (S2) | `FR-03` | Nút / Liên kết "Quay lại đăng nhập" tại Bước 1 màn hình Quên mật khẩu | Tại Bước 1 của màn hình `/forgot-password`, giao diện phải có nút hoặc liên kết `"Quay lại đăng nhập"` dẫn về `/login`. |
| `CHK-028` | `IA-03` | `/login` (S2) | `FR-23` | Liên kết chuyển đổi giữa Đăng nhập, Đăng ký và Quên mật khẩu | Từ `/login` có liên kết hoạt động đúng sang `/register` (`"Đăng ký ngay"`) và `/forgot-password` (`"Quên mật khẩu?"`); từ `/register` có liên kết quay về `/login`. |
| `CHK-029` | `IA-03` | `/login` (S2) | `FR-23` | Sử dụng SPA Router Link thay vì thẻ `<a href>` gây tải lại toàn trang | Liên kết `"Quên mật khẩu?"` trên trang `/login` nên dùng `<Link to="/forgot-password">` của React Router thay vì `<a href="/forgot-password">` gây full-page reload. |
| `CHK-030` | `IA-03` | `/register` (S1) | `FR-01` | Tự động chuyển hướng tới trang Đăng nhập sau khi đăng ký thành công | Sau khi gửi biểu mẫu Đăng ký hợp lệ thành công, hệ thống tự động điều hướng người dùng sang trang `/login`. |
| `CHK-031` | `IA-03` | Navbar chung (S1, S2, S3) | `FR-23` | Điều hướng về Trang chủ khi nhấp vào Logo "EShop" | Nhấp vào thương hiệu `"EShop"` ở góc trái thanh Header luôn điều hướng chính xác về trang chủ (`/`). |

---

## 4. Nhóm IA-04: Phản hồi & Trạng thái (Feedback & State — `FR-24`, `FR-10`, `FR-11`)

| Mã số | Khía cạnh | Màn hình | Tham chiếu Spec | Nội dung kiểm tra (Checklist Item) | Kết quả kỳ vọng (Expected UI Behavior) |
| :---: | :---: | :---: | :---: | :--- | :--- |
| `CHK-032` | `IA-04` | `/profile` (S3) | `FR-24` | Minh họa trực quan (Icon/Illustration) cho Trạng thái trống (Empty State) của Lịch sử đơn hàng | Khi người dùng chưa có đơn hàng nào (`orders.length === 0`), khu vực Lịch sử đơn hàng phải hiển thị icon/hình minh họa kèm thông điệp thân thiện và nút gợi ý mua sắm (không chỉ hiện mỗi dòng chữ `<p>Bạn chưa có đơn hàng nào.</p>`). |
| `CHK-033` | `IA-04` | `/profile` (S3) | `FR-24` | Hộp thoại xác nhận (Confirmation Dialog) trước khi Hủy đơn hàng | Khi người dùng nhấn nút `"Hủy đơn"` trong bảng Lịch sử đơn hàng, hệ thống phải hiển thị hộp thoại xác nhận (Confirm Dialog) trước khi thực thi hủy đơn. |
| `CHK-034` | `IA-04` | `/profile` (S3) | `FR-11` | Việt hóa và phân biệt màu sắc cho Huy hiệu Trạng thái đơn hàng (Status Badge) | Trạng thái đơn hàng trong bảng Lịch sử đơn hàng phải được dịch sang tiếng Việt (`Chờ xác nhận`, `Đã xác nhận`, `Đang giao`, `Đã giao`, `Đã hủy`) và có màu nền/chữ phân biệt rõ ràng. |
| `CHK-035` | `IA-04` | `/profile` (S3) | `FR-10`, `FR-11` | Ẩn nút "Hủy đơn" khi đơn hàng đang ở trạng thái "Đang giao" (`shipping`) | Theo State Machine (`FR-10`), khi đơn hàng ở trạng thái `shipping` (`Đang giao`), khách hàng không được tự hủy đơn; giao diện phải ẩn hoặc khóa nút `"Hủy đơn"` đối với trạng thái này. |
| `CHK-036` | `IA-04` | `/profile`, `/forgot-password` (S2, S3) | `FR-22`, `FR-24` | Hiển thị thông báo phản hồi trực tuyến (Inline/Toast) thay vì hộp thoại `window.alert()` | Khi cập nhật hồ sơ, hủy đơn hoặc đặt lại mật khẩu có lỗi/thành công, hệ thống hiển thị thông báo inline trên giao diện hoặc toast thay vì dùng hộp thoại `alert()` của trình duyệt gây chặn luồng thao tác. |
| `CHK-037` | `IA-04` | `/profile` (S3) | `FR-04` | Tính nhất quán giữa gợi ý Placeholder và thông báo lỗi kiểm tra Số điện thoại | Ô Số điện thoại hiển thị `placeholder="VD: 0912345678"` (bắt đầu bằng `0`, 10 số theo đúng `FR-04`) thì bộ kiểm tra hợp lệ phải chấp nhận số điện thoại bắt đầu bằng `0` (không được báo lỗi từ chối số `0` ở đầu). |
| `CHK-038` | `IA-04` | `/register`, `/forgot-password` (S1, S2) | `FR-01`, `FR-03` | Tính chính xác giữa thông báo yêu cầu mật khẩu mạnh trên UI và logic kiểm tra | Thông báo gợi ý yêu cầu mật khẩu có ký tự đặc biệt (`@ $ ! % * ? &`) thì bộ kiểm tra trên UI phải chấp nhận đúng các ký tự đặc biệt đó (không được dùng sai biểu thức chính quy `\s` bắt buộc khoảng trắng). |
| `CHK-039` | `IA-04` | `/forgot-password` (S2) | `FR-03` | Độ dài mã OTP hiển thị và nhãn hướng dẫn tại Bước 2 Quên mật khẩu | Hệ thống phải sinh mã OTP 6 chữ số và nhãn hướng dẫn phải ghi `"Mã OTP (6 số)"` (không được sinh 4 số và ghi nhãn `"Mã OTP (4 số)"`). |
| `CHK-040` | `IA-04` | `/profile` (S3) | `FR-24` | Hiển thị trạng thái yêu cầu đăng nhập khi truy cập `/profile` lúc chưa xác thực | Khi chưa đăng nhập mà truy cập `/profile`, hệ thống hiển thị thông báo rõ ràng hoặc điều hướng về trang đăng nhập kèm liên kết thao tác. |
