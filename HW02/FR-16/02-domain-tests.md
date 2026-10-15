# Báo cáo Kiểm thử Miền giá trị (Domain Testing): FR-16 (Import Sản phẩm từ CSV)

## 1. Phương pháp & Nguyên tắc Áp dụng
1. Áp dụng kỹ thuật **Phân hoạch tương đương (Equivalence Partitioning)** cho từng khía cạnh của chức năng Import sản phẩm từ file CSV:
   - **Định dạng file & Header:** File `.csv` chuẩn, file sai đuôi mở rộng (`.txt`, `.xlsx`), file sai cấu trúc header.
   - **Đặc tả RFC 4180:** Xử lý trường dữ liệu chứa dấu phẩy được bao bọc bởi dấu nháy kép `""` vs trường không có nháy kép.
   - **Tính hợp lệ của dữ liệu từng dòng:** `name` hợp lệ vs `name` rỗng; `price > 0` vs `price <= 0` hoặc sai kiểu; `category_id` tồn tại vs không tồn tại.
   - **Quy tắc giao dịch nguyên tử (Atomic Transaction / Rollback All-or-Nothing):** File 100% dòng hợp lệ vs File có 1 dòng bất kỳ bị lỗi (ở đầu, giữa hoặc cuối file).
   - **Quyền hạn truy cập (`actor_role`):** Token Admin hợp lệ vs User thường vs Không gửi token.
2. Thiết kế ca kiểm thử theo nguyên tắc **Single-fault** và **Multi-fault** (kiểm thử giao dịch nguyên tử):
   - Ca kiểm thử Happy Path import thành công nhiều dòng.
   - Các ca kiểm thử vi phạm định dạng file, header, và cú pháp RFC 4180.
   - Các ca kiểm thử then chốt về tính nguyên tử: File có $N$ dòng, trong đó có 1 dòng vi phạm ràng buộc thì **cả $N$ dòng đều không được lưu vào CSDL** (Rollback 100%).
   - Ca kiểm thử phân quyền truy cập API import.

---

## 2. Bảng Phân hoạch Tương đương & Chọn Giá trị Đại diện (Domain Table)

### 2.1 Định dạng file & Cấu trúc Header
| Mã lớp | Mô tả lớp tương đương | Hợp lệ / Không hợp lệ | Giá trị đại diện | Lý giải chọn giá trị (Rationale) |
| :--- | :--- | :---: | :--- | :--- |
| **EC-FL-V1** | File có đuôi mở rộng `.csv` | Hợp lệ | `products_valid.csv` | Định dạng chuẩn theo yêu cầu FR-16 |
| **EC-FL-I1** | File không phải định dạng `.csv` | Không hợp lệ | `products.txt`, `products.xlsx` | Kiểm tra bộ lọc định dạng file tải lên |
| **EC-HD-V1** | Header chuẩn `name,price,description,imageUrl,category_id` | Hợp lệ | Dòng 1: `name,price,description,imageUrl,category_id` | Cấu trúc header chuẩn quy định trong đặc tả |
| **EC-HD-I1** | Header sai tên cột hoặc thiếu cột bắt buộc | Không hợp lệ | Dòng 1: `title,cost,desc` | Kiểm tra hệ thống nhận diện đúng cấu trúc cột |

---

### 2.2 Xử lý cú pháp RFC 4180 (Dấu phẩy trong nháy kép)
| Mã lớp | Mô tả lớp tương đương | Hợp lệ / Không hợp lệ | Giá trị đại diện | Lý giải chọn giá trị (Rationale) |
| :--- | :--- | :---: | :--- | :--- |
| **EC-RF-V1** | Trường chứa dấu phẩy được bọc trong cặp nháy kép `""` | Hợp lệ | `"Laptop gaming, màn hình 15.6 inch, 144Hz"` | Chuẩn RFC 4180 cho phép trường text chứa dấu phẩy |
| **EC-RF-I1** | Trường chứa dấu phẩy nhưng không có nháy kép | Không hợp lệ | `Laptop gaming, màn hình 15.6 inch, 144Hz` | Dẫn đến thừa số cột phân tách bởi dấu phẩy |

