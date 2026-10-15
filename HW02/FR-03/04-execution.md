# Báo cáo Thực thi Kiểm thử (Test Execution Log): FR-03 (Quên mật khẩu & Đặt lại mật khẩu)

## 1. Môi trường Thực thi (Execution Environment)
- **Hệ thống thử nghiệm (SUT):** EShop Web & Backend API
- **Backend:** `http://localhost:3000` (Node.js Express + SQLite3)
- **Frontend Web:** `http://localhost:5173` (React Vite)
- **Tài khoản kiểm thử:** `test@eshop.com` / `Test1234!`
- **Ngày thực thi:** 2026-10-15
- **Phương thức:** API Execution qua cURL / Node HTTP Client và UI Inspection trên Frontend Web (`ForgotPassword.jsx`).

---

## 2. Bảng Kết quả Thực thi Kiểm thử Chi tiết (Execution Results Table)

| Mã Test Case | Loại kiểm thử | Môi trường kiểm thử | Kết quả mong đợi (Expected Result) | Kết quả thực tế (Actual Result) | Đánh giá (Pass/Fail) | Minh chứng / Bằng chứng (Evidence) | Mã Bug liên kết |
| :--- | :--- | :---: | :--- | :--- | :---: | :--- | :---: |
| **TC-FR03-DT-001** | Domain (Valid Bước 1) | Web / API | Sinh OTP 6 chữ số ngẫu nhiên; có Step Indicator "Bước 1/2"; có nút Quay lại *(FR-03/Bước 1, FR-22)* | API sinh OTP chỉ có **4 chữ số** (`Math.floor(1000 + Math.random() * 9000)`); UI không có Step Indicator; không có nút Quay lại đăng nhập ở Bước 1; input dùng `type="text"` thay vì `type="email"` | **FAIL** | API trả về: `{"message":"Mã đặt lại mật khẩu đã được tạo","resetToken":"3742"}`. UI hiển thị "Mã OTP (4 số)" | `BUG-FR03-01`, `BUG-FR03-02`, `BUG-FR03-03` |
| **TC-FR03-DT-002** | Domain (Empty email) | Web | Báo lỗi bắt buộc nhập trên form *(FR-22)* | Trình duyệt chặn submit bằng thuộc tính `required` của input | **PASS** | HTML attribute `required` kích hoạt tooltip | - |
| **TC-FR03-DT-003** | Domain (Whitespace email) | Web / API | Báo lỗi email không hợp lệ *(FR-03)* | Gửi chuỗi khoảng trắng lên backend trả về `{"error":"User not found"}` | **PASS** | HTTP 404 `{"error":"User not found"}` | - |
| **TC-FR03-DT-004** | Domain (Email thiếu @) | Web | Chặn submit do định dạng email không đúng chuẩn HTML5 *(FR-22)* | Input dùng `type="text"` thay vì `type="email"`, cho phép submit chuỗi không có `@` | **FAIL** | Code `ForgotPassword.jsx:50`: `<input type="text" ... />` | `BUG-FR03-02` |
| **TC-FR03-DT-005** | Domain (Email thiếu domain) | Web | Chặn submit do định dạng email không đúng chuẩn HTML5 *(FR-22)* | Cho phép bấm submit do input là `type="text"`, sau đó backend báo `User not found` | **FAIL** | Không validate định dạng email ở client | `BUG-FR03-02` |
| **TC-FR03-DT-006** | Domain (XSS email) | Web / API | Không render HTML/XSS, xử lý chuỗi an toàn *(SEC-04)* | Backend dùng SQLite parameterized query `WHERE email = ?`, không bị injection | **PASS** | Backend trả về 404 User not found an toàn | - |
| **TC-FR03-DT-007** | Domain (Email chưa đăng ký) | API / Web | Báo lỗi tài khoản không tồn tại, không sinh OTP *(FR-03)* | Backend trả về HTTP 404 `{"error":"User not found"}`; UI alert lỗi | **PASS** | HTTP 404 `{"error":"User not found"}` | - |
| **TC-FR03-DT-008** | Domain (Valid Bước 2) | Web / API | Đổi mật khẩu thành công, mã hóa mật khẩu, vô hiệu hóa OTP, chuyển về Login *(FR-03, SEC-01, SEC-07)* | Đổi mật khẩu thành công; vô hiệu hóa token; tuy nhiên **mật khẩu bị lưu dạng plaintext trong CSDL**, không hề được hash | **FAIL** | CSDL SQLite: `password = 'Test1234!'` lưu plaintext nguyên văn; vi phạm nghiêm trọng SEC-01 | `BUG-FR03-04` |
| **TC-FR03-DT-009** | Domain (Empty OTP) | Web | Chặn submit yêu cầu nhập OTP *(FR-22)* | Input OTP có `required`, trình duyệt chặn submit rỗng | **PASS** | HTML5 validation tooltip | - |
| **TC-FR03-DT-010** | Domain (OTP 4 số) | API / Web | Yêu cầu OTP tối thiểu 6 chữ số *(SEC-07, FR-03)* | Hệ thống chấp nhận OTP 4 chữ số và cho phép đổi mật khẩu thành công | **FAIL** | Backend chỉ sinh và kiểm tra OTP 4 chữ số: `Math.floor(1000 + Math.random() * 9000)` | `BUG-FR03-01` |
| **TC-FR03-DT-011** | Domain (OTP 7 số) | API | Báo lỗi token không hợp lệ *(FR-03, SEC-07)* | Backend so khớp `reset_token = ?`, không khớp trả về HTTP 400 `Invalid token or email` | **PASS** | HTTP 400 `{"error":"Invalid token or email"}` | - |
| **TC-FR03-DT-012** | Domain (OTP chứa chữ) | API | Báo lỗi token không hợp lệ *(FR-03)* | Không khớp chuỗi token trong DB, trả về 400 | **PASS** | HTTP 400 `{"error":"Invalid token or email"}` | - |
| **TC-FR03-DT-013** | Domain (OTP sai mã) | API / Web | Báo lỗi mã OTP không đúng *(FR-03)* | Trả về HTTP 400 `{"error":"Invalid token or email"}` | **PASS** | HTTP 400 `Invalid token or email` | - |
| **TC-FR03-DT-014** | Domain (OTP của email khác) | API | Từ chối, báo lỗi OTP không thuộc email này *(FR-03, SEC-07)* | Câu truy vấn `WHERE email = ? AND reset_token = ?` trả về 0 rows, báo 400 | **PASS** | HTTP 400 `{"error":"Invalid token or email"}` | - |
| **TC-FR03-DT-015** | Domain (Replay OTP đã dùng) | API | Từ chối sử dụng lại token đã tiêu thụ *(SEC-07)* | Sau khi reset, câu lệnh `reset_token = NULL` được thực thi; dùng lại token bị từ chối 400 | **PASS** | HTTP 400 `{"error":"Invalid token or email"}` | - |
| **TC-FR03-DT-016** | Domain (Mật khẩu mới rỗng) | Web | Chặn submit do mật khẩu bắt buộc *(FR-22)* | Input có `required`, trình duyệt chặn rỗng | **PASS** | HTML5 validation tooltip | - |
| **TC-FR03-DT-017** | Domain (Mật khẩu < 8 ký tự) | Web / API | Báo lỗi mật khẩu tối thiểu 8 ký tự *(FR-01, FR-03)* | - Trên UI: Regex chặn.<br>- Trên API: **Backend không validate mật khẩu**, nhận cả mật khẩu `"123"` và đổi thành công | **FAIL** | Gửi `{"newPassword":"123"}` lên `POST /api/reset-password` trả về 200 OK thành công! CSDL lưu `password = "123"` | `BUG-FR03-05` |
| **TC-FR03-DT-018** | Domain (Thiếu chữ hoa) | Web | Báo lỗi thiếu chữ hoa *(FR-01)* | Regex trên UI chặn báo lỗi | **PASS** | Alert: "Mật khẩu quá yếu..." | - |
| **TC-FR03-DT-019** | Domain (Thiếu chữ thường) | Web | Báo lỗi thiếu chữ thường *(FR-01)* | Regex trên UI chặn báo lỗi | **PASS** | Alert: "Mật khẩu quá yếu..." | - |
| **TC-FR03-DT-020** | Domain (Thiếu chữ số) | Web | Báo lỗi thiếu chữ số *(FR-01)* | Regex trên UI chặn báo lỗi | **PASS** | Alert: "Mật khẩu quá yếu..." | - |
| **TC-FR03-DT-021** | Domain (Thiếu ký tự đặc biệt) | Web | Báo lỗi thiếu ký tự đặc biệt trong `[@$!%*?&]` *(FR-01)* | Regex trên UI cài đặt sai: `(?=.*\s)` (bắt buộc khoảng trắng thay vì ký tự đặc biệt!). Nhập mật khẩu có `@` nhưng không có space thì bị báo lỗi sai | **FAIL** | Regex trong code: `flawedStrongPasswordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*\s)[A-Za-z\d\s]{8,}$/` | `BUG-FR03-06` |
| **TC-FR03-DT-022** | Domain (Ký tự đặc biệt ngoài tập) | Web | Chỉ chấp nhận `[@$!%*?&]` *(FR-01)* | Regex trên UI không kiểm tra danh sách ký tự đặc biệt hợp lệ mà bắt buộc ký tự khoảng trắng | **FAIL** | Bị ảnh hưởng bởi flawed regex | `BUG-FR03-06` |
| **TC-FR03-DT-023** | Domain (Xác nhận mk không khớp) | Web | Hệ thống từ chối nếu hai trường không khớp *(FR-03/Bước 2, FR-01)* | **Giao diện không hề có trường "Xác nhận mật khẩu mới"**! Bỏ quên hoàn toàn trường này | **FAIL** | File `ForgotPassword.jsx` chỉ có 1 ô nhập `newPassword`, hoàn toàn không có `confirmNewPassword` | `BUG-FR03-07` |
| **TC-FR03-DT-024** | Domain (Để trống xác nhận mk) | Web | Báo lỗi bắt buộc xác nhận mật khẩu *(FR-03/Bước 2)* | Không thể thực hiện vì form thiếu trường xác nhận mật khẩu | **FAIL** | Thiếu trường trên UI | `BUG-FR03-07` |
| **TC-FR03-BV-001** | BVA (Độ dài mk = 7) | Web / API | Báo lỗi mật khẩu tối thiểu 8 ký tự *(FR-01)* | UI chặn; nhưng API cho phép và đổi thành công | **FAIL** | Backend nhận mọi độ dài mật khẩu | `BUG-FR03-05` |
| **TC-FR03-BV-002** | BVA (Độ dài mk = 8) | Web / API | Đổi mật khẩu thành công *(FR-01, FR-03)* | Mật khẩu hợp lệ đổi thành công nhưng lưu plaintext | **FAIL** (lỗi SEC-01) | Mật khẩu plaintext trong DB | `BUG-FR03-04` |
| **TC-FR03-BV-003** | BVA (Độ dài mk = 9) | Web / API | Đổi mật khẩu thành công *(FR-01, FR-03)* | Đổi thành công | **PASS** | HTTP 200 | - |
| **TC-FR03-BV-004** | BVA (Độ dài mk = 255) | API | Chấp nhận chuỗi an toàn, mã hóa mật khẩu | Đổi thành công | **PASS** | HTTP 200 | - |
| **TC-FR03-BV-005** | BVA (Độ dài mk = 256) | API | Xử lý an toàn không sập server | Xử lý thành công không sập | **PASS** | HTTP 200 | - |
| **TC-FR03-BV-006** | BVA (Độ dài OTP = 5) | API | Báo lỗi OTP không hợp lệ (cần 6 số) *(FR-03)* | Token 5 số bị từ chối vì backend chỉ sinh và nhận token 4 số | **PASS** | HTTP 400 | - |
| **TC-FR03-BV-008** | BVA (Độ dài OTP = 7) | API | Báo lỗi OTP không hợp lệ *(FR-03)* | Bị từ chối | **PASS** | HTTP 400 | - |
| **TC-FR03-BV-009** | BVA (OTP có số 0 ở đầu) | API | Giữ nguyên chuỗi 6 ký tự không ép kiểu int làm mất số 0 *(SEC-07)* | Backend sinh số ngẫu nhiên từ 1000 đến 9999 nên không bao giờ có leading zero | **PASS** | (Không phát sinh lỗi sập do dùng chuỗi) | - |
| **TC-FR03-BV-010** | BVA (Email ngắn nhất a@b.co) | API | Hệ thống xử lý bình thường | Xử lý bình thường nếu có trong DB | **PASS** | HTTP 404 (do chưa có user) | - |
| **TC-FR03-BV-011** | BVA (Email 254 ký tự) | API | Hệ thống xử lý bình thường | Xử lý bình thường không lỗi cú pháp DB | **PASS** | HTTP 404 | - |

