# Deploy and Host Postiz on Railway – Social Media Scheduler

Postiz is an open-source social media scheduling platform — a self-hosted alternative to Buffer and Hootsuite that publishes to 30+ platforms from one content calendar, with no per-channel fees. Deploy it on Railway to schedule posts across every connected account while keeping your posts, media, and channel credentials on infrastructure you control.

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/postiz-social-scheduler?referralCode=zxcgoT&utm_medium=integration&utm_source=template&utm_campaign=generic)

## 🚀 Quick Start Deployment Guide

### Step 1: Deploy on Railway
1. Click **Deploy on Railway** above
2. Wait for all three services — Postiz, PostgreSQL and Redis — to finish building

### Step 2: Mount the uploads volume
1. Add a Railway Volume mounted at `/uploads` on the Postiz service
2. With `STORAGE_PROVIDER=local`, every uploaded image and video lives there — without the volume they vanish on redeploy

### Step 3: Set your URLs
1. Set `MAIN_URL` and `FRONTEND_URL` to your Railway public domain
2. Set `NEXT_PUBLIC_BACKEND_URL` to the publicly reachable backend URL — it is baked into the frontend at build time, so a wrong value breaks the UI in the browser
3. Redeploy after changing it so the frontend is rebuilt

### Step 4: Set your secrets and workers
1. Generate `JWT_SECRET` with `openssl rand -hex 32`
2. Confirm `IS_GENERAL=true`
3. Confirm `RUN_CRON=true` — without the cron worker, scheduled posts queue but never publish

### Step 5: Create your account
1. Open your Railway domain and register the first account
2. Then set `DISABLE_REGISTRATION=true` and redeploy, so nobody else can sign up

### Step 6: Connect your channels
1. From the app, connect each social account you want to publish to
2. Each platform runs its own OAuth flow and needs your callback URL to match `MAIN_URL` exactly
3. Schedule a test post and confirm it publishes at the scheduled time

## About Hosting Postiz

This template deploys three services on Railway: the Postiz app (`ghcr.io/gitroomhq/postiz-app:v2.11.3`) running its Next.js frontend, Node.js backend and cron worker; PostgreSQL for users, connected channels, posts and schedules; and Redis for the background job queue and session cache. Media uploaded with `STORAGE_PROVIDER=local` is written to a volume at `/uploads`, with Postgres and Redis holding their own volumes at `/var/lib/postgresql/data` and `/data`.

Three Railway services run at flat compute cost, regardless of how many channels you connect or posts you schedule. Buffer and Hootsuite bill per channel or per seat, so the gap widens with every account you add:

| Tool | Pricing | Control | Strength |
|---|---|---|---|
| Buffer | Per channel, per month | Vendor-hosted | Managed service, official platform partnerships, no maintenance |
| Hootsuite | Per seat, per month | Vendor-hosted | Mature analytics and team workflows |
| Later / Publer and similar | Tiered per channel or seat | Vendor-hosted | Turnkey onboarding |

## Common Use Cases

- **Flat-cost multi-platform scheduling:** run one content calendar across 30+ platforms without paying per connected channel.
- **Agency / multi-brand publishing:** run social publishing for several brands or clients from a single deployment.
- **Data ownership:** keep post history, media and channel credentials on infrastructure you control.
- **Team tool replacement:** replace a per-seat social tool for a small team at flat infrastructure cost.

## Dependencies for Postiz Hosting

### Deployment Dependencies

- [Postiz (upstream source, AGPL-3.0)](https://github.com/gitroomhq/postiz-app)
- [Postiz documentation](https://docs.postiz.com/)

## ⚙️ Configuration

| Variable | Required | Description |
|---|---|---|
| `MAIN_URL` | Yes | Public URL of the deployment — your Railway domain |
| `FRONTEND_URL` | Yes | URL the frontend is served from. Normally the same as `MAIN_URL` |
| `NEXT_PUBLIC_BACKEND_URL` | Yes | Backend URL baked into the frontend build. Must be publicly reachable from the browser |
| `JWT_SECRET` | Yes | Signing secret for auth tokens. Generate with `openssl rand -hex 32` |
| `DATABASE_URL` | Yes | Postgres connection string, injected by Railway from the database service |
| `REDIS_URL` | Yes | Redis connection string, injected by Railway from the Redis service |
| `IS_GENERAL` | Yes | Set to `true` for the self-hosted build |
| `STORAGE_PROVIDER` | No | Where uploaded media goes. `local` writes to the `/uploads` volume; switch to an object store if you outgrow it |
| `RUN_CRON` | No | Set to `true` so scheduled posts actually publish. Without the cron worker, posts queue but never go out |
| `DISABLE_REGISTRATION` | No | Set to `true` after creating your account, or anyone with the URL can register |

## 🐳 Self-Host with Docker Compose

```bash
git clone https://github.com/sahilrupani/postiz-railway-template
cd postiz-railway-template
cp .env.example .env
```

Generate the required secret and set it in `.env`:

```bash
openssl rand -hex 32   # use this value for JWT_SECRET
```

Fill in `MAIN_URL`, `FRONTEND_URL`, `NEXT_PUBLIC_BACKEND_URL`, `DATABASE_URL` and `REDIS_URL` in `.env`, then start the stack:

```bash
docker compose up -d
```

Open the app at the `MAIN_URL` you configured to create your first account.

## ❓ Frequently Asked Questions (FAQ)

### How much does it cost to run Postiz on Railway?
Three Railway services at flat compute cost. Unlike Buffer or Hootsuite there is no per-channel or per-seat charge, so connecting more accounts does not raise the bill.

### Is my data private?
Yes. Posts, media and channel credentials live in your own Postgres and volume on Railway. Content still passes to each social platform when it publishes, which is inherent to posting.

### How many platforms does it support?
30+ social platforms. Each is connected through its own OAuth flow from inside the app.

### What licence is Postiz under?
The upstream project is AGPL-3.0. Review the upstream repository before using it commercially or offering it as a service.

### Can I stop other people signing up?
Yes — set `DISABLE_REGISTRATION=true` after creating your own account.

### Can I migrate off Railway later?
Yes. It is a standard container plus Postgres and Redis; carry a database dump and the `/uploads` volume to any Docker host.

### Why do scheduled posts never publish, even though they appear in the calendar?
The cron worker is not running. Set `RUN_CRON=true` and redeploy — the queue fills but nothing drains without it.

### Why does the UI load but every API call fail in the browser?
`NEXT_PUBLIC_BACKEND_URL` is wrong. It is compiled into the frontend bundle, so it must be the publicly reachable backend URL, and the app must be rebuilt after changing it.

### Why do uploaded images disappear after a redeploy?
No volume is mounted at `/uploads` while `STORAGE_PROVIDER=local`. Add a Railway Volume at that path.

### Why does connecting a social channel fail at the OAuth callback?
`MAIN_URL` does not match the callback URL registered with that platform. They must be identical, including scheme and any trailing path.

### Why can strangers create accounts on my instance?
`DISABLE_REGISTRATION` is not set. Enable it once your own account exists.

## 🛠️ Support & Issues

For deployment issues with this template, open an issue at [github.com/sahilrupani/postiz-railway-template/issues](https://github.com/sahilrupani/postiz-railway-template/issues) with a description of the problem, steps to reproduce, and relevant logs.

---

*This template deploys [Postiz](https://github.com/gitroomhq/postiz-app), an open-source project. This is a community-maintained Railway template and is not affiliated with or endorsed by the Postiz maintainers or Railway.*