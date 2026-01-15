# Setup Salesforce CLI Action

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Setup%20Salesforce%20CLI-blue.svg)](https://github.com/marketplace/actions/setup-salesforce-cli)
[![GitHub release](https://img.shields.io/github/v/release/rdbumstead/setup-salesforce-action)](https://github.com/rdbumstead/setup-salesforce-action/releases)
[![Test Action](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test.yml/badge.svg)](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test.yml)
[![Cross Platform Tests](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-cross-platform.yml/badge.svg)](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-cross-platform.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

> **📢 v2.0.0 Released!** Now with full Windows and macOS support!
> [Migration Guide](docs/MIGRATION_V1_TO_V2.md) | [What's New](#-whats-new-in-v2)

> Flexible, fast, and production-ready Salesforce CLI setup with JWT authentication and optional tooling.

Created by [Ryan Bumstead](https://github.com/rdbumstead) | [Portfolio](https://github.com/rdbumstead/salesforce-platform-architect-portfolio) | [Contact](mailto:ryan@ryanbumstead.com)

---

## 🎉 What's New in v2

### 🌐 Cross-Platform Support

Run your Salesforce CI/CD on **any platform**:

- ✅ **Ubuntu (Linux)** - Fastest, most cost-effective
- ✅ **Windows** - Native Windows tooling support, PowerShell & Bash
- ✅ **macOS** - Native Mac environment for Mac-first teams

### 🔌 Custom Plugin Support

Install any Salesforce CLI plugin you need:

```yaml
custom_sf_plugins: "sfdx-hardis,@salesforce/plugin-packaging"
```

### 📂 Enhanced Multi-Directory Support

New `source_flags` output for seamless multi-directory workflows:

```yaml
- id: setup
  uses: rdbumstead/setup-salesforce-action@v2
  with:
    source_dirs: "force-app,packages/core"
- run: sf project deploy start ${{ steps.setup.outputs.source_flags }}
```

### 💪 Better Developer Experience

- Automatic directory creation (no more missing folder errors)
- Improved error messages with actionable feedback
- Network retry logic (3 attempts for resilience)
- Stricter input validation
- Platform-specific optimizations

**Upgrading from v1?** It's seamless - just change `@v1` to `@v2`! See [Migration Guide](docs/MIGRATION_V1_TO_V2.md).

---

## ✨ Features

- 🚀 **Lightning Fast** - 20-45 seconds setup with intelligent caching (vs 2-3 min without)
- 🌐 **Cross-Platform** - Full support for Ubuntu, Windows, and macOS runners
- 🔐 **Secure JWT Auth** - Non-interactive, certificate-based authentication
- ⚙️ **Fully Optional** - Install only what you need, nothing more
- 🎯 **Flexible Config** - From minimal CLI-only to full-stack development
- 📦 **Smart Caching** - Automatic dependency caching across runs
- 🛠️ **Dev Tools** - Optional Prettier, ESLint, LWC Jest integration
- 🔌 **Plugin Ready** - Optional delta deployments and code analysis
- ⚡ **Zero Bloat** - Default setup is minimal and fast

---

## 🚀 Quick Start

### Minimal Setup (Just CLI + Auth)

```yaml
- name: Setup Salesforce
  uses: rdbumstead/setup-salesforce-action@v2
  with:
    jwt_key: ${{ secrets.SFDX_JWT_KEY }}
    client_id: ${{ secrets.SFDX_CLIENT_ID }}
    username: ${{ vars.SFDX_USERNAME }}
```

**Setup time:** ~20 seconds (cached) | ~1.5 minutes (first run)

---

### Recommended Setup (Delta + Scanner)

```yaml
- name: Setup Salesforce
  uses: rdbumstead/setup-salesforce-action@v2
  with:
    jwt_key: ${{ secrets.SFDX_JWT_KEY }}
    client_id: ${{ secrets.SFDX_CLIENT_ID }}
    username: ${{ vars.SFDX_USERNAME }}
    install_delta: "true"
    install_scanner: "true"
```

**Setup time:** ~35 seconds (cached) | ~2 minutes (first run)

---

### Full-Stack Development

```yaml
- name: Setup Salesforce
  uses: rdbumstead/setup-salesforce-action@v2
  with:
    jwt_key: ${{ secrets.SFDX_JWT_KEY }}
    client_id: ${{ secrets.SFDX_CLIENT_ID }}
    username: ${{ vars.SFDX_USERNAME }}
    install_delta: "true"
    install_scanner: "true"
    install_prettier: "true"
    install_eslint: "true"
    install_lwc_jest: "true"
```

**Setup time:** ~45 seconds (cached) | ~2.5 minutes (first run)

---

## 📋 Inputs

### Authentication Inputs

_Required unless `skip_auth: 'true'` is set_

| Name        | Description                                               |
| ----------- | --------------------------------------------------------- |
| `jwt_key`   | JWT private key for authentication (entire file contents) |
| `client_id` | Connected App consumer key                                |
| `username`  | Salesforce username                                       |

### Configuration

| Name           | Default     | Description                        |
| -------------- | ----------- | ---------------------------------- |
| `is_dev_hub`   | `false`     | Set as default Dev Hub             |
| `node_version` | `20`        | Node.js version to use             |
| `skip_auth`    | `false`     | Skip JWT authentication (CLI only) |
| `alias`        | `TargetOrg` | Alias name for authenticated org   |
| `source_dirs`  | `force-app` | Comma-separated source directories |
| `strict`       | `false`     | Fail on optional tool errors       |

### Salesforce Plugins

| Name                | Default | Description                                   |
| ------------------- | ------- | --------------------------------------------- |
| `install_scanner`   | `false` | Install @salesforce/plugin-code-analyzer      |
| `install_delta`     | `false` | Install sfdx-git-delta                        |
| `custom_sf_plugins` | `""`    | Comma-separated list of additional SF plugins |

### Development Tools

| Name               | Default | Description                             |
| ------------------ | ------- | --------------------------------------- |
| `install_prettier` | `false` | Install Prettier and Salesforce plugins |
| `install_eslint`   | `false` | Install ESLint and Salesforce plugins   |
| `install_lwc_jest` | `false` | Install sfdx-lwc-jest for LWC testing   |

### Outputs

| Name             | Description                                         |
| ---------------- | --------------------------------------------------- |
| `org_id`         | Authenticated Salesforce Org ID                     |
| `org_edition`    | Salesforce Edition (Developer, Enterprise, etc.)    |
| `org_type`       | Organization type (Production, Sandbox, Scratch)    |
| `username`       | Authenticated username                              |
| `instance_url`   | Org instance URL                                    |
| `sf_cli_version` | Installed Salesforce CLI version                    |
| `source_flags`   | Resolved source directory flags for SF CLI commands |

---

## 🛠️ Setup Guide

<details>
<summary><b>Step 1: Create Connected App in Salesforce</b></summary>

### 1.3 Additional Resources

- [JWT Auth Flow Documentation](https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_auth_jwt_flow.htm)
- [External Client Apps Settings](https://help.salesforce.com/s/articleView?id=release-notes.rn_security_external_client_apps.htm)

</details>

<details>
<summary><b>Step 2: Configure GitHub Secrets</b></summary>

Repository → Settings → Secrets and variables → Actions

**Add Secrets:**

`SFDX_JWT_KEY`: Paste entire contents of `server.key` including BEGIN/END lines

`SFDX_CLIENT_ID`: Paste Consumer Key from Salesforce (starts with 3MVG...)

**Add Variables:**

`SFDX_USERNAME`: Your Salesforce username (e.g., admin@company.com)

</details>

<details>
<summary><b>Step 3: Test Your Setup</b></summary>

Create `.github/workflows/test.yml`:

```yaml
name: Test Setup
on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Salesforce
        uses: rdbumstead/setup-salesforce-action@v2
        with:
          jwt_key: ${{ secrets.SFDX_JWT_KEY }}
          client_id: ${{ secrets.SFDX_CLIENT_ID }}
          username: ${{ vars.SFDX_USERNAME }}

      - name: Verify
        run: sf org display
```

</details>

---

## 🎯 Use Cases

<details>
<summary><b>Minimal Apex Deployment</b></summary>

```yaml
name: Deploy Apex

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Salesforce
        uses: rdbumstead/setup-salesforce-action@v2
        with:
          jwt_key: ${{ secrets.SFDX_JWT_KEY }}
          client_id: ${{ secrets.SFDX_CLIENT_ID }}
          username: ${{ vars.SFDX_USERNAME }}

      - name: Deploy
        run: sf project deploy start --source-dir force-app
```

</details>

<details>
<summary><b>Delta Deployment Pipeline</b></summary>

```yaml
name: Delta Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Salesforce
        uses: rdbumstead/setup-salesforce-action@v2
        with:
          jwt_key: ${{ secrets.SFDX_JWT_KEY }}
          client_id: ${{ secrets.SFDX_CLIENT_ID }}
          username: ${{ vars.SFDX_USERNAME }}
          install_delta: "true"

      - name: Generate Delta
        run: |
          sf sgd:source:delta --to HEAD --from HEAD^ --output changed-sources/

      - name: Deploy Delta
        run: |
          sf project deploy start \
            --manifest changed-sources/package/package.xml \
            --test-level RunLocalTests
```

</details>

<details>
<summary><b>Code Quality Check</b></summary>

```yaml
name: Quality Check

on:
  pull_request:
    branches: [main]

jobs:
  quality:
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

      - name: Check Formatting
        run: prettier --check "force-app/**/*.{cls,js,html}"

      - name: Scan Code
        run: sf scanner:run --target force-app --severity-threshold 2
```

</details>

<details>
<summary><b>LWC Development with Testing</b></summary>

```yaml
name: LWC Tests

on:
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Salesforce
        uses: rdbumstead/setup-salesforce-action@v2
        with:
          jwt_key: ${{ secrets.SFDX_JWT_KEY }}
          client_id: ${{ secrets.SFDX_CLIENT_ID }}
          username: ${{ vars.SFDX_USERNAME }}
          install_eslint: "true"
          install_lwc_jest: "true"

      - name: Lint JavaScript
        run: eslint force-app/**/lwc/**/*.js

      - name: Run Unit Tests
        run: npm run test:unit
```

</details>

<details>
<summary><b>Multi-Environment Deployment</b></summary>

```yaml
name: Multi-Env Deploy

on:
  push:
    branches: [main]

jobs:
  dev:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Salesforce (Dev)
        uses: rdbumstead/setup-salesforce-action@v2
        with:
          jwt_key: ${{ secrets.SFDX_JWT_KEY }}
          client_id: ${{ secrets.SFDX_CLIENT_ID }}
          username: ${{ vars.SFDX_DEV_USERNAME }}

      - run: sf project deploy start --source-dir force-app

  prod:
    needs: dev
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - name: Setup Salesforce (Prod)
        uses: rdbumstead/setup-salesforce-action@v2
        with:
          jwt_key: ${{ secrets.SFDX_JWT_KEY }}
          client_id: ${{ secrets.SFDX_CLIENT_ID }}
          username: ${{ vars.SFDX_PROD_USERNAME }}
          install_delta: "true"

      - run: |
          sf sgd:source:delta --to HEAD --from HEAD^ --output changed/
          sf project deploy start --manifest changed/package/package.xml
```

</details>

<details>
<summary><b>CLI Installation Only (No Auth)</b></summary>

```yaml
name: Install CLI

on: [push]

jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Salesforce CLI
        uses: rdbumstead/setup-salesforce-action@v2
        with:
          skip_auth: "true"
          install_delta: "true"

      - name: Custom Auth
        run: |
          # Your custom authentication logic here
          sf org login web
```

</details>

<details>
<summary><b>Multi-Org Setup</b></summary>

Authenticate to multiple orgs in the same workflow:

```yaml
name: Multi-Org Workflow

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Development Org
        uses: rdbumstead/setup-salesforce-action@v2
        with:
          jwt_key: ${{ secrets.SFDX_JWT_KEY }}
          client_id: ${{ secrets.SFDX_CLIENT_ID }}
          username: ${{ vars.DEV_USERNAME }}
          alias: "dev"
          install_delta: "true"

      - name: Setup Production Org
        uses: rdbumstead/setup-salesforce-action@v2
        with:
          jwt_key: ${{ secrets.SFDX_JWT_KEY }}
          client_id: ${{ secrets.SFDX_CLIENT_ID }}
          username: ${{ vars.PROD_USERNAME }}
          alias: "prod"
          install_delta: "true"

      - name: Deploy to Development
        run: |
          sf project deploy start --target-org dev --test-level NoTestRun

      - name: Deploy to Production
        run: |
          sf project deploy start --target-org prod --test-level RunLocalTests
```

</details>

<details>
<summary><b>Cross-Platform Testing</b></summary>

Test your code on all platforms:

```yaml
name: Cross-Platform Tests

on: [push, pull_request]

jobs:
  test:
    name: Test on ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]

    steps:
      - uses: actions/checkout@v4

      - name: Setup Salesforce
        uses: rdbumstead/setup-salesforce-action@v2
        with:
          jwt_key: ${{ secrets.SFDX_JWT_KEY }}
          client_id: ${{ secrets.SFDX_CLIENT_ID }}
          username: ${{ vars.SFDX_USERNAME }}
          install_lwc_jest: "true"

      - name: Run Tests
        run: npm test
```

</details>

<details>
<summary><b>Custom Plugins</b></summary>

Install additional Salesforce CLI plugins:

```yaml
name: Custom Plugins

on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Salesforce with Custom Plugins
        uses: rdbumstead/setup-salesforce-action@v2
        with:
          jwt_key: ${{ secrets.SFDX_JWT_KEY }}
          client_id: ${{ secrets.SFDX_CLIENT_ID }}
          username: ${{ vars.SFDX_USERNAME }}
          custom_sf_plugins: "sfdx-hardis,@salesforce/plugin-packaging"

      - name: Use Custom Plugin
        run: sf hardis:org:diagnose:legacyapi
```

</details>

<details>
<summary><b>Multi-Directory Projects</b></summary>

Handle multiple source directories:

```yaml
name: Multi-Directory Deploy

on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Salesforce
        id: setup
        uses: rdbumstead/setup-salesforce-action@v2
        with:
          jwt_key: ${{ secrets.SFDX_JWT_KEY }}
          client_id: ${{ secrets.SFDX_CLIENT_ID }}
          username: ${{ vars.SFDX_USERNAME }}
          source_dirs: "force-app,packages/core,packages/utils"

      - name: Deploy All Directories
        run: sf project deploy start ${{ steps.setup.outputs.source_flags }}
```

</details>

---

## ⚡ Performance Comparison

| Configuration  | Tools                 | Cache Hit | Cache Miss |
| -------------- | --------------------- | --------- | ---------- |
| Minimal        | CLI only              | ~25s      | ~1.5 min   |
| Recommended    | CLI + delta + scanner | ~55s      | ~2 min     |
| Full Stack     | All tools             | ~55s      | ~2.5 min   |
| Custom Plugins | CLI + heavy plugins   | ~95s      | ~3 min     |

**Why so fast?**

- Intelligent caching of CLI and plugins
- Conditional installation (skip if cached)
- Parallel npm operations
- No redundant downloads
- Platform-specific optimizations

> [!NOTE]
> Windows runners on GitHub Actions are **notoriously slower** due to file system differences and lower I/O performance. Expect execution times to be **2-3x longer** on Windows compared to Ubuntu or macOS. We highly recommend using Ubuntu runners for your primary CI/CD pipelines for best performance.

---

## 🤝 Related Projects

- **[Salesforce Platform Portfolio](https://github.com/rdbumstead/salesforce-platform-architect-portfolio)** - Enterprise Salesforce architecture examples and governance patterns
- **Salesforce CI/CD Platform** _(Coming Soon)_ - Complete pipeline templates and reusable workflows

---

## 📊 Comparison with Alternatives

| Feature               | This Action (v2) | Manual Setup | Salesforce Official |
| --------------------- | ---------------- | ------------ | ------------------- |
| Setup Time (cached)   | 25s - 1.5m       | N/A          | 60-90s              |
| Cross-Platform        | ✅ Win/Mac/Linux | ✅           | ⚠️ Limited          |
| JWT Auth Built-in     | ✅               | ❌           | ❌                  |
| Optional Plugins      | ✅               | ❌           | ❌                  |
| Custom Plugins        | ✅               | ✅           | ❌                  |
| Dev Tools Integration | ✅               | ❌           | ❌                  |
| Smart Caching         | ✅               | ❌           | ⚠️ Basic            |
| Skip Auth Option      | ✅               | ✅           | ❌                  |
| Minimal by Default    | ✅               | N/A          | ❌                  |
| Source Dir Resolution | ✅               | ❌           | ❌                  |

---

## 🔧 Troubleshooting

<details>
<summary><b>Authentication Failed</b></summary>

**Error:** `ERROR running org:login:jwt: We encountered a JSON web token error`

**Solutions:**

1. Verify JWT key includes BEGIN/END lines
2. Wait 2-10 minutes after creating Connected App
3. Check username matches org
4. Verify Consumer Key is correct
5. Ensure Connected App has OAuth scopes
6. **(Winter '25+)** Verify "Allow creation of connected apps" is enabled in Setup

- Setup → External Client Apps → Settings
- Toggle on "Allow creation of connected apps"

</details>

<details>
<summary><b>Plugin Installation Timeout</b></summary>

**Error:** `Plugin installation timed out`

**Solutions:**

1. Retry workflow (intermittent npm issues)
2. Check internet connectivity
3. Disable unused plugins
4. Action has automatic retry logic (3 attempts)

</details>

<details>
<summary><b>Cache Not Working</b></summary>

**Symptom:** Setup takes 2+ minutes every time

**Solutions:**

1. Check cache hit/miss in logs
2. Verify actions/cache@v4 permissions
3. Ensure cache hasn't exceeded 10GB limit

</details>

<details>
<summary><b>Windows/macOS Issues</b></summary>

See [Platform Support](docs/PLATFORM_SUPPORT.md) for platform-specific troubleshooting.

</details>

---

## 🗺️ Roadmap

- [x] Cross-platform support (Windows, macOS)
- [x] Custom plugin installation
- [x] Multi-directory source handling
- [ ] External Client App support (Winter '25+ orgs)
- [ ] CLI version auto-detection
- [ ] Enhanced validation outputs
- [ ] Integration with Salesforce Code Analyzer v4

---

## 📝 Documentation

- 📖 [Quick Start Guide](docs/QUICKSTART.md) - Get started in 15 minutes
- 📄 [Migration from v1](docs/MIGRATION_V1_TO_V2.md) - Upgrade guide
- 🖥️ [Platform Support](docs/PLATFORM_SUPPORT.md) - Windows, macOS, Linux details
- 🧪 [Testing Strategy](docs/TESTING_STRATEGY.md) - How we test
- 🔧 [Troubleshooting](docs/TROUBLESHOOTING.md) - Common issues and solutions
- 🤝 [Contributing](CONTRIBUTING.md) - How to contribute

---

## 📝 Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history.

---

## 🤝 Contributing

Contributions welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## 📄 License

MIT © [Ryan Bumstead](https://ryanbumstead.com)

---

## 💬 Support

- 🐞 [Report Bug](https://github.com/rdbumstead/setup-salesforce-action/issues)
- 💡 [Request Feature](https://github.com/rdbumstead/setup-salesforce-action/issues)
- 📧 [Contact](mailto:ryan@ryanbumstead.com)
- 💼 [LinkedIn](https://linkedin.com/in/ryanbumstead)
- 🌐 [GitHub Profile](https://github.com/rdbumstead)

---

**Built with ❤️ for the Salesforce Community by [Ryan Bumstead](https://github.com/rdbumstead)**

⭐ Star this repo if it helped you! | 🔗 [More Projects](https://github.com/rdbumstead?tab=repositories)
