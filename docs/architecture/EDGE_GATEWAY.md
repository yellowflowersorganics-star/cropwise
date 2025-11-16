# 🏗️ CropWise - Edge Gateway Architecture
## Organization → Unit → Zone → ESP32 → Devices

**Updated Architecture**: ESP32s connect to local Raspberry Pi, not directly to cloud

---

## 📊 Hierarchical Structure

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ORGANIZATION (Customer/Tenant)                    │
│                     e.g., "Green Valley Mushrooms"                  │
└─────────────────────────┬───────────────────────────────────────────┘
                          │
          ┌───────────────┴────────────────┐
          │                                │
    ┌─────▼─────┐                    ┌────▼─────┐
    │  UNIT 1   │                    │  UNIT 2  │
    │ Building A│                    │Building B│
    │ (Location)│                    │(Location)│
    └─────┬─────┘                    └────┬─────┘
          │                               │
          │                               │
┌─────────▼─────────────────┐    ┌────────▼──────────────────┐
│   Raspberry Pi Gateway    │    │  Raspberry Pi Gateway     │
│   IP: 192.168.1.100      │    │  IP: 192.168.1.200       │
│   Local MQTT Broker       │    │  Local MQTT Broker        │
│   Node-RED Runtime        │    │  Node-RED Runtime         │
└─────────┬─────────────────┘    └────────┬──────────────────┘
          │                               │
    ┌─────┴─────────┐              ┌──────┴──────┐
    │               │              │             │
┌───▼───┐      ┌───▼───┐      ┌───▼───┐    ┌───▼───┐
│Zone A │      │Zone B │      │Zone C │    │Zone D │
│Room 1 │      │Room 2 │      │Room 3 │    │Room 1 │
└───┬───┘      └───┬───┘      └───┬───┘    └───┬───┘
    │              │              │            │
┌───▼────┐    ┌───▼────┐    ┌───▼────┐  ┌───▼────┐
│ESP32_01│    │ESP32_02│    │ESP32_03│  │ESP32_04│
└───┬────┘    └───┬────┘    └───┬────┘  └───┬────┘
    │             │             │           │
    ├─Sensors     ├─Sensors     ├─Sensors   ├─Sensors
    │ -Temp/RH    │ -Temp/RH    │ -Temp/RH  │ -Temp/RH
    │ -CO₂        │ -CO₂        │ -CO₂      │ -CO₂
    │ -Light      │ -Light      │ -Light    │ -Light
    │             │             │           │
    └─Relays      └─Relays      └─Relays    └─Relays
      -Fan          -Fan          -Fan        -Fan
      -Humidifier   -Humidifier   -Humidifier -Humidifier
      -Heater       -Heater       -Chiller    -Boiler
      -Light        -Light        -FCU        -AHU
```

---

## 🔄 Communication Flow

### 1. Local Communication (ESP32 ↔ Raspberry Pi)

```
ESP32 Controller (Zone A)
    │ WiFi: 192.168.1.x
    │ MQTT: mosquitto://192.168.1.100:1883
    ▼
Raspberry Pi Gateway (Unit 1)
    │ Topic: unit1/zone_a/telemetry
    │ Local processing & aggregation
    │ Node-RED flows
    ▼
Buffer & Forward to Cloud
```

**Benefits**:
- ✅ Works even if internet is down
- ✅ Faster response times (local network)
- ✅ Reduced cloud bandwidth costs
- ✅ Better security (Pi acts as firewall)

### 2. Cloud Communication (Raspberry Pi ↔ Cloud)

```
Raspberry Pi Gateway
    │ Internet/WiFi
    │ MQTT/TLS: mqtt.cropwise.cloud:8883
    ▼
Cloud MQTT Broker (EMQX)
    │ Topic: yfcloud/<org_id>/<unit_id>/#
    ▼
CropWise Cloud API
    │ Parse, Store, Analyze
    ▼
Web Dashboard / Mobile App
```

**Benefits**:
- ✅ Aggregated data (less cloud traffic)
- ✅ Single connection per unit (not per zone)
- ✅ Edge processing reduces cloud load
- ✅ Centralized monitoring

---

## 📡 MQTT Topic Structure

### Local Topics (ESP32 → Raspberry Pi)

```
# Pattern
unit<unit_id>/zone_<zone_id>/<message_type>

