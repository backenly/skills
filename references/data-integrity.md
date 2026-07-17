# Data integrity

Bad data is expensive to unwind after it accumulates. These constraints stop it from entering in the first place. The database enforces them for every write, from every source, which application checks cannot promise.

## Foreign keys, so relationships stay real

Every column that points at another table needs a foreign key constraint. Without one, you can delete a user and leave their posts pointing at nothing, or insert a comment on a post that does not exist. The constraint makes those states impossible.

Bad:

```sql
create table posts (
  id uuid primary key default gen_random_uuid(),
  author_id uuid not null   -- points at users, but nothing enforces it
);
```

Good:

```sql
create table posts (
  id uuid primary key default gen_random_uuid(),
  author_id uuid not null references users(id) on delete cascade
);
```

Choose the delete behavior deliberately. `on delete cascade` removes the children with the parent. `on delete restrict` blocks the delete while children exist. `on delete set null` orphans them intentionally. Pick one on purpose.

## Types that match the data

The type is the first line of correctness. The wrong type invites wrong values.

- Money is `numeric`, never `float`. Floating point cannot represent `0.10` exactly, and rounding errors in money are real bugs.
- Timestamps are `timestamptz`, never text. Text timestamps do not sort, compare, or convert time zones correctly.
- A fixed set of states (`draft`, `published`, `archived`) is constrained, either with a `check` constraint or an enum, not left as a free string that accepts typos.

```sql
create table orders (
  id uuid primary key default gen_random_uuid(),
  total numeric(12,2) not null,
  status text not null default 'pending'
    check (status in ('pending','paid','shipped','cancelled')),
  created_at timestamptz not null default now()
);
```

## Not-null and unique, where the data demands it

A column that must always have a value is `NOT NULL`. A column that must not repeat (email, username, slug) has a unique constraint. Enforcing these in the database means no code path, now or later, can violate them.

```sql
alter table users
  alter column email set not null,
  add constraint users_email_unique unique (email);
```

## Timestamps on every table

Every table carries `created_at` and `updated_at`, both `timestamptz` and defaulted. You will need them for sorting, debugging, auditing, and pagination, and adding them after the table has data is more work than starting with them.

```sql
created_at timestamptz not null default now(),
updated_at timestamptz not null default now()
```

Keep `updated_at` current with a trigger, or set it in your update path, so it means what it says.
