# Migration Guide

Complete guide for migrating from v1 to v2 of the Setup Salesforce CLI Action.

## Executive Summary

**v2 is 100% backward compatible with v1.** Simply changing `@v1` to `@v2` will work immediately with no other changes required.

**Migration Time:** 5-10 minutes  
**Risk Level:** Minimal  
**Breaking Changes:** None

## What's New in v2

### 1. Cross-Platform Support 🌐

Run on Windows and macOS in addition to Linux:

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: rdbumstead/setup-salesforce-action@v2
```

### 2. Custom Salesforce CLI Plugin Installation 🔌

Install any SF CLI plugin you need:

```yaml
- uses: rdbumstead/setup-salesforce-action@v2
  with:
    custom_sf_plugins: "sfdx-hardis,@salesforce/plugin-packaging"
```

### 3. Enhanced Multi-Directory Support 📂

New `source_flags` output for seamless multi-directory deployments:

```yaml
- id: setup
  uses: rdbumstead/setup-salesforce-action@v2
  with:
    source_dirs: "force-app,packages/core,packages/utils"

- run: sf project deploy start ${{ steps.setup.outputs.source_flags }}
```

### 4. Automatic Directory Creation 🛠️

No more errors when `force-app` doesn't exist - v2 creates missing default directories automatically.

### 5. Better Developer Experience 💪

- Enhanced error messages with actionable guidance
- Network retry logic (3 attempts for resilience)
- Stricter input validation catches issues earlier
- Platform-specific optimizations for each OS

## Migration Steps

### Step 1: Update Workflow Files

```bash
# Using sed (Linux/macOS)
find .github/workflows -type f -name "*.yml" -exec sed -i 's/@v1/@v2/g' {} +

# Using PowerShell (Windows)
Get-ChildItem .github/workflows -Filter *.yml | ForEach-Object {
    (Get-Content $_.FullName) -replace '@v1', '@v2' | Set-Content $_.FullName
}
```

### Step 2: Test on Non-Production Branch

```bash
git checkout -b upgrade-to-v2
git add .github/workflows/
git commit -m "chore: upgrade setup-salesforce-action to v2"
git push origin upgrade-to-v2
```

### Step 3: Verify Workflows Pass

- Check that all workflow runs complete successfully
- Verify authentication still works
- Confirm deployments function as before

### Step 4: Merge to Production

```bash
git checkout main
git merge upgrade-to-v2
git push origin main
```

## Feature Comparison: v1 vs v2

| Feature                 | v1  | v2  | Notes                 |
| ----------------------- | --- | --- | --------------------- |
| **Platform Support**    |     |     |                       |
| Linux (Ubuntu)          | ✅  | ✅  | Fastest               |
| Windows                 | ❌  | ✅  | Full support          |
| macOS                   | ❌  | ✅  | Full support          |
| **Plugin Management**   |     |     |                       |
| sfdx-git-delta          | ✅  | ✅  | Unchanged             |
| Code Analyzer           | ✅  | ✅  | Unchanged             |
| Custom SF Plugins       | ❌  | ✅  | **New in v2**         |
| **Advanced Features**   |     |     |                       |
| Multi-Directory Support | ⚠️  | ✅  | Manual → Automatic    |
| source_flags Output     | ❌  | ✅  | **New in v2**         |
| Network Retry Logic     | ⚠️  | ✅  | Enhanced (3 attempts) |

✅ = Supported | ⚠️ = Limited/Manual | ❌ = Not Available

## Rollback Plan

If you need to rollback to v1:

```yaml
uses: rdbumstead/setup-salesforce-action@v1
```

---

## Support

Need help with migration?

- 🐞 [Report Issue](https://github.com/rdbumstead/setup-salesforce-action/issues)
- 💡 [Ask Question](https://github.com/rdbumstead/setup-salesforce-action/discussions)
- 📧 [Email Support](mailto:ryan@ryanbumstead.com)

---

**Happy migrating!** 🚀
