---
name: authz-matrix-audit
description: Use when an app has multiple user roles, tiers, or tenants and you need to know whether each role can actually do only what it should - mapping an authorization / permissions / access-control matrix, hunting IDOR, privilege escalation, cross-tenant leaks, or RLS gaps, or verifying "who can see/do what" before launch.
---

# AuthZ Matrix Audit

## Overview

Build two matrices — **expected** (what each principal *should* be able to do) and **observed** (what it *can* do, proven by probes) — and report the diff. The diff is the deliverable.

**Core principle: the expected matrix never comes from the code.** Reading the code and writing down what it does produces a matrix that agrees with every bug. Expected comes from the spec and the user; observed comes from probes; static reading only tells you where to probe.

**Stack-specific reference:** Supabase / Postgres → [supabase.md](supabase.md). Read it before Phase 3 on a Supabase project.

## Safety contract — read before touching anything

Probing authz means attempting forbidden actions. If authz is broken, the forbidden action *succeeds*. Every rule below exists so that success lands somewhere harmless. **These are not judgment calls. No exceptions, including when the user seems to want speed.**

1. **Local only, enforced in code.** The probe harness starts with a preflight that aborts unless every target (app URL, API URL, DB host) resolves to `localhost` / `127.0.0.1` / `::1`. Hard-coded allowlist. No `--force`, no env override, no "just this once". If the local stack isn't running, stop and tell the user — never fall back to another target.
2. **Explicit config, never ambient env.** The harness reads targets from its own `authz-probe.config` file, not from `.env*`. It aborts if `NODE_ENV=production` or if any configured URL contains a hosted domain (e.g. `supabase.co`, `vercel.app`, the project's prod domain).
3. **No privileged credentials in probes.** Never load a service-role / admin / root key in the harness. It bypasses authz, so every probe it touches passes falsely, and it can do anything. Principals authenticate as seeded users only.
4. **DB writes run inside a transaction that always rolls back.** `BEGIN` → probes (each under a savepoint) → `ROLLBACK` in a `finally`. The harness refuses to execute any SQL containing `COMMIT`, `END`, `DROP`, `TRUNCATE`, `ALTER`, `GRANT`, `REVOKE`, or `CREATE`. Assert row counts before/after the run are identical.
5. **HTTP-level writes are off by default.** Read probes (GET, page loads, list/select) run freely against local. Write probes over HTTP (POST/PUT/PATCH/DELETE, server actions, RPC via REST) cannot be rolled back, so they:
   - run only after the user says yes **in this session**, after you've shown them the exact command and the list of endpoints;
   - target only **sentinel rows** the harness itself identifies (e.g. a seeded row whose name starts `authz_probe_`), never real seeded records;
   - **skip every endpoint with an external side effect** — anything calling email/SMS/push, payments, webhooks, third-party APIs, or file deletion in storage. Grep for the SDKs (Resend, SendGrid, Twilio, Stripe, `fetch(` to external hosts) and list the skipped endpoints in the report as "human-verify".
6. **You never run state-changing project commands.** No DB resets, migrations, pushes, seeds, `link`, or deploys — not to set up, not to clean up. If the seed lacks a principal you need (e.g. a second tenant), stop and ask the user to add it. The user runs resets.
7. **Show before first run.** Before the first execution, print the resolved targets, the principal list, and the mode (read-only / DB-rollback / HTTP-write), and wait for the user's go-ahead.

**Red flags — stop if you catch yourself thinking:** "it's only the dev project", "the service key would make setup easier", "I'll clean up the rows after", "this endpoint is probably safe to POST", "the reset is quick". All mean: you are about to break the contract.

## Phase 1 — Inventory (read-only)

**Principals** — not just roles. Include each of:
- anonymous / signed-out
- every role / tier
- same role, **different owner** (user A vs user B's record)
- same role, **different tenant / org / office**
- lifecycle states: invited-not-accepted, deactivated, downgraded mid-session, admin impersonating

**Surfaces** — every entry point, not just pages:
- pages/routes (and what the middleware guards vs what the page itself checks)
- API routes, server actions, RPC/database functions
- tables/views (row-level policies), storage buckets/objects, realtime channels
- background jobs, webhooks, cron, edge functions, exports/downloads

**Actions** — list, read-one, create, update (per sensitive field if the tier differs), delete, plus domain actions (invite, approve, export, impersonate).

Write the coverage list. Every surface × action must end up in the matrix or be explicitly excluded with a reason.

## Phase 2 — Expected matrix (from the spec and the user, never the code)

1. Fill cells from the spec, PRD, decision log, or README. Cite the source per cell (`spec §4`, `D12`).
2. Batch every cell the spec doesn't decide into questions for the user, grouped by surface. Ask; record the answer and the date as the source.
3. Cells still unanswered are marked `?` — they are open questions in the report, not assumptions.

Cell values: `allow`, `deny`, `own` (only rows they own), `tenant` (only rows in their tenant), `field:<list>` (partial), `?`.

## Phase 3 — Observed matrix

1. **Static pass** (read-only): for each surface, note where authz is enforced — middleware, handler, policy, function, nowhere. Flag surfaces that bypass the policy layer (privileged clients in handlers, definer-rights functions, views that skip policies). This decides where to probe hardest; it is not evidence.
2. **Build the harness** in the project at `scripts/authz-probe.*` with `scripts/authz-probe.config`, obeying the safety contract. One probe per matrix cell: sign in as the principal, attempt the action, record `allowed` / `denied` / `error` plus row count or status.
3. **Probe the deny cells hardest.** An `allow` that works is a smoke test; a `deny` that's actually allowed is the finding. For `own`/`tenant` cells, probe *another* owner's/tenant's row by ID.
4. **Silent denial ≠ denial proof.** Row-filtered systems return empty or "0 rows affected" rather than errors. Probe with a known-existing row ID and check what came back.

## Phase 4 — Diff and report

Write `authz/matrix.md` in the project:

| Surface | Action | Principal | Expected (source) | Observed | Status |
|---|---|---|---|---|---|

Status: `ok`, **`OVER`** (allowed but should be denied — security finding), `UNDER` (denied but should be allowed — bug), `UNPROBED` (with reason), `?` (open question).

Then:
- **Findings**, `OVER` first: surface, principal, the probe that proved it, where enforcement is missing, the fix.
- **Unprobed** list (external side effects, HTTP writes not approved) — these become human-verify items.
- The harness stays in the repo as a regression guard. Its one-line result goes in the status line (`authz probes 212/214, 2 OVER`), not in `testPlan.md`.
- `testPlan.md` gets only what needs human eyes: role checks where the UI must hide something, and the unprobed side-effect endpoints.

Report only. Fix what the user approves; re-run the harness after each fix.

## Common mistakes

- **Deriving "expected" from the code.** Rubber-stamps every bug.
- **Principals = roles.** Most real leaks are same-role, different owner/tenant.
- **Only probing reads.** Update/delete of another tenant's row is where escalation lives.
- **Trusting the middleware / the hidden button.** Probe the API and data layer directly.
- **Treating empty results as denial** without a known-existing target row.
- **Service-role key in the harness.** Every probe passes; nothing is tested.
