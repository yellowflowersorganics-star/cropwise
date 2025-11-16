# 🐙 GitHub Repository Setup Guide

Complete guide to set up your CropWise repository on GitHub.

**Repository**: `https://github.com/yellowflowersorganics-star/cropwise`

---

## ✅ Initial Setup Checklist

### 1. **Repository Settings**

Visit: `https://github.com/yellowflowersorganics-star/cropwise/settings`

#### **General Settings**

- [ ] **Repository name**: `cropwise`
- [ ] **Description**: Add project description
  ```
  🌱 Enterprise IoT-Powered Farm Management Platform - AI-ready, scalable smart agriculture software
  ```
- [ ] **Website**: Add your website URL (if you have one)
- [ ] **Topics**: Add relevant tags
  ```
  agriculture, iot, farming, esp32, mqtt, nodejs, react, postgresql, smart-farming, 
  precision-agriculture, mushroom-farming, vertical-farming, greenhouse, agtech
  ```
- [ ] **Features**:
  - ✅ **Wikis**: Enable for additional documentation
  - ✅ **Issues**: Enable for bug tracking
  - ✅ **Discussions**: Enable for community Q&A
  - ❌ **Projects**: Optional (enable if you want project boards)
  - ❌ **Sponsorships**: Optional

---

### 2. **Create Environment Branches**

We follow a **multi-environment branching strategy** that maps to AWS environments:

```bash
# Create develop branch from main
git checkout -b develop
git push origin develop

# Create staging branch from main
git checkout main
git checkout -b staging
git push origin staging

# Set develop as default branch on GitHub (recommended for development)
# Go to Settings → Branches → Default branch → Change to 'develop'
```

---

#### **Branch-to-Environment Mapping**

| Branch | Environment | AWS Resources | Auto-Deploy | Purpose |
|--------|-------------|---------------|-------------|---------|
| `develop` | **Development** | cropwise-dev-* | ✅ Yes | Active development, frequent changes |
| `staging` | **Staging** | cropwise-stage-* | ✅ Yes | Pre-production testing, QA validation |
| `main` | **Production** | cropwise-prod-* | ✅ Yes | Live production, stable releases |
| `feature/*` | Local | None | ❌ No | Feature development, no deploy |
| `hotfix/*` | All 3 | All environments | ⚠️ Manual | Critical bug fixes |

---

#### **Workflow**

```
┌─────────────┐
│ feature/xyz │  ← Developer creates feature branch
└──────┬──────┘
       │ PR + Review
       ↓
┌─────────────┐
│   develop   │  ← Auto-deploys to Development (cropwise-dev-*)
└──────┬──────┘
       │ PR + Testing
       ↓
┌─────────────┐
│   staging   │  ← Auto-deploys to Staging (cropwise-stage-*)
└──────┬──────┘
       │ PR + Approval
       ↓
┌─────────────┐
│     main    │  ← Auto-deploys to Production (cropwise-prod-*)
└─────────────┘
```

**Why 3 branches?**
- `develop` = Daily development, latest features (may be unstable)
- `staging` = Pre-production testing, QA approval needed
- `main` = Production-ready, requires multiple approvals
- Complete isolation between environments
- Test changes thoroughly before production

---

### 3. **Branch Protection Rules**

Visit: `https://github.com/yellowflowersorganics-star/cropwise/settings/branches`

Configure protection for all 3 environment branches with appropriate strictness levels.

---

#### **Protect `main` Branch (Production)**

Click "Add rule" and configure:

**Branch name pattern**: `main`

**Settings to enable**:
- ✅ **Require a pull request before merging**
  - ✅ Require approvals: **2** (strict for production)
  - ✅ Dismiss stale pull request approvals when new commits are pushed
  - ✅ Require review from Code Owners
  - ✅ Require approval from someone other than last pusher
  
- ✅ **Require status checks to pass before merging**
  - ✅ Require branches to be up to date before merging
  - Status checks to require (add after CI/CD is set up):
    - `lint-backend`
    - `lint-frontend`
    - `test-backend`
    - `test-frontend`
    - `test-integration`
    - `security-scan`
    - `build-backend`
    - `build-frontend`
    - `deploy-staging` (staging must pass before production)

