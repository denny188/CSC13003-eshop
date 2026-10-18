# Báo cáo Tổng kết Bài tập lớn HW02: Domain Testing & Boundary Value Analysis (EShop)

- **Sinh viên thực hiện:** Nguyễn Lê Hữu Điền
- **Mã số sinh viên:** `23120233`
- **Lớp:** CSC13003 — Kiểm thử Phần mềm
- **Repository bài làm:** [https://github.com/denny188/CSC13003-eshop](https://github.com/denny188/CSC13003-eshop)
- **Nhánh kiểm thử (Branch):** `hw02-23120233`
- **Hệ thống thử nghiệm (SUT):** EShop (Node.js/Express, SQLite, React Web, React Native Expo Mobile)

---

## 1. Bảng Tự Đánh Giá (Self-Assessment Table)

| STT | Tiêu chí đánh giá (Criteria) | Điểm tối đa (Max) | Điểm tự đánh giá (Self-Assessed) | Minh chứng hoàn thành |
| :---: | :--- | :---: | :---: | :--- |
| 1 | **Feature 1 (Pool A - FR-03):** Domain + Boundary | 2.0 | **2.0** | Hoàn thành đủ 5 file tại `HW02/FR-03/`, 34 test cases, 7 bug reports |
| 2 | **Feature 2 (Pool B - FR-10):** Domain + Boundary | 2.0 | **2.0** | Hoàn thành đủ 5 file tại `HW02/FR-10/`, 30 test cases, 3 bug reports |
| 3 | **Feature 3 (Pool C - FR-15 & FR-16):** Domain + Boundary | 2.0 | **2.0** | Hoàn thành 10 file tại `HW02/FR-15/` & `HW02/FR-16/`, 50 test cases, 12 bug reports |
| 4 | **Feature 4 (Pool D - FR-20 Mobile: FR-07):** Domain + Boundary | 2.0 | **2.0** | Hoàn thành đủ 5 file tại `HW02/FR-20-mobile/`, 19 test cases, 6 bug reports |
| 5 | **Agent Skills + YouTube Demo Video + AI Audit Report** | 2.0 | **2.0** | Đầy đủ `.agent/SKILL.md`, 30 logs audit tại `HW02/ai-audit/`, bài luận AI Critique, và Video demo YouTube |
| **TỔNG** | **Tổng điểm đồ án HW02** | **10.0** | **10.0** | **100% yêu cầu đạt mức tối đa** |

---

## 2. Báo cáo Tổng hợp Thống kê Kiểm thử (Test Summary Report)

| Nhóm (Pool) | Mã tính năng (FR) | Tên tính năng | Nền tảng | Thiết kế (Designed) | Thực thi (Executed) | Đạt (Passed) | Thất bại (Failed) | Chưa chạy | Số Bug phát hiện |
| :---: | :---: | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Pool A** | **FR-03** | Quên mật khẩu & Đặt lại mật khẩu (2 bước) | Web / API | 34 | 34 | 22 | 12 | 0 | **7 bugs** |
| **Pool B** | **FR-10** | Trạng thái đơn hàng (Order State Machine) | Web / API | 30 | 30 | 27 | 3 | 0 | **3 bugs** |
| **Pool C** | **FR-15** | Quản lý sản phẩm (Product CRUD) | Web Admin | 26 | 26 | 11 | 15 | 0 | **7 bugs** |
| **Pool C** | **FR-16** | Import sản phẩm từ CSV | Web Admin / API | 24 | 24 | 11 | 13 | 0 | **5 bugs** |
| **Pool D** | **FR-20** | Giỏ hàng Mobile (theo chuẩn FR-07) | React Native Mobile | 19 | 19 | 8 | 11 | 0 | **6 bugs** |
| **TỔNG CỘNG** | **5 Features** | **Đầy đủ 4 Pools** | **Đa nền tảng** | **133** | **133** | **79 (59.4%)** | **54 (40.6%)** | **0** | **28 bugs** |

---

## 3. Danh sách 28 Lỗi Đã Đăng lên GitHub Issues

Toàn bộ 28 lỗi phát hiện được đã được tự động đăng lên GitHub Issues của repository [denny188/CSC13003-eshop/issues](https://github.com/denny188/CSC13003-eshop/issues) kèm đầy đủ nhãn phân loại (labels), mô tả, các bước tái hiện và giải pháp:

- **FR-03 (Quên mật khẩu):**
  - [Issue #1](https://github.com/denny188/CSC13003-eshop/issues/1): `[BUG-FR03-01]` Mã OTP chỉ sinh ra 4 chữ số thay vì 6 chữ số
  - [Issue #2](https://github.com/denny188/CSC13003-eshop/issues/2): `[BUG-FR03-02]` Trường Email dùng sai type="text"
  - [Issue #3](https://github.com/denny188/CSC13003-eshop/issues/3): `[BUG-FR03-03]` Giao diện thiếu chỉ báo bước (Step Indicator)
  - [Issue #4](https://github.com/denny188/CSC13003-eshop/issues/4): `[BUG-FR03-04]` Mật khẩu lưu dạng plaintext trong CSDL SQLite
  - [Issue #5](https://github.com/denny188/CSC13003-eshop/issues/5): `[BUG-FR03-05]` Backend không kiểm tra độ mạnh mật khẩu khi reset
  - [Issue #6](https://github.com/denny188/CSC13003-eshop/issues/6): `[BUG-FR03-06]` Regex kiểm tra mật khẩu ở Client sai quy chuẩn
  - [Issue #7](https://github.com/denny188/CSC13003-eshop/issues/7): `[BUG-FR03-07]` Giao diện thiếu trường Xác nhận mật khẩu mới

- **FR-10 (Trạng thái Đơn hàng):**
  - [Issue #8](https://github.com/denny188/CSC13003-eshop/issues/8): `[BUG-FR10-01]` Khách hàng vẫn tự hủy được đơn khi đang ở trạng thái shipping
  - [Issue #9](https://github.com/denny188/CSC13003-eshop/issues/9): `[BUG-FR10-02]` Cho phép chuyển trạng thái từ canceled sang delivered
  - [Issue #10](https://github.com/denny188/CSC13003-eshop/issues/10): `[BUG-FR10-03]` Lỗ hổng Broken Access Control tại API chuyển trạng thái đơn hàng

- **FR-15 (Quản lý Sản phẩm Web Admin):**
  - [Issue #11](https://github.com/denny188/CSC13003-eshop/issues/11): `[BUG-FR15-01]` Backend không kiểm tra bắt buộc tên sản phẩm
  - [Issue #12](https://github.com/denny188/CSC13003-eshop/issues/12): `[BUG-FR15-02]` Chấp nhận giá sản phẩm bằng 0 hoặc số âm
  - [Issue #13](https://github.com/denny188/CSC13003-eshop/issues/13): `[BUG-FR15-03]` Form thiếu thuộc tính required ở ô giá
  - [Issue #14](https://github.com/denny188/CSC13003-eshop/issues/14): `[BUG-FR15-04]` Chấp nhận mã danh mục không tồn tại
  - [Issue #15](https://github.com/denny188/CSC13003-eshop/issues/15): `[BUG-FR15-05]` Khi sửa 1 sản phẩm thì toàn bộ sản phẩm khác bị đổi tên theo
  - [Issue #16](https://github.com/denny188/CSC13003-eshop/issues/16): `[BUG-FR15-06]` Các API CRUD sản phẩm hoàn toàn mở không có bảo vệ
  - [Issue #17](https://github.com/denny188/CSC13003-eshop/issues/17): `[BUG-FR15-07]` Chấp nhận tên sản phẩm dài vượt quá 255 ký tự

- **FR-16 (Import Sản phẩm từ CSV):**
  - [Issue #18](https://github.com/denny188/CSC13003-eshop/issues/18): `[BUG-FR16-01]` Parser CSV Client dùng split(",") làm vỡ cấu trúc cột RFC 4180
  - [Issue #19](https://github.com/denny188/CSC13003-eshop/issues/19): `[BUG-FR16-02]` Form upload thiếu accept=".csv" và thiếu validate header
  - [Issue #20](https://github.com/denny188/CSC13003-eshop/issues/20): `[BUG-FR16-03]` API import thiếu validation từng dòng (giá âm, rỗng)
  - [Issue #21](https://github.com/denny188/CSC13003-eshop/issues/21): `[BUG-FR16-04]` Vi phạm giao dịch nguyên tử All-or-Nothing, không có rollback khi lỗi
  - [Issue #22](https://github.com/denny188/CSC13003-eshop/issues/22): `[BUG-FR16-05]` API import thiếu kiểm tra quyền Admin

- **FR-20 Mobile (Giỏ hàng):**
  - [Issue #23](https://github.com/denny188/CSC13003-eshop/issues/23): `[BUG-MOB07-01]` Giao diện thiếu nút tăng giảm số lượng (+/-)
  - [Issue #24](https://github.com/denny188/CSC13003-eshop/issues/24): `[BUG-MOB07-02]` Nhập số lượng trong giỏ bị tự động cộng thừa 1 (parsed + 1)
  - [Issue #25](https://github.com/denny188/CSC13003-eshop/issues/25): `[BUG-MOB07-03]` Nút Xóa sản phẩm xóa ngay mà không có Dialog xác nhận
  - [Issue #26](https://github.com/denny188/CSC13003-eshop/issues/26): `[BUG-MOB07-04]` Nhãn tổng tiền hiển thị sai thành "Tổng tạm tính" thay vì "Tổng cộng"
  - [Issue #27](https://github.com/denny188/CSC13003-eshop/issues/27): `[BUG-MOB07-05]` Giỏ hàng trống thiếu hình ảnh minh họa
  - [Issue #28](https://github.com/denny188/CSC13003-eshop/issues/28): `[BUG-MOB07-06]` Nút quay lại trang chủ bị đặt sai nhãn thành "← Mua tiếp"

---

## 4. Cấu trúc Thư mục Nộp bài (Submission Structure)

- **Cấu trúc lưu trữ thư mục `HW02/`:**
  - `HW02/FR-03/`: Bộ 5 file kiểm thử tính năng Quên mật khẩu.
  - `HW02/FR-10/`: Bộ 5 file kiểm thử Order State Machine.
  - `HW02/FR-15/`: Bộ 5 file kiểm thử Product CRUD Web Admin.
  - `HW02/FR-16/`: Bộ 5 file kiểm thử Product CSV Import Web Admin.
  - `HW02/FR-20-mobile/`: Bộ 5 file kiểm thử Giỏ hàng Mobile React Native.
  - `HW02/bugs/`: 28 file báo cáo lỗi chi tiết chuẩn GitHub Issue.
  - `HW02/ai-audit/`: 30 file nhật ký tương tác và phản biện kiểm toán AI.
  - `HW02/AI_Critique.md`: Báo cáo phản biện năng lực AI theo chuẩn Bloom-AI G9.3.
  - `HW02/commit_log.txt`: Nhật ký 35 commits Git theo từng bước kiểm thử.
