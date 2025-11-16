# Getting Started with CropWise

Complete guide to setting up and running CropWise from scratch.

## 📋 Prerequisites

### Software Requirements
- **Node.js**: 18.0 or higher
- **npm**: 9.0 or higher
- **PostgreSQL**: 15 or higher
- **Redis**: 7 or higher
- **MQTT Broker**: Mosquitto 2.0 or higher
- **Docker** (optional but recommended)
- **PlatformIO** (for ESP32 development)

### Hardware Requirements (for complete setup)
- **ESP32 DevKit**: ESP-WROOM-32 module
- **Sensors**: SHT31, MH-Z19C, soil moisture, light sensor
- **Actuators**: Relay modules, fans, pumps, lights
- **Development Computer**: Windows/Mac/Linux with USB port

## 🚀 Quick Start with Docker

The fastest way to get started:

```bash
# Clone the repository
git clone https://github.com/yellowflowers/cropwise.git
cd cropwise

# Start all services
docker-compose up -d

# Check logs
docker-compose logs -f

# Access services:
# - Frontend: http://localhost:8080
# - Backend API: http://localhost:3000
# - MQTT Broker: mqtt://localhost:1883
# - Node-RED: http://localhost:1880
```

## 🔧 Manual Setup

### 1. Backend Setup

```bash
cd backend

# Install dependencies
npm install

# Configure environment
cp ../.env.example .env
# Edit .env with your database credentials

# Create database
createdb cropwise_db

# Run migrations
npm run migrate

# Seed sample data (optional)
npm run seed

# Start development server
npm run dev
```

Backend will be running at `http://localhost:3000`

### 2. Frontend Setup

```bash
cd frontend

# Install dependencies
npm install

# Create environment file
echo "VITE_API_BASE_URL=http://localhost:3000/api" > .env

# Start development server
npm run dev
```

Frontend will be running at `http://localhost:8080`

### 3. MQTT Broker Setup

```bash
# Install Mosquitto
# Ubuntu/Debian:
sudo apt-get install mosquitto mosquitto-clients

# macOS:
brew install mosquitto

# Windows:
# Download from https://mosquitto.org/download/

# Create password file
mosquitto_passwd -c password.txt cropwise
# Enter password when prompted

# Start broker
mosquitto -c mosquitto.conf
```

### 4. ESP32 Firmware Setup

```bash
cd edge/esp32

# Edit src/config.h with your WiFi and MQTT credentials

# Build and upload
pio run -t upload

# Monitor serial output
pio device monitor
```

## 👤 Creating Your First User

### Via Web UI
1. Navigate to `http://localhost:8080/register`
2. Fill in registration form
3. Click "Create Account"
4. You'll be automatically logged in

### Via API (curl)

```bash
curl -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "farmer@example.com",
    "password": "securepassword",
    "firstName": "John",
    "lastName": "Farmer",
    "organizationName": "Green Valley Farm"
  }'
```

## 🏗️ Creating Your First Farm

1. Log in to the dashboard
2. Navigate to **Farms** → **Add Farm**
3. Fill in farm details:
   - Name: "Demo Farm"
   - Location: Your location
   - Type: Indoor/Greenhouse/Outdoor
   - Total Area: 100 m²
4. Click **Create Farm**

## 📦 Creating Your First Zone

1. Go to **Zones** → **Add Zone**
2. Select your farm
3. Configure zone:
   - Name: "Zone A"
   - Area: 10 m²
4. Click **Create Zone**

## 🌱 Using a Crop Recipe

### Option 1: Use Existing Recipe

1. Navigate to **Crop Recipes** → **Browse Public Recipes**
2. Find "Oyster Mushroom" recipe
3. Click **Clone to My Recipes**
4. Go to **Zones** → Select your zone
5. Click **Assign Recipe**
6. Choose the mushroom recipe
7. Click **Start Batch**

### Option 2: Create Custom Recipe

1. Go to **Crop Recipes** → **Create New**
2. Fill in basic info:
   ```
   Crop ID: my-custom-crop-v1
   Crop Name: My Custom Crop
   Crop Type: vegetable
   ```
3. Define stages:
   ```
   Stage 1: Germination
   - Duration: 5 days
   - Temperature: 25°C
   - Humidity: 80%
   - Light Hours: 0
   
   Stage 2: Growth
   - Duration: 30 days
   - Temperature: 22°C
   - Humidity: 70%
   - Light Hours: 14
   ```
4. Save recipe
5. Assign to a zone

## 🔌 Connecting Your First Device

### 1. Provision ESP32

In `edge/esp32/src/config.h`:

```cpp
#define WIFI_SSID "YourWiFiName"
#define WIFI_PASSWORD "YourWiFiPassword"

#define MQTT_BROKER "localhost"  // or your server IP
#define MQTT_PORT 1883
#define MQTT_USERNAME "cropwise"
#define MQTT_PASSWORD "your_mqtt_password"
```

### 2. Upload Firmware

```bash
cd edge/esp32
pio run -t upload
```

### 3. Register Device in Dashboard

1. Go to **Devices** → **Register New Device**
2. Enter device ID (visible in serial monitor)
3. Assign to a zone
4. Save

### 4. Verify Connection

Check serial monitor:
```
WiFi connected
IP address: 192.168.1.100
MQTT broker connected
Telemetry published
```

## 📊 Monitoring Your Setup

### Real-time Dashboard

1. Navigate to **Dashboard**
2. View:
   - Active zones
   - Current environmental conditions
   - Recent alerts
   - System status

### Zone Detail View

1. Go to **Zones** → Select your zone
2. Monitor:
   - Current stage and progress
   - Temperature, humidity, CO₂ graphs
   - Actuator states
   - Historical data

### Device Management

1. Go to **Devices**
2. Check:
   - Device status (online/offline)
   - Last seen timestamp
   - Firmware version
   - Signal strength

## 🎯 Next Steps

1. **Explore Analytics**
   - View yield predictions
   - Compare different batches
   - Energy consumption reports

2. **Set Up Alerts**
   - Configure email/SMS notifications
   - Set custom thresholds
   - Create automation rules

3. **Integrate with Node-RED**
   - Build custom automation flows
   - Create advanced dashboards
   - Connect external services

4. **Scale Your Operation**
   - Add more zones
   - Create zone templates
   - Implement multi-farm management

5. **Optimize with ML**
   - Collect historical data
   - Train yield prediction models
   - Fine-tune recipes based on results

## 🆘 Troubleshooting

### Backend won't start

```bash
# Check PostgreSQL is running
sudo service postgresql status

# Check Redis is running
redis-cli ping

# Check logs
npm run dev
```

### ESP32 won't connect

1. Verify WiFi credentials
2. Check MQTT broker is accessible
3. Ensure firewall allows port 1883
4. Check serial monitor for error messages

### Data not showing in dashboard

1. Verify device is publishing telemetry
2. Check MQTT broker logs
3. Ensure backend MQTT service is running
4. Check browser console for errors

## 📚 Further Reading

- [Architecture Overview](ARCHITECTURE.md)
- [API Documentation](API.md)
- [ESP32 Development Guide](../edge/esp32/README.md)
- [Recipe Development Guide](RECIPE_DEVELOPMENT.md)
- [Deployment Guide](DEPLOYMENT.md)

## 💬 Community & Support

- **Documentation**: https://docs.cropwise.io
- **GitHub Issues**: https://github.com/yellowflowers/cropwise/issues
- **Discord**: https://discord.gg/cropwise
- **Email**: support@yellowflowers.tech

---

**Welcome to CropWise! 🌱 Happy growing!**

