# Platform Support Guide

This document details cross-platform support for the Setup Salesforce CLI action across Ubuntu, Windows, and macOS runners.

## 🖥️ Supported Platforms

| Platform                          | Status             | Node Versions | Notes                                               |
| --------------------------------- | ------------------ | ------------- | --------------------------------------------------- |
| **Ubuntu** (latest, 22.04, 20.04) | ✅ Fully Supported | 18, 20, 22    | Recommended for best performance                    |
| **Windows** (latest, 2022, 2019)  | ✅ Fully Supported | 18, 20, 22    | PowerShell and Bash shells supported                |
| **macOS** (latest, 13, 12)        | ✅ Fully Supported | 18, 20, 22    | Homebrew required (pre-installed on GitHub runners) |

## 🚀 Quick Start by Platform

### Ubuntu (Linux)

```yaml
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
          username: ${{ secrets.SFDX_USERNAME }}

      - name: Deploy
        run: sf project deploy start --source-dir force-app
```

**Advantages:**

- Fastest execution time
- Most cost-effective (GitHub Actions billing)
- Best caching performance
- Recommended for CI/CD pipelines

### Windows

```yaml
jobs:
  deploy:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Salesforce
        uses: rdbumstead/setup-salesforce-action@v2
        with:
          jwt_key: ${{ secrets.SFDX_JWT_KEY }}
          client_id: ${{ secrets.SFDX_CLIENT_ID }}
          username: ${{ secrets.SFDX_USERNAME }}

      # You can use either PowerShell or Bash
      - name: Deploy (PowerShell)
        shell: pwsh
        run: sf project deploy start --source-dir force-app

      # Or use Bash (Git Bash is available)
      - name: Deploy (Bash)
        shell: bash
        run: sf project deploy start --source-dir force-app
```

**Advantages:**

- Native Windows environment for Windows-specific tooling
- PowerShell script integration
- Better for teams primarily on Windows

**Windows-Specific Features:**

- Automatic `jq` installation via Chocolatey
- Windows-specific cache paths (`~/AppData/Local/sf`)
- `icacls` for secure JWT key permissions

### macOS

```yaml
jobs:
  deploy:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Salesforce
        uses: rdbumstead/setup-salesforce-action@v2
        with:
          jwt_key: ${{ secrets.SFDX_JWT_KEY }}
          client_id: ${{ secrets.SFDX_CLIENT_ID }}
          username: ${{ secrets.SFDX_USERNAME }}

      - name: Deploy
        run: sf project deploy start --source-dir force-app
```

**Advantages:**

- Native macOS environment
- Better for teams primarily on Mac
- Required for macOS-specific testing

**macOS-Specific Features:**

- Automatic `jq` installation via Homebrew
- Homebrew pre-installed on GitHub-hosted runners

## 🧪 Cross-Platform Testing

Test your Salesforce code across all platforms:

```yaml
name: Cross-Platform Tests

on: [push, pull_request]

jobs:
  test:
    name: Test on ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [18, 20]

    steps:
      - uses: actions/checkout@v4

      - name: Setup Salesforce
        uses: rdbumstead/setup-salesforce-action@v2
        with:
          jwt_key: ${{ secrets.SFDX_JWT_KEY }}
          client_id: ${{ secrets.SFDX_CLIENT_ID }}
          username: ${{ secrets.SFDX_USERNAME }}
          node_version: ${{ matrix.node-version }}
          install_lwc_jest: "true"

      - name: Run Tests
        shell: bash
        run: npm test
```

## 📦 Dependency Management

### jq (JSON processor)

The action automatically installs `jq` on all platforms:

| Platform | Installation Method | Automatic? |
| -------- | ------------------- | ---------- |
| Ubuntu   | `apt-get`           | ✅ Yes     |
| Windows  | `chocolatey`        | ✅ Yes     |
| macOS    | `brew`              | ✅ Yes     |

### Node.js

Node.js is installed via `actions/setup-node@v4` and works identically across all platforms.

