# 🏷️ GitHub Issues to Create

**Purpose:** Complete issue-based workflow setup  
**Created:** November 2024

Copy each issue below and create it in GitHub. Once created, no direct commits allowed!

---

## 🔴 PRIORITY 1: Rebranding Verification

### Issue #1: Verify CropWise Rebranding

```markdown
**Title:** [TASK] Verify CropWise rebranding across entire codebase

**Labels:** task, testing, priority-high

**Description:**

## 📋 Task Description
Verify that the automated rebranding from SmartCrop to CropWise was successful across all 167 updated files.

## 🎯 Acceptance Criteria
- [ ] Review `git diff` for all changes
- [ ] Verify README.md shows CropWise branding
- [ ] Check CHANGELOG.md has rebrand notice
- [ ] Test Docker Compose: `docker-compose up -d`
- [ ] Test Backend starts: `cd backend && npm start`
- [ ] Test Frontend starts: `cd frontend && npm run dev`
- [ ] Check frontend UI shows "CropWise" not "SmartCrop"
- [ ] Verify all links still work
- [ ] Check package.json files are correct

## 🔗 Related
- Rebranding script: `scripts/rebrand-to-cropwise.py`
- Plan: `REBRANDING_AND_DOCS_PLAN.md`

## ✅ Definition of Done
- [ ] All services start without errors
- [ ] No "SmartCrop" references in user-facing UI
- [ ] PR created with verification results
- [ ] Changes committed to develop branch
```

---

## 🟡 PRIORITY 2: Documentation Consolidation

### Issue #2: Consolidate Quick Start Guides

```markdown
**Title:** [DOCS] Consolidate Quick Start and Installation Guides

**Labels:** documentation, refactor

**Description:**

## 📚 Documentation Task

### What needs to be documented?
Consolidate 4 overlapping quick start/installation guides into 2 well-organized files.

### Current State
- `QUICK_START.md` (395 lines) - Docker/local setup
- `CROPWISE_QUICK_START.md` (90 lines) - AWS URLs, domain
- `docs/GETTING_STARTED.md` (349 lines) - Full tutorial
- `docs/INSTALLATION.md` (595 lines) - Comprehensive installation

### Desired State
**NEW Structure:**
```
ROOT:
  QUICK_START.md (NEW - 150 lines max)
  └─ 5-minute Docker quick start only

docs/01-getting-started/:
  INSTALLATION.md (ENHANCED)
  └─ Comprehensive installation for all platforms
  
  FIRST_STEPS.md (NEW)
  └─ Post-install tutorial (farms, zones, devices)
```

## 🎯 Scope
- [x] User-facing documentation
- [x] Developer documentation

## ✅ Checklist
- [ ] Create streamlined `QUICK_START.md` (Docker only)
- [ ] Merge best content into `docs/INSTALLATION.md`
- [ ] Create `docs/FIRST_STEPS.md`
- [ ] Delete `CROPWISE_QUICK_START.md`
- [ ] Delete old `docs/GETTING_STARTED.md`
- [ ] Test all instructions work
- [ ] Update links in other docs
- [ ] PR created and reviewed
```

---

### Issue #3: Consolidate AWS Deployment Guides

```markdown
**Title:** [DOCS] Consolidate AWS deployment guides into single comprehensive guide

**Labels:** documentation, refactor, aws

**Description:**

## 📚 Documentation Task

### What needs to be documented?
Merge 7 scattered AWS deployment files into ONE comprehensive guide.

### Current State (7 files!)
1. `DEPLOY_TO_AWS_NOW.md` - Elastic Beanstalk quick start
2. `AWS_INFRASTRUCTURE_SETUP.md` - ECS/ECR manual setup
3. `AWS_RDS_POSTGRESQL_SETUP.md` - Database setup
4. `AWS_SECURITY_CHECKLIST.md` - Security considerations
5. `docs/AWS_DEPLOYMENT_GUIDE.md` - Comprehensive (742 lines)
6. `docs/AWS_QUICK_START.md` - Quick deploy
7. `QUICK_START_ELASTIC_BEANSTALK.md` - EB specific

### Desired State
**Single file:** `docs/AWS_DEPLOYMENT.md` with chapters:
- Chapter 1: Quick Start
- Chapter 2: Prerequisites
- Chapter 3: Infrastructure Setup
- Chapter 4: Database Setup (RDS)
- Chapter 5: Deployment Options (EB, ECS, EC2)
- Chapter 6: Post-Deployment
- Chapter 7: Security Checklist
- Chapter 8: Monitoring & Scaling
- Chapter 9: Troubleshooting

## 🎯 Scope
- [x] Deployment documentation
- [x] AWS infrastructure

## ✅ Checklist
- [ ] Create comprehensive `docs/AWS_DEPLOYMENT.md`
- [ ] Merge all 7 files into chapters
- [ ] Test instructions on fresh AWS account
- [ ] Delete 7 old AWS files
- [ ] Update links in other documentation
- [ ] Add to `docs/README.md` index
- [ ] PR created and reviewed
```

