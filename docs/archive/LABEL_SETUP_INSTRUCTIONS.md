# 🏷️ GitHub Labels Setup

**Quick Fix for Label Errors**

---

## Problem

When running `create-github-issues.sh`, you get errors like:
```
could not add label: 'task' not found
could not add label: 'refactor' not found
```

This happens because the labels don't exist in your repository yet.

---

## Solution: Run Label Setup Script

### Option 1: Automated Setup (Recommended)

```bash
# Create all labels at once
chmod +x scripts/setup-github-labels.sh
./scripts/setup-github-labels.sh
```

This creates **30+ labels** including:
- Issue types (bug, feature, task, docs, refactor)
- Priorities (critical, high, medium, low)
- Status (in-progress, blocked, needs-review)
- Categories (aws, github, ci-cd, frontend, backend, iot)
- Special (hotfix, question, good-first-issue)

### Option 2: Manual Creation

Go to GitHub → Repository → Issues → Labels → New label

Create these essential labels:

| Label | Color | Description |
|-------|-------|-------------|
| `task` | `#0e8a16` | General development task |
| `documentation` | `#0075ca` | Documentation improvements |
| `refactor` | `#fbca04` | Code refactoring |
| `priority-high` | `#d93f0b` | High priority |
| `priority-critical` | `#b60205` | Critical priority |
| `aws` | `#FF9900` | AWS related |
| `github` | `#000000` | GitHub related |
| `ci-cd` | `#5319e7` | CI/CD related |

### Option 3: Use GitHub's Label Sync Workflow

The workflow file already exists: `.github/workflows/setup-labels.yml`

1. Go to GitHub → Actions
2. Find "Setup Repository Labels" workflow
3. Click "Run workflow"
4. Wait for completion

---

## After Creating Labels

Run the issue creation script again:

```bash
./scripts/create-github-issues.sh
```

Now all issues will be created with proper labels!

---

## Verify Labels

```bash
# List all labels
gh label list

# Or view in browser
gh repo view --web
# Then go to: Issues → Labels
```

---

## Current Status

Based on your output:
- ✅ Issue #11 was created successfully
- ⚠️ Labels were not applied (they didn't exist)
- ⚠️ Some issues may have been created without labels

### What to Do:

1. **Run label setup:**
   ```bash
   chmod +x scripts/setup-github-labels.sh
   ./scripts/setup-github-labels.sh
   ```

2. **Update existing issues** (add labels manually):
   - Go to each issue on GitHub
   - Click "Labels" on the right
   - Add appropriate labels

3. **Or delete and recreate:**
   ```bash
   # Delete issues #6-11 (if they exist without labels)
   gh issue close 6 7 8 9 10 11 --reason "not planned"
   
   # Then run again
   ./scripts/create-github-issues.sh
   ```

---

## Quick Reference

### Essential Labels to Create

```bash
# Run these commands one by one if script doesn't work
gh label create "task" --color "0e8a16" --description "General task"
gh label create "documentation" --color "0075ca" --description "Documentation"
gh label create "refactor" --color "fbca04" --description "Refactoring"
gh label create "priority-high" --color "d93f0b" --description "High priority"
gh label create "priority-critical" --color "b60205" --description "Critical"
gh label create "aws" --color "FF9900" --description "AWS"
gh label create "github" --color "000000" --description "GitHub"
gh label create "ci-cd" --color "5319e7" --description "CI/CD"
gh label create "testing" --color "c5def5" --description "Testing"
```

---

## Troubleshooting

### "Label already exists" error
✅ This is fine! The label is already there, script continues.

### "Permission denied" error
❌ You need admin access to create labels. Ask repository owner.

### Labels not applying to old issues
✅ Expected. Add labels manually or recreate issues.

---

**Next Step:** Run `scripts/setup-github-labels.sh` and then create issues again!

