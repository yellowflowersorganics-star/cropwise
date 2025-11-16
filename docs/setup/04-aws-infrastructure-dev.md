# 🏗️ AWS Infrastructure Setup Guide

Complete step-by-step guide to set up AWS infrastructure for CropWise across all environments.

---

## 📋 Overview

You'll create separate AWS resources for **Development**, **Staging**, and **Production** environments:

```
┌──────────────────────────────────────────────────────────────────┐
│                   AWS Infrastructure Per Environment              │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  1. ECR (Docker Registry) - 1 repository with environment tags   │
│     └─ Store backend Docker images (dev, stage, prod tags)      │
│                                                                   │
│  2. ECS Clusters - 3 clusters (dev, stage, prod)                │
│     └─ Run backend containers in isolation                       │
│                                                                   │
│  3. Application Load Balancers - 3 ALBs (dev, stage, prod)      │
│     └─ Route traffic to backend services                         │
│                                                                   │
│  4. S3 Buckets - 3 buckets (dev, stage, prod)                   │
│     └─ Host frontend static files                                │
│                                                                   │
│  5. CloudFront - 3 distributions (dev, stage, prod)              │
│     └─ Deliver frontend globally with CDN                        │
│                                                                   │
│  6. RDS PostgreSQL - 3 databases ✅ (Already done!)              │
│     └─ Separate databases per environment                        │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### **Environment Summary**

| Environment | Purpose | Cost/Month | Uptime SLA |
|-------------|---------|------------|------------|
| **Development** | Testing & development | ~$50 | No SLA (can stop anytime) |
| **Staging** | Pre-production testing | ~$80 | 95% (business hours) |
| **Production** | Live application | ~$150-200 | 99.9% (24/7) |

**Total Estimated Cost**: ~$280-330/month for all 3 environments  
**Estimated Setup Time**: 2-3 hours for all environments

---

## 🚀 STEP 1: Create ECR Repository

**What it is**: Docker registry to store your backend images

**Strategy**: Use **ONE repository** with **environment-specific tags** (dev, stage, prod)

### **1.1 Go to ECR Console**
👉 https://console.aws.amazon.com/ecr/repositories

### **1.2 Create Repository**

1. Click **"Create repository"**
2. Configure:
   - **Visibility**: Private
   - **Repository name**: `cropwise-backend`
   - **Tag immutability**: Disabled (allows tag updates)
   - **Scan on push**: Enabled (automatic security scanning)
   - **Encryption**: AES-256 (default)
3. Click **"Create repository"**

### **1.3 Note the Repository URI**

You'll see something like:
```
123456789012.dkr.ecr.ap-south-1.amazonaws.com/cropwise-backend
```

**Save this!** You'll use it with different tags:
```
# Development
123456789012.dkr.ecr.ap-south-1.amazonaws.com/cropwise-backend:dev

# Staging
123456789012.dkr.ecr.ap-south-1.amazonaws.com/cropwise-backend:stage

