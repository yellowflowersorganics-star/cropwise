# IoT Integration Guide - Raspberry Pi 5 + ESP32

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     CropWise Server                      │
│                    (Cloud/Local Server)                      │
└──────────────────────────┬──────────────────────────────────┘
                           │ REST API (WiFi/Internet)
                           │
┌──────────────────────────▼──────────────────────────────────┐
│               Raspberry Pi 5 (Farm Gateway)                  │
│  - MQTT Broker (Mosquitto)                                  │
│  - Data Aggregation                                         │
│  - API Communication                                        │
│  - Zone Control Logic                                       │
└──┬────────┬────────┬────────┬────────┬──────────────────────┘
   │ MQTT   │ MQTT   │ MQTT   │ MQTT   │
   │        │        │        │        │
┌──▼──┐  ┌──▼──┐  ┌──▼──┐  ┌──▼──┐  ┌──▼──┐
│ESP32│  │ESP32│  │ESP32│  │ESP32│  │ESP32│
│Zone1│  │Zone2│  │Zone3│  │Zone4│  │Zone5│
└──┬──┘  └──┬──┘  └──┬──┘  └──┬──┘  └──┬──┘
   │        │        │        │        │
Sensors  Sensors  Sensors  Sensors  Sensors
```

## 📦 Bill of Materials (Per Zone)

### ESP32 Zone Node
- **ESP32 Dev Board** - $5-8
- **DHT22 Temperature/Humidity Sensor** - $3-5
- **MQ-135 Air Quality/CO2 Sensor** - $2-4
- **BH1750 Light Sensor (I2C)** - $1-2
- **5V Power Supply** - $3-5
- **Jumper Wires** - $2
- **Enclosure** (optional) - $5-10

**Total per zone: ~$16-36**

### Raspberry Pi 5 Gateway (One per Farm)
- **Raspberry Pi 5 (4GB or 8GB)** - $60-80
- **MicroSD Card (32GB+)** - $10-15
- **Power Supply** - $10
- **Case** (optional) - $10-20

**Total: ~$90-125**

---

## 🔧 Part 1: Raspberry Pi 5 Setup

### 1.1 Install Operating System

1. Download Raspberry Pi OS (64-bit) from [raspberrypi.com](https://www.raspberrypi.com/software/)
2. Flash to microSD using Raspberry Pi Imager
3. Boot and complete initial setup
4. Enable SSH: `sudo raspi-config` → Interface Options → SSH → Enable

### 1.2 Install MQTT Broker (Mosquitto)

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Mosquitto MQTT broker
sudo apt install mosquitto mosquitto-clients -y

# Enable and start Mosquitto
sudo systemctl enable mosquitto
sudo systemctl start mosquitto

# Check status
sudo systemctl status mosquitto
```

### 1.3 Configure Mosquitto

Edit the configuration file:

```bash
sudo nano /etc/mosquitto/mosquitto.conf
```

Add the following:

```conf
# Basic Configuration
listener 1883
allow_anonymous true

# Persistence
persistence true
persistence_location /var/lib/mosquitto/

# Logging
log_dest file /var/log/mosquitto/mosquitto.log
log_type all

# Connection settings
max_connections -1
max_queued_messages 1000

# For production, add authentication:
# allow_anonymous false
# password_file /etc/mosquitto/passwd
```

**Optional: Set up MQTT authentication (Recommended for Production)**

```bash
# Create password file
sudo mosquitto_passwd -c /etc/mosquitto/passwd cropwise

# Update mosquitto.conf
sudo nano /etc/mosquitto/mosquitto.conf
# Set: allow_anonymous false
# Set: password_file /etc/mosquitto/passwd

# Restart Mosquitto
sudo systemctl restart mosquitto
```

### 1.4 Test MQTT Broker

Open two terminals:

**Terminal 1 (Subscribe):**
```bash
mosquitto_sub -h localhost -t "test/topic" -v
```

