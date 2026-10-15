# Phân tích Yêu cầu Kiểm thử: FR-15 (Quản lý Sản phẩm - Product CRUD)

## 1. Trích dẫn Yêu cầu từ Đặc tả (Spec Excerpts)

### 1.1 Đặc tả chức năng (README.md - FR-15)
- **FR-15: Quản lý Sản phẩm (Product CRUD):**
  - Admin có thể Thêm / Xem / Sửa / Xóa sản phẩm.
  - **Ràng buộc đầu vào:**
    - **Tên sản phẩm (`name`):** bắt buộc, tối đa 255 ký tự.
    - **Giá (`price`):** bắt buộc, phải là số **dương** (> 0).
    - **Danh mục (`category_id`):** bắt buộc, phải chọn từ danh sách có sẵn.
  - **Ràng buộc khi Sửa (Update isolation):**
    - *"Khi Sửa một sản phẩm, chỉ sản phẩm đó bị thay đổi — các sản phẩm khác giữ nguyên."*

### 1.2 Yêu cầu Phân quyền & Bảo mật (SEC-02, SEC-03)
- Quản lý sản phẩm (Thêm/Sửa/Xóa) là tính năng quản trị của Admin, yêu cầu JWT Token có `role = 'admin'`. Người dùng không có quyền hoặc khách vãng lai không được phép chỉnh sửa/xóa sản phẩm.

### 1.3 Đặc tả API (api_specification.md)
- `GET /api/products`: Lấy danh sách sản phẩm.
- `GET /api/products/:id`: Xem chi tiết sản phẩm.
- `POST /api/products`: Thêm sản phẩm mới (Body: `name`, `price`, `description`, `imageUrl`, `category_id`).
- `PUT /api/products/:id`: Cập nhật thông tin sản phẩm theo ID.
- `DELETE /api/products/:id`: Xóa sản phẩm theo ID.

---

## 2. Bảng phân tích Biến đầu vào (Input Variables)

| Tên biến | Vị trí (UI / API) | Kiểu dữ liệu | Ràng buộc giá trị / Định dạng | Bắt buộc? | Nguồn đặc tả (Spec Ref) |
| :--- | :--- | :--- | :--- | :---: | :--- |
| `name` | Form Input / API Body | Chuỗi ký tự (String) | Bắt buộc; không được rỗng; độ dài từ 1 đến 255 ký tự; không chứa mã độc XSS/HTML | Có | FR-15, FR-22 |
| `price` | Form Input / API Body | Số thực / Số nguyên (Number) | Bắt buộc; phải là số dương ($price > 0$) | Có | FR-15, FR-22 |
| `category_id` | Select dropdown / API Body | Số nguyên (Integer) | Bắt buộc; phải chọn từ danh sách danh mục có sẵn (`categories.id`) | Có | FR-15 |
| `description` | Textarea / API Body | Chuỗi ký tự (String) | Tùy chọn (Mô tả sản phẩm); text an toàn | Không | api_spec 3.3 |
| `imageUrl` | Form Input / API Body | Chuỗi ký tự (URL) | Tùy chọn; định dạng URL hình ảnh hợp lệ | Không | api_spec 3.3, FR-05 |
| `product_id` (`:id`) | URL Param (`/api/products/:id`) | Số nguyên dương (Integer) | Phải tồn tại trong CSDL khi Sửa hoặc Xóa | Có (Sửa/Xóa) | api_spec 3.3 |
| `actor_role` | JWT Token Header | Phân quyền (Enum) | Phải là `admin` (`role = 'admin'`) | Có | SEC-03 |

---

## 3. Danh mục Hành vi & Kết quả đầu ra cần kiểm tra (Expected Outputs & Behaviors)

1. **Thêm sản phẩm mới (Create):**
   - Khi nhập đầy đủ `name` hợp lệ (1-255 ký tự), `price > 0`, chọn `category_id` có sẵn: Sản phẩm mới được tạo trong CSDL, danh sách sản phẩm hiển thị sản phẩm mới, trả về HTTP 200.
   - Khi `name` rỗng hoặc toàn khoảng trắng: Hệ thống từ chối, báo lỗi bắt buộc nhập.
   - Khi `price <= 0` (ví dụ `0`, `-1000`): Hệ thống từ chối, báo lỗi giá phải là số dương.
   - Khi `category_id` không hợp lệ hoặc không tồn tại: Hệ thống từ chối, báo lỗi danh mục.
2. **Xem chi tiết & Danh sách sản phẩm (Read):**
   - Danh sách sản phẩm hiển thị đầy đủ thông tin: Tên, Giá, Ảnh, Danh mục.
3. **Cập nhật sản phẩm (Update Isolation):**
   - Khi sửa thông tin sản phẩm $A$: Chỉ duy nhất thông tin của sản phẩm $A$ bị thay đổi trong CSDL và trên giao diện. Tất cả các sản phẩm khác ($B, C, ...$) phải **hoàn toàn giữ nguyên thông tin ban đầu**.
4. **Xóa sản phẩm (Delete):**
   - Khi xóa sản phẩm theo ID hợp lệ: Sản phẩm bị xóa khỏi hệ thống, danh sách không còn sản phẩm đó.
   - Khi xóa với ID không tồn tại: Trả về lỗi phù hợp.
5. **Kiểm soát truy cập (Access Control):**
   - Các API Thêm/Sửa/Xóa sản phẩm nếu được gọi bởi User thường hoặc Client không có token Admin phải bị từ chối với HTTP 401/403 (SEC-02, SEC-03).

---

## 4. Các điểm mơ hồ / Khe hở đặc tả cần xác nhận (Ambiguities & Spec Gaps)

1. **Ràng buộc tên sản phẩm trùng lặp (Unique name):**
   - Đặc tả FR-15 không quy định tên sản phẩm có bắt buộc duy nhất (unique) hay không.
2. **Đơn vị và kiểu số của `price`:**
   - Hệ thống cho phép giá dạng số thực (ví dụ: `1000.5`) hay bắt buộc là số nguyên VNĐ?
3. **Xóa sản phẩm đang nằm trong đơn hàng (Foreign Key constraint):**
   - Khi xóa sản phẩm đã có đơn hàng tham chiếu, hệ thống xử lý xóa mềm (soft delete), chặn xóa (restrict), hay cascade?
