# 👨‍💻 Developer Guide

**Complete guide for CropWise developers**

---

## 📑 Table of Contents

- [Getting Started](#-getting-started)
- [Development Environment Setup](#-development-environment-setup)
- [Architecture Overview](#-architecture-overview)
- [Coding Standards](#-coding-standards)
- [Git Workflow](#-git-workflow)
- [Testing Guidelines](#-testing-guidelines)
- [API Development](#-api-development)
- [Frontend Development](#-frontend-development)
- [Database Migrations](#-database-migrations)
- [Debugging](#-debugging)
- [Performance Optimization](#-performance-optimization)
- [Security Best Practices](#-security-best-practices)
- [Code Review Guidelines](#-code-review-guidelines)
- [Troubleshooting](#-troubleshooting)

---

## 🚀 Getting Started

### Prerequisites

**Required:**
- Node.js 18+ and npm 9+
- Git 2.30+
- PostgreSQL 15+ (or SQLite for development)
- Docker & Docker Compose (optional but recommended)
- Code editor (VS Code recommended)

**Recommended:**
- Redis 7+
- Postman or Insomnia (API testing)
- PgAdmin or DBeaver (database management)
- Git GUI client (GitKraken, Sourcetree)

### First Time Setup

```bash
# 1. Clone the repository
git clone https://github.com/your-org/cropwise.git
cd cropwise

# 2. Install backend dependencies
cd backend
npm install

# 3. Setup environment variables
cp .env.example .env
# Edit .env with your configurations

# 4. Run database migrations
npm run migrate

# 5. Seed database (optional)
npm run seed

# 6. Start backend server
npm run dev

# 7. Install frontend dependencies (new terminal)
cd ../frontend
npm install

# 8. Setup frontend environment
cp .env.example .env

# 9. Start frontend dev server
npm run dev

# 10. Access application
# Frontend: http://localhost:5173
# Backend API: http://localhost:8080
# API Docs: http://localhost:8080/api-docs
```

### VS Code Extensions (Recommended)

```json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-typescript-next",
    "bradlc.vscode-tailwindcss",
    "dsznajder.es7-react-js-snippets",
    "formulahendry.auto-rename-tag",
    "christian-kohler.path-intellisense",
    "ms-azuretools.vscode-docker",
    "eamodio.gitlens",
    "github.vscode-pull-request-github"
  ]
}
```

Save this in `.vscode/extensions.json`

---

## 🛠️ Development Environment Setup

### Database Setup

#### Option 1: SQLite (Quick Development)

```bash
# Already configured in .env
DB_DIALECT=sqlite
DB_STORAGE=./database.sqlite

# Migrations run automatically
npm run migrate
```

#### Option 2: PostgreSQL (Production-like)

```bash
# Install PostgreSQL 15
# macOS: brew install postgresql@15
# Ubuntu: sudo apt install postgresql-15
# Windows: Download from postgresql.org

# Start PostgreSQL
# macOS: brew services start postgresql@15
# Ubuntu: sudo systemctl start postgresql
# Windows: Start via Services

# Create database
psql -U postgres
CREATE DATABASE cropwise_dev;
CREATE USER cropwise WITH PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE cropwise_dev TO cropwise;
\q

# Update .env
DB_DIALECT=postgres
DB_HOST=localhost
DB_PORT=5432
DB_NAME=cropwise_dev
DB_USER=cropwise
DB_PASSWORD=secure_password

# Run migrations
npm run migrate
```

#### Option 3: Docker Compose (Easiest)

```bash
# Start all services (PostgreSQL, Redis, MQTT)
docker-compose up -d postgres redis mqtt

# Backend connects automatically
npm run dev
```

### Redis Setup (Optional - for caching)

```bash
# macOS: brew install redis && brew services start redis
# Ubuntu: sudo apt install redis-server && sudo systemctl start redis
# Windows: Use Docker

# Test connection
redis-cli ping
# Should return: PONG

# Update .env
REDIS_ENABLED=true
REDIS_HOST=localhost
REDIS_PORT=6379
```

### MQTT Broker Setup (Optional - for IoT)

```bash
# Using Docker
docker run -d -p 1883:1883 -p 9001:9001 eclipse-mosquitto

# Or install locally
# macOS: brew install mosquitto
# Ubuntu: sudo apt install mosquitto

# Update .env
MQTT_ENABLED=true
MQTT_BROKER_URL=mqtt://localhost:1883
```

### Environment Variables Reference

**Backend (.env)**

```bash
# Server
NODE_ENV=development
PORT=8080
API_URL=http://localhost:8080

# Database
DB_DIALECT=sqlite                    # or postgres
DB_HOST=localhost                    # for postgres
DB_PORT=5432                         # for postgres
DB_NAME=cropwise_dev                # for postgres
DB_USER=cropwise                    # for postgres
DB_PASSWORD=secure_password          # for postgres
DB_STORAGE=./database.sqlite         # for sqlite
DB_LOGGING=true

# Authentication
JWT_SECRET=your_super_secret_key_change_in_production
JWT_EXPIRES_IN=7d
GOOGLE_CLIENT_ID=your_google_oauth_client_id
GOOGLE_CLIENT_SECRET=your_google_oauth_secret

# Redis (optional)
REDIS_ENABLED=false
REDIS_HOST=localhost
REDIS_PORT=6379

# MQTT (optional)
MQTT_ENABLED=false
MQTT_BROKER_URL=mqtt://localhost:1883
MQTT_CLIENT_ID=cropwise_backend

# Twilio (SMS/WhatsApp - optional)
TWILIO_ENABLED=false
TWILIO_ACCOUNT_SID=your_account_sid
TWILIO_AUTH_TOKEN=your_auth_token
TWILIO_PHONE_NUMBER=+1234567890

# Email (optional)
EMAIL_ENABLED=false
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@gmail.com
SMTP_PASSWORD=your_app_password

# Logging
LOG_LEVEL=debug                      # debug, info, warn, error
LOG_TO_FILE=true
```

**Frontend (.env)**

```bash
# API Configuration
VITE_API_URL=http://localhost:8080
VITE_API_TIMEOUT=30000

# Google OAuth
VITE_GOOGLE_CLIENT_ID=your_google_oauth_client_id

# Feature Flags
VITE_ENABLE_IOT=true
VITE_ENABLE_NOTIFICATIONS=true
VITE_ENABLE_ANALYTICS=true

# Environment
VITE_ENV=development
```

---

## 🏗️ Architecture Overview

### System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Frontend Layer                        │
│  React 18 + Vite + Tailwind CSS + Zustand + React Router   │
└────────────────────────┬────────────────────────────────────┘
                         │ REST API (Axios)
                         │
┌────────────────────────▼────────────────────────────────────┐
│                      Backend API Layer                       │
│              Express.js + Node.js 18                         │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────┐        │
│  │   Routes    │  │ Controllers  │  │ Middleware  │        │
│  └──────┬──────┘  └──────┬───────┘  └──────┬──────┘        │
│         │                │                  │                │
│         └────────────────┴──────────────────┘                │
│                          │                                   │
│                 ┌────────▼────────┐                          │
│                 │    Services      │                          │
│                 └────────┬────────┘                          │
└──────────────────────────┼──────────────────────────────────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
    ┌────▼─────┐    ┌─────▼──────┐    ┌────▼─────┐
    │PostgreSQL│    │   Redis    │    │   MQTT   │
    │   (RDS)  │    │  (Cache)   │    │ Broker   │
    └──────────┘    └────────────┘    └────┬─────┘
                                             │
                                    ┌────────▼─────────┐
                                    │   IoT Devices    │
                                    │ ESP32 + RPi      │
                                    └──────────────────┘
```

### Backend Architecture

```
backend/
├── src/
│   ├── config/              # Configuration files
│   │   ├── database.js      # Sequelize configuration
│   │   └── redis.js         # Redis client setup
│   │
│   ├── controllers/         # Request handlers
│   │   ├── authController.js
│   │   ├── farmController.js
│   │   ├── batchController.js
│   │   └── ...
│   │
│   ├── middleware/          # Express middleware
│   │   ├── auth.js          # JWT authentication
│   │   ├── validation.js    # Request validation
│   │   ├── errorHandler.js  # Error handling
│   │   └── rateLimit.js     # Rate limiting
│   │
│   ├── models/              # Sequelize models (35+ models)
│   │   ├── User.js
│   │   ├── Farm.js
│   │   ├── Batch.js
│   │   └── ...
│   │
│   ├── routes/              # API routes
│   │   ├── auth.routes.js
│   │   ├── farm.routes.js
│   │   └── ...
│   │
│   ├── services/            # Business logic
│   │   ├── authService.js
│   │   ├── notificationService.js
│   │   ├── mqttService.js
│   │   └── ...
│   │
│   ├── utils/               # Utility functions
│   │   └── logger.js
│   │
│   └── index.js             # Application entry point
│
├── tests/                   # Test files
├── package.json
└── .env.example
```

### Frontend Architecture

```
frontend/
├── src/
│   ├── components/          # Reusable UI components
│   │   ├── ui/              # Base UI components
│   │   │   ├── Button.jsx
│   │   │   ├── Card.jsx
│   │   │   ├── Modal.jsx
│   │   │   └── ...
│   │   ├── forms/           # Form components
│   │   ├── charts/          # Chart components
│   │   └── layout/          # Layout components
│   │
│   ├── pages/               # Page components (39+ pages)
│   │   ├── Dashboard.jsx
│   │   ├── Farms/
│   │   ├── Batches/
│   │   ├── Analytics/
│   │   └── ...
│   │
│   ├── layouts/             # Page layouts
│   │   ├── MainLayout.jsx
│   │   └── AuthLayout.jsx
│   │
│   ├── services/            # API services
│   │   ├── api.js           # Axios instance
│   │   ├── authService.js
│   │   └── farmService.js
│   │
│   ├── stores/              # Zustand stores
│   │   └── authStore.js
│   │
│   ├── context/             # React Context
│   │   └── AuthContext.jsx
│   │
│   ├── App.jsx              # Root component
│   ├── main.jsx             # Application entry
│   └── index.css            # Global styles
│
├── public/                  # Static assets
├── package.json
└── vite.config.js
```

### Request Flow

```
1. User Action → 2. React Component → 3. API Service → 4. Backend Route
                                                            ↓
                                                       5. Middleware
                                                            ↓
                                                       6. Controller
                                                            ↓
                                                       7. Service Layer
                                                            ↓
                                                       8. Database Query
                                                            ↓
                                                       9. Response ← ← ← ← ←
```

---

## 📝 Coding Standards

### JavaScript/Node.js Style Guide

**Use ES6+ Features**

```javascript
// ✅ Good: Use const/let
const farmId = 123;
let status = 'active';

// ❌ Bad: Avoid var
var farmId = 123;

// ✅ Good: Arrow functions
const calculateYield = (weight, batches) => weight / batches;

// ✅ Good: Template literals
const message = `Farm ${farmId} has ${count} active batches`;

// ✅ Good: Destructuring
const { name, location, type } = farm;

// ✅ Good: Async/await
async function fetchFarms() {
  const response = await api.get('/farms');
  return response.data;
}
```

**Error Handling**

```javascript
// ✅ Good: Try-catch with async/await
async function createBatch(batchData) {
  try {
    const batch = await Batch.create(batchData);
    return { success: true, data: batch };
  } catch (error) {
    logger.error('Failed to create batch:', error);
    throw new AppError('Failed to create batch', 500);
  }
}

// ✅ Good: Promise rejection handling
fetchData()
  .then(data => processData(data))
  .catch(error => handleError(error));

// ❌ Bad: Unhandled promise
fetchData().then(data => processData(data)); // No .catch()
```

**Function Naming**

```javascript
// ✅ Good: Descriptive function names
function calculateBioEfficiency(weight, substrateWeight) { }
function validateBatchData(data) { }
async function fetchActiveHarvests() { }

// ❌ Bad: Unclear names
function calc(a, b) { }
function doIt() { }
function getData() { }
```

**Comments & Documentation**

```javascript
/**
 * Calculate biological efficiency of a mushroom batch
 * @param {number} harvestWeight - Total fresh weight of harvests (kg)
 * @param {number} substrateWeight - Dry weight of substrate used (kg)
 * @returns {number} Biological efficiency as percentage
 */
function calculateBioEfficiency(harvestWeight, substrateWeight) {
  if (substrateWeight === 0) {
    throw new Error('Substrate weight cannot be zero');
  }
  
  // Bio-efficiency = (Fresh weight / Dry substrate weight) × 100
  return (harvestWeight / substrateWeight) * 100;
}
```

### React/JSX Style Guide

**Component Structure**

```jsx
// ✅ Good: Functional component with hooks
import React, { useState, useEffect } from 'react';
import { Card, Button } from '../components/ui';
import { fetchFarms } from '../services/farmService';

/**
 * FarmList component displays all farms
 * @component
 */
function FarmList() {
  const [farms, setFarms] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadFarms();
  }, []);

  const loadFarms = async () => {
    try {
      const data = await fetchFarms();
      setFarms(data);
    } catch (error) {
      console.error('Failed to load farms:', error);
    } finally {
      setLoading(false);
    }
  };

  if (loading) return <div>Loading...</div>;

  return (
    <div className="grid grid-cols-3 gap-4">
      {farms.map(farm => (
        <Card key={farm.id}>
          <h3>{farm.name}</h3>
          <p>{farm.location}</p>
        </Card>
      ))}
    </div>
  );
}

export default FarmList;
```

**Props and PropTypes**

```jsx
import PropTypes from 'prop-types';

/**
 * BatchCard component
 * @param {Object} batch - Batch data
 * @param {Function} onEdit - Edit callback
 */
function BatchCard({ batch, onEdit }) {
  return (
    <Card>
      <h3>{batch.batchCode}</h3>
      <Button onClick={() => onEdit(batch)}>Edit</Button>
    </Card>
  );
}

BatchCard.propTypes = {
  batch: PropTypes.shape({
    id: PropTypes.number.isRequired,
    batchCode: PropTypes.string.isRequired,
    status: PropTypes.string,
  }).isRequired,
  onEdit: PropTypes.func.isRequired,
};

export default BatchCard;
```

**State Management**

```jsx
// ✅ Good: Zustand for global state
// stores/authStore.js
import create from 'zustand';

export const useAuthStore = create((set) => ({
  user: null,
  token: null,
  setAuth: (user, token) => set({ user, token }),
  logout: () => set({ user: null, token: null }),
}));

// Usage in component
import { useAuthStore } from '../stores/authStore';

function Profile() {
  const { user, logout } = useAuthStore();
  
  return (
    <div>
      <h2>{user.name}</h2>
      <Button onClick={logout}>Logout</Button>
    </div>
  );
}
```

### Database Patterns

**Model Definition**

```javascript
// models/Batch.js
const { DataTypes } = require('sequelize');

module.exports = (sequelize) => {
  const Batch = sequelize.define('Batch', {
    id: {
      type: DataTypes.INTEGER,
      primaryKey: true,
      autoIncrement: true,
    },
    batchCode: {
      type: DataTypes.STRING(50),
      allowNull: false,
      unique: true,
      validate: {
        notEmpty: true,
      },
    },
    status: {
      type: DataTypes.ENUM('preparation', 'inoculation', 'incubation', 'fruiting', 'completed'),
      defaultValue: 'preparation',
    },
    // ... more fields
  }, {
    tableName: 'batches',
    timestamps: true,
    indexes: [
      { fields: ['batchCode'] },
      { fields: ['status'] },
      { fields: ['farmId', 'createdAt'] },
    ],
  });

  Batch.associate = (models) => {
    Batch.belongsTo(models.Farm, { foreignKey: 'farmId' });
    Batch.hasMany(models.Harvest, { foreignKey: 'batchId' });
  };

  return Batch;
};
```

**Query Patterns**

```javascript
// ✅ Good: Use transactions for multiple operations
async function createBatchWithHarvest(batchData, harvestData) {
  const transaction = await sequelize.transaction();
  
  try {
    const batch = await Batch.create(batchData, { transaction });
    const harvest = await Harvest.create({
      ...harvestData,
      batchId: batch.id,
    }, { transaction });
    
    await transaction.commit();
    return { batch, harvest };
  } catch (error) {
    await transaction.rollback();
    throw error;
  }
}

// ✅ Good: Use includes for associations
const batches = await Batch.findAll({
  where: { status: 'active' },
  include: [
    {
      model: Farm,
      attributes: ['name', 'location'],
    },
    {
      model: Harvest,
      where: { harvestDate: { [Op.gte]: startDate } },
      required: false, // LEFT JOIN
    },
  ],
  order: [['createdAt', 'DESC']],
  limit: 20,
});
```

### API Design Patterns

**Controller Pattern**

```javascript
// controllers/batchController.js
const { Batch, Farm } = require('../models');
const { AppError } = require('../utils/errors');

/**
 * Get all batches
 * @route GET /api/batches
 */
exports.getBatches = async (req, res, next) => {
  try {
    const { page = 1, limit = 20, status, farmId } = req.query;
    
    const where = {};
    if (status) where.status = status;
    if (farmId) where.farmId = farmId;
    
    const offset = (page - 1) * limit;
    
    const { count, rows } = await Batch.findAndCountAll({
      where,
      include: [{ model: Farm, attributes: ['name'] }],
      limit: parseInt(limit),
      offset: parseInt(offset),
      order: [['createdAt', 'DESC']],
    });
    
    res.json({
      success: true,
      data: rows,
      pagination: {
        total: count,
        page: parseInt(page),
        pages: Math.ceil(count / limit),
      },
    });
  } catch (error) {
    next(error);
  }
};

/**
 * Create new batch
 * @route POST /api/batches
 */
exports.createBatch = async (req, res, next) => {
  try {
    const batchData = req.body;
    
    // Validation
    if (!batchData.farmId) {
      throw new AppError('Farm ID is required', 400);
    }
    
    const batch = await Batch.create({
      ...batchData,
      userId: req.user.id, // From auth middleware
    });
    
    res.status(201).json({
      success: true,
      data: batch,
    });
  } catch (error) {
    next(error);
  }
};
```

**Route Definition**

```javascript
// routes/batch.routes.js
const express = require('express');
const router = express.Router();
const batchController = require('../controllers/batchController');
const { authenticate, authorize } = require('../middleware/auth');
const { validateBatch } = require('../middleware/validation');

// All routes require authentication
router.use(authenticate);

// Public routes (all authenticated users)
router.get('/', batchController.getBatches);
router.get('/:id', batchController.getBatch);

// Manager+ only
router.post('/', 
  authorize(['manager', 'admin']),
  validateBatch,
  batchController.createBatch
);

router.patch('/:id',
  authorize(['manager', 'admin']),
  validateBatch,
  batchController.updateBatch
);

// Admin only
router.delete('/:id',
  authorize(['admin']),
  batchController.deleteBatch
);

module.exports = router;
```

---

## 🔀 Git Workflow

### Branch Strategy

```
main (production)
  ↑
  ├── release/v1.2.0 (release preparation)
  │
develop (integration)
  ↑
  ├── feature/batch-qr-codes
  ├── feature/analytics-dashboard
  ├── fix/harvest-validation-bug
  └── hotfix/critical-auth-fix
```

**Branch Types:**

1. **main** - Production-ready code
2. **develop** - Integration branch
3. **feature/** - New features
4. **fix/** - Bug fixes
5. **hotfix/** - Critical production fixes
6. **release/** - Release preparation

### Commit Message Format

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code formatting (no functional changes)
- `refactor`: Code refactoring
- `perf`: Performance improvements
- `test`: Adding or updating tests
- `chore`: Build process or tooling changes
- `ci`: CI/CD changes

**Examples:**

```bash
# Feature commit
git commit -m "feat(batches): add QR code generation for batch tracking"

# Bug fix
git commit -m "fix(harvest): prevent negative weight values in validation"

# Breaking change
git commit -m "feat(api)!: change batch status enum values

BREAKING CHANGE: Batch status 'in_progress' renamed to 'active'.
Update all API calls to use new status value."

# Multiple changes
git commit -m "feat(dashboard): add real-time sensor data display

- Add WebSocket connection for sensor updates
- Create SensorChart component with Recharts
- Update dashboard layout to accommodate sensor widgets
- Add error handling for connection failures

Closes #123"
```

### Daily Workflow

```bash
# 1. Start new feature
git checkout develop
git pull origin develop
git checkout -b feature/my-feature

# 2. Make changes and commit frequently
git add .
git commit -m "feat(scope): description"

# 3. Keep feature branch updated
git fetch origin
git rebase origin/develop

# 4. Push to remote
git push origin feature/my-feature

# 5. Create Pull Request on GitHub

# 6. Address review comments
git add .
git commit -m "refactor: address review comments"
git push origin feature/my-feature

# 7. After merge, delete branch
git checkout develop
git pull origin develop
git branch -d feature/my-feature
```

### PR Workflow

1. **Create Issue First** ⚠️ REQUIRED
2. Create feature branch
3. Develop and test locally
4. Push and create PR
5. Link PR to issue: "Closes #123"
6. Wait for CI checks to pass
7. Request review from team
8. Address feedback
9. Get approval
10. Squash and merge
11. Delete branch

---

## 🧪 Testing Guidelines

### Backend Testing

**Unit Tests (Jest + Supertest)**

```javascript
// tests/unit/services/batchService.test.js
const batchService = require('../../../src/services/batchService');
const { Batch } = require('../../../src/models');

// Mock database
jest.mock('../../../src/models');

describe('Batch Service', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('createBatch', () => {
    it('should create a new batch successfully', async () => {
      const batchData = {
        batchCode: 'BATCH-001',
        farmId: 1,
        recipeId: 1,
      };

      Batch.create.mockResolvedValue({
        id: 1,
        ...batchData,
      });

      const result = await batchService.createBatch(batchData);

      expect(result).toHaveProperty('id');
      expect(result.batchCode).toBe('BATCH-001');
      expect(Batch.create).toHaveBeenCalledWith(batchData);
    });

    it('should throw error for duplicate batch code', async () => {
      const batchData = { batchCode: 'BATCH-001' };

      Batch.create.mockRejectedValue(
        new Error('Duplicate batch code')
      );

      await expect(
        batchService.createBatch(batchData)
      ).rejects.toThrow('Duplicate batch code');
    });
  });
});
```

**Integration Tests**

```javascript
// tests/integration/batch.test.js
const request = require('supertest');
const app = require('../../src/index');
const { sequelize } = require('../../src/models');

describe('Batch API', () => {
  let authToken;

  beforeAll(async () => {
    await sequelize.sync({ force: true });
    
    // Create test user and get token
    const response = await request(app)
      .post('/api/auth/register')
      .send({
        email: 'test@example.com',
        password: 'password123',
        name: 'Test User',
      });
    
    authToken = response.body.token;
  });

  afterAll(async () => {
    await sequelize.close();
  });

  describe('POST /api/batches', () => {
    it('should create a new batch', async () => {
      const response = await request(app)
        .post('/api/batches')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          batchCode: 'BATCH-001',
          farmId: 1,
          recipeId: 1,
        });

      expect(response.status).toBe(201);
      expect(response.body.success).toBe(true);
      expect(response.body.data).toHaveProperty('id');
    });

    it('should return 401 without authentication', async () => {
      const response = await request(app)
        .post('/api/batches')
        .send({ batchCode: 'BATCH-002' });

      expect(response.status).toBe(401);
    });
  });
});
```

**Running Tests**

```bash
# Run all tests
npm test

# Run with coverage
npm test -- --coverage

# Run specific test file
npm test -- batch.test.js

# Run tests in watch mode
npm test -- --watch

# Run integration tests only
npm test -- --testPathPattern=integration
```

### Frontend Testing

**Component Tests (Vitest + React Testing Library)**

```jsx
// tests/components/BatchCard.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import BatchCard from '../../src/components/BatchCard';

describe('BatchCard', () => {
  const mockBatch = {
    id: 1,
    batchCode: 'BATCH-001',
    status: 'active',
    cropName: 'Oyster Mushroom',
  };

  it('renders batch information correctly', () => {
    render(<BatchCard batch={mockBatch} />);
    
    expect(screen.getByText('BATCH-001')).toBeInTheDocument();
    expect(screen.getByText('Oyster Mushroom')).toBeInTheDocument();
  });

  it('calls onEdit when edit button is clicked', () => {
    const onEdit = vi.fn();
    render(<BatchCard batch={mockBatch} onEdit={onEdit} />);
    
    const editButton = screen.getByRole('button', { name: /edit/i });
    fireEvent.click(editButton);
    
    expect(onEdit).toHaveBeenCalledWith(mockBatch);
  });
});
```

**Running Frontend Tests**

```bash
# Run tests
npm test

# Run with UI
npm test -- --ui

# Run with coverage
npm test -- --coverage
```

---

## 🎯 API Development

### Creating a New API Endpoint

**1. Define Model**

```javascript
// models/Task.js
module.exports = (sequelize, DataTypes) => {
  const Task = sequelize.define('Task', {
    title: {
      type: DataTypes.STRING,
      allowNull: false,
    },
    description: DataTypes.TEXT,
    status: {
      type: DataTypes.ENUM('pending', 'in_progress', 'completed'),
      defaultValue: 'pending',
    },
    dueDate: DataTypes.DATE,
  });

  Task.associate = (models) => {
    Task.belongsTo(models.User, { foreignKey: 'assignedTo' });
    Task.belongsTo(models.Batch, { foreignKey: 'batchId' });
  };

  return Task;
};
```

**2. Create Controller**

```javascript
// controllers/taskController.js
const { Task, User } = require('../models');

exports.getTasks = async (req, res, next) => {
  try {
    const tasks = await Task.findAll({
      include: [{ model: User, attributes: ['name', 'email'] }],
    });
    
    res.json({ success: true, data: tasks });
  } catch (error) {
    next(error);
  }
};

exports.createTask = async (req, res, next) => {
  try {
    const task = await Task.create(req.body);
    res.status(201).json({ success: true, data: task });
  } catch (error) {
    next(error);
  }
};
```

**3. Define Routes**

```javascript
// routes/task.routes.js
const express = require('express');
const router = express.Router();
const taskController = require('../controllers/taskController');
const { authenticate } = require('../middleware/auth');

router.use(authenticate);

router.get('/', taskController.getTasks);
router.post('/', taskController.createTask);
router.get('/:id', taskController.getTask);
router.patch('/:id', taskController.updateTask);
router.delete('/:id', taskController.deleteTask);

module.exports = router;
```

**4. Register Routes**

```javascript
// src/index.js
const taskRoutes = require('./routes/task.routes');

app.use('/api/tasks', taskRoutes);
```

**5. Test with Postman**

```bash
# GET all tasks
GET http://localhost:8080/api/tasks
Authorization: Bearer YOUR_JWT_TOKEN

# POST new task
POST http://localhost:8080/api/tasks
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json

{
  "title": "Water plants",
  "description": "Water all plants in zone A",
  "status": "pending",
  "assignedTo": 1,
  "batchId": 5,
  "dueDate": "2025-01-20"
}
```

### API Response Format

**Success Response:**

```json
{
  "success": true,
  "data": {
    "id": 1,
    "title": "Task title",
    "status": "completed"
  },
  "message": "Task created successfully"
}
```

**List Response (with pagination):**

```json
{
  "success": true,
  "data": [...],
  "pagination": {
    "total": 150,
    "page": 1,
    "pages": 15,
    "limit": 10
  }
}
```

**Error Response:**

```json
{
  "success": false,
  "error": {
    "message": "Validation failed",
    "code": "VALIDATION_ERROR",
    "details": [
      {
        "field": "email",
        "message": "Email is required"
      }
    ]
  }
}
```

---

## 🎨 Frontend Development

### Creating a New Page

**1. Create Page Component**

```jsx
// pages/Tasks/TaskList.jsx
import React, { useState, useEffect } from 'react';
import { Card, Button, Table } from '../../components/ui';
import { fetchTasks } from '../../services/taskService';

function TaskList() {
  const [tasks, setTasks] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadTasks();
  }, []);

  const loadTasks = async () => {
    try {
      const data = await fetchTasks();
      setTasks(data);
    } catch (error) {
      console.error('Failed to load tasks:', error);
    } finally {
      setLoading(false);
    }
  };

  if (loading) {
    return <div className="flex justify-center p-8">Loading...</div>;
  }

  return (
    <div className="container mx-auto p-6">
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-3xl font-bold">Tasks</h1>
        <Button onClick={() => navigate('/tasks/new')}>
          + New Task
        </Button>
      </div>

      <Card>
        <Table
          columns={[
            { key: 'title', label: 'Title' },
            { key: 'status', label: 'Status' },
            { key: 'dueDate', label: 'Due Date' },
          ]}
          data={tasks}
        />
      </Card>
    </div>
  );
}

export default TaskList;
```

**2. Create API Service**

```javascript
// services/taskService.js
import api from './api';

export const fetchTasks = async (params = {}) => {
  const response = await api.get('/tasks', { params });
  return response.data.data;
};

export const createTask = async (taskData) => {
  const response = await api.post('/tasks', taskData);
  return response.data.data;
};

export const updateTask = async (id, taskData) => {
  const response = await api.patch(`/tasks/${id}`, taskData);
  return response.data.data;
};

export const deleteTask = async (id) => {
  await api.delete(`/tasks/${id}`);
};
```

**3. Add Route**

```jsx
// App.jsx
import TaskList from './pages/Tasks/TaskList';

function App() {
  return (
    <Router>
      <Routes>
        <Route path="/tasks" element={<TaskList />} />
        {/* ... other routes */}
      </Routes>
    </Router>
  );
}
```

### Tailwind CSS Patterns

```jsx
// Layout patterns
<div className="container mx-auto px-4 py-6">
  {/* Content */}
</div>

// Card patterns
<div className="bg-white rounded-lg shadow-md p-6">
  {/* Card content */}
</div>

// Grid layouts
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  {/* Grid items */}
</div>

// Flex layouts
<div className="flex items-center justify-between">
  <span>Left content</span>
  <button>Right button</button>
</div>

// Responsive text
<h1 className="text-2xl md:text-3xl lg:text-4xl font-bold">
  Heading
</h1>

// Status badges
<span className={`px-3 py-1 rounded-full text-sm font-medium ${
  status === 'active' ? 'bg-green-100 text-green-800' :
  status === 'pending' ? 'bg-yellow-100 text-yellow-800' :
  'bg-gray-100 text-gray-800'
}`}>
  {status}
</span>
```

---

## 🗄️ Database Migrations

### Creating a Migration

```bash
# Create new migration
npx sequelize-cli migration:generate --name add-tasks-table

# This creates: migrations/20250115120000-add-tasks-table.js
```

**Migration File:**

```javascript
// migrations/20250115120000-add-tasks-table.js
'use strict';

module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.createTable('tasks', {
      id: {
        type: Sequelize.INTEGER,
        primaryKey: true,
        autoIncrement: true,
      },
      title: {
        type: Sequelize.STRING,
        allowNull: false,
      },
      description: {
        type: Sequelize.TEXT,
      },
      status: {
        type: Sequelize.ENUM('pending', 'in_progress', 'completed'),
        defaultValue: 'pending',
      },
      assignedTo: {
        type: Sequelize.INTEGER,
        references: {
          model: 'users',
          key: 'id',
        },
        onUpdate: 'CASCADE',
        onDelete: 'SET NULL',
      },
      createdAt: {
        type: Sequelize.DATE,
        allowNull: false,
      },
      updatedAt: {
        type: Sequelize.DATE,
        allowNull: false,
      },
    });

    // Add indexes
    await queryInterface.addIndex('tasks', ['status']);
    await queryInterface.addIndex('tasks', ['assignedTo']);
  },

  async down(queryInterface, Sequelize) {
    await queryInterface.dropTable('tasks');
  }
};
```

### Running Migrations

```bash
# Run all pending migrations
npm run migrate

# Or with sequelize-cli directly
npx sequelize-cli db:migrate

# Undo last migration
npx sequelize-cli db:migrate:undo

# Undo all migrations
npx sequelize-cli db:migrate:undo:all

# Undo to specific migration
npx sequelize-cli db:migrate:undo:to --to 20250115120000-add-tasks-table.js
```

---

## 🐛 Debugging

### Backend Debugging

**VS Code Launch Configuration:**

```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Backend",
      "skipFiles": ["<node_internals>/**"],
      "program": "${workspaceFolder}/backend/src/index.js",
      "envFile": "${workspaceFolder}/backend/.env",
      "console": "integratedTerminal"
    }
  ]
}
```

**Logging with Winston:**

```javascript
const logger = require('../utils/logger');

// Log levels: error, warn, info, debug
logger.info('Batch created', { batchId: batch.id });
logger.error('Failed to create batch', { error: error.message });
logger.debug('Processing batch data', { data: batchData });
```

**Database Query Logging:**

```javascript
// Enable in development
// config/database.js
module.exports = {
  development: {
    dialect: 'sqlite',
    logging: console.log, // Log all SQL queries
  },
};
```

### Frontend Debugging

**React DevTools:**
- Install [React DevTools](https://chrome.google.com/webstore/detail/react-developer-tools/)
- Inspect component hierarchy
- View props and state
- Profile performance

**Network Debugging:**

```javascript
// Add request/response interceptors
// services/api.js
import axios from 'axios';

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
});