- ✅ **Require conversation resolution before merging**

- ✅ **Require signed commits** (recommended for production)

- ✅ **Require linear history** (clean history for production)

- ✅ **Require deployments to succeed before merging** (optional)
  - Environment: `production`

- ✅ **Include administrators** (apply rules to admins too)

- ✅ **Restrict who can push to matching branches**
  - Add only: `Maintainers` or `Release Managers`

- ❌ **Allow force pushes**: **Disabled** (never force push to production!)
- ❌ **Allow deletions**: **Disabled**

**Save changes**

---

#### **Protect `staging` Branch (Staging)**

Click "Add rule" again:

**Branch name pattern**: `staging`

**Settings to enable**:
- ✅ **Require a pull request before merging**
  - ✅ Require approvals: **1** (moderate for staging)
  - ✅ Dismiss stale pull request approvals when new commits are pushed
  
- ✅ **Require status checks to pass before merging**
  - ✅ Require branches to be up to date before merging
  - Status checks to require:
    - `lint-backend`
    - `lint-frontend`
    - `test-backend`
    - `test-frontend`
    - `test-integration`
    - `security-scan`
    - `build-backend`
    - `build-frontend`

- ✅ **Require conversation resolution before merging**

- ⭕ **Require signed commits** (optional for staging)

- ⭕ **Require linear history** (optional for staging)

- ✅ **Include administrators**

- ✅ **Restrict who can push to matching branches**
  - Add: `Developers` team

- ❌ **Allow force pushes**: **Disabled** (staging should be stable)
- ❌ **Allow deletions**: **Disabled**

**Save changes**

---

#### **Protect `develop` Branch (Development)**

Click "Add rule" again:

**Branch name pattern**: `develop`

**Settings to enable**:
- ✅ **Require a pull request before merging**
  - ✅ Require approvals: **1** (lighter for development)
  - ⭕ Dismiss stale pull request approvals (optional)
  
- ✅ **Require status checks to pass before merging**
  - ✅ Require branches to be up to date before merging
  - Status checks to require (minimal for faster development):
    - `lint-backend`
    - `lint-frontend`
    - `test-backend`
    - `test-frontend`
    - `build-backend`
    - `build-frontend`

- ✅ **Require conversation resolution before merging**

- ⭕ **Require signed commits** (optional)

- ⭕ **Require linear history** (optional)

- ✅ **Include administrators**

- ⭕ **Restrict who can push** (allow all developers)

- ✅ **Allow force pushes**: **Enabled** (allow rebasing for cleaner history)
  - ⚠️ Only for `develop`, not for `staging` or `main`!
- ❌ **Allow deletions**: **Disabled**

**Save changes**

---

#### **Branch Protection Summary**

| Setting | `main` (Prod) | `staging` | `develop` (Dev) |
|---------|---------------|-----------|-----------------|
| **Required approvals** | 2 | 1 | 1 |
| **Status checks** | All (9) | All (8) | Essential (6) |
| **Code owners** | Required | Optional | Optional |
| **Signed commits** | Required | Optional | Optional |
| **Linear history** | Required | Optional | Optional |
| **Force pushes** | ❌ Never | ❌ Never | ✅ Allowed |
| **Delete branch** | ❌ Never | ❌ Never | ❌ Never |
| **Direct push** | ❌ Never | ❌ Never | ❌ Never |

---

### 4. **GitHub Actions Secrets**

Visit: `https://github.com/yellowflowersorganics-star/cropwise/settings/secrets/actions`

Click "New repository secret" for each. **Total: 30+ secrets** for complete multi-environment setup.

---

#### **AWS Credentials**

| Secret Name | Value | Description |
|-------------|-------|-------------|
| `AWS_REGION` | `ap-south-1` | AWS region for all resources |
| `AWS_ACCESS_KEY_ID` | Your AWS access key | AWS authentication |
| `AWS_SECRET_ACCESS_KEY` | Your AWS secret key | AWS authentication |
| `AWS_ECR_REPOSITORY` | `cropwise-backend` | ECR repository name |

---

#### **Database URLs (3 environments)**

