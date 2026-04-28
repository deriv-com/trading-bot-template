# Getting Started

Set up your fork of the Trading Bot Template, wire in your own brand and OAuth credentials, and get the dev server running.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Fork the Repository](#fork-the-repository)
- [Project Setup](#project-setup)
- [Register Your OAuth Client](#register-your-oauth-client)
- [Development Workflow](#development-workflow)
- [Project Structure](#project-structure)
- [Available Commands](#available-commands)
- [Environment Variables](#environment-variables)
- [First Run Checklist](#first-run-checklist)
- [Path Aliases](#path-aliases)
- [Code Style](#code-style)
- [Next Steps](#next-steps)

---

## Prerequisites

| Requirement | Version                             | Purpose                                                 |
| ----------- | ----------------------------------- | ------------------------------------------------------- |
| **Node.js** | 20.x                                | JavaScript runtime (enforced in `package.json` engines) |
| **npm**     | 9+                                  | Package management                                      |
| **Git**     | 2.30+                               | Version control                                         |
| **Browser** | Chrome, Firefox, or Safari (latest) | Dev server runs on HTTPS; WebCrypto required for PKCE   |

Verify your environment:

```bash
node --version   # Should output v20.x.x
npm --version    # Should output 9.x or higher
git --version    # Should output 2.30+
```

You will also need, eventually:

- A **Deriv OAuth client ID** registered against the domain you intend to deploy to. Local development uses `https://localhost:8443` as the redirect URI.

---

## Fork the Repository

This repo is a template. The expected workflow is:

1. Fork it on GitHub to your own organization
2. Clone your fork locally
3. Customize, commit, and push to your own remote
4. Deploy your fork — not upstream — to your hosting provider

```bash
git clone https://github.com/<your-org>/<your-fork>.git
cd <your-fork>
```

If you prefer to mirror without a fork:

```bash
git clone <upstream-url> trading-bot-template
cd trading-bot-template
git remote set-url origin <your-new-remote>
```

---

## Project Setup

### 1. Install Dependencies

```bash
npm install
```

### 2. Configure Your Brand

Edit `brand.config.json` in the project root with your brand details:

```json
{
    "brand_name": "YourBrand",
    "brand_domain": "yourbrand.com",
    "domain_name": "YourBrand.com",
    "colors": {
        "primary": "#3b82f6",
        "secondary": "#64748b"
    },
    "platform": {
        "name": "Your Trading Platform"
    }
}
```

> See [White Labeling Guide](./03-white-labeling.md) for the complete configuration reference, including logo, typography, menus, and footer toggles.

### 3. Generate Brand CSS

```bash
npm run generate:brand-css
```

Validates your color configuration and generates CSS custom properties in `src/components/shared/styles/_themes.scss`. Re-run this every time you change `brand.config.json`.

### 4. Add Your OAuth Credentials

Create a `.env` file in the project root and set your Deriv OAuth client ID:

```bash
CLIENT_ID=your_deriv_oauth_client_id
# APP_ID=12345   # Optional — only if you maintain a Legacy Deriv API app
```

Without `CLIENT_ID`, the login flow will redirect to Deriv's OAuth server but will not be authorized for your domain. See [Register Your OAuth Client](#register-your-oauth-client) below.

### 5. Start the Development Server

```bash
npm start
```

Visit `https://localhost:8443` to see your platform running.

> The dev server is HTTPS by default (via `@rsbuild/plugin-basic-ssl`). Your browser will warn about the self-signed cert — accept it to continue. HTTPS is required because PKCE relies on `crypto.subtle.digest()`.

---

## Register Your OAuth Client

Login does not work out of the box — Deriv needs to know about your domain before it will accept the redirect.

1. Register an OAuth application at [developers.deriv.com](https://developers.deriv.com/) for your redirect URI. For local development this is `https://localhost:8443/`; for production it is your deployed domain (e.g., `https://your-fork.vercel.app/` or `https://bot.yourbrand.com/`).
2. Copy the client ID Deriv issues you.
3. Set it as `CLIENT_ID` in `.env` for local development, and in your deployment environment's secrets (e.g., Vercel → Project Settings → Environment Variables) for staging/production.
4. Also set `platform.hostname.production.com` in `brand.config.json` to the same hostname you registered (no protocol, no trailing slash). The code uses this value to decide whether to connect to the production or staging WebSocket — if it doesn't match your deployed hostname, the app will silently run against staging in production.

> **Practical tip:** For a brand-new deploy you usually need to ship once to get a stable hostname *before* you can register with Deriv. See [Deployment — Example: deploying to Vercel](../README.md#example-deploying-to-vercel) for one concrete order of operations; the same shape applies to other static hosts.

> Do not change the `platform.auth2_url` values in `brand.config.json` — they must stay pointed at Deriv's OAuth server. See [Authentication](./04-authentication.md) and [Changelog — Configuration Constraints](./08-changelog.md#configuration-constraints).

---

## Development Workflow

```
1. Edit brand.config.json        (branding changes)
2. npm run generate:brand-css    (regenerate CSS variables)
3. npm start                     (dev server, HMR)
4. Make code changes             (auto-reloads)
5. npm test                      (run tests)
6. npm run test:lint             (check code quality)
7. npm run build                 (production build)
```

---

## Project Structure

```
trading-bot-template/
├── brand.config.json              # Central branding configuration
├── rsbuild.config.ts              # Build configuration (RSBuild)
├── jest.config.ts                 # Test configuration
├── tsconfig.json                  # TypeScript configuration
├── package.json                   # Dependencies and scripts
│
├── public/                        # Static assets
│   ├── index.html                 # HTML entry point
│   └── images/                    # Static images
│
├── scripts/                       # Build and utility scripts
│   └── generate-brand-css.js      # Brand CSS generator
│
├── src/
│   ├── main.tsx                   # Application entry point
│   ├── app/
│   │   ├── App.tsx                # Root component (routing, OAuth orchestration)
│   │   ├── CoreStoreProvider.tsx  # Bridges API layer with MobX stores
│   │   └── AuthWrapper.tsx        # Authentication wrapper
│   │
│   ├── adapters/                  # Third-party library adapters
│   │   └── smartcharts-champion/  # Trading charts adapter
│   │
│   ├── components/
│   │   ├── layout/
│   │   │   ├── app-logo/          # Brand logo component (swap this out)
│   │   │   ├── header/            # Desktop & mobile header/menu
│   │   │   └── footer/            # Footer with theme toggle
│   │   └── shared/                # Shared UI components
│   │
│   ├── external/
│   │   ├── bot-skeleton/          # Core bot runtime, Blockly blocks, WebSocket
│   │   └── indicators/            # SMA, EMA, Bollinger Bands, MACD, RSI
│   │
│   ├── hooks/                     # React hooks (useStore, useLogout, useOAuthCallback…)
│   │   └── remote-config/         # Stub feature-flag hook (replace if using Growthbook)
│   │
│   ├── pages/                     # Route-level components
│   │   ├── dashboard/
│   │   ├── bot-builder/
│   │   ├── chart/
│   │   ├── tutorials/
│   │   └── main/
│   │
│   ├── services/                  # OAuth token exchange, DerivWS accounts
│   ├── stores/                    # MobX stores (RootStore + feature stores)
│   ├── styles/                    # Global SCSS entry
│   └── utils/                     # ErrorLogger, helpers
│
│   # Auto-generated (do not edit by hand):
│   src/components/shared/styles/_themes.scss   # written by generate-brand-css.js
│
└── user-guide/                    # This documentation
```

---

## Available Commands

### Development

| Command         | Description                                  |
| --------------- | -------------------------------------------- |
| `npm start`     | Start dev server at `https://localhost:8443` |
| `npm run watch` | Build in watch mode (no server)              |
| `npm run serve` | Serve the production build locally           |

### Building

| Command                 | Description                                       |
| ----------------------- | ------------------------------------------------- |
| `npm run build`         | Production build to `dist/`                       |
| `npm run build:analyze` | Production build + bundle analyzer (port 8888)    |

### Testing

| Command                          | Description              |
| -------------------------------- | ------------------------ |
| `npm test`                       | Run all tests (Jest)     |
| `npm test -- --watch`            | Run tests in watch mode  |
| `npm test -- dashboard.spec.tsx` | Run a specific test file |
| `npm run coverage`               | Generate coverage report |

### Code Quality

| Command              | Description                         |
| -------------------- | ----------------------------------- |
| `npm run test:lint`  | Run Prettier + ESLint checks        |
| `npm run test:fix`   | Auto-fix formatting and lint issues |
| `npm run type-check` | TypeScript type-check, no emit      |

### Branding

| Command                      | Description                                     |
| ---------------------------- | ----------------------------------------------- |
| `npm run generate:brand-css` | Generate CSS variables from `brand.config.json` |

---

## Environment Variables

Environment variables are injected via RSBuild's `source.define` in `rsbuild.config.ts`. Put them in `.env` at the project root for local dev, and in your host's environment for staging/production.

### Authentication (Required for Login)

| Variable       | Required?        | Description                                                                             | Example                           |
| -------------- | ---------------- | --------------------------------------------------------------------------------------- | --------------------------------- |
| `CLIENT_ID`    | Yes              | OAuth client ID you registered with Deriv for your domain                               | `32izC2lBT4MmiSNWuxq2l`           |
| `APP_ID`       | Optional         | Legacy Deriv API app ID (only needed if you maintain a Legacy Deriv API app)            | `12345`                           |
| `GD_CLIENT_ID` | Optional         | Google Drive OAuth client (enables cloud save/load for strategies)                      | `xxxx.apps.googleusercontent.com` |
| `GD_APP_ID`    | Optional         | Google Drive app ID                                                                     | `123456789`                       |
| `GD_API_KEY`   | Optional         | Google Drive API key                                                                    | `AIza...`                         |

### Translations (Optional)

| Variable               | Description                     | Example                                |
| ---------------------- | ------------------------------- | -------------------------------------- |
| `TRANSLATIONS_CDN_URL` | Translation files CDN URL       | `https://cdn.example.com/translations` |
| `R2_PROJECT_NAME`      | Crowdin project name            | `dbot`                                 |
| `CROWDIN_BRANCH_NAME`  | Crowdin branch for translations | `master`                               |

> **Note:** The application is wrapped with `@deriv-com/translations` `TranslationProvider`, but multi-language support **only works** when you have a Crowdin project configured with translation files served via a CDN. Without this, the app defaults to English and functions normally. To show the language switcher in the UI, set `enable_language_settings: true` in `brand.config.json` (see [White Labeling Guide — Footer](./03-white-labeling.md#footer)). If you are not using translations, leave it as `false` to hide the language selector.

### Monitoring (Optional)

None of these are required — the base template ships without the monitoring stack. See [Monitoring & Analytics](./07-monitoring-analytics.md) for how to re-enable each package.

| Variable                             | Description                  |
| ------------------------------------ | ---------------------------- |
| `DATADOG_APPLICATION_ID`             | Datadog RUM application ID   |
| `DATADOG_CLIENT_TOKEN`               | Datadog RUM client token     |
| `DATADOG_SESSION_REPLAY_SAMPLE_RATE` | Session replay sample rate   |
| `DATADOG_SESSION_SAMPLE_RATE`        | Session sample rate          |
| `RUDDERSTACK_KEY`                    | Rudderstack write key        |
| `TRACKJS_TOKEN`                      | TrackJS error tracking token |
| `POSTHOG_KEY`                        | PostHog API key              |
| `POSTHOG_HOST`                       | PostHog host URL             |
| `GROWTHBOOK_CLIENT_KEY`              | Growthbook client key        |
| `GROWTHBOOK_DECRYPTION_KEY`          | Growthbook decryption key    |

Reference the variables in `rsbuild.config.ts`:

```typescript
source: {
    define: {
        'process.env.CLIENT_ID': JSON.stringify(process.env.CLIENT_ID),
        // ... other variables
    },
},
```

---

## First Run Checklist

After initial setup, verify these items:

- [ ] `npm install` completes without errors
- [ ] `brand.config.json` has your brand name, domain, and colors
- [ ] `npm run generate:brand-css` validates your brand config
- [ ] `.env` contains `CLIENT_ID` with your Deriv OAuth client ID
- [ ] `npm start` launches the dev server at `https://localhost:8443`
- [ ] The browser loads the app (accept the self-signed cert warning)
- [ ] Your brand colors appear correctly in the UI
- [ ] Your logo displays in the header (if you updated `BrandLogo.tsx`)
- [ ] Clicking Login redirects to Deriv's OAuth server
- [ ] After authenticating, you are redirected back to `/` and logged in
- [ ] `npm test` passes
- [ ] `npm run build` produces a production build without errors

---

## Path Aliases

The project uses a single wildcard alias — `@/*` maps to `src/*` — configured in both `tsconfig.json` (`paths`) and `rsbuild.config.ts`. Use it for every import instead of relative paths:

```typescript
import { useStore } from '@/hooks/useStore';
import { ErrorLogger } from '@/utils/error-logger';
import BrandLogo from '@/components/layout/app-logo/BrandLogo';
```

Common roots you'll import from: `@/components`, `@/hooks`, `@/utils`, `@/constants`, `@/stores`, `@/services`, `@/external`, `@/adapters`, `@/pages`.

---

## Code Style

- **Formatter:** Prettier (auto-runs via lint-staged on commit)
- **Linter:** ESLint with TypeScript parser
- **Import order** (enforced by `eslint-plugin-simple-import-sort`):
    1. `react` first
    2. External packages
    3. Packages starting with `@`
    4. Internal aliases (`@/components`, `@/utils`, etc.)
    5. Relative imports (`../`, `./`)
    6. Style imports (`.scss`)
- **Commit messages:** Conventional commits enforced via commitlint:
    - `feat:` New features
    - `fix:` Bug fixes
    - `refactor:` Code refactoring
    - `test:` Test additions/changes
    - `docs:` Documentation changes
    - `chore:` Maintenance tasks

---

## Next Steps

| Topic                             | Guide                                                  |
| --------------------------------- | ------------------------------------------------------ |
| Understand the architecture       | [Architecture Overview](./02-architecture-overview.md) |
| Customize branding and appearance | [White Labeling Guide](./03-white-labeling.md)         |
| Set up authentication             | [Authentication Guide](./04-authentication.md)         |
| Configure WebSocket connections   | [WebSocket Integration](./05-websocket-integration.md) |
| Set up error handling             | [Error Handling Guide](./06-error-handling.md)         |
| Add monitoring and analytics      | [Monitoring & Analytics](./07-monitoring-analytics.md) |
| See what changed from Deriv Bot   | [Changelog](./08-changelog.md)                         |
