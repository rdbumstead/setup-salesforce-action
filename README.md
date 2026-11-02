# Setup Salesforce CLI (GitHub Action)

Deterministic, secure Salesforce CLI setup for real CI/CD pipelines.

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Setup%20Salesforce%20CLI-blue.svg)](https://github.com/marketplace/actions/setup-salesforce-cli)
[![GitHub release](https://img.shields.io/github/v/release/rdbumstead/setup-salesforce-action)](https://github.com/rdbumstead/setup-salesforce-action/releases)
[![Critical Tests](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-critical.yml/badge.svg)](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-critical.yml)
[![Plugin Tests](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-plugins.yml/badge.svg)](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-plugins.yml)
[![Authentication Tests](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-auth.yml/badge.svg)](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-auth.yml)
[![Cross Platform Tests](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-cross-platform.yml/badge.svg)](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-cross-platform.yml)
[![Invariants Tests](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-invariants.yml/badge.svg)](https://github.com/rdbumstead/setup-salesforce-action/actions/workflows/test-invariants.yml)

> Fast, cached, and configurable Salesforce CLI setup for GitHub Actions.  
> Designed for production pipelines, mono-repos, and enterprise Salesforce teams.

---

## Why This Action

Most Salesforce pipelines fail due to:

- Non-deterministic CLI installs
- Slow, repeated setup
- Fragile auth handling
- One-size-fits-all workflows

This action solves those problems by providing a **stable execution layer** that workflows can reliably depend on.

## Key Capabilities

- **Fast**: ~20 to 60s with caching
- **Secure Auth**: JWT, SFDX Auth URL, or Access Token
- **Smart Caching**: CLI + plugins cached across runs
- **Mono-Repo Ready**: Multi-directory source resolution
- **Extensible**: Optional plugins and dev tools
- **Well Tested**: Linux, macOS, and Windows runners

---

## Quick Start

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

## Authentication Methods

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

## Outputs

Useful for conditional and reusable workflows:

- `org_id`
- `org_type`
- `org_edition`
- `api_version`
- `auth_performed`
- `sf_cli_version`
- `source_flags`
- `used_default_node` _(new in v3.0)_ - Whether default Node.js version was used
- `used_default_cli_version` _(new in v3.0)_ - Whether default CLI version was used
- `used_default_api_version` _(new in v3.0)_ - Whether API version was auto-detected
- `cli_binary_path` _(new in v3.0)_ - Absolute path to `sf` executable (for custom tooling)
- `validated_config` _(new in v3.0)_ - JSON summary of final configuration (for auditing)

**Example:**

```yaml
- run: sf project deploy start ${{ steps.setup.outputs.source_flags }}
```

---

## Guaranteed Invariants

This action guarantees the following invariants on successful completion:

- **CLI installed and callable** - `sf` command is available and functional
- **Authenticated org available** - Org is reachable under the specified alias (unless `skip_auth: true`)
- **Instance URL resolved** - Org instance URL is known
- **API version resolved** - Salesforce API version is determined
- **No secrets in logs** - Sensitive credentials are never written to logs

If any invariant fails, the action will **fail fast** and report the specific violation.

### Why This Matters

Partial failures in foundation primitives are dangerous. This action will never report success if:

- CLI installs but is non-functional
- Authentication succeeds but org is unreachable
- Org metadata queries fail

This makes the action suitable for use as a primitive in complex CI/CD workflows.

---

## Versioning Policy

This action follows strict [Semantic Versioning](https://semver.org/):

- **MAJOR** (v3.0.0) - Breaking changes to guarantees or behavior
- **MINOR** (v3.1.0) - New features, backward compatible
- **PATCH** (v3.0.1) - Bug fixes only

### Guarantees

- Defaults never change in MINOR versions (e.g., CLI version, Node version). Exception: when a default runtime reaches its upstream end of life, a MINOR release may move the default to the current LTS line, and the CHANGELOG calls it out.
- Breaking changes only in MAJOR versions
- Consumers should pin to MAJOR version: `@v3`

### What Counts as Breaking

- Removing inputs or outputs
- Changing default values
- New invariant validations (may fail previously passing workflows)
- Changes to output formats

**Recommendation**: Always pin to a major version in production workflows (`uses: rdbumstead/setup-salesforce-action@v3`).

---

## Caching Strategy

This action uses intelligent caching to optimize setup time (~20-60s cached vs ~1.5-3min cold).

### Cache Key Design

Cache keys are composed of:

```
sf-v3-{OS}-node{version}-cli{resolved_version}-tools{hash}
```

- **OS**: Linux, macOS, or Windows (prevents cross-platform corruption)
- **Node version**: From `node_version` input
- **CLI version**: Resolved from `cli_version` input (see below)
- **Tools hash**: SHA256 of all optional tooling flags

### CLI Version Resolution

When `cli_version: "latest"` (default):

1. **Try npm registry** - Fetches actual version via `npm view @salesforce/cli version` (10s timeout)
2. **Fallback to time-based rotation** - If npm unreachable, uses `latest-YYYY-MM` format
   - Ensures cache refreshes monthly even during npm outages
   - Prevents indefinite staleness

When explicit version specified (e.g., `cli_version: "2.30.8"`):

- Uses exact version in cache key
- Controlled by `cli_version_for_cache` input (major/minor/exact granularity)

### Cache Hit/Miss Scenarios

**Cache HIT**:

- Same OS, Node version, CLI version, and tooling
- CLI already functional from previous run

**Cache MISS**:

- Different OS or Node version
- CLI version changed (or monthly rotation occurred)
- Tooling flags changed (added/removed plugins)
- First run in new environment

### Force Cache Refresh

To force a fresh install:

1. Change `cli_version` to explicit version, or
2. Change `cli_version_for_cache` granularity, or
3. Wait for monthly rotation (if using `latest`)

---

## Documentation

Full documentation lives in `/docs`:

- [Action Overview](docs/OVERVIEW.md)
- [Quick Start Guide](docs/QUICKSTART.md)
- [Migration Guide](docs/MIGRATION.md)
- [Testing Strategy](docs/TESTING_STRATEGY.md)
- [Platform Support](docs/PLATFORM_SUPPORT.md)
- [Troubleshooting](docs/TROUBLESHOOTING.md)
- [Architecture](docs/ARCHITECTURE.md) _(new in v3.0)_

### Roadmap

- [x] Cross-platform support (Windows, macOS)
- [x] Custom plugin installation
- [x] Multi-directory source handling
- [x] External Client App support (Winter '25+ orgs)
- [x] Enhanced CLI version resolution & reporting (v2.1+)
- [ ] Org limits & usage outputs
- [ ] SARIF output support
- [ ] Reusable CI/CD workflow templates

---

## Credits & Acknowledgments

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

## Support

- **Issues**: [GitHub Issues](https://github.com/rdbumstead/setup-salesforce-action/issues)
- **LinkedIn**: [Ryan Bumstead](https://linkedin.com/in/ryanbumstead)

_Built for Salesforce teams who want boring, repeatable pipelines._