---

### Issue #4: Consolidate GitHub Setup Guides

```markdown
**Title:** [DOCS] Consolidate GitHub and workflow documentation

**Labels:** documentation, refactor, github

**Description:**

## 📚 Documentation Task

### Current State (6 files)
1. `SETUP_BRANCH_PROTECTION.md` - Branch protection setup
2. `GITHUB_SETUP_CHECKLIST.md` - GitHub configuration
3. `docs/GITHUB_SETUP_GUIDE.md` - Complete guide
4. `WORKFLOW_DIAGRAM.md` - Visual workflows
5. `docs/GIT_WORKFLOW.md` - Git workflow guide
6. `CICD_SETUP_COMPLETE.md` - CI/CD summary

### Desired State
```
docs/04-operations/:
  GITHUB_SETUP.md (CONSOLIDATED)
  └─ Chapters: Repo Setup, Branch Protection, CI/CD, Labels
  
  GIT_WORKFLOW.md (ENHANCED)
  └─ Daily git workflow with diagrams
  
  RELEASE_PROCESS.md (KEEP)
  └─ Already comprehensive
```

## ✅ Checklist
- [ ] Create consolidated `docs/GITHUB_SETUP.md`
- [ ] Enhance `docs/GIT_WORKFLOW.md` with diagrams
- [ ] Move `CICD_SETUP_COMPLETE.md` to `docs/`
- [ ] Delete redundant root-level files
- [ ] Test all GitHub setup instructions
- [ ] Update links
- [ ] PR created and reviewed
```

---

### Issue #5: Reorganize Documentation Structure

```markdown
**Title:** [REFACTOR] Reorganize documentation into 8-category structure

**Labels:** refactor, documentation, priority-high

**Description:**

## 🔧 Refactoring Task

### What needs to be refactored?
Reorganize 72 markdown files into a clean 8-category structure.

### Why?
- 42 files in root (should be ~5)
- Hard to find information
- ~30% duplicate content
- No clear organization

### Proposed Structure
```
docs/
├── 01-getting-started/
├── 02-development/
├── 03-deployment/
├── 04-operations/
├── 05-features/
├── 06-integrations/
├── 07-reference/
├── 08-team/
└── archive/
```

Full plan: `REBRANDING_AND_DOCS_PLAN.md` (Phase 3)

## 🎯 Goals
- [ ] Reduce root files from 42 to ~5
- [ ] Eliminate duplicate content
- [ ] Create logical 8-category structure
- [ ] Create `docs/README.md` hub

## ⚠️ Risks
- Breaking external links (mitigate with redirects note)

## ✅ Success Criteria
- [ ] All docs in proper category folders
- [ ] `docs/README.md` index created
- [ ] Root has max 5 docs files
- [ ] All internal links updated
- [ ] No duplicate content
```

---

### Issue #6: Create Documentation Hub

```markdown
**Title:** [DOCS] Create docs/README.md as central documentation hub

**Labels:** documentation

**Description:**

## 📚 Documentation Task

### What needs to be documented?
Create a central `docs/README.md` file that serves as the main entry point for all documentation.

### Structure
```markdown
# CropWise Documentation Hub

## Quick Links
- [Installation](01-getting-started/INSTALLATION.md)
- [Quick Start](../QUICK_START.md)
- [Developer Guide](02-development/DEVELOPER_GUIDE.md)
- [Deployment Guide](03-deployment/DEPLOYMENT_GUIDE.md)

## Documentation by Category
[8 categories with descriptions]

