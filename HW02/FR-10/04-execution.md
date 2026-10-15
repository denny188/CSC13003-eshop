# Báo cáo Thực thi Kiểm thử (Test Execution Log): FR-10 (Trạng thái đơn hàng)

## 1. Môi trường Thực thi (Execution Environment)
- **Hệ thống thử nghiệm (SUT):** EShop Backend REST API & SQLite Database
- **Base URL:** `http://localhost:3000`
- **Tài khoản kiểm thử:**
  - Admin: `admin@eshop.com` / `Admin123!` (Role: `admin`)
  - User: `test@eshop.com` / `Test1234!` (Role: `user`)
- **Ngày thực thi:** 2026-10-15
- **Công cụ:** Node.js Fetch HTTP Client & SQLite3 direct validation.

---

## 2. Bảng Kết quả Thực thi Chi tiết (Execution Results Table)

| Mã Test Case | Loại kiểm thử | Chuyển đổi / Hành vi kiểm tra | Kết quả mong đợi (Expected Result) | Kết quả thực tế (Actual Result) | Đánh giá (Pass/Fail) | Minh chứng / Bằng chứng (Evidence) | Mã Bug liên kết |
| :--- | :--- | :--- | :--- | :--- | :---: | :--- | :---: |
| **TC-FR10-DT-001** | Domain | `pending` $\rightarrow$ `confirmed` (Admin) | Cập nhật thành công, HTTP 200 *(FR-10)* | Cập nhật thành công sang `confirmed`, trả về HTTP 200 | **PASS** | HTTP 200 `{"message":"Order status updated"}` | - |
| **TC-FR10-DT-002** | Domain | `confirmed` $\rightarrow$ `shipping` (Admin) | Cập nhật thành công, HTTP 200 *(FR-10)* | Cập nhật thành công sang `shipping`, trả về HTTP 200 | **PASS** | HTTP 200 `{"message":"Order status updated"}` | - |
| **TC-FR10-DT-003** | Domain | `shipping` $\rightarrow$ `delivered` (Admin) | Cập nhật thành công, HTTP 200 *(FR-10)* | Cập nhật thành công sang `delivered`, trả về HTTP 200 | **PASS** | HTTP 200 `{"message":"Order status updated"}` | - |
| **TC-FR10-DT-004** | Domain | `pending` $\rightarrow$ `canceled` (User cancel) | Hủy thành công, HTTP 200 *(FR-10, api_spec 4.6)* | Hủy thành công sang `canceled`, trả về HTTP 200 | **PASS** | HTTP 200 `{"message":"Order canceled successfully"}` | - |
| **TC-FR10-DT-005** | Domain | `confirmed` $\rightarrow$ `canceled` (User cancel) | Hủy thành công, HTTP 200 *(FR-10, FR-20)* | Hủy thành công sang `canceled`, trả về HTTP 200 | **PASS** | HTTP 200 `{"message":"Order canceled successfully"}` | - |
| **TC-FR10-DT-006** | Domain | `pending` $\rightarrow$ `canceled` (Admin cancel) | Admin hủy thành công, HTTP 200 *(FR-10)* | Cập nhật thành công sang `canceled`, trả về HTTP 200 | **PASS** | HTTP 200 `{"message":"Order status updated"}` | - |
| **TC-FR10-DT-007** | Domain | `confirmed` $\rightarrow$ `canceled` (Admin cancel) | Admin hủy thành công, HTTP 200 *(FR-10)* | Cập nhật thành công sang `canceled`, trả về HTTP 200 | **PASS** | HTTP 200 `{"message":"Order status updated"}` | - |
| **TC-FR10-DT-008** | Domain | `shipping` $\rightarrow$ `canceled` (User cancel) | **Từ chối hủy**, HTTP 400 *(FR-10: Khi đơn ở shipping, User không được tự hủy)* | Hệ thống **cho phép User tự hủy đơn hàng đang shipping** và chuyển sang `canceled` với HTTP 200! | **FAIL** | Endpoint `PUT /api/orders/:id/cancel` cho phép hủy khi `shipping`. Code chỉ chặn `delivered` hoặc `canceled` (server.js:329) | `BUG-FR10-01` |
| **TC-FR10-DT-009** | Domain | `delivered` $\rightarrow$ Any | Từ chối, HTTP 400 *(FR-10: delivered là trạng thái kết thúc)* | Từ chối hủy, trả về HTTP 400 `Cannot cancel this order.` | **PASS** | HTTP 400 `{"error":"Cannot cancel this order."}` | - |
| **TC-FR10-DT-010** | Domain | `canceled` $\rightarrow$ `delivered` (Admin chuyển) | **Từ chối**, HTTP 400 *(FR-10: canceled là trạng thái kết thúc, không được chuyển sang trạng thái khác)* | Hệ thống **cho phép Admin chuyển từ canceled sang delivered** với HTTP 200! | **FAIL** | Code `server.js:550`: `if (currentStatus === "canceled" && status === "delivered") isValidTransition = true;` Vi phạm trực tiếp đặc tả trạng thái kết thúc | `BUG-FR10-02` |
| **TC-FR10-DT-011** | Domain | `canceled` $\rightarrow$ `pending` | Từ chối, HTTP 400 *(FR-10: canceled là trạng thái kết thúc)* | Báo lỗi `Invalid state transition from canceled to pending`, HTTP 400 | **PASS** | HTTP 400 | - |
| **TC-FR10-DT-012** | Domain | `pending` $\rightarrow$ `shipping` (Nhảy cóc) | Từ chối chuyển đổi không hợp lệ, HTTP 400 *(FR-10)* | Báo lỗi `Invalid state transition`, HTTP 400 | **PASS** | HTTP 400 | - |
| **TC-FR10-DT-013** | Domain | `pending` $\rightarrow$ `delivered` (Nhảy cóc) | Từ chối chuyển đổi không hợp lệ, HTTP 400 *(FR-10)* | Báo lỗi `Invalid state transition`, HTTP 400 | **PASS** | HTTP 400 | - |
| **TC-FR10-DT-014** | Domain | `confirmed` $\rightarrow$ `delivered` (Nhảy cóc) | Từ chối chuyển đổi không hợp lệ, HTTP 400 *(FR-10)* | Báo lỗi `Invalid state transition`, HTTP 400 | **PASS** | HTTP 400 | - |
| **TC-FR10-DT-015** | Domain | `shipping` $\rightarrow$ `confirmed` (Đảo ngược) | Từ chối chuyển đổi ngược, HTTP 400 *(FR-10)* | Báo lỗi `Invalid state transition`, HTTP 400 | **PASS** | HTTP 400 | - |
| **TC-FR10-DT-016** | Domain | `confirmed` $\rightarrow$ `pending` (Đảo ngược) | Từ chối chuyển đổi ngược, HTTP 400 *(FR-10)* | Báo lỗi `Invalid state transition`, HTTP 400 | **PASS** | HTTP 400 | - |
| **TC-FR10-DT-017** | Domain | Trạng thái lạ (`"refunded"`) | Từ chối, HTTP 400 *(FR-10)* | Báo lỗi `Invalid state transition`, HTTP 400 | **PASS** | HTTP 400 | - |
| **TC-FR10-DT-018** | Domain | Trạng thái để trống (`""`) | Từ chối, HTTP 400 *(FR-10)* | Báo lỗi `Invalid state transition`, HTTP 400 | **PASS** | HTTP 400 | - |
| **TC-FR10-DT-019** | Domain | User A hủy đơn của User B (IDOR) | Từ chối truy cập trái phép, HTTP 403/404 *(SEC-02)* | Trả về HTTP 404 `Order not found` (do query có `user_id = ?`), ngăn chặn thành công IDOR | **PASS** | HTTP 404 `{"error":"Order not found"}` | - |
| **TC-FR10-DT-020** | Domain | Không có JWT Token | Từ chối Unauthorized, HTTP 401 *(SEC-02)* | Trả về HTTP 401 `{"error":"Unauthorized"}` | **PASS** | HTTP 401 | - |
| **TC-FR10-DT-021** | Domain | User thường gọi API Admin | **Từ chối Forbidden**, HTTP 403 *(SEC-03: API Admin phải kiểm tra role = 'admin')* | API Admin `PUT /api/admin/orders/:id/status` **chấp nhận token của User thường** và cập nhật trạng thái thành công với HTTP 200! | **FAIL** | Endpoint Admin chỉ dùng middleware `authenticateToken` chung, hoàn toàn không kiểm tra `req.user.role === 'admin'`. Vi phạm nghiêm trọng SEC-03 | `BUG-FR10-03` |
| **TC-FR10-BV-001** | BVA | Admin hủy đơn ở `shipping` | Phản hồi lỗi nhất quán hoặc từ chối theo sơ đồ FR-10 | Báo lỗi `Invalid state transition from shipping to canceled`, HTTP 400 | **PASS** (Tuân thủ sơ đồ FSM) | HTTP 400 | - |
| **TC-FR10-BV-002** | BVA | Self-loop `pending` $\rightarrow$ `pending` | Từ chối chuyển đổi trùng lặp, HTTP 400 *(FR-10)* | Báo lỗi `Invalid state transition`, HTTP 400 | **PASS** | HTTP 400 | - |
| **TC-FR10-BV-003** | BVA | Self-loop `confirmed` $\rightarrow$ `confirmed` | Từ chối chuyển đổi trùng lặp, HTTP 400 *(FR-10)* | Báo lỗi `Invalid state transition`, HTTP 400 | **PASS** | HTTP 400 | - |
| **TC-FR10-BV-004** | BVA | Self-loop `shipping` $\rightarrow$ `shipping` | Từ chối chuyển đổi trùng lặp, HTTP 400 *(FR-10)* | Báo lỗi `Invalid state transition`, HTTP 400 | **PASS** | HTTP 400 | - |
| **TC-FR10-BV-005** | BVA | Self-loop `delivered` $\rightarrow$ `delivered` | Từ chối vì đã ở trạng thái kết thúc, HTTP 400 *(FR-10)* | Báo lỗi `Invalid state transition`, HTTP 400 | **PASS** | HTTP 400 | - |
| **TC-FR10-BV-006** | BVA | Self-loop `canceled` $\rightarrow$ `canceled` | Từ chối vì đã ở trạng thái kết thúc, HTTP 400 *(FR-10)* | Báo lỗi `Invalid state transition`, HTTP 400 | **PASS** | HTTP 400 | - |
| **TC-FR10-BV-007** | BVA | Biên `order_id` = 0 ($min - 1$) | Báo lỗi 404 không tìm thấy đơn hàng | Trả về HTTP 404 `{"error":"Order not found"}` | **PASS** | HTTP 404 | - |
| **TC-FR10-BV-008** | BVA | Biên `order_id` âm (`-1`) | Báo lỗi 404 không tìm thấy đơn hàng | Trả về HTTP 404 `{"error":"Order not found"}` | **PASS** | HTTP 404 | - |
| **TC-FR10-BV-009** | BVA | Biên `order_id` là chuỗi (`"abc"`) | Báo lỗi 404 không tìm thấy đơn hàng | Trả về HTTP 404 `{"error":"Order not found"}` | **PASS** | HTTP 404 | - |
| **TC-FR10-BV-010** | BVA | Biên `order_id` không tồn tại (`999999`) | Báo lỗi 404 không tìm thấy đơn hàng | Trả về HTTP 404 `{"error":"Order not found"}` | **PASS** | HTTP 404 | - |