| Secret Name | Value | Description |
|-------------|-------|-------------|
| `DATABASE_URL_DEV` | `postgresql://cropwise_dev_admin:PASSWORD@cropwise-dev-db.XXX.rds.amazonaws.com:5432/cropwise_dev` | Development database |
| `DATABASE_URL_STAGE` | `postgresql://cropwise_stage_admin:PASSWORD@cropwise-stage-db.XXX.rds.amazonaws.com:5432/cropwise_stage` | Staging database |
| `DATABASE_URL_PROD` | `postgresql://cropwise_prod_admin:PASSWORD@cropwise-prod-db.XXX.rds.amazonaws.com:5432/cropwise_prod` | Production database |

---

#### **Development Environment**

| Secret Name | Value | Description |
|-------------|-------|-------------|
| `BACKEND_URL_DEV` | `http://cropwise-dev-alb-XXX.elb.amazonaws.com` | Dev backend ALB URL |
| `FRONTEND_URL_DEV` | `https://dXXXXXXXXX.cloudfront.net` | Dev frontend CloudFront URL |
| `CLOUDFRONT_DISTRIBUTION_ID_DEV` | `EXXXXXXXXXXXXX` | Dev CloudFront distribution ID |
| `S3_BUCKET_DEV` | `cropwise-dev-frontend` | Dev S3 bucket name |
| `ECS_CLUSTER_DEV` | `cropwise-dev-cluster` | Dev ECS cluster name |
| `ECS_SERVICE_DEV` | `cropwise-backend-dev-service` | Dev ECS service name |
| `ECS_TASK_DEFINITION_DEV` | `cropwise-backend-dev` | Dev task definition family |

---

#### **Staging Environment**

| Secret Name | Value | Description |
|-------------|-------|-------------|
| `BACKEND_URL_STAGE` | `http://cropwise-stage-alb-YYY.elb.amazonaws.com` | Stage backend ALB URL |
| `FRONTEND_URL_STAGE` | `https://dYYYYYYYYY.cloudfront.net` | Stage frontend CloudFront URL |
| `CLOUDFRONT_DISTRIBUTION_ID_STAGE` | `EYYYYYYYYYYY` | Stage CloudFront distribution ID |
| `S3_BUCKET_STAGE` | `cropwise-stage-frontend` | Stage S3 bucket name |
| `ECS_CLUSTER_STAGE` | `cropwise-stage-cluster` | Stage ECS cluster name |
| `ECS_SERVICE_STAGE` | `cropwise-backend-stage-service` | Stage ECS service name |
| `ECS_TASK_DEFINITION_STAGE` | `cropwise-backend-stage` | Stage task definition family |

---

#### **Production Environment**

| Secret Name | Value | Description |
|-------------|-------|-------------|
| `BACKEND_URL_PROD` | `http://cropwise-prod-alb-ZZZ.elb.amazonaws.com` | Prod backend ALB URL |
| `FRONTEND_URL_PROD` | `https://dZZZZZZZZZ.cloudfront.net` | Prod frontend CloudFront URL |
| `CLOUDFRONT_DISTRIBUTION_ID_PROD` | `EZZZZZZZZZZ` | Prod CloudFront distribution ID |
| `S3_BUCKET_PROD` | `cropwise-prod-frontend` | Prod S3 bucket name |
| `ECS_CLUSTER_PROD` | `cropwise-prod-cluster` | Prod ECS cluster name |
| `ECS_SERVICE_PROD` | `cropwise-backend-prod-service` | Prod ECS service name |
| `ECS_TASK_DEFINITION_PROD` | `cropwise-backend-prod` | Prod task definition family |

---

#### **Application Secrets (Shared)**

| Secret Name | Value | Description |
|-------------|-------|-------------|
| `JWT_SECRET` | Generate strong random string | JWT token signing |
| `SESSION_SECRET` | Generate strong random string | Session encryption |
| `GOOGLE_CLIENT_ID` | Your Google OAuth client ID | Google authentication |
| `GOOGLE_CLIENT_SECRET` | Your Google OAuth secret | Google authentication |

---

#### **Notifications (Optional)**

