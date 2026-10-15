# Báo cáo Thực thi Kiểm thử (Test Execution Log): FR-15 (Quản lý Sản phẩm)

## 1. Môi trường Thực thi (Execution Environment)
- **Hệ thống thử nghiệm (SUT):** EShop Web Admin (`http://localhost:5174`) & Backend REST API (`http://localhost:3000`)
- **Tài khoản kiểm thử:**
  - Admin: `admin@eshop.com` / `Admin123!`
  - User: `test@eshop.com` / `Test1234!`
- **Ngày thực thi:** 2026-10-15
- **Công cụ:** Node.js Fetch HTTP Client & Code Inspection tại `frontend-admin/src/App.jsx`.

---

## 2. Bảng Kết quả Thực thi Chi tiết (Execution Results Table)

| Mã Test Case | Loại kiểm thử | Mục tiêu kiểm tra | Kết quả mong đợi (Expected Result) | Kết quả thực tế (Actual Result) | Đánh giá (Pass/Fail) | Minh chứng / Bằng chứng (Evidence) | Mã Bug liên kết |
| :--- | :--- | :--- | :--- | :--- | :---: | :--- | :---: |
| **TC-FR15-DT-001** | Domain | Tạo sản phẩm hợp lệ | Tạo sản phẩm thành công, lưu CSDL, HTTP 200 *(FR-15)* | Tạo thành công, lưu CSDL, trả về ID mới | **PASS** | HTTP 200 `{"message":"Product created","id":...}` | - |
| **TC-FR15-DT-002** | Domain | Tên để trống (`""`) | Báo lỗi bắt buộc nhập tên sản phẩm *(FR-15, FR-22)* | - Trên UI: input có `required` nên form chặn.<br>- Trên API: **Backend không validate**, nhận `name: ""` và tạo sản phẩm rỗng thành công! | **FAIL** | Gửi `name: ""` lên `POST /api/products` trả về HTTP 200 `{"message":"Product created"}` | `BUG-FR15-01` |
| **TC-FR15-DT-003** | Domain | Tên toàn khoảng trắng | Báo lỗi tên không hợp lệ *(FR-15)* | Backend chấp nhận chuỗi khoảng trắng và lưu vào DB với HTTP 200 | **FAIL** | HTTP 200 `Product created` | `BUG-FR15-01` |
| **TC-FR15-DT-004** | Domain | XSS trong tên sản phẩm | Escape HTML an toàn khi render *(SEC-04)* | React tự động escape text trong thẻ JSX khi render | **PASS** | XSS được escape, không kích hoạt script | - |
| **TC-FR15-DT-005** | Domain | Giá bằng 0 (`price = 0`) | **Từ chối**, báo lỗi giá phải là số dương (> 0) *(FR-15)* | Hệ thống **chấp nhận giá bằng 0** và tạo sản phẩm thành công với HTTP 200! | **FAIL** | Code `server.js:167-177` hoàn toàn không có câu lệnh kiểm tra `price > 0` | `BUG-FR15-02` |
| **TC-FR15-DT-006** | Domain | Giá âm (`price = -50000`) | **Từ chối**, báo lỗi giá phải là số dương (> 0) *(FR-15)* | Hệ thống **chấp nhận giá âm** (`-50000`) và tạo sản phẩm thành công với HTTP 200! | **FAIL** | CSDL lưu sản phẩm với `price = -50000` | `BUG-FR15-02` |
| **TC-FR15-DT-007** | Domain | Để trống giá tiền | Báo lỗi giá bắt buộc phải nhập *(FR-15, FR-22)* | Ô giá trên form Web Admin (`App.jsx:500-508`) **không có thuộc tính `required`**! Cho phép submit không có giá | **FAIL** | Code `App.jsx:503`: `<input placeholder="Giá tiền" type="number" ... />` thiếu `required` | `BUG-FR15-03` |
| **TC-FR15-DT-008** | Domain | Giá là chuỗi ký tự | Báo lỗi kiểu dữ liệu số *(FR-15)* | SQLite tự ép kiểu hoặc lưu chuỗi, không kiểm tra | **FAIL** | API không kiểm tra kiểu dữ liệu của `price` | `BUG-FR15-02` |
| **TC-FR15-DT-009** | Domain | Danh mục không tồn tại (`9999`) | Báo lỗi danh mục phải chọn từ danh sách có sẵn *(FR-15)* | Backend chấp nhận `category_id: 9999` dù danh mục này không tồn tại trong bảng `categories` | **FAIL** | SQLite mặc định không bật `PRAGMA foreign_keys = ON`, API không validate ID danh mục | `BUG-FR15-04` |
| **TC-FR15-DT-010** | Domain | Để trống danh mục | Báo lỗi bắt buộc chọn danh mục *(FR-15)* | Backend nhận `null` và lưu thành công | **FAIL** | Không validate danh mục | `BUG-FR15-04` |
| **TC-FR15-DT-011** | Domain | **Update Isolation (Tính cô lập khi sửa)** | Khi Sửa một sản phẩm, **chỉ sản phẩm đó bị thay đổi — các sản phẩm khác giữ nguyên** *(FR-15)* | **LỖI NGHIÊM TRỌNG:** Khi sửa 1 sản phẩm trên Web Admin, code cố tình ghi đè toàn bộ danh sách sản phẩm bằng tên sản phẩm vừa sửa (`fakeMassUpdatedProducts`)! Khiến tất cả sản phẩm khác đều bị đổi tên theo! | **FAIL** (Critical Bug) | Code `App.jsx:110-114`: `const fakeMassUpdatedProducts = products.map((p) => ({ ...p, name: productForm.name })); setProducts(fakeMassUpdatedProducts);` | `BUG-FR15-05` |
| **TC-FR15-DT-012** | Domain | Xóa sản phẩm hợp lệ | Xóa sản phẩm thành công, HTTP 200 *(FR-15)* | Xóa sản phẩm thành công | **PASS** | HTTP 200 `{"message":"Product deleted"}` | - |
| **TC-FR15-DT-013** | Domain | User thường gọi POST /api/products | **Từ chối Forbidden**, HTTP 403 *(SEC-03)* | API `POST /api/products` **hoàn toàn mở (public)**, không hề có middleware xác thực token hay kiểm tra Admin! Khách hoặc User thường đều tự do thêm sản phẩm! | **FAIL** | Code `server.js:167`: `app.post("/api/products", (req, res) => { ... })` không có `authenticateToken` | `BUG-FR15-06` |
| **TC-FR15-DT-014** | Domain | User thường gọi PUT /api/products/:id | **Từ chối Forbidden**, HTTP 403 *(SEC-03)* | API `PUT /api/products/:id` **hoàn toàn mở**, ai cũng sửa được sản phẩm | **FAIL** | Code `server.js:179`: `app.put("/api/products/:id", (req, res) => { ... })` không có xác thực | `BUG-FR15-06` |
| **TC-FR15-DT-015** | Domain | User thường gọi DELETE /api/products/:id | **Từ chối Forbidden**, HTTP 403 *(SEC-03)* | API `DELETE /api/products/:id` **hoàn toàn mở**, ai cũng xóa được sản phẩm | **FAIL** | Code `server.js:191`: `app.delete("/api/products/:id", (req, res) => { ... })` không có xác thực | `BUG-FR15-06` |
| **TC-FR15-DT-016** | Domain | Không token gọi API Thêm/Sửa/Xóa | Từ chối 401 Unauthorized *(SEC-02)* | Không có token vẫn gọi thành công (do API không chặn) | **FAIL** | Không kiểm tra xác thực | `BUG-FR15-06` |
| **TC-FR15-BV-001** | BVA | `name` length = 1 ($min$) | Tạo sản phẩm thành công *(FR-15)* | Tạo thành công | **PASS** | HTTP 200 | - |
| **TC-FR15-BV-002** | BVA | `name` length = 2 ($min + 1$) | Tạo sản phẩm thành công *(FR-15)* | Tạo thành công | **PASS** | HTTP 200 | - |
| **TC-FR15-BV-003** | BVA | `name` length = 254 ($max - 1$) | Tạo sản phẩm thành công *(FR-15)* | Tạo thành công | **PASS** | HTTP 200 | - |
| **TC-FR15-BV-004** | BVA | `name` length = 255 ($max$) | Tạo sản phẩm thành công *(FR-15)* | Tạo thành công | **PASS** | HTTP 200 | - |
| **TC-FR15-BV-005** | BVA | `name` length = 256 ($max + 1$) | **Từ chối**, báo lỗi tên tối đa 255 ký tự *(FR-15)* | Hệ thống **chấp nhận tên dài 256 ký tự** và lưu thành công với HTTP 200! Không kiểm tra độ dài tối đa | **FAIL** | CSDL SQLite cột `name TEXT` nhận chuỗi 256 ký tự mà không báo lỗi | `BUG-FR15-07` |
| **TC-FR15-BV-006** | BVA | `price = 1` ($min$ nguyên dương) | Tạo sản phẩm thành công *(FR-15)* | Tạo thành công | **PASS** | HTTP 200 | - |
| **TC-FR15-BV-007** | BVA | `price = 0.01` ($min$ số thực dương) | Tạo sản phẩm thành công *(FR-15)* | Tạo thành công | **PASS** | HTTP 200 | - |
| **TC-FR15-BV-008** | BVA | `price = -0.01` (Sát dưới 0) | Từ chối, báo lỗi giá phải là số dương *(FR-15)* | Hệ thống chấp nhận giá âm | **FAIL** | Bị ảnh hưởng bởi lỗi thiếu kiểm tra giá dương | `BUG-FR15-02` |
| **TC-FR15-BV-009** | BVA | `price` rất lớn (`10^12`) | Xử lý an toàn không sập hệ thống | Xử lý an toàn, lưu thành công | **PASS** | HTTP 200 | - |
| **TC-FR15-BV-010** | BVA | `product_id = 0` khi Sửa/Xóa | Báo lỗi không tìm thấy sản phẩm | Báo lỗi hoặc không thay đổi gì | **PASS** | Không làm hỏng dữ liệu | - |

