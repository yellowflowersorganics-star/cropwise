# 🚀 CropWise - AWS Quick Start (5 Minutes)

Get CropWise running on AWS in 5 minutes!

## Prerequisites

```bash
# 1. Install AWS CLI
pip install awscli --upgrade

# 2. Configure AWS
aws configure
# Enter: Access Key, Secret Key, Region (ap-south-1), Output (json)

# 3. Install Elastic Beanstalk CLI
pip install awsebcli --upgrade
```

## Option 1: Elastic Beanstalk (Fastest - 5 minutes)

### Step 1: Run Setup Script
```bash
chmod +x scripts/eb-deploy.sh
./scripts/eb-deploy.sh
```

### Step 2: Set Environment Variables
```bash
cd backend

eb setenv \
  DB_HOST=your-rds-endpoint \
  DB_PASSWORD=your-password \
  REDIS_HOST=your-redis-endpoint \
  JWT_SECRET=$(openssl rand -base64 32) \
  GOOGLE_CLIENT_ID=your-client-id \
  GOOGLE_CLIENT_SECRET=your-secret
```

### Step 3: Deploy
```bash
eb deploy
```

**Done!** Your app is live at: `http://YOUR-EB-URL.elasticbeanstalk.com`

---

## Option 2: CloudFormation (Complete Infrastructure - 15 minutes)

### Step 1: Deploy Infrastructure
```bash
chmod +x scripts/cloudformation-deploy.sh
./scripts/cloudformation-deploy.sh
```

This creates:
- ✅ VPC with public/private subnets
- ✅ RDS PostgreSQL
- ✅ ElastiCache Redis
- ✅ ECS Cluster
- ✅ Load Balancer
- ✅ Security Groups

### Step 2: Deploy Containers
```bash
chmod +x scripts/aws-deploy.sh
./scripts/aws-deploy.sh
```

**Done!** Your app is live at the ALB DNS name.

---

## Option 3: Manual EC2 (Full Control - 30 minutes)

### Step 1: Launch EC2
```bash
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t3.small \
  --key-name your-key \
  --security-group-ids sg-xxx \
  --user-data file://scripts/ec2-user-data.sh
```

### Step 2: SSH and Deploy
```bash
ssh -i your-key.pem ec2-user@YOUR_EC2_IP

cd /home/ec2-user
git clone https://github.com/your-repo/cropwise.git
cd cropwise/backend
npm install --production
pm2 start src/index.js
```

---

## 🎯 What You Get

After deployment:

```
Frontend:  https://your-alb-dns.amazonaws.com
Backend:   https://your-alb-dns.amazonaws.com/api
Health:    https://your-alb-dns.amazonaws.com/health
Database:  PostgreSQL (Multi-AZ)
Cache:     Redis (ElastiCache)
IoT:       AWS IoT Core (MQTT)
```

---

## 💰 Cost Estimate

**Free Tier (First Year):**
- $0-20/month

**After Free Tier:**
- Basic: ~$50/month
- Production: ~$100/month
- Enterprise: ~$300+/month

---

## 🔥 Quick Commands

```bash
# View logs
eb logs                    # Elastic Beanstalk
aws logs tail /ecs/cropwise-backend --follow  # ECS

# Update environment variables
eb setenv KEY=value

# Deploy new version
eb deploy

# Restart application
eb restart

# Delete everything
./scripts/aws-cleanup.sh
```

---

## 🆘 Troubleshooting

**App won't start?**
```bash
eb logs
# Check for: database connection, Redis connection, env vars
```

**Can't connect to database?**
```bash
# Check security group allows inbound from ECS
# Verify DB_HOST endpoint is correct
```

**High costs?**
```bash
# Use t3.micro/small instances
# Enable auto-scaling
# Use Reserved Instances (30-40% savings)
```

---

## 📚 Full Documentation

- [Complete AWS Deployment Guide](./AWS_DEPLOYMENT_GUIDE.md)
- [Google OAuth Setup](../backend/GOOGLE_OAUTH_SETUP.md)

---

**Need help?** Open an issue on GitHub!

