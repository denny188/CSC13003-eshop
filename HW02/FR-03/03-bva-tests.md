# Báo cáo Phân tích Giá trị Biên (Boundary Value Analysis - BVA): FR-03

## 1. Phương pháp & Nguyên tắc Áp dụng
1. Áp dụng kỹ thuật **Boundary Value Analysis (BVA)** cho tất cả các biến đầu vào có miền giá trị có thứ tự (ordered domain) hoặc ràng buộc về độ dài/số lượng ký tự (length boundaries).
2. Tập giá trị biên tiêu chuẩn được khảo sát bao gồm:
   $$\{min - 1, \; min, \; min + 1, \; nominal, \; max - 1, \; max, \; max + 1\}$$
   - Đối với các cận không có giới hạn trên tường minh trong spec (open-ended boundaries như độ dài mật khẩu), cận $min$ được xác định chính xác theo spec ($min = 8$), và khảo sát các giá trị lân cận ($7, 8, 9$).
   - Đối với biến yêu cầu độ dài chính xác $L$ (fixed length như OTP đúng 6 chữ số: $min = max = 6$), tập biên là:
     $$\{L - 1 = 5, \; L = 6, \; L + 1 = 7\}$$
3. Xử lý trùng lặp: Các trường hợp biên đã được kiểm tra đầy đủ ở Step 2 (Domain Testing) được ghi rõ và tham chiếu chéo (cross-reference), tránh trùng lặp dư thừa nhưng vẫn đảm bảo độ phủ 100% các điểm biên.

---

## 2. Phân tích Các Điểm Biên theo Biến (Boundary Identification)

### 2.1 Biến `newPassword` (Độ dài mật khẩu)
- **Ràng buộc từ Spec (FR-01, FR-03):** Mật khẩu tối thiểu 8 ký tự ($min = 8$). Không có giới hạn tối đa tường minh nhưng thông thường các hệ thống giới hạn an toàn ở mức 128 hoặc 255 ký tự.
- **Tập điểm biên khảo sát:**
  - $min - 1 = 7$ ký tự (Không hợp lệ - Invalid Boundary)
  - $min = 8$ ký tự (Hợp lệ - Cận dưới chuẩn)
  - $min + 1 = 9$ ký tự (Hợp lệ - Ngay trên cận dưới)
  - $nominal = 12$ ký tự (Hợp lệ - Giá trị điển hình)
  - Độ dài rất lớn (Stress boundary): 255 ký tự (Hợp lệ) và 256 ký tự (Kiểm tra giới hạn buffer/DB).

### 2.2 Biến `resetToken` (Độ dài mã OTP)
- **Ràng buộc từ Spec (FR-03, SEC-07):** Mã OTP gồm đúng 6 chữ số ngẫu nhiên ($L = 6$).
- **Tập điểm biên khảo sát:**
  - $L - 1 = 5$ chữ số (Không hợp lệ - Thiếu 1 ký số)
  - $L = 6$ chữ số (Hợp lệ - Độ dài chuẩn)
  - $L + 1 = 7$ chữ số (Không hợp lệ - Thừa 1 ký số)
  - **Biên giá trị số học (Numerical range của 6 chữ số):**
    - Cận dưới: `000000` (Kiểm tra xử lý số 0 ở đầu chuỗi - Leading zeros)
    - Cận trên: `999999` (Giá trị lớn nhất có 6 chữ số)

### 2.3 Biến `email` (Độ dài chuỗi Email)
- **Ràng buộc chuẩn RFC 5321:** Tối đa 254 ký tự.
- **Tập điểm biên khảo sát:**
  - Cận dưới: Chuỗi 0 ký tự (Rỗng - đã bao phủ ở DT-002), Chuỗi 1 ký tự (`a` - Không hợp lệ).
  - Email ngắn nhất hợp lệ theo cú pháp RFC: `a@b.co` (6 ký tự).
  - Email dài 254 ký tự (Hợp lệ) và 255 ký tự (Vượt ngưỡng RFC).

---

## 3. Bảng Ca Kiểm thử Giá trị Biên (BVA Test Cases)

