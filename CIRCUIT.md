# FingerAttend — Circuit & Wiring Reference

> ESP8266 NodeMCU · SSD1306 OLED · R307 Fingerprint Sensor · Passive Buzzer

---

## Components Required

| # | Component | Specification |
|---|-----------|--------------|
| 1 | Microcontroller | ESP8266 NodeMCU v1.0 (CP2102) |
| 2 | Display | SSD1306 OLED 128×64 I2C (0.96") |
| 3 | Fingerprint Sensor | R307 / R503 Optical (UART) |
| 4 | Buzzer | Passive buzzer (3.3V compatible) |
| 5 | Power | USB 5V via NodeMCU USB port |

---

## Pin Mapping

### SSD1306 OLED Display (I2C)

| OLED Pin | NodeMCU Pin | Wire Color | Notes |
|----------|-------------|------------|-------|
| VCC | 3.3V | 🔴 Red | 3.3V only — NOT 5V |
| GND | GND | ⚫ Black | Any GND pin |
| SCL | D1 (GPIO5) | 🔵 Blue | I2C clock |
| SDA | D2 (GPIO4) | 🟢 Green | I2C data |

> I2C Address: `0x3C`
> Initialized with: `Wire.begin(D2, D1)`

---

### R307 Fingerprint Sensor (UART via SoftwareSerial)

| Sensor Pin | NodeMCU Pin | Wire Color | Notes |
|------------|-------------|------------|-------|
| VCC | 5V (VIN) | 🔴 Red | Needs 5V — use VIN not 3.3V |
| GND | GND | ⚫ Black | Any GND pin |
| TX (sensor) | D5 (GPIO14) | 🔵 Blue | Sensor TX → MCU RX |
| RX (sensor) | D6 (GPIO12) | 🟢 Green | Sensor RX → MCU TX |

> Baud rate: `57600`
> SoftwareSerial: `SoftwareSerial mySerial(D5, D6)`
> ⚠️ Cross-connect: Sensor TX → D5 (MCU reads), Sensor RX → D6 (MCU writes)

---

### Passive Buzzer

| Buzzer Pin | NodeMCU Pin | Wire Color | Notes |
|------------|-------------|------------|-------|
| + (positive) | D3 (GPIO0) | 🟣 Purple | PWM signal via tone() |
| − (negative) | GND | ⚫ Black | Any GND pin |

> Uses Arduino `tone()` / `noTone()` for frequency control
> ⚠️ Use **passive** buzzer only — active buzzers ignore tone() frequency

---

## Power Summary

```
USB 5V
  └── NodeMCU onboard regulator
        ├── 3.3V pin  ──→  OLED VCC
        └── VIN (5V)  ──→  Fingerprint VCC
```

> The OLED and buzzer run on 3.3V.
> The R307 fingerprint sensor requires 5V — use the VIN pin (bypasses regulator).

---

## Full Wiring Table (Quick Reference)

| From (NodeMCU) | To (Component) | Color | Function |
|----------------|----------------|-------|----------|
| 3.3V | OLED VCC | 🔴 Red | Power |
| GND | OLED GND | ⚫ Black | Ground |
| D1 | OLED SCL | 🔵 Blue | I2C Clock |
| D2 | OLED SDA | 🟢 Green | I2C Data |
| D3 | Buzzer + | 🟣 Purple | PWM tone |
| GND | Buzzer − | ⚫ Black | Ground |
| VIN (5V) | FP VCC | 🔴 Red | Power |
| GND | FP GND | ⚫ Black | Ground |
| D5 | FP TX | 🔵 Blue | Serial RX (MCU reads) |
| D6 | FP RX | 🟢 Green | Serial TX (MCU writes) |

---

## I2C Bus Notes

- Both OLED **SCL and SDA** lines should have **4.7kΩ pull-up resistors** to 3.3V
- Most SSD1306 breakout boards include these resistors onboard
- If OLED doesn't initialize, check pull-ups first

---

## Libraries Required

Install via Arduino IDE → Library Manager:

```
Adafruit SSD1306        by Adafruit
Adafruit GFX Library    by Adafruit
Adafruit Fingerprint    by Adafruit
ESP8266WiFi             (built-in with ESP8266 board package)
ESP8266HTTPClient       (built-in with ESP8266 board package)
ESP8266WebServer        (built-in with ESP8266 board package)
```

Board package URL (paste in Arduino Preferences):
```
http://arduino.esp8266.com/stable/package_esp8266com_index.json
```

Board settings:
```
Board:       NodeMCU 1.0 (ESP-12E Module)
CPU Freq:    80 MHz
Flash Size:  4MB (FS:2MB OTA:~1019KB)
Upload Speed: 115200
```

---

## Schematic (ASCII)

```
                    ┌─────────────────────┐
                    │   NodeMCU ESP8266   │
                    │                     │
  ┌──────────┐      │  3.3V ──────────────┼──→ OLED VCC
  │SSD1306   │      │  GND  ──────────────┼──→ OLED GND
  │OLED      │◄─────┤  D1   ──────────────┼──→ OLED SCL
  │128x64    │      │  D2   ──────────────┼──→ OLED SDA
  └──────────┘      │                     │
                    │  D3   ──────────────┼──→ Buzzer +
  ┌──────────┐      │  GND  ──────────────┼──→ Buzzer −
  │Passive   │◄─────┤                     │
  │Buzzer    │      │  VIN  ──────────────┼──→ FP VCC
  └──────────┘      │  GND  ──────────────┼──→ FP GND
                    │  D5   ◄─────────────┼──  FP TX
  ┌──────────┐      │  D6   ──────────────┼──→ FP RX
  │R307      │◄─────┤                     │
  │Fingerprint│     │  WiFi ═════════════ ┼══→ Hotspot
  └──────────┘      │                     │     (pwm / 12345678)
                    └─────────────────────┘
                               │
                             USB 5V
```

---

## Troubleshooting

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| OLED blank | Wrong I2C address | Try `0x3D` instead of `0x3C` |
| OLED garbled | SDA/SCL swapped | Swap D1 and D2 |
| Fingerprint not found | Using 3.3V | Move to VIN (5V) |
| Fingerprint ghost reads | Sensor noise | Already fixed in code with triple-read validation |
| Buzzer silent | Active buzzer used | Replace with passive buzzer |
| Buzzer always on | D3 floating at boot | Add 10kΩ pull-down resistor on D3 |
| WiFi not connecting | Hotspot not on | Enable hotspot: SSID `pwm` / Pass `12345678` |
| Google Sheets timeout | SSL issue | `client.setInsecure()` already set in code |

---

*FingerAttend — Open Source Fingerprint Attendance System*
*Hardware: ESP8266 · Software: Arduino C++ · Cloud: Google Sheets*
