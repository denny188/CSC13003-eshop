# Báo cáo Phân tích Giá trị Biên (Boundary Value Analysis - BVA): FR-15 (Quản lý Sản phẩm)

## 1. Phương pháp & Nguyên tắc Áp dụng
1. Áp dụng kỹ thuật **Boundary Value Analysis (BVA)** cho các biến có miền giá trị có thứ tự (ordered domains) và ràng buộc về độ dài/số lượng trong đặc tả FR-15:
   - **Tên sản phẩm (`name` length boundaries):** Ràng buộc đặc tả là *"tối đa 255 ký tự"*, tối thiểu 1 ký tự.
     Tập điểm biên khảo sát:
     - Cận dưới: $min - 1 = 0$ (rỗng), $min = 1$ ký tự, $min + 1 = 2$ ký tự.
     - Cận trên: $max - 1 = 254$ ký tự, $max = 255$ ký tự, $max + 1 = 256$ ký tự.
   - **Giá sản phẩm (`price` value boundaries):** Ràng buộc đặc tả là *"số dương (> 0)"*. Đơn vị tiền nhỏ nhất có thể là $1$ hoặc $0.01$ (số thập phân).
     Tập điểm biên khảo sát:
     - Giá trị cận: $-1$ (âm), $0$ (cận ranh giới bị cấm), $0.01$ hoặc $1$ (cận dưới nhỏ nhất hợp lệ), $2$ ($min + 1$).
     - Biên trên: Giá trị tiền rất lớn (Stress boundary: `1,000,000,000` hoặc tràn số `Number.MAX_SAFE_INTEGER`).
   - **Mã sản phẩm (`product_id` boundaries khi Sửa/Xóa):** Cận dưới $0, -1$, và ID không tồn tại lớn.
2. Loại bỏ trùng lặp: Các trường hợp đã được kiểm tra ở Step 2 được ghi chú đối chiếu chéo (Cross-reference), tập trung mở rộng các điểm biên sát nút.

---

## 2. Bảng Phân tích Các Điểm Biên (Boundary Identification)

### 2.1 Biên Độ dài Tên sản phẩm (`name` length)
- **Cận dưới ($min = 1$ ký tự):**
  - $min - 1 = 0$ ký tự: Chuỗi rỗng (Không hợp lệ - đã có ở TC-FR15-DT-002).
  - $min = 1$ ký tự: Chuỗi đúng 1 ký tự (Hợp lệ - Cận dưới nhỏ nhất).
  - $min + 1 = 2$ ký tự: Chuỗi 2 ký tự (Hợp lệ).
- **Cận trên ($max = 255$ ký tự):**
  - $max - 1 = 254$ ký tự: Chuỗi 254 ký tự (Hợp lệ - Sát cận trên).
  - $max = 255$ ký tự: Chuỗi đúng 255 ký tự (Hợp lệ - Cận trên tối đa theo spec).
  - $max + 1 = 256$ ký tự: Chuỗi 256 ký tự (Không hợp lệ - Vượt quá giới hạn spec).

### 2.2 Biên Giá trị số học của Giá (`price` value)
- **Cận không âm / số dương ($price > 0$):**
  - $min - 1 = 0$: Giá bằng 0 (Không hợp lệ - Vi phạm số dương).
  - $min = 1$ ₫: Giá nguyên dương nhỏ nhất (Hợp lệ).
  - $min = 0.01$: Giá thập phân dương nhỏ nhất (Hợp lệ / Kiểm tra xử lý số thực).
  - Sát dưới: $-0.01$ hoặc $-1$ (Không hợp lệ - Số âm sát biên).
- **Cận trên (Upper value):**
  - Giá trị rất lớn: `999,999,999,999` (Stress/Overflow boundary).

---

## 3. Bảng Ca Kiểm thử Giá trị Biên (BVA Test Cases)

