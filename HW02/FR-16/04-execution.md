# Báo cáo Thực thi Kiểm thử (Test Execution Log): FR-16 (Import Sản phẩm từ CSV)

## 1. Môi trường Thực thi (Execution Environment)
- **Hệ thống thử nghiệm (SUT):** EShop Web Admin (`http://localhost:5174`) & Backend REST API (`http://localhost:3000`)
- **Tài khoản kiểm thử:**
  - Admin: `admin@eshop.com` / `Admin123!`
  - User thường: `test@eshop.com` / `Test1234!`
- **Ngày thực thi:** 2026-10-15
- **Công cụ & Phương pháp:** Code inspection (`frontend-admin/src/App.jsx:355-425`, `backend/server.js:199-241`) kết hợp chạy kịch bản tự động Node.js HTTP request và đối chiếu trực tiếp dữ liệu CSDL SQLite.

---

## 2. Bảng Kết quả Thực thi Chi tiết (Execution Results Table)

| Mã Test Case | Loại kiểm thử | Mục tiêu kiểm tra | Kết quả mong đợi (Expected Result) | Kết quả thực tế (Actual Result) | Đánh giá (Pass/Fail) | Minh chứng / Bằng chứng (Evidence) | Mã Bug liên kết |
| :--- | :--- | :--- | :--- | :--- | :---: | :--- | :---: |
| **TC-FR16-DT-001** | Domain | Import file CSV 3 dòng hợp lệ (Happy Path) | Import thành công 3/3 sản phẩm, lưu CSDL, trả về thông báo thành công *(FR-16)* | Import thành công 3 sản phẩm, CSDL tăng 3 | **PASS** | HTTP 200 `message: "Import hoàn tất: 3/3 sản phẩm được thêm"` | - |
| **TC-FR16-DT-002** | Domain | **Xử lý chuẩn RFC 4180 (dấu phẩy trong dấu nháy kép `""`)** | Phân tách chính xác các trường có dấu phẩy bọc trong nháy kép `""`, không bị xô lệch cột *(FR-16)* | **LỖI NGHIÊM TRỌNG:** Phía frontend chỉ dùng hàm `line.split(",")` thô sơ (`App.jsx:371`), không nhận diện được dấu nháy kép RFC 4180. Dấu phẩy trong chuỗi `"RAM 16GB, SSD 512GB"` bị chia tách làm lệch toàn bộ cột: `description` nhận `"RAM 16GB`, `imageUrl` nhận `SSD 512GB"`, `category_id` nhận `http://...`! | **FAIL** (Critical Bug) | Kiểm tra `App.jsx:370-377`: `line.split(",")` làm sai lệch cấu trúc dữ liệu của toàn bộ các dòng theo chuẩn RFC 4180 | `BUG-FR16-01` |
| **TC-FR16-DT-003** | Domain | Upload file không phải định dạng `.csv` | Hệ thống từ chối tải lên file `.txt` / `.xlsx` *(FR-16)* | Thẻ input trên giao diện `App.jsx:356-384` **không có thuộc tính `accept=".csv"`**! Người dùng có thể chọn bất kỳ loại file nào (hình ảnh, text, binary...) để parse | **FAIL** | Thẻ `<input type="file" ... />` thiếu `accept=".csv"` | `BUG-FR16-02` |
| **TC-FR16-DT-004** | Domain | Header CSV sai quy chuẩn | Báo lỗi cấu trúc file/header không hợp lệ *(FR-16)* | Mã nguồn frontend map cứng header từ dòng 1 không qua validation; backend không kiểm tra header bắt buộc | **FAIL** | Thiếu kiểm tra validation header chuẩn của file CSV | `BUG-FR16-02` |
| **TC-FR16-DT-005** | Domain | Dòng có `name` rỗng | Báo lỗi dòng thiếu tên sản phẩm, từ chối import *(FR-16)* | Backend phát hiện `!row.name`, đẩy vào mảng `errors` và không insert dòng đó | **PASS** | Backend trả về `errors: ["Hàng ...: Thiếu tên sản phẩm"]` | - |
| **TC-FR16-DT-006** | Domain | Dòng có `price <= 0` (giá âm hoặc bằng 0) | **Từ chối**, báo lỗi giá phải là số dương (> 0) *(FR-16)* | **LỖI:** Backend `server.js:213-232` **hoàn toàn không kiểm tra `price > 0`**! Hệ thống chấp nhận cả `price = -50000` và `price = 0`, ghi thẳng vào CSDL SQLite | **FAIL** | Gửi mảng có `price: -50000`, API vẫn insert thành công và trả về `inserted: 1` | `BUG-FR16-03` |
| **TC-FR16-DT-007** | Domain | Dòng có `price` là chuỗi chữ | Báo lỗi kiểu dữ liệu số *(FR-16)* | Backend không ép kiểu/validate số, ghi chuỗi thẳng vào cột price | **FAIL** | Không kiểm tra `isNaN(row.price)` | `BUG-FR16-03` |
| **TC-FR16-DT-008** | Domain | Dòng có `category_id` không tồn tại | Báo lỗi danh mục không hợp lệ *(FR-16, FR-15)* | Backend gán fallback `category_id || 1`, nếu gửi ID lạ vẫn chấp nhận do SQLite không bật Foreign Keys | **FAIL** | Không kiểm tra ràng buộc khóa ngoại danh mục | `BUG-FR16-03` |
| **TC-FR16-DT-009** | Domain | **Tính nguyên tử All-or-Nothing (Lỗi ở dòng cuối)** | *"Nếu có lỗi ở bất kỳ dòng nào, toàn bộ import phải được **rollback** (giao dịch nguyên tử — all-or-nothing)"* *(FR-16)* | **LỖI ĐẶC BIỆT NGHIÊM TRỌNG:** Backend **hoàn toàn không sử dụng Database Transaction** (`BEGIN TRANSACTION / ROLLBACK`)! Hệ thống dùng vòng lặp `rows.forEach()` insert độc lập từng dòng. Khi gặp dòng lỗi, các dòng hợp lệ trước đó **đã bị commit vào CSDL và không thể hoàn tác**, gây rò rỉ dữ liệu dở dang (partial insert)! | **FAIL** (Critical Bug) | Batch 3 dòng (dòng 1, 2 hợp lệ, dòng 3 rỗng tên): CSDL tăng 2 sản phẩm! Vi phạm trắng trợn quy tắc Rollback All-or-Nothing | `BUG-FR16-04` |
| **TC-FR16-DT-010** | Domain | Rollback All-or-Nothing khi lỗi ở dòng giữa | Toàn bộ giao dịch bị Rollback, không thêm sản phẩm nào *(FR-16)* | Các dòng hợp lệ xung quanh dòng lỗi vẫn bị chèn vào CSDL | **FAIL** | Vòng lặp `server.js:213` xử lý từng dòng độc lập, không có transaction | `BUG-FR16-04` |
| **TC-FR16-DT-011** | Domain | Báo cáo kết quả chi tiết | Hiển thị báo cáo rõ ràng: số dòng thành công, số dòng lỗi và lý do *(FR-16)* | API trả về JSON gồm `inserted` và mảng `errors`. Web Admin hiển thị kết quả | **PASS** | Giao diện hiển thị số dòng thành công và danh sách lỗi | - |
| **TC-FR16-DT-012** | Domain | User thường gọi `POST /api/admin/import-products` | **Từ chối Forbidden (403)** *(FR-12, SEC-03)* | **LỖI BẢO MẬT:** API chỉ dùng `authenticateToken` chung chung mà **KHÔNG kiểm tra `role === 'admin'`**! Tài khoản người dùng thường mang token `role: 'user'` vẫn gọi API import và chèn dữ liệu vào hệ thống bình thường với HTTP 200! | **FAIL** (Security Bug) | Gửi request kèm token của `test@eshop.com`, API phản hồi HTTP 200 `inserted: 1` | `BUG-FR16-05` |
| **TC-FR16-DT-013** | Domain | Không token gọi API import | Từ chối Unauthorized (401) *(SEC-02)* | Middleware `authenticateToken` chặn lại với HTTP 401 | **PASS** | HTTP 401 `{"error":"Unauthorized"}` | - |
| **TC-FR16-BV-001** | BVA | File rỗng 0 byte | Báo lỗi không có dữ liệu để import *(FR-16)* | Báo lỗi không có dữ liệu | **PASS** | Trả về thông báo lỗi phù hợp | - |
| **TC-FR16-BV-002** | BVA | File chỉ có dòng Header (0 dòng dữ liệu) | Báo lỗi không có dòng sản phẩm nào *(FR-16)* | Client parse ra mảng rỗng `[]`, backend từ chối với HTTP 400 | **PASS** | HTTP 400 `{"error":"Không có dữ liệu để import"}` | - |
| **TC-FR16-BV-003** | BVA | `row_count = 1` ($min$ dòng hợp lệ) | Import thành công 1 sản phẩm | Import thành công 1 sản phẩm | **PASS** | HTTP 200 `inserted: 1` | - |
| **TC-FR16-BV-004** | BVA | `row_count = 2` ($min + 1$ dòng) | Import thành công 2 sản phẩm | Import thành công 2 sản phẩm | **PASS** | HTTP 200 `inserted: 2` | - |
| **TC-FR16-BV-005** | BVA | `error_row_pos = 1` (Lỗi dòng đầu) | Rollback toàn bộ các dòng sau *(FR-16)* | Các dòng sau vẫn bị insert vào CSDL bình thường | **FAIL** | Lỗi thiếu Transaction rollback | `BUG-FR16-04` |
| **TC-FR16-BV-006** | BVA | `error_row_pos = N` (Lỗi dòng cuối trong file 5 dòng) | Rollback toàn bộ $N-1$ dòng trước đó *(FR-16)* | Cả $N-1$ dòng trước đều bị ghi vào CSDL, không hề rollback | **FAIL** | Lỗi thiếu Transaction rollback | `BUG-FR16-04` |
| **TC-FR16-BV-007** | BVA | `price = 1` ($min$ giá nguyên dương) | Import thành công với giá 1 ₫ | Import thành công | **PASS** | HTTP 200 | - |
| **TC-FR16-BV-008** | BVA | `price = 0` ($min - 1$ giá vi phạm) | **Từ chối**, báo lỗi giá phải là số dương *(FR-16)* | Hệ thống **chấp nhận giá bằng 0** và insert thành công vào CSDL! | **FAIL** | Backend thiếu validation `price > 0` | `BUG-FR16-03` |
| **TC-FR16-BV-009** | BVA | `name` length = 1 ($min$) | Import thành công tên 1 ký tự | Import thành công | **PASS** | HTTP 200 | - |
| **TC-FR16-BV-010** | BVA | `name` length = 255 ($max$) | Import thành công tên 255 ký tự | Import thành công | **PASS** | HTTP 200 | - |
| **TC-FR16-BV-011** | BVA | `name` length = 256 ($max + 1$) | Báo lỗi tên vượt quá 255 ký tự, rollback import *(FR-16, FR-15)* | Hệ thống không validate độ dài, lưu trọn chuỗi 256 ký tự vào SQLite | **FAIL** | Backend thiếu ràng buộc độ dài tên tối đa | `BUG-FR16-03` |

