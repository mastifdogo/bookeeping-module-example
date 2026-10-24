# Hello module for Ledger

An example module for [Ledger](https://github.com/mastifdogo/Bookeeping), the
double-entry bookkeeping app. It adds a **Hello** page to Ledger's sidebar
showing:

- a summary of your books (number of accounts, total debits per currency)
- a list of notes, stored in the module's own database table

Copy this repository to start your own module.

## Install

In Ledger (0.6.0 or newer), open **Modules**, enter
`mastifdogo/bookeeping-module-example` and choose **Install**.

## Files

| File | Purpose |
|---|---|
| `ledger-module.json` | The manifest: id, name, version, and which files to use |
| `server.mjs` | Backend: `register(ctx)` adds API routes under `/api/v1/m/hello/` |
| `ui/index.html` | The page shown inside Ledger (served at `/m/hello/`) |
| `migrations/001_notes.sql` | Creates the `notes` table in the module's own schema (`mod_hello`) |
| `.github/workflows/release.yml` | Publishes a release when the version changes |

The full module API is in
[docs/MODULES.md](https://github.com/mastifdogo/Bookeeping/blob/main/docs/MODULES.md).

## Releasing a new version

1. Change the code.
2. Bump `version` in `ledger-module.json`, e.g. `1.0.0` → `1.1.0`.
3. Push to `main`. The workflow checks the module and publishes release
   `v1.1.0`.

Ledger picks up the new version on its update schedule, or right away
with **Update modules now** on the Modules page.

New database changes go in a new file (`migrations/002_....sql`). Never
edit a migration that has already been released.
