# 🚀 CropWise - Quick Start Guide

**Status**: ✅ Setup Complete | Dependencies Installed | Ready to Run

---

## 🎯 Choose Your Path

### Path 1: Docker (Fastest - Recommended)
```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

**Access**:
- Frontend: http://localhost:8080
- Backend API: http://localhost:3000
- MQTT Broker: mqtt://localhost:1883
- Node-RED: http://localhost:1880

### Path 2: Local Development
```bash
# Terminal 1 - Backend
cd backend
npm run dev

# Terminal 2 - Frontend
cd frontend
npm run dev
```

**Access**:
- Frontend: http://localhost:8080
- Backend API: http://localhost:3000

**Note**: You'll need PostgreSQL, Redis, and MQTT broker running locally.

### Path 3: ESP32 Firmware Only
```bash
cd edge/esp32

# Edit configuration
# Edit src/config.h with your WiFi and MQTT credentials

# Upload firmware
pio run -t upload

# Monitor serial output
pio device monitor
```

---

## 📝 First-Time Setup Checklist

### ✅ Already Done
- [x] Project structure created
- [x] Backend dependencies installed (715 packages)
- [x] Frontend dependencies installed (462 packages)
- [x] All configuration files created
- [x] Documentation written

### 🔄 Next Steps (Do These)

#### 1. Configure Environment
```bash
# Backend needs database configuration
# The .env file is blocked for security, so configure manually:

# Option A: Use Docker (auto-configured)
docker-compose up -d

# Option B: Create .env manually with these settings:
# backend/.env:
NODE_ENV=development
PORT=3000
DB_HOST=localhost
DB_PORT=5432
DB_NAME=cropwise_db
DB_USER=cropwise_user
DB_PASSWORD=your_password
MQTT_BROKER_URL=mqtt://localhost:1883
REDIS_HOST=localhost
JWT_SECRET=your_jwt_secret_change_this
```

#### 2. Start Development Servers

**Option A: Using Docker** (Easiest)
```bash
docker-compose up -d
```
✅ This starts everything: PostgreSQL, Redis, MQTT, Backend, Frontend

**Option B: Manual Start**
```bash
# Ensure PostgreSQL, Redis, and MQTT are running

# Terminal 1: Backend
cd backend
npm run dev

# Terminal 2: Frontend  
cd frontend
npm run dev
```

#### 3. Create Your First User
1. Open http://localhost:8080/register
2. Fill in the form:
   - First Name: John
   - Last Name: Doe
   - Email: john@example.com
   - Password: password123
   - Organization: Demo Farm
3. Click "Create Account"
4. You'll be logged in automatically

#### 4. Create Your First Farm
1. Navigate to **Farms** in the sidebar
2. Click **Add Farm**
3. Fill in:
   - Name: Demo Farm
   - Location: Your location
   - Farm Type: Indoor
   - Total Area: 100 m²
4. Click **Create**

#### 5. Create a Zone
1. Navigate to **Zones**
2. Click **Add Zone**
3. Fill in:
   - Name: Zone A
   - Farm: Demo Farm
   - Area: 10 m²
4. Click **Create**

#### 6. Use a Crop Recipe
1. Navigate to **Crop Recipes**
2. Browse available recipes
3. Select "Oyster Mushroom" recipe
4. Clone it to your recipes
5. Go back to **Zones** → Zone A
6. Click **Assign Recipe**
7. Select the mushroom recipe
8. Click **Start Batch**

---

## 🔌 Connecting ESP32 (Hardware Setup)

### Hardware You Need
- ESP32 DevKit (ESP-WROOM-32)
- SHT31 sensor (Temperature & Humidity)
- Breadboard and jumper wires
- USB cable
- Optional: CO₂ sensor, relays, actuators

### Wiring
```
ESP32          SHT31
GPIO 21   →    SDA
GPIO 22   →    SCL
3.3V      →    VCC
GND       →    GND
```

### Flash Firmware
```bash
cd edge/esp32

# Edit WiFi credentials in src/config.h
# Edit MQTT broker address

# Install PlatformIO if not installed
pip install platformio

# Build and upload
pio run -t upload

# Monitor serial output
pio device monitor
```

### Register Device
1. Copy Device ID from serial monitor (e.g., ESP32_ABCD1234)
2. Go to **Devices** in dashboard
3. Click **Register Device**
4. Enter device ID
5. Assign to Zone A
6. Save

---

## 📊 Verify Everything Works

### Check Backend
```bash
curl http://localhost:3000/api

