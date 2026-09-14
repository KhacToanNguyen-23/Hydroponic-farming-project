# Phase 03: Tích hợp Cloud App (Blynk IoT / MQTT) & PUSH Notification

**Spec Story Mapping:**
- P1: Cài đặt thời gian và tần suất tưới từ xa qua App
- P2: Theo dõi nhiệt độ & độ ẩm không khí xung quanh 5 trụ
- FR-01: Kết nối Wifi & Đồng bộ thời gian NTP
- FR-05: Đẩy dữ liệu cảm biến lên App di động

---

## 1. Bảng Ánh xạ Chân Ảo (Virtual Pins Mapping - Blynk / MQTT Topics)

| Virtual Pin (Blynk) | Giao thức MQTT Topic | Hướng | Mô tả | Dạng dữ liệu |
|---------------------|----------------------|-------|-------|--------------|
| `V0` | `hydro/pump/command` | Downstream (App -> ESP) | Công tắc Bật/Tắt Bơm thủ công | Enum (0: Off, 1: On) |
| `V1` | `hydro/pump/status` | Upstream (ESP -> App) | Trạng thái thực tế Máy Bơm | Enum (0: Off, 1: On) |
| `V2` | `hydro/config/run_time` | Bi-directional | Thời gian Bật tưới (phút) | Integer (1..120) |
| `V3` | `hydro/config/stop_time` | Bi-directional | Thời gian Nghỉ tưới (phút) | Integer (1..240) |
| `V4` | `hydro/sensors/temp` | Upstream (ESP -> App) | Nhiệt độ môi trường | Float (°C) |
| `V5` | `hydro/sensors/humidity` | Upstream (ESP -> App) | Độ ẩm không khí | Float (%) |
| `V6` | `hydro/sensors/rain` | Upstream (ESP -> App) | Trạng thái Mưa | Enum (0: Tịt, 1: Mưa) |
| `V7` | `hydro/sensors/water_low` | Upstream (ESP -> App) | Trạng thái Mực Nước | Enum (0: Đầy, 1: Cạn) |

---

## 2. Cấu hình Blynk IoT Dashboard trên Điện thoại

1. Tạo Template mới: `Hydroponic_5Towers_ESP32`.
2. Tạo Widget trên Dashboard App:
   * **Switch Widget (V0):** Nút bật/tắt thủ công máy bơm.
   * **Numeric Input / Slider Widget (V2 & V3):** Cài số phút Tưới (V2) và Nghỉ (V3).
   * **Gauge Widget (V4 & V5):** Hiển thị Nhiệt độ (°C) và Độ ẩm (%).
   * **LED Widget (V6 & V7):** Đèn màu đỏ báo Cạn nước (V7) và đèn màu xanh dương báo Mưa (V6).
3. **Cấu hình Event Cảnh báo PUSH:**
   * Tạo Event `water_dry_alarm`: Gửi Push Notification "⚠️ CANH BÁO: Thùng nước hồi lưu bị cạn! Máy bơm đã tự động tắt." khi `V7 == 1`.

---

## 3. Tự động Đồng bộ & Tự kết nối lại (Auto Reconnect)

```cpp
void checkWifiAndBlynk() {
    if (WiFi.status() != WL_CONNECTED) {
        WiFi.reconnect();
    } else {
        if (!Blynk.connected()) {
            Blynk.connect();
        }
    }
}
```
* Hàm `checkWifiAndBlynk()` chạy định kỳ mỗi 30 giây trong background Task mà không làm treo vòng lặp tưới offline của ESP32.
