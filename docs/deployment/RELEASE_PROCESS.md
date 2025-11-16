# 🚀 Release Process

**Complete guide to releasing new versions of CropWise**

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Release Types](#-release-types)
- [Versioning Strategy](#-versioning-strategy)
- [Release Schedule](#-release-schedule)
- [Release Checklist](#-release-checklist)
- [Step-by-Step Process](#-step-by-step-process)
- [Hotfix Process](#-hotfix-process)
- [Post-Release](#-post-release)
- [Rollback Procedure](#-rollback-procedure)

---

## 🎯 Overview

CropWise follows [Semantic Versioning](https://semver.org/) (SemVer) and uses a structured release process to ensure quality and stability in production.

**Release Cycle:** 2-4 weeks  
**Release Day:** Monday (for major/minor), Any day (for patches)  
**Code Freeze:** 3 days before release  
**Testing Period:** 2-3 days

---

## 📦 Release Types

### Major Release (X.0.0)

**When:** Breaking changes or significant new features  
**Frequency:** 6-12 months  
**Examples:** v2.0.0, v3.0.0

**Requires:**
- Breaking API changes documented
- Migration guide provided
- Extended testing period (1 week)
- Stakeholder approval

**Example Changes:**
- Database schema redesign
- API endpoint restructure
- Removal of deprecated features
- Major architecture changes

### Minor Release (1.X.0)

**When:** New features without breaking changes  
**Frequency:** 2-4 weeks  
**Examples:** v1.1.0, v1.2.0

**Requires:**
- Feature complete and tested
- Documentation updated
- Standard testing period (2-3 days)

**Example Changes:**
- New API endpoints
- New dashboard features
- Performance improvements
- Enhanced reporting

### Patch Release (1.0.X)

**When:** Bug fixes and minor improvements  
**Frequency:** As needed (weekly if required)  
**Examples:** v1.0.1, v1.0.2

**Requires:**
- Critical bugs fixed
- Regression testing passed
- Quick testing (1 day)

**Example Changes:**
- Bug fixes
- Security patches
- Performance tweaks
- Documentation fixes

---

## 🔢 Versioning Strategy

### Semantic Versioning (SemVer)

```
MAJOR.MINOR.PATCH

Example: v1.5.3
         │ │ └─ Patch (bug fixes)
         │ └─── Minor (new features)
         └───── Major (breaking changes)
```

### Version Bumping Rules

```javascript
// Current version: v1.5.3

// Bug fix → Patch
v1.5.3 → v1.5.4

// New feature → Minor
v1.5.3 → v1.6.0

// Breaking change → Major
v1.5.3 → v2.0.0
```

### Pre-release Versions

```
v1.5.0-alpha.1    # Alpha release (internal testing)
v1.5.0-beta.1     # Beta release (limited users)
v1.5.0-rc.1       # Release Candidate (final testing)
v1.5.0            # Stable release
```

### Tag Format

```bash
# Stable releases
git tag -a v1.5.0 -m "Release v1.5.0"

# Pre-releases
git tag -a v1.5.0-beta.1 -m "Beta release v1.5.0-beta.1"
```

---

## 📅 Release Schedule

### Typical Release Timeline (Minor Release)

```
Week 1-3: Development Phase
├─ Day 1-15: Feature development
├─ Day 16-18: Code freeze
├─ Day 19-21: Testing & bug fixes
└─ Day 22: Release day

Week 4: Stabilization
├─ Day 1-3: Monitor production
├─ Day 4-7: Patch releases if needed
└─ Start next release cycle
```

### Release Calendar Example

| Date | Version | Type | Features |
|------|---------|------|----------|
| Jan 15, 2025 | v1.1.0 | Minor | Batch QR codes, Analytics dashboard |
| Jan 22, 2025 | v1.1.1 | Patch | Fix harvest validation bug |
| Feb 5, 2025 | v1.2.0 | Minor | Task automation, Mobile app v1 |
| Feb 12, 2025 | v1.2.1 | Patch | Performance optimizations |
| Mar 1, 2025 | v2.0.0 | Major | New API v2, Database redesign |

---

## ✅ Release Checklist

### Pre-Release (Code Freeze - 3 days before)

- [ ] **Code Complete**
  - [ ] All features merged to `develop` branch
  - [ ] All PRs reviewed and approved
  - [ ] No pending critical bugs
  
- [ ] **Create Release Branch**
  - [ ] Branch created: `release/v1.x.0`
  - [ ] Branch pushed to GitHub
  - [ ] Team notified of code freeze

- [ ] **Update Version Numbers**
  - [ ] Update `backend/package.json` version
  - [ ] Update `frontend/package.json` version
  - [ ] Update `README.md` version badge
  
- [ ] **Documentation**
  - [ ] CHANGELOG.md updated with all changes
  - [ ] API documentation updated
  - [ ] User guide updated for new features
  - [ ] Migration guide created (if needed)

### Testing Phase (Days 1-3)

- [ ] **Automated Testing**
  - [ ] All unit tests pass
  - [ ] All integration tests pass
  - [ ] CI/CD pipeline passes
  - [ ] Security scan complete
  
- [ ] **Manual Testing**
  - [ ] Smoke testing complete
  - [ ] Regression testing complete
  - [ ] Cross-browser testing (Chrome, Firefox, Safari)
  - [ ] Mobile testing (iOS, Android)
  - [ ] Performance testing
  - [ ] Load testing (if major release)
  
- [ ] **Deployment Testing**
  - [ ] Test deployment to staging environment
  - [ ] Database migrations tested
  - [ ] Rollback procedure tested
  - [ ] Health checks verified

### Release Day

- [ ] **Final Preparations**
  - [ ] All tests passing
  - [ ] No critical bugs
  - [ ] Team available for monitoring
  - [ ] Rollback plan ready
  
- [ ] **Deployment**
  - [ ] Merge release branch to `main`
  - [ ] Create and push version tag
  - [ ] GitHub Actions triggers deployment
  - [ ] Monitor deployment progress
  
- [ ] **Verification**
  - [ ] Production health checks pass
  - [ ] Smoke tests on production
  - [ ] Key features verified
  - [ ] Performance metrics normal
  
- [ ] **Communication**
  - [ ] Release notes published
  - [ ] Team notified on Slack
  - [ ] Users notified (email/in-app)
  - [ ] Social media announcement (if major release)

### Post-Release (Days 1-3)

- [ ] **Merge Back**
  - [ ] Merge release branch to `develop`
  - [ ] Delete release branch
  - [ ] Update project board
  
- [ ] **Monitoring**
  - [ ] Monitor error rates
  - [ ] Monitor performance metrics
  - [ ] Monitor user feedback
  - [ ] Check for critical issues
  
- [ ] **Documentation**
  - [ ] Update version in all docs
  - [ ] Archive release notes
  - [ ] Update roadmap

---

## 📋 Step-by-Step Process

### 1. Code Freeze (Monday, Week 4)

```bash
# Create release branch from develop
git checkout develop
git pull origin develop
git checkout -b release/v1.5.0

# Update version numbers
cd backend
npm version 1.5.0 --no-git-tag-version

cd ../frontend
npm version 1.5.0 --no-git-tag-version

# Commit version bump
git add .
git commit -m "chore: bump version to 1.5.0"

# Push release branch
git push origin release/v1.5.0
```

### 2. Update CHANGELOG.md

```markdown
## [1.5.0] - 2025-01-15

### Added
- Batch QR code generation and scanning
- Advanced analytics dashboard with custom date ranges
- Export reports to PDF/CSV
- WhatsApp notifications for critical alerts

### Changed
- Improved batch list performance by 40%
- Updated UI with new color scheme
- Enhanced mobile responsiveness

### Fixed
- Fixed harvest weight validation bug
- Resolved timezone issues in reports
- Fixed memory leak in WebSocket connections

### Security
- Updated dependencies with security patches
- Implemented rate limiting on sensitive endpoints
```

### 3. Testing (Tuesday-Thursday)

```bash
# Deploy to staging
git push origin release/v1.5.0
# CI/CD automatically deploys to staging

# Run automated tests
npm test

# Manual testing checklist
# - Test all new features
# - Test critical user flows
# - Test on multiple browsers
# - Test on mobile devices

# Fix any bugs found
git checkout release/v1.5.0
# Make fixes...
git commit -m "fix: resolve issue found in testing"
git push origin release/v1.5.0
```

### 4. Final Review (Thursday Evening)

```bash
# Team review meeting
# - Review test results
# - Review open issues
# - Make go/no-go decision

# If approved, schedule Friday release
# If not approved, continue fixing and retest
```

### 5. Release Day (Friday Morning)

```bash
# 1. Merge to main
git checkout main
git pull origin main
git merge --no-ff release/v1.5.0 -m "Release v1.5.0"
git push origin main

# 2. Create and push tag
git tag -a v1.5.0 -m "Release v1.5.0

New Features:
- Batch QR codes
- Analytics dashboard
- PDF/CSV exports
- WhatsApp notifications

See CHANGELOG.md for complete list of changes."

git push origin v1.5.0

# 3. GitHub Actions automatically:
#    - Runs CI/CD pipeline
#    - Builds Docker images
#    - Deploys to production
#    - Creates GitHub Release

# 4. Monitor deployment
# Watch GitHub Actions workflow
# Monitor CloudWatch logs
# Check health endpoints
```

### 6. Create GitHub Release

Go to GitHub → Releases → Draft new release

**Tag:** `v1.5.0`  
**Title:** `CropWise v1.5.0 - Analytics & QR Codes`

**Description:**

```markdown
## 🚀 CropWise v1.5.0

Released on January 15, 2025

### 🎉 New Features

#### Batch QR Codes
- Generate QR codes for each batch
- Scan QR codes with mobile device
- Quick access to batch details

#### Advanced Analytics Dashboard
- Custom date range selection
- Export to PDF/CSV
- Interactive charts with drill-down
- Performance insights

#### Enhanced Notifications
- WhatsApp notifications for critical alerts
- Customizable notification preferences
- Multi-channel delivery (email, SMS, WhatsApp)

### 🔧 Improvements

- 40% faster batch list loading
- Improved mobile responsiveness
- Updated UI design
- Better error messages

### 🐛 Bug Fixes

- Fixed harvest weight validation
- Resolved timezone issues in reports
- Fixed WebSocket memory leak

### 🔐 Security

- Updated all dependencies
- Rate limiting on sensitive endpoints
- Enhanced input validation

### 📦 Installation

```bash
docker pull cropwise/backend:v1.5.0
docker pull cropwise/frontend:v1.5.0
```

### 📚 Documentation

- [User Guide](https://docs.cropwise.io/v1.5.0)
- [API Documentation](https://api.cropwise.io/docs)
- [Migration Guide](https://docs.cropwise.io/v1.5.0/migration)

### 🙏 Contributors

Thank you to all contributors who made this release possible!

---

**Full Changelog**: https://github.com/your-org/cropwise/compare/v1.4.0...v1.5.0
```

### 7. Merge Back to Develop

```bash
# Merge release changes back to develop
git checkout develop
git pull origin develop
git merge --no-ff release/v1.5.0 -m "Merge release v1.5.0 to develop"
git push origin develop

# Delete release branch
git branch -d release/v1.5.0
git push origin --delete release/v1.5.0
```

### 8. Post-Release Communication

**Slack Announcement:**

```
🚀 CropWise v1.5.0 is now LIVE! 🎉

✨ New Features:
• Batch QR codes
• Advanced analytics
• PDF/CSV exports
• WhatsApp notifications

🔧 Improvements:
• 40% faster performance
• Better mobile experience

📖 Full release notes: https://github.com/your-org/cropwise/releases/tag/v1.5.0

Great work team! 👏
```

**Email to Users:**

```
Subject: CropWise v1.5.0 Released - New Analytics & QR Codes!

Hi there,

We're excited to announce the release of CropWise v1.5.0!

[Include key features, screenshots, and links]

Upgrade today to access these new features!

Best regards,
The CropWise Team
```

---

## 🔥 Hotfix Process

Hotfixes are for critical production bugs that require immediate attention.

### When to Create a Hotfix

- **Critical bugs** affecting all users
- **Security vulnerabilities**
- **Data loss issues**
- **Complete service outage**

### Hotfix Workflow

```bash
# 1. Create hotfix branch from main
git checkout main
git pull origin main
git checkout -b hotfix/v1.5.1

# 2. Fix the critical issue
# Make necessary changes...

# 3. Update version
cd backend
npm version patch --no-git-tag-version

# 4. Update CHANGELOG
# Add hotfix entry

# 5. Commit fix
git add .
git commit -m "fix: critical bug in harvest calculation"

# 6. Test fix thoroughly
npm test

# 7. Merge to main
git checkout main
git merge --no-ff hotfix/v1.5.1
git push origin main

# 8. Tag hotfix
git tag -a v1.5.1 -m "Hotfix v1.5.1: Fix harvest calculation bug"
git push origin v1.5.1

# 9. Merge to develop
git checkout develop
git merge --no-ff hotfix/v1.5.1
git push origin develop

# 10. Delete hotfix branch
git branch -d hotfix/v1.5.1
```

**Timeline:** 1-4 hours (depending on urgency)

---

## 🎉 Post-Release

### Monitor Production (First 24 Hours)

```bash
# Check error rates
aws cloudwatch get-metric-statistics \
  --metric-name ErrorCount \
  --namespace CropWise \
  --start-time 2025-01-15T00:00:00Z \
  --end-time 2025-01-16T00:00:00Z \
  --period 3600 \
  --statistics Sum

# Monitor logs
aws logs tail /ecs/cropwise-backend --follow

# Check performance
# View CloudWatch dashboards
# Monitor response times
# Check database performance
```

### Gather Feedback

- Monitor #general Slack channel
- Check support tickets
- Review app analytics
- Read user feedback

### Plan Next Release

- Create milestone for next version
- Prioritize features
- Update roadmap
- Schedule next release

---

## ⏪ Rollback Procedure

If critical issues are discovered after release:

### Quick Rollback (< 1 hour)

```bash
# 1. Identify previous stable version
PREVIOUS_VERSION="v1.4.0"

# 2. Revert main branch
git checkout main
git revert -m 1 HEAD  # Revert merge commit
git push origin main

# 3. Redeploy previous version
git checkout ${PREVIOUS_VERSION}
./scripts/aws-deploy.sh

# 4. Notify team
# Post in Slack
# Update status page
```

### Database Rollback

```bash
# If database migrations were applied
cd backend

# Check migration status
npx sequelize-cli db:migrate:status

# Undo last migration
npx sequelize-cli db:migrate:undo

# Verify database state
psql -U cropwise -d cropwise -c "\d"
```

### Communication During Rollback

```
🚨 PRODUCTION ROLLBACK IN PROGRESS

We've discovered a critical issue with v1.5.0 and are rolling back to v1.4.0.

Timeline:
- 10:00 AM: Issue detected
- 10:15 AM: Rollback initiated
- 10:30 AM: Rollback complete (expected)

Users may experience brief service interruption.

We'll provide updates as we resolve the issue.
```

---

## 📊 Release Metrics

Track these metrics for each release:

### Development Metrics

- **Development time**: 2-3 weeks
- **Number of commits**: ~50-100
- **Number of PRs**: ~10-20
- **Code review time**: 2-3 days
- **Testing time**: 2-3 days

### Quality Metrics

- **Test coverage**: > 80%
- **Bugs found in testing**: < 10
- **Critical bugs**: 0
- **Security vulnerabilities**: 0

### Deployment Metrics

- **Deployment time**: 10-15 minutes
- **Downtime**: 0 seconds (zero-downtime)
- **Rollback time**: < 5 minutes
- **Time to first production issue**: > 24 hours (goal)

### User Impact Metrics

- **User satisfaction**: > 4.5/5
- **Feature adoption**: > 60% in first week
- **Support tickets**: < 10 in first week
- **Performance improvement**: Measurable

---

## 🔖 Release Templates

### Release Branch Name

```
release/vMAJOR.MINOR.PATCH
release/v1.5.0
```

### Commit Messages

```
chore: bump version to 1.5.0
docs: update CHANGELOG for v1.5.0
release: v1.5.0
```

### Tag Message

```
Release v1.5.0

New Features:
- Feature 1
- Feature 2

Improvements:
- Improvement 1

Bug Fixes:
- Fix 1

See CHANGELOG.md for complete details.
```

---

## 📞 Release Team

### Roles & Responsibilities

**Release Manager** (rotates)
- Coordinates release process
- Makes go/no-go decisions
- Communicates with stakeholders

**Tech Lead**
- Reviews code quality
- Approves technical changes
- Resolves technical blockers

**QA Lead**
- Oversees testing
- Signs off on release
- Tracks bugs

**DevOps Engineer**
- Handles deployment
- Monitors production
- Manages rollbacks

**Product Manager**
- Prioritizes features
- Reviews release notes
- Communicates with users

---

## 📚 Additional Resources

- [Semantic Versioning Spec](https://semver.org/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Git Flow](https://nvie.com/posts/a-successful-git-branching-model/)
- [Release Engineering Best Practices](https://www.atlassian.com/continuous-delivery/release-management)

---

**Last Updated**: November 2024  
**Version**: 1.0.0  
**Next Review**: March 2025

Happy Releasing! 🚀

