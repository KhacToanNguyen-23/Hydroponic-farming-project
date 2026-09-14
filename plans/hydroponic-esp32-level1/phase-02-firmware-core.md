# Phase 02: Lập trình Firmware Core trên ESP32 (Arduino C++)

**Spec Story Mapping:**
- P1: Tự động ngắt/giảm tưới khi trời mưa
- P1: Tự động ngắt bơm khi cạn nước
- P2: Thuật toán bù nhiệt độ môi trường (> 35°C)
- P2: Cảnh báo cúp điện 220V qua Mini UPS 5V
- FR-02: Chạy timer chu kỳ tưới
- FR-06: Offline Failsafe (vẫn chạy độc lập khi mất Wifi)

---

## 1. State Machine & Thứ tự Ưu tiên Xử lý

Mô hình trạng thái máy bơm được tính toán mỗi 1 giây (`1000ms` loop) dựa trên bảng ưu tiên sau:

| Ưu tiên | Trạng thái / Điều kiện | Hành động ESP32 | Lý do |
|---------|------------------------|-----------------|-------|
| 1 (Cao nhất) | Phao mực nước = CẠN (`LOW`) | **NGẮT BƠM NGAY LẬP TỨC** | Bảo vệ chống cháy máy bơm |
| 2 | Điện 220V = CÚP (`GPIO12 == LOW`) | **DUY TRÌ SỐNG VÀ GỬI ALERT PUSH** | Cảnh báo mất điện qua pin Mini UPS |
| 3 | Cảm biến Mưa = MƯA (`LOW`) | **TẠM DỪNG BƠM** | Tránh úng rễ, tràn thùng |
| 4 | Người dùng Bật Bơm Thủ công trên App | **BẬT BƠM** (Ghi đè) | Ưu tiên lệnh người dùng |
| 5 (Mặc định) | Timer Chu kỳ + **Bù nhiệt độ > 35°C** | **BẬT / TẮT Theo Lịch Bù Nhiệt** | Tránh rễ bị khô héo vào trưa nắng |

---

## 2. Mã nguồn Minh họa Core Logic Bù Nhiệt (C++)

```cpp
#include <Preferences.h>
#include <DHT.h>

#define PIN_RELAY 26
#define PIN_RAIN 27
#define PIN_FLOAT_WATER 14
#define PIN_GRID_SENSE 12
#define PIN_DHT 4

DHT dht(PIN_DHT, DHT22);
Preferences prefs;

uint32_t baseRunTimeMinutes = 15;   // Thời gian tưới gốc
uint32_t baseStopTimeMinutes = 30;  // Thời gian nghỉ gốc

bool isPumpRunning = false;
unsigned long lastStateChangeMs = 0;
bool isGridPowerLostAlertSent = false;

void setup() {
    pinMode(PIN_RELAY, OUTPUT);
    pinMode(PIN_RAIN, INPUT_PULLUP);
    pinMode(PIN_FLOAT_WATER, INPUT_PULLUP);
    pinMode(PIN_GRID_SENSE, INPUT_PULLUP);
    digitalWrite(PIN_RELAY, LOW);

    dht.begin();
    prefs.begin("hydro_config", false);
    baseRunTimeMinutes = prefs.getUInt("run_time", 15);
    baseStopTimeMinutes = prefs.getUInt("stop_time", 30);
}

void loop() {
    bool isDry = (digitalRead(PIN_FLOAT_WATER) == LOW);
    bool isRaining = (digitalRead(PIN_RAIN) == LOW);
    bool isGridLost = (digitalRead(PIN_GRID_SENSE) == LOW);

    // 1. Kiểm tra Cạn Nước
    if (isDry) {
        setPumpState(false);
        return;
    }

    // 2. Kiểm tra Mất điện 220V
    if (isGridLost && !isGridPowerLostAlertSent) {
        sendPushNotification("⚠️ CẢNH BÁO: Mất điện lưới 220V! ESP32 đang dùng pin dự phòng.");
        isGridPowerLostAlertSent = true;
    } else if (!isGridLost) {
        isGridPowerLostAlertSent = false;
    }

    // 3. Kiểm tra Mưa
    if (isRaining) {
        setPumpState(false);
        return;
    }

    // 4. Thuật toán Bù nhiệt độ > 35°C
    float temp = dht.readTemperature();
    uint32_t activeRunTime = baseRunTimeMinutes;
    uint32_t activeStopTime = baseStopTimeMinutes;

    if (!isnan(temp) && temp > 35.0) {
        activeRunTime = (uint32_t)(baseRunTimeMinutes * 1.3);   // Tăng 30% thời lượng tưới
        activeStopTime = (uint32_t)(baseStopTimeMinutes * 0.8); // Giảm 20% thời lượng nghỉ
    }

    // 5. Chạy Chu kỳ Timer Offline Failsafe
    unsigned long currentMs = millis();
    unsigned long intervalMs = isPumpRunning ? (activeRunTime * 60000UL) : (activeStopTime * 60000UL);

    if (currentMs - lastStateChangeMs >= intervalMs) {
        lastStateChangeMs = currentMs;
        setPumpState(!isPumpRunning);
    }
}

void setPumpState(bool turnOn) {
    isPumpRunning = turnOn;
    digitalWrite(PIN_RELAY, turnOn ? HIGH : LOW);
}
```
