# Phân tích Yêu cầu Kiểm thử: FR-10 (Trạng thái đơn hàng - Order State Machine)

## 1. Trích dẫn Yêu cầu từ Đặc tả (Spec Excerpts)

### 1.1 Đặc tả chức năng (README.md - FR-10)
- **FR-10: Trạng thái Đơn hàng (Order State Machine):**
  - Đơn hàng có **5 trạng thái**: `pending`, `confirmed`, `shipping`, `delivered`, `canceled`.
  - Sơ đồ chuyển đổi trạng thái hợp lệ:
    - `pending` $\xrightarrow{\text{Admin xác nhận}}$ `confirmed`
    - `confirmed` $\xrightarrow{\text{Admin giao hàng}}$ `shipping`
    - `shipping` $\xrightarrow{\text{Admin hoàn tất}}$ `delivered`
    - `pending` $\xrightarrow{\text{User / Admin hủy}}$ `canceled`
    - `confirmed` $\xrightarrow{\text{User / Admin hủy}}$ `canceled`
  - **Ràng buộc trạng thái kết thúc (Final States):**
    - `delivered` và `canceled` là **trạng thái kết thúc** — không được phép chuyển sang bất kỳ trạng thái nào khác.
    - Khi đơn hàng ở trạng thái `shipping`, **User không được phép tự hủy** — chỉ Admin mới có thể thao tác.
    - Mọi chuyển đổi không hợp lệ phải trả về lỗi với thông báo phù hợp.

### 1.2 Tham chiếu chéo các tính năng liên quan
- **FR-18: Quản lý Đơn hàng (Admin):** Admin có thể chuyển đổi trạng thái đơn hàng theo đúng State Machine đã định nghĩa ở FR-10.
- **FR-20: Phân hệ Mobile:** Chức năng Hủy đơn hàng tuân theo đúng State Machine ở FR-10 (chỉ được hủy khi `pending` hoặc `confirmed`).
- **SEC-02 & SEC-03:** API Admin phải kiểm tra `role = 'admin'` trong Token, không chỉ kiểm tra sự tồn tại của Token.

### 1.3 Đặc tả API (api_specification.md)
- `PUT /api/orders/:id/cancel`: User hủy đơn hàng. Header `Authorization: Bearer <user_token>`. Chỉ được thực hiện khi đơn hàng chưa giao (`pending` hoặc `confirmed`).
- `PUT /api/admin/orders/:id/status`: Admin cập nhật trạng thái đơn hàng. Header `Authorization: Bearer <admin_token>`. Body: `{"status": "<target_status>"}`.

---

## 2. Bảng phân tích Biến đầu vào (Input Variables)

| Tên biến | Vị trí (UI / API) | Kiểu dữ liệu | Miền giá trị / Ràng buộc | Bắt buộc? | Nguồn đặc tả (Spec Ref) |
| :--- | :--- | :--- | :--- | :---: | :--- |
| `order_id` (`:id`) | URL Param (`/api/orders/:id/cancel`, `/api/admin/orders/:id/status`) | Số nguyên dương (Integer) | Phải tồn tại trong CSDL bảng `orders` | Có | api_spec 4.6, 6.2 |
| `current_status` | Trạng thái hệ thống (DB state) | Chuỗi định danh (Enum) | `pending`, `confirmed`, `shipping`, `delivered`, `canceled` | Có | FR-10 |
| `target_status` | API Body `status` (`PUT /api/admin/orders/:id/status`) | Chuỗi định danh (Enum) | `pending`, `confirmed`, `shipping`, `delivered`, `canceled` hoặc giá trị bất kỳ | Có (Admin API) | FR-10, api_spec 6.2 |
| `actor_role` | Context xác thực qua JWT Token (`Authorization`) | Phân quyền (Enum) | `admin` (vai trò quản trị), `user` (người mua thông thường), `anonymous` (chưa đăng nhập) | Có | FR-10, SEC-02, SEC-03 |
| `order_owner` | Quan hệ sở hữu (Authorization context) | Logic Boolean | `true` (User là chủ sở hữu của đơn hàng), `false` (User khác cố tình hủy đơn không phải của mình) | Có (User API) | FR-10, FR-04, SEC-02 |

---

## 3. Ma trận Chuyển đổi Trạng thái Lý thuyết (5x5 State Transition Matrix)

