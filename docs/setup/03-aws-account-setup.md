# ☁️ Step 3: AWS Account Setup

Complete guide to set up and configure your AWS account for CropWise.

**Prerequisites**: [Step 2 (Local Development)](02-local-development.md) completed

**Time Required**: 15-30 minutes

**Cost**: Free tier eligible (first 12 months)

---

## 📋 What You'll Accomplish

By the end of this guide, you'll have:
- ✅ AWS account created and verified
- ✅ IAM user with appropriate permissions
- ✅ AWS CLI configured locally
- ✅ Billing alerts set up
- ✅ Cost monitoring dashboard configured
- ✅ Ready to create AWS infrastructure

---

## 🛠️ Step-by-Step Instructions

### 1. Create AWS Account

#### **1.1 Sign Up**

1. Go to https://aws.amazon.com
2. Click "Create an AWS Account"
3. Fill in:
   - Email address (use your organization email)
   - Password
   - AWS account name: `cropwise-production` (or your company name)

4. **Contact Information**:
   - Account type: **Professional** (recommended) or Personal
   - Fill in business details

5. **Payment Information**:
   - Add credit/debit card
   - ⚠️ Note: AWS will charge $1 for verification (refunded)

6. **Identity Verification**:
   - Enter phone number
   - Verify with code

7. **Support Plan**:
   - Select **Basic Support (Free)** for now
   - You can upgrade later if needed

8. **Wait for account activation** (usually 5-10 minutes)

---

### 2. Secure Root Account

#### **2.1 Enable MFA on Root Account**

**Why?** Root account has unlimited access - must be protected!

1. Sign in to AWS Console: https://console.aws.amazon.com
2. Click your account name (top right) → **Security Credentials**
3. Find **Multi-factor authentication (MFA)**
4. Click **Activate MFA**
5. Choose MFA device type:
   - **Virtual MFA device** (recommended)
   - Apps: Google Authenticator, Authy, Microsoft Authenticator

6. Follow setup wizard:
   - Scan QR code with authenticator app
   - Enter two consecutive MFA codes
   - Click **Assign MFA**

7. ✅ Verify: You should see "MFA device assigned"

---

#### **2.2 Set Account Alias**

Make your sign-in URL easier to remember:

1. Go to IAM Dashboard: https://console.aws.amazon.com/iam
2. Under "AWS Account", find **Account Alias**
3. Click **Create**
4. Enter: `cropwise-prod` (or your preferred alias)
5. Your new sign-in URL: `https://cropwise-prod.signin.aws.amazon.com/console`

---

### 3. Create IAM Admin User

**⚠️ Important**: Never use root account for daily operations!

#### **3.1 Create IAM User**

1. Go to IAM → Users: https://console.aws.amazon.com/iam/home#/users
2. Click **Add users**
3. **User details**:
   - User name: `cropwise-admin`
   - ✅ Select AWS credentials type: **Password** (for console access)
   - ✅ Select: **Access key** (for CLI/API)

4. **Set permissions**:
   - Select **Attach policies directly**
   - Search and check: `AdministratorAccess`
   - (Later you can create more restrictive policies)

5. **Tags** (optional but recommended):
   - Key: `Environment`, Value: `production`
   - Key: `Project`, Value: `CropWise`
   - Key: `Role`, Value: `Admin`

6. **Review and create**
7. ⚠️ **Download credentials**:
   - Save the `.csv` file securely
   - Or note down:
     - Access key ID
     - Secret access key
     - Console password

---

#### **3.2 Enable MFA for IAM User**

1. Sign in as IAM user: `https://cropwise-prod.signin.aws.amazon.com/console`
   - Account ID: `cropwise-prod`
   - IAM user name: `cropwise-admin`
   - Password: (from previous step)

2. Go to IAM → Users → `cropwise-admin`
3. Click **Security credentials** tab
4. Under **Multi-factor authentication (MFA)**, click **Assign MFA device**
5. Follow the same process as root account MFA

---

### 4. Install and Configure AWS CLI

#### **4.1 Install AWS CLI**

**Windows**:
```powershell
# Using MSI installer
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi

# Or using Chocolatey
choco install awscli -y
```

**macOS**:
```bash
brew install awscli
```

