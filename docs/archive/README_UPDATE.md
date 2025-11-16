# 🌱 CropWise (Yellow Flowers SmartFarm Cloud)

> **Enterprise-grade IoT platform for controlled-environment agriculture with edge-gateway architecture**

A commercial multi-tenant SaaS platform for precision farming that enables farmers to manage mushroom farms, vertical farms, greenhouses, and hydroponic systems through intelligent climate control and crop recipes.

---

## 🏗️ **NEW Architecture: Edge-Gateway Model**

### Organization → Unit → Zone → ESP32

```
🏢 Organization (Customer/Tenant)
    │
    ├── 🏭 Unit 1 (Building A)
    │   ├── 🍓 Raspberry Pi Gateway ──────┐ (WiFi → Cloud)
    │   ├── 🚪 Zone A → 📱 ESP32_01 ──────┤
    │   ├── 🚪 Zone B → 📱 ESP32_02 ──────┤ (Local MQTT)
    │   └── 🚪 Zone C → 📱 ESP32_03 ──────┘
    │
    └── 🏭 Unit 2 (Building B)
        ├── 🍓 Raspberry Pi Gateway ──────┐ (WiFi → Cloud)
        ├── 🚪 Zone D → 📱 ESP32_04 ──────┤
        └── 🚪 Zone E → 📱 ESP32_05 ──────┘ (Local MQTT)
```

### Communication Flow

```
ESP32 Controllers
    │ WiFi → Local Network
    │ MQTT: 192.168.1.100:1883
    ▼
Raspberry Pi Gateway (per unit)
    │ Aggregates data from all zones
    │ Buffers during internet outages
    │ Local automation (Node-RED)
    ▼
    │ Internet → MQTT/TLS
    │ Topics: yfcloud/<org>/<unit>/*
    ▼
CropWise Cloud (SaaS)
    │ Multi-tenant backend
    │ Crop recipes & analytics
    │ Billing & subscriptions
    ▼
Web Dashboard / Mobile App
```

### ✅ **Why This Architecture?**

| Benefit | Description |
|:--------|:------------|
| **Works Offline** | Local control continues without internet connection |
| **Lower Cost** | 1 cloud connection per unit (not per zone/device) |
| **Faster Response** | Local commands execute in <50ms vs 200-500ms |
| **Better Security** | ESP32s never exposed to internet directly |
| **Easy Scaling** | Add zones without increasing cloud load |
| **Edge Processing** | Aggregate and compress data at gateway |
| **Centralized OTA** | Update all ESP32s from Raspberry Pi |

---

## 🧩 System Components

### 1. **ESP32 Zone Controller** (One per room)
- **Reads sensors**: Temperature, Humidity, CO₂, Light
- **Controls relays**: Fan, Humidifier, Heater, Chiller, FCU, AHU, Light
- **Executes recipes**: PID control for environmental setpoints
- **Publishes telemetry**: Every 60 seconds to local Pi
- **Connection**: WiFi → Local Raspberry Pi MQTT

### 2. **Raspberry Pi Gateway** (One per unit/building)
- **Local MQTT broker**: Mosquitto for ESP32s
- **Data aggregation**: Combines all zone data
- **Node-RED automation**: Local logic and rules
- **Buffer storage**: SQLite for internet outages
- **Cloud bridge**: Forwards aggregated data
- **OTA management**: Updates ESP32 firmware

### 3. **Cloud Backend** (Multi-tenant SaaS)
- **User management**: Organizations, users, roles
- **Unit management**: Buildings, zones, devices
- **Crop recipes**: Environmental parameters per crop/stage
- **Telemetry storage**: InfluxDB (time-series) + PostgreSQL
- **Analytics**: Yield tracking, insights, alerts
- **Billing**: Razorpay subscription management

### 4. **Web Dashboard** (React)
- **Real-time monitoring**: Temperature, humidity, CO₂ charts
- **Zone control**: Start/stop, apply recipes, setpoints
- **Multi-unit view**: Manage multiple locations
- **Analytics**: Historical data, reports, exports
- **User management**: Invite team members, roles

---

## 💾 Database Schema (Updated)

```sql
-- Multi-tenant hierarchy
organizations (id, name, subscriptionTier, maxZones, maxUsers, ...)
    ↓
units (id, organizationId, name, gatewayId, gatewayStatus, totalZones, ...)
    ↓
zones (id, organizationId, unitId, name, activeRecipeId, status, ...)
    ↓
devices (id, organizationId, unitId, zoneId, deviceType, deviceId, status, ...)
    ↓
telemetry (timestamp, zoneId, temperature, humidity, co2, ...)
```

