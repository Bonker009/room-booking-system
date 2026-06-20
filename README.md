# Room Booking System

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Next.js](https://img.shields.io/badge/Next.js-16-black)](https://nextjs.org)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-blue)](https://typescriptlang.org)
[![Deploy on Vercel](https://img.shields.io/badge/Deploy-Vercel-black)](https://vercel.com)

An open-source **room booking** system: browse, create, edit, and delete reservations with rich calendar views. Built with Next.js App Router, Better Auth, SQLite, and Keycloak OIDC for enterprise-grade authentication.

**Live demo:** https://room-booking-lac.vercel.app

## Features

- 📅 **Calendar views** — month, week, day, year, and agenda with drag-and-drop rescheduling
- 🔁 **Recurring bookings** — daily, weekly, or monthly with an end date
- 📆 **Multi-day bookings** — one record per day, linked by a series ID
- ✅ **Approval flow** — specific rooms require admin approval
- 🏠 **Kiosk mode** — live room status board for displays (no login required)
- 📊 **Timetable view** — room × time grid for a selected day
- 📤 **CSV export** — download filtered booking tables
- 🐳 **Docker ready** — single `docker compose up` deployment
- 🔐 **Keycloak OIDC** — enterprise SSO via Better Auth

## Stack

- **Next.js 16** (App Router), **React 19**, **TypeScript**, **Tailwind CSS 4**
- **Better Auth** + **better-sqlite3** (`data/auth.db`)
- **Keycloak** via Generic OAuth plugin (`providerId`: `keycloak`)
- Bookings persisted in SQLite **`data/bookings.db`** (`lib/bookings-store.ts`, `lib/db.ts`)
- **Calendar views** via vendored `calendar/` module ([big-calendar](https://github.com/lramos33/big-calendar))

## Prerequisites

- Node.js 20+
- A **Keycloak** realm and OIDC client (see [Authentication](#authentication))

## Quick Start

1. Clone the repo and install dependencies:

   ```bash
   git clone https://github.com/Bonker009/room-booking-system.git
   cd room-booking-system
   npm ci
   ```

2. Copy environment variables and fill in secrets:

   ```bash
   cp .env.example .env.local
   ```

3. Apply Better Auth DB migrations:

   ```bash
   npx @better-auth/cli@latest migrate --config lib/auth.ts --yes
   ```

4. Run the dev server:

   ```bash
   npm run dev
   ```

5. Open [http://localhost:3000](http://localhost:3000). Unauthenticated users are redirected to `/sign-in`.

## Authentication

Login is **Keycloak-only** (no email/password). Flow:

1. User hits a protected route → `middleware.ts` checks for a session cookie; if missing → redirect to `/sign-in`.
2. User clicks **Continue with Keycloak** → Better Auth starts OAuth2 with `providerId: "keycloak"`.
3. After Keycloak redirects back, Better Auth creates a session cookie.
4. Booking **API routes** additionally verify the session server-side (`lib/require-session.ts`).

### Environment Variables

| Variable | Purpose |
|----------|---------|
| `BETTER_AUTH_SECRET` | Encryption/signing key — **≥ 32 characters** (`openssl rand -base64 32`). |
| `BETTER_AUTH_URL` | Public base URL of this app, no trailing slash (e.g. `http://localhost:3000`). |
| `BETTER_AUTH_TRUSTED_ORIGINS` | Optional; comma-separated extra allowed origins. |
| `KEYCLOAK_CLIENT_ID` | Keycloak client ID. |
| `KEYCLOAK_CLIENT_SECRET` | Keycloak client secret (confidential client). |
| `KEYCLOAK_ISSUER` | Full realm issuer URL, e.g. `https://your-keycloak.example.com/realms/<realm>`. |

### Keycloak Client Settings

In the Keycloak admin console, for the client used by this app:

| Setting | Value |
|---------|-------|
| **Valid redirect URIs** | `{BETTER_AUTH_URL}/api/auth/oauth2/callback/keycloak` |
| **Web origins** | Your app origin (e.g. `http://localhost:3000`) |

### Key Auth Files

| Path | Role |
|------|------|
| `lib/auth.ts` | Better Auth server config (SQLite path, Keycloak plugin, `nextCookies()`). |
| `lib/auth-client.ts` | Browser client + `genericOAuthClient()` plugin. |
| `app/api/auth/[...all]/route.ts` | Auth HTTP handler (`toNextJsHandler`). |
| `middleware.ts` | Redirect unauthenticated users to `/sign-in`. |
| `app/sign-in/page.tsx` | Keycloak sign-in button. |

## API Reference

All booking endpoints require an authenticated session (cookie).

| Endpoint | Description |
|----------|-------------|
| `GET /api/bookings` | List bookings with optional filters, search, sort, and pagination |
| `GET /api/bookings/availability` | Per-room free/busy for a given slot |
| `POST /api/bookings` | Create a single-day booking |
| `POST /api/bookings/bulk` | Create a date-range booking (one record per day; HTTP 207 on partial success) |
| `GET/PUT/DELETE /api/bookings/[id]` | Read, update, or delete a booking |
| `GET /api/kiosk/status` | Live room status (no auth required) |

**Rooms** are defined in `lib/rooms.ts`. The dashboard defaults to the **Table** view. Switch to **Calendar** or **Timetable** for other layouts.

## Docker

```bash
docker compose up -d --build
```

- Mounts **`./data:/app/data`** so `auth.db` and `bookings.db` persist across rebuilds
- Set auth env vars in **`.env`** (see `docker-compose.yml`)
- `BETTER_AUTH_URL` must match the browser origin used to access the app

See [`docs/agents/docker.md`](docs/agents/docker.md) for full details.

## Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Dev server (Turbopack) |
| `npm run build` | Production build |
| `npm run start` | Start production server |
| `npm run lint` | ESLint |
| `npm run migrate:bookings` | Import `data/bookings.json` → `data/bookings.db` |

## Contributing

Contributions are welcome! To get started:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes using [Conventional Commits](https://www.conventionalcommits.org): `git commit -m "feat: add your feature"`
4. Push to your branch: `git push origin feature/your-feature`
5. Open a Pull Request

Please follow the existing code style and conventions described in [`AGENTS.md`](AGENTS.md).

## License

This project is licensed under the [MIT License](LICENSE).
