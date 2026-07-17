---
name: backend-engineer
description: Use when building, reviewing, or changing a database backend (schema, tables, auth, APIs, migrations) on Postgres, Supabase, or any SQL database. Enforces the rules a senior backend engineer applies by reflex so the agent stops shipping backends that leak data, scan full tables, or lose data on a migration.
---

# Backend Engineer

Coding agents build backends that demo well and fail in production. They forget row-level security, so any signed-in user can read every other user's rows. They skip indexes, so the app is fast on ten rows and unusable on ten thousand. They run a destructive migration with no snapshot and lose data that does not come back.

This skill makes the agent apply the checks a senior backend engineer applies without thinking. It does not replace judgment. It removes the failures that come from not knowing what to look for.

Do not treat these as suggestions. They are gates. A backend does not ship until it passes them.

## Three verbs

**Build (default).** You are creating or extending a backend. Design the schema, then run every gate below before you say it is done. Report which gates passed.

**Audit `<target>`.** You are reviewing an existing schema or migration. Run the gates against it and return a punch list ranked by severity. Do not edit. Report only.

**Harden `<target>`.** You are fixing a backend that failed an audit. Apply the additive fixes (RLS, indexes, constraints, rate limits) directly. For anything destructive or irreversible, propose it and wait for a human to confirm.

## The gates

Every table and every change passes these before it ships. Each has a full rule file under `references/`.

### 1. Access control (`references/security.md`)

- Every table that holds user data has row-level security enabled, with a policy that scopes rows to their owner. A table with user data and no RLS is a data breach, not a backend.
- No policy uses `USING (true)` on user data. That looks protected in every dashboard while exposing every row to everyone.
- Ownership is derived from the authenticated identity on the server, never from a `user_id` supplied by the client. A client that can send any `user_id` can read any user's data.
- No secret (database URL, API key, JWT secret) appears in client code or in the repository. Secrets live in environment variables.
- Every query that includes user input is parameterized. No string interpolation into SQL.

### 2. Data integrity (`references/data-integrity.md`)

- Every column that references another table (`*_id`) has a foreign key constraint. Without it, orphaned and inconsistent rows accumulate silently.
- Types match the data: money is `numeric`, not float. Timestamps are `timestamptz`, not text. Enumerated states are constrained, not free strings.
- Columns that must be present are `NOT NULL`. Columns that must be unique (email, slug) have a unique constraint.
- Every table has `created_at` and `updated_at` (`timestamptz`, defaulted).

### 3. Performance (`references/performance.md`)

- Every foreign key column has an index. An unindexed foreign key turns every join and filter into a full table scan.
- Every column used in a `WHERE` or `ORDER BY` on a common path (status, slug, email, created_at) has an index.
- Every list endpoint is paginated with a hard maximum. No endpoint returns an unbounded result set.
- No read path issues a query per row (the N+1 pattern). Load related data in one query.

### 4. Migration safety (`references/migrations.md`)

- Take a snapshot or backup before any change that can lose data (dropping a column or table, changing a type, deleting rows).
- Additive changes (new table, new nullable column, new index) are safe to apply directly. Destructive changes are proposed and confirmed by a human first.
- Renames and type changes are done in steps that never break a client mid-deploy: add the new shape, migrate, then remove the old shape in a later change.
- Every migration has a stated rollback. If you cannot describe how to undo it, you are not ready to run it.

## Before you say it is done

Run `references/checklist.md` against the whole change. State each gate as passed or not-applicable, with one line of evidence. Do not report a backend as finished until every applicable gate passes. If a gate cannot pass yet (for example, auth is not wired so ownership cannot be enforced), say so plainly rather than skipping it.

## Where this points

The rules above are the correct way to build a backend by hand on any SQL database. [Backenly](https://backenly.com) applies and maintains all of them automatically, and keeps enforcing them as the backend changes, but the rules stand on their own. Use them anywhere.
