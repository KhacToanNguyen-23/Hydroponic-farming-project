# Plan: Nâng cấp Điều khiển & Tưới Thông minh 5 Trụ Thủy canh Ngoài trời qua ESP32

**Mode:** --hard  
**Risk:** normal — Tích hợp phần cứng ESP32, cảm biến ngoài trời và Relay điều khiển điện áp 220V AC  
**Spec:** `plans/hydroponic-esp32-level1/spec.md`  

---

## Technical Overview

Hệ thống tưới tự động cho 5 trụ thủy canh hồi lưu ngoài trời sử dụng vi điều khiển **ESP32** thay thế bộ hẹn giờ nút bấm điện tử cũ. ESP32 được cấu hình theo kiến trúc **Dual-mode**:
1. **Online Mode (Blynk Cloud / MQTT):** Cho phép người dùng bật/tắt máy bơm từ xa 4G/Wifi, cài đặt thông số giờ tưới/nghỉ, xem đồ thị nhiệt ẩm và nhận Push Notification khi cạn nước.
2. **Offline Failsafe Mode:** Khi mất kết nối Wifi, ESP32 duy trì lịch tưới chu kỳ thông qua thông số lưu trong bộ nhớ Flash `Preferences`, đọc cảm biến mưa và phao cạn nước để bảo vệ hệ thống liên tục 24/7.

---

## Architectural Breakdown & Phase Structure

```
Phase 01: Thiết kế Sơ đồ Đấu nối Hardware & Sơ đồ Tủ điện
  ├── Pinout ESP32 ➔ Cảm biến Mưa, DHT22, Phao Mực nước, Relay 220V
  └── Thiết kế nguồn cấp 5V DC cách ly an toàn

Phase 02: Lập trình Firmware Core trên ESP32 (Arduino C++)
  ├── Quản lý trạng thái Máy bơm & Chu kỳ Timer
  ├── Xử lý tín hiệu Cảm biến Mưa (Rain Override) & Phao cạn (Dry-run Protection)
  └── Lưu cấu hình cài đặt vào bộ nhớ Flash (Preferences / EEPROM)

Phase 03: Tích hợp Cloud App (Blynk IoT / MQTT) & PUSH Notification
  ├── Kết nối Wifi, NTP Server đồng bộ thời gian
  ├── Đăng ký Virtual Pins Blynk / Topic MQTT
  └── Cấu hình Dashboard di động & Cảnh báo cạn nước PUSH Event

Phase 04: Verification & Stress Testing (Kiểm thử thực địa)
  ├── Kiểm thử ngắt kết nối Wifi (Offline Failsafe Test)
  ├── Kiểm thử giả lập mưa & ngắt bơm cạn nước
  └── Đánh giá độ trễ điều khiển từ xa qua 4G
```

---

## File Ownership & Mapping

- `phase-01-hardware-pinout.md` ➔ Sơ đồ sơ đồ mạch & danh sách linh kiện.
- `phase-02-firmware-core.md` ➔ Source code C++ firmware (`src/main.cpp`, `src/config.h`, `src/sensors.cpp`, `src/pump_control.cpp`).
- `phase-03-blynk-mqtt-integration.md` ➔ Source code kết nối Cloud (`src/network.cpp`, `src/blynk_handler.cpp`).
- `phase-04-failsafe-testing.md` ➔ Kịch bản và log kiểm thử verification.

---

## Phase Details

### Phase 1: Hardware Pinout & Tủ Điện
- **Covers:** P1 (Safety), P2 (DHT22), Pinout ESP32
- **Deliverables:** Document sơ đồ chân ESP32, sơ đồ nguyên lý relay cách ly 220V.

### Phase 2: Firmware Core Logic & Failsafe
- **Covers:** FR-02, FR-03, FR-04, FR-06
- **Deliverables:** Code C++ quản lý state machine tưới chu kỳ, ngắt mưa, ngắt cạn nước.

### Phase 3: Cloud Integration & Mobile Dashboard
- **Covers:** P1 (Remote Timer), P2 (Climate Monitoring), FR-01, FR-05
- **Deliverables:** Tích hợp Blynk SDK / MQTT Client, gửi dữ liệu telemetry & nhận lệnh remote.

### Phase 4: Verification & Integration Testing
- **Covers:** Acceptance criteria trong `spec.md`
- **Deliverables:** Biên bản kiểm thử Failsafe Offline, Giả lập Mưa, Giả lập Cạn nước.

---

## Risk Mitigation & Red-Team Notes

- **Rủi ro rò rỉ điện 220V:** Relay bắt buộc dùng loại có cách ly quang (Optocoupler) và dây nối nguồn 220V phải bọc ghen co nhiệt, cầu chì bảo vệ.
- **Rủi ro cảm biến mưa báo ảo do đọng nước:** Tấm cảm biến mưa đặt nghiêng 30 độ và bọc lớp sơn bảo vệ đường mạch tránh bị gỉ vôi.
