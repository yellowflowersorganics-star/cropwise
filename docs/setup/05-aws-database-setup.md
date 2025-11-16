# 🗄️ AWS RDS PostgreSQL Setup Guide

Complete step-by-step guide to set up PostgreSQL on AWS RDS for CropWise across all environments.

---

## 📋 Table of Contents

1. [Environment Overview](#environment-overview)
2. [Development Environment Setup](#development-environment-setup)
3. [Staging Environment Setup](#staging-environment-setup)
4. [Production Environment Setup](#production-environment-setup)
5. [Getting Your DATABASE_URLs](#getting-your-database_urls)
6. [Testing the Connection](#testing-the-connection)
7. [Security Configuration](#security-configuration)
8. [Pricing Estimates](#pricing-estimates)

---

## 🌍 Environment Overview

You'll create **3 separate databases**, one for each environment:

| Environment | Instance ID | Database Name | Instance Size | Storage | Cost/Month |
|-------------|-------------|---------------|---------------|---------|------------|
| **Development** | `cropwise-dev-db` | `cropwise_dev` | db.t3.micro | 20 GB | ~$15 or FREE |
| **Staging** | `cropwise-stage-db` | `cropwise_stage` | db.t3.small | 50 GB | ~$35 |
| **Production** | `cropwise-prod-db` | `cropwise_prod` | db.t3.medium | 100 GB | ~$100 (Multi-AZ) |

### What You'll Get:
```bash
# Three separate DATABASE_URLs for GitHub Secrets:
DATABASE_URL_DEV=postgresql://cropwise_dev_admin:DevPass123!@cropwise-dev-db.abc123.ap-south-1.rds.amazonaws.com:5432/cropwise_dev

DATABASE_URL_STAGE=postgresql://cropwise_stage_admin:StagePass123!@cropwise-stage-db.xyz456.ap-south-1.rds.amazonaws.com:5432/cropwise_stage

DATABASE_URL_PROD=postgresql://cropwise_prod_admin:ProdPass123!@cropwise-prod-db.def789.ap-south-1.rds.amazonaws.com:5432/cropwise_prod
```

---

## 🔧 Development Environment Setup

Perfect for local development and testing. Can use Free Tier if available.

### **Step 1: Navigate to RDS**

1. Go to: https://console.aws.amazon.com
2. Sign in with your AWS account
3. **Select region**: **ap-south-1** (N. Virginia)
4. In search bar, type: **RDS** → Click **RDS**
5. Click **Create database** button (orange)

---

### **Step 2: Basic Configuration**

#### **Database Creation Method:**
- ✅ **Standard create**

#### **Engine Options:**
- ✅ **Engine type**: PostgreSQL
- ✅ **Version**: PostgreSQL 15.x (latest stable)

#### **Templates:**
- ✅ **Free tier** (if available)
- OR **Dev/Test** (if free tier not available)

---

### **Step 3: Settings (Development)**

#### **DB Instance Identifier:**
```
cropwise-dev-db
```

#### **Credentials Settings:**
**Master Username:**
```
cropwise_dev_admin
```

**Master Password:**
```
CropWiseDev2025!
```
⚠️ **Save this password!** You'll need it for DATABASE_URL_DEV

---

### **Step 4: Instance Configuration (Development)**

#### **DB Instance Class:**
```
✅ db.t3.micro
- 1 vCPU, 1 GB RAM
- Free tier eligible (750 hours/month)
- Cost: FREE (first 12 months) or ~$15/month
```

#### **Storage:**
- **Storage Type**: General Purpose SSD (gp3)
- **Allocated Storage**: **20 GB**
- **Storage Autoscaling**: ✅ Enable (Max: 100 GB)

---

### **Step 5: Connectivity (Development)**

#### **Network Settings:**
- **Compute Resource**: Don't connect to EC2
- **Network Type**: IPv4
- **VPC**: Default VPC
- **DB Subnet Group**: default
- **Public Access**: ✅ **Yes** (easier for development)
- **VPC Security Group**: Create new → `cropwise-dev-db-sg`
- **Availability Zone**: No preference
- **Database Port**: 5432

---

### **Step 6: Database Authentication**

- ✅ **Password authentication**

---

### **Step 7: Monitoring (Development)**

#### **Enhanced Monitoring:**
- ⭕ **Disable** (save cost for dev)

#### **Performance Insights:**
- ⭕ **Disable** (optional for dev)

---

### **Step 8: Additional Configuration (Development)**

Click **"Additional configuration"** to expand:

#### **Database Options:**
**Initial Database Name:**
```
cropwise_dev
```
⚠️ **IMPORTANT**: Don't leave this blank!

#### **Backup:**
- **Enable automated backups**: ✅ Yes
- **Backup retention**: **7 days**
- **Backup window**: 03:00-04:00 UTC

#### **Encryption:**
- ✅ **Enable encryption**

#### **Log Exports:**
- ✅ PostgreSQL log (useful for debugging)

#### **Maintenance:**
- **Auto minor version upgrade**: ✅ Enable
- **Maintenance window**: No preference

#### **Deletion Protection:**
- ⭕ **Disable** (easier to delete/recreate in dev)

---

### **Step 9: Create Development Database**

1. Review all settings
2. **Estimated cost**: ~$15/month or FREE
3. Click **"Create database"**
4. ⏱️ Wait 5-10 minutes for status: **Available**

---

## 🎭 Staging Environment Setup

Similar to production but with reduced resources for cost optimization.

### **Step 1: Navigate to RDS**

1. Go to: https://console.aws.amazon.com/rds/
2. Click **Create database**

---

### **Step 2: Basic Configuration**

#### **Database Creation Method:**
- ✅ **Standard create**

#### **Engine Options:**
- ✅ **Engine type**: PostgreSQL
- ✅ **Version**: PostgreSQL 15.x

#### **Templates:**
- ✅ **Dev/Test** (we'll customize for staging)

---

### **Step 3: Settings (Staging)**

#### **DB Instance Identifier:**
```
cropwise-stage-db
```

#### **Credentials Settings:**
**Master Username:**
```
cropwise_stage_admin
```

**Master Password:**
```
CropWiseStage2025!Secure
```
⚠️ **Save this password!** You'll need it for DATABASE_URL_STAGE

---

### **Step 4: Instance Configuration (Staging)**

#### **DB Instance Class:**
```
✅ db.t3.small
- 2 vCPU, 2 GB RAM
- Good for staging tests with realistic load
- Cost: ~$35/month
```

#### **Storage:**
- **Storage Type**: General Purpose SSD (gp3)
- **Allocated Storage**: **50 GB**
- **Storage Autoscaling**: ✅ Enable (Max: 200 GB)

---

### **Step 5: Connectivity (Staging)**

#### **Network Settings:**
- **Compute Resource**: Don't connect to EC2
- **Network Type**: IPv4
- **VPC**: Default VPC (or your custom VPC)
- **DB Subnet Group**: default
- **Public Access**: ⭕ **No** (more secure, connect via VPC)
- **VPC Security Group**: Create new → `cropwise-stage-db-sg`
- **Availability Zone**: No preference
- **Database Port**: 5432

---

### **Step 6: Database Authentication**

- ✅ **Password authentication**

---

### **Step 7: Monitoring (Staging)**

#### **Enhanced Monitoring:**
- ✅ **Enable**
- **Granularity**: 60 seconds

#### **Performance Insights:**
- ✅ **Enable**
- **Retention**: 7 days (free)

---

### **Step 8: Additional Configuration (Staging)**

Click **"Additional configuration"** to expand:

#### **Database Options:**
**Initial Database Name:**
```
cropwise_stage
```

#### **Backup:**
- **Enable automated backups**: ✅ Yes
- **Backup retention**: **14 days** (longer than dev)
- **Backup window**: 03:00-04:00 UTC

#### **Encryption:**
- ✅ **Enable encryption**

#### **Log Exports:**
- ✅ PostgreSQL log
- ✅ Upgrade log

#### **Maintenance:**
- **Auto minor version upgrade**: ✅ Enable
- **Maintenance window**: Sunday 04:00-05:00 UTC

#### **Deletion Protection:**
- ✅ **Enable** (prevent accidental deletion)

---

### **Step 9: Create Staging Database**

1. Review all settings
2. **Estimated cost**: ~$35/month
3. Click **"Create database"**
4. ⏱️ Wait 5-10 minutes for status: **Available**

---

## 🚀 Production Environment Setup

Full production configuration with high availability and enhanced monitoring.

### **Step 1: Navigate to RDS**

1. Go to: https://console.aws.amazon.com/rds/
2. Click **Create database**

---

### **Step 2: Basic Configuration**

#### **Database Creation Method:**
- ✅ **Standard create**

#### **Engine Options:**
- ✅ **Engine type**: PostgreSQL
- ✅ **Version**: PostgreSQL 15.x

#### **Templates:**
- ✅ **Production** (enables Multi-AZ and best practices)

---

### **Step 3: Settings (Production)**

#### **DB Instance Identifier:**
```
cropwise-prod-db
```

#### **Credentials Settings:**
**Master Username:**
```
cropwise_prod_admin
```

**Master Password:**
```
Generate a STRONG password:
CropWiseProd2025!SecureP@ssw0rd#2024
```
⚠️ **CRITICAL**: Save in password manager! You'll need it for DATABASE_URL_PROD

---

### **Step 4: Instance Configuration (Production)**

#### **Availability and Durability:**
- ✅ **Multi-AZ deployment** (automatic failover)

#### **DB Instance Class:**
```
✅ db.t3.medium (or larger based on load)
- 2 vCPU, 4 GB RAM
- Good for 1000-10000 users
- Cost: ~$60/month (×2 for Multi-AZ = ~$120/month)

Consider upgrading to:
- db.t3.large (8 GB RAM) for 10000+ users
- db.m6g.large (8 GB RAM, ARM-based, better performance)
```

#### **Storage:**
- **Storage Type**: General Purpose SSD (gp3)
- **Allocated Storage**: **100 GB**
- **Storage Autoscaling**: ✅ Enable (Max: 1000 GB)
- **Provisioned IOPS**: Not needed unless you have high I/O requirements

---

### **Step 5: Connectivity (Production)**

#### **Network Settings:**
- **Compute Resource**: Don't connect to EC2
- **Network Type**: IPv4
- **VPC**: Your production VPC (or default)
- **DB Subnet Group**: default (should span multiple AZs)
- **Public Access**: ⭕ **No** (security best practice)
  - Only accessible from VPC (your ECS/EC2 instances)
  - Use bastion host or VPN for admin access
- **VPC Security Group**: Create new → `cropwise-prod-db-sg`
- **Availability Zone**: No preference (Multi-AZ handles this)
- **Database Port**: 5432

---

### **Step 6: Database Authentication**

- ✅ **Password authentication**
- Optional: Enable IAM database authentication (advanced)

---

### **Step 7: Monitoring (Production)**

#### **Enhanced Monitoring:**
- ✅ **Enable**
- **Granularity**: 60 seconds
- **Monitoring Role**: Create new role (automatic)

#### **Performance Insights:**
- ✅ **Enable**
- **Retention**: **7 days** (free) or longer (paid)

---

### **Step 8: Additional Configuration (Production)**

Click **"Additional configuration"** to expand:

#### **Database Options:**
**Initial Database Name:**
```
cropwise_prod
```

#### **DB Parameter Group:**
- Keep default: `default.postgres15`
- Optional: Create custom parameter group for optimization

#### **Backup:**
- **Enable automated backups**: ✅ Yes
- **Backup retention**: **30 days** (maximum for point-in-time recovery)
- **Backup window**: 03:00-04:00 UTC (low-traffic time)
- **Copy tags to snapshots**: ✅ Enable

#### **Encryption:**
- ✅ **Enable encryption** (required for production)
- **Master key**: Use default AWS KMS key (or custom key)

#### **Log Exports:**
- ✅ PostgreSQL log
- ✅ Upgrade log

#### **Maintenance:**
- **Auto minor version upgrade**: ✅ Enable (security patches)
- **Maintenance window**: Sunday 04:00-06:00 UTC (low-traffic period)

#### **Deletion Protection:**
- ✅ **Enable** (prevent accidental deletion!)

---

### **Step 9: Create Production Database**

1. **CAREFULLY** review all settings
2. **Estimated cost**: ~$100-120/month (Multi-AZ)
3. Click **"Create database"**
4. ⏱️ Wait 10-15 minutes for status: **Available** (Multi-AZ takes longer)

---

## 🔗 Getting Your DATABASE_URLs

You need to get the endpoint for each database and construct three separate DATABASE_URLs.

### **Step 1: Wait for All Databases to be Available**

1. Go to: https://console.aws.amazon.com/rds/home
2. Click **Databases** in left sidebar
3. Wait until all databases show Status: **Available** ✅
   - `cropwise-dev-db` → Available
   - `cropwise-stage-db` → Available
   - `cropwise-prod-db` → Available

⏱️ This may take 5-15 minutes per database.

---

### **Step 2: Get Development Endpoint**

1. Click on database name: **`cropwise-dev-db`**
2. In **Connectivity & security** tab, find:
   ```
   Endpoint: cropwise-dev-db.c9abc123xyz.ap-south-1.rds.amazonaws.com
   Port: 5432
   ```
3. Copy the endpoint

### **Step 3: Construct DATABASE_URL_DEV**

Format:
```
postgresql://[username]:[password]@[endpoint]:[port]/[database_name]
```

Your Development URL:
```bash
# Replace YOUR_DEV_ENDPOINT with the actual endpoint from Step 2
DATABASE_URL_DEV=postgresql://cropwise_dev_admin:CropWiseDev2025!@YOUR_DEV_ENDPOINT:5432/cropwise_dev

# Example:
DATABASE_URL_DEV=postgresql://cropwise_dev_admin:CropWiseDev2025!@cropwise-dev-db.c9abc123xyz.ap-south-1.rds.amazonaws.com:5432/cropwise_dev
```

⚠️ **Password URL Encoding**: If your password has special characters, encode them:
- `@` → `%40`
- `!` → `%21`
- `#` → `%23`

---

### **Step 4: Get Staging Endpoint**

1. Go back to: https://console.aws.amazon.com/rds/home → Databases
2. Click on database name: **`cropwise-stage-db`**
3. In **Connectivity & security** tab, copy the endpoint:
   ```
   Endpoint: cropwise-stage-db.d7def456abc.ap-south-1.rds.amazonaws.com
   Port: 5432
   ```

### **Step 5: Construct DATABASE_URL_STAGE**

Your Staging URL:
```bash
# Replace YOUR_STAGE_ENDPOINT with the actual endpoint
DATABASE_URL_STAGE=postgresql://cropwise_stage_admin:CropWiseStage2025!Secure@YOUR_STAGE_ENDPOINT:5432/cropwise_stage

# Example:
DATABASE_URL_STAGE=postgresql://cropwise_stage_admin:CropWiseStage2025%21Secure@cropwise-stage-db.d7def456abc.ap-south-1.rds.amazonaws.com:5432/cropwise_stage
```

Note: `!` is encoded as `%21` in the example above.

---

### **Step 6: Get Production Endpoint**

1. Go back to: https://console.aws.amazon.com/rds/home → Databases
2. Click on database name: **`cropwise-prod-db`**
3. In **Connectivity & security** tab, copy the endpoint:
   ```
   Endpoint: cropwise-prod-db.e8ghi789jkl.ap-south-1.rds.amazonaws.com
   Port: 5432
   ```
4. **Multi-AZ Note**: The endpoint is the same even with Multi-AZ. AWS handles failover automatically.

### **Step 7: Construct DATABASE_URL_PROD**

Your Production URL:
```bash
# Replace YOUR_PROD_ENDPOINT with the actual endpoint
DATABASE_URL_PROD=postgresql://cropwise_prod_admin:YOUR_STRONG_PASSWORD@YOUR_PROD_ENDPOINT:5432/cropwise_prod

# Example (with encoded special characters):
DATABASE_URL_PROD=postgresql://cropwise_prod_admin:CropWiseProd2025%21SecureP%40ssw0rd%23@cropwise-prod-db.e8ghi789jkl.ap-south-1.rds.amazonaws.com:5432/cropwise_prod
```

---

### **Step 8: Add All URLs to GitHub Secrets**

1. Go to: https://github.com/yellowflowersorganics-star/cropwise/settings/secrets/actions

**Add Development Database:**
2. Click **"New repository secret"**
3. Name: `DATABASE_URL_DEV`
4. Value: Your complete DATABASE_URL_DEV from Step 3
5. Click **"Add secret"**

**Add Staging Database:**
6. Click **"New repository secret"**
7. Name: `DATABASE_URL_STAGE`
8. Value: Your complete DATABASE_URL_STAGE from Step 5
9. Click **"Add secret"**

**Add Production Database:**
10. Click **"New repository secret"**
11. Name: `DATABASE_URL_PROD`
12. Value: Your complete DATABASE_URL_PROD from Step 7
13. Click **"Add secret"**

### **Step 9: Verify GitHub Secrets**

Your GitHub Secrets should now have:
```
✅ DATABASE_URL_DEV   → Development database
✅ DATABASE_URL_STAGE → Staging database
✅ DATABASE_URL_PROD  → Production database
```

---

## 🔒 Security Configuration

Each environment needs proper security group configuration.

### **Development Database Security Group**

Your development database (`cropwise-dev-db`) has **Public Access: Yes**, so you need to configure allowed IPs.

1. Go to: https://console.aws.amazon.com/rds/home → Databases
2. Click: **`cropwise-dev-db`**
3. Under **Connectivity & security**, click the VPC security group: **`cropwise-dev-db-sg`**
4. Click **Inbound rules** tab
5. Click **Edit inbound rules**
6. Add rule:

```
Type: PostgreSQL
Protocol: TCP
Port: 5432
Source Type: My IP
Source: (Auto-populated with your current IP)
Description: My development machine
```

7. Click **Save rules**

**To find your IP manually**: https://whatismyipaddress.com

**Multiple developers?** Add one rule per developer:
```
Rule 1: 123.45.67.89/32 - Developer 1
Rule 2: 98.76.54.32/32 - Developer 2
Rule 3: 111.222.333.444/32 - Developer 3
```

⚠️ **Never use 0.0.0.0/0** for development (anyone can connect!)

---

### **Staging Database Security Group**

Your staging database (`cropwise-stage-db`) has **Public Access: No** (private).

1. Go to: https://console.aws.amazon.com/rds/home → Databases
2. Click: **`cropwise-stage-db`**
3. Under **Connectivity & security**, click the VPC security group: **`cropwise-stage-db-sg`**
4. Click **Inbound rules** tab
5. Click **Edit inbound rules**
6. Add rules:

#### **Rule 1: Allow ECS Tasks**
```
Type: PostgreSQL
Protocol: TCP
Port: 5432
Source Type: Security Group
Source: [Your ECS task security group ID] (e.g., sg-0abc123def456)
Description: CropWise staging ECS tasks
```

#### **Rule 2: Allow Bastion Host (Optional, for admin access)**
```
Type: PostgreSQL
Protocol: TCP
Port: 5432
Source Type: Security Group
Source: [Your bastion host security group ID]
Description: Bastion host for admin access
```

7. Click **Save rules**

**How to find ECS security group:**
- Go to ECS → Clusters → `cropwise-stage-cluster` → Services
- Click your service → Tasks → Click a task
- Under **Network** section, find Security Group ID

---

### **Production Database Security Group**

Your production database (`cropwise-prod-db`) has **Public Access: No** (private, most secure).

1. Go to: https://console.aws.amazon.com/rds/home → Databases
2. Click: **`cropwise-prod-db`**
3. Under **Connectivity & security**, click the VPC security group: **`cropwise-prod-db-sg`**
4. Click **Inbound rules** tab
5. Click **Edit inbound rules**
6. Add rules:

#### **Rule 1: Allow Production ECS Tasks**
```
Type: PostgreSQL
Protocol: TCP
Port: 5432
Source Type: Security Group
Source: [Production ECS task security group ID]
Description: CropWise production ECS tasks ONLY
```

#### **Rule 2: Allow Bastion/VPN (Emergency Admin Access)**
```
Type: PostgreSQL
Protocol: TCP
Port: 5432
Source Type: Security Group
Source: [Bastion security group ID]
Description: Emergency admin access via bastion
```

7. Click **Save rules**

⚠️ **Production Security Best Practices:**
- **NO** direct IP addresses allowed
- **NO** public access
- **ONLY** allow specific security groups (ECS tasks, bastion)
- **ENABLE** VPC Flow Logs for monitoring
- **USE** AWS Secrets Manager for credentials rotation

---

### **Security Group Summary**

| Environment | Public Access | Allowed Sources |
|-------------|---------------|-----------------|
| **Development** | ✅ Yes | Your IP address(es) |
| **Staging** | ❌ No | ECS security group + Bastion |
| **Production** | ❌ No | ECS security group + Bastion (emergency only) |

---

## 🧪 Testing the Connection

Test each database separately to ensure all three environments work correctly.

### **Test Development Database**

#### **Option 1: Using psql (Command Line)**

```powershell
# Install PostgreSQL client (if not installed)
# Download from: https://www.postgresql.org/download/windows/

# Connect to DEVELOPMENT database
psql "postgresql://cropwise_dev_admin:CropWiseDev2025!@YOUR_DEV_ENDPOINT:5432/cropwise_dev"

# Should see:
# cropwise_dev=>

# Test query:
SELECT 'Development DB Connected!' as message, NOW() as connected_at;

# Exit:
\q
```

#### **Option 2: Using pgAdmin (GUI)**

1. Download pgAdmin: https://www.pgadmin.org/download/
2. Install and open pgAdmin
3. Right-click **Servers** → **Register** → **Server**
4. **General** tab:
   - Name: `CropWise Development`
5. **Connection** tab:
   - Host: `YOUR_DEV_ENDPOINT`
   - Port: `5432`
   - Database: `cropwise_dev`
   - Username: `cropwise_dev_admin`
   - Password: `CropWiseDev2025!`
   - Save password: ✅
6. Click **Save**

---

### **Test Staging Database**

⚠️ **Note**: Staging has **Public Access: No**, so you can only connect from:
- Within the VPC (ECS tasks)
- Via bastion host
- After temporarily enabling public access

#### **Temporary Public Access for Testing (Optional)**

1. Go to RDS Console → `cropwise-stage-db`
2. Click **Modify**
3. Change **Public access** to **Yes**
4. Click **Continue** → **Apply immediately**
5. Wait 2-3 minutes

#### **Test Connection:**

```powershell
# Connect to STAGING database
psql "postgresql://cropwise_stage_admin:CropWiseStage2025!Secure@YOUR_STAGE_ENDPOINT:5432/cropwise_stage"

# Should see:
# cropwise_stage=>

# Test query:
SELECT 'Staging DB Connected!' as message, NOW() as connected_at;
```

#### **After Testing:**
1. Go back to RDS Console → Modify
2. Change **Public access** back to **No**
3. Apply immediately

---

### **Test Production Database**

⚠️ **IMPORTANT**: Production has **Public Access: No** for security.

**DO NOT enable public access on production!**

#### **Option 1: Test from ECS Task (Recommended)**

Once your backend is deployed to ECS:

```bash
# SSH into ECS task or check logs
# Your backend should connect automatically using DATABASE_URL_PROD

# Look for successful connection logs
```

#### **Option 2: Test via Bastion Host**

If you have a bastion host set up:

```bash
# SSH to bastion host
ssh -i your-key.pem ec2-user@bastion-ip

# From bastion, connect to production DB
psql "postgresql://cropwise_prod_admin:YOUR_PROD_PASSWORD@YOUR_PROD_ENDPOINT:5432/cropwise_prod"
```

#### **Option 3: Using AWS Systems Manager Session Manager**

1. Use SSM Session Manager to connect to an EC2 instance in the same VPC
2. Install PostgreSQL client
3. Test connection from there

---

### **Node.js Test Script (All Environments)**

Create `test-all-databases.js`:

```javascript
// test-all-databases.js
const { Client } = require('pg');

const databases = [
  {
    name: 'Development',
    url: process.env.DATABASE_URL_DEV || 'postgresql://cropwise_dev_admin:CropWiseDev2025!@YOUR_DEV_ENDPOINT:5432/cropwise_dev'
  },
  {
    name: 'Staging',
    url: process.env.DATABASE_URL_STAGE || 'postgresql://cropwise_stage_admin:CropWiseStage2025!Secure@YOUR_STAGE_ENDPOINT:5432/cropwise_stage'
  },
  {
    name: 'Production',
    url: process.env.DATABASE_URL_PROD // Only from ECS
  }
];

async function testDatabase(name, connectionString) {
  if (!connectionString) {
    console.log(`⏭️  Skipping ${name} (no connection string)`);
    return;
  }

  const client = new Client({
    connectionString,
    ssl: {
      rejectUnauthorized: false // Required for RDS
    }
  });

  try {
    console.log(`\n🔄 Testing ${name} database...`);
    await client.connect();
    console.log(`✅ ${name}: Connected successfully!`);
    
    const result = await client.query('SELECT NOW() as time, version() as version');
    console.log(`📅 ${name}: Server time:`, result.rows[0].time);
    console.log(`🗄️  ${name}: PostgreSQL:`, result.rows[0].version.split(' ')[1]);
    
    await client.end();
    console.log(`✅ ${name}: Test completed!`);
  } catch (err) {
    console.error(`❌ ${name}: Connection failed:`, err.message);
  }
}

async function testAll() {
  console.log('🧪 Testing all CropWise databases...\n');
  
  for (const db of databases) {
    await testDatabase(db.name, db.url);
  }
  
  console.log('\n✅ All tests completed!');
}

testAll();
```

**Run the test:**

```powershell
cd C:\Users\praghav\smartcrop-os\backend

# Test development only
$env:DATABASE_URL_DEV="postgresql://cropwise_dev_admin:CropWiseDev2025!@YOUR_DEV_ENDPOINT:5432/cropwise_dev"
node test-all-databases.js

# Test all (if you have access)
$env:DATABASE_URL_DEV="..."
$env:DATABASE_URL_STAGE="..."
$env:DATABASE_URL_PROD="..."
node test-all-databases.js
```

---

### **Expected Output:**

```
🧪 Testing all CropWise databases...

🔄 Testing Development database...
✅ Development: Connected successfully!
📅 Development: Server time: 2025-11-16T10:30:45.123Z
🗄️  Development: PostgreSQL: 15.5
✅ Development: Test completed!

🔄 Testing Staging database...
✅ Staging: Connected successfully!
📅 Staging: Server time: 2025-11-16T10:30:46.456Z
🗄️  Staging: PostgreSQL: 15.5
✅ Staging: Test completed!

⏭️  Skipping Production (no connection string)

✅ All tests completed!
```

---

## 💰 Pricing Estimates

### **Complete Cost Breakdown (All 3 Environments)**

#### **Development Environment**
```
Instance: db.t3.micro (1 vCPU, 1 GB RAM)
  - 730 hours/month × $0.017/hour = $12.41
Storage: 20 GB GP3 SSD
  - 20 GB × $0.115/GB = $2.30
Backups: 20 GB (7 days retention)
  - FREE (equal to storage size)
Monitoring: Disabled (save cost)
  - $0
────────────────────────────────────────
Development Total: ~$15/month
OR FREE (if using Free Tier)
```

#### **Staging Environment**
```
Instance: db.t3.small (2 vCPU, 2 GB RAM)
  - 730 hours/month × $0.034/hour = $24.82
Storage: 50 GB GP3 SSD
  - 50 GB × $0.115/GB = $5.75
Backups: 50 GB (14 days retention)
  - ~25 GB extra × $0.095/GB = $2.38
Enhanced Monitoring: 60 seconds
  - $1.50
Performance Insights: 7 days (free tier)
  - FREE
────────────────────────────────────────
Staging Total: ~$35/month
```

#### **Production Environment (Multi-AZ)**
```
Instance: db.t3.medium Multi-AZ (2 vCPU, 4 GB RAM)
  - 730 hours/month × $0.068/hour × 2 (Multi-AZ) = $99.28
Storage: 100 GB GP3 SSD
  - 100 GB × $0.115/GB × 2 (Multi-AZ) = $23.00
Backups: 100 GB (30 days retention)
  - ~100 GB extra × $0.095/GB = $9.50
Enhanced Monitoring: 60 seconds
  - $1.50
Performance Insights: 7 days
  - FREE
────────────────────────────────────────
Production Total: ~$133/month
```

#### **Total Cost for All 3 Environments**
```
Development:  $15/month (or FREE)
Staging:      $35/month
Production:   $133/month
────────────────────────────────────────
TOTAL:        ~$183/month
              ~$2,196/year
```

---

### **Free Tier Benefits (First 12 Months)**

If your AWS account is less than 12 months old:

```
✅ db.t3.micro instance (750 hours/month) → FREE
✅ 20 GB General Purpose SSD → FREE
✅ 20 GB backup storage → FREE

Your Development database will be FREE for 750 hours/month!
(That's 24/7 if you run only one t3.micro instance)

Adjusted Total with Free Tier:
Development:  $0/month (FREE TIER)
Staging:      $35/month
Production:   $133/month
────────────────────────────────────────
TOTAL:        ~$168/month
              ~$2,016/year
```

---

### **Cost Optimization Tips**

#### **1. Start with Smaller Production Instance**
```
Instead of: db.t3.medium Multi-AZ → ~$133/month
Use: db.t3.small Multi-AZ → ~$80/month
Savings: ~$53/month ($636/year)
```

#### **2. Use Single-AZ for Early Production**
```
Instead of: db.t3.medium Multi-AZ → ~$133/month
Use: db.t3.medium Single-AZ → ~$67/month
Savings: ~$66/month ($792/year)

⚠️ Trade-off: Lower availability, manual failover
Add Multi-AZ when you have paying customers
```

#### **3. Reduce Backup Retention**
```
Production: 30 days → 14 days
Savings: ~$5/month ($60/year)
```

#### **4. Use Reserved Instances (1-3 year commitment)**
```
On-Demand: db.t3.medium × 730 hours = $49.64/month
Reserved (1-year, no upfront): $32.12/month
Savings: ~$17.52/month ($210/year)

Reserved (1-year, all upfront): $28.47/month
Savings: ~$21.17/month ($254/year)
```

#### **5. Set Up Billing Alerts**

1. Go to AWS Billing Console
2. Create Budget:
   - Budget name: `RDS Monthly Budget`
   - Amount: `$200`
   - Alerts at: 80% ($160), 100% ($200), 120% ($240)

#### **6. Delete Non-Production Databases When Not Needed**
```
# Take snapshot before deleting
aws rds create-db-snapshot \
  --db-instance-identifier cropwise-dev-db \
  --db-snapshot-identifier cropwise-dev-snapshot-2025-11-16

# Delete instance (saves $15/month)
aws rds delete-db-instance \
  --db-instance-identifier cropwise-dev-db

# Restore when needed (from snapshot)
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier cropwise-dev-db \
  --db-snapshot-identifier cropwise-dev-snapshot-2025-11-16
```

---

### **Budget-Friendly Production Setup**

If you're just starting and want to minimize costs:

```
Development: db.t3.micro (FREE with Free Tier)
  - Cost: $0/month

Staging: db.t3.micro (share with dev if needed)
  - Cost: $15/month (or combine with dev)

Production: db.t3.small Single-AZ (start small)
  - Cost: ~$40/month
  - Upgrade to Multi-AZ when revenue > $1000/month

────────────────────────────────────────
Budget-Friendly Total: ~$55/month
                       ~$660/year
```

**When to Upgrade:**
- 100-1000 users → db.t3.small ($40/month)
- 1000-5000 users → db.t3.medium ($67/month Single-AZ)
- 5000-10000 users → db.t3.medium Multi-AZ ($133/month)
- 10000+ users → db.t3.large or m6g instances ($200-400/month)

---

### **Cost Monitoring**

Set up CloudWatch billing alarms:

```bash
# Create SNS topic for alerts
aws sns create-topic --name billing-alerts

# Subscribe your email
aws sns subscribe \
  --topic-arn arn:aws:sns:ap-south-1:YOUR_ACCOUNT:billing-alerts \
  --protocol email \
  --notification-endpoint your-email@example.com

# Create billing alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "RDS-Monthly-Budget-Alert" \
  --alarm-description "Alert when RDS costs exceed $200" \
  --metric-name EstimatedCharges \
  --namespace AWS/Billing \
  --statistic Maximum \
  --period 21600 \
  --evaluation-periods 1 \
  --threshold 200 \
  --comparison-operator GreaterThanThreshold
```

---

## 🎯 Database Initialization

After creating all three databases, initialize them with your schema and seed data.

### **Initialize Development Database**

```powershell
cd C:\Users\praghav\smartcrop-os\backend

# Set development DATABASE_URL
$env:DATABASE_URL="postgresql://cropwise_dev_admin:CropWiseDev2025!@YOUR_DEV_ENDPOINT:5432/cropwise_dev"

# Run migrations
npm install
npx prisma migrate dev
# OR for TypeORM:
npm run migration:run

# Seed development data
npm run seed:dev
```

**What to seed in Development:**
- Test users (admin, regular users)
- Sample farms and fields
- Mock sensor data
- Development API keys

---

### **Initialize Staging Database**

```powershell
cd C:\Users\praghav\smartcrop-os\backend

# Set staging DATABASE_URL
$env:DATABASE_URL="postgresql://cropwise_stage_admin:CropWiseStage2025!Secure@YOUR_STAGE_ENDPOINT:5432/cropwise_stage"

# Run migrations (deploy only, no dev changes)
npx prisma migrate deploy
# OR for TypeORM:
npm run migration:run

# Seed staging data (production-like but fake)
npm run seed:stage
```

**What to seed in Staging:**
- Realistic test users
- Production-like data volumes
- Test payment data (Stripe test mode)
- Staging API keys

---

### **Initialize Production Database**

⚠️ **IMPORTANT**: Run production migrations carefully!

```powershell
cd C:\Users\praghav\smartcrop-os\backend

# Set production DATABASE_URL
$env:DATABASE_URL="postgresql://cropwise_prod_admin:YOUR_PROD_PASSWORD@YOUR_PROD_ENDPOINT:5432/cropwise_prod"

# Run migrations (deploy only)
npx prisma migrate deploy
# OR for TypeORM:
npm run migration:run

# NO SEEDING in production!
# Production data comes from real users
```

**Production Best Practices:**
- ✅ Test migrations on staging first
- ✅ Take manual snapshot before migrating
- ✅ Run during low-traffic hours
- ✅ Have rollback plan ready
- ❌ Never seed fake data in production

---

### **Using SQL Scripts Directly**

If you prefer manual SQL:

```powershell
# Development
psql "postgresql://cropwise_dev_admin:CropWiseDev2025!@YOUR_DEV_ENDPOINT:5432/cropwise_dev" < schema.sql

# Staging
psql "postgresql://cropwise_stage_admin:CropWiseStage2025!Secure@YOUR_STAGE_ENDPOINT:5432/cropwise_stage" < schema.sql

# Production (be careful!)
psql "postgresql://cropwise_prod_admin:YOUR_PROD_PASSWORD@YOUR_PROD_ENDPOINT:5432/cropwise_prod" < schema.sql
```

---

### **Verify Initialization**

After running migrations, verify tables were created:

```sql
-- Connect to each database and run:
\dt  -- List all tables

-- Should see tables like:
-- users, farms, fields, sensors, readings, etc.

-- Check row counts
SELECT 
  schemaname,
  tablename,
  n_live_tup as row_count
FROM pg_stat_user_tables
ORDER BY n_live_tup DESC;
```

---

## 📊 Monitoring Your Database

### **CloudWatch Metrics (Automatic)**
Go to: https://console.aws.amazon.com/cloudwatch/

**Key Metrics to Watch:**
- **CPUUtilization** - Should stay below 80%
- **FreeableMemory** - Monitor for memory issues
- **DatabaseConnections** - Track connection usage
- **ReadLatency / WriteLatency** - Performance monitoring
- **FreeStorageSpace** - Prevent running out of space

### **Performance Insights**
1. Go to RDS Console
2. Click your database
3. Click **Performance Insights** tab
4. View real-time queries and performance

### **Set Up CloudWatch Alarms**
```
Alarm: High CPU Usage
Threshold: CPUUtilization > 80% for 5 minutes
Action: Send SNS notification to your email
```

---

## 🔄 Backup and Recovery

### **Automatic Backups**
- **Enabled by default** (7-35 days retention)
- **Point-in-time recovery** available
- **Zero downtime** backup process

### **Manual Snapshots**
1. Go to RDS Console
2. Select your database
3. **Actions** → **Take snapshot**
4. Name: `cropwise-prod-manual-backup-2025-11-14`
5. **Manual snapshots never expire** until you delete them

### **Restore from Backup**
1. Go to **Snapshots** in RDS Console
2. Select snapshot
3. **Actions** → **Restore snapshot**
4. Creates a **new database instance**
5. Update DATABASE_URL to point to restored instance

---

## 🚨 Common Issues and Solutions

### **Issue 1: Can't connect to database**

**Solution:**
1. Check security group allows your IP
2. Verify Public Access is enabled (for external connections)
3. Check VPC and subnet configuration
4. Verify username/password are correct
5. Ensure you're using the correct endpoint

### **Issue 2: "too many connections" error**

**Solution:**
```sql
-- Check current connections
SELECT count(*) FROM pg_stat_activity;

-- Max connections for db.t3.micro: ~87
-- Max connections for db.t3.small: ~158

-- Configure connection pooling in your app:
// backend/config/database.js
pool: {
  min: 2,
  max: 10  // Don't exceed instance limit
}
```

### **Issue 3: Slow queries**

**Solution:**
1. Enable Performance Insights
2. Check for missing indexes
3. Analyze slow query logs in CloudWatch
4. Consider upgrading instance size

### **Issue 4: Running out of storage**

**Solution:**
- Storage autoscaling should handle this
- If disabled, manually increase storage:
  1. Select database → **Modify**
  2. Increase **Allocated storage**
  3. Apply immediately or during maintenance window

---

## ✅ Post-Setup Checklist

After creating all three databases:

### **Development Database Checklist**
- [ ] `cropwise-dev-db` status is "Available"
- [ ] Successfully connected using psql or pgAdmin
- [ ] Added `DATABASE_URL_DEV` to GitHub Secrets
- [ ] Security group allows your IP address
- [ ] Backups enabled (7 days retention)
- [ ] Tested connection from local machine
- [ ] Ran migrations and seed data
- [ ] Password saved in password manager

### **Staging Database Checklist**
- [ ] `cropwise-stage-db` status is "Available"
- [ ] Public Access set to "No"
- [ ] Added `DATABASE_URL_STAGE` to GitHub Secrets
- [ ] Security group allows ECS task security group
- [ ] Backups enabled (14 days retention)
- [ ] Enhanced monitoring enabled
- [ ] Performance Insights enabled
- [ ] Tested connection from VPC or bastion
- [ ] Ran migrations
- [ ] Password saved in password manager

### **Production Database Checklist**
- [ ] `cropwise-prod-db` status is "Available"
- [ ] Multi-AZ deployment enabled
- [ ] Public Access set to "No"
- [ ] Added `DATABASE_URL_PROD` to GitHub Secrets
- [ ] Security group allows ONLY ECS tasks (no direct IPs)
- [ ] Backups enabled (30 days retention)
- [ ] Enhanced monitoring enabled (60 seconds)
- [ ] Performance Insights enabled
- [ ] Deletion protection enabled
- [ ] Encryption enabled
- [ ] CloudWatch alarms configured
- [ ] Tested connection from ECS task
- [ ] Ran migrations (tested on staging first)
- [ ] Password saved in password manager
- [ ] Manual snapshot taken

### **General Checklist**
- [ ] All three databases show "Available" status
- [ ] All three `DATABASE_URL_*` secrets in GitHub
- [ ] Billing alerts configured ($200 threshold)
- [ ] Cost monitoring dashboard set up
- [ ] Database connection pooling configured
- [ ] Backup and restore process documented
- [ ] Emergency contact list for DB issues
- [ ] Database maintenance window scheduled

---

## 📚 Additional Resources

- **AWS RDS Documentation**: https://docs.aws.amazon.com/rds/
- **PostgreSQL Documentation**: https://www.postgresql.org/docs/
- **RDS PostgreSQL Best Practices**: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_BestPractices.html
- **Cost Calculator**: https://calculator.aws/#/

---

## 🎯 Quick Reference - DATABASE_URL Format

### **Format**
```bash
postgresql://[username]:[password]@[endpoint]:[port]/[database_name]
```

### **Development**
```bash
# Database Identifier
cropwise-dev-db

# Username
cropwise_dev_admin

# Connection String
postgresql://cropwise_dev_admin:CropWiseDev2025!@cropwise-dev-db.c9abc123xyz.ap-south-1.rds.amazonaws.com:5432/cropwise_dev

# GitHub Secret Name
DATABASE_URL_DEV
```

### **Staging**
```bash
# Database Identifier
cropwise-stage-db

# Username
cropwise_stage_admin

# Connection String
postgresql://cropwise_stage_admin:CropWiseStage2025!Secure@cropwise-stage-db.d7def456abc.ap-south-1.rds.amazonaws.com:5432/cropwise_stage

# GitHub Secret Name
DATABASE_URL_STAGE
```

### **Production**
```bash
# Database Identifier
cropwise-prod-db

# Username
cropwise_prod_admin

# Connection String
postgresql://cropwise_prod_admin:YOUR_STRONG_PASSWORD@cropwise-prod-db.e8ghi789jkl.ap-south-1.rds.amazonaws.com:5432/cropwise_prod

# GitHub Secret Name
DATABASE_URL_PROD
```

### **URL Encoding Special Characters**

If your password contains special characters, encode them:

| Character | Encoded | Example |
|-----------|---------|---------|
| `@` | `%40` | `Pass@123` → `Pass%40123` |
| `!` | `%21` | `Pass!123` → `Pass%21123` |
| `#` | `%23` | `Pass#123` → `Pass%23123` |
| `$` | `%24` | `Pass$123` → `Pass%24123` |
| `%` | `%25` | `Pass%123` → `Pass%25123` |
| `&` | `%26` | `Pass&123` → `Pass%26123` |
| `=` | `%3D` | `Pass=123` → `Pass%3D123` |

**Example:**
```bash
# Original password: CropWise@2025!
# Encoded password: CropWise%402025%21

postgresql://cropwise_prod_admin:CropWise%402025%21@endpoint:5432/cropwise_prod
```

---

### **Summary Table**

| Environment | Instance ID | Database Name | Username | Public Access | Multi-AZ | Monthly Cost |
|-------------|-------------|---------------|----------|---------------|----------|--------------|
| **Dev** | `cropwise-dev-db` | `cropwise_dev` | `cropwise_dev_admin` | Yes | No | ~$15 (or FREE) |
| **Stage** | `cropwise-stage-db` | `cropwise_stage` | `cropwise_stage_admin` | No | No | ~$35 |
| **Prod** | `cropwise-prod-db` | `cropwise_prod` | `cropwise_prod_admin` | No | Yes | ~$133 |

---

**Created**: November 14, 2025  
**Last Updated**: November 16, 2025  
**For**: CropWise - All Environments (Dev, Stage, Production)


