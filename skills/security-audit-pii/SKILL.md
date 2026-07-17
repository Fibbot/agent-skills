---
name: security-audit-pii
description: PII-focused security audit with a structured deliverable - PII map, lifecycle checks, severity-rated findings with remediation and verification steps, and a production-readiness gate. Use for apps that store personal data, when asked "is this safe to ship", or to complement a generic /security-review with privacy-specific coverage.
---

# PII Security Audit

Complements the built-in `/security-review` (which covers generic vulnerability classes: injection, XSS, CSRF, secrets, dependencies). This skill adds what that doesn't: the **PII lifecycle**, a **structured findings deliverable**, and a **go/no-go production gate**. Run `/security-review` first, then this.

## Deliverables

1. **Executive Summary** — risk posture, highest-risk issues, go/no-go recommendation.
2. **Findings List** — each finding: ID, Title, Severity, Likelihood, Impact; affected files/endpoints; minimal step-by-step exploit scenario; code evidence; **exact remediation**; **verification steps to confirm the fix**.
3. **PII Map** — where PII is collected, validated, stored, transmitted, logged, exported, deleted.
4. **Threat Model Snapshot** — trust boundaries, attacker types, top abuse cases.

## Severity

- **Critical:** account takeover, auth bypass, RCE, SQLi, mass PII exposure, secrets exfiltration path.
- **High:** privilege escalation, IDOR on sensitive data, SSRF with metadata access, persistent XSS, weak crypto on PII.
- **Medium:** limited exposure, feasible DoS, insecure defaults, missing rate limits on sensitive routes.
- **Low:** hardening gaps without a clear exploit path.

## PII Lifecycle Checks (the core of this audit)

First enumerate every PII field (names, emails, addresses, phone, DOB, IDs, tokens, IPs, device IDs) and derived data (profiles, scores, logs containing identifiers). Then, per field/path:

- **Collection:** minimization, validation, client/server consistency.
- **Transmission:** TLS enforced, secure cookies, no mixed content.
- **Storage:** encryption at rest; field-level encryption for highly sensitive attributes; key management and rotation.
- **Access:** authorization on every read; least privilege for services and humans; audit trails for PII access.
- **Logging:** no PII in logs (bodies, headers, query params); redact or salted-hash identifiers.
- **Retention/Deletion:** enforced retention windows and deletion workflows; backups/replicas accounted for.
- **Export:** protected, rate-limited, audited; no mass-export/scraping path.

### Data minimization & exposure

- Responses contain only necessary fields; serializers/DTOs private-by-default.
- Pagination on all user-data list endpoints.
- No account-existence leaks via timing or error messages.

### Tenant isolation & IDOR (the most common real failure)

- Object ownership checks on every resource fetched by ID.
- Tenant derived from session, never from user-supplied IDs.
- List endpoints filter by tenant/user; verify row-level security policies if used.

## Production Readiness Gate

The application is **not production-ready** if any of:

- Unauthenticated access to any PII
- Any tenant isolation bypass / IDOR exposing data
- Known exploitable injection (SQLi / command / SSRF to metadata)
- Secrets in repo or client bundle
- Missing authorization on any PII-accessing endpoint
- No audit logging for privileged/PII access

## Final Report Template

Executive summary (1 page) → risk register table → PII map → detailed findings → remediation plan by priority (0–7 / 7–30 / 30–90 days) → verification checklist.
