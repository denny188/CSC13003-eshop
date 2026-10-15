# Báo cáo Kiểm thử Miền giá trị (Domain Testing): FR-03 (Quên mật khẩu & Đặt lại mật khẩu)

## 1. Phương pháp Áp dụng
Áp dụng kỹ thuật **Phân hoạch tương đương (Equivalence Partitioning)** theo quy chuẩn:
1. Xác định các biến đầu vào và phân chia miền giá trị thành các **Lớp tương đương hợp lệ (Valid Classes)** và **Lớp tương đương không hợp lệ (Invalid Classes)**.
2. Chọn đúng **1 giá trị đại diện (Representative Value)** cho mỗi lớp kèm theo lý giải (Rationale).
3. Thiết kế ca kiểm thử theo nguyên tắc **Single-fault (chỉ chọn 1 lớp không hợp lệ tại một thời điểm)** để đảm bảo lỗi phát sinh được quy gán chính xác về biến tương ứng.
4. Xây dựng các ca kiểm thử tích hợp cho các biến có tương tác phụ thuộc lẫn nhau (ví dụ: `newPassword` vs `confirmNewPassword`, `resetToken` vs `email`).

---

## 2. Bảng Phân hoạch Tương đương & Chọn Giá trị Đại diện (Domain Table)

### 2.1 Biến `email` (Bước 1: Yêu cầu mã OTP)
| Mã lớp | Mô tả lớp tương đương | Hợp lệ / Không hợp lệ | Giá trị đại diện | Lý giải chọn giá trị (Rationale) |
| :--- | :--- | :---: | :--- | :--- |
| **EC-EM-V1** | Email đúng định dạng và đã đăng ký trong hệ thống | Hợp lệ | `test@eshop.com` | Email tài khoản user chuẩn đã tạo sẵn trong CSDL SUT |
| **EC-EM-I1** | Email để trống (Empty string) | Không hợp lệ | `""` (chuỗi rỗng) | Kiểm tra bắt buộc nhập (mandatory / HTML5 validation) |
| **EC-EM-I2** | Chỉ chứa ký tự khoảng trắng (Whitespace-only) | Không hợp lệ | `"   "` | Kiểm tra hàm trim/strip khoảng trắng |
| **EC-EM-I3** | Sai định dạng: Thiếu ký tự `@` | Không hợp lệ | `testeshop.com` | Kiểm tra cú pháp chuẩn RFC của trường email |
| **EC-EM-I4** | Sai định dạng: Thiếu phần domain sau `@` | Không hợp lệ | `test@` | Kiểm tra cú pháp tên miền email |
| **EC-EM-I5** | Sai định dạng: Chứa ký tự đặc biệt ngoài chuẩn | Không hợp lệ | `test<script>@eshop.com` | Kiểm tra chống XSS và tính an toàn của chuỗi |
| **EC-EM-I6** | Đúng định dạng nhưng chưa từng đăng ký | Không hợp lệ | `notfound_2026@eshop.com` | Kiểm tra xử lý tài khoản không tồn tại trong hệ thống |

---

### 2.2 Biến `resetToken` (Bước 2: Mã xác thực OTP)
*Ràng buộc từ spec FR-03 & SEC-07: Chuỗi gồm đúng 6 chữ số ngẫu nhiên, gắn với email yêu cầu, chưa hết hạn, dùng 1 lần.*