### Salesforce CLI

Installed via npm globally, works identically across all platforms.

## 💾 Caching Strategy

The action uses platform-aware caching:

### Linux/macOS Cache Paths

```yaml
- ~/.npm
- ~/.local/share/sf
- ~/.cache/node-gyp
```

### Windows Cache Paths

```yaml
- ~/.npm
- ~/AppData/Local/sf
- ~/AppData/Roaming/npm
- ~/.cache/node-gyp
```

### Cache Key Format

```
sf-v3-{OS}-node{version}-cli{version}-tools{hash}
```

**Example Keys:**

- `sf-v3-Linux-node20-cli2.30.8-toolsabcd1234`
- `sf-v3-Windows-node20-cli2.30.8-toolsabcd1234`
- `sf-v3-macOS-node20-cli2.30.8-toolsabcd1234`

This ensures each OS maintains its own cache while sharing the same configuration.

## 🔐 Security Considerations

### JWT Key File Permissions

The action handles JWT key security differently per platform:

**Linux/macOS:**

```bash
chmod 600 jwt.key
```

**Windows:**

```bash
icacls jwt.key /inheritance:r
icacls jwt.key /grant:r "$env:USERNAME:(R)"
```

Both methods ensure only the current user can read the JWT private key.

## 🎯 Shell Selection

Different shells are available on different platforms:

### Bash (Default)

```yaml
- name: Run Command
  shell: bash
  run: echo "Works on all platforms"
```

- ✅ Ubuntu: Native
- ✅ Windows: Git Bash (included)
- ✅ macOS: Native

### PowerShell

```yaml
- name: Run Command
  shell: pwsh
  run: Write-Host "PowerShell Core"
```

- ✅ Ubuntu: Available
- ✅ Windows: Native (PowerShell 7+)
- ✅ macOS: Available

### Platform-Specific Shells

```yaml
# Windows only - PowerShell 5.1
- name: Windows PowerShell
  if: runner.os == 'Windows'
  shell: powershell
  run: Write-Host "Windows PowerShell"

# Unix shells
- name: Unix Shell
  if: runner.os != 'Windows'
  shell: bash
  run: echo "Unix shell"
```

## 🐞 Platform-Specific Troubleshooting

### Windows Issues

**Issue**: Chocolatey command not found

```yaml
# Solution: Use GitHub-hosted runners which include Chocolatey
runs-on: windows-latest
```

**Issue**: Path length limitations (260 characters)

```yaml
# Solution: Enable long paths in your self-hosted runner
# Or use shorter project paths
```

**Issue**: Line ending problems (CRLF vs LF)

```yaml
# Solution: Configure git to handle line endings
- name: Configure Git
  run: git config --global core.autocrlf false
```

### macOS Issues

**Issue**: Homebrew not found (self-hosted runners)

```yaml
# Solution: Install Homebrew first
- name: Install Homebrew
  if: runner.os == 'macOS'
  run: |
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**Issue**: Code signing/Gatekeeper issues

```yaml
# GitHub-hosted runners don't have this issue
# Self-hosted: Disable Gatekeeper or sign your scripts
```

### Linux Issues

**Issue**: Missing sudo access (self-hosted)

```yaml
# Solution: Ensure runner user has sudo access for apt-get
# Or pre-install dependencies
```

## 📊 Performance Benchmarks

Typical action execution times (with cache):

| Platform | CLI Install | With Tools | First Run (no cache) |
| -------- | ----------- | ---------- | -------------------- |
| Ubuntu   | ~25s        | ~55s       | ~1.5 min             |
| macOS    | ~40s        | ~2 min     | ~3.5 min             |
| Windows  | ~5-7 min    | ~8-15 min  | ~15-25 min           |

> [!WARNING]
> **Windows Performance:** Windows runners on GitHub Actions are significantly slower due to file system differences and lower I/O performance. Expect execution times to be **10-15x longer** than Ubuntu. We strongly recommend using Ubuntu runners (`runs-on: ubuntu-latest`) for your primary CI/CD pipelines for best performance and cost efficiency.

**Factors affecting performance:**

- Network speed
- Cache hit/miss
- Number of tools installed
- Runner specifications (for self-hosted)
- **Operating System** (Ubuntu fastest, Windows slowest)

**Cost Optimization:**

- Ubuntu: 1x cost (recommended)
- macOS: 10x cost
- Windows: 2x cost but 10-15x slower execution = poor value

## 🏗️ Self-Hosted Runners

### Ubuntu Self-Hosted

**Prerequisites:**

```bash
# Install dependencies
sudo apt-get update
sudo apt-get install -y curl git jq

