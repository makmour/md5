# MD5.me redesign — deploy notes

## What's in this drop

- **`index.html`** — single-file static app. ~82 KB uncompressed, ~25 KB gzipped. Zero JS deps, fonts loaded from Google Fonts with `display=swap`.

That's the entire site. The seven tools (MD5, SHA-1/256/384/512, bcrypt + phpass, password generator, WP secret keys, .htpasswd, WP password-reset SQL) are all wired into one tabbed UI. URL hash routes work: `md5.me/#bcrypt`, `md5.me/#wpsalt`, etc.

## Cryptographic verification (already done)

All implementations were verified against canonical references before delivery:

- **MD5** — passes all 7 RFC 1321 test vectors
- **bcrypt ($2y$)** — 5 hashes verified by `bcryptjs`, including UTF-8 and 100-char passwords
- **phpass ($P$)** — 3 hashes verified by Solar Designer's canonical PHP `PasswordHash` class (the same code WordPress ships)
- **APR1 ($apr1$)** — bit-for-bit match with `openssl passwd -apr1`
- **SHA family** — uses native `window.crypto.subtle.digest()` so correctness is the browser's problem

## Deploy on RunCloud

Two paths depending on what you want.

### Path A — simplest (recommended)

Replace the PHP web app with a static one. In RunCloud:

1. Create a new web app on `md5.me` of type **"Static HTML"** (not PHP).
2. Upload `index.html` to the web root. That's it.
3. Update the existing `md5.me` DNS / host binding to point at the new app, or just swap the document root of the existing app from `/var/www/.../md5/` to a new directory containing only `index.html`.

Static apps on RunCloud have no PHP-FPM pool, no `open_basedir`, no `session_save_path` — your current warning literally cannot reproduce.

### Path B — keep the PHP app, just drop in `index.html`

If you want to leave the existing app config alone (because of analytics, cron, whatever):

1. Rename the current `index.php` → `index.php.old` (or delete it).
2. Drop `index.html` into the same web root.
3. In RunCloud → your web app → **Settings** → set the index file priority so `index.html` resolves before `index.php`. Alternatively, in nginx config, ensure `index index.html index.php;` (in that order).
4. Old paths `password-generator.php` and `wordpress-secret-key-generator.php` should 301 to `/#password` and `/#wpsalt` respectively. Add to the nginx config of the web app:

```nginx
location = /password-generator.php             { return 301 /#password; }
location = /wordpress-secret-key-generator.php { return 301 /#wpsalt; }
```

This preserves any existing inbound links and SEO equity.

## About that PHP warning

For context on the original `session_save_path()` error you were seeing in `g-functions.php`:

```
File(/home/mdfiveme/tmp) is not within the allowed path(s):
(/home/runcloud/webapps/md5:/var/lib/php/session:/tmp)
```

`g-functions.php` is calling `session_save_path('/home/mdfiveme/tmp')`, which was the path on the previous host. RunCloud's `open_basedir` doesn't include it. Two ways to fix on the old PHP version, if you ever need to:

```php
// option 1 — let PHP use the system default (/var/lib/php/session)
// just delete the session_save_path() call

// option 2 — point at a path RunCloud allows
$tmp = __DIR__ . '/tmp';
if (!is_dir($tmp)) mkdir($tmp, 0700, true);
session_save_path($tmp);
```

But if you go static-only, this is moot.

## SEO / continuity checklist after deploy

- [ ] Confirm `https://md5.me/#md5`, `#sha`, `#bcrypt`, `#password`, `#wpsalt`, `#htpasswd`, `#wpreset` all switch the active tool.
- [ ] Add 301s for the two old PHP URLs (above).
- [ ] Submit the new `sitemap.xml` (single URL: `https://md5.me/`) to Google Search Console — the old per-tool URLs will drop out naturally.
- [ ] Drop a real `og.png` (1200×630) at `/og.png` for link unfurls.
- [ ] If you want analytics, add Plausible or Umami in the `<head>` — they're privacy-respecting and won't undermine the "nothing leaves your browser" claim. Avoid Statcounter and GA4 here.
- [ ] Add a `Content-Security-Policy` response header for bonus points (nginx):
  ```
  add_header Content-Security-Policy "default-src 'self'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src https://fonts.gstatic.com; img-src 'self' data:; script-src 'self' 'unsafe-inline'; connect-src 'none'" always;
  ```
  `connect-src 'none'` is the strong move — it blocks the page from making *any* outbound network call. The "0 network calls since load" pill in the header becomes enforced by the browser, not just observed.

## Things I deliberately did NOT include

- **Statcounter snippet** — its old HTTP URL was a mixed-content risk and the value-vs-trust trade-off is poor for a security audience. If you want stats, use Plausible.
- **The "worst passwords of 2013" list** — gone. Replaced with timeless guidance ("length beats complexity").
- **The "WordPress stores MD5 hashes" line** — gone. Replaced with accurate framing about bcrypt/phpass.
- **A favicon/og.png** — placeholders referenced in the HTML (`/favicon.svg`, `/favicon.ico`, `/apple-touch-icon.png`, `/og.png`). Generate or drop in your own. If you want me to mint them from the existing `M5` brand mark, say the word.

## Connection to your business

I added one footer block, "Built by a WordPress security consultant," with a CTA to **sudowp.com**. It's calm, on-topic (people generating salts mid-incident *are* your buyer), and doesn't dilute the tool. If you'd rather route to AmIHacked.com instead — or A/B them — change the `href="https://sudowp.com/"` in the `.biz` block.

## Want me to follow up with...

- Favicon set + og.png in the brand palette
- A Plausible self-hosted Docker compose for the same Hetzner box
- The WP-side companion plugin: a single-click "rotate salts now" admin tool that pulls fresh keys from `md5.me/#wpsalt` (or generates them locally) — fits cleanly with your SudoWP / MCP-bridge thesis
- The `/about.html`, `/contact.html`, `/privacy.html` static pages with matching style

Just say which.
