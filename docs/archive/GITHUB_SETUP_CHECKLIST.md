# ✅ GitHub Setup Checklist for CropWise

Follow these steps to complete your GitHub repository setup.

**Repository**: https://github.com/yellowflowersorganics-star/cropwise

---

## ✅ Step 1: Develop Branch Created ✓

**Status**: COMPLETED ✓

The `develop` branch has been created and pushed to GitHub.

---

## 📋 Step 2: Set Default Branch to `develop`

**Why?** Following GitFlow, developers should branch from `develop`, not `main`.

**Steps**:

1. Go to: https://github.com/yellowflowersorganics-star/cropwise/settings/branches

2. Under "Default branch", click the ↔️ switch icon next to `main`

3. Select `develop` from the dropdown

4. Click "Update"

5. Confirm the change

**Result**: New PRs will default to `develop` branch

---

## 🔒 Step 3: Enable Branch Protection Rules

### Protect `main` Branch

**URL**: https://github.com/yellowflowersorganics-star/cropwise/settings/branch_protection_rules/new

**Settings**:

1. **Branch name pattern**: `main`

2. **Check these boxes**:

   **Protect matching branches**:
   - ✅ Require a pull request before merging
     - ✅ Require approvals: `2`
     - ✅ Dismiss stale pull request approvals when new commits are pushed
   
   - ✅ Require status checks to pass before merging
     - ✅ Require branches to be up to date before merging
     - (Status checks will appear after first CI/CD run)
   
   - ✅ Require conversation resolution before merging
   
   - ✅ Require signed commits (optional but recommended)
   
   - ✅ Include administrators
   
   - ✅ Restrict who can push to matching branches
     - Add yourself and other maintainers
   
   - ✅ Allow force pushes: ❌ DISABLED
   - ✅ Allow deletions: ❌ DISABLED

3. Click **Create** at the bottom

### Protect `develop` Branch

**Repeat the process with these settings**:

1. **Branch name pattern**: `develop`

2. **Settings**:
   - ✅ Require a pull request before merging
     - ✅ Require approvals: `1` (less strict than main)
   
   - ✅ Require status checks to pass before merging
   
   - ✅ Require conversation resolution before merging
   
   - ✅ Include administrators
   
   - ✅ Allow force pushes: ✅ ENABLED (allows rebasing)

3. Click **Create**

---

## 📝 Step 4: Add Repository Description & Topics

**URL**: https://github.com/yellowflowersorganics-star/cropwise

1. Click the ⚙️ **Settings** icon next to "About" (top right of main page)

2. **Description**:
   ```
   🌱 Enterprise IoT-Powered Farm Management Platform - AI-ready, scalable smart agriculture software with ESP32 + MQTT + React + Node.js
   ```

3. **Website** (if you have one):
   ```
   https://yourdomain.com
   ```

4. **Topics** (add all of these):
   ```
   agriculture
   iot
   farming
   esp32
   mqtt
   nodejs
   react
   postgresql
   smart-farming
   precision-agriculture
   mushroom-farming
   vertical-farming
   greenhouse
   agtech
   farm-management
   crop-monitoring
   environmental-monitoring
   typescript
   javascript
   ```

5. Check: ✅ **Include in the home page**

6. Click **Save changes**

---

## 💬 Step 5: Enable GitHub Discussions

**URL**: https://github.com/yellowflowersorganics-star/cropwise/settings

1. Scroll down to **Features** section

2. Check ✅ **Discussions**

3. Click **Set up discussions**

4. Default categories will be created:
   - 💬 General
   - 💡 Ideas
   - 🙏 Q&A
   - 🎉 Show and tell
   - 📣 Announcements

5. Click **Start discussion** to create welcome post (optional)

---

## 🏷️ Step 6: Add Issue Labels

**URL**: https://github.com/yellowflowersorganics-star/cropwise/labels

Click **New label** for each:

### Type Labels
| Name | Color | Description |
|------|-------|-------------|
| `bug` | `#d73a4a` | Something isn't working |
| `enhancement` | `#a2eeef` | New feature or request |
| `documentation` | `#0075ca` | Improvements to documentation |
| `question` | `#d876e3` | Further information requested |

### Priority Labels
| Name | Color | Description |
|------|-------|-------------|
| `priority: critical` | `#b60205` | Needs immediate attention |
| `priority: high` | `#d93f0b` | High priority |
| `priority: medium` | `#fbca04` | Medium priority |
| `priority: low` | `#0e8a16` | Low priority |

### Area Labels
| Name | Color | Description |
|------|-------|-------------|
| `area: backend` | `#5319e7` | Backend-related |
| `area: frontend` | `#1d76db` | Frontend-related |
| `area: iot` | `#c5def5` | IoT/Hardware-related |
| `area: docs` | `#d4c5f9` | Documentation |

---

## 🔐 Step 7: Add GitHub Actions Secrets

**URL**: https://github.com/yellowflowersorganics-star/cropwise/settings/secrets/actions

Click **New repository secret** for each:

### Essential Secrets (Add These Now)

