# 💡 ESP8266 ESPHome Streetlight Relay (V2.0)

An ultra-reliable, autonomous ESPHome firmware designed to control outdoor streetlights or heavy-duty contactors. This system is engineered to function 100% offline by utilizing a battery-backed hardware clock, ensuring the light schedule remains accurate even during total network failure or frequent power cycles.

---

## ✨ The Core Philosophy: "True Autonomy"

### 🕒 1. Hardware-Backed Offline Scheduling (DS1307 RTC)
Standard smart relays lose their sense of time when the internet or power drops. This firmware integrates a **DS1307 Real-Time Clock (RTC)** with a CR2032 backup battery to maintain a "source of truth" for time.
* **NTP Synchronization:** When online, the ESP8266 periodically synchronizes with Google/NTP servers and pushes the atomic time to the DS1307 hardware using a native sync action.
* **Offline Resilience:** If the router is off, the ESP8266 reads the hardware clock on boot. It mathematically calculates if it *should* be ON or OFF immediately, ensuring the light is never stuck in the wrong state.
* **Midnight Crossover:** The logic handles schedules that span across midnight (e.g., ON at 6:30 PM, OFF at 6:00 AM) using minute-since-midnight calculations.

### 🎛️ 2. Dynamic Home Assistant Integration
Unlike static firmware, this version exposes native **Time Picker** entities to Home Assistant.
* **Flash Persistence:** Changes made via the Home Assistant UI are instantly committed to the ESP8266's internal flash memory (`restore_value: true`). 
* **Zero-Lag Readout:** A 12-hour formatted text sensor ("Active Schedule") provides instant visual confirmation on your dashboard whenever a time setting is modified.

### 🛜 3. Fallback Connectivity & Web Server
If your home automation server is down, the device remains controllable.
* **`HDz_Relay` Hotspot:** If the Wemos cannot find Wi-Fi for 1 minute, it broadcasts its own password-protected AP.
* **Local Dashboard:** Includes a secure local Web Server (Port 80) to toggle the relay or manually sync the RTC time from your phone’s browser.

### 🥷 4. Stealth LED & Diagnostics
* **LED Timeout:** The onboard status LED remains active only for **90 seconds** after boot or a state change to prevent attracting insects to the enclosure.
* **Live Telemetry:** Monitors Signal Strength (RSSI), Uptime (fixed for 32-bit overflow), and DC Supply Voltage (VCC).

---

## 🛠️ Hardware & Pinout

| Component | ESP8266 Pin | Notes |
| :--- | :--- | :--- |
| **Relay Control** | `GPIO4` (D2) | High-level trigger for 5V relay (Switching 220V Contactor). |
| **I2C SDA** | `GPIO14` (D5) | Data line for DS1307 RTC Module. |
| **I2C SCL** | `GPIO5` (D1) | Clock line for DS1307 RTC Module. |
| **Manual Button** | `GPIO12` (D6) | Momentary push button with 50ms software debounce. |
| **Status LED** | `GPIO2` (D4) | Internal Wemos LED (Inverted logic). |

---

## 🔧 Hardware Modification Note
To ensure a **10-year battery life** for the DS1307 RTC when using a standard non-rechargeable CR2032 battery, the **charging resistor (201)** on the "Tiny RTC" module was removed. This prevents the 5V rail from attempting to charge the lithium cell, preventing leakage and premature failure.

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