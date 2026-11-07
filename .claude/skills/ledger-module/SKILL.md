---
name: ledger-module
description: Work on this Ledger module (a plugin for the Ledger bookkeeping app, installed from this GitHub repository): add features, fix bugs, update it for new Ledger versions, check it, and release it. Use for any change to this repository's module code, manifest, migrations, UI or releases.
---

# Working on this Ledger module

This repository is a **Ledger module**: `ledger-module.json` (manifest),
`server.mjs` (runs inside the Ledger server), `ui/` (its page in Ledger) and
`migrations/` (SQL in its own schema). Ledger installs the latest GitHub
Release of this repository. Ledger itself is
[mastifdogo/Bookeeping](https://github.com/mastifdogo/Bookeeping).

## 1. Get a Ledger checkout and read the module API

The API reference and the full workflow live in the Ledger repository.

- Use `$LEDGER_DIR`, a sibling checkout (`../Accounting`, `../Bookeeping`),
  or clone one: `git clone --depth 1 https://github.com/mastifdogo/Bookeeping /tmp/ledger-for-modules`
  (`scripts/check.sh` does this for you).
- Read **`<ledger>/docs/MODULE_LLM_CONTEXT.md`** in full before changing
  anything: manifest, `ctx` API and return shapes, `ctx.settings`,
  `ctx.dashboard`, `ctx.llm`, `ctx.gmail`, `ctx.search`, roles, migrations,
  UI and security rules.
- Then follow **`<ledger>/.claude/skills/ledger-module/SKILL.md`** (design
  rules, implementation rules, checks, publishing). Treat this repository as
  the module directory it talks about. Where the docs and Ledger's code
  disagree, the code wins (`backend/src/module-host.ts`, `modules.ts`,
  `module-settings.ts`).

Set `"ledger": ">=x.y.z"` in the manifest to the oldest Ledger with every
feature the module uses (the context file says which version added what;
`git -C <ledger> describe --tags --abbrev=0` is the latest release).

## 2. Rules that bite

- No npm packages at runtime: Ledger installs the release as-is. Only
  `node:*` built-ins, global `fetch`, and this repo's own files. (A
  `package.json` here is only for running tests.)
- Never edit a released migration; add the next `NNN_name.sql`.
- Money is integer cents per currency; post to the books only with
  `ctx.ledger.createJournalEntry(entry, req.user)`.
- AI goes through `ctx.llm` and Gmail through `ctx.gmail` (Ledger ≥ 0.10.0):
  no API keys, OAuth, IMAP or Apps Script bridges of the module's own for
  those. The admin allows the module under Settings → AI / Gmail.
- Bad input answers 4xx (`throw new ctx.HttpError(422, "…")`), never 500.

## 3. Check before saying it works

```sh
npm test                      # this repo's own tests, if package.json has them
(pg_isready || service postgresql start) >/dev/null
TEST_DATABASE_URL=postgres://postgres:postgres@127.0.0.1:5432/postgres \
  sh .claude/skills/ledger-module/scripts/check.sh
```

`check.sh` runs Ledger's checker on this repo: manifest, files, imports,
migrations, then the module loaded into a real Ledger on a scratch database
(every GET route, POST `{}` must give 4xx, UI, settings). Add `--serve 8098`
to keep it running and test the real flows with `curl` and Playwright (it
prints the session cookie). If PostgreSQL has no password yet:
`runuser -u postgres -- psql -qc "ALTER USER postgres PASSWORD 'postgres'"`.

## 4. Release (only when the user asks)

**Never push to the Ledger repository (`mastifdogo/Bookeeping`) while
working on this module.** Push only to this repository. If the module needs
a change in Ledger (a bug fix, a new `ctx` feature, a docs or checker fix),
make it in a local Ledger checkout, tell the user what it is and why, and
leave pushing it to them. The module must work without that change.

1. Bump `version` in `ledger-module.json` (and note the change in the README).
2. Commit, tag `v<version>`, push the tag, and create a GitHub Release from
   it (not a draft or pre-release). If tags or releases can't be created from
   this session, tell the user which commit to release in GitHub's web UI.
3. Ledger installs it on its update schedule or with "Update modules now"
   (Settings → Modules).

## This module: Hello (`id: hello`, schema `mod_hello`)

The example module for people writing their own: a summary of the books, a
notes list in the module's own table, and (in newer versions) a to-do and
notifications on the dashboard and a settings form.

- This repository publishes Ledger's `examples/modules/hello/` as an
  installable module. Ledger's copy is the source: change it there (in the
  Ledger repo, by the user, see above) and copy it here, rather than letting
  the two drift.
- Layout: `server.mjs` (routes: summary, notes), `migrations/001_notes.sql`,
  `ui/index.html`. No test suite; `check.sh` is the test.
- Releasing here is automatic: `.github/workflows/release.yml` publishes a
  GitHub Release whenever `version` in `ledger-module.json` changes on
  `main`. So bumping the version and pushing to `main` is a release: only do
  it when the user asks.
- Known follow-ups:
  - This repo is at 1.0.0 (`"ledger": ">=0.6.0"`), while Ledger's
    `examples/modules/hello` is at 1.3.0 (dashboard to-do and notifications,
    a settings form, a drawn icon; needs Ledger ≥ 0.9.1). Bring it up to date
    when the user asks.