// Log all requests
api.interceptors.request.use(request => {
  console.log('Starting Request', request);
  return request;
});

// Log all responses
api.interceptors.response.use(
  response => {
    console.log('Response:', response);
    return response;
  },
  error => {
    console.error('Response Error:', error);
    return Promise.reject(error);
  }
);

export default api;
```

---

## ⚡ Performance Optimization

### Backend Performance

**1. Database Query Optimization**

```javascript
// ❌ Bad: N+1 query problem
const batches = await Batch.findAll();
for (const batch of batches) {
  const farm = await Farm.findByPk(batch.farmId); // N queries!
}

// ✅ Good: Use eager loading
const batches = await Batch.findAll({
  include: [{ model: Farm }], // Single query with JOIN
});
```

**2. Caching with Redis**

```javascript
const redis = require('../config/redis');

async function getFarms() {
  const cacheKey = 'farms:all';
  
  // Try cache first
  const cached = await redis.get(cacheKey);
  if (cached) {
    return JSON.parse(cached);
  }
  
  // Query database
  const farms = await Farm.findAll();
  
  // Cache for 5 minutes
  await redis.setex(cacheKey, 300, JSON.stringify(farms));
  
  return farms;
}
```

**3. Pagination**

```javascript
// Always paginate large datasets
const { page = 1, limit = 20 } = req.query;
const offset = (page - 1) * limit;

