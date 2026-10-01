# AGENTS.md

## What this project is

A single static page: `index.html` (all CSS and JS inline, 1161 lines) plus one
local asset, `hero-video.mp4` (used as the hero background, and again further
down the page). There is **no build step, no package manager, no framework, no
backend and no API calls** — the "contact form" (`index.html:959`) is
`onsubmit="event.preventDefault(); alert(...)"` and never sends anything.
Images and social links point at remote URLs; fonts (Google Fonts) and icons
(Font Awesome / Remixicon from cdnjs) come from CDNs, so the browser needs
outbound internet for full fidelity.

**No environment variables or credentials are required.**

## Running it

```sh
docker compose -f docker-compose.base44.yml up -d
```

- `web` — `nginx:alpine`, serving the repo **bind-mounted read-only** at
  `/usr/share/nginx/html`, so edits to `index.html` are live on refresh.
  Host port **3000** → container 80.
- `nginx.base44.conf` is mounted as `/etc/nginx/nginx.conf` and sets
  `user root;`. This is deliberate: the sandbox clones the repo into a
  mode-`0700`, root-owned directory, which the image's default `nginx` worker
  user cannot traverse — every request 403s without it.
- The config also `deny all`s dotfiles so the bind mount cannot leak `.git`.

## Verifying

```sh
curl -s -o /dev/null -w '%{http_code} %{content_type}\n' http://localhost:3000/            # 200 text/html
curl -s -o /dev/null -w '%{http_code} %{content_type}\n' http://localhost:3000/hero-video.mp4  # 200 video/mp4
```

## Editing notes

- There is **no hot-reload dev server** (static files, no watcher), so after
  changing `index.html` the preview needs a browser refresh — call
  `reload_preview` when a change must be shown.
- `try_files` ends in `=404`, not `/index.html`: the page navigates only with
  hash anchors, so a catch-all fallback is unnecessary and produced a
  redirect cycle when a file could not be read.
