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
- **`gh` CLI is fair game.** Code may use the `gh` command-line
  tool to inspect workflow runs, view PR/issue state, fetch
  deploy logs, or check repo metadata when it's faster than
  redirecting the user to a browser. Do NOT use `gh` to *modify*
  repo state via API (creating issues, merging PRs, editing
  releases, etc.) without an explicit task instruction.

---

## 2026-05-06 · Task 06 - Statcounter + mobile UX + strikethrough

- **Goal:** Add privacy-first analytics, fix mobile tab
  navigation, fix hero strikethrough on mobile, update
  /privacy to disclose Statcounter honestly.
- **Changes:**
  - Added Statcounter async snippet (project 9988271,
    invisible, HTTPS-only) to all four HTML pages
    immediately before </body>.
  - Fixed mobile tab navigation: changed `.tabs` from
    `flex-wrap: wrap` to `flex-wrap: nowrap` so tabs stay
    on a single horizontally-scrollable row. Added
    `.tabs::-webkit-scrollbar { display: none; }` to
    hide the scrollbar on webkit browsers.
  - Fixed hero strikethrough: removed `transform:
    rotate(-2deg)` from `.strike::after` and adjusted
    `top: 56%` to `top: 52%`. Horizontal line renders
    pixel-perfect at any size and never drifts on mobile.
  - Updated `public/privacy/index.html`: added Statcounter
    disclosure paragraph in "What data is processed",
    updated "No analytics" bullet to "No advertising
    analytics", bumped "Last updated" date.
  - SHA fixup: replaced `<SHA-PLACEHOLDER>` in Task 05
    entry with `e75c38f0c773247d8129e8bb9448343b8317549f`.
- **Verified (pre-push):**
  - Local HTTP server test: Statcounter snippet present
    in all four pages (`grep -r "sc_project" public/`
    should return 4 results).
  - Strikethrough fix visible in local browser on mobile
    viewport (DevTools responsive mode).
- **Verified (post-push):**
  - GH Actions deploy green.
  - `curl -s https://md5.me/ | grep -i statcounter`
    returns the snippet.
  - Mobile browser: tabs scroll horizontally on the
    homepage.
  - NOTE: Statcounter will NOT record visits yet - the
    CSP still has connect-src 'none' until the user
    updates it in RunCloud NGINX Config panel. See
    post-deploy instructions below.
- **Open items:**
  - Task 06 SHA fixup on next task (per the rule).
  - User must update CSP in RunCloud NGINX Config panel
    after this deploy (see post-deploy instructions in
    Task 06 chat session).
  - Self-host Google Fonts.
  - Cloudflare cache rules.
  - HSTS preload (deferred).
- **Commit:** <SHA-PLACEHOLDER> · https://github.com/makmour/md5/commit/<SHA-PLACEHOLDER>

---

## 2026-05-06 · Task 05 - nginx security headers hardening

- **Goal:** Ship security headers and HSTS for md5.me. All work
  happened server-side via RunCloud panel and SSH - no repo
  changes.
- **Changes (server-side, not in git):**
  - HSTS enabled via RunCloud SSL/TLS panel:
    `Strict-Transport-Security: max-age=31536000`
  - Four security headers added via RunCloud NGINX Config,
    `headers` type, name `hardening`, webapp `md5`:
    - `Content-Security-Policy` - tight policy with
      `connect-src 'none'` (browser-enforces zero network calls),
      `frame-ancestors 'none'`, `upgrade-insecure-requests`
    - `Referrer-Policy: strict-origin-when-cross-origin`
    - `Permissions-Policy` - disables camera, mic, geolocation,
      payment, usb, interest-cohort
    - `Cross-Origin-Opener-Policy: same-origin`
  - RunCloud already sets `server_tokens off` and
    `X-Content-Type-Options: nosniff` and
    `X-Frame-Options: SAMEORIGIN` natively - no duplication needed
  - `absolute_redirect off` deferred - cosmetic fix for curl
    output only; Cloudflare + HSTS makes it irrelevant for
    real users
- **Verified:**
  - All 7 headers confirmed live via curl:
    `strict-transport-security: max-age=31536000`
    `content-security-policy: default-src 'self'; ...`
    `referrer-policy: strict-origin-when-cross-origin`
    `permissions-policy: interest-cohort=(), ...`
    `cross-origin-opener-policy: same-origin`
    `x-content-type-options: nosniff`
    `x-frame-options: SAMEORIGIN`
  - Headers apply to /about/ and /privacy/ (site-wide)
  - Browser DevTools console: no CSP violations on homepage
    or tool tabs (console errors were a browser extension,
    not site CSP)
- **Open items:**
  - Task 05 SHA fixup on next task (per the rule)
  - `absolute_redirect off` deferred indefinitely
  - HSTS preload submission deferred (decide later whether
    to bump max-age to 2 years and submit to hstspreload.org)
  - Self-host Google Fonts (removes third-party dependency,
    mentioned in /privacy as "evaluating")
  - Privacy-respecting analytics
  - Cloudflare cache rules for static assets
