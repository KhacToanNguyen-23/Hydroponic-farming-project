# Phase 03: Tích hợp Cloud App (Blynk IoT / MQTT) & PUSH Notification

**Spec Story Mapping:**
- P1: Cài đặt thời gian và tần suất tưới từ xa qua App
- P2: Theo dõi nhiệt độ & độ ẩm không khí xung quanh 5 trụ
- P2: Cảnh báo mất điện 220V PUSH Alert
- FR-01: Kết nối Wifi & Đồng bộ thời gian NTP
- FR-08: Nhắc vệ sinh cảm biến mưa định kỳ 14 ngày/lần

---

## 1. Bảng Ánh xạ Chân Ảo (Virtual Pins Mapping)

| Virtual Pin | Giao thức MQTT Topic | Hướng | Mô tả | Dạng dữ liệu |
|-------------|----------------------|-------|-------|--------------|
| `V0` | `hydro/pump/command` | Downstream | Nút Bật/Tắt Bơm thủ công | Enum (0: Off, 1: On) |
| `V1` | `hydro/pump/status` | Upstream | Trạng thái thực tế Máy Bơm | Enum (0: Off, 1: On) |
| `V2` | `hydro/config/run_time` | Bi-directional | Thời gian Bật tưới (phút) | Integer (1..120) |
| `V3` | `hydro/config/stop_time` | Bi-directional | Thời gian Nghỉ tưới (phút) | Integer (1..240) |
| `V4` | `hydro/sensors/temp` | Upstream | Nhiệt độ môi trường | Float (°C) |
| `V5` | `hydro/sensors/humidity` | Upstream | Độ ẩm không khí | Float (%) |
| `V6` | `hydro/sensors/rain` | Upstream | Trạng thái Mưa | Enum (0: Tịt, 1: Mưa) |
| `V7` | `hydro/sensors/water_low` | Upstream | Trạng thái Mực Nước | Enum (0: Đầy, 1: Cạn) |
| `V8` | `hydro/status/power_grid` | Upstream | Trạng thái Nguồn 220V | Enum (0: Mất, 1: Có) |
| `V9` | `hydro/status/climate_adj` | Upstream | Báo trạng thái Bù nhiệt | Enum (0: Off, 1: Active) |

---

## 2. Event Cảnh báo PUSH & Nhắc Nhở Bảo trì

1. **Event `water_dry_alarm`:** Push Alert *"⚠️ CẢNH BÁO: Thùng nước hồi lưu bị cạn! Máy bơm đã tự động ngắt."*
2. **Event `power_outage_alarm`:** Push Alert *"⚠️ CẢNH BÁO: Mất điện lưới 220V! ESP32 đang duy trì bằng nguồn pin dự phòng."*
3. **Event `rain_sensor_maintenance`:** Push Notification mỗi 14 ngày: *"🔔 NHẮC NHỞ: Hãy lau chùi vệ sinh tấm cảm biến mưa ngoài trời."*
