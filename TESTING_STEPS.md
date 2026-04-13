# Step-by-Step CI/CD Testing Guide

Follow these steps to test your DcPwr_template CI/CD workflows.

---

## Prerequisites

Before you start, make sure you have:
- ✅ Git installed and configured
- ✅ GitHub repository created (e.g., `your-org/DcPwr_template`)
- ✅ PowerShell terminal open in `c:\DcPwr_template`

---

## Step 1: Initial Setup and Push to GitHub

```powershell
# Step 1.1: Check current directory
cd c:\DcPwr_template
pwd  # Should show: C:\DcPwr_template

# Step 1.2: Verify .repo-config is set to "template"
Get-Content .repo-config | Select-String "type"
# Should show: "type": "template"

# Step 1.3: Initialize git (if not already done)
git init
git branch -M main

# Step 1.4: Stage all files
git add .

# Step 1.5: Create initial commit
git commit -m "Initial template setup with CI/CD workflows"

# Step 1.6: Add your GitHub remote (REPLACE with your repo URL)
git remote add origin https://github.com/YOUR-ORG/DcPwr_template.git

# Step 1.7: Push to GitHub
git push -u origin main
```

**Expected Result:**
- ✅ `build_and_test.yml` workflow triggers automatically
- ✅ Go to `https://github.com/YOUR-ORG/DcPwr_template/actions`
- ✅ You should see "Build and Test" workflow running

**What to Check:**
1. Open GitHub Actions tab
2. Click on the "Build and Test" workflow run
3. Look for:
   - "Repository Type: template"
   - "TEMPLATE BUILD COMPLETE"
   - "✓ Build successful"
   - "⚠️ NuGet deployment SKIPPED (template mode)"

---

## Step 2: Test Build and Test Workflow (Feature Branch)

```powershell
# Step 2.1: Create a feature branch
git checkout -b feature/test-build-workflow

# Step 2.2: Make a small change to test the workflow
echo "`n// Test feature branch workflow" >> Driver-Stub/DCPwrDriver.cs

# Step 2.3: Commit the change
git add .
git commit -m "Test: Trigger build_and_test workflow"

# Step 2.4: Push feature branch
git push origin feature/test-build-workflow
```

**Expected Result:**
- ✅ `build_and_test.yml` workflow triggers again
- ✅ Builds and tests the feature branch
- ✅ Shows template mode (no deployment)

**What to Check:**
1. Go to GitHub Actions tab
2. You should see TWO workflow runs now:
   - One for `main` branch (from Step 1)
   - One for `feature/test-build-workflow` branch (from Step 2)
3. Both should show "TEMPLATE BUILD COMPLETE"

---

## Step 3: Test Template Release Workflow (Version Tag)

```powershell
# Step 3.1: Switch back to main branch
git checkout main

# Step 3.2: Pull latest changes (if any)
git pull origin main

# Step 3.3: Create a version tag (THIS IS THE TEMPLATE RELEASE METHOD)
git tag -a v1.0.0 -m "Template release v1.0.0 - Initial stable version"

# Step 3.4: Push the tag to GitHub
git push origin v1.0.0
```

**Expected Result:**
- ✅ `template_release.yml` workflow triggers
- ✅ Creates a GitHub Release automatically
- ✅ Validates build and tests
- ✅ NO NuGet deployment (templates don't deploy)

**What to Check:**
1. Go to GitHub Actions tab → Look for "Template Release (Tag)" workflow
2. Go to Releases: `https://github.com/YOUR-ORG/DcPwr_template/releases`
3. You should see release `v1.0.0` with:
   - Release title: "Template Release v1.0.0"
   - Release notes with usage instructions
   - Source code downloads
4. Click into the release to see full documentation

---

## Step 4: Verify Release_* Workflows DO NOT Trigger (Template Behavior)

```powershell
# Step 4.1: Create a Release branch (just to test behavior)
git checkout -b Release_2026-04-15

# Step 4.2: Push the Release branch
git push origin Release_2026-04-15
```

**Expected Result:**
- ✅ `build_and_test.yml` runs (because it triggers on all branches)
- ❌ `release_pr.yml` does NOT run (only triggers on PRs to Release_*)
- ❌ `pr_closed.yml` does NOT run (only triggers on merged PRs)

**What to Check:**
1. Go to GitHub Actions tab
2. Only "Build and Test" workflow runs
3. No "Release PR" or "PR Closed" workflows (they need PRs, not just branch pushes)

---

## Step 5: Test Release PR Workflow (Optional - Shows Dormant Behavior)

