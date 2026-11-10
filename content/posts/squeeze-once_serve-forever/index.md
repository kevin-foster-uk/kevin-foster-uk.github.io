---
title: "Squeeze Once, Serve Forever"
draft: false
date: 2026-11-10T07:00:00.000Z
description: "Your SPA's first impression is a cold download of everything. Pre-compressing assets at build time and letting nginx serve them as-is makes that first visit smaller and faster, for almost no effort."
categories:
  - Software Dev
  - Performance
tags:
  - Software Dev
  - Performance
  - Nginx
  - SPA
---

### Pre-Compressing SPA Assets for nginx

We spend a lot of time making single-page apps feel fast once they are running. Route transitions, optimistic updates, skeleton screens. But the most important performance moment is the one we test least: the **cold visit**. A new user, an empty cache, and a blank page that will stay blank until the JavaScript bundle has downloaded, parsed and executed.

For that user, every byte counts. And there is a cheap win that a lot of deployments leave on the table.

### What Happens on a Cold Visit

A typical SPA is a tiny `index.html` and a handful of large, hashed JavaScript and CSS files. On a first visit nothing is cached, so the browser has to fetch all of it before the app can render anything useful.

Most nginx setups already have `gzip on`, so the assets do get compressed. But they get compressed **on every request**, on the fly, and that comes with a compromise:

- **Low compression levels:** On-the-fly compression has to be quick, so it runs at a modest level. Smaller files are possible; you just cannot afford them per request.
- **Repeated work:** The same unchanging bundle is compressed again and again, for every user, burning CPU on a result that never changes.
- **gzip only:** Stock nginx does not do Brotli, which is noticeably better for text assets and supported by every modern browser.

The thing is, your built assets are **static**. They do not change between deploys. So why compress them at request time at all?

### Compress at Build Time Instead

The idea is simple. After your build, produce a `.gz` and a `.br` version of each text asset alongside the original:

```text
dist/assets/index-3f9a1c2e.js
dist/assets/index-3f9a1c2e.js.gz
dist/assets/index-3f9a1c2e.js.br
```

Then tell nginx to look for those files first. If the browser accepts Brotli and a `.br` file exists, nginx sends it straight from disk. If it accepts gzip, it gets the `.gz`. Anything else gets the original. No compression happens at request time at all.

Because the work is done once, at build time, you can turn the dial all the way up. Brotli at quality 11 is far too slow to run per request, but on a build server nobody cares about an extra second.

### How Much Difference Does It Make?

I tested this on a well-known real-world bundle, the production build of React DOM 18.3.1 (about 129 KB minified):

| Method                         | Size    | vs. original | Time to compress |
| ------------------------------ | ------- | ------------ | ---------------- |
| Uncompressed                   | 131,835 | -            | -                |
| gzip level 1                   | 48,431  | -63%         | ~2 ms            |
| gzip level 6 (common default)  | 43,119  | -67%         | ~3 ms            |
| gzip level 9                   | 43,029  | -67%         | ~4 ms            |
| Brotli quality 4               | 43,509  | -67%         | ~2 ms            |
| **Brotli quality 11**          | **37,180** | **-72%**  | ~217 ms          |

Brotli at maximum quality comes out **around 14% smaller than typical on-the-fly gzip**, but takes roughly 80 times longer to produce. That trade-off is terrible per request and irrelevant at build time. That is the whole argument in one table.

14% of one library might not sound dramatic, but it applies to every JavaScript and CSS file in your app, on every first visit, and it matters most for exactly the users you most want to win over: people on mobile networks seeing your product for the first time. As a bonus, the server stops spending CPU on compression entirely, and pre-compressed responses go out with a proper `Content-Length` rather than chunked encoding.

### Step 1: Generate the Files

Most bundlers have a plugin for this (Vite, webpack and Rollup all do), but I prefer a small post-build script. It works with any toolchain and has no dependencies beyond `gzip` and `brotli`:

```bash
#!/usr/bin/env bash
# precompress.sh - create .gz and .br files next to built assets
set -euo pipefail
DIST="${1:-dist}"

find "$DIST" -type f \
  \( -name '*.js' -o -name '*.mjs' -o -name '*.css' -o -name '*.html' \
     -o -name '*.svg' -o -name '*.json' -o -name '*.wasm' -o -name '*.txt' \) \
  -size +1k -print0 |
while IFS= read -r -d '' f; do
  gzip   -9 -k -f -n "$f"   # keep original, overwrite old .gz, no timestamp
  brotli -q 11 -k -f "$f"   # keep original, overwrite old .br
done
```

Hook it into your build:

```json
"scripts": {
  "build": "vite build && ./precompress.sh dist"
}
```

A few notes:

