# 🌿 Nâng cấp Hệ thống Tưới Thông minh 5 Trụ Thủy canh qua ESP32 IoT

[![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Hardware-ESP32%20DevKit-green.svg)](https://www.espressif.com/)
[![Protocol](https://img.shields.io/badge/Protocol-MQTT%20%7C%20Blynk-orange.svg)](https://blynk.io/)

> [!IMPORTANT]
> **PHẠM VI DỰ ÁN (PROJECT SCOPE):**  
> Dự án này tập trung **phát triển & tích hợp giải pháp phần mềm IoT, vi điều khiển ESP32 và các cảm biến hỗ trợ** nhằm nâng cấp thông minh hóa cho một hệ thống trồng rau thủy canh hồi lưu 5 trụ **ĐÃ CÓ SẴN VÀ ĐANG VẬN HÀNH THỰC TẾ**.  
>  
> ⚠️ **Dự án KHÔNG bao gồm:** Thi công khung giàn, làm trụ trồng, lắp ống cơ khí hay tự tay build hệ thống thủy canh từ con số 0.

---

## 📌 Bối cảnh & Lý do Nâng cấp

Hệ thống trồng rau thủy canh 5 trụ ngoài trời hiện tại đã vận hành ổn định. Tuy nhiên, tủ điện cũ chỉ trang bị **bộ hẹn giờ điện tử công tắc cơ thủ công**:
* **Hạn chế cũ:**
  * Muốn thay đổi thời gian tưới/nghỉ phải ra tận tủ điện để bấm nút cài thủ công.
  * Tưới cố định theo giờ nên khi **trời mưa lớn**, hệ thống vẫn tưới ➔ Lãng phí điện, ngập úng rễ và làm loãng dung dịch dinh dưỡng.
  * Khi thùng chứa bị **cạn nước**, máy bơm vẫn chạy ➔ Nguy cơ cháy máy bơm 220V.
  * Không theo dõi được nhiệt độ và độ ẩm thực tế ngoài vườn khi đi xa.

* **Giải pháp Phần mềm & IoT mới:**
  * Bổ sung vi điều khiển **ESP32** kết nối Wifi/4G.
  * Tích hợp cảm biến mưa, cảm biến nhiệt/ẩm (DHT22) và phao cảm biến cạn nước.
  * Phát triển **Firmware C++ (Dual Core + Offline Failsafe)** & **Giao diện di động (Blynk / MQTT)** để điều khiển và giám sát từ xa.

---

## ✨ Tính năng Nổi bật của Phần mềm IoT

- 📱 **Điều khiển & Cài đặt Từ xa (Remote Control):** Thay đổi số phút Bật tưới / Nghỉ tưới và Bật/Tắt bơm tức thì qua 4G/Wifi từ bất kỳ đâu.
- 🌧️ **Tự động Tạm hoãn khi Mưa (Rain Override):** Phát hiện mưa ngay lập tức ➔ Ngắt lượt tưới hiện tại để bảo vệ dinh dưỡng và tiết kiệm điện.
- 🛡️ **Bảo vệ Máy Bơm chống Cháy Khô (Dry-Run Protection):** Phao mực nước hạ xuống mức cạn ➔ Relay lập tức ngắt máy bơm 220V và gửi thông báo cảnh báo PUSH tới điện thoại.
- ⚡ **Cơ chế Tự chủ Offline (Failsafe Mode):** Khi mất kết nối Wifi/Internet, ESP32 tự nhảy về lịch tưới chu kỳ lưu trong bộ nhớ Flash (`Preferences`), duy trì tưới đúng giờ 24/7 mà không bị gián đoạn.

---

## 🔌 Sơ đồ Đấu nối Phần cứng (Level 1)

```
                       +-------------------+
                       |    ESP32 DEVKIT   |
                       |                   |
        [5V Nguồn] ----| VIN           GND |---- [GND Chung]
        [3.3V Out] ----| 3V3          GPIO2 |---- [LED Trạng thái]
   [Relay Bơm In] ----| GPIO26        GPIO4 |---- [DHT22 Data]
  [Cảm biến Mưa DO]---| GPIO27       GPIO14 |---- [Phao Cạn Nước DO]
                       +-------------------+
```

---

## 📂 Cấu trúc Thư mục Dự án

```text
Hydroponic-farming-project/
├── README.md                           # Tài liệu giới thiệu dự án (File này)
├── DOC_HE_THONG_THUY_CANH_ESP32.md     # Tài liệu hướng dẫn kỹ thuật cho thành viên team
├── feature_list.json                   # Danh mục quản lý tính năng và tiêu chí nghiệm thu
├── image/                              # Hình ảnh hiện trạng phần cứng & 5 trụ thủy canh
│   ├── setup.jpg
│   ├── electronictimer.jpg
│   ├── trunuoc.jpg
│   └── ...
└── plans/                              # Kế hoạch phát triển phần mềm chi tiết
    ├── hydroponic-esp32-level1/
    │   ├── spec.md                     # Đặc tả yêu cầu kỹ thuật (Specification)
    │   ├── plan.md                     # Kế hoạch triển khai tổng thể
    │   ├── phase-01-hardware-pinout.md # Thiết kế Sơ đồ mạch
    │   ├── phase-02-firmware-core.md   # Thiết kế Firmware & State Machine
    │   ├── phase-03-blynk-mqtt-integration.md # Tích hợp Cloud App & PUSH Alert
    │   └── phase-04-failsafe-testing.md       # Kịch bản kiểm thử nghiệm thu
    └── reports/                        # Báo cáo brainstorm khảo sát ban đầu
```

---

## 📖 Tài liệu Tham khảo

- [Tài liệu Chi tiết cho Team Phát triển](file:///d:/Project/FptProject/Hydroponic-farming-project/DOC_HE_THONG_THUY_CANH_ESP32.md)
- [Đặc tả Kỹ thuật (Spec.md)](file:///d:/Project/FptProject/Hydroponic-farming-project/plans/hydroponic-esp32-level1/spec.md)
- [Kế hoạch Chi tiết (Plan.md)](file:///d:/Project/FptProject/Hydroponic-farming-project/plans/hydroponic-esp32-level1/plan.md)

---

*Project developed for Hydroponic Smart Agriculture Upgrade.*
