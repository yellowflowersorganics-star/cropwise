# 🚀 CropWise - Complete AWS Deployment Guide

This guide will walk you through deploying CropWise to AWS from scratch.

## 📋 Table of Contents

1. [Prerequisites](#prerequisites)
2. [Architecture Overview](#architecture-overview)
3. [Deployment Options](#deployment-options)
4. [Option A: Elastic Beanstalk (Easiest)](#option-a-elastic-beanstalk)
5. [Option B: ECS with Fargate](#option-b-ecs-with-fargate)
6. [Option C: EC2 Manual Setup](#option-c-ec2-manual)
7. [Post-Deployment Setup](#post-deployment-setup)
8. [Monitoring & Maintenance](#monitoring--maintenance)
9. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### 1. AWS Account
- Create account at [aws.amazon.com](https://aws.amazon.com)
- Verify email and set up billing

### 2. Install AWS CLI
```bash
# macOS
brew install awscli

# Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Windows
# Download from: https://aws.amazon.com/cli/
```

### 3. Configure AWS CLI
```bash
aws configure

# Enter:
# AWS Access Key ID: (from AWS Console -> IAM)
# AWS Secret Access Key: (from AWS Console -> IAM)
# Default region: ap-south-1
# Default output format: json
```

### 4. Install Additional Tools
```bash
# Elastic Beanstalk CLI
pip install awsebcli --upgrade --user

# Docker (for containerized deployments)
# Install from: https://www.docker.com/get-started

# jq (JSON processor)
# macOS: brew install jq
# Linux: sudo apt-get install jq
```

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      AWS Cloud (VPC)                        │
│                                                             │
│  ┌──────────────┐         ┌──────────────┐                │
│  │  Internet    │────────▶│   Route 53   │                │
│  │  Gateway     │         │     (DNS)    │                │
│  └──────────────┘         └──────────────┘                │
│         │                                                   │
│         ▼                                                   │
│  ┌──────────────────────────────────────────┐             │
│  │   Application Load Balancer (ALB)        │             │
│  │   - Port 80 (HTTP) → 443 (HTTPS)         │             │
│  │   - SSL/TLS termination                  │             │
│  └──────────────────────────────────────────┘             │
│         │                                                   │
│         ├─────────────────┬────────────────┐              │
│         ▼                 ▼                ▼               │
│  ┌─────────────┐   ┌─────────────┐  ┌─────────────┐     │
│  │   Backend   │   │   Backend   │  │  Frontend   │     │
│  │  (Node.js)  │   │  (Node.js)  │  │   (React)   │     │
│  │   ECS/EC2   │   │   ECS/EC2   │  │   ECS/S3    │     │
│  └─────────────┘   └─────────────┘  └─────────────┘     │
│         │                 │                               │
│         └────────┬────────┘                               │
│                  ▼                                         │
│         ┌────────────────────┐                            │
│         │   ElastiCache      │                            │
│         │     (Redis)        │                            │
│         └────────────────────┘                            │
│                  │                                         │
│                  ▼                                         │
│         ┌────────────────────┐                            │
│         │   RDS PostgreSQL   │                            │
│         │   (Multi-AZ)       │                            │
│         └────────────────────┘                            │
│                  │                                         │
│                  ▼                                         │
│         ┌────────────────────┐                            │
│         │   AWS IoT Core     │                            │
│         │   (MQTT Broker)    │                            │
│         └────────────────────┘                            │
│                  │                                         │
│                  ▼                                         │
│         ┌────────────────────┐                            │
│         │    ESP32 Devices   │                            │
│         └────────────────────┘                            │
└─────────────────────────────────────────────────────────────┘
```

---

## Deployment Options

Choose based on your needs:

| Option | Difficulty | Cost | Best For |
|--------|-----------|------|----------|
| **A. Elastic Beanstalk** | ⭐ Easy | $$ | Quick deployment, managed infrastructure |
| **B. ECS with Fargate** | ⭐⭐⭐ Medium | $$$ | Containerized apps, scalability |
| **C. EC2 Manual** | ⭐⭐⭐⭐ Hard | $ | Full control, custom setup |

---

## Option A: Elastic Beanstalk (Recommended for Beginners)

### Estimated Setup Time: 30 minutes
### Monthly Cost: ~$50-100

### Step 1: Create RDS Database

```bash
# Create database
aws rds create-db-instance \
  --db-instance-identifier cropwise-db \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --master-username cropwise_admin \
  --master-user-password YOUR_STRONG_PASSWORD_HERE \
  --allocated-storage 20 \
  --backup-retention-period 7 \
  --publicly-accessible false

# Wait for database to be available (~10 minutes)
aws rds wait db-instance-available --db-instance-identifier cropwise-db

# Get database endpoint
aws rds describe-db-instances \
  --db-instance-identifier cropwise-db \
  --query 'DBInstances[0].Endpoint.Address' \
  --output text
```

### Step 2: Create ElastiCache Redis

```bash
# Create Redis cluster
aws elasticache create-cache-cluster \
  --cache-cluster-id cropwise-redis \
  --cache-node-type cache.t3.micro \
  --engine redis \
  --num-cache-nodes 1

# Get Redis endpoint
aws elasticache describe-cache-clusters \
  --cache-cluster-id cropwise-redis \
  --show-cache-node-info \
  --query 'CacheClusters[0].CacheNodes[0].Endpoint.Address' \
  --output text
```

### Step 3: Initialize Elastic Beanstalk

```bash
cd backend

# Initialize EB application
eb init cropwise-backend \
  --platform "Node.js 18 running on 64bit Amazon Linux 2023" \
  --region ap-south-1

# Create environment
eb create cropwise-production \
  --instance-type t3.small \
  --envvars \
    NODE_ENV=production,\
    DB_HOST=YOUR_RDS_ENDPOINT,\
    DB_PORT=5432,\
    DB_NAME=cropwise_db,\
    DB_USER=cropwise_admin,\
    DB_PASSWORD=YOUR_DB_PASSWORD,\
    REDIS_HOST=YOUR_REDIS_ENDPOINT,\
    REDIS_PORT=6379,\
    JWT_SECRET=YOUR_JWT_SECRET,\
    GOOGLE_CLIENT_ID=YOUR_GOOGLE_CLIENT_ID,\
    GOOGLE_CLIENT_SECRET=YOUR_GOOGLE_CLIENT_SECRET
```

### Step 4: Deploy Backend

```bash
# Deploy
eb deploy

# Check status
eb status

# Open in browser
eb open

# View logs
eb logs
```

### Step 5: Deploy Frontend to S3 + CloudFront

```bash
cd ../frontend

# Build production bundle
npm run build

# Create S3 bucket
aws s3 mb s3://cropwise-frontend-YOUR_UNIQUE_ID

# Upload files
aws s3 sync dist/ s3://cropwise-frontend-YOUR_UNIQUE_ID --acl public-read

# Create CloudFront distribution
aws cloudfront create-distribution \
  --origin-domain-name cropwise-frontend-YOUR_UNIQUE_ID.s3.amazonaws.com \
  --default-root-object index.html
```

### Step 6: Configure Custom Domain (Optional)

```bash
# 1. Register domain in Route 53 or use existing
# 2. Create SSL certificate in ACM
aws acm request-certificate \
  --domain-name yourdomain.com \
  --validation-method DNS

# 3. Update EB environment
eb setenv FRONTEND_URL=https://yourdomain.com

# 4. Add domain to CloudFront distribution
```

---

## Option B: ECS with Fargate (Docker Containers)

### Estimated Setup Time: 1-2 hours
### Monthly Cost: ~$70-150

### Step 1: Run Infrastructure Setup Script

```bash
chmod +x scripts/aws-setup.sh
./scripts/aws-setup.sh
```

This creates:
- VPC with public/private subnets
- Security groups
- RDS PostgreSQL
- ElastiCache Redis
- ECS Cluster
- Application Load Balancer

**⏳ Wait 10-15 minutes for RDS and Redis to finish creating**

### Step 2: Create Task Definitions

```bash
# Backend Task Definition
aws ecs register-task-definition --cli-input-json file://aws/ecs-backend-task.json

# Frontend Task Definition  
aws ecs register-task-definition --cli-input-json file://aws/ecs-frontend-task.json
```

### Step 3: Deploy Containers

```bash
# Run deployment script
chmod +x scripts/aws-deploy.sh
./scripts/aws-deploy.sh
```

This will:
- Build Docker images
- Push to ECR
- Update ECS services
- Deploy new containers

### Step 4: Create ECS Services

```bash
# Backend service
aws ecs create-service \
  --cluster cropwise-cluster \
  --service-name cropwise-backend \
  --task-definition cropwise-backend \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[SUBNET_ID_1,SUBNET_ID_2],securityGroups=[SG_ID],assignPublicIp=ENABLED}" \
  --load-balancers targetGroupArn=TARGET_GROUP_ARN,containerName=backend,containerPort=3000

# Frontend service  
aws ecs create-service \
  --cluster cropwise-cluster \
  --service-name cropwise-frontend \
  --task-definition cropwise-frontend \
  --desired-count 2 \
  --launch-type FARGATE
```

### Step 5: Configure Load Balancer

```bash
# Get ALB DNS name from aws-config.json
cat aws-config.json | jq -r '.albDns'

# Your application will be available at:
# http://YOUR_ALB_DNS_NAME
```

---

## Option C: EC2 Manual Setup

### Estimated Setup Time: 2-3 hours
### Monthly Cost: ~$30-80

### Step 1: Launch EC2 Instance

```bash
# Create key pair
aws ec2 create-key-pair \
  --key-name cropwise-key \
  --query 'KeyMaterial' \
  --output text > cropwise-key.pem

chmod 400 cropwise-key.pem

# Launch instance
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t3.small \
  --key-name cropwise-key \
  --security-group-ids sg-xxxxxxxxx \
  --subnet-id subnet-xxxxxxxxx \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=cropwise-server}]'
```

### Step 2: SSH and Install Dependencies

```bash
# SSH into instance
ssh -i cropwise-key.pem ec2-user@YOUR_EC2_IP

# Update system
sudo yum update -y

# Install Node.js
curl -sL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install -y nodejs

# Install PostgreSQL
sudo yum install -y postgresql15-server
sudo postgresql-setup --initdb
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Install Redis
sudo yum install -y redis
sudo systemctl start redis
sudo systemctl enable redis

# Install Nginx
sudo yum install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Install PM2 (Process Manager)
sudo npm install -g pm2
```

### Step 3: Clone and Configure Application

```bash
# Clone repository
git clone https://github.com/your-username/cropwise.git
cd cropwise

# Install backend dependencies
cd backend
npm install --production

# Create .env file
cat > .env << EOF
NODE_ENV=production
PORT=3000
DB_HOST=localhost
DB_PORT=5432
DB_NAME=cropwise_db
DB_USER=postgres
DB_PASSWORD=your_password
REDIS_HOST=localhost
REDIS_PORT=6379
JWT_SECRET=your_jwt_secret_here
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
EOF

# Start backend with PM2
pm2 start src/index.js --name cropwise-backend
pm2 save
pm2 startup
```

### Step 4: Configure Nginx

```bash
sudo nano /etc/nginx/conf.d/cropwise.conf
```

Add:
```nginx
server {
    listen 80;
    server_name your_domain.com;

    # Frontend
    location / {
        root /home/ec2-user/cropwise/frontend/dist;
        try_files $uri $uri/ /index.html;
    }

    # Backend API
    location /api {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```bash
# Test configuration
sudo nginx -t

# Restart Nginx
sudo systemctl restart nginx
```

### Step 5: Build and Deploy Frontend

```bash
cd /home/ec2-user/cropwise/frontend
npm install
npm run build
```

---

## Post-Deployment Setup

### 1. Configure Environment Variables

Update these critical environment variables:

```bash
# Backend
JWT_SECRET=<generate-strong-secret>
GOOGLE_CLIENT_ID=<your-google-client-id>
GOOGLE_CLIENT_SECRET=<your-google-client-secret>
FRONTEND_URL=https://yourdomain.com

# Update Google OAuth redirect URI to:
# https://api.yourdomain.com/api/auth/google/callback
```

### 2. Set Up SSL/HTTPS

#### For Elastic Beanstalk:
```bash
# Request certificate
aws acm request-certificate \
  --domain-name yourdomain.com \
  --validation-method DNS

# Add to load balancer
eb config
# Update 03_https.config with certificate ARN
```

#### For EC2/ALB:
```bash
# Install Certbot
sudo yum install -y certbot python3-certbot-nginx

# Get certificate
sudo certbot --nginx -d yourdomain.com
```

### 3. Configure DNS

```bash
# Create hosted zone
aws route53 create-hosted-zone --name yourdomain.com

# Create A record pointing to your ALB/EC2
aws route53 change-resource-record-sets \
  --hosted-zone-id YOUR_ZONE_ID \
  --change-batch file://dns-record.json
```

### 4. Set Up AWS IoT Core for MQTT

```bash
# Create IoT Thing
aws iot create-thing --thing-name cropwise-esp32-001

# Create certificate
aws iot create-keys-and-certificate \
  --set-as-active \
  --certificate-pem-outfile certificate.pem \
  --private-key-outfile private.key

# Create IoT Policy
aws iot create-policy \
  --policy-name cropwise-policy \
  --policy-document file://iot-policy.json

# Attach policy to certificate
aws iot attach-policy \
  --policy-name cropwise-policy \
  --target CERTIFICATE_ARN
```

### 5. Initialize Database

```bash
# Run migrations
cd backend
npm run migrate

# Seed data (optional)
npm run seed
```

---

## Monitoring & Maintenance

### CloudWatch Dashboards

```bash
# Create dashboard
aws cloudwatch put-dashboard \
  --dashboard-name cropwise-monitoring \
  --dashboard-body file://cloudwatch-dashboard.json
```

### Set Up Alarms

```bash
# CPU Utilization alarm
aws cloudwatch put-metric-alarm \
  --alarm-name cropwise-high-cpu \
  --alarm-description "Alert when CPU > 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold

# Database connections alarm
aws cloudwatch put-metric-alarm \
  --alarm-name cropwise-db-connections \
  --metric-name DatabaseConnections \
  --namespace AWS/RDS \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold
```

### Automated Backups

```bash
# RDS automated backups (already enabled)
# Backup retention: 7 days

# Manual snapshot
aws rds create-db-snapshot \
  --db-instance-identifier cropwise-db \
  --db-snapshot-identifier cropwise-backup-$(date +%Y%m%d)
```

### Log Management

```bash
# View logs (Elastic Beanstalk)
eb logs

# View logs (ECS)
aws logs tail /ecs/cropwise-backend --follow

# Export logs to S3
aws logs create-export-task \
  --log-group-name /aws/elasticbeanstalk/cropwise-production \
  --from $(date -d '7 days ago' +%s)000 \
  --to $(date +%s)000 \
  --destination cropwise-logs-backup
```

---

## Troubleshooting

### Application Won't Start

```bash
# Check logs
eb logs  # Elastic Beanstalk
aws logs tail /ecs/cropwise-backend --follow  # ECS

# Common issues:
# 1. Environment variables missing
# 2. Database connection failed
# 3. Redis connection failed
# 4. Port already in use
```

### Database Connection Issues

```bash
# Check RDS status
aws rds describe-db-instances \
  --db-instance-identifier cropwise-db

# Check security group rules
aws ec2 describe-security-groups \
  --group-ids sg-xxxxxxxxx

# Test connection from EC2
psql -h YOUR_RDS_ENDPOINT -U cropwise_admin -d cropwise_db
```

### High Costs

```bash
# Check cost breakdown
aws ce get-cost-and-usage \
  --time-period Start=2024-01-01,End=2024-01-31 \
  --granularity MONTHLY \
  --metrics BlendedCost

# Optimize:
# 1. Use Reserved Instances for 1-year commitment (save 30-40%)
# 2. Enable auto-scaling to reduce idle resources
# 3. Use Spot Instances for non-critical workloads
# 4. Set up CloudWatch alarms for cost anomalies
```

### Deployment Failures

```bash
# Rollback (Elastic Beanstalk)
eb deploy --version <previous-version-label>

# Rollback (ECS)
aws ecs update-service \
  --cluster cropwise-cluster \
  --service cropwise-backend \
  --task-definition cropwise-backend:PREVIOUS_REVISION
```

---

## Cost Optimization Tips

1. **Use Free Tier**: First 12 months
   - EC2: 750 hours/month t2.micro
   - RDS: 750 hours/month db.t2.micro
   - S3: 5GB storage
   - CloudFront: 50GB data transfer

2. **Right-size Instances**
   - Start with t3.micro/small
   - Monitor and adjust based on usage

3. **Auto-scaling**
   - Scale down during low traffic
   - Scale up during peak hours

4. **Reserved Instances**
   - 1-year term: 30-40% savings
   - 3-year term: 50-60% savings

5. **Use S3 for Static Content**
   - Much cheaper than serving from EC2/ECS

---

## Support & Resources

- **AWS Documentation**: https://docs.aws.amazon.com/
- **AWS Free Tier**: https://aws.amazon.com/free/
- **Cost Calculator**: https://calculator.aws/
- **CropWise Issues**: https://github.com/your-repo/issues

---

## Next Steps

✅ Application deployed to AWS  
✅ SSL/HTTPS configured  
✅ Custom domain set up  
⬜ Google OAuth configured  
⬜ AWS IoT Core for MQTT  
⬜ Monitoring and alerts  
⬜ Backup strategy  
⬜ CI/CD pipeline  

**Congratulations! Your CropWise is now running on AWS!** 🎉

