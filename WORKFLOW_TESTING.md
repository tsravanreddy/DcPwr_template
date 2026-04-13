# CI/CD Workflow Testing Guide

This guide explains how to test each workflow in your DcPwr_template repository.

## Workflow Overview

| Workflow | File | Trigger | Purpose |
|----------|------|---------|---------|
| **Build and Test** | `build_and_test.yml` | Push to any branch, PRs to main | Validate code changes |
| **Release PR** | `release_pr.yml` | PR to `Release_*` branch | Validate release candidate |
| **PR Closed** | `pr_closed.yml` | Merge PR to `Release_*` | Deploy release |
| **Template Release** | `template_release.yml` | Push tag `v*.*.*` | Create template release |

---

## Testing Each Workflow

### 1️⃣ Build and Test Workflow

**File:** `.github/workflows/build_and_test.yml`

**Test Steps:**
```powershell
# Create and push a feature branch
git checkout -b feature/test-build
git add .
git commit -m "Test build workflow"
git push origin feature/test-build
```

**What to Expect:**
- ✅ Detects repository type from `.repo-config`
- ✅ Runs `dotnet restore`, `dotnet build`, `dotnet test`
- ✅ Shows "TEMPLATE BUILD COMPLETE" (since this is a template)
- ✅ Displays "NuGet deployment SKIPPED (template mode)"

**GitHub Actions Tab:** You'll see a workflow run for "Build and Test"

---

### 2️⃣ Release PR Workflow

**File:** `.github/workflows/release_pr.yml`

**Test Steps:**
```powershell
# Create a release branch
git checkout -b Release_2026-04-15
git push origin Release_2026-04-15

# Go to GitHub web UI
# Create a Pull Request: feature branch → Release_2026-04-15
```

**What to Expect:**
- ✅ Validates repository configuration
- ⚠️ Shows warning "Templates should create release TAGS, not release branches"
- ✅ Runs build and tests anyway
- ✅ Generates release notes preview

**GitHub Actions Tab:** You'll see "Release PR Workflow" run

**Note for Templates:** This workflow warns you because templates should use tags (see workflow 4). However, it will still validate for testing purposes.

---

### 3️⃣ PR Closed (Deployment) Workflow

**File:** `.github/workflows/pr_closed.yml`

**Test Steps:**
```powershell
# After creating PR in step 2, merge it via GitHub UI
# The workflow triggers automatically when PR is merged
```

**What to Expect (for templates):**
- ⚠️ Detects `"type": "template"` in `.repo-config`
- ⚠️ Shows "TEMPLATE REPOSITORY DETECTED"
- ⚠️ Skips deployment
- ✅ Suggests using tags instead: `git tag -a v1.0.0`

**What to Expect (for drivers):**
- ✅ Builds production release
- ✅ Creates NuGet package
- ✅ Publishes to NuGet (when secrets configured)
- ✅ Creates GitHub release

**GitHub Actions Tab:** You'll see "PR Closed (Release Deployment)"

**Testing Drivers:** To test driver deployment, change `.repo-config`:
```json
{
  "repository": {
    "type": "driver",        // ← Change this
    "deployment_enabled": true
  }
}
```

---

### 4️⃣ Template Release Workflow (Recommended for Templates)

**File:** `.github/workflows/template_release.yml`

**Test Steps:**
```powershell
# Ensure you're on main branch
git checkout main
git pull

# Create and push a version tag
git tag -a v1.0.0 -m "Template release v1.0.0"
git push origin v1.0.0
```

**What to Expect:**
- ✅ Detects template configuration
- ✅ Runs full build and test validation
- ✅ Creates GitHub Release with:
  - Release notes
  - Usage instructions
  - Source code archive
- ✅ Shows "TEMPLATE RELEASE CREATED"
- ❌ NO NuGet deployment

**GitHub Releases:**
- Go to `https://github.com/your-org/DcPwr_template/releases`
- You'll see release `v1.0.0` with full documentation

**This is the PRIMARY way to release templates!**

---

## Workflow Comparison: Template vs Driver

### Template Repository Behavior

| Workflow | Template Behavior |
|----------|-------------------|
| build_and_test | ✅ Build + Test only, no deployment |
| release_pr | ⚠️ Warns to use tags instead |
| pr_closed | ⚠️ Skips deployment, suggests tags |
| template_release | ✅ **THIS IS THE WAY** - Creates GitHub release |

### Driver Repository Behavior

| Workflow | Driver Behavior |
|----------|-----------------|
| build_and_test | ✅ Build + Test, ready for deployment |
| release_pr | ✅ Validates release candidate |
| pr_closed | ✅ **DEPLOYS TO NUGET** |
| template_release | ⚠️ Warns this is for drivers, not templates |

---

## Converting Template to Driver

To test driver workflows in your template:

1. **Edit `.repo-config`:**
   ```json
   {
     "repository": {
       "type": "driver",              // ← Change
       "deployment_enabled": true     // ← Change
     },
     "driver": {
       "name": "N6705cDriver"         // ← Update
     }
   }
   ```

2. **Commit and push:**
   ```powershell
   git add .repo-config
   git commit -m "Convert to driver for testing"
   git push origin main
   ```

3. **Now test release workflows:**
   - `build_and_test` will show "DRIVER BUILD COMPLETE"
   - `release_pr` will validate as a driver
   - `pr_closed` will attempt deployment (configure secrets first!)

---

## Checking Workflow Results

### GitHub Actions Tab
```
https://github.com/your-org/DcPwr_template/actions
```

### View Logs
1. Click on any workflow run
2. Click on job name (e.g., "build-and-test")
3. Expand steps to see detailed output

### Look For These Key Messages

**Template Mode:**
- ✅ "Repository Type: template"
- ✅ "NuGet deployment SKIPPED (template mode)"
- ✅ "TEMPLATE BUILD COMPLETE"

**Driver Mode:**
- ✅ "Repository Type: driver"
- ✅ "Deployment Enabled: true"
- ✅ "DRIVER BUILD COMPLETE"

---

## Quick Test Commands

```powershell
# Test build_and_test.yml
git checkout -b feature/test
echo "# Test" >> README.md
git add .
git commit -m "Test build workflow"
git push origin feature/test

# Test template_release.yml (recommended for templates)
git checkout main
git tag -a v1.0.0 -m "Test release"
git push origin v1.0.0

# Test release_pr.yml + pr_closed.yml
git checkout -b Release_2026-04-15
git push origin Release_2026-04-15
# Create PR via GitHub UI
# Merge PR to trigger pr_closed.yml

# Check workflow status
# Visit: https://github.com/your-org/DcPwr_template/actions
```

---

## Common Issues

### "workflow not found"
- **Cause:** Workflow files not in `.github/workflows/` directory
- **Fix:** Ensure all `.yml` files are in correct location

### "Repository Type: null"
- **Cause:** `.repo-config` file missing or invalid JSON
- **Fix:** Validate JSON syntax: `jq '.' .repo-config`

### Workflow doesn't trigger
- **Cause:** Incorrect branch name or trigger condition
- **Fix:** Check `on:` section in workflow file

### Deployment skipped
- **Expected for templates:** They shouldn't deploy to NuGet
- **For drivers:** Check `deployment_enabled: true` in `.repo-config`

---

## Next Steps

1. ✅ Test all four workflows locally
2. ✅ Verify template mode skips deployments
3. ✅ Convert to driver and test deployment workflows
4. ✅ Configure NuGet secrets for actual deployment
5. ✅ Document your specific workflow preferences

**Happy Testing! 🚀**
