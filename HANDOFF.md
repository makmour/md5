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
- **Commit:** e1b917b8afa45f986fd61580d0fba9bec779c444 · https://github.com/makmour/md5/commit/e1b917b8afa45f986fd61580d0fba9bec779c444
