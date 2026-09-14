# Phase 02: Lập trình Firmware Core trên ESP32 (Arduino C++)

**Spec Story Mapping:**
- P1: Tự động ngắt/giảm tưới khi trời mưa
- P1: Tự động ngắt bơm khi cạn nước
- FR-02: Chạy timer chu kỳ tưới
- FR-06: Offline Failsafe (vẫn chạy độc lập khi mất Wifi)

---

## 1. Cấu trúc Source Code Firmware (PlatformIO / Arduino IDE)

```
firmware/
├── platformio.ini
└── src/
    ├── main.cpp                // State Machine & Task Scheduler
    ├── config.h                // Định nghĩa Chân GPIO & Hằng số
    ├── sensors.h / .cpp        // Đọc DHT22, Cảm biến Mưa, Phao cạn
    ├── pump_control.h / .cpp   // Điều khiển Relay & Logic Timer
    └── storage.h / .cpp        // Đọc/Ghi bộ nhớ Flash Preferences
```

---

## 2. Logic State Machine Điều khiển Máy Bơm

Mô hình trạng thái máy bơm được tính toán mỗi 1 giây (`1000ms` loop) dựa trên bảng ưu tiên sau:

| Ưu tiên | Trạng thái / Điều kiện | Hành động ESP32 | Lý do |
|---------|------------------------|-----------------|-------|
| 1 (Cao nhất) | Phao mực nước = CẠN (`LOW`) | **NGẮT BƠM NGAY LẬP TỨC** | Bảo vệ chống cháy máy bơm |
| 2 | Cảm biến Mưa = MƯA (`LOW`) | **TẠM DỪNG BƠM** | Tránh úng rễ, tràn thùng |
| 3 | Người dùng Bật Bơm Thủ công trên App | **BẬT BƠM** (Ghi đè) | Ưu tiên lệnh người dùng |
| 4 (Mặc định) | Timer Chu kỳ (Bật X min, Nghỉ Y min) | **BẬT / TẮT Theo Lịch** | Lịch tưới thủy canh cơ bản |

---

## 3. Mã nguồn Minh họa Core Logic (Pseudo Code C++)

```cpp
#include <Preferences.h>
#include <DHT.h>

// Định nghĩa Chân
#define PIN_RELAY 26
#define PIN_RAIN 27
#define PIN_FLOAT_WATER 14
#define PIN_DHT 4

Preferences prefs;

// Cấu hình Timer mặc định (Lưu trong Flash)
uint32_t runTimeMinutes = 15;   // Tưới 15 phút
uint32_t stopTimeMinutes = 30;  // Nghỉ 30 phút

bool isPumpRunning = false;
unsigned long lastStateChangeMs = 0;

void setup() {
    pinMode(PIN_RELAY, OUTPUT);
    pinMode(PIN_RAIN, INPUT_PULLUP);
    pinMode(PIN_FLOAT_WATER, INPUT_PULLUP);
    digitalWrite(PIN_RELAY, LOW); // Tắt bơm ban đầu

    // Tải cấu hình từ Flash
    prefs.begin("hydro_config", false);
    runTimeMinutes = prefs.getUInt("run_time", 15);
    stopTimeMinutes = prefs.getUInt("stop_time", 30);
}

void loop() {
    bool isDry = (digitalRead(PIN_FLOAT_WATER) == LOW); // Phao cạn nước
    bool isRaining = (digitalRead(PIN_RAIN) == LOW);    // Có mưa

    // 1. Bảo vệ chống cháy cạn nước
    if (isDry) {
        setPumpState(false);
        return;
    }

    // 2. Tự động tạm hoãn khi trời mưa
    if (isRaining) {
        setPumpState(false);
        return;
    }

    // 3. Chạy Chu kỳ Timer Offline Failsafe
    unsigned long currentMs = millis();
    unsigned long intervalMs = isPumpRunning ? (runTimeMinutes * 60000UL) : (stopTimeMinutes * 60000UL);

    if (currentMs - lastStateChangeMs >= intervalMs) {
        lastStateChangeMs = currentMs;
        setPumpState(!isPumpRunning); // Đảo trạng thái tưới / nghỉ
    }
}

void setPumpState(bool turnOn) {
    isPumpRunning = turnOn;
    digitalWrite(PIN_RELAY, turnOn ? HIGH : LOW);
}
```