| Secret Name | Example Value | Where to Get It |
|-------------|---------------|-----------------|
| `VITE_API_URL` | `http://localhost:3000` or `https://api.yourdomain.com` | Your API URL |

### AWS Secrets (Add When Deploying)

| Secret Name | Example Value | Where to Get It |
|-------------|---------------|-----------------|
| `AWS_ACCESS_KEY_ID` | `AKIAIOSFODNN7EXAMPLE` | AWS IAM Console |
| `AWS_SECRET_ACCESS_KEY` | `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY` | AWS IAM Console |
| `AWS_ACCESS_KEY_ID_PROD` | (separate key for prod) | AWS IAM Console |
| `AWS_SECRET_ACCESS_KEY_PROD` | (separate secret for prod) | AWS IAM Console |

### CloudFront Secrets (Add When Using S3+CloudFront)

| Secret Name | Example Value | Where to Get It |
|-------------|---------------|-----------------|
| `CLOUDFRONT_DEV_DISTRIBUTION_ID` | `E1234EXAMPLE` | AWS CloudFront Console |
| `CLOUDFRONT_PROD_DISTRIBUTION_ID` | `E5678EXAMPLE` | AWS CloudFront Console |

### Optional Secrets (Add As Needed)

| Secret Name | Example Value | Where to Get It |
|-------------|---------------|-----------------|
| `GOOGLE_CLIENT_ID` | `123456-abc.apps.googleusercontent.com` | Google Cloud Console |
| `GOOGLE_CLIENT_SECRET` | `GOCSPX-abc123` | Google Cloud Console |
| `TWILIO_ACCOUNT_SID` | `AC1234567890abcdef` | Twilio Console |
| `TWILIO_AUTH_TOKEN` | `abc123def456` | Twilio Console |
| `SLACK_WEBHOOK_URL` | `https://hooks.slack.com/services/T00/B00/XXX` | Slack App Settings |
| `DOCKERHUB_USERNAME` | `your-username` | Docker Hub |
| `DOCKERHUB_TOKEN` | `dckr_pat_abc123` | Docker Hub |

**How to create AWS Access Keys**:
1. Go to AWS IAM Console
2. Users → Your User → Security credentials
3. Create access key
4. Download and save securely

---

## 🚀 Step 8: Create First Release (v1.0.0)

**URL**: https://github.com/yellowflowersorganics-star/cropwise/releases/new

1. **Choose a tag**: `v1.0.0` (type and press "Create new tag")

2. **Target**: `main` (default)

3. **Release title**: `CropWise v1.0.0 - Initial Release`

4. **Description**: Copy this:

```markdown
# 🌱 CropWise v1.0.0 - Initial Release

## 🎉 First Production Release!

CropWise is now production-ready! This is the first stable release of our enterprise IoT-powered farm management platform.

### ✨ Key Features

#### **Core Farm Management**
- ✅ Complete farm and zone management
- ✅ Multi-stage growing recipes with environmental parameters
- ✅ Batch tracking from inoculation to harvest
- ✅ Harvest recording with per-flush yield and quality grading
- ✅ Real-time analytics and comprehensive reporting

#### **IoT Integration**
- ✅ Hierarchical ESP32 Master-Slave architecture
- ✅ MQTT broker integration with Raspberry Pi gateway
- ✅ Real-time environmental monitoring (temp, humidity, CO₂, light)
- ✅ Equipment control (fans, lights, irrigation, heaters)
- ✅ Recipe execution engine with manual stage approval

#### **Operations Management**
- ✅ Inventory tracking with low stock alerts
- ✅ Task management with assignments and recurrence
- ✅ Employee management with RBAC
- ✅ Labor tracking (clock in/out) and cost analysis
- ✅ Quality control with inspection and defect tracking
- ✅ SOP management with version control

#### **Advanced Features**
- ✅ Multi-channel notifications (Email, SMS, WhatsApp)
- ✅ Profitability analytics (ROI, profit margins, batch comparisons)
- ✅ Mobile-responsive UI with Tailwind CSS
- ✅ Google OAuth 2.0 authentication
- ✅ Comprehensive API documentation
- ✅ PostgreSQL with AI/ML readiness (pgvector, TimescaleDB)

### 📦 Tech Stack

**Backend**: Node.js 18, Express, PostgreSQL 15, Sequelize, Redis 7  
**Frontend**: React 18, Vite, Tailwind CSS, Zustand, Recharts  
**IoT**: ESP32, MQTT (Mosquitto), Raspberry Pi 5, ESP-NOW  
**Deployment**: Docker, AWS (RDS, S3, CloudFront, Elastic Beanstalk)

### 🚀 Quick Start

```bash
# Clone repository
git clone https://github.com/yellowflowersorganics-star/cropwise.git
cd cropwise

# Using Docker
docker-compose up -d

