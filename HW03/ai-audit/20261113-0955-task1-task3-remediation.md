# AI Audit Log — Khắc phục Nợ kỹ thuật Task 1 & Phần đầu Task 3

- **Declaration:** I use AI tools for the following tasks.
- **Name of the AI tool:** Antigravity Agent
- **Date and time:** 2026-11-13 09:55:00 +07:00
- **Task / Step:** Audit & Remediation of Technical Debt across Task 1 (`HW03/task1-gui/`, `HW03/bugs/`, GitHub Issues `#29`–`#40`, `HW03/ai-audit/`) and completed portion of Task 3 (`HW03/task3-cross-platform/`).

---

## 1. Student Prompt

```text
oke sửa tất cả đi
```
*(Phê duyệt khắc phục toàn bộ các nợ kỹ thuật đã rà soát ở lượt trước trên Task 1, GitHub Issues #29–#40, file Excel, và phần đầu Task 3 trước khi tiếp tục BrowserStack/LambdaTest & Task 2).*

---

## 2. AI Output Summary

1. **Sửa bằng chứng ảnh chụp Task 1 (`HW03/task1-gui/screenshots/`):**
   - Chụp lại `BUG-GUI-04_tab_order.png` bằng thao tác nhấn phím `Tab` thật qua CDP (`Input.dispatchKeyEvent`) để trình duyệt tự vẽ viền tiêu điểm thực tế, loại bỏ CSS outline giả.
   - Chụp lại `BUG-GUI-08_navbar.png` qua luồng người dùng thật (đăng nhập tài khoản sinh viên `23120233@student.hcmus.edu.vn`, đặt tên dài chứa thẻ HTML `<b>...</b>` qua `PUT /api/users/me`, bấm `"Thêm vào giỏ"` trên `/` để giỏ hàng có sản phẩm rồi chuyển sang `/profile`), chứng minh đồng thời lỗi tràn chữ dài (`CHK-046`), hiển thị HTML từ `dangerouslySetInnerHTML` (`CHK-055`) và thiếu badge giỏ hàng (`CHK-024`), sau đó khôi phục lại tên sạch trong CSDL.
   - Tách ảnh trùng MD5 và chụp bổ sung 2 ảnh minh chứng còn thiếu: `BUG-GUI-12_dark_mode.png` (emulated `prefers-color-scheme: dark` trên `/`) và `BUG-GUI-12_rtl.png` (`dir="rtl"` trên `/profile`), đồng thời cập nhật `BUG-GUI-10_profile.png`, `BUG-GUI-03_headings.png`, `BUG-GUI-05_form_labels.png`.
2. **Chuẩn hóa tài liệu thực thi & Excel Task 1:**
   - Chuyển đổi toàn bộ 43 đường dẫn tuyệt đối `file:///d:/...` trong [`HW03/task1-gui/03-execution.md`](../task1-gui/03-execution.md) thành đường dẫn tương đối (`./HW03_GUI_Checklist.xlsx`, `./screenshots/*.png`).
   - Sửa bảng tổng hợp đếm số bug theo từng nhóm (`IA-01` = 6, `IA-02` = 4, `IA-03` = 3, `IA-04` = 6; quy về 12 Bug duy nhất), làm rõ ranh giới `CHK-040` vs `CHK-052`, cập nhật kết quả kiểm chứng thực tế `CHK-051` (`:focus-visible` với phím `Tab` thật) và đồng bộ tham chiếu `CHK-042`, `CHK-044`, `CHK-053`.
   - Tái tạo [`HW03/task1-gui/HW03_GUI_Checklist.xlsx`](../task1-gui/HW03_GUI_Checklist.xlsx) với bảng `Test_Summary` dùng công thức Excel động (`COUNTIFS`, `=C5+D5`, `SUM`, `=F5/E5` định dạng `0.00%`) và giữ nguyên văn bản tiếng Việt thuần sạch sẽ.
3. **Đồng bộ Bug Reports (`HW03/bugs/`) & Sửa tất cả 12 GitHub Issues (`#29`–`#40`):**
   - Cập nhật `BUG-GUI-08.md` (rõ phạm vi `Header/Navbar & Login.jsx:50` và bước tái hiện) và `BUG-GUI-12.md` (gắn đủ 3 ảnh chụp Responsive `375px`, Dark Mode, RTL).
   - Cập nhật qua GitHub REST API toàn bộ Issues `#29`–`#40`: gắn lại đúng nhãn `severity: critical / high / medium / low` theo mức nghiêm trọng trong `BUG-GUI-01..12.md`, xóa dòng tự liên kết thừa `- **GitHub Issue Link:**` ở cuối nội dung Issue, và cập nhật URL ảnh RAW `?v=3`.
4. **Khắc phục phần đầu Task 3 (`HW03/task3-cross-platform/`):**
   - Chụp lại 3 ảnh **cửa sổ Google Chrome `157.0.8103.78` thật trên Windows 11** (`TASK3_CHROME_WINDOWS_login.png`, `register.png`, `profile.png`) qua `PrintWindow` DPI-aware với thanh tiêu đề cửa sổ, thanh địa chỉ `localhost:5173`, thông tin `23120233 - Nguyễn Lê Hữu Điền` trong form/hồ sơ, 2 đơn hàng mẫu thật, và footer không bị watermark che khuất.
   - Viết lại toàn bộ [`HW03/task3-cross-platform/matrix.md`](../task3-cross-platform/matrix.md) đồng bộ 1-1 (`55/55` mục `CHK-001`..`CHK-055`) với kết quả Chrome của Task 1 (`14 Passed / 41 Failed`), và đưa Firefox/Safari về trạng thái trung thực **`Pending Verification`** chờ chạy thực tế trên BrowserStack/LambdaTest.
5. **Chuẩn hóa AI Audit Logs (`HW03/ai-audit/`):**
   - Sửa thông tin công cụ thực thi sai (`Playwright` → `Node.js & Chrome DevTools Protocol (CDP)`) trong `20261110-1315-task1-step3-execution.md`, thống nhất khai báo `Name of the AI tool` trên các nhật ký, và cập nhật nhật ký Task 3 `20261111-1235-task3-cross-platform.md`.

---

## 3. Human Review & Verification Decisions

- **Kiểm chứng tự động & thủ công:** Toàn bộ ảnh mới chụp được kiểm tra trực quan (khung hình, màu sắc, nội dung MSSV, không chồng chữ); kiểm tra tự động xác nhận `0` đường dẫn `file:///`, `0` ảnh trùng MD5 ngoài chủ đích, `55/55` khớp tuyệt đối giữa Task 1 (`03-execution.md`, `HW03_GUI_Checklist.xlsx`) và Task 3 Chrome Baseline (`matrix.md`), cùng `12/12` GitHub Issues mang đúng nhãn `severity:*`.
- **Trạng thái Git:** Chưa tự động commit; chờ sinh viên rà soát lần cuối trước khi `git add` / `git commit` / `git push` theo đúng quy định của quy trình làm việc.
