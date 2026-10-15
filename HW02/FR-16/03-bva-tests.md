# Báo cáo Phân tích Giá trị Biên (Boundary Value Analysis - BVA): FR-16 (Import Sản phẩm từ CSV)

## 1. Phương pháp & Nguyên tắc Áp dụng
1. Áp dụng kỹ thuật **Boundary Value Analysis (BVA)** cho các biến có miền giá trị có thứ tự (ordered domains), số lượng dòng trong file CSV, độ dài chuỗi ký tự và giá trị tiền tệ:
   - **Số lượng dòng trong file CSV (`row_count` boundaries):**
     - $0$ dòng dữ liệu (File rỗng hoàn toàn 0 byte hoặc file chỉ có 1 dòng header): $min - 1$.
     - $1$ dòng dữ liệu hợp lệ: $min$ (File nhỏ nhất có thể import).
     - $2$ dòng dữ liệu: $min + 1$.
     - Số lượng dòng lớn (Stress boundary: 50 dòng, 500 dòng).
   - **Vị trí dòng bị lỗi trong file CSV (`error_row_position` boundaries):**
     - Dòng lỗi ở vị trí đầu tiên ($row = 1$ sau header).
     - Dòng lỗi ở vị trí giữa file ($row = k$).
     - Dòng lỗi ở vị trí cuối cùng ($row = N$): Điểm biên nhạy cảm nhất để kiểm tra tính nguyên tử (Rollback All-or-Nothing).
   - **Độ dài tên sản phẩm (`name` length boundaries):** Kế thừa từ FR-15 ($min = 1$ ký tự, $max = 255$ ký tự, $256$ ký tự).
   - **Giá trị tiền (`price` value boundaries):** $price = 0$, $price = 1$ ₫, $price = -1$ ₫, $price = 0.01$, giá cực lớn.
2. Loại bỏ trùng lặp: Các trường hợp đã được kiểm tra ở Step 2 được ghi chú đối chiếu chéo (Cross-reference), tập trung mở rộng các điểm biên sát nút và kiểm tra tải giao dịch.

---

## 2. Bảng Phân tích Các Điểm Biên (Boundary Identification)

### 2.1 Biên Số lượng dòng của File CSV (`row_count`)
- **Cận dưới ($min = 1$ dòng dữ liệu):**
  - $0$ dòng dữ liệu (File 0 byte hoặc chỉ có header): Cận dưới không hợp lệ / Empty file.
  - $1$ dòng dữ liệu: File import nhỏ nhất hợp lệ.
  - $2$ dòng dữ liệu: File lân cận cận dưới.
- **Cận tải lớn (Stress Boundary):**
  - File 50 dòng hoặc 100 dòng: Kiểm tra tốc độ xử lý batch và tính toàn vẹn của vòng lặp insert.

---

### 2.2 Biên Vị trí dòng lỗi trong giao dịch (`error_row_position`)
- **Dòng lỗi ở vị trí đầu tiên ($pos = 1$):** Kiểm tra hệ thống dừng ngay từ đầu.
- **Dòng lỗi ở vị trí cuối cùng ($pos = N$):** Điểm biên tới hạn — nếu backend insert tuần tự mà không có transaction/rollback thì $N-1$ dòng trước đó sẽ bị rò rỉ vào CSDL.

---

### 2.3 Biên Giá trị `price` và Độ dài `name` trong CSV
- `price = 0`: Cận ranh giới cấm.
- `price = 1`: Giá nguyên dương nhỏ nhất hợp lệ.
- `price = -1`: Giá âm sát 0 không hợp lệ.
- `name` = 1 ký tự: Ngắn nhất hợp lệ.
- `name` = 255 ký tự: Dài nhất hợp lệ theo spec.
- `name` = 256 ký tự: Vượt quá giới hạn trên 1 ký tự.

---

## 3. Bảng Ca Kiểm thử Giá trị Biên (BVA Test Cases)

