# Báo cáo Kiểm thử Miền giá trị (Domain Testing): FR-20 Mobile (Tính năng Giỏ hàng FR-07)

## 1. Phương pháp & Nguyên tắc Áp dụng
1. Áp dụng kỹ thuật **Phân hoạch tương đương (Equivalence Partitioning)** cho các luồng hành vi nghiệp vụ của Giỏ hàng trên ứng dụng di động React Native (theo FR-07 và FR-20):
   - **Thêm vào giỏ hàng:** Sản phẩm mới chưa có trong giỏ vs Sản phẩm đã có trong giỏ (kiểm tra gộp số lượng và không tạo dòng mới).
   - **Điều chỉnh số lượng:** Tăng/giảm số lượng bằng nút `+`/`-` vs Nhập số lượng trực tiếp trong ô `TextInput`.
   - **Xóa sản phẩm khỏi giỏ:** Hành động bấm nút "Xóa" kèm theo phản hồi của Dialog/Alert xác nhận (Chọn "Đồng ý" xóa vs Chọn "Hủy").
   - **Nhãn hiển thị tổng tiền:** Nhãn hiển thị đúng theo đặc tả ("Tổng cộng") vs Nhãn vi phạm đặc tả ("Tổng tạm tính").
   - **Trạng thái giỏ hàng trống (Empty State):** Giỏ hàng có sản phẩm vs Giỏ hàng rỗng (kiểm tra thông báo và hình minh họa).
   - **Điều hướng và Badge số lượng:** Nút "Tiếp tục mua sắm" quay về trang chủ và badge số lượng trên header.
2. Thiết kế ca kiểm thử theo nguyên tắc **Single-fault** và kiểm thử tương tác người dùng (UI/UX interaction testing trên nền tảng di động).

---

## 2. Bảng Phân hoạch Tương đương & Chọn Giá trị Đại diện (Domain Table)

### 2.1 Thêm sản phẩm vào giỏ (Add to Cart)
| Mã lớp | Mô tả lớp tương đương | Hợp lệ / Không hợp lệ | Giá trị đại diện | Lý giải chọn giá trị (Rationale) |
| :--- | :--- | :---: | :--- | :--- |
| **EC-AD-V1** | Thêm một sản phẩm chưa có trong giỏ hàng | Hợp lệ | Sản phẩm ID 1, số lượng 1 | Tạo một dòng mới trong danh sách giỏ hàng |
| **EC-AD-V2** | Thêm một sản phẩm đã có sẵn trong giỏ hàng | Hợp lệ | Sản phẩm ID 1 đã có trong giỏ, bấm thêm tiếp | Kiểm tra quy tắc gộp số lượng, không tạo dòng thứ hai trùng lặp (FR-07) |
| **EC-AD-I1** | Thêm sản phẩm với số lượng không hợp lệ ($\le 0$) | Không hợp lệ | Số lượng nhập = 0 hoặc -1 | Hệ thống phải chuẩn hóa hoặc báo lỗi, tối thiểu là 1 |

---

### 2.2 Điều chỉnh số lượng trong giỏ (Quantity Manipulation)
| Mã lớp | Mô tả lớp tương đương | Hợp lệ / Không hợp lệ | Giá trị đại diện | Lý giải chọn giá trị (Rationale) |
| :--- | :--- | :---: | :--- | :--- |
| **EC-QT-V1** | Tăng/giảm số lượng thông qua nút bấm `+` / `-` | Hợp lệ | Bấm nút `+` hoặc `-` | Giao diện di động phải có nút `+`/`-` để thao tác thuận tiện |
| **EC-QT-V2** | Nhập trực tiếp số lượng nguyên dương hợp lệ | Hợp lệ | Nhập `3` vào ô số lượng | Cập nhật chính xác số lượng = 3, thành tiền và tổng tiền tính lại tương ứng |
| **EC-QT-I1** | Nhập số lượng không phải số hoặc số âm/bằng 0 | Không hợp lệ | Nhập `0` hoặc `"abc"` | Hệ thống không được crash, chuẩn hóa về 1 hoặc giữ giá trị cũ |

---

### 2.3 Xóa sản phẩm khỏi giỏ & Dialog xác nhận (Removal with Confirmation)
| Mã lớp | Mô tả lớp tương đương | Hợp lệ / Không hợp lệ | Giá trị đại diện | Lý giải chọn giá trị (Rationale) |
| :--- | :--- | :---: | :--- | :--- |
| **EC-DL-V1** | Bấm Xóa -> Hiển thị Dialog xác nhận -> Chọn Đồng ý | Hợp lệ | Xác nhận xóa trên dialog | Bắt buộc có dialog xác nhận trước khi xóa; xóa xong sản phẩm biến mất khỏi giỏ |
| **EC-DL-V2** | Bấm Xóa -> Hiển thị Dialog xác nhận -> Chọn Hủy | Hợp lệ | Bấm nút "Hủy" trên dialog | Sản phẩm vẫn giữ nguyên trong giỏ hàng, không bị xóa |
| **EC-DL-I1** | Bấm Xóa nhưng hệ thống xóa ngay không qua Dialog | Không hợp lệ | Xóa tức thì không có confirmation | Vi phạm trực tiếp quy tắc: *"Nút Xóa sản phẩm phải có dialog xác nhận trước khi thực hiện"* (FR-07) |