const { count, rows } = await Batch.findAndCountAll({
  limit: parseInt(limit),
  offset: parseInt(offset),
});
```

### Frontend Performance

**1. Code Splitting**

```jsx
// Lazy load pages
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./pages/Dashboard'));
const Analytics = lazy(() => import('./pages/Analytics'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/" element={<Dashboard />} />
        <Route path="/analytics" element={<Analytics />} />
      </Routes>
    </Suspense>
  );
}
```

**2. Memoization**

```jsx
import { useMemo, useCallback } from 'react';

function BatchList({ batches, filters }) {
  // Memoize expensive calculations
  const filteredBatches = useMemo(() => {
    return batches.filter(batch => 
      batch.status === filters.status
    );
  }, [batches, filters.status]);

  // Memoize callback functions
  const handleEdit = useCallback((batch) => {
    // Handle edit
  }, []);

  return (
    <div>
      {filteredBatches.map(batch => (
        <BatchCard 
          key={batch.id} 
          batch={batch}
          onEdit={handleEdit}
        />
      ))}
    </div>
  );
}
```

**3. React Query for Data Fetching**

```jsx
import { useQuery } from '@tanstack/react-query';

function FarmList() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['farms'],
    queryFn: fetchFarms,
    staleTime: 5 * 60 * 1000, // 5 minutes
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return <div>{/* Render farms */}</div>;
}
```

---

## 🔐 Security Best Practices

### Authentication & Authorization

```javascript
// ✅ Always hash passwords
const bcrypt = require('bcryptjs');
const hashedPassword = await bcrypt.hash(password, 10);

