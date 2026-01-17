# Setup Salesforce CLI (GitHub Action)

Deterministic, secure Salesforce CLI setup for real CI/CD pipelines.

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Setup%20Salesforce%20CLI-blue.svg)](https://github.com/marketplace/actions/setup-salesforce-cli)
[![GitHub release](https://img.shields.io/github/v/release/rdbumstead/setup-salesforce-action)](https://github.com/rdbumstead/setup-salesforce-action/releases)
[![Critical Tests](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-critical.yml/badge.svg)](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-critical.yml)
[![Plugin Tests](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-plugins.yml/badge.svg)](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-plugins.yml)
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
- [ ] Enhanced CLI version resolution & reporting
- [ ] Org limits & usage outputs
- [ ] SARIF output support
- [ ] Reusable CI/CD workflow templates

---

## 🤝 Support

- 🐞 **Issues**: [GitHub Issues](https://github.com/rdbumstead/setup-salesforce-action/issues)
- 💬 **LinkedIn**: [Ryan Bumstead](https://linkedin.com/in/ryanbumstead)

_Built for Salesforce teams who want boring, repeatable pipelines._