| Mã lớp | Mô tả lớp tương đương | Hợp lệ / Không hợp lệ | Giá trị đại diện | Lý giải chọn giá trị (Rationale) |
| :--- | :--- | :---: | :--- | :--- |
| **EC-TK-V1** | Đúng 6 chữ số, đúng token đã cấp cho email đó, chưa dùng, còn hạn | Hợp lệ | `Token cấp phát thực tế` (vd: `482915`) | Token sinh ra từ Bước 1 cho email yêu cầu |
| **EC-TK-I1** | Để trống | Không hợp lệ | `""` | Kiểm tra trường bắt buộc nhập |
| **EC-TK-I2** | Độ dài không đủ 6 chữ số (< 6 ký tự) | Không hợp lệ | `"1234"` | Kiểm tra chặn OTP ngắn hơn quy định |
| **EC-TK-I3** | Độ dài vượt quá 6 chữ số (> 6 ký tự) | Không hợp lệ | `"1234567"` | Kiểm tra chặn OTP dài hơn quy định |
| **EC-TK-I4** | Chứa ký tự chữ hoặc ký tự đặc biệt (không phải số) | Không hợp lệ | `"12AB56"` | Kiểm tra ràng buộc chỉ nhận ký số (digits only) |
| **EC-TK-I5** | 6 chữ số nhưng sai mã (không khớp mã hệ thống sinh) | Không hợp lệ | `"999999"` | Kiểm tra xác thực token sai giá trị |
| **EC-TK-I6** | Đúng mã nhưng là của tài khoản email khác (cross-account) | Không hợp lệ | `Token của admin@eshop.com` | Kiểm tra ràng buộc SEC-07: OTP chỉ hợp lệ cho email yêu cầu |
| **EC-TK-I7** | Mã OTP đã từng được sử dụng thành công trước đó (Replay attack) | Không hợp lệ | `Token vừa reset xong` | Kiểm tra vô hiệu hóa mã sau khi dùng (SEC-07) |

---

### 2.3 Biến `newPassword` (Bước 2: Mật khẩu mới)
*Ràng buộc từ spec FR-01: Độ dài $\ge 8$, có ít nhất 1 chữ hoa, 1 chữ thường, 1 chữ số, 1 ký tự đặc biệt trong `[@$!%*?&]`.*

| Mã lớp | Mô tả lớp tương đương | Hợp lệ / Không hợp lệ | Giá trị đại diện | Lý giải chọn giá trị (Rationale) |
| :--- | :--- | :---: | :--- | :--- |
| **EC-PW-V1** | Thỏa mãn đầy đủ 5 tiêu chí độ mạnh mật khẩu | Hợp lệ | `P@ssword123` | Độ dài 11, có đủ hoa/thường/số/đặc biệt hợp lệ `@` |
| **EC-PW-I1** | Để trống | Không hợp lệ | `""` | Kiểm tra trường bắt buộc nhập |
| **EC-PW-I2** | Độ dài ngắn hơn 8 ký tự (< 8 ký tự) | Không hợp lệ | `Ab1!xyz` | 7 ký tự (thiếu độ dài dù đủ loại ký tự) |
| **EC-PW-I3** | Thiếu chữ hoa (không có `[A-Z]`) | Không hợp lệ | `password123!` | Đủ độ dài, có thường, số, đặc biệt nhưng thiếu hoa |
| **EC-PW-I4** | Thiếu chữ thường (không có `[a-z]`) | Không hợp lệ | `PASSWORD123!` | Đủ độ dài, có hoa, số, đặc biệt nhưng thiếu thường |
| **EC-PW-I5** | Thiếu chữ số (không có `[0-9]`) | Không hợp lệ | `Password!!` | Đủ độ dài, có hoa, thường, đặc biệt nhưng thiếu số |
| **EC-PW-I6** | Thiếu ký tự đặc biệt (không có `[@$!%*?&]`) | Không hợp lệ | `Password123` | Đủ độ dài, có hoa, thường, số nhưng không có đặc biệt |
| **EC-PW-I7** | Có ký tự đặc biệt nhưng nằm ngoài tập hợp cho phép | Không hợp lệ | `Password123#` | Chứa `#` (không nằm trong `[@$!%*?&]`) |

---

