# Báo cáo Kiểm thử Miền giá trị (Domain Testing): FR-10 (Trạng thái đơn hàng)

## 1. Phương pháp & Nguyên tắc Áp dụng
1. Áp dụng kỹ thuật **Kiểm thử Miền giá trị (Domain Testing)** kết hợp mô hình hóa **Máy trạng thái hữu hạn (Finite State Machine - FSM)** cho các biến rời rạc/liệt kê (Enumerated domain):
   - Phân hoạch miền giá trị của `current_status` và `target_status` thành các tập trạng thái hợp lệ `{pending, confirmed, shipping, delivered, canceled}` và các giá trị không hợp lệ.
   - Phân hoạch biến quyền tác nhân (`actor_role`): `admin`, `user` (chính chủ), `user` (khác chủ - IDOR), `anonymous` (không có token).
2. Xây dựng các ca kiểm thử theo nguyên tắc:
   - **Tất cả các chuyển đổi trạng thái hợp lệ (Valid Transitions):** Kiểm tra đường đi chuẩn (Happy path) của đơn hàng.
   - **Chuyển đổi trạng thái không hợp lệ (Invalid Transitions):** Áp dụng nguyên tắc single-fault, thử từng bước nhảy trạng thái sai quy tắc, nhảy vượt cấp, hoặc chuyển đổi từ các trạng thái kết thúc (`delivered`, `canceled`).
   - **Kiểm soát quyền truy cập & Phân quyền (Role-based / Authorization checks):** Kiểm tra khả năng hủy đơn của User khi đơn ở trạng thái cấm (`shipping`), và cố tình hủy đơn hàng của người khác.

---

## 2. Bảng Phân hoạch Tương đương & Chọn Giá trị Đại diện (Domain Table)

### 2.1 Biến trạng thái đơn hàng (`current_status` & `target_status`)
| Mã lớp | Mô tả lớp tương đương | Hợp lệ / Không hợp lệ | Giá trị đại diện | Lý giải chọn giá trị (Rationale) |
| :--- | :--- | :---: | :--- | :--- |
| **EC-ST-V1** | Trạng thái Chờ xác nhận | Hợp lệ | `"pending"` | Trạng thái khởi tạo mặc định của mọi đơn hàng mới |
| **EC-ST-V2** | Trạng thái Đã xác nhận | Hợp lệ | `"confirmed"` | Đơn hàng đã được Admin duyệt |
| **EC-ST-V3** | Trạng thái Đang giao hàng | Hợp lệ | `"shipping"` | Đơn hàng đang trên đường vận chuyển |
| **EC-ST-V4** | Trạng thái Đã giao thành công (Kết thúc) | Hợp lệ | `"delivered"` | Trạng thái kết thúc thành công, không chuyển tiếp |
| **EC-ST-V5** | Trạng thái Đã hủy (Kết thúc) | Hợp lệ | `"canceled"` | Trạng thái kết thúc do bị hủy bởi User hoặc Admin |
| **EC-ST-I1** | Trạng thái không tồn tại trong hệ thống | Không hợp lệ | `"refunded"` | Giá trị chuỗi lạ nằm ngoài enum 5 trạng thái |
| **EC-ST-I2** | Trạng thái để trống hoặc null | Không hợp lệ | `""` | Kiểm tra bắt buộc truyền trường `status` |
| **EC-ST-I3** | Sai kiểu dữ liệu (Số học / Boolean) | Không hợp lệ | `123` | Kiểm tra kiểm soát kiểu dữ liệu của API |

---