**Linux**:
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

**Verify installation**:
```bash
aws --version
# Should show: aws-cli/2.x.x or higher
```

---

#### **4.2 Configure AWS CLI**

```bash
aws configure
```

Enter your credentials:
```
AWS Access Key ID [None]: YOUR_ACCESS_KEY_ID
AWS Secret Access Key [None]: YOUR_SECRET_ACCESS_KEY
Default region name [None]: ap-south-1
Default output format [None]: json
```

**Verify configuration**:
```bash
# Test AWS CLI
aws sts get-caller-identity

# Should return:
# {
#     "UserId": "AIDXXXXXXXXXXXXXXXXXX",
#     "Account": "123456789012",
#     "Arn": "arn:aws:iam::123456789012:user/cropwise-admin"
# }
```

---

#### **4.3 Configure MFA for CLI** (Optional but recommended)

If your IAM user has MFA enabled, you'll need a session token:

Create `~/.aws/credentials` with:
```ini
[default]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_SECRET_KEY

[cropwise-mfa]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_SECRET_KEY
aws_session_token = (will be generated)
```

Get temporary credentials with MFA:
```bash
aws sts get-session-token \
  --serial-number arn:aws:iam::ACCOUNT_ID:mfa/cropwise-admin \
  --token-code 123456 \
  --duration-seconds 129600

# Use the returned credentials
```

---

### 5. Set Up Billing Alerts

**Why?** Avoid unexpected charges!

#### **5.1 Enable Billing Alerts**

1. Sign in as root account
2. Go to **Billing Dashboard**: https://console.aws.amazon.com/billing
3. Click **Billing Preferences** (left sidebar)
4. Check the following boxes:
   - ✅ Receive PDF Invoice By Email
   - ✅ Receive Free Tier Usage Alerts
   - ✅ Receive Billing Alerts
5. Enter email address
6. Click **Save preferences**

---

#### **5.2 Create CloudWatch Billing Alarm**

1. Go to CloudWatch: https://console.aws.amazon.com/cloudwatch
2. **Important**: Change region to **US East (N. Virginia) us-east-1**
   - (Billing metrics only available in us-east-1)
3. Go to **Alarms** → **Billing**
4. Click **Create alarm**
5. **Specify metric**:
   - Metric name: **EstimatedCharges**
   - Currency: **USD**
   - Statistic: **Maximum**
   - Period: **6 hours**

6. **Conditions**:
   - Threshold type: **Static**
   - Whenever EstimatedCharges is: **Greater**
   - Than: `10` (or your preferred threshold)

7. **Configure actions**:
   - Alarm state trigger: **In alarm**
   - Send notification to: **Create new topic**
   - Topic name: `cropwise-billing-alerts`
   - Email: Your email address
   - Click **Create topic**

8. **Name the alarm**:
   - Name: `CropWise-Billing-Alert-$10`
   - Description: `Alert when monthly bill exceeds $10`

9. Click **Create alarm**
10. ✅ Check your email and **confirm the SNS subscription**

---

#### **5.3 Create Budget**

1. Go to AWS Budgets: https://console.aws.amazon.com/billing/home#/budgets
2. Click **Create budget**
3. **Budget setup**:
   - Budget type: **Cost budget**
   - Period: **Monthly**
   - Budgeting method: **Fixed**
   - Budget amount: `$50` (adjust based on your needs)

4. **Budget scope**:
   - Filter by: **All AWS services** (or specific services)

5. **Configure alerts**:
   - Alert 1: When actual cost is > 80% of budgeted amount
   - Alert 2: When actual cost is > 100% of budgeted amount
   - Alert 3: When forecasted cost is > 100% of budgeted amount

6. Email recipients: Your email
7. Click **Create budget**

---

### 6. Enable AWS Cost Explorer

1. Go to **Cost Management**: https://console.aws.amazon.com/cost-management
2. Click **Cost Explorer**
3. Click **Enable Cost Explorer** (if not already enabled)
4. Wait 24 hours for data to populate

**Cost Explorer helps you:**
- Visualize spending over time
- Identify cost drivers
- Forecast future costs
- Find optimization opportunities

---

### 7. Create Additional IAM Users (Optional)

For team members, create users with limited permissions:

