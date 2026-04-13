# DC Power Driver Template

**Version:** 1.0.0  
**Type:** Template Repository  
**Purpose:** Starting point for creating DC Power instrument driver repositories

---

## Overview

This is a **GitHub template repository** for creating DC Power instrument drivers within the Viasat PTE CCS framework.

### What This Template Provides

✅ **Driver-Stub** - Skeleton driver implementation  
✅ **Test-Stub** - xUnit test framework and basic tests  
✅ **CI/CD Workflows** - GitHub Actions for validation and release  
✅ **Configuration** - `.repo-config` file for repository type detection  
✅ **.NET 8** - Modern .NET project structure

### What This Template Does NOT Do

❌ **Does NOT deploy to NuGet** (templates are for creation, not deployment)  
❌ **Does NOT contain actual driver implementation** (stub code only)  
❌ **Does NOT connect to real instruments** (placeholders only)

---

## Repository Structure

```
DCPwr_template/
├── .github/
│   └── workflows/
│       ├── build_and_test.yml    # Feature/main branch CI
│       ├── release_pr.yml        # Release PR validation
│       ├── pr_closed.yml         # Release deployment
│       └── template_release.yml  # Template tag releases
├── Driver-Stub/
│   ├── DCPwrDriver.cs            # Driver implementation stub
│   ├── DCPwrDriverStub.csproj    # Driver project file
│   └── README.md                 # Driver documentation
├── Test-Stub/
│   ├── DCPwrDriverTests.cs       # Test implementation stub
│   ├── DCPwrDriverStub.Tests.csproj # Test project file
│   └── README.md                 # Test documentation
├── .repo-config                  # Repository configuration (KEY FILE!)
├── .gitignore                    # Git ignore patterns
├── DCPwrTemplate.sln             # Visual Studio solution
└── README.md                     # This file
```

---

## CI/CD Workflows

### 1. Build and Test (`build_and_test.yml`)

**Triggers:** 
- Push to any branch (except gh-pages and Release_*)
- Pull requests to main/master

**What it does:**
- ✅ Detects repository type from `.repo-config`
- ✅ Restores NuGet packages
- ✅ Builds solution
- ✅ Runs tests
- ✅ Shows deployment status based on repo type

**For Templates:** Build + Test only (NO deployment)  
**For Drivers:** Build + Test + Ready for deployment

---

### 2. Release PR Validation (`release_pr.yml`)

**Triggers:** Pull request to `Release_*` branches

**What it does:**
- ✅ Validates repository configuration
- ✅ Builds release candidate
- ✅ Runs comprehensive test suite
- ✅ Validates version numbers
- ✅ Generates release notes preview
- ⚠️ Warns if template tries to use release branches

**Note:** Templates should use tags, drivers use release branches

---

### 3. Release Deployment (`pr_closed.yml`)

**Triggers:** When PR is merged to `Release_*` branch

**What it does (for drivers):**
- ✅ Builds production release
- ✅ Runs final tests
- ✅ Creates NuGet package
- ✅ Publishes to NuGet (when configured)
- ✅ Creates GitHub release

**For templates:** Skips deployment, suggests using tags instead

---

### 4. Template Release (`template_release.yml`)

**Triggers:** Push of version tags (v*.*.*)

