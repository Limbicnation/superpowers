---
name: security-threat-modeler
description: Use when reviewing PRs, implementing features with user input, authentication, authorization, or data handling—before merge
---

# Security Threat Modeler

## Overview

Security vulnerabilities slip through because they're not checked systematically. This skill applies OWASP Top 10 analysis to every code change.

**Core principle:** Every code change touching user input, auth, or data requires threat assessment BEFORE merge.

## When to Use

- PR/code review involving user-facing features
- Any code handling user input (forms, APIs, file uploads)
- Authentication/authorization changes
- Database queries with dynamic values
- File system operations with user-provided paths
- External API integrations
- Secrets or credential handling

**Always use when you see:**

- String concatenation in queries
- `eval()`, `exec()`, or dynamic code execution
- Missing input validation
- Hardcoded credentials
- Disabled security headers

## The OWASP Top 10 Checklist

For each code change, check systematically:

| Rank | Vulnerability | What to Look For |
|------|--------------|------------------|
| A01 | **Broken Access Control** | Missing auth checks, IDOR (direct object references), privilege escalation |
| A02 | **Cryptographic Failures** | Weak algorithms, plaintext secrets, missing HTTPS |
| A03 | **Injection** | SQL, NoSQL, OS command, LDAP, XPath injection |
| A04 | **Insecure Design** | Missing rate limiting, no abuse case consideration |
| A05 | **Security Misconfiguration** | Debug enabled, default credentials, verbose errors |
| A06 | **Vulnerable Components** | Outdated dependencies with known CVEs |
| A07 | **Auth Failures** | Weak passwords, missing MFA, session fixation |
| A08 | **Data Integrity Failures** | Unsigned updates, insecure deserialization |
| A09 | **Logging Failures** | Missing audit logs, logging sensitive data |
| A10 | **SSRF** | User-controlled URLs, internal network access |

## Taint Analysis Process

**Track data flow from untrusted sources to dangerous sinks:**

```
SOURCE (Untrusted)          →    SINK (Dangerous)
─────────────────────────────────────────────────
request.body                →    SQL query
request.params              →    File path
request.headers             →    eval/exec
user input                  →    HTML output (XSS)
file upload                 →    filesystem write
environment variable        →    command execution
```

**For each data path:**

1. Identify the source (where does data enter?)
2. Trace through transformations
3. Check: Is input validated/sanitized before sink?
4. If not → VULNERABILITY

## Severity Classification

| Level | Criteria | Action |
|-------|----------|--------|
| 🔴 **CRITICAL** | RCE, auth bypass, data breach | Block merge, fix immediately |
| 🟠 **HIGH** | SQL injection, XSS, IDOR | Block merge, prioritize fix |
| 🟡 **MEDIUM** | Info disclosure, missing rate limit | Fix before production |
| 🟢 **LOW** | Minor info leak, best practice | Track in backlog |

## Output Format

**For each finding:**

```markdown
🔴 CRITICAL: SQL Injection (Line 45)

**Vulnerable Code:**
query = f"SELECT * FROM users WHERE id = {user_input}"

**Attack Vector:**
user_input = "1; DROP TABLE users--"

**Fix:**
query = "SELECT * FROM users WHERE id = ?"
cursor.execute(query, (user_input,))

**Test Case:**
def test_sql_injection_prevented():
    malicious = "1; DROP TABLE users--"
    response = client.get(f"/users/{malicious}")
    assert response.status_code == 400  # Rejected
```

## STRIDE Threat Model (For Features)

When reviewing a new feature, document threats:

| Threat | Question | Mitigation |
|--------|----------|------------|
| **S**poofing | Can attacker impersonate user? | Strong auth, MFA |
| **T**ampering | Can data be modified? | Input validation, checksums |
| **R**epudiation | Can actions be denied? | Audit logging |
| **I**nformation Disclosure | Can data leak? | Encryption, access control |
| **D**enial of Service | Can service be overwhelmed? | Rate limiting |
| **E**levation of Privilege | Can attacker gain access? | Principle of least privilege |

## Integration with Existing Skills

- **REQUIRED:** Use `superpowers:test-driven-development` to write security test cases
- **REQUIRED:** Use `superpowers:verification-before-completion` to confirm fixes work
- **RECOMMENDED:** Use `superpowers:systematic-debugging` if vulnerability is complex

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "It's internal only" | Internal apps get compromised. Apply same rigor. |
| "We'll add security later" | Security debt compounds. Fix now. |
| "Input is validated elsewhere" | Verify. Trust but verify. Defense in depth. |
| "Low priority feature" | Attackers target low-priority features. |
| "Just a quick fix" | Quick fixes introduce vulnerabilities. |

## Quick Reference

```
Before merging ANY code:

□ User input → validated/sanitized?
□ SQL queries → parameterized?
□ Auth → required on all endpoints?
□ Secrets → not hardcoded?
□ Dependencies → no known CVEs?
□ Errors → no stack traces exposed?
□ Logging → no sensitive data logged?
```