**Terminal 2 (Publish):**
```bash
mosquitto_pub -h localhost -t "test/topic" -m "Hello MQTT!"
```

You should see "Hello MQTT!" in Terminal 1.

### 1.5 Install Python Dependencies

```bash
# Install pip if not already installed
sudo apt install python3-pip -y

# Install required libraries
pip3 install paho-mqtt requests schedule python-dotenv

# Verify installation
python3 -c "import paho.mqtt.client as mqtt; print('MQTT library OK')"
```

### 1.6 Create Gateway Directory

```bash
mkdir -p /home/pi/cropwise-gateway
cd /home/pi/cropwise-gateway
```

### 1.7 Create Gateway Script

Create `gateway.py`:

```bash
nano gateway.py
```

Paste the following code:

```python
#!/usr/bin/env python3
"""
CropWise - Raspberry Pi Gateway
Receives data from ESP32 devices via MQTT and forwards to CropWise API
"""

import json
import time
import requests
import paho.mqtt.client as mqtt
from datetime import datetime
from collections import defaultdict
import os
from dotenv import load_dotenv

# Load configuration
load_dotenv()

# Configuration
MQTT_BROKER = "localhost"
MQTT_PORT = 1883
MQTT_TOPIC_PREFIX = "cropwise"  # ESP32s publish to cropwise/zone/<zone_id>/sensors

API_BASE_URL = os.getenv('API_URL', 'http://localhost:3000/api')
AUTH_TOKEN = os.getenv('AUTH_TOKEN', '')
GATEWAY_ID = os.getenv('GATEWAY_ID', 'RPI5-GATEWAY-001')

# Store latest readings per zone
latest_readings = defaultdict(dict)
last_send_time = defaultdict(float)
SEND_INTERVAL = 300  # Send to API every 5 minutes

class CropWiseGateway:
    def __init__(self):
        self.mqtt_client = mqtt.Client(client_id=GATEWAY_ID)
        self.mqtt_client.on_connect = self.on_connect
        self.mqtt_client.on_message = self.on_message
        self.mqtt_client.on_disconnect = self.on_disconnect
        
    def on_connect(self, client, userdata, flags, rc):
        """Callback when connected to MQTT broker"""
        if rc == 0:
            print(f"✅ Connected to MQTT Broker at {MQTT_BROKER}:{MQTT_PORT}")
            # Subscribe to all zone sensor data
            client.subscribe(f"{MQTT_TOPIC_PREFIX}/zone/+/sensors")
            client.subscribe(f"{MQTT_TOPIC_PREFIX}/zone/+/status")
            print(f"📡 Subscribed to: {MQTT_TOPIC_PREFIX}/zone/+/sensors")
        else:
            print(f"❌ Failed to connect to MQTT Broker, return code {rc}")
    
    def on_disconnect(self, client, userdata, rc):
        """Callback when disconnected from MQTT broker"""
        if rc != 0:
            print(f"⚠️ Unexpected MQTT disconnect. Reconnecting...")
            
    def on_message(self, client, userdata, msg):
        """Callback when message received from ESP32"""
        try:
            topic = msg.topic
            payload = json.loads(msg.payload.decode())
            
            # Extract zone ID from topic: cropwise/zone/<zone_id>/sensors
            parts = topic.split('/')
            if len(parts) >= 3 and parts[2]:
                zone_id = parts[2]
                
                # Store latest readings
                latest_readings[zone_id] = {
                    'zoneId': zone_id,
                    'deviceId': payload.get('device_id', 'unknown'),
                    'temperature': payload.get('temperature'),
                    'humidity': payload.get('humidity'),
                    'co2': payload.get('co2'),
                    'light': payload.get('light'),
                    'airflow': payload.get('airflow'),
                    'soilMoisture': payload.get('soil_moisture'),
                    'source': 'sensor',
                    'timestamp': payload.get('timestamp', datetime.now().isoformat())
                }
                
                print(f"📊 Zone {zone_id}: Temp={payload.get('temperature')}°C, "
                      f"Humidity={payload.get('humidity')}%, "
                      f"CO2={payload.get('co2')}ppm")
                
                # Send to API if interval elapsed
                current_time = time.time()
                if current_time - last_send_time[zone_id] >= SEND_INTERVAL:
                    self.send_to_api(latest_readings[zone_id])
                    last_send_time[zone_id] = current_time
                    
        except json.JSONDecodeError as e:
            print(f"❌ Invalid JSON from {msg.topic}: {e}")
        except Exception as e:
            print(f"❌ Error processing message: {e}")
    
    def send_to_api(self, data):
        """Send sensor data to CropWise API"""
        try:
            url = f"{API_BASE_URL}/telemetry/readings"
            headers = {
                "Authorization": f"Bearer {AUTH_TOKEN}",
                "Content-Type": "application/json"
            }
            
            response = requests.post(url, json=data, headers=headers, timeout=10)
            
            if response.status_code == 201:
                print(f"✅ Sent to API: Zone {data['zoneId']} at {datetime.now()}")
            else:
                print(f"⚠️ API Error {response.status_code}: {response.text}")
                
        except requests.exceptions.RequestException as e:
            print(f"❌ API connection error: {e}")
        except Exception as e:
            print(f"❌ Unexpected error: {e}")
    
    def run(self):
        """Start the gateway"""
        print("=" * 60)
        print("🚀 CropWise Gateway Starting...")
        print(f"📡 Gateway ID: {GATEWAY_ID}")
        print(f"🔌 MQTT Broker: {MQTT_BROKER}:{MQTT_PORT}")
        print(f"🌐 API Server: {API_BASE_URL}")
        print(f"⏰ Send Interval: {SEND_INTERVAL}s")
        print("=" * 60)
        
        # Connect to MQTT broker
        try:
            self.mqtt_client.connect(MQTT_BROKER, MQTT_PORT, 60)
            self.mqtt_client.loop_forever()
        except KeyboardInterrupt:
            print("\n🛑 Gateway stopped by user")
            self.mqtt_client.disconnect()
        except Exception as e:
            print(f"❌ Fatal error: {e}")

if __name__ == "__main__":
    gateway = CropWiseGateway()
    gateway.run()
```

