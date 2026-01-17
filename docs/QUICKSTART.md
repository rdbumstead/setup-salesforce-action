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

## 🚀 Step 1: Setup Salesforce External Client App (5 minutes)

### 1.1 Enable External Client Apps

**Required for all orgs:** As of the Winter '25 release (October 2024), Salesforce recommends using External Client Apps for CI/CD integrations.

1. Setup → Quick Find → Search **"External Client Apps"**
2. Click **Settings** (in External Client Apps section)
3. Toggle **ON** the setting **"Enable External Client Apps"**
4. Click **Save**

**Note:** This is a one-time org-level setting that enables the External Client Apps feature.

### 1.2 Generate JWT Key Pair

Generate your certificate and private key **before** creating the External Client App:

```bash
# Generate private key
openssl genrsa -out server.key 2048

# Generate certificate
openssl req -new -x509 -key server.key -out server.crt -days 3650

# When prompted, you can use any values for the certificate fields
# Common Name, Organization, etc. - these don't affect functionality
```

You now have:

- `server.crt` - Certificate to upload to Salesforce
- `server.key` - Private key to store as GitHub Secret

### 1.3 Create External Client App in Salesforce

1. Setup → Quick Find → Search **"External Client Apps"**
2. Click **New**
3. Fill in the basic information:

   - **External Client App Name**: `GitHub Actions CI`
   - **Contact Email**: your@email.com
   - **Description** (optional): `JWT authentication for GitHub Actions`

4. **Configure OAuth Settings**:
   - ✅ Check **"Enable OAuth Settings"**
   - **Callback URL**: `http://localhost:1717/OauthRedirect`
     - _Note: This URL is not used for JWT auth but is required by Salesforce_
5. **Enable JWT Bearer Flow**:
   - ✅ Check **"Use digital signatures"**
   - Click **Choose File** and upload your `server.crt` certificate
6. **Select OAuth Scopes**:
   - Add these scopes from "Available OAuth Scopes" to "Selected OAuth Scopes":
     - ✅ **Access and manage your data (api)**
     - ✅ **Perform requests on your behalf at any time (refresh_token, offline_access)**
7. Click **Save**

8. **Note your Consumer Key**:

   - After saving, you'll see the **Consumer Key** (also called Client ID)
   - Copy this - you'll need it for GitHub Secrets
   - The Consumer Secret is not needed for JWT authentication

9. **Wait 2-10 minutes** for the External Client App to propagate

### 1.4 Configure Pre-Authorization (Recommended)

To avoid manual authorization:

1. Return to your External Client App
2. Click **Manage**
3. Click **Edit Policies**
4. Under **OAuth Policies**:
   - **Permitted Users**: Select **"All users may self-authorize"**
   - **IP Relaxation**: Select **"Relax IP restrictions"** (for GitHub Actions)
5. Click **Save**

### Alternative: Using Traditional Connected Apps

If your org is on a pre-Winter '25 release, or you prefer the traditional approach, you can still use Connected Apps:

<details>
<summary>Click to expand: Connected App Instructions (Legacy)</summary>

1. Setup → App Manager → **New Connected App**
2. Fill in basic info:
   - Connected App Name: `GitHub Actions CI`
   - Contact Email: your@email.com
3. Enable OAuth Settings:
   - ✅ Enable OAuth Settings
   - Callback URL: `http://localhost:1717/OauthRedirect`
   - ✅ Use digital signatures
   - Upload your `server.crt` certificate
   - OAuth Scopes: **Access and manage your data (api)**, **Perform requests on your behalf at any time (refresh_token, offline_access)**
4. **Save** and note the **Consumer Key** (Client ID)
5. Wait 2-10 minutes for propagation

**Note:** Connected Apps and External Client Apps both work identically with this GitHub Action. The authentication flow and credentials are the same.

</details>

---

## 🔐 Step 2: Configure GitHub Secrets (2 minutes)

### Go to: Repository → Settings → Secrets and variables → Actions

#### Create Repository Secrets:

```
SFDX_JWT_KEY         → Contents of server.key file
SFDX_CLIENT_ID       → Consumer Key from External Client App
```

**How to get the JWT key contents:**

```bash
# View the entire private key
cat server.key

# Copy everything including the header and footer:
# -----BEGIN RSA PRIVATE KEY-----
# [multiple lines of encrypted key]
# -----END RSA PRIVATE KEY-----
```

**Important:** Copy the **entire** contents of `server.key` including the `BEGIN` and `END` lines.

#### Create Repository Variables:

```
SFDX_USERNAME        → Your Salesforce username (the org you authenticated)
```

**For multiple environments** (optional):

```
PROD_USERNAME        → Production org username
UAT_USERNAME         → UAT org username
INT_USERNAME         → Integration org username
```

---

## 🔍 Verification

After creating your External Client App and secrets, verify the setup works:

### Test Authentication Locally (Optional)

```bash
# Test JWT authentication with your new External Client App
sf org login jwt \
  --client-id YOUR_CONSUMER_KEY \
  --jwt-key-file server.key \
  --username your@email.com \
  --instance-url https://login.salesforce.com

# If successful, you'll see:
# Successfully authorized your@email.com with org ID 00D...
```

If this works locally, it will work in GitHub Actions!

---

## 💡 Pro Tips

### Security Best Practices

1. **Never commit** `server.key` to your repository
2. **Add to .gitignore**:
   ```bash
   echo "server.key" >> .gitignore
   echo "server.crt" >> .gitignore
   ```
3. **Store backups** of your certificate and key in a secure password manager
4. **Rotate keys** annually or when team members leave

### Troubleshooting Authentication

If authentication fails:

- ✅ Verify `server.crt` was uploaded to the External Client App
- ✅ Ensure the entire `server.key` content is in `SFDX_JWT_KEY` secret
- ✅ Check that Consumer Key matches `SFDX_CLIENT_ID`
- ✅ Confirm username is correct
- ✅ Wait the full 10 minutes after creating the External Client App
- ✅ For sandboxes, use `--instance-url https://test.salesforce.com`

### Multiple Orgs/Environments

You can use the **same External Client App and certificate** across multiple orgs:

1. Create the External Client App in each org (upload same `server.crt`)
2. Use the same `SFDX_JWT_KEY` secret
3. Create separate secrets/variables for each org's Consumer Key and Username:
   ```
   PROD_CLIENT_ID / PROD_USERNAME
   UAT_CLIENT_ID / UAT_USERNAME
   INT_CLIENT_ID / INT_USERNAME
   ```

Or use **one External Client App in production** and authenticate to other orgs using the same credentials (recommended for simplicity).

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

### Alternative Authentication Methods (v2.1+)

Don't want to set up JWT certificates? Use **SFDX Auth URL** instead:

```yaml
- name: Setup Salesforce
  uses: rdbumstead/setup-salesforce-action@v2
  with:
    auth_method: "sfdx-url"
    sfdx_auth_url: ${{ secrets.SFDX_AUTH_URL }}
```

**To get your SFDX Auth URL:**

```bash
sf org display --target-org YourOrg --verbose --json | jq -r '.result.sfdxAuthUrl'
```

Store this as a GitHub secret named `SFDX_AUTH_URL`.

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
- 🔄 [Migration Guide](MIGRATION.md) - Upgrade from v1 or v2
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
