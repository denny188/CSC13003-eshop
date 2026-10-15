# Báo cáo Kiểm thử Miền giá trị (Domain Testing): FR-15 (Quản lý Sản phẩm)

## 1. Phương pháp & Nguyên tắc Áp dụng
1. Áp dụng kỹ thuật **Phân hoạch tương đương (Equivalence Partitioning)** cho từng biến đầu vào của chức năng CRUD sản phẩm:
   - `name`: Lớp hợp lệ (chuỗi chuẩn) và các lớp không hợp lệ (rỗng, toàn khoảng trắng, chứa script XSS).
   - `price`: Lớp hợp lệ ($price > 0$) và các lớp không hợp lệ ($price = 0$, $price < 0$, chuỗi ký tự, để trống).
   - `category_id`: Lớp hợp lệ (ID tồn tại trong `categories`) và các lớp không hợp lệ (ID không tồn tại, để trống, số âm).
   - Quyền hạn (`actor_role`): Admin hợp lệ, User thường (không có quyền), và Không xác thực.
2. Thiết kế ca kiểm thử theo nguyên tắc **Single-fault**:
   - 1 ca kiểm thử hợp lệ toàn diện cho chức năng Thêm mới (Create).
   - Các ca kiểm thử đơn lỗi cho từng biến nhập liệu không hợp lệ.
   - Các ca kiểm thử chức năng Xem (Read), Sửa (Update), và Xóa (Delete).
   - **Ca kiểm thử đặc biệt về tính cô lập (Update Isolation):** Kiểm tra khi sửa sản phẩm $A$, các sản phẩm khác $B, C$ có bị thay đổi dữ liệu ngoài ý muốn hay không.

---

## 2. Bảng Phân hoạch Tương đương & Chọn Giá trị Đại diện (Domain Table)

### 2.1 Biến Tên sản phẩm (`name`)
| Mã lớp | Mô tả lớp tương đương | Hợp lệ / Không hợp lệ | Giá trị đại diện | Lý giải chọn giá trị (Rationale) |
| :--- | :--- | :---: | :--- | :--- |
| **EC-NM-V1** | Tên hợp lệ thông thường (1-255 ký tự) | Hợp lệ | `"Áo thun Polo Nam EShop"` | Tên sản phẩm điển hình có dấu tiếng Việt |
| **EC-NM-I1** | Tên để trống (Empty string) | Không hợp lệ | `""` | Kiểm tra bắt buộc nhập (mandatory) |
| **EC-NM-I2** | Chỉ chứa ký tự khoảng trắng | Không hợp lệ | `"    "` | Kiểm tra hàm trim/strip khoảng trắng |
| **EC-NM-I3** | Chứa mã độc XSS / thẻ HTML | Không hợp lệ | `"<script>alert('XSS')</script>"` | Kiểm tra an toàn dữ liệu đầu vào (SEC-04) |

---

### 2.2 Biến Giá sản phẩm (`price`)
| Mã lớp | Mô tả lớp tương đương | Hợp lệ / Không hợp lệ | Giá trị đại diện | Lý giải chọn giá trị (Rationale) |
| :--- | :--- | :---: | :--- | :--- |
| **EC-PR-V1** | Giá là số dương (> 0) | Hợp lệ | `150000` | Giá trị tiền nguyên dương phổ biến |
| **EC-PR-I1** | Giá bằng 0 | Không hợp lệ | `0` | Vi phạm điều kiện số dương (> 0) |
| **EC-PR-I2** | Giá là số âm (< 0) | Không hợp lệ | `-50000` | Vi phạm điều kiện số dương (> 0) |
| **EC-PR-I3** | Để trống giá tiền | Không hợp lệ | `null` hoặc `""` | Kiểm tra bắt buộc nhập |
| **EC-PR-I4** | Giá là chuỗi ký tự không phải số | Không hợp lệ | `"mot_trieu"` | Kiểm tra kiểm soát kiểu dữ liệu |

---

### 2.3 Biến Danh mục (`category_id`)
| Mã lớp | Mô tả lớp tương đương | Hợp lệ / Không hợp lệ | Giá trị đại diện | Lý giải chọn giá trị (Rationale) |
| :--- | :--- | :---: | :--- | :--- |
| **EC-CT-V1** | Danh mục hợp lệ tồn tại trong CSDL | Hợp lệ | `1` | ID danh mục có sẵn trong bảng `categories` |
| **EC-CT-I1** | Danh mục không tồn tại trong CSDL | Không hợp lệ | `9999` | Khóa ngoại không tồn tại |
| **EC-CT-I2** | Để trống danh mục | Không hợp lệ | `null` hoặc `""` | Kiểm tra bắt buộc chọn danh mục |

---