### 2.4 Biến `confirmNewPassword` (Bước 2: Xác nhận mật khẩu mới)
| Mã lớp | Mô tả lớp tương đương | Hợp lệ / Không hợp lệ | Giá trị đại diện | Lý giải chọn giá trị (Rationale) |
| :--- | :--- | :---: | :--- | :--- |
| **EC-CF-V1** | Trùng khớp 100% với `newPassword` | Hợp lệ | Giống `newPassword` | Đảm bảo người dùng xác nhận đúng |
| **EC-CF-I1** | Không trùng khớp với `newPassword` | Không hợp lệ | Khác `newPassword` | Kiểm tra cơ chế từ chối khi xác nhận sai |
| **EC-CF-I2** | Để trống | Không hợp lệ | `""` | Bắt buộc phải xác nhận lại mật khẩu |

---

## 3. Danh sách Ca Kiểm thử Chi tiết (Test Cases)

| Mã Test Case | Lớp tương đương bao phủ | Điều kiện tiên quyết (Preconditions) | Các bước thực hiện (Steps) | Dữ liệu kiểm thử (Test Data) | Kết quả mong đợi (Expected Result) | Độ ưu tiên | Trạng thái |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: | :---: |
| **TC-FR03-DT-001** | **EC-EM-V1** (Valid All Bước 1) | Tài khoản `test@eshop.com` đã tồn tại | 1. Truy cập trang Quên mật khẩu<br>2. Nhập email<br>3. Bấm "Lấy mã OTP" | `email`: `test@eshop.com` | Hệ thống sinh OTP 6 chữ số, chuyển sang Bước 2 (hoặc hiển thị OTP), có Step Indicator "Bước 1/2", nút Quay lại *(FR-03/Bước 1)* | High | Not executed |
| **TC-FR03-DT-002** | **EC-EM-I1** | Mở form Bước 1 | 1. Để trống email<br>2. Bấm "Lấy mã OTP" | `email`: `""` | Báo lỗi bắt buộc nhập, không gửi request *(FR-22)* | High | Not executed |
| **TC-FR03-DT-003** | **EC-EM-I2** | Mở form Bước 1 | 1. Nhập khoảng trắng<br>2. Bấm "Lấy mã OTP" | `email`: `"   "` | Báo lỗi email không hợp lệ *(FR-03/Bước 1)* | Medium | Not executed |
| **TC-FR03-DT-004** | **EC-EM-I3** | Mở form Bước 1 | 1. Nhập email thiếu `@`<br>2. Bấm "Lấy mã OTP" | `email`: `testeshop.com` | Client báo lỗi định dạng HTML5 email *(FR-22)* | High | Not executed |
| **TC-FR03-DT-005** | **EC-EM-I4** | Mở form Bước 1 | 1. Nhập email thiếu domain<br>2. Bấm "Lấy mã OTP" | `email`: `test@` | Client báo lỗi định dạng email *(FR-22)* | Medium | Not executed |
| **TC-FR03-DT-006** | **EC-EM-I5** | Mở form Bước 1 | 1. Nhập ký tự độc hại<br>2. Bấm "Lấy mã OTP" | `email`: `test<script>@eshop.com` | Chặn định dạng, không kích hoạt script *(SEC-04)* | High | Not executed |
| **TC-FR03-DT-007** | **EC-EM-I6** | Mở form Bước 1 | 1. Nhập email không có trong DB<br>2. Bấm "Lấy mã OTP" | `email`: `notfound_2026@eshop.com` | Báo lỗi người dùng không tồn tại / từ chối *(FR-03/Bước 1)* | High | Not executed |
| **TC-FR03-DT-008** | **EC-TK-V1, EC-PW-V1, EC-CF-V1** (Valid All Bước 2) | Đã lấy mã OTP hợp lệ từ Bước 1 | 1. Nhập OTP hợp lệ<br>2. Nhập Mật khẩu mới mạnh<br>3. Nhập Xác nhận khớp<br>4. Bấm "Đặt lại mật khẩu" | `resetToken`: `<OTP sinh từ Bước 1>`, `newPassword`: `P@ssword123`, `confirmNewPassword`: `P@ssword123` | Đổi mật khẩu thành công, vô hiệu hóa OTP, chuyển hướng về trang Đăng nhập *(FR-03/Bước 2, SEC-07)* | High | Not executed |
| **TC-FR03-DT-009** | **EC-TK-I1** | Đang ở Bước 2 | 1. Để trống OTP<br>2. Nhập mật khẩu hợp lệ<br>3. Bấm Submit | `resetToken`: `""` | Báo lỗi bắt buộc nhập OTP *(FR-22)* | High | Not executed |
| **TC-FR03-DT-010** | **EC-TK-I2** | Đang ở Bước 2 | 1. Nhập OTP ngắn 4 số<br>2. Nhập mật khẩu hợp lệ<br>3. Bấm Submit | `resetToken`: `"1234"` | Báo lỗi OTP phải đủ 6 chữ số *(FR-03/Bước 1, SEC-07)* | High | Not executed |
| **TC-FR03-DT-011** | **EC-TK-I3** | Đang ở Bước 2 | 1. Nhập OTP dài 7 số<br>2. Bấm Submit | `resetToken`: `"1234567"` | Báo lỗi OTP không hợp lệ *(FR-03/Bước 1, SEC-07)* | Medium | Not executed |
| **TC-FR03-DT-012** | **EC-TK-I4** | Đang ở Bước 2 | 1. Nhập OTP chứa chữ<br>2. Bấm Submit | `resetToken`: `"12AB56"` | Báo lỗi OTP chỉ gồm chữ số *(FR-03/Bước 1)* | Medium | Not executed |
| **TC-FR03-DT-013** | **EC-TK-I5** | Đang ở Bước 2 | 1. Nhập OTP sai mã<br>2. Bấm Submit | `resetToken`: `"999999"` | Báo lỗi mã OTP không đúng *(FR-03/Bước 2)* | High | Not executed |
| **TC-FR03-DT-014** | **EC-TK-I6** | Đang ở Bước 2 (email A) | 1. Dùng OTP của email B nhập cho email A<br>2. Bấm Submit | `resetToken`: `<OTP của admin@eshop.com>` | Từ chối, báo lỗi OTP không thuộc email này *(FR-03/Bước 2, SEC-07)* | High | Not executed |
| **TC-FR03-DT-015** | **EC-TK-I7** | Đã reset thành công ở TC-FR03-DT-008 | 1. Dùng lại mã OTP vừa thực hiện thành công<br>2. Bấm Submit | `resetToken`: `<OTP đã dùng>` | Từ chối, báo lỗi token hết hiệu lực/đã dùng *(SEC-07)* | High | Not executed |
| **TC-FR03-DT-016** | **EC-PW-I1** | Đang ở Bước 2 | 1. Nhập OTP đúng<br>2. Để trống mật khẩu mới<br>3. Bấm Submit | `newPassword`: `""` | Báo lỗi bắt buộc nhập mật khẩu *(FR-22)* | High | Not executed |
| **TC-FR03-DT-017** | **EC-PW-I2** | Đang ở Bước 2 | 1. Nhập mật khẩu 7 ký tự<br>2. Xác nhận khớp<br>3. Bấm Submit | `newPassword`: `Ab1!xyz` | Báo lỗi mật khẩu tối thiểu 8 ký tự *(FR-01, FR-03)* | High | Not executed |
| **TC-FR03-DT-018** | **EC-PW-I3** | Đang ở Bước 2 | 1. Nhập mật khẩu thiếu chữ hoa<br>2. Xác nhận khớp<br>3. Bấm Submit | `newPassword`: `password123!` | Báo lỗi thiếu ít nhất 1 chữ hoa *(FR-01, FR-03)* | High | Not executed |
| **TC-FR03-DT-019** | **EC-PW-I4** | Đang ở Bước 2 | 1. Nhập mật khẩu thiếu chữ thường<br>2. Xác nhận khớp<br>3. Bấm Submit | `newPassword`: `PASSWORD123!` | Báo lỗi thiếu ít nhất 1 chữ thường *(FR-01, FR-03)* | High | Not executed |
| **TC-FR03-DT-020** | **EC-PW-I5** | Đang ở Bước 2 | 1. Nhập mật khẩu thiếu số<br>2. Xác nhận khớp<br>3. Bấm Submit | `newPassword`: `Password!!` | Báo lỗi thiếu ít nhất 1 chữ số *(FR-01, FR-03)* | High | Not executed |
| **TC-FR03-DT-021** | **EC-PW-I6** | Đang ở Bước 2 | 1. Nhập mật khẩu thiếu ký tự đặc biệt<br>2. Xác nhận khớp<br>3. Bấm Submit | `newPassword`: `Password123` | Báo lỗi thiếu ký tự đặc biệt trong tập `[@$!%*?&]` *(FR-01, FR-03)* | High | Not executed |
| **TC-FR03-DT-022** | **EC-PW-I7** | Đang ở Bước 2 | 1. Nhập mật khẩu có ký tự đặc biệt ngoài tập (`#`)<br>2. Bấm Submit | `newPassword`: `Password123#` | Báo lỗi ký tự đặc biệt phải thuộc `[@$!%*?&]` *(FR-01)* | High | Not executed |
| **TC-FR03-DT-023** | **EC-CF-I1** | Đang ở Bước 2 | 1. Nhập mật khẩu mới<br>2. Nhập xác nhận không khớp<br>3. Bấm Submit | `newPassword`: `P@ssword123`, `confirmNewPassword`: `P@ssword456` | Báo lỗi hai mật khẩu không khớp nhau *(FR-03/Bước 2)* | High | Not executed |
| **TC-FR03-DT-024** | **EC-CF-I2** | Đang ở Bước 2 | 1. Nhập mật khẩu mới<br>2. Để trống xác nhận<br>3. Bấm Submit | `confirmNewPassword`: `""` | Báo lỗi bắt buộc xác nhận mật khẩu *(FR-03/Bước 2)* | Medium | Not executed |

