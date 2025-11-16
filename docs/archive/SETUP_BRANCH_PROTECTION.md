# 🚀 Quick Setup: Branch Protection Rules

## ⏱️ Time Required: 10 minutes

Follow these exact steps to enable branch protection on your repository.

---

## 📍 Step 1: Navigate to Settings

1. Go to your repository: `https://github.com/yellowflowersorganics-star/Yellow-Flowers-Organics`
2. Click **"Settings"** tab (top navigation, next to "Insights")
3. Click **"Branches"** in the left sidebar (under "Code and automation" section)

**You should see**: "Branch protection rules" section

---

## 🛡️ Step 2: Protect Main Branch

### Click "Add rule" or "Add branch protection rule"

### Enter Branch Name Pattern
```
main
```

### Enable These Settings (scroll down to check each):

#### ✅ Require a pull request before merging
- [x] Check this box
- **Dropdown opens** - configure:
  - [x] **Required number of approvals before merging**: Select **1** (or 2 for more safety)
  - [x] **Dismiss stale pull request approvals when new commits are pushed**
  - [ ] Require review from Code Owners (skip for now - optional)
  - [ ] Restrict who can dismiss pull request reviews (skip)
  - [ ] Allow specified actors to bypass (skip)

#### ✅ Require status checks to pass before merging
- [x] Check this box
- **Note**: Status checks will appear after your CI workflow runs once
- **After first CI run**, you'll see:
  - Search for: `validate`
  - Search for: `test`
  - Search for: `build`
  - Click to add each one
- [x] **Require branches to be up to date before merging**

#### ✅ Require conversation resolution before merging
- [x] Check this box

#### ⚠️ Require signed commits
- [ ] Leave unchecked (unless your team uses GPG signing)

#### ✅ Require linear history
- [x] Check this box (prevents messy merge commits)

#### ⚠️ Require deployments to succeed before merging
- [ ] Leave unchecked (optional - for staging environments)