---

### 2.3 Tính hợp lệ của dòng dữ liệu (Row Validity)
| Mã lớp | Mô tả lớp tương đương | Hợp lệ / Không hợp lệ | Giá trị đại diện | Lý giải chọn giá trị (Rationale) |
| :--- | :--- | :---: | :--- | :--- |
| **EC-RW-V1** | Dòng chứa đầy đủ `name` hợp lệ, `price > 0`, `category_id` tồn tại | Hợp lệ | `Chuột không dây,250000,Mô tả chuột,,1` | Dòng dữ liệu mẫu chuẩn |
| **EC-RW-I1** | Dòng có trường `name` bị để trống | Không hợp lệ | `,150000,Mô tả,,1` | Vi phạm ràng buộc name không được rỗng |
| **EC-RW-I2** | Dòng có `price = 0` hoặc số âm (`price < 0`) | Không hợp lệ | `Bàn phím cơ,-100000,Mô tả,,1` | Vi phạm ràng buộc giá phải là số dương |
| **EC-RW-I3** | Dòng có `price` là chuỗi chữ không phải số | Không hợp lệ | `Bàn phím cơ,mien_phi,Mô tả,,1` | Sai kiểu dữ liệu số |
| **EC-RW-I4** | Dòng có `category_id` không tồn tại trong CSDL | Không hợp lệ | `Tai nghe,300000,Mô tả,,9999` | Khóa ngoại danh mục không tồn tại |

---

### 2.4 Tính nguyên tử của giao dịch (Atomic Transaction & Rollback)
| Mã lớp | Mô tả lớp tương đương | Hợp lệ / Không hợp lệ | Giá trị đại diện | Lý giải chọn giá trị (Rationale) |
| :--- | :--- | :---: | :--- | :--- |
| **EC-TX-V1** | Tất cả các dòng trong file đều hợp lệ | Hợp lệ | File 5 dòng đều hợp lệ | Toàn bộ 5 sản phẩm được lưu vào CSDL |
| **EC-TX-I1** | File có 1 dòng bị lỗi nằm ở dòng cuối cùng | Không hợp lệ | 4 dòng đầu hợp lệ, dòng thứ 5 có `price = -100` | Kiểm tra rollback all-or-nothing: 4 dòng đầu KHÔNG được phép lưu vào CSDL |
| **EC-TX-I2** | File có 1 dòng bị lỗi nằm ở dòng giữa file | Không hợp lệ | Dòng 1 hợp lệ, dòng 2 `name` rỗng, dòng 3 hợp lệ | Toàn bộ giao dịch phải bị hủy, CSDL không thêm dòng nào |

---

### 2.5 Quyền hạn truy cập (`actor_role`)
| Mã lớp | Mô tả lớp tương đương | Hợp lệ / Không hợp lệ | Giá trị đại diện | Lý giải chọn giá trị (Rationale) |
| :--- | :--- | :---: | :--- | :--- |
| **EC-RO-V1** | Tài khoản Admin thực hiện import | Hợp lệ | JWT Token của `admin@eshop.com` | Được phép truy cập tính năng import |
| **EC-RO-I1** | Tài khoản User thông thường gọi import | Không hợp lệ | JWT Token của `test@eshop.com` | Bị từ chối với HTTP 403 Forbidden |
| **EC-RO-I2** | Khách vãng lai không có token gọi import | Không hợp lệ | Không gửi Header Authorization | Bị từ chối với HTTP 401 Unauthorized |

---

## 3. Danh sách Ca Kiểm thử Chi tiết (Domain Test Cases)