## For Different Roles
- New Users: Start here...
- Developers: Start here...
- DevOps: Start here...
- Contributors: Start here...
```

## ✅ Checklist
- [ ] Create `docs/README.md`
- [ ] Link to all major documentation
- [ ] Add role-based guides
- [ ] Add search tips
- [ ] Link from main README.md
- [ ] Verify all links work
```

---

### Issue #7: Archive Historical Documents

```markdown
**Title:** [REFACTOR] Move historical documents to archive

**Labels:** refactor, documentation

**Description:**

## 🔧 Refactoring Task

### What needs to be refactored?
Move outdated/historical documents to `docs/archive/` directory.

### Files to Archive
1. `EXECUTIVE_SUMMARY.md` - Business doc (outdated)
2. `HARVEST_SYSTEM_COMPLETE.md` - Feature now in main docs
3. `HARVEST_TRACKING_SUMMARY.md` - Merged into user guide
4. `COMMERCIALIZATION_STATUS.md` - Business planning doc
5. `README_UPDATE.md` - Temporary notes
6. `REGISTRATION_FIX.md` - Old bug (should be in CHANGELOG)
7. `MIGRATION_PLAN.md` - Historical

### Approach
1. Create `docs/archive/` directory
2. Move files with git mv (preserve history)
3. Add `docs/archive/README.md` explaining archive
4. Update any references to archived docs

## ✅ Success Criteria
- [ ] Archive directory created
- [ ] Files moved preserving git history
- [ ] Archive README explains purpose
- [ ] No broken links to archived docs
```

---

## 🟢 PRIORITY 3: Branch Protection & Workflow

### Issue #8: Update Branch Protection Rules

```markdown
**Title:** [TASK] Update branch protection to prevent direct commits

**Labels:** task, github, priority-high

**Description:**

## 📋 Task Description
Update branch protection rules for `main` and `develop` branches to enforce issue-based workflow.

## 🎯 Current vs Desired State

### Current
- Some branch protection exists
- Direct commits still possible
- No issue requirement

### Desired
- **NO direct commits** to main/develop
- All changes via Pull Request
- Every PR must link to GitHub issue
- Status checks must pass
- At least 1 review required

## 🔧 Branch Protection Settings

### For `main` branch:
```yaml
Protection Rules:
  ✅ Require pull request before merging
  ✅ Require approvals: 1
  ✅ Dismiss stale reviews
  ✅ Require review from Code Owners
  ✅ Require status checks (CI/CD)
  ✅ Require branches to be up to date
  ✅ Require conversation resolution
  ✅ Require linear history
  ✅ Include administrators
  ✅ Restrict pushes (none allowed directly)
  ✅ Allow force pushes: OFF
  ✅ Allow deletions: OFF
```

### For `develop` branch:
```yaml
Protection Rules:
  ✅ Require pull request before merging
  ✅ Require approvals: 1
  ✅ Require status checks (CI/CD)
  ✅ Require linked issue (pr-issue-validator)
  ✅ Include administrators
  ✅ Restrict pushes (none allowed directly)
```

## ✅ Acceptance Criteria
- [ ] `main` branch protection configured
- [ ] `develop` branch protection configured
- [ ] Test: Try direct commit (should fail)
- [ ] Test: Create PR without issue (should fail)
- [ ] Test: Create PR with issue (should work)
- [ ] Document settings in `SETUP_BRANCH_PROTECTION.md`

## 📝 Notes
- Existing workflow: `.github/workflows/pr-issue-validator.yml`
- Documentation: `SETUP_BRANCH_PROTECTION.md`
```

---

### Issue #9: Create Issue-Based Workflow Guide

```markdown
**Title:** [DOCS] Create comprehensive issue-based workflow guide

**Labels:** documentation, github

**Description:**

## 📚 Documentation Task

### What needs to be documented?
Complete guide for the new issue-based development workflow.

### Content Needed

#### 1. ISSUE_WORKFLOW.md
```markdown
# Issue-Based Development Workflow

## Core Principles
- Every change requires a GitHub issue
- No direct commits to main/develop
- All changes via Pull Requests
- Every PR links to an issue

## Workflow Steps
1. Create GitHub Issue
2. Create feature branch
3. Make changes
4. Create Pull Request (link issue)
5. Code Review
6. Tests pass
7. Merge to develop
8. Deploy to staging
9. Merge to main
10. Deploy to production

## Issue Types
- Features: [FEATURE]
- Bugs: [BUG]
- Tasks: [TASK]
- Documentation: [DOCS]
- Refactoring: [REFACTOR]