---

## 4. Ma trận Độ phủ Lớp tương đương (Coverage Matrix)

| Lớp tương đương | Mã Test Case bao phủ |
| :--- | :--- |
| **EC-EM-V1** | TC-FR03-DT-001 |
| **EC-EM-I1** | TC-FR03-DT-002 |
| **EC-EM-I2** | TC-FR03-DT-003 |
| **EC-EM-I3** | TC-FR03-DT-004 |
| **EC-EM-I4** | TC-FR03-DT-005 |
| **EC-EM-I5** | TC-FR03-DT-006 |
| **EC-EM-I6** | TC-FR03-DT-007 |
| **EC-TK-V1** | TC-FR03-DT-008 |
| **EC-TK-I1** | TC-FR03-DT-009 |
| **EC-TK-I2** | TC-FR03-DT-010 |
| **EC-TK-I3** | TC-FR03-DT-011 |
| **EC-TK-I4** | TC-FR03-DT-012 |
| **EC-TK-I5** | TC-FR03-DT-013 |
| **EC-TK-I6** | TC-FR03-DT-014 |
| **EC-TK-I7** | TC-FR03-DT-015 |
| **EC-PW-V1** | TC-FR03-DT-008 |
| **EC-PW-I1** | TC-FR03-DT-016 |
| **EC-PW-I2** | TC-FR03-DT-017 |
| **EC-PW-I3** | TC-FR03-DT-018 |
| **EC-PW-I4** | TC-FR03-DT-019 |
| **EC-PW-I5** | TC-FR03-DT-020 |
| **EC-PW-I6** | TC-FR03-DT-021 |
| **EC-PW-I7** | TC-FR03-DT-022 |
| **EC-CF-V1** | TC-FR03-DT-008 |
| **EC-CF-I1** | TC-FR03-DT-023 |
| **EC-CF-I2** | TC-FR03-DT-024 |

**Tổng số test case thiết kế trong Step 2:** **24 test cases** (100% các lớp tương đương hợp lệ và không hợp lệ đều được bao phủ).