| Secret Name | Value | Description |
|-------------|-------|-------------|
| `SLACK_WEBHOOK_URL` | Your Slack webhook URL | Deployment notifications |
| `TWILIO_ACCOUNT_SID` | Your Twilio SID | SMS/WhatsApp notifications |
| `TWILIO_AUTH_TOKEN` | Your Twilio auth token | SMS/WhatsApp notifications |

---

#### **Quick Add All Secrets (PowerShell)**

```powershell
# Set repository
$REPO = "yellowflowersorganics-star/cropwise"

# AWS Core
gh secret set AWS_REGION --body "ap-south-1" --repo $REPO
gh secret set AWS_ACCESS_KEY_ID --body "YOUR_AWS_KEY" --repo $REPO
gh secret set AWS_SECRET_ACCESS_KEY --body "YOUR_AWS_SECRET" --repo $REPO
gh secret set AWS_ECR_REPOSITORY --body "cropwise-backend" --repo $REPO

# Databases
gh secret set DATABASE_URL_DEV --body "postgresql://USER:PASS@HOST:5432/cropwise_dev" --repo $REPO
gh secret set DATABASE_URL_STAGE --body "postgresql://USER:PASS@HOST:5432/cropwise_stage" --repo $REPO
gh secret set DATABASE_URL_PROD --body "postgresql://USER:PASS@HOST:5432/cropwise_prod" --repo $REPO

# Development
gh secret set BACKEND_URL_DEV --body "http://YOUR-DEV-ALB.elb.amazonaws.com" --repo $REPO
gh secret set FRONTEND_URL_DEV --body "https://YOUR-DEV-CF.cloudfront.net" --repo $REPO
gh secret set CLOUDFRONT_DISTRIBUTION_ID_DEV --body "EXXXX" --repo $REPO
gh secret set S3_BUCKET_DEV --body "cropwise-dev-frontend" --repo $REPO
gh secret set ECS_CLUSTER_DEV --body "cropwise-dev-cluster" --repo $REPO
gh secret set ECS_SERVICE_DEV --body "cropwise-backend-dev-service" --repo $REPO
gh secret set ECS_TASK_DEFINITION_DEV --body "cropwise-backend-dev" --repo $REPO

# Staging
gh secret set BACKEND_URL_STAGE --body "http://YOUR-STAGE-ALB.elb.amazonaws.com" --repo $REPO
gh secret set FRONTEND_URL_STAGE --body "https://YOUR-STAGE-CF.cloudfront.net" --repo $REPO
gh secret set CLOUDFRONT_DISTRIBUTION_ID_STAGE --body "EYYYY" --repo $REPO
gh secret set S3_BUCKET_STAGE --body "cropwise-stage-frontend" --repo $REPO
gh secret set ECS_CLUSTER_STAGE --body "cropwise-stage-cluster" --repo $REPO
gh secret set ECS_SERVICE_STAGE --body "cropwise-backend-stage-service" --repo $REPO
gh secret set ECS_TASK_DEFINITION_STAGE --body "cropwise-backend-stage" --repo $REPO

# Production
gh secret set BACKEND_URL_PROD --body "http://YOUR-PROD-ALB.elb.amazonaws.com" --repo $REPO
gh secret set FRONTEND_URL_PROD --body "https://YOUR-PROD-CF.cloudfront.net" --repo $REPO
gh secret set CLOUDFRONT_DISTRIBUTION_ID_PROD --body "EZZZZ" --repo $REPO
gh secret set S3_BUCKET_PROD --body "cropwise-prod-frontend" --repo $REPO
gh secret set ECS_CLUSTER_PROD --body "cropwise-prod-cluster" --repo $REPO
gh secret set ECS_SERVICE_PROD --body "cropwise-backend-prod-service" --repo $REPO
gh secret set ECS_TASK_DEFINITION_PROD --body "cropwise-backend-prod" --repo $REPO

# Application Secrets
gh secret set JWT_SECRET --body "$(openssl rand -base64 32)" --repo $REPO
gh secret set SESSION_SECRET --body "$(openssl rand -base64 32)" --repo $REPO
gh secret set GOOGLE_CLIENT_ID --body "YOUR_GOOGLE_CLIENT_ID" --repo $REPO
gh secret set GOOGLE_CLIENT_SECRET --body "YOUR_GOOGLE_CLIENT_SECRET" --repo $REPO
```

