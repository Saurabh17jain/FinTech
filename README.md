# Fintech — Next.js · Better Auth · Inngest

**Fintech** is a production-ready template for a modern stock market web app: live prices, search, personalized alerts, interactive charts, AI-powered insights, daily news summary, and watchlists. Built with **Next.js**, authentication via **Better Auth**, and background automation via **Inngest**. Designed for performance, security, and fast developer iteration.

---

## Table of contents

1. [Demo & screenshots](#demo--screenshots)
2. [Key features](#key-features)
3. [Tech stack](#tech-stack)
4. [Architecture overview](#architecture-overview)
5. [Getting started (local dev)](#getting-started-local-dev)
6. [Environment variables](#environment-variables)
7. [Running & testing](#running--testing)
8. [Deploying](#deploying)
9. [Features implementation notes](#features-implementation-notes)
10. [Security & privacy](#security--privacy)
11. [Troubleshooting](#troubleshooting)
12. [Contributing](#contributing)
13. [License](#license)

---

## Demo & screenshots

> Add a hosted demo link and screenshots here after deployment. Keep screenshots small and show: home (live feed), ticker detail (chart + insights), alerts modal, and daily summary.

---

## Key features

* **Live prices** via WebSockets (or server-sent events) for near real-time updates.
* **Search** stocks by symbol or company name with instant suggestions and fuzzy matching.
* **Personalized alerts**: price targets, % change, volume spikes; delivered via in-app notifications, email, or webhook.
* **Interactive charts** with zoom, pan, indicators (SMA, EMA, RSI) and range selector.
* **AI-powered insights**: automated short summaries, pattern detection, and signal explanations using an LLM (e.g., OpenAI) and model prompts tailored to finance context.
* **Daily news summary** aggregated from news APIs and summarized into a concise morning briefing.
* **Watchlists** with drag-and-drop ordering and saved layout per user.
* **Automations** using Inngest for scheduled jobs (daily summary, alert dispatching, data cleanup).
* **Secure auth** using Better Auth for passwordless or social login flows and role-based access controls.
* **Offline-first UX** (basic caching and optimistic updates) for a responsive feel.

---

## Tech stack

* **Frontend / SSR**: Next.js (App Router), React, TypeScript
* **Auth**: Better Auth (passwordless / OAuth flows) — integrates with Next.js middleware
* **Automation & serverless workers**: Inngest for background jobs and workflows
* **Realtime**: WebSockets (native or provider like Pusher / Ably) or server-sent events; fallback polling
* **Charts**: Recharts / Chart.js / ApexCharts (choose one) for interactive visuals
* **AI**: OpenAI (or compatible LLM provider) for insights, summarization, and signal generation
* **Data & APIs**: Financial market data provider (Finnhub / IEX Cloud / Alpaca / polygon.io) + news provider (NewsAPI, Alpha Vantage news feed, or provider bundled with market data)
* **DB / persistence**: PostgreSQL (recommended) or any managed DB (Planetscale, Supabase)
* **Queue / background tasks**: Inngest + serverless platform (Vercel/Netlify) or containerized workers
* **Email / Notifications**: SendGrid / Postmark / SES and optional SMS providers for critical alerts

---

## Architecture overview

1. **Next.js frontend** (SSR & client): pages for market overview, ticker detail, watchlists, alerts, and settings.
2. **Realtime layer**: a small WebSocket server or third-party provider streams price updates to subscribed clients; server fan-outs updates to authenticated users only.
3. **API layer**: Next.js API routes (or separate API service) that talk to the DB, Better Auth, Inngest, and market data providers.
4. **Inngest workflows**: scheduled and event-driven workflows — e.g., `daily-news-summary`, `evaluate-alerts`, `cleanup-old-data`, and `generate-ai-insights`.
5. **Auth layer**: Better Auth issues user sessions / JWTs; middleware in Next.js validates requests.
6. **AI services**: server-side prompts & caching; expensive calls are performed asynchronously via Inngest with results stored and available via API.

---

## Getting started (local dev)

> Prereqs: Node.js (v18+), Yarn or npm, PostgreSQL (or SQLite for quick start), and accounts/keys for third-party services.

1. Clone repo

```bash
git clone https://github.com/your-org/realtime-stock-app.git
cd realtime-stock-app
```

2. Install

```bash
# npm
npm install
# or yarn
yarn
```

3. Create `.env.local` based on `.env.example` (see next section)

4. Initialize the database (example using Prisma)

```bash
npx prisma migrate dev --name init
```

5. Start development server (Next.js)

```bash
npm run dev
# or
yarn dev
```

6. Start Inngest local runner (if using local development mode)

```bash
# If using Inngest CLI/local runner
inngest run -f ./inngest/index.ts
```

7. (Optional) Start a local WebSocket server for simulated price feed

```bash
node ./scripts/mock-price-feed.js
```

---

## Environment variables

Use `.env.local` in development. Example variables:

```
# App
NEXT_PUBLIC_APP_NAME="Fintech"
NEXT_PUBLIC_BASE_URL=http://localhost:3000

# Market data provider
MARKET_API_KEY=your_market_api_key
MARKET_WS_URL=wss://streaming.market.provider
MARKET_REST_URL=https://api.market.provider

# News provider
NEWS_API_KEY=your_news_api_key

# Auth (Better Auth)
BETTER_AUTH_CLIENT_ID=xxx
BETTER_AUTH_CLIENT_SECRET=xxx
BETTER_AUTH_ISSUER=https://auth.better.example
BETTER_AUTH_CALLBACK_URL=${NEXT_PUBLIC_BASE_URL}/api/auth/callback

# Inngest
INNGEST_API_KEY=inngest_live_or_dev_key
INNGEST_WEBHOOK_SECRET=...

# AI / LLM
OPENAI_API_KEY=sk-...
OPENAI_MODEL=gpt-4o-mini

# Database
DATABASE_URL=postgresql://user:pass@localhost:5432/realtime_stock?schema=public

# Notifications
SENDGRID_API_KEY=...
SMTP_FROM=no-reply@yourdomain.com
```

> **Note**: Prefix `NEXT_PUBLIC_` only for variables safe to expose to the browser (eg. public config). Secrets (API keys, DB URLs) must remain server-side.

---

## Running & testing

* `npm run dev` — run Next.js in development
* `npm run build` — build for production
* `npm start` — run production server
* `npm run lint` — run ESLint
* `npm run test` — run unit & integration tests
* `npm run test:e2e` — run Playwright/Cypress e2e tests (if included)

### Storybook

If components are isolated, run `npm run storybook` to see UI components.

---

## Deploying

This project is optimized for Vercel but can be deployed to any Node host.

1. Push code to GitHub
2. Connect repo to Vercel
3. Add environment variables in Vercel dashboard (do not expose secrets)
4. Add Inngest webhook URL to Inngest dashboard and point to your Vercel function endpoint (or use Inngest hosted runners)
5. Configure scheduled jobs in Inngest for daily summaries and alert evaluation

For containerized deployments (AWS ECS, GCP Cloud Run), build Docker images and ensure you have secure secrets management for keys & DB.

---

## Features implementation notes

### Live prices

* Use a server-side WebSocket that subscribes to the market provider and fans out messages.
* Channel auth per user watchlist to reduce bandwidth.
* Fallback: HTTP polling every N seconds when WebSockets are unavailable.

### Search

* Use an index-backed search (Postgres `pg_trgm` or Algolia) for fast fuzzy search.
* Provide autocomplete and recent searches.

### Alerts

* Alerts are persisted in DB. Inngest workflows run frequently or event-driven to evaluate alert rules.
* When an alert condition is met, push to the user via WebSocket + store a notification record; optionally send email/SMS.

### Charts

* Server supplies historical OHLCV data aggregated at multiple resolutions.
* Charts fetch the smallest resolution required and progressively load higher resolution on zoom.

### AI Insights

* Run LLM calls server-side and cache results per symbol & timeframe.
* Use Inngest to offload heavy LLM calls and return a placeholder while insights are generated.
* Add guardrails in prompts to avoid hallucinations (e.g., include `source` evidence and a confidence score).

### Daily News Summary

* Inngest scheduled function fetches news, deduplicates by similarity, summarizes using an LLM, and stores/resolves a short digest for each user’s watchlist.

---

## Security & privacy

* Store API keys & secrets in environment variables or a secrets manager (do NOT commit them).
* Rate-limit public endpoints and validate all incoming webhook requests (use HMAC verification).
* Use HTTPS and set `Strict-Transport-Security`, `Content-Security-Policy`, `X-Frame-Options`, and other relevant headers.
* For financial data and alerts, maintain an audit log of actions and dispatched alerts.
* Use role-based access control for admin operations.

---

## Troubleshooting

* **No live data**: check `MARKET_WS_URL` and your provider plan; confirm WebSocket connection in browser devtools.
* **Auth failing**: confirm Better Auth client credentials and callback URLs match the dashboard settings.
* **Inngest jobs not running**: confirm Inngest key, runner is active (if self-hosted), and webhook endpoints are reachable.

---

## Contributing

1. Fork the repo
2. Create a feature branch `feat/your-feature`
3. Commit and push
4. Open a PR with a description of changes and screenshots if UI changed

Please follow conventional commits and run tests / linters before opening a PR.

---

## Folder structure (recommended)

```
/ (root)
├─ app/                 # Next.js App Router pages & layouts
├─ components/          # React components
├─ lib/                 # helper functions (api clients, auth helpers)
├─ services/            # market-data adapters, news adapters
├─ inngest/             # functions and workflows for Inngest
├─ scripts/             # local utilities (mock feed, seed data)
├─ prisma/ or migrations/# DB schema
├─ public/              # static assets
├─ tests/               # unit/integration/e2e
└─ README.md
```

---

## Roadmap ideas

* Paper trading sandbox
* Portfolio analytics & tax reports
* Social features: shareable watchlists and annotations
* Mobile PWA & push notifications
* Advanced AI trader assistant with explainable signals

---

## License

MIT © Saurabh kumar Jain & Shamma

---

If you'd like, I can also:

* generate a `.env.example` file matching the required env vars,
* scaffold the Inngest workflow code (example `daily-news-summary` and `evaluate-alerts`),
* create a simple mock price feed script for local testing, or
* produce a concise `CONTRIBUTING.md` and `CODE_OF_CONDUCT.md`.

Tell me which follow-up you'd like and I will add it to the repo.
