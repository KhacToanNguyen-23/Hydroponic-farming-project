# TÀI LIỆU KỸ THUẬT & HƯỚNG DẪN DỰ ÁN
## Nâng cấp Hệ thống Tưới Thông minh 5 Trụ Thủy canh Ngoài trời qua ESP32 IoT

**Dự án:** Hydroponic Farming IoT Upgrade  
**Phiên bản:** 1.1 (Cập nhật Đề xuất & Tối ưu hóa từ Thành viên Team)  
**Ngày cập nhật:** 14/09/2026  
**Đối tượng đọc:** Các thành viên phát triển, kỹ thuật phần cứng & vận hành hệ thống  

---

## 1. 🎯 Tổng quan & Mục tiêu Dự án

Hệ thống trồng rau thủy canh hồi lưu 5 trụ đứng hiện tại đã đi vào hoạt động ngoài trời. 
- **Hiện trạng cũ:** Hệ thống dùng bộ hẹn giờ điện tử nút bấm cơ thủ công tại tủ điện. Hạn chế: không chỉnh được từ xa, tưới cố định theo giờ nên khi trời mưa gây lãng phí điện/tràn loãng dinh dưỡng, và không tự ngắt khi cạn nước thùng chứa.
- **Mục tiêu nâng cấp:**
  1. **Điều khiển từ xa qua Smartphone:** Cài đặt lịch tưới/nghỉ và Bật/Tắt thủ công từ bất kỳ đâu qua 4G/Wifi.
  2. **Tưới thông minh thích ứng thời tiết (Thuật toán Bù nhiệt):** Tự động tạm dừng tưới khi có mưa; tự động tăng thời lượng tưới khi nhiệt độ > 35°C để rễ cây không bị khô héo.
  3. **Bảo vệ phần cứng & Cảnh báo Mất điện:** Tự động ngắt máy bơm 220V khi cạn nước; gửi thông báo PUSH khi điện lưới 220V bị cúp nhờ mạch pin dự phòng Mini UPS 5V.
  4. **Chống nhiễu & Cơ chế Offline Failsafe:** Lắp mạch **RC Snubber** dập xung máy bơm chống reset ESP32. Khi mất mạng Wifi, ESP32 vẫn tự động tưới chu kỳ độc lập 24/7.

---

## 2. 🔌 Kiến trúc Phần cứng & Linh kiện Chi tiết (Đã Tối ưu)

### 2.1. Danh sách linh kiện

| STT | Tên linh kiện | Số lượng | Vai trò / Thông số | Đóng góp Tối ưu |
|-----|---------------|----------|-------------------|------------------|
| 1 | **ESP32 DevKit V1** | 1 | Vi điều khiển trung tâm (Wifi + Bluetooth + Dual Core) | Xử lý Task Failsafe & Cloud |
| 2 | **Nguồn xung 5V 2A & Mạch Mini UPS 5V** | 1 | Cấp nguồn 5V DC & duy trì pin sạc dự phòng cho ESP32 | **Gửi cảnh báo PUSH khi cúp điện 220V** |
| 3 | **Module Relay 1 Kênh (5V) + RC Snubber** | 1 | Có Opto cách ly quang + Mạch dập xung RC Snubber | **Triệt tiêu nhiễu xung điện từ khi ngắt/mở bơm 220V** |
| 4 | **Cảm biến Mưa** | 1 | Tấm gạt mưa kim loại chống oxy hóa / Quang học | Nhắc vệ sinh 2 tuần/lần trên App |
| 5 | **Cảm biến Nhiệt độ & Độ ẩm DHT22/SHT30** | 1 | Đo nhiệt độ (°C) và độ ẩm không khí (%) | **Chạy thuật toán bù nhiệt độ > 35°C** |
| 6 | **Phao cảm biến mực nước (Float Switch)** | 1 | Phao công tắc từ gắn thùng chứa | Ngắt máy bơm ngay khi cạn nước |
| 7 | **Cảm biến Ánh sáng (BH1750 / LDR)** | 1 | Đo lux ánh sáng mặt trời | Dự phòng Giai đoạn 2 (Cắt nắng tự động) |
| 8 | **Tủ điện nhựa IP65 & Dây nối** | 1 | Tủ bảo vệ chống nước chứa board mạch | Tận dụng tủ điện cũ |

### 2.2. Sơ đồ Đấu nối Chân (Pinout Diagram)

```
                       +-------------------+
                       |    ESP32 DEVKIT   |
                       |                   |
        [5V Mini UPS] -| VIN           GND |---- [GND Chung]
        [3.3V Out] ----| 3V3          GPIO2 |---- [Đèn LED Trạng thái]
   [Relay Bơm In] ----| GPIO26        GPIO4 |---- [DHT22 Data]
  [Cảm biến Mưa DO]---| GPIO27       GPIO14 |---- [Phao Cạn Nước DO]
 [Giám sát 220V In]---| GPIO12       GPIO13 |---- [BH1750 / LDR Data]
                       +-------------------+
```

