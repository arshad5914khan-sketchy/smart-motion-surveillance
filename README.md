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
[JPEG image of the intruder]
```

---

## Author

**Mohammed Arshad Khan**  
ECE Undergraduate — SR University, Warangal  
[LinkedIn](https://www.linkedin.com/in/mohammed-arshad-khan-6b27402b6/) | [GitHub](https://github.com/arshad5914khan-sketchy)

CODE WORD:

/*
  ============================================================
  Smart Motion Surveillance System
  Author : Mohammed Arshad Khan
  Board  : ESP32-CAM (AI-Thinker)
  Libs   : UniversalTelegramBot, ArduinoJson, esp32-camera
  ============================================================

  WIRING GUIDE
  ─────────────────────────────────────────────────
  ESP32-CAM (AI-Thinker) — Camera is onboard.

  PIR Sensor (HC-SR501)
    VCC    → 5V
    GND    → GND
    OUT    → GPIO 13  (only free GPIO on AI-Thinker)

  Buzzer (Active)
    +      → GPIO 12
    -      → GND
    NOTE   : Use a transistor (BC547 + 1kΩ) if buzzer
             draws more than 40mA

  Programming
    Use an FTDI programmer (3.3V logic)
    Connect IO0 → GND during upload, disconnect to run.
  ─────────────────────────────────────────────────

  SETUP STEPS
  1. Create a Telegram bot via @BotFather → copy token
  2. Message your bot, then visit:
     https://api.telegram.org/bot<TOKEN>/getUpdates
     to get your chat_id
  3. Paste both below
  4. Set your WiFi credentials
  ─────────────────────────────────────────────────
*/

#include "esp_camera.h"
#include <WiFi.h>
#include <WiFiClientSecure.h>
#include <UniversalTelegramBot.h>
#include <ArduinoJson.h>

// ── USER CONFIGURATION ────────────────────────────
const char* WIFI_SSID      = "YOUR_WIFI_SSID";
const char* WIFI_PASSWORD  = "YOUR_WIFI_PASSWORD";
#define BOT_TOKEN            "YOUR_TELEGRAM_BOT_TOKEN"
#define CHAT_ID              "YOUR_CHAT_ID"
// ─────────────────────────────────────────────────

// ── Pin Definitions ──────────────────────────────
#define PIR_PIN   13
#define BUZZER    12

// ── Timing ───────────────────────────────────────
#define COOLDOWN_MS   15000   // 15s between alerts (avoid spam)

// ── AI-Thinker ESP32-CAM Camera Pins ─────────────
#define PWDN_GPIO_NUM     32
#define RESET_GPIO_NUM    -1
#define XCLK_GPIO_NUM      0
#define SIOD_GPIO_NUM     26
#define SIOC_GPIO_NUM     27
#define Y9_GPIO_NUM       35
#define Y8_GPIO_NUM       34
#define Y7_GPIO_NUM       39
#define Y6_GPIO_NUM       36
#define Y5_GPIO_NUM       21
#define Y4_GPIO_NUM       19
#define Y3_GPIO_NUM       18
#define Y2_GPIO_NUM        5
#define VSYNC_GPIO_NUM    25
#define HREF_GPIO_NUM     23
#define PCLK_GPIO_NUM     22

WiFiClientSecure client;
UniversalTelegramBot bot(BOT_TOKEN, client);
unsigned long lastAlert = 0;

// ─────────────────────────────────────────────────
void initCamera() {
  camera_config_t config;
  config.ledc_channel  = LEDC_CHANNEL_0;
  config.ledc_timer    = LEDC_TIMER_0;
  config.pin_d0        = Y2_GPIO_NUM;
  config.pin_d1        = Y3_GPIO_NUM;
  config.pin_d2        = Y4_GPIO_NUM;
  config.pin_d3        = Y5_GPIO_NUM;
  config.pin_d4        = Y6_GPIO_NUM;
  config.pin_d5        = Y7_GPIO_NUM;
  config.pin_d6        = Y8_GPIO_NUM;
  config.pin_d7        = Y9_GPIO_NUM;
  config.pin_xclk      = XCLK_GPIO_NUM;
  config.pin_pclk      = PCLK_GPIO_NUM;
  config.pin_vsync     = VSYNC_GPIO_NUM;
  config.pin_href      = HREF_GPIO_NUM;
  config.pin_sscb_sda  = SIOD_GPIO_NUM;
  config.pin_sscb_scl  = SIOC_GPIO_NUM;
  config.pin_pwdn      = PWDN_GPIO_NUM;
  config.pin_reset     = RESET_GPIO_NUM;
  config.xclk_freq_hz  = 20000000;
  config.pixel_format  = PIXFORMAT_JPEG;
  config.frame_size    = FRAMESIZE_VGA;   // 640x480
  config.jpeg_quality  = 12;              // 0–63, lower = better quality
  config.fb_count      = 1;

  esp_err_t err = esp_camera_init(&config);
  if (err != ESP_OK) {
    Serial.printf("[ERROR] Camera init failed: 0x%x\n", err);
    while (true) delay(1000);
  }
  Serial.println("[OK] Camera initialized.");
}

// ─────────────────────────────────────────────────
void connectWiFi() {
  Serial.printf("Connecting to %s", WIFI_SSID);
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.printf("\n[OK] Connected. IP: %s\n", WiFi.localIP().toString().c_str());
  client.setCACert(TELEGRAM_CERTIFICATE_ROOT);
}

// ─────────────────────────────────────────────────
void triggerBuzzer(int durationMs) {
  digitalWrite(BUZZER, HIGH);
  delay(durationMs);
  digitalWrite(BUZZER, LOW);
}

// ─────────────────────────────────────────────────
void sendAlertWithPhoto() {
  Serial.println("📸 Capturing image...");

  camera_fb_t* fb = esp_camera_fb_get();
  if (!fb) {
    Serial.println("[ERROR] Camera capture failed.");
    bot.sendMessage(CHAT_ID, "⚠️ Motion detected but camera capture failed!", "");
    return;
  }

  Serial.printf("Image size: %d bytes\n", fb->len);

  // Send text alert first
  bot.sendMessage(CHAT_ID,
    "🚨 *INTRUDER ALERT* 🚨\n"
    "Motion detected at your location!\n"
    "📸 Photo attached.", "Markdown");

  // Send photo
  bool sent = bot.sendPhotoByBinary(
    CHAT_ID,
    "image/jpeg",
    fb->len,
    [](uint8_t* buffer, size_t size, void* data) {
      camera_fb_t* frame = (camera_fb_t*)data;
      memcpy(buffer, frame->buf, frame->len);
    },
    fb
  );

  esp_camera_fb_return(fb);

  if (sent) {
    Serial.println("[OK] Alert + photo sent to Telegram.");
  } else {
    Serial.println("[WARN] Photo send failed, text alert was sent.");
  }
}

// ─────────────────────────────────────────────────
void setup() {
  Serial.begin(115200);
  pinMode(PIR_PIN, INPUT);
  pinMode(BUZZER,  OUTPUT);
  digitalWrite(BUZZER, LOW);

  initCamera();
  connectWiFi();

  bot.sendMessage(CHAT_ID,
    "✅ *Smart Motion Surveillance System Online*\n"
    "Monitoring for intrusions...", "Markdown");

  Serial.println("\n=== Smart Motion Surveillance System ===");
  Serial.println("Waiting for motion...\n");
}

// ─────────────────────────────────────────────────
void loop() {
  int motionState = digitalRead(PIR_PIN);

  if (motionState == HIGH) {
    unsigned long now = millis();
    if (now - lastAlert > COOLDOWN_MS) {
      lastAlert = now;
      Serial.println("⚠  MOTION DETECTED!");

      triggerBuzzer(1000);
      sendAlertWithPhoto();
    } else {
      Serial.printf("Motion detected — cooldown active (%lus remaining)\n",
        (COOLDOWN_MS - (now - lastAlert)) / 1000);
    }
  }

  delay(200);
}