| Mã Test Case | Biến & Điểm biên kiểm tra | Loại biên (Valid / Invalid) | Dữ liệu kiểm thử (Test Data) | Các bước thực hiện & Tiền điều kiện | Kết quả mong đợi (Expected Result) | Độ ưu tiên | Trạng thái | Ghi chú & Đối chiếu Step 2 |
| :--- | :--- | :---: | :--- | :--- | :--- | :---: | :---: | :--- |
| **TC-FR03-BV-001** | `newPassword` length = 7 ($min - 1$) | Invalid Boundary | `P@ss12a` (7 ký tự, đủ 4 loại ký tự) | Tiền điều kiện: Ở Step 2.<br>1. Nhập OTP hợp lệ<br>2. Nhập mật khẩu 7 ký tự<br>3. Xác nhận khớp<br>4. Bấm Đặt lại mật khẩu | Hệ thống báo lỗi mật khẩu phải dài tối thiểu 8 ký tự *(FR-01, FR-03)* | High | Not executed | BVA cụ thể cho cận $min-1$ |
| **TC-FR03-BV-002** | `newPassword` length = 8 ($min$) | Valid Boundary | `P@ssw12a` (đúng 8 ký tự, đủ 4 loại ký tự) | 1. Nhập OTP hợp lệ<br>2. Nhập mật khẩu 8 ký tự<br>3. Xác nhận khớp<br>4. Bấm Đặt lại mật khẩu | Đổi mật khẩu thành công, vô hiệu hóa OTP, chuyển về login *(FR-01, FR-03, SEC-07)* | High | Not executed | Cận dưới hợp lệ chính xác $min$ |
| **TC-FR03-BV-003** | `newPassword` length = 9 ($min + 1$) | Valid Boundary | `P@ssw123a` (9 ký tự, đủ 4 loại ký tự) | 1. Nhập OTP hợp lệ<br>2. Nhập mật khẩu 9 ký tự<br>3. Xác nhận khớp<br>4. Bấm Đặt lại mật khẩu | Đổi mật khẩu thành công *(FR-01, FR-03)* | Medium | Not executed | Điểm biên lân cận $min+1$ |
| **TC-FR03-BV-004** | `newPassword` length = 255 (Max boundary) | Valid Boundary | Chuỗi 255 ký tự thỏa mãn đủ 4 nhóm ký tự | 1. Nhập OTP hợp lệ<br>2. Nhập mật khẩu 255 ký tự<br>3. Bấm Đặt lại mật khẩu | Hệ thống chấp nhận và mã hóa an toàn bằng bcrypt/argon2 *(SEC-01)* | Medium | Not executed | Biên trên thông thường của trường text |
| **TC-FR03-BV-005** | `newPassword` length = 256 (Max + 1) | Invalid Boundary | Chuỗi 256 ký tự | 1. Nhập OTP hợp lệ<br>2. Nhập mật khẩu 256 ký tự<br>3. Bấm Đặt lại mật khẩu | Báo lỗi độ dài vượt quá giới hạn hoặc xử lý an toàn không sập server | Low | Not executed | Kiểm tra giới hạn tràn buffer/DoS |
| **TC-FR03-BV-006** | `resetToken` length = 5 ($L - 1$) | Invalid Boundary | `"12345"` (5 chữ số) | 1. Nhập OTP 5 chữ số<br>2. Nhập mật khẩu hợp lệ<br>3. Bấm Đặt lại mật khẩu | Báo lỗi OTP không hợp lệ, yêu cầu đúng 6 chữ số *(FR-03/Bước 1, SEC-07)* | High | Not executed | Khảo sát sát cận độ dài OTP |
| **TC-FR03-BV-007** | `resetToken` length = 6 ($L$ chuẩn) | Valid Boundary | `<OTP 6 số thực tế>` (vd: `654321`) | Đã bao phủ tại **TC-FR03-DT-008** | Tham chiếu chéo TC-FR03-DT-008 *(Độ dài chuẩn)* | - | Tham chiếu | Trùng khớp với TC-FR03-DT-008 |
| **TC-FR03-BV-008** | `resetToken` length = 7 ($L + 1$) | Invalid Boundary | `"1234567"` (7 chữ số) | 1. Nhập OTP 7 chữ số<br>2. Nhập mật khẩu hợp lệ<br>3. Bấm Đặt lại mật khẩu | Báo lỗi OTP không hợp lệ *(FR-03/Bước 1, SEC-07)* | High | Not executed | Tham chiếu tương đương TC-FR03-DT-011 |
| **TC-FR03-BV-009** | `resetToken` chứa số 0 ở đầu (Leading zero) | Valid Boundary (Format) | `"012345"` (chuỗi 6 ký số, bắt đầu bằng `0`) | 1. Gửi request hoặc nhập OTP có chữ số `0` ở đầu<br>2. Submit | Xử lý chuỗi chính xác 6 ký tự, không bị ép kiểu làm mất số 0 thành 5 số (`12345`) *(FR-03/Bước 1, SEC-07)* | High | Not executed | Lỗi tiềm ẩn do ép kiểu int tự động |
| **TC-FR03-BV-010** | `email` có độ dài tối thiểu hợp lệ ($min$) | Valid Boundary | `a@b.co` (6 ký tự) | 1. Đăng ký email `a@b.co`<br>2. Thực hiện quên mật khẩu với email này | Hệ thống gửi OTP thành công *(FR-01, FR-03)* | Medium | Not executed | Biên độ dài email ngắn nhất |
| **TC-FR03-BV-011** | `email` có độ dài tối đa chuẩn RFC (254 ký tự) | Valid Boundary | `<Email hợp lệ đúng 254 ký tự>` | 1. Nhập email 254 ký tự đã đăng ký<br>2. Bấm Lấy mã OTP | Hệ thống chấp nhận và xử lý bình thường *(RFC 5321)* | Low | Not executed | Biên trên độ dài email |

---

## 4. Tổng kết Độ phủ Giá trị Biên (BVA Coverage Summary)

- **Tổng số ca kiểm thử BVA mới bổ sung:** **9 test cases** (`TC-FR03-BV-001` đến `TC-FR03-BV-006`, `TC-FR03-BV-008` đến `TC-FR03-BV-011`).
- **Ca kiểm thử tham chiếu chéo (trùng khớp với Domain Testing):** **1 test case** (`TC-FR03-BV-007` $\rightarrow$ `TC-FR03-DT-008`).
- **Tổng số test cases lũy kế của FR-03 (Domain + BVA):** **33 test cases** (24 Domain + 9 BVA mới).
