# Spec: Nâng cấp Điều khiển & Tưới Thông minh 5 Trụ Thủy canh Ngoài trời qua ESP32 (Mức 1)

**Date:** 2026-09-14  
**Status:** Approved  

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

- **[P2]** As a Người trồng rau, I want to Theo dõi nhiệt độ & độ ẩm không khí xung quanh 5 trụ từ xa so that Biết được thời tiết thực tế tại vườn để điều chỉnh chế độ tưới phù hợp.  
  *Accepted when:* Dữ liệu từ cảm biến DHT22/SHT30 hiển thị liên tục và cập nhật mỗi 30 giây trên ứng dụng di động.

- **[P3]** *(Out of scope - Giai đoạn 2)* Tự động đo và châm phân dinh dưỡng dựa trên cảm biến EC & pH.

---

## Functional Requirements

1. **FR-01:** ESP32 kết nối Wifi và đồng bộ thời gian thực từ Internet (NTP Server) để chạy timer chính xác.
2. **FR-02:** ESP32 cho phép thiết lập chế độ tưới Chu kỳ (VD: Bật X phút, Tắt Y phút) hoặc Theo khung giờ cố định trong ngày thông qua App IoT (Blynk / MQTT).
3. **FR-03:** Đọc tín hiệu Digital/Analog từ Cảm biến mưa. Khi phát hiện mưa (trạng thái LOW/mưa), kích hoạt chế độ "Rain Override" ngắt Relay máy bơm.
4. **FR-04:** Đọc tín hiệu từ Phao cảm biến mực nước. Ngắt Relay máy bơm khi mức nước dưới ngưỡng an toàn và gửi sự kiện cảnh báo.
5. **FR-05:** Đọc dữ liệu Nhiệt độ & Độ ẩm không khí từ cảm biến DHT22/SHT30 và gửi lên Dashboard di động.
6. **FR-06:** Cơ chế Failsafe (Offline mode): Nếu mất kết nối Wifi, ESP32 vẫn tự động chạy lịch tưới chu kỳ mặc định lưu trong bộ nhớ Flash (EEPROM/Preferences).

---

## Non-Functional Requirements

- **Latency:** Thời gian phản hồi điều khiển Bật/Tắt máy bơm thủ công từ App < 2 giây.
- **Reliability:** Hệ thống hoạt động liên tục 24/7, tự khởi động lại và kết nối lại Wifi nếu bị ngắt điện hoặc mất mạng.
- **Safety:** Relay cách ly điện áp điều khiển 5V DC với điện áp máy bơm 220V AC.

---

## Success Criteria

- [ ] Điều khiển Bật/Tắt máy bơm 220V từ xa qua Smartphone đạt tỉ lệ thành công > 99%.
- [ ] Khi nhỏ nước lên cảm biến mưa, máy bơm ngắt trong vòng < 2 giây.
- [ ] Khi nhấc phao mực nước xuống (giả lập cạn nước), máy bơm ngắt ngay lập tức và gửi thông báo về điện thoại.
- [ ] ESP32 tiếp tục tưới đúng chu kỳ khi bị ngắt kết nối Wifi.

---

## Out of Scope

- Đo nồng độ dinh dưỡng EC và nồng độ pH dung dịch.
- Tự động bơm châm thêm nước vào thùng chứa.
- Xây dựng ứng dụng di động Native riêng (dành cho Giai đoạn phát triển tiếp theo).

---

## Assumptions

- Vị trí đặt 5 trụ thủy canh có phủ sóng Wifi đủ mạnh (-70 dBm trở lên).
- Nguồn điện 220V AC tại vị trí tủ điện hoạt động ổn định.
