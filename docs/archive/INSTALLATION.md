# 🚀 CropWise - Installation Guide

Complete installation instructions for all platforms.

---

## 📋 Prerequisites

Before you begin, ensure you have:

- **Node.js 18+** ([Download](https://nodejs.org/))
- **Git** ([Download](https://git-scm.com/))
- **Database**: PostgreSQL 15+ or SQLite 3+
- **Redis 7+** (optional, recommended for production)
- **Docker** (optional, for containerized setup)

---

## 🎯 Quick Start (Recommended)

### Option 1: Docker Compose (Fastest)

```bash
# 1. Clone repository
git clone https://github.com/yellowflowersorganics-star/cropwise.git
cd cropwise

# 2. Start all services
docker-compose up -d

# 3. Access application
# Frontend: http://localhost:8080
# Backend API: http://localhost:3000
# API Docs: http://localhost:8080/api-docs
```

**That's it!** ✅ All services are running.

---

## 💻 Manual Installation

### Step 1: Clone Repository

```bash
git clone https://github.com/yellowflowersorganics-star/cropwise.git
cd cropwise
```

### Step 2: Backend Setup

```bash
# Navigate to backend
cd backend

# Install dependencies
npm install

# Copy environment file
cp .env.example .env

# Edit configuration (important!)
nano .env
```

**Configure `.env` file**:
```bash
# Database (use SQLite for quick start)
DB_DIALECT=sqlite
DB_STORAGE=./cropwise.db

# Or PostgreSQL for production
# DB_DIALECT=postgres
# DB_HOST=localhost
# DB_PORT=5432
# DB_NAME=cropwise_db
# DB_USER=cropwise_admin
# DB_PASSWORD=your_password

# JWT Secret (generate a random string)
JWT_SECRET=your_super_secret_key_minimum_32_characters_long

# Google OAuth (optional, for Google login)
GOOGLE_CLIENT_ID=your_google_client_id.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your_google_client_secret
GOOGLE_CALLBACK_URL=http://localhost:3000/api/auth/google/callback

# Frontend URL
FRONTEND_URL=http://localhost:8080
```

**Generate JWT Secret**:
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

**Run database migrations**:
```bash
npm run migrate
```

**Start backend**:
```bash
# Development mode (with auto-reload)
npm run dev

# Production mode
npm start
```

**Verify backend is running**:
```bash
curl http://localhost:3000/health
# Expected: {"status":"healthy",...}
```

### Step 3: Frontend Setup

Open a **new terminal**:

```bash
# Navigate to frontend
cd frontend

# Install dependencies
npm install

# Copy environment file
cp .env.example .env

# Edit configuration
nano .env
```

**Configure `.env` file**:
```bash
# Backend API URL
VITE_API_URL=http://localhost:3000
```

**Start frontend**:
```bash
# Development mode
npm run dev

# Production build
npm run build
npm run preview
```

**Access application**:
- Open browser: **http://localhost:8080**
- Or if using Vite dev server: **http://localhost:5173**

---

## 🐘 PostgreSQL Setup (Production)

### Install PostgreSQL

**Ubuntu/Debian**:
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

**macOS**:
```bash
brew install postgresql@15
brew services start postgresql@15
```

**Windows**:
- Download installer from [postgresql.org](https://www.postgresql.org/download/windows/)

### Create Database

```bash
# Switch to postgres user
sudo -i -u postgres

# Open psql
psql

# Create database
CREATE DATABASE cropwise_db;

# Create user
CREATE USER cropwise_admin WITH PASSWORD 'your_secure_password';

# Grant privileges
GRANT ALL PRIVILEGES ON DATABASE cropwise_db TO cropwise_admin;

# Exit
\q
exit
```

### Update Backend Configuration

```bash
# Edit backend/.env
DB_DIALECT=postgres
DB_HOST=localhost
DB_PORT=5432
DB_NAME=cropwise_db
DB_USER=cropwise_admin
DB_PASSWORD=your_secure_password
```

### Run Migrations

```bash
cd backend
npm run migrate
```

---

## 🔴 Redis Setup (Optional)

### Install Redis

**Ubuntu/Debian**:
```bash
sudo apt install redis-server
sudo systemctl start redis
sudo systemctl enable redis
```

**macOS**:
```bash
brew install redis
brew services start redis
```

**Windows**:
- Use Docker or WSL2

### Test Redis

```bash
redis-cli ping
# Expected: PONG
```

### Update Backend Configuration

```bash
# Edit backend/.env
REDIS_HOST=localhost
REDIS_PORT=6379
```

---

## 🦟 MQTT Broker Setup (For IoT)

### Install Mosquitto

**Ubuntu/Debian**:
```bash
sudo apt install mosquitto mosquitto-clients
sudo systemctl start mosquitto
sudo systemctl enable mosquitto
```

**macOS**:
```bash
brew install mosquitto
brew services start mosquitto
```

### Configure Mosquitto

```bash
# Create config file
sudo nano /etc/mosquitto/mosquitto.conf
```

**Add**:
```
listener 1883
allow_anonymous true
```

**Restart**:
```bash
sudo systemctl restart mosquitto
```

### Test MQTT

```bash
# Subscribe to test topic
mosquitto_sub -h localhost -t 'test' &

# Publish message
mosquitto_pub -h localhost -t 'test' -m 'Hello MQTT'
```

### Update Backend Configuration

```bash
# Edit backend/.env
MQTT_BROKER_URL=mqtt://localhost:1883
```

---

## 🌐 Google OAuth Setup (Optional)

### Create Google OAuth Credentials

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create new project or select existing
3. Enable **Google+ API**
4. Go to **Credentials** → **Create Credentials** → **OAuth 2.0 Client ID**
5. Application type: **Web application**
6. Authorized redirect URIs:
   - `http://localhost:3000/api/auth/google/callback` (development)
   - `https://yourdomain.com/api/auth/google/callback` (production)
7. Copy **Client ID** and **Client Secret**

### Update Backend Configuration

```bash
# Edit backend/.env
GOOGLE_CLIENT_ID=your_google_client_id.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your_google_client_secret
GOOGLE_CALLBACK_URL=http://localhost:3000/api/auth/google/callback
```

---

## 🐳 Docker Installation (Alternative)

### Prerequisites

- Docker 20+
- Docker Compose 2+

### Setup

```bash
# Clone repository
git clone https://github.com/yellowflowersorganics-star/cropwise.git
cd cropwise

# Create environment files
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env

# Edit configurations
nano backend/.env
nano frontend/.env

# Start services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

### Services Started

- **Backend**: http://localhost:3000
- **Frontend**: http://localhost:8080
- **PostgreSQL**: localhost:5432
- **Redis**: localhost:6379
- **Mosquitto**: localhost:1883

---

## 🔧 Development Setup

### VS Code Extensions (Recommended)

- ESLint
- Prettier
- Docker
- GitLens
- Thunder Client (API testing)
- SQLite Viewer

### Configure Git

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

### Install Development Tools

```bash
# Nodemon (auto-reload)
npm install -g nodemon

# PM2 (process manager)
npm install -g pm2

# Sequelize CLI (database migrations)
npm install -g sequelize-cli
```

---

## 🧪 Verify Installation

### Backend Health Check

```bash
curl http://localhost:3000/health
```

**Expected Response**:
```json
{
  "status": "healthy",
  "timestamp": "2024-11-14T10:30:00.000Z",
  "uptime": 123.45,
  "environment": "development"
}
```

### Create Test User

```bash
curl -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "TestPassword123",
    "firstName": "Test",
    "lastName": "User",
    "organizationName": "Test Farm"
  }'
```

### Login

1. Open http://localhost:8080
2. Click "Sign In"
3. Enter credentials:
   - Email: test@example.com
   - Password: TestPassword123
4. You should see the dashboard

---

## 🚀 Production Deployment

For production deployment, see:
- [AWS Deployment Guide](DEPLOY_TO_AWS_NOW.md)
- [Administrator Guide](ADMIN_GUIDE.md)

---

## 🔄 Updating

### Pull Latest Changes

```bash
# Stop services
pm2 stop cropwise-backend

# Pull updates
git pull origin main

# Update dependencies
cd backend && npm install
cd ../frontend && npm install

# Run migrations
cd backend
npm run migrate

# Rebuild frontend
cd ../frontend
npm run build

# Restart services
pm2 restart cropwise-backend
```

---

## 🗑️ Uninstallation

### Docker

```bash
# Stop and remove containers
docker-compose down -v

# Remove images
docker rmi cropwise-backend cropwise-frontend
```

### Manual

```bash
# Stop services
pm2 delete cropwise-backend

# Remove PostgreSQL database
sudo -u postgres psql
DROP DATABASE cropwise_db;
DROP USER cropwise_admin;
\q

# Remove application files
rm -rf /path/to/cropwise

# Remove Node modules (frees space)
rm -rf ~/.npm
```

---

## ❓ Troubleshooting

**Issue**: Port 3000 already in use

**Solution**:
```bash
# Find and kill process
sudo lsof -i :3000
sudo kill -9 <PID>

# Or change port in backend/.env
PORT=3001
```

**Issue**: Database connection failed

**Solution**:
```bash
# Test PostgreSQL connection
psql -h localhost -U cropwise_admin -d cropwise_db

# Check PostgreSQL is running
sudo systemctl status postgresql

# Restart PostgreSQL
sudo systemctl restart postgresql
```

**Issue**: npm install fails

**Solution**:
```bash
# Clear npm cache
npm cache clean --force

# Delete node_modules
rm -rf node_modules package-lock.json

# Reinstall
npm install
```

For more troubleshooting, see [Troubleshooting Guide](TROUBLESHOOTING.md).

---

## 📚 Next Steps

After installation:

1. **Read [User Guide](USER_GUIDE.md)** - Learn how to use the system
2. **Set up IoT devices** - See [IoT Integration Guide](IOT_INTEGRATION_GUIDE.md)
3. **Configure notifications** - Enable email/SMS alerts
4. **Add team members** - Invite employees to collaborate
5. **Create your first farm** - Start managing your operation

---

## 🆘 Need Help?

- **Documentation**: [docs.cropwise.io](https://docs.cropwise.io)
- **Support**: support@cropwise.io
- **Community**: [community.cropwise.io](https://community.cropwise.io)
- **Issues**: [GitHub Issues](https://github.com/yellowflowersorganics-star/cropwise/issues)

---

**Happy Farming! 🌱**