# Examples - ESP32 publishes to:
unit1/zone_a/telemetry         → Sensor readings
unit1/zone_a/status            → Device status
unit1/zone_a/alert             → Warnings/errors

# Raspberry Pi publishes to:
unit1/zone_a/setpoints         → Environmental targets
unit1/zone_a/command           → Control commands
unit1/zone_a/config            → Configuration updates
```

### Cloud Topics (Raspberry Pi ↔ Cloud)

```
# Pattern
yfcloud/<org_id>/<unit_id>/<message_type>

# Examples - Raspberry Pi publishes to:
yfcloud/org_abc123/unit_001/telemetry_aggregated
yfcloud/org_abc123/unit_001/gateway_status
yfcloud/org_abc123/unit_001/alert

# Cloud publishes to:
yfcloud/org_abc123/unit_001/setpoints_bulk
yfcloud/org_abc123/unit_001/command
yfcloud/org_abc123/unit_001/firmware_update
```

---

## 🔧 Device Types & Roles

### 1. ESP32 Zone Controller
**Role**: Control one zone (room)  
**Connection**: WiFi → Local Raspberry Pi  
**MQTT**: Subscribe to local broker  
**Functions**:
- Read sensors (Temp, RH, CO₂, Light)
- Control relays (Fan, Humidifier, Heater, etc.)
- Execute recipe setpoints
- Report telemetry every 60 seconds
- Handle local commands

**Configuration**:
```cpp
// ESP32 config
WiFi SSID: "Unit1_Network"
MQTT Broker: "192.168.1.100"
MQTT Port: 1883
Zone ID: "zone_a"
Unit ID: "unit_001"
```

### 2. Raspberry Pi Gateway
**Role**: Local hub for one unit  
**Connection**: WiFi → Internet → Cloud  
**MQTT**: Local broker + Cloud client  
**Functions**:
- Run local MQTT broker (Mosquitto)
- Aggregate data from all ESP32s in unit
- Run Node-RED for automation
- Buffer data during internet outages
- Forward to cloud
- Handle OTA updates for ESP32s

**Software Stack**:
```bash
OS: Raspberry Pi OS Lite
MQTT: Mosquitto (local broker)
Automation: Node-RED
Language: Node.js / Python
Storage: SQLite (local buffer)
```

### 3. Cloud Backend
**Role**: Multi-tenant SaaS platform  
**Connection**: Public internet  
**Functions**:
- Receive aggregated data from all units
- Store in InfluxDB (telemetry) + PostgreSQL (metadata)
- Recipe management
- User authentication
- Analytics & ML
- Billing
- Dashboard APIs

---

## 💾 Data Flow Examples

### Example 1: Temperature Reading

```
1. SHT31 Sensor (Zone A)
   Temperature: 24.5°C
   
2. ESP32_01 reads sensor
   
3. ESP32_01 publishes to local MQTT:
   Topic: unit1/zone_a/telemetry
   Payload: {
     "temperature": 24.5,
     "humidity": 72.3,
     "co2": 850,
     "timestamp": 1699878900
   }
   
4. Raspberry Pi receives & aggregates:
   - Combines data from all zones
   - Adds unit context
   - Buffers locally
   
5. Raspberry Pi publishes to cloud:
   Topic: yfcloud/org_abc123/unit_001/telemetry_aggregated
   Payload: {
     "unit_id": "unit_001",
     "gateway_id": "rpi_mac_address",
     "zones": [
       {
         "zone_id": "zone_a",
         "temperature": 24.5,
         "humidity": 72.3,
         "co2": 850
       },
       {
         "zone_id": "zone_b",
         "temperature": 23.8,
         "humidity": 75.1,
         "co2": 900
       }
     ],
     "timestamp": 1699878900
   }
   
6. Cloud API receives, parses, stores in InfluxDB

7. Dashboard queries and displays
```

### Example 2: Recipe Setpoint Update

```
1. User updates recipe in dashboard

2. Cloud backend calculates new setpoints
   Zone A: Temp 25°C, RH 75%, CO₂ 900ppm

3. Cloud publishes to MQTT:
   Topic: yfcloud/org_abc123/unit_001/setpoints_bulk
   Payload: {
     "zones": [
       {
         "zone_id": "zone_a",
         "temperature": 25.0,
         "humidity": 75.0,
         "co2": 900
       }
     ]
   }

4. Raspberry Pi receives and distributes:
   Topic: unit1/zone_a/setpoints
   Payload: {
     "temperature": 25.0,
     "humidity": 75.0,
     "co2": 900
   }

