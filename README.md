# Trading Bot Template

> A white-label starter for building and deploying your own visual trading bot platform on top of the Deriv trading API. Fork it, brand it, deploy it.

![Prerequisite](https://img.shields.io/badge/node-20.x-blue.svg)
![Prerequisite](https://img.shields.io/badge/npm-9.x-blue.svg)
![Build](https://img.shields.io/badge/build-RSBuild-green.svg)
![Framework](https://img.shields.io/badge/framework-React%2018-blue.svg)

This repository is a **template**, not a finished product. It is intended to be forked, customized with your own brand, and deployed to your own domain. The trading engine, OAuth flow, and WebSocket integration all point at Deriv's infrastructure out of the box — everything else (branding, theming, menu, logo, fonts, analytics, error reporting) is yours to configure.

---

## Table of Contents

- [What You Get](#what-you-get)
- [Who This Is For](#who-this-is-for)
- [Quick Start (Fork → Brand → Run)](#quick-start-fork--brand--run)
- [Prerequisites](#prerequisites)
- [Documentation](#documentation)
- [Project Layout](#project-layout)
- [Configuration at a Glance](#configuration-at-a-glance)
- [Things You Must Not Change](#things-you-must-not-change)
- [Deployment](#deployment)
- [Contributing](#contributing)
- [License](#license)

---

## What You Get

- **Visual Bot Builder** — Drag-and-drop strategy builder powered by Blockly, with a library of pre-built trading blocks.
- **Integrated Charts** — SmartCharts (TradingView-style) with the standard set of technical indicators (SMA, EMA, Bollinger Bands, MACD, RSI).
- **Dashboard** — Bot performance, recent activity, and quick actions.
- **OAuth 2.0 with PKCE** — Production-ready authentication flow against Deriv's OAuth server.
- **Authenticated WebSocket connection** — Real-time market data, balance, trade execution, and account switching via the DerivWS API.
- **White-label configuration** — A single `brand.config.json` drives your colors, typography, logo, domain, menus, and theme behavior.
- **Centralized error logging** — `ErrorLogger` utility with a pluggable interface for Sentry, TrackJS, or any other reporting service.
- **Optional monitoring stack** — Guides for re-enabling Datadog RUM, TrackJS, Rudderstack analytics, and Growthbook feature flags (removed from the base to keep the bundle lean).
- **Fast builds** — RSBuild for sub-second dev server startup and optimized production bundles.

## Who This Is For

Developers who want to ship a branded derivatives trading bot application without building the Blockly integration, bot runtime, OAuth flow, or WebSocket layer from scratch. You are expected to be comfortable with:

- React, TypeScript, and modern JavaScript tooling
- Deploying a static SPA to your own infrastructure (Vercel, Netlify, Cloudflare Pages, S3+CloudFront, etc.)
- Registering an OAuth app with Deriv to get a `CLIENT_ID` for your domain

---

## Quick Start (Fork → Brand → Run)

```bash
# 1. Fork this repo on GitHub, then clone your fork
git clone https://github.com/<your-org>/<your-fork>.git
cd <your-fork>

# 2. Install dependencies (Node 20.x required)
npm install

# 3. Configure your brand
#    Edit brand.config.json: brand_name, domain, colors, logo, typography
#    See user-guide/03-white-labeling.md for the full reference

# 4. Generate brand CSS (validates your config, writes src/styles/_themes.scss)
npm run generate:brand-css

# 5. Add your OAuth credentials
#    Create .env and set CLIENT_ID to the OAuth client ID you registered with Deriv
echo "CLIENT_ID=your_deriv_oauth_client_id" > .env

# 6. Start the dev server
npm start
#    → https://localhost:8443
```

Then walk through the full setup in [Getting Started](./user-guide/01-getting-started.md).

---

## Prerequisites

| Requirement | Version                             | Why                                                      |
| ----------- | ----------------------------------- | -------------------------------------------------------- |
| Node.js     | 20.x                                | Enforced via `package.json` engines                      |
| npm         | 9.x                                 | Package manager                                          |
| Git         | 2.30+                               | Version control                                          |
| Browser     | Chrome, Firefox, or Safari (latest) | Dev server runs on HTTPS; WebCrypto is required for PKCE |

You will also need a **Deriv OAuth client ID** registered against the domain you intend to deploy to. Without it, login will fail — the authentication flow and the WebSocket handshake both depend on it.

---

## Documentation

All the setup, configuration, and architectural context lives under [`user-guide/`](./user-guide). Start here:

| #   | Guide                                                             | What's inside                                                       |
| --- | ----------------------------------------------------------------- | ------------------------------------------------------------------- |
| 01  | [Getting Started](./user-guide/01-getting-started.md)             | Prerequisites, project setup, commands, environment variables       |
| 02  | [Architecture Overview](./user-guide/02-architecture-overview.md) | Layers, MobX stores, RxJS streams, bot engine, build system         |
| 03  | [White Labeling](./user-guide/03-white-labeling.md)               | Branding, colors, typography, logo, menus, theme configuration      |
| 04  | [Authentication](./user-guide/04-authentication.md)               | OAuth 2.0 with PKCE, token exchange, session management, logout     |
| 05  | [WebSocket Integration](./user-guide/05-websocket-integration.md) | Connection architecture, public vs authenticated endpoints, DerivWS |
| 06  | [Error Handling](./user-guide/06-error-handling.md)               | Centralized `ErrorLogger`, Sentry/TrackJS integration, migration    |
| 07  | [Monitoring & Analytics](./user-guide/07-monitoring-analytics.md) | Re-enabling Datadog, TrackJS, Rudderstack, Growthbook               |
| 08  | [Changelog](./user-guide/08-changelog.md)                         | What changed from the original Deriv Bot to this template           |

New to the template? Read them in order. Just need to re-skin? Jump straight to [White Labeling](./user-guide/03-white-labeling.md).

---

## Project Layout

- [`brand.config.json`](./brand.config.json) — white-label config (brand, colors, logo, domain)
- [`src/`](./src) — application source ([`app/`](./src/app), [`pages/`](./src/pages), [`stores/`](./src/stores), [`services/`](./src/services), [`hooks/`](./src/hooks), [`components/layout/`](./src/components/layout))
- [`src/external/bot-skeleton/`](./src/external/bot-skeleton) — bot runtime, Blockly blocks, WebSocket layer
- [`src/external/indicators/`](./src/external/indicators) — SMA, EMA, Bollinger Bands, MACD, RSI
- [`scripts/generate-brand-css.js`](./scripts/generate-brand-css.js) — generates `src/styles/_themes.scss` from `brand.config.json`
- [`user-guide/`](./user-guide) — developer documentation (see above)

For the layer-by-layer breakdown see [Architecture Overview](./user-guide/02-architecture-overview.md); for the full directory tree see [Getting Started — Project Structure](./user-guide/01-getting-started.md#project-structure).

---

## Configuration at a Glance

- **`brand.config.json`** drives every visual/identity knob — brand, colors, typography, logo, footer, hostnames. Edit it and run `npm run generate:brand-css`. Full reference: [White Labeling Guide](./user-guide/03-white-labeling.md).
- **`.env`** holds secrets. `CLIENT_ID` is required for login; everything else (`APP_ID`, Google Drive, translations, monitoring credentials) is optional. Full table: [Getting Started — Environment Variables](./user-guide/01-getting-started.md#environment-variables).
- **npm scripts** — see `package.json`. Most common: `npm start`, `npm run build`, `npm test`, `npm run generate:brand-css`. Full list: [Getting Started — Available Commands](./user-guide/01-getting-started.md#available-commands).

---

## Things You Must Not Change

The template relies on Deriv's infrastructure for OAuth and for the WebSocket trading API. Keep these values in `brand.config.json` pointed at Deriv — changing them will break login and all trading functionality:

- `platform.auth2_url.production` → `https://auth.deriv.com/oauth2/`
- `platform.auth2_url.staging` → `https://staging-auth.deriv.com/oauth2/`
- `platform.derivws.url.production` → `https://api.derivws.com/trading/v1/`
- `platform.derivws.url.staging` → `https://staging-api.derivws.com/trading/v1/`

Everything else (brand name, colors, logo, fonts, your own `platform.hostname`, menus, footer toggles) is yours to change.

See [White Labeling — Configuration Constraints](./user-guide/03-white-labeling.md#authentication-urls) and [Changelog — Configuration Constraints](./user-guide/08-changelog.md#configuration-constraints).

---

## Deployment

Any static host will work — the build output in `dist/` is a plain SPA.

1. Run `npm run build` and ship the `dist/` directory.
2. Register a Deriv OAuth client for your deployed domain and set `CLIENT_ID` in your host's environment variables.
3. Set `platform.hostname.production.com` in `brand.config.json` to your deployed hostname (no protocol, no trailing slash) so `isProduction()` detects the right environment and connects to the production WebSocket. The hostname you put here must match the redirect URI you register with Deriv.
4. Make sure your host serves `index.html` for unknown routes (SPA fallback) — OAuth redirects back to `/?code=...&state=...` and the `App` component handles the callback inline.

### Example: deploying to Vercel

This is one concrete path that works — any static host (Netlify, Cloudflare Pages, S3+CloudFront, your own infra) will do, but the shape of the steps is the same. Adapt as needed for your host.

1. **Fork & clone** — fork this repo to your GitHub org (e.g. `your-org/your-fork`) and clone locally.
2. **Configure locally** — edit `brand.config.json` (brand, colors, logo, and especially `platform.hostname.production.com` → the domain you'll deploy to), run `npm install` then `npm run generate:brand-css`, commit, push.
3. **Create a Vercel project** — import your GitHub repo. **Override the Output Directory to `dist`** (Vercel's default is wrong for RSBuild). Framework preset can be left as "Other"; Vercel picks up `npm run build` automatically.
4. **Deploy once** — let Vercel do the first deploy so you have a stable domain (e.g. `your-fork.vercel.app` or your custom domain). Login won't work yet.
5. **Register the OAuth app with Deriv** — at [developers.deriv.com](https://developers.deriv.com/), register a new app using your deployed domain as the redirect URI (`https://your-fork.vercel.app/`), copy the **Client ID** Deriv issues.
6. **Add env vars on Vercel** — in Project Settings → Environment Variables, add `CLIENT_ID=<the_id_from_deriv>`. Add any optional ones (`APP_ID`, Google Drive, monitoring) here too.
7. **Redeploy** — env vars are injected at build time, so push a commit or click "Redeploy" in Vercel. Login now works.

On other hosts the equivalents are: set the output/publish directory to `dist`, deploy once to get a stable URL, register that URL with Deriv, add `CLIENT_ID` to the host's environment variables, trigger a rebuild.

---

## Contributing

This repo is intended as a template; most users will want to maintain their own fork rather than upstream changes. That said, bug fixes and improvements that apply to every downstream fork are welcome.

- Conventional commits: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`
- Run `npm run test:lint` before pushing
- Run `npm test` and make sure the build still passes

For bot builder and Blockly block changes, look in [`src/external/bot-skeleton/scratch/blocks/`](./src/external/bot-skeleton).

---

## License

See [LICENSE](./LICENSE).