# Production (versioned)
123456789012.dkr.ecr.ap-south-1.amazonaws.com/cropwise-backend:v1.0.0
123456789012.dkr.ecr.ap-south-1.amazonaws.com/cropwise-backend:prod (latest)
```

### **1.4 Set Up Lifecycle Policy (Optional but Recommended)**

Keep your ECR storage costs low by auto-deleting old images:

1. Select your repository: `cropwise-backend`
2. Click **Lifecycle Policy** → **Create rule**
3. Add rules:

**Rule 1: Keep last 10 dev images**
```json
{
  "rulePriority": 1,
  "description": "Keep last 10 dev images",
  "selection": {
    "tagStatus": "tagged",
    "tagPrefixList": ["dev"],
    "countType": "imageCountMoreThan",
    "countNumber": 10
  },
  "action": {
    "type": "expire"
  }
}
```

**Rule 2: Keep last 10 stage images**
```json
{
  "rulePriority": 2,
  "description": "Keep last 10 stage images",
  "selection": {
    "tagStatus": "tagged",
    "tagPrefixList": ["stage"],
    "countType": "imageCountMoreThan",
    "countNumber": 10
  },
  "action": {
    "type": "expire"
  }
}
```

**Rule 3: Keep all production images (v*)**
- Don't expire production images - keep for rollback

4. Click **Save**

---

## 🐳 STEP 2: Create ECS Clusters

**What it is**: Container orchestration service to run your backend

**Strategy**: Create **3 separate clusters** for complete environment isolation

### **2.1 Go to ECS Console**
👉 https://console.aws.amazon.com/ecs/home#/clusters

---

### **2.2 Create Development Cluster**

1. Click **"Create cluster"**
2. Configure:
   - **Cluster name**: `cropwise-dev-cluster`
   - **Infrastructure**: AWS Fargate (serverless, no EC2 to manage)
   - **Monitoring**: ⭕ Disable Container Insights (save cost for dev)
   - **Tags**: 
     - Key: `Environment`, Value: `development`
     - Key: `Project`, Value: `cropwise`
3. Click **"Create"**

**Wait**: 30 seconds for cluster to be created

---

### **2.3 Create Staging Cluster**

1. Click **"Create cluster"** again
2. Configure:
   - **Cluster name**: `cropwise-stage-cluster`
   - **Infrastructure**: AWS Fargate
   - **Monitoring**: ✅ Enable Container Insights (useful for debugging)
   - **Tags**: 
     - Key: `Environment`, Value: `staging`
     - Key: `Project`, Value: `cropwise`
3. Click **"Create"**

---

### **2.4 Create Production Cluster**

1. Click **"Create cluster"** again
2. Configure:
   - **Cluster name**: `cropwise-prod-cluster`
   - **Infrastructure**: AWS Fargate
   - **Monitoring**: ✅ Enable Container Insights (essential for production)
   - **Tags**: 
     - Key: `Environment`, Value: `production`
     - Key: `Project`, Value: `cropwise`
     - Key: `CostCenter`, Value: `cropwise-prod`
3. Click **"Create"**

---

### **2.5 Verify All Clusters**

You should now see 3 clusters:
```
✅ cropwise-dev-cluster    (Active)
✅ cropwise-stage-cluster  (Active)
✅ cropwise-prod-cluster   (Active)
```

**Cluster Isolation Benefits:**
- Complete separation between environments
- Different IAM permissions per cluster
- Independent scaling and monitoring
- No accidental deployments to wrong environment

---

## ⚖️ STEP 3: Create Application Load Balancers

**What it is**: Distributes traffic to your backend containers

**Strategy**: Create **3 separate ALBs** for each environment

### **3.1 Go to EC2 Console - Load Balancers**
👉 https://console.aws.amazon.com/ec2/home#LoadBalancers

---

### **3.2 Create Development Load Balancer**

#### **Step 1: Create Target Group First**

1. Go to: https://console.aws.amazon.com/ec2/home#TargetGroups
2. Click **"Create target group"**
3. Configure:
   - **Target type**: IP addresses (for AWS Fargate)
   - **Target group name**: `cropwise-dev-backend-tg`
   - **Protocol**: HTTP
   - **Port**: 3000 (your backend port)
   - **VPC**: Default VPC (or your custom VPC)
   - **Protocol version**: HTTP1
   
4. **Health check settings**:
   - **Health check protocol**: HTTP
   - **Health check path**: `/health` (or `/api/health`)
   - **Healthy threshold**: 2
   - **Unhealthy threshold**: 2
   - **Timeout**: 5 seconds
   - **Interval**: 30 seconds
   
5. **Tags**:
   - Key: `Environment`, Value: `development`
   - Key: `Project`, Value: `cropwise`

6. Click **"Next"** → **"Create target group"** (don't register targets yet)

#### **Step 2: Create Load Balancer**

1. Go back to: https://console.aws.amazon.com/ec2/home#LoadBalancers
2. Click **"Create load balancer"**
3. Select **"Application Load Balancer"** → **"Create"**

4. **Basic Configuration:**
   - **Name**: `cropwise-dev-alb`
   - **Scheme**: Internet-facing
   - **IP address type**: IPv4

5. **Network Mapping:**
   - **VPC**: Default VPC (same as target group)
   - **Mappings**: Select **at least 2 availability zones**
     - ✅ ap-south-1a
     - ✅ ap-south-1b
     - ⭕ ap-south-1c (optional)

6. **Security Groups:**
   - Click **"Create new security group"**
   - **Security group name**: `cropwise-dev-alb-sg`
   - **Description**: CropWise Development ALB
   - **Inbound rules**:
     ```
     Type: HTTP
     Protocol: TCP
     Port: 80
     Source: 0.0.0.0/0 (Anywhere)
     Description: Allow HTTP from internet
     ```
   - Click **"Create security group"**
   - Go back to ALB creation and select the new security group

7. **Listeners and routing:**
   - **Protocol**: HTTP
   - **Port**: 80
   - **Default action**: Forward to `cropwise-dev-backend-tg`

8. **Tags**:
   - Key: `Environment`, Value: `development`
   - Key: `Project`, Value: `cropwise`

9. Click **"Create load balancer"**

#### **Step 3: Wait and Get DNS Name**

- Wait 2-5 minutes for status to become **Active**
- Note the DNS name:
  ```
  cropwise-dev-alb-1234567890.ap-south-1.elb.amazonaws.com
  ```
- **This is your development backend API URL!**

---

### **3.3 Create Staging Load Balancer**

Repeat the process for staging:

#### **Target Group:**
- Name: `cropwise-stage-backend-tg`
- Port: 3000
- Health check path: `/health`
- Tags: `Environment=staging`

#### **Load Balancer:**
- Name: `cropwise-stage-alb`
- Scheme: Internet-facing
- Security group: Create `cropwise-stage-alb-sg` (same inbound rules)
- Listener: HTTP:80 → `cropwise-stage-backend-tg`
- Tags: `Environment=staging`

#### **DNS Name:**
```
cropwise-stage-alb-9876543210.ap-south-1.elb.amazonaws.com
```

---

### **3.4 Create Production Load Balancer**

#### **Target Group:**
- Name: `cropwise-prod-backend-tg`
- Port: 3000
- Health check path: `/health`
- **Health check settings** (stricter for production):
  - Healthy threshold: 3
  - Unhealthy threshold: 2
  - Timeout: 5 seconds
  - Interval: 30 seconds
- Tags: `Environment=production`

#### **Load Balancer:**
- Name: `cropwise-prod-alb`
- Scheme: Internet-facing
- **Network mapping**: All 3 availability zones (for high availability)
  - ✅ ap-south-1a
  - ✅ ap-south-1b
  - ✅ ap-south-1c
- Security group: Create `cropwise-prod-alb-sg`
  - **Inbound rules**:
    ```
    HTTP (80) from 0.0.0.0/0
    HTTPS (443) from 0.0.0.0/0 (for production SSL)
    ```
- **Enable access logs** (production best practice):
  - Create S3 bucket: `cropwise-prod-alb-logs`
  - Enable access logs to this bucket
- **Enable deletion protection** (prevent accidental deletion)
- Listener: HTTP:80 → `cropwise-prod-backend-tg`
- Tags: `Environment=production`, `CostCenter=cropwise-prod`

#### **DNS Name:**
```
cropwise-prod-alb-5555555555.ap-south-1.elb.amazonaws.com
```

---

### **3.5 Verify All Load Balancers**

You should now have 3 ALBs:
```
✅ cropwise-dev-alb    (Active) → Dev backend
✅ cropwise-stage-alb  (Active) → Stage backend
✅ cropwise-prod-alb   (Active) → Prod backend
```

### **3.6 Save All DNS Names**

Add these to your GitHub Secrets or `.env`:
```bash
# Development
BACKEND_URL_DEV=http://cropwise-dev-alb-XXX.ap-south-1.elb.amazonaws.com

# Staging
BACKEND_URL_STAGE=http://cropwise-stage-alb-XXX.ap-south-1.elb.amazonaws.com

