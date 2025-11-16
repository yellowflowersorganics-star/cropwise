# 🚀 Deploy CropWise to AWS - Quick Start

**Estimated Time:** 45-60 minutes  
**Estimated Cost:** $50-150/month

---

## ✅ Pre-Deployment Checklist

Before we start, make sure you have:

- [ ] AWS Account created and verified
- [ ] Credit card added to AWS account
- [ ] AWS CLI installed on your machine
- [ ] Docker installed and running
- [ ] Git repository up to date

---

## 🎯 Deployment Options

We'll use **Option B: AWS Elastic Beanstalk** (Easiest and Recommended)

### Why Elastic Beanstalk?
- ✅ **Fastest deployment** (~30 minutes)
- ✅ **Managed infrastructure** (no server management)
- ✅ **Auto-scaling** included
- ✅ **Load balancer** included
- ✅ **Easy rollback** if needed
- ✅ **$50-100/month** (affordable for startups)

---

## 📋 Step-by-Step Deployment Guide

### **Step 1: Install AWS CLI** (5 minutes)

#### Windows (PowerShell as Administrator):
```powershell
# Download and install AWS CLI
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi

# Verify installation
aws --version
```

#### Alternative (if link doesn't work):
1. Download from: https://awscli.amazonaws.com/AWSCLIV2.msi
2. Double-click to install
3. Restart PowerShell

---

### **Step 2: Configure AWS Credentials** (5 minutes)

```powershell
# Configure AWS CLI
aws configure

# You'll be prompted for:
# AWS Access Key ID: [Get from AWS Console]
# AWS Secret Access Key: [Get from AWS Console]
# Default region name: ap-south-1
# Default output format: json
```

#### How to Get AWS Keys:
1. Go to: https://console.aws.amazon.com/iam/
2. Click "Users" → "Your Username"
3. Click "Security credentials" tab
4. Click "Create access key"
5. Choose "Command Line Interface (CLI)"
6. Copy the Access Key ID and Secret Access Key

---

### **Step 3: Install Elastic Beanstalk CLI** (5 minutes)

```powershell
# Install Python (if not installed)
# Download from: https://www.python.org/downloads/

# Install EB CLI
pip install awsebcli --upgrade --user

# Verify installation
eb --version
```

---

### **Step 4: Create Environment Variables** (5 minutes)

Create a file named `.env.production` in your `backend/` folder:

```bash
# Database (RDS PostgreSQL)
NODE_ENV=production
DB_DIALECT=postgres
DB_HOST=YOUR_RDS_ENDPOINT_HERE
DB_PORT=5432
DB_NAME=cropwise_db
DB_USER=cropwise_admin
DB_PASSWORD=YOUR_STRONG_PASSWORD_HERE

# JWT Secret (generate a strong random string)
JWT_SECRET=YOUR_SUPER_SECRET_JWT_KEY_HERE_MINIMUM_32_CHARS

# Redis (ElastiCache)
REDIS_HOST=YOUR_REDIS_ENDPOINT_HERE
REDIS_PORT=6379

# MQTT (IoT Core or Mosquitto)
MQTT_BROKER_URL=mqtt://YOUR_MQTT_BROKER_HERE:1883

# Google OAuth
GOOGLE_CLIENT_ID=YOUR_GOOGLE_CLIENT_ID
GOOGLE_CLIENT_SECRET=YOUR_GOOGLE_CLIENT_SECRET
GOOGLE_CALLBACK_URL=https://YOUR_DOMAIN.com/api/auth/google/callback

# Twilio (for SMS/WhatsApp)
TWILIO_ACCOUNT_SID=your_twilio_sid
TWILIO_AUTH_TOKEN=your_twilio_token
TWILIO_PHONE_NUMBER=+1234567890
TWILIO_WHATSAPP_NUMBER=+14155238886

# Frontend URL
FRONTEND_URL=https://YOUR_DOMAIN.com

# Port
PORT=3000
```

---