### 2.4 Biến Quyền hạn Tác nhân (`actor_role`)
| Mã lớp | Mô tả lớp tương đương | Hợp lệ / Không hợp lệ | Giá trị đại diện | Lý giải chọn giá trị (Rationale) |
| :--- | :--- | :---: | :--- | :--- |
| **EC-RO-V1** | Quản trị viên (Admin) | Hợp lệ | Token của `admin@eshop.com` | Có quyền Thêm / Sửa / Xóa sản phẩm |
| **EC-RO-I1** | Người dùng thường (User) | Không hợp lệ | Token của `test@eshop.com` | Không được phép Thêm / Sửa / Xóa sản phẩm |
| **EC-RO-I2** | Khách vãng lai chưa đăng nhập | Không hợp lệ | Không có header Authorization | Phải bị từ chối với HTTP 401 |

---

## 3. Danh sách Ca Kiểm thử Chi tiết (Domain Test Cases)

| Mã Test Case | Lớp tương đương bao phủ | Điều kiện tiên quyết (Preconditions) | Các bước thực hiện (Steps) | Dữ liệu kiểm thử (Test Data) | Kết quả mong đợi (Expected Result) | Độ ưu tiên | Trạng thái |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: | :---: |
| **TC-FR15-DT-001** | **EC-NM-V1, EC-PR-V1, EC-CT-V1, EC-RO-V1** (Valid Create) | Đăng nhập Admin | 1. Mở form Thêm sản phẩm<br>2. Nhập tên, giá, chọn danh mục<br>3. Bấm Lưu sản phẩm | `name`: `"Áo thun Polo Nam"`, `price`: `150000`, `category_id`: `1`, `imageUrl`: `"https://placehold.co/300"` | Tạo sản phẩm thành công, lưu vào CSDL, hiển thị trên danh sách, HTTP 200 *(FR-15)* | High | Not executed |
| **TC-FR15-DT-002** | **EC-NM-I1** (Tên rỗng) | Đang ở form Thêm/Sửa SP | 1. Để trống tên SP<br>2. Nhập giá hợp lệ<br>3. Bấm Lưu | `name`: `""`, `price`: `100000` | Báo lỗi bắt buộc nhập tên sản phẩm, chặn submit *(FR-15, FR-22)* | High | Not executed |
| **TC-FR15-DT-003** | **EC-NM-I2** (Tên toàn khoảng trắng) | Đang ở form Thêm/Sửa SP | 1. Nhập khoảng trắng vào tên<br>2. Nhập giá hợp lệ<br>3. Bấm Lưu | `name`: `"    "`, `price`: `100000` | Báo lỗi tên không được chỉ chứa khoảng trắng *(FR-15)* | Medium | Not executed |
| **TC-FR15-DT-004** | **EC-NM-I3** (XSS trong tên) | Đang ở form Thêm SP | 1. Nhập mã script vào tên SP<br>2. Bấm Lưu và xem hiển thị | `name`: `"<script>alert('XSS')</script>"`, `price`: `100000` | Escape HTML an toàn khi hiển thị, không thực thi script *(SEC-04)* | High | Not executed |
| **TC-FR15-DT-005** | **EC-PR-I1** (Giá = 0) | Đang ở form Thêm SP | 1. Nhập tên hợp lệ<br>2. Nhập giá bằng 0<br>3. Bấm Lưu | `name`: `"Sản phẩm Giá 0"`, `price`: `0` | Báo lỗi giá phải là số dương (> 0) *(FR-15)* | High | Not executed |
| **TC-FR15-DT-006** | **EC-PR-I2** (Giá âm < 0) | Đang ở form Thêm SP | 1. Nhập tên hợp lệ<br>2. Nhập giá âm<br>3. Bấm Lưu | `name`: `"Sản phẩm Giá Âm"`, `price`: `-50000` | Báo lỗi giá phải là số dương (> 0) *(FR-15)* | High | Not executed |
| **TC-FR15-DT-007** | **EC-PR-I3** (Để trống giá) | Đang ở form Thêm SP | 1. Nhập tên hợp lệ<br>2. Để trống ô giá<br>3. Bấm Lưu | `name`: `"Sản phẩm Không Giá"`, `price`: `""` | Báo lỗi giá bắt buộc phải nhập *(FR-15, FR-22)* | High | Not executed |
| **TC-FR15-DT-008** | **EC-PR-I4** (Giá là chuỗi) | Gọi API POST /api/products | 1. Gửi request tạo SP với giá là text | `name`: `"SP Test"`, `price`: `"mot_trieu"` | Báo lỗi kiểu dữ liệu giá không hợp lệ *(FR-15)* | Medium | Not executed |
| **TC-FR15-DT-009** | **EC-CT-I1** (Danh mục không tồn tại) | Gọi API tạo sản phẩm | 1. Gửi request tạo SP với category_id lạ | `name`: `"SP Test"`, `price`: `100000`, `category_id`: `9999` | Từ chối tạo sản phẩm, báo lỗi danh mục không hợp lệ *(FR-15)* | High | Not executed |
| **TC-FR15-DT-010** | **EC-CT-I2** (Để trống danh mục) | Gọi API tạo sản phẩm | 1. Gửi request thiếu trường `category_id` | `name`: `"SP Test"`, `price`: `100000`, `category_id`: `null` | Báo lỗi bắt buộc chọn danh mục *(FR-15)* | Medium | Not executed |
| **TC-FR15-DT-011** | **Update Isolation** (Sửa sản phẩm độc lập) | Có ít nhất 2 sản phẩm trong CSDL ($P_1, P_2$) | 1. Sửa tên và giá của sản phẩm $P_1$<br>2. Lưu thay đổi<br>3. Kiểm tra thông tin của $P_1$ và $P_2$ trên UI và CSDL | Sửa $P_1$: `name = "Tên Mới P1"`, `price = 200000` | Chỉ $P_1$ bị thay đổi; thông tin của $P_2$ và các sản phẩm khác phải **giữ nguyên 100%** *(FR-15: Khi Sửa một sản phẩm, chỉ sản phẩm đó bị thay đổi — các sản phẩm khác giữ nguyên)* | Critical | Not executed |
| **TC-FR15-DT-012** | **Delete Product** (Xóa sản phẩm) | Có sản phẩm $P_3$ trong CSDL | 1. Bấm nút Xóa sản phẩm $P_3$<br>2. Kiểm tra lại danh sách | ID sản phẩm cần xóa | Sản phẩm bị xóa khỏi CSDL và danh sách UI, HTTP 200 *(FR-15)* | High | Not executed |
| **TC-FR15-DT-013** | **EC-RO-I1** (User thường gọi API tạo SP) | Đăng nhập tài khoản User | 1. Gửi POST `/api/products` với token User | Token của `test@eshop.com` | Từ chối Forbidden (403), không tạo sản phẩm *(SEC-03)* | High | Not executed |
| **TC-FR15-DT-014** | **EC-RO-I1** (User thường gọi API sửa SP) | Đăng nhập tài khoản User | 1. Gửi PUT `/api/products/:id` với token User | Token của `test@eshop.com` | Từ chối Forbidden (403), không sửa sản phẩm *(SEC-03)* | High | Not executed |
| **TC-FR15-DT-015** | **EC-RO-I1** (User thường gọi API xóa SP) | Đăng nhập tài khoản User | 1. Gửi DELETE `/api/products/:id` với token User | Token của `test@eshop.com` | Từ chối Forbidden (403), không xóa sản phẩm *(SEC-03)* | High | Not executed |
| **TC-FR15-DT-016** | **EC-RO-I2** (Không có token gọi API Thêm/Sửa/Xóa) | Không gửi token | 1. Gửi POST/PUT/DELETE tới `/api/products` | Không có header Authorization | Từ chối Unauthorized (401) *(SEC-02)* | High | Not executed |

