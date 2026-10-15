# Phân tích Khoảng trống AI (AI Gap Analysis): FR-10 (Trạng thái đơn hàng)

> **Lưu ý:** Báo cáo này do **sinh viên trực tiếp sở hữu, đánh giá và hoàn thiện** nhằm đáp ứng mục tiêu chuẩn đầu ra Bloom-AI cấp độ G9.3 (Phân tích & Phản biện).

---

## 1. Các Ca Kiểm thử mà AI bỏ sót ban đầu (Missed Test Cases)
*(Đánh giá xem trong quá trình AI phân tích State Machine ở Step 2 & Step 3 có bỏ qua khía cạnh nào không?)*

- **Ca kiểm thử 1 (Gợi ý):** Kiểm tra tính đồng thời (Race condition): Nếu User và Admin cùng thực hiện thao tác trên một đơn hàng tại cùng một thời điểm (ví dụ: User bấm hủy đơn `pending` cùng lúc Admin bấm duyệt `confirmed`), hệ thống có cơ chế Transaction Locking để tránh xung đột trạng thái hay không?
- **Ca kiểm thử 2 (Gợi ý):** Tác động phụ tới tồn kho hàng hóa (Inventory side effects): Khi trạng thái chuyển sang `canceled`, hệ thống có tự động hoàn lại số lượng tồn kho của sản phẩm trong đơn hàng hay không (Spec không đề cập chi tiết DB trigger).
- **Ca kiểm thử bổ sung của sinh viên:**
  - *[Sinh viên bổ sung thêm các case phát hiện được]*

---

## 2. Các Lỗi (Bugs) mà AI có thể đã bỏ sót nếu chỉ nhìn ở một góc độ
*(Đánh giá các khuyết tật mà AI có thể bị khuất tầm nhìn nếu không kiểm thử đa chiều)*

- **Lỗi bỏ sót tiềm ẩn (Gợi ý):**
  - **Lỗi phân quyền Broken Access Control (`BUG-FR10-03`):** Nếu AI chỉ kiểm thử luồng chức năng với tài khoản Admin mà không chủ động dùng Token của User thường để gọi chéo API Admin, lỗi bảo mật cực kỳ nghiêm trọng này sẽ bị bỏ sót hoàn toàn.
  - **Lỗi chuyển đổi trái phép `canceled` $\rightarrow$ `delivered` (`BUG-FR10-02`):** Lỗi này nằm ẩn sâu trong mã nguồn logic `if (currentStatus === "canceled" && status === "delivered") isValidTransition = true;`. Nếu không khảo sát ma trận chuyển đổi 5x5 đầy đủ từ các trạng thái kết thúc, AI sẽ không bao giờ ngờ tới việc hệ thống cho phép "hồi sinh" một đơn hàng đã hủy thành đã giao.
- **Lỗi bổ sung của sinh viên:**
  - *[Sinh viên ghi nhận thêm]*

---

## 3. Lý do AI bỏ sót hoặc có hạn chế (Why the AI Missed It)

1. **Chất lượng câu lệnh (Prompt Quality):**
   - Nếu prompt chỉ yêu cầu "kiểm tra sơ đồ chuyển trạng thái", AI thường chỉ tập trung vào các trạng thái nối tiếp xuôi chiều (`pending` $\rightarrow$ `confirmed` $\rightarrow$ `shipping` $\rightarrow$ `delivered`), ít khi tự động nghĩ ra việc kiểm tra chuyển đổi chéo giữa hai trạng thái kết thúc (`canceled` sang `delivered`).
2. **Hạn chế của công cụ AI (Tool Limitations):**
   - AI không tự động suy diễn các kịch bản tấn công leo quyền (Privilege Escalation) nếu không được hướng dẫn áp dụng các tiêu chuẩn bảo mật (như OWASP Top 10 API Security).
3. **Độ phức tạp và sự thiếu nhất quán của bản đặc tả (Feature Complexity & Spec Gaps):**
   - Đặc tả FR-10 có sự mâu thuẫn nhẹ giữa sơ đồ vẽ (không có mũi tên từ `shipping` sang `canceled`) và lời văn (*"chỉ Admin mới có thể thao tác"*), khiến việc xác định hành vi chuẩn của Admin khi đơn hàng ở `shipping` đòi hỏi sự phân tích và xác minh cẩn trọng.

---

## 4. Những điều chỉnh của Sinh viên sau khi Review (What I Changed After Review)
*(Sinh viên tóm tắt các điểm bản thân đã can thiệp, phản biện hoặc điều chỉnh kết quả của AI)*

- Yêu cầu AI xây dựng ma trận trạng thái đầy đủ 5x5 (25 tổ hợp) thay vì chỉ kiểm tra các nhánh vẽ trên sơ đồ.
- Chỉ đạo kiểm tra phân quyền chặt chẽ giữa User và Admin trên từng endpoint nhạy cảm (SEC-02, SEC-03).
- *[Sinh viên bổ sung thêm các đóng góp cá nhân]*