#### ❌ Lock branch
- [ ] **DO NOT CHECK** (this makes branch read-only - don't use!)

#### ✅ Do not allow bypassing the above settings
- [x] Check this box (even admins must follow rules)

#### ⚠️ Restrict who can push to matching branches
- [ ] Leave unchecked (for small teams)
- **Note**: For large teams, you can restrict to specific users/teams

#### ❌ Allow force pushes
- [ ] **DO NOT CHECK** - Leave this DISABLED!

#### ❌ Allow deletions
- [ ] **DO NOT CHECK** - Leave this DISABLED!

### Save the Rule

Scroll to bottom and click: **"Create"** or **"Save changes"**

✅ **Main branch is now protected!**

---

## 🔧 Step 3: Protect Develop Branch (Optional but Recommended)

### Click "Add rule" again

### Enter Branch Name Pattern
```
develop
```

### Enable These Settings:

#### ✅ Require a pull request before merging
- [x] Check this box
- **Configuration**:
  - [x] **Required approvals**: Select **1**
  - [x] **Dismiss stale pull request approvals**

#### ✅ Require status checks to pass before merging
- [x] Check this box
- [x] **Require branches to be up to date**

#### ✅ Require conversation resolution before merging
- [x] Check this box

#### ⚠️ Require linear history
- [x] Optional - check if you want clean history on develop too

#### ⚠️ Do not allow bypassing the above settings
- [ ] **Leave unchecked** (allow admin flexibility on develop)

#### ❌ Allow force pushes
- [ ] **DO NOT CHECK** - Leave DISABLED!

#### ❌ Allow deletions
- [ ] **DO NOT CHECK** - Leave DISABLED!

### Save the Rule

Click: **"Create"**

✅ **Develop branch is now protected!**

---

## 🧪 Step 4: Verify Protection is Working

### Wait for CI Workflow to Run

1. Your push just triggered the CI workflow
2. Go to **"Actions"** tab in your repository
3. You should see "CI Checks" workflow running
4. Wait for it to complete (usually 1-2 minutes)

### Test Direct Push (Should Fail)

```bash
# Try to push directly to main - this SHOULD FAIL
git checkout main
git pull origin main
echo "test" >> test.txt
git add test.txt
git commit -m "test: Direct push"
git push origin main
```

**Expected Error**:
```
remote: error: GH006: Protected branch update failed for refs/heads/main.
remote: error: Changes must be made through a pull request.
```

✅ **If you see this error, protection is working correctly!**

### Clean Up Test

```bash
# Remove the test commit
git reset --hard HEAD~1
```

---

## ✅ Step 5: Enable Status Checks (After CI Runs)

After your CI workflow completes once:

1. Go back to **Settings → Branches**
2. Click **"Edit"** on the main branch protection rule
3. Scroll to **"Require status checks to pass before merging"**
4. In the search box, you should now see:
   - `validate` ← Click to add
   - `test` ← Click to add
   - `build` ← Click to add
5. Scroll down and click **"Save changes"**

✅ **Status checks are now required!**

---

## 📋 Quick Verification Checklist

After setup, verify:

- [ ] Go to repository Settings → Branches
- [ ] You see 2 branch protection rules:
  - [ ] `main` branch rule with icon 🔒
  - [ ] `develop` branch rule with icon 🔒
- [ ] Try direct push to main (should fail)
- [ ] Try direct push to develop (should fail)
- [ ] Go to Actions tab and see CI workflow ran successfully
- [ ] Status checks added to main branch protection

---

## 🎯 What This Accomplishes

### Before Branch Protection
```
Developer → git push origin main → ❌ Code directly in production (RISKY!)
```

### After Branch Protection
```
Developer → Feature Branch → Pull Request → Code Review → 
CI Tests Pass → Approval → Merge → ✅ Safe deployment
```

---

## 🚦 Normal Workflow Now

### 1. Create Feature Branch
```bash
git checkout main
git pull origin main
git checkout -b feature/my-new-feature
```

### 2. Make Changes and Commit
```bash
# ... make your changes ...
git add .
git commit -m "feat: Add new feature"
git push origin feature/my-new-feature
```

### 3. Create Pull Request on GitHub
1. Go to your repository on GitHub
2. You'll see a yellow banner: "Compare & pull request" - click it
3. Or go to "Pull requests" tab → "New pull request"
4. **Base**: `main` ← **Compare**: `feature/my-new-feature`
5. Fill in PR description
6. Click "Create pull request"

### 4. Wait for Checks and Review
- ✅ CI checks will run automatically
- ✅ Request review from team member
- ✅ Address any review comments
- ✅ Wait for approval

### 5. Merge
- After approval and checks pass, click **"Merge pull request"**
- Choose merge method:
  - **Squash and merge** (recommended - cleaner history)
  - **Rebase and merge** (linear history)
  - **Create a merge commit** (full history)

### 6. Delete Feature Branch
- After merge, GitHub will prompt: "Delete branch"
- Click it to keep repository clean

---

## 🆘 Common Issues

### Issue: "This branch is out-of-date with the base branch"

**Solution**:
```bash
git checkout feature/my-branch
git fetch origin
git merge origin/main
git push origin feature/my-branch
```

### Issue: "Review required - At least 1 approving review is required"

**Solution**:
- Ask a teammate to review your PR
- Go to PR → "Reviewers" → Select reviewer
- They'll review and click "Approve"

### Issue: "Required status checks have not passed"

**Solution**:
- Go to PR → "Checks" tab
- See which check failed
- Fix the issue locally
- Push the fix to your branch
- Checks will run again automatically

### Issue: Admin needs to push directly (EMERGENCY ONLY)

**Solution**:
1. Go to Settings → Branches
2. Edit main branch rule
3. **Temporarily** uncheck "Do not allow bypassing"
4. Make your emergency push
5. **IMMEDIATELY** re-enable "Do not allow bypassing"
6. Document why bypass was needed

---

## 📞 Need Help?

- **Can't find Settings tab?** Make sure you're a repository admin
- **Don't see Branches in sidebar?** Scroll down in Settings
- **Status checks not appearing?** Wait for CI workflow to run once
- **Still having issues?** Check [BRANCH_PROTECTION.md](BRANCH_PROTECTION.md) for detailed troubleshooting

---

## 🎉 You're Done!

Your repository now has professional-grade branch protection! 

**Next Steps**:
1. Share this guide with your team
2. Create a test PR to practice the new workflow
3. Update [CONTRIBUTING.md](CONTRIBUTING.md) with your team's specific workflow

**Benefits**:
- ✅ No accidental direct commits to production
- ✅ All code is reviewed before merging
- ✅ Automated tests prevent bugs
- ✅ Cleaner git history
- ✅ Better team collaboration

---

**Remember**: Branch protection is your friend, not an obstacle! It prevents costly mistakes and improves code quality. 🚀

