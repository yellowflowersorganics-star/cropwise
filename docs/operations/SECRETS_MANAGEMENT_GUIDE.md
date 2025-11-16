# 🔐 Secrets Management Guide

**Complete guide to managing secrets, credentials, and sensitive data in CropWise**

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Types of Secrets](#-types-of-secrets)
- [Security Best Practices](#-security-best-practices)
- [Local Development](#-local-development)
- [CI/CD Secrets](#-cicd-secrets)
- [Production Secrets](#-production-secrets)
- [Secret Rotation](#-secret-rotation)
- [Emergency Procedures](#-emergency-procedures)
- [Audit & Compliance](#-audit--compliance)

---

## 🎯 Overview

**Never commit secrets to git!** This guide covers all aspects of secrets management for CropWise.

### What are Secrets?

Secrets include:
- API keys and tokens
- Database passwords
- JWT secrets
- OAuth client secrets
- AWS access keys
- Third-party service credentials
- Encryption keys
- SSL/TLS certificates

---

## 🔑 Types of Secrets

### 1. Database Credentials

```bash
DB_HOST=localhost
DB_PORT=5432
DB_NAME=cropwise
DB_USER=cropwise
DB_PASSWORD=SecureRandomPassword123!
```

**Requirements:**
- Minimum 16 characters
- Mix of uppercase, lowercase, numbers, symbols
- Different for each environment
- Rotated every 90 days

### 2. Authentication Secrets

```bash
# JWT Secret (used to sign tokens)
JWT_SECRET=base64EncodedRandomString...
JWT_EXPIRES_IN=7d

# Google OAuth
GOOGLE_CLIENT_ID=123456789012-abc...
GOOGLE_CLIENT_SECRET=GOCSPX-abc...

# Session Secret
SESSION_SECRET=anotherRandomString...
```

### 3. AWS Credentials

```bash
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=wJalr...
AWS_REGION=ap-south-1
```

### 4. Third-Party Services

```bash
# Twilio (SMS/WhatsApp)
TWILIO_ACCOUNT_SID=AC...
TWILIO_AUTH_TOKEN=...
TWILIO_PHONE_NUMBER=+1234567890

# SendGrid (Email)
SENDGRID_API_KEY=SG....

# Sentry (Error tracking)
SENTRY_DSN=https://...@sentry.io/...
```

### 5. Encryption Keys

```bash
# Data encryption
ENCRYPTION_KEY=32ByteRandomKey...
ENCRYPTION_ALGORITHM=aes-256-gcm

# API encryption
API_ENCRYPTION_KEY=...
```

---

## 🛡️ Security Best Practices

### Rule #1: Never Commit Secrets

❌ **NEVER DO THIS:**

```javascript
// Bad - hardcoded secret
const JWT_SECRET = 'mysecretkey123';

// Bad - committed .env file
// .env file in git history
```

✅ **ALWAYS DO THIS:**

```javascript
// Good - read from environment
const JWT_SECRET = process.env.JWT_SECRET;

// Good - validate secret exists
if (!JWT_SECRET) {
  throw new Error('JWT_SECRET environment variable is required');
}
```

### Rule #2: Use Strong Secrets

```bash
# ❌ Bad - weak password
DB_PASSWORD=password123

# ✅ Good - strong random password
DB_PASSWORD=k8$mP9#nQ2@wX5!zR7^vL4&hJ6*fD3

# Generate strong secrets
openssl rand -base64 32
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

### Rule #3: Different Secrets per Environment

```bash
# Development
JWT_SECRET=dev_secret_for_local_only

# Staging
JWT_SECRET=staging_secret_different_from_dev

# Production
JWT_SECRET=prod_super_secret_extra_secure
```

### Rule #4: Principle of Least Privilege

- Only grant minimum required permissions
- Use separate AWS IAM users for different services
- Restrict database user permissions

```sql
-- ❌ Bad - too many permissions
GRANT ALL PRIVILEGES ON DATABASE cropwise TO app_user;

-- ✅ Good - only what's needed
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;
REVOKE CREATE, DROP ON DATABASE cropwise FROM app_user;
```

### Rule #5: Regular Rotation

**Rotation Schedule:**
- Development: Every 6 months
- Staging: Every 3 months
- Production: Every 90 days
- After security incident: Immediately

---

## 💻 Local Development

### Setup Environment Variables

**1. Create .env file**

```bash
cd backend
cp .env.example .env
```

**2. Edit .env with your secrets**

```bash
# backend/.env
NODE_ENV=development
PORT=8080

# Database
DB_DIALECT=sqlite
DB_STORAGE=./database.sqlite

# JWT (generate new one for your machine)
JWT_SECRET=your_local_jwt_secret_here

# Google OAuth (get from Google Console)
GOOGLE_CLIENT_ID=your_client_id
GOOGLE_CLIENT_SECRET=your_client_secret

# Optional services (can leave empty for local dev)
TWILIO_ENABLED=false
EMAIL_ENABLED=false
```

**3. Never commit .env**

Verify `.gitignore` includes:

```
# .gitignore
.env
.env.*
!.env.example
*.pem
*.key
secrets/
```

### Generate Strong Secrets

**Create a script: `scripts/generate-secrets.sh`**

```bash
#!/bin/bash

echo "🔐 Generating Secrets for CropWise"
echo ""

# JWT Secret
echo "JWT_SECRET=$(openssl rand -base64 64 | tr -d '\n')"

# Session Secret
echo "SESSION_SECRET=$(openssl rand -base64 32 | tr -d '\n')"

# Database Password
echo "DB_PASSWORD=$(openssl rand -base64 24 | tr -d '\n' | tr '+/' '-_')"

# Encryption Key (32 bytes for AES-256)
echo "ENCRYPTION_KEY=$(openssl rand -hex 32)"

# API Key
echo "API_KEY=$(uuidgen | tr -d '\n')"

echo ""
echo "✅ Secrets generated! Copy these to your .env file"
echo "⚠️  NEVER commit these secrets to git!"
```

**Usage:**

```bash
chmod +x scripts/generate-secrets.sh
./scripts/generate-secrets.sh > secrets.txt
# Copy values to .env
# Delete secrets.txt after copying
rm secrets.txt
```

### Environment-Specific Files

```bash
# Development
.env.development

# Testing
.env.test

# Production (never in repo!)
.env.production
```

**In package.json:**

```json
{
  "scripts": {
    "dev": "NODE_ENV=development node -r dotenv/config src/index.js dotenv_config_path=.env.development",
    "test": "NODE_ENV=test jest",
    "start": "NODE_ENV=production node src/index.js"
  }
}
```

---

## 🔧 CI/CD Secrets

### GitHub Actions Secrets

**1. Navigate to Repository Settings**

```
Repository → Settings → Secrets and variables → Actions
```

**2. Add Required Secrets**

Click **"New repository secret"** for each:

#### AWS Deployment Secrets

```
Name: AWS_ACCESS_KEY_ID
Value: AKIA...

Name: AWS_SECRET_ACCESS_KEY
Value: wJalr...

Name: AWS_ACCESS_KEY_ID_PROD
Value: AKIA... (different from dev)

Name: AWS_SECRET_ACCESS_KEY_PROD
Value: wJalr... (different from dev)
```

#### Docker Registry

```
Name: DOCKERHUB_USERNAME
Value: yourcompany

Name: DOCKERHUB_TOKEN
Value: dckr_pat_...
```

#### Application Secrets

```
Name: JWT_SECRET
Value: (generated secret)

Name: DB_PASSWORD
Value: (generated password)

Name: GOOGLE_CLIENT_ID
Value: 123456789012-abc...

Name: GOOGLE_CLIENT_SECRET
Value: GOCSPX-...
```

#### Notification Secrets

```
Name: SLACK_WEBHOOK_URL
Value: https://hooks.slack.com/services/...

Name: SENTRY_DSN
Value: https://...@sentry.io/...
```

#### Optional Secrets

```
Name: CLOUDFRONT_PROD_DISTRIBUTION_ID
Value: E1234567890ABC

Name: CODECOV_TOKEN
Value: (from codecov.io)
```

### Using Secrets in Workflows

```yaml
# .github/workflows/deploy.yml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-south-1

      - name: Deploy with secrets
        env:
          JWT_SECRET: ${{ secrets.JWT_SECRET }}
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
        run: |
          # Deploy commands here
```

### GitHub Environments

**Create environments for better security:**

```
Settings → Environments → New environment
```

**Development Environment:**
```
Name: development
Secrets:
  - AWS_ACCESS_KEY_ID (dev)
  - AWS_SECRET_ACCESS_KEY (dev)
Protection rules: None
```

**Production Environment:**
```
Name: production
Secrets:
  - AWS_ACCESS_KEY_ID_PROD
  - AWS_SECRET_ACCESS_KEY_PROD
Protection rules:
  - Required reviewers: 1
  - Deployment branches: main only
```

**Use in workflow:**

```yaml
jobs:
  deploy-prod:
    environment:
      name: production
      url: https://cropwise.io
    steps:
      # Secrets from 'production' environment
```

---

## ☁️ Production Secrets

### AWS Secrets Manager

**Best practice for production secrets**

#### 1. Create Secrets in AWS

```bash
# Create database password secret
aws secretsmanager create-secret \
  --name cropwise/prod/db-password \
  --description "Production database password" \
  --secret-string "SuperSecurePassword123!"

# Create JWT secret
aws secretsmanager create-secret \
  --name cropwise/prod/jwt-secret \
  --secret-string "$(openssl rand -base64 64)"

# Create object with multiple secrets
aws secretsmanager create-secret \
  --name cropwise/prod/app-secrets \
  --secret-string '{
    "JWT_SECRET": "...",
    "DB_PASSWORD": "...",
    "API_KEY": "..."
  }'
```

#### 2. IAM Policy for ECS Task

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue"
      ],
      "Resource": [
        "arn:aws:secretsmanager:ap-south-1:123456789012:secret:cropwise/prod/*"
      ]
    }
  ]
}
```

#### 3. ECS Task Definition with Secrets

```json
{
  "family": "cropwise-backend",
  "containerDefinitions": [
    {
      "name": "cropwise-backend",
      "image": "cropwise/backend:latest",
      "secrets": [
        {
          "name": "DB_PASSWORD",
          "valueFrom": "arn:aws:secretsmanager:ap-south-1:123456789012:secret:cropwise/prod/db-password"
        },
        {
          "name": "JWT_SECRET",
          "valueFrom": "arn:aws:secretsmanager:ap-south-1:123456789012:secret:cropwise/prod/jwt-secret"
        }
      ],
      "environment": [
        {
          "name": "NODE_ENV",
          "value": "production"
        }
      ]
    }
  ]
}
```

#### 4. Access Secrets in Application

```javascript
// backend/src/config/secrets.js
const AWS = require('aws-sdk');

const secretsManager = new AWS.SecretsManager({
  region: process.env.AWS_REGION || 'ap-south-1'
});

async function getSecret(secretName) {
  try {
    const data = await secretsManager.getSecretValue({
      SecretId: secretName
    }).promise();

    if ('SecretString' in data) {
      return JSON.parse(data.SecretString);
    } else {
      // Binary secret
      const buff = Buffer.from(data.SecretBinary, 'base64');
      return buff.toString('ascii');
    }
  } catch (error) {
    console.error(`Error retrieving secret ${secretName}:`, error);
    throw error;
  }
}

// Load secrets on startup
async function loadSecrets() {
  if (process.env.NODE_ENV === 'production') {
    const secrets = await getSecret('cropwise/prod/app-secrets');
    process.env.JWT_SECRET = secrets.JWT_SECRET;
    process.env.DB_PASSWORD = secrets.DB_PASSWORD;
    process.env.API_KEY = secrets.API_KEY;
  }
}

module.exports = { loadSecrets, getSecret };
```

**In index.js:**

```javascript
const { loadSecrets } = require('./config/secrets');

async function startServer() {
  // Load secrets from AWS Secrets Manager
  await loadSecrets();
  
  // Now start the server with secrets available
  const app = require('./app');
  const port = process.env.PORT || 8080;
  
  app.listen(port, () => {
    console.log(`Server running on port ${port}`);
  });
}

startServer().catch(error => {
  console.error('Failed to start server:', error);
  process.exit(1);
});
```

### AWS Systems Manager Parameter Store

**Alternative to Secrets Manager (free tier)**

```bash
# Create parameters
aws ssm put-parameter \
  --name /cropwise/prod/db-password \
  --value "SuperSecurePassword123!" \
  --type SecureString \
  --key-id alias/aws/ssm

# Get parameter
aws ssm get-parameter \
  --name /cropwise/prod/db-password \
  --with-decryption \
  --query Parameter.Value \
  --output text
```

### Environment Variables on EC2/ECS

**For ECS:**
Use secrets in task definition (shown above)

**For EC2:**
Use user data or Systems Manager

```bash
#!/bin/bash
# EC2 User Data

# Export secrets from Parameter Store
export DB_PASSWORD=$(aws ssm get-parameter \
  --name /cropwise/prod/db-password \
  --with-decryption \
  --query Parameter.Value \
  --output text)

export JWT_SECRET=$(aws ssm get-parameter \
  --name /cropwise/prod/jwt-secret \
  --with-decryption \
  --query Parameter.Value \
  --output text)

# Start application
cd /app
npm start
```

---

## 🔄 Secret Rotation

### Manual Rotation Process

**1. Generate New Secret**

```bash
NEW_PASSWORD=$(openssl rand -base64 24)
echo $NEW_PASSWORD
```

**2. Update Database Password**

```sql
-- Connect to database
ALTER USER cropwise WITH PASSWORD 'new_password_here';
```

**3. Update Application Configuration**

```bash
# Update in AWS Secrets Manager
aws secretsmanager update-secret \
  --secret-id cropwise/prod/db-password \
  --secret-string "new_password_here"

# Or update GitHub secret
# Go to Settings → Secrets → Edit secret
```

**4. Restart Application**

```bash
# ECS
aws ecs update-service \
  --cluster cropwise-prod-cluster \
  --service cropwise-backend \
  --force-new-deployment

# Wait for new tasks to start with new secret
aws ecs wait services-stable \
  --cluster cropwise-prod-cluster \
  --services cropwise-backend
```

**5. Verify**

```bash
# Check application logs
aws logs tail /ecs/cropwise-backend --follow

# Test API
curl https://api.cropwise.io/health
```

### Automated Rotation (AWS Secrets Manager)

```bash
# Enable automatic rotation
aws secretsmanager rotate-secret \
  --secret-id cropwise/prod/db-password \
  --rotation-lambda-arn arn:aws:lambda:ap-south-1:123456789012:function:cropwise-rotate-secret \
  --rotation-rules AutomaticallyAfterDays=90
```

**Lambda function for rotation:**

```javascript
// lambda/rotate-secret.js
const AWS = require('aws-sdk');
const secretsManager = new AWS.SecretsManager();
const rds = new AWS.RDS();

exports.handler = async (event) => {
  const token = event.Token;
  const step = event.Step;
  
  if (step === 'createSecret') {
    // Generate new password
    const newPassword = generatePassword();
    await secretsManager.putSecretValue({
      SecretId: event.SecretId,
      ClientRequestToken: token,
      SecretString: newPassword,
      VersionStages: ['AWSPENDING']
    }).promise();
  }
  
  if (step === 'setSecret') {
    // Update database with new password
    const newPassword = await getSecret(event.SecretId, 'AWSPENDING');
    await updateDatabasePassword(newPassword);
  }
  
  if (step === 'testSecret') {
    // Test new password works
    const newPassword = await getSecret(event.SecretId, 'AWSPENDING');
    await testDatabaseConnection(newPassword);
  }
  
  if (step === 'finishSecret') {
    // Mark new version as current
    await secretsManager.updateSecretVersionStage({
      SecretId: event.SecretId,
      VersionStage: 'AWSCURRENT',
      MoveToVersionId: token,
      RemoveFromVersionId: event.OldVersionId
    }).promise();
  }
};
```

### Rotation Checklist

- [ ] Generate new secret
- [ ] Test new secret in non-production
- [ ] Update secret in secrets manager
- [ ] Deploy application with new secret
- [ ] Verify application works
- [ ] Remove old secret after grace period (24 hours)
- [ ] Document rotation in changelog
- [ ] Notify team of rotation

---

## 🚨 Emergency Procedures

### Secret Leaked/Compromised

**IMMEDIATE ACTION (within 1 hour):**

1. **Revoke the secret**
   ```bash
   # AWS - Delete access key
   aws iam delete-access-key \
     --access-key-id AKIA_LEAKED_KEY
   
   # Database - Change password immediately
   ALTER USER cropwise WITH PASSWORD 'new_emergency_password';
   ```

2. **Generate new secret**
   ```bash
   openssl rand -base64 32
   ```

3. **Update all systems**
   ```bash
   # Update secrets manager
   aws secretsmanager update-secret \
     --secret-id cropwise/prod/jwt-secret \
     --secret-string "new_secret_here"
   ```

4. **Force restart all services**
   ```bash
   # ECS
   aws ecs update-service \
     --cluster cropwise-prod-cluster \
     --service cropwise-backend \
     --force-new-deployment
   ```

5. **Invalidate all active sessions**
   ```javascript
   // In application
   await redis.flushall(); // Clear all sessions
   ```

6. **Monitor for suspicious activity**
   ```bash
   # Check CloudWatch logs
   aws logs filter-log-events \
     --log-group-name /ecs/cropwise-backend \
     --filter-pattern "ERROR"
   ```

### Secret Exposed in Git

**If secret committed to git:**

```bash
# 1. Remove from history (careful!)
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch path/to/file" \
  --prune-empty --tag-name-filter cat -- --all

# 2. Force push (only if absolutely necessary)
git push origin --force --all

# 3. Immediately rotate the exposed secret
# Follow rotation procedure above

# 4. Notify team
echo "Secret exposed and rotated. All team members must pull latest changes."
```

**Better approach - Use BFG Repo-Cleaner:**

```bash
# Install BFG
brew install bfg  # macOS
# or download from https://rtyley.github.io/bfg-repo-cleaner/

# Remove secrets
bfg --replace-text passwords.txt
git reflog expire --expire=now --all
git gc --prune=now --aggressive

# Force push
git push --force
```

---

## 📊 Audit & Compliance

### Secret Access Logging

**Enable CloudTrail for AWS:**

```bash
aws cloudtrail create-trail \
  --name cropwise-secrets-audit \
  --s3-bucket-name cropwise-cloudtrail-logs

aws cloudtrail start-logging \
  --name cropwise-secrets-audit
```

**Monitor secret access:**

```bash
# View who accessed secrets
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=GetSecretValue \
  --max-results 50
```

### Compliance Checklist

- [ ] All secrets encrypted at rest
- [ ] All secrets encrypted in transit (TLS)
- [ ] No secrets in source code
- [ ] No secrets in git history
- [ ] Secrets rotated per schedule
- [ ] Access to secrets logged
- [ ] Principle of least privilege applied
- [ ] Secrets isolated per environment
- [ ] Emergency procedures documented
- [ ] Team trained on secrets management

### Secrets Inventory

**Maintain a secrets inventory spreadsheet:**

| Secret Name | Type | Location | Owner | Last Rotated | Next Rotation |
|------------|------|----------|-------|--------------|---------------|
| DB_PASSWORD | Database | AWS Secrets Manager | DevOps | 2024-01-15 | 2024-04-15 |
| JWT_SECRET | Auth | AWS Secrets Manager | Backend | 2024-01-15 | 2024-04-15 |
| AWS Keys | Cloud | IAM | DevOps | 2024-02-01 | 2024-05-01 |

**Do NOT store actual secrets in this document!**

---

## 📚 Quick Reference

### Generate Secrets

```bash
# Strong password
openssl rand -base64 32

# Hex string
openssl rand -hex 32

# UUID
uuidgen

# Node.js
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"
```

### Check for Leaked Secrets

```bash
# Install truffleHog
pip install truffleHog

# Scan repository
trufflehog --regex --entropy=True .

# Scan git history
trufflehog --regex --entropy=True https://github.com/your-org/cropwise
```

### Validate .gitignore

```bash
# Check what's being tracked
git ls-files

# Ensure .env is not tracked
git ls-files | grep .env
# Should return nothing

# If .env is tracked, remove it
git rm --cached .env
git commit -m "chore: remove .env from tracking"
```

---

## 🛡️ Security Scanning

### Automated Secret Scanning

**GitHub Secret Scanning** (Free for public repos)
- Automatically enabled
- Scans for common secrets
- Alerts repository owners

**git-secrets (AWS)**

```bash
# Install
brew install git-secrets

# Setup
cd cropwise
git secrets --install
git secrets --register-aws

# Scan
git secrets --scan
git secrets --scan-history
```

**detect-secrets**

```bash
# Install
pip install detect-secrets

# Create baseline
detect-secrets scan > .secrets.baseline

# Check for new secrets
detect-secrets scan --baseline .secrets.baseline
```

---

## 📖 Additional Resources

### Official Documentation
- [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/)
- [GitHub Actions Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
- [OWASP Secrets Management](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)

### Tools
- [1Password CLI](https://1password.com/downloads/command-line/)
- [HashiCorp Vault](https://www.vaultproject.io/)
- [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/)
- [Doppler](https://www.doppler.com/)

---

## 🆘 Support

**Questions about secrets management?**
- Slack: #security channel
- Email: security@cropwise.io
- Emergency: security-urgent@cropwise.io

**Report security issues:**
- Email: security@cropwise.io
- Do NOT create public GitHub issues for security vulnerabilities

---

**Last Updated:** November 2024  
**Version:** 1.0.0  
**Next Review:** February 2025

**Remember: Security is everyone's responsibility!** 🔐

