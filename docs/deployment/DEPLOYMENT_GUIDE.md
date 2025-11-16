# 🚀 Deployment Guide

**Complete deployment guide for CropWise - from development to production**

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Deployment Options](#-deployment-options)
- [Prerequisites](#-prerequisites)
- [Local Development](#-local-development)
- [Docker Deployment](#-docker-deployment)
- [AWS Production Deployment](#-aws-production-deployment)
- [Environment Configuration](#-environment-configuration)
- [Database Setup](#-database-setup)
- [SSL/TLS Configuration](#-ssltls-configuration)
- [Monitoring & Logging](#-monitoring--logging)
- [Backup & Recovery](#-backup--recovery)
- [Scaling](#-scaling)
- [Troubleshooting](#-troubleshooting)

---

## 🎯 Overview

CropWise supports multiple deployment strategies:

| Environment | Use Case | Infrastructure | Cost | Complexity |
|------------|----------|----------------|------|------------|
| **Local** | Development | Your machine | Free | ⭐ Low |
| **Docker** | Dev/Testing | Docker containers | Free-$10/mo | ⭐⭐ Medium |
| **AWS** | Production | Cloud-native | $50-200/mo | ⭐⭐⭐ High |
| **Self-Hosted** | Custom | Your server | Variable | ⭐⭐⭐⭐ Expert |

---

## 🔧 Prerequisites

### General Requirements

- **Git** 2.30+
- **Node.js** 18+ and npm 9+
- **PostgreSQL** 15+ (production)
- **Redis** 7+ (optional but recommended)
- **SSL Certificate** (for production)

### AWS Requirements

- AWS Account with billing enabled
- AWS CLI v2 installed and configured
- IAM user with appropriate permissions
- Domain name (for production)

### Docker Requirements

- Docker 20.10+
- Docker Compose 2.0+
- 4GB+ RAM available

---

## 💻 Local Development

Perfect for development and testing.

### Quick Start (5 minutes)

```bash
# 1. Clone repository
git clone https://github.com/your-org/cropwise.git
cd cropwise

# 2. Backend setup
cd backend
npm install
cp .env.example .env
npm run migrate
npm start

# 3. Frontend setup (new terminal)
cd frontend
npm install
cp .env.example .env
npm run dev

# 4. Access application
# Frontend: http://localhost:5173
# Backend: http://localhost:8080
```

### Configuration

**Backend .env:**

```bash
NODE_ENV=development
PORT=8080
DB_DIALECT=sqlite
DB_STORAGE=./database.sqlite
JWT_SECRET=your_dev_secret_change_me
```

**Frontend .env:**

```bash
VITE_API_URL=http://localhost:8080
```

---

## 🐳 Docker Deployment

Best for staging and consistent development environments.

### Single Command Deployment

```bash
# Start all services
docker-compose up -d

# Access application
# Frontend: http://localhost:8080
# Backend API: http://localhost:8080/api
# PostgreSQL: localhost:5432
# Redis: localhost:6379
# MQTT: localhost:1883

# View logs
docker-compose logs -f

# Stop all services
docker-compose down

# Stop and remove volumes (⚠️ deletes database)
docker-compose down -v
```

### Docker Compose Configuration

```yaml
# docker-compose.yml
version: '3.8'

services:
  # PostgreSQL Database
  postgres:
    image: postgres:15-alpine
    container_name: cropwise-postgres
    environment:
      POSTGRES_DB: cropwise
      POSTGRES_USER: cropwise
      POSTGRES_PASSWORD: ${DB_PASSWORD:-secure_password}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./docker/postgres/init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U cropwise"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis Cache
  redis:
    image: redis:7-alpine
    container_name: cropwise-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # MQTT Broker (for IoT)
  mqtt:
    image: eclipse-mosquitto:2
    container_name: cropwise-mqtt
    ports:
      - "1883:1883"
      - "9001:9001"
    volumes:
      - ./docker/mosquitto/mosquitto.conf:/mosquitto/config/mosquitto.conf
      - mqtt_data:/mosquitto/data
      - mqtt_logs:/mosquitto/log

  # Backend API
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: cropwise-backend
    environment:
      NODE_ENV: production
      PORT: 8080
      DB_DIALECT: postgres
      DB_HOST: postgres
      DB_PORT: 5432
      DB_NAME: cropwise
      DB_USER: cropwise
      DB_PASSWORD: ${DB_PASSWORD:-secure_password}
      REDIS_HOST: redis
      REDIS_PORT: 6379
      MQTT_BROKER_URL: mqtt://mqtt:1883
      JWT_SECRET: ${JWT_SECRET}
    ports:
      - "8080:8080"
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    volumes:
      - ./backend/logs:/app/logs
    restart: unless-stopped

  # Frontend (Production Build)
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.prod
    container_name: cropwise-frontend
    environment:
      VITE_API_URL: http://localhost:8080
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - backend
    volumes:
      - ./ssl:/etc/nginx/ssl:ro
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
  mqtt_data:
  mqtt_logs:
```

### Building Custom Images

```bash
# Build backend image
docker build -t cropwise/backend:latest ./backend

# Build frontend image
docker build -t cropwise/frontend:latest -f ./frontend/Dockerfile.prod ./frontend

# Push to Docker Hub (optional)
docker login
docker push cropwise/backend:latest
docker push cropwise/frontend:latest
```

---

## ☁️ AWS Production Deployment

Enterprise-grade cloud deployment with auto-scaling and high availability.

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      CloudFront (CDN)                        │
│                    (Frontend Distribution)                   │
└────────────────────────┬────────────────────────────────────┘
                         │
         ┌───────────────┴────────────────┐
         │                                │
    ┌────▼─────┐                  ┌──────▼──────┐
    │    S3    │                  │     ALB     │
    │ (Static) │                  │(Load Balance│
    └──────────┘                  └──────┬──────┘
                                         │
                         ┌───────────────┴────────────────┐
                         │                                │
                    ┌────▼─────┐                  ┌───────▼──────┐
                    │   ECS    │                  │     ECS      │
                    │(Backend 1)                  │ (Backend 2)  │
                    └────┬─────┘                  └───────┬──────┘
                         │                                │
                         └────────────┬───────────────────┘
                                      │
                         ┌────────────┴────────────┐
                         │                         │
                    ┌────▼─────┐           ┌──────▼──────┐
                    │   RDS    │           │ ElastiCache │
                    │(PostgreSQL)          │   (Redis)   │
                    └──────────┘           └─────────────┘
```

### Step-by-Step AWS Deployment

#### 1. Initial Setup

```bash
# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Configure AWS CLI
aws configure
# AWS Access Key ID: YOUR_ACCESS_KEY
# AWS Secret Access Key: YOUR_SECRET_KEY
# Default region: ap-south-1 (or your preferred region)
# Default output format: json

# Verify configuration
aws sts get-caller-identity
```

#### 2. Create VPC and Networking

```bash
# Clone infrastructure repository
cd aws

# Deploy VPC using CloudFormation
aws cloudformation create-stack \
  --stack-name cropwise-vpc \
  --template-body file://cloudformation-template.yaml \
  --parameters \
    ParameterKey=Environment,ParameterValue=production \
    ParameterKey=VpcCIDR,ParameterValue=10.0.0.0/16

# Wait for stack creation
aws cloudformation wait stack-create-complete \
  --stack-name cropwise-vpc

# Get stack outputs
aws cloudformation describe-stacks \
  --stack-name cropwise-vpc \
  --query 'Stacks[0].Outputs'
```

#### 3. Create RDS PostgreSQL Database

```bash
# Create DB subnet group
aws rds create-db-subnet-group \
  --db-subnet-group-name cropwise-db-subnet \
  --db-subnet-group-description "CropWise DB Subnet" \
  --subnet-ids subnet-xxxxx subnet-yyyyy

# Create RDS instance
aws rds create-db-instance \
  --db-instance-identifier cropwise-db \
  --db-instance-class db.t3.medium \
  --engine postgres \
  --engine-version 15.4 \
  --master-username cropwise \
  --master-user-password "SECURE_PASSWORD_HERE" \
  --allocated-storage 100 \
  --storage-type gp3 \
  --db-subnet-group-name cropwise-db-subnet \
  --vpc-security-group-ids sg-xxxxx \
  --backup-retention-period 7 \
  --preferred-backup-window "03:00-04:00" \
  --preferred-maintenance-window "mon:04:00-mon:05:00" \
  --multi-az \
  --storage-encrypted \
  --publicly-accessible false \
  --tags Key=Name,Value=cropwise-production-db

# Wait for database to be available (takes 10-15 minutes)
aws rds wait db-instance-available \
  --db-instance-identifier cropwise-db

# Get database endpoint
aws rds describe-db-instances \
  --db-instance-identifier cropwise-db \
  --query 'DBInstances[0].Endpoint.Address' \
  --output text
```

#### 4. Create ElastiCache Redis Cluster

```bash
# Create cache subnet group
aws elasticache create-cache-subnet-group \
  --cache-subnet-group-name cropwise-cache-subnet \
  --cache-subnet-group-description "CropWise Cache Subnet" \
  --subnet-ids subnet-xxxxx subnet-yyyyy

# Create Redis cluster
aws elasticache create-replication-group \
  --replication-group-id cropwise-redis \
  --replication-group-description "CropWise Redis Cache" \
  --engine redis \
  --engine-version 7.0 \
  --cache-node-type cache.t3.micro \
  --num-cache-clusters 2 \
  --automatic-failover-enabled \
  --cache-subnet-group-name cropwise-cache-subnet \
  --security-group-ids sg-xxxxx \
  --at-rest-encryption-enabled \
  --transit-encryption-enabled

# Wait for Redis to be available
aws elasticache wait replication-group-available \
  --replication-group-id cropwise-redis

# Get Redis endpoint
aws elasticache describe-replication-groups \
  --replication-group-id cropwise-redis \
  --query 'ReplicationGroups[0].NodeGroups[0].PrimaryEndpoint.Address' \
  --output text
```

#### 5. Create ECR Repositories

```bash
# Create backend repository
aws ecr create-repository \
  --repository-name cropwise/backend \
  --image-scanning-configuration scanOnPush=true

# Create frontend repository
aws ecr create-repository \
  --repository-name cropwise/frontend \
  --image-scanning-configuration scanOnPush=true

# Login to ECR
aws ecr get-login-password --region ap-south-1 | \
  docker login --username AWS --password-stdin \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com

# Build and push images
docker build -t cropwise/backend:latest ./backend
docker tag cropwise/backend:latest \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com/cropwise/backend:latest
docker push 123456789012.dkr.ecr.ap-south-1.amazonaws.com/cropwise/backend:latest

docker build -t cropwise/frontend:latest -f ./frontend/Dockerfile.prod ./frontend
docker tag cropwise/frontend:latest \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com/cropwise/frontend:latest
docker push 123456789012.dkr.ecr.ap-south-1.amazonaws.com/cropwise/frontend:latest
```

#### 6. Create ECS Cluster

```bash
# Create ECS cluster
aws ecs create-cluster \
  --cluster-name cropwise-production \
  --capacity-providers FARGATE FARGATE_SPOT \
  --default-capacity-provider-strategy \
    capacityProvider=FARGATE,weight=1 \
    capacityProvider=FARGATE_SPOT,weight=4

# Create task execution role
aws iam create-role \
  --role-name ecsTaskExecutionRole \
  --assume-role-policy-document file://iam-roles.json

# Attach policies
aws iam attach-role-policy \
  --role-name ecsTaskExecutionRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy

# Register task definition
aws ecs register-task-definition \
  --cli-input-json file://ecs-backend-task.json

# Create ECS service
aws ecs create-service \
  --cluster cropwise-production \
  --service-name cropwise-backend \
  --task-definition cropwise-backend:1 \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-xxxxx,subnet-yyyyy],securityGroups=[sg-xxxxx],assignPublicIp=ENABLED}" \
  --load-balancers "targetGroupArn=arn:aws:elasticloadbalancing:...,containerName=cropwise-backend,containerPort=8080"
```

#### 7. Deploy Frontend to S3 + CloudFront

```bash
# Create S3 bucket
aws s3 mb s3://cropwise-frontend-prod

# Enable static website hosting
aws s3 website s3://cropwise-frontend-prod \
  --index-document index.html \
  --error-document index.html

# Set bucket policy
aws s3api put-bucket-policy \
  --bucket cropwise-frontend-prod \
  --policy file://s3-bucket-policy.json

# Build frontend
cd frontend
npm install
npm run build

# Deploy to S3
aws s3 sync dist/ s3://cropwise-frontend-prod --delete

# Create CloudFront distribution
aws cloudfront create-distribution \
  --origin-domain-name cropwise-frontend-prod.s3.ap-south-1.amazonaws.com \
  --default-root-object index.html

# Get CloudFront URL
aws cloudfront list-distributions \
  --query "DistributionList.Items[?Comment=='CropWise Frontend'].DomainName" \
  --output text
```

#### 8. Configure Domain and SSL

```bash
# Request SSL certificate in ACM
aws acm request-certificate \
  --domain-name cropwise.io \
  --subject-alternative-names www.cropwise.io \
  --validation-method DNS

# Get validation CNAME records
aws acm describe-certificate \
  --certificate-arn arn:aws:acm:... \
  --query 'Certificate.DomainValidationOptions'

# Add CNAME records to your DNS provider

# Wait for validation (can take 30 minutes)
aws acm wait certificate-validated \
  --certificate-arn arn:aws:acm:...

# Update CloudFront with custom domain
aws cloudfront update-distribution \
  --id DISTRIBUTION_ID \
  --aliases cropwise.io www.cropwise.io \
  --viewer-certificate "ACMCertificateArn=arn:aws:acm:...,SSLSupportMethod=sni-only,MinimumProtocolVersion=TLSv1.2_2021"
```

#### 9. Run Database Migrations

```bash
# Connect to bastion host or use AWS Systems Manager Session Manager
aws ssm start-session --target i-xxxxx

# On bastion host
git clone https://github.com/your-org/cropwise.git
cd cropwise/backend

# Install dependencies
npm install

# Set environment variables
export DB_DIALECT=postgres
export DB_HOST=cropwise-db.xxxxx.ap-south-1.rds.amazonaws.com
export DB_PORT=5432
export DB_NAME=cropwise
export DB_USER=cropwise
export DB_PASSWORD=SECURE_PASSWORD

# Run migrations
npm run migrate

# Optional: Seed initial data
npm run seed
```

#### 10. Configure Monitoring

```bash
# Create CloudWatch Log Groups
aws logs create-log-group --log-group-name /ecs/cropwise-backend
aws logs create-log-group --log-group-name /ecs/cropwise-frontend

# Create CloudWatch Alarms
aws cloudwatch put-metric-alarm \
  --alarm-name cropwise-high-cpu \
  --alarm-description "Alert when CPU exceeds 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/ECS \
  --statistic Average \
  --period 300 \
  --evaluation-periods 2 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=ClusterName,Value=cropwise-production

# Enable RDS Enhanced Monitoring
aws rds modify-db-instance \
  --db-instance-identifier cropwise-db \
  --monitoring-interval 60 \
  --monitoring-role-arn arn:aws:iam::123456789012:role/rds-monitoring-role
```

### Automated Deployment Script

```bash
#!/bin/bash
# scripts/aws-deploy.sh

set -e

echo "🚀 Starting CropWise deployment to AWS..."

# Variables
REGION="ap-south-1"
CLUSTER_NAME="cropwise-production"
SERVICE_NAME="cropwise-backend"
TASK_FAMILY="cropwise-backend"
ECR_REGISTRY="123456789012.dkr.ecr.${REGION}.amazonaws.com"
IMAGE_TAG=$(git rev-parse --short HEAD)

# 1. Build Docker image
echo "📦 Building Docker image..."
docker build -t cropwise/backend:${IMAGE_TAG} ./backend

# 2. Login to ECR
echo "🔐 Logging into ECR..."
aws ecr get-login-password --region ${REGION} | \
  docker login --username AWS --password-stdin ${ECR_REGISTRY}

# 3. Tag and push image
echo "⬆️  Pushing image to ECR..."
docker tag cropwise/backend:${IMAGE_TAG} \
  ${ECR_REGISTRY}/cropwise/backend:${IMAGE_TAG}
docker tag cropwise/backend:${IMAGE_TAG} \
  ${ECR_REGISTRY}/cropwise/backend:latest
docker push ${ECR_REGISTRY}/cropwise/backend:${IMAGE_TAG}
docker push ${ECR_REGISTRY}/cropwise/backend:latest

# 4. Update task definition
echo "📝 Updating task definition..."
TASK_DEFINITION=$(aws ecs describe-task-definition \
  --task-definition ${TASK_FAMILY} \
  --query 'taskDefinition' \
  --output json)

NEW_TASK_DEFINITION=$(echo ${TASK_DEFINITION} | \
  jq --arg IMAGE "${ECR_REGISTRY}/cropwise/backend:${IMAGE_TAG}" \
  '.containerDefinitions[0].image = $IMAGE')

aws ecs register-task-definition \
  --cli-input-json "${NEW_TASK_DEFINITION}"

# 5. Update ECS service
echo "🔄 Updating ECS service..."
aws ecs update-service \
  --cluster ${CLUSTER_NAME} \
  --service ${SERVICE_NAME} \
  --task-definition ${TASK_FAMILY} \
  --force-new-deployment

# 6. Wait for deployment
echo "⏳ Waiting for deployment to complete..."
aws ecs wait services-stable \
  --cluster ${CLUSTER_NAME} \
  --services ${SERVICE_NAME}

echo "✅ Deployment complete!"
```

Make script executable:

```bash
chmod +x scripts/aws-deploy.sh
./scripts/aws-deploy.sh
```

---

## 🔐 Environment Configuration

### Production Environment Variables

**Backend (.env.production)**

```bash
# Server
NODE_ENV=production
PORT=8080
API_URL=https://api.cropwise.io

# Database (RDS)
DB_DIALECT=postgres
DB_HOST=cropwise-db.xxxxx.ap-south-1.rds.amazonaws.com
DB_PORT=5432
DB_NAME=cropwise
DB_USER=cropwise
DB_PASSWORD=SECURE_RANDOM_PASSWORD_HERE
DB_SSL=true
DB_LOGGING=false

# Redis (ElastiCache)
REDIS_ENABLED=true
REDIS_HOST=cropwise-redis.xxxxx.cache.amazonaws.com
REDIS_PORT=6379
REDIS_TLS=true

# Authentication
JWT_SECRET=SECURE_RANDOM_SECRET_CHANGE_THIS
JWT_EXPIRES_IN=7d
GOOGLE_CLIENT_ID=your_production_google_client_id
GOOGLE_CLIENT_SECRET=your_production_google_secret

# MQTT (IoT Core)
MQTT_ENABLED=true
MQTT_BROKER_URL=mqtts://xxxxx-ats.iot.ap-south-1.amazonaws.com:8883
MQTT_CLIENT_ID=cropwise_backend_prod

# Twilio (SMS/WhatsApp)
TWILIO_ENABLED=true
TWILIO_ACCOUNT_SID=your_twilio_sid
TWILIO_AUTH_TOKEN=your_twilio_token
TWILIO_PHONE_NUMBER=+1234567890

# Email (SES)
EMAIL_ENABLED=true
SMTP_HOST=email-smtp.ap-south-1.amazonaws.com
SMTP_PORT=587
SMTP_USER=your_ses_smtp_username
SMTP_PASSWORD=your_ses_smtp_password
EMAIL_FROM=noreply@cropwise.io

# Logging
LOG_LEVEL=info
LOG_TO_FILE=true
LOG_TO_CLOUDWATCH=true

# Security
CORS_ORIGIN=https://cropwise.io,https://www.cropwise.io
RATE_LIMIT_WINDOW=15m
RATE_LIMIT_MAX=100

# Monitoring
SENTRY_DSN=https://xxxxx@sentry.io/xxxxx
APM_ENABLED=true
```

**Frontend (.env.production)**

```bash
# API Configuration
VITE_API_URL=https://api.cropwise.io
VITE_API_TIMEOUT=30000

# Google OAuth
VITE_GOOGLE_CLIENT_ID=your_production_google_client_id

# Feature Flags
VITE_ENABLE_IOT=true
VITE_ENABLE_NOTIFICATIONS=true
VITE_ENABLE_ANALYTICS=true

# Monitoring
VITE_SENTRY_DSN=https://xxxxx@sentry.io/xxxxx

# Environment
VITE_ENV=production
```

### Secrets Management

**Using AWS Secrets Manager:**

```bash
# Store database password
aws secretsmanager create-secret \
  --name cropwise/db-password \
  --secret-string "SECURE_PASSWORD"

# Store JWT secret
aws secretsmanager create-secret \
  --name cropwise/jwt-secret \
  --secret-string "$(openssl rand -base64 32)"

# Retrieve secret
aws secretsmanager get-secret-value \
  --secret-id cropwise/db-password \
  --query SecretString \
  --output text
```

**In ECS Task Definition:**

```json
{
  "secrets": [
    {
      "name": "DB_PASSWORD",
      "valueFrom": "arn:aws:secretsmanager:ap-south-1:123456789012:secret:cropwise/db-password"
    },
    {
      "name": "JWT_SECRET",
      "valueFrom": "arn:aws:secretsmanager:ap-south-1:123456789012:secret:cropwise/jwt-secret"
    }
  ]
}
```

---

## 🗄️ Database Setup

### Initial Database Setup

```sql
-- Create database
CREATE DATABASE cropwise;

-- Create user
CREATE USER cropwise WITH ENCRYPTED PASSWORD 'secure_password';

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE cropwise TO cropwise;

-- Connect to database
\c cropwise

-- Grant schema privileges
GRANT ALL ON SCHEMA public TO cropwise;

-- Enable required extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm"; -- For full-text search
CREATE EXTENSION IF NOT EXISTS "btree_gin"; -- For indexes
```

### Run Migrations

```bash
# On production server
cd /app/backend

# Run migrations
NODE_ENV=production npm run migrate

# Verify migrations
npx sequelize-cli db:migrate:status
```

### Database Backup

```bash
# Manual backup
pg_dump -h cropwise-db.xxxxx.ap-south-1.rds.amazonaws.com \
  -U cropwise \
  -d cropwise \
  -F c \
  -f backup_$(date +%Y%m%d_%H%M%S).dump

# Restore backup
pg_restore -h cropwise-db.xxxxx.ap-south-1.rds.amazonaws.com \
  -U cropwise \
  -d cropwise \
  -F c \
  backup_20250115_120000.dump
```

---

## 🔒 SSL/TLS Configuration

### Using Let's Encrypt (Free)

```bash
# Install certbot
sudo apt install certbot python3-certbot-nginx

# Generate certificate
sudo certbot --nginx -d cropwise.io -d www.cropwise.io

# Auto-renewal (cron job)
sudo crontab -e
# Add: 0 0 * * * certbot renew --quiet
```

### Using AWS Certificate Manager (ACM)

```bash
# Request certificate
aws acm request-certificate \
  --domain-name cropwise.io \
  --subject-alternative-names www.cropwise.io api.cropwise.io \
  --validation-method DNS

# Certificate is automatically renewed by AWS
```

---

## 📊 Monitoring & Logging

### CloudWatch Logs

```bash
# View logs
aws logs tail /ecs/cropwise-backend --follow

# Search logs
aws logs filter-log-events \
  --log-group-name /ecs/cropwise-backend \
  --filter-pattern "ERROR"

# Create metric filter
aws logs put-metric-filter \
  --log-group-name /ecs/cropwise-backend \
  --filter-name ErrorCount \
  --filter-pattern "ERROR" \
  --metric-transformations \
    metricName=ErrorCount,metricNamespace=CropWise,metricValue=1
```

### Application Monitoring (Sentry)

```javascript
// backend/src/index.js
const Sentry = require('@sentry/node');

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1,
});

// Error handler
app.use(Sentry.Handlers.errorHandler());
```

### Health Checks

```javascript
// backend/src/routes/health.js
router.get('/health', async (req, res) => {
  try {
    // Check database
    await sequelize.authenticate();
    
    // Check Redis
    await redis.ping();
    
    res.json({
      status: 'healthy',
      timestamp: new Date().toISOString(),
      uptime: process.uptime(),
      database: 'connected',
      cache: 'connected',
    });
  } catch (error) {
    res.status(503).json({
      status: 'unhealthy',
      error: error.message,
    });
  }
});
```

---

## 💾 Backup & Recovery

### Automated Backups

**RDS Automated Backups:**

```bash
# Enable automated backups (already configured)
aws rds modify-db-instance \
  --db-instance-identifier cropwise-db \
  --backup-retention-period 7 \
  --preferred-backup-window "03:00-04:00"

# Create manual snapshot
aws rds create-db-snapshot \
  --db-instance-identifier cropwise-db \
  --db-snapshot-identifier cropwise-backup-$(date +%Y%m%d)

# List snapshots
aws rds describe-db-snapshots \
  --db-instance-identifier cropwise-db
```

**Application Data Backup:**

```bash
#!/bin/bash
# scripts/backup.sh

BACKUP_DIR="/backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
S3_BUCKET="s3://cropwise-backups"

# Backup database
pg_dump -h $DB_HOST -U $DB_USER -d $DB_NAME -F c \
  -f ${BACKUP_DIR}/db_${TIMESTAMP}.dump

# Backup uploaded files
tar -czf ${BACKUP_DIR}/files_${TIMESTAMP}.tar.gz /app/uploads

# Upload to S3
aws s3 cp ${BACKUP_DIR}/db_${TIMESTAMP}.dump ${S3_BUCKET}/database/
aws s3 cp ${BACKUP_DIR}/files_${TIMESTAMP}.tar.gz ${S3_BUCKET}/files/

# Cleanup local backups older than 7 days
find ${BACKUP_DIR} -type f -mtime +7 -delete

echo "✅ Backup completed: ${TIMESTAMP}"
```

### Disaster Recovery

**Recovery Time Objective (RTO):** < 1 hour  
**Recovery Point Objective (RPO):** < 15 minutes

**Recovery Steps:**

```bash
# 1. Restore RDS from snapshot
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier cropwise-db-restored \
  --db-snapshot-identifier cropwise-backup-20250115

# 2. Update ECS service with new DB endpoint
aws ecs update-service \
  --cluster cropwise-production \
  --service cropwise-backend \
  --force-new-deployment

# 3. Verify application health
curl https://api.cropwise.io/health
```

---

## 📈 Scaling

### Horizontal Scaling

**Auto-scaling ECS Service:**

```bash
# Register scalable target
aws application-autoscaling register-scalable-target \
  --service-namespace ecs \
  --resource-id service/cropwise-production/cropwise-backend \
  --scalable-dimension ecs:service:DesiredCount \
  --min-capacity 2 \
  --max-capacity 10

# Create scaling policy (CPU-based)
aws application-autoscaling put-scaling-policy \
  --service-namespace ecs \
  --resource-id service/cropwise-production/cropwise-backend \
  --scalable-dimension ecs:service:DesiredCount \
  --policy-name cpu-scaling-policy \
  --policy-type TargetTrackingScaling \
  --target-tracking-scaling-policy-configuration '{
    "TargetValue": 70.0,
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ECSServiceAverageCPUUtilization"
    },
    "ScaleInCooldown": 300,
    "ScaleOutCooldown": 60
  }'
```

### Vertical Scaling

**Upgrade RDS Instance:**

```bash
# Upgrade to larger instance
aws rds modify-db-instance \
  --db-instance-identifier cropwise-db \
  --db-instance-class db.t3.large \
  --apply-immediately
```

### Database Scaling

**Read Replicas:**

```bash
# Create read replica
aws rds create-db-instance-read-replica \
  --db-instance-identifier cropwise-db-replica \
  --source-db-instance-identifier cropwise-db \
  --db-instance-class db.t3.medium

# Update application to use read replica for read operations
```

---

## 🐛 Troubleshooting

### Common Issues

**Issue: ECS Task Fails to Start**

```bash
# Check task logs
aws ecs describe-tasks \
  --cluster cropwise-production \
  --tasks TASK_ARN

# View CloudWatch logs
aws logs tail /ecs/cropwise-backend --follow

# Common causes:
# 1. Incorrect environment variables
# 2. Database connection failure
# 3. Insufficient task memory/CPU
```

**Issue: Database Connection Timeout**

```bash
# Check security group rules
aws ec2 describe-security-groups --group-ids sg-xxxxx

# Verify database is accessible
telnet cropwise-db.xxxxx.ap-south-1.rds.amazonaws.com 5432

# Check RDS instance status
aws rds describe-db-instances \
  --db-instance-identifier cropwise-db \
  --query 'DBInstances[0].DBInstanceStatus'
```

**Issue: High Memory Usage**

```bash
# Check ECS metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/ECS \
  --metric-name MemoryUtilization \
  --dimensions Name=ClusterName,Value=cropwise-production \
  --start-time 2025-01-15T00:00:00Z \
  --end-time 2025-01-15T23:59:59Z \
  --period 300 \
  --statistics Average

# Increase task memory if needed
```

**Issue: Slow API Response**

```bash
# Enable query logging
export DB_LOGGING=true

# Check slow queries in RDS Performance Insights

# Add database indexes
# See backend/migrations/ for index migrations
```

### Rollback Deployment

```bash
# List previous task definitions
aws ecs list-task-definitions \
  --family-prefix cropwise-backend \
  --sort DESC

# Rollback to previous version
aws ecs update-service \
  --cluster cropwise-production \
  --service cropwise-backend \
  --task-definition cropwise-backend:PREVIOUS_VERSION \
  --force-new-deployment
```

---

## 📞 Support

### Production Support

- **Email**: devops@cropwise.io
- **Slack**: #production-support
- **On-call**: See PagerDuty schedule

### Emergency Procedures

1. **Check Health**: https://status.cropwise.io
2. **View Logs**: AWS CloudWatch
3. **Rollback**: Use previous task definition
4. **Contact**: devops@cropwise.io

---

**Last Updated**: November 2024  
**Version**: 1.0.0  
**Maintainer**: CropWise DevOps Team