**Device Types**:
- `raspberry_pi_gateway` - One per unit
- `esp32_controller` - One per zone
- `sensor` - Individual sensors (future)
- `actuator` - Individual actuators (future)

---

## 📡 MQTT Topics

### Local (ESP32 → Raspberry Pi)

```
# ESP32 publishes
unit1/zone_a/telemetry         → {"temperature": 24.5, "humidity": 72.3, "co2": 850}
unit1/zone_a/status            → {"online": true, "uptime": 86400}
unit1/zone_a/alert             → {"type": "temp_high", "value": 28.5}

# Raspberry Pi publishes
unit1/zone_a/setpoints         → {"temperature": 25.0, "humidity": 75.0}
unit1/zone_a/command           → {"action": "restart"}
unit1/zone_a/config            → {"recipe": "mushroom_fruiting"}
```

### Cloud (Raspberry Pi → Cloud)

```
# Raspberry Pi publishes
yfcloud/org_abc123/unit_001/telemetry_aggregated  → All zones data
yfcloud/org_abc123/unit_001/gateway_status        → Pi health

# Cloud publishes
yfcloud/org_abc123/unit_001/setpoints_bulk        → Update all zones
yfcloud/org_abc123/unit_001/firmware_update       → OTA trigger
```

---

## 🚀 Quick Start

### Option 1: Local Development

```bash
# 1. Clone repository
git clone https://github.com/yellowflowersorganics-star/cropwise.git
cd cropwise

# 2. Start backend (uses SQLite)
cd backend
npm install
npm run dev
# Backend: http://localhost:3000

# 3. Start frontend
cd ../frontend
npm install
npm run dev
# Frontend: http://localhost:5173

# 4. Create account
# Open http://localhost:5173/register
# Email: your@email.com
# Password: your_password
```

### Option 2: Docker (All services)

```bash
# Start everything
docker-compose up -d

# Services:
# - Backend: http://localhost:3000
# - Frontend: http://localhost:5173
# - PostgreSQL: localhost:5432
# - MQTT Broker: localhost:1883
# - Redis: localhost:6379
```

### Option 3: Hardware Setup

#### **A. Setup Raspberry Pi Gateway**

```bash
# On Raspberry Pi
curl -sSL https://raw.githubusercontent.com/yellowflowers/cropwise-gateway/main/install.sh | bash

# Configure
sudo nano /etc/cropwise/gateway.json
# Set: organization_id, unit_id, cloud credentials

# Start service
sudo systemctl enable cropwise-gateway
sudo systemctl start cropwise-gateway
```

#### **B. Flash ESP32 Controller**

```bash
# On your computer
cd edge/esp32

# Copy and edit config
cp src/config.h.example src/config.h
nano src/config.h

# Set:
# - ORGANIZATION_ID
# - UNIT_ID
# - ZONE_ID
# - WIFI_SSID (local unit network)
# - MQTT_BROKER (Raspberry Pi IP)
# - MQTT_USERNAME/PASSWORD

# Flash
pio run -t upload
pio device monitor  # View logs
```

#### **C. Register Devices in Dashboard**

1. Login to dashboard
2. Go to "Units" → "Add Unit"
   - Name: "Building A"
   - Gateway ID: (Raspberry Pi MAC address)
3. Go to "Zones" → "Add Zone"
   - Unit: Building A
   - Name: "Incubation Room 1"
   - ESP32 ID: (MAC address)
4. View real-time telemetry!

---

## 📊 Subscription Tiers

| Tier | Price | Zones | Users | Features |
|:-----|:------|:------|:------|:---------|
| **Trial** | Free (14 days) | 1 | 2 | Basic monitoring |
| **Starter** | ₹1,500/mo | 1 | 3 | Local control |
| **Growth** | ₹3,000/mo | 5 | 10 | Cloud sync, Analytics |
| **Enterprise** | ₹6,000/mo | 10+ | 50 | AI insights, API access, White-label |

---

## 🎯 Use Cases

### ✅ Mushroom Farming
- **Incubation**: 25°C, 85% RH, 1000ppm CO₂
- **Fruiting**: 18°C, 90% RH, 800ppm CO₂
- **Multi-zone**: Different stages in different rooms
- **Automation**: Automatic climate transitions

### ✅ Vertical Farms
- **LED control**: Spectrum, intensity, photoperiod
- **Nutrient dosing**: EC, pH monitoring
- **Climate**: Temperature, humidity per rack
- **Yield tracking**: Per crop cycle