# Should return:
# {
#   "name": "CropWise API",
#   "version": "1.0.0",
#   "status": "operational"
# }
```

### Check Frontend
Open http://localhost:8080 - should see login page

### Check MQTT
```bash
# Subscribe to telemetry topic
mosquitto_sub -h localhost -t "cropwise/+/telemetry" -u cropwise -P your_password

# You should see device telemetry messages
```

### Check ESP32
Serial monitor should show:
```
WiFi connected
IP address: 192.168.1.XXX
MQTT broker connected
Telemetry published
Temp: 24.5°C, RH: 72.3%, CO2: 800ppm
```

---

## 🎓 Learn the System

### Key Concepts

**1. Crop Recipe**
- JSON file defining environmental parameters per growth stage
- Example: Mushroom has 4 stages (Spawn Run, Pinning, Fruiting, Harvest)
- Each stage has specific temperature, humidity, CO₂, light settings

**2. Zone**
- Physical growing area in your farm
- Can run one crop recipe at a time
- Tracks batch start date and current stage

**3. Recipe Execution**
- Backend calculates current stage based on start date
- Sends setpoints to ESP32 via MQTT
- ESP32 controls actuators to maintain setpoints
- Telemetry flows back to cloud

**4. Stage Transitions**
- Automatic based on elapsed days
- No manual intervention needed
- System updates setpoints when stage changes

### Workflow
```
1. Create Farm → 2. Add Zone → 3. Assign Recipe → 4. Start Batch
                                                            ↓
                                                    5. Connect Device
                                                            ↓
                                                    6. Monitor & Harvest
```

---

## 🐛 Troubleshooting

### Backend Won't Start
```bash
# Check if PostgreSQL is running
# Docker: docker-compose ps
# Manual: sudo service postgresql status

# Check if port 3000 is available
# Windows: netstat -ano | findstr :3000
# Linux/Mac: lsof -i :3000

# Check logs
cd backend
npm run dev
# Look for error messages
```

### Frontend Won't Start
```bash
# Check if port 8080 is available
# Windows: netstat -ano | findstr :8080

# Rebuild
cd frontend
rm -rf node_modules
npm install
npm run dev
```

### ESP32 Won't Connect
1. Check WiFi credentials in `src/config.h`
2. Check MQTT broker IP address
3. Verify firewall allows port 1883
4. Check serial monitor for error messages
5. Try restarting ESP32

### Database Connection Error
```bash
# Using Docker: Database auto-configured
docker-compose up -d postgres

# Manual: Create database
createdb cropwise_db

# Check credentials in backend/.env
DB_HOST=localhost
DB_NAME=cropwise_db
DB_USER=cropwise_user
DB_PASSWORD=your_password
```

---

## 📚 Next Steps After Setup

### For Developers
1. **Implement Stub Controllers**
   - Zone management
   - Device management
   - Telemetry handling
   - Analytics

2. **Build UI Components**
   - Zone detail page
   - Recipe editor
   - Real-time charts
   - Device management UI

3. **Add Tests**
   - Unit tests for recipe engine
   - Integration tests for APIs
   - E2E tests for UI

### For Users
1. **Test Different Recipes**
   - Try mushroom recipe
   - Test lettuce recipe
   - Create custom recipe

2. **Monitor Your Crops**
   - Watch telemetry data
   - Track stage progression
   - Compare batches

3. **Optimize**
   - Adjust recipe parameters
   - Fine-tune based on results
   - Share successful recipes

---

## 🆘 Get Help

- **Documentation**: See `docs/GETTING_STARTED.md` for detailed guide
- **Test Report**: See `TEST_REPORT.md` for setup verification
- **Project Summary**: See `PROJECT_SUMMARY.md` for architecture overview
- **Issues**: Create issue on GitHub
- **Questions**: Check documentation or ask in Discord

---

## ✅ Setup Status

| Component | Status | Action |
|:----------|:-------|:-------|
| Project Files | ✅ Complete | Ready |
| Backend Deps | ✅ Installed | Ready |
| Frontend Deps | ✅ Installed | Ready |
| Configuration | ⚠️ Needs .env | Configure before running |
| Database | ⏳ Not started | Start PostgreSQL or use Docker |
| Services | ⏳ Not started | Run `docker-compose up` |

**Next Command to Run**:
```bash
docker-compose up -d
```

---

**🌱 You're ready to grow with CropWise!**

