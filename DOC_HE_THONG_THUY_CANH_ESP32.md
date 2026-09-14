# TÀI LIỆU KỸ THUẬT & HƯỚNG DẪN DỰ ÁN
## Nâng cấp Hệ thống Tưới Thông minh 5 Trụ Thủy canh Ngoài trời qua ESP32 IoT

**Dự án:** Hydroponic Farming IoT Upgrade  
**Phiên bản:** 1.0 (Giai đoạn 1 - Mức 1)  
**Ngày cập nhật:** 14/09/2026  
**Đối tượng đọc:** Các thành viên phát triển, kỹ thuật phần cứng & vận hành hệ thống  

---

## 1. 🎯 Tổng quan & Mục tiêu Dự án

Hệ thống trồng rau thủy canh hồi lưu 5 trụ đứng hiện tại đã đi vào hoạt động ngoài trời. 
- **Hiện trạng cũ:** Hệ thống dùng bộ hẹn giờ điện tử nút bấm cơ thủ công tại tủ điện. Hạn chế: không chỉnh được từ xa, tưới cố định theo giờ nên khi trời mưa gây lãng phí điện/tràn loãng dinh dưỡng, và không tự ngắt khi cạn nước thùng chứa.
- **Mục tiêu nâng cấp:**
  1. **Điều khiển từ xa qua Smartphone:** Cài đặt lịch tưới/nghỉ và Bật/Tắt thủ công từ bất kỳ đâu qua 4G/Wifi.
  2. **Tưới thông minh thích ứng thời tiết:** Tự động tạm dừng tưới khi có mưa, tự điều chỉnh tần suất tưới theo nhiệt độ/độ ẩm môi trường.
  3. **Bảo vệ phần cứng an toàn:** Tự động ngắt máy bơm 220V và gửi thông báo cảnh báo PUSH khi thùng chứa bị cạn nước.
  4. **Cơ chế Offline Failsafe:** Khi mất mạng Wifi, ESP32 vẫn tự động tưới chu kỳ độc lập 24/7.

---

## 2. 🔌 Kiến trúc Phần cứng & Danh sách Linh kiện (Level 1)

### 2.1. Danh sách linh kiện

| STT | Tên linh kiện | Số lượng | Vai trò / Thông số |
|-----|---------------|----------|-------------------|
| 1 | **ESP32 DevKit V1** | 1 | Vi điều khiển trung tâm (Wifi + Bluetooth + Dual Core) |
| 2 | **Nguồn xung 5V 2A** | 1 | Chuyển đổi 220V AC -> 5V DC cấp cho ESP32 & Relay |
| 3 | **Module Relay 1 Kênh (5V)** | 1 | Có Opto cách ly quang, điều khiển Bật/Tắt máy bơm 220V |
| 4 | **Cảm biến Mưa (Raindrop Sensor)** | 1 | Tấm gạt mưa kim loại chống gỉ, phát hiện mưa ngoài trời |
| 5 | **Cảm biến Nhiệt độ & Độ ẩm DHT22** | 1 | Đo nhiệt độ (°C) và độ ẩm không khí (%) xung quanh 5 trụ |
| 6 | **Phao cảm biến mực nước (Float Switch)** | 1 | Phao công tắc từ gắn thùng chứa, ngắt bơm khi cạn |
| 7 | **Tủ điện nhựa IP65 & Dây nối** | 1 | Tủ bảo vệ chống nước chứa board mạch và relay |

### 2.2. Sơ đồ Đấu nối Chân (Pinout Diagram)

```
                       +-------------------+
                       |    ESP32 DEVKIT   |
                       |                   |
        [5V Nguồn] ----| VIN           GND |---- [GND Chung]
        [3.3V Out] ----| 3V3          GPIO2 |---- [Đèn LED Trạng thái]
   [Relay Bơm In] ----| GPIO26        GPIO4 |---- [DHT22 Data]
  [Cảm biến Mưa DO]---| GPIO27       GPIO14 |---- [Phao Cạn Nước DO]
                       +-------------------+
```

* **Dây nguồn Bơm 220V AC:**
  * Dây N (Lạnh) 220V -> Nối trực tiếp vào Máy bơm.
  * Dây L (Nóng) 220V -> Nối qua cổng `COM` và `NO` của Relay -> Nối vào Máy bơm.