| Mã Test Case | Biến & Điểm biên kiểm tra | Loại biên (Valid / Invalid) | Dữ liệu kiểm thử (Test Data) | Các bước thực hiện | Kết quả mong đợi (Expected Result) | Độ ưu tiên | Trạng thái | Ghi chú & Đối chiếu Step 2 |
| :--- | :--- | :---: | :--- | :--- | :--- | :---: | :---: | :--- |
| **TC-FR16-BV-001** | `row_count` = 0 (File 0 byte) | Invalid Boundary | File CSV hoàn toàn rỗng (0 bytes) | 1. Tải lên file 0 byte<br>2. Bấm nút Import | Báo lỗi file rỗng, không có dữ liệu để import *(FR-16)* | High | Not executed | Cận dưới file rỗng hoàn toàn |
| **TC-FR16-BV-002** | `row_count` = 0 (Chỉ có dòng Header) | Invalid Boundary | File CSV chỉ có duy nhất 1 dòng header: `name,price,description,imageUrl,category_id` | 1. Tải lên file chỉ có header<br>2. Bấm Import | Báo lỗi không có dòng sản phẩm nào để import *(FR-16)* | High | Not executed | File có cấu trúc nhưng thiếu dữ liệu |
| **TC-FR16-BV-003** | `row_count` = 1 ($min$ dòng hợp lệ) | Valid Boundary | File CSV gồm header + đúng 1 dòng sản phẩm hợp lệ | 1. Tải lên file 1 dòng<br>2. Bấm Import | Import thành công đúng 1 sản phẩm, CSDL tăng 1, báo cáo 1/1 thành công *(FR-16)* | High | Not executed | Cận dưới số dòng hợp lệ |
| **TC-FR16-BV-004** | `row_count` = 2 ($min + 1$ dòng) | Valid Boundary | File CSV gồm header + 2 dòng sản phẩm hợp lệ | 1. Tải file 2 dòng<br>2. Bấm Import | Import thành công 2 sản phẩm *(FR-16)* | Medium | Not executed | Điểm biên lân cận $min+1$ |
| **TC-FR16-BV-005** | `error_row_pos` = 1 (Lỗi ngay dòng 1 trong file 3 dòng) | Invalid / Rollback Boundary | Dòng 1: `name` rỗng; Dòng 2 & 3 hợp lệ | 1. Tải lên file CSV<br>2. Bấm Import<br>3. Kiểm tra CSDL | Toàn bộ import bị Rollback: Cả dòng 2 và 3 đều không được lưu vào CSDL *(FR-16)* | High | Not executed | Lỗi ở vị trí dòng dữ liệu đầu tiên |
| **TC-FR16-BV-006** | `error_row_pos` = $N$ (Lỗi ở dòng cuối cùng trong file 5 dòng) | Critical Rollback Boundary | 4 dòng đầu hợp lệ; Dòng thứ 5 có `price = 0` | 1. Ghi nhận số SP trong CSDL ($N_0$)<br>2. Import file 5 dòng<br>3. Kiểm tra số SP trong CSDL | Toàn bộ bị **Rollback**: 4 dòng đầu KHÔNG được lưu. Tổng số SP trong CSDL vẫn là $N_0$ *(FR-16)* | Critical | Not executed | Điểm biên cốt lõi kiểm tra All-or-Nothing |
| **TC-FR16-BV-007** | `price` = 1 ($min$ giá nguyên dương trong CSV) | Valid Boundary | 1 dòng sản phẩm có `price = 1` | 1. Tải file CSV có giá bằng 1<br>2. Bấm Import | Import thành công sản phẩm với giá 1 ₫ *(FR-16, FR-15)* | High | Not executed | Giá trị nguyên dương nhỏ nhất |
| **TC-FR16-BV-008** | `price` = 0 ($min - 1$ giá vi phạm) | Invalid Boundary | 1 dòng sản phẩm có `price = 0` | 1. Tải file CSV có giá bằng 0<br>2. Bấm Import | Từ chối import, báo lỗi giá phải là số dương (> 0) *(FR-16)* | Critical | Not executed | Cận ranh giới bị cấm |
| **TC-FR16-BV-009** | `name` length = 1 ($min$ độ dài tên) | Valid Boundary | Tên sản phẩm trong CSV là `"X"` (1 ký tự), `price = 10000` | 1. Import file CSV<br>2. Kiểm tra CSDL | Import thành công sản phẩm tên 1 ký tự *(FR-16, FR-15)* | Medium | Not executed | Cận dưới độ dài tên |
| **TC-FR16-BV-010** | `name` length = 255 ($max$ độ dài tên) | Valid Boundary | Tên sản phẩm trong CSV đúng 255 ký tự, `price = 50000` | 1. Import file CSV<br>2. Kiểm tra CSDL | Import thành công sản phẩm tên 255 ký tự *(FR-16, FR-15)* | High | Not executed | Cận trên độ dài tên |
| **TC-FR16-BV-011** | `name` length = 256 ($max + 1$) | Invalid Boundary | Tên sản phẩm trong CSV có 256 ký tự | 1. Import file CSV<br>2. Kiểm tra phản hồi | Báo lỗi tên vượt quá giới hạn 255 ký tự, rollback import *(FR-16, FR-15)* | High | Not executed | Vượt giới hạn trên 1 ký tự |

---

## 4. Tổng kết Độ phủ Giá trị Biên (BVA Coverage Summary)

- **Tổng số ca kiểm thử BVA mới bổ sung:** **11 test cases** (`TC-FR16-BV-001` đến `TC-FR16-BV-011`).
- **Phân bố kiểm tra biên:**
  - Biên số lượng dòng (0 dòng, 0 byte, 1 dòng, 2 dòng): 4 test cases.
  - Biên vị trí lỗi kiểm tra Rollback All-or-Nothing (dòng đầu, dòng cuối): 2 test cases.
  - Biên giá trị tiền tệ ($price = 0, 1$): 2 test cases.
  - Biên độ dài chuỗi ký tự tên ($name = 1, 255, 256$): 3 test cases.
- Kết hợp với 13 test cases của Domain Testing (Step 2), tổng số ca kiểm thử thiết kế cho FR-16 là **24 test cases**.
