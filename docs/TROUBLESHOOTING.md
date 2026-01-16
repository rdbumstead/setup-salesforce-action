# Troubleshooting Guide

## 🐞 Common Issues and Solutions

---

## Installation Issues

### Network Timeout During npm Install

**Problem**: npm install fails with network timeout

```
npm ERR! network timeout
npm ERR! network This is a problem related to network connectivity
```

**Solution**: The action now includes automatic retry logic (3 attempts). If it still fails:

1. Check GitHub Actions outage status
2. Try running the workflow again
3. Use a specific CLI version instead of "latest"

```yaml
- uses: rdbumstead/setup-salesforce-action@v2
  with:
    cli_version: "2.30.8" # Pin to specific version
```

---

### Plugin Installation Fails

**Problem**: Custom plugin fails to install

```
Error installing plugin: network error
```

**Solution**:

1. **Check plugin name** - Ensure it's correct and available
2. **Try with strict mode off**:

```yaml
- uses: rdbumstead/setup-salesforce-action@v2
  with:
    custom_sf_plugins: "your-plugin"
    strict: "false" # Continue on plugin errors
```

3. **Install manually** after action:

```yaml
- uses: rdbumstead/setup-salesforce-action@v2
- run: echo y | sf plugins install your-plugin
```

---

### Disk Space Issues

**Problem**: Runner out of disk space

```
Error: No space left on device
```

**Solution**:

1. **GitHub-hosted runners**: GitHub automatically manages cache (10GB limit per repo)
2. **Self-hosted runners**: Clear old caches manually

```bash
# On self-hosted runner
rm -rf ~/.npm/_cacache
rm -rf ~/.local/share/sf
```

3. **Reduce tools installed**:

```yaml
# Only install what you need
install_delta: "true"
# Don't install: scanner, prettier, eslint, jest if not needed
```

---

## Authentication Issues

### JWT Authentication Fails

**Problem**: Authentication fails with "invalid JWT"

```
Error: JWT validation failed
```

**Checklist**:

- [ ] JWT key format is correct (begins with `-----BEGIN RSA PRIVATE KEY-----`)
- [ ] No extra whitespace or line breaks in secret
- [ ] Certificate uploaded to Connected App matches the key
- [ ] Client ID (Consumer Key) is correct
- [ ] Username is correct
- [ ] Instance URL is correct (`login.salesforce.com` for prod, `test.salesforce.com` for sandbox)

**Solution**:

```bash
# Verify JWT key format locally
cat server.key | head -1
# Should show: -----BEGIN RSA PRIVATE KEY-----

# Check for line ending issues
cat server.key | file -
# Should NOT show "with CRLF line terminators"
```

**Fix for Windows line endings**:

```bash
dos2unix server.key
```

---

### External Client Apps Not Enabled (Winter '25+)

**Problem**: Authentication fails with "Connected App is not authorized"

```
Error: The client application is not authorized to access this organization
Error: invalid_client_id: client identifier invalid
```

**Cause**: Winter '25+ orgs require External Client Apps to be enabled before Connected Apps can authenticate via JWT.

**Solution**:

1. **Enable External Client Apps** in your Salesforce org:

   ```
   Setup → Quick Find → "External Client Apps"
   → Settings (in External Client Apps section)
   → Toggle ON "Enable External Client Apps"
   → Save
   ```

2. **Verify your Connected App settings**:

   ```
   Setup → App Manager → [Your Connected App]
   → Edit Policies
   → OAuth Policies → Permitted Users: "All users may self-authorize"
   → Save
   ```

3. **Wait 2-10 minutes** for the changes to propagate

4. **Retry your workflow**

