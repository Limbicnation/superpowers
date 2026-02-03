---
name: dependency-health-guardian
description: Use when starting a project, reviewing dependencies, or before major releases—to check for CVEs, deprecations, and upgrade paths
---

# Dependency Health Guardian

## Overview

Outdated dependencies are the #1 source of preventable vulnerabilities. This skill systematically audits and remediates dependency health.

**Core principle:** No release without dependency audit. No CVE without remediation plan.

## When to Use

- Starting work on an inherited/legacy codebase
- Before any major release or deployment
- When security scanner flags issues
- Quarterly dependency maintenance
- After seeing "X vulnerabilities found" warnings
- When a dependency announces EOL/deprecation

## The Audit Process

### Phase 1: Discovery

**Identify all dependency manifests:**

| Ecosystem | Manifest Files |
|-----------|----------------|
| Node.js | `package.json`, `package-lock.json`, `yarn.lock` |
| Python | `requirements.txt`, `Pipfile`, `pyproject.toml`, `poetry.lock` |
| Go | `go.mod`, `go.sum` |
| Rust | `Cargo.toml`, `Cargo.lock` |
| Ruby | `Gemfile`, `Gemfile.lock` |
| Java | `pom.xml`, `build.gradle` |

**Generate full dependency tree:**

```bash
# Node.js
npm ls --all

# Python
pip list --outdated
pipdeptree

# Go
go mod graph
```

### Phase 2: Vulnerability Scan

**Run ecosystem-specific scanners:**

```bash
# Node.js
npm audit
npx snyk test

# Python
pip-audit
safety check

# Go
govulncheck ./...

# Multi-ecosystem
trivy fs .
grype .
```

**Output format:**

```
┌─────────────────────────────────────────────────────┐
│ CRITICAL: lodash@4.17.15                            │
│ CVE-2021-23337 - Prototype Pollution                │
│ Fix: Upgrade to 4.17.21                             │
│ Breaking changes: None                              │
└─────────────────────────────────────────────────────┘
```

### Phase 3: Prioritization Matrix

| Priority | Criteria | Action |
|----------|----------|--------|
| 🔴 **P0** | CRITICAL CVE, actively exploited | Fix within 24 hours |
| 🟠 **P1** | HIGH CVE, public exploit available | Fix within 1 week |
| 🟡 **P2** | MEDIUM CVE, deprecation in <6 months | Plan for next sprint |
| 🟢 **P3** | LOW CVE, minor deprecation | Track in backlog |

### Phase 4: Remediation Plan

**For each vulnerable dependency:**

1. **Check upgrade path:**

   ```bash
   npm outdated lodash
   npm view lodash versions
   ```

2. **Review breaking changes:**
   - Check CHANGELOG.md
   - Review migration guides
   - Identify affected code paths

3. **Create upgrade PR:**

   ```bash
   npm install lodash@latest
   npm test  # Verify tests pass
   ```

4. **Document workarounds if upgrade blocked:**

   ```markdown
   ## lodash@4.17.15 → 4.17.21 BLOCKED
   
   Reason: Breaking change in _.merge behavior
   Workaround: Pin to 4.17.20, scheduled for Q2 refactor
   Risk accepted by: @security-team (2024-01-15)
   ```

## Dependency Lock File Hygiene

```bash
# Regenerate lock files periodically
rm package-lock.json && npm install

# Check for integrity
npm ci  # Fails if lock file doesn't match

# Audit lock file diff in PRs
git diff package-lock.json | grep "integrity"
```

## Automated Monitoring Setup

**GitHub Dependabot (.github/dependabot.yml):**

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
```

**GitLab Dependency Scanning:**

```yaml
include:
  - template: Security/Dependency-Scanning.gitlab-ci.yml
```

## Common Patterns

### Transitive Vulnerability

```
your-app → lib-a@1.0 → vulnerable-dep@1.0

Fix: Check if lib-a has updated version using non-vulnerable dep
     Or: Use npm overrides / resolutions to force version
```

### Version Conflict

```
lib-a requires lodash@^4.0
lib-b requires lodash@^3.0

Fix: Find compatible version, or update lib-b, or use aliases
```

## Quick Reference

```
Dependency Audit Checklist:

□ Run vulnerability scanner (npm audit, pip-audit, etc.)
□ Prioritize by severity (CRITICAL → LOW)
□ Check upgrade paths for top issues
□ Review breaking changes
□ Update with test verification
□ Document any accepted risks
□ Set up automated monitoring
□ Schedule next audit (quarterly)
```

## Integration with Existing Skills

- **REQUIRED:** Use `superpowers:test-driven-development` to verify upgrades don't break functionality
- **RECOMMENDED:** Use `superpowers:writing-plans` for complex multi-dependency upgrades