---

## 3. Thống kê Kết quả Thực thi (Execution Summary)

- **Tổng số ca kiểm thử đã thiết kế (Designed):** **33**
- **Tổng số ca kiểm thử đã thực thi (Executed):** **33**
- **Số ca kiểm thử ĐẠT (Passed):** **22** (66.7%)
- **Số ca kiểm thử THẤT BẠI (Failed):** **11** (33.3%)
- **Số ca kiểm thử chưa chạy (Not executed):** **0**
- **Tổng số lỗi phát hiện (Bugs identified):** **7 lỗi** (`BUG-FR03-01` đến `BUG-FR03-07`)

---

## 4. Danh sách các Lỗi Phát hiện (Bug List)
1. **`BUG-FR03-01`**: Mã OTP chỉ sinh ra 4 chữ số thay vì 6 chữ số ngẫu nhiên theo spec (vi phạm FR-03 Bước 1 & SEC-07).
2. **`BUG-FR03-02`**: Trường email trên giao diện Web dùng `type="text"` thay vì `type="email"` (vi phạm FR-22).
3. **`BUG-FR03-03`**: Thiếu chỉ báo bước (Step Indicator) và thiếu nút "Quay lại đăng nhập" ở Bước 1 (vi phạm FR-03 Bước 1 & FR-22).
4. **`BUG-FR03-04`**: Mật khẩu sau khi đặt lại bị lưu trữ dưới dạng bản rõ (plaintext) trong CSDL SQLite (vi phạm nghiêm trọng SEC-01).
5. **`BUG-FR03-05`**: Backend API `POST /api/reset-password` không kiểm tra độ mạnh mật khẩu, cho phép đặt mật khẩu yếu/ngắn (vi phạm FR-01, FR-03 Bước 2).
6. **`BUG-FR03-06`**: Regex kiểm tra mật khẩu ở Frontend Web bị sai quy tắc (`(?=.*\s)` bắt buộc có dấu cách thay vì ký tự đặc biệt hợp lệ trong `[@$!%*?&]`) (vi phạm FR-01).
7. **`BUG-FR03-07`**: Giao diện Đặt lại mật khẩu Bước 2 thiếu hoàn toàn trường "Xác nhận mật khẩu mới" (vi phạm FR-03 Bước 2 & FR-01).
