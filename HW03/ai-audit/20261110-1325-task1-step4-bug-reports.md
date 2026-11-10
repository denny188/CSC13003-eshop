# AI Audit Log — Task 1 (Step 1.4): Structured GUI Bug Reports

- **Declaration:** I use AI tools for the following tasks.
- **Name of the AI tool:** Antigravity Agent (Gemini / Claude) with Skill `hw03-gui-checklist`
- **Date and time:** 2026-11-10 13:25:00 +07:00
- **Task / Step:** Task 1 — Step 1.4: Consolidate the 41 failed checklist items into 12 structured GUI Bug Reports (`HW03/bugs/BUG-GUI-01.md` → `BUG-GUI-12.md`) with embedded screenshots and code references.

---

## 1. Student Prompt

```text
Áp dụng skill hw03-gui-checklist (Bước 4): Từ 41 mục Failed trong báo cáo thực thi 03-execution.md, hãy tổng hợp và viết các báo cáo lỗi chi tiết trong thư mục HW03/bugs/ (từ BUG-GUI-01.md đến BUG-GUI-12.md). Mỗi báo cáo lỗi cần có đầy đủ: Khía cạnh IA, Màn hình & dòng code bị ảnh hưởng, Mức độ nghiêm trọng (Severity), Mã Checklist liên quan, Mô tả lỗi, Các bước tái hiện, Kết quả mong đợi vs. Kết quả thực tế, Ảnh chụp màn hình đính kèm và Đề xuất khắc phục.
```

---

## 2. AI Output Summary

AI đã tổng hợp 41 mục Failed thành **12 báo cáo lỗi độc lập, rõ ràng (`BUG-GUI-01` → `BUG-GUI-12`)** trong `HW03/bugs/`:
- `BUG-GUI-01`: Không nhất quán ngôn ngữ trên `/login` (`"Username"`, `"Sign In"`).
- `BUG-GUI-02`: Sai màu nút hành động tích cực trên `/register` (đỏ) & `/forgot-password` (xanh lá) và độ tương phản chữ gợi ý quá thấp (`2.54:1`).
- `BUG-GUI-03`: Tiêu đề trang `/login` hiển thị sai thành `"Đăng Ký"` và toàn bộ các trang thiếu thẻ `<h1>`.
- `BUG-GUI-04`: Thứ tự phím `Tab` trên `/login` bị đảo lộn do gán cứng `tabIndex={1}` ở nút Submit.
- `BUG-GUI-05`: Toàn bộ trường bắt buộc thiếu dấu `*` cạnh nhãn, thiếu liên kết `<label htmlFor>`/`<input id>` và thiếu `autocomplete`.
- `BUG-GUI-06`: Ô Mật khẩu trên `/login` dùng `type="text"` làm lộ mật khẩu dạng rõ; các ô Email và SĐT dùng sai `type="text"`.
- `BUG-GUI-07`: Thông báo lỗi trên `/login` nằm bên dưới nút Submit; `/profile` & `/forgot-password` dùng `window.alert()` chặn luồng.
- `BUG-GUI-08`: Thanh Navbar không highlight trang hiện hành, thiếu badge giỏ hàng, nút đăng xuất ghi `"Thoát"`, thiếu Breadcrumb và dùng `dangerouslySetInnerHTML`.
- `BUG-GUI-09`: Màn hình `/forgot-password` & `/register` thiếu Step Indicator, thiếu nút `"Quay lại đăng nhập"` ở Bước 1, thiếu trường `"Xác nhận mật khẩu"` và sai nhãn OTP 4 số.
- `BUG-GUI-10`: Trang `/profile` có Empty State thiếu hình minh họa, hủy đơn không có hộp thoại xác nhận, vẫn hiện nút Hủy khi đơn `"Đang giao"` và ngõ cụt khi chưa đăng nhập.
- `BUG-GUI-11`: Regex kiểm tra trên UI bị lỗi logic: từ chối SĐT bắt đầu bằng số `0` và từ chối mật khẩu có ký tự đặc biệt chuẩn.
- `BUG-GUI-12`: Bảng đơn hàng 5 cột bị tràn trên màn hình `375px`, thiếu hỗ trợ Dark Mode / RTL và không khóa nút Submit khi đang gửi API.

---

## 3. Human Review & Verification Decisions

- **Đánh giá của sinh viên:** Sinh viên đã kiểm tra nội dung cả 12 file bug report, xác nhận mỗi bug đều gắn đúng mã `CHK-...`, đúng dòng code trong `frontend-web/src/` và liên kết đúng tới file ảnh chụp màn hình trong `HW03/task1-gui/screenshots/`.