---

## 3. 🧠 Logic Vận hành & Cơ chế An toàn (State Machine)

ESP32 chạy chu kỳ kiểm tra mỗi 1 giây (`1000ms`) và đưa ra quyết định theo thứ tự **ưu tiên từ cao xuống thấp**:

```
 ┌─────────────────────────────────────────────────────────┐
 │ LEVEL 1: Phao mực nước = CẠN?                             │
 │ ➔ NGẮT BƠM TỨC THÌ + GỬI PUSH ALARM CANH BÁO            │
 └────────────────────────────┬────────────────────────────┘
                              │ NO
                              ▼
 ┌─────────────────────────────────────────────────────────┐
 │ LEVEL 2: Cảm biến Mưa = CÓ MƯA?                          │
 │ ➔ TẠM DỪNG BƠM (Rain Override)                          │
 └────────────────────────────┬────────────────────────────┘
                              │ NO
                              ▼
 ┌─────────────────────────────────────────────────────────┐
 │ LEVEL 3: Lệnh Bật/Tắt thủ công từ App Smartphone?         │
 │ ➔ BẬT/TẮT THEO LỆNH NGƯỜI DÙNG                           │
 └────────────────────────────┬────────────────────────────┘
                              │ NO
                              ▼
 ┌─────────────────────────────────────────────────────────┐
 │ LEVEL 4: Lịch Chu kỳ Timer mặc định                     │
 │ ➔ TỰ BẬT (VD: 15 PHÚT) ➔ TỰ NGHỈ (VD: 30 PHÚT)          │
 └─────────────────────────────────────────────────────────┘
```

### 🔑 Cơ chế Offline Failsafe (Khi đứt Wifi):
* Giá trị thời gian tưới/nghỉ được lưu bền vững trong bộ nhớ **Flash Preferences** trên ESP32.
* Nếu bị mất Wifi/Internet, ESP32 tự chuyển sang chế độ **Offline Timer**, tiếp tục tưới đúng chu kỳ và ngắt khi mưa/cạn nước bình thường.

---

## 4. 🌐 Kết nối Điều khiển Từ xa (App Mobile & Cloud)

* **Giao thức kết nối:** MQTT / Blynk IoT Cloud.
* **Khả năng truy cập từ xa:**
  * Đặt ESP32 kết nối Wifi tại vườn.
  * Điện thoại bật **4G/5G** hoặc Wifi bất kỳ ở xa vẫn điều khiển được hệ thống.
* **Giao diện App bao gồm:**
  * **Nút bấm Bật/Tắt Bơm** tức thì.
  * **Ô cài đặt số phút Tưới và số phút Nghỉ**.
  * **Đồ thị Nhiệt độ & Độ ẩm** thời gian thực.
  * **Đèn báo Trạng thái:** Báo Mưa & Báo Cạn Nước.

---

## 5. 🧪 Kịch bản Kiểm thử cho Team Kỹ thuật (Verification Test)

1. **Test Điều khiển 4G:** Tắt Wifi trên điện thoại, dùng 4G bấm bật/tắt bơm. Phản hồi yêu cầu `< 2 giây`.
2. **Test Cảm biến Mưa:** Nhỏ giọt nước lên tấm cảm biến mưa ngoài trời ➔ Relay phải ngắt bơm lập tức.
3. **Test Cạn Nước:** Nhấc phao mực nước trong thùng chứa xuống ➔ Relay ngắt bơm ngay lập tức & ứng dụng hiển thị PUSH Alarm.
4. **Test Mất Wifi:** Rút nguồn Modem Wifi ➔ Hệ thống vẫn duy trì tự bật/tắt máy bơm theo đúng số phút đã cài.

---

## 🚀 6. Lộ trình Phát triển Tiếp theo (Roadmap)

* **Giai đoạn 1 (Hiện tại):** Hoàn thiện phần cứng Mức 1, Blynk Cloud & Failsafe Timer.
* **Giai đoạn 2 (Tương lai):** 
  * Tự thiết kế ứng dụng di động riêng (Flutter / React Native).
  * Tích hợp thêm cảm biến đo nồng độ dinh dưỡng **EC** và độ pH trong nước.

---
*Tài liệu lưu hành nội bộ dự án Hydroponic IoT Upgrade.*
