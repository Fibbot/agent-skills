---
name: security-audit-pii
description: Use when an app stores personal data (names, emails, health, financial, contact details) and the user asks whether it's safe to ship, wants a privacy or PII review, or needs a whole-codebase security audit before launch.
---

# PII Security Audit

A whole-codebase audit centered on the **PII lifecycle**, with a baseline vulnerability sweep and a **go/no-go gate**. The built-in `/security-review` only reviews pending branch changes. Use it on PRs; it is not a substitute for this.

**Authorization:** if the app has more than one role, tier, or tenant, **REQUIRED SUB-SKILL:** use `authz-matrix-audit` first. This audit consumes its matrix and `OVER` findings; don't re-derive them here.

Report only. Fix what the user approves.

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
- **Access:** covered by `authz-matrix-audit` (multi-role apps) or the ownership check below (single-role apps). Plus least privilege for services and humans.
- **Logging:** no PII in logs (bodies, headers, query params); redact or salted-hash identifiers.
- **Retention/Deletion:** enforced retention windows and deletion workflows; backups/replicas accounted for.
- **Export:** protected, rate-limited, audited; no mass-export/scraping path.

### Data minimization & exposure

- Responses contain only necessary fields; serializers/DTOs private-by-default.
- Pagination on all user-data list endpoints.
- No account-existence leaks via timing or error messages.

### Ownership (single-role apps only)

For an app with one role and no tenants: every resource fetched by ID checks it belongs to the session user, and list endpoints filter by that user. Anything with more roles or tenants goes through `authz-matrix-audit`.

## Baseline vulnerability sweep (brief)

- Injection: SQL/command built from input; raw query helpers.
- XSS: raw HTML rendering (`dangerouslySetInnerHTML`, `v-html`, markdown without sanitizing).
- CSRF on cookie-authenticated mutations; SSRF on any server-side fetch of user-supplied URLs.
- Secrets: in the repo, in git history, or shipped in the client bundle (grep build output for keys; any `NEXT_PUBLIC_`/`VITE_` var holding a secret).
- Dependencies: `npm audit` / `pip-audit`; only criticals reachable from app code matter here.

## Production Readiness Gate

The application is **not production-ready** if any of:

- Unauthenticated access to any PII
- Any `OVER` cell on a PII surface in the authz matrix (IDOR, cross-tenant read/write, privilege escalation)
- Known exploitable injection (SQLi / command / SSRF to metadata)
- Secrets in repo or client bundle
- PII sent to logs or third-party services (analytics, error trackers) unredacted

**Recommended at scale, not blockers:** audit logging for PII access, field-level encryption and key rotation, automated retention/deletion, export rate limiting. List them as hardening items when the app is small; promote them when it has real users, regulatory exposure (HIPAA, GDPR), or a team with privileged access.

## Final Report Template

Executive summary with go/no-go → PII map → findings (severity-ordered) → remediation in three buckets: **fix before ship** (gate items), **fix soon**, **hardening backlog**. Verification for each fix is a command or probe the builder runs, not a `testPlan.md` item, unless it needs human eyes.