### ✅ Greenhouses
- **Ventilation**: Automatic fan/vent control
- **Heating**: Boiler, FCU, AHU management
- **Irrigation**: Scheduled watering
- **Multi-crop**: Different zones for different crops

### ✅ Hydroponics
- **Water quality**: pH, EC, temperature
- **Pump scheduling**: Flood/drain cycles
- **Nutrient mixing**: Automated dosing
- **Monitoring**: 24/7 alerts

---

## 🛠️ Tech Stack

| Layer | Technology |
|:------|:-----------|
| **Backend** | Node.js, Express, Sequelize, PostgreSQL, InfluxDB, Redis, MQTT.js |
| **Frontend** | React 18, Vite, Tailwind CSS, Zustand, TanStack Query, Recharts |
| **Edge Gateway** | Raspberry Pi 4B, Node.js, Mosquitto MQTT, Node-RED, SQLite |
| **Controller** | ESP32, Arduino/PlatformIO, MQTT, SHT31, MH-Z19C, BH1750 |
| **Infrastructure** | Docker, Docker Compose, GitHub Actions |
| **Cloud** | AWS/Azure/GCP, EMQX Cloud, Razorpay |

---

## 📚 Documentation

- **[Architecture Details](./ARCHITECTURE_SUMMARY.md)** - Complete system design
- **[Edge Gateway Architecture](./docs/ARCHITECTURE_EDGE_GATEWAY.md)** - Local network setup
- **[Getting Started Guide](./docs/GETTING_STARTED.md)** - Step-by-step tutorial
- **[API Documentation](./docs/API_UNITS.md)** - REST API reference
- **[Commercial Platform](./docs/COMMERCIAL_PLATFORM.md)** - Business model
- **[Migration Plan](./docs/MIGRATION_PLAN.md)** - Implementation roadmap

---

## 📦 Project Structure

```
cropwise/
├── backend/                    # Node.js backend
│   ├── src/
│   │   ├── models/            # Sequelize models (Organization, Unit, Zone, Device, ...)
│   │   ├── routes/            # API routes (units, zones, devices, telemetry, ...)
│   │   ├── services/          # Business logic (MQTT, billing, recipe engine, ...)
│   │   ├── middleware/        # Auth, tenant context, error handling
│   │   └── config/            # Database, MQTT, Redis config
│   └── package.json
│
├── frontend/                   # React dashboard
│   ├── src/
│   │   ├── pages/             # Dashboard, Farms, Zones, Analytics, Settings
│   │   ├── components/        # Reusable UI components
│   │   ├── hooks/             # Custom React hooks
│   │   └── store/             # Zustand state management
│   └── package.json
│
├── edge/                       # Edge devices
│   ├── esp32/                 # ESP32 firmware (PlatformIO)
│   │   ├── src/
│   │   │   ├── main.cpp
│   │   │   ├── config.h.example
│   │   │   ├── sensors.cpp    # Sensor drivers
│   │   │   ├── actuators.cpp  # Relay control
│   │   │   └── mqtt_client.cpp
│   │   └── platformio.ini
│   │
│   └── raspberry-pi/          # Gateway software (separate repo)
│       └── gateway.js
│
├── shared/                     # Shared resources
│   ├── schemas/               # JSON schemas (crop recipes, MQTT messages)
│   └── examples/              # Example crop recipes
│
├── docs/                       # Documentation
│   ├── GETTING_STARTED.md
│   ├── ARCHITECTURE_EDGE_GATEWAY.md
│   ├── API_UNITS.md
│   ├── COMMERCIAL_PLATFORM.md
│   └── MIGRATION_PLAN.md
│
├── docker-compose.yml          # Multi-service Docker setup
├── ARCHITECTURE_SUMMARY.md     # System overview
└── README.md                   # This file
```

---

## 🤝 Contributing

We welcome contributions! Please see [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines.

---

## 📄 License

MIT License - see [LICENSE](./LICENSE)

---

## 📞 Support & Contact

- **Website**: https://yellowflowers.tech
- **Email**: support@yellowflowers.tech
- **Documentation**: https://docs.yellowflowers.tech
- **GitHub Issues**: https://github.com/yellowflowersorganics-star/cropwise/issues

---

**Built with ❤️ for controlled-environment agriculture** 🌱🍄🥬🍓

**Yellow Flowers SmartFarm Cloud** - Grow smarter, not harder.

