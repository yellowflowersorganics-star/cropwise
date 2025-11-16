# 🏗️ CropWise Architecture Summary
## Updated: Edge Gateway Model (Org → Unit → Zone → ESP32)

---

## 📊 Hierarchical Structure

```
Organization (Customer/Tenant)
    └── Multiple Units (Buildings/Locations)
        └── One Raspberry Pi Gateway per Unit
            └── Multiple Zones (Rooms)
                └── One ESP32 Controller per Zone
                    └── Sensors + Relays + Devices
```

---

## 🌐 Real-World Example: Mushroom Farm

```
🏢 Organization: "Fresh Mushrooms Pvt Ltd"
    │
    ├── 🏭 Unit 1: "Main Building" (Location A)
    │   │
    │   ├── 🍓 Raspberry Pi Gateway
    │   │   ├── IP: 192.168.1.100
    │   │   ├── Local MQTT Broker (port 1883)
    │   │   └── Node-RED Automation
    │   │
    │   ├── 🚪 Zone A: "Incubation Room 1"
    │   │   └── 📱 ESP32_01
    │   │       ├── Sensors: Temp/RH, CO₂, Light
    │   │       └── Relays → Fan, Humidifier, Heater, Light
    │   │
    │   ├── 🚪 Zone B: "Incubation Room 2"
    │   │   └── 📱 ESP32_02
    │   │
    │   ├── 🚪 Zone C: "Fruiting Room 1"
    │   │   └── 📱 ESP32_03
    │   │
    │   └── 🚪 Zone D: "Fruiting Room 2"
    │       └── 📱 ESP32_04
    │
    └── 🏭 Unit 2: "North Farm" (Location B)
        │
        ├── 🍓 Raspberry Pi Gateway
        │   ├── IP: 192.168.2.100
        │   └── Local MQTT Broker
        │
        ├── 🚪 Zone E: "Growing Room 1"
        │   └── 📱 ESP32_05
        │
        └── 🚪 Zone F: "Growing Room 2"
            └── 📱 ESP32_06
```

---

## 🔄 Communication Flow

### 1. **Local Network** (ESP32 → Raspberry Pi)

```
ESP32 Zone Controller
    │ WiFi: 192.168.1.x
    │ Protocol: MQTT (Local)
    │ Broker: 192.168.1.100:1883
    ▼
Raspberry Pi Gateway
    │ Aggregation & Processing
    │ Local Control Logic
    │ Data Buffering
    ▼
```

**Topics (Local)**:
```
unit1/zone_a/telemetry      → Sensor data
unit1/zone_a/status         → Device status
unit1/zone_a/setpoints      ← Control commands
unit1/zone_a/command        ← Actions
```

### 2. **Internet** (Raspberry Pi → Cloud)

```
Raspberry Pi Gateway
    │ Internet/WiFi
    │ Protocol: MQTT/TLS
    │ Broker: mqtt.cropwise.cloud:8883
    ▼
Cloud MQTT Broker (EMQX)
    │ Authentication & Routing
    ▼
CropWise Cloud Backend
    │ API, Database, Analytics
    ▼
Web Dashboard / Mobile App
```

**Topics (Cloud)**:
```
yfcloud/org_abc123/unit_001/telemetry_aggregated
yfcloud/org_abc123/unit_001/gateway_status
yfcloud/org_abc123/unit_001/setpoints_bulk
```

---

## 🧩 Components

### 1. **ESP32 Zone Controller**
- **Role**: Control one zone (room)
- **Connection**: WiFi → Local Raspberry Pi
- **Functions**:
  - Read sensors (Temp, RH, CO₂, Light)
  - Control relays (Fan, Humidifier, Heater, etc.)
  - Execute crop recipe setpoints
  - Publish telemetry every 60 seconds
  - Subscribe to local commands

### 2. **Raspberry Pi Gateway**
- **Role**: Local hub for one unit (building)
- **Connection**: WiFi → Internet → Cloud
- **Functions**:
  - Run local MQTT broker (Mosquitto)
  - Aggregate data from all ESP32s in unit
  - Run Node-RED for local automation
  - Buffer data during internet outages
  - Forward aggregated data to cloud
  - Manage OTA updates for ESP32s