---

## 3. Thống kê Kết quả Thực thi (Execution Summary)

- **Tổng số ca kiểm thử đã thiết kế (Designed):** **31**
- **Tổng số ca kiểm thử đã thực thi (Executed):** **31**
- **Số ca kiểm thử ĐẠT (Passed):** **28** (90.3%)
- **Số ca kiểm thử THẤT BẠI (Failed):** **3** (9.7%)
- **Số ca kiểm thử chưa chạy (Not executed):** **0**
- **Tổng số lỗi phát hiện (Bugs identified):** **3 lỗi nghiêm trọng** (`BUG-FR10-01`, `BUG-FR10-02`, `BUG-FR10-03`)

---

## 4. Danh sách các Lỗi Phát hiện (Bug List)
1. **`BUG-FR10-01`**: Khách hàng (User) vẫn có thể tự hủy đơn hàng khi đơn hàng đang ở trạng thái `shipping` (Vi phạm trực tiếp FR-10: *"Khi đơn hàng đã ở trạng thái shipping, User không được phép tự hủy"*).
2. **`BUG-FR10-02`**: Hệ thống cho phép chuyển đổi trạng thái từ `canceled` sang `delivered` (Vi phạm quy định trạng thái kết thúc trong FR-10: *"delivered và canceled là trạng thái kết thúc — không được phép chuyển sang bất kỳ trạng thái nào khác"*).
3. **`BUG-FR10-03`**: API Admin `PUT /api/admin/orders/:id/status` không kiểm tra quyền Admin (`role = 'admin'`), cho phép người dùng bình thường tự ý đổi trạng thái đơn hàng bất kỳ (Lỗ hổng Broken Access Control - Vi phạm nghiêm trọng SEC-03).