| Mã Test Case | Lớp tương đương bao phủ | Điều kiện tiên quyết (Preconditions) | Các bước thực hiện (Steps) | Dữ liệu kiểm thử (Test Data) | Kết quả mong đợi (Expected Result) | Độ ưu tiên | Trạng thái |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: | :---: |
| **TC-FR16-DT-001** | **EC-FL-V1, EC-HD-V1, EC-RW-V1, EC-TX-V1, EC-RO-V1** (Happy Path) | Đăng nhập Admin, trang Web Admin | 1. Tải lên file CSV chứa 3 sản phẩm hợp lệ<br>2. Bấm nút Import sản phẩm<br>3. Kiểm tra thông báo và danh sách sản phẩm | File CSV có header chuẩn và 3 dòng dữ liệu hợp lệ | Import thành công 3/3 sản phẩm, CSDL lưu đủ 3 sản phẩm mới, hiển thị báo cáo thành công *(FR-16)* | High | Not executed |
| **TC-FR16-DT-002** | **EC-RF-V1** (RFC 4180 Description có dấu phẩy) | Đăng nhập Admin | 1. Tải lên file CSV có cột description chứa dấu phẩy được bọc trong cặp nháy kép `""`<br>2. Bấm Import | `Laptop X,15000000,"RAM 16GB, SSD 512GB, Intel i7",,1` | Parse chính xác RFC 4180, không bị lệch cột, import thành công mô tả đầy đủ có dấu phẩy *(FR-16)* | Critical | Not executed |
| **TC-FR16-DT-003** | **EC-FL-I1** (Upload file không phải CSV) | Đang ở form Import | 1. Chọn file có đuôi `.txt` hoặc `.xlsx`<br>2. Bấm tải lên | File `test_products.txt` | Hệ thống từ chối nhận file, thông báo định dạng file phải là `.csv` *(FR-16)* | High | Not executed |
| **TC-FR16-DT-004** | **EC-HD-I1** (Header sai quy chuẩn) | Đang ở form Import | 1. Tải lên file CSV có header `title,cost,desc`<br>2. Bấm Import | File CSV có header không khớp chuẩn quy định | Báo lỗi cấu trúc file/header không hợp lệ, không thực hiện import *(FR-16)* | High | Not executed |
| **TC-FR16-DT-005** | **EC-RW-I1** (Dòng có name rỗng) | Đang ở form Import | 1. Tải lên file CSV chứa 1 dòng có `name` rỗng | `,200000,Mô tả SP,,1` | Báo lỗi dòng thiếu tên sản phẩm, từ chối import *(FR-16)* | High | Not executed |
| **TC-FR16-DT-006** | **EC-RW-I2** (Dòng có giá <= 0) | Đang ở form Import | 1. Tải lên file CSV có dòng `price = -50000` | `Sản phẩm Giá Âm,-50000,Mô tả,,1` | Báo lỗi giá phải là số dương (> 0), từ chối import *(FR-16)* | Critical | Not executed |
| **TC-FR16-DT-007** | **EC-RW-I3** (Dòng có giá là chuỗi ký tự) | Đang ở form Import | 1. Tải lên file CSV có dòng `price = "mien_phi"` | `Sản phẩm Free,mien_phi,Mô tả,,1` | Báo lỗi định dạng giá không hợp lệ *(FR-16)* | Medium | Not executed |
| **TC-FR16-DT-008** | **EC-RW-I4** (Dòng có category_id không tồn tại) | Đang ở form Import | 1. Tải lên file CSV có dòng `category_id = 9999` | `Sản phẩm Lạ,100000,Mô tả,,9999` | Báo lỗi danh mục không hợp lệ, không import *(FR-16, FR-15)* | High | Not executed |
| **TC-FR16-DT-009** | **EC-TX-I1** (Rollback All-or-Nothing khi lỗi ở dòng cuối) | Đang ở form Import, kiểm tra số lượng SP ban đầu trong CSDL ($N_0$) | 1. Tải file CSV gồm 3 dòng: 2 dòng đầu hợp lệ, dòng thứ 3 có `price = -1000`<br>2. Bấm Import<br>3. Kiểm tra số lượng SP trong CSDL | Dòng 1 & 2 hợp lệ, Dòng 3 giá âm | Toàn bộ thao tác bị **Rollback**: 2 dòng đầu KHÔNG được thêm vào CSDL. CSDL vẫn giữ nguyên $N_0$ sản phẩm. Báo cáo rõ dòng 3 bị lỗi *(FR-16: Nếu có lỗi ở bất kỳ dòng nào, toàn bộ import phải được rollback)* | Critical | Not executed |
| **TC-FR16-DT-010** | **EC-TX-I2** (Rollback All-or-Nothing khi lỗi ở dòng giữa) | Đang ở form Import | 1. Tải file CSV: Dòng 1 hợp lệ, Dòng 2 rỗng tên, Dòng 3 hợp lệ<br>2. Bấm Import<br>3. Kiểm tra CSDL | Dòng 1 & 3 hợp lệ, Dòng 2 rỗng tên | Toàn bộ giao dịch bị Rollback, CSDL không tăng thêm sản phẩm nào *(FR-16)* | Critical | Not executed |
| **TC-FR16-DT-011** | **Báo cáo kết quả chi tiết (Summary Reporting)** | Đang ở form Import | 1. Import file CSV chứa các dòng lỗi<br>2. Đọc thông báo phản hồi trên UI/API | File CSV có dòng lỗi | Hiển thị báo cáo rõ ràng: số dòng thành công, số dòng lỗi và lý do chi tiết từng dòng *(FR-16)* | High | Not executed |
| **TC-FR16-DT-012** | **EC-RO-I1** (User thường gọi API import) | Đăng nhập tài khoản User | 1. Gửi `POST /api/admin/import-products` kèm token User | Token của `test@eshop.com` | Trả về 403 Forbidden, từ chối import *(FR-12, SEC-03)* | High | Not executed |
| **TC-FR16-DT-013** | **EC-RO-I2** (Khách vãng lai không token gọi import) | Không gửi token | 1. Gửi `POST /api/admin/import-products` không có Header Authorization | Không có header Authorization | Trả về 401 Unauthorized *(SEC-02)* | High | Not executed |

