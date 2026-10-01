# OfferPlay API watch

External watchdog for `https://api.offerplay.in`. Every 5 minutes a GitHub Actions
runner probes the API **the way the mobile app does** and demands real JSON:

| Probe | Expects |
|---|---|
| `GET /health` | `{"status":"ok"}` — the Node process is alive |
| `GET /api/app/feature-flags` | JSON with `"success":true` — a public app endpoint |
| `GET /api/redeem/packages` | JSON `401 {"success":false,…}` — a protected app endpoint still answers JSON |

Each probe retries 3× over ~40 s before counting as failed. If **any** probe fails the
run turns red and an **emergency alert** goes out; when the next run passes again a
"recovered" notice follows.

## Why not a normal uptime check?

During the 2026-10-01 incident the server answered **HTTP 200** with an HTML
"One moment, please…" page (the hosting provider's anti-bot challenge) on every
`/api/…` path, while `/health` was fine. Ordinary monitors saw "200 OK" and stayed
quiet. This watchdog checks `Content-Type` and body, so that exact failure is caught.

## Alert channels

- **ntfy** (primary, free): pushes to a private topic stored in the repo secret `NTFY_TOPIC`.
  Founders subscribe in the ntfy app (Android / iOS) and set the subscription to
  max-priority alarm + bypass Do-Not-Disturb, so an outage wakes them up.
- **Telegram** (optional): set secrets `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_IDS`
  (comma-separated) to also message a Telegram bot chat.

The topic name is **not** in this repo on purpose — anyone who knows it could spam
the founders' phones.

## Test it

Actions → **OfferPlay API watch** → *Run workflow* → tick **test_alert** → Run.
A `[TEST]` alert is delivered without any real outage.

## What an alert means / what to do

The API is returning something other than JSON to external clients. First suspect:
Cloudways → Server → **Security → Firewall → (gear)** → anti-bot / Splash Screen
is ON again → turn it **OFF** (or have Cloudways support whitelist `api.offerplay.in`
in WebShield). Then confirm `https://api.offerplay.in/api/app/feature-flags` returns JSON.

## Housekeeping

`keepalive.yml` makes one empty commit a month so GitHub never auto-disables the
schedule for inactivity.
