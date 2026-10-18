# [BUG-FR10-02] Hệ thống cho phép chuyển đổi trạng thái từ "canceled" sang "delivered"

- **Feature (FR):** `FR-10: Trạng thái Đơn hàng (Order State Machine)` & `FR-18: Quản lý Đơn hàng (Admin)`
- **Môi trường (Environment):** Backend REST API (`PUT /api/admin/orders/:id/status`)
- **Mức độ nghiêm trọng (Severity):** **High** (Vi phạm tính toàn vẹn của máy trạng thái - Trạng thái kết thúc bị phá vỡ)
- **Mã Test Case liên quan:** `TC-FR10-DT-010`

---

### 1. Mô tả lỗi (Description)
Theo đặc tả nghiệp vụ FR-10, `canceled` (đã hủy) và `delivered` (đã giao) là hai trạng thái kết thúc (Terminal States) của vòng đời đơn hàng, tuyệt đối không được phép chuyển sang bất kỳ trạng thái nào khác. Tuy nhiên, logic xử lý chuyển đổi trạng thái của Admin lại cố tình cài đặt một ngoại lệ cho phép chuyển đổi từ `canceled` sang `delivered`. Kết quả là Admin có thể "hồi sinh" một đơn hàng đã bị hủy và đánh dấu thành đã giao thành công.

### 2. Các bước tái hiện (Steps to Reproduce)
1. Đăng nhập với tài khoản Admin (`admin@eshop.com`) để lấy token xác thực.
2. Xác định một đơn hàng đang ở trạng thái `canceled` (ví dụ `order_id = 5`).
3. Gửi request cập nhật trạng thái đơn hàng sang `delivered`:
   ```bash
   curl -X PUT http://localhost:3000/api/admin/orders/5/status \
     -H "Authorization: Bearer <ADMIN_JWT_TOKEN>" \
     -H "Content-Type: application/json" \
     -d '{"status": "delivered"}'
   ```
4. Quan sát mã phản hồi trả về từ server.

### 3. Kết quả mong đợi (Expected Result)
- Theo đặc tả **FR-10**: *"delivered và canceled là trạng thái kết thúc — không được phép chuyển sang bất kỳ trạng thái nào khác."*
- Hệ thống phải từ chối chuyển đổi và trả về HTTP 400 Bad Request: `{"error": "Invalid state transition from canceled to delivered"}`.

### 4. Kết quả thực tế (Actual Result)
- Server trả về HTTP 200 OK:
  ```json
  {"message": "Order status updated"}
  ```
- Trạng thái đơn hàng trong CSDL bị cập nhật từ `canceled` thành `delivered`.
- Đoạn mã lỗi tại `backend/server.js:550-551`:
  ```javascript
  if (currentStatus === "canceled" && status === "delivered")
    isValidTransition = true;
  ```

### 5. Đề xuất khắc phục (Recommendation)
Xóa bỏ dòng điều kiện sai lệch này trong `backend/server.js:550-551`. Khi `currentStatus` là `canceled` hoặc `delivered`, không được gán `isValidTransition = true` cho bất kỳ `status` nào.

---
- **GitHub Issue Link:** [Issue #9](https://github.com/denny188/CSC13003-eshop/issues/9)
