# Troubleshooting: "I see two versions" / "the update won't push"

Read this whenever an installed Fierce plugin's version doesn't match what's in this repo, whenever `ListPlugins` (or the Cowork Plugins screen) shows a stale description, or whenever `git push` from Cowork fails. This has happened more than once (see commit history: 3.6.1, 3.6.2, 3.7.0, 3.7.1, 3.8.0 all hit some version of this), and the fix is always the same shape.

## Why this happens

Two things drift independently:

1. **The plugin installed in a Cowork session** (`~/.claude/plugins/synced/fierce-coo-orchestrator/` inside that session, or whatever `ListPlugins` reports) can be edited and version-bumped by any Cowork session working on it, without that session ever touching this git repo.
2. **This repo** (`marketplace.json` at the root, plus the plugin's own `plugin.json`) only updates when someone actually commits and pushes to it.

Nothing keeps these in sync automatically. A session can bump the installed plugin to, say, 3.8.0, while the repo (and therefore `marketplace.json`, and therefore what a fresh install pulls) is still sitting on 3.7.1. That mismatch is what "I see two plugins" or "the update won't push" usually is.

**Update as of 2026-09-03: the per-plugin "Update" button now works** for this personal git marketplace. It was confirmed broken from 2026-08-19 through at least 2026-08-26 (matches Anthropic GitHub issue #65426, closed "not planned" at the time), and the standing advice below was to always remove-and-reinstall instead. Ryan confirmed Update working twice on 2026-09-03, landing correctly on 3.8.0 and then 3.9.0. Try Update first now. Remove-and-reinstall (Customize > Plugins > remove `fierce-coo-orchestrator`, then Browse > Personal > fierce-plugins > "+") remains a safe fallback if Update ever looks stuck or greyed out again, don't assume it's permanently fixed without checking `ListPlugins` or the installed version after clicking it.

## Diagnosis: find out what actually changed

1. Check the installed plugin's version: read `.claude-plugin/plugin.json` inside the currently-synced plugin folder, or ask `ListPlugins`.
2. Check the repo's version: read `plugins/fierce-coo-orchestrator/.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` in this repo. **Check both files**: they need to carry the same version number, and it's easy to bump one and forget the other (this has happened before).
3. If the installed version is ahead of the repo, something was built in a Cowork session and never pushed. Diff every file between the installed copy and the repo copy (not just the ones you assume changed) to find exactly what's new. Don't guess and don't skip files, the actual diff is the only reliable source of truth.

## Fix: get the repo caught up

1. Copy the changed files from the installed plugin into this repo's matching paths under `plugins/fierce-coo-orchestrator/`, overwriting the stale versions.
2. Bump the version number in **both** `plugin.json` and `marketplace.json` to match the installed version. Do this even if it feels redundant, the marketplace won't reflect the update otherwise.
3. Add a changelog entry at the top of `plugins/fierce-coo-orchestrator/README.md` describing what changed, following the existing entries' format.
4. `git add -A && git commit -m "..."` describing the change.

## The step that actually blocks pushes: credentials

If you're doing this from inside a Cowork session using the device bridge (`device_bash` reaching a connected Mac folder), **the push will fail**:

```
fatal: could not read Username for 'https://github.com': No such device or address
```

That sandbox is an isolated Linux VM with no GitHub credential helper and no `gh` CLI, it cannot authenticate to GitHub no matter how correct the commit is. This is not something to work around by hunting for or entering a token inside that sandbox.

**The fix:** finish the commit from inside the Cowork session (that part works fine, since it's just writing to the mounted folder and running local git), then run the actual push from a shell that has real GitHub credentials, i.e. Terminal on Ryan's Mac:

```
cd ~/fierce-plugins && git push origin main
```

If a Cowork session gets this far and can't push, it should say so plainly, hand over that exact command, and stop, rather than trying to find another way to authenticate.

## After pushing

1. Confirm: `git log origin/main --oneline -3` should show the new commit at the top.
2. In Cowork: use the marketplace "Update" button on `fierce-coo-orchestrator` (confirmed working as of 2026-09-03, see above). If it looks stuck or greyed out, fall back to removing the installed plugin and reinstalling fresh from Browse > Personal > fierce-plugins > "+".
3. Confirm via `ListPlugins` that the version now matches, and that there is only one `fierce-coo-orchestrator` entry, not two. Note `ListPlugins` can lag slightly on cache; if it looks stale, say so rather than asserting it as ground truth, and defer to what Ryan sees in the actual Cowork UI.

## Applied to

- 2026-08-26: v3.8.0 (meeting action-item sync merged into `fierce-meeting-accountability`) was installed and version-bumped in a Cowork session but never pushed; `marketplace.json` was still 3.7.1. Diagnosed via this exact procedure, repo brought current, pushed from Ryan's Mac Terminal after the device-bridge push failed on missing credentials.
- 2026-09-03: confirmed the "Update" button fix (see the note near the top of this doc); Ryan landed 3.8.0 and then 3.9.0 via Update, not remove-and-reinstall. Docs corrected here and in the README's 3.9.1 note rather than silently, so the earlier "confirmed broken" finding stays on record as accurate for its time.
- 2026-09-03: two independent 3.8.0s landed in parallel (the meeting-accountability sync above, and a new `fierce-state-compliance` sub-agent built in a separate session that had fetched an older `main`). `git merge` itself also repeatedly hit the device-bridge sandbox's lock-file issue (see the note below), on top of the credential gap: `git merge`, `git add`, and `git commit` each acquire and release `.git/index.lock` (and `git merge` additionally touches `.git/HEAD.lock` / `.git/ORIG_HEAD.lock`), and the sandbox blocks the final unlink of a lock file even when the underlying write succeeded. The lock is then left behind and blocks the *next* git command, including, in this case, a second internal locking phase within the same `git merge` invocation. Every git operation that writes (add, commit, merge) needs the relevant `.git/*.lock` file cleared by Ryan first if a prior write happened in the same session; `git status`/`git diff`/`git fetch` can trigger this too. Resolved by having Ryan delete each stale lock file as it appeared, then hand-merging the two 3.8.0 changes into one 3.9.0 (rather than trusting automatic merge to land cleanly given the repeated lock interruptions), verifying the combined `plugin.json`/`marketplace.json`/`README.md`/`SKILL.md` by hand before committing.
