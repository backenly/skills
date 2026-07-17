# Performance

A backend that is fast on ten rows tells you nothing. These are the choices that decide whether it is still fast on ten thousand.

## Index every foreign key

Postgres does not create an index on a foreign key column for you. Without one, every join and every filter on that column reads the whole table. This is the single most common reason a backend that felt fast in development crawls in production.

Bad. `comments.post_id` has a foreign key but no index, so loading a post's comments scans all comments:

```sql
create table comments (
  id uuid primary key default gen_random_uuid(),
  post_id uuid not null references posts(id),
  body text
);
```

Good:

```sql
create index comments_post_id_idx on comments(post_id);
```

Apply this to every `*_id` column that references another table.

## Index the columns you filter and sort on

The columns in your `WHERE` and `ORDER BY` clauses on common paths need indexes too. The usual suspects are `status`, `slug`, `email`, `created_at`, and any column behind a "list the recent" or "find by" query.

```sql
create index posts_status_idx on posts(status);
create index posts_created_at_idx on posts(created_at desc);
create unique index users_email_idx on users(email);
```

A unique index on `email` does double duty: it enforces uniqueness and makes lookups by email fast.

## Paginate every list, with a hard cap

An endpoint that returns every row works until a table is large, then it returns megabytes, times out, or exhausts memory. Every list path has a default page size and a maximum it will never exceed, no matter what the client asks for.

Bad:

```
GET /posts   ->   returns all posts
```

Good:

```
GET /posts?limit=20&offset=0
-- server clamps limit to a maximum (for example 100) and defaults it when absent
```

Keyset pagination (ordering by an indexed column and paging on its last seen value) stays fast at any depth, where large offsets slow down. Prefer it for feeds and long lists.

## Do not query in a loop (N+1)

Fetching a list and then issuing one more query per item is the N+1 pattern. Ten items become eleven queries; a thousand items become a thousand and one. Load the related data in a single query with a join or an `IN` clause.

Bad:

```
const posts = await getPosts()
for (const post of posts) {
  post.author = await getUser(post.author_id)   // one query per post
}
```

Good:

```
const posts = await getPosts()
const authorIds = posts.map(p => p.author_id)
const authors = await getUsersByIds(authorIds)   // one query for all
// then map authors back onto posts in memory
```

## Select the columns you need

`SELECT *` pulls every column, including large text and json you did not ask for, over the wire on every request. On hot paths, name the columns you actually use.
