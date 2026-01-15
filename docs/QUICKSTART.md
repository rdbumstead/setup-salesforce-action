# Quick Start Guide

Get your Salesforce CI/CD pipeline running in **under 15 minutes**!

## 🎯 What You'll Build

A complete CI/CD pipeline with:

- ✅ Automated setup of Salesforce CLI
- ✅ Cross-platform compatibility (Linux, Windows, macOS)
- ✅ Code quality and testing tools included

---

## 📋 Prerequisites

- [ ] GitHub repository with Salesforce code
- [ ] Salesforce org(s) with Connected App
- [ ] JWT private key for authentication

---

## 🚀 Step 1: Setup Salesforce Connected App (5 minutes)

### 1.1 Enable External Client Apps (Winter '25+)

**Important:** As of Winter '25, you must enable this setting first.

1. Setup → Quick Find → Search **"External Client Apps"**
2. Click **Settings** (in External Client Apps section)
3. Toggle **ON** the setting **"Enable External Client Apps"**
4. Click **Save**

**Note:** This is a one-time org-level setting. Skip if already enabled or on pre-Winter '25 org.

### 1.2 Create Connected App in Salesforce

1. Setup → App Manager → **New Connected App**
2. Fill in basic info:
   - Connected App Name: `GitHub Actions CI`
   - Contact Email: your@email.com
