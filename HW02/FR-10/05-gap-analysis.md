# Phân tích Khoảng trống AI (AI Gap Analysis): FR-10 (Trạng thái đơn hàng)

Báo cáo phân tích và phản biện năng lực AI trong quá trình kiểm thử máy trạng thái đơn hàng FR-10, đáp ứng chuẩn đầu ra Bloom-AI cấp độ G9.3 (Phân tích & Đánh giá).

---

## 1. Các Ca Kiểm thử mà AI bỏ sót ban đầu (Missed Test Cases)

1. **Kiểm tra tương tranh và xung đột trạng thái (Race conditions & Concurrency):**
   - *Mô tả:* Kịch bản hai tác nhân cùng thao tác đồng thời trên một đơn hàng (ví dụ: Khách hàng bấm gửi yêu cầu hủy đơn khi đơn đang ở `pending` đúng vào thời điểm Quản trị viên bấm xác nhận `confirmed`). Hệ thống cần có cơ chế khóa dòng dữ liệu (Row locking / Transaction Isolation) để tránh tình trạng đơn hàng vừa bị hủy vừa được xác nhận vận chuyển.
   - *Đánh giá:* AI khi thiết kế ca kiểm thử FSM thường xem xét hệ thống ở trạng thái tuần tự đơn luồng (Single-thread sequential), chưa chủ động đưa ra kịch bản kiểm thử tương tranh.
2. **Kiểm tra tác động phụ tới tồn kho hàng hóa (Inventory Side Effects):**
   - *Mô tả:* Khi đơn hàng chuyển trạng thái sang `canceled`, hệ thống có tự động hoàn lại số lượng tồn kho của các sản phẩm tương ứng hay không.

---

## 2. Các Lỗi (Bugs) mà AI có thể đã bỏ sót nếu chỉ nhìn ở một góc độ

1. **Lỗ hổng leo quyền Broken Access Control (`BUG-FR10-03`):**
   - *Phân tích:* Nếu người kiểm thử hoặc AI chỉ kiểm thử chức năng theo đúng kịch bản nghiệp vụ thông thường (Admin dùng token Admin để đổi trạng thái đơn hàng), API này sẽ luôn phản hồi HTTP 200 và được đánh giá là ĐẠT. Lỗ hổng nghiêm trọng này chỉ lộ diện khi người kiểm thử có tư duy bảo mật (Security mindset), chủ động tráo đổi Header bằng JWT Token của User thông thường để gọi vào API Admin (`PUT /api/admin/orders/:id/status`).
2. **Lỗi logic chuyển đổi trái phép giữa hai trạng thái kết thúc `canceled` $\rightarrow$ `delivered` (`BUG-FR10-02`):**
   - *Phân tích:* Lỗi này nằm trong câu lệnh `if (currentStatus === "canceled" && status === "delivered") isValidTransition = true;` ở backend. Nếu chỉ kiểm thử các đường đi xuôi chiều trên sơ đồ đồ thị, người kiểm thử sẽ không bao giờ nghĩ đến việc kiểm tra chuyển đổi từ một đơn hàng đã hủy thành đã giao. Chỉ khi xây dựng ma trận trạng thái đầy đủ 5x5 (25 cặp trạng thái), lỗi này mới bị bắt trọn vẹn.

---

## 3. Lý do AI bỏ sót hoặc có hạn chế (Why the AI Missed It)

1. **Chất lượng câu lệnh (Prompt Quality):**
   - Khi prompt chỉ yêu cầu "kiểm tra các bước chuyển đổi theo sơ đồ", AI có xu hướng bám sát các mũi tên trên sơ đồ minh họa (`pending` $\rightarrow$ `confirmed` $\rightarrow$ `shipping` $\rightarrow$ `delivered`), thiếu sự chủ động khảo sát các đường biên nghịch chiều hoặc các đường chuyển chéo giữa các trạng thái kết thúc.
2. **Hạn chế của công cụ AI (Tool Limitations):**
   - AI chưa tự động tích hợp các bộ quy tắc kiểm thử bảo mật nâng cao (như kiểm thử IDOR, kiểm thử leo quyền Broken Access Control) vào kiểm thử chức năng thông thường nếu không có chỉ thị rõ ràng từ sinh viên.
3. **Sự mâu thuẫn trong văn bản đặc tả (Spec Gap):**
   - Có sự không nhất quán giữa sơ đồ hình vẽ (không có mũi tên hủy từ `shipping`) và câu chữ mô tả (*"chỉ Admin mới có thể thao tác"*), khiến AI ban đầu lúng túng trong việc xác định liệu Admin có quyền hủy đơn đang giao hay không.

---

## 4. Những điều chỉnh của Sinh viên sau khi Review (What I Changed After Review)

- **Mở rộng mô hình FSM thành Ma trận chuyển đổi 5x5:** Chỉ đạo AI không chỉ vẽ lại các nhánh có sẵn trên lược đồ mà phải lập ma trận toàn diện 25 ô để kiểm thử tất cả các chuyển đổi không hợp lệ (nhảy cóc, đảo ngược, tự lặp lại).
- **Yêu cầu kiểm thử phân quyền độc lập:** Yêu cầu AI kiểm tra nghiêm ngặt yêu cầu SEC-03 bằng cách gửi request với token của khách hàng thông thường để phát hiện lỗ hổng thiếu middleware xác thực quyền Admin trên route quản trị đơn hàng.
