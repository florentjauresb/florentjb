# AGENTS.md — setup notes for this repository

## What this project is
A **static portfolio site**: one `index.html` (inline CSS + JS) plus `CV_Bolou_Florent_Jaures.pdf`,
`sitemap.xml`, `robots.txt` and a Google verification file. There is **no build step, no package
manager, no backend and no database**. External dependencies are public CDNs only
(GSAP 3.12.5 from cdnjs, Google Fonts, images on `media.base44.com`), so **no credentials are needed**.

## Running it here (Base44 sandbox)
```bash
docker compose -f docker-compose.base44.yml up -d --build
docker compose -f docker-compose.base44.yml ps
curl -sI http://localhost:3000/     # expects HTTP 200 and the portfolio HTML
```
The `web` service is a plain `nginx:alpine` serving the bind-mounted repository root on host
port 3000, so the container always serves the *live* cloned source — no rebuild is needed after an
edit to `index.html` (just refresh; there is no file watcher, so ask for a preview reload if the
page looks stale).

## Non-obvious quirks (learned the hard way)
- **`.base44/nginx.conf` must be mounted as the full `/etc/nginx/nginx.conf`, not as a `conf.d`
  snippet.** The sandbox clones the repo into a root-only (0700) directory, so nginx's default
  `user nginx;` worker cannot read the mounted files and every request returns **403 Forbidden**.
  The full config sets `user root;` (only inside this throwaway dev container) to fix it without
  touching repository permissions.
- **Healthcheck must target `127.0.0.1`, not `localhost`.** Inside the nginx image `localhost`
  resolves to `::1` first, and nginx here listens on IPv4 only, so a `localhost` probe fails.
- `Cache-Control: no-store` is added in that config so the preview never shows a cached page.

## How to verify the app actually works
- `curl -s http://localhost:3000/ | md5sum` should equal `md5sum index.html` (proves the served
  page *is* the current source, not a prebuilt artifact).
- In the preview: the hero (`h1` "Bolou Florent Jaurès") renders, all sections exist
  (`hero, about, stats, skills, projects, certs, timeline, manifeste, contact`), `window.gsap` is
  defined, and there are no console errors, no failed requests and no horizontal overflow.