### **Step 5: Create RDS PostgreSQL Database** (15 minutes)

```powershell
# Create RDS instance
aws rds create-db-instance `
  --db-instance-identifier cropwise-db `
  --db-instance-class db.t3.micro `
  --engine postgres `
  --engine-version 15.4 `
  --master-username cropwise_admin `
  --master-user-password YOUR_STRONG_PASSWORD_HERE `
  --allocated-storage 20 `
  --backup-retention-period 7 `
  --storage-encrypted `
  --publicly-accessible false `
  --region ap-south-1

# Wait for database to be available (takes ~10 minutes)
aws rds wait db-instance-available --db-instance-identifier cropwise-db

# Get the database endpoint
aws rds describe-db-instances `
  --db-instance-identifier cropwise-db `
  --query 'DBInstances[0].Endpoint.Address' `
  --output text
```

**Copy the endpoint URL and update `.env.production` file's `DB_HOST` value**

---

### **Step 6: Create ElastiCache Redis** (Optional but Recommended)

```powershell
# Create security group for ElastiCache
aws elasticache create-cache-cluster `
  --cache-cluster-id cropwise-redis `
  --cache-node-type cache.t3.micro `
  --engine redis `
  --num-cache-nodes 1 `
  --region ap-south-1

# Wait for creation (~5 minutes)
aws elasticache wait cache-cluster-available --cache-cluster-id cropwise-redis

# Get Redis endpoint
aws elasticache describe-cache-clusters `
  --cache-cluster-id cropwise-redis `
  --show-cache-node-info `
  --query 'CacheClusters[0].CacheNodes[0].Endpoint.Address' `
  --output text
```

**Copy the Redis endpoint and update `.env.production` file's `REDIS_HOST` value**

---

### **Step 7: Initialize Elastic Beanstalk** (5 minutes)

```powershell
# Navigate to backend directory
cd backend

# Initialize Elastic Beanstalk application
eb init

# You'll be prompted:
# 1. Select a default region: (choose ap-south-1 or your preferred region)
# 2. Enter Application Name: cropwise
# 3. Do you want to set up SSH: Y (recommended)
# 4. Select platform: Node.js
# 5. Select platform version: Node.js 18 running on 64bit Amazon Linux 2023
# 6. Do you want to continue with CodeCommit? N
```

---

### **Step 8: Create Elastic Beanstalk Environment** (10 minutes)

```powershell
# Create environment with database and load balancer
eb create cropwise-production `
  --elb-type application `
  --database `
  --database.engine postgres `
  --database.username cropwise_admin `
  --instance-type t3.small `
  --envvars NODE_ENV=production

# This will:
# ✅ Create EC2 instances
# ✅ Create Application Load Balancer
# ✅ Set up Auto Scaling
# ✅ Deploy your backend

# Wait for environment to be ready (~10 minutes)
eb status
```

---

### **Step 9: Set Environment Variables** (5 minutes)

```powershell
# Set all environment variables from .env.production file
eb setenv `
  DB_DIALECT=postgres `
  DB_HOST=YOUR_RDS_ENDPOINT_HERE `
  DB_PORT=5432 `
  DB_NAME=cropwise_db `
  DB_USER=cropwise_admin `
  DB_PASSWORD=YOUR_STRONG_PASSWORD_HERE `
  JWT_SECRET=YOUR_SUPER_SECRET_JWT_KEY_HERE `
  REDIS_HOST=YOUR_REDIS_ENDPOINT_HERE `
  REDIS_PORT=6379 `
  FRONTEND_URL=https://YOUR_DOMAIN.com `
  GOOGLE_CLIENT_ID=YOUR_GOOGLE_CLIENT_ID `
  GOOGLE_CLIENT_SECRET=YOUR_GOOGLE_CLIENT_SECRET

# Elastic Beanstalk will automatically restart with new variables
```

---

### **Step 10: Deploy Backend** (5 minutes)

```powershell
# Deploy current version
eb deploy

# Wait for deployment to complete
eb status

# Check if deployment was successful
eb health

# Open the application in browser
eb open
```

