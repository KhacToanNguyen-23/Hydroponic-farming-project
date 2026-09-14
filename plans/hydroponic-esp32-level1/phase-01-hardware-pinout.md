# Phase 01: Sơ đồ Đấu nối Hardware & Sơ đồ Tủ điện

**Spec Story Mapping:**
- P1: Bật/Tắt an toàn máy bơm 220V
- P1: Bảo vệ máy bơm chống cháy do cạn nước
- P2: Đọc cảm biến môi trường DHT22

---

## 1. Danh sách Linh kiện Phần cứng (Mức 1)

| STT | Tên linh kiện | Số lượng | Thông số kỹ thuật | Ghi chú |
|-----|---------------|----------|-------------------|---------|
| 1 | ESP32 DevKit V1 | 1 | 30 Pin, Dual Core, Wifi + BLE | Vi điều khiển trung tâm |
| 2 | Nguồn Nguồn xung 5V 2A | 1 | Input 220V AC -> Output 5V DC | Cấp nguồn cho ESP32 & Relay |
| 3 | Module Relay 1 Kênh | 1 | 5V DC, Trigger High/Low, Opto cách ly, chịu tải 250V 10A AC | Điều khiển máy bơm 220V |
| 4 | Cảm biến Mưa | 1 | Dạng tấm kìm loại gạt mưa + Board so sánh LM393 | Đặt ngoài trời hứng mưa |
| 5 | Cảm biến Nhiệt/Ẩm DHT22 | 1 | Đo nhiệt độ -40..80°C, Độ ẩm 0..100% | Đặt trong hộp thoáng khí |
| 6 | Phao cảm biến mực nước | 1 | Phao điện tiếp điểm từ (Float Switch) | Gắn đáy/thành thùng chứa |
| 7 | Tủ điện & Dây nối | 1 | Hộp nhựa IP65 cách điện | Tận dụng tủ điện cũ |

---

## 2. Sơ đồ Chân Đấu Nối (ESP32 Pinout)

```
                       +-------------------+
                       |    ESP32 DEVKIT   |
                       |                   |
        [5V Nguồn] ----| VIN           GND |---- [GND Chung]
        [3.3V Out] ----| 3V3          GPIO2 |---- [LED Onboard Signal]
   [Relay Bơm In] ----| GPIO26        GPIO4 |---- [DHT22 Data Pin]
  [Cảm biến Mưa DO]---| GPIO27       GPIO14 |---- [Phao Cạn Nước DO]
                       +-------------------+
```

* **Chi tiết đấu nối:**
  * **Relay Máy Bơm:** `VCC` -> 5V, `GND` -> GND, `IN` -> `GPIO26` (ESP32).
    * Relay `COM` nối với dây L (Nóng) 220V AC.
    * Relay `NO` (Normally Open) nối vào 1 cực của Máy Bơm.
  * **Cảm biến Mưa (Raindrop Sensor):** `VCC` -> 3.3V, `GND` -> GND, `DO` (Digital Out) -> `GPIO27` (Có điện trở kéo lên/kéo xuống).
  * **Cảm biến Nhiệt ẩm DHT22:** `VCC` -> 3.3V, `GND` -> GND, `DATA` -> `GPIO4` (Nối trở Pull-up 10k Ohm vào 3.3V).
  * **Phao Mực Nước (Float Switch):** 1 chân nối `GND`, 1 chân nối `GPIO14` (Cấu hình `INPUT_PULLUP` trên ESP32).

---

## 3. Quy trình Kiểm tra Phần cứng

1. Dùng đồng hồ VOM đo điện áp nguồn 5V DC sau adapter đảm bảo đạt 4.9V - 5.2V.
2. Kiểm tra tính cách ly giữa mạch 5V DC của ESP32 và mạch 220V AC của Máy Bơm qua Relay trước khi cắm điện lưới.