### 2.2 Biến Tác nhân thực thi (`actor_role` & `order_owner`)
| Mã lớp | Mô tả lớp tương đương | Hợp lệ / Không hợp lệ | Giá trị đại diện | Lý giải chọn giá trị (Rationale) |
| :--- | :--- | :---: | :--- | :--- |
| **EC-AC-V1** | Quản trị viên hệ thống có JWT Token hợp lệ | Hợp lệ | `admin@eshop.com` (role: `admin`) | Toàn quyền chuyển đổi theo đúng state machine |
| **EC-AC-V2** | Khách hàng là chủ sở hữu của đơn hàng | Hợp lệ | `test@eshop.com` (user sở hữu đơn) | Có quyền hủy đơn khi ở `pending` hoặc `confirmed` |
| **EC-AC-I1** | Khách hàng cố tình hủy đơn hàng của người khác (IDOR) | Không hợp lệ | User A gọi cancel `order_id` của User B | Kiểm tra kiểm soát quyền sở hữu (Object-level authorization) |
| **EC-AC-I2** | Người dùng chưa đăng nhập (Không truyền JWT Token) | Không hợp lệ | Không gửi header `Authorization` | Kiểm tra xác thực SEC-02 |
| **EC-AC-I3** | Khách hàng thường gọi API cập nhật trạng thái của Admin | Không hợp lệ | User token gọi `PUT /api/admin/orders/:id/status` | Kiểm tra phân quyền SEC-03 (`role = 'admin'`) |

---

## 3. Danh sách Ca Kiểm thử Miền giá trị Chi tiết (Domain Test Cases)

