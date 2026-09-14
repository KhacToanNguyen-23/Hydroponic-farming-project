# ESP32 Smart Irrigation IoT System for 5-Tower Hydroponics

[![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Hardware-ESP32%20DevKit-green.svg)](https://www.espressif.com/)
[![Protocol](https://img.shields.io/badge/Protocol-MQTT%20%7C%20Blynk-orange.svg)](https://blynk.io/)

> [!IMPORTANT]
> **PROJECT SCOPE NOTICE:**  
> This project focuses exclusively on **developing software, firmware, and integrating IoT sensors/controllers** to add smart capabilities to an **EXISTING AND FULLY OPERATIONAL 5-tower recirculating hydroponic system**.  
>  
> This project does **NOT** involve building hydroponic towers, mechanical plumbing, structural frames, or constructing a hydroponic farm from scratch.

---

## Project Context and Upgrade Motivation

The existing 5-tower outdoor hydroponic farming system is already operational. However, its original control box relied on a basic **manual electronic timer switch**:

* **Legacy Limitations:**
  * Adjusting irrigation/pause durations required physical interaction with buttons at the outdoor control panel.
  * Irrigation ran strictly on fixed time slots. During **heavy rain**, the system continued watering, leading to wasted electricity, root oversaturation, and nutrient solution dilution.
  * In the event of a **depleted water reservoir**, the pump continued to run, risking 220V AC pump motor burn-out.
  * Power outages caused the entire system to go down silently without any remote notification.
  * Inductive voltage spikes from switching 220V AC pump motors could cause microcontroller instability or resets.

* **Software and IoT Upgrade Solution:**
  * Addition of an **ESP32 microcontroller** with Wi-Fi and 4G connectivity.
  * Integration of rain detection sensors, ambient climate sensors (DHT22/SHT30), reservoir float switches, and a 5V Mini UPS backup module.
  * Inclusion of **RC Snubber noise filters** across 220V relay contacts to suppress inductive spikes and prevent ESP32 resets.
  * Development of **C++ Firmware (Dual-Core execution, Climate Compensation, and Offline Failsafe)** & **Mobile Application Interface (Blynk / MQTT)**.

---

## Core Software & IoT Features

- **Remote Control and Configuration:** Adjust watering and pause durations, or manually trigger the pump via 4G/Wi-Fi from anywhere.
- **Climate Compensation Algorithm:** Automatically adjusts irrigation intervals when ambient temperature exceeds 35°C (increases watering duration by 30% and reduces pause duration by 20%) to prevent root dehydration during hot afternoons.
- **Rain Sensor Override:** Detects rainfall instantly and suspends current watering cycles to preserve nutrient concentrations and save energy. Includes periodic maintenance reminders on the app for sensor cleaning.
- **Dry-Run Protection:** Triggers an immediate pump shutdown via relay when the reservoir float switch detects low water levels, accompanied by push notification alerts.
- **Power Outage Alert:** A 5V Mini UPS backup circuit keeps the ESP32 alive during 220V grid failure, triggering an immediate "220V Power Grid Lost" push alert to the mobile device.
- **RC Snubber Noise Filtering:** Hardware RC snubber circuit prevents inductive EMF spikes from restarting the ESP32 during pump motor switching operations.
- **Offline Failsafe Execution:** If Wi-Fi or Internet connectivity is interrupted, the ESP32 automatically reverts to internal interval timers stored in non-volatile flash memory (`Preferences`), ensuring uninterrupted 24/7 operation.

---

## Hardware Pinout Diagram

```
                       +-------------------+
                       |    ESP32 DEVKIT   |
                       |                   |
        [5V Mini UPS] -| VIN           GND |---- [Common GND]
        [3.3V Out] ----| 3V3          GPIO2 |---- [Status LED]
   [Pump Relay In] ----| GPIO26        GPIO4 |---- [DHT22 Data]
  [Rain Sensor DO] ----| GPIO27       GPIO14 |---- [Float Switch DO]
 [220V Grid Sense] ----| GPIO12       GPIO13 |---- [Light Sensor BH1750]
                       +-------------------+
```

---

## Repository Structure

```text
Hydroponic-farming-project/
├── README.md                           # Project documentation (This file)
├── DOC_HE_THONG_THUY_CANH_ESP32.md     # Team technical documentation
├── feature_list.json                   # Feature tracking and verification requirements
├── image/                              # Hardware and current system setup photos
│   ├── setup.jpg
│   ├── electronictimer.jpg
│   ├── trunuoc.jpg
│   └── ...
└── plans/                              # Detailed software planning and specifications
    ├── hydroponic-esp32-level1/
    │   ├── spec.md                     # Technical requirements specification
    │   ├── plan.md                     # Overall implementation plan
    │   ├── phase-01-hardware-pinout.md # Circuit and pinout design
    │   ├── phase-02-firmware-core.md   # Firmware state machine design
    │   ├── phase-03-blynk-mqtt-integration.md # Cloud app & push alert integration
    │   └── phase-04-failsafe-testing.md       # Verification test suite
    └── reports/                        # Initial brainstorm and analysis reports
```

---

## Related Documentation

- [Technical Documentation (Vietnamese)](file:///d:/Project/FptProject/Hydroponic-farming-project/DOC_HE_THONG_THUY_CANH_ESP32.md)
- [Technical Specification (spec.md)](file:///d:/Project/FptProject/Hydroponic-farming-project/plans/hydroponic-esp32-level1/spec.md)
- [Implementation Plan (plan.md)](file:///d:/Project/FptProject/Hydroponic-farming-project/plans/hydroponic-esp32-level1/plan.md)

---

*Project developed for Hydroponic Smart Agriculture Software & IoT Upgrade.*
