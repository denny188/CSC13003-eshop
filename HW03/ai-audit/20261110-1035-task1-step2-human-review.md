# AI Audit Log — Task 1 (Step 1.2): Critical Human Review & AI Gap Analysis

- **Declaration:** I use AI tools for the following tasks.
- **Name of the AI tool:** Antigravity Agent (Gemini / Claude) with Skill `hw03-gui-checklist`
- **Date and time:** 2026-11-10 10:35:00 +07:00
- **Task / Step:** Task 1 — Step 1.2: Critically review the initial 40-item AI checklist and document 15 human-contributed checklist items (`CHK-041` → `CHK-055`) along with root-cause explanations of why the AI missed them.

---

## 1. Student Prompt

```text
Áp dụng skill hw03-gui-checklist (Bước 2): Tôi đã rà soát bộ 40 mục ban đầu trong 01-ai-initial-checklist.md và nhận thấy AI chỉ mới kiểm tra các quy tắc viết sẵn trong README.md (FR-21..FR-24). Hãy cùng tôi bổ sung thêm 15 mục kiểm tra chuyên sâu (CHK-041 đến CHK-055) mà AI đã bỏ sót trên các màn hình /register, /login, /forgot-password, /profile và thanh Navbar, bao gồm:
- Accessibility (WCAG contrast cho chữ text-gray-400, liên kết <label htmlFor> và <input id>, focus ring, role="alert" cho thông báo lỗi)
- Bố cục & đa thiết bị (Dark mode, RTL layout, responsive 375px cho bảng đơn hàng 5 cột, zoom 200%, tràn chữ khi tên dài)
- Công thái học biểu mẫu & trạng thái runtime (nút ẩn/hiện mật khẩu, autocomplete, type="tel" cho SĐT, ngõ cụt điều hướng khi vào /profile lúc chưa đăng nhập, khóa nút Submit khi đang chờ API, và lỗi render dangerouslySetInnerHTML trên Header).
Với mỗi mục bổ sung, hãy phân tích rõ lý do vì sao AI bỏ sót ở lần sinh đầu tiên. Lưu vào HW03/task1-gui/02-human-review.md.
```

---

## 2. AI Output Summary

AI đã tổng hợp phân tích phản biện của sinh viên và lập bảng 15 mục kiểm tra bổ sung (`CHK-041` → `CHK-055`) trong `HW03/task1-gui/02-human-review.md`, nâng tổng số mục GUI Checklist lên **55 mục** (16 mục `IA-01`, 16 mục `IA-02`, 11 mục `IA-03`, 12 mục `IA-04`). Mỗi mục đều đi kèm giải thích nguyên nhân AI bỏ sót dựa trên 3 nhóm lý do:
1. *Spec-Centrism Bias / Prompt Scope Limitation* (chỉ dịch xuôi các gạch đầu dòng trong `README.md`).
2. *Model Limitation / Lack of Visual & Spatial Rendering Perception* (không tính toán độ tương phản quang học hoặc kích thước hộp khi render ở màn hình `375px` / zoom `200%`).
3. *Interface-Specific Characteristic* (bỏ qua các chi tiết triển khai cụ thể trong DOM React như thiếu `htmlFor`/`id`, `dangerouslySetInnerHTML`, hay ngõ cụt `"Vui lòng đăng nhập"`).

---

## 3. Human Review & Verification Decisions

- **Đánh giá của sinh viên:** Đã kiểm tra từng mục trong 15 mục bổ sung đối chiếu với mã nguồn thực tế (`App.jsx`, `Register.jsx`, `Login.jsx`, `ForgotPassword.jsx`, `Profile.jsx`). Tất cả 15 mục đều xác đáng, không trùng lặp với 40 mục đầu và phản ánh đúng các khiếm khuyết thực sự trên giao diện EShop.
- **Quyết định:** Duyệt toàn bộ 55 mục (`CHK-001` → `CHK-055`) để chuyển sang Bước 1.3: Thực thi kiểm thử trực tiếp trên hệ thống (Test Execution), ghi nhận kết quả `Passed`/`Failed` + `Notes` và chụp ảnh màn hình minh chứng cho các mục `Failed`.
