# 💡 ESP8266 ESPHome Streetlight Relay

An ultra-reliable, autonomous ESPHome firmware designed to control outdoor streetlights or heavy-duty contactors using an ESP8266 (Wemos D1 Mini). 

Standard smart plugs fail when the internet goes down or power is cut. This firmware is engineered to handle frequent power cuts and network drops smoothly. It relies on exact NTP time and mathematical scheduling to ensure the light *always* recovers to its correct state, even if your home automation server (like Home Assistant) goes completely offline.

---

## ✨ The Core Philosophy: "Set It and Forget It"

### 🕒 1. The Autonomous, Math-Based Schedule
Most smart relays rely on a central server to send "Turn On" and "Turn Off" commands. If the power drops at 18:29 and comes back at 18:31, a standard smart plug misses the 18:30 "Turn On" command and stays in the dark all night.

**How this firmware fixes it:**
Instead of waiting for triggers, this ESP8266 mathematically calculates if it *should* be on whenever it boots up. 
* It grabs the current time via NTP.
* It converts the current time and target times into "minutes since midnight".
* It runs a `lambda` calculation to check if the current time falls within the active window.
* If the power comes back on at 2:00 AM, the math instantly evaluates to `true`, and the streetlight turns back on immediately.

### 🛜 2. The Fallback Access Point (AP) & Captive Portal
If your home router dies, you don't lose control of the streetlight. The ESP8266 automatically deploys a fallback network.
* **The Trigger:** If the Wemos cannot find your home Wi-Fi for exactly 1 minute (`ap_timeout: 1min`), it starts broadcasting its own Wi-Fi network called **`HDz_Relay`**.
* **The Captive Portal:** When you connect your smartphone to `HDz_Relay`, the Wemos intercepts your phone's internet check and pushes a "Sign in to network" notification to your phone screen. *(Note: Turn OFF Mobile Data on your phone to ensure this prompt appears).*

### 🖥️ 3. The Local Fallback Web Server
Once you tap the "Sign in" notification, you are greeted by a secure, password-protected local Web Server hosted entirely on the ESP8266 chip. From this offline dashboard, you can manually toggle the relay, view live telemetry (Uptime, Signal Strength, Voltage), and click **"Sync time from browser"** to manually push your phone's exact time to the Wemos, fixing the schedule without the internet!

### 🥷 4. Stealth LED Mode
Outdoor status LEDs attract insects to your weatherproof enclosure. 
* The onboard blue LED on the Wemos only shines for **90 seconds** on boot, or when the relay changes state. 
* After 90 seconds, it goes completely dark. 
* If the Wi-Fi connection drops, the LED will wake up and blink for 90 seconds to warn you before going back to sleep.

---

## 🛠️ Hardware & Pinout

This project was built using a **Wemos D1 Mini** driving a standard 5V Relay module. Because LED streetlight drivers have massive inrush current, the 5V relay is **only** used to switch the magnetic coil of a heavy-duty 220V AC Contactor. 

| Component | ESP8266 Pin | Notes |
| :--- | :--- | :--- |
| **Relay Control** | `GPIO4` (D2) | Triggers the 5V relay module (which triggers the contactor). |
| **Manual Button** | `GPIO12` (D6) | Connect a momentary push button between D6 and GND for physical override. |
| **Status LED** | `GPIO2` (D4) | Internal Wemos LED (Inverted). |

---

## 🔐 Required Secrets Configuration

To keep your network secure, all passwords have been moved to variables. You must add the following keys to your ESPHome `secrets.yaml` file before compiling:

```yaml
# Your Main Home Wi-Fi Network
wifi_ssid: "Your_Home_WiFi_Name"
wifi_password: "Your_Home_WiFi_Password"

# ESPHome Security
api_key: "your_generated_api_key_here="
ota_password: "your_secure_ota_password"

# Fallback Web Server Dashboard Login
web_username: "admin"
web_password: "A_Secure_Password"

# Hotspot Password (for the HDz_Relay network)
ap_password: "12345678"