# Install Node.js (if not using actions/setup-node)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### Windows Self-Hosted

**Prerequisites:**

```powershell
# Install Chocolatey
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# Install dependencies
choco install git nodejs jq -y
```

### macOS Self-Hosted

**Prerequisites:**

```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install dependencies
brew install git node jq
```

## 🔄 Migration Between Platforms

### Moving from Ubuntu to Windows

No code changes needed in most cases:

```yaml
# Before
runs-on: ubuntu-latest

# After
runs-on: windows-latest
```

**Considerations:**

- Verify shell scripts work with Git Bash
- Check for hardcoded Unix paths
- Test file permission handling

### Moving from Windows to Ubuntu

```yaml
# Before
runs-on: windows-latest

# After
runs-on: ubuntu-latest
```

**Considerations:**

- Replace PowerShell-specific commands with bash
- Update any Windows-specific paths
- Faster execution, lower cost

## 🎨 Best Practices

### 1. Use Matrix Builds for Cross-Platform Testing

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
runs-on: ${{ matrix.os }}
```

### 2. Default to Ubuntu for Speed

```yaml
# Use Ubuntu for most CI/CD
runs-on: ubuntu-latest
# Use Windows/macOS only when needed
```

### 3. Use Bash for Portability

```yaml
# Works everywhere
shell: bash
run: |
  sf --version
  echo "Cross-platform command"
```

### 4. Platform-Specific Steps When Necessary

```yaml
- name: Windows-Specific Setup
  if: runner.os == 'Windows'
  shell: pwsh
  run: |
    # Windows-specific commands

- name: Unix-Specific Setup
  if: runner.os != 'Windows'
  shell: bash
  run: |
    # Unix-specific commands
```

### 5. Test on Target Platform

If your team uses macOS for development, include macOS in your CI:

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, macos-latest]
```

## 📝 Platform-Specific Examples

### Example: Windows with PowerShell

```yaml
name: Windows Deploy

on: [push]

jobs:
  deploy:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Salesforce
        uses: rdbumstead/setup-salesforce-action@v2
        with:
          jwt_key: ${{ secrets.SFDX_JWT_KEY }}
          client_id: ${{ secrets.SFDX_CLIENT_ID }}
          username: ${{ secrets.SFDX_USERNAME }}
          install_delta: "true"

      - name: Deploy with PowerShell
        shell: pwsh
        run: |
          $env:SFDX_JSON_TO_STDOUT = "true"
          sf project deploy start --source-dir force-app --json | ConvertFrom-Json
```

### Example: macOS with Homebrew Tools

```yaml
name: macOS Build

on: [push]

jobs:
  build:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Salesforce
        uses: rdbumstead/setup-salesforce-action@v2
        with:
          skip_auth: "true"

      - name: Install Additional Tools
        run: |
          brew install tree
          tree force-app
```

## 🔗 Related Resources

- [GitHub Actions: Runner Images](https://github.com/actions/runner-images)
- [Salesforce CLI on Different Platforms](https://developer.salesforce.com/docs/atlas.en-us.sfdx_setup.meta/sfdx_setup/sfdx_setup_install_cli.htm)
- [Cross-Platform Shell Scripting](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsshell)

---

**Need help with a specific platform?** [Open an issue](https://github.com/rdbumstead/setup-salesforce-action/issues) with details about your platform and use case.