---

## 3. Thống kê Kết quả Thực thi (Execution Summary)

- **Tổng số ca kiểm thử đã thiết kế (Designed):** **26**
- **Tổng số ca kiểm thử đã thực thi (Executed):** **26**
- **Số ca kiểm thử ĐẠT (Passed):** **11** (42.3%)
- **Số ca kiểm thử THẤT BẠI (Failed):** **15** (57.7%)
- **Số ca kiểm thử chưa chạy (Not executed):** **0**
- **Tổng số lỗi phát hiện (Bugs identified):** **7 lỗi** (`BUG-FR15-01` đến `BUG-FR15-07`)

---

## 4. Danh sách các Lỗi Phát hiện (Bug List)
1. **`BUG-FR15-01`**: Backend không kiểm tra bắt buộc tên sản phẩm, cho phép tạo sản phẩm có tên rỗng hoặc toàn khoảng trắng (Vi phạm FR-15).
2. **`BUG-FR15-02`**: Hệ thống cho phép nhập giá tiền bằng 0 hoặc số âm (Vi phạm FR-15: *"Giá: bắt buộc, phải là số dương (> 0)"*).
3. **`BUG-FR15-03`**: Ô nhập giá tiền trên form Web Admin thiếu thuộc tính `required` (Vi phạm FR-15 & FR-22).
4. **`BUG-FR15-04`**: Cho phép tạo sản phẩm với `category_id` không tồn tại hoặc để trống (Vi phạm FR-15: *"Danh mục: bắt buộc, phải chọn từ danh sách có sẵn"*).
5. **`BUG-FR15-05`** *(Lỗi nghiêm trọng - Seeded bug)*: Khi Sửa một sản phẩm trên Web Admin, toàn bộ danh sách sản phẩm khác đều bị đổi tên theo sản phẩm vừa sửa (Vi phạm trực tiếp FR-15: *"Khi Sửa một sản phẩm, chỉ sản phẩm đó bị thay đổi — các sản phẩm khác giữ nguyên"*).
6. **`BUG-FR15-06`**: Toàn bộ các API Thêm/Sửa/Xóa sản phẩm (`POST/PUT/DELETE /api/products`) hoàn toàn mở (public), không yêu cầu Token hay quyền Admin (Vi phạm nghiêm trọng SEC-02 & SEC-03).
7. **`BUG-FR15-07`**: Hệ thống cho phép nhập tên sản phẩm dài vượt quá 255 ký tự (Vi phạm FR-15: *"Tên sản phẩm: tối đa 255 ký tự"*).