Make it executable:
```bash
chmod +x gateway.py
```

### 1.8 Create Configuration File

Create `.env` file:

```bash
nano .env
```

Add configuration:

```env
# CropWise API Configuration
API_URL=http://YOUR_SERVER_IP:3000/api
AUTH_TOKEN=your_jwt_token_here

# Gateway Configuration
GATEWAY_ID=RPI5-GATEWAY-001
FARM_ID=your_farm_id_here
```

**How to get AUTH_TOKEN:**
1. Open CropWise in browser
2. Login
3. Press F12 → Network tab
4. Refresh page
5. Click any API request
6. Copy the `Authorization: Bearer <TOKEN>` value

### 1.9 Test Gateway

```bash
cd /home/pi/cropwise-gateway
python3 gateway.py
```

You should see:
```
🚀 CropWise Gateway Starting...
✅ Connected to MQTT Broker at localhost:1883
📡 Subscribed to: cropwise/zone/+/sensors
```

### 1.10 Setup Auto-Start Service

Create systemd service:

```bash
sudo nano /etc/systemd/system/cropwise-gateway.service
```

Add the following:

```ini
[Unit]
Description=CropWise Gateway Service
After=network.target mosquitto.service

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi/cropwise-gateway
ExecStart=/usr/bin/python3 /home/pi/cropwise-gateway/gateway.py
Restart=always
RestartSec=10
Environment=PYTHONUNBUFFERED=1

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable cropwise-gateway
sudo systemctl start cropwise-gateway
sudo systemctl status cropwise-gateway

# View logs
sudo journalctl -u cropwise-gateway -f
```

---

## 🔌 Part 2: ESP32 Setup

### 2.1 Hardware Connections

**ESP32 Pinout for Zone Sensors:**

