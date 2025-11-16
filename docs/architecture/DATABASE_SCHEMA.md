# 🗄️ CropWise - Database Architecture & AI Readiness

## 📊 Current Database Stack

### **Database Type: SQL (PostgreSQL + SQLite)**

#### **Why SQL (Relational) Database?**

CropWise uses **PostgreSQL** (production) and **SQLite** (development) - both SQL databases. Here's why:

### ✅ **Reasons for Choosing SQL:**

#### 1. **Structured Agricultural Data**
Your farm management system has highly **relational data**:
```
Farm → Zones → Batches → Harvests → Quality Checks
  ↓
Recipes → Recipe Stages → Environmental Parameters
  ↓
Tasks → Employees → Work Logs → Cost Entries
  ↓
Equipment → Control Commands → IoT Telemetry
```

**SQL excels at:**
- Complex JOIN operations (e.g., "Show all batches with their zones, farms, recipes, and harvest yields")
- Foreign key constraints (data integrity - can't delete a Farm if Zones exist)
- ACID transactions (critical for financial data like revenue, costs, invoices)

#### 2. **Data Integrity & Consistency**
- **Transactions**: When recording a harvest, you need to:
  - Insert harvest record
  - Update batch status
  - Adjust inventory (substrate consumed)
  - Create quality check entry
  - All these must succeed or fail together (ACID)

#### 3. **Complex Queries & Analytics**
Your dashboard needs queries like:
```sql
SELECT 
  f.name AS farm_name,
  z.name AS zone_name,
  b.batch_number,
  SUM(h.yield_kg) AS total_yield,
  AVG(qc.score) AS avg_quality,
  (SUM(r.amount) - SUM(c.amount)) AS profit
FROM batches b
JOIN zones z ON b.zone_id = z.id
JOIN farms f ON z.farm_id = f.id
LEFT JOIN harvests h ON h.batch_id = b.id
LEFT JOIN quality_checks qc ON qc.batch_id = b.id
LEFT JOIN revenues r ON r.batch_id = b.id
LEFT JOIN cost_entries c ON c.batch_id = b.id
GROUP BY f.id, z.id, b.id
ORDER BY profit DESC;
```

SQL databases are **optimized for these queries**.

#### 4. **Sequelize ORM Benefits**
- **Model Relationships**: Easily define associations
- **Migrations**: Version control for database schema changes
- **TypeScript-friendly**: Strong typing for models
- **Database-agnostic**: Switch between SQLite (dev) and PostgreSQL (production) seamlessly

---

## 🤖 AI & Machine Learning Readiness

### **✅ YES - PostgreSQL is EXCELLENT for AI/ML with Nvidia Jetson**

Here's why PostgreSQL is perfect for your AI future:

### 1. **pgvector Extension - Native Vector Storage**
PostgreSQL supports **pgvector** for storing AI embeddings:

```sql
-- Install pgvector extension
CREATE EXTENSION vector;

-- Store crop image embeddings (from Jetson AI model)
CREATE TABLE crop_images (
  id SERIAL PRIMARY KEY,
  batch_id INTEGER REFERENCES batches(id),
  image_url TEXT,
  embedding vector(512),  -- 512-dimensional vector from ResNet/EfficientNet
  disease_prediction JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Find similar images using cosine similarity
SELECT image_url, 
       embedding <-> '[0.1, 0.2, ..., 0.9]'::vector AS distance
FROM crop_images
ORDER BY distance
LIMIT 10;
```

**Use Cases:**
- Disease detection similarity search
- Visual quality grading
- Anomaly detection in mushroom growth

### 2. **TimescaleDB Extension - Time-Series for IoT + AI**
PostgreSQL + TimescaleDB = Perfect for sensor data + AI predictions:

```sql
-- Install TimescaleDB
CREATE EXTENSION IF NOT EXISTS timescaledb;

-- Convert telemetry table to hypertable
SELECT create_hypertable('telemetries', 'timestamp');

-- Store AI predictions alongside sensor data
CREATE TABLE ai_predictions (
  time TIMESTAMPTZ NOT NULL,
  zone_id INTEGER,
  predicted_yield NUMERIC,
  predicted_quality NUMERIC,
  confidence_score NUMERIC,
  model_version TEXT
);

SELECT create_hypertable('ai_predictions', 'time');

-- Continuous aggregates for ML feature engineering
CREATE MATERIALIZED VIEW hourly_features
WITH (timescaledb.continuous) AS
SELECT 
  time_bucket('1 hour', timestamp) AS hour,
  zone_id,
  AVG(temperature) AS avg_temp,
  STDDEV(temperature) AS temp_variance,
  AVG(humidity) AS avg_humidity,
  MAX(co2) AS max_co2
FROM telemetries
GROUP BY hour, zone_id;
```

### 3. **PostgreSQL ML Extension (PL/Python)**
Run ML models **directly in the database**:

```sql
-- Create a Python function for yield prediction
CREATE OR REPLACE FUNCTION predict_yield(
  temp NUMERIC, 
  humidity NUMERIC, 
  co2 NUMERIC
) RETURNS NUMERIC AS $$
import pickle
import numpy as np

# Load pre-trained model (trained on Jetson)
with open('/models/yield_predictor.pkl', 'rb') as f:
    model = pickle.load(f)

# Make prediction
features = np.array([[temp, humidity, co2]])
prediction = model.predict(features)[0]

return float(prediction)
$$ LANGUAGE plpython3u;

-- Use it in queries
SELECT 
  zone_id,
  predict_yield(temperature, humidity, co2) AS predicted_yield
FROM telemetries
WHERE timestamp > NOW() - INTERVAL '1 hour';
```

### 4. **JSONB for Flexible AI Metadata**
Store complex AI outputs:

```sql
-- Store disease detection results from Jetson Nano
INSERT INTO quality_checks (
  batch_id,
  inspector_id,
  ai_analysis
) VALUES (
  123,
  456,
  '{
    "model": "YOLOv8-disease-v2.1",
    "detections": [
      {
        "class": "bacterial_blotch",
        "confidence": 0.87,
        "bbox": [120, 340, 80, 60],
        "severity": "moderate"
      },
      {
        "class": "healthy_cap",
        "confidence": 0.95,
        "bbox": [220, 100, 120, 90]
      }
    ],
    "overall_grade": "B",
    "timestamp": "2024-11-14T10:30:00Z"
  }'::JSONB
);

-- Query AI results
SELECT 
  batch_id,
  ai_analysis->'detections'->0->>'class' AS primary_issue,
  ai_analysis->'detections'->0->>'confidence' AS confidence
FROM quality_checks
WHERE ai_analysis->'detections'->0->>'class' = 'bacterial_blotch';
```

---

## 🚀 Nvidia Jetson Integration Architecture

### **Recommended Setup:**

```
┌─────────────────────────────────────────────────────────┐
│                   CropWise Platform                 │
│                                                         │
│  ┌─────────────────┐         ┌──────────────────┐    │
│  │  React/Native   │         │  Backend API     │    │
│  │  Frontend       │◄────────┤  Node.js/Express │    │
│  │  (Web/Mobile)   │  REST   │                  │    │
│  └─────────────────┘  API    └──────────────────┘    │
│                                        │               │
│                                        ▼               │
│                              ┌──────────────────┐     │
│                              │  PostgreSQL +    │     │
│                              │  pgvector +      │     │
│                              │  TimescaleDB     │     │
│                              └──────────────────┘     │
└─────────────────────────────────────────────────────────┘
                                        ▲
                                        │ SQL Queries
                                        │ Insert predictions
                                        │
┌─────────────────────────────────────────────────────────┐
│               Farm Edge Computing Layer                 │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Raspberry Pi 5 Gateway                          │  │
│  │  - MQTT Broker (Mosquitto)                       │  │
│  │  - Data aggregation                              │  │
│  │  - Sends to cloud PostgreSQL                     │  │
│  └──────────────────────────────────────────────────┘  │
│                          │                              │
│         ┌────────────────┼────────────────┐            │
│         ▼                ▼                ▼             │
│  ┌──────────┐     ┌──────────┐    ┌──────────┐       │
│  │ ESP32    │     │ ESP32    │    │ ESP32    │       │
│  │ Master   │     │ Sensor   │    │ Irrigation│      │
│  └──────────┘     └──────────┘    └──────────┘       │
│                                                         │
│  ┌────────────────────────────────────────────────┐   │
│  │  Nvidia Jetson Nano/Orin Nano                  │   │
│  │  ┌──────────────────────────────────────────┐  │   │
│  │  │  AI/ML Tasks:                            │  │   │
│  │  │  • Disease detection (camera input)      │  │   │
│  │  │  • Growth stage classification           │  │   │
│  │  │  • Quality grading                       │  │   │
│  │  │  • Yield prediction (time-series)        │  │   │
│  │  │  • Anomaly detection                     │  │   │
│  │  │  • Object counting (mushroom counting)   │  │   │
│  │  └──────────────────────────────────────────┘  │   │
│  │                                                  │   │
│  │  Writes results to PostgreSQL via REST API      │   │
│  └────────────────────────────────────────────────┘   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### **Data Flow:**

1. **Real-time Sensor Data**: ESP32 → Raspberry Pi → PostgreSQL (TimescaleDB)
2. **AI Inference**: Jetson Nano reads from PostgreSQL + Camera → Runs inference → Writes predictions back
3. **Web/Mobile App**: Queries PostgreSQL for both sensor data AND AI predictions

---

## ☁️ AWS Deployment Architecture

### **Production AWS Stack:**

```
┌────────────────────────────────────────────────────────┐
│                    AWS Cloud (VPC)                     │
│                                                        │
│  ┌─────────────────┐         ┌──────────────────┐   │
│  │  CloudFront CDN │         │  Route 53 (DNS)  │   │
│  │  (React App)    │         │  cropwise.io    │   │
│  └─────────────────┘         └──────────────────┘   │
│           │                            │              │
│           ▼                            ▼              │
│  ┌─────────────────────────────────────────────┐    │
│  │  Application Load Balancer (ALB)           │    │
│  │  - SSL/TLS (HTTPS)                         │    │
│  │  - Auto-scaling target groups              │    │
│  └─────────────────────────────────────────────┘    │
│           │                                           │
│           ▼                                           │
│  ┌─────────────────────────────────────────────┐    │
│  │  ECS Fargate / EKS (Kubernetes)            │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐ │    │
│  │  │ Backend  │  │ Backend  │  │ Backend  │ │    │
│  │  │ Node.js  │  │ Node.js  │  │ Node.js  │ │    │
│  │  │ Container│  │ Container│  │ Container│ │    │
│  │  └──────────┘  └──────────┘  └──────────┘ │    │
│  └─────────────────────────────────────────────┘    │
│           │                                           │
│           ├──────────────────┬────────────────┐     │
│           ▼                  ▼                ▼      │
│  ┌──────────────┐   ┌──────────────┐  ┌─────────┐ │
│  │ ElastiCache  │   │  RDS Aurora  │  │   S3    │ │
│  │   (Redis)    │   │ PostgreSQL   │  │ Storage │ │
│  │              │   │              │  │         │ │
│  │ • Sessions   │   │ • All data   │  │• Images │ │
│  │ • Cache      │   │ • pgvector   │  │• Logs   │ │
│  │ • Rate limit │   │ • TimescaleDB│  │• Backups│ │
│  └──────────────┘   └──────────────┘  └─────────┘ │
│                              │                       │
│                              ▼                       │
│                     ┌─────────────────┐             │
│                     │  AWS Backup     │             │
│                     │  (Auto backups) │             │
│                     └─────────────────┘             │
│                                                      │
│  ┌────────────────────────────────────────────┐    │
│  │  IoT Core (Optional - for Jetson/ESP32)   │    │
│  │  - MQTT at scale                           │    │
│  │  - Device shadow                           │    │
│  │  - Rules engine → PostgreSQL               │    │
│  └────────────────────────────────────────────┘    │
│                                                      │
│  ┌────────────────────────────────────────────┐    │
│  │  SageMaker (Optional - ML Training)        │    │
│  │  - Train models on historical data         │    │
│  │  - Deploy to Jetson via IoT Core           │    │
│  └────────────────────────────────────────────┘    │
└────────────────────────────────────────────────────┘
```

### **AWS Services Breakdown:**

| Service | Purpose | Cost Estimate |
|---------|---------|---------------|
| **RDS Aurora PostgreSQL** | Main database (serverless auto-scaling) | $50-200/month |
| **ElastiCache Redis** | Session management, caching | $15-50/month |
| **ECS Fargate** | Backend containers (auto-scaling) | $30-150/month |
| **S3** | Image storage, backups | $5-20/month |
| **CloudFront** | React app CDN | $5-20/month |
| **Application Load Balancer** | Traffic distribution | $20-40/month |
| **Route 53** | DNS management | $1-5/month |
| **AWS IoT Core** (Optional) | MQTT for 1000s of devices | $10-100/month |
| **SageMaker** (Optional) | ML training | Pay per use |
| **Total** | Small-Medium Operation | **$136-585/month** |

---

## 📱 Mobile App Architecture (React Native)

### **✅ YES - Your Backend is Ready for Mobile Apps**

CropWise is built with a **RESTful API** that works perfectly with mobile apps.

### **Tech Stack for Mobile:**

```
┌─────────────────────────────────────────────┐
│         React Native Mobile App             │
│  (Single codebase → iOS + Android)          │
│                                             │
│  ┌───────────────────────────────────────┐ │
│  │  Screens:                             │ │
│  │  • Dashboard (farm overview)          │ │
│  │  • Zone monitoring (real-time charts)│ │
│  │  • Task management                    │ │
│  │  • Batch tracking                     │ │
│  │  • Harvest recording                  │ │
│  │  • Quality checks (camera)            │ │
│  │  • Alerts & notifications             │ │
│  │  • Employee management                │ │
│  └───────────────────────────────────────┘ │
│                                             │
│  ┌───────────────────────────────────────┐ │
│  │  Libraries:                           │ │
│  │  • React Navigation (routing)         │ │
│  │  • Zustand (state management)         │ │
│  │  • Victory Native (charts)            │ │
│  │  • React Native Camera                │ │
│  │  • MQTT.js (real-time IoT)            │ │
│  │  • Axios (API calls)                  │ │
│  │  • react-native-push-notifications    │ │
│  └───────────────────────────────────────┘ │
│                                             │
│              ▼ REST API                     │
│  ┌───────────────────────────────────────┐ │
│  │  Same Backend API as Web App          │ │
│  │  https://api.cropwise.io             │ │
│  └───────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
```

### **Why React Native?**

✅ **Single Codebase**: 95% code reuse between iOS and Android
✅ **Fast Development**: You already know React!
✅ **Native Performance**: Access camera, notifications, GPS
✅ **Expo Support**: Easy testing and deployment
✅ **Large Ecosystem**: Rich library support

### **Reusable Components from Web:**
- State management (Zustand stores)
- API services (`frontend/src/services/`)
- Business logic
- Chart configurations

---

## 🔮 Future AI/ML Use Cases with PostgreSQL

### 1. **Predictive Yield Modeling**
```sql
-- Store training data
CREATE TABLE yield_training_data AS
SELECT 
  b.id AS batch_id,
  AVG(t.temperature) AS avg_temp,
  AVG(t.humidity) AS avg_humidity,
  AVG(t.co2) AS avg_co2,
  SUM(h.yield_kg) AS actual_yield
FROM batches b
JOIN zones z ON b.zone_id = z.id
JOIN telemetries t ON t.zone_id = z.id
LEFT JOIN harvests h ON h.batch_id = b.id
WHERE b.status = 'completed'
GROUP BY b.id;

-- Jetson trains model on this data
-- Jetson writes predictions back
INSERT INTO ai_predictions (batch_id, predicted_yield, confidence)
VALUES (123, 45.7, 0.89);
```

### 2. **Disease Detection & Prevention**
```sql
-- Store image analysis results
CREATE TABLE disease_detections (
  id SERIAL PRIMARY KEY,
  batch_id INTEGER,
  image_embedding vector(512),
  disease_type TEXT,
  confidence NUMERIC,
  treatment_recommended TEXT,
  detected_at TIMESTAMP DEFAULT NOW()
);

-- Find similar disease patterns
SELECT * FROM disease_detections
WHERE image_embedding <-> $current_embedding::vector < 0.3
ORDER BY confidence DESC
LIMIT 5;
```

### 3. **Optimization Recommendations**
```sql
-- ML model suggests optimal conditions
CREATE TABLE optimization_recommendations (
  zone_id INTEGER,
  recommended_temp NUMERIC,
  recommended_humidity NUMERIC,
  expected_yield_increase NUMERIC,
  confidence NUMERIC,
  generated_at TIMESTAMP
);
```

---

## 🎯 Migration Strategy (If Needed)

### **Current: PostgreSQL (SQL)**
### **Future Options:**

#### Option 1: **PostgreSQL + MongoDB (Hybrid)**
- Keep PostgreSQL for transactional data (farms, batches, costs)
- Add MongoDB for:
  - Raw IoT telemetry (high write throughput)
  - AI model metadata (complex nested structures)
  - Log aggregation

#### Option 2: **PostgreSQL + TimescaleDB (Time-Series)**
- Keep PostgreSQL for core data
- TimescaleDB extension for:
  - Sensor data (automatic partitioning)
  - AI predictions over time
  - Continuous aggregates for ML features

#### Option 3: **Stay 100% PostgreSQL** (Recommended)
- Modern PostgreSQL handles:
  - Transactional data ✅
  - Time-series (with TimescaleDB) ✅
  - Vector embeddings (with pgvector) ✅
  - JSON documents (JSONB) ✅
  - Spatial data (PostGIS) ✅

**Verdict: You don't need NoSQL!**

---

## ✅ Summary: You're Future-Ready!

### **Your Current Stack:**
- ✅ PostgreSQL (production) + SQLite (dev)
- ✅ Sequelize ORM (database-agnostic)
- ✅ RESTful API (works with any frontend)
- ✅ JWT authentication
- ✅ Docker-ready

### **AI/ML Ready:**
- ✅ pgvector for embeddings
- ✅ TimescaleDB for time-series
- ✅ JSONB for flexible AI outputs
- ✅ Python integration (PL/Python)
- ✅ Indexes for fast queries

### **AWS Ready:**
- ✅ RDS Aurora PostgreSQL (managed, auto-scaling)
- ✅ ElastiCache Redis (caching)
- ✅ ECS/Fargate (containerized backend)
- ✅ S3 (image storage)
- ✅ CloudFront (CDN)

### **Mobile Ready:**
- ✅ REST API works with React Native
- ✅ State management reusable
- ✅ Push notifications supported
- ✅ Camera integration for quality checks

---

## 🚀 Next Steps

1. **Nvidia Jetson Integration:**
   - Set up Jetson Nano with Python + PyTorch
   - Create REST API endpoints for AI inference results
   - Install pgvector on PostgreSQL
   - Train models using historical data

2. **AWS Deployment:**
   - Follow `docs/AWS_DEPLOYMENT_GUIDE.md`
   - Set up RDS Aurora PostgreSQL
   - Deploy backend to ECS Fargate
   - Configure IoT Core for ESP32/Jetson

3. **Mobile App Development:**
   - Create React Native project
   - Reuse API services from web app
   - Implement push notifications
   - Add camera for quality checks

4. **AI/ML Pipeline:**
   - Export training data from PostgreSQL
   - Train models on Jetson (or SageMaker)
   - Deploy models to edge (Jetson)
   - Stream predictions back to PostgreSQL

---

## 📚 Additional Resources

- [PostgreSQL + AI/ML Guide](https://www.postgresql.org/docs/current/plpython.html)
- [pgvector Documentation](https://github.com/pgvector/pgvector)
- [TimescaleDB Time-Series](https://docs.timescale.com/)
- [Nvidia Jetson AI Projects](https://developer.nvidia.com/embedded/community/jetson-projects)
- [React Native + REST API](https://reactnative.dev/docs/network)
- [AWS IoT Core](https://aws.amazon.com/iot-core/)

---

**Questions? Let's discuss your AI/ML implementation strategy!** 🚀