| Mã Test Case | Lớp tương đương / Chuyển đổi kiểm tra | Điều kiện tiên quyết (Preconditions) | Các bước thực hiện (Steps) | Dữ liệu kiểm thử (Test Data) | Kết quả mong đợi (Expected Result) | Độ ưu tiên | Trạng thái |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: | :---: |
| **TC-FR10-DT-001** | **EC-ST-V1 $\rightarrow$ EC-ST-V2** (Admin xác nhận) | Đơn hàng đang ở `pending` | 1. Admin đăng nhập lấy token<br>2. Gửi request cập nhật status | `id`: `<order_id>`, `status`: `"confirmed"`, Actor: Admin | Cập nhật thành công sang `confirmed`, HTTP 200 *(FR-10)* | High | Not executed |
| **TC-FR10-DT-002** | **EC-ST-V2 $\rightarrow$ EC-ST-V3** (Admin giao hàng) | Đơn hàng đang ở `confirmed` | 1. Gửi request cập nhật status | `id`: `<order_id>`, `status`: `"shipping"`, Actor: Admin | Cập nhật thành công sang `shipping`, HTTP 200 *(FR-10)* | High | Not executed |
| **TC-FR10-DT-003** | **EC-ST-V3 $\rightarrow$ EC-ST-V4** (Admin hoàn tất giao) | Đơn hàng đang ở `shipping` | 1. Gửi request cập nhật status | `id`: `<order_id>`, `status`: `"delivered"`, Actor: Admin | Cập nhật thành công sang `delivered` (Final), HTTP 200 *(FR-10)* | High | Not executed |
| **TC-FR10-DT-004** | **EC-ST-V1 $\rightarrow$ EC-ST-V5** (User hủy đơn pending) | Đơn hàng đang ở `pending` của User | 1. User gọi API hủy đơn hàng của mình | Endpoint: `PUT /api/orders/:id/cancel`, Actor: User chủ đơn | Hủy thành công sang `canceled`, HTTP 200 *(FR-10, api_spec 4.6)* | High | Not executed |
| **TC-FR10-DT-005** | **EC-ST-V2 $\rightarrow$ EC-ST-V5** (User hủy đơn confirmed) | Đơn hàng đang ở `confirmed` của User | 1. User gọi API hủy đơn hàng của mình | Endpoint: `PUT /api/orders/:id/cancel`, Actor: User chủ đơn | Hủy thành công sang `canceled`, HTTP 200 *(FR-10, FR-20)* | High | Not executed |
| **TC-FR10-DT-006** | **EC-ST-V1 $\rightarrow$ EC-ST-V5** (Admin hủy đơn pending) | Đơn hàng đang ở `pending` | 1. Admin gửi request cập nhật status | `id`: `<order_id>`, `status`: `"canceled"`, Actor: Admin | Hủy thành công sang `canceled`, HTTP 200 *(FR-10)* | Medium | Not executed |
| **TC-FR10-DT-007** | **EC-ST-V2 $\rightarrow$ EC-ST-V5** (Admin hủy đơn confirmed) | Đơn hàng đang ở `confirmed` | 1. Admin gửi request cập nhật status | `id`: `<order_id>`, `status`: `"canceled"`, Actor: Admin | Hủy thành công sang `canceled`, HTTP 200 *(FR-10)* | Medium | Not executed |
| **TC-FR10-DT-008** | **EC-ST-V3 $\rightarrow$ EC-ST-V5** (User cố hủy khi shipping) | Đơn hàng đang ở `shipping` | 1. User gọi API hủy đơn khi đang giao hàng | Endpoint: `PUT /api/orders/:id/cancel`, Actor: User | Từ chối hủy, trả về HTTP 400 Bad Request *(FR-10: User không được tự hủy khi shipping)* | High | Not executed |
| **TC-FR10-DT-009** | **EC-ST-V4 $\rightarrow$ Any** (Hủy đơn delivered) | Đơn hàng đang ở `delivered` | 1. User hoặc Admin gọi lệnh hủy hoặc đổi trạng thái | Endpoint: cancel hoặc update `status` | Từ chối, trả về HTTP 400 *(FR-10: delivered là trạng thái kết thúc)* | High | Not executed |
| **TC-FR10-DT-010** | **EC-ST-V5 $\rightarrow$ EC-ST-V4** (Chuyển canceled sang delivered) | Đơn hàng đang ở `canceled` | 1. Admin gửi cập nhật `status: "delivered"` | `id`: `<order_id>`, `status`: `"delivered"` | Từ chối, trả về HTTP 400 *(FR-10: canceled là trạng thái kết thúc)* | High | Not executed |
| **TC-FR10-DT-011** | **EC-ST-V5 $\rightarrow$ Any** (Phục hồi đơn đã hủy) | Đơn hàng đang ở `canceled` | 1. Admin gửi cập nhật `status: "pending"` | `id`: `<order_id>`, `status`: `"pending"` | Từ chối, trả về HTTP 400 *(FR-10: canceled là trạng thái kết thúc)* | High | Not executed |
| **TC-FR10-DT-012** | **EC-ST-V1 $\rightarrow$ EC-ST-V3** (Nhảy cóc pending lên shipping) | Đơn hàng đang ở `pending` | 1. Admin cập nhật trực tiếp lên `shipping` | `id`: `<order_id>`, `status`: `"shipping"` | Từ chối chuyển đổi không hợp lệ, HTTP 400 *(FR-10)* | High | Not executed |
| **TC-FR10-DT-013** | **EC-ST-V1 $\rightarrow$ EC-ST-V4** (Nhảy cóc pending lên delivered) | Đơn hàng đang ở `pending` | 1. Admin cập nhật trực tiếp lên `delivered` | `id`: `<order_id>`, `status`: `"delivered"` | Từ chối chuyển đổi không hợp lệ, HTTP 400 *(FR-10)* | High | Not executed |
| **TC-FR10-DT-014** | **EC-ST-V2 $\rightarrow$ EC-ST-V4** (Nhảy cóc confirmed lên delivered) | Đơn hàng đang ở `confirmed` | 1. Admin cập nhật trực tiếp lên `delivered` | `id`: `<order_id>`, `status`: `"delivered"` | Từ chối chuyển đổi không hợp lệ, HTTP 400 *(FR-10)* | High | Not executed |
| **TC-FR10-DT-015** | **EC-ST-V3 $\rightarrow$ EC-ST-V2** (Đảo ngược shipping về confirmed) | Đơn hàng đang ở `shipping` | 1. Admin gửi cập nhật ngược về `confirmed` | `id`: `<order_id>`, `status`: `"confirmed"` | Từ chối chuyển đổi ngược, HTTP 400 *(FR-10)* | Medium | Not executed |
| **TC-FR10-DT-016** | **EC-ST-V2 $\rightarrow$ EC-ST-V1** (Đảo ngược confirmed về pending) | Đơn hàng đang ở `confirmed` | 1. Admin gửi cập nhật ngược về `pending` | `id`: `<order_id>`, `status`: `"pending"` | Từ chối chuyển đổi ngược, HTTP 400 *(FR-10)* | Medium | Not executed |
| **TC-FR10-DT-017** | **EC-ST-I1** (Trạng thái lạ không tồn tại) | Đơn hàng bất kỳ | 1. Admin gửi `status` lạ | `status`: `"refunded"` | Từ chối, trả về HTTP 400 *(FR-10)* | High | Not executed |
| **TC-FR10-DT-018** | **EC-ST-I2** (Trạng thái để trống) | Đơn hàng bất kỳ | 1. Admin gửi `status: ""` | `status`: `""` | Từ chối, trả về HTTP 400 *(FR-10)* | Medium | Not executed |
| **TC-FR10-DT-019** | **EC-AC-I1** (IDOR: User A hủy đơn User B) | Đơn hàng thuộc User B | 1. User A đăng nhập lấy token<br>2. Gửi request hủy đơn của User B | `PUT /api/orders/<id_B>/cancel` với Token A | Từ chối truy cập trái phép, HTTP 403 hoặc 404 *(SEC-02)* | High | Not executed |
| **TC-FR10-DT-020** | **EC-AC-I2** (Không có Token) | Đơn hàng bất kỳ | 1. Gửi request hủy đơn không kèm Header Authorization | Không có header Auth | Từ chối Unauthorized, HTTP 401 *(SEC-02)* | High | Not executed |
| **TC-FR10-DT-021** | **EC-AC-I3** (User thường gọi API Admin) | Đơn hàng bất kỳ | 1. User gửi request cập nhật status tới API Admin | `PUT /api/admin/orders/:id/status` với Token User | Từ chối Forbidden (yêu cầu role admin), HTTP 403 *(SEC-03)* | High | Not executed |