---

### 2.4 Nhãn hiển thị Tổng tiền (Total Label Compliance)
| Mã lớp | Mô tả lớp tương đương | Hợp lệ / Không hợp lệ | Giá trị đại diện | Lý giải chọn giá trị (Rationale) |
| :--- | :--- | :---: | :--- | :--- |
| **EC-LB-V1** | Nhãn hiển thị chính xác: **"Tổng cộng"** | Hợp lệ | Text: `"Tổng cộng: ... ₫"` | Đúng tuyệt đối theo yêu cầu FR-07 |
| **EC-LB-I1** | Nhãn hiển thị sai lệch: **"Tổng tạm tính"** | Không hợp lệ | Text: `"Tổng tạm tính: ... ₫"` | Vi phạm quy định FR-07: *"không phải 'Tổng tạm tính'"* |

---

### 2.5 Trạng thái Giỏ hàng trống & Điều hướng (Empty State & Navigation)
| Mã lớp | Mô tả lớp tương đương | Hợp lệ / Không hợp lệ | Giá trị đại diện | Lý giải chọn giá trị (Rationale) |
| :--- | :--- | :---: | :--- | :--- |
| **EC-ES-V1** | Giỏ hàng trống có cả hình minh họa và thông báo | Hợp lệ | Icon/Hình ảnh giỏ hàng + Text thông báo | Tuân thủ FR-07: *"Giỏ hàng trống phải có hình minh họa và thông báo rõ ràng"* |
| **EC-ES-I1** | Giỏ hàng trống chỉ có text đơn thuần, thiếu hình minh họa | Không hợp lệ | Chỉ có `<Text>` thông báo | Vi phạm yêu cầu phải có hình minh họa |
| **EC-NV-V1** | Nút quay về trang chủ có nhãn chuẩn **"Tiếp tục mua sắm"** | Hợp lệ | Nút: `"Tiếp tục mua sắm"` | Tuân thủ đặc tả FR-07 |
| **EC-NV-I1** | Nút quay về trang chủ có nhãn sai chuẩn (vd: *"← Mua tiếp"*) | Không hợp lệ | Nút: `"← Mua tiếp"` | Không đồng bộ với đặc tả chuẩn FR-07 |

---

## 3. Danh sách Ca Kiểm thử Chi tiết (Domain Test Cases)

