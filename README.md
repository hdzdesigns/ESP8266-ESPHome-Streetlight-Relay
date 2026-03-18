# 💡 ESP8266 ESPHome Streetlight Relay (V2.0)

An ultra-reliable, autonomous ESPHome firmware designed to control outdoor streetlights or heavy-duty contactors. This system is engineered to function 100% offline by utilizing a temperature-compensated hardware clock, ensuring the light schedule remains accurate even during total network failure or frequent power cycles.

---

## ✨ The Core Philosophy: "True Autonomy"

### 🕒 1. Hardware-Backed Offline Scheduling (DS3231 RTC)
Standard smart relays lose their sense of time when the internet or power drops. This firmware integrates a **DS3231 High-Precision RTC** (HW-084/ZS-042 module) with a CR2032 backup battery to maintain a "source of truth."

* **Superior Accuracy:** Unlike the standard DS1307, the DS3231 is temperature-compensated (TCXO), remaining accurate to within ~2 minutes per year even in varying outdoor temperatures.
* **Platform Compatibility:** Although the hardware is a DS3231, this project utilizes the ESPHome **`ds1307`** time platform, which is register-compatible with the DS3231.
* **NTP Synchronization:** When online, the ESP8266 synchronizes with NTP servers and pushes the accurate time to the hardware clock via the `ds1307.write_time` action.
* **Offline Resilience:** On boot, the ESP8266 reads the hardware clock immediately. It mathematically evaluates if the relay *should* be ON or OFF, preventing the light from staying off until the next scheduled trigger.

### 🎛️ 2. Dynamic Home Assistant Integration
This version replaces static hardcoded times with native **Time Picker** entities in Home Assistant.

* **Flash Persistence:** Changes made via the Home Assistant UI are instantly committed to the ESP8266's internal flash memory (`restore_value: true`). 
* **Instant Feedback:** A 12-hour formatted text sensor ("Active Schedule") provides immediate confirmation on your dashboard whenever a time setting is modified.

### 🛜 3. Fallback Connectivity & Web Server
If your home automation server or router is down, the device remains controllable.

* **`HDz_Relay` Hotspot:** If the Wemos cannot find Wi-Fi for 1 minute, it broadcasts its own password-protected AP.
* **Local Dashboard:** Includes a secure local Web Server (Port 80) to toggle the relay or manually sync the RTC time from your phone’s browser.

### 🥷 4. Stealth LED & Diagnostics
* **LED Timeout:** The onboard status LED remains active only for **90 seconds** after boot or a state change to avoid attracting insects to the enclosure.
* **Live Telemetry:** Monitors Signal Strength (RSSI), Uptime (32-bit overflow protected), and DC Supply Voltage (VCC).

---

## 🛠️ Hardware & Pinout

| Component | ESP8266 Pin | Notes |
| :--- | :--- | :--- |
| **Relay Control** | `GPIO4` (D2) | High-level trigger for 5V relay (Switching 220V Contactor). |
| **I2C SDA** | `GPIO14` (D5) | Data line for RTC Module. |
| **I2C SCL** | `GPIO5` (D1) | Clock line for RTC Module. |
| **Manual Button** | `GPIO12` (D6) | Momentary push button with 50ms software debounce. |
| **Status LED** | `GPIO2` (D4) | Internal Wemos LED (Inverted logic). |

---

## 🔧 Hardware Modification (HW-084 / ZS-042)
To ensure a **10-year battery life** for the RTC when using a standard non-rechargeable CR2032 battery:
1.  **Power:** The module is powered via the **5V rail** for stable I2C logic.
2.  **The "201" Mod:** The **charging resistor (labeled 201)** was removed from the RTC module. This disables the onboard trickle-charging circuit, making it safe for standard lithium coin cells and preventing battery swelling/leakage.

---

## 🔐 Required Secrets Configuration

Ensure your `secrets.yaml` contains the following keys before compiling:

```yaml
# WiFi Configuration
wifi_ssid: "Your_SSID"
wifi_password: "Your_Password"

# Security
api_key: "your_generated_api_key"
ota_password: "your_ota_password"

# Local Web Server Dashboard
web_username: "admin"
web_password: "Your_Secure_Password"

# Hotspot Recovery
ap_password: "Your_AP_Password"