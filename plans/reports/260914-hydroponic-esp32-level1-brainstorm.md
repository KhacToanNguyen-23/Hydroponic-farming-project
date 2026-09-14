# Brainstorm: Nâng cấp Hệ thống Thủy canh 5 Trụ Ngoài trời với ESP32 & Cảm biến Mức 1

**Date:** 2026-09-14

## Ideas Explored

1. **Option A: ESP32 + Blynk IoT App (MQTT/REST API dưới nền)**
   - *Ưu điểm:* Cấu hình giao diện kéo thả nhanh chóng trên Smartphone, có sẵn Push Notification khi cạn nước, quản lý cài đặt từ xa qua Cloud. Dễ chuyển đổi API sang App riêng về sau.
   - *Chi phí/Rủi ro:* Giới hạn tính năng nâng cao ở tài khoản miễn phí.

2. **Option B: ESP32 + Custom MQTT Broker (Mosquitto/HiveMQ) + Web Dashboard / App MQTT**
   - *Ưu điểm:* Hoàn toàn làm chủ dữ liệu, chuẩn hóa 100% cho việc tự phát triển App riêng (Flutter/React Native) sau này.
   - *Chi phí/Rủi ro:* Cần thiết lập MQTT Broker và dựng giao diện ban đầu.

3. **Option C: ESP32 Web Server Trực tiếp (Access Point + Station Mode)**
   - *Ưu điểm:* Vẫn cài đặt được từ xa ở gần ngay cả khi mất kết nối Internet/Wifi nhà.
   - *Chi phí/Rủi ro:* Không nhận được cảnh báo PUSH xa khi ở ngoài mạng LAN nếu không có Cloud Sync.

## User's Direction

- **Hiện trạng:** 5 trụ thủy canh đặt ngoài trời (hứng mưa trực tiếp). Hệ thống cũ chỉ có bộ hẹn giờ điện tử nút nhấn thủ công tại tủ điện.
- **Phạm vi triển khai:** Tập trung **Mức 1** (Cảm biến Mưa + Cảm biến Nhiệt độ/Độ ẩm DHT22 + Phao mực nước thùng chứa + Relay máy bơm 220V + Nguồn ESP32).
- **Logic tưới:** Hybrid (Tưới chu kỳ timer cơ sở + Tự động tạm dừng/giảm thời gian tưới khi có mưa + Tăng tần suất tưới khi nhiệt độ > 35°C + Tắt máy bơm khẩn cấp khi phao báo cạn nước).
- **Nền tảng ứng dụng:** Dùng ứng dụng IoT có sẵn (Blynk IoT / MQTT Dashboard) trước để kiểm soát từ xa ngay, thiết kế kiến trúc phẳng (giao thức MQTT/API) để sẵn sàng mở rộng App riêng về sau.

## Open Questions

1. Tín hiệu Wifi tại vị trí đặt 5 trụ ngoài trời có đủ mạnh không (cần repeater hoặc anten ngoài cho ESP32 không)?
2. Loại phao mực nước và cảm biến mưa được chọn có độ bền cao khi phơi nắng mưa liên tục hay không?

## Risks

1. **Failsafe khi mất Wifi:** ESP32 phải giữ được RTC (Real-Time Clock) hoặc Timer bộ nhớ flash để tiếp tục tưới offline nếu đứt mạng Internet.
2. **Bảo vệ cảm biến ngoài trời:** Cảm biến mưa và DHT22 cần có vỏ/mái che bảo vệ mạch điện tử khỏi nước mưa gây ngắn mạch.
