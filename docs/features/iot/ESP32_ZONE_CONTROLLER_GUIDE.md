# 🤖 ESP32 Zone Controller - Complete Setup Guide

Transform your ESP32 into an intelligent zone controller with display, equipment control, and recipe execution!

---

## 🎯 Overview

The ESP32 Zone Controller provides:
- 📺 **OLED/TFT Display** - Shows recipe, environment, equipment status
- ⚡ **8 Relay Control** - Fans, heaters, lights, pumps, etc.
- 🎚️ **2 PWM Outputs** - Variable speed control (VFD, dimmers)
- 🌡️ **Environmental Sensors** - Temperature, humidity, light, CO2
- 📡 **MQTT Communication** - Real-time control from CropWise
- 🎮 **Local Control** - 3 buttons for manual operation
- 🪄 **Auto Recipe Execution** - Follows recipe stages automatically

---

## 📦 Hardware Requirements

### **Required Components:**

| Component | Quantity | Purpose | Price (approx) |
|-----------|----------|---------|----------------|
| ESP32 DevKit | 1 | Main controller | $8 |
| DHT22 | 1 | Temperature & Humidity | $5 |
| BH1750 | 1 | Light sensor | $3 |
| MQ-135 | 1 | CO2/Air quality | $4 |
| 8-Channel Relay Module | 1 | Equipment control | $10 |
| SSD1306 OLED (128x64) | 1 | Display | $5 |
| Push Buttons | 3 | Navigation | $1 |
| Resistors 10kΩ | 3 | Pull-up for buttons | $1 |
| Jumper Wires | 1 set | Connections | $5 |
| Breadboard/PCB | 1 | Prototyping | $5 |
| 5V Power Supply (3A+) | 1 | Power | $10 |
| **TOTAL** | | | **~$57** |

### **Optional Components:**
- ILI9341 TFT Display (320x240) - Better visuals ($15)
- DS18B20 Waterproof sensor - For substrate temperature ($5)
- Soil moisture sensor ($3)
- Rotary encoder instead of buttons ($5)

---

## 🔌 Wiring Diagram

### **ESP32 Pin Connections:**

```
ESP32 DevKit          Component
========================================
GPIO 4   -----------> DHT22 Data
GPIO 34  -----------> MQ-135 Analog Out
SDA (21) -----------> BH1750 SDA, OLED SDA
SCL (22) -----------> BH1750 SCL, OLED SCL

GPIO 5   -----------> PWM Fan Speed
GPIO 18  -----------> PWM Humidifier Level

GPIO 13  -----------> Relay 1 (Exhaust Fan)
GPIO 12  -----------> Relay 2 (Circulation Fan)
GPIO 14  -----------> Relay 3 (Humidifier)
GPIO 27  -----------> Relay 4 (Heater)
GPIO 26  -----------> Relay 5 (Cooler)
GPIO 25  -----------> Relay 6 (Grow Lights)
GPIO 33  -----------> Relay 7 (Irrigation Pump)
GPIO 32  -----------> Relay 8 (CO2 Valve)

GPIO 19  -----------> Button UP (with 10kΩ pullup)
GPIO 21  -----------> Button DOWN (with 10kΩ pullup)
GPIO 22  -----------> Button SELECT (with 10kΩ pullup)

GND      -----------> All GND connections
3.3V     -----------> DHT22, BH1750, OLED VCC
5V       -----------> Relay module VCC
```

### **Important Notes:**
- ⚠️ **Relay Module:** Use 5V power, connect GND to ESP32 GND
- ⚠️ **Relay Control:** Most modules are **Active LOW** (GPIO LOW = ON)
- ⚠️ **Power:** ESP32 draws ~250mA, relays ~70mA each, use 3A+ supply
- ⚠️ **Safety:** Use optocouplers for AC equipment control

---

## 💻 Software Setup

### **Step 1: Install Arduino IDE**