**Reference**: [Salesforce Winter '25 Release Notes](https://help.salesforce.com/s/articleView?id=release-notes.rn_security_external_client_apps.htm)

---

### Cannot Create Connected App (Winter '25+)

**Problem**: "New Connected App" button is grayed out or missing

```
Error: You don't have permission to create Connected Apps
```

**Solutions**:

**Option 1: Enable External Client Apps First**

```
Setup → Quick Find → "External Client Apps"
→ Settings
→ Toggle ON "Enable External Client Apps"
→ Save
→ Return to App Manager → New Connected App (now available)
```

**Option 2: Check User Permissions**

- Ensure you have "Customize Application" permission
- Or "Manage Connected Apps" permission
- System Administrator profile has these by default

**Option 3: Create External Client App Instead** (recommended)

As of Winter '25, External Client Apps are the recommended approach:

```
Setup → Quick Find → "External Client Apps"
→ New
→ Fill in app details
→ Enable OAuth Settings (same as Connected App)
→ Use digital signatures
→ Upload certificate
→ Add OAuth scopes: api, refresh_token, offline_access
→ Save
```

External Client Apps work identically with this action - just use the Client ID and JWT key as normal.

**Benefits of External Client Apps:**

- Enhanced security controls
- Better audit logging
- Recommended by Salesforce for CI/CD
- Same authentication flow

---

## 🆕 Winter '25 Changes Quick Reference

| Issue                          | Solution                                  |
| ------------------------------ | ----------------------------------------- |
| "New Connected App" grayed out | Enable External Client Apps in Settings   |
| "client identifier invalid"    | Enable External Client Apps + wait 10 min |
| "not authorized to access"     | Check OAuth policies in Connected App     |
| Can't create Connected App     | Use External Client App instead           |

---

**Note for Documentation**: This change affects all orgs upgraded to Winter '25 or later. Teams using older org versions can continue using the original Connected App creation flow without enabling External Client Apps.

### Wrong Instance URL

**Problem**: Authentication fails for sandbox org

```
Error: invalid_grant: authentication failure
```

**Solution**: Use correct instance URL

```yaml
- uses: rdbumstead/setup-salesforce-action@v2
  with:
    instance_url: "https://test.salesforce.com" # For sandboxes
```

---

### SFDX Auth URL Issues (v2.1+)

**Problem**: SFDX Auth URL authentication fails

```
Error: Invalid SFDX Auth URL format
```

**Checklist**:

- [ ] Auth URL starts with `force://`
- [ ] Auth URL was copied completely
- [ ] No extra whitespace or line breaks in secret
- [ ] The org's session is still valid

**Solution to get a fresh Auth URL**:

```bash
# Authenticate first (if needed)
sf org login web --alias YourOrg

# Get the SFDX Auth URL
sf org display --target-org YourOrg --verbose --json | jq -r '.result.sfdxAuthUrl'
```

Store the full URL as a secret named `SFDX_AUTH_URL`.

---

### Access Token Issues (v2.1+)

**Problem**: Access token authentication fails

```
Error: INVALID_SESSION_ID: Session expired or invalid
```

**Common causes**:

1. **Token expired** - Access tokens are short-lived (typically 2 hours)
2. **Instance URL mismatch** - Token domain doesn't match instance_url
3. **Token revoked** - Session was invalidated

**Solution**:

1. **Generate a fresh token** just before running the workflow
2. **Use JWT or SFDX Auth URL** for production CI/CD (recommended)
3. **Verify instance URL matches the token's org**

```yaml
- uses: rdbumstead/setup-salesforce-action@v2
  with:
    auth_method: "access-token"
    access_token: ${{ secrets.SF_ACCESS_TOKEN }}
    instance_url: "https://your-org.my.salesforce.com" # Must match token
```

> ⚠️ **Note**: Access tokens are recommended only for manual/ad-hoc deployments, not production CI/CD.

---

## Caching Issues

### Cache Not Working

**Problem**: Action always installs from scratch

```
Installing Salesforce CLI...
Installing sfdx-git-delta...
```

**Diagnosis**:

1. Check cache hit in action logs:

```
Cache restored from key: sf-v2-Linux-node20-...
```

**Solution**:

1. **First run**: Cache miss is normal
2. **Subsequent runs**: Should show cache hit
3. **Cache invalidation**: Happens when:
   - Node version changes
   - CLI version changes
   - Tools/plugins change
   - OS changes

**Force cache refresh**:

```yaml
# Change any input to invalidate cache
node_version: "20" # Change to "18" temporarily
```

---

### Stale Cache

**Problem**: Old CLI version cached

```
Salesforce CLI already installed
Installed: @salesforce/cli/2.25.0  # But you want 2.30.8
```

**Solution**: Clear cache by changing version

```yaml
# Method 1: Specify exact version
cli_version: "2.30.8"

# Method 2: Change Node version temporarily
node_version: "18" # Then change back to "20"
```

---

## Plugin Issues

### Plugin Version Conflicts

**Problem**: Custom plugin conflicts with built-in

```
Plugin conflict detected
```

**Solution**:

1. **Remove built-in**: Don't install conflicting built-in plugin
2. **Use only custom**:

```yaml
install_delta: "false" # Don't install built-in
custom_sf_plugins: "sfdx-git-delta@7.0.0" # Install specific version
```

---

### Plugin Not Found

**Problem**: Custom plugin doesn't exist

```
Error: Plugin not found: my-plugin
```

**Solution**:

1. **Verify plugin name** on npm: https://www.npmjs.com/package/your-plugin
2. **Check spelling**: Case-sensitive
3. **Try scoped name**: `@scope/plugin-name`

---

## Directory Issues

### Source Directory Not Found

**Problem**: Deploy fails with directory not found

```
Error: Directory 'force-app' not found
```

**Solution**: The action creates directories automatically, but if you have a custom structure:

```yaml
- uses: rdbumstead/setup-salesforce-action@v2
  with:
    source_dirs: "src,packages/core" # Your actual directories
```

---

### Multiple Directories Not Working

**Problem**: Only first directory deploying

```
Deploying from: force-app
# packages/core not included
```

**Solution**: Use the `source_flags` output

```yaml
- id: setup
  uses: rdbumstead/setup-salesforce-action@v2
  with:
    source_dirs: "force-app,packages/core"

- run: sf project deploy start ${{ steps.setup.outputs.source_flags }}
```

---

## Platform-Specific Issues

### Windows: jq Not Found

**Problem**: jq command fails on Windows

```
jq: command not found
```

**Solution**:

1. **GitHub-hosted**: Should install automatically via Chocolatey
2. **Self-hosted**: Install Chocolatey first

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

---

### macOS: Homebrew Errors

**Problem**: brew command fails

```
Error: brew: command not found
```

**Solution**:

1. **GitHub-hosted**: Homebrew pre-installed
2. **Self-hosted**: Install Homebrew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

---

### Windows: Path Length Issues

**Problem**: File paths too long

```
Error: Path too long
```

**Solution**: Enable long paths

```powershell
# As Administrator
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD -Force
```

Or use shorter project paths.

---

## Output Issues

### Outputs Are Empty

**Problem**: Step outputs return nothing

```
org_id:
org_type:
```

**Solution**:

1. **Check authentication**: Outputs only populated if `skip_auth: "false"`
2. **Check step ID**: Must have `id:` to reference outputs

```yaml
- id: setup-sf # Required!
  uses: rdbumstead/setup-salesforce-action@v2

- run: echo "${{ steps.setup-sf.outputs.org_id }}"
```

---

### source_flags Empty

**Problem**: source_flags output is empty

```
source_flags:
```

**Solution**: Provide source_dirs input

```yaml
- id: setup
  uses: rdbumstead/setup-salesforce-action@v2
  with:
    source_dirs: "force-app" # Explicit
```

---

## Performance Issues

### Action Takes Too Long

**Problem**: Action runs for 5+ minutes

```
Installing tools... (taking forever)
```

**Optimization**:

1. **Are you using a Windows runner?**
   Windows runners on GitHub Actions have significantly slower I/O than Linux.

   - **Solution**: Switch to `runs-on: ubuntu-latest` if cross-platform support isn't strictly required.

2. **Only install needed tools**:

```yaml
# Don't install everything
install_scanner: "false"
install_prettier: "false"
install_eslint: "false"
install_lwc_jest: "false"
```

2. **Use cache** (automatic on second run)

3. **Pin CLI version** (faster than "latest")

```yaml
cli_version: "2.30.8"
```

---

## Error Handling

### Cryptic Error Messages

**Problem**: Error doesn't explain what's wrong

```
Error: Command failed
```

**Solution**: Enable verbose output

```yaml
- uses: rdbumstead/setup-salesforce-action@v2
  with:
    strict: "true" # Fail with clear errors
```

Then check full action logs in GitHub Actions UI.

---

## Getting More Help

### Still Stuck?

1. **Check full action logs**:

   - Go to GitHub Actions → Your workflow run → Expand each step

2. **Search existing issues**:

   - https://github.com/rdbumstead/setup-salesforce-action/issues

3. **Open a new issue** with:

   - Complete error message
   - Workflow file (remove secrets!)
   - Action logs
   - Environment (OS, Node version)

4. **Ask in Discussions**:
   - https://github.com/rdbumstead/setup-salesforce-action/discussions

---

## Quick Reference

### Common Solutions

| Problem             | Quick Fix                      |
| ------------------- | ------------------------------ |
| Network timeout     | Wait and retry                 |
| Auth fails          | Check JWT key format           |
| Cache stale         | Change CLI version             |
| Plugin conflict     | Don't install built-in version |
| Directory not found | Check source_dirs input        |
| Outputs empty       | Add step ID                    |
| Too slow            | Install fewer tools            |
| Windows jq error    | Use GitHub-hosted runner       |

---

## Debug Mode

For maximum verbosity, use all these together:

```yaml
- uses: rdbumstead/setup-salesforce-action@v2
  with:
    strict: "true"
    # ... your other inputs

- name: Debug Info
  run: |
    echo "SF CLI Version:"
    sf --version
    echo "Installed Plugins:"
    sf plugins
    echo "Orgs:"
    sf org list
    echo "Environment:"
    env | grep -i sf || true
```

---

_Last updated: January 2026_