#### **Developer User** (Read-only + Deploy permissions)

```bash
# Create developer user
aws iam create-user --user-name cropwise-developer

# Attach policies
aws iam attach-user-policy \
  --user-name cropwise-developer \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

aws iam attach-user-policy \
  --user-name cropwise-developer \
  --policy-arn arn:aws:iam::aws:policy/AmazonECS_FullAccess
```

#### **CI/CD User** (For GitHub Actions)

```bash
# Create CI/CD user
aws iam create-user --user-name cropwise-github-actions

# Create access key
aws iam create-access-key --user-name cropwise-github-actions

# Attach minimal required policies
aws iam attach-user-policy \
  --user-name cropwise-github-actions \
  --policy-arn arn:aws:iam::aws:policy/AmazonECS_FullAccess

aws iam attach-user-policy \
  --user-name cropwise-github-actions \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryPowerUser
```

---

## 📊 Understanding AWS Regions

### **What is ap-south-1?**
- **Region Name**: Asia Pacific (Mumbai)
- **Location**: India
- **Latency**: Best for India, South Asia, Middle East
- **Availability Zones**: 3 (ap-south-1a, ap-south-1b, ap-south-1c)

### **Why ap-south-1?**
- ✅ Low latency for Indian users
- ✅ Data sovereignty (data stays in India)
- ✅ All required services available
- ✅ Competitive pricing

### **Resources per Region**
Most AWS resources are region-specific:
- ✅ EC2, ECS, RDS, S3: Region-specific
- ❌ IAM, CloudFront, Route53: Global

---

## 💰 Cost Estimation

### **Free Tier (First 12 Months)**
- EC2: 750 hours/month of t2.micro
- RDS: 750 hours/month of db.t2.micro
- S3: 5 GB storage
- CloudFront: 50 GB data transfer
- Lambda: 1M requests/month

### **Estimated Monthly Costs (After Free Tier)**

**Development Environment**:
- ECS Fargate (backend): $15-20
- RDS db.t3.micro: $15-20
- S3 + CloudFront: $5-10
- **Total**: ~$35-50/month

**Staging + Production**:
- Add ~$100-150/month for each environment
- **Total for 3 environments**: $250-350/month

### **Cost Optimization Tips**
1. Use Free Tier first
2. Start with smallest instance sizes
3. Use Spot Instances for non-critical workloads
4. Enable auto-shutdown for dev environments
5. Use S3 Intelligent-Tiering
6. Clean up unused resources regularly

---

## ✅ Verification Checklist

Before moving to the next step, ensure:

- [ ] AWS account created and verified
- [ ] Root account MFA enabled
- [ ] IAM admin user created with MFA
- [ ] AWS CLI installed and configured
- [ ] `aws sts get-caller-identity` works
- [ ] Billing alerts configured
- [ ] Budget created ($50 or your amount)
- [ ] Cost Explorer enabled
- [ ] Account alias set up
- [ ] Credentials saved securely
- [ ] Region set to `ap-south-1`

---

## 🎉 What's Next?

Your AWS account is ready for infrastructure setup!

**Next Step**: [Step 4: AWS Infrastructure (Development)](04-aws-infrastructure-dev.md)

---

## 📚 Related Documentation

- [AWS Infrastructure Setup](../../AWS_INFRASTRUCTURE_SETUP.md) - Detailed AWS setup
- [AWS Security Checklist](../../AWS_SECURITY_CHECKLIST.md) - Security best practices
- [Deployment Guide](../deployment/DEPLOYMENT_GUIDE.md) - Full deployment process

---

## 🛑 Troubleshooting

### **Problem: "Access key doesn't work"**

```bash
# Verify credentials
aws configure list

# Test with STS
aws sts get-caller-identity

# If fails, reconfigure
aws configure
```

### **Problem: "Billing alerts not working"**

- Ensure you're in us-east-1 region for CloudWatch alarms
- Verify SNS subscription is confirmed (check email)
- Wait 24 hours for billing data to populate

### **Problem: "Permission denied"**

- Ensure IAM user has `AdministratorAccess` policy attached
- Check if MFA is required for the policy
- Verify access keys are correct

---

**Last Updated**: November 16, 2025