**Your backend API is now live!** 🎉

Copy the URL (e.g., `http://cropwise-production.ap-south-1.elasticbeanstalk.com`)

---

### **Step 11: Deploy Frontend to S3 + CloudFront** (10 minutes)

```powershell
# Navigate to frontend directory
cd ..\frontend

# Update API URL in frontend
# Edit frontend/.env.production:
VITE_API_URL=http://YOUR_BACKEND_URL_FROM_STEP_10.elasticbeanstalk.com

# Build production frontend
npm run build

# Create S3 bucket for frontend
aws s3 mb s3://cropwise-frontend-YOUR_UNIQUE_ID --region ap-south-1

# Enable static website hosting
aws s3 website s3://cropwise-frontend-YOUR_UNIQUE_ID `
  --index-document index.html `
  --error-document index.html

# Upload built files
aws s3 sync dist/ s3://cropwise-frontend-YOUR_UNIQUE_ID --delete

# Make bucket public (for website hosting)
aws s3api put-bucket-policy `
  --bucket cropwise-frontend-YOUR_UNIQUE_ID `
  --policy '{
    "Version": "2012-10-17",
    "Statement": [{
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::cropwise-frontend-YOUR_UNIQUE_ID/*"
    }]
  }'

# Get website URL
echo "Your frontend is live at: http://cropwise-frontend-YOUR_UNIQUE_ID.s3-website-ap-south-1.amazonaws.com"
```

---

### **Step 12: (Optional) Set Up CloudFront CDN** (10 minutes)

```powershell
# Create CloudFront distribution for faster global access
aws cloudfront create-distribution `
  --origin-domain-name cropwise-frontend-YOUR_UNIQUE_ID.s3-website-ap-south-1.amazonaws.com `
  --default-root-object index.html

# Wait for distribution to deploy (~15 minutes)
# You'll get a CloudFront URL like: https://d123456abcdef.cloudfront.net
```

---

### **Step 13: (Optional) Set Up Custom Domain** (15 minutes)

If you have a domain (e.g., `cropwise.io`):

```powershell
# 1. Create hosted zone in Route 53
aws route53 create-hosted-zone --name cropwise.io --caller-reference $(date +%s)

# 2. Point your domain registrar's nameservers to Route 53 nameservers

# 3. Create alias record for frontend
aws route53 change-resource-record-sets `
  --hosted-zone-id YOUR_ZONE_ID `
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "www.cropwise.io",
        "Type": "A",
        "AliasTarget": {
          "HostedZoneId": "Z2FDTNDATAQYW2",
          "DNSName": "YOUR_CLOUDFRONT_DOMAIN.cloudfront.net",
          "EvaluateTargetHealth": false
        }
      }
    }]
  }'

# 4. Create SSL certificate
aws acm request-certificate `
  --domain-name cropwise.io `
  --subject-alternative-names *.cropwise.io `
  --validation-method DNS

# 5. Verify domain ownership (follow email instructions)
```

---

## ✅ Post-Deployment Checklist

After deployment, verify:

- [ ] Backend health check: `https://YOUR_BACKEND_URL/health`
- [ ] Frontend loads correctly
- [ ] User registration works
- [ ] Google OAuth works (update callback URL in Google Console)
- [ ] Database connection works (create a test farm)
- [ ] IoT MQTT connection works (if using IoT devices)

---

## 🔧 Common Issues & Fixes

### Issue 1: "Database connection failed"
**Solution:**
```powershell
# Check RDS security group allows connections from Elastic Beanstalk
# Update RDS security group to allow inbound on port 5432 from EB security group
```

### Issue 2: "Google OAuth redirect_uri_mismatch"
**Solution:**
1. Go to Google Cloud Console
2. Update Authorized redirect URIs to include:
   - `https://YOUR_DOMAIN.com/api/auth/google/callback`