## Branch Naming
- feature/123-issue-title
- bugfix/123-issue-title
- docs/123-issue-title
- refactor/123-issue-title

## PR Title Format
- [Fixes #123] Issue title
- [Closes #123] Issue title
- [Resolves #123] Issue title

## Emergency Hotfixes
Only exception: Critical production bugs
- Create issue first
- Use hotfix/ branch
- Fast-track review
- Document in issue
```

## ✅ Checklist
- [ ] Create `ISSUE_WORKFLOW.md`
- [ ] Add examples and screenshots
- [ ] Document emergency process
- [ ] Link from CONTRIBUTING.md
- [ ] Link from docs/README.md
- [ ] Update onboarding guide
```

---

## 🔵 PRIORITY 4: CI/CD & Automation

### Issue #10: Add Issue Requirement to CI/CD

```markdown
**Title:** [TASK] Enhance PR validation to require linked GitHub issues

**Labels:** task, ci-cd, github

**Description:**

## 📋 Task Description
Enhance existing PR issue validator to be more strict and informative.

## Current State
- Basic validator exists: `.github/workflows/pr-issue-validator.yml`
- May allow some PRs without issues

## Desired Enhancements
1. **Strict validation**
   - Block PR if no issue linked
   - Check for valid issue number
   - Verify issue is open
   
2. **Better feedback**
   - Comment on PR with instructions
   - Link to issue workflow guide
   - Suggest issue template

3. **Exemptions** (optional)
   - Dependabot PRs
   - Documentation typo fixes
   - Hotfixes (must have hotfix label)

## ✅ Acceptance Criteria
- [ ] PR without issue is blocked
- [ ] Clear error message shown
- [ ] Bot comments with helpful instructions
- [ ] Exemptions work (if implemented)
- [ ] Test with real PR
- [ ] Document in workflow guide
```

---

## 📋 Creating These Issues

### Method 1: Manual (GitHub Web UI)
1. Go to your repository on GitHub
2. Click "Issues" tab
3. Click "New Issue"
4. Select appropriate template
5. Copy/paste content from above
6. Add labels
7. Click "Submit new issue"

### Method 2: GitHub CLI (Faster)

```bash
# Install GitHub CLI if needed
# https://cli.github.com/

# Create issues from command line
gh issue create --title "[TASK] Verify CropWise rebranding" \
  --label "task,testing,priority-high" \
  --body-file issue-1.md

# Repeat for each issue...
```

### Method 3: Script (Bulk Create)

Create `scripts/create-github-issues.sh`:

```bash
#!/bin/bash

# Issue #1: Rebranding verification
gh issue create \
  --title "[TASK] Verify CropWise rebranding across entire codebase" \
  --label "task,testing,priority-high" \
  --body "$(cat <<EOF
## 📋 Task Description
Verify that the automated rebranding from SmartCrop to CropWise was successful.

## 🎯 Acceptance Criteria
- [ ] Review git diff
- [ ] Test Docker Compose
- [ ] Test Backend/Frontend
- [ ] Verify UI shows CropWise

See GITHUB_ISSUES_TO_CREATE.md for full details.
EOF
)"

# Add more issues...
```

---

## ✅ After Creating Issues

1. **Pin important issues** (Issues #1, #8, #9)
2. **Create project board** (optional)
   - Columns: To Do, In Progress, Review, Done
   - Add all issues to board
3. **Assign issues** to team members
4. **Set milestones**:
   - v1.1: Rebranding & Branch Protection
   - v1.2: Documentation Consolidation
   - v2.0: Full restructure

---

## 🚀 Next Steps (After Issues Created)

1. ✅ **Issue #8 FIRST** - Set up branch protection
2. ✅ **Issue #9 SECOND** - Create workflow guide
3. ✅ **Issue #1** - Verify rebranding
4. Then pick any issue and start working!

---

**Remember:** Once branch protection is enabled, you'll create a branch for each issue:

```bash
# For issue #5
git checkout -b refactor/5-reorganize-docs-structure
# Make changes
git commit -m "refactor: reorganize docs structure [#5]"
git push origin refactor/5-reorganize-docs-structure
# Create PR on GitHub linking to #5
```

---

**Created:** November 2024  
**Status:** Ready to create  
**Total Issues:** 10 (prioritized)

