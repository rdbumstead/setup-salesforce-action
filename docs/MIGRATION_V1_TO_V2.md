# Migration Guide: v1 to v2

Complete guide for migrating from v1 to v2 of the Setup Salesforce CLI Action.

## 🎯 Executive Summary

**v2 is 100% backward compatible with v1.** Simply changing `@v1` to `@v2` will work immediately with no other changes required.

**Migration Time:** 5-10 minutes  
**Risk Level:** Minimal  
**Breaking Changes:** None

---

## ✅ Breaking Changes

**None!** v2 maintains complete backward compatibility with v1.

All v1 workflows will continue to work without modification after upgrading to v2.

---

## 🎉 What's New in v2

### 1. Cross-Platform Support 🌐

Run on Windows and macOS in addition to Linux:

```yaml
# Works on any platform now!
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: rdbumstead/setup-salesforce-action@v2
```

**Why this matters:**

- Test on your team's primary OS
- Windows-specific tooling support
- Mac-native development workflows
- Better CI/CD coverage

### 2. Custom Salesforce CLI Plugin Installation 🔌

Install any SF CLI plugin you need:

```yaml
- uses: rdbumstead/setup-salesforce-action@v2
  with:
    custom_sf_plugins: "sfdx-hardis,@salesforce/plugin-packaging"
```

**Why this matters:**

- No more manual plugin installation steps
- Cleaner workflows
- Automatic plugin caching
- Support for any npm-published SF CLI plugin

### 3. Enhanced Multi-Directory Support 📂

New `source_flags` output for seamless multi-directory deployments:

```yaml
- id: setup
  uses: rdbumstead/setup-salesforce-action@v2
  with:
    source_dirs: "force-app,packages/core,packages/utils"

- run: sf project deploy start ${{ steps.setup.outputs.source_flags }}
```

**Why this matters:**

- Automatic flag generation for complex projects
- No more manual `--source-dir` repetition
- Cleaner deployment commands

### 4. Automatic Directory Creation 🛠️

No more errors when `force-app` doesn't exist:

```yaml
# v2 automatically creates missing default directories
# Useful for testing and edge cases
```

**Why this matters:**

- Fewer workflow failures in test scenarios
- Better handling of non-standard project structures
- Graceful fallbacks

### 5. Better Developer Experience 💪

- **Enhanced error messages** with actionable guidance
- **Network retry logic** (3 attempts for resilience)
- **Stricter input validation** catches issues earlier
- **Platform-specific optimizations** for each OS

---

## 📋 Migration Steps

### Step 1: Update Workflow Files

**Find and replace** in all `.github/workflows/*.yml` files:

```bash
# Using sed (Linux/macOS)
find .github/workflows -type f -name "*.yml" -exec sed -i 's/@v1/@v2/g' {} +

# Using PowerShell (Windows)
Get-ChildItem .github/workflows -Filter *.yml | ForEach-Object {
    (Get-Content $_.FullName) -replace '@v1', '@v2' | Set-Content $_.FullName
}

# Manual method
# Open each workflow file and change:
uses: rdbumstead/setup-salesforce-action@v1
# to:
uses: rdbumstead/setup-salesforce-action@v2
```

### Step 2: Test on Non-Production Branch

```bash
# Create test branch
git checkout -b upgrade-to-v2

# Commit changes
git add .github/workflows/
git commit -m "chore: upgrade setup-salesforce-action to v2"

# Push and create PR
git push origin upgrade-to-v2
```

### Step 3: Verify Workflows Pass

- Check that all workflow runs complete successfully
- Verify authentication still works
- Confirm deployments function as before
- Review action logs for any warnings

### Step 4: Merge to Production

```bash
# After successful testing
git checkout main
git merge upgrade-to-v2
git push origin main
```

---

## 🔄 Migration Scenarios

### Scenario 1: Basic Setup (No Changes Needed)

**Before (v1):**

```yaml
- uses: rdbumstead/setup-salesforce-action@v1
  with:
    jwt_key: ${{ secrets.SFDX_JWT_KEY }}
    client_id: ${{ secrets.SFDX_CLIENT_ID }}
    username: ${{ vars.SFDX_USERNAME }}
```

**After (v2):**

```yaml
- uses: rdbumstead/setup-salesforce-action@v2
  with:
    jwt_key: ${{ secrets.SFDX_JWT_KEY }}
    client_id: ${{ secrets.SFDX_CLIENT_ID }}
    username: ${{ vars.SFDX_USERNAME }}
```

✅ **Result:** Works identically, now with cross-platform support!

---

### Scenario 2: With Plugins (No Changes Needed)

**Before (v1):**

```yaml
- uses: rdbumstead/setup-salesforce-action@v1
  with:
    jwt_key: ${{ secrets.SFDX_JWT_KEY }}
    client_id: ${{ secrets.SFDX_CLIENT_ID }}
    username: ${{ vars.SFDX_USERNAME }}
    install_delta: "true"
    install_scanner: "true"
```

**After (v2):**