- **Commit:** e75c38f0c773247d8129e8bb9448343b8317549f · https://github.com/makmour/md5/commit/e75c38f0c773247d8129e8bb9448343b8317549f

---

## 2026-05-06 · Task 04 - About / Contact / Privacy + shared CSS

- **Goal:** Replace the three footer 404s with real pages and
  extract shared chrome CSS to a single source of truth.
- **Changes:**
  - Created `public/style.css` with CSS variables, base reset,
    topbar/brand/net-pill, biz/footer, sr-only utility, and new
    `.page` rules for content pages. ~180 lines total.
  - Modified `public/index.html`: removed the chrome rules now
    living in style.css from the inline `<style>` block, added
    `<link rel="stylesheet" href="/style.css">`, updated three
    footer links from `.html` paths to clean URLs.
  - Created `public/about/index.html` (product-only voice,
    per Task 04 design questions).
  - Created `public/contact/index.html` (LinkedIn + GitHub,
    no email inbox, GDPR routing via LinkedIn).
  - Created `public/privacy/index.html` (Plus 1 Limited as
    data controller, Cloudflare and Google Fonts disclosed,
    origin server described without naming the stack, "Last
    updated" set to today).
  - SHA fixup: replaced `<SHA-PLACEHOLDER>` in Task 03's entry
    with `9a08bffc220f2a35322630360f622a10dd89ad77`.
- **Verified (pre-push):**
  - `git status` shows expected file set staged.
  - Local browser test: open `public/index.html` directly,
    confirm chrome still renders correctly (it should, since
    style.css is referenced via `/style.css` which fails
    locally - so do this test through a local HTTP server
    like `python3 -m http.server 8000` from inside `public/`).
- **Verified (post-push):**
  - GH Actions run completed green.
  - `curl -sI https://md5.me/about` returns HTTP/2 200.
  - `curl -sI https://md5.me/contact` returns HTTP/2 200.
  - `curl -sI https://md5.me/privacy` returns HTTP/2 200.
  - `curl -sI https://md5.me/style.css` returns HTTP/2 200.
  - https://md5.me/ (homepage) still renders correctly with
    chrome CSS now coming from external file.
- **Open items:**
  - Task 04 SHA fixup on next task (per the rule).
  - Cloudflare cache rules for static assets and HTML.
  - Privacy-respecting analytics.
  - Self-host Google Fonts to remove the third-party dependency
    (mentioned in /privacy as "evaluating").
- **Commit:** d03d26f52c2568790f2000cd2cc66062364c7c5f · https://github.com/makmour/md5/commit/d03d26f52c2568790f2000cd2cc66062364c7c5f

---

## 2026-05-06 · Task 03 — Brand assets + M5→MD5 + housekeeping

- **Goal:** First production deploy of user-facing changes.
  Add the favicon set + og.png referenced (but 404ing) since
  Task 02; align the site's header brand mark with the wordmark
  (M5 → MD5); promote carry-over fixups from Task 02.
- **Changes:**
  - Added `public/favicon.svg`, `public/favicon.ico` (multi-res
    16/32/48), `public/favicon-96x96.png`,
    `public/apple-touch-icon.png` (180×180), `public/og.png`
    (1200×630). Generated locally in Task 03a from SVG sources
    in `.assets-src/` (gitignored).
  - Brand mark in `public/index.html` updated from "M5" to "MD5"
    (HTML content + CSS `width: 30px → 40px`).
  - SHA fixup: replaced `<SHA-PLACEHOLDER>` in Task 02's entry
    with `1fff25ed0a2b5f545bd066b763bd3616166c1a5f`.
  - Promoted `.secrets/` and `.assets-src/` from local-only
    `.git/info/exclude` to committed `.gitignore`. Removed the
    corresponding lines from `.git/info/exclude`.
  - Removed vestigial `.github/workflows/.gitkeep`.
  - Added "gh CLI is fair game" rule to HANDOFF Project rules.
- **Verified (pre-push):**
  - `git status` shows the expected file set staged (assets +
    index.html + HANDOFF + .gitignore + deletion of .gitkeep).
  - File sizes for assets sane: og.png ~80KB, favicon.ico ~15KB.
- **Verified (post-push):**
  - GH Actions run completed green.
  - `curl -sI https://md5.me/favicon.ico` returns HTTP/2 200.
  - `curl -sI https://md5.me/og.png` returns HTTP/2 200.
  - https://md5.me/ — header shows "MD5" tile (not "M5"),
    favicon visible in browser tab, no 404s in DevTools network.
- **Open items:**
  - Task 03 SHA fixup on next task (per the rule).
  - /about, /contact, /privacy static pages.
  - Cloudflare cache rules for static assets (favicons + og.png
    can be cached aggressively; HTML should remain dynamic).
  - Privacy-respecting analytics.
- **Commit:** 9a08bffc220f2a35322630360f622a10dd89ad77 · https://github.com/makmour/md5/commit/9a08bffc220f2a35322630360f622a10dd89ad77

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
- **Commit:** 1fff25ed0a2b5f545bd066b763bd3616166c1a5f · https://github.com/makmour/md5/commit/1fff25ed0a2b5f545bd066b763bd3616166c1a5f

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