**What it does:**
- ✅ Validates template configuration
- ✅ Runs full build and test suite
- ✅ Creates GitHub Release with tag
- ✅ Generates release notes
- ✅ Makes template available for driver creation
- ❌ NO NuGet deployment (templates aren't packages)

**Use this for:** Creating stable, versioned template releases

---

## Creating a Concrete Driver from This Template

### ⚠️ CRITICAL: Always use release tags, NEVER main branch!

```bash
# ✅ CORRECT - Use release tag
git clone --branch v1.0.0 https://github.com/your-org/DcPwr_template N6705C-Driver
cd N6705C-Driver

# ❌ WRONG - Don't use main branch
# git clone https://github.com/your-org/DcPwr_template  # Uses unstable main!
```

### Step-by-Step Activation Process

#### Step 1: Edit `.repo-config`

```json
{
  "repository": {
    "type": "driver",              // ← Change from "template"
    "deployment_enabled": true     // ← Change from false
  },
  "template": {
    "version": "1.0.0",            // Keep as reference
    "stable_tag": "v1.0.0",
    "instrument_class": "DCPwr",
    "base_namespace": "Viasat.PTE.CCS"
  },
  "driver": {
    "name": "N6705cDriver",        // ← Change to your driver name
    "initial_version": "1.0.0",
    "description": "Keysight N6705C DC Power Supply Driver"  // ← Update description
  },
  "autobot": {
    "component_script": "n6705cdriver.py",    // ← Update script name
    "build_key": "--n6705cdriver",            // ← Update build key
    "base_version": "1.0.0"
  }
}
```

#### Step 2: Rename and Update Projects

```bash
# Rename directories
mv Driver-Stub N6705C-Driver
mv Test-Stub N6705C-Tests

# Update .csproj files
# - Change RootNamespace
# - Change AssemblyName
# - Update project references

# Update .sln file
# - Update project paths
```

#### Step 3: Implement Driver Logic

```csharp
// In N6705C-Driver/N6705CDriver.cs
public class N6705CDriver : VisaInstrument, IDCPwr
{
    // Implement actual SCPI commands
    public void SetVoltage(double voltage)
    {
        Write($"VOLT {voltage}");
    }
    
    // ... more implementation
}
```

#### Step 4: Commit and Push

```bash
git add .
git commit -m "Activate N6705C driver from template v1.0.0"
git push origin main
```

#### Step 5: CI/CD Automatically Activates!

The workflow reads `.repo-config` and detects `"type": "driver"`, enabling:
- ✅ Full build and test
- ✅ NuGet package deployment (when configured)
- ✅ Release workflows

---

## Testing Locally

### Build the Template

```bash
# Restore dependencies
dotnet restore

# Build solution
dotnet build --configuration Release

# Run tests
dotnet test --configuration Release
```

### Expected Results

```
Build succeeded.
    0 Warning(s)
    0 Error(s)

Test run for DCPwrDriverStub.Tests.dll
Passed!  - Failed:     0, Passed:     7, Skipped:     0, Total:     7
```

---

## Key Configuration: `.repo-config`

This file is the **single source of truth** for repository behavior.

### Template Configuration

```json
{
  "repository": {
    "type": "template",           // ← Prevents NuGet deployment
    "deployment_enabled": false   // ← CI/CD validation only
  }
}
```

### Driver Configuration (After Activation)

```json
{
  "repository": {
    "type": "driver",             // ← Enables NuGet deployment
    "deployment_enabled": true    // ← Full CI/CD pipeline
  }
}
```

### Why This Approach Works

✅ No special permissions needed (driver developers can't use secrets/variables)  
✅ Visible in code (self-documenting)  
✅ Preserves template version metadata  
✅ Single source of truth for configuration  
✅ Git history shows exactly what changed

---

## Template Versioning

### Development Flow

```
feature branch → main (unstable) → Test → Release Tag (v1.0.0) → ✅ SAFE TO USE
```

### Release Strategy

- **main branch** - Unstable, active development
- **v1.0.0, v1.1.0, v1.2.0** - Stable release tags
- **Only release tags are safe for driver creation**

### Creating a Template Release

```bash
# After testing on main branch
git tag -a v1.0.0 -m "Release v1.0.0: Initial stable template"
git push origin v1.0.0

# template_release.yml workflow automatically creates the GitHub release
```

### Creating a Driver Release

```bash
# Create release branch
git checkout -b Release_2026-04-15

# Make final adjustments, update versions
git add .
git commit -m "Prepare release 2026-04-15"
git push origin Release_2026-04-15

# Create PR to Release_2026-04-15
# release_pr.yml validates the release

# After PR approval and merge
# pr_closed.yml automatically deploys to NuGet
```

---

## Differences: Template vs Driver

| Aspect | Template Repo (This) | Concrete Driver Repo |
|--------|---------------------|----------------------|
| **`.repo-config` type** | `"template"` | `"driver"` |
| **`.repo-config` deployment** | `false` | `true` |
| **Feature Builds** | Build + Test only | Build + Test + Deploy |
| **NuGet Deployment** | ❌ Never | ✅ Always |
| **Purpose** | Source for creation | Actual instrument driver |
| **Code** | Stubs/placeholders | Real implementation |

---

## Quick Reference

### Check Repository Type

```bash
jq -r '.repository.type' .repo-config
# Output: "template"
```

### Verify Configuration

```bash
jq '.' .repo-config
# Shows full configuration
```

### Build Commands

```bash
# Restore
dotnet restore

# Build
dotnet build --configuration Release

# Test
dotnet test --configuration Release

# Clean
dotnet clean
```

---

## Support and Documentation

- **Template Issues:** Report issues with template structure/CI/CD
- **Driver Questions:** Consult InstrumentFoundation documentation
- **CI/CD Help:** Review GitHub Actions workflow files

---

## Version History

### v1.0.0 (Current)
- Initial template release
- .NET 8 support
- Basic CI/CD workflows
- Driver and test stubs
- Repository type detection

---

## License

Copyright © 2026 Viasat, Inc. All rights reserved.