### 3. **Cloud Backend**
- **Role**: Multi-tenant SaaS platform
- **Connection**: Public internet
- **Functions**:
  - User authentication & authorization
  - Crop recipe management
  - Data storage (InfluxDB + PostgreSQL)
  - Analytics & ML insights
  - Subscription & billing
  - Dashboard APIs

### 4. **Web Dashboard**
- **Role**: User interface
- **Tech**: React + Tailwind CSS + Vite
- **Features**:
  - Real-time monitoring
  - Recipe management
  - Zone control
  - Analytics & reports
  - Multi-unit management

---

## 💾 Database Models

### Core Hierarchy

```sql
-- Organizations (Tenants/Customers)
organizations (id, name, subscriptionTier, maxZones, maxUsers, ...)

-- Units (Buildings/Locations)
units (id, organizationId, name, gatewayId, gatewayStatus, totalZones, ...)

-- Zones (Rooms)
zones (id, organizationId, unitId, name, activeRecipeId, status, ...)

-- Devices (ESP32s, Raspberry Pis)
devices (id, organizationId, unitId, zoneId, deviceId, deviceType, status, ...)

-- Crop Recipes
crop_recipes (id, organizationId, name, crop, stages, ...)

-- Telemetry (Time-series)
telemetry (timestamp, organizationId, unitId, zoneId, temperature, humidity, co2, ...)
```

---

## 🔐 Security Architecture

### Layer 1: Local Network (ESP32 ↔ Pi)
- ✅ WPA2/WPA3 WiFi encryption
- ✅ MQTT username/password per device
- ✅ Local network isolation (no internet exposure)
- ✅ Optional VLAN for IoT devices

### Layer 2: Gateway (Raspberry Pi)
- ✅ Firewall (only outbound MQTT/HTTPS allowed)
- ✅ Encrypted local data buffer
- ✅ SSH key-based access only
- ✅ Automatic security updates
- ✅ Optional VPN for remote access

### Layer 3: Cloud (Pi ↔ Backend)
- ✅ TLS 1.3 for all MQTT traffic
- ✅ Client certificates per gateway
- ✅ JWT authentication for API
- ✅ Multi-tenant data isolation
- ✅ Encrypted data at rest (AES-256)

---

## 📡 API Endpoints

### Authentication
- `POST /api/auth/register` - Create account
- `POST /api/auth/login` - Login
- `POST /api/auth/refresh` - Refresh token

### Units (Buildings)
- `GET /api/units` - List all units
- `POST /api/units` - Create new unit
- `GET /api/units/:id` - Get unit details
- `PUT /api/units/:id` - Update unit
- `POST /api/units/:id/gateway-heartbeat` - Gateway status update

### Zones (Rooms)
- `GET /api/zones` - List zones
- `POST /api/zones` - Create zone
- `GET /api/zones/:id` - Get zone details
- `PUT /api/zones/:id/recipe` - Apply crop recipe
- `POST /api/zones/:id/start` - Start zone
- `POST /api/zones/:id/stop` - Stop zone

### Devices
- `GET /api/devices` - List devices
- `POST /api/devices` - Register device
- `GET /api/devices/:id` - Device details

### Crop Recipes
- `GET /api/crop-recipes` - List recipes
- `POST /api/crop-recipes` - Create recipe
- `GET /api/crop-recipes/:id` - Recipe details

### Telemetry
- `GET /api/telemetry` - Query telemetry data
- `GET /api/analytics/dashboard` - Dashboard stats

---

## 🚀 Deployment Options

### Option 1: Development (Local)
```bash
# Backend
cd backend && npm install && npm run dev

# Frontend
cd frontend && npm install && npm run dev

# Database
Uses SQLite by default (cropwise.db)
```

### Option 2: Production (Docker)
```bash
# Start all services
docker-compose up -d

# Services:
# - Backend: http://localhost:3000
# - Frontend: http://localhost:5173
# - PostgreSQL: localhost:5432
# - MQTT Broker: localhost:1883
```

### Option 3: Cloud (AWS/Azure/GCP)
- **Backend**: ECS/App Service/Cloud Run
- **Database**: RDS PostgreSQL + InfluxDB Cloud
- **MQTT**: EMQX Cloud or HiveMQ Cloud
- **Frontend**: S3 + CloudFront / Azure Static Web Apps / Firebase Hosting

