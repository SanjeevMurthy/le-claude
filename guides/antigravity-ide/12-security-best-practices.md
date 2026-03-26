# 12 — Security Best Practices

## Overview

Antigravity's powerful autonomous capabilities introduce unique security considerations. Agents that can read files, execute commands, and browse the web require carefully configured security boundaries. This chapter documents security risks identified by researchers and the best practices to mitigate them.

## Threat Model

```
+------------------------------------------------------------------+
|                ANTIGRAVITY THREAT MODEL                            |
|                                                                   |
|  ┌──────────────────────────────────────────────────────────┐    |
|  │                  ATTACK SURFACES                         │    |
|  │                                                          │    |
|  │  1. PROMPT INJECTION                                     │    |
|  │     Malicious instructions embedded in:                  │    |
|  │     ├── Code comments                                    │    |
|  │     ├── README files                                     │    |
|  │     ├── Package metadata                                 │    |
|  │     ├── Web pages (via read_url_content)                 │    |
|  │     └── MCP server responses                             │    |
|  │                                                          │    |
|  │  2. DATA EXFILTRATION                                    │    |
|  │     Agent reads sensitive data and sends it externally:  │    |
|  │     ├── API keys from .env files                         │    |
|  │     ├── Database credentials                             │    |
|  │     ├── Private keys                                     │    |
|  │     └── Proprietary source code                          │    |
|  │                                                          │    |
|  │  3. REMOTE COMMAND EXECUTION                             │    |
|  │     Agent executes malicious commands via:               │    |
|  │     ├── Terminal command injection                        │    |
|  │     ├── Script execution                                 │    |
|  │     └── Package install hooks                            │    |
|  │                                                          │    |
|  │  4. SUPPLY CHAIN ATTACKS                                 │    |
|  │     Malicious dependencies introduced by:                │    |
|  │     ├── Agent adding untrusted packages                  │    |
|  │     ├── Auto-updating to vulnerable versions             │    |
|  │     └── Executing post-install scripts                   │    |
|  └──────────────────────────────────────────────────────────┘    |
+------------------------------------------------------------------+
```

## Known Vulnerabilities

Based on security research (notably from embracethered.com):

| Vulnerability | Risk | Mitigation |
|---|---|---|
| **Data exfiltration via `read_url_content`** | Agent can send collected data to external URL encoded in request parameters | Configure network restrictions; review all external URL accesses |
| **Indirect prompt injection** | Hidden instructions in code/data cause agent to perform unintended actions | Review agent actions; use Planning mode for oversight |
| **Terminal injection** | Agent executes commands constructed from untrusted input | Configure deny lists; avoid Turbo mode for untrusted codebases |
| **Markdown image rendering** | Attacker embeds tracking pixels to exfiltrate data via image URLs | Disable auto-rendering of external images |

## Security Configuration Hierarchy

### Level 1: Maximum Security (Production/Sensitive Code)

```markdown
# .agents/rules/security-maximum.md
---
description: "Maximum security configuration for sensitive codebases"
activation: always
---

# Maximum Security Rules

## Agent Restrictions
- Review policy: Request review (manual approval for all changes)
- Terminal execution: Off (no auto-execution)
- Never access external URLs without developer approval
- Never install new dependencies without developer approval
- Never modify .env, .gitignore, or configuration files without review

## Code Security
- All secrets must be in environment variables
- Never log sensitive data (passwords, tokens, PII, credit cards)
- Input validation required at all system boundaries
- Output encoding required for all user-facing output
- SQL queries must use parameterized statements
- File paths must be validated (no path traversal)

## Dependency Security
- Run `npm audit` before approving any dependency change
- Pin all dependency versions (no `^` or `~` ranges)
- Review changelogs for major version updates
- Never use `--force` or `--legacy-peer-deps` without justification
```

### Level 2: Balanced Security (Team Development)

```markdown
# .agents/rules/security-balanced.md
---
description: "Balanced security for team development"
activation: always
---

# Balanced Security Rules

## Agent Configuration
- Review policy: Agent decides
- Terminal execution: Auto (with allow/deny lists)
- External URL access: Allowed for documentation sites only
- Dependency installation: Agent can add well-known packages

## Automated Checks
- Run `npm audit` after any dependency change
- Run SAST scan on modified files
- Check for hardcoded secrets before committing
- Validate CORS configuration on API changes
```

### Level 3: Maximum Autonomy (Personal/Experimental)

```markdown
# .agents/rules/security-autonomous.md
---
description: "Maximum autonomy for personal projects"
activation: always
---

# Autonomous Security Rules

## Agent Configuration
- Review policy: Always proceed
- Terminal execution: Turbo
- External access: Unrestricted
- Dependency installation: Unrestricted

## Still Required
- No committing secrets to git
- Run `npm audit` periodically
- Use HTTPS for all external connections
```

## Secure Coding Skills

### Secure coding Skill for Agents

