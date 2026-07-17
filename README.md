<p align="center">
  <img src="assets/backenly-icon.png" width="52" height="52" alt="Backenly" />
</p>

<h1 align="center">Backend Engineer</h1>

<p align="center">
  A backend engineering skill for Claude Code, Cursor, and Codex that stops your agent shipping backends that leak data, scan full tables, or lose data on a migration.
</p>

<p align="center">
  <a href="https://backenly.com"><strong>Backenly</strong></a>
  &nbsp;&middot;&nbsp;
  <a href="./SKILL.md">Skill</a>
  &nbsp;&middot;&nbsp;
  <a href="./references/">Rules</a>
  &nbsp;&middot;&nbsp;
  <a href="./LICENSE">MIT</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT License" />
  <img src="https://img.shields.io/badge/Claude%20Code%20%C2%B7%20Cursor%20%C2%B7%20Codex-8B5CF6" alt="Claude Code, Cursor, Codex" />
</p>

---

Coding agents are good at making a backend that demos. They are bad at the parts you only notice in production: the missing row-level security that lets any user read every other user's rows, the missing index that makes the app crawl once a table is real, the migration that drops a column and takes the data with it.

This skill gives the agent the checks a senior backend engineer runs by reflex, and refuses to call a backend done until it passes them. It works on Postgres, Supabase, or any SQL database. It is not a framework and it does not lock you into anything. It is a set of rules.

## Install

```bash
npx skills add backenly/backend-engineer
```

That drops the skill into your agent:

- Claude Code: `~/.claude/skills/backend-engineer/`
- Cursor: `.cursor/rules/backend-engineer.mdc`
- Codex: `~/.codex/skills/backend-engineer/`

Or copy [SKILL.md](./SKILL.md) and [references/](./references/) in by hand.

## Three verbs

| Verb | What it does |
| --- | --- |
| **build** (default) | Design the schema, then pass every gate before calling it done. Reports which gates passed. |
| **audit** `<target>` | Review an existing schema or migration against the gates. Returns a ranked punch list. No edits. |
| **harden** `<target>` | Fix the holes an audit found. Applies additive fixes directly; proposes destructive ones for a human to confirm. |

## The gates

A backend does not ship until it passes these. Each has a full rule file with bad-to-good examples under [references/](./references/).

- **Access control** ([security.md](./references/security.md)): row-level security on every table with user data, ownership from the verified identity, no secrets in the repo, parameterized queries.
- **Data integrity** ([data-integrity.md](./references/data-integrity.md)): foreign keys on every relationship, correct types, not-null and unique where the data demands it, timestamps on every table.
- **Performance** ([performance.md](./references/performance.md)): an index on every foreign key and hot filter column, pagination with a hard cap, no query-per-row.
- **Migration safety** ([migrations.md](./references/migrations.md)): snapshot before anything destructive, staged renames that never break a client mid-deploy, a stated rollback for every change.

The full pre-ship list is in [references/checklist.md](./references/checklist.md).

## From the team building Backenly

These are the correct rules for building a backend by hand, on any database. We also build [Backenly](https://backenly.com): autonomous backend infrastructure that plans, builds, runs, and keeps itself healthy. It applies and maintains every rule here for you automatically, and keeps enforcing them as the backend changes. The rules stand on their own, though. Use them wherever you build.

## Contributing

Pull requests are not open yet while the rule-set is still settling. If you have a check that belongs here, or a case where the skill got something wrong, open an issue. Feedback is welcome.

## License

MIT. Use it, fork it, ship it. See [LICENSE](./LICENSE).