1. Download: [https://www.arduino.cc/en/software](https://www.arduino.cc/en/software)
2. Install for your OS (Windows/Mac/Linux)

### **Step 2: Add ESP32 Board Support**

1. Open Arduino IDE
2. Go to **File** → **Preferences**
3. Add to **Additional Board Manager URLs**:
   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```
4. Go to **Tools** → **Board** → **Boards Manager**
5. Search "ESP32" and install **esp32 by Espressif Systems**

### **Step 3: Install Required Libraries**

Go to **Sketch** → **Include Library** → **Manage Libraries**, then install:

| Library | Version | Purpose |
|---------|---------|---------|
| PubSubClient | 2.8+ | MQTT client |
| Adafruit GFX | Latest | Graphics library |
| Adafruit SSD1306 | Latest | OLED display |
| DHT sensor library | Latest | DHT22 sensor |
| BH1750 | Latest | Light sensor |
| ArduinoJson | 6.x | JSON parsing |

### **Step 4: Download Firmware**

1. Copy firmware from: `esp32-firmware/CropWiseZoneController/CropWiseZoneController.ino`
2. Open in Arduino IDE

### **Step 5: Configure**

Edit these lines at the top of the code:

```cpp
// WiFi Credentials
const char* WIFI_SSID = "YourWiFiSSID";
const char* WIFI_PASSWORD = "YourWiFiPassword";

// MQTT Broker (Raspberry Pi)
const char* MQTT_BROKER = "192.168.1.100";  // Your Raspberry Pi IP
const int MQTT_PORT = 1883;

// Device ID (MUST BE UNIQUE!)
const char* DEVICE_ID = "ESP32-ZONE-A";  // Change for each ESP32!
```

**Important:** Each ESP32 must have a **unique DEVICE_ID**!

Examples:
- `ESP32-ZONE-A`
- `ESP32-ZONE-B`
- `ESP32-INCUBATION-ROOM-1`

### **Step 6: Upload**

1. Connect ESP32 via USB
2. Select **Tools** → **Board** → **ESP32 Dev Module**
3. Select **Tools** → **Port** → (your COM port)
4. Click **Upload** button (→)
5. Wait for "Done uploading"

### **Step 7: Monitor**

1. Open **Tools** → **Serial Monitor**
2. Set baud rate to **115200**
3. You should see:
   ```
   ========================================
   CropWise - ESP32 Zone Controller
   ========================================
   
   ✓ GPIO pins initialized
   ✓ Display initialized
   ✓ DHT22 initialized
   ✓ BH1750 initialized
   Connecting to WiFi.......
   ✓ WiFi connected
     IP: 192.168.1.150
   Connecting to MQTT broker... connected!
   ✓ Subscribed to: cropwise/ESP32-ZONE-A/command/#
   ✓ Setup complete!
   ```

---

## 🧪 Testing

### **Test 1: Display Check**

The OLED should show the Recipe page. Use buttons to navigate:
- **UP Button:** Previous page
- **DOWN Button:** Next page
- **SELECT Button:** (Reserved for future menu)

Pages:
1. **RECIPE** - Shows active recipe and progress
2. **ENVIRONMENT** - Shows sensor readings
3. **EQUIPMENT** - Shows equipment states
4. **NETWORK** - Shows WiFi/MQTT status

### **Test 2: Sensor Check**

Serial monitor should show every 5 seconds:
```
📊 Sensors: T=25.3°C, H=85.2%, L=150lx, CO2=800ppm
```

### **Test 3: MQTT Communication**

In backend, test equipment control:

```bash
curl -X POST http://localhost:3000/api/equipment/{equipmentId}/on \
  -H "Authorization: Bearer YOUR_TOKEN"
```

Serial monitor shows:
```
📥 MQTT Message Received:
  Topic: cropwise/ESP32-ZONE-A/command/fan/ExhaustFan
  Payload: {"commandId":"...","commandType":"turn_on",...}
🎮 Executing command: turn_on for ExhaustFan
✓ ExhaustFan ON
✅ Command ACK sent
```

And the relay should **click**! ⚡

---

## 🎨 Display Pages Explained

### **Page 1: RECIPE**
```
=== RECIPE ===

Oyster Mushroom

Stage: Pinning
Day 3/10-12

Progress: 75%
```

Shows:
- Crop name
- Current stage name
- Days in stage / expected duration
- Overall progress percentage

### **Page 2: ENVIRONMENT**
```
== ENVIRONMENT ==

Temp: 24.5 C
Humid: 87.3 %
Light: 150 lx
CO2: 1200 ppm
```

Shows real-time sensor readings.

### **Page 3: EQUIPMENT**
```
== EQUIPMENT ==

ExhaustFan: OFF
CirculationFan: ON
Humidifier: ON
Heater: OFF
Cooler: OFF
```

Shows first 5 equipment states.

### **Page 4: NETWORK**
```
=== NETWORK ===

WiFi: OK
MQTT: OK
IP: 192.168.1.150
RSSI: -45 dBm
```

Shows connectivity status.

---

## 🔧 Equipment Mapping

### **Default Equipment Configuration:**

| Relay | Equipment | Typical Use |
|-------|-----------|-------------|
| 1 | Exhaust Fan | Remove excess CO2, control temperature |
| 2 | Circulation Fan | Air circulation within zone |
| 3 | Humidifier | Increase humidity (on/off or PWM) |
| 4 | Heater | Increase temperature |
| 5 | Cooler/AC | Decrease temperature |
| 6 | Grow Lights | LED grow lights (on/off or dimming) |
| 7 | Irrigation Pump | Water/nutrient delivery |
| 8 | CO2 Valve | CO2 injection (if used) |

### **PWM Outputs:**

| Pin | Use | Range |
|-----|-----|-------|
| GPIO 5 | Fan Speed (VFD) | 0-100% |
| GPIO 18 | Humidifier Level | 0-100% |

---

## 🪄 Recipe Execution Flow

### **1. Backend Starts Recipe**

```javascript
POST /api/recipe-execution/start
{
  "zoneId": "zone-a-id",
  "recipeId": "oyster-3-flush-id",
  "batchNumber": "BATCH-001"
}
```

### **2. ESP32 Receives Config**

MQTT topic: `cropwise/ESP32-ZONE-A/display/recipe`

```json
{
  "recipeName": "Oyster Mushroom",
  "currentStage": 0,
  "stageName": "Incubation",
  "totalStages": 3,
  "progress": 0,
  "daysInStage": 0,
  "expectedDuration": "15-20",
  "status": "active"
}
```

### **3. Display Updates**

OLED now shows:
```
=== RECIPE ===

Oyster Mushroom

Stage: Incubation
Day 0/15-20

Progress: 0%
```

### **4. Backend Sends Equipment Commands**

For Incubation stage (Oyster Mushroom):
- Turn OFF exhaust fan (minimal air exchange)
- Set heater to 24-26°C
- Set humidifier to 85-90%
- Turn OFF lights

MQTT commands sent automatically!

### **5. ESP32 Executes**

```
📥 MQTT: Turn OFF ExhaustFan
✓ ExhaustFan OFF
📥 MQTT: Set Heater ON
✓ Heater ON
```

### **6. Auto Control**

ESP32 monitors sensors and adjusts:
- If temp < 24°C → Turn ON heater
- If temp > 26°C → Turn OFF heater
- If humidity < 85% → Turn ON humidifier
- If humidity > 90% → Turn OFF humidifier

### **7. Progress Monitoring**

Every hour, backend checks:
- Is stage duration reached?
- If YES → Request manager approval!

### **8. Stage Transition**

Manager approves → Backend sends new stage config → ESP32 updates automatically!

```
=== RECIPE ===

Oyster Mushroom

Stage: Pinning
Day 0/10-12

Progress: 75%
```

New equipment commands:
- Turn ON exhaust fan (50% speed) - Air exchange
- Lower temperature to 18-20°C
- Increase humidity to 95%
- Turn ON lights (12h/day)

---

## 🛠️ Troubleshooting

### **Problem: Display not working**

**Solutions:**
1. Check I2C address:
   ```cpp
   // Try 0x3C (default) or 0x3D
   display.begin(SSD1306_SWITCHCAPVCC, 0x3C)
   ```
2. Verify wiring: SDA to GPIO 21, SCL to GPIO 22
3. Test with I2C Scanner sketch

### **Problem: WiFi not connecting**

**Solutions:**
1. Check SSID and password (case-sensitive!)
2. Ensure 2.4GHz WiFi (ESP32 doesn't support 5GHz)
3. Move closer to router
4. Check Serial Monitor for error messages

### **Problem: MQTT not connecting**

**Solutions:**
1. Verify Raspberry Pi IP address
2. Check Mosquitto is running:
   ```bash
   sudo systemctl status mosquitto
   ```
3. Test with mosquitto_sub:
   ```bash
   mosquitto_sub -h localhost -t 'cropwise/#' -v
   ```
4. Check firewall rules

### **Problem: Sensors not reading**

**DHT22:**
- Check wiring (Data to GPIO 4, VCC to 3.3V)
- Wait 2 seconds after power-on
- Try different DHT22 (they can fail)

**BH1750:**
- Check I2C wiring
- Verify address (0x23 or 0x5C)

**MQ-135:**
- Needs 24-48 hour burn-in period
- Readings are approximate without calibration

### **Problem: Relays not switching**

**Solutions:**
1. Check relay module power (5V, GND)
2. Verify GPIO connections
3. Check Active HIGH vs Active LOW:
   ```cpp
   // Active LOW (common):
   digitalWrite(pin, LOW);  // ON
   digitalWrite(pin, HIGH); // OFF
   
   // Active HIGH (rare):
   digitalWrite(pin, HIGH); // ON
   digitalWrite(pin, LOW);  // OFF
   ```
4. Test with multimeter: GPIO should output 3.3V HIGH, 0V LOW

### **Problem: PWM not working**

**Solutions:**
1. Check PWM pin connections
2. Verify frequency and resolution:
   ```cpp
   ledcSetup(channel, 5000, 8);  // 5kHz, 8-bit
   ```
3. Test with simple code:
   ```cpp
   ledcWrite(0, 128);  // 50% duty cycle
   ```

---

## 📊 Performance Specs

| Metric | Value |
|--------|-------|
| Sensor Read Interval | 5 seconds |
| MQTT Publish Interval | 30 seconds |
| Display Update Rate | 1 Hz (1 second) |
| Command Response Time | <100ms |
| WiFi Range | 30-50 meters |
| Power Consumption | 250-500mA (ESP32 + relays) |
| Max Concurrent Relays | 8 |
| PWM Frequency | 5 kHz |
| PWM Resolution | 8-bit (0-255) |

---

## 🎯 Advanced Features

### **Feature 1: Recipe Selection Menu**

Add local recipe selection using buttons (future enhancement):
- Press SELECT to enter menu
- Use UP/DOWN to browse recipes
- Press SELECT to start recipe
- Syncs with backend

### **Feature 2: Manual Equipment Override**

- Press SELECT on Equipment page
- Use UP/DOWN to select equipment
- Press SELECT to toggle ON/OFF
- Automatically switches to "manual" mode

### **Feature 3: Data Logging to SD Card**

Add SD card module:
- Log sensor data every minute
- Store command history
- Offline operation support

### **Feature 4: Web Interface**

ESP32 can host a web server:
- Access via browser: `http://192.168.1.150`
- View live data
- Control equipment
- No backend needed for basic operation

---

## 🔐 Security Considerations

### **1. WiFi Security**
- Use WPA2 encryption
- Strong password
- Isolated IoT network (recommended)

### **2. MQTT Security**
- Enable authentication:
  ```cpp
  mqtt.connect(clientId, "username", "password");
  ```
- Use TLS/SSL (advanced):
  ```cpp
  WiFiClientSecure espClient;
  ```

### **3. Physical Security**
- Protect ESP32 from moisture
- Use enclosure
- Fuse/circuit breaker for AC equipment

---

## 📈 Performance Optimization

### **1. Reduce MQTT Traffic**
```cpp
// Only publish when values change significantly
if (abs(newTemp - lastTemp) > 0.5) {
  publishStatus();
  lastTemp = newTemp;
}
```

### **2. Battery Optimization (if using battery)**
```cpp
// Deep sleep between readings
ESP.deepSleep(60e6);  // Sleep 60 seconds
```

### **3. Faster Display**
```cpp
// Only update changed sections
display.fillRect(x, y, w, h, BLACK);  // Clear section
display.setCursor(x, y);
display.println(newText);
display.display();
```

---

## ✅ Pre-Flight Checklist

Before deploying:

- [ ] WiFi SSID and password configured
- [ ] Raspberry Pi IP address correct
- [ ] Unique DEVICE_ID set
- [ ] All sensors tested and reading
- [ ] All relays tested and switching
- [ ] Display showing correct info
- [ ] MQTT connection established
- [ ] Equipment registered in backend
- [ ] Zone created with correct deviceId
- [ ] Power supply adequate (3A+)
- [ ] Enclosure weatherproof (if needed)
- [ ] Safety features tested (emergency stop?)

---

## 🚀 Deployment Tips

### **Multiple Zones:**

For a farm with 5 zones:
1. Flash firmware to 5 ESP32s
2. Set unique DEVICE_IDs:
   - `ESP32-INCUBATION`
   - `ESP32-PINNING-1`
   - `ESP32-PINNING-2`
   - `ESP32-FRUITING-1`
   - `ESP32-FRUITING-2`
3. Register each in backend as Equipment
4. Create 5 Zones in backend
5. Link ESP32s to Zones via `deviceId`
6. Deploy and monitor!

### **Mushroom Farm Example:**

**Zone A - Incubation Room:**
- ESP32-INCUBATION
- Equipment: Heater, Humidifier
- No fans, no lights
- Temp: 24-26°C, Humidity: 85-90%

**Zone B - Pinning Room:**
- ESP32-PINNING
- Equipment: Fans, Heater, Humidifier, Lights
- Air exchange: HIGH
- Temp: 18-20°C, Humidity: 95%, Light: 12h/day

**Zone C - Fruiting Room:**
- ESP32-FRUITING
- Equipment: Fans, Irrigation, Lights
- Air exchange: MEDIUM
- Harvest every 7 days

---

## 🎓 Learning Resources

- **ESP32 Basics:** [randomnerdtutorials.com/esp32](https://randomnerdtutorials.com)
- **MQTT Protocol:** [mqtt.org](https://mqtt.org)
- **Relay Control:** YouTube: "ESP32 Relay Control Tutorial"
- **Sensor Calibration:** [Adafruit Learning System](https://learn.adafruit.com)

---

## 🎉 Congratulations!

You now have a **fully functional IoT zone controller**!

Your ESP32 can:
- ✅ Display recipe progress in real-time
- ✅ Control 8+ equipment automatically
- ✅ Monitor environment 24/7
- ✅ Receive commands from CropWise
- ✅ Report status via MQTT
- ✅ Operate in auto or manual mode

**This is professional-grade farm automation!** 🍄🤖🌱

Need help? Check troubleshooting or create an issue on GitHub!

