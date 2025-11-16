# 🔐 GitHub Secrets Setup Guide

Complete guide to gather and configure all secrets needed for CropWise GitHub Actions CI/CD pipeline.

---

## 📋 Quick Navigation

- [Priority 1: Essential Secrets](#priority-1-essential-secrets-add-now)
- [Priority 2: AWS Deployment](#priority-2-aws-deployment-secrets)
- [Priority 3: Optional Features](#priority-3-optional-feature-secrets)
- [How to Add Secrets to GitHub](#how-to-add-secrets-to-github)
- [Secrets Template](#secrets-template)
- [Security Best Practices](#security-best-practices)

---

## 🎯 Priority 1: Essential Secrets (Add Now)

These secrets are **required** for the CI/CD pipeline to work.

### 1. `VITE_API_URL`

**What it is**: Frontend API endpoint URL

**Where to add it**:
```
Name: VITE_API_URL
Value: http://localhost:3000
```

**Notes**:
- For local development: `http://localhost:3000`
- For production: `https://api.yourdomain.com`
- This tells the frontend where to find the backend API

**Status**: ✅ You should have already added this!

---

## 🚀 Priority 2: AWS Deployment Secrets

These secrets are needed **only when you're ready to deploy to AWS**.

### 2. `AWS_ACCESS_KEY_ID` (Development)

**What it is**: AWS credentials for DEV environment deployments

**How to get it**:
1. Log in to AWS Console: https://console.aws.amazon.com
2. Go to **IAM** (Identity and Access Management)
3. Click **Users** → **Add users**
4. Username: `cropwise-dev-deployer`
5. Access type: ✅ **Programmatic access**
6. Permissions: Attach policies:
   - `AmazonEC2ContainerRegistryFullAccess`
   - `AmazonECS_FullAccess`
   - `AmazonS3FullAccess`
   - `CloudFrontFullAccess`
   - `AWSElasticBeanstalkFullAccess`
7. Click through to **Create user**
8. **Copy the Access Key ID** (starts with `AKIA...`)

**Where to add it**:
```
Name: AWS_ACCESS_KEY_ID
Value: AKIA... (your key)
```

---

### 3. `AWS_SECRET_ACCESS_KEY` (Development)

**What it is**: Secret key paired with the Access Key ID above

**How to get it**:
- Same IAM user creation process as above
- **Copy the Secret Access Key** (shown only once!)
- ⚠️ **CRITICAL**: Save this immediately, you can't see it again!

**Where to add it**:
```
Name: AWS_SECRET_ACCESS_KEY
Value: (your secret key - long string)
```

---

### 4. `AWS_REGION`

**What it is**: AWS region where you want to deploy

**Common regions**:
- `us-east-1` - US East (N. Virginia) - Most services, cheapest
- `us-west-2` - US West (Oregon)
- `eu-west-1` - Europe (Ireland)
- `ap-south-1` - Asia Pacific (Mumbai)
- `ap-southeast-1` - Asia Pacific (Singapore)

**Where to add it**:
```
Name: AWS_REGION
Value: us-east-1
```

**Recommendation**: Use `us-east-1` unless you have a specific reason to use another region.

---

### 5. `AWS_ACCESS_KEY_ID_PROD` (Production)

**What it is**: Separate AWS credentials for PRODUCTION deployments

**How to get it**:
- Same process as #2, but create a different user: `cropwise-prod-deployer`
- **Best Practice**: Use separate credentials for prod to limit blast radius

**Where to add it**:
```
Name: AWS_ACCESS_KEY_ID_PROD
Value: AKIA... (production key)
```

---

### 6. `AWS_SECRET_ACCESS_KEY_PROD` (Production)

**What it is**: Secret key for production AWS account

**How to get it**:
- Same as #3, but for the production IAM user

**Where to add it**:
```
Name: AWS_SECRET_ACCESS_KEY_PROD
Value: (production secret key)
```

---

### 7. `CLOUDFRONT_DEV_DISTRIBUTION_ID` (Optional)

**What it is**: CloudFront CDN distribution ID for dev environment

**How to get it**:
1. Log in to AWS Console
2. Go to **CloudFront**
3. Create a distribution (if you haven't already)
4. Copy the **Distribution ID** (e.g., `E1234ABCD5678`)

**Where to add it**:
```
Name: CLOUDFRONT_DEV_DISTRIBUTION_ID
Value: E1234ABCD5678
```

**Notes**:
- Only needed if you want to use CloudFront CDN
- Skip this if you're using Elastic Beanstalk only

---

### 8. `CLOUDFRONT_PROD_DISTRIBUTION_ID` (Optional)

**What it is**: CloudFront CDN distribution ID for production

**How to get it**:
- Same as #7, but for production distribution

**Where to add it**:
```
Name: CLOUDFRONT_PROD_DISTRIBUTION_ID
Value: E5678EFGH9012
```

---

## 🔧 Priority 3: Optional Feature Secrets

These secrets enable **optional features**. Add them when you're ready to use these features.

### 9. `GOOGLE_CLIENT_ID`

**What it is**: OAuth 2.0 client ID for Google Sign-In

**How to get it**:
1. Go to: https://console.cloud.google.com
2. Create a new project (or select existing)
3. Enable **Google+ API**
4. Go to **Credentials**
5. Click **Create Credentials** → **OAuth 2.0 Client ID**
6. Application type: **Web application**
7. Authorized JavaScript origins:
   ```
   http://localhost:8080
   https://yourdomain.com
   ```
8. Authorized redirect URIs:
   ```
   http://localhost:8080/auth/google/callback
   https://yourdomain.com/auth/google/callback
   ```
9. Copy the **Client ID** (ends with `.apps.googleusercontent.com`)

**Where to add it**:
```
Name: GOOGLE_CLIENT_ID
Value: 1234567890-abc123def456.apps.googleusercontent.com
```

**Documentation**: See `docs/GOOGLE_OAUTH_CHECKLIST.md` for detailed setup

---

### 10. `GOOGLE_CLIENT_SECRET`

**What it is**: Secret paired with Google Client ID

**How to get it**:
- Same OAuth setup as #9
- Copy the **Client Secret**

**Where to add it**:
```
Name: GOOGLE_CLIENT_SECRET
Value: GOCSPX-abc123def456...
```

---

### 11. `TWILIO_ACCOUNT_SID`

**What it is**: Twilio credentials for SMS and WhatsApp notifications

**How to get it**:
1. Sign up for Twilio: https://www.twilio.com/try-twilio
2. Go to **Console**: https://console.twilio.com
3. Copy your **Account SID** (starts with `AC...`)

**Where to add it**:
```
Name: TWILIO_ACCOUNT_SID
Value: ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**Cost**:
- Free trial: $15 credit
- SMS: ~$0.0075 per message
- WhatsApp: ~$0.005 per message

---

### 12. `TWILIO_AUTH_TOKEN`

**What it is**: Authentication token for Twilio API

**How to get it**:
- Same Twilio Console as #11
- Copy your **Auth Token** (click to reveal)

**Where to add it**:
```
Name: TWILIO_AUTH_TOKEN
Value: (your auth token)
```

---

### 13. `TWILIO_PHONE_NUMBER`

**What it is**: Your Twilio phone number for sending SMS

**How to get it**:
1. In Twilio Console
2. Go to **Phone Numbers** → **Buy a Number**
3. Select a number (costs ~$1/month)
4. Copy the number in E.164 format: `+15551234567`

**Where to add it**:
```
Name: TWILIO_PHONE_NUMBER
Value: +1555XXXXXXX
```

---

### 14. `TWILIO_WHATSAPP_NUMBER`

**What it is**: Twilio WhatsApp sandbox number

**How to get it**:
1. In Twilio Console
2. Go to **Messaging** → **Try it out** → **Send a WhatsApp message**
3. Follow instructions to join sandbox
4. Copy the WhatsApp number: `whatsapp:+14155238886`

**Where to add it**:
```
Name: TWILIO_WHATSAPP_NUMBER
Value: whatsapp:+1415XXXXXXX
```

**Notes**:
- Sandbox is free for testing
- For production, you need to apply for a WhatsApp Business Account

---

### 15. `SLACK_WEBHOOK_URL` (CI/CD Notifications)

**What it is**: Slack webhook for deployment notifications

**How to get it**:
1. Go to your Slack workspace
2. Click **Apps** → **Browse App Directory**
3. Search for **Incoming Webhooks**
4. Click **Add to Slack**
5. Choose a channel (e.g., `#deployments`)
6. Copy the **Webhook URL**

**Where to add it**:
```
Name: SLACK_WEBHOOK_URL
Value: https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXX
```

**What you'll get**:
- Deployment start/success/failure notifications
- Build status updates
- Security scan results

---

### 16. `DOCKERHUB_USERNAME` (Optional)

**What it is**: Docker Hub username for publishing images

**How to get it**:
1. Create account: https://hub.docker.com/signup
2. Your username is your Docker Hub account name

**Where to add it**:
```
Name: DOCKERHUB_USERNAME
Value: yourcompanyname
```

**Notes**:
- Only needed if you want to publish images to Docker Hub
- Free tier: 1 private repository, unlimited public

---

### 17. `DOCKERHUB_TOKEN`

**What it is**: Access token for Docker Hub authentication

**How to get it**:
1. Log in to Docker Hub
2. Go to **Account Settings** → **Security**
3. Click **New Access Token**
4. Description: `GitHub Actions`
5. Access permissions: **Read, Write, Delete**
6. Copy the token (shown only once!)

**Where to add it**:
```
Name: DOCKERHUB_TOKEN
Value: dckr_pat_abc123...
```

---

### 18. `JWT_SECRET`

**What it is**: Secret key for signing JWT tokens

**How to generate it**:

**Option A: Node.js**
```bash
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
```

**Option B: PowerShell**
```powershell
[System.Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes((New-Guid).ToString() + (New-Guid).ToString()))
```

**Option C: Online**
- Visit: https://randomkeygen.com
- Use the "504-bit WPA Key" or "CodeIgniter Encryption Keys"

**Where to add it**:
```
Name: JWT_SECRET
Value: (generated random string - at least 32 characters)
```

**Example**:
```
JWT_SECRET: a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6
```

---

### 19. `SESSION_SECRET`

**What it is**: Secret key for session encryption

**How to generate it**:
- Same methods as #18 (JWT_SECRET)
- Generate a **different** random string

**Where to add it**:
```
Name: SESSION_SECRET
Value: (different random string)
```

---

### 20. `DATABASE_URL` (Production)

**What it is**: PostgreSQL database connection string for production

**Format**:
```
postgresql://username:password@hostname:5432/database_name
```

**Example** (AWS RDS):
```
postgresql://cropwise:SecurePass123!@cropwise-db.abc123.us-east-1.rds.amazonaws.com:5432/cropwise_prod
```

**How to get it**:
1. After creating RDS instance in AWS
2. Go to RDS Console
3. Click your database instance
4. Copy the **Endpoint** (hostname)
5. Combine with username/password you set during creation

**Where to add it**:
```
Name: DATABASE_URL
Value: postgresql://user:pass@host:5432/dbname
```

---

### 21. `REDIS_URL` (Production)

**What it is**: Redis connection string for caching/sessions

**Format**:
```
redis://hostname:6379
```

**Example** (AWS ElastiCache):
```
redis://cropwise-redis.abc123.0001.use1.cache.amazonaws.com:6379
```

**How to get it**:
1. After creating ElastiCache cluster
2. Go to ElastiCache Console
3. Click your cluster
4. Copy the **Primary Endpoint**

**Where to add it**:
```
Name: REDIS_URL
Value: redis://your-redis-endpoint:6379
```

---

## 📝 How to Add Secrets to GitHub

### Step-by-Step Guide

1. **Go to your repository**:
   ```
   https://github.com/yellowflowersorganics-star/cropwise
   ```

2. **Navigate to Settings**:
   - Click **Settings** tab (top right)

3. **Open Secrets Section**:
   - Left sidebar: **Secrets and variables** → **Actions**

4. **Add a new secret**:
   - Click **New repository secret**
   - **Name**: Enter the secret name (e.g., `VITE_API_URL`)
   - **Secret**: Paste the secret value
   - Click **Add secret**

5. **Verify it's added**:
   - You should see it in the list
   - You can't view the value (security feature)
   - You can update or delete it

### Direct Link
👉 https://github.com/yellowflowersorganics-star/cropwise/settings/secrets/actions

---

## 📋 Secrets Template

Copy this template and fill in your values:

```bash
# ============================================
# CropWise - GitHub Secrets
# ============================================

# Priority 1: Essential (Add Now)
# --------------------------------
VITE_API_URL=http://localhost:3000


# Priority 2: AWS Deployment (Add when ready)
# --------------------------------------------
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=...
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID_PROD=AKIA...
AWS_SECRET_ACCESS_KEY_PROD=...
CLOUDFRONT_DEV_DISTRIBUTION_ID=E...
CLOUDFRONT_PROD_DISTRIBUTION_ID=E...


# Priority 3: Optional Features (Add as needed)
# ----------------------------------------------

# Google OAuth
GOOGLE_CLIENT_ID=...apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=GOCSPX-...

# Twilio (SMS/WhatsApp)
TWILIO_ACCOUNT_SID=ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_AUTH_TOKEN=your_auth_token_here
TWILIO_PHONE_NUMBER=+1555XXXXXXX
TWILIO_WHATSAPP_NUMBER=whatsapp:+1415XXXXXXX

# CI/CD Notifications
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/...

# Docker Hub (Optional)
DOCKERHUB_USERNAME=yourcompany
DOCKERHUB_TOKEN=dckr_pat_...

# Security Keys
JWT_SECRET=...
SESSION_SECRET=...

# Production Database
DATABASE_URL=postgresql://user:pass@host:5432/db
REDIS_URL=redis://host:6379
```

---

## 🔒 Security Best Practices

### DO ✅

1. **Use different credentials for dev and prod**
   - Separate AWS accounts if possible
   - Different database instances
   - Separate API keys

2. **Rotate secrets regularly**
   - AWS keys: Every 90 days
   - JWT secrets: Every 6 months
   - Database passwords: Every 90 days

3. **Use least privilege**
   - Give AWS IAM users only necessary permissions
   - Use separate read-only credentials for monitoring

4. **Store secrets securely locally**
   - Use password manager (1Password, LastPass, Bitwarden)
   - Never commit secrets to git
   - Use `.env` files (already in `.gitignore`)

5. **Enable GitHub secret scanning**
   - Automatically detects leaked secrets
   - Get alerts if secrets are committed

### DON'T ❌

1. **Never commit secrets to git**
   ```bash
   # Bad - hardcoded secret
   const API_KEY = "abc123"
   
   # Good - use environment variables
   const API_KEY = process.env.API_KEY
   ```

2. **Never share secrets in plain text**
   - Don't send via email
   - Don't post in Slack/Discord
   - Use secure sharing tools (1Password, AWS Secrets Manager)

3. **Never use the same secret everywhere**
   - Different secret for dev/staging/prod
   - Different JWT secret per environment

4. **Never use weak secrets**
   ```bash
   # Bad
   JWT_SECRET=secret123
   
   # Good
   JWT_SECRET=a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6
   ```

---

## 📊 Secrets Checklist

Track your progress:

### Essential (Required for CI/CD)
- [ ] `VITE_API_URL`

### AWS Deployment (Required for AWS deploy)
- [ ] `AWS_ACCESS_KEY_ID`
- [ ] `AWS_SECRET_ACCESS_KEY`
- [ ] `AWS_REGION`
- [ ] `AWS_ACCESS_KEY_ID_PROD`
- [ ] `AWS_SECRET_ACCESS_KEY_PROD`
- [ ] `CLOUDFRONT_DEV_DISTRIBUTION_ID` (optional)
- [ ] `CLOUDFRONT_PROD_DISTRIBUTION_ID` (optional)

### Google OAuth (Required for Google Sign-In)
- [ ] `GOOGLE_CLIENT_ID`
- [ ] `GOOGLE_CLIENT_SECRET`

### Twilio (Required for SMS/WhatsApp)
- [ ] `TWILIO_ACCOUNT_SID`
- [ ] `TWILIO_AUTH_TOKEN`
- [ ] `TWILIO_PHONE_NUMBER`
- [ ] `TWILIO_WHATSAPP_NUMBER`

### CI/CD (Optional but recommended)
- [ ] `SLACK_WEBHOOK_URL`
- [ ] `DOCKERHUB_USERNAME`
- [ ] `DOCKERHUB_TOKEN`

### Security (Required for production)
- [ ] `JWT_SECRET`
- [ ] `SESSION_SECRET`
- [ ] `DATABASE_URL`
- [ ] `REDIS_URL`

---

## 🎯 Recommended Setup Order

### Phase 1: Basic CI/CD (Now)
1. ✅ `VITE_API_URL` - Already added!

### Phase 2: AWS Deployment (When ready to deploy)
2. `AWS_ACCESS_KEY_ID`
3. `AWS_SECRET_ACCESS_KEY`
4. `AWS_REGION`
5. `JWT_SECRET`
6. `SESSION_SECRET`

### Phase 3: Authentication (Before users sign up)
7. `GOOGLE_CLIENT_ID`
8. `GOOGLE_CLIENT_SECRET`

### Phase 4: Database (Before production)
9. `DATABASE_URL`
10. `REDIS_URL`

### Phase 5: Notifications (When needed)
11. `TWILIO_ACCOUNT_SID`
12. `TWILIO_AUTH_TOKEN`
13. `TWILIO_PHONE_NUMBER`
14. `SLACK_WEBHOOK_URL`

### Phase 6: Production Deployment
15. `AWS_ACCESS_KEY_ID_PROD`
16. `AWS_SECRET_ACCESS_KEY_PROD`
17. `CLOUDFRONT_PROD_DISTRIBUTION_ID`

---

## 📞 Need Help?

**Common Issues**:

1. **Secret not working in Actions**
   - Check the name matches exactly (case-sensitive)
   - Re-run the workflow after adding secret
   - Secrets are only available after next workflow run

2. **AWS credentials not working**
   - Verify IAM permissions
   - Check region matches
   - Ensure Access Key is Active (not deleted/rotated)

3. **Google OAuth errors**
   - Verify redirect URIs match exactly
   - Check authorized origins include your domain
   - See `docs/GOOGLE_OAUTH_CHECKLIST.md`

4. **Twilio errors**
   - Verify phone numbers are in E.164 format
   - Check account balance (trial/paid)
   - WhatsApp requires sandbox setup for testing

---

## 📚 Related Documentation

- **AWS Setup**: `DEPLOY_TO_AWS_NOW.md`
- **Google OAuth**: `docs/GOOGLE_OAUTH_CHECKLIST.md`
- **CI/CD Guide**: `docs/CICD_SETUP_GUIDE.md`
- **Security Guide**: `docs/SECURITY_GUIDE.md`

---

**Last Updated**: November 2025

**Version**: 2.0.0

**Maintained by**: CropWise Team