---

## 3. Thống kê Kết quả Thực thi (Execution Summary)

- **Tổng số ca kiểm thử đã thiết kế (Designed):** **24**
- **Tổng số ca kiểm thử đã thực thi (Executed):** **24**
- **Số ca kiểm thử ĐẠT (Passed):** **11** (45.8%)
- **Số ca kiểm thử THẤT BẠI (Failed):** **13** (54.2%)
- **Số ca kiểm thử chưa chạy (Not executed):** **0**
- **Tổng số lỗi phát hiện (Bugs identified):** **5 lỗi nghiêm trọng** (`BUG-FR16-01` đến `BUG-FR16-05`)

---

## 4. Danh sách các Lỗi Phát hiện (Bug List)
1. **`BUG-FR16-01`**: Trình phân tích CSV phía Client không tuân thủ chuẩn RFC 4180 (dùng `split(",")` làm vỡ cấu trúc cột khi trường text chứa dấu phẩy trong cặp nháy kép `""`).
2. **`BUG-FR16-02`**: Giao diện Web Admin không giới hạn định dạng file upload (`accept=".csv"`) và không xác thực cấu trúc Header chuẩn.
3. **`BUG-FR16-03`**: Backend API `POST /api/admin/import-products` thiếu toàn bộ các lớp validation nghiệp vụ cho từng dòng (chấp nhận `price <= 0`, `name > 255` ký tự, không kiểm tra kiểu số).
4. **`BUG-FR16-04`**: Hệ thống vi phạm nguyên tắc giao dịch nguyên tử (Atomic Transaction): Hoàn toàn không có cơ chế **Rollback All-or-Nothing** khi phát hiện dòng lỗi, gây rò rỉ dữ liệu dở dang (Partial Inserts).
5. **`BUG-FR16-05`**: Lỗ hổng phân quyền nghiêm trọng tại API `/api/admin/import-products`: Thiếu middleware kiểm tra vai trò Quản trị viên (`role === 'admin'`), cho phép người dùng thường thực hiện thao tác quản trị.
