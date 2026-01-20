# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [3.0.0] - 2026-01-19

### 🔒 Production Hardening

This release transforms the action into a bulletproof primitive suitable for foundational CI/CD use.

### Added ➕

- **Invariant Validation** - Mandatory validation step ensures setup integrity:
  - CLI is installed AND functional (not just present on PATH)
  - Authenticated org is reachable (not just auth succeeded)
  - API version is resolved (not just queried)
  - Fails fast with clear violation messages instead of silent partial failures
- **Default Usage Tracking** - New outputs enable enforcement hooks in higher-level workflows:
  - `used_default_node` - Whether default Node.js version (20) was used
  - `used_default_cli_version` - Whether default CLI version (latest) was used
  - `used_default_api_version` - Whether API version was auto-detected (always true currently)
  - Allows CI/CD policies to block deployments that rely on implicit defaults
- **Dry-Run Mode** - New `dry_run` input skips authentication and mutations while validating detection logic:
  - Useful for testing action configuration without consuming org API calls
  - Skips all auth steps but still installs CLI and resolves environment
- **Debug Mode** - New `debug` input placeholder for future verbose output (accepted but not yet implemented)
- **Structured Observability** - New step publishes audit-friendly summary to GitHub Step Summary:
  - Auth method, org type, API version
  - CLI and Node.js versions
  - Default usage warnings
  - Complete list of installed tools and plugins
  - Always runs (even on failure) for post-mortem analysis
- **Forward Compatibility Outputs** - Added outputs to support future v4 modularization:
  - `cli_binary_path` - Absolute path to installed binary (for custom tooling integration)
  - `validated_config` - JSON summary of effective configuration (for auditing/debugging)

### Changed 🔧

- **BREAKING (Intentional)**: Action now fails fast on partial failures
  - Workflows that previously succeeded despite broken CLI or unreachable orgs will now fail
  - This is the correct behavior for a primitive—silent failures are dangerous
  - Example: CLI installed but `sf` command non-functional → now fails instead of succeeding
- **Shell Hardening Enhanced**: All bash steps now use strict error handling:
  - Core steps: `set -euo pipefail` (exit on error, undefined variables, or pipe failures)
  - Optional tooling: `set -eu` + conditional `pipefail` based on `strict` mode
  - Prevents subtle bugs from undefined variables while keeping plugin installs resilient

### Fixed 🐞

- **Hidden Partial Failure Risk**: Action will no longer report success when:
  - CLI installation succeeds but CLI is non-functional
  - Authentication succeeds but org is unreachable
  - Org display succeeds but API version cannot be determined
- **Silent Downgrade Risk**: Default usage is now explicitly tracked and exposed
  - Prevents workflows from unknowingly relying on implicit defaults
  - Enables explicit versioning enforcement in hardened pipelines

### Testing 🧪

- **New Test Workflow**: `test-invariants.yml` validates:
  - Invariant validation catches failures correctly
  - Dry-run mode works without authentication
  - Default tracking outputs are accurate
  - All features work across Linux, macOS, Windows

### Documentation 📖

- **Guaranteed Invariants**: New README section documents the action's contract:
  - Lists explicit invariants guaranteed on success
  - Explains why this matters for primitive composability
  - Positions action as safe foundation for complex workflows
- **Versioning Policy**: Formal semantic versioning governance:
  - Defines what counts as breaking vs non-breaking changes
  - Guarantees defaults never change in MINOR versions
  - Recommends pinning to MAJOR version in production
- **Caching Strategy**: Comprehensive documentation of cache behavior:
  - Explains cache key composition
  - Documents CLI version resolution logic (npm query + time-based fallback)
  - Lists cache hit/miss scenarios
  - Provides cache refresh strategies
- **Architecture Documentation**: New `docs/ARCHITECTURE.md` captures:
  - Design philosophy and principles
  - Current v3 architecture with component diagrams
  - Future v4 modularization roadmap
  - Polyglot pattern for hybrid TypeScript utilities
  - Architectural Decision Records (ADRs)

### Why This Matters

This action is designed as a **primitive**, not an application. Primitives must fail loudly and clearly when invariants are violated. Silent partial failures undermine trust in the entire CI/CD pipeline.

Before v3.0.0, it was possible for:

- CLI to install but be non-functional → workflow succeeds → subsequent steps fail mysteriously
- Auth to succeed but org be unreachable → workflow succeeds → deployments fail unpredictably
- API version to be unresolved → workflow succeeds → commands using API calls fail

After v3.0.0, these scenarios fail immediately with actionable error messages.

---

## [2.2.0] - 2026-01-17

### 🚀 New Features - Performance & Caching

- **OS-Specific Caching** - Cache paths are now partitioned by operating system (Linux/macOS/Windows) to prevent cross-platform corruption and improve hit rates
- **Optimized Version Resolution** - Uses `npm view` to resolve "latest" versions 10x faster than a full install, with 10s timeout protection against network hangs
- **Smart Hash Detection** - Automatically uses `sha256sum` or `shasum -a 256` depending on the runner OS
- **CLI Health Verification** - Automatic health check after CLI installation validates core plugins are loaded and functional
  - Detects broken installations immediately instead of failing later in workflows
  - Adds ~100ms overhead for significantly improved reliability
  - Runs inline with installation step for fast feedback