```yaml
- uses: rdbumstead/setup-salesforce-action@v2
  with:
    jwt_key: ${{ secrets.SFDX_JWT_KEY }}
    client_id: ${{ secrets.SFDX_CLIENT_ID }}
    username: ${{ vars.SFDX_USERNAME }}
    install_delta: "true"
    install_scanner: "true"
```

✅ **Result:** Works identically, plugins now cached per platform!

---

### Scenario 3: Custom Plugin Installation (Optional Enhancement)

**Before (v1):**

```yaml
- uses: rdbumstead/setup-salesforce-action@v1
  with:
    jwt_key: ${{ secrets.SFDX_JWT_KEY }}
    client_id: ${{ secrets.SFDX_CLIENT_ID }}
    username: ${{ vars.SFDX_USERNAME }}

# Manual plugin installation
- name: Install Custom Plugin
  run: echo y | sf plugins install sfdx-hardis
```

**After (v2) - Enhanced:**

```yaml
- uses: rdbumstead/setup-salesforce-action@v2
  with:
    jwt_key: ${{ secrets.SFDX_JWT_KEY }}
    client_id: ${{ secrets.SFDX_CLIENT_ID }}
    username: ${{ vars.SFDX_USERNAME }}
    custom_sf_plugins: "sfdx-hardis" # Consolidated!
```

✅ **Result:** Cleaner workflow, automatic caching of custom plugins

---

### Scenario 4: Multi-Directory Projects (Optional Enhancement)

**Before (v1):**

```yaml
- uses: rdbumstead/setup-salesforce-action@v1
  with:
    jwt_key: ${{ secrets.SFDX_JWT_KEY }}
    client_id: ${{ secrets.SFDX_CLIENT_ID }}
    username: ${{ vars.SFDX_USERNAME }}

- name: Deploy
  run: |
    sf project deploy start \
      --source-dir force-app \
      --source-dir packages/core \
      --source-dir packages/utils
```

**After (v2) - Enhanced:**

```yaml
- id: setup
  uses: rdbumstead/setup-salesforce-action@v2
  with:
    jwt_key: ${{ secrets.SFDX_JWT_KEY }}
    client_id: ${{ secrets.SFDX_CLIENT_ID }}
    username: ${{ vars.SFDX_USERNAME }}
    source_dirs: "force-app,packages/core,packages/utils"

- name: Deploy
  run: sf project deploy start ${{ steps.setup.outputs.source_flags }}
```

✅ **Result:** Simpler, more maintainable deployment commands

---

### Scenario 5: Cross-Platform Testing (New Capability)

**After (v2) - New:**

```yaml
jobs:
  test:
    name: Test on ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]

    steps:
      - uses: actions/checkout@v4

      - uses: rdbumstead/setup-salesforce-action@v2
        with:
          jwt_key: ${{ secrets.SFDX_JWT_KEY }}
          client_id: ${{ secrets.SFDX_CLIENT_ID }}
          username: ${{ vars.SFDX_USERNAME }}

      - name: Run Tests
        run: npm test
```

✅ **Result:** Comprehensive cross-platform testing coverage

---

## 🧪 Testing Checklist

Before merging your v2 migration:

### Basic Functionality

- [ ] Workflows trigger correctly
- [ ] CLI installs successfully
- [ ] Authentication completes
- [ ] Deployments execute
- [ ] Tests run as expected

### Plugin Testing

- [ ] Built-in plugins install (delta, scanner, etc.)
- [ ] Custom plugins install (if using `custom_sf_plugins`)
- [ ] All plugins are cached properly

### Output Validation

- [ ] `org_id` output is populated
- [ ] `org_type` output is correct
- [ ] `org_edition` output is accurate
- [ ] `source_flags` output is correct (if using `source_dirs`)

### Error Handling

- [ ] Missing inputs produce clear errors
- [ ] Authentication failures are reported
- [ ] Network issues trigger retries

### Platform Testing (Optional)

- [ ] Test on Ubuntu (Linux)
- [ ] Test on Windows (if your team uses Windows)
- [ ] Test on macOS (if your team uses Mac)

---

## 🔧 Troubleshooting Migration Issues

### Issue: Workflows Not Triggering

**Cause:** Syntax error in workflow file

**Solution:**

```bash
# Validate YAML syntax
yamllint .github/workflows/*.yml

# Or use GitHub's workflow validator
# Commit and push - GitHub will show syntax errors
```

---

### Issue: "Invalid Action Version"

**Cause:** Typo in version tag

**Solution:**

```yaml
# Correct
uses: rdbumstead/setup-salesforce-action@v2

# Incorrect
uses: rdbumstead/setup-salesforce-action@2.0.0  # Don't use full version
uses: rdbumstead/setup-salesforce-action@v2.0   # Use @v2 for latest v2.x
```

---

### Issue: Cache Not Working After Upgrade

**Cause:** Cache keys changed between versions

**Solution:**

- This is expected on first run after upgrade
- Cache will rebuild automatically
- Subsequent runs will be fast again
- No action needed