// ✅ Use JWT securely
const jwt = require('jsonwebtoken');
const token = jwt.sign(
  { userId: user.id, role: user.role },
  process.env.JWT_SECRET,
  { expiresIn: '7d' }
);

// ✅ Protect routes
router.post('/batches', authenticate, authorize(['manager', 'admin']), createBatch);

// ❌ Never expose secrets
// Don't: const SECRET = 'mysecret';
// Do: const SECRET = process.env.JWT_SECRET;
```

### Input Validation

```javascript
const Joi = require('joi');

const batchSchema = Joi.object({
  batchCode: Joi.string().required().min(3).max(50),
  farmId: Joi.number().integer().required(),
  status: Joi.string().valid('preparation', 'inoculation', 'incubation'),
});

// Validate in middleware
exports.validateBatch = (req, res, next) => {
  const { error } = batchSchema.validate(req.body);
  if (error) {
    return res.status(400).json({ error: error.details[0].message });
  }
  next();
};
```

### SQL Injection Prevention

```javascript
// ✅ Good: Use Sequelize parameterized queries
const batches = await Batch.findAll({
  where: { status: req.query.status }, // Safe
});

// ❌ Bad: Raw SQL with string concatenation
const query = `SELECT * FROM batches WHERE status = '${req.query.status}'`;
// This is vulnerable to SQL injection!
```

### XSS Prevention

```jsx
// ✅ Good: React escapes by default
<div>{userInput}</div>