---

### 5. **Enable GitHub Actions & Workflows**

Visit: `https://github.com/yellowflowersorganics-star/cropwise/actions`

GitHub Actions should be automatically enabled. Your workflows should be configured for all 3 environments:

---

#### **Required Workflows**

Create these workflow files in `.github/workflows/`:

**1. Development Deployment** (`.github/workflows/deploy-dev.yml`)
```yaml
name: Deploy to Development

on:
  push:
    branches: [develop]
  workflow_dispatch:

jobs:
  deploy-dev:
    runs-on: ubuntu-latest
    environment: development
    steps:
      # Build and deploy to dev environment
      # Use: DATABASE_URL_DEV, ECS_CLUSTER_DEV, etc.
```

**2. Staging Deployment** (`.github/workflows/deploy-staging.yml`)
```yaml
name: Deploy to Staging

on:
  push:
    branches: [staging]
  workflow_dispatch:

jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment: staging
    steps:
      # Build and deploy to staging environment
      # Use: DATABASE_URL_STAGE, ECS_CLUSTER_STAGE, etc.
```

**3. Production Deployment** (`.github/workflows/deploy-prod.yml`)
```yaml
name: Deploy to Production

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy-prod:
    runs-on: ubuntu-latest
    environment: production
    steps:
      # Build and deploy to production environment
      # Use: DATABASE_URL_PROD, ECS_CLUSTER_PROD, etc.
      # Requires manual approval (set in GitHub environment settings)
```

---

#### **GitHub Environments Setup**

Visit: `https://github.com/yellowflowersorganics-star/cropwise/settings/environments`

Create 3 environments with different protection rules:

**1. Development Environment**
- Name: `development`
- Protection rules: None (auto-deploy)
- Reviewers: None required
- Wait timer: 0 minutes
- Deployment branches: Only `develop`

**2. Staging Environment**
- Name: `staging`
- Protection rules: Optional reviewers
- Reviewers: 1 developer (optional)
- Wait timer: 0 minutes
- Deployment branches: Only `staging`

**3. Production Environment**
- Name: `production`
- Protection rules: **Required reviewers**
- Reviewers: **2 maintainers** (must approve before deploy)
- Wait timer: 5 minutes (safety delay)
- Deployment branches: Only `main`
- ⚠️ This prevents accidental production deployments!

---

#### **Workflow Triggers Summary**

| Workflow | Trigger | Environment | Auto-Deploy | Requires Approval |
|----------|---------|-------------|-------------|-------------------|
| `deploy-dev.yml` | Push to `develop` | Development | ✅ Yes | ❌ No |
| `deploy-staging.yml` | Push to `staging` | Staging | ✅ Yes | ⭕ Optional |
| `deploy-prod.yml` | Push to `main` | Production | ⏸️ Wait 5min | ✅ Yes (2 people) |
| `ci.yml` | All PRs | None | N/A | N/A (just tests) |

---

#### **First Run**

1. **Test Development Deploy**:
   ```bash
   git checkout develop
   echo "# Test dev deploy" >> README.md
   git add README.md
   git commit -m "test: trigger dev deployment"
   git push origin develop
   ```
   - Check Actions tab → Should see "Deploy to Development"
   - Should auto-deploy to cropwise-dev-* resources

2. **Test Staging Deploy**:
   ```bash
   git checkout staging
   git merge develop
   git push origin staging
   ```
   - Check Actions tab → Should see "Deploy to Staging"
   - Should auto-deploy to cropwise-stage-* resources

3. **Test Production Deploy** (with approval):
   ```bash
   git checkout main
   git merge staging
   git push origin main
   ```
   - Check Actions tab → Should see "Deploy to Production"
   - Should wait for 2 approvals before deploying
   - Should deploy to cropwise-prod-* resources

---

#### **Troubleshooting Workflows**