```powershell
# This step is done via GitHub UI, not PowerShell

# Step 5.1: Go to your GitHub repository
# Step 5.2: Click "Pull Requests" → "New Pull Request"
# Step 5.3: Base: Release_2026-04-15 ← Compare: feature/test-build-workflow
# Step 5.4: Create the Pull Request
```

**Expected Result:**
- ✅ `release_pr.yml` workflow triggers
- ⚠️ Shows warning: "Templates should create release TAGS, not release branches"
- ✅ Still runs build and test for validation
- ✅ Generates release notes preview

**What to Check:**
1. GitHub Actions tab → "Release PR Workflow"
2. Look for warning messages about using tags instead
3. Build and tests still pass
4. **DO NOT MERGE THIS PR** (it's just for testing)

---

## Step 6: Local Build Verification

```powershell
# Step 6.1: Switch back to main
git checkout main

# Step 6.2: Restore dependencies
dotnet restore

# Step 6.3: Build the solution
dotnet build --configuration Release

# Step 6.4: Run tests
dotnet test --configuration Release
```

**Expected Local Output:**
```
Build succeeded.
    0 Warning(s)
    0 Error(s)

Test run for DCPwrDriverStub.Tests.dll
Passed!  - Failed:     0, Passed:     7, Skipped:     0, Total:     7
```

---

## Step 7: View All Workflow Results

```powershell
# Open GitHub Actions page in browser
start "https://github.com/YOUR-ORG/DcPwr_template/actions"

# Open GitHub Releases page in browser
start "https://github.com/YOUR-ORG/DcPwr_template/releases"
```

**What You Should See:**

### GitHub Actions Tab:
- ✅ Multiple "Build and Test" runs (from main, feature branch, release branch)
- ✅ One "Template Release (Tag)" run (from v1.0.0 tag)
- ✅ One "Release PR Workflow" run (if you did Step 5)
- ✅ All showing green checkmarks (passed)

### GitHub Releases Tab:
- ✅ Release `v1.0.0` 
- ✅ Full release notes
- ✅ Usage instructions
- ✅ Source code downloads

---

## Summary: Template Workflow Behavior

| Workflow | Trigger | Template Behavior | Result |
|----------|---------|-------------------|--------|
| **build_and_test.yml** | Push to any branch | ✅ RUNS | Build + Test, NO deployment |
| **template_release.yml** | Tag `v*.*.*` | ✅ RUNS | Creates GitHub Release |
| **release_pr.yml** | PR to `Release_*` | 📦 DORMANT | Won't trigger (no PRs to Release_*) |
| **pr_closed.yml** | Merge PR to `Release_*` | 📦 DORMANT | Won't trigger (no merged PRs) |

**Key Points:**
- ✅ Templates use TAGS for releases (`git tag v1.0.0`)
- ✅ `release_pr.yml` and `pr_closed.yml` are included but unused
- ✅ When someone creates a DRIVER from this template, all 4 workflows become active

---

## Testing Driver Behavior (Optional)

Want to test how this works when converted to a driver?

```powershell
# Step D1: Edit .repo-config
code .repo-config
# Change: "type": "template" → "driver"
# Change: "deployment_enabled": false → true

# Step D2: Commit the change
git add .repo-config
git commit -m "Convert to driver for testing"
git push origin main

# Step D3: Now test Release_* workflows
git checkout -b Release_2026-04-20
git push origin Release_2026-04-20

# Step D4: Create PR via GitHub UI: main → Release_2026-04-20
# Step D5: Merge the PR

# Expected: pr_closed.yml runs and attempts deployment!
```

---

## Troubleshooting

### Workflow doesn't appear in Actions tab
- Check: `.github/workflows/` directory exists
- Check: Workflow files are valid YAML
- Check: You pushed to GitHub (not just committed locally)

### "Build and Test" fails
```powershell
# Run locally to see error
dotnet build --configuration Release
dotnet test --configuration Release
```

### Can't see workflow output
- Go to Actions tab
- Click on workflow run name
- Click on job name (e.g., "build-and-test")
- Expand each step to see logs

### Want to test again with fresh tag
```powershell
# Delete tag locally and remotely
git tag -d v1.0.0
git push origin :refs/tags/v1.0.0

# Create new tag
git tag -a v1.0.1 -m "Template release v1.0.1"
git push origin v1.0.1
```

---

## Next Steps

After successful testing:

1. ✅ Keep `.repo-config` as `"type": "template"`
2. ✅ Document your template version in README
3. ✅ Use this template to create actual drivers:
   ```powershell
   git clone --branch v1.0.0 https://github.com/YOUR-ORG/DcPwr_template N6705C-Driver
   cd N6705C-Driver
   # Edit .repo-config to activate driver mode
   ```

4. ✅ When creating drivers, all 4 workflows will be ready to use!

---

**Happy Testing! 🚀**
