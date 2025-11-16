# 💻 Step 2: Local Development Environment Setup

Complete guide to set up CropWise for local development.

**Prerequisites**: [Step 1 (GitHub Setup)](01-github-setup.md) completed

**Time Required**: 30-45 minutes

---

## 📋 What You'll Accomplish

By the end of this guide, you'll have:
- ✅ Cloned repository locally
- ✅ All dependencies installed
- ✅ Local PostgreSQL database running
- ✅ Backend API running on http://localhost:3000
- ✅ Frontend app running on http://localhost:5173
- ✅ Environment variables configured
- ✅ Docker and Docker Compose ready

---

## 🛠️ Step-by-Step Instructions

### 1. Install Prerequisites

#### **1.1 Install Node.js**

**Windows**:
```powershell
# Using Chocolatey
choco install nodejs-lts -y

# Or download from https://nodejs.org (LTS version)
```

**macOS**:
```bash
# Using Homebrew
brew install node@18
```

**Linux** (Ubuntu/Debian):
```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
```

**Verify installation**:
```bash
node --version  # Should be 18.x or higher
npm --version   # Should be 9.x or higher
```

---

#### **1.2 Install Git**

**Windows**:
```powershell
choco install git -y
```

**macOS**:
```bash
brew install git
```

**Linux**:
```bash
sudo apt-get install git -y
```

**Verify**:
```bash
git --version
```

---

#### **1.3 Install Docker Desktop**

**Download**: https://www.docker.com/products/docker-desktop

**Verify**:
```bash
docker --version
docker-compose --version
```

---

#### **1.4 Install PostgreSQL** (Option 1: Native)

**Windows**:
```powershell
choco install postgresql15 -y
```

**macOS**:
```bash
brew install postgresql@15
brew services start postgresql@15
```

**Linux**:
```bash
sudo apt-get install postgresql-15 postgresql-contrib -y
sudo systemctl start postgresql
```

**Or use Docker** (Option 2: Recommended for beginners):
```bash
# PostgreSQL will run via docker-compose (see step 4)
```

---

### 2. Clone the Repository

```bash
# Clone the repository
git clone https://github.com/yellowflowersorganics-star/cropwise.git
cd cropwise

# Verify you're on the develop branch
git checkout develop
git pull origin develop
```

---

### 3. Set Up Environment Variables

#### **3.1 Backend Environment**

```bash
cd backend
cp .env.example .env
```

Edit `.env` with your local values:

```bash
# Database
DATABASE_URL=postgresql://cropwise_user:cropwise_pass@localhost:5432/cropwise_dev

# Server
NODE_ENV=development
PORT=3000
FRONTEND_URL=http://localhost:5173

# JWT Secret (generate a random string)
JWT_SECRET=your_super_secret_key_minimum_32_characters_long

# Google OAuth (get from Google Cloud Console - see Step 6)
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
GOOGLE_CALLBACK_URL=http://localhost:3000/api/auth/google/callback

# Redis (optional for local, required for production)
REDIS_URL=redis://localhost:6379

# MQTT (for IoT features)
MQTT_BROKER_URL=mqtt://localhost:1883
MQTT_USERNAME=cropwise
MQTT_PASSWORD=cropwise_mqtt_pass

# AWS (optional for local development)
AWS_REGION=ap-south-1
AWS_ACCESS_KEY_ID=your_aws_key
AWS_SECRET_ACCESS_KEY=your_aws_secret
S3_BUCKET_NAME=cropwise-dev-uploads

# Twilio (optional)
TWILIO_ACCOUNT_SID=your_twilio_sid
TWILIO_AUTH_TOKEN=your_twilio_token
TWILIO_PHONE_NUMBER=+1234567890

# Slack (optional)
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/YOUR/WEBHOOK/URL
```

#### **3.2 Frontend Environment**

```bash
cd ../frontend
cp .env.example .env
```

Edit `.env`:

```bash
VITE_API_URL=http://localhost:3000
VITE_WS_URL=ws://localhost:3000
VITE_GOOGLE_CLIENT_ID=your_google_client_id
```

---

### 4. Start Services with Docker Compose

**Option 1: Full Stack (Recommended)**

```bash
# From project root
docker-compose up -d

# This starts:
# - PostgreSQL (port 5432)
# - Redis (port 6379)
# - Mosquitto MQTT (port 1883)
# - pgAdmin (port 5050) - Optional DB GUI
```

**Verify services are running**:
```bash
docker-compose ps
```

You should see:
- `postgres` - Up
- `redis` - Up
- `mosquitto` - Up

**Access pgAdmin** (Database GUI):
- URL: http://localhost:5050
- Email: admin@cropwise.com
- Password: admin

