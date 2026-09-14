# Plan: Nâng cấp Điều khiển & Tưới Thông minh 5 Trụ Thủy canh Ngoài trời qua ESP32 (Phiên bản Tối ưu hóa Team)

**Mode:** --hard  
**Risk:** normal — Tích hợp phần cứng ESP32, cảm biến ngoài trời, mạch lọc RC Snubber, mạch nguồn dự phòng Mini UPS 5V và Relay điều khiển điện áp 220V AC  
**Spec:** `plans/hydroponic-esp32-level1/spec.md`  

---

## Technical Overview

Hệ thống tưới tự động cho 5 trụ thủy canh hồi lưu ngoài trời sử dụng vi điều khiển **ESP32** thay thế bộ hẹn giờ nút bấm điện tử cũ. Hệ thống vận hành theo kiến trúc **Dual-mode & Climate Adaptive**:
1. **Online Mode (Blynk Cloud / MQTT):** Cho phép người dùng bật/tắt máy bơm từ xa 4G/Wifi, cài đặt thông số giờ tưới/nghỉ, xem đồ thị nhiệt ẩm, nhận Push Notification khi cạn nước, nhận Push Alert khi cúp điện 220V và nhắc nhở vệ sinh cảm biến mưa định kỳ.
2. **Climate Compensation:** Tự động điều chỉnh thời lượng tưới khi nhiệt độ > 35°C (tăng 30% thời gian tưới, giảm 20% thời gian nghỉ).
3. **Offline Failsafe Mode:** Khi mất kết nối Wifi, ESP32 duy trì lịch tưới chu kỳ thông qua thông số lưu trong bộ nhớ Flash `Preferences`, đọc cảm biến mưa và phao cạn nước để bảo vệ hệ thống liên tục 24/7.
4. **Hardware Anti-Interference:** Tích hợp mạch lọc **RC Snubber** triệt tiêu xung điện từ của máy bơm 220V, đảm bảo ESP32 hoạt động ổn định không bị giật reset.

---

## Architectural Breakdown & Phase Structure

```
Phase 01: Thiết kế Sơ đồ Đấu nối Hardware, Mạch lọc RC Snubber & Mini UPS
  ├── Pinout ESP32 ➔ Cảm biến Mưa, DHT22, Phao Mực nước, Relay 220V, Giám sát 220V Grid
  ├── Mạch dập xung RC Snubber song song tiếp điểm Relay/Máy bơm 220V
  └── Mạch nguồn sạc dự phòng Mini UPS 5V duy trì ESP32 khi mất điện

Phase 02: Lập trình Firmware Core trên ESP32 (Arduino C++)
  ├── Quản lý trạng thái Máy bơm, Chu kỳ Timer & State Machine
  ├── Thuật toán Bù trừ Nhiệt độ (Temp > 35°C ➔ Run +30%, Stop -20%)
  ├── Xử lý tín hiệu Cảm biến Mưa & Phao cạn (Dry-run Protection)
  └── Lưu cấu hình cài đặt vào bộ nhớ Flash (Preferences / EEPROM)

Phase 03: Tích hợp Cloud App (Blynk IoT / MQTT) & PUSH Alerts
  ├── Kết nối Wifi, NTP Server đồng bộ thời gian
  ├── Cảnh báo PUSH khi mất điện lưới 220V (gửi qua nguồn Mini UPS 5V)
  ├── Cảnh báo PUSH khi cạn nước thùng chứa
  └── Thông báo bảo trì định kỳ vệ sinh cảm biến mưa (14 ngày/lần)

Phase 04: Verification & Stress Testing (Kiểm thử thực địa)
  ├── Kiểm thử bù nhiệt độ (> 35°C)
  ├── Kiểm thử cảnh báo mất điện lưới 220V
  ├── Kiểm thử 100 lần đóng ngắt bơm 220V với mạch RC Snubber (Anti-reset)
  └── Kiểm thử ngắt kết nối Wifi & Giả lập mưa/cạn nước
```

---

## File Ownership & Mapping

- `phase-01-hardware-pinout.md` ➔ Sơ đồ sơ đồ mạch, linh kiện Mini UPS & RC Snubber.
- `phase-02-firmware-core.md` ➔ Source code C++ firmware (`src/main.cpp`, `src/config.h`, `src/sensors.cpp`, `src/pump_control.cpp`).
- `phase-03-blynk-mqtt-integration.md` ➔ Source code kết nối Cloud & Alerts (`src/network.cpp`, `src/blynk_handler.cpp`).
- `phase-04-failsafe-testing.md` ➔ Kịch bản và log kiểm thử verification.

---

## Phase Details

### Phase 1: Hardware Pinout, RC Snubber & Mini UPS
- **Covers:** P1 (Safety), P2 (DHT22), P2 (Power Outage), RC Snubber
- **Deliverables:** Document sơ đồ chân ESP32, sơ đồ nguyên lý RC Snubber & Mini UPS 5V.

### Phase 2: Firmware Core Logic, Climate Adjustment & Failsafe
- **Covers:** FR-02, FR-03, FR-04, FR-05, FR-06, FR-07
- **Deliverables:** Code C++ quản lý state machine tưới chu kỳ, ngắt mưa, ngắt cạn nước, bù nhiệt > 35°C.

### Phase 3: Cloud Integration, Mobile Dashboard & Alerts
- **Covers:** P1 (Remote Timer), P2 (Climate Monitoring), FR-01, FR-07, FR-08
- **Deliverables:** Tích hợp Blynk SDK / MQTT Client, cảnh báo PUSH mất điện & nhắc bảo vệ cảm biến mưa.

### Phase 4: Verification & Integration Testing
- **Covers:** Acceptance criteria trong `spec.md`
- **Deliverables:** Biên bản kiểm thử Failsafe Offline, Bù nhiệt, Cúp điện 220V, Chống nhiễu RC Snubber.