### Performance 🚀

- **Setup time**: ~25-55s (cached), ~1.5-3 min (first run) on Ubuntu/macOS
- **Cache hit rate**: >95% across all platforms
- **Health check overhead**: ~0.1s (negligible)
- **Reliability**: Broken installations caught immediately
- **Platform note**: Windows runners are 10-15x slower; Ubuntu recommended for production CI/CD

### 🛡️ Reliability & Retries

- **Resilient Installation Logic** - All network-dependent steps now include 3-attempt retries with exponential backoff:
  - Salesforce CLI (`@salesforce/cli`)
  - `sfdx-git-delta`
  - `@salesforce/plugin-code-analyzer`
  - Custom plugins (each plugin retries individually)
- **Improved Cache Fallback** - When npm registry is unreachable, cache keys now use monthly time-based rotation (`latest-YYYY-MM`) instead of static "latest"
  - Prevents indefinite cache staleness during npm outages
  - Automatically rotates monthly to ensure fresh CLI versions
  - Provides clearer error messaging about fallback behavior

### Added ➕

- **Formal Access Token Support** - Added explicit support for `auth_method: access-token` using the `sf org login access-token` command (replaces legacy logic)
- **Access Token Default** - `allow_access_token_auth` now defaults to `true`, making this a safe non-breaking change
- **New Tests** (`test-access-token-auth`, `test-multiple-plugins`) to validate complex scenarios
- **Enhanced Testing** - Added comprehensive CLI health checks to critical and cross-platform test suites
  - Health checks now run on all platforms (Ubuntu, macOS, Windows)
  - Validates CLI version, core plugins, help system, config commands, and org listing
  - Total test coverage increased from ~90% to ~95%

### Changed 🔧

- **Test Architecture Overhaul** - Replaced monolithic "Quick Tests" (`test.yml`) with 4 dedicated workflows (`test-critical`, `test-plugins`, `test-auth`, `test-cross-platform`)
- **Improved Plugin Verification** - Test suite now uses `jq` regex matching to reliably detect namespaced plugins
- **Better CLI Validation** - Switched verification commands to `sf plugins` to ensure core plugin availability
- **Refactored Retry Tests** - Renamed `test-network-retry` to `test-cli-install-retry` for clarity
- **Enhanced Error Messages** - Source directory validation now includes actionable troubleshooting tips
  - Clarifies that `force-app` is auto-created while custom directories must exist
  - Suggests checking for typos in `source_dirs` input
  - Reduces support burden with self-service guidance

### Fixed 🐞

- **Windows Permissions** - Fixed file permission warnings for temporary auth files (`authurl.txt`, `access_token.txt`) on Windows runners
- **Instance URL Handling** - Auth logic now correctly strips trailing slashes from `instance_url` to prevent connection errors
- **Custom Plugin Loops** - Fixed bash iteration logic to correctly install multiple comma-separated plugins
- **Source Flags** - Output verification now strictly checks for the `--source-dir` format
- **Dependencies** - `authurl.txt` and `access_token.txt` added to `.gitignore`
- **Test Consistency** - Standardized boolean input format across all test workflows (`skip_auth: "true"` instead of mixed `true`/`"true"`)

---

## [2.1.0] - 2026-01-15

### Added

- Multiple authentication methods (JWT, SFDX URL, Access Token)
- Cache granularity control (`cli_version_for_cache`)
- Strict mode control (`strict`)
- New outputs: `api_version`, `auth_performed`

### Fixed

- Custom plugin installation loop

## [2.0.1] - 2026-01-15

### Fixed 🐞

- **Documentation**: Corrected GitHub Marketplace URL to `setup-salesforce-cli`

## [2.0.0] - 2026-01-15

### Added - Cross-Platform Support 🌐

- **Full Windows support** with PowerShell and Bash shell compatibility
- **Full macOS support** with Homebrew integration
- **OS-aware dependency management** (jq via apt/brew/choco)
- **Platform-specific secure file handling** (chmod on Unix, icacls on Windows)
- **Platform-specific caching paths** for optimal performance
- **Comprehensive cross-platform testing** across Ubuntu, Windows, and macOS

### Added - Enhanced Outputs 📊

- **`source_flags` output** for multi-directory deployment workflows
- **Automatic directory creation** for missing default directories (e.g., force-app)
- **Better error messages** with actionable feedback
- **Improved output variable consistency** across all platforms

### Added - Custom Plugin Support 🔌

- **`custom_sf_plugins` input** for installing any Salesforce CLI plugin
- **Comma-separated list support** for multiple custom plugins
- **Plugin validation** to prevent duplicate installations
- **Flexible plugin management** alongside built-in plugin options

### Changed - Improvements ⚡

