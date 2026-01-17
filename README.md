# Setup Salesforce CLI (GitHub Action)

Deterministic, secure Salesforce CLI setup for real CI/CD pipelines.

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Setup%20Salesforce%20CLI-blue.svg)](https://github.com/marketplace/actions/setup-salesforce-cli)
[![GitHub release](https://img.shields.io/github/v/release/rdbumstead/setup-salesforce-action)](https://github.com/rdbumstead/setup-salesforce-action/releases)
[![Critical Tests](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-critical.yml/badge.svg)](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-critical.yml)
[![Plugin Tests](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-plugins.yml/badge.svg)](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-plugins.yml)
[![Authentication Tests](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-auth.yml/badge.svg)](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-auth.yml)
[![Cross Platform Tests](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-cross-platform.yml/badge.svg)](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-cross-platform.yml)

> Fast, cached, and configurable Salesforce CLI setup for GitHub Actions.  
> Designed for production pipelines, mono-repos, and enterprise Salesforce teams.

---

## 🚀 Why This Action

Most Salesforce pipelines fail due to:

- ❌ Non-deterministic CLI installs
- ❌ Slow, repeated setup
- ❌ Fragile auth handling
- ❌ One-size-fits-all workflows

This action solves those problems by providing a **stable execution layer** that workflows can reliably depend on.

## ✨ Key Capabilities

- **⚡ Fast**: ~20–60s with caching
- **🔐 Secure Auth**: JWT, SFDX Auth URL, or Access Token
- **📦 Smart Caching**: CLI + plugins cached across runs
- **📂 Mono-Repo Ready**: Multi-directory source resolution
- **🔌 Extensible**: Optional plugins and dev tools
- **🧪 Well Tested**: Linux, macOS, and Windows runners

---

## 🧩 Quick Start

### Minimal Setup (CLI + Auth)

```yaml
- name: Setup Salesforce
  uses: rdbumstead/setup-salesforce-action@v2
  with:
    jwt_key: ${{ secrets.SFDX_JWT_KEY }}
    client_id: ${{ secrets.SFDX_CLIENT_ID }}
    username: ${{ vars.SFDX_USERNAME }}
```

### Recommended Setup (Delta + Code Analyzer)

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

---

## 🔐 Authentication Methods

| Method            | Use Case               |
| ----------------- | ---------------------- |
| **JWT** (default) | Production CI/CD       |
| **SFDX Auth URL** | Sandboxes, quick setup |
| **Access Token**  | Advanced integrations  |

**Example (SFDX Auth URL):**

```yaml
auth_method: "sfdx-url"
sfdx_auth_url: ${{ secrets.SFDX_AUTH_URL }}
```

---

## 📤 Outputs

Useful for conditional and reusable workflows:

- `org_id`
- `org_type`
- `org_edition`
- `api_version`
- `auth_performed`
- `sf_cli_version`
- `source_flags`

**Example:**

```yaml
- run: sf project deploy start ${{ steps.setup.outputs.source_flags }}
```

---

## 📚 Documentation

Full documentation lives in `/docs`:

- 📖 [Action Overview](docs/OVERVIEW.md)
- 🚀 [Quick Start Guide](docs/QUICKSTART.md)
- 🔄 [Migration Guide](docs/MIGRATION.md)
- 🧪 [Testing Strategy](docs/TESTING_STRATEGY.md)
- 🖥️ [Platform Support](docs/PLATFORM_SUPPORT.md)
- 🔧 [Troubleshooting](docs/TROUBLESHOOTING.md)

### 🧭 Roadmap

- [x] Cross-platform support (Windows, macOS)
- [x] Custom plugin installation
- [x] Multi-directory source handling
- [x] External Client App support (Winter '25+ orgs)
- [x] Enhanced CLI version resolution & reporting (v2.1+)
- [ ] Org limits & usage outputs
- [ ] SARIF output support
- [ ] Reusable CI/CD workflow templates

---

## 🙏 Credits & Acknowledgments

This action orchestrates the installation of several best-in-class open-source tools. We recommend starring their repositories and reviewing their specific documentation:

- **[sfdx-git-delta](https://github.com/scolladon/sfdx-git-delta)** by [Sebastien Colladon](https://github.com/scolladon)
  _Used for the `install_delta` feature. This tool is essential for generating delta deployments._
- **[Salesforce Code Analyzer](https://github.com/forcedotcom/code-analyzer)** by Salesforce
  _Used for the `install_scanner` feature. Provides PMD, ESLint, and RetireJS scanning._
- **[Prettier Plugin Apex](https://github.com/dangmai/prettier-plugin-apex)**
  _Used for the `install_prettier` feature to format Apex code._
- **[LWC ESLint Plugin](https://github.com/salesforce/eslint-plugin-lwc)**
  _Used for the `install_eslint` feature to lint Lightning Web Components._

### Tested With

We explicitly verify compatibility with popular ecosystem plugins in our [test suite](.github/workflows/test-plugins.yml), including:

- **[sfdx-hardis](https://github.com/hardisgroupcom/sfdx-hardis)** (CI/CD orchestration)
- **[sfpowerscripts](https://github.com/dxatscale/sfpowerscripts)** (Release management)

---

## 🤝 Support

- 🐞 **Issues**: [GitHub Issues](https://github.com/rdbumstead/setup-salesforce-action/issues)
- 💬 **LinkedIn**: [Ryan Bumstead](https://linkedin.com/in/ryanbumstead)

_Built for Salesforce teams who want boring, repeatable pipelines._
