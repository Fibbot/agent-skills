# Supabase / Postgres reference for authz-matrix-audit

## Local-only targets

A local stack (`supabase start`) listens on:

| Service | Allowed target |
|---|---|
| API (PostgREST, Auth, Storage) | `http://127.0.0.1:54321` |
| Postgres | `postgresql://postgres:postgres@127.0.0.1:54322/postgres` |
| App (Next dev) | `http://localhost:3000` |

Confirm with `supabase status` (read-only). If it isn't running, stop and ask the user to start it.

**Never run:** `supabase db reset`, `supabase db push`, `supabase migration up|repair`, `supabase link`, `supabase seed`, anything with `--linked`, `--project-ref`, or a non-local `--db-url`, `supabase functions deploy`. Same for ORM equivalents (`prisma migrate`, `drizzle-kit push`) and any `npm run db:*`.

**Never load** `SUPABASE_SERVICE_ROLE_KEY` (or `service_role` JWTs) in the harness. It bypasses RLS. Use the anon key plus a user session.

## Harness preflight (copy first, keep first)

```ts
// scripts/authz-probe.ts: runs before anything else, no bypass
const LOCAL = new Set(["localhost", "127.0.0.1", "::1"]);
const BLOCKED = [/supabase\.co$/, /supabase\.in$/, /vercel\.app$/ /* + prod domain */];
const FORBIDDEN_SQL = /\b(commit|end|drop|truncate|alter|grant|revoke|create)\b/i;

function preflight(cfg: { appUrl: string; apiUrl: string; dbUrl: string }) {
  if (process.env.NODE_ENV === "production") throw new Error("NODE_ENV=production");
  for (const u of [cfg.appUrl, cfg.apiUrl, cfg.dbUrl]) {
    const host = new URL(u).hostname;
    if (!LOCAL.has(host) || BLOCKED.some((r) => r.test(host)))
      throw new Error(`Refusing non-local target: ${u}`);
  }
  if (Object.values(cfg).some((v) => /service_role/.test(String(v))))
    throw new Error("Service-role credential in probe config");
}
```

Every SQL string passes `FORBIDDEN_SQL` before execution; a match aborts the whole run.

## DB-level probes (rollback-safe, default on)

This simulates exactly what PostgREST does per request: switch to the `authenticated` role and set the JWT claims. RLS then applies as it would in production.

```ts
const client = await pool.connect();
const before = await rowCounts(client); // counts for every probed table
try {
  await client.query("begin");
  for (const p of probes) {
    await client.query("savepoint p");
    await client.query("set local role authenticated"); // or anon
    await client.query("select set_config('request.jwt.claims', $1, true)", [
      JSON.stringify({ sub: p.userId, role: "authenticated", ...p.extraClaims }),
    ]);
    const r = await client.query(p.sql, p.params).catch((e) => ({ error: e }));
    record(p, r);                       // rowCount / rows / error
    await client.query("rollback to savepoint p");
    await client.query("reset role");
  }
} finally {
  await client.query("rollback");
  client.release();
}
const after = await rowCounts(pool);
assertEqual(before, after); // must be identical; mismatch = stop and tell the user
```

- Never use `psql --single-transaction`. It **commits** at the end.
- Write probes use `... returning id`. Under RLS, a denied `update`/`delete` affects **0 rows with no error**. `rowCount > 0` on a deny cell is the finding.
- A denied `insert` raises `new row violates row-level security policy`. That's a correct deny.
- For `own`/`tenant` cells, target a **known-existing** row belonging to another user/tenant (look it up first as `postgres`, read-only), then probe it as the principal.

## Where Supabase authz leaks (probe these hardest)

- **RLS disabled** on a table in an exposed schema: `select relname from pg_class where relrowsecurity = false and relnamespace = 'public'::regnamespace and relkind = 'r'`.
- **Policies that check role but not ownership/tenant** (e.g. `using (auth.role() = 'authenticated')`).
- **`update` policies without `with check`**: a user can move a row into another tenant.
- **Tenant/role read from `user_metadata`**: users can edit it. Only `app_metadata` or a server-maintained table is trustworthy.
- **`security definer` functions** callable by `anon`/`authenticated` that skip their own checks: `select proname from pg_proc where prosecdef and pronamespace = 'public'::regnamespace`.
- **Views without `security_invoker = true`** run as the owner and bypass RLS.
- **Storage**: `storage.objects` policies per bucket; public buckets; signed-URL generation endpoints.
- **Realtime**: channel authorization / RLS on subscribed tables.
- **Next.js server actions / route handlers using a service-role client**: RLS doesn't apply, so authz lives only in that handler's code. Every one needs an HTTP probe.
- **Middleware-only guards**: server actions and route handlers are reachable directly, so middleware matching the page path protects nothing else.

## HTTP-level probes (opt-in per session)

- Sign in each principal with `supabase.auth.signInWithPassword` against `127.0.0.1:54321` using seeded logins from the project README. Then call PostgREST/RPC with that session, or the app's routes with its cookies.
- Reads: run freely.
- Writes: only after the user approves in-session, only against `authz_probe_` sentinel rows, never endpoints with external side effects.
