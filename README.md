# md5.me

A small, fast, client-side cryptographic toolkit for WordPress
admins and sysadmins. MD5, SHA, bcrypt, phpass, password
generator, WordPress salts, .htpasswd, and a WordPress password-
reset SQL helper.

Everything runs in the browser. Nothing is sent to a server.

**Live:** https://md5.me/

## Project layout

- `public/` — the deployable static site (single `index.html`)
- `docs/` — deploy notes and project documentation
- `.github/workflows/` — CI/CD (deploys `public/` to RunCloud on
  push to `main`)
- `HANDOFF.md` — append-only log of completed dev tasks. Read
  this first when picking up where a previous session left off.

## Local development

No build step. Open `public/index.html` directly in a browser, or
serve the directory:

```
cd public && python3 -m http.server 8000
```

Then visit http://localhost:8000/.

## Deploy

See `docs/DEPLOY.md` for the full deployment story. In short:
pushes to `main` trigger a GitHub Action that SSHes into the
RunCloud VPS and runs `git pull` in the web root.

## Working with this repo

Tasks are drafted in chat, executed by Claude Code in WSL, and
logged in `HANDOFF.md` after each completion. Each `HANDOFF.md`
entry is short — goal, changes, verification, open items,
commit SHA — and new entries go at the top.