* **Đấu nối chống nhiễu cho Máy bơm 220V:**
  * Mạch **RC Snubber** (Điện trở 100 Ohm + Tụ 0.1uF 400V) đấu **song song với 2 tiếp điểm COM - NO của Relay** (hoặc song song với 2 cuộn dây đầu vào máy bơm 220V).

---

## 3. 🧠 Logic Vận hành & Cơ chế Bù nhiệt (State Machine)

ESP32 chạy chu kỳ kiểm tra mỗi 1 giây (`1000ms`) và đưa ra quyết định theo thứ tự **ưu tiên từ cao xuống thấp**:

```
 ┌─────────────────────────────────────────────────────────┐
 │ LEVEL 1: Phao mực nước = CẠN?                             │
 │ ➔ NGẮT BƠM TỨC THÌ + GỬI PUSH ALARM CANH BÁO            │
 └────────────────────────────┬────────────────────────────┘
                              │ NO
                              ▼
 ┌─────────────────────────────────────────────────────────┐
 │ LEVEL 2: Điện lưới 220V = CÚP (Cảm biến Vin 220V Off)?    │
 │ ➔ NGUỒN MINI UPS DUY TRÌ ESP32 + GỬI PUSH "MẤT ĐIỆN 220V"│
 └────────────────────────────┬────────────────────────────┘
                              │ NO
                              ▼
 ┌─────────────────────────────────────────────────────────┐
 │ LEVEL 3: Cảm biến Mưa = CÓ MƯA?                          │
 │ ➔ TẠM DỪNG BƠM (Rain Override)                          │
 └────────────────────────────┬────────────────────────────┘
                              │ NO
                              ▼
 ┌─────────────────────────────────────────────────────────┐
 │ LEVEL 4: Lệnh Bật/Tắt thủ công từ App Smartphone?         │
 │ ➔ BẬT/TẮT THEO LỆNH NGƯỜI DÙNG                           │
 └────────────────────────────┬────────────────────────────┘
                              │ NO
                              ▼
 ┌─────────────────────────────────────────────────────────┐
 │ LEVEL 5: Thuật toán Bù nhiệt & Chu kỳ Timer              │
 │ ➔ Nếu Temp > 35°C: Thời gian Tưới +30%, Thời gian Nghỉ -20%│
 │ ➔ Nếu Bình thường: Bật X phút (VD: 15p) - Nghỉ Y phút (VD: 30p)│
 └─────────────────────────────────────────────────────────┘
```

---

## 4. 🌐 Tính năng App & Cảnh báo Nâng cao

* **Bù nhiệt độ thông minh:** App hiển thị trạng thái "Đang bù nhiệt giữa trưa (Nhiệt độ > 35°C)".
* **Cảnh báo mất điện lưới PUSH Alert:** Khi cúp điện 220V, điện thoại nhận ngay cảnh báo: *"⚠️ CẢNH BÁO: Hệ thống bị mất điện lưới 220V! ESP32 đang chạy nguồn dự phòng."*
* **Thông báo Bảo trì Định kỳ:** Mỗi 14 ngày, App tự động xuất thông báo: *"🔔 NHẮC NHỞ: Hãy lau chùi vệ sinh bề mặt tấm cảm biến mưa ngoài trời."*

---

## 5. 🧪 Kịch bản Kiểm thử Tối ưu (Verification Test)

1. **Test Bù nhiệt:** Thổi hơi nóng vào DHT22 (> 35°C) ➔ Thời gian tưới tăng 30%, thời gian nghỉ giảm 20%.
2. **Test Cảnh báo Mất điện:** Ngắt CB điện 220V ➔ ESP32 sống nhờ Mini UPS 5V gửi PUSH "Mất điện 220V" trong `< 5 giây`.
3. **Test Chống nhiễu Máy bơm:** Bật/Tắt máy bơm 220V 100 lần liên tục ➔ ESP32 không bị reset hay giật đơ màn hình nhờ mạch RC Snubber.
4. **Test Mưa & Cạn Nước:** Nhỏ giọt nước / nhấc phao ➔ Bơm ngắt tức thì & gửi cảnh báo.

---

## 🚀 6. Lộ trình Phát triển Tiếp theo (Roadmap)

* **Giai đoạn 1 (Hiện tại):** Hoàn thiện phần cứng Mức 1, Blynk Cloud, Failsafe Timer, Bù nhiệt độ, Mini UPS & RC Snubber.
* **Giai đoạn 2 (Tương lai):** 
  * Tích hợp Cảm biến Ánh sáng (BH1750) điều khiển rèm/lưới cắt nắng tự động.
  * Tự thiết kế ứng dụng di động riêng (Flutter / React Native).
  * Tích hợp thêm cảm biến đo nồng độ dinh dưỡng **EC** và độ pH trong nước.

---
*Tài liệu lưu hành nội bộ dự án Hydroponic IoT Upgrade — Đã tích hợp đóng góp từ team.*