- **Workflow not running?** Check if Actions are enabled in Settings → Actions
- **Secrets missing?** Verify all environment-specific secrets are added
- **Deployment failing?** Check AWS credentials and resource names
- **Approval not working?** Ensure environment protection rules are configured

---

### 6. **Repository Labels**

Visit: `https://github.com/yellowflowersorganics-star/cropwise/labels`

Add these labels for better issue/PR management:

#### **Type Labels**
- `bug` 🐛 - Something isn't working
- `enhancement` ✨ - New feature or request
- `documentation` 📚 - Documentation improvements
- `question` ❓ - Further information requested
- `duplicate` 👥 - Duplicate issue
- `invalid` ❌ - Invalid issue
- `wontfix` 🚫 - Won't be fixed

#### **Priority Labels**
- `priority: critical` 🔴 - Needs immediate attention
- `priority: high` 🟠 - High priority
- `priority: medium` 🟡 - Medium priority
- `priority: low` 🟢 - Low priority

#### **Status Labels**
- `status: in progress` 🔄 - Currently being worked on
- `status: blocked` 🚧 - Blocked by dependencies
- `status: needs review` 👀 - Needs code review
- `status: needs testing` 🧪 - Needs testing

#### **Area Labels**
- `area: backend` - Backend-related
- `area: frontend` - Frontend-related
- `area: iot` - IoT/Hardware-related
- `area: docs` - Documentation
- `area: ci/cd` - CI/CD pipeline

---

### 7. **Issue Templates**

Already created! ✅

Your issue templates are in:
- `.github/ISSUE_TEMPLATE/bug_report.md`
- `.github/ISSUE_TEMPLATE/feature_request.md`

Verify they appear when creating a new issue:
`https://github.com/yellowflowersorganics-star/cropwise/issues/new/choose`

---

### 8. **Pull Request Template**

Already created! ✅

Your PR template is in:
- `.github/PULL_REQUEST_TEMPLATE.md`

Verify it appears when creating a new PR.

---

### 9. **Repository About Section**

Visit: `https://github.com/yellowflowersorganics-star/cropwise`

Click the ⚙️ gear icon next to "About" and add:

**Description**:
```
🌱 Enterprise IoT-Powered Farm Management Platform - AI-ready, scalable smart agriculture software with ESP32 + MQTT + React + Node.js
```

**Website**: Your domain (if you have one)

**Topics**: (add all relevant tags)
```
agriculture, iot, farming, esp32, mqtt, nodejs, react, postgresql, 
smart-farming, precision-agriculture, mushroom-farming, vertical-farming, 
greenhouse, agtech, smart-agriculture, farm-management, crop-monitoring
```

**Check boxes**:
- [ ] ✅ Include in the home page

**Releases**: Will appear automatically when you create releases

---

### 10. **Create Initial Release**

Visit: `https://github.com/yellowflowersorganics-star/cropwise/releases/new`

**Tag version**: `v1.0.0`

**Release title**: `CropWise v1.0.0 - Initial Release`

**Description**: Use content from `CHANGELOG.md` or:

```markdown
# 🌱 CropWise v1.0.0 - Initial Release

## 🎉 First Production Release!

CropWise is now production-ready! This is the first stable release of our enterprise IoT-powered farm management platform.

### ✨ Key Features

#### **Farm Management**
- Complete farm and zone management
- Multi-stage growing recipes
- Batch tracking from inoculation to harvest
- Harvest recording with quality grading
- Real-time analytics and reporting

#### **IoT Integration**
- Hierarchical ESP32 Master-Slave architecture
- MQTT broker integration
- Real-time environmental monitoring
- Equipment control (fans, lights, irrigation)
- Recipe execution with stage approval

#### **Operations Management**
- Inventory tracking with low stock alerts
- Task management with assignments
- Employee management with RBAC
- Labor tracking and cost analysis
- Quality control and defect tracking
- SOP management

#### **Advanced Features**
- Multi-channel notifications (Email, SMS, WhatsApp)
- Profitability analytics (ROI, profit margins)
- Mobile-responsive UI
- Google OAuth authentication
- Comprehensive API documentation

### 🚀 Getting Started

See our [Installation Guide](https://github.com/yellowflowersorganics-star/cropwise/blob/main/docs/INSTALLATION.md)

### 📚 Documentation

- [User Guide](https://github.com/yellowflowersorganics-star/cropwise/blob/main/docs/USER_GUIDE.md)
- [Admin Guide](https://github.com/yellowflowersorganics-star/cropwise/blob/main/docs/ADMIN_GUIDE.md)
- [API Documentation](https://github.com/yellowflowersorganics-star/cropwise/blob/main/docs/)

### 🐛 Found a bug?

Please [report it](https://github.com/yellowflowersorganics-star/cropwise/issues/new?template=bug_report.md)

### 💬 Questions?

Check our [FAQ](https://github.com/yellowflowersorganics-star/cropwise/blob/main/docs/FAQ.md) or [start a discussion](https://github.com/yellowflowersorganics-star/cropwise/discussions)

---

**Full Changelog**: [View All Changes](https://github.com/yellowflowersorganics-star/cropwise/blob/main/CHANGELOG.md)
```