---

## 4. Ma trận Độ phủ Lớp tương đương (Coverage Matrix)

| Lớp tương đương | Mã Test Case bao phủ |
| :--- | :--- |
| **EC-ST-V1** | TC-FR10-DT-001, TC-FR10-DT-004, TC-FR10-DT-006, TC-FR10-DT-012, TC-FR10-DT-013 |
| **EC-ST-V2** | TC-FR10-DT-001, TC-FR10-DT-002, TC-FR10-DT-005, TC-FR10-DT-007, TC-FR10-DT-014, TC-FR10-DT-016 |
| **EC-ST-V3** | TC-FR10-DT-002, TC-FR10-DT-003, TC-FR10-DT-008, TC-FR10-DT-015 |
| **EC-ST-V4** | TC-FR10-DT-003, TC-FR10-DT-009, TC-FR10-DT-010 |
| **EC-ST-V5** | TC-FR10-DT-004, TC-FR10-DT-005, TC-FR10-DT-006, TC-FR10-DT-007, TC-FR10-DT-010, TC-FR10-DT-011 |
| **EC-ST-I1** | TC-FR10-DT-017 |
| **EC-ST-I2** | TC-FR10-DT-018 |
| **EC-AC-V1** | TC-FR10-DT-001, TC-FR10-DT-002, TC-FR10-DT-003, TC-FR10-DT-006, TC-FR10-DT-007 |
| **EC-AC-V2** | TC-FR10-DT-004, TC-FR10-DT-005 |
| **EC-AC-I1** | TC-FR10-DT-019 |
| **EC-AC-I2** | TC-FR10-DT-020 |
| **EC-AC-I3** | TC-FR10-DT-021 |

**Tổng số test case thiết kế trong Step 2 (FR-10):** **21 test cases** (Bao phủ toàn diện 100% các lớp tương đương trạng thái và vai trò tác nhân).