### Issue 3: "Cannot read environment variables"
**Solution:**
```powershell
# Re-set environment variables
eb setenv KEY=VALUE

# Restart environment
eb restart
```

### Issue 4: "Frontend shows 'Network Error'"
**Solution:**
- Update `VITE_API_URL` in frontend `.env.production`
- Rebuild frontend: `npm run build`
- Re-upload to S3: `aws s3 sync dist/ s3://YOUR_BUCKET --delete`

---

## 📊 Monitoring Your Deployment

### View Logs:
```powershell
# Backend logs
eb logs

# Follow logs in real-time
eb logs --stream
```

### Monitor Health:
```powershell
# Check application health
eb health

# View metrics in AWS Console
# https://console.aws.amazon.com/elasticbeanstalk/
```

### Set Up Alarms:
```powershell
# Create CloudWatch alarm for high CPU
aws cloudwatch put-metric-alarm `
  --alarm-name cropwise-high-cpu `
  --alarm-description "Alert when CPU exceeds 80%" `
  --metric-name CPUUtilization `
  --namespace AWS/ElasticBeanstalk `
  --statistic Average `
  --period 300 `
  --threshold 80 `
  --comparison-operator GreaterThanThreshold `
  --evaluation-periods 2
```

---

## 💰 Cost Breakdown (Monthly)

| Service | Configuration | Cost |
|---------|--------------|------|
| **Elastic Beanstalk** | t3.small instance | $15-30 |
| **RDS PostgreSQL** | db.t3.micro | $15-25 |
| **ElastiCache Redis** | cache.t3.micro | $12-20 |
| **S3** | Frontend hosting | $1-5 |
| **CloudFront** | CDN | $1-10 |
| **Route 53** | DNS | $0.50 |
| **Data Transfer** | Outbound | $5-20 |
| **Total** | | **$50-110/month** |

**To reduce costs:**
- Use AWS Free Tier (first 12 months)
- Scale down during off-hours
- Use Reserved Instances for predictable workloads

---

## 🚀 Quick Commands Reference

```powershell
# Deploy backend
cd backend
eb deploy

# View logs
eb logs

# SSH into instance
eb ssh

# Check status
eb status

# Restart application
eb restart

# Deploy frontend
cd frontend
npm run build
aws s3 sync dist/ s3://YOUR_BUCKET --delete

# Invalidate CloudFront cache (if using CDN)
aws cloudfront create-invalidation `
  --distribution-id YOUR_DISTRIBUTION_ID `
  --paths "/*"
```

---

## 📚 Next Steps After Deployment

1. **Set up CI/CD Pipeline** (GitHub Actions or AWS CodePipeline)
2. **Configure Backups** (RDS automated backups + snapshots)
3. **Set up Monitoring** (CloudWatch dashboards + alerts)
4. **Enable SSL/TLS** (AWS Certificate Manager)
5. **Configure Auto-Scaling** (based on CPU/memory metrics)
6. **Set up AWS IoT Core** (for ESP32/Raspberry Pi devices)
7. **Implement Rate Limiting** (AWS WAF)
8. **Add DDoS Protection** (AWS Shield)

---

## 🆘 Need Help?

- **AWS Support**: https://console.aws.amazon.com/support/
- **CropWise Documentation**: `docs/AWS_DEPLOYMENT_GUIDE.md`
- **AWS Free Tier**: https://aws.amazon.com/free/
- **Elastic Beanstalk Docs**: https://docs.aws.amazon.com/elasticbeanstalk/

---

## ✅ Deployment Complete!

Your CropWise platform is now live on AWS! 🎉

**Your URLs:**
- **Backend API**: `http://YOUR_BACKEND_URL.elasticbeanstalk.com`
- **Frontend App**: `http://YOUR_BUCKET.s3-website-ap-south-1.amazonaws.com`
- **API Docs**: `http://YOUR_FRONTEND_URL/api-docs`

**Share with your team and start managing your smart farm! 🌱🚀**

---

**Ready to deploy? Let's start with Step 1!** 🚀

