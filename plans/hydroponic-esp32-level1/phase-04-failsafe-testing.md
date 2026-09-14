# Phase 04: Verification & Stress Testing (Kiểm thử thực địa)

**Spec Story Mapping:**
- Acceptance Criteria Verification: Tất cả các tiêu chí nghiệm thu trong `spec.md`.

---

## 1. Kịch bản Kiểm thử Nghiệm thu

### Test Suite 1: Thuật toán Bù nhiệt độ môi trường (> 35°C)
- **Các bước:** Dùng máy sấy tóc / hơi nóng thổi vào cảm biến DHT22 cho đến khi giá trị > 35°C.
- **Kỳ vọng:** Thời gian tưới tự động tăng 30%, thời gian nghỉ tự động giảm 20%. App hiển thị trạng thái `Đang bù nhiệt giữa trưa`.

---

### Test Suite 2: Cảnh báo Mất điện 220V qua Mini UPS 5V
- **Các bước:** Rút phích cắm nguồn 220V AC của tủ điện (chỉ để ESP32 chạy pin Mini UPS 5V).
- **Kỳ vọng:** Trong vòng `< 5 giây`, điện thoại nhận được Push Notification: *"⚠️ CẢNH BÁO: Mất điện lưới 220V!"*.

---

### Test Suite 3: Kiểm thử Chống nhiễu Máy bơm 220V với Mạch RC Snubber
- **Các bước:** Cho ESP32 kích Bật/Tắt Relay máy bơm 220V liên tục 100 lần (mỗi lần cách nhau 2 giây).
- **Kỳ vọng:** Mạch RC Snubber dập tắt hoàn toàn xung cảm ứng điện từ, ESP32 hoạt động ổn định 100% không bị reset hay treo giật.

---

### Test Suite 4: Giả lập Mưa & Cạn Nước
- **Các bước:** Nhỏ nước lên cảm biến mưa / nhấc phao cạn nước trong thùng chứa.
- **Kỳ vọng:** Máy bơm ngắt lập tức (< 2 giây) và gửi thông báo cảnh báo về Smartphone.