```markdown
---
name: "Secure Coding Practice"
description: "Security-focused development skill that enforces OWASP Top 10 protections, input validation, authentication best practices, and secure data handling."
---

# Secure Coding Skill

## OWASP Top 10 Checklist

### A01 — Broken Access Control
- [ ] Authorization checks at every endpoint
- [ ] Principle of least privilege
- [ ] CORS properly configured
- [ ] Rate limiting implemented

### A02 — Cryptographic Failures
- [ ] TLS for all data in transit
- [ ] Strong hashing for passwords (bcrypt, argon2)
- [ ] No sensitive data in URLs
- [ ] Encryption for data at rest (PII, financial data)

### A03 — Injection
- [ ] Parameterized queries for all database operations
- [ ] Input validation with allowlists
- [ ] Output encoding for HTML/JS contexts
- [ ] Command injection prevention (no shell exec with user input)

### A04 — Insecure Design
- [ ] Threat model documented
- [ ] Failure modes identified
- [ ] Security controls centralized (not per-endpoint)

### A05 — Security Misconfiguration
- [ ] Default credentials changed
- [ ] Error messages don't expose internals
- [ ] Security headers configured (CSP, HSTS, X-Frame-Options)
- [ ] Debug mode disabled in production

### A06 — Vulnerable Components
- [ ] Dependencies up to date
- [ ] No known vulnerabilities (npm audit / pip audit)
- [ ] Unused dependencies removed

### A07 — Authentication Failures
- [ ] Multi-factor authentication for admin
- [ ] Password complexity requirements
- [ ] Account lockout after failed attempts
- [ ] Session timeout configured

### A08 — Software & Data Integrity
- [ ] CI/CD pipeline integrity checks
- [ ] Dependency integrity verification (lockfile)
- [ ] Signed commits required

### A09 — Security Logging & Monitoring
- [ ] Authentication events logged
- [ ] Authorization failures logged
- [ ] Input validation failures logged
- [ ] No sensitive data in logs

### A10 — Server-Side Request Forgery
- [ ] URL validation for server-side requests
- [ ] Allowlist for outbound connections
- [ ] No user-controlled URLs in server requests
```

## Dependency Vulnerability Scanning

### Automated Scanning Workflow

```markdown
# .agents/workflows/security-scan.md
---
description: "Run comprehensive security scan on the codebase"
---

## Steps

1. Run dependency audit:
// turbo
2. Run: `npm audit --json > /tmp/npm-audit.json`
3. Parse the audit results and summarize critical/high/medium/low counts

4. Run secret scanning:
// turbo
5. Run: `npx secretlint "**/*" --format json || true`
6. Report any detected secrets

7. Run SAST (if configured):
// turbo
8. Run: `npx eslint --rule '{"security/detect-object-injection": "error", "security/detect-non-literal-require": "error"}' src/ || true`

9. Check for outdated dependencies:
// turbo
10. Run: `npm outdated --json || true`

11. Produce a security report summarizing all findings
```

## Secrets Detection

### Pre-Commit Hook Pattern

```bash
#!/bin/bash
# .git/hooks/pre-commit
# Prevent committing secrets

patterns=(
  'AKIA[0-9A-Z]{16}'           # AWS Access Key
  'AIza[0-9A-Za-z_-]{35}'      # Google API Key
  'sk-[a-zA-Z0-9]{48}'         # OpenAI API Key
  'ghp_[a-zA-Z0-9]{36}'        # GitHub Personal Access Token
  'password\s*=\s*["\x27].+["\x27]'  # Hardcoded passwords
)

for pattern in "${patterns[@]}"; do
  if git diff --cached | grep -qE "$pattern"; then
    echo "ERROR: Potential secret detected matching pattern: $pattern"
    echo "Please remove secrets before committing."
    exit 1
  fi
done
```

### Agent Rule for Secrets

```markdown
# .agents/rules/no-secrets.md
---
description: "Prevent agents from committing secrets or credentials"
activation: always
---

# No Secrets Rule

NEVER include in any file:
- API keys, tokens, or credentials
- Database connection strings with passwords
- Private keys or certificates
- Encryption keys
- OAuth client secrets

ALWAYS:
- Use environment variables for all secrets
- Reference .env.example for required variables
- Use placeholder values in configuration examples
- Never log sensitive values (mask with ***)
```

## Infrastructure Security

### Docker Security

```dockerfile
# Secure Dockerfile pattern
FROM node:20-slim AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM node:20-slim
# Run as non-root user
RUN addgroup --system app && adduser --system --group app
WORKDIR /app
COPY --from=builder --chown=app:app /app/dist ./dist
COPY --from=builder --chown=app:app /app/node_modules ./node_modules
USER app
EXPOSE 3000
HEALTHCHECK --interval=30s CMD wget -qO- http://localhost:3000/health || exit 1
CMD ["node", "dist/index.js"]
```

---

*Previous: [11 — Advanced Configurations](./11-advanced-configurations.md) | Next: [13 — Performance Optimization](./13-performance-optimization.md)*
