# Upgrading from v1 to v2

Quick reference guide for upgrading to v2.

## 🎯 TL;DR

**Just change `@v1` to `@v2` - fully backward compatible!**

```yaml
# Before (v1)
- uses: rdbumstead/setup-salesforce-action@v1

# After (v2)
- uses: rdbumstead/setup-salesforce-action@v2
```

That's it! No other changes required.

---

## ✅ Zero Breaking Changes

v2 is **100% backward compatible** with v1. All existing workflows will continue to work without modification.

### What Stays the Same

- ✅ All v1 inputs work identically in v2
- ✅ All v1 outputs work identically in v2
- ✅ Authentication process unchanged
- ✅ Caching behavior improved but compatible
- ✅ Performance characteristics maintained
- ✅ Default behaviors unchanged

---

## 🎉 New Features You Can Use

Once you upgrade to v2, you can optionally take advantage of these new features:

### 1. Cross-Platform Support 🌐

**Run on Windows or macOS** in addition to Linux:

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: rdbumstead/setup-salesforce-action@v2
        # Works identically on all platforms!
```

### 2. Custom Salesforce CLI Plugins 🔌

**Install any SF CLI plugin** you need:

```yaml
- uses: rdbumstead/setup-salesforce-action@v2
  with:
    # New in v2
    custom_sf_plugins: "sfdx-hardis,@salesforce/plugin-packaging"
```

### 3. Multi-Directory Source Handling 📂

**Better support for complex project structures**:

```yaml
- id: setup
  uses: rdbumstead/setup-salesforce-action@v2
  with:
    source_dirs: "force-app,packages/core,packages/utils"

# New in v2: source_flags output
- run: sf project deploy start ${{ steps.setup.outputs.source_flags }}
```

### 4. Automatic Directory Creation 🛠️

**No more errors for missing directories**:

```yaml
# v2 automatically creates force-app if it doesn't exist
# Useful for testing and edge cases
```

### 5. Enhanced Error Messages 💬

**Better feedback when things go wrong**:

- More actionable error messages
- Clearer validation failures
- Platform-specific troubleshooting hints

---

## 📋 Migration Checklist

### For Existing Projects

- [ ] Change `@v1` to `@v2` in all workflow files
- [ ] Test on a non-production branch first
- [ ] Verify workflows run successfully
- [ ] (Optional) Add new v2 features as needed
- [ ] Update your documentation to reference v2

### Testing Your Migration

1. **Create a test branch**:

   ```bash
   git checkout -b upgrade-to-v2
   ```

2. **Update workflow files**:

   ```bash
   # Find and replace in all workflow files
   find .github/workflows -type f -name "*.yml" -exec sed -i 's/@v1/@v2/g' {} +
   ```

3. **Commit and push**:

   ```bash
   git add .github/workflows/
   git commit -m "chore: upgrade setup-salesforce-action to v2"
   git push origin upgrade-to-v2
   ```

4. **Open a PR and verify**:

   - All checks should pass as before
   - Review action logs for any warnings

5. **Merge when confident**:
   ```bash
   git checkout main
   git merge upgrade-to-v2
   ```

---

## 🆕 Optional Enhancements

After upgrading, consider these optional improvements:

### Add Custom Plugins

If you're manually installing plugins in later steps, consolidate them:

```yaml
# Before (v1)
- uses: rdbumstead/setup-salesforce-action@v1
- run: echo y | sf plugins install sfdx-hardis

# After (v2) - cleaner!
- uses: rdbumstead/setup-salesforce-action@v2
  with:
    custom_sf_plugins: "sfdx-hardis"
```

### Use Source Flags Output

If you have multiple source directories:

```yaml
# Before (v1)
- uses: rdbumstead/setup-salesforce-action@v1
- run: |
    sf project deploy start \
      --source-dir force-app \
      --source-dir packages/core \
      --source-dir packages/utils

# After (v2) - automatic!
- id: setup
  uses: rdbumstead/setup-salesforce-action@v2
  with:
    source_dirs: "force-app,packages/core,packages/utils"
- run: sf project deploy start ${{ steps.setup.outputs.source_flags }}
```

### Add Cross-Platform Testing

Expand your test coverage:

```yaml
# New in v2 - test on multiple platforms
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
runs-on: ${{ matrix.os }}
```

---

## 🔍 Version Comparison

| Feature                   | v1  | v2  |
| ------------------------- | --- | --- |
| Linux (Ubuntu) support    | ✅  | ✅  |
| Windows support           | ❌  | ✅  |
| macOS support             | ❌  | ✅  |
| JWT authentication        | ✅  | ✅  |
| Built-in plugins          | ✅  | ✅  |
| Custom plugins            | ❌  | ✅  |
| Multi-directory handling  | ⚠️  | ✅  |
| Auto directory creation   | ❌  | ✅  |
| source_flags output       | ❌  | ✅  |
| Enhanced error messages   | ⚠️  | ✅  |
| Platform-specific caching | ❌  | ✅  |
| Retry logic               | ⚠️  | ✅  |

✅ = Full support | ⚠️ = Partial/manual | ❌ = Not available

---

## ⚠️ Known Migration Issues

### None!

We haven't identified any breaking changes or migration issues. If you encounter any problems after upgrading, please [open an issue](https://github.com/rdbumstead/setup-salesforce-action/issues).

---

## 📚 Additional Resources

- 📖 [Full Migration Guide](MIGRATION_V1_TO_V2.md) - Detailed migration information
- 🌐 [Platform Support](PLATFORM_SUPPORT.md) - Cross-platform details
- 🔧 [Troubleshooting](TROUBLESHOOTING.md) - Common issues and solutions
- 📋 [Changelog](../CHANGELOG.md) - Complete version history

---

## 💬 Need Help?

- 🐞 [Report an issue](https://github.com/rdbumstead/setup-salesforce-action/issues)
- 💡 [Ask a question](https://github.com/rdbumstead/setup-salesforce-action/discussions)
- 📧 [Email support](mailto:ryan@ryanbumstead.com)

---

**Upgrade time:** < 5 minutes  
**Risk level:** Minimal (100% backward compatible)  
**Effort:** Find and replace `@v1` with `@v2`

Happy upgrading! 🚀