- **Keep the originals.** Some clients do not send `Accept-Encoding`, and nginx needs the uncompressed file to fall back to.
- **Skip images and fonts.** PNG, JPEG, WebP and WOFF2 are already compressed. Compressing them again wastes disk and gains nothing.
- **Skip tiny files.** Below about 1 KB the overhead is not worth it.
- **Want a few more percent on the gzip side?** Swap `gzip -9` for [Zopfli](https://github.com/google/zopfli), which produces fully gzip-compatible output around 4% smaller still, at a much higher (build-time only) cost.

### Step 2: Teach nginx to Use Them

gzip support is built in through the `gzip_static` module, which is included in most distribution packages. Brotli needs the [ngx_brotli](https://github.com/google/ngx_brotli) module. On Debian and Ubuntu that is one package:

```bash
sudo apt install libnginx-mod-http-brotli-static
```

The package registers the module for you. If you build nginx yourself, you will need a `load_module` line for `ngx_http_brotli_static_module.so` at the top of `nginx.conf`.

Then the server block:

```nginx
server {
    listen 443 ssl;
    server_name app.example.com;
    # ssl_certificate / ssl_certificate_key as usual

    root  /var/www/app/dist;
    index index.html;

    # Fallback: compress anything we did not pre-compress
    gzip            on;
    gzip_vary       on;
    gzip_comp_level 5;
    gzip_types      text/css application/javascript application/json
                    image/svg+xml application/wasm;

    # Serve pre-compressed files when they exist
    gzip_static   on;
    brotli_static on;

    # Hashed assets never change, so cache them hard
    location /assets/ {
        add_header Cache-Control "public, max-age=31536000, immutable";
        try_files $uri =404;
    }

    # Everything else is the SPA shell
    location / {
        add_header Cache-Control "no-cache";
        try_files $uri $uri/ /index.html;
    }
}
```

The important lines:

- **`gzip_static on` / `brotli_static on`:** When a request for `app.js` comes in, nginx checks for `app.js.br` or `app.js.gz` and serves it if the browser accepts that encoding. Brotli wins when both are on offer.
- **`gzip_vary on`:** Adds `Vary: Accept-Encoding`, so CDNs and proxies do not hand a Brotli response to a client that cannot read it.
- **`gzip on`:** Still worth keeping as a safety net for anything the script skipped, such as small files or API responses proxied through the same server.
- **`try_files ... /index.html`:** The usual SPA fallback. It plays nicely with pre-compression: a deep link like `/settings/profile` is rewritten to `/index.html`, and nginx will happily serve `index.html.br` if it exists.

The caching headers are not strictly part of pre-compression, but they complete the picture. Pre-compression makes the cold visit cheaper; `immutable` on hashed assets means the warm visit costs nothing at all. The shell stays `no-cache` so a new deploy is picked up straight away.

### Step 3: Check It Actually Works

Ask for the asset with different `Accept-Encoding` headers and look at what comes back:

```bash
curl -sI -H 'Accept-Encoding: br' https://app.example.com/assets/index-3f9a1c2e.js \
  | grep -iE 'content-(encoding|length)'
```

Against my test setup with the React DOM bundle:

```text
Accept-Encoding: br        ->  Content-Encoding: br    Content-Length: 37180
Accept-Encoding: gzip      ->  Content-Encoding: gzip  Content-Length: 42793
Accept-Encoding: identity  ->  (none)                  Content-Length: 131835
```

The `Content-Length` matching the size of the `.br` file on disk is the giveaway that nginx is serving it directly. In the browser, the Network panel in DevTools shows the same thing: check the `content-encoding` response header and compare the transferred size against the resource size.

### The One Gotcha: Stale Files

nginx does **not** check whether the `.br` or `.gz` file is newer than the original. If it exists, it gets served.

I tested this by overwriting a bundle with new content but leaving the old `.br` beside it. nginx carried on serving the old 37 KB file as if nothing had changed. With content-hashed filenames this rarely bites, because a changed file gets a new name. But `index.html` keeps the same name on every build, and a stale `index.html.br` pointing at old bundles is a confusing bug to chase.

The fix is boring and reliable: **always regenerate compressed files as part of the build, and deploy the whole `dist` directory as a unit** rather than copying individual files onto a live server. The `-f` flags in the script above take care of the first half.

### What About Zstandard?

Zstandard (`zstd`) is now supported as a content encoding by Chromium-based browsers and Firefox, and third-party nginx modules exist for it. For static assets, though, Brotli at quality 11 generally still produces smaller files, and it is already supported everywhere that matters. Zstd shines for on-the-fly compression where speed matters. For pre-compressed assets, Brotli plus a gzip fallback remains the pragmatic choice for now.

### The Bottom Line

Your built assets are static, so compress them like static files: once, as hard as possible, before they ever reach the server. A short build script and two lines of nginx config give your first-time visitors noticeably smaller downloads and take compression work off your server entirely.

It is one of the rare performance improvements with no real downside. Squeeze once, serve forever.
