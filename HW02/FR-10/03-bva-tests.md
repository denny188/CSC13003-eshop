# Báo cáo Phân tích Giá trị Biên (Boundary Value Analysis - BVA): FR-10 (Trạng thái đơn hàng)

## 1. Phương pháp & Nguyên tắc Áp dụng
1. Áp dụng kỹ thuật **Boundary Value Analysis (BVA)** cho hệ thống máy trạng thái (State Machine BVA):
   - **Biên chuyển đổi trạng thái (Transition Boundaries):** Ranh giới giữa một chuyển đổi trạng thái được phép (Valid Transition Boundary) và một chuyển đổi bị cấm ngay liền kề (Invalid Transition Boundary).
   - **Biên trạng thái kết thúc (Terminal State Boundaries):** Trạng thái `delivered` và `canceled` là biên chặn cứng của vòng đời đơn hàng, không được có bất kỳ bước chuyển tiếp nào.
   - **Biên tác nhân (Actor Boundaries):** Ranh giới quyền hạn giữa `User` và `Admin` tại trạng thái nhạy cảm `shipping` (User cấm hủy nhưng Admin có quyền thao tác).
2. Áp dụng BVA cho biến tham số định danh đơn hàng (`order_id` numeric boundaries):
   - Cận dưới: $min = 1$, $min - 1 = 0$, số âm `-1`.
   - Cận trên: ID lớn vượt quá dữ liệu (`999999`).
3. Loại bỏ trùng lặp: Các ca chuyển đổi đã kiểm tra đầy đủ ở Step 2 được ghi chú đối chiếu chéo (Cross-reference), tập trung thiết kế các ca kiểm thử biên trạng thái chuyên sâu.

---

## 2. Bảng Phân tích Các Điểm Biên (State & Parameter Boundaries)

### 2.1 Biên Chuyển trạng thái Vòng đời (Lifecycle Transition Boundaries)
- **Biên `pending`:**
  - Biên tiến hợp lệ: $\rightarrow$ `confirmed` (Valid Boundary)
  - Biên hủy hợp lệ: $\rightarrow$ `canceled` (Valid Boundary)
  - Biên vượt cấp: $\rightarrow$ `shipping` (Invalid Boundary - Nhảy cóc qua confirmed)
- **Biên `confirmed`:**
  - Biên tiến hợp lệ: $\rightarrow$ `shipping` (Valid Boundary)
  - Biên lùi: $\rightarrow$ `pending` (Invalid Boundary - Đảo ngược tiến trình)
- **Biên `shipping` (Điểm biên nhạy cảm nhất về quyền và trạng thái):**
  - Biên tiến hợp lệ: $\rightarrow$ `delivered` (Valid Boundary)
  - Biên hủy bởi User: $\rightarrow$ `canceled` (Invalid Boundary đối với User)
  - Biên hủy bởi Admin: $\rightarrow$ `canceled` (Biên kiểm tra giới hạn của Admin)
- **Biên Trạng thái kết thúc (Terminal Boundaries: `delivered` & `canceled`):**
  - Từ `delivered` $\rightarrow$ `pending`, `confirmed`, `shipping`, `canceled` (Mọi chuyển đổi đều là Invalid Boundary)
  - Từ `canceled` $\rightarrow$ `pending`, `confirmed`, `shipping`, `delivered` (Mọi chuyển đổi đều là Invalid Boundary)

### 2.2 Biên Giá trị số học của `order_id`
- $min = 1$ (ID đơn hàng nhỏ nhất có thể tồn tại)
- $min - 1 = 0$ (ID bằng 0 - Không hợp lệ)
- Số âm: `-1` (ID âm - Không hợp lệ)
- Chuỗi ký tự: `"abc"` (Không phải số nguyên)

---

## 3. Bảng Ca Kiểm thử Giá trị Biên (BVA Test Cases)