// ⚠️  Careful: dangerouslySetInnerHTML
<div dangerouslySetInnerHTML={{ __html: sanitizedHTML }} />
// Only use with sanitized HTML (use DOMPurify library)

// ✅ Sanitize HTML
import DOMPurify from 'dompurify';
const sanitizedHTML = DOMPurify.sanitize(userInput);
```

---

## 👀 Code Review Guidelines

### As a Reviewer

**What to Look For:**

1. **Functionality**
   - Does the code do what it's supposed to?
   - Are edge cases handled?
   - Is error handling comprehensive?

2. **Code Quality**
   - Is the code readable and maintainable?
   - Are naming conventions followed?
   - Is there duplicate code that should be refactored?

3. **Tests**
   - Are there tests for new features?
   - Do tests cover edge cases?
   - Are tests meaningful?

4. **Security**
   - Are inputs validated?
   - Are secrets properly secured?
   - Is authentication/authorization correct?

5. **Performance**
   - Are database queries optimized?
   - Is pagination used for large datasets?
   - Are expensive operations cached?

6. **Documentation**
   - Are complex parts documented?
   - Are API changes documented?
   - Is CHANGELOG.md updated?

**Review Tone:**

```
✅ Good: "Consider using Promise.all() here to run these requests in parallel for better performance."

