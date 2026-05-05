# 🪖 Industrial Safety Helmet

![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)
![Platform](https://img.shields.io/badge/Platform-ESP32-blue)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

> ESP32-based smart safety helmet with MPU6050, MQ-135 & DHT11 sensors. Real-time fall detection, fatigue monitoring and emergency alerts via a Supervisor Dashboard.

---

## 📌 Overview

Industrial accidents often go undetected until it's too late. This project proposes a **Smart PPE (Personal Protective Equipment)** solution — an intelligent helmet that continuously monitors the worker's environment and physical state, triggering immediate alerts when danger is detected.

The system detects:
- 🚨 **Falls** (via accelerometer)
- 🌡️ **Critical temperatures**
- ☁️ **Toxic gas exposure**
- 😴 **Worker fatigue**
- 🆘 **Manual SOS trigger**

All alerts are sent in real-time to a **supervisor dashboard** via Blynk IoT cloud.

---

## 🏗️ System Architecture

```
[MPU6050] ──┐
[MQ-135]  ──┤                          ┌──► [LCD Screen]  (Worker)
[DHT11]   ──┼──► [ESP32] ──► Logic ───┼──► [LEDs + Buzzer]
[SOS Btn] ──┤                          └──► [Blynk Cloud] (Supervisor)
[Cancel]  ──┘
```

### State Machine

```
🟢 SAFE  ──────►  🟡 WARNING (15s delay)  ──────►  🔴 EMERGENCY
                        │
                  [Cancel Button]
                        │
                        ▼
                     🟢 SAFE
```

---

## 🔧 Hardware Components

| Component | Role | GPIO Pin |
|-----------|------|----------|
| ESP32 | Main microcontroller | — |
| MPU6050 ⭐ | Accelerometer 6-axis (CORE) | SDA=21, SCL=22 |
| MQ-135 | Gas / pollution detection | GPIO 34 (analog) |
| DHT11 | Temperature + humidity | GPIO 4 |
| LED Red | Emergency indicator | GPIO 2 |
| LED Yellow | Warning indicator | GPIO 13 |
| LED Green | Safe indicator | GPIO 14 |
| Buzzer | Audio alert | GPIO 12 |
| SOS Button | Manual emergency trigger | GPIO 35 |
| Cancel Button | Cancel warning | GPIO 36 |
| LCD I2C | Worker display | SDA=21, SCL=22 |
| Li-Po Battery | Power supply | — |
| Safety Helmet | Physical housing | — |

> ❌ KY-002 (vibration) and KY-039 (pulse) are **not used** — replaced by MPU6050 analysis.

---

## ⚡ Detection Logic

### Alert Thresholds

| Event | Severity | Action | Delay |
|-------|----------|--------|-------|
| Gas > 350 ppm | 🔴 CRITICAL | EMERGENCY | 0s |
| SOS Button | 🔴 CRITICAL | EMERGENCY | 0s |
| Fall (acc > 2g) | 🟡 SUSPECT | WARNING | 15s |
| Tilt > 30s | 🔴 SERIOUS | EMERGENCY | 30s |
| Immobility > 1 min | 🟡 ATTENTION | WARNING | 60s |
| Temperature > 50°C | 🟡 ATTENTION | WARNING | 0s |
| Temperature > 55°C | 🔴 CRITICAL | EMERGENCY | 0s |
| Fatigue score > 60% | 🟡 ATTENTION | WARNING | 5 min |

### 😴 Fatigue Detection Algorithm

The fatigue score is computed from **4 MPU6050 indicators**:

| Indicator | Max Points | Condition |
|-----------|-----------|-----------|
| Prolonged immobility | 30 pts | No movement for 5+ min |
| Low movement rate | 30 pts | < 20 movements/min |
| Average tilt | 20 pts | Average angle > 30° |
| Oscillations | 20 pts | Swaying detected |

**Result:** If total score > 60% → WARNING "Fatigue detected"

---

## 📁 Project Structure

```
Industrial-Safety-Helmet/
│
├── README.md
├── .gitignore
├── LICENSE
│
├── src/
│   ├── main.cpp               # Main ESP32 code
│   ├── sensors.cpp            # MQ-135, DHT11, MPU6050
│   ├── fatigue.cpp            # Fatigue detection algorithm
│   ├── alerts.cpp             # LEDs, Buzzer, State machine
│   └── blynk.cpp              # Cloud & notifications
│
├── hardware/
│   ├── wiring_diagram.png     # Wiring schematic
│   └── components.md          # Components list + GPIO map
│
├── docs/
│   └── project_plan.docx      # Full project documentation
│
└── tests/
    ├── test_mpu6050.cpp        # Fall & fatigue tests
    ├── test_mq135.cpp          # Gas detection tests
    └── test_dht11.cpp          # Temperature tests
```

---

## 🛠️ Setup & Installation

### Prerequisites

- [Arduino IDE](https://www.arduino.cc/en/software) or [PlatformIO](https://platformio.org/)
- ESP32 board support installed)

### Required Libraries

```
DHT sensor library       (Adafruit)
MPU6050                  (Electronic Cats)
LiquidCrystal_I2C
```

### Configuration

Create a `secrets.h` file (never pushed to GitHub):

```cpp
#define WIFI_SSID     "your_wifi_name"
#define WIFI_PASSWORD "your_wifi_password"
#define BLYNK_TOKEN   "your_blynk_token"
```

Then include it in `main.cpp`:

```cpp
#include "secrets.h"
```

---

## 🧪 Testing Scenarios

| Test | Procedure | Expected Result |
|------|-----------|----------------|
| Fall detection | Drop helmet from 50cm | SAFE → WARNING → EMERGENCY |
| Gas detection | Approach gas source | Immediate EMERGENCY |
| Temperature | Heat to 50°C / 55°C | WARNING / EMERGENCY |
| Fatigue | Stay still for 5 min | Fatigue score rises → WARNING |
| SOS Button | Press SOS | Immediate EMERGENCY |
| Cancel Button | Press Cancel in WARNING | Back to SAFE |
| Dashboard| Check app | Real-time data displayed |

---



## 📦 Deliverables

- ✅ Full ESP32 source code (all sensors integrated)
- ✅ Documented fatigue detection algorithm
- ✅ SAFE/WARNING/EMERGENCY state machine
- ✅ dashboard (supervisor)
- ✅ Calibration & maintenance procedure
- ✅ Final demo

---

## 👤 Author

**[BA ABDOULAYE]**  
Academic Project — [UCA / ENSA SAFI]  
Year: 2026

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).
