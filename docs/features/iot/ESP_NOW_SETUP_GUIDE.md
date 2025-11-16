# 🔧 CropWise - ESP-NOW Setup Guide v2.0

## 📋 Table of Contents
1. [Overview](#overview)
2. [Hardware Requirements](#hardware-requirements)
3. [Wiring Diagrams](#wiring-diagrams)
4. [Software Installation](#software-installation)
5. [Configuration Steps](#configuration-steps)
6. [MAC Address Setup](#mac-address-setup)
7. [Testing & Verification](#testing--verification)
8. [Troubleshooting](#troubleshooting)
9. [Production Deployment](#production-deployment)

---

## 🎯 Overview

This guide will walk you through setting up a **Hierarchical Master-Slave IoT Architecture** using ESP32 microcontrollers with ESP-NOW for local communication and MQTT for cloud connectivity.

**Key Benefits:**
- ✅ Only 1 MQTT connection per zone (Master)
- ✅ ESP-NOW for fast peer-to-peer communication (< 10ms)
- ✅ Scalable from 1 to 6 ESP32s without backend changes
- ✅ Battery operation possible for sensor slaves
- ✅ No WiFi credentials needed for slaves

---

## 📦 Hardware Requirements

### Common Components (All Nodes)
| Component | Specifications | Where to Buy | Price |
|-----------|----------------|--------------|-------|
| ESP32 DevKit v1 | Dual-core 240MHz, WiFi+BT | Amazon, AliExpress | $6-8 |
| 5V Power Supply | 2A minimum (USB or DC) | Amazon | $5-7 |
| Micro-USB Cable | Data + Power | Amazon | $3 |
| Breadboard (optional) | 830 points for prototyping | Amazon | $5 |
| Jumper Wires | Male-Male, Male-Female | Amazon | $5 |
| Enclosure | Weatherproof if needed | Amazon | $8-12 |

### ESP32-MASTER Specific
| Component | Specifications | Where to Buy | Price |
|-----------|----------------|--------------|-------|
| **Sensors** | | | |
| DHT22 | Temperature + Humidity | Amazon | $5 |
| BH1750 | Light sensor (I2C) | Amazon | $3 |
| MQ-135 | CO2/Air quality | Amazon | $8 |
| **Output** | | | |
| 8-channel Relay Module | 5V, Active LOW | Amazon | $8 |
| **Display** | | | |
| OLED 0.96" 128x64 | I2C (SSD1306) | Amazon | $6 |
| **Optional** | | | |
| Buttons | Momentary push button | Amazon | $5/10pcs |

**Master Total:** ~$55

### ESP32-SLAVE-SENSOR Specific
| Component | Specifications | Where to Buy | Price |
|-----------|----------------|--------------|-------|
| DHT22 | Temperature + Humidity | Amazon | $5 |
| BH1750 | Light sensor | Amazon | $3 |
| MQ-135 | CO2 sensor | Amazon | $8 |
| Soil Moisture | Capacitive (optional) | Amazon | $5 |
| **Battery (Optional)** | | | |
| 18650 Battery | 3.7V Li-ion | Amazon | $8 |
| TP4056 Charger | Micro-USB charging | Amazon | $2 |

**Sensor Slave Total:** ~$23-31

### ESP32-SLAVE-IRRIGATION Specific
| Component | Specifications | Where to Buy | Price |
|-----------|----------------|--------------|-------|
| 4-channel Relay Module | 5V, 10A | Amazon | $6 |
| Solenoid Valves | 12V, 1/2", NC | Amazon | $5 each |
| Water Pump | 12V DC, submersible | Amazon | $12 |
| Flow Sensor | YF-S201 (Hall effect) | Amazon | $10 |
| Pressure Sensor | 0-1.2 MPa analog | Amazon | $8 |
| 12V Power Supply | 3A for pump + valves | Amazon | $12 |

**Irrigation Slave Total:** ~$68

### ESP32-SLAVE-LIGHTING Specific
| Component | Specifications | Where to Buy | Price |
|-----------|----------------|--------------|-------|
| LED Grow Lights | 12V, dimmable (PWM) | Amazon | $15 each |
| MOSFET Modules | IRF520, for PWM | Amazon | $5/4pcs |
| 12V Power Supply | 5A for LED strips | Amazon | $15 |

**Lighting Slave Total:** ~$50

---

## 🔌 Wiring Diagrams

### 1. ESP32-MASTER Wiring

```
┌─────────────────────────────────────────────────────────────┐
│                     ESP32-MASTER                             │
└─────────────────────────────────────────────────────────────┘

Power:
  5V → VIN (ESP32)
  GND → GND (ESP32)

I2C Sensors (OLED + BH1750):
  GPIO 21 (SDA) → OLED SDA, BH1750 SDA
  GPIO 22 (SCL) → OLED SCL, BH1750 SCL
  3.3V → VCC (OLED, BH1750)
  GND → GND (OLED, BH1750)

DHT22 Temperature/Humidity:
  GPIO 4 → DHT22 DATA
  3.3V → DHT22 VCC
  GND → DHT22 GND

MQ-135 CO2 Sensor:
  GPIO 34 (ADC) → MQ-135 A0 (Analog Out)
  5V → MQ-135 VCC
  GND → MQ-135 GND

8-Channel Relay Module (Active LOW):
  GPIO 13 → Relay 1 (IN1) - Exhaust Fan
  GPIO 14 → Relay 2 (IN2) - Circulation Fan
  GPIO 27 → Relay 3 (IN3) - Heater
  GPIO 26 → Relay 4 (IN4) - Humidifier
  GPIO 25 → Relay 5 (IN5) - Grow Lights
  GPIO 33 → Relay 6 (IN6) - Irrigation Pump
  GPIO 32 → Relay 7 (IN7) - Water Valve
  GPIO 15 → Relay 8 (IN8) - Spare/Cooler
  5V → VCC (Relay Module)
  GND → GND (Relay Module)

PWM Outputs (Fan Speed Control):
  GPIO 16 → Exhaust Fan PWM (via MOSFET/Motor Driver)
  GPIO 17 → Circulation Fan PWM

Button (Manual Override):
  GPIO 0 (BOOT) → One side of button
  GND → Other side of button
  (Internal pull-up enabled)

Status LED:
  GPIO 2 → Built-in LED (or external LED + resistor)
```

### Pin Assignment Table (Master)

| GPIO | Function | Component | Type |
|------|----------|-----------|------|
| 0 | Button | Boot button / Manual | Input (Pull-up) |
| 2 | LED | Status indicator | Output |
| 4 | DHT22 | Temperature/Humidity | 1-Wire |
| 13 | Relay 1 | Exhaust Fan | Output (LOW=ON) |
| 14 | Relay 2 | Circulation Fan | Output (LOW=ON) |
| 15 | Relay 8 | Spare | Output (LOW=ON) |
| 16 | PWM 1 | Exhaust Fan Speed | PWM Output |
| 17 | PWM 2 | Circulation Fan Speed | PWM Output |
| 21 | I2C SDA | OLED + BH1750 | I2C |
| 22 | I2C SCL | OLED + BH1750 | I2C |
| 25 | Relay 5 | Grow Lights | Output (LOW=ON) |
| 26 | Relay 4 | Humidifier | Output (LOW=ON) |
| 27 | Relay 3 | Heater | Output (LOW=ON) |
| 32 | Relay 7 | Water Valve | Output (LOW=ON) |
| 33 | Relay 6 | Irrigation Pump | Output (LOW=ON) |
| 34 | ADC | MQ-135 CO2 | Analog Input |

---

### 2. ESP32-SLAVE-SENSOR Wiring

```
┌─────────────────────────────────────────────────────────────┐
│                  ESP32-SLAVE-SENSOR                          │
└─────────────────────────────────────────────────────────────┘

Power:
  5V → VIN (ESP32)  OR  3.7V Battery → VIN (via TP4056)
  GND → GND

I2C Light Sensor (BH1750):
  GPIO 21 (SDA) → BH1750 SDA
  GPIO 22 (SCL) → BH1750 SCL
  3.3V → BH1750 VCC
  GND → BH1750 GND

DHT22:
  GPIO 4 → DHT22 DATA
  3.3V → DHT22 VCC
  GND → DHT22 GND

MQ-135 CO2:
  GPIO 34 → MQ-135 A0
  5V → MQ-135 VCC
  GND → MQ-135 GND

Soil Moisture (Optional):
  GPIO 35 → Capacitive sensor A0
  3.3V → Sensor VCC
  GND → Sensor GND

Status LED:
  GPIO 2 → Built-in LED

Button (Manual Test):
  GPIO 0 → Button
  GND → Button
```

**Note:** Sensor slaves have NO WiFi configured - only ESP-NOW!

---

### 3. ESP32-SLAVE-IRRIGATION Wiring

```
┌─────────────────────────────────────────────────────────────┐
│                ESP32-SLAVE-IRRIGATION                        │
└─────────────────────────────────────────────────────────────┘

Power:
  5V → VIN (ESP32)
  GND → GND
  12V Power Supply → Pump + Valves

4-Channel Relay Module:
  GPIO 13 → Relay 1 (IN1) - Water Pump
  GPIO 14 → Relay 2 (IN2) - Valve 1 (Zone 1)
  GPIO 27 → Relay 3 (IN3) - Valve 2 (Zone 2)
  GPIO 26 → Relay 4 (IN4) - Valve 3 (Zone 3)
  5V → Relay VCC
  GND → Relay GND

Flow Sensor (YF-S201):
  GPIO 4 → Flow sensor SIGNAL (pulse output)
  5V → Flow sensor VCC
  GND → Flow sensor GND

Pressure Sensor (Analog):
  GPIO 34 → Pressure sensor A0
  5V → Pressure sensor VCC
  GND → Pressure sensor GND

Water Level (Optional):
  GPIO 35 → Water level sensor A0
  5V → Sensor VCC
  GND → Sensor GND

12V Pump & Valves:
  Relay 1 COM → 12V+
  Relay 1 NO → Pump +
  Pump - → 12V GND
  
  Relay 2 COM → 12V+
  Relay 2 NO → Valve 1 +
  Valve 1 - → 12V GND
  
  (Same for Valves 2, 3, 4)
```

**⚠️ IMPORTANT:** 
- Use flyback diodes across pump/valves (12V coils)
- Ensure relay module can handle 12V load
- Water-proof all connections!

---

### 4. ESP32-SLAVE-LIGHTING Wiring

```
┌─────────────────────────────────────────────────────────────┐
│                ESP32-SLAVE-LIGHTING                          │
└─────────────────────────────────────────────────────────────┘

Power:
  5V → VIN (ESP32)
  GND → GND
  12V Power Supply → LED Strips

PWM MOSFET Modules (IRF520):
  GPIO 13 → MOSFET 1 PWM (LED Zone 1)
  GPIO 14 → MOSFET 2 PWM (LED Zone 2)
  GPIO 27 → MOSFET 3 PWM (LED Zone 3)
  GPIO 26 → MOSFET 4 PWM (LED Zone 4)
  5V → MOSFET VCC
  GND → MOSFET GND

LED Grow Light Strips (12V):
  12V+ → All LED+ (common positive)
  
  MOSFET 1 OUT → LED Zone 1 -
  MOSFET 2 OUT → LED Zone 2 -
  MOSFET 3 OUT → LED Zone 3 -
  MOSFET 4 OUT → LED Zone 4 -
  
  MOSFET GND → 12V GND

Status LED:
  GPIO 2 → Built-in LED

Button (Manual Override):
  GPIO 0 → Button
  GND → Button
```

**PWM Settings:**
- Frequency: 1 kHz
- Resolution: 10-bit (0-1023)
- 0% = OFF, 100% = Full brightness

---

## 💻 Software Installation

### 1. Install Arduino IDE

1. Download Arduino IDE from [arduino.cc](https://www.arduino.cc/en/software)
2. Install for your OS (Windows/Mac/Linux)

### 2. Install ESP32 Board Support

1. Open Arduino IDE
2. Go to **File → Preferences**
3. Add to "Additional Board Manager URLs":
   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```
4. Go to **Tools → Board → Boards Manager**
5. Search for "esp32"
6. Install **"esp32 by Espressif Systems"** (latest version)

### 3. Install Required Libraries

Go to **Tools → Manage Libraries** and install:

| Library | Version | Purpose |
|---------|---------|---------|
| PubSubClient | 2.8+ | MQTT client (Master only) |
| ArduinoJson | 6.21+ | JSON parsing |
| DHT sensor library | 1.4+ | DHT22 sensor |
| Adafruit BH1750 | 1.3+ | Light sensor |
| Adafruit GFX Library | 1.11+ | Graphics for OLED |
| Adafruit SSD1306 | 2.5+ | OLED display driver |

**Note:** ESP-NOW is built into ESP32 core - no library needed!

### 4. Download Firmware

Clone or download the CropWise repository:

```bash
git clone https://github.com/yellowflowersorganics-star/cropwise.git
cd cropwise/esp32-firmware
```

**Firmware Structure:**
```
esp32-firmware/
├── ESP32-MASTER/
│   └── ESP32-MASTER.ino
├── ESP32-SLAVE-SENSOR/
│   └── ESP32-SLAVE-SENSOR.ino
├── ESP32-SLAVE-IRRIGATION/
│   └── ESP32-SLAVE-IRRIGATION.ino
└── ESP32-SLAVE-LIGHTING/
    └── ESP32-SLAVE-LIGHTING.ino
```

---

## ⚙️ Configuration Steps

### Step 1: Configure Master Node

#### 1.1 Open Master Firmware

```bash
Arduino IDE → File → Open → esp32-firmware/ESP32-MASTER/ESP32-MASTER.ino
```

#### 1.2 Edit WiFi Credentials (Lines 70-71)

```cpp
// WiFi Configuration
const char* WIFI_SSID = "YourWiFiSSID";        // ← CHANGE THIS
const char* WIFI_PASSWORD = "YourWiFiPassword"; // ← CHANGE THIS
```

#### 1.3 Edit MQTT Configuration (Lines 74-77)

```cpp
// MQTT Configuration
const char* MQTT_BROKER = "192.168.1.100";  // ← Raspberry Pi IP
const int MQTT_PORT = 1883;
const char* MQTT_CLIENT_ID = "ESP32-MASTER-ZONE-A";  // ← Unique per zone
```

#### 1.4 Edit Zone Configuration (Lines 80-82)

```cpp
// Zone Configuration
const char* FARM_ID = "farm1";      // ← Your farm ID
const char* ZONE_ID = "zone-a";     // ← Your zone ID
String DEVICE_ID = "ESP32-MASTER-ZONE-A";
```

#### 1.5 Upload to ESP32

1. Connect ESP32 via USB
2. **Tools → Board → ESP32 Dev Module**
3. **Tools → Port → Select COM port** (e.g., COM3, /dev/ttyUSB0)
4. **Tools → Upload Speed → 921600**
5. Click **Upload** (→ button)
6. Wait for "Done uploading"

#### 1.6 Get Master MAC Address

1. Open **Tools → Serial Monitor** (115200 baud)
2. Press **RESET** button on ESP32
3. Look for output:

```
═══════════════════════════════════════════════════════
  CropWise - ESP32 MASTER CONTROLLER v2.0
  Architecture: Hierarchical Master-Slave
═══════════════════════════════════════════════════════

Initializing GPIO... ✓
Initializing sensors... [Light] ✓
Initializing display... ✓
Connecting to WiFi..... ✓
IP Address: 192.168.1.150
MAC Address: 24:6F:28:AB:CD:EF    ← COPY THIS!
Initializing ESP-NOW (Master)... ✓
✅ MASTER CONTROLLER READY
🎯 Mode: Standalone
```

4. **COPY THE MAC ADDRESS!** You'll need it for slave configuration!

---

### Step 2: Configure Sensor Slave

#### 2.1 Open Sensor Firmware

```bash
Arduino IDE → File → Open → esp32-firmware/ESP32-SLAVE-SENSOR/ESP32-SLAVE-SENSOR.ino
```

#### 2.2 Edit Slave Identity (Lines 60-62)

```cpp
// Slave Identity
#define SLAVE_ID 1           // ← Unique ID (1, 2, 3...)
#define SLAVE_TYPE 1         // 1 = Sensor node
String SLAVE_NAME = "SENSOR-A";  // ← Descriptive name
```

**For second sensor slave:**
```cpp
#define SLAVE_ID 2
String SLAVE_NAME = "SENSOR-B";
```

#### 2.3 Edit Master MAC Address (Line 65)

```cpp
// Master ESP32 MAC Address (GET THIS FROM MASTER SERIAL OUTPUT!)
uint8_t masterMAC[] = {0x24, 0x6F, 0x28, 0xAB, 0xCD, 0xEF};  // ← REPLACE THIS!
```

**How to convert MAC:**
- Master MAC: `24:6F:28:AB:CD:EF`
- Array format: `{0x24, 0x6F, 0x28, 0xAB, 0xCD, 0xEF}`

#### 2.4 Upload to ESP32

1. Connect sensor slave ESP32
2. Select board & port
3. Click **Upload**

#### 2.5 Verify Registration

1. Open Serial Monitor (115200 baud)
2. Look for:

```
═══════════════════════════════════════════════════════
  CropWise - ESP32 SENSOR SLAVE v2.0
  Slave ID: 1 (SENSOR-A)
═══════════════════════════════════════════════════════

My MAC Address: A4:CF:12:34:56:78    ← Note this for reference
⚠️ COPY THIS MAC TO MASTER IF NOT ALREADY DONE!

Initializing ESP-NOW (Slave)... ✓
Registering with master... ✓
📡 Registration packet sent to master
📥 Requesting current config from master...

✅ SENSOR SLAVE READY
📡 Sending data to master...

📊 T=24.5°C H=65.0% CO2=850ppm L=1200lux Soil=45%
📤 Data sent to master
```

3. **Check Master Serial Monitor:**

```
📥 Slave registration request:
   Type: Sensor
   ID: 1
✅ Slave SENSOR-A registered successfully!
📡 Config sent to SENSOR-A

📊 SENSOR-A: T=24.5°C H=65.0% CO2=850 Light=1200
```

✅ **Success!** Slave is registered and sending data!

---

### Step 3: Configure Irrigation Slave (Optional)

#### 3.1 Open Irrigation Firmware

```bash
Arduino IDE → File → Open → esp32-firmware/ESP32-SLAVE-IRRIGATION/ESP32-SLAVE-IRRIGATION.ino
```

#### 3.2 Edit Configuration (Lines 60-65)

```cpp
#define SLAVE_ID 2           // ← Unique ID
#define SLAVE_TYPE 2         // 2 = Irrigation node
String SLAVE_NAME = "IRRIGATION-A";

uint8_t masterMAC[] = {0x24, 0x6F, 0x28, 0xAB, 0xCD, 0xEF};  // ← Master MAC
```

#### 3.3 Adjust Flow Sensor Calibration (Line 78)

```cpp
#define FLOW_CALIBRATION_FACTOR 7.5  // ← Pulses per liter (YF-S201: ~7.5)
```

**To calibrate:**
1. Measure actual water volume (e.g., 10 liters)
2. Count pulses from flow sensor
3. Calculate: `pulses / liters = calibration factor`

#### 3.4 Upload & Verify

Follow same process as sensor slave.

**Test irrigation:**
- Press boot button on irrigation slave
- Should start 60-second manual irrigation cycle
- Check Serial Monitor for flow rate reporting

---

### Step 4: Configure Lighting Slave (Optional)

#### 4.1 Open Lighting Firmware

```bash
Arduino IDE → File → Open → esp32-firmware/ESP32-SLAVE-LIGHTING/ESP32-SLAVE-LIGHTING.ino
```

#### 4.2 Edit Configuration (Lines 60-65)

```cpp
#define SLAVE_ID 3           // ← Unique ID
#define SLAVE_TYPE 3         // 3 = Lighting node
String SLAVE_NAME = "LIGHTING-A";

uint8_t masterMAC[] = {0x24, 0x6F, 0x28, 0xAB, 0xCD, 0xEF};  // ← Master MAC
```

#### 4.3 Adjust PWM Settings (Lines 75-76)

```cpp
#define PWM_FREQ 1000        // 1 kHz (adjust if LED flickers)
#define PWM_RESOLUTION 10    // 10-bit (0-1023)
```

#### 4.4 Upload & Verify

**Test lighting:**
- Press boot button on lighting slave
- Lights should toggle ON/OFF at 100%
- Check Serial Monitor for status updates

---

## 🧪 Testing & Verification

### Test 1: Master Standalone Mode

**Goal:** Verify Master can operate independently

1. Power on Master ONLY (no slaves)
2. Open Serial Monitor
3. Check:
   - ✅ WiFi connected
   - ✅ MQTT connected
   - ✅ Local sensors reading
   - ✅ Display showing data
   - ✅ Mode: Standalone

**Expected Output:**
```
✅ MASTER CONTROLLER READY
🎯 Mode: Standalone
📡 Waiting for backend commands...

📊 T=24.5°C H=65.0% CO2=850 Light=1200
📤 Status published to backend
```

### Test 2: Slave Registration

**Goal:** Verify slaves can register with Master

1. Power on Master
2. Power on Sensor Slave
3. Wait 10 seconds
4. Check Master Serial Monitor:

```
📥 Slave registration request:
   Type: Sensor
   ID: 1
✅ Slave SENSOR-A registered successfully!
📡 Config sent to SENSOR-A
🎯 Mode: Master-Slave  ← Changed from Standalone!
```

5. Check Slave Serial Monitor:

```
Registering with master... ✓
📥 Stage config received from master!
✅ SENSOR SLAVE READY
📤 Data sent to master
```

✅ **Pass:** Slave registered and sending data

❌ **Fail:** Check MAC address configuration

### Test 3: Sensor Data Aggregation

**Goal:** Verify Master aggregates data from multiple sensors

1. Power on Master + 2 Sensor Slaves
2. Wait for registration
3. Check Master Serial Monitor:

```
📊 SENSOR-A: T=24.8°C H=86.5% CO2=2100 Light=1200
📊 SENSOR-B: T=25.2°C H=88.0% CO2=1950 Light=1150

📊 Aggregated: Temp=25.0°C Humidity=87.2%  ← Average of 3 sensors
📤 Status published to backend
```

4. Verify backend receives JSON:

```json
{
  "environment": {
    "temperature": 25.0,    // ← Averaged
    "humidity": 87.2,       // ← Averaged
    "co2": 2025,
    "light": 1175
  },
  "sensors": [
    {"location": "master", "temperature": 25.0, "online": true},
    {"location": "SENSOR-A", "temperature": 24.8, "online": true},
    {"location": "SENSOR-B", "temperature": 25.2, "online": true}
  ],
  "slaves": {
    "count": 2,
    "mode": "master-slave"
  }
}
```

✅ **Pass:** Data aggregated correctly

### Test 4: Recipe Start via MQTT

**Goal:** Verify Master receives backend commands

1. Ensure Master connected to MQTT broker
2. Use MQTT client (MQTT Explorer, mosquitto_pub) to publish:

**Command:**
```bash
mosquitto_pub -h 192.168.1.100 -t cropwise/farm1/zone-a/master/command/recipe/start -m '{
  "recipeName": "Test Recipe",
  "recipeId": "test-123",
  "currentStage": 0,
  "stageName": "Incubation",
  "stageConfig": {
    "temperature": {"optimal": 25},
    "humidity": {"optimal": 87},
    "co2": {"optimal": 2000},
    "light": 0,
    "irrigation": {"enabled": false},
    "equipment": {
      "ExhaustFan": 30,
      "CirculationFan": 0,
      "Heater": 100,
      "Humidifier": 85,
      "GrowLights": 0
    }
  }
}'
```

3. Check Master Serial Monitor:

```
📥 MQTT: cropwise/farm1/zone-a/master/command/recipe/start
🔥 RECIPE START command received
✓ Local equipment configured
📡 Stage config broadcast to 2 slaves
✅ Recipe started: Test Recipe - Incubation
```

4. Check equipment:
   - Exhaust fan running at 30%
   - Heater relay ON
   - Humidifier relay ON
   - Lights OFF

5. Check OLED display:

```
Test Recipe
Incubation

T:25.0C H:87.0%
CO2:2000 L:0

Slaves:2/2 online
FHU- WiFi:OK
```

✅ **Pass:** Recipe command executed successfully

### Test 5: Irrigation Cycle

**Goal:** Verify irrigation slave executes watering schedule

1. Send irrigation schedule via MQTT:

```bash
mosquitto_pub -h 192.168.1.100 -t cropwise/farm1/zone-a/master/command/recipe/transition -m '{
  "newStage": 1,
  "stageName": "Fruiting",
  "stageConfig": {
    "irrigation": {
      "enabled": true,
      "frequency": 1,
      "duration": 10,
      "times": ["15:30"]
    }
  }
}'
```

2. Check Irrigation Slave Serial Monitor:

```
📥 Irrigation schedule received from master!
   Enabled: 1 times/day
   Duration: 10 seconds
   Time 1: 15:30
✅ Irrigation activated!
```

3. Manually trigger (press boot button):

```
💧 STARTING IRRIGATION CYCLE
✓ Pump ON, duration: 10 seconds
💧 Flow: 2.50 L/min, Total: 0.42 L
💧 STOPPING IRRIGATION
   Duration: 10 seconds
   Water used: 0.42 liters
📤 Irrigation complete report sent to master
```

4. Check Master receives event:

```
💧 Irrigation complete: 0.4L used in 10s
📤 Event published to backend
```

✅ **Pass:** Irrigation cycle completed successfully

### Test 6: ESP-NOW Range Test

**Goal:** Verify communication range

1. Power Master + Sensor Slave
2. Place both next to each other (< 1m)
3. Verify communication working (check heartbeats)
4. Gradually move slave away:
   - 5m: ✅ Should work perfectly
   - 10m: ✅ Should work
   - 20m (through walls): ✅ Should work
   - 50m (line of sight): ✅ Should work
   - 100m: ⚠️ May be unreliable

5. Watch Master Serial Monitor for signal strength (RSSI):

```
📊 SENSOR-A: T=24.5°C H=65% RSSI=-45dBm  ← Excellent (-30 to -50)
📊 SENSOR-A: T=24.5°C H=65% RSSI=-65dBm  ← Good (-50 to -70)
📊 SENSOR-A: T=24.5°C H=65% RSSI=-80dBm  ← Marginal (-70 to -80)
⚠️ Slave SENSOR-A OFFLINE (no heartbeat)  ← Too far! (< -80)
```

**RSSI Guide:**
- `-30 to -50 dBm`: Excellent (< 10m)
- `-50 to -70 dBm`: Good (10-30m)
- `-70 to -80 dBm`: Fair (30-50m)
- `< -80 dBm`: Poor/unreliable

✅ **Pass:** Slave communicates reliably within expected range

---

## 🐛 Troubleshooting

### Problem 1: Slave Not Registering

**Symptoms:**
- Slave boots but Master doesn't show registration
- No data received at Master

**Solutions:**

1. **Check MAC Address:**
   - Verify Master MAC is correct in slave code
   - Format: `{0x24, 0x6F, 0x28, 0xAB, 0xCD, 0xEF}`
   - No colons, use `0x` prefix

2. **Check WiFi Mode:**
   - Master must be in `WIFI_MODE_APSTA` (AP+STA)
   - Slave must be in `WIFI_STA`

3. **Check ESP-NOW Channel:**
   - Both must be on same WiFi channel
   - Set `peerInfo.channel = 0` (auto-detect)

4. **Restart Both Devices:**
   - Power cycle Master first
   - Then power cycle Slave
   - Wait 10 seconds for registration

5. **Check Serial Output:**
   - Slave should print: `Registering with master... ✓`
   - Master should print: `Slave SENSOR-A registered successfully!`

### Problem 2: Master Not Connecting to MQTT

**Symptoms:**
- Master boots but no MQTT connection
- Display shows "WiFi:X" or "MQTT:X"

**Solutions:**

1. **Check WiFi Credentials:**
   ```cpp
   const char* WIFI_SSID = "YourActualSSID";
   const char* WIFI_PASSWORD = "YourActualPassword";
   ```

2. **Check Raspberry Pi IP:**
   - On Raspberry Pi: `hostname -I`
   - Update Master code: `const char* MQTT_BROKER = "192.168.1.XXX";`

3. **Check Mosquitto Running:**
   ```bash
   # On Raspberry Pi
   sudo systemctl status mosquitto
   
   # Should show: "active (running)"
   # If not:
   sudo systemctl restart mosquitto
   ```

4. **Test MQTT Connection:**
   ```bash
   # From laptop (same network)
   mosquitto_sub -h 192.168.1.100 -t test
   
   # From another terminal
   mosquitto_pub -h 192.168.1.100 -t test -m "hello"
   
   # Should receive "hello"
   ```

5. **Check Firewall:**
   ```bash
   # On Raspberry Pi
   sudo ufw allow 1883/tcp
   ```

### Problem 3: Sensors Reading NaN or 0

**Symptoms:**
- Display shows `T:nanC H:nan%`
- Or constant `T:25.0C H:60.0%` (default values)

**Solutions:**

1. **DHT22:**
   - Check wiring (DATA to GPIO 4, VCC to 3.3V, GND to GND)
   - Add 10kΩ pull-up resistor between DATA and VCC
   - Wait 2 seconds after boot for sensor to initialize
   - Try different GPIO pin (edit code)

2. **BH1750 Light Sensor:**
   - Check I2C wiring (SDA=21, SCL=22)
   - Check I2C address (default: 0x23)
   - Test with I2C scanner:
   ```cpp
   Wire.begin(21, 22);
   Wire.beginTransmission(0x23);
   if (Wire.endTransmission() == 0) {
     Serial.println("BH1750 found!");
   }
   ```

3. **MQ-135 CO2 Sensor:**
   - Requires 24-48 hour burn-in period!
   - May show inaccurate readings initially
   - Check analog pin (GPIO 34)
   - Calibrate in fresh air (400 ppm)

### Problem 4: Relays Not Switching

**Symptoms:**
- Equipment doesn't turn on/off
- Relay LEDs not lighting

**Solutions:**

1. **Check Relay Logic Level:**
   - Most modules are **Active LOW**
   - `digitalWrite(RELAY_PIN, LOW)` = ON
   - `digitalWrite(RELAY_PIN, HIGH)` = OFF

2. **Check Power:**
   - Relay module needs 5V power
   - ESP32 outputs 3.3V signals (usually enough)
   - If not, use level shifter or logic-level relays

3. **Check Wiring:**
   - GPIO 13-15, 25-27, 32-33 → Relay IN1-IN8
   - 5V → Relay VCC
   - GND → Relay GND (common ground!)

4. **Test Manually:**
   ```cpp
   void loop() {
     digitalWrite(RELAY_1, LOW);  // ON
     delay(2000);
     digitalWrite(RELAY_1, HIGH); // OFF
     delay(2000);
   }
   ```

### Problem 5: OLED Display Blank

**Symptoms:**
- Display powered but shows nothing

**Solutions:**

1. **Check I2C Address:**
   - Default: `0x3C`
   - Some modules use `0x3D`
   - Test both:
   ```cpp
   if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
     if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3D)) {
       Serial.println("Display not found!");
     }
   }
   ```

2. **Check Wiring:**
   - SDA = GPIO 21
   - SCL = GPIO 22
   - VCC = 3.3V (or 5V depending on module)
   - GND = GND

3. **Test with Simple Code:**
   ```cpp
   display.clearDisplay();
   display.setTextSize(1);
   display.setTextColor(SSD1306_WHITE);
   display.setCursor(0, 0);
   display.println("Hello!");
   display.display();
   ```

### Problem 6: Flow Sensor Not Reading

**Symptoms:**
- Flow rate always 0.0 L/min
- Water flowing but no pulses

**Solutions:**

1. **Check Sensor Direction:**
   - Arrow on sensor → direction of flow
   - Reverse if installed backwards

2. **Check Interrupt:**
   ```cpp
   pinMode(FLOW_SENSOR_PIN, INPUT_PULLUP);
   attachInterrupt(digitalPinToInterrupt(FLOW_SENSOR_PIN), flowPulseCounter, RISING);
   ```

3. **Test Interrupt:**
   ```cpp
   volatile uint16_t testCount = 0;
   
   void IRAM_ATTR testISR() {
     testCount++;
   }
   
   void setup() {
     attachInterrupt(digitalPinToInterrupt(4), testISR, RISING);
   }
   
   void loop() {
     Serial.println(testCount);  // Should increment when water flows
     delay(1000);
   }
   ```

4. **Calibrate Factor:**
   - Measure actual water volume
   - Adjust `FLOW_CALIBRATION_FACTOR`

### Problem 7: ESP-NOW Send Failed

**Symptoms:**
- Slave Serial: "⚠️ Send failed!"
- Master Serial: No data received

**Solutions:**

1. **Check Peer Added:**
   ```cpp
   if (esp_now_add_peer(&peerInfo) == ESP_OK) {
     Serial.println("Peer added!");
   } else {
     Serial.println("Failed to add peer!");
   }
   ```

2. **Check Payload Size:**
   - ESP-NOW max payload: 250 bytes
   - `sizeof(espnow_message_t)` must be ≤ 250

3. **Check WiFi Channel:**
   - Both nodes must be on same channel
   - Set `peerInfo.channel = 0` (auto)

4. **Increase Send Timeout:**
   - Add delay between sends: `delay(10)`

---

## 🚀 Production Deployment

### Enclosure Design

**Master Node:**
- Weatherproof IP65 enclosure (if outdoor)
- Ventilation holes for sensors (with mesh)
- Cable glands for relay connections
- DIN rail mounting (optional)

**Slave Nodes:**
- Smaller IP65 enclosures
- Battery compartment (if wireless)
- Sensor mounting brackets
- UV-resistant plastic (if outdoor)

### Power Supply

**Master:**
- 5V 3A power supply (USB or DC jack)
- Consider UPS/battery backup

**Slaves:**
- Sensors: Battery + solar panel (optional)
- Irrigation: 12V 3A power supply
- Lighting: 12V 5A power supply

### Wiring Best Practices

1. **Use Stranded Wire:**
   - More flexible than solid core
   - 18-22 AWG for power
   - 24-28 AWG for signals

2. **Color Coding:**
   - Red: Positive power
   - Black: Ground
   - Yellow: Signals
   - Blue: I2C/Serial

3. **Ferrules:**
   - Use crimp ferrules on wire ends
   - Prevents fraying in screw terminals

4. **Cable Management:**
   - Use zip ties or Velcro
   - Label all cables
   - Leave service slack

### Testing Checklist

Before final deployment:

- [ ] All nodes registered successfully
- [ ] Sensor readings accurate
- [ ] Equipment switching correctly
- [ ] MQTT commands working
- [ ] Recipe execution working
- [ ] Irrigation cycles tested
- [ ] Lighting control tested
- [ ] Display showing correct info
- [ ] Range test passed
- [ ] Power supply stable
- [ ] Enclosures sealed
- [ ] All cables secured
- [ ] Emergency stop tested
- [ ] Backup/restore tested
- [ ] Documentation complete

---

## 📚 Additional Resources

**ESP32 Documentation:**
- [ESP32 Arduino Core Docs](https://docs.espressif.com/projects/arduino-esp32)
- [ESP-NOW Guide](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/network/esp_now.html)

**Sensor Datasheets:**
- [DHT22 Datasheet](https://www.sparkfun.com/datasheets/Sensors/Temperature/DHT22.pdf)
- [BH1750 Datasheet](https://www.mouser.com/datasheet/2/348/bh1750fvi-e-186247.pdf)
- [MQ-135 Datasheet](https://www.olimex.com/Products/Components/Sensors/Gas/SNS-MQ135/resources/SNS-MQ135.pdf)

**CropWise Docs:**
- [Architecture Documentation](./IOT_ARCHITECTURE_V2.md)
- [Backend API Reference](./API_DOCUMENTATION.md)
- [MQTT Topics Guide](./MQTT_TOPICS.md)

---

## 🎉 Success!

If you've completed this guide, you now have a fully functional **Hierarchical Master-Slave IoT Architecture** with:

✅ ESP32-MASTER as central hub  
✅ ESP-NOW slaves for distributed sensing  
✅ MQTT connectivity to CropWise backend  
✅ Scalable from 1 to 6 ESP32s  
✅ Real-time environmental monitoring  
✅ Automated equipment control  
✅ Professional-grade reliability  

**Next Steps:**
1. Deploy to production environment
2. Calibrate sensors for accuracy
3. Set up monitoring dashboard
4. Configure alerting rules
5. Train operators on system

**Happy Growing! 🌱✨**

---

**Need Help?**
- GitHub Issues: [CropWise](https://github.com/yellowflowersorganics-star/cropwise/issues)
- Email: support@cropwise.io
- Discord: [CropWise Community](https://discord.gg/cropwise)

