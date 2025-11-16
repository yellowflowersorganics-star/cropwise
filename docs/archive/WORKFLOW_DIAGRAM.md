# 📊 Visual Workflow Diagrams

Complete visual guide to all workflows in the Yellow Flowers Organic Farm project.

---

## 🔄 Standard Development Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                    STANDARD FEATURE WORKFLOW                     │
└─────────────────────────────────────────────────────────────────┘

START
  │
  ├─► 1. Create Issue on GitHub
  │     ├─ Go to Issues → New Issue
  │     ├─ Choose template (Bug/Feature/Docs/Article)
  │     ├─ Fill out template
  │     └─ Submit issue (Note issue #)
  │
  ├─► 2. Update Local Repository
  │     └─ git checkout main
  │     └─ git pull origin main
  │
  ├─► 3. Create Feature Branch
  │     └─ git checkout -b feature/your-feature-name
  │
  ├─► 4. Develop Feature
  │     ├─ Edit files
  │     ├─ Test locally (npm start)
  │     └─ Test on mobile/tablet/desktop
  │
  ├─► 5. Commit Changes
  │     ├─ git add .
  │     └─ git commit -m "type: description"
  │
  ├─► 6. Push to GitHub
  │     └─ git push origin feature/your-feature-name
  │
  ├─► 7. Create Pull Request
  │     ├─ Go to GitHub → Pull Requests → New PR
  │     ├─ Fill PR template
  │     ├─ Add "Closes #123" (REQUIRED!)
  │     └─ Request reviewers
  │
  ├─► 8. Automated Checks Run
  │     ├─ PR Issue Validator ✓
  │     ├─ CI Checks (validate, test, build) ✓
  │     └─ All checks must pass
  │
  ├─► 9. Code Review
  │     ├─ Reviewer examines code
  │     ├─ Reviewer leaves comments/suggestions
  │     └─ You address feedback (repeat 4-6 if needed)
  │
  ├─► 10. Approval & Merge
  │     ├─ Reviewer approves PR
  │     ├─ All checks pass ✓
  │     ├─ Merge PR (Squash and merge)
  │     └─ Delete feature branch
  │
  └─► 11. Deployment
        ├─ GitHub Actions auto-deploys to AWS S3
        ├─ Site updates in ~2-3 minutes
        └─ Verify deployment
        
END ✅
```

---

## 🚨 Hotfix Workflow (Emergency)

```
┌─────────────────────────────────────────────────────────────────┐
│                      HOTFIX WORKFLOW                            │
│                    (For Critical Bugs)                           │
└─────────────────────────────────────────────────────────────────┘

PRODUCTION BUG DISCOVERED! 🔴
  │
  ├─► 1. Create Hotfix Issue (Priority: Critical)
  │     └─ Document the critical bug
  │
  ├─► 2. Branch from Main (Production)
  │     ├─ git checkout main
  │     ├─ git pull origin main
  │     └─ git checkout -b hotfix/critical-bug-description
  │
  ├─► 3. Fix Bug Quickly
  │     ├─ Make minimal necessary changes
  │     ├─ Test fix thoroughly
  │     └─ Verify bug is resolved
  │
  ├─► 4. Commit & Push
  │     ├─ git add .
  │     ├─ git commit -m "fix: critical bug description"
  │     └─ git push origin hotfix/critical-bug-description
  │
  ├─► 5. Create PR (Fast-Track)
  │     ├─ Link to critical issue
  │     ├─ Label as "hotfix"
  │     └─ Request immediate review
  │
  ├─► 6. Quick Review & Merge
  │     ├─ Review ASAP (< 1 hour)
  │     ├─ Approve if fix is correct
  │     └─ Merge to main
  │
  ├─► 7. Auto-Deploy to Production
  │     └─ GitHub Actions deploys immediately
  │
  ├─► 8. Verify Fix in Production
  │     └─ Test that bug is resolved
  │
  └─► 9. Merge Back to Develop
        ├─ git checkout develop
        ├─ git merge hotfix/critical-bug-description
        ├─ git push origin develop
        └─ Delete hotfix branch
        
HOTFIX COMPLETE ✅
Total Time: 2-4 hours
```

---

## 🚀 Release Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                      RELEASE WORKFLOW                            │
└─────────────────────────────────────────────────────────────────┘

WEEK 1-3: DEVELOPMENT PHASE
  │
  ├─► Multiple features developed in parallel
  │     ├─ feature/article-1 → develop
  │     ├─ feature/article-2 → develop
  │     ├─ fix/bug-xyz → develop
  │     └─ All merged to develop branch
  │
  ↓

WEEK 4: RELEASE PREPARATION
  │
  ├─► Day 1-2: Code Freeze & Branch Creation
  │     ├─ Announce code freeze to team
  │     ├─ git checkout develop
  │     ├─ git checkout -b release/v1.1.0
  │     └─ git push origin release/v1.1.0
  │
  ├─► Day 3-4: QE Testing
  │     ├─ Full regression testing
  │     ├─ Cross-browser testing
  │     ├─ Mobile device testing
  │     ├─ Performance testing
  │     └─ Accessibility testing
  │
  ├─► Day 4-5: Bug Fixes (on release branch)
  │     ├─ Fix critical bugs only
  │     ├─ Re-test after fixes
  │     └─ All tests pass ✓
  │
  ├─► Day 5: Finalize Release
  │     ├─ Update CHANGELOG.md
  │     ├─ Write release notes
  │     └─ Final approval from team
  │
  ↓

RELEASE DAY
  │
  ├─► 1. Merge to Main
  │     ├─ git checkout main
  │     ├─ git merge release/v1.1.0 --no-ff
  │     └─ git push origin main
  │
  ├─► 2. Create Tag
  │     ├─ git tag -a v1.1.0 -m "Release v1.1.0"
  │     └─ git push origin v1.1.0
  │
  ├─► 3. Automated Deployment
  │     ├─ GitHub Actions triggered by tag
  │     ├─ Deploys to AWS S3
  │     └─ Creates GitHub Release
  │
  ├─► 4. Verify Production
  │     ├─ Smoke tests on live site
  │     ├─ Check all critical paths
  │     └─ Monitor for errors
  │
  ├─► 5. Merge Back to Develop
  │     ├─ git checkout develop
  │     ├─ git merge release/v1.1.0 --no-ff
  │     └─ git push origin develop
  │
  └─► 6. Post-Release
        ├─ Announce release to stakeholders
        ├─ Monitor analytics
        └─ Gather feedback
        
RELEASE COMPLETE ✅ 🎉
```

---

## ✅ PR Validation Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                   PR VALIDATION WORKFLOW                         │
└─────────────────────────────────────────────────────────────────┘

DEVELOPER CREATES PR
  │
  ├─► GitHub Actions Triggered
  │     ├─ PR Issue Validator
  │     ├─ CI Checks (validate, test, build)
  │     └─ All workflows run in parallel
  │
  ↓

PR ISSUE VALIDATOR
  │
  ├─► Checks PR Body & Title for Issue Reference
  │     ├─ Pattern: #123
  │     ├─ Pattern: Closes #123
  │     ├─ Pattern: Fixes #456
  │     ├─ Pattern: Resolves #789
  │     └─ Pattern: Related to #999
  │
  ├─► Issue Found? ────┐
  │                     │
  YES ✅               NO ❌
  │                     │
  ├─► Post Success     ├─► Post Failure Comment
  │   Comment          │     ├─ Explain requirement
  │   └─ Lists issues  │     ├─ Show how to fix
  │                    │     └─ Link to docs
  │                    │
  └─► Check PASSES ✓   └─► Check FAILS ✗
        │                    │
        │                    └─► PR BLOCKED
        │                         Cannot merge
        │
        ↓

CI CHECKS
  │
  ├─► Validate Job
  │     ├─ Check HTML files
  │     ├─ Check CSS files
  │     ├─ Check JavaScript files
  │     └─ Check code formatting
  │
  ├─► Test Job
  │     └─ Run test suite
  │
  └─► Build Job
        ├─ Verify required files
        ├─ Count articles
        └─ Build check
        
ALL CHECKS PASS ✓
  │
  └─► PR Ready for Review
        ├─ Reviewer can approve
        ├─ Conversations must be resolved
        └─ Then can merge
```

---

## 🔍 Issue to Deployment Flow

```
┌─────────────────────────────────────────────────────────────────┐
│              COMPLETE ISSUE-TO-DEPLOYMENT FLOW                   │
└─────────────────────────────────────────────────────────────────┘

USER/TEAM MEMBER HAS IDEA
  │
  ├─► 1. Create Issue
  │     ├─ Go to GitHub Issues
  │     ├─ Choose template
  │     ├─ Fill out details
  │     └─ Submit → Issue #42 created
  │
  ├─► 2. Issue Triage
  │     ├─ Team reviews issue
  │     ├─ Assigns priority label
  │     ├─ Assigns to team member
  │     └─ Issue moved to "In Progress"
  │
  ├─► 3. Development
  │     ├─ Dev creates branch
  │     ├─ Implements feature
  │     └─ Tests locally
  │
  ├─► 4. Pull Request
  │     ├─ Creates PR
  │     ├─ Links to Issue #42
  │     └─ Automated checks run
  │
  ├─► 5. Code Review
  │     ├─ Reviewer examines code
  │     ├─ Provides feedback
  │     └─ Approves when ready
  │
  ├─► 6. Merge to Develop
  │     ├─ PR merged
  │     └─ Issue #42 auto-closes
  │
  ├─► 7. Integration Testing
  │     └─ Feature tested in develop branch
  │
  ├─► 8. Release Planning
  │     └─ Feature included in next release
  │
  ├─► 9. Release to Production
  │     ├─ Merge to main
  │     ├─ Tag created
  │     └─ Auto-deploy to AWS
  │
  └─► 10. Feature Live! 🎉
        ├─ Available to users
        ├─ Monitored for issues
        └─ Issue marked as "Done"

TOTAL TIME: 1-4 weeks (depending on complexity)
```

---

## 🌳 Branch Strategy Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                      BRANCH STRATEGY                             │
└─────────────────────────────────────────────────────────────────┘

main (production) ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ↑                                                            ↑
  │ merge                                                merge │
  │                                                            │
release/v1.1.0 ──────────────────────────────────────────────┘
  ↑
  │ create
  │
develop ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ↑         ↑         ↑         ↑         ↑
  │         │         │         │         │
  │         │         │         │         └─ feature/article-5
  │         │         │         │
  │         │         │         └─────────── feature/article-4
  │         │         │
  │         │         └──────────────────── fix/navigation-bug
  │         │
  │         └─────────────────────────────── feature/article-3
  │
  └───────────────────────────────────────── feature/article-2


BRANCH TYPES:

main
  ├─ Always production-ready
  ├─ Protected (no direct commits)
  ├─ Auto-deploys to AWS S3
  └─ Tagged with version numbers

develop
  ├─ Integration branch
  ├─ Contains next release features
  ├─ Protected (requires PR)
  └─ Base for feature branches

feature/*
  ├─ New features or enhancements
  ├─ Branch from: develop
  ├─ Merge to: develop
  └─ Delete after merge

fix/*
  ├─ Bug fixes
  ├─ Branch from: develop
  ├─ Merge to: develop
  └─ Delete after merge

hotfix/*
  ├─ Critical production fixes
  ├─ Branch from: main
  ├─ Merge to: main AND develop
  └─ Delete after merge

release/*
  ├─ Release preparation
  ├─ Branch from: develop
  ├─ Merge to: main AND develop
  └─ Delete after merge
```

---

## 🔒 Protection Rules Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                   BRANCH PROTECTION RULES                        │
└─────────────────────────────────────────────────────────────────┘

main BRANCH (Strictest)
├─ ✅ Require pull request
│  └─ Require 2 approvals
├─ ✅ Require status checks
│  ├─ PR Issue Validator
│  ├─ CI: validate
│  ├─ CI: test
│  └─ CI: build
├─ ✅ Require conversation resolution
├─ ✅ Require linear history
├─ ✅ No bypassing allowed
├─ ❌ No force push
└─ ❌ No deletions

develop BRANCH (Moderate)
├─ ✅ Require pull request
│  └─ Require 1 approval
├─ ✅ Require status checks
│  ├─ PR Issue Validator
│  └─ CI checks
├─ ✅ Require conversation resolution
├─ ⚠️  Admin can bypass (emergencies)
├─ ❌ No force push
└─ ❌ No deletions

feature/* BRANCHES (Flexible)
├─ ✅ Can force push (your own branch)
├─ ✅ Can delete after merge
└─ No protection rules


ENFORCEMENT FLOW:

Developer attempts push to main
  │
  ├─► GitHub checks protection rules
  │
  ├─► Direct push? ────────────► ❌ BLOCKED
  │                               "Must use PR"
  │
  └─► Via PR?
        │
        ├─► Has issue link? ───► ❌ BLOCKED
        │                         "Must link issue"
        │
        ├─► Status checks pass? ─► ❌ BLOCKED
        │                          "Checks must pass"
        │
        ├─► Has approval? ───────► ❌ BLOCKED
        │                          "Need approval"
        │
        └─► All requirements met ► ✅ ALLOW MERGE
```

---

## 📝 Issue Template Selection

```
┌─────────────────────────────────────────────────────────────────┐
│                    ISSUE TEMPLATE DECISION TREE                  │
└─────────────────────────────────────────────────────────────────┘

WHAT DO YOU WANT TO DO?
  │
  ├─► Something is broken?
  │   └─► 🐛 Bug Report Template
  │       ├─ Describe the bug
  │       ├─ Steps to reproduce
  │       ├─ Expected vs actual behavior
  │       ├─ Browser & device info
  │       └─ Screenshots & console errors
  │
  ├─► Want a new feature?
  │   └─► ✨ Feature Request Template
  │       ├─ Problem statement
  │       ├─ Proposed solution
  │       ├─ Alternatives considered
  │       ├─ Benefits & priority
  │       └─ Mockups/examples
  │
  ├─► Documentation unclear?
  │   └─► 📚 Documentation Update Template
  │       ├─ What's wrong/missing?
  │       ├─ Location in docs
  │       ├─ Suggested improvement
  │       └─ Severity & willingness to fix
  │
  ├─► Want a new article?
  │   └─► 📝 New Article Request Template
  │       ├─ Article title & category
  │       ├─ Description & key points
  │       ├─ Target audience
  │       ├─ References/sources
  │       └─ Can you write it?
  │
  └─► Just have a question?
        └─► 💬 GitHub Discussions
            (Not an issue template)
```

---

## 🎯 Testing Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                      TESTING WORKFLOW                            │
└─────────────────────────────────────────────────────────────────┘

CODE CHANGES MADE
  │
  ├─► 1. LOCAL TESTING (Developer)
  │     ├─ Run on local server
  │     ├─ Test in Chrome DevTools
  │     ├─ Test responsive (375px, 768px, 1920px)
  │     ├─ Check console for errors
  │     └─ Verify all links work
  │
  ├─► 2. AUTOMATED TESTING (CI)
  │     ├─ HTML validation
  │     ├─ CSS validation
  │     ├─ JavaScript validation
  │     ├─ Code formatting check
  │     └─ Build verification
  │
  ├─► 3. PR REVIEW (Peer)
  │     ├─ Reviewer pulls branch
  │     ├─ Tests locally
  │     ├─ Checks code quality
  │     └─ Leaves feedback
  │
  ├─► 4. QE TESTING (Before Release)
  │     ├─ Full regression suite
  │     ├─ Cross-browser testing
  │     │   ├─ Chrome
  │     │   ├─ Firefox
  │     │   └─ Safari
  │     ├─ Device testing
  │     │   ├─ Desktop (1920px)
  │     │   ├─ Tablet (768px)
  │     │   └─ Mobile (375px)
  │     ├─ Performance testing (Lighthouse)
  │     ├─ Accessibility testing (WAVE/axe)
  │     └─ Sign off if all pass
  │
  └─► 5. PRODUCTION SMOKE TEST (After Deploy)
        ├─ Critical paths functional
        ├─ No 404 errors
        ├─ No console errors
        ├─ Performance acceptable
        └─ All pass ✅ → Release complete


TESTING MATRIX:

                Desktop  Tablet  Mobile
Chrome             ✓       ✓       ✓
Firefox            ✓       ✓       ✓
Safari             ✓       ✓       ✓
Edge               ✓       -       -
```

---

## 🚀 Deployment Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                    DEPLOYMENT PIPELINE                           │
└─────────────────────────────────────────────────────────────────┘

DEVELOPER MERGES PR TO MAIN
  │
  ├─► GitHub Actions Triggered
  │     └─ Workflow: deploy-aws.yml
  │
  ├─► 1. Checkout Code
  │     └─ Latest code from main branch
  │
  ├─► 2. Configure AWS Credentials
  │     ├─ AWS_ACCESS_KEY_ID (secret)
  │     ├─ AWS_SECRET_ACCESS_KEY (secret)
  │     └─ AWS_REGION (secret)
  │
  ├─► 3. Sync Files to S3
  │     ├─ aws s3 sync . s3://bucket
  │     ├─ Exclude: .git/, .github/, node_modules/
  │     └─ Set cache headers
  │
  ├─► 4. Set Content Types
  │     ├─ *.html → text/html
  │     ├─ *.css → text/css
  │     └─ *.js → application/javascript
  │
  ├─► 5. Invalidate CloudFront (Optional)
  │     └─ Clear CDN cache
  │
  └─► 6. Deployment Complete ✅
        ├─ Website live in 2-3 minutes
        └─ Notification sent

DEPLOYMENT TIME: ~3 minutes
AVAILABILITY: 99.99% uptime
```

---

**Questions?** See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed workflows!

**Last Updated**: November 2024  
**Version**: 1.0.0