```
┌─────────────────────────────────────────┐
│              ESP32 Dev Board             │
├─────────────────────────────────────────┤
│                                         │
│  3.3V ──────────┬─── DHT22 VCC         │
│                 └─── BH1750 VCC         │
│                                         │
│  GPIO4 ───────────── DHT22 DATA         │
│                                         │
│  GPIO21 ─────────── BH1750 SDA          │
│  GPIO22 ─────────── BH1750 SCL          │
│                                         │
│  GPIO34 ─────────── MQ-135 AOUT         │
│                                         │
│  5V ──────────────── MQ-135 VCC         │
│                                         │
│  GND ────────┬────── DHT22 GND          │
│              ├────── BH1750 GND         │
│              └────── MQ-135 GND         │
│                                         │
└─────────────────────────────────────────┘
```

### 2.2 Install Arduino IDE

1. Download from [arduino.cc](https://www.arduino.cc/en/software)
2. Install ESP32 board support:
   - File → Preferences
   - Additional Board Manager URLs: `https://dl.espressif.com/dl/package_esp32_index.json`
   - Tools → Board → Boards Manager → Search "ESP32" → Install

### 2.3 Install Required Libraries

In Arduino IDE:
- Sketch → Include Library → Manage Libraries
- Install:
  - `PubSubClient` by Nick O'Leary
  - `DHT sensor library` by Adafruit
  - `Adafruit Unified Sensor`
  - `BH1750` by Christopher Laws
  - `ArduinoJson` by Benoit Blanchon

### 2.4 ESP32 Code

Create new sketch in Arduino IDE:

```cpp
/*
 * CropWise - ESP32 Zone Sensor Node
 * Reads environmental sensors and publishes to MQTT
 */

#include <WiFi.h>
#include <PubSubClient.h>
#include <DHT.h>
#include <Wire.h>
#include <BH1750.h>
#include <ArduinoJson.h>

// ============ CONFIGURATION - UPDATE THESE ============
const char* WIFI_SSID = "YOUR_WIFI_SSID";
const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";
const char* MQTT_BROKER = "192.168.1.100";  // Raspberry Pi IP
const int MQTT_PORT = 1883;
const char* DEVICE_ID = "ESP32-ZONE-001";
const char* ZONE_ID = "your_zone_id_here";  // Get from CropWise
// =====================================================

// MQTT Topics
char SENSOR_TOPIC[100];
char STATUS_TOPIC[100];

// Sensor Pins
#define DHT_PIN 4
#define DHT_TYPE DHT22
#define MQ135_PIN 34

// Sensors
DHT dht(DHT_PIN, DHT_TYPE);
BH1750 lightMeter;

WiFiClient espClient;
PubSubClient mqttClient(espClient);

// Timing
unsigned long lastSensorRead = 0;
const unsigned long SENSOR_INTERVAL = 60000;  // Read every 60 seconds

void setup() {
  Serial.begin(115200);
  Serial.println("\n🚀 CropWise ESP32 Sensor Node");
  Serial.println("==================================");
  Serial.printf("Device ID: %s\n", DEVICE_ID);
  Serial.printf("Zone ID: %s\n", ZONE_ID);
  
  // Initialize sensors
  dht.begin();
  Wire.begin();
  
  if (lightMeter.begin()) {
    Serial.println("✅ BH1750 Light Sensor initialized");
  } else {
    Serial.println("⚠️ BH1750 not found, check wiring");
  }
  
  // Setup MQTT topics
  sprintf(SENSOR_TOPIC, "cropwise/zone/%s/sensors", ZONE_ID);
  sprintf(STATUS_TOPIC, "cropwise/zone/%s/status", ZONE_ID);
  
  // Connect to WiFi
  connectWiFi();
  
  // Setup MQTT
  mqttClient.setServer(MQTT_BROKER, MQTT_PORT);
  mqttClient.setCallback(mqttCallback);
  
  Serial.println("✅ Setup complete!");
  Serial.println("==================================\n");
}

void loop() {
  // Maintain MQTT connection
  if (!mqttClient.connected()) {
    reconnectMQTT();
  }
  mqttClient.loop();
  
  // Read and publish sensors
  if (millis() - lastSensorRead >= SENSOR_INTERVAL) {
    readAndPublishSensors();
    lastSensorRead = millis();
  }
}

void connectWiFi() {
  Serial.print("📡 Connecting to WiFi");
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  
  int attempts = 0;
  while (WiFi.status() != WL_CONNECTED && attempts < 20) {
    delay(500);
    Serial.print(".");
    attempts++;
  }
  
  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("\n✅ WiFi connected!");
    Serial.print("IP address: ");
    Serial.println(WiFi.localIP());
    Serial.print("Signal strength: ");
    Serial.print(WiFi.RSSI());
    Serial.println(" dBm");
  } else {
    Serial.println("\n❌ WiFi connection failed!");
  }
}

void reconnectMQTT() {
  while (!mqttClient.connected()) {
    Serial.print("🔌 Connecting to MQTT broker...");
    
    if (mqttClient.connect(DEVICE_ID)) {
      Serial.println(" Connected!");
      
      // Publish online status
      publishStatus("online");
      
      // Subscribe to control topics (future use)
      char controlTopic[100];
      sprintf(controlTopic, "cropwise/zone/%s/control", ZONE_ID);
      mqttClient.subscribe(controlTopic);
      
    } else {
      Serial.print(" Failed, rc=");
      Serial.print(mqttClient.state());
      Serial.println(" Retrying in 5 seconds...");
      delay(5000);
    }
  }
}

void mqttCallback(char* topic, byte* payload, unsigned int length) {
  // Handle control messages (future: actuator control)
  Serial.print("📥 Message on topic: ");
  Serial.println(topic);
  
  // Parse and handle commands here
}

void readAndPublishSensors() {
  Serial.println("📊 Reading sensors...");
  
  // Read DHT22
  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();
  
  // Read light sensor
  float light = lightMeter.readLightLevel();
  
  // Read MQ-135 (CO2 approximation)
  int co2Raw = analogRead(MQ135_PIN);
  int co2 = map(co2Raw, 0, 4095, 400, 2000);  // Basic mapping, calibrate as needed
  
  // Check if readings are valid
  if (isnan(temperature) || isnan(humidity)) {
    Serial.println("❌ Failed to read from DHT sensor!");
    return;
  }
  
  // Display readings
  Serial.printf("   Temperature: %.1f°C\n", temperature);
  Serial.printf("   Humidity: %.1f%%\n", humidity);
  Serial.printf("   CO2: %d ppm\n", co2);
  Serial.printf("   Light: %.0f lux\n", light);
  
  // Create JSON payload
  StaticJsonDocument<256> doc;
  doc["device_id"] = DEVICE_ID;
  doc["zone_id"] = ZONE_ID;
  doc["temperature"] = round(temperature * 10) / 10.0;
  doc["humidity"] = round(humidity * 10) / 10.0;
  doc["co2"] = co2;
  doc["light"] = (int)light;
  doc["airflow"] = 0;  // Add if you have airflow sensor
  doc["soil_moisture"] = nullptr;
  doc["timestamp"] = millis();
  
  // Serialize and publish
  char jsonBuffer[256];
  serializeJson(doc, jsonBuffer);
  
  if (mqttClient.publish(SENSOR_TOPIC, jsonBuffer)) {
    Serial.println("✅ Data published to MQTT");
  } else {
    Serial.println("❌ Failed to publish data!");
  }
  
  Serial.println();
}

void publishStatus(const char* status) {
  StaticJsonDocument<128> doc;
  doc["device_id"] = DEVICE_ID;
  doc["status"] = status;
  doc["uptime"] = millis() / 1000;
  doc["rssi"] = WiFi.RSSI();
  
  char jsonBuffer[128];
  serializeJson(doc, jsonBuffer);
  mqttClient.publish(STATUS_TOPIC, jsonBuffer);
}
```

### 2.5 Configure and Upload

1. **Update configuration in code:**
   ```cpp
   const char* WIFI_SSID = "YourWiFiName";
   const char* WIFI_PASSWORD = "YourWiFiPassword";
   const char* MQTT_BROKER = "192.168.1.100";  // Your RPi IP
   const char* DEVICE_ID = "ESP32-ZONE-001";
   const char* ZONE_ID = "abc-123-def-456";    // From CropWise
   ```

2. **Get Zone ID from CropWise:**
   - Login to CropWise
   - Go to Zones page
   - Click on a zone
   - Copy ID from URL: `http://localhost:5173/zones/<ZONE_ID>`

3. **Connect ESP32 to computer via USB**

4. **Select board and port:**
   - Tools → Board → ESP32 Dev Module
   - Tools → Port → (select your COM port)

5. **Upload:**
   - Click Upload button (→)
   - Wait for "Done uploading"

6. **Open Serial Monitor:**
   - Tools → Serial Monitor
   - Set baud rate to 115200
   - You should see connection status

---

## 📊 Part 3: Monitoring and Testing

### 3.1 Monitor MQTT Traffic on Raspberry Pi

```bash
# Subscribe to all cropwise topics
mosquitto_sub -h localhost -t "cropwise/#" -v

# Subscribe to specific zone
mosquitto_sub -h localhost -t "cropwise/zone/YOUR_ZONE_ID/sensors" -v
```

### 3.2 Test Complete Data Flow

1. **ESP32 reads sensors** (every 60 seconds)
2. **ESP32 publishes to MQTT** → Check Serial Monitor
3. **RPi receives MQTT** → Check `mosquitto_sub` output
4. **RPi sends to API** (every 5 minutes) → Check gateway logs
5. **View in CropWise** → Zone Detail page

### 3.3 View Gateway Logs

```bash
# Real-time logs
sudo journalctl -u cropwise-gateway -f

# Last 100 lines
sudo journalctl -u cropwise-gateway -n 100

# Today's logs
sudo journalctl -u cropwise-gateway --since today
```

### 3.4 Common Issues and Solutions

**Issue: ESP32 won't connect to WiFi**
- Check SSID and password
- Ensure 2.4GHz WiFi (ESP32 doesn't support 5GHz)
- Check signal strength

**Issue: MQTT connection failed**
- Verify Raspberry Pi IP address
- Check Mosquitto is running: `sudo systemctl status mosquitto`
- Test with: `mosquitto_pub -h RASPBERRY_PI_IP -t test -m "hello"`

**Issue: No data in CropWise**
- Check AUTH_TOKEN is valid (try logging out and in)
- Verify gateway logs: `sudo journalctl -u cropwise-gateway -f`
- Check API_URL in `.env` file

---

## 🎯 Part 4: Production Deployment

### 4.1 Multiple Zones Setup

For each additional zone:

1. Flash new ESP32 with unique:
   ```cpp
   const char* DEVICE_ID = "ESP32-ZONE-002";  // Increment
   const char* ZONE_ID = "zone_uuid_2";       // Different zone
   ```

2. No changes needed on Raspberry Pi - automatically handles multiple zones!

### 4.2 Power Considerations

**ESP32:**
- Power consumption: ~200mA active, ~20mA deep sleep
- Use 5V 2A power supply per ESP32
- Or use USB power banks for portable monitoring

**Raspberry Pi 5:**
- Requires 5V 5A USB-C power supply
- Use official Raspberry Pi power supply for reliability

### 4.3 Enclosures

**ESP32 Nodes:**
- Use waterproof IP65 enclosures for high humidity
- Mount DHT22 outside enclosure with cable gland
- Add ventilation holes for MQ-135

**Raspberry Pi:**
- Use official Raspberry Pi case with cooling
- Mount in climate-controlled area if possible

### 4.4 Network Reliability

**Static IP for Raspberry Pi:**

```bash
sudo nano /etc/dhcpcd.conf
```

Add:
```conf
interface wlan0
static ip_address=192.168.1.100/24
static routers=192.168.1.1
static domain_name_servers=8.8.8.8 8.8.4.4
```

**WiFi Reconnection for ESP32:**
Already handled in code - automatically reconnects

### 4.5 Data Backup

**Backup MQTT configuration:**
```bash
cp /etc/mosquitto/mosquitto.conf /home/pi/backup/
```

**Backup gateway code:**
```bash
cp -r /home/pi/cropwise-gateway /home/pi/backup/
```

---

## 📈 Advanced Features (Future)

### Actuator Control
- Add relays to ESP32 for fans, heaters, humidifiers
- Subscribe to control topics from CropWise
- Implement PID control loops

### Edge Computing
- Run TensorFlow Lite on RPi for local predictions
- Detect anomalies before sending to cloud
- Offline operation mode

### Multiple Farms
- Deploy multiple Raspberry Pi gateways
- Each gateway manages one farm
- Central CropWise server

### OTA Updates
- Update ESP32 firmware over WiFi
- No need to physically access devices
- Use ESP32 OTA library

---

## 🔒 Security Best Practices

1. **Enable MQTT Authentication**
   ```bash
   sudo mosquitto_passwd -c /etc/mosquitto/passwd cropwise
   ```

2. **Use HTTPS for API**
   - Enable SSL in production
   - Use Let's Encrypt certificates

3. **Firewall Rules**
   ```bash
   sudo ufw allow 1883  # MQTT
   sudo ufw allow 22    # SSH
   sudo ufw enable
   ```

4. **Regular Updates**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

---

## 📞 Troubleshooting Commands

```bash
# Check Mosquitto status
sudo systemctl status mosquitto

# Check gateway service
sudo systemctl status cropwise-gateway

# View gateway logs
sudo journalctl -u cropwise-gateway -f

# Test MQTT locally
mosquitto_pub -h localhost -t test -m "hello"
mosquitto_sub -h localhost -t test

# Check Raspberry Pi IP
hostname -I

# Check open ports
sudo netstat -tulpn | grep LISTEN

# Restart services
sudo systemctl restart mosquitto
sudo systemctl restart cropwise-gateway
```

---

## 📚 Additional Resources

- [MQTT Protocol Documentation](https://mqtt.org/mqtt-specification/)
- [ESP32 Arduino Core](https://github.com/espressif/arduino-esp32)
- [Mosquitto Documentation](https://mosquitto.org/documentation/)
- [Paho MQTT Python Client](https://www.eclipse.org/paho/index.php?page=clients/python/docs/index.php)

---

## ✅ Checklist

### Raspberry Pi Setup
- [ ] Raspberry Pi OS installed
- [ ] Mosquitto MQTT broker installed and running
- [ ] Python dependencies installed
- [ ] Gateway script created
- [ ] Configuration file (.env) created with valid tokens
- [ ] Gateway service enabled and running
- [ ] Can subscribe to MQTT topics with mosquitto_sub

### ESP32 Setup (Per Zone)
- [ ] Sensors connected correctly (DHT22, MQ-135, BH1750)
- [ ] Arduino IDE installed with ESP32 support
- [ ] Required libraries installed
- [ ] Code uploaded with correct WiFi/MQTT/Zone config
- [ ] Serial monitor shows successful connection
- [ ] Data visible in MQTT with mosquitto_sub

### Integration Testing
- [ ] ESP32 publishes data to MQTT (check Serial Monitor)
- [ ] Raspberry Pi receives data (check mosquitto_sub)
- [ ] Gateway forwards to CropWise API (check logs)
- [ ] Data visible in CropWise Zone Detail page
- [ ] Charts updating with real sensor data

---

**🎉 Your professional IoT mushroom farm monitoring system is ready!**

For questions or issues, refer to the troubleshooting section or contact support.

