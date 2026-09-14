# Phase 04: Verification & Stress Testing (Kiểm thử thực địa)

**Spec Story Mapping:**
- Acceptance Criteria Verification: Tất cả các tiêu chí nghiệm thu trong `spec.md`.

---

## 1. Kịch bản Kiểm thử Nghiệm thu (Test Suites)

### Test Suite 1: Điều khiển Từ xa qua 4G (Remote Control Test)
- **Các bước:**
  1. Điện thoại bật 4G (tắt Wifi).
  2. Bấm nút Bật Bơm trên App Blynk / MQTT.
  3. Thay đổi Thời gian Tưới từ 15 phút thành 20 phút trên App.
- **Kỳ vọng:**
  * Relay đóng lập tức, máy bơm chạy, đèn LED tín hiệu trên ESP32 sáng (< 2 giây).
  * ESP32 lưu giá trị `20 phút` vào bộ nhớ Flash.

---

### Test Suite 2: Giả lập Mưa (Rain Override Test)
- **Các bước:**
  1. Để máy bơm đang ở trạng thái BẬT theo lịch tưới.
  2. Nhỏ 2-3 giọt nước lên tấm cảm biến mưa ngoài trời.
- **Kỳ vọng:**
  * Relay lập tức NGẮT máy bơm trong vòng < 2 giây.
  * App hiển thị trạng thái `Trời mưa - Tạm dừng tưới`.
  * Dùng khăn lau khô cảm biến ➔ Sau 5 giây máy bơm khôi phục chạy lại theo lịch.

---

### Test Suite 3: Bảo vệ Chống Cháy Máy Bơm khi Cạn Nước (Dry-run Protection)
- **Các bước:**
  1. Cho máy bơm đang chạy.
  2. Dùng tay nhấc phao mực nước trong thùng chứa xuống (giả lập cạn nước).
- **Kỳ vọng:**
  * Relay lập tức NGẮT máy bơm ngay lập tức (0.5 giây).
  * Điện thoại nhận Push Notification: "⚠️ CANH BÁO: Thùng nước hồi lưu bị cạn!".
  * Thử ấn nút Bật Bơm trên App ➔ ESP32 từ chối bật bơm và gửi cảnh báo "Không thể bật máy bơm do thùng cạn nước".

---

### Test Suite 4: Kiềm thử Đứt Mạng Wifi (Offline Failsafe Test)
- **Các bước:**
  1. Cài đặt lịch tưới: Bật 1 phút - Nghỉ 1 phút.
  2. Rút cắm nguồn Router Wifi (Tắt hoàn toàn mạng Wifi).
- **Kỳ vọng:**
  * ESP32 phát hiện mất Wifi, vẫn duy trì đếm thời gian: Bật 1 phút ➔ Tắt 1 phút ➔ Bật 1 phút đều đặn.
  * Cắm lại Router Wifi ➔ ESP32 tự kết nối lại với Cloud trong vòng 30 giây mà không cần reset nguồn.