---

## 4. Ma trận Độ phủ Lớp tương đương (Coverage Matrix)

| Lớp tương đương | Mã Test Case bao phủ |
| :--- | :--- |
| **EC-NM-V1** | TC-FR15-DT-001, TC-FR15-DT-011 |
| **EC-NM-I1** | TC-FR15-DT-002 |
| **EC-NM-I2** | TC-FR15-DT-003 |
| **EC-NM-I3** | TC-FR15-DT-004 |
| **EC-PR-V1** | TC-FR15-DT-001, TC-FR15-DT-011 |
| **EC-PR-I1** | TC-FR15-DT-005 |
| **EC-PR-I2** | TC-FR15-DT-006 |
| **EC-PR-I3** | TC-FR15-DT-007 |
| **EC-PR-I4** | TC-FR15-DT-008 |
| **EC-CT-V1** | TC-FR15-DT-001, TC-FR15-DT-011 |
| **EC-CT-I1** | TC-FR15-DT-009 |
| **EC-CT-I2** | TC-FR15-DT-010 |
| **EC-RO-V1** | TC-FR15-DT-001, TC-FR15-DT-011, TC-FR15-DT-012 |
| **EC-RO-I1** | TC-FR15-DT-013, TC-FR15-DT-014, TC-FR15-DT-015 |
| **EC-RO-I2** | TC-FR15-DT-016 |

**Tổng số test case thiết kế trong Step 2 (FR-15):** **16 test cases** (Bao phủ 100% các lớp tương đương và ràng buộc nghiệp vụ cốt lõi).
