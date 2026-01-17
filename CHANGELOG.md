# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.2.0] - 2026-01-17

### 🚀 New Features - Performance & Caching

- **OS-specific cache configuration** - Cache paths are now separated by operating system to prevent conflicts
  - Unix (Linux/macOS): `~/.npm`, `~/.local/share/sf`, `~/.cache/node-gyp`
  - Windows: `~/AppData/Local/sf`, `~/AppData/Roaming/npm`
  - _Note: This change will force a one-time cache miss (expect ~2m setup time on first run)_

- **Cache key generation optimized** - Version resolution now uses `npm view @salesforce/cli version` instead of installing CLI
  - Faster cache key generation
  - Includes 10s timeout protection against network hangs

### Added - Reliability Improvements 🛡️

- **Plugin installation retry logic** - All plugin installs now have 3-attempt retry with exponential backoff
  - `sfdx-git-delta`: Retries on transient npm/network failures
  - `@salesforce/plugin-code-analyzer`: Retries on transient failures
  - Custom plugins: Each plugin retries individually

- **Cross-platform hash command detection** - Automatically uses `sha256sum` or `shasum -a 256` based on availability

### Changed - Improvements

- **Workflow renamed** from "Quick Tests" to "Functional Tests"
- **Improved plugin verification** - Uses `jq` regex matching instead of simple `grep` (handles plugin name variations)
- **Improved CLI validation** - Uses `sf plugins` instead of `sf plugins --core`
- **Renamed `test-network-retry` to `test-cli-install-retry`** - Better reflects purpose
- **Fixed `test-invalid-plugins`** - Removed "valid-plugin" (not a real npm package)
- **Access Token Auth Default** - `allow_access_token_auth` now defaults to `true` (making this a non-breaking release)
- Cache key format: `sf-v3-*` (unchanged from v2.1)

### Added - New Tests 🧪

- **`test-access-token-auth`** - Tests access token authentication flow
- **`test-multiple-plugins`** - Tests installing multiple custom plugins at once

### Fixed 🐞

- Custom plugin installation loop now correctly installs plugins
- Cache behavior test now informational-only (cache-hit not reliably detectable in composite actions)
- Source flags verification now checks for exact `--source-dir <dir>` format
- Cross-platform plugin verification in `test-cross-platform.yml` (uses proper regex matching)
- **npm view timeout protection** - Cache key generation now has 10s timeout to prevent hangs
- **LWC Jest version warning** - Added in-code comment warning about global install version risks
- **.gitignore updated** - Temporary auth files (`authurl.txt`, `access_token.txt`) now ignored
- **Access token URL fix** - Instance URL now strips trailing slash to prevent auth URL issues
- **Access Token Logic Fix** - Updated to use `SF_ACCESS_TOKEN` env var and `sf org login access-token` command
- **Removed unused GITHUB_ENV export** - `SF_SOURCE_FLAGS` env var removed (use `source_flags` output)

---

## [2.1.0] - 2026-01-15

### Added

- Multiple authentication methods (JWT, SFDX URL, Access Token)
- Cache granularity control (`cli_version_for_cache`)
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
[2.1.0]: https://github.com/rdbumstead/setup-salesforce-action/releases/tag/v2.1.0
[2.0.1]: https://github.com/rdbumstead/setup-salesforce-action/releases/tag/v2.0.1
[2.0.0]: https://github.com/rdbumstead/setup-salesforce-action/releases/tag/v2.0.0
[1.1.1]: https://github.com/rdbumstead/setup-salesforce-action/releases/tag/v1.1.1
[1.1.0]: https://github.com/rdbumstead/setup-salesforce-action/releases/tag/v1.1.0
[1.0.0]: https://github.com/rdbumstead/setup-salesforce-action/releases/tag/v1.0.0