---

### 5. Initialize the Database

```bash
cd backend

# Install dependencies
npm install

# Run database migrations
npm run migrate

# Seed database with test data (optional)
npm run seed
```

**Verify database**:
```bash
# Using psql
psql -h localhost -U cropwise_user -d cropwise_dev

# Or use pgAdmin at http://localhost:5050
```

---

### 6. Start Backend Server

```bash
cd backend

# Development mode (with hot reload)
npm run dev

# The backend should start at http://localhost:3000
```

**Test backend**:
```bash
# Health check
curl http://localhost:3000/health

# Should return: {"status":"ok","timestamp":"..."}
```

---

### 7. Start Frontend Server

Open a **new terminal**:

```bash
cd frontend

# Install dependencies
npm install

# Start development server
npm run dev

# The frontend should start at http://localhost:5173
```

**Access the app**:
- Open browser to http://localhost:5173
- You should see the CropWise login page

---

### 8. Verify Local Setup

#### **✅ Backend Checks**

```bash
# 1. Health endpoint
curl http://localhost:3000/health

# 2. API docs
curl http://localhost:3000/api-docs

# 3. Database connection
npm run db:check
```

#### **✅ Frontend Checks**

1. Open http://localhost:5173 in browser
2. Check browser console for errors
3. Try to navigate between pages

#### **✅ Database Checks**

```bash
# Connect to database
psql -h localhost -U cropwise_user -d cropwise_dev

# Run queries
\dt  # List tables
SELECT * FROM users LIMIT 5;
\q   # Exit
```

---

## 🎯 Quick Start Commands

Once everything is set up, use these commands daily:

```bash
# Start all services
docker-compose up -d

# Start backend (Terminal 1)
cd backend && npm run dev

# Start frontend (Terminal 2)
cd frontend && npm run dev

# Stop all services
docker-compose down
```

---

## 🛑 Troubleshooting

### **Problem: "Port 3000 already in use"**

```bash
# Windows
netstat -ano | findstr :3000
taskkill /PID <PID> /F

# macOS/Linux
lsof -ti:3000 | xargs kill -9
```

### **Problem: "Cannot connect to PostgreSQL"**

```bash
# Check if PostgreSQL is running
docker-compose ps

# Restart PostgreSQL
docker-compose restart postgres

# Check logs
docker-compose logs postgres
```

### **Problem: "npm install fails"**

```bash
# Clear npm cache
npm cache clean --force

# Delete node_modules and reinstall
rm -rf node_modules package-lock.json
npm install
```

### **Problem: "Database migrations fail"**

```bash
# Reset database
npm run db:reset

# Or manually drop and recreate
psql -h localhost -U postgres
DROP DATABASE cropwise_dev;
CREATE DATABASE cropwise_dev;
\q

# Run migrations again
npm run migrate
```

---

## 📝 Additional Configuration

### **VS Code Extensions** (Recommended)

Install these for better development experience:
- ESLint
- Prettier
- Docker
- PostgreSQL
- GitLens
- Thunder Client (API testing)

### **VS Code Settings**

Create `.vscode/settings.json`:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "files.exclude": {
    "**/node_modules": true,
    "**/.git": true,
    "**/dist": true,
    "**/build": true
  }
}
```

---

## 🔐 Security Best Practices

### **Never commit these files:**
- `.env`
- `.env.local`
- `*.log`
- `node_modules/`
- `dist/` or `build/`
- AWS credentials
- API keys

### **Generate strong secrets:**

```bash
# Generate JWT secret
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"

# Generate random password
node -e "console.log(require('crypto').randomBytes(16).toString('base64'))"
```

---

## ✅ Verification Checklist

Before moving to the next step, ensure:

- [ ] Node.js 18+ installed
- [ ] Git installed and configured
- [ ] Docker Desktop running
- [ ] PostgreSQL accessible (via Docker or native)
- [ ] Backend running at http://localhost:3000
- [ ] Frontend running at http://localhost:5173
- [ ] Database migrations completed
- [ ] Health check endpoint returns OK
- [ ] No errors in terminal/browser console
- [ ] Can navigate the frontend app

---

## 🎉 What's Next?

Your local development environment is ready!

**Next Step**: [Step 3: AWS Account Setup](03-aws-account-setup.md)

---

## 📚 Related Documentation

- [Developer Guide](../development/DEVELOPER_GUIDE.md) - Full development workflow
- [Git Workflow](../development/GIT_WORKFLOW.md) - Branching and PR process
- [Troubleshooting](../deployment/TROUBLESHOOTING.md) - Common issues
- [Contributing](../development/CONTRIBUTING.md) - How to contribute

---

**Last Updated**: November 16, 2025

