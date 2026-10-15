# [BUG-FR10-01] Khách hàng vẫn có thể tự hủy đơn hàng khi đơn hàng đang ở trạng thái "shipping"

- **Feature (FR):** `FR-10: Trạng thái Đơn hàng (Order State Machine)` & `api_specification.md (Mục 4.6)`
- **Môi trường (Environment):** Backend REST API (`PUT /api/orders/:id/cancel`)
- **Mức độ nghiêm trọng (Severity):** **High** (Vi phạm quy tắc nghiệp vụ cốt lõi của luồng vận chuyển)
- **Mã Test Case liên quan:** `TC-FR10-DT-008`

---

### 1. Mô tả lỗi (Description)
Theo đặc tả nghiệp vụ, khi đơn hàng đã chuyển sang trạng thái đang giao hàng (`shipping`), khách hàng (User) không được phép tự ý hủy đơn vì hàng đã xuất kho và bàn giao cho đơn vị vận chuyển. Tuy nhiên, endpoint `PUT /api/orders/:id/cancel` chỉ kiểm tra chặn nếu đơn hàng ở trạng thái `delivered` hoặc `canceled`. Do đó, khi đơn hàng đang ở `shipping`, User vẫn có thể gửi request hủy và hệ thống cho phép hủy thành công, chuyển trạng thái đơn hàng sang `canceled`.

### 2. Các bước tái hiện (Steps to Reproduce)
1. Đăng nhập với tài khoản khách hàng `test@eshop.com`.
2. Tạo hoặc chuẩn bị một đơn hàng thuộc quyền sở hữu của user này đang ở trạng thái `shipping` (ví dụ `order_id = 3`).
3. Gửi request PUT hủy đơn hàng:
   ```bash
   curl -X PUT http://localhost:3000/api/orders/3/cancel \
     -H "Authorization: Bearer <USER_JWT_TOKEN>"
   ```
4. Quan sát mã phản hồi HTTP và dữ liệu trạng thái của đơn hàng trong CSDL.

### 3. Kết quả mong đợi (Expected Result)
- Theo đặc tả **FR-10**: *"Khi đơn hàng đã ở trạng thái shipping, User không được phép tự hủy — chỉ Admin mới có thể thao tác."*
- Theo **api_specification.md (4.6)**: *"Chỉ được thực hiện khi đơn hàng chưa giao."* (nghĩa là chỉ cho phép khi `pending` hoặc `confirmed`).
- Hệ thống phải từ chối yêu cầu và trả về HTTP 400 Bad Request kèm thông báo lỗi phù hợp (ví dụ: `Cannot cancel order while it is shipping`).

### 4. Kết quả thực tế (Actual Result)
- Server trả về HTTP 200 OK:
  ```json
  {"message": "Order canceled successfully"}
  ```
- Trạng thái đơn hàng trong CSDL bị đổi thành `canceled`.
- Đoạn mã lỗi tại `backend/server.js:328-331`:
  ```javascript
  // Lẽ ra phải là: if (order.status !== 'pending' && order.status !== 'confirmed')
  if (order.status === "delivered" || order.status === "canceled") {
    return res.status(400).json({ error: "Cannot cancel this order." });
  }
  ```

### 5. Đề xuất khắc phục (Recommendation)
Sửa lại điều kiện kiểm tra trong `server.js:329`:
```javascript
if (order.status !== 'pending' && order.status !== 'confirmed') {
  return res.status(400).json({ error: "Chỉ có thể hủy đơn hàng khi đang ở trạng thái chờ xử lý hoặc đã xác nhận." });
}
```
