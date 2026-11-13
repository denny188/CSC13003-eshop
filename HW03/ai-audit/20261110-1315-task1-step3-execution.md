# AI Audit Log — Task 1 (Step 1.3): GUI Checklist Execution & Screenshot Evidence

- **Declaration:** I use AI tools for the following tasks.
- **Name of the AI tool:** Antigravity Agent (with Skill `hw03-gui-checklist`) + Node.js & Chrome DevTools Protocol (CDP)
- **Date and time:** 2026-11-10 13:15:00 +07:00
- **Task / Step:** Task 1 — Step 1.3: Execute the 55-item GUI Checklist against the live EShop SUT (`http://localhost:5173`), record Pass/Fail status and detailed Notes, capture screenshots for Failed items with student overlay (`23120233@student.hcmus.edu.vn`), and generate `HW03_GUI_Checklist.xlsx`.

---

## 1. Student Prompt

```text
Áp dụng skill hw03-gui-checklist (Bước 3): Hãy khởi chạy Backend (:3000) và Frontend Web (:5173), sau đó thực thi toàn bộ 55 mục trong bảng GUI Checklist (CHK-001 đến CHK-055) trên trình duyệt Google Chrome.
- Đánh dấu từng mục là Passed hoặc Failed.
- Điền cột Notes giải thích rõ nguyên nhân và dòng code gây lỗi cho tất cả các mục Failed.
- Chụp ảnh màn hình thực tế trên trình duyệt Chrome (chỉ cho các mục Failed theo yêu cầu đề bài) có đóng dấu thông tin sinh viên "23120233@student.hcmus.edu.vn | 23120233 - Nguyễn Lê Hữu Điền" và lưu vào HW03/task1-gui/screenshots/.
- Xuất kết quả ra cả file Markdown (HW03/task1-gui/03-execution.md) và file Excel (HW03/task1-gui/HW03_GUI_Checklist.xlsx).
```

---

## 2. AI Output Summary

- Đã thực thi toàn bộ **55 mục** trên trình duyệt Google Chrome đối với 3 cụm màn hình (`/register`, `/login` + `/forgot-password`, `/profile` + `Header`).
- **Kết quả thống kê:**
  - **Tổng số mục:** 55
  - **Passed:** 14 mục (25.45%)
  - **Failed:** 41 mục (74.55%) — được gom thành **12 báo cáo lỗi (`BUG-GUI-01` → `BUG-GUI-12`)**.
- Đã chụp bộ ảnh minh chứng cho toàn bộ các mục `Failed` tại `HW03/task1-gui/screenshots/` và xuất bảng tổng hợp sang định dạng Excel `HW03/task1-gui/HW03_GUI_Checklist.xlsx`.

---

## 3. Human Review & Verification Decisions

- **Kiểm chứng của sinh viên:** Sinh viên đã đối chiếu từng kết quả `Passed`/`Failed`, kiểm tra nội dung cột `Notes` và xem lại toàn bộ ảnh chụp màn hình trong `HW03/task1-gui/screenshots/` để đảm bảo hình ảnh phản ánh đúng lỗi giao diện thực tế và chỉ đính kèm cho các mục `Failed`.