---

## 4. Ma trận Độ phủ Lớp tương đương (Coverage Matrix)

| Lớp tương đương | Mã Test Case bao phủ |
| :--- | :--- |
| **EC-FL-V1** | TC-FR16-DT-001, TC-FR16-DT-002 |
| **EC-FL-I1** | TC-FR16-DT-003 |
| **EC-HD-V1** | TC-FR16-DT-001, TC-FR16-DT-002 |
| **EC-HD-I1** | TC-FR16-DT-004 |
| **EC-RF-V1** | TC-FR16-DT-002 |
| **EC-RF-I1** | TC-FR16-DT-004 |
| **EC-RW-V1** | TC-FR16-DT-001, TC-FR16-DT-002 |
| **EC-RW-I1** | TC-FR16-DT-005, TC-FR16-DT-010 |
| **EC-RW-I2** | TC-FR16-DT-006, TC-FR16-DT-009 |
| **EC-RW-I3** | TC-FR16-DT-007 |
| **EC-RW-I4** | TC-FR16-DT-008 |
| **EC-TX-V1** | TC-FR16-DT-001 |
| **EC-TX-I1** | TC-FR16-DT-009 |
| **EC-TX-I2** | TC-FR16-DT-010 |
| **EC-RO-V1** | TC-FR16-DT-001, TC-FR16-DT-002 |
| **EC-RO-I1** | TC-FR16-DT-012 |
| **EC-RO-I2** | TC-FR16-DT-013 |

**Tổng số test case Domain Testing:** **13 test cases** (Bao phủ đầy đủ các phân vùng file, header, chuẩn RFC 4180, validation từng dòng, tính nguyên tử all-or-nothing và phân quyền bảo mật).