# Or manual setup
cd backend && npm install && npm start
cd ../frontend && npm install && npm run dev
```

### 📚 Documentation

- **[Installation Guide](https://github.com/yellowflowersorganics-star/cropwise/blob/main/docs/INSTALLATION.md)** - Get started in 5 minutes
- **[User Guide](https://github.com/yellowflowersorganics-star/cropwise/blob/main/docs/USER_GUIDE.md)** - 70-page complete manual
- **[Admin Guide](https://github.com/yellowflowersorganics-star/cropwise/blob/main/docs/ADMIN_GUIDE.md)** - System administration
- **[IoT Setup](https://github.com/yellowflowersorganics-star/cropwise/blob/main/docs/IOT_INTEGRATION_GUIDE.md)** - Hardware integration
- **[API Docs](https://github.com/yellowflowersorganics-star/cropwise/blob/main/docs/)** - Complete API reference

### 🐛 Found a Bug?

Please [report it](https://github.com/yellowflowersorganics-star/cropwise/issues/new?template=bug_report.md)

### 💡 Feature Request?

[Share your idea](https://github.com/yellowflowersorganics-star/cropwise/issues/new?template=feature_request.md)

### 💬 Questions?

- Check our [FAQ](https://github.com/yellowflowersorganics-star/cropwise/blob/main/docs/FAQ.md)
- [Start a discussion](https://github.com/yellowflowersorganics-star/cropwise/discussions)
- Email: support@cropwise.io

### 🙏 Contributors

Thank you to all contributors who made this release possible!

---

**Full Changelog**: https://github.com/yellowflowersorganics-star/cropwise/blob/main/CHANGELOG.md

**License**: MIT - Free for commercial use
```

5. Check: ✅ **Set as the latest release**

6. Click **Publish release**

---

## 🧪 Step 9: Test CI/CD Pipeline

Let's verify GitHub Actions works:

```bash
# Create a test feature branch
git checkout develop
git checkout -b feature/test-github-actions

# Make a small change
echo "# GitHub Actions Test" >> README.md

# Commit and push
git add README.md
git commit -m "test: verify GitHub Actions CI/CD pipeline"
git push origin feature/test-github-actions
```

Then:
1. Go to: https://github.com/yellowflowersorganics-star/cropwise/pulls
2. Click **New pull request**
3. Base: `develop` ← Compare: `feature/test-github-actions`
4. Click **Create pull request**
5. Go to **Actions** tab and watch the pipeline run
6. If successful, merge the PR (or close it if just testing)

---

## ✅ Step 10: Verify Everything Works

### Check These Pages:

- [ ] **Main Page**: https://github.com/yellowflowersorganics-star/cropwise
  - Description and topics visible
  - README displays correctly
  - Badges show proper colors

- [ ] **Actions**: https://github.com/yellowflowersorganics-star/cropwise/actions
  - Workflows visible
  - Can view workflow files

- [ ] **Issues**: https://github.com/yellowflowersorganics-star/cropwise/issues
  - Templates work when creating new issue

- [ ] **Pull Requests**: Try creating one
  - Template appears
  - Branch protection works

- [ ] **Discussions**: https://github.com/yellowflowersorganics-star/cropwise/discussions
  - Categories visible
  - Can create new discussion

- [ ] **Releases**: https://github.com/yellowflowersorganics-star/cropwise/releases
  - v1.0.0 release visible

---

## 📊 Final Status Check

### Repository Configuration
- [x] `main` branch exists ✓
- [x] `develop` branch created ✓
- [ ] `develop` set as default branch
- [ ] Branch protection rules configured
- [ ] Repository description added
- [ ] Topics added
- [ ] Discussions enabled

### GitHub Actions
- [x] CI/CD workflows committed ✓
- [ ] GitHub secrets added
- [ ] First workflow run successful

### Community
- [x] Issue templates ✓
- [x] PR template ✓
- [ ] Labels created
- [ ] First release created
- [ ] Security policy added (optional)

### Documentation
- [x] README.md complete ✓
- [x] CHANGELOG.md ✓
- [x] CONTRIBUTING.md ✓
- [x] All guides in /docs ✓

---

## 🎯 Priority Order

**Do Today** (15 minutes):
1. ✅ Set `develop` as default branch
2. ✅ Add repository description & topics
3. ✅ Enable discussions
4. ✅ Add `VITE_API_URL` secret

**Do This Week**:
5. ✅ Configure branch protection
6. ✅ Add labels
7. ✅ Create v1.0.0 release
8. ✅ Test CI/CD pipeline

**Do When Deploying**:
9. ✅ Add AWS secrets
10. ✅ Add other API keys as needed

---

## 🆘 Need Help?

- **Full Guide**: See `docs/GITHUB_SETUP_GUIDE.md`
- **Git Workflow**: See `docs/GIT_WORKFLOW.md`
- **Questions**: Create a discussion or open an issue

---

## ✨ Quick Commands Reference

```bash
# Switch between branches
git checkout main
git checkout develop

# Create feature branch (from develop)
git checkout develop
git checkout -b feature/my-feature

# Push feature and create PR
git push origin feature/my-feature
# Then create PR on GitHub

# Update your branch with latest develop
git fetch origin
git rebase origin/develop

# Delete merged branch
git branch -d feature/my-feature
git push origin --delete feature/my-feature
```

---

**🎉 You're all set! Your repository is ready for professional development!**

**Next**: Follow the steps above to complete the GitHub setup, then you're ready to deploy to AWS! 🚀

