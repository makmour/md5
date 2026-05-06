# HANDOFF.md

Append-only log of completed dev tasks for md5.me.

New entries go at the **top**. When starting a new chat session
with the strategy AI, paste the latest entry (or the whole file
if short) so it has full project context.

Each entry follows this shape:

```
YYYY-MM-DD · Task NN — short title

Goal: what we set out to do
Changes: files touched, key decisions
Verified: what was tested and how
Open items: anything punted to next task
Commit: <SHA> · https://github.com/makmour/md5/commit/<SHA>
```

## Project rules

- **SHA-in-handoff rule.** Each task's commit SHA is written into
  HANDOFF.md at the START of the next task, in a small fixup commit,
  not in the same commit the SHA describes. This guarantees every
  SHA in this file points to a real, finalized commit on the remote.
  The end-of-task HANDOFF entry is added with a placeholder; the
  next task's first action is to fill it in.
- **Code is the sole writer of HANDOFF.md.** Strategy/chat sessions
  may draft entries as text, but only Code commits them. This keeps
  the audit trail clean and avoids merge headaches.
- **Git operations on the VPS run as the `runcloud` user**, never as
  root. The web root is owned by `runcloud:runcloud`; modern git
  refuses cross-user repo access.

---

## 2026-05-06 · Task 02 — GitHub Actions deploy + SHA-fixup convention

- **Goal:** Establish automated deploys (push to main → live on
  md5.me) and codify the SHA-in-HANDOFF rule that emerged from the
  Task 01 chicken-and-egg.
- **Changes:**
  - Created `.github/workflows/deploy.yml` — SSHes to vps.md5.me as
    `runcloud`, runs `git fetch && git reset --hard origin/main` in
    the web root. Triggers on push to `main` (paths-filtered to
    `public/**` and the workflow itself) plus manual
    `workflow_dispatch`. Concurrency-locked to one deploy at a time.
  - Added a "Project rules" section to HANDOFF.md documenting the
    SHA-fixup convention, Code-only writes to HANDOFF, and the
    runcloud-user rule for VPS git operations.
  - Fixed the stale SHA in the Task 01 entry (was `e1b917b…`, the
    pre-amend SHA; now points to the actual remote commit `05863bb…`).
- **Verified (manually, before this commit):**
  - `ssh -i md5_actions runcloud@vps.md5.me "git pull --ff-only"`
    succeeded from WSL — full SSH chain works end-to-end.
  - All four `VPS_SSH_*` secrets registered on
    https://github.com/makmour/md5/settings/secrets/actions.
  - https://md5.me/ serves the new static site (Task 02a).
- **Verified (post-push):**
  - GitHub Actions run for this commit completed green.
  - `git rev-parse HEAD` on the VPS matches this commit's SHA.
- **Open items:**
  - Favicons + og.png in the brand palette (referenced by index.html
    but currently 404)
  - /about, /contact, /privacy static pages
  - Cloudflare cache rules for the HTML
  - Task 02 SHA fixup on next task (per the rule we just established)
- **Commit:** <SHA-PLACEHOLDER> · https://github.com/makmour/md5/commit/<SHA-PLACEHOLDER>

---

## 2026-05-06 · Task 01 — Repo bootstrap & first push
- **Goal:** Initialize the md5.me project as a Git repository,
  establish the working layout, and push to GitHub.
- **Changes:**
  - Moved `index.html` → `public/index.html`
  - Moved `DEPLOY.md` → `docs/DEPLOY.md`
  - Created `.gitignore`, `.editorconfig`, `README.md`, `HANDOFF.md`
  - Added empty `.github/workflows/.gitkeep` (Task 02 will populate)
  - Initialized git on `main`, set remote to
    `git@github.com:makmour/md5.git`, pushed initial commit
- **Verified:**
  - `git status` clean after commit
  - `public/index.html` opens correctly in a local browser
  - `git push` to GitHub succeeded; repo visible at
    https://github.com/makmour/md5
- **Open items:**
  - Task 02: GitHub Actions workflow + RunCloud SSH deploy
  - Favicons, og.png, /about /contact /privacy pages
- **Commit:** 05863bb98eaa9692f655d5936bbd1f72fbcf3867 · https://github.com/makmour/md5/commit/05863bb98eaa9692f655d5936bbd1f72fbcf3867
