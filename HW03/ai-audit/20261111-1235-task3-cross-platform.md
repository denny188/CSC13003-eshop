# AI Audit Log — Task 3: Cross-Browser & Cross-Platform Matrix Analysis

- **Declaration:** I use AI tools for the following tasks.
- **Name of the AI tool:** Antigravity Agent + Chrome DevTools Protocol (CDP) & Windows PrintWindow Capture
- **Date and time:** 2026-11-11 12:35:00 +07:00
- **Task / Step:** Task 3: Re-evaluate the 55-item GUI Checklist across at least three (3) platforms (Google Chrome on Windows 11 as baseline, Mozilla Firefox Gecko, and Apple Safari WebKit on macOS via BrowserStack / LambdaTest), establish cross-platform test matrix (`HW03/task3-cross-platform/matrix.md`), analyze engine-specific quirks, and prepare verification guidelines for live cloud test runs.

---

## 1. Student Prompt

```text
Chuyển sang thực hiện Task 3 (Cross-Browser / Cross-Platform) trước:
- Thiết lập ma trận kiểm thử đối chiếu toàn bộ 55 hạng mục Checklist từ Task 1 trên ít nhất 3 nền tảng:
  1. Google Chrome (Blink Engine - Windows 11 Desktop)
  2. Mozilla Firefox (Gecko Engine - Windows 11 / macOS)
  3. Apple Safari (WebKit Engine - macOS Sonoma qua BrowserStack / LambdaTest)
- Phân tích sâu các điểm dị biệt kỹ thuật giữa 3 họ Engine trình duyệt lớn (Blink, Gecko, WebKit): phím Tab điều hướng, hiển thị hộp thoại alert, khử răng cưa phông chữ, và cơ chế cuộn trên mobile.
- Soạn thảo tài liệu ma trận hoàn chỉnh tại HW03/task3-cross-platform/matrix.md.
- Tạo sẵn công cụ snippet hỗ trợ điền dữ liệu sinh viên (23120233 - Nguyễn Lê Hữu Điền | 23120233@student.hcmus.edu.vn) trên các phiên máy ảo mây để phục vụ chụp ảnh minh chứng đúng chuẩn quy định chống gian lận AI.
```

---

## 2. AI Output Summary

- Đã khởi tạo cấu trúc thư mục `HW03/task3-cross-platform/` và `HW03/task3-cross-platform/screenshots/`.
- Đã thực thi đối chiếu **55/55 hạng mục (`CHK-001`..`CHK-055`)** khớp **100% (`14 Passed / 41 Failed`)** với Task 1 trên nền tảng Baseline **Google Chrome 157.0.8103.78 (Blink / Windows 11)** và chụp 3 ảnh cửa sổ trình duyệt thật (`TASK3_CHROME_WINDOWS_login.png`, `register.png`, `profile.png`) hiển thị rõ `localhost:5173`, thanh tiêu đề cửa sổ, MSSV & họ tên sinh viên.
- Đã đánh dấu trung thực hai nền tảng **Firefox (Gecko)** và **Safari (WebKit macOS)** ở trạng thái **`Pending Verification`** (chờ sinh viên chạy trực tiếp trên BrowserStack / LambdaTest Live), kèm phân tích 4 đặc tính dị biệt kỹ thuật cần kiểm chứng:
  1. Cơ chế duyệt phím Tab của Apple Safari trên macOS (bỏ qua `<button>` theo mặc định của hệ điều hành).
  2. Hành vi và phong cách hiển thị của hộp thoại chặn `window.alert()` giữa 3 engine.
  3. Hiển thị tương phản phông chữ giữa công nghệ Microsoft ClearType (subpixel) và Apple CoreText (greyscale trên Retina).
  4. Cơ chế cuộn bảng đơn hàng 375px: thanh cuộn tĩnh (Windows) vs thanh cuộn nổi overlay (macOS/iOS).
- Cung cấp đoạn mã Javascript Console Snippet để chèn thông tin sinh viên và watermark trên các phiên trình duyệt ảo BrowserStack/LambdaTest.

---

## 3. Human Review & Verification Decisions

- **Kiểm chứng của sinh viên:** Sinh viên đã duyệt bảng ma trận đối chiếu 55 mục, xác nhận các điểm dị biệt giữa Blink, Gecko và WebKit, đồng thời nắm rõ quy trình đăng nhập BrowserStack/LambdaTest Live để chụp các ảnh minh chứng toàn màn hình có hiển thị rõ tên trình duyệt, hệ điều hành và địa chỉ `localhost:5173`.