❌ Bad: "This code is wrong. You should use Promise.all()."

✅ Good: "Great implementation! One suggestion: we could extract this logic into a utility function for reusability."

❌ Bad: "This is bad code."
```

### As a Pull Request Author

**Checklist Before Requesting Review:**

- [ ] All tests pass locally
- [ ] Linting passes with no errors
- [ ] Self-reviewed the code
- [ ] Removed debug statements and console.logs
- [ ] Added/updated tests
- [ ] Updated documentation
- [ ] Updated CHANGELOG.md
- [ ] Rebased on latest develop
- [ ] PR description is clear and complete

---

## 🐛 Troubleshooting

### Common Issues

**Database Connection Fails**

```bash
# Error: Unable to connect to database
# Solution:
1. Check PostgreSQL is running: pg_isready
2. Verify .env credentials match database
3. Check PostgreSQL logs: tail -f /var/log/postgresql/postgresql-15-main.log
```

**Port Already in Use**

```bash
# Error: Port 8080 is already in use
# Solution:
# Find process using port
lsof -i :8080
# Kill process
kill -9 <PID>
# Or change PORT in .env
```

**Module Not Found**

```bash
# Error: Cannot find module 'some-package'
# Solution:
rm -rf node_modules package-lock.json
npm install
```

**CORS Errors**

```javascript
// Add to backend: src/index.js
const cors = require('cors');
app.use(cors({
  origin: 'http://localhost:5173', // Frontend URL
  credentials: true,
}));
```

**JWT Invalid Signature**

```bash
# Error: JWT signature verification failed
# Solution:
# Ensure JWT_SECRET is same in .env
# Clear browser localStorage and login again
```

---

## 📚 Additional Resources

### Official Documentation
- [Node.js Docs](https://nodejs.org/docs)
- [Express.js Guide](https://expressjs.com/guide)
- [React Documentation](https://react.dev)
- [Sequelize Docs](https://sequelize.org)
- [PostgreSQL Manual](https://www.postgresql.org/docs)

### Recommended Reading
- [Clean Code by Robert C. Martin](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)
- [JavaScript: The Good Parts](https://www.amazon.com/JavaScript-Good-Parts-Douglas-Crockford/dp/0596517742)
- [You Don't Know JS (book series)](https://github.com/getify/You-Dont-Know-JS)

### Video Tutorials
- [Node.js Crash Course](https://www.youtube.com/watch?v=fBNz5xF-Kx4)
- [React Tutorial for Beginners](https://www.youtube.com/watch?v=SqcY0GlETPk)
- [PostgreSQL Tutorial](https://www.youtube.com/watch?v=qw--VYLpxG4)

---

## 🤝 Getting Help

### Internal Resources
1. **Team Chat**: Slack #dev-help channel
2. **Weekly Standup**: Monday 10 AM
3. **Code Review Sessions**: Wednesday 2 PM
4. **Documentation**: Check `docs/` folder first

### External Resources
1. **Stack Overflow**: [Tag: cropwise]
2. **GitHub Issues**: Report bugs and request features
3. **Community Forum**: https://community.cropwise.io

### Contact
- **Tech Lead**: tech-lead@cropwise.io
- **DevOps**: devops@cropwise.io
- **General**: dev@cropwise.io

---

**Last Updated**: November 2024  
**Version**: 1.0.0  
**Maintainer**: CropWise Development Team

Happy Coding! 🚀

