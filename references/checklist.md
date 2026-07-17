# Ship checklist

Run this against the whole change before reporting a backend as done. For each line, state passed, or not-applicable with a reason. Do not report finished until every applicable line passes.

## Access control

- [ ] Every table with user data has RLS enabled and an owner-scoped policy.
- [ ] No policy uses `USING (true)` on private data.
- [ ] Ownership comes from the authenticated identity, not a client-supplied id.
- [ ] No secret is present in the code or a committed file.
- [ ] Every query with user input is parameterized.

## Data integrity

- [ ] Every `*_id` column has a foreign key constraint with a chosen delete behavior.
- [ ] Money is `numeric`, timestamps are `timestamptz`, fixed states are constrained.
- [ ] Required columns are `NOT NULL`; unique columns have a unique constraint.
- [ ] Every table has `created_at` and `updated_at`.

## Performance

- [ ] Every foreign key column has an index.
- [ ] Columns filtered or sorted on common paths are indexed.
- [ ] Every list endpoint is paginated with a hard maximum.
- [ ] No read path issues a query per row.

## Migration safety (only when changing an existing backend)

- [ ] A snapshot exists before any destructive change.
- [ ] Destructive changes were confirmed by a human, not applied silently.
- [ ] Renames and type changes were staged so no client breaks mid-deploy.
- [ ] Every migration has a stated rollback.

## Report format

State the result plainly, for example:

```
Access control:   pass (RLS + owner policy on notes, orders; auth via verified token)
Data integrity:   pass (FKs on all *_id; numeric money; timestamps present)
Performance:      pass (indexes on all FKs + status, created_at; lists capped at 100)
Migration safety: n/a (new backend, no existing data)
```
