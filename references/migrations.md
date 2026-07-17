# Migration safety

Building a backend is forgiving. Changing one that already holds real data is not. A single careless migration can delete data that does not come back or break every client at once. These rules keep changes reversible and non-breaking.

## Snapshot before anything destructive

Before any change that can lose data, take a backup or snapshot you can restore from. Destructive means dropping a column or table, changing a column type, or deleting rows. Additive changes (a new table, a new nullable column, a new index) cannot lose data and do not need this, but destructive ones always do.

If you cannot restore, you cannot safely run the change.

## Additive is safe, destructive is confirmed

Apply additive changes directly. Propose destructive ones and let a human confirm before running them, because the cost of a wrong destructive migration is unrecoverable and the cost of asking is a few seconds.

- Safe to apply: create table, add nullable column, add index, add constraint that current data already satisfies.
- Confirm first: drop column, drop table, change type, delete rows, add a `NOT NULL` column with no default to a table that has rows (it fails or forces a value on existing rows).

## Change shape in steps, so nothing breaks mid-deploy

A rename or type change done in one step breaks every client that expects the old shape the instant it runs. Do it in stages so the old and new shapes coexist until every client has moved.

To rename `full_name` to `name`:

1. Add the new column `name`.
2. Backfill it from `full_name` and keep both in sync (write to both, or a trigger).
3. Move clients to read and write `name`.
4. In a later migration, once nothing uses it, drop `full_name`.

The same staged approach applies to changing a type: add a new column of the new type, backfill, switch clients, drop the old column.

## Every migration states its rollback

Before running a migration, write down how to undo it. For additive changes the rollback is usually a drop. For a backfill it is a description of how to reverse or discard the change. If you cannot state the rollback, you do not understand the migration well enough to run it yet.

## Adding a NOT NULL column to a populated table

This is a common footgun. Adding a `NOT NULL` column with no default to a table that already has rows fails, because the existing rows have no value for it. Add it with a default, or add it nullable, backfill, then set `NOT NULL`.

```sql
-- safe on a populated table
alter table users add column locale text not null default 'en';

-- or, when there is no sensible default
alter table users add column locale text;          -- nullable first
update users set locale = 'en' where locale is null; -- backfill
alter table users alter column locale set not null;  -- then enforce
```
