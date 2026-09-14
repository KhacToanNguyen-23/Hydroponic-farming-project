# Spec: Nâng cấp Điều khiển & Tưới Thông minh 5 Trụ Thủy canh Ngoài trời qua ESP32 (Mức 1)

**Date:** 2026-09-14  
**Status:** Approved (Updated with Team Feedback)  

---

## Problem Statement
Hệ thống 5 trụ thủy canh hồi lưu ngoài trời hiện tại dùng bộ hẹn giờ điện tử cố định tại tủ điện, không thể cài đặt từ xa và không điều chỉnh được thời lượng tưới khi trời mưa hoặc nắng nóng bất thường, dẫn đến lãng phí điện/nước, làm loãng dung dịch hoặc nguy cơ cháy máy bơm khi hết nước trong thùng chứa.

---

## User Stories

- **[P1]** As a Người quản lý hệ thống thủy canh, I want to Cài đặt thời gian và tần suất tưới từ xa qua ứng dụng di động so that Tôi không cần phải ra tận tủ điện để chỉnh thủ công.  
  *Accepted when:* Thay đổi thông số timer trên App di động và ESP32 cập nhật lịch tưới thành công trong vòng < 5 giây.

- **[P1]** As a Hệ thống thủy canh tự động, I want to Tự động ngắt/giảm tưới khi phát hiện trời mưa so that Tránh ngập úng rễ, làm loãng dung dịch dinh dưỡng và tiết kiệm điện.  
  *Accepted when:* Cảm biến mưa bị ướt, ESP32 ngắt ngay lập tức lượt tưới hiện tại và tạm hoãn lịch tưới tiếp theo cho đến khi cảm biến khô.

- **[P1]** As a Hệ thống bảo vệ phần cứng, I want to Tự động tắt máy bơm và phát cảnh báo khi nước trong thùng chứa bị cạn so that Máy bơm 220V không bị cháy do chạy khô.  
  *Accepted when:* Phao mực nước hạ xuống mức cạn, Relay máy bơm ngắt ngay lập tức và phát thông báo PUSH "Cảnh báo cạn nước" tới điện thoại.

- **[P2]** As a Người trồng rau, I want to Theo dõi nhiệt độ & độ ẩm không khí và tự động điều chỉnh chu kỳ tưới khi thời tiết quá nóng so that Rễ cây không bị khô héo vào giữa trưa nắng gắt (> 35°C).  
  *Accepted when:* Nhiệt độ đo được từ DHT22/SHT30 > 35°C, ESP32 tự động tăng thời gian tưới thêm 30% và giảm thời gian nghỉ 20%.

- **[P2]** As a Người quản lý hệ thống, I want to Nhận thông báo tức thì khi điện lưới 220V bị cúp so that Biết được sự cố điện để kịp thời xử lý.  
  *Accepted when:* Điện lưới 220V ngắt, mạch pin dự phòng Mini UPS 5V giữ ESP32 hoạt động thêm và gửi cảnh báo PUSH "Mất điện lưới 220V" tới điện thoại.

- **[P3]** *(Out of scope - Giai đoạn 2)* Tự động đo cường độ ánh sáng (BH1750/LDR) điều khiển lưới cắt nắng và châm phân dinh dưỡng dựa trên cảm biến EC & pH.

---

## Functional Requirements

1. **FR-01:** ESP32 kết nối Wifi và đồng bộ thời gian thực từ Internet (NTP Server) để chạy timer chính xác.
2. **FR-02:** ESP32 cho phép thiết lập chế độ tưới Chu kỳ (VD: Bật X phút, Tắt Y phút) hoặc Theo khung giờ cố định trong ngày thông qua App IoT (Blynk / MQTT).
3. **FR-03:** Đọc tín hiệu Digital/Analog từ Cảm biến mưa. Khi phát hiện mưa (trạng thái LOW/mưa), kích hoạt chế độ "Rain Override" ngắt Relay máy bơm.
4. **FR-04:** Đọc tín hiệu từ Phao cảm biến mực nước. Ngắt Relay máy bơm khi mức nước dưới ngưỡng an toàn và gửi sự kiện cảnh báo.
5. **FR-05:** Đọc dữ liệu Nhiệt độ & Độ ẩm không khí từ cảm biến DHT22/SHT30 và chạy thuật toán bù trừ thời gian tưới/nghỉ theo ngưỡng nhiệt độ.
6. **FR-06:** Cơ chế Failsafe (Offline mode): Nếu mất kết nối Wifi, ESP32 vẫn tự động chạy lịch tưới chu kỳ mặc định lưu trong bộ nhớ Flash (`Preferences`).
7. **FR-07:** Phát hiện trạng thái mất nguồn 220V qua chân giám sát điện áp và phát thông báo PUSH qua nguồn pin dự phòng Mini UPS 5V.
8. **FR-08:** Nhắc nhở vệ sinh cảm biến mưa định kỳ mỗi 14 ngày qua thông báo trên App.

---

## Non-Functional Requirements

- **Latency:** Thời gian phản hồi điều khiển Bật/Tắt máy bơm thủ công từ App < 2 giây.
- **Reliability:** Mạch có lắp thêm **Mạch lọc RC Snubber (dịch chuyển pha & dập xung)** song song tiếp điểm Relay/Máy bơm 220V để triệt tiêu nhiễu tải cảm, chống reset giật ESP32 khi đóng ngắt bơm.
- **Safety:** Relay cách ly điện áp điều khiển 5V DC với điện áp máy bơm 220V AC qua Optocoupler.

---

## Success Criteria

- [ ] Điều khiển Bật/Tắt máy bơm 220V từ xa qua Smartphone đạt tỉ lệ thành công > 99%.
- [ ] Khi nhỏ nước lên cảm biến mưa, máy bơm ngắt trong vòng < 2 giây.
- [ ] Khi nhấc phao mực nước xuống (giả lập cạn nước), máy bơm ngắt ngay lập tức và gửi thông báo về điện thoại.
- [ ] Nhiệt độ > 35°C ➔ ESP32 tự điều chỉnh thời gian tưới tăng 30% và nghỉ giảm 20%.
- [ ] Ngắt CB điện 220V ➔ Điện thoại nhận PUSH "Cảnh báo mất điện lưới 220V" trong vòng < 5 giây.
- [ ] Khởi động/tắt máy bơm 100 lần liên tục không gây reset vi điều khiển ESP32 nhờ mạch lọc RC Snubber.

---

## Out of Scope

- Tự động điều khiển lưới cắt nắng tự động (LDR/BH1750).
- Đo nồng độ dinh dưỡng EC và nồng độ pH dung dịch.
- Tự động bơm châm thêm nước vào thùng chứa.
- Xây dựng ứng dụng di động Native riêng (dành cho Giai đoạn phát triển tiếp theo).

---

## Assumptions

- Vị trí đặt 5 trụ thủy canh có phủ sóng Wifi đủ mạnh (-70 dBm trở lên).
- Nguồn điện 220V AC tại vị trí tủ điện hoạt động ổn định.