| Trạng thái hiện tại (`current_status`) | Đích: `pending` | Đích: `confirmed` | Đích: `shipping` | Đích: `delivered` | Đích: `canceled` |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **`pending`** | Không hợp lệ (Self) | **HỢP LỆ (Admin)** | Không hợp lệ | Không hợp lệ | **HỢP LỆ (User / Admin)** |
| **`confirmed`** | Không hợp lệ | Không hợp lệ (Self) | **HỢP LỆ (Admin)** | Không hợp lệ | **HỢP LỆ (User / Admin)** |
| **`shipping`** | Không hợp lệ | Không hợp lệ | Không hợp lệ (Self) | **HỢP LỆ (Admin)** | Không hợp lệ (User cấm hủy; Admin cũng không có đường chuyển sang canceled) |
| **`delivered`** | Không hợp lệ (Final) | Không hợp lệ (Final) | Không hợp lệ (Final) | Không hợp lệ (Self) | Không hợp lệ (Final) |
| **`canceled`** | Không hợp lệ (Final) | Không hợp lệ (Final) | Không hợp lệ (Final) | Không hợp lệ (Final) | Không hợp lệ (Self) |

*Tổng cộng có 25 tổ hợp chuyển trạng thái, trong đó chỉ có 5 chuyển đổi hợp lệ và 20 chuyển đổi không hợp lệ.*

---

## 4. Danh mục Hành vi & Kết quả đầu ra cần kiểm tra (Expected Outputs & Behaviors)

1. **Chuyển đổi hợp lệ (Admin):**
   - `pending` $\rightarrow$ `confirmed`: Cập nhật status thành công, trả về HTTP 200.
   - `confirmed` $\rightarrow$ `shipping`: Cập nhật status thành công, trả về HTTP 200.
   - `shipping` $\rightarrow$ `delivered`: Cập nhật status thành công, trả về HTTP 200.
   - `pending` $\rightarrow$ `canceled`: Hủy thành công, trả về HTTP 200.
   - `confirmed` $\rightarrow$ `canceled`: Hủy thành công, trả về HTTP 200.
2. **Hủy đơn hàng của User (`PUT /api/orders/:id/cancel`):**
   - Khi đơn ở `pending` hoặc `confirmed` VÀ thuộc sở hữu của User: Cho phép hủy, cập nhật sang `canceled`, trả về HTTP 200.
   - Khi đơn ở `shipping`, `delivered`, hoặc `canceled`: Từ chối hủy, trả về HTTP 400 Bad Request kèm thông báo lỗi phù hợp.
   - Khi User cố hủy đơn hàng của User khác (IDOR): Trả về HTTP 403 Forbidden hoặc 404 Not Found, không cho phép hủy.
3. **Chuyển đổi không hợp lệ & Trạng thái kết thúc:**
   - Khi đơn hàng ở `delivered` hoặc `canceled`, mọi yêu cầu chuyển sang bất kỳ trạng thái nào khác (kể cả từ Admin) đều phải bị từ chối với HTTP 400.
   - Gửi giá trị `status` lạ/không tồn tại (vd: `"fake_status"`, `""`, số): Phải trả về HTTP 400.
4. **Phân quyền Admin:**
   - Gọi API Admin `/api/admin/orders/:id/status` bằng token của User thường hoặc không có token: Phải bị từ chối với HTTP 401/403 (SEC-02, SEC-03).

---

## 5. Các điểm mơ hồ / Khe hở đặc tả cần xác nhận (Ambiguities & Spec Gaps)

1. **Khả năng Admin hủy đơn ở trạng thái `shipping`:**
   - Trong sơ đồ FR-10: Mũi tên hủy sang `canceled` chỉ xuất phát từ `pending` và `confirmed`. Tuy nhiên câu chữ spec ghi: *"Khi đơn hàng đã ở trạng thái shipping, User không được phép tự hủy — chỉ Admin mới có thể thao tác."*
   - *Cần làm rõ:* Câu chữ này ngụ ý Admin có được phép hủy đơn khi đang ở `shipping` không, hay sơ đồ hình vẽ là tuyệt đối (chỉ `pending` và `confirmed` mới có nhánh sang `canceled`)?
2. **Khôi phục đơn hàng đã hủy (Reactivate/Rollback):**
   - Spec khẳng định `canceled` là trạng thái kết thúc, không được phép chuyển đi đâu.
3. **Hủy đơn khi chưa đăng nhập hoặc IDOR:**
   - API User cancel chỉ dựa vào JWT token hay có cần kiểm tra quyền sở hữu đối với người dùng ẩn danh không?