# Production
BACKEND_URL_PROD=http://cropwise-prod-alb-XXX.ap-south-1.elb.amazonaws.com
```

**Note**: For production, you'll want to add a custom domain and SSL (covered in a separate guide)

---

## 🚀 STEP 4: Create ECS Task Definition

**What it is**: Blueprint for running your backend container

### **4.1 Go to ECS Console - Task Definitions**
👉 https://console.aws.amazon.com/ecs/home#/taskDefinitions

### **4.2 Create Task Definition**

1. Click **"Create new task definition"**
2. **Task definition family**: `cropwise-backend-dev`
3. **Launch type**: AWS Fargate

**Infrastructure (Development):**
- **Operating system**: Linux/X86_64
- **CPU**: 0.25 vCPU (256) - Smaller for dev
- **Memory**: 512 MB - Smaller for dev

**Container Configuration:**
- **Container name**: `cropwise-backend-dev`
- **Image URI**: Your ECR URI from Step 1.4
  ```
  123456789012.dkr.ecr.ap-south-1.amazonaws.com/cropwise-backend:dev
  ```
- **Port mappings**: 3000 (TCP)

**Environment Variables:**
Click "Add" for each:
```
NODE_ENV = development
DATABASE_URL = (Use DEV database value from GitHub Secret)
JWT_SECRET = (Use DEV value from GitHub Secret)
SESSION_SECRET = (Use DEV value from GitHub Secret)
GOOGLE_CLIENT_ID = (Use value from GitHub Secret)
GOOGLE_CLIENT_SECRET = (Use value from GitHub Secret)
REDIS_URL = redis://cropwise-dev-redis:6379
MQTT_BROKER_URL = mqtt://cropwise-dev-mqtt:1883
```

**Logging:**
- Enable CloudWatch Logs
- Log group: `/ecs/cropwise-backend-dev`

4. Click **"Create"**

### **4.3 Create Staging Task Definition**

Repeat the process with staging-specific values:

1. Go to: https://console.aws.amazon.com/ecs/home#/taskDefinitions
2. Click **"Create new task definition"**
3. Family name: `cropwise-backend-stage`

**Infrastructure (Staging):**
- **Operating system**: Linux/X86_64
- **CPU**: 0.5 vCPU (512) - Medium for staging
- **Memory**: 1 GB - Medium for staging

**Container Configuration:**
- **Container name**: `cropwise-backend-stage`
- **Image URI**: 
  ```
  123456789012.dkr.ecr.ap-south-1.amazonaws.com/cropwise-backend:stage
  ```
- **Port mappings**: 3000 (TCP)

**Environment Variables:**
```
NODE_ENV = staging
DATABASE_URL = (Use STAGING database value from GitHub Secret)
JWT_SECRET = (Use STAGING value from GitHub Secret)
SESSION_SECRET = (Use STAGING value from GitHub Secret)
GOOGLE_CLIENT_ID = (Use value from GitHub Secret)
GOOGLE_CLIENT_SECRET = (Use value from GitHub Secret)
REDIS_URL = redis://cropwise-stage-redis:6379
MQTT_BROKER_URL = mqtt://cropwise-stage-mqtt:1883
```

**Logging:**
- Log group: `/ecs/cropwise-backend-stage`

### **4.4 Create Production Task Definition**

Repeat the process with production-specific values:

1. Go to: https://console.aws.amazon.com/ecs/home#/taskDefinitions
2. Click **"Create new task definition"**
3. Family name: `cropwise-backend-prod`

**Infrastructure (Production):**
- **Operating system**: Linux/X86_64
- **CPU**: 1 vCPU (1024) - Larger for production
- **Memory**: 2 GB - Larger for production

**Container Configuration:**
- **Container name**: `cropwise-backend-prod`
- **Image URI**: Use version tags for production
  ```
  123456789012.dkr.ecr.ap-south-1.amazonaws.com/cropwise-backend:v1.0.0
  # Or use 'prod' tag for latest production build
  ```
- **Port mappings**: 3000 (TCP)

**Environment Variables:**
```
NODE_ENV = production
DATABASE_URL = (Use PRODUCTION database value from GitHub Secret)
JWT_SECRET = (Use PRODUCTION value from GitHub Secret)
SESSION_SECRET = (Use PRODUCTION value from GitHub Secret)
GOOGLE_CLIENT_ID = (Use value from GitHub Secret)
GOOGLE_CLIENT_SECRET = (Use value from GitHub Secret)
REDIS_URL = redis://cropwise-prod-redis:6379
MQTT_BROKER_URL = mqtt://cropwise-prod-mqtt:1883
```

**Logging:**
- Log group: `/ecs/cropwise-backend-prod`

**Important Notes:**
- Each environment has its own task definition family
- Use environment-specific container names to avoid conflicts
- Use different image tags (dev/stage/prod or version numbers)
- Each environment has separate environment variables and secrets
- Production uses more resources (CPU/Memory) than dev/staging

---

## 🎯 STEP 5: Create ECS Services

**What it is**: Runs and maintains your containers in each cluster

**Strategy**: Create **3 separate services** in their respective clusters

---

### **5.1 Create Development Service**

1. Go to: https://console.aws.amazon.com/ecs/home#/clusters
2. Click: **`cropwise-dev-cluster`**
3. Click **"Create service"**

#### **Configure Development Service:**

**Environment:**
- **Compute options**: Launch type
- **Launch type**: Fargate

**Deployment Configuration:**
- **Application type**: Service
- **Task definition**: 
  - **Family**: `cropwise-backend-dev`
  - **Revision**: Latest
- **Service name**: `cropwise-backend-dev-service`
- **Desired tasks**: **1** (one container for dev)

**Networking:**
- **VPC**: Default VPC (same as ALB)
- **Subnets**: Select all available subnets
- **Security group**: 
  - Create new: `cropwise-dev-ecs-sg`
  - **Inbound rules**:
    ```
    Type: Custom TCP
    Protocol: TCP
    Port: 3000
    Source: [cropwise-dev-alb-sg security group ID]
    Description: Allow traffic from dev ALB
    ```
  - **Outbound rules**: All traffic (for database, internet access)

**Load Balancing:**
- **Load balancer type**: Application Load Balancer
- **Container to load balance**: `cropwise-backend-dev:3000`
- **Use an existing load balancer**: `cropwise-dev-alb`
- **Target group**: Use existing → `cropwise-dev-backend-tg`
- **Health check grace period**: 60 seconds

**Auto Scaling:**
- ⭕ **Disable** for development (not needed)

**Tags:**
- Key: `Environment`, Value: `development`
- Key: `Project`, Value: `cropwise`

4. Click **"Create"**

#### **Wait and Verify**

- Wait 3-5 minutes for service to start
- Status should show: **1/1 tasks running**
- Check **Tasks** tab → Click task → **Logs** tab for startup messages

#### **Test Development Backend**

```powershell
# Test health endpoint
curl http://cropwise-dev-alb-XXXXXXXXXX.ap-south-1.elb.amazonaws.com/health
```

Expected response:
```json
{
  "status": "ok",
  "environment": "development",
  "database": "connected"
}
```

---

### **5.2 Create Staging Service**

1. Go to: https://console.aws.amazon.com/ecs/home#/clusters
2. Click: **`cropwise-stage-cluster`**
3. Click **"Create service"**

#### **Configure Staging Service:**

**Environment:**
- **Launch type**: Fargate

**Deployment Configuration:**
- **Task definition**: `cropwise-backend-stage` (Latest)
- **Service name**: `cropwise-backend-stage-service`
- **Desired tasks**: **1** (can increase for load testing)

**Networking:**
- **VPC**: Default VPC
- **Subnets**: Select all
- **Security group**: Create `cropwise-stage-ecs-sg`
  - Inbound: Port 3000 from `cropwise-stage-alb-sg`

**Load Balancing:**
- **Load balancer**: `cropwise-stage-alb`
- **Target group**: `cropwise-stage-backend-tg`
- **Health check grace period**: 60 seconds

**Auto Scaling:**
- ⭕ **Optional** for staging
- If enabled:
  - Min: 1, Max: 2
  - Target CPU: 70%

**Tags:**
- Key: `Environment`, Value: `staging`

4. Click **"Create"**

---

### **5.3 Create Production Service**

1. Go to: https://console.aws.amazon.com/ecs/home#/clusters
2. Click: **`cropwise-prod-cluster`**
3. Click **"Create service"**

#### **Configure Production Service:**

**Environment:**
- **Launch type**: Fargate

**Deployment Configuration:**
- **Task definition**: `cropwise-backend-prod` (Latest or specific version)
- **Service name**: `cropwise-backend-prod-service`
- **Desired tasks**: **2** (minimum for high availability)

**Deployment options:**
- **Deployment type**: Rolling update
- **Minimum healthy percent**: 100
- **Maximum percent**: 200
- **Deployment circuit breaker**: ✅ Enable (with rollback)

**Networking:**
- **VPC**: Default VPC
- **Subnets**: Select all (for high availability across AZs)
- **Security group**: Create `cropwise-prod-ecs-sg`
  - **Inbound rules**:
    ```
    Type: Custom TCP
    Port: 3000
    Source: [cropwise-prod-alb-sg]
    Description: Allow traffic from prod ALB only
    ```

**Load Balancing:**
- **Load balancer**: `cropwise-prod-alb`
- **Target group**: `cropwise-prod-backend-tg`
- **Health check grace period**: 90 seconds (more time for prod startup)

**Auto Scaling** (Recommended for Production):**
- ✅ **Enable**
- **Minimum tasks**: 2
- **Maximum tasks**: 10
- **Scaling policy**:
  - **Target tracking**: ECS service average CPU utilization
  - **Target value**: 70%
  - **Scale-in cooldown**: 300 seconds
  - **Scale-out cooldown**: 60 seconds

**Service Discovery (Optional):**
- Enable if using service mesh or microservices

**Tags:**
- Key: `Environment`, Value: `production`
- Key: `CostCenter`, Value: `cropwise-prod`

4. Click **"Create"**

---

### **5.4 Verify All Services**

You should now have 3 services running:

```
✅ cropwise-dev-cluster
   └─ cropwise-backend-dev-service (1/1 tasks running)

