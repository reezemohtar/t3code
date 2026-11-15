# reezemohtar/t3code — Reeze's fork of T3 Code

Fork of [pingdotgg/t3code](https://github.com/pingdotgg/t3code) (MIT). Upstream's `LICENSE` stays as is.

## Why

To add Threadbear-style workflow features (decision cards, a Highlights / needs-you panel,
Inbox vs Running / Waiting / Hibernating sidebar, projects view with counts, "wake when"
snooze, starred chats, a side-by-side A/B/C review window, "Carry on" next steps, Pocket,
librarian chore spotting, per-project panels) while staying current with upstream's
~3 nightlies a day. Design reference: [mikecann/threadbear](https://github.com/mikecann/threadbear) (MIT).

## Branch model

| Branch | What it is | Who writes |
| --- | --- | --- |
| `main` | Exact mirror of `pingdotgg/t3code` `main`. Never commit here. | `upstream-sync` workflow only |
| `reeze/main` | Integration branch and repo default. Upstream + fork features. | Merged PRs only |
| `reeze/feat/<nn>-<slug>` | One branch per feature batch, PR into `reeze/main`. | Coding agents |
| `reeze/sync-<date>` | Hand-resolved upstream merge when the sync PR conflicts. | Coding agents |

Rules that keep upstream merges cheap:

- Fork features live in **new files** under `apps/web/src/fork/`, `apps/server/src/fork/`,
  `packages/contracts/src/fork/` (or similar). Edits to upstream files are small hooks.
- Every fork feature sits behind a fork flag (default off until Reeze switches it on).
- **No changes to upstream's database migrations or `statev2` tables.** Fork state lives in
  a separate fork-owned store, so the stock app can open the same T3 home again.
- Merge upstream with a **merge commit**, never squash or rebase.

## How upstream sync works

`.github/workflows/upstream-sync.yml` runs daily at 05:17 WITA (and on demand from the
Actions tab). It:

1. switches off upstream's own workflows in this fork (they need upstream's paid runners and secrets);
2. fast-forwards `main` from upstream (best effort; if upstream changed workflow files the
   default token can't push them, and the run summary says to run `gh repo sync reezemohtar/t3code -b main`);
3. opens one PR `pingdotgg:main → reeze/main` labelled `upstream-sync`, or leaves the open one
   (its head follows upstream automatically);
4. adds the `conflict` label when GitHub says it can't merge cleanly.

It uses only the default `GITHUB_TOKEN` and never merges. Merging the sync PR is a normal
reviewed merge; if it is labelled `conflict`, resolve on `reeze/sync-<date>`.

## Switching over and back

Reeze's live T3 app is **not** switched to this fork without his separate yes.

- **Try it (safe, no switch):** run the fork from a checkout with its own T3 home, e.g.
  `vp run dev` in the worktree (state goes to the worktree's `.t3`), seeded from a read-only
  copy of real data with `vp run migrate-dev-db`. It never opens `~/.t3/userdata` and never
  runs live agents. Open it in a browser. The stock app keeps running untouched.
- **Side-by-side desktop build (planned, brief 00):** the fork desktop app gets its own app
  id, product name, Electron profile and T3 home (`~/.t3-reeze`), so both apps can be
  installed at once. Unverified until built.
- **Switch over (needs Reeze's yes):** quit the stock app, start the fork against the chosen
  T3 home. Because fork state is kept outside upstream's tables, threads stay readable by stock.
- **Go back:** quit the fork, reopen the stock T3 Code app. Unverified risk: if the fork runs a
  newer upstream base than the stock app, upstream migrations may have moved the database
  forward; update the stock app to the same or newer channel before reopening it.