**Check**: ✅ Set as the latest release

**Publish release**

---

### 11. **Enable Discussions**

Visit: `https://github.com/yellowflowersorganics-star/cropwise/settings`

Scroll to "Features" → Check ✅ **Discussions**

Then set up categories:
- 💬 **General** - General discussion
- 💡 **Ideas** - Feature requests and ideas
- 🙏 **Q&A** - Questions and answers
- 🎉 **Show and tell** - Share your farm setup
- 📣 **Announcements** - Project announcements

---

### 12. **Add README Badges**

Already included in your README! ✅

Make sure they're visible:
- Version badge
- License badge
- Node.js badge
- PostgreSQL badge
- React badge

---

### 13. **Create Wiki (Optional)**

Visit: `https://github.com/yellowflowersorganics-star/cropwise/wiki`

Click "Create the first page"

**Suggested pages**:
- Home (overview)
- Installation
- Configuration
- Troubleshooting
- FAQ
- Hardware Setup
- Deployment

---

### 14. **Security Policy**

Create `.github/SECURITY.md`:

Visit: `https://github.com/yellowflowersorganics-star/cropwise/security/policy`

Or create the file manually with:

```markdown
# Security Policy

## Supported Versions

| Version | Supported          |
| ------- | ------------------ |
| 1.0.x   | :white_check_mark: |
| < 1.0   | :x:                |

## Reporting a Vulnerability

**DO NOT** create a public GitHub issue for security vulnerabilities.

Instead, please email: **security@cropwise.io**

Include:
- Description of vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (optional)

We commit to:
- Acknowledge within 24 hours
- Provide status update within 7 days
- Fix critical issues within 48 hours
- Credit you in security advisory (if desired)

## Security Best Practices

See our [Security Guide](https://github.com/yellowflowersorganics-star/cropwise/blob/main/docs/SECURITY_GUIDE.md)
```

---

### 15. **Code Owners (Optional)**

Create `.github/CODEOWNERS`:

```
# Global owners
* @your-github-username

# Backend
/backend/ @your-github-username

# Frontend
/frontend/ @your-github-username

# Documentation
/docs/ @your-github-username

# CI/CD
/.github/ @your-github-username

# IoT
/esp32-firmware/ @your-github-username
```

---

### 16. **Funding (Optional)**

Create `.github/FUNDING.yml`:

```yaml
# Sponsorship options
github: [your-github-username]
patreon: cropwiseos
ko_fi: cropwiseos
custom: ["https://cropwise.io/donate"]
```

---

## 🎯 Post-Setup Tasks

### **Immediate Actions**

1. **Test CI/CD Pipeline**
   ```bash
   # Create a test branch
   git checkout -b test/ci-pipeline
   
   # Make a small change
   echo "# Test" >> README.md
   
   # Commit and push
   git add README.md
   git commit -m "test: trigger CI pipeline"
   git push origin test/ci-pipeline
   
   # Create PR and watch Actions tab
   ```

2. **Verify Branch Protection**
   - Try to push directly to `main` (should fail)
   - Create a PR and verify approval is required

3. **Test Issue Templates**
   - Create a test bug report
   - Create a test feature request
   - Verify templates work correctly