3. Enable OAuth Settings:
   - ✅ Enable OAuth Settings
   - ✅ Use digital signatures
   - Upload certificate (you'll generate this next)
   - OAuth Scopes: `api`, `refresh_token`, `offline_access`
4. **Save** and note the **Consumer Key** (Client ID)
5. Wait 2-10 minutes for propagation

### 1.3 Generate JWT Key Pair

```bash
# Generate private key
openssl genrsa -out server.key 2048

# Generate certificate
openssl req -new -x509 -key server.key -out server.crt -days 3650

# server.crt → Upload to Connected App
# server.key → Store as GitHub Secret (next step)
```

---

## 🔐 Step 2: Configure GitHub Secrets (2 minutes)

### Go to: Repository → Settings → Secrets and variables → Actions

#### Create Repository Secrets:

```
SFDX_JWT_KEY         → Contents of server.key file
SFDX_CLIENT_ID       → Consumer Key from Connected App
```

#### Create Repository Variables:

```
VALIDATION_USERNAME  → Your Salesforce username
PROD_USERNAME        → Production org username (if different)
UAT_USERNAME         → UAT org username (if different)
INT_USERNAME         → Integration org username (if different)
```

---

## 📝 Step 3: Add Workflow Files (3 minutes)

### 3.1 Create Directory Structure

```bash
mkdir -p .github/workflows
```

### 3.2 Create a Workflow File

Create `.github/workflows/test-salesforce.yml` with the following content:

```yaml
name: Validate Salesforce
on: [push, pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Salesforce
        uses: rdbumstead/setup-salesforce-action@v2
        with:
          jwt_key: ${{ secrets.SFDX_JWT_KEY }}
          client_id: ${{ secrets.SFDX_CLIENT_ID }}
          username: ${{ vars.SFDX_USERNAME }}
          install_scanner: "true"
          install_prettier: "true"

      - name: Run Tests
        run: sf project deploy validate --source-dir force-app
```

---

---

## 🧪 Step 4: Test Your Setup (2 minutes)

### 5.1 Create a Test PR

```bash
# Create a test branch
git checkout -b test-cicd

# Make a small change
echo "// Test change" >> force-app/main/default/classes/SomeClass.cls

# Commit and push
git add .
git commit -m "test: CI/CD pipeline"
git push origin test-cicd
```

### 5.2 Open Pull Request

1. Go to GitHub → Pull Requests → New Pull Request
2. Base: `main` (or your default branch)
3. Compare: `test-cicd`
4. Create Pull Request

### 5.3 Watch the Magic! ✨

You should see:

- ✅ Salesforce CLI setup
- ✅ Authentication success
- ✅ Verification command running

---

## 🎉 You're Done!

Your Salesforce CI/CD pipeline is now active!

---

## 🔧 Customization Options

### Change Source Directories

If your Salesforce code isn't in `force-app`:

```yaml
- uses: rdbumstead/setup-salesforce-action@v2
  with:
    source_dirs: "src,packages/core"
```

### Add Custom Plugins

If you need custom SF CLI plugins:

```yaml
- uses: rdbumstead/setup-salesforce-action@v2
  with:
    custom_sf_plugins: "sfdx-hardis,your-plugin"
```

### Use Different Platforms

Run on Windows or macOS:

```yaml
jobs:
  test:
    runs-on: windows-latest # or macos-latest
    steps:
      - uses: rdbumstead/setup-salesforce-action@v2
        # Works on all platforms!
```

---

## 🛠️ Troubleshooting

### "Authentication Failed"

**Check:**

- ✅ JWT key copied correctly (no line breaks)
- ✅ Client ID matches Connected App
- ✅ Username is correct
- ✅ Certificate uploaded to Connected App

**Fix:**

```bash
# Verify JWT key format
cat server.key | head -1
# Should show: -----BEGIN RSA PRIVATE KEY-----
```

### "No Changes Detected"

This is normal if:

- Your PR has no Salesforce metadata changes
- You only modified docs/tests

### "Code Analysis Failed"

**Check:**

- Review the code analysis output
- Fix any violations
- Adjust `severity_threshold` if needed

### Platform-Specific Issues

See [PLATFORM_SUPPORT.md](PLATFORM_SUPPORT.md) and [TROUBLESHOOTING.md](TROUBLESHOOTING.md) for detailed help.

---

## 📚 Next Steps

### Learn More:

- 📖 [Full Documentation](../README.md)
- 📄 [Upgrade from V1](UPGRADE.md) - If migrating from v1
- 🔄 [Migration Guide](MIGRATION_V1_TO_V2.md) - Detailed migration info
- 🖥️ [Platform Support](PLATFORM_SUPPORT.md) - Cross-platform details
- 🧪 [Testing Strategy](TESTING_STRATEGY.md) - Testing approach

### Add More Features:

1. **Cross-Platform Testing**
   - Test on Windows, macOS, Linux
   - Matrix builds for comprehensive coverage
   - See [PLATFORM_SUPPORT.md](PLATFORM_SUPPORT.md)

---

## ✅ Success Checklist

After setup, verify:

- [ ] PR creates and runs automatically
- [ ] Action installs CLI successfully
- [ ] Authentication works
- [ ] Validation command runs

---

## 💡 Pro Tips

### 1. Use Branch Protection Rules

Settings → Branches → Add rule:

- ✅ Require status checks to pass
- ✅ Require branches to be up to date
- ✅ Include administrators

### 2. Create .prettierrc

```json
{
  "printWidth": 120,
  "tabWidth": 2,
  "trailingComma": "none",
  "overrides": [
    {
      "files": "*.{cls,trigger}",
      "options": { "parser": "apex" }
    }
  ]
}
```

### 3. Create .eslintrc.json

```json
{
  "extends": ["@salesforce/eslint-config-lwc/recommended"]
}
```

### 4. Add package.json (for LWC tests)

```json
{
  "scripts": {
    "test": "sfdx-lwc-jest"
  },
  "devDependencies": {
    "@salesforce/sfdx-lwc-jest": "latest"
  }
}
```

### 5. Choose the Right Platform

- **Ubuntu**: Fastest, most cost-effective (recommended)
- **Windows**: If you need Windows-specific tooling **(expect 2-3x slower execution)**
- **macOS**: If your team uses Macs primarily

See [PLATFORM_SUPPORT.md](PLATFORM_SUPPORT.md) for details.

---

## 🆘 Get Help

- 🐞 [Report Issues](https://github.com/rdbumstead/setup-salesforce-action/issues)
- 💬 [Ask Questions](https://github.com/rdbumstead/setup-salesforce-action/discussions)
- 📖 [Documentation](../README.md)
- 🔧 [Troubleshooting Guide](TROUBLESHOOTING.md)

---

**Setup time:** ~15 minutes  
**Result:** Enterprise-grade CI/CD pipeline ✅  
**Difficulty:** Beginner-friendly 🟢  
**Platforms:** Linux, Windows, macOS 🌐

Happy deploying! 🚀
