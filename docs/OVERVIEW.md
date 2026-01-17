# Setup Salesforce CLI Action – Overview

This document explains what this action is, what it intentionally is **not**, and how it fits into a larger Salesforce CI/CD architecture.

---

## 🎯 Design Philosophy

This action is intentionally **foundational**.

It does **not**:

- Enforce a deployment strategy
- Dictate branching models
- Replace CI/CD orchestration tools

Instead, it provides a **deterministic execution layer** that workflows can safely depend on.

> Think of this action as **infrastructure**, not automation.

---

## 🧱 What This Action Provides

### Deterministic CLI Environment

- Controlled Node.js version
- Explicit Salesforce CLI versioning
- Optional cache granularity (`major`, `minor`, `exact`)

### Authentication as a First-Class Concern

- **JWT** authentication for production (using External Client Apps or Connected Apps)
- **SFDX Auth URL** for simplicity
- **Access token** support for advanced use cases

Authentication is **validated** and **introspected**, not assumed.

**Winter '25+ Recommendation**: This action supports both modern **External Client Apps** (recommended for orgs on Winter '25/October 2024 or later) and traditional **Connected Apps**. Both use identical JWT authentication flows and work seamlessly with this action.

---

## 🔐 Authentication Design

This action treats authentication as **infrastructure**, not configuration:

1. **Multiple Methods Supported**:

   - JWT with External Client Apps (recommended for Winter '25+)
   - JWT with Connected Apps (legacy, still supported)
   - SFDX Auth URL (simpler, refresh token based)
   - Access Token (advanced scenarios, short-lived)

2. **Validation First**:

   - All inputs are validated before attempting authentication
   - Clear error messages identify missing or malformed credentials
   - Platform-specific handling (file permissions, secure cleanup)

3. **Introspection**:

   - After authentication, org details are queried and exposed as outputs
   - Org type (Production/Sandbox/Scratch) is detected automatically
   - API version, org edition, and other metadata available for downstream steps

4. **Security**:
   - JWT keys are never logged or exposed
   - Temporary files (JWT keys, auth URLs) are securely deleted after use
   - Platform-specific secure file permissions (chmod on Unix, icacls on Windows)

---

## 📦 Caching Strategy

This action caches:

1. Salesforce CLI installation
2. Installed CLI plugins
3. Node tooling where applicable

Caching is:

- **Predictable**
- **Transparent** in logs
- **Configurable** via `cli_version_for_cache`

The design favors **maintainability** over micro-optimizations.

---

## 📂 Source Directory Resolution

Modern Salesforce projects are often:

- Mono-repos
- Package-based
- Multi-team

This action:

1. Accepts multiple `source_dirs`
2. Ensures directories exist
3. Exposes resolved `source_flags`

This allows downstream workflows to remain **simple** and **generic**.

---

## 🔌 Optional Tooling Model

All tooling is **opt-in**:

- `sfdx-git-delta`
- Salesforce Code Analyzer
- Prettier
- ESLint
- LWC Jest
- Custom plugins

**Strict mode** allows teams to choose whether optional tooling failures should fail the pipeline.

---

## 🧪 Testing & Reliability

This action is tested against:

- **Ubuntu**
- **macOS**
- **Windows**

Tests include:

- Auth paths
- Plugin installation
- Cache behavior
- Error handling
- Invalid input scenarios

The goal is **predictable failure**, not silent success.

---

## 🧩 Intended Usage Pattern

This action is designed to be consumed by:

- Reusable workflows
- Organization-wide CI/CD templates
- Platform teams supporting multiple projects

It pairs naturally with:

- Reusable validation workflows
- Reusable deployment workflows
- Template repositories

---

## 🛣️ Long-Term Direction

This action is the first building block of a larger **Salesforce DevOps platform** that will include:

- Opinionated reusable workflows
- Governance-ready templates
- Consistent validation and deployment stages

Those concerns are intentionally **out of scope** here.

---

## ✅ Summary

If you need:

- A fast, deterministic Salesforce CLI setup
- Secure, non-interactive authentication
- Support for real-world repository structures

**This action is the correct foundation.**
