# smart-motion-surveillance
ESP32-CAM security system that detects intrusion, captures real-time images, and sends instant Telegram photo alerts.
# 📷 Smart Motion Surveillance System

An IoT-based smart security system built on the **ESP32-CAM** that detects human intrusion using a PIR sensor, captures real-time images, triggers an onsite buzzer alarm, and delivers instant photo alerts via **Telegram**.

---

##  Features

- Motion detection using HC-SR501 PIR sensor
- Real-time image capture using ESP32-CAM (OV2640)
- Instant Telegram alert with photo attachment on intrusion
- Active buzzer for onsite audio alarm
- 15-second cooldown to prevent alert spam
- Fully wireless — operates over WiFi

---

##  Hardware Required

| Component | Quantity |
|---|---|
| ESP32-CAM (AI-Thinker) | 1 |
| HC-SR501 PIR Motion Sensor | 1 |
| Active Buzzer | 1 |
| BC547 NPN Transistor + 1kΩ Resistor | 1 |
| FTDI USB-to-Serial Programmer (3.3V) | 1 |
| Jumper Wires | — |
| 5V Power Supply | 1 |

---

##  Wiring

| Component | ESP32-CAM Pin |
|---|---|
| PIR Sensor OUT | GPIO 13 |
| Buzzer (+) | GPIO 12 |
| Buzzer (−) | GND |

>  Use a BC547 transistor between GPIO 12 and buzzer if your buzzer draws more than 40mA.  
> ESP32-CAM GPIO 12 and 13 are the only free GPIOs on AI-Thinker boards.

---

##  Libraries Required

Install via Arduino IDE Library Manager:

```
UniversalTelegramBot   — Brian Lough
ArduinoJson            — Benoit Blanchon
esp32-camera           — Espressif (comes with ESP32 board package)
```

---

##  Telegram Bot Setup

1. Open Telegram → search **@BotFather**
2. Send `/newbot` and follow prompts → copy your **Bot Token**
3. Start a chat with your bot
4. Visit this URL in your browser:
   ```
   https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates
   ```
5. Find `"chat":{"id":XXXXXXXXX}` — that's your **Chat ID**
6. Paste both into the `.ino` file:
   ```cpp
   #define BOT_TOKEN   "YOUR_TELEGRAM_BOT_TOKEN"
   #define CHAT_ID     "YOUR_CHAT_ID"
   ```

---

## ⚙️ Configuration

Open `smart_motion_surveillance.ino` and fill in:

```cpp
const char* WIFI_SSID     = "YOUR_WIFI_SSID";
const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";
#define BOT_TOKEN           "YOUR_TELEGRAM_BOT_TOKEN"
#define CHAT_ID             "YOUR_CHAT_ID"
```

Adjust cooldown (default 15s) if needed:
```cpp
#define COOLDOWN_MS   15000
```

---

##  How to Flash

1. Connect ESP32-CAM to FTDI programmer:
   - ESP32 GND → FTDI GND
   - ESP32 5V  → FTDI VCC (5V)
   - ESP32 U0R → FTDI TX
   - ESP32 U0T → FTDI RX
   - **ESP32 IO0 → GND** (programming mode)
2. Open `smart_motion_surveillance.ino` in Arduino IDE
3. Select Board: **AI Thinker ESP32-CAM**
4. Click **Upload**
5. After upload: **disconnect IO0 from GND**, press ESP32 RESET button
6. Open Serial Monitor at **115200 baud**

---

##  Serial Monitor Output (Example)

```
=== Smart Motion Surveillance System ===
Waiting for motion...

⚠  MOTION DETECTED!
📸 Capturing image...
Image size: 24318 bytes
[OK] Alert + photo sent to Telegram.

Motion detected — cooldown active (12s remaining)
```

##  Telegram Alert Preview

When triggered, you receive:
```
🚨 INTRUDER ALERT 
Motion detected at your location!
📸 Photo attached.
[JPEG image of the intruder]
```

---

## Author

**Mohammed Arshad Khan**  
ECE Undergraduate — SR University, Warangal  
[LinkedIn](https://www.linkedin.com/in/mohammed-arshad-khan-6b27402b6/) | [GitHub](https://github.com/arshad5914khan-sketchy)