---

## 📈 Advantages of This Architecture

| Feature | Benefit |
|:--------|:--------|
| **Resilience** | Local control works even without internet |
| **Cost** | 1 cloud connection per unit (not per zone) |
| **Speed** | Local commands execute in <50ms |
| **Security** | ESP32s never exposed to internet |
| **Scalability** | Add zones without increasing cloud load |
| **Bandwidth** | Aggregated data reduces cloud traffic |
| **Maintenance** | Centralized OTA updates from Pi |

---

## 🎯 Use Cases

### ✅ Mushroom Farms
- Precise temperature/humidity control
- CO₂ monitoring for fruiting
- Multi-zone incubation/fruiting
- Automated climate recipes

### ✅ Vertical Farms
- LED light control
- Nutrient dosing
- Climate control per rack
- Yield tracking

### ✅ Greenhouses
- Temperature/humidity management
- Ventilation control
- Irrigation automation
- Multi-zone crop management

### ✅ Hydroponics
- pH/EC monitoring
- Nutrient dosing
- Water temperature control
- Pump scheduling

---

## 📊 Subscription Tiers

| Tier | Price | Zones | Users | Features |
|:-----|:------|:------|:------|:---------|
| **Trial** | Free (14 days) | 1 | 2 | Basic monitoring |
| **Starter** | ₹1,500/month | 1 | 3 | Local control |
| **Growth** | ₹3,000/month | 5 | 10 | Cloud sync, Analytics |
| **Enterprise** | ₹6,000/month | 10 | 50 | AI insights, API, White-label |

---

## 🛠️ Tech Stack

### Backend
- **Framework**: Node.js + Express
- **Database**: PostgreSQL (metadata) + InfluxDB (telemetry)
- **ORM**: Sequelize
- **Authentication**: JWT + bcrypt
- **MQTT**: MQTT.js client
- **Cache**: Redis
- **Billing**: Razorpay

### Frontend
- **Framework**: React 18
- **Build**: Vite
- **Styling**: Tailwind CSS
- **State**: Zustand
- **Data Fetching**: TanStack Query
- **Charts**: Recharts
- **Forms**: React Hook Form

### Edge
- **Gateway**: Raspberry Pi 4B + Node.js
- **Local MQTT**: Mosquitto
- **Automation**: Node-RED
- **Controller**: ESP32 + Arduino/PlatformIO
- **Sensors**: SHT31, MH-Z19C, BH1750
- **Relays**: 4/8-channel relay modules

### Infrastructure
- **Containerization**: Docker + Docker Compose
- **Orchestration**: Kubernetes (optional)
- **CI/CD**: GitHub Actions
- **Monitoring**: Prometheus + Grafana
- **Logging**: ELK Stack

---

## 📝 Quick Start

### 1. Clone Repository
```bash
git clone https://github.com/yellowflowersorganics-star/cropwise.git
cd cropwise
```

### 2. Start Backend
```bash
cd backend
npm install
npm run dev
```

### 3. Start Frontend
```bash
cd frontend
npm install
npm run dev
```

### 4. Configure ESP32
```bash
cd edge/esp32
cp src/config.h.example src/config.h
# Edit config.h with your settings
pio run -t upload
```

### 5. Setup Raspberry Pi
```bash
# On Raspberry Pi
curl -sSL https://raw.githubusercontent.com/yellowflowers/cropwise-gateway/main/install.sh | bash
```

---

## 📚 Documentation

- [Getting Started Guide](./docs/GETTING_STARTED.md)
- [Edge Gateway Architecture](./docs/ARCHITECTURE_EDGE_GATEWAY.md)
- [API Documentation](./docs/API_UNITS.md)
- [Commercialization Plan](./docs/COMMERCIAL_PLATFORM.md)
- [Migration Roadmap](./docs/MIGRATION_PLAN.md)

---

## 🤝 Support

- **Email**: support@yellowflowers.tech
- **Documentation**: https://docs.yellowflowers.tech
- **GitHub**: https://github.com/yellowflowersorganics-star/cropwise

---

**Built with ❤️ for controlled-environment agriculture** 🌱🍄🥬

