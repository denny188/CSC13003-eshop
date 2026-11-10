# AI Audit Log — Task 1 (Step 1.1): Initial AI-Generated GUI Checklist

- **Declaration:** I use AI tools for the following tasks.
- **Name of the AI tool:** Antigravity Agent (Gemini 4 Argon/ Claude Opus 5.5) with Skill `hw03-gui-checklist`
- **Date and time:** 2026-11-10 10:30:00 +07:00
- **Task / Step:** Task 1 — Step 1.1: Generate an initial 40-item GUI Checklist covering `IA-01`, `IA-02`, `IA-03`, and `IA-04` across Screens S1 (`/register`), S2 (`/login`, `/forgot-password`), and S3 (`/profile`).

---

## 1. Student Prompt

```text
Áp dụng skill hw03-gui-checklist (Bước 1): Dựa trên đặc tả README.md (cụ thể là các mục FR-01, FR-02, FR-03, FR-04, FR-11 và yêu cầu giao diện FR-21, FR-22, FR-23, FR-24), hãy thiết kế bộ GUI Checklist ban đầu gồm 40 mục (CHK-001 đến CHK-040) cho 3 cụm màn hình của tôi:
- S1: Đăng ký tài khoản (/register)
- S2: Đăng nhập (/login) & Quên mật khẩu (/forgot-password)
- S3: Hồ sơ cá nhân & Lịch sử đơn hàng (/profile) kèm thanh điều hướng Header chung.
Phân bổ đều qua 4 khía cạnh IA-01 (General UI standards), IA-02 (Forms), IA-03 (Navigation), và IA-04 (Feedback / state). Lưu kết quả vào HW03/task1-gui/01-ai-initial-checklist.md.
```

---

## 2. AI Output Summary

AI đã phân tích đặc tả `FR-21`..`FR-24` kết hợp với các yêu cầu nghiệp vụ `FR-01`..`FR-04`, `FR-11` và sinh ra bảng 40 mục kiểm tra (`CHK-001` → `CHK-040`) trong `HW03/task1-gui/01-ai-initial-checklist.md`:
- **IA-01 (General UI standards):** 10 mục (`CHK-001` → `CHK-010`) kiểm tra ngôn ngữ tiếng Việt, màu sắc nút bấm (xanh dương cho tích cực, đỏ cho hủy bỏ), định dạng tiền tệ `₫`, thẻ `<h1>` duy nhất mỗi trang, và thứ tự Tab Order.
- **IA-02 (Forms):** 12 mục (`CHK-011` → `CHK-022`) kiểm tra dấu `*` ở trường bắt buộc, `type="email"`, `type="password"`, vị trí thông báo lỗi phía trên nút Submit, Step Indicator ở form 2 bước, và trường Xác nhận mật khẩu.
- **IA-03 (Navigation):** 9 mục (`CHK-023` → `CHK-031`) kiểm tra highlight trang hiện tại trên Navbar, badge số lượng giỏ hàng, nhãn nút `"Đăng xuất"`, Breadcrumb ở trang con, và nút `"Quay lại đăng nhập"`.
- **IA-04 (Feedback / state):** 9 mục (`CHK-032` → `CHK-040`) kiểm tra Empty State của lịch sử đơn hàng, hộp thoại xác nhận khi hủy đơn, màu sắc/nhãn tiếng Việt của trạng thái đơn hàng, và thông báo phản hồi.

---

## 3. Human Review & Verification Decisions

- **Đánh giá của sinh viên:** Bộ 40 mục ban đầu do AI sinh ra đã bám sát các quy tắc tường minh được viết sẵn trong văn bản `README.md` (`FR-21` → `FR-24`).
- **Điểm hạn chế phát hiện được (sẽ bổ sung ở Bước 1.2 — `02-human-review.md`):**
  Vì AI chủ yếu đọc văn bản đặc tả (`README.md`), AI bị thiên lệch theo đặc tả (*spec-centrism bias*) và **bỏ sót 15 khía cạnh phi chức năng / chuẩn công nghiệp về UI/UX & Accessibility (WCAG 2.1)** không được ghi từng chữ trong `README.md`, bao gồm:
  1. Liên kết ngữ nghĩa `<label htmlFor>` với `<input id>` (nhấp vào nhãn không focus được vào ô nhập).
  2. Độ tương phản màu sắc văn bản phụ (`text-gray-400` trên nền trắng ở `/register` chỉ đạt ~2.54:1, không đạt chuẩn WCAG AA 4.5:1).
  3. Nút ẩn/hiện mật khẩu (Password Visibility Toggle) giúp giảm lỗi gõ nhầm mật khẩu phức tạp.
  4. Thuộc tính `autocomplete` của trình duyệt cho các trường nhạy cảm (`email`, `current-password`, `new-password`, `tel`).
  5. Giao diện chế độ tối (Dark Mode / `prefers-color-scheme: dark`).
  6. Bố cục đáp ứng trên màn hình hẹp di động (`375px` — bảng Lịch sử đơn hàng 5 cột bị tràn ngang).
  7. Khả năng phóng to 200% (Browser Zoom) & tràn văn bản khi tên người dùng/địa chỉ quá dài.
  8. Lỗ hổng hiển thị HTML thô (`dangerouslySetInnerHTML`) làm vỡ giao diện thanh Navbar khi tên người dùng chứa ký tự `<` hoặc thẻ HTML.
  9. Trạng thái Loading / khóa nút Submit (`disabled`) khi đang gửi request mạng để chống bấm đúp (Double Submit).