---

### Issue: Custom Plugins Not Installing

**Cause:** Plugin name incorrect or not available

**Solution:**

```yaml
# Verify plugin exists on npm
# https://www.npmjs.com/package/your-plugin-name

# Use exact name from npm
custom_sf_plugins: "sfdx-hardis"  # Correct
custom_sf_plugins: "hardis"       # Incorrect
```

---

## 📊 Feature Comparison: v1 vs v2

| Feature                 | v1       | v2       | Notes                                 |
| ----------------------- | -------- | -------- | ------------------------------------- |
| **Platform Support**    |          |          |                                       |
| Linux (Ubuntu)          | ✅       | ✅       | Fastest, most cost-effective          |
| Windows                 | ❌       | ✅       | Full PowerShell and Bash support      |
| macOS                   | ❌       | ✅       | Native Mac environment                |
| **Core Features**       |          |          |                                       |
| JWT Authentication      | ✅       | ✅       | Unchanged                             |
| CLI Installation        | ✅       | ✅       | Unchanged                             |
| Node Version Control    | ✅       | ✅       | Unchanged                             |
| Smart Caching           | ✅       | ✅       | Enhanced with platform-specific paths |
| **Plugin Management**   |          |          |                                       |
| sfdx-git-delta          | ✅       | ✅       | Unchanged                             |
| Code Analyzer           | ✅       | ✅       | Unchanged                             |
| Custom SF Plugins       | ❌       | ✅       | **New in v2**                         |
| **Development Tools**   |          |          |                                       |
| Prettier                | ✅       | ✅       | Unchanged                             |
| ESLint                  | ✅       | ✅       | Unchanged                             |
| LWC Jest                | ✅       | ✅       | Unchanged                             |
| **Advanced Features**   |          |          |                                       |
| Multi-Directory Support | ⚠️       | ✅       | Manual → Automatic                    |
| source_flags Output     | ❌       | ✅       | **New in v2**                         |
| Auto Directory Creation | ❌       | ✅       | **New in v2**                         |
| Network Retry Logic     | ⚠️       | ✅       | Enhanced (3 attempts)                 |
| **Error Handling**      |          |          |                                       |
| Input Validation        | ✅       | ✅       | Enhanced messages in v2               |
| Error Messages          | ⚠️       | ✅       | More actionable in v2                 |
| Strict Mode             | ❌       | ✅       | **New in v2**                         |
| **Performance**         |          |          |                                       |
| Setup Time (cached)     | 20-45s   | 20-45s   | Maintained                            |
| Setup Time (first run)  | 1.5-2.5m | 1.5-2.5m | Maintained                            |
| Cache Hit Rate          | ~95%     | >95%     | Slightly improved                     |

✅ = Supported | ⚠️ = Limited/Manual | ❌ = Not Available

---

## 🎯 Rollback Plan

If you need to rollback to v1 for any reason:

```yaml
# Change back to v1
uses: rdbumstead/setup-salesforce-action@v1

# Commit and push
git add .github/workflows/
git commit -m "chore: rollback to v1"
git push
```

v1 will continue to be maintained for critical security updates, but new features will only be added to v2.

---

## 🚀 Post-Migration Enhancements

After successfully migrating, consider these v2-specific enhancements:

### 1. Add Cross-Platform CI

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    runs-on: ${{ matrix.os }}
```

### 2. Consolidate Plugin Installation

Move manual plugin steps into the action:

```yaml
custom_sf_plugins: "sfdx-hardis,@salesforce/plugin-packaging"
```

### 3. Simplify Multi-Directory Deployments

Use the new `source_flags` output:

```yaml
source_dirs: "force-app,packages/core"
# Then use: ${{ steps.setup.outputs.source_flags }}
```

### 4. Enable Strict Mode

Catch errors earlier:

```yaml
strict: "true"
```

---

## 📚 Additional Resources

- 📖 [README.md](../README.md) - Complete documentation
- ⬆️ [UPGRADE.md](UPGRADE.md) - Quick upgrade reference
- 🖥️ [PLATFORM_SUPPORT.md](PLATFORM_SUPPORT.md) - Platform-specific details
- 🔧 [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Common issues
- 📋 [CHANGELOG.md](../CHANGELOG.md) - Complete version history

### Salesforce Platform Changes

**Note:** If you're setting up authentication on a Winter '25+ org, you'll need to enable External Client Apps before creating Connected Apps. See the [README.md](../README.md) setup guide for updated instructions.

---

## 💬 Support

Need help with migration?

- 🐞 [Report Issue](https://github.com/rdbumstead/setup-salesforce-action/issues)
- 💡 [Ask Question](https://github.com/rdbumstead/setup-salesforce-action/discussions)
- 📧 [Email Support](mailto:ryan@ryanbumstead.com)

---

**Migration Time:** 5-10 minutes  
**Difficulty:** Easy (find and replace)  
**Breaking Changes:** None  
**Recommendation:** Migrate at your earliest convenience to benefit from v2 features

Happy migrating! 🚀