4. **Test Pull Request Template**
   - Create a test PR
   - Verify template appears with all sections

### **Communication**

1. **Announce Repository**
   - Share repository link with team
   - Post in relevant communities
   - Update your website/profile

2. **Social Media** (if applicable)
   - Twitter/X
   - LinkedIn
   - Reddit (r/homeautomation, r/farming)

### **Documentation**

1. **Update Links**
   - Verify all GitHub links work
   - Check external links
   - Test documentation navigation

2. **Screenshots**
   - Add screenshots to README
   - Update documentation images
   - Create GIFs for features

---

## ✅ Final Checklist

### **Repository Setup**
- [ ] Repository description and topics added
- [ ] README badges visible
- [ ] Security policy added (`.github/SECURITY.md`)
- [ ] Code owners configured (`.github/CODEOWNERS`)
- [ ] Issue and PR templates verified
- [ ] Labels created (type, priority, status, area)
- [ ] Discussions enabled
- [ ] Wiki created (optional)

### **Branch Configuration**
- [ ] `develop` branch created (Development environment)
- [ ] `staging` branch created (Staging environment)
- [ ] `main` branch exists (Production environment)
- [ ] Default branch set to `develop`

### **Branch Protection (3 branches)**
- [ ] `main` protection: 2 approvals, all checks, signed commits
- [ ] `staging` protection: 1 approval, all checks
- [ ] `develop` protection: 1 approval, essential checks, allow force push

### **GitHub Secrets (30+ secrets)**
- [ ] AWS credentials (4 secrets)
- [ ] Database URLs (3 secrets: DEV, STAGE, PROD)
- [ ] Development environment (7 secrets)
- [ ] Staging environment (7 secrets)
- [ ] Production environment (7 secrets)
- [ ] Application secrets (4 secrets: JWT, Session, Google)
- [ ] Notifications (optional: 3 secrets)

### **GitHub Environments**
- [ ] `development` environment created (no approvals)
- [ ] `staging` environment created (optional approval)
- [ ] `production` environment created (2 approvals + 5min delay)

### **GitHub Actions Workflows**
- [ ] `.github/workflows/ci.yml` - CI/CD pipeline (PRs)
- [ ] `.github/workflows/deploy-dev.yml` - Deploy to development
- [ ] `.github/workflows/deploy-staging.yml` - Deploy to staging
- [ ] `.github/workflows/deploy-prod.yml` - Deploy to production
- [ ] `.github/workflows/pr-issue-validator.yml` - Issue validation

### **Testing**
- [ ] CI/CD pipeline tested (create test PR)
- [ ] Development deployment tested
- [ ] Staging deployment tested
- [ ] Production deployment tested (with approval)
- [ ] Branch protection verified (try direct push to main)

### **AWS Integration**
- [ ] All 3 RDS databases created (dev, stage, prod)
- [ ] All 3 ECS clusters created
- [ ] All 3 Application Load Balancers created
- [ ] All 3 S3 buckets created
- [ ] All 3 CloudFront distributions created
- [ ] URLs added to GitHub Secrets

### **Documentation**
- [ ] Initial release created (v1.0.0)
- [ ] Documentation links verified
- [ ] Screenshots added to README
- [ ] CHANGELOG.md updated

### **Team & Access**
- [ ] Team members invited
- [ ] Roles assigned (Maintainers, Developers)
- [ ] Repository shared with team

---

## 🆘 Troubleshooting

### **GitHub Actions not running?**

1. Check if Actions are enabled: Settings → Actions
2. Verify secrets are added correctly
3. Check workflow syntax (YAML)
4. Look for error messages in Actions tab

### **Branch protection not working?**

1. Ensure you're not an admin (admins can bypass rules)
2. Check "Include administrators" is enabled
3. Verify status checks are correctly named

### **Can't push to repository?**

1. Check your authentication (SSH key or token)
2. Verify you have write access
3. Ensure branch protection allows your push

---

## 📞 Need Help?

- **GitHub Docs**: https://docs.github.com
- **Community Forum**: https://github.community
- **Support**: support@github.com

---

**🎉 Congratulations! Your repository is now professionally set up!**

*Last updated: November 2025*