✅ cropwise-stage-cluster
   └─ cropwise-backend-stage-service (1/1 tasks running)

✅ cropwise-prod-cluster
   └─ cropwise-backend-prod-service (2/2 tasks running)
```

---

### **5.5 Test All Backends**

```powershell
# Test Development
curl http://cropwise-dev-alb-XXX.ap-south-1.elb.amazonaws.com/health

# Test Staging
curl http://cropwise-stage-alb-XXX.ap-south-1.elb.amazonaws.com/health

# Test Production
curl http://cropwise-prod-alb-XXX.ap-south-1.elb.amazonaws.com/health
```

Expected responses should show correct environment:
```json
{
  "status": "ok",
  "environment": "development|staging|production",
  "database": "connected",
  "version": "1.0.0"
}
```

---

## 📦 STEP 6: Create S3 Buckets for Frontend

**What it is**: Storage for your frontend static files (React, Vue, Angular, etc.)

**Strategy**: Create **3 separate S3 buckets** for complete isolation

### **6.1 Go to S3 Console**
👉 https://s3.console.aws.amazon.com/s3/buckets

---

### **6.2 Create Development Bucket**

1. Click **"Create bucket"**
2. Configure:
   - **Bucket name**: `cropwise-dev-frontend` (must be globally unique)
     - If taken, try: `cropwise-dev-frontend-2025` or `cropwise-dev-YOUR_COMPANY`
   - **AWS Region**: ap-south-1 (same as other resources)
   - **Object Ownership**: ACLs disabled (recommended)
   - **Block Public Access settings**: 
     - ⚠️ **UNCHECK** "Block all public access" (CloudFront needs access)
     - Acknowledge the warning
   - **Bucket Versioning**: ⭕ Disable (not critical for dev)
   - **Default encryption**: ✅ Enable (SSE-S3)
   - **Tags**:
     - Key: `Environment`, Value: `development`
     - Key: `Project`, Value: `cropwise`
3. Click **"Create bucket"**

#### **Enable Static Website Hosting:**

1. Click your bucket: `cropwise-dev-frontend`
2. Go to **Properties** tab
3. Scroll to **Static website hosting** → Click **"Edit"**
4. Configure:
   - ✅ **Enable**
   - **Hosting type**: Host a static website
   - **Index document**: `index.html`
   - **Error document**: `index.html` (for SPA routing - React/Vue/Angular)
5. Click **"Save changes"**
6. Note the **Bucket website endpoint**: 
   ```
   http://cropwise-dev-frontend.s3-website-ap-south-1.amazonaws.com
   ```

#### **Configure Bucket Policy:**

1. Go to **Permissions** tab
2. **Bucket policy** → Click **"Edit"**
3. Paste this policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::cropwise-dev-frontend/*"
    }
  ]
}
```

4. **Important**: Replace `cropwise-dev-frontend` with YOUR actual bucket name
5. Click **"Save changes"**

---

### **6.3 Create Staging Bucket**

Repeat for staging:

1. Click **"Create bucket"**
2. Configure:
   - **Bucket name**: `cropwise-stage-frontend`
   - **Region**: ap-south-1
   - **Block Public Access**: UNCHECK all
   - **Bucket Versioning**: ⭕ Disable
   - **Encryption**: ✅ Enable (SSE-S3)
   - **Tags**: `Environment=staging`
3. **Enable Static Website Hosting**:
   - Index: `index.html`
   - Error: `index.html`
