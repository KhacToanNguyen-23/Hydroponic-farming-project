# Phase 01: Sơ đồ Đấu nối Hardware, Mạch lọc RC Snubber & Mini UPS

**Spec Story Mapping:**
- P1: Bật/Tắt an toàn máy bơm 220V
- P1: Bảo vệ máy bơm chống cháy do cạn nước
- P2: Đọc cảm biến môi trường DHT22
- P2: Cảnh báo mất điện 220V qua nguồn dự phòng Mini UPS 5V
- NFR: Mạch lọc RC Snubber chống nhiễu reset ESP32 khi ngắt bơm

---

## 1. Danh sách Linh kiện Phần cứng Chi tiết

| STT | Tên linh kiện | Số lượng | Thông số kỹ thuật | Ghi chú |
|-----|---------------|----------|-------------------|---------|
| 1 | **ESP32 DevKit V1** | 1 | 30 Pin, Dual Core, Wifi + BLE | Vi điều khiển trung tâm |
| 2 | **Mạch Nguồn Nguồn xung 5V 2A + Mạch Mini UPS 5V** | 1 | Input 220V AC -> 5V DC, kèm mạch sạc pin 18650 3.7V | Cấp nguồn liên tục cho ESP32 kể cả khi cúp điện |
| 3 | **Module Relay 1 Kênh** | 1 | 5V DC, Trigger High/Low, Opto cách ly, chịu tải 250V 10A AC | Điều khiển máy bơm 220V |
| 4 | **Mạch lọc RC Snubber** | 1 | Điện trở 100Ω 2W + Tụ 0.1µF 400V AC | **Đấu song song với tiếp điểm Relay / Máy bơm 220V** |
| 5 | **Module Giám sát Điện áp 220V (Opto Isolator Module)** | 1 | Input 220V AC -> Output 3.3V/5V DC Signal | Nhận biết trạng thái điện lưới 220V có hay cúp |
| 6 | **Cảm biến Mưa** | 1 | Dạng tấm kim loại gạt mưa / Quang học + Board LM393 | Đặt ngoài trời hứng mưa |
| 7 | **Cảm biến Nhiệt/Ẩm DHT22 / SHT30** | 1 | Đo nhiệt độ -40..80°C, Độ ẩm 0..100% | Đặt trong hộp thoáng khí |
| 8 | **Phao cảm biến mực nước** | 1 | Phao điện tiếp điểm từ (Float Switch) | Gắn đáy/thành thùng chứa |
| 9 | **Cảm biến Ánh sáng (BH1750/LDR)** | 1 | Giao tiếp I2C / Analog | Dự phòng Giai đoạn 2 |
| 10 | **Tủ điện nhựa IP65 & Dây nối** | 1 | Hộp nhựa IP65 cách điện | Tận dụng tủ điện cũ |

---

## 2. Sơ đồ Chân Đấu Nối (ESP32 Pinout)

```
                       +-------------------+
                       |    ESP32 DEVKIT   |
                       |                   |
        [5V Mini UPS] -| VIN           GND |---- [GND Chung]
        [3.3V Out] ----| 3V3          GPIO2 |---- [LED Trạng thái]
   [Relay Bơm In] ----| GPIO26        GPIO4 |---- [DHT22 Data]
  [Cảm biến Mưa DO]---| GPIO27       GPIO14 |---- [Phao Cạn Nước DO]
 [Giám sát 220V In]---| GPIO12       GPIO13 |---- [Light Sensor BH1750]
                       +-------------------+
```

### Chi tiết mạch dập xung RC Snubber:
```
 Relay COM (220V L In) ───┬───────────── (Tiếp điểm Relay) ─────────────┬─── Relay NO ───> Máy Bơm 220V
                          │                                             │
                          └───[ Điện trở 100Ω 2W ]───[ Tụ 0.1µF 400V ]──┘
                                      (Mạch RC Snubber)
```

---

## 3. Quy trình Kiểm tra Phần cứng

1. Đo điện áp đầu ra nguồn Mini UPS 5V khi cắm điện 220V (đạt 5.0V - 5.1V) và khi ngắt điện 220V (đạt 4.8V - 5.0V từ pin 18650).
2. Kiểm tra chân `GPIO12` xuống mức `LOW` khi cúp điện 220V.
3. Bật/tắt Relay máy bơm 220V 20 lần để kiểm tra mạch RC Snubber dập xung điện tử hoàn toàn.