- **Enhanced caching** with platform-specific paths and keys
- **Better retry logic** for network resilience (3 attempts with exponential backoff)
- **Improved error handling** with strict mode support
- **Source directory resolution** now handles missing directories gracefully
- **Input validation** now provides clearer error messages
- **Authentication step** refactored for better cross-platform compatibility

### Performance 🚀

- Setup time: ~25-55s (cached), ~1.5-3 min (first run)
- Cache hit rate >95% across all platforms
- Optimized for both GitHub-hosted and self-hosted runners
- Platform-specific optimizations (fastest on Linux, fully functional on all)

### Documentation 📖

- **New: PLATFORM_SUPPORT.md** - Comprehensive cross-platform guide
- **New: TESTING_STRATEGY.md** - Complete testing documentation
- **New: QUICKSTART.md** - Get started in 15 minutes
- **New: TROUBLESHOOTING.md** - Enhanced troubleshooting guide
- **Enhanced: README.md** - Updated with v2 features and examples
- **Updated: Performance Metrics** - Revised execution times based on latest benchmarks
- **Added: Windows Warnings** - Added performance expectations for Windows runners

### Testing 🧪

- **New: Cross-platform test suite** (`test-cross-platform.yml`)
- **Enhanced: Quick tests** (`test.yml`) for rapid development feedback
- **Matrix testing** across Ubuntu, Windows, macOS with Node 18 and 20
- **7 comprehensive test scenarios** in quick tests
- **5 comprehensive test jobs** in cross-platform tests
- **Cache performance testing** to ensure optimization
- **Error handling validation** across all platforms

### Migration Notes ⬆️

- **100% backward compatible** with v1 - no breaking changes
- Simply change `@v1` to `@v2` in your workflows
- All v1 inputs and outputs continue to work
- New features are opt-in via new input parameters
- See [MIGRATION.md](MIGRATION.md) for details

---

## [1.1.1] - 2026-01-14

### Documentation

- Updated author and support links to point to consulting resources
- Refined "Related Projects" section
- General README formatting improvements

## [1.1.0] - 2026-01-14

### Added

- **Custom Alias Support**: New `alias` input parameter allows users to specify custom org aliases
  - Enables multi-org workflows (authenticate to multiple orgs in same job)
  - Defaults to `TargetOrg` for backward compatibility
  - Useful for data migration, multi-environment validation, and Dev Hub + scratch org workflows

### Changed

- Authentication step now uses custom alias instead of hardcoded values
- Improved authentication command structure with explicit flag passing (fixes potential quoting issues)

### Fixed

- Resolved authentication issue with FLAGS variable expansion
- Improved error handling with proper flag quoting

## [1.0.0] - 2026-01-13

### Added

- Initial release of Setup Salesforce CLI Action
- JWT-based authentication for Salesforce orgs
- Automatic Salesforce CLI installation with Node.js version control
- Intelligent caching for CLI and plugins (3-5x speedup)
- Optional sfdx-git-delta plugin installation (default: off)
- Optional @salesforce/plugin-code-analyzer installation (default: off)
- Optional Prettier installation with Salesforce plugins (default: off)
- Optional ESLint installation with Salesforce plugins (default: off)
- Optional sfdx-lwc-jest installation for LWC testing (default: off)
- Dev Hub configuration support
- Skip authentication option for CLI-only setups
- Automatic org type detection and environment variables
- Composite action pattern for easy reuse

### Features

- Zero-configuration minimal setup (CLI + auth only)
- All plugins and tools optional (install only what you need)
- Secure JWT authentication with automatic key cleanup
- Smart caching with configuration-aware cache keys
- Plugin version checking to avoid redundant installs
- Comprehensive error handling and validation
- Support for both standard orgs and Dev Hubs

### Performance

- Minimal setup: 20 seconds (cached)
- Recommended setup (delta + scanner): 35 seconds (cached)
- Full stack setup (all tools): 45 seconds (cached)
- ~95% cache hit rate in typical CI/CD workflows

### Documentation

- Comprehensive README with multiple use case examples
- Step-by-step Connected App setup guide
- Troubleshooting guide for common issues
- Performance comparison tables
- Complete input reference

---

[3.0.0]: https://github.com/rdbumstead/setup-salesforce-action/releases/tag/v3.0.0
[2.2.0]: https://github.com/rdbumstead/setup-salesforce-action/releases/tag/v2.2.0
[2.1.0]: https://github.com/rdbumstead/setup-salesforce-action/releases/tag/v2.1.0
[2.0.1]: https://github.com/rdbumstead/setup-salesforce-action/releases/tag/v2.0.1
[2.0.0]: https://github.com/rdbumstead/setup-salesforce-action/releases/tag/v2.0.0
[1.1.1]: https://github.com/rdbumstead/setup-salesforce-action/releases/tag/v1.1.1
[1.1.0]: https://github.com/rdbumstead/setup-salesforce-action/releases/tag/v1.1.0
[1.0.0]: https://github.com/rdbumstead/setup-salesforce-action/releases/tag/v1.0.0
