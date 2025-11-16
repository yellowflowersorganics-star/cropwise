# ⚡ Quick Start with AWS Elastic Beanstalk (No Domain Required)

**Fastest way to deploy CropWise** - Setup in 15 minutes!

---

## 🎯 What is Elastic Beanstalk?

AWS Elastic Beanstalk automatically creates and manages all infrastructure for you:
- ✅ Load Balancer (automatic)
- ✅ Auto Scaling (automatic)
- ✅ Health Monitoring (automatic)
- ✅ Log Management (automatic)
- ✅ SSL Certificates (automatic with domain)

**You just upload your code, AWS handles everything else!**

---

## 🚀 Quick Setup (15 Minutes)

### **STEP 1: Install EB CLI**

```powershell
# Install Python (if not installed)
# Download from: https://www.python.org/downloads/

# Install EB CLI
pip install awsebcli --upgrade --user

# Verify installation
eb --version
# Should show: EB CLI 3.x.x
```

### **STEP 2: Initialize Elastic Beanstalk**

```powershell
cd C:\Users\praghav\cropwise\backend

# Initialize EB in your project
eb init

# Follow prompts:
# - Select region: ap-south-1
# - Application name: cropwise-backend
# - Platform: Docker
# - Do you want to set up SSH: No (or Yes if you want access)
```

### **STEP 3: Create Environment**

```powershell
# Create development environment
eb create cropwise-dev `
  --instance-type t3.small `
  --envvars `
    DATABASE_URL=$env:DATABASE_URL,`
    JWT_SECRET=$env:JWT_SECRET,`
    SESSION_SECRET=$env:SESSION_SECRET,`
    GOOGLE_CLIENT_ID=$env:GOOGLE_CLIENT_ID,`
    GOOGLE_CLIENT_SECRET=$env:GOOGLE_CLIENT_SECRET

# Wait 5-10 minutes for environment creation
```

This command creates:
- ✅ EC2 instances
- ✅ Load Balancer  
- ✅ Auto Scaling group
- ✅ Security groups
- ✅ CloudWatch monitoring
- ✅ All networking

### **STEP 4: Get Your Backend URL**

After creation, you'll see:
```
Environment created: cropwise-dev
URL: cropwise-dev.ap-south-1.elasticbeanstalk.com
```

**This is your backend API URL!**

```powershell
# Test it
curl http://cropwise-dev.ap-south-1.elasticbeanstalk.com/health
```

### **STEP 5: Deploy Frontend to S3**

```powershell
cd C:\Users\praghav\cropwise\frontend

# Build frontend with backend URL
$env:VITE_API_URL="http://cropwise-dev.ap-south-1.elasticbeanstalk.com"
npm run build

# Create S3 bucket
aws s3 mb s3://cropwise-frontend-dev

# Upload frontend
aws s3 sync dist/ s3://cropwise-frontend-dev --delete

# Enable static website hosting
aws s3 website s3://cropwise-frontend-dev `
  --index-document index.html `
  --error-document index.html

# Make bucket public
$policy = @"
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "PublicReadGetObject",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::cropwise-frontend-dev/*"
  }]
}
"@

Set-Content -Path policy.json -Value $policy
aws s3api put-bucket-policy --bucket cropwise-frontend-dev --policy file://policy.json
Remove-Item policy.json
```

### **STEP 6: Get Your Frontend URL**

```powershell
# Get S3 website URL
aws s3api get-bucket-website --bucket cropwise-frontend-dev

# Your URL will be:
# http://cropwise-frontend-dev.s3-website-ap-south-1.amazonaws.com
```

---

## ✅ Update GitHub Secrets

Add your URLs to GitHub Secrets:

```
Name: VITE_API_URL
Value: http://cropwise-dev.ap-south-1.elasticbeanstalk.com

Name: FRONTEND_URL (if needed)
Value: http://cropwise-frontend-dev.s3-website-ap-south-1.amazonaws.com
```

👉 https://github.com/yellowflowersorganics-star/cropwise/settings/secrets/actions/new

---

## 🔐 Update Google OAuth

Add your Elastic Beanstalk URLs:

1. Go to: https://console.cloud.google.com/apis/credentials
2. Click your OAuth 2.0 Client ID
3. **Add Authorized JavaScript Origins:**
   ```
   http://cropwise-frontend-dev.s3-website-ap-south-1.amazonaws.com
   http://localhost:8080
   ```
4. **Add Authorized Redirect URIs:**
   ```
   http://cropwise-dev.ap-south-1.elasticbeanstalk.com/api/auth/google/callback
   http://cropwise-frontend-dev.s3-website-ap-south-1.amazonaws.com/auth/google/callback
   http://localhost:3000/api/auth/google/callback
   ```
5. Click **"Save"**

---

## 🎯 Test Your Deployment

```powershell
# Test backend
curl http://cropwise-dev.ap-south-1.elasticbeanstalk.com/health

# Open frontend
start http://cropwise-frontend-dev.s3-website-ap-south-1.amazonaws.com

# Check logs
eb logs
```

---

## 🔄 Future Deployments

Super simple:

```powershell
cd C:\Users\praghav\cropwise\backend

# Deploy backend
git commit -am "Update backend"
eb deploy

# Deploy frontend
cd ../frontend
npm run build
aws s3 sync dist/ s3://cropwise-frontend-dev --delete
```

---

## 💰 Cost with Elastic Beanstalk

```
Elastic Beanstalk: $0 (free)
EC2 t3.small: ~$30/month
Load Balancer: ~$18/month  
RDS db.t3.micro: ~$15/month
S3 + Data Transfer: ~$5/month

Total: ~$70/month (much simpler to manage!)
```

---

## 📊 Elastic Beanstalk vs Manual ECS Setup

| Feature | Elastic Beanstalk | Manual ECS |
|---------|-------------------|------------|
| Setup Time | 15 minutes ⚡ | 45 minutes 🐌 |
| Complexity | Very Easy ✅ | Complex ❌ |
| Auto Scaling | Built-in ✅ | Manual setup ❌ |
| Health Monitoring | Automatic ✅ | Manual setup ❌ |
| Rolling Updates | Automatic ✅ | Manual setup ❌ |
| Cost | ~$70/month 💰 | ~$55/month 💰 |
| Control | Less control | More control |

**Recommendation**: Use Elastic Beanstalk for simplicity!

---

## 🚀 Production Setup

When ready for production:

```powershell
# Create production environment
eb create cropwise-prod `
  --instance-type t3.medium `
  --scale 2 `
  --envvars `
    DATABASE_URL=$env:DATABASE_URL_PROD,`
    JWT_SECRET=$env:JWT_SECRET,`
    SESSION_SECRET=$env:SESSION_SECRET

# Create production frontend bucket
aws s3 mb s3://cropwise-frontend-prod
```

---

**Created**: November 14, 2025  
**Recommended For**: Easiest deployment path

