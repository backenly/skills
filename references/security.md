# Access control

Most backend security holes are not exotic. They are the same few mistakes, made because the person building did not know the check existed. These are those checks.

## Row-level security is not optional

A table that holds data belonging to specific users must have row-level security (RLS) enabled, with a policy that limits each row to its owner. Without RLS, the database returns every row to anyone who can reach the table, and the only thing standing between one user and everyone else's data is application code that is one bug away from failing.

Enabling RLS with no policy denies all access. You must add a policy.

Bad. Any authenticated user reads every user's notes:

```sql
create table notes (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null,
  body text
);
-- no RLS. every row is readable by everyone.
```

Good. Each user sees only their own rows:

```sql
create table notes (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references users(id),
  body text,
  created_at timestamptz not null default now()
);

alter table notes enable row level security;

create policy notes_owner_select on notes
  for select using (user_id = auth_uid());

create policy notes_owner_modify on notes
  for all using (user_id = auth_uid()) with check (user_id = auth_uid());
```

`auth_uid()` stands for however your platform exposes the authenticated user id inside a policy. On Supabase it is `auth.uid()`. The point is the same everywhere: the row is scoped to the identity the server verified, not to anything the client sent.

## A policy of `USING (true)` is a fake lock

RLS can be enabled and still expose everything if the policy is unconditional. This is worse than no RLS, because every dashboard reports the table as protected.

Bad:

```sql
create policy notes_all on notes for select using (true);
```

Good: scope to the owner, as above. If a table is genuinely public (a list of countries, published blog posts), that is fine, but make it a deliberate decision and write it down, do not reach `USING (true)` by accident on a table with private data.

## Never trust a client-supplied user id

If ownership is decided by a `user_id` that arrives in the request body or query string, any client can set it to any value and read or write another user's data. Ownership must come from the verified session on the server.

Bad:

```
POST /notes  { "user_id": "any-id-i-want", "body": "..." }
```

Good: the server ignores any `user_id` in the request and uses the id from the authenticated token. In an RLS policy, this is exactly what `auth_uid()` enforces.

## Secrets do not live in the repository

Database connection strings, API keys, JWT signing secrets, and OAuth client secrets belong in environment variables, never in source code, and never in a committed `.env` file. Commit a `.env.example` with blank or placeholder values instead. A secret committed once stays in git history forever, even after you delete it, so treat any committed secret as compromised and rotate it.

## Parameterize every query

String-interpolating user input into SQL is how injection happens. Use parameterized queries or a query builder that parameterizes for you.

Bad:

```
db.query("select * from notes where id = '" + req.params.id + "'")
```

Good:

```
db.query("select * from notes where id = $1", [req.params.id])
```