| Mã Test Case | Biến & Điểm biên kiểm tra | Loại biên (Valid / Invalid) | Dữ liệu kiểm thử (Test Data) | Các bước thực hiện | Kết quả mong đợi (Expected Result) | Độ ưu tiên | Trạng thái | Ghi chú & Đối chiếu Step 2 |
| :--- | :--- | :---: | :--- | :--- | :--- | :---: | :---: | :--- |
| **TC-FR15-BV-001** | `name` length = 1 ($min$) | Valid Boundary | `name`: `"A"`, `price`: `10000` | 1. Nhập tên 1 ký tự<br>2. Nhập giá hợp lệ<br>3. Bấm Lưu sản phẩm | Tạo/Lưu sản phẩm thành công, HTTP 200 *(FR-15)* | High | Not executed | Cận dưới độ dài tên ngắn nhất |
| **TC-FR15-BV-002** | `name` length = 2 ($min + 1$) | Valid Boundary | `name`: `"AB"`, `price`: `10000` | 1. Nhập tên 2 ký tự<br>2. Bấm Lưu | Tạo/Lưu sản phẩm thành công, HTTP 200 *(FR-15)* | Medium | Not executed | Điểm biên lân cận $min+1$ |
| **TC-FR15-BV-003** | `name` length = 254 ($max - 1$) | Valid Boundary | Chuỗi 254 ký tự chữ cái, `price`: `50000` | 1. Nhập tên 254 ký tự<br>2. Bấm Lưu | Tạo/Lưu sản phẩm thành công, HTTP 200 *(FR-15)* | Medium | Not executed | Sát cận trên tối đa |
| **TC-FR15-BV-004** | `name` length = 255 ($max$) | Valid Boundary | Chuỗi đúng 255 ký tự chữ cái, `price`: `50000` | 1. Nhập tên 255 ký tự<br>2. Bấm Lưu | Tạo/Lưu sản phẩm thành công, HTTP 200 *(FR-15: Tối đa 255 ký tự)* | High | Not executed | Cận trên chính xác theo spec |
| **TC-FR15-BV-005** | `name` length = 256 ($max + 1$) | Invalid Boundary | Chuỗi 256 ký tự chữ cái, `price`: `50000` | 1. Nhập tên 256 ký tự<br>2. Bấm Lưu | Hệ thống từ chối, báo lỗi tên vượt quá 255 ký tự *(FR-15)* | High | Not executed | Vượt giới hạn trên 1 ký tự |
| **TC-FR15-BV-006** | `price` = 1 ($min$ nguyên dương) | Valid Boundary | `name`: `"Bút bi"`, `price`: `1` | 1. Nhập giá bằng 1<br>2. Bấm Lưu | Tạo/Lưu sản phẩm thành công *(FR-15: số dương > 0)* | High | Not executed | Giá trị nguyên dương nhỏ nhất |
| **TC-FR15-BV-007** | `price` = 0.01 ($min$ số thực dương) | Valid Boundary | `name`: `"Kẹo"`, `price`: `0.01` | 1. Nhập giá bằng 0.01<br>2. Bấm Lưu | Chấp nhận giá số thực dương hoặc làm tròn an toàn *(FR-15)* | Medium | Not executed | Khảo sát số thực sát 0 |
| **TC-FR15-BV-008** | `price` = -0.01 (Sát dưới 0) | Invalid Boundary | `name`: `"Lỗi"`, `price`: `-0.01` | 1. Nhập giá bằng -0.01<br>2. Bấm Lưu | Từ chối, báo lỗi giá phải là số dương *(FR-15)* | High | Not executed | Sát biên âm |
| **TC-FR15-BV-009** | `price` rất lớn (`1,000,000,000,000`) | Stress Boundary | `price`: `1000000000000` | 1. Nhập giá tiền 1 nghìn tỷ<br>2. Bấm Lưu | Hệ thống xử lý an toàn không sập DB, hiển thị đúng định dạng tiền tệ *(FR-05, FR-21)* | Low | Not executed | Kiểm tra tràn số hoặc hiển thị chuỗi |
| **TC-FR15-BV-010** | `product_id` = 0 ($min - 1$ khi Sửa/Xóa) | Invalid Boundary | `id`: `0` | Gọi PUT/DELETE `/api/products/0` | Báo lỗi không tìm thấy sản phẩm (404) | Medium | Not executed | Cận dưới của ID sản phẩm |

---

## 4. Tổng kết Độ phủ Giá trị Biên (BVA Coverage Summary)

- **Tổng số ca kiểm thử BVA mới bổ sung:** **10 test cases** (`TC-FR15-BV-001` đến `TC-FR15-BV-010`).
- **Các ca biên đã khảo sát ở Step 2:** `price = 0` (TC-FR15-DT-005), `price = -50000` (TC-FR15-DT-006), `name` rỗng (TC-FR15-DT-002).
- **Tổng số test cases lũy kế của FR-15 (Domain + BVA):** **26 test cases** (16 Domain + 10 BVA mới).