5. ESP32_01 receives and applies:
   - Updates PID controller targets
   - Adjusts fan/humidifier/heater
   - Confirms receipt
```

---

## 🔐 Security Layers

### Layer 1: Local Network (ESP32 → Pi)
```
- WPA2/WPA3 WiFi encryption
- MQTT username/password per ESP32
- Local network isolation (VLAN optional)
- No internet exposure
```

### Layer 2: Gateway (Raspberry Pi)
```
- Firewall: Only allow outbound MQTT/HTTPS
- VPN option for remote access
- Local data encryption
- SSH key-based access only
- Automatic security updates
```

### Layer 3: Cloud (Pi → Backend)
```
- TLS 1.3 for all MQTT traffic
- Client certificates per gateway
- JWT authentication for API
- Organization-level data isolation
- Encrypted data at rest
```

---

## 🏭 Device Configuration

### ESP32 Controller Configuration

```json
{
  "device_type": "esp32_controller",
  "device_id": "ESP32_AABBCCDDEEFF",
  "organization_id": "org_abc123",
  "unit_id": "unit_001",
  "zone_id": "zone_a",
  "network": {
    "ssid": "Unit1_Network",
    "password": "encrypted"
  },
  "mqtt": {
    "broker": "192.168.1.100",
    "port": 1883,
    "username": "esp32_zone_a",
    "password": "encrypted",
    "topics": {
      "telemetry": "unit1/zone_a/telemetry",
      "status": "unit1/zone_a/status",
      "setpoints_sub": "unit1/zone_a/setpoints",
      "command_sub": "unit1/zone_a/command"
    }
  },
  "sensors": {
    "temperature_humidity": {
      "type": "SHT31",
      "i2c_address": "0x44"
    },
    "co2": {
      "type": "MH-Z19C",
      "uart_rx": 16,
      "uart_tx": 17
    }
  },
  "relays": {
    "fan": { "pin": 25 },
    "humidifier": { "pin": 26 },
    "heater": { "pin": 27 },
    "light": { "pin": 32 },
    "chiller": { "pin": 33 },
    "fcu": { "pin": 14 }
  }
}
```

### Raspberry Pi Gateway Configuration

```json
{
  "device_type": "raspberry_pi_gateway",
  "gateway_id": "RPI_B827EB123456",
  "organization_id": "org_abc123",
  "unit_id": "unit_001",
  "network": {
    "wifi_ssid": "Internet_Network",
    "ethernet": true,
    "ip_address": "192.168.1.100"
  },
  "local_mqtt": {
    "broker_port": 1883,
    "max_connections": 50,
    "persistence": true
  },
  "cloud_mqtt": {
    "broker": "mqtt.cropwise.cloud",
    "port": 8883,
    "tls": true,
    "client_cert": "/etc/cropwise/gateway.crt",
    "topics": {
      "telemetry": "yfcloud/org_abc123/unit_001/telemetry_aggregated",
      "status": "yfcloud/org_abc123/unit_001/gateway_status",
      "setpoints_sub": "yfcloud/org_abc123/unit_001/setpoints_bulk"
    }
  },
  "zones": [
    {
      "zone_id": "zone_a",
      "esp32_id": "ESP32_AABBCCDDEEFF",
      "room_name": "Room 1"
    },
    {
      "zone_id": "zone_b",
      "esp32_id": "ESP32_112233445566",
      "room_name": "Room 2"
    }
  ],
  "buffer": {
    "max_size_mb": 100,
    "retention_hours": 72
  }
}
```

---

## 🔧 Installation Process

### Step 1: Set Up Unit (Building/Location)

1. **Physical Setup**
   - Install WiFi router/access point
   - Set up Raspberry Pi in accessible location
   - Power outlets for all equipment

2. **Raspberry Pi Setup**
   ```bash
   # Flash Raspberry Pi OS
   # Install dependencies
   sudo apt update && sudo apt upgrade -y
   sudo apt install mosquitto mosquitto-clients nodejs npm
   
   # Install CropWise Gateway software
   cd /opt
   git clone https://github.com/yellowflowers/cropwise-gateway.git
   cd cropwise-gateway
   npm install
   
   # Configure
   sudo nano /etc/cropwise/gateway.json
   # Add organization_id, unit_id, cloud credentials
   
   # Start services
   sudo systemctl enable cropwise-gateway
   sudo systemctl start cropwise-gateway
   ```

### Step 2: Provision ESP32 Controllers

1. **Flash Firmware**
   ```bash
   cd edge/esp32
   # Edit src/config.h with:
   # - WiFi SSID (unit network)
   # - Local Pi IP address
   # - Unit ID and Zone ID
   
   pio run -t upload
   ```

2. **Register in Cloud**
   - Dashboard → Units → Unit 1 → Add Zone
   - Enter ESP32 MAC address
   - Assign zone name
   - Generate MQTT credentials

3. **Physical Installation**
   - Mount ESP32 in zone/room
   - Connect sensors
   - Connect relay outputs to devices
   - Power on and verify connection

### Step 3: Verify Operation

```bash
# On Raspberry Pi, check local MQTT:
mosquitto_sub -h localhost -t "unit1/#" -v

