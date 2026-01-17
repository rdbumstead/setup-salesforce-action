# Testing Strategy

This document explains the comprehensive testing strategy for the **Setup Salesforce CLI Action**. Our testing architecture ensures reliability, security, and cross-platform compatibility through a four-pillar testing approach.

## 🎯 Testing Philosophy

We use a tiered testing approach to balance fast feedback loops with exhaustive validation:

1. **Critical Tests** (`test-critical.yml`): Fast, no-secret sanity checks running on every push.
2. **Plugin Tests** (`test-plugins.yml`): Dedicated validation for ecosystem tools and plugins.
3. **Authentication Tests** (`test-auth.yml`): Deep validation of all auth methods (requires secrets).
4. **Cross-Platform Tests** (`test-cross-platform.yml`): Exhaustive matrix validation for releases.

---

## 📊 Workflows Overview

| Workflow     | File                      | Frequency         | Focus                               | Secrets?   |
| :----------- | :------------------------ | :---------------- | :---------------------------------- | :--------- |
| **Critical** | `test-critical.yml`       | Every Push/PR     | Core Logic, Caching, Error Handling | ❌ No      |
| **Plugins**  | `test-plugins.yml`        | Every Push/PR     | Plugins, Scanners, Dev Tools        | ❌ No      |
| **Auth**     | `test-auth.yml`           | Owner Push/Manual | JWT, SFDX URL, Access Token         | ✅ Yes     |
| **Release**  | `test-cross-platform.yml` | Releases/Manual   | OS Compatibility, Security          | ⚠️ Partial |

---

## 🚀 1. Critical Tests (`test-critical.yml`)

**Purpose**: Fast feedback for "did I break the build?" questions. These tests never fail due to missing secrets.

### Key Scenarios

- **CLI Installation**: Verifies `sf` installs correctly without auth.
- **Error Handling**: Ensures the action fails gracefully when inputs are missing.
- **Cache Behavior**: Verifies cache misses (first run) and hits (second run).
- **Source Flags**: cross-validates output logic for `source_dirs`.
- **Retry Logic**: Simulates network failures to test retry mechanisms.

---

## 🔌 2. Plugin Tests (`test-plugins.yml`)

**Purpose**: Validate the installation and execution of external tools and plugins.

### Key Scenarios

- **Custom Plugins**: Installs external plugins (e.g., `sfdx-hardis`, `@salesforce/plugin-packaging`).
- **Built-in Tools**: Verifies `install_scanner`, `install_delta` logic.
- **Dev Tools**: Checks `prettier`, `eslint`, `sfdx-lwc-jest`.
- **Strict Mode**: Verifies that invalid plugin names fail the build when `strict: true`.

---

## 🔐 3. Authentication Tests (`test-auth.yml`)

**Purpose**: Verify that the action can actually authenticate with Salesforce. These tests require repository secrets and typically skip on forks.

### Key Scenarios

- **JWT Auth**: The gold standard for CI/CD.
- **SFDX URL**: Alternate auth method.
- **Access Token**: Using ephemeral tokens minted dynamically.
- **Output Validation**: Verification of `org_id`, `username`, `instance_url` outputs.
- **Dev Hub**: Validates `is_dev_hub: true` configuration.

> **Note**: These tests are gated to run only for the repository owner or manual dispatch to prevent failure in forks.

---

## 🌍 4. Cross-Platform Tests (`test-cross-platform.yml`)

**Purpose**: The final gate before a release. Ensures behavior is identical across OSs.

### Matrix Coverage

- **OS**: Ubuntu, Windows, macOS
- **Node**: v18, v20

### Key Scenarios

- **Platform Specifics**: Checks `timeout` command availability (macOS), Path handling (Windows).
- **Security Hygiene**: Verifies removal of `jwt.key`, `access_token.txt` files after run (Unix).
- **Cache Robustness**: Verifies detailed cache restoration logic across platforms.

---

## 🧪 Running Tests Locally

You can use [act](https://github.com/nektos/act) to run these workflows locally.

### Run Critical Tests

```bash
# Fastest way to check your changes
act -j summary -W .github/workflows/test-critical.yml
```

### Run Plugin Tests

```bash
act -j summary -W .github/workflows/test-plugins.yml
```

### Run Auth Tests (Requires Secrets)

Create a `.secrets` file with `SFDX_JWT_KEY`, `SFDX_CLIENT_ID`, etc.

```bash
act -j summary -W .github/workflows/test-auth.yml --secret-file .secrets
```

---

## � Troubleshooting

| Problem                                 | Likely Workflow       | Check                                          |
| :-------------------------------------- | :-------------------- | :--------------------------------------------- |
| **"Action failed with missing inputs"** | `test-critical`       | unexpected `strict` mode behavior?             |
| **"Plugin not found"**                  | `test-plugins`        | NPM registry issues or regex validation error? |
| **"Org login failed"**                  | `test-auth`           | Expired certificate or invalid secrets?        |
| **"File permission error"**             | `test-cross-platform` | Windows vs Unix `chmod` logic discrepancy?     |

---

**Questions?** Open an issue or discussion!
