# Phân tích Yêu cầu Kiểm thử: FR-16 (Import Sản phẩm từ CSV)

## 1. Trích dẫn Yêu cầu từ Đặc tả (Spec Excerpts)

### 1.1 Đặc tả chức năng (README.md - FR-16)
- **FR-16: Import Sản phẩm từ CSV:**
  - Admin có thể tải lên file CSV để import nhiều sản phẩm cùng lúc.
  - **Yêu cầu file CSV:**
    - Đuôi file phải là `.csv`.
    - Dòng đầu tiên là header: `name,price,description,imageUrl,category_id`.
    - Hỗ trợ các trường có chứa dấu phẩy nếu được bọc trong dấu nháy kép (RFC 4180).
  - **Validation trước khi import:**
    - `name` không được rỗng.
    - `price` phải là số dương (> 0).
  - **Quy tắc giao dịch (Transactional Integrity / Atomicity):**
    - *"Nếu có lỗi ở bất kỳ dòng nào, toàn bộ import phải được **rollback** (giao dịch nguyên tử — all-or-nothing)."*
  - **Báo cáo kết quả:**
    - *"Hệ thống hiển thị báo cáo rõ ràng: bao nhiêu dòng thành công, bao nhiêu dòng lỗi và lý do."*

### 1.2 Yêu cầu Phân quyền & Bảo mật (SEC-02, SEC-03, FR-12)
- Phân hệ Admin chỉ dành cho tài khoản có `role = 'admin'`.
- Tất cả các API Admin (`/api/admin/*`) đều phải yêu cầu:
  1. Token JWT hợp lệ.
  2. `role = 'admin'` trong Token.
- Khách vãng lai hoặc User thông thường không được phép gọi API import.

### 1.3 Đặc tả API (api_specification.md)
- `POST /api/admin/import-products`: Nhận danh sách sản phẩm để import vào CSDL.
- Headers: `Authorization: Bearer <admin_token>`, `Content-Type: application/json`.
- Body: `{ "products": [ { "name": "...", "price": 100000, ... } ] }` hoặc file upload.

---

## 2. Bảng phân tích Biến đầu vào (Input Variables)

| Tên biến | Vị trí (UI / API) | Kiểu dữ liệu | Ràng buộc giá trị / Định dạng | Bắt buộc? | Nguồn đặc tả (Spec Ref) |
| :--- | :--- | :--- | :--- | :---: | :--- |
| `file` (Extension) | File Picker / UI | File `.csv` | Đuôi mở rộng file bắt buộc phải là `.csv`, từ chối file lạ (`.txt`, `.json`, `.exe`) | Có | FR-16 |
| `file_header` | Dòng 1 của CSV | Chuỗi ký tự | Phải có định dạng đúng chuẩn: `name,price,description,imageUrl,category_id` | Có | FR-16 |
| `name` (từng dòng) | Cột 1 / Payload | Chuỗi ký tự (String) | Bắt buộc, không được để rỗng hoặc khoảng trắng, tối đa 255 ký tự (kế thừa FR-15) | Có | FR-16 |
| `price` (từng dòng) | Cột 2 / Payload | Số (Number) | Bắt buộc, phải là số thực/nguyên dương ($price > 0$) | Có | FR-16 |
| `description` (từng dòng) | Cột 3 / Payload | Chuỗi ký tự (String) | Tùy chọn; có thể chứa dấu phẩy nếu được bọc trong dấu nháy kép `""` (RFC 4180) | Không | FR-16 |
| `imageUrl` (từng dòng) | Cột 4 / Payload | Chuỗi ký tự (URL) | Tùy chọn; đường dẫn ảnh an toàn | Không | FR-16 |
| `category_id` (từng dòng)| Cột 5 / Payload | Số nguyên (Integer) | Bắt buộc; phải tham chiếu đến ID danh mục đang tồn tại | Có | FR-16, FR-15 |
| `products` (Batch array) | Request Body (API) | Mảng đối tượng (Array) | Mảng danh sách sản phẩm; không được rỗng `[]` | Có | api_spec |
| `actor_role` | JWT Header | Phân quyền (Enum) | Bắt buộc phải là tài khoản có vai trò `admin` | Có | FR-12, SEC-03 |

---

## 3. Danh mục Hành vi & Kết quả đầu ra cần kiểm tra (Expected Outputs & Behaviors)

1. **Kiểm tra tính hợp lệ của định dạng CSV & Parser RFC 4180:**
   - Hệ thống chấp nhận file `.csv` hợp lệ và từ chối các định dạng file không phải `.csv`.
   - Hệ thống phân tách chính xác các trường chứa dấu phẩy được bao bọc bởi cặp ngoặc kép `""` theo chuẩn RFC 4180 (ví dụ trường mô tả: `"Laptop màn hình 15.6 inch, RAM 16GB, SSD 512GB"` không bị chia tách nhầm thành nhiều cột).
2. **Kiểm tra xác thực dữ liệu từng dòng (Row Validation):**
   - Dòng hợp lệ: `name` có giá trị, `price > 0`, `category_id` tồn tại.
   - Dòng không hợp lệ: `name` bị rỗng, `price <= 0` (giá = 0 hoặc âm), `price` là chữ, `category_id` không tồn tại hoặc sai định dạng.
3. **Tính nguyên tử của giao dịch (Transactional Rollback / All-or-Nothing):**
   - Nếu **tất cả** các dòng đều hợp lệ: Toàn bộ sản phẩm trong file được thêm mới thành công vào CSDL.
   - Nếu **bất kỳ dòng nào** bị lỗi (kể cả dòng cuối cùng trong file 100 dòng): Toàn bộ thao tác import phải bị **rollback 100%**, không có bất kỳ sản phẩm nào trong file được lưu lại vào CSDL.
4. **Báo cáo kết quả chi tiết:**
   - Khi thành công: Hiển thị rõ tổng số dòng import thành công.
   - Khi có lỗi / rollback: Hiển thị báo cáo rõ ràng chỉ rõ dòng bị lỗi, số lượng lỗi và nguyên nhân cụ thể từng lỗi.
5. **Kiểm soát truy cập (Access Control / SEC-03):**
   - API `/api/admin/import-products` phải được bảo vệ nghiêm ngặt: yêu cầu Token hợp lệ và `role = 'admin'`. Người dùng thông thường hoặc khách vãng lai phải bị từ chối với mã lỗi 401/403.

---

## 4. Các điểm mơ hồ / Khe hở đặc tả cần xác nhận (Ambiguities & Spec Gaps)

1. **Cơ chế tải lên và xử lý CSV (Client-side vs Server-side parsing):**
   - Đặc tả yêu cầu tải lên file CSV, nhưng phía frontend có tự phân tích (parse) thô sơ bằng `split(",")` rồi gửi JSON lên backend hay gửi trực tiếp file multipart lên server để backend parse chuẩn RFC 4180?
2. **Xử lý sản phẩm trùng lặp (Duplicate Handling):**
   - Nếu trong file CSV có sản phẩm trùng tên với sản phẩm đã có sẵn trong cơ sở dữ liệu, hệ thống xử lý ghi đè (update), thêm mới tạo bản sao (duplicate), hay báo lỗi từ chối?
3. **Trường hợp file CSV rỗng hoặc chỉ có dòng Header:**
   - File CSV rỗng hoàn toàn hoặc file chỉ chứa đúng 1 dòng header mà không có dòng dữ liệu nào thì hệ thống xử lý phản hồi thông báo như thế nào?