| Mã Test Case | Biến & Điểm biên kiểm tra | Loại biên (Valid / Invalid) | Dữ liệu kiểm thử (Test Data) | Các bước thực hiện & Tiền điều kiện | Kết quả mong đợi (Expected Result) | Độ ưu tiên | Trạng thái | Ghi chú & Đối chiếu Step 2 |
| :--- | :--- | :---: | :--- | :--- | :--- | :---: | :---: | :--- |
| **TC-FR10-BV-001** | Admin cố hủy đơn hàng đang ở `shipping` | Boundary Check (Admin Transition) | `status`: `"canceled"`, Actor: Admin | Đơn hàng đang ở `shipping`.<br>1. Admin gửi PUT `/api/admin/orders/:id/status` với `status: "canceled"` | Trả về thông báo lỗi hoặc xử lý nhất quán theo spec quy định *(FR-10: Cần làm rõ)* | High | Not executed | Kiểm tra biên quyền Admin tại trạng thái `shipping` |
| **TC-FR10-BV-002** | Tự chuyển đổi cùng trạng thái (`pending` $\rightarrow$ `pending`) | Self-loop Boundary | `status`: `"pending"`, Actor: Admin | Đơn hàng đang ở `pending`.<br>1. Admin gửi cập nhật `status: "pending"` | Từ chối hoặc báo không có thay đổi, không tạo chuyển đổi thừa | Medium | Not executed | Biên tự chuyển trạng thái giống nhau |
| **TC-FR10-BV-003** | Tự chuyển đổi cùng trạng thái (`confirmed` $\rightarrow$ `confirmed`) | Self-loop Boundary | `status`: `"confirmed"`, Actor: Admin | Đơn hàng đang ở `confirmed`.<br>1. Admin gửi cập nhật `status: "confirmed"` | Từ chối hoặc báo lỗi Invalid transition | Medium | Not executed | Biên tự chuyển trạng thái giống nhau |
| **TC-FR10-BV-004** | Tự chuyển đổi cùng trạng thái (`shipping` $\rightarrow$ `shipping`) | Self-loop Boundary | `status`: `"shipping"`, Actor: Admin | Đơn hàng đang ở `shipping`.<br>1. Admin gửi cập nhật `status: "shipping"` | Từ chối hoặc báo lỗi Invalid transition | Medium | Not executed | Biên tự chuyển trạng thái giống nhau |
| **TC-FR10-BV-005** | Tự chuyển đổi cùng trạng thái (`delivered` $\rightarrow$ `delivered`) | Terminal Self-loop Boundary | `status`: `"delivered"`, Actor: Admin | Đơn hàng đã ở `delivered`.<br>1. Admin gửi cập nhật `status: "delivered"` | Từ chối vì đã ở trạng thái kết thúc *(FR-10)* | High | Not executed | Biên trạng thái kết thúc |
| **TC-FR10-BV-006** | Tự chuyển đổi cùng trạng thái (`canceled` $\rightarrow$ `canceled`) | Terminal Self-loop Boundary | `status`: `"canceled"`, Actor: Admin | Đơn hàng đã ở `canceled`.<br>1. Admin gửi cập nhật `status: "canceled"` | Từ chối vì đã ở trạng thái kết thúc *(FR-10)* | High | Not executed | Biên trạng thái kết thúc |
| **TC-FR10-BV-007** | Biên `order_id` = 0 ($min - 1$) | Invalid Boundary | `id`: `0` | Gửi PUT `/api/admin/orders/0/status` hoặc `/api/orders/0/cancel` | Báo lỗi 404 Order not found hoặc 400 Invalid ID | Medium | Not executed | Cận dưới của ID đơn hàng |
| **TC-FR10-BV-008** | Biên `order_id` âm (`-1`) | Invalid Boundary | `id`: `-1` | Gửi PUT `/api/admin/orders/-1/status` | Báo lỗi 404 Order not found | Medium | Not executed | Giá trị số âm |
| **TC-FR10-BV-009** | Biên `order_id` không phải số (`"abc"`) | Invalid Boundary | `id`: `"abc"` | Gửi PUT `/api/admin/orders/abc/status` | Báo lỗi 404 hoặc 400 Bad Request | Medium | Not executed | Sai kiểu dữ liệu ID |
| **TC-FR10-BV-010** | Biên `order_id` rất lớn không tồn tại (`999999`) | Upper Range Boundary | `id`: `999999` | Gửi PUT `/api/admin/orders/999999/status` | Trả về HTTP 404 `Order not found` | High | Not executed | Biên ngoài dải ID trong CSDL |

---

## 4. Tổng kết Độ phủ Giá trị Biên (BVA Coverage Summary)

- **Tổng số ca kiểm thử BVA mới bổ sung:** **10 test cases** (`TC-FR10-BV-001` đến `TC-FR10-BV-010`).
- **Các ca biên quan trọng đã bao phủ ở Step 2:** Chuyển đổi từ `delivered`/`canceled` sang trạng thái khác đã có tại `TC-FR10-DT-009`, `TC-FR10-DT-010`, `TC-FR10-DT-011`.
- **Tổng số test cases lũy kế của FR-10 (Domain + BVA):** **31 test cases** (21 Domain + 10 BVA mới).