4. **Bucket Policy**: Same as above, replace bucket name with `cropwise-stage-frontend`

---

### **6.4 Create Production Bucket**

Production bucket with best practices:

1. Click **"Create bucket"**
2. Configure:
   - **Bucket name**: `cropwise-prod-frontend`
   - **Region**: ap-south-1
   - **Block Public Access**: UNCHECK all
   - **Bucket Versioning**: ✅ **Enable** (important for rollback)
   - **Default encryption**: ✅ Enable (SSE-S3 or SSE-KMS for extra security)
   - **Object Lock**: ⭕ Disable (unless required for compliance)
   - **Tags**: `Environment=production`, `CostCenter=cropwise-prod`
3. **Enable Static Website Hosting**:
   - Index: `index.html`
   - Error: `index.html`
4. **Bucket Policy**:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::cropwise-prod-frontend/*"
    }
  ]
}
```

---

### **6.5 Verify All Buckets**

You should now have 3 S3 buckets:

```
✅ cropwise-dev-frontend   (Static website hosting enabled)
✅ cropwise-stage-frontend (Static website hosting enabled)
✅ cropwise-prod-frontend  (Static website hosting enabled, versioning enabled)
```

### **6.6 Test Direct S3 Access** (Optional)

Upload a test `index.html` to verify:

```powershell
# Create test file
echo "<h1>CropWise Dev Frontend</h1>" > index.html

# Upload to dev bucket
aws s3 cp index.html s3://cropwise-dev-frontend/

# Test in browser:
# http://cropwise-dev-frontend.s3-website-ap-south-1.amazonaws.com
```

---

## 🌐 STEP 7: Create CloudFront Distributions

**What it is**: Global CDN (Content Delivery Network) for fast frontend delivery worldwide

**Strategy**: Create **3 separate CloudFront distributions** for each S3 bucket

### **7.1 Go to CloudFront Console**
👉 https://console.aws.amazon.com/cloudfront/home

---

### **7.2 Create Development Distribution**

1. Click **"Create distribution"**

#### **Origin Settings:**
- **Origin domain**: Select from dropdown:
  - `cropwise-dev-frontend.s3.ap-south-1.amazonaws.com`
- **Origin path**: Leave empty
- **Name**: Auto-generated (fine as-is)
- **Origin access**: Public (simpler for dev)
- **Enable Origin Shield**: ⭕ No (not needed for dev)

#### **Default Cache Behavior:**
- **Path pattern**: Default (*)
- **Compress objects automatically**: ✅ Yes
- **Viewer protocol policy**: **Redirect HTTP to HTTPS**
- **Allowed HTTP methods**: GET, HEAD, OPTIONS
- **Restrict viewer access**: ⭕ No
- **Cache key and origin requests**:
  - **Cache policy**: CachingOptimized
  - **Origin request policy**: None (or CORS-S3Origin if using API calls)
- **Response headers policy**: None

#### **Function Associations:**
- Leave empty (no edge functions for now)

#### **Settings:**
- **Price class**: Use all edge locations (or "US, Canada, Europe" to save cost)
- **AWS WAF web ACL**: ⭕ Do not enable (not needed for dev)
- **Alternate domain name (CNAME)**: Leave empty (use CloudFront URL)
- **Custom SSL certificate**: Default CloudFront Certificate (*.cloudfront.net)
- **Supported HTTP versions**: HTTP/2, HTTP/3
- **Default root object**: `index.html`
- **Standard logging**: ⭕ Off (save cost for dev)
- **IPv6**: ✅ On
- **Description**: CropWise Development Frontend
- **Tags**:
  - Key: `Environment`, Value: `development`
  - Key: `Project`, Value: `cropwise`

2. Click **"Create distribution"**

#### **Wait for Deployment:**
- Status will show **"Deploying"** for 5-15 minutes
- When complete, status changes to **"Enabled"**
- **Do not close the browser!** You need the distribution details

#### **Get Your Frontend URL:**

You'll see the **Distribution domain name**:
```
d1a2b3c4d5e6f7.cloudfront.net
```

**This is your DEV frontend URL!**
```
https://d1a2b3c4d5e6f7.cloudfront.net
```

#### **Add Error Pages (Critical for SPAs - React/Vue/Angular):**

1. Click your distribution ID
2. Go to **Error pages** tab
3. Click **"Create custom error response"**
4. Configure for 403 errors:
   - **HTTP error code**: 403 (Forbidden)
   - **Customize error response**: Yes
   - **Response page path**: `/index.html`
   - **HTTP response code**: 200 (OK)
   - Click **"Create custom error response"**

5. Click **"Create custom error response"** again
6. Configure for 404 errors:
   - **HTTP error code**: 404 (Not Found)
   - **Customize error response**: Yes
   - **Response page path**: `/index.html`
   - **HTTP response code**: 200 (OK)
   - Click **"Create custom error response"**

⚠️ **Why?** SPAs use client-side routing. Without this, refreshing `/dashboard` returns 404. This fix redirects all 404s to index.html, letting React Router handle routing.

#### **Copy Distribution ID:**

Find the **Distribution ID** (e.g., `E1234ABCDEFGH`) and save it.

---

### **7.3 Create Staging Distribution**

Repeat for staging:

1. Click **"Create distribution"**
2. **Origin domain**: `cropwise-stage-frontend.s3.ap-south-1.amazonaws.com`
3. **Settings**: Same as development
   - **Description**: CropWise Staging Frontend
   - **Tags**: `Environment=staging`
4. **Create** → Wait 5-15 minutes
5. **Error pages**: Add custom error responses for 403 and 404
6. **Distribution domain**: 
   ```
   d9x8y7z6w5v4u3.cloudfront.net
   ```
7. **Distribution ID**: `E5678MNOPQRST`

---

### **7.4 Create Production Distribution**

Production with enhanced settings:

1. Click **"Create distribution"**
2. **Origin domain**: `cropwise-prod-frontend.s3.ap-south-1.amazonaws.com`

#### **Production-Specific Settings:**

- **Price class**: Use all edge locations (best performance)
- **AWS WAF web ACL**: ✅ Consider enabling for DDoS protection (costs extra)
- **Standard logging**: ✅ **Enable**
  - **S3 bucket**: Create new `cropwise-prod-cloudfront-logs`
  - **Log prefix**: `cloudfront/`
  - **Cookie logging**: Off
- **Description**: CropWise Production Frontend
- **Tags**: `Environment=production`, `CostCenter=cropwise-prod`

3. **Create** → Wait 10-20 minutes (production may take longer)
4. **Error pages**: Add custom error responses for 403 and 404 (critical!)
5. **Distribution domain**:
   ```
   d3t2s1r0q9p8o7.cloudfront.net
   ```
6. **Distribution ID**: `E9012UVWXYZAB`

---

### **7.5 Verify All Distributions**

You should now have 3 CloudFront distributions:

```
✅ Development
   Distribution ID: E1234ABCDEFGH
   URL: https://d1a2b3c4d5e6f7.cloudfront.net
   Origin: cropwise-dev-frontend

✅ Staging
   Distribution ID: E5678MNOPQRST
   URL: https://d9x8y7z6w5v4u3.cloudfront.net
   Origin: cropwise-stage-frontend

✅ Production
   Distribution ID: E9012UVWXYZAB
   URL: https://d3t2s1r0q9p8o7.cloudfront.net
   Origin: cropwise-prod-frontend
```

---

### **7.6 Test CloudFront Distributions**

Upload a test file and verify CDN caching:

```powershell
# Create test index.html
echo "<h1>CropWise Dev - It Works!</h1>" > index.html

# Upload to dev bucket
aws s3 cp index.html s3://cropwise-dev-frontend/ --cache-control "max-age=300"

# Test CloudFront URL (wait 60 seconds for propagation)
# Open in browser:
# https://d1a2b3c4d5e6f7.cloudfront.net
```

Expected: You should see "CropWise Dev - It Works!"

---

### **7.7 Invalidate Cache (When Deploying)**

After deploying new frontend code, invalidate CloudFront cache:

```powershell
# Invalidate development
aws cloudfront create-invalidation \
  --distribution-id E1234ABCDEFGH \
  --paths "/*"

# Invalidate staging
aws cloudfront create-invalidation \
  --distribution-id E5678MNOPQRST \
  --paths "/*"

# Invalidate production
aws cloudfront create-invalidation \
  --distribution-id E9012UVWXYZAB \
  --paths "/*"
```

**Note**: First 1,000 invalidations per month are free, then $0.005 per path.

---

## ✅ STEP 8: Update GitHub Secrets

Add all environment-specific secrets to GitHub Actions:

👉 https://github.com/yellowflowersorganics-star/cropwise/settings/secrets/actions

### **ECR Repository**

```
Name: AWS_ECR_REPOSITORY
Value: cropwise-backend
```

### **Development Environment Secrets**

```
Name: BACKEND_URL_DEV
Value: http://cropwise-dev-alb-XXXXXXXXXX.ap-south-1.elb.amazonaws.com

Name: FRONTEND_URL_DEV
Value: https://d1a2b3c4d5e6f7.cloudfront.net

Name: CLOUDFRONT_DISTRIBUTION_ID_DEV
Value: E1234ABCDEFGH

Name: S3_BUCKET_DEV
Value: cropwise-dev-frontend

Name: ECS_CLUSTER_DEV
Value: cropwise-dev-cluster

Name: ECS_SERVICE_DEV
Value: cropwise-backend-dev-service

Name: ECS_TASK_DEFINITION_DEV
Value: cropwise-backend-dev
```

### **Staging Environment Secrets**

```
Name: BACKEND_URL_STAGE
Value: http://cropwise-stage-alb-YYYYYYYYYY.ap-south-1.elb.amazonaws.com

Name: FRONTEND_URL_STAGE
Value: https://d9x8y7z6w5v4u3.cloudfront.net

Name: CLOUDFRONT_DISTRIBUTION_ID_STAGE
Value: E5678MNOPQRST

Name: S3_BUCKET_STAGE
Value: cropwise-stage-frontend

Name: ECS_CLUSTER_STAGE
Value: cropwise-stage-cluster

Name: ECS_SERVICE_STAGE
Value: cropwise-backend-stage-service

Name: ECS_TASK_DEFINITION_STAGE
Value: cropwise-backend-stage
```

### **Production Environment Secrets**

```
Name: BACKEND_URL_PROD
Value: http://cropwise-prod-alb-ZZZZZZZZZZ.ap-south-1.elb.amazonaws.com

Name: FRONTEND_URL_PROD
Value: https://d3t2s1r0q9p8o7.cloudfront.net

Name: CLOUDFRONT_DISTRIBUTION_ID_PROD
Value: E9012UVWXYZAB

Name: S3_BUCKET_PROD
Value: cropwise-prod-frontend

Name: ECS_CLUSTER_PROD
Value: cropwise-prod-cluster

Name: ECS_SERVICE_PROD
Value: cropwise-backend-prod-service

Name: ECS_TASK_DEFINITION_PROD
Value: cropwise-backend-prod
```

### **Database URLs (Already Added)**

```
✅ DATABASE_URL_DEV   (from RDS setup)
✅ DATABASE_URL_STAGE (from RDS setup)
✅ DATABASE_URL_PROD  (from RDS setup)
```

### **Other Secrets (Already Added)**

```
✅ AWS_REGION (ap-south-1)
✅ AWS_ACCESS_KEY_ID
✅ AWS_SECRET_ACCESS_KEY
✅ JWT_SECRET
✅ SESSION_SECRET
✅ GOOGLE_CLIENT_ID
✅ GOOGLE_CLIENT_SECRET
```

---

### **Quick Add Script**

Run this in PowerShell (replace values with your actual resources):

```powershell
# Set your repository
$REPO = "yellowflowersorganics-star/cropwise"

# Development
gh secret set BACKEND_URL_DEV --body "http://YOUR-DEV-ALB.elb.amazonaws.com" --repo $REPO
gh secret set FRONTEND_URL_DEV --body "https://YOUR-DEV-CLOUDFRONT.cloudfront.net" --repo $REPO
gh secret set CLOUDFRONT_DISTRIBUTION_ID_DEV --body "YOUR-DEV-DIST-ID" --repo $REPO
gh secret set S3_BUCKET_DEV --body "cropwise-dev-frontend" --repo $REPO
gh secret set ECS_CLUSTER_DEV --body "cropwise-dev-cluster" --repo $REPO
gh secret set ECS_SERVICE_DEV --body "cropwise-backend-dev-service" --repo $REPO
gh secret set ECS_TASK_DEFINITION_DEV --body "cropwise-backend-dev" --repo $REPO

# Staging
gh secret set BACKEND_URL_STAGE --body "http://YOUR-STAGE-ALB.elb.amazonaws.com" --repo $REPO
gh secret set FRONTEND_URL_STAGE --body "https://YOUR-STAGE-CLOUDFRONT.cloudfront.net" --repo $REPO
gh secret set CLOUDFRONT_DISTRIBUTION_ID_STAGE --body "YOUR-STAGE-DIST-ID" --repo $REPO
gh secret set S3_BUCKET_STAGE --body "cropwise-stage-frontend" --repo $REPO
gh secret set ECS_CLUSTER_STAGE --body "cropwise-stage-cluster" --repo $REPO
gh secret set ECS_SERVICE_STAGE --body "cropwise-backend-stage-service" --repo $REPO
gh secret set ECS_TASK_DEFINITION_STAGE --body "cropwise-backend-stage" --repo $REPO

# Production
gh secret set BACKEND_URL_PROD --body "http://YOUR-PROD-ALB.elb.amazonaws.com" --repo $REPO
gh secret set FRONTEND_URL_PROD --body "https://YOUR-PROD-CLOUDFRONT.cloudfront.net" --repo $REPO
gh secret set CLOUDFRONT_DISTRIBUTION_ID_PROD --body "YOUR-PROD-DIST-ID" --repo $REPO
gh secret set S3_BUCKET_PROD --body "cropwise-prod-frontend" --repo $REPO
gh secret set ECS_CLUSTER_PROD --body "cropwise-prod-cluster" --repo $REPO
gh secret set ECS_SERVICE_PROD --body "cropwise-backend-prod-service" --repo $REPO
gh secret set ECS_TASK_DEFINITION_PROD --body "cropwise-backend-prod" --repo $REPO

# ECR Repository
gh secret set AWS_ECR_REPOSITORY --body "cropwise-backend" --repo $REPO
```

---

## 🔐 STEP 9: Update Google OAuth

Add all environment URLs to Google Cloud Console:

1. Go to: https://console.cloud.google.com/apis/credentials
2. Click your OAuth 2.0 Client ID
3. **Add Authorized JavaScript Origins:**
   ```
   # Local Development
   http://localhost:8080
   http://localhost:3000
   
   # AWS Development
   https://d1a2b3c4d5e6f7.cloudfront.net
   
   # AWS Staging
   https://d9x8y7z6w5v4u3.cloudfront.net
   
   # AWS Production
   https://d3t2s1r0q9p8o7.cloudfront.net
   ```

4. **Add Authorized Redirect URIs:**
   ```
   # Local Development
   http://localhost:3000/api/auth/google/callback
   
   # AWS Development Backend
   http://cropwise-dev-alb-XXX.ap-south-1.elb.amazonaws.com/api/auth/google/callback
   
   # AWS Development Frontend
   https://d1a2b3c4d5e6f7.cloudfront.net/auth/google/callback
   
   # AWS Staging Backend
   http://cropwise-stage-alb-YYY.ap-south-1.elb.amazonaws.com/api/auth/google/callback
   
   # AWS Staging Frontend
   https://d9x8y7z6w5v4u3.cloudfront.net/auth/google/callback
   
   # AWS Production Backend
   http://cropwise-prod-alb-ZZZ.ap-south-1.elb.amazonaws.com/api/auth/google/callback
   
   # AWS Production Frontend
   https://d3t2s1r0q9p8o7.cloudfront.net/auth/google/callback
   ```

5. Click **"Save"**

⚠️ **Important**: Replace the CloudFront and ALB URLs with your actual URLs from Steps 3 and 7!

---

## 🚀 STEP 10: Deploy!

Now you're ready to deploy to all environments:

```powershell
cd C:\Users\praghav\cropwise

# Deploy to Development
git checkout develop
git push origin develop

# Deploy to Staging
git checkout staging
git push origin staging

# Deploy to Production
git checkout main
git push origin main

# Watch deployments
start https://github.com/yellowflowersorganics-star/cropwise/actions
```

Your CI/CD pipeline will automatically:
1. Build Docker images
2. Push to ECR with appropriate tags (dev/stage/prod)
3. Deploy backend to ECS
4. Build and deploy frontend to S3
5. Invalidate CloudFront cache

---

## 📋 Complete Infrastructure Checklist

Verify all resources are created for each environment:

### **✅ ECR (Shared)**
- [ ] Repository: `cropwise-backend`
- [ ] Lifecycle policy configured

### **✅ Development Environment**
- [ ] RDS: `cropwise-dev-db` (PostgreSQL)
- [ ] ECS Cluster: `cropwise-dev-cluster`
- [ ] ECS Task Definition: `cropwise-backend-dev`
- [ ] ECS Service: `cropwise-backend-dev-service` (1 task)
- [ ] Application Load Balancer: `cropwise-dev-alb`
- [ ] Target Group: `cropwise-dev-backend-tg`
- [ ] ALB Security Group: `cropwise-dev-alb-sg`
- [ ] ECS Security Group: `cropwise-dev-ecs-sg`
- [ ] S3 Bucket: `cropwise-dev-frontend`
- [ ] CloudFront Distribution (with error pages for SPA)
- [ ] CloudWatch Log Group: `/ecs/cropwise-backend-dev`

### **✅ Staging Environment**
- [ ] RDS: `cropwise-stage-db` (PostgreSQL)
- [ ] ECS Cluster: `cropwise-stage-cluster`
- [ ] ECS Task Definition: `cropwise-backend-stage`
- [ ] ECS Service: `cropwise-backend-stage-service` (1 task)
- [ ] Application Load Balancer: `cropwise-stage-alb`
- [ ] Target Group: `cropwise-stage-backend-tg`
- [ ] ALB Security Group: `cropwise-stage-alb-sg`
- [ ] ECS Security Group: `cropwise-stage-ecs-sg`
- [ ] S3 Bucket: `cropwise-stage-frontend`
- [ ] CloudFront Distribution (with error pages for SPA)
- [ ] CloudWatch Log Group: `/ecs/cropwise-backend-stage`
- [ ] Container Insights enabled

### **✅ Production Environment**
- [ ] RDS: `cropwise-prod-db` (PostgreSQL, Multi-AZ)
- [ ] ECS Cluster: `cropwise-prod-cluster`
- [ ] ECS Task Definition: `cropwise-backend-prod`
- [ ] ECS Service: `cropwise-backend-prod-service` (2+ tasks, auto-scaling)
- [ ] Application Load Balancer: `cropwise-prod-alb` (deletion protection)
- [ ] Target Group: `cropwise-prod-backend-tg`
- [ ] ALB Security Group: `cropwise-prod-alb-sg`
- [ ] ECS Security Group: `cropwise-prod-ecs-sg`
- [ ] S3 Bucket: `cropwise-prod-frontend` (versioning enabled)
- [ ] CloudFront Distribution (with logging and error pages)
- [ ] CloudWatch Log Group: `/ecs/cropwise-backend-prod`
- [ ] Container Insights enabled
- [ ] ALB Access Logs enabled

### **✅ GitHub Secrets (25 total)**
- [ ] `AWS_REGION`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`
- [ ] `AWS_ECR_REPOSITORY`
- [ ] `DATABASE_URL_DEV`, `DATABASE_URL_STAGE`, `DATABASE_URL_PROD`
- [ ] `BACKEND_URL_DEV`, `BACKEND_URL_STAGE`, `BACKEND_URL_PROD`
- [ ] `FRONTEND_URL_DEV`, `FRONTEND_URL_STAGE`, `FRONTEND_URL_PROD`
- [ ] `CLOUDFRONT_DISTRIBUTION_ID_DEV`, `CLOUDFRONT_DISTRIBUTION_ID_STAGE`, `CLOUDFRONT_DISTRIBUTION_ID_PROD`
- [ ] `S3_BUCKET_DEV`, `S3_BUCKET_STAGE`, `S3_BUCKET_PROD`
- [ ] `ECS_CLUSTER_DEV`, `ECS_CLUSTER_STAGE`, `ECS_CLUSTER_PROD`
- [ ] `ECS_SERVICE_DEV`, `ECS_SERVICE_STAGE`, `ECS_SERVICE_PROD`
- [ ] `ECS_TASK_DEFINITION_DEV`, `ECS_TASK_DEFINITION_STAGE`, `ECS_TASK_DEFINITION_PROD`
- [ ] `JWT_SECRET`, `SESSION_SECRET`
- [ ] `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`

### **✅ Google OAuth Configured**
- [ ] Added all 3 CloudFront URLs to authorized origins
- [ ] Added all 6 redirect URIs (3 backends + 3 frontends)

---

## 💰 Complete Cost Estimate

### **Per Environment Breakdown**

#### **Development Environment**
```
RDS (db.t3.micro): $15/month (or FREE)
ECS Fargate (0.25 vCPU, 512MB, 1 task): $6/month
ALB: $22/month
S3 (minimal usage): $1/month
CloudFront (minimal traffic): $5/month
Data Transfer: $3/month
────────────────────────────────────────
Development Total: ~$52/month ($37 with Free Tier)
```

#### **Staging Environment**
```
RDS (db.t3.small): $35/month
ECS Fargate (0.5 vCPU, 1GB, 1 task): $12/month
ALB: $22/month
S3 (moderate usage): $2/month
CloudFront (moderate traffic): $10/month
Data Transfer: $5/month
Container Insights: $2/month
────────────────────────────────────────
Staging Total: ~$88/month
```

#### **Production Environment**
```
RDS (db.t3.medium Multi-AZ): $133/month
ECS Fargate (1 vCPU, 2GB, 2 tasks): $72/month
ALB (with access logs): $25/month
S3 (high usage + logs): $5/month
CloudFront (high traffic + logs): $25/month
Data Transfer: $15/month
Container Insights: $5/month
ALB/CloudFront Logs Storage: $3/month
────────────────────────────────────────
Production Total: ~$283/month
```

### **Total Monthly Cost (All 3 Environments)**
```
Development:    $52  ($37 with Free Tier)
Staging:        $88
Production:    $283
────────────────────────────────────────
TOTAL:         $423/month ($408 with Free Tier)
               ~$5,076/year (~$4,896 with Free Tier)
```

### **Cost Optimization for Smaller Scale**

If you're just getting started:

```
# Budget-Friendly Setup
Development:   $37/month (Free Tier RDS, minimal Fargate)
Staging:       $0 (skip initially, test on dev)
Production:    $120/month (Single-AZ RDS, 1 Fargate task)
────────────────────────────────────────
Budget Total:  ~$157/month
               ~$1,884/year

# When to upgrade:
- Add staging when team > 3 people
- Add Multi-AZ when revenue > $5k/month
- Scale Fargate tasks when CPU > 70%
```

### **Additional Costs to Consider**

- **Custom Domain**: $12-50/year (Route 53 + domain)
- **SSL Certificate**: FREE (AWS Certificate Manager)
- **Monitoring/Alerts**: $5-10/month (CloudWatch alarms)
- **Backup Storage**: $2-5/month (S3 for backups)
- **WAF (DDoS Protection)**: $5-20/month (if enabled)

---

## 📊 Resource Summary Table

| Resource | Development | Staging | Production | Total |
|----------|-------------|---------|------------|-------|
| **RDS PostgreSQL** | db.t3.micro<br>20GB | db.t3.small<br>50GB | db.t3.medium Multi-AZ<br>100GB | 3 databases |
| **ECS Clusters** | cropwise-dev-cluster | cropwise-stage-cluster | cropwise-prod-cluster | 3 clusters |
| **ECS Tasks** | 0.25 vCPU, 512MB<br>1 task | 0.5 vCPU, 1GB<br>1 task | 1 vCPU, 2GB<br>2 tasks + autoscaling | 4-12 tasks total |
| **Load Balancers** | cropwise-dev-alb | cropwise-stage-alb | cropwise-prod-alb | 3 ALBs |
| **S3 Buckets** | cropwise-dev-frontend | cropwise-stage-frontend | cropwise-prod-frontend<br>(versioned) | 3 buckets |
| **CloudFront** | 1 distribution | 1 distribution | 1 distribution<br>(with logging) | 3 distributions |
| **Monthly Cost** | ~$52 | ~$88 | ~$283 | ~$423 |

---

## 🎯 Next Steps

1. **Complete this infrastructure setup** for all 3 environments
2. **Test each environment** individually
3. **Update GitHub Secrets** with all URLs and IDs
4. **Configure Google OAuth** with all URLs
5. **Test CI/CD pipeline** with a small change
6. **Monitor costs** in AWS Cost Explorer
7. **Set up billing alerts** ($100, $200, $400)
8. **Document** any environment-specific configurations

---

**Created**: November 14, 2025  
**Last Updated**: November 16, 2025  
**For**: CropWise - Complete Multi-Environment AWS Infrastructure
