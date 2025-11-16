# 🔀 CropWise - Git Workflow & Branching Strategy

## 📋 Table of Contents

1. [Branching Strategy](#branching-strategy)
2. [Branch Types](#branch-types)
3. [Workflow](#workflow)
4. [Pull Request Process](#pull-request-process)
5. [Release Strategy](#release-strategy)
6. [Versioning](#versioning)
7. [CI/CD Pipeline](#cicd-pipeline)
8. [Best Practices](#best-practices)

---

## 🌲 Branching Strategy

We follow **GitFlow** - a robust branching model perfect for production applications.

```
┌─────────────────────────────────────────────────────────────┐
│                     Branch Structure                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  main (production)     ═══════●═══════●═══════●            │
│                              ╱         ╱                    │
│  release/*            ─────●─────────●                      │
│                          ╱   ╲     ╱   ╲                    │
│  develop              ●═══════●═══●═════●                   │
│                      ╱ ╲     ╱ ╲   ╲                        │
│  feature/*        ──●   ●───●   ●   ●                       │
│                      ╲ ╱         ╲ ╱                        │
│  hotfix/*             ●           ●                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 📂 Branch Types

### 1. **`main`** (Production Branch)
- **Purpose**: Production-ready code
- **Protection**: Protected, no direct pushes
- **Deployment**: Auto-deploys to AWS Production
- **Merges from**: `release/*` or `hotfix/*` only
- **Lifetime**: Permanent

**Rules:**
- ✅ Always stable and deployable
- ✅ Every commit is a production release
- ✅ Tagged with version numbers (v1.0.0, v1.1.0)
- ❌ Never commit directly
- ❌ Never merge feature branches directly

### 2. **`develop`** (Development Branch)
- **Purpose**: Integration branch for features
- **Protection**: Protected, requires PR approval
- **Deployment**: Auto-deploys to AWS Development
- **Merges from**: `feature/*`, `release/*`, `hotfix/*`
- **Lifetime**: Permanent

**Rules:**
- ✅ Latest development changes
- ✅ Should always build successfully
- ✅ Integration point for all features
- ❌ May contain unstable features

### 3. **`feature/*`** (Feature Branches)
- **Purpose**: Develop new features
- **Naming**: `feature/ISSUE-123-add-harvest-tracking`
- **Branch from**: `develop`
- **Merge to**: `develop`
- **Lifetime**: Temporary (deleted after merge)

**Examples:**
```bash
feature/add-iot-mqtt-integration
feature/FARM-456-implement-quality-control
feature/update-api-documentation
```

**Rules:**
- ✅ One feature per branch
- ✅ Regularly sync with `develop`
- ✅ Squash commits before merging
- ❌ Don't let them become stale (> 2 weeks)

### 4. **`release/*`** (Release Branches)
- **Purpose**: Prepare for production release
- **Naming**: `release/v1.2.0`
- **Branch from**: `develop`
- **Merge to**: `main` AND `develop`
- **Lifetime**: Temporary (deleted after merge)

**Rules:**
- ✅ Only bug fixes, no new features
- ✅ Update version numbers
- ✅ Update CHANGELOG.md
- ✅ Final testing before production

### 5. **`hotfix/*`** (Hotfix Branches)
- **Purpose**: Emergency production fixes
- **Naming**: `hotfix/v1.2.1-fix-authentication-bug`
- **Branch from**: `main`
- **Merge to**: `main` AND `develop`
- **Lifetime**: Temporary (deleted after merge)

**Rules:**
- ✅ Critical bugs only
- ✅ Fast-tracked review process
- ✅ Increment patch version
- ✅ Must not break existing features

### 6. **`bugfix/*`** (Bug Fix Branches)
- **Purpose**: Fix non-critical bugs
- **Naming**: `bugfix/ISSUE-789-fix-chart-rendering`
- **Branch from**: `develop`
- **Merge to**: `develop`
- **Lifetime**: Temporary (deleted after merge)

---

## 🔄 Workflow

### **Scenario 1: Developing a New Feature**

```bash
# 1. Create feature branch from develop
git checkout develop
git pull origin develop
git checkout -b feature/add-employee-dashboard

# 2. Make your changes
git add .
git commit -m "feat: add employee dashboard with charts"

# 3. Keep feature branch up to date
git fetch origin
git rebase origin/develop

# 4. Push feature branch
git push origin feature/add-employee-dashboard

# 5. Create Pull Request on GitHub
# - Base: develop
# - Compare: feature/add-employee-dashboard
# - Fill PR template
# - Request reviews

# 6. After approval, squash and merge
# Branch is automatically deleted
```

### **Scenario 2: Creating a Release**

```bash
# 1. Create release branch from develop
git checkout develop
git pull origin develop
git checkout -b release/v1.3.0

# 2. Update version numbers
# - backend/package.json: "version": "1.3.0"
# - frontend/package.json: "version": "1.3.0"
# Update CHANGELOG.md

git add .
git commit -m "chore: bump version to 1.3.0"

# 3. Final testing and bug fixes only
# (If bugs found, fix them in this branch)

# 4. Merge to main (creates production release)
git checkout main
git pull origin main
git merge --no-ff release/v1.3.0
git tag -a v1.3.0 -m "Release version 1.3.0"
git push origin main --tags

# 5. Merge back to develop
git checkout develop
git merge --no-ff release/v1.3.0
git push origin develop

# 6. Delete release branch
git branch -d release/v1.3.0
git push origin --delete release/v1.3.0
```

### **Scenario 3: Emergency Hotfix**

```bash
# 1. Create hotfix branch from main
git checkout main
git pull origin main
git checkout -b hotfix/v1.3.1-fix-authentication

# 2. Fix the critical bug
git add .
git commit -m "fix: resolve JWT token expiration issue"

# 3. Merge to main
git checkout main
git merge --no-ff hotfix/v1.3.1-fix-authentication
git tag -a v1.3.1 -m "Hotfix version 1.3.1"
git push origin main --tags

# 4. Merge to develop
git checkout develop
git merge --no-ff hotfix/v1.3.1-fix-authentication
git push origin develop

# 5. Delete hotfix branch
git branch -d hotfix/v1.3.1-fix-authentication
git push origin --delete hotfix/v1.3.1-fix-authentication
```

---

## 🔍 Pull Request Process

### **PR Template** (See `.github/PULL_REQUEST_TEMPLATE.md`)

Every PR must include:

1. **Description**: What changes were made and why
2. **Type of Change**: Feature, Bug Fix, Hotfix, Documentation
3. **Testing**: How was this tested?
4. **Screenshots**: UI changes (if applicable)
5. **Checklist**: Code quality, tests, documentation

### **PR Review Checklist**

**Code Quality:**
- [ ] Code follows project style guide
- [ ] No console.log() or debug statements
- [ ] Proper error handling
- [ ] No hardcoded values
- [ ] Comments for complex logic

**Testing:**
- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Manual testing completed
- [ ] No breaking changes (or documented)

**Documentation:**
- [ ] README updated (if needed)
- [ ] API docs updated (if needed)
- [ ] CHANGELOG.md updated

**Security:**
- [ ] No secrets or API keys committed
- [ ] Input validation implemented
- [ ] SQL injection prevention
- [ ] XSS prevention

### **Approval Requirements**

| Branch | Approvals Required | Who Can Approve |
|--------|-------------------|-----------------|
| `main` | 2 reviewers | Senior Developers, Team Lead |
| `develop` | 1 reviewer | Any Developer |
| `feature/*` | 1 reviewer | Any Developer |
| `hotfix/*` | 1 reviewer (fast-track) | Senior Developers |

---

## 🚀 Release Strategy

### **Release Types**

#### **1. Major Release (v2.0.0)**
- **When**: Breaking changes, major features
- **Frequency**: Every 6-12 months
- **Process**: Full QA cycle, staging testing, rollback plan
- **Announcement**: Blog post, email to users, migration guide

#### **2. Minor Release (v1.3.0)**
- **When**: New features, enhancements
- **Frequency**: Every 2-4 weeks
- **Process**: Staging testing, automated tests
- **Announcement**: Release notes, changelog

#### **3. Patch Release (v1.3.1)**
- **When**: Bug fixes, small improvements
- **Frequency**: As needed (hotfixes)
- **Process**: Automated tests, quick deployment
- **Announcement**: Changelog update

### **Release Checklist**

**Pre-Release:**
- [ ] All features merged to `develop`
- [ ] All tests passing (unit, integration, E2E)
- [ ] Code review completed
- [ ] Documentation updated
- [ ] CHANGELOG.md updated
- [ ] Version numbers bumped

**Release:**
- [ ] Create `release/*` branch
- [ ] Deploy to staging environment
- [ ] QA testing on staging
- [ ] Fix any bugs found
- [ ] Merge to `main`
- [ ] Tag release with version
- [ ] Deploy to production

**Post-Release:**
- [ ] Monitor production logs
- [ ] Check error rates
- [ ] Verify all services running
- [ ] Announce release
- [ ] Merge back to `develop`
- [ ] Plan next sprint

---

## 🏷️ Versioning (Semantic Versioning)

We follow **SemVer 2.0.0**: `MAJOR.MINOR.PATCH`

```
Version Format: X.Y.Z

X = MAJOR version (breaking changes)
Y = MINOR version (new features, backward compatible)
Z = PATCH version (bug fixes, backward compatible)
```

### **Examples:**

| Version | Type | Description |
|---------|------|-------------|
| **v1.0.0** | Initial | First production release |
| **v1.1.0** | Minor | Added employee management |
| **v1.1.1** | Patch | Fixed dashboard chart bug |
| **v1.2.0** | Minor | Added IoT integration |
| **v2.0.0** | Major | Redesigned database schema (breaking) |

### **Pre-Release Versions:**

```
v1.3.0-alpha.1    # Alpha testing
v1.3.0-beta.2     # Beta testing
v1.3.0-rc.1       # Release candidate
```

### **Version Bumping:**

```bash
# Patch (1.2.3 → 1.2.4)
npm version patch

# Minor (1.2.3 → 1.3.0)
npm version minor

# Major (1.2.3 → 2.0.0)
npm version major

# Pre-release (1.2.3 → 1.2.4-alpha.0)
npm version prerelease --preid=alpha
```

---

## ⚙️ CI/CD Pipeline

### **Automated Workflows**

```
┌─────────────────────────────────────────────────────────┐
│                  CI/CD Pipeline Flow                    │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Push to feature/*                                      │
│     └─→ Run Linting                                    │
│         └─→ Run Unit Tests                             │
│             └─→ Build Check                            │
│                 └─→ Comment Status on PR               │
│                                                         │
│  Merge to develop                                       │
│     └─→ Run All Tests                                  │
│         └─→ Build Docker Images                        │
│             └─→ Deploy to DEV Environment              │
│                 └─→ Run Integration Tests              │
│                     └─→ Notify Team (Slack)            │
│                                                         │
│  Merge to main (via release/*)                         │
│     └─→ Run Full Test Suite                            │
│         └─→ Build Production Images                    │
│             └─→ Deploy to Staging                      │
│                 └─→ Run E2E Tests                      │
│                     └─→ Manual Approval Required       │
│                         └─→ Deploy to Production       │
│                             └─→ Create GitHub Release  │
│                                 └─→ Notify Team        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### **Environment Strategy**

| Environment | Branch | URL | Purpose |
|-------------|--------|-----|---------|
| **Development** | `develop` | `dev.cropwise.io` | Latest features, unstable |
| **Staging** | `release/*` | `staging.cropwise.io` | Pre-production testing |
| **Production** | `main` | `www.cropwise.io` | Live application |

---

## ✅ Best Practices

### **Commit Messages**

Follow **Conventional Commits** format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Formatting, missing semicolons
- `refactor`: Code restructuring
- `perf`: Performance improvements
- `test`: Adding tests
- `chore`: Maintenance tasks
- `ci`: CI/CD changes

**Examples:**
```bash
feat(auth): add Google OAuth integration

Implemented Google OAuth 2.0 authentication flow with JWT token generation.
Users can now sign in using their Google accounts.

Closes #123

---

fix(dashboard): resolve chart rendering issue

Fixed bug where yield charts were not displaying correctly
when data contained null values.

Fixes #456

---

docs(api): update API documentation for harvest endpoints

Added examples for POST /api/harvests endpoint with
request/response samples.
```

### **Branch Naming**

```bash
# Feature branches
feature/add-employee-management
feature/FARM-123-implement-task-scheduler

# Bug fix branches
bugfix/fix-dashboard-charts
bugfix/ISSUE-456-resolve-login-error

# Hotfix branches
hotfix/v1.2.1-fix-critical-security-bug

# Release branches
release/v1.3.0
```

### **Code Review Guidelines**

**For Authors:**
- ✅ Keep PRs small (<400 lines)
- ✅ Write clear PR descriptions
- ✅ Add screenshots for UI changes
- ✅ Respond to review comments promptly
- ✅ Test your changes thoroughly

**For Reviewers:**
- ✅ Review within 24 hours
- ✅ Be constructive, not critical
- ✅ Ask questions, don't assume
- ✅ Approve only if you understand the code
- ✅ Test locally if possible

### **Merge Strategies**

| Branch Type | Merge Strategy | Reason |
|-------------|---------------|--------|
| `feature/*` → `develop` | **Squash and merge** | Clean history, one commit per feature |
| `develop` → `release/*` | **Merge commit** | Preserve feature branches |
| `release/*` → `main` | **Merge commit** | Mark release points |
| `hotfix/*` → `main` | **Merge commit** | Track emergency fixes |

---

## 🔒 Branch Protection Rules

### **`main` Branch Protection**

- ✅ Require pull request before merging
- ✅ Require 2 approvals
- ✅ Require status checks to pass
- ✅ Require branches to be up to date
- ✅ Require conversation resolution
- ✅ Do not allow bypassing (including admins)
- ✅ Restrict who can push (Team Leads only)

### **`develop` Branch Protection**

- ✅ Require pull request before merging
- ✅ Require 1 approval
- ✅ Require status checks to pass
- ✅ Allow force pushes (for rebasing)

---

## 📊 Metrics & Monitoring

Track these metrics for healthy Git workflow:

- **PR Merge Time**: Target < 24 hours
- **Build Success Rate**: Target > 95%
- **Code Coverage**: Target > 80%
- **Deployment Frequency**: Target 2-4 times/month
- **Mean Time to Recovery**: Target < 1 hour
- **Change Failure Rate**: Target < 15%

---

## 🚨 Emergency Procedures

### **Rollback Production**

```bash
# 1. Identify the last good version
git tag --sort=-version:refname

# 2. Create hotfix branch from that tag
git checkout -b hotfix/rollback-to-v1.2.3 v1.2.3

# 3. Deploy immediately
git checkout main
git merge hotfix/rollback-to-v1.2.3
git push origin main

# 4. Deploy to production (triggers CI/CD)
```

### **Revert a Bad Commit**

```bash
# Find the commit hash
git log --oneline

# Revert the commit (creates a new commit)
git revert <commit-hash>

# Push the revert
git push origin main
```

---

## 📚 Resources

- **Git Documentation**: https://git-scm.com/doc
- **GitFlow Cheatsheet**: https://danielkummer.github.io/git-flow-cheatsheet/
- **Conventional Commits**: https://www.conventionalcommits.org/
- **Semantic Versioning**: https://semver.org/
- **GitHub Flow**: https://guides.github.com/introduction/flow/

---

## ✅ Quick Reference

```bash
# Clone repository
git clone https://github.com/yellowflowersorganics-star/cropwise.git

# Set up Git Flow (one-time)
git flow init

# Start new feature
git flow feature start add-dashboard

# Finish feature (merges to develop)
git flow feature finish add-dashboard

# Start release
git flow release start v1.3.0

# Finish release (merges to main + develop, creates tag)
git flow release finish v1.3.0

# Start hotfix
git flow hotfix start v1.3.1-fix-bug

# Finish hotfix
git flow hotfix finish v1.3.1-fix-bug
```

---

**Questions? Need clarification? Check our [Contributing Guide](CONTRIBUTING.md)** 🚀

