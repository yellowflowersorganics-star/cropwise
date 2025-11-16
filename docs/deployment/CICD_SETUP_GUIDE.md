# 🚀 CI/CD Setup Guide for CropWise

Complete guide to set up and manage your CI/CD pipeline with GitHub Actions.

**Repository**: https://github.com/yellowflowersorganics-star/cropwise

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [GitHub Secrets Setup](#github-secrets-setup)
4. [CI/CD Pipeline Workflow](#cicd-pipeline-workflow)
5. [Testing the Pipeline](#testing-the-pipeline)
6. [Monitoring & Troubleshooting](#monitoring--troubleshooting)
7. [Best Practices](#best-practices)

---

## 🎯 Overview

Your CropWise repository has **two automated workflows**:

### **1. CI/CD Pipeline** (`.github/workflows/ci.yml`)
**Triggers**: Every push and pull request to `main` or `develop`

**What it does**:
- ✅ Lints code (ESLint)
- ✅ Runs unit tests (Backend + Frontend)
- ✅ Builds Docker images
- ✅ Runs security scans (Trivy)
- ✅ Deploys to DEV (on `develop` push)
- ✅ Deploys to PRODUCTION (on `main` push, with approval)
- ✅ Sends Slack notifications

### **2. Release Workflow** (`.github/workflows/release.yml`)
**Triggers**: When you push a version tag (e.g., `v1.0.0`)

**What it does**:
- ✅ Creates GitHub release with notes
- ✅ Builds production Docker images
- ✅ Pushes to Docker Hub
- ✅ Deploys to production
- ✅ Runs health checks
- ✅ Notifies team

---

## ✅ Prerequisites

Before setting up CI/CD, ensure you have:

- [x] GitHub repository created ✓
- [x] `main` and `develop` branches ✓
- [x] GitHub Actions enabled (automatic)
- [ ] AWS account (for deployment)
- [ ] Docker Hub account (optional, for images)
- [ ] Slack workspace (optional, for notifications)

---

## 🔐 GitHub Secrets Setup

Visit: https://github.com/yellowflowersorganics-star/cropwise/settings/secrets/actions

### **Required Secrets (Add These Now)**

Click "New repository secret" for each:

#### **1. Frontend API URL**
| Secret Name | Value | Description |
|-------------|-------|-------------|
| `VITE_API_URL` | `http://localhost:3000` | For local dev |
|  | OR `https://api-dev.yourdomain.com` | For DEV deployment |
|  | OR `https://api.yourdomain.com` | For PROD deployment |

**Note**: You can use environment-specific URLs by adding more secrets like:
- `VITE_API_URL_DEV`
- `VITE_API_URL_STAGING`
- `VITE_API_URL_PROD`

---

### **AWS Deployment Secrets (Add When Deploying)**

#### **2. AWS Credentials for DEV/Staging**
| Secret Name | Example Value | Where to Get |
|-------------|---------------|--------------|
| `AWS_ACCESS_KEY_ID` | `AKIAIOSFODNN7EXAMPLE` | AWS IAM Console |
| `AWS_SECRET_ACCESS_KEY` | `wJalrXUtnFEMI/K7MDENG...` | AWS IAM Console |

**How to create**:
```bash
# 1. Go to AWS Console → IAM → Users
# 2. Select your user → Security credentials
# 3. Click "Create access key"
# 4. Download CSV and save securely
# 5. Add to GitHub secrets
```

#### **3. AWS Credentials for Production (Separate Keys!)**
| Secret Name | Example Value | Why Separate? |
|-------------|---------------|---------------|
| `AWS_ACCESS_KEY_ID_PROD` | `AKIAXXXPRODXXX` | Security isolation |
| `AWS_SECRET_ACCESS_KEY_PROD` | `wJalrXXXPRODXXX...` | Prevent accidents |

---

#### **4. CloudFront Distribution IDs**
| Secret Name | Example Value | Where to Find |
|-------------|---------------|---------------|
| `CLOUDFRONT_DEV_DISTRIBUTION_ID` | `E1234ABCDEF` | AWS CloudFront Console |
| `CLOUDFRONT_PROD_DISTRIBUTION_ID` | `E5678GHIJKL` | AWS CloudFront Console |

**How to find**:
```bash
# AWS Console → CloudFront → Distributions
# Copy the "ID" column value
```

---

### **Optional Secrets (Add As Needed)**

#### **5. Google OAuth (For Authentication)**
| Secret Name | Where to Get |
|-------------|--------------|
| `GOOGLE_CLIENT_ID` | Google Cloud Console → APIs & Services → Credentials |
| `GOOGLE_CLIENT_SECRET` | Same location |

#### **6. Twilio (For SMS/WhatsApp Notifications)**
| Secret Name | Where to Get |
|-------------|--------------|
| `TWILIO_ACCOUNT_SID` | Twilio Console → Account Info |
| `TWILIO_AUTH_TOKEN` | Same location |

#### **7. Slack (For Deployment Notifications)**
| Secret Name | Where to Get |
|-------------|--------------|
| `SLACK_WEBHOOK_URL` | Slack → Apps → Incoming Webhooks |

**How to create Slack webhook**:
```bash
# 1. Go to https://api.slack.com/apps
# 2. Create New App → From scratch
# 3. Enable "Incoming Webhooks"
# 4. Add webhook to workspace
# 5. Copy webhook URL
# 6. Add to GitHub secrets
```

#### **8. Docker Hub (For Hosting Images)**
| Secret Name | Where to Get |
|-------------|--------------|
| `DOCKERHUB_USERNAME` | Your Docker Hub username |
| `DOCKERHUB_TOKEN` | Account Settings → Security → New Access Token |

---

## 🔄 CI/CD Pipeline Workflow

### **Pipeline Stages**

```
┌─────────────────────────────────────────────────────────┐
│              CropWise CI/CD Pipeline                │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. CODE QUALITY                                        │
│     ├─→ Lint Backend (ESLint)                          │
│     └─→ Lint Frontend (ESLint)                         │
│                                                         │
│  2. TESTING                                             │
│     ├─→ Backend Unit Tests (Jest)                      │
│     │   └─→ PostgreSQL + Redis (test containers)       │
│     └─→ Frontend Tests (Vitest)                        │
│                                                         │
│  3. BUILD                                               │
│     ├─→ Build Backend Docker Image                     │
│     └─→ Build Frontend (Vite)                          │
│                                                         │
│  4. SECURITY                                            │
│     ├─→ Trivy Vulnerability Scan                       │
│     └─→ npm audit (dependencies)                       │
│                                                         │
│  5. DEPLOY (if on develop branch)                      │
│     ├─→ Push to AWS ECR                                │
│     ├─→ Update ECS Service                             │
│     ├─→ Deploy Frontend to S3                          │
│     ├─→ Invalidate CloudFront Cache                    │
│     └─→ Send Slack Notification                        │
│                                                         │
│  6. PRODUCTION DEPLOY (if on main branch)              │
│     ├─→ ⏸️ Manual Approval Required                     │
│     ├─→ Deploy to Production                           │
│     ├─→ Run Health Checks                              │
│     ├─→ Create GitHub Release                          │
│     └─→ Notify Team                                    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

### **Deployment Strategy**

| Branch | Environment | Auto-Deploy | Approval Required | URL |
|--------|-------------|-------------|-------------------|-----|
| `develop` | Development | ✅ Yes | ❌ No | dev.cropwise.io |
| `release/*` | Staging | ✅ Yes | ❌ No | staging.cropwise.io |
| `main` | Production | ⏸️ Manual | ✅ Yes | www.cropwise.io |

---

## 🧪 Testing the Pipeline

### **Step 1: Test with Feature Branch**

Let's create a test branch and verify the pipeline works:

```bash
# 1. Switch to develop branch
git checkout develop
git pull origin develop

# 2. Create a test feature branch
git checkout -b feature/test-cicd

# 3. Make a small change
echo "" >> README.md

# 4. Commit and push
git add README.md
git commit -m "test(ci): verify CI/CD pipeline functionality"
git push origin feature/test-cicd
```

### **Step 2: Create Pull Request**

1. Go to: https://github.com/yellowflowersorganics-star/cropwise/pulls
2. Click "New pull request"
3. **Base**: `develop` ← **Compare**: `feature/test-cicd`
4. Click "Create pull request"
5. Fill in the PR template

### **Step 3: Watch the Pipeline**

1. Go to **Actions** tab: https://github.com/yellowflowersorganics-star/cropwise/actions
2. Click on your workflow run
3. Watch each job execute:
   - ✅ Lint Code
   - ✅ Test Backend
   - ✅ Test Frontend
   - ✅ Build Backend
   - ✅ Build Frontend
   - ✅ Security Scan

### **Step 4: Check Status Checks**

Back in your PR:
- All status checks should be **green** ✅
- If any fail, click "Details" to see logs
- Fix issues and push again

### **Step 5: Merge PR**

If all checks pass:
1. Click "Merge pull request"
2. Choose "Squash and merge"
3. Confirm merge
4. **Watch**: Deploy to DEV job will run automatically!

---

## 🎯 Testing Deployment Pipeline

### **Test DEV Deployment**

```bash
# 1. Make sure secrets are added (AWS keys, etc.)
# 2. Push to develop branch
git checkout develop
git pull origin develop

# 3. Make a change
echo "# Test deployment" >> README.md
git add README.md
git commit -m "test(deploy): verify DEV deployment"
git push origin develop

# 4. Watch Actions tab
# Should see "Deploy to Development" job run
```

**What to verify**:
- ✅ Docker image builds successfully
- ✅ Image pushed to ECR
- ✅ ECS service updated
- ✅ Frontend deployed to S3
- ✅ CloudFront cache invalidated
- ✅ Slack notification sent (if configured)

---

### **Test Production Deployment**

```bash
# 1. Create release branch
git checkout develop
git pull origin develop
git checkout -b release/v1.0.1

# 2. Update version
# Edit package.json versions
# Update CHANGELOG.md

git add .
git commit -m "chore: bump version to 1.0.1"
git push origin release/v1.0.1

# 3. Create PR to main
# Go to GitHub, create PR: base main ← compare release/v1.0.1

# 4. Merge PR (after approval)
# Production deployment will start

# 5. Manual approval required
# Go to Actions tab → Click running workflow
# Click "Review deployments" → Approve

# 6. Watch deployment complete
```

---

## 📊 Monitoring & Troubleshooting

### **Viewing Workflow Runs**

**All Runs**: https://github.com/yellowflowersorganics-star/cropwise/actions

**Filter by**:
- Event (push, pull_request)
- Branch (main, develop)
- Status (success, failure)
- Actor (who triggered)

---

### **Common Issues & Solutions**

#### **❌ Issue: Lint Errors**

**Symptom**: "Lint Code" job fails

**Solution**:
```bash
# Run locally to see errors
cd backend
npm run lint

cd ../frontend
npm run lint

# Fix errors, commit, push
```

---

#### **❌ Issue: Tests Failing**

**Symptom**: "Test Backend" or "Test Frontend" fails

**Solution**:
```bash
# Run tests locally
cd backend
npm test

cd ../frontend
npm test

# Fix failing tests
# Commit and push
```

---

#### **❌ Issue: Build Fails**

**Symptom**: "Build Backend" or "Build Frontend" fails

**Solution**:
```bash
# Check if it builds locally
cd backend
docker build -t test-backend .

cd ../frontend
npm run build

# Fix build errors
```

---

#### **❌ Issue: Security Scan Fails**

**Symptom**: Trivy finds vulnerabilities

**Solution**:
```bash
# Update dependencies
cd backend
npm audit fix
npm update

cd ../frontend
npm audit fix
npm update

# Commit updated package files
git add package*.json
git commit -m "fix(deps): update dependencies for security"
git push
```

---

#### **❌ Issue: Deployment Fails**

**Symptom**: "Deploy to Development" fails

**Common causes**:
1. **Missing secrets**: Check GitHub secrets are added
2. **AWS permissions**: Verify IAM user has correct permissions
3. **Wrong AWS region**: Check region matches your resources
4. **ECS service doesn't exist**: Create it first

**Solution**:
```bash
# Check AWS credentials work
aws sts get-caller-identity

# Verify ECS cluster exists
aws ecs list-clusters

# Check S3 bucket exists
aws s3 ls s3://your-bucket-name
```

---

### **Reading Logs**

**In GitHub Actions**:
1. Click on the failed job
2. Expand failed step
3. Read error message
4. Copy relevant logs for debugging

**Tip**: Look for:
- `Error:` messages
- `FAILED` status
- `exit code 1` or higher
- Stack traces

---

## 🔧 Customizing the Pipeline

### **Skip CI for Certain Commits**

Add `[skip ci]` to commit message:

```bash
git commit -m "docs: update README [skip ci]"
```

**When to use**:
- Documentation-only changes
- Minor README updates
- Non-code changes

---

### **Run Specific Jobs Only**

Edit `.github/workflows/ci.yml`:

```yaml
# Only run on certain paths
on:
  push:
    paths:
      - 'backend/**'  # Only backend changes
      - 'frontend/**'  # Only frontend changes
```

---

### **Add More Status Checks**

Edit `.github/workflows/ci.yml` and add jobs:

```yaml
  e2e-tests:
    name: E2E Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run E2E Tests
        run: npm run test:e2e
```

---

### **Custom Deployment Steps**

Add to deploy job:

```yaml
  - name: Run Database Migrations
    run: |
      aws ecs run-task \
        --cluster cropwise-prod \
        --task-definition migration-task \
        --launch-type FARGATE

  - name: Warm Up Cache
    run: |
      curl -X POST https://api.yourdomain.com/admin/warm-cache
```

---

## ✅ Best Practices

### **1. Never Commit Secrets**

❌ **Bad**:
```javascript
const apiKey = "sk_live_abc123";
```

✅ **Good**:
```javascript
const apiKey = process.env.API_KEY;
```

---

### **2. Use Matrix Strategy for Multi-Version Tests**

```yaml
strategy:
  matrix:
    node-version: [18, 20]
    database: [postgres, mysql]
```

---

### **3. Cache Dependencies**

Already configured in workflows:
```yaml
- uses: actions/setup-node@v4
  with:
    cache: 'npm'  # Speeds up installs
```

---

### **4. Parallel Jobs**

Run independent jobs in parallel:
```yaml
jobs:
  lint:     # Runs in parallel with test
  test:     # Runs in parallel with lint
  build:    # Waits for lint + test
    needs: [lint, test]
```

---

### **5. Fail Fast**

Stop early if critical checks fail:
```yaml
strategy:
  fail-fast: true
```

---

### **6. Use Environments for Protection**

Already configured:
```yaml
environment:
  name: production
  url: https://www.cropwise.io
```

This requires manual approval before production deploy.

---

## 📈 Monitoring Success

### **Key Metrics to Track**

1. **Build Success Rate**: Target > 95%
2. **Average Build Time**: Target < 10 minutes
3. **Deployment Frequency**: Target 2-4 times/week
4. **Mean Time to Recovery**: Target < 1 hour
5. **Change Failure Rate**: Target < 15%

### **View Insights**

Visit: https://github.com/yellowflowersorganics-star/cropwise/insights/actions

Shows:
- Workflow run times
- Success/failure rates
- Most run workflows

---

## 🎓 Advanced Features

### **1. Scheduled Runs**

Run tests nightly:
```yaml
on:
  schedule:
    - cron: '0 2 * * *'  # 2 AM daily
```

### **2. Manual Triggers**

Allow manual workflow runs:
```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        default: 'staging'
```

### **3. Reusable Workflows**

Create workflow templates to reuse:
```yaml
# .github/workflows/reusable-deploy.yml
on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
```

---

## 📞 Getting Help

### **GitHub Actions Documentation**
- https://docs.github.com/en/actions

### **Debugging Actions**
- Enable debug logging: Add secret `ACTIONS_STEP_DEBUG` = `true`

### **Community**
- GitHub Community: https://github.community
- Stack Overflow: Tag `github-actions`

---

## ✅ Setup Checklist

- [ ] All GitHub secrets added
- [ ] First workflow run successful
- [ ] Status checks appearing in PRs
- [ ] DEV deployment working
- [ ] Production deployment requires approval
- [ ] Slack notifications configured (optional)
- [ ] Docker Hub publishing working (optional)
- [ ] Team understands workflow
- [ ] Monitoring dashboard set up
- [ ] Rollback procedure documented

---

## 🎯 Quick Reference

### **View All Workflows**
```
https://github.com/yellowflowersorganics-star/cropwise/actions
```

### **Re-run Failed Job**
1. Click on workflow run
2. Click "Re-run jobs"
3. Select "Re-run failed jobs"

### **Cancel Running Workflow**
1. Click on running workflow
2. Click "Cancel workflow"

### **Download Logs**
1. Click on completed workflow
2. Click ⋮ (three dots)
3. Click "Download log archive"

---

**🎉 Your CI/CD pipeline is ready! Every push will trigger automated testing and deployment!**

*Last updated: November 2025*