| Mã Test Case | Lớp tương đương bao phủ | Điều kiện tiên quyết (Preconditions) | Các bước thực hiện (Steps) | Dữ liệu kiểm thử (Test Data) | Kết quả mong đợi (Expected Result) | Độ ưu tiên | Trạng thái |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: | :---: |
| **TC-MOB07-DT-001** | **EC-AD-V1** (Thêm sản phẩm mới) | Màn hình Home trên Mobile | 1. Chọn sản phẩm $A$<br>2. Bấm "Thêm vào giỏ"<br>3. Mở màn hình Giỏ hàng | Sản phẩm $A$ (Giá: 100,000 ₫) | Giỏ hàng hiển thị 1 dòng sản phẩm $A$ với số lượng 1, thành tiền 100,000 ₫, HTTP 200 *(FR-07)* | High | Not executed |
| **TC-MOB07-DT-002** | **EC-AD-V2** (Gộp sản phẩm trùng lặp) | Giỏ hàng đã có sản phẩm $A$ (SL = 1) | 1. Quay lại trang chi tiết sản phẩm $A$<br>2. Nhập số lượng 2 và bấm "Thêm vào giỏ"<br>3. Mở Giỏ hàng kiểm tra | Sản phẩm $A$, số lượng thêm = 2 | Số lượng của dòng sản phẩm $A$ tăng lên thành 3. **Không tạo thêm dòng mới** *(FR-07: Thêm cùng một sản phẩm vào giỏ sẽ tăng số lượng, không tạo dòng mới)* | Critical | Not executed |
| **TC-MOB07-DT-003** | **EC-QT-V1** (Nút điều chỉnh +/-) | Giỏ hàng có sản phẩm | 1. Kiểm tra giao diện dòng sản phẩm trong giỏ hàng | Dòng sản phẩm trong giỏ | Có nút tăng (`+`) và giảm (`-`) số lượng trực quan, bấm vào tăng/giảm số lượng 1 đơn vị *(FR-07: Số lượng có nút +/- để chỉnh)* | High | Not executed |
| **TC-MOB07-DT-004** | **EC-QT-V2** (Nhập số lượng trực tiếp) | Giỏ hàng có sản phẩm $B$ (SL = 1) | 1. Nhấp vào ô số lượng<br>2. Nhập số `5`<br>3. Quan sát số lượng và thành tiền | Nhập `5` | Số lượng cập nhật chính xác là `5`. Thành tiền cập nhật $= price \times 5$. Tổng tiền tính lại chuẩn xác | Critical | Not executed |
| **TC-MOB07-DT-005** | **EC-DL-V1, EC-DL-I1** (Dialog xác nhận khi xóa sản phẩm) | Giỏ hàng đang có 1 sản phẩm | 1. Bấm nút "Xóa" trên dòng sản phẩm<br>2. Quan sát phản hồi của ứng dụng | Bấm nút "Xóa" | Hệ thống **bắt buộc phải hiển thị Dialog/Alert xác nhận** trước khi xóa *(FR-07: Nút Xóa sản phẩm phải có dialog xác nhận trước khi thực hiện)* | Critical | Not executed |
| **TC-MOB07-DT-006** | **EC-DL-V2** (Hủy xóa trên Dialog) | Dialog xác nhận xóa đang mở | 1. Bấm nút "Hủy" trên Dialog xác nhận<br>2. Quan sát giỏ hàng | Bấm "Hủy" | Dialog đóng lại, sản phẩm vẫn giữ nguyên trong giỏ hàng, không bị xóa *(FR-07)* | High | Not executed |
| **TC-MOB07-DT-007** | **EC-LB-V1, EC-LB-I1** (Nhãn hiển thị Tổng tiền) | Giỏ hàng có ít nhất 1 sản phẩm | 1. Mở màn hình Giỏ hàng<br>2. Đọc nhãn hiển thị tại dòng tổng tiền | Giỏ hàng có sản phẩm | Nhãn hiển thị chính xác là **"Tổng cộng"** (Nghiêm cấm nhãn *"Tổng tạm tính"*) *(FR-07: Tổng tiền hiển thị nhãn chính xác: "Tổng cộng" (không phải "Tổng tạm tính"))* | Critical | Not executed |
| **TC-MOB07-DT-008** | **EC-ES-V1, EC-ES-I1** (Hình minh họa giỏ hàng trống) | Giỏ hàng hoàn toàn trống (0 sản phẩm) | 1. Mở màn hình Giỏ hàng khi chưa mua gì<br>2. Quan sát giao diện màn hình | Giỏ hàng trống | Màn hình hiển thị rõ ràng: **Hình minh họa** (hình ảnh/icon giỏ hàng trống) và thông điệp: "Giỏ hàng của bạn đang trống" *(FR-07: Giỏ hàng trống phải có hình minh họa và thông báo rõ ràng)* | High | Not executed |
| **TC-MOB07-DT-009** | **EC-NV-V1, EC-NV-I1** (Nút Tiếp tục mua sắm) | Giỏ hàng đang hiển thị | 1. Kiểm tra các nút điều hướng trên màn hình Giỏ hàng | Nút quay về trang chủ | Có nút/liên kết hiển thị đúng nhãn: **"Tiếp tục mua sắm"**; bấm vào chuyển về trang chủ an toàn *(FR-07)* | Medium | Not executed |
| **TC-MOB07-DT-010** | **Badge số lượng trên Navbar** | Màn hình bất kỳ trên Mobile | 1. Thêm 2 sản phẩm khác nhau vào giỏ (tổng 3 món)<br>2. Quan sát biểu tượng giỏ hàng trên thanh điều hướng | Giỏ hàng có 3 món hàng | Thanh điều hướng hiển thị badge số lượng sản phẩm rõ ràng, đồng bộ số lượng tức thì *(FR-23)* | High | Not executed |

---

## 4. Ma trận Độ phủ Lớp tương đương (Coverage Matrix)

| Lớp tương đương | Mã Test Case bao phủ |
| :--- | :--- |
| **EC-AD-V1** | TC-MOB07-DT-001 |
| **EC-AD-V2** | TC-MOB07-DT-002 |
| **EC-AD-I1** | TC-MOB07-DT-001, TC-MOB07-DT-002 |
| **EC-QT-V1** | TC-MOB07-DT-003 |
| **EC-QT-V2** | TC-MOB07-DT-004 |
| **EC-QT-I1** | TC-MOB07-DT-004 |
| **EC-DL-V1** | TC-MOB07-DT-005 |
| **EC-DL-V2** | TC-MOB07-DT-006 |
| **EC-DL-I1** | TC-MOB07-DT-005 |
| **EC-LB-V1** | TC-MOB07-DT-007 |
| **EC-LB-I1** | TC-MOB07-DT-007 |
| **EC-ES-V1** | TC-MOB07-DT-008 |
| **EC-ES-I1** | TC-MOB07-DT-008 |
| **EC-NV-V1** | TC-MOB07-DT-009 |
| **EC-NV-I1** | TC-MOB07-DT-009 |

**Tổng số test case Domain Testing:** **10 test cases** (Bao phủ 100% các phân vùng chức năng và ràng buộc đặc tả của FR-07 trên Mobile).
