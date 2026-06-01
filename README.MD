# 💡 LumiTech Light Therapy Device

> An ESP32/ESP8266-powered dual-channel light therapy lamp with physical button controls and a real-time browser-based interface.

---

## Overview

LumiTech is an embedded firmware project for a **Wi-Fi connected light therapy lamp** that drives independent White LED and UVB LED channels. Intensity and mode can be controlled via physical buttons on the device **or** from any browser on the local network — no app required.

The async web server keeps the main loop free for responsive physical button handling, with debounced inputs and immediate LED updates.

---

## Features

- **Dual-channel PWM output** — independent White LED (GPIO 27) and UVB LED (GPIO 14) at 34 kHz
- **3 operating modes** — Off → White LED only → White LED + UVB (cycled via button or browser)
- **3 lux levels** — Low (30%), Medium (65%), High (80%) duty cycle
- **Physical buttons** — debounced mode and lux controls (GPIO 18, 19)
- **Async web interface** — served from PROGMEM via `ESPAsyncWebServer`
- **Real-time state polling** — browser polls `/state` every second to reflect current mode and lux level
- **Dual platform support** — compiles for both ESP32 and ESP8266 via `#ifdef`

---

## Hardware Requirements

| Component | Details |
|---|---|
| Microcontroller | ESP32 or ESP8266 |
| White LED output | GPIO 27 (PWM Channel 0) |
| UVB LED output | GPIO 14 (PWM Channel 1) |
| Mode button | GPIO 18 |
| Lux button | GPIO 19 |
| Network | 2.4 GHz Wi-Fi access point |

---

## Operating Modes

| Mode | White LED | UVB LED |
|---|---|---|
| 0 — Off | ✗ | ✗ |
| 1 — White Only | ✓ | ✗ |
| 2 — White + UVB | ✓ | ✓ |

Both LEDs always run at the currently selected lux level.

## Lux Levels

| Level | Duty Cycle |
|---|---|
| Low | 30% |
| Medium | 65% |
| High | 80% |

---

## PWM Configuration

| Parameter | Value |
|---|---|
| Frequency | 34,000 Hz |
| Resolution | 8-bit (0–255) |
| White LED channel | 0 |
| UVB LED channel | 1 |

---

## Getting Started

### 1. Configure Wi-Fi credentials

Open the firmware file and fill in your network details:

```cpp
const char* ssid = "your-network-name";
const char* password = "your-password";
```

### 2. Flash the firmware

Open the file in Arduino IDE with the appropriate board package installed (ESP32 or ESP8266), then upload to your device.

### 3. Connect and control

After boot, open the Serial Monitor at **115200 baud**. Once connected to Wi-Fi, the device prints its local IP:

```
Light Therapy Lamp Initialized.
Connecting to WiFi...
192.168.x.xxx
```

Navigate to that IP in any browser on the same network to open the control panel, or use the physical buttons directly.

---

## Web API

The device exposes three simple HTTP endpoints:

| Endpoint | Method | Description |
|---|---|---|
| `/` | GET | Serves the web control UI |
| `/update` | GET | Cycles to the next mode (0 → 1 → 2 → 0) |
| `/updateluxLevel` | GET | Cycles to the next lux level (Low → Med → High → Low) |
| `/state` | GET | Returns current mode and lux level as plain text |

The browser UI polls `/state` every 1000 ms to keep the display in sync.

---

## How It Works

On boot, PWM channels are configured at 34 kHz / 8-bit resolution and attached to their respective GPIO pins. The `ESPAsyncWebServer` handles all HTTP requests via non-blocking callbacks — the web UI and API routes are registered in `setup()` and served from PROGMEM, requiring no filesystem.

The `loop()` handles only physical button input: both buttons are debounced (50 ms threshold) and cycle their respective state variable on each press, immediately calling `updateLEDs()` to apply the change.

`updateLEDs()` reads the current `mode` and `luxLevel` and writes the appropriate duty cycle to each PWM channel — or zero if that channel is inactive in the current mode.

---

## Dependencies

Install via the Arduino Library Manager or PlatformIO:

| Library | Platform | Source |
|---|---|---|
| `WiFi.h` | ESP32 | Arduino core (built-in) |
| `ESP8266WiFi.h` | ESP8266 | ESP8266 Arduino core (built-in) |
| [`AsyncTCP`](https://github.com/me-no-dev/AsyncTCP) | ESP32 | me-no-dev (manual install) |
| [`ESPAsyncTCP`](https://github.com/me-no-dev/ESPAsyncTCP) | ESP8266 | me-no-dev (manual install) |
| [`ESPAsyncWebServer`](https://github.com/me-no-dev/ESPAsyncWebServer) | Both | me-no-dev (manual install) |
| `Arduino.h` | Both | Arduino core (built-in) |

> **Note:** `AsyncTCP`, `ESPAsyncTCP`, and `ESPAsyncWebServer` are not available in the Arduino Library Manager. Download them from GitHub and install manually, or add via PlatformIO (`me-no-dev/ESPAsyncWebServer`).

---

## Safety Notice

> ⚠️ **UVB radiation can cause skin and eye damage.** Always use appropriate shielding and protective eyewear when working with UVB LEDs. This project is intended for prototyping and research purposes. Consult a medical professional before using for any therapeutic application.

---

## Credits

- Original repository: [@proscio/LightTherapy](https://github.com/proscio/LightTherapy)
- This fork maintained by [@pputrevu-sys](https://github.com/pputrevu-sys)

---

## License

This project is open source. Refer to the original repository for license details.
