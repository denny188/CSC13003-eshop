# [BUG-FR03-03] Giao diện thiếu chỉ báo bước (Step Indicator) và thiếu nút "Quay lại đăng nhập" ở Bước 1

- **Feature (FR):** `FR-03: Quên mật khẩu & Đặt lại mật khẩu` & `FR-22: Form Requirements`
- **Môi trường (Environment):** Frontend Web (`http://localhost:5173`)
- **Mức độ nghiêm trọng (Severity):** **Medium** (Vi phạm UX và yêu cầu đặc tả luồng đa bước)
- **Mã Test Case liên quan:** `TC-FR03-DT-001`

---

### 1. Mô tả lỗi (Description)
Tại giao diện Quên mật khẩu (`/forgot-password`), mặc dù quy trình gồm 2 bước (Bước 1: lấy OTP, Bước 2: đặt lại mật khẩu), màn hình hoàn toàn không có thanh chỉ báo tiến trình (Step Indicator) để người dùng biết mình đang ở bước nào (ví dụ "Bước 1 / 2"). Đồng thời, ở Bước 1 không có liên kết hoặc nút bấm "Quay lại đăng nhập".

### 2. Các bước tái hiện (Steps to Reproduce)
1. Truy cập trang `http://localhost:5173/forgot-password`.
2. Quan sát giao diện ở màn hình Bước 1:
   - Chỉ có tiêu đề `<h2>Quên Mật Khẩu</h2>`, một ô nhập Email và nút `Lấy mã OTP`.
   - Hoàn toàn không có Step Indicator.
   - Không có nút/link quay lại trang Đăng nhập (`/login`).

### 3. Kết quả mong đợi (Expected Result)
- Theo đặc tả **FR-03 / Bước 1**:
  - *"Giao diện phải hiển thị chỉ báo bước (Step Indicator) — ví dụ: 'Bước 1 / 2'."*
  - *"Có nút Quay lại đăng nhập."*
- Theo **FR-22**: *"Các form có từ 2 bước trở lên phải có Step Indicator rõ ràng."*

### 4. Kết quả thực tế (Actual Result)
- Giao diện không render Step Indicator ở cả 2 bước.
- Tại Bước 1, không có nút/link để người dùng chuyển hướng về trang Đăng nhập.

### 5. Đề xuất khắc phục (Recommendation)
Thêm thành phần hiển thị bước (ví dụ: `<div className="text-sm text-gray-500 text-center mb-4">Bước {step} / 2</div>`) và bổ sung nút/link quay về login:
```jsx
<Link to="/login" className="block text-center text-blue-600 mt-4 hover:underline">
  ← Quay lại đăng nhập
</Link>
```