# Should see:
# unit1/zone_a/telemetry {...}
# unit1/zone_b/telemetry {...}

# Check cloud connection:
tail -f /var/log/cropwise/gateway.log

# Should see:
# [INFO] Connected to cloud MQTT
# [INFO] Published telemetry for 2 zones
```

---

## 📊 Advantages of This Architecture

### vs Direct ESP32 → Cloud

| Feature | Direct to Cloud | Via Gateway (Our Arch) |
|:--------|:---------------|:-----------------------|
| **Internet dependency** | High (always needed) | Low (local control works) |
| **Cloud bandwidth** | High (every ESP32) | Low (aggregated) |
| **Response time** | 200-500ms | 10-50ms (local) |
| **Cost** | High (per device) | Low (per unit) |
| **Reliability** | Single point of failure | Local fallback |
| **Security** | Direct exposure | Gateway firewall |
| **Maintenance** | Per device OTA | Centralized from Pi |

### Benefits

1. **Resilience**: Local control continues during internet outages
2. **Cost**: 10 ESP32s = 1 cloud connection instead of 10
3. **Speed**: Local commands execute in milliseconds
4. **Security**: ESP32s never exposed to internet
5. **Scalability**: Add zones without cloud load
6. **Bandwidth**: Aggregate and compress data
7. **Edge Computing**: Process locally, send summaries

---

## 🎯 Use Cases

### Use Case 1: Mushroom Farm (10 Rooms)

```
Organization: "Fresh Mushrooms Pvt Ltd"
├── Unit 1: "Main Building"
│   ├── Raspberry Pi Gateway (192.168.1.100)
│   ├── Zone 1-5: Incubation Rooms (ESP32 each)
│   └── Zone 6-10: Fruiting Rooms (ESP32 each)
│
Device count:
- 1 Raspberry Pi
- 10 ESP32s
- 10 Temp/RH sensors
- 10 CO₂ sensors
- 40 Relays (4 per zone)

Network:
- Local: 192.168.1.0/24
- Internet: Single connection
- Cloud connections: 1 (from Pi)
```

### Use Case 2: Multi-Location Farm

```
Organization: "Green Valley Organics"
├── Unit 1: "North Farm" (Location A)
│   ├── Raspberry Pi
│   └── 5 Zones
│
├── Unit 2: "South Farm" (Location B)
│   ├── Raspberry Pi
│   └── 8 Zones
│
├── Unit 3: "Processing Unit" (Location C)
│   ├── Raspberry Pi
│   └── 3 Zones

Total:
- 3 Raspberry Pis
- 16 ESP32s
- 3 Cloud connections (one per unit)
- 1 unified dashboard
```

---

## 📝 Next Steps

1. ✅ **Models created** - Unit, updated Zone/Device
2. ⏳ **Update Organization** - Add unit relationship
3. ⏳ **Create Unit controller** - CRUD APIs
4. ⏳ **Update MQTT service** - Handle aggregated topics
5. ⏳ **Raspberry Pi software** - Gateway application
6. ⏳ **Update ESP32 firmware** - Local MQTT connection
7. ⏳ **Dashboard UI** - Unit management interface

---

**This architecture gives you:**
- ✅ Better reliability (works offline)
- ✅ Lower costs (fewer cloud connections)
- ✅ Faster response (local control)
- ✅ Better security (gateway firewall)
- ✅ Easier scaling (add zones freely)
- ✅ Professional edge-computing design

**Your platform is now enterprise-grade!** 🏭🚀

