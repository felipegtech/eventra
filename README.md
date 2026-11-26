# Eventra

Internal operations dashboard for an event production agency to manage its daily workflow: the
**Clients** requesting events, the **Vendors** catalog delivering services, and the **Events**
that connect both, with budget, schedule, and lifecycle status tracking.

The repository is organized as a multi-app workspace:

- **Backend** (`backend/`) — REST API built with NestJS 12 (ESM), TypeORM, and SQLite
  (`better-sqlite3`). Owns persistence, validation, seed data, and JWT/RBAC security.
- **Frontend** (`frontend/`) — Internal operations SPA built with Vue 3, Vite, and TypeScript.
  Keeps no domain data locally; reads and writes exclusively through the backend API.
- **Public Portal** (`nuxt/`) — Companion Nuxt 4 site demonstrating hybrid rendering modes
  (SSG guides, SSR vendor directory via Nitro API routes, and CSR budget estimator).

## Stack

### Backend

- **NestJS 12** (ESM) + **TypeORM** over **SQLite** (`better-sqlite3`)
- **class-validator** + **class-transformer** with global `ValidationPipe`
- **JWT** authentication (`@nestjs/jwt`) + **bcrypt** password hashing + global `AuthGuard` /
  `RolesGuard`
- **Vitest** for unit tests, **Oxlint** + **Prettier** for code quality

### Frontend

- **Vue 3** (`<script setup lang="ts">`) + **TypeScript** + **Vite**
- **Axios** for all API calls (with global 401 session-expiry interceptor)
- **Pinia** for session state only (`SessionStore`)
- **Vue Router** with authentication and admin role guards
- **Chart.js** wrapped in a single reusable `BaseChartComponent`
- **vue-toastification** for feedback notifications
- **Oxlint**, **ESLint**, **vue-tsc**, and **Prettier**

### Public Portal

- **Nuxt 4** with per-route `routeRules` (SSG, SSR, CSR) and Nitro server routes

## Requirements

- **Node.js** `^22.18.0 || >=24.12.0`
- **Docker** with the Compose plugin (for containerized deployment)

## Running Locally

Run the backend first so the API and SQLite database are ready before the frontend loads.

### 1. Backend

```sh
cd backend
npm install
npm run start:dev
```

Listens on `http://localhost:3000/api` (`GET /api` returns `API is running`). On first startup,
TypeORM creates and seeds `backend/database.sqlite` automatically.

### 2. Frontend

```sh
cd frontend
npm install
npm run dev
```

Runs the Vite dev server (default `http://localhost:5173`) using `VITE_API_BASE_URL` from
`frontend/.env.development`.

### 3. Public Portal (Nuxt)

```sh
cd nuxt
npm install
npm run dev
```

## Quality Scripts

### Backend (`backend/`)

```sh
npm run start:dev    # Watch mode API
npm run build        # Compile & type-check (nest build)
npm run lint         # Oxlint over src/
npm run format       # Prettier over src/
npm test             # Vitest unit tests
```

### Frontend (`frontend/`)

```sh
npm run dev          # Vite dev server
npm run type-check   # vue-tsc --build
npm run build        # type-check + production bundle
npm run lint         # Oxlint + ESLint
npm run format       # Prettier over src/
```

### Public Portal (`nuxt/`)

```sh
npm run dev          # Nuxt dev server
npm run build        # Hybrid Nitro production build
npm run lint         # Oxlint
npm run format       # Prettier
```

## Deployment (Docker Compose)

The full system runs with multi-stage Docker images (`backend` with a persistent `backend-data`
SQLite volume on port `3000`, and `frontend` served via Nginx on port `80`). Compiled `dist/`
directories and `.env` secrets are never committed to Git.

```sh
git clone https://github.com/felipegtech/eventra.git
cd eventra
./deploy.sh <VM_EXTERNAL_IP>
```

`deploy.sh` configures `VITE_API_BASE_URL` and `CORS_ORIGIN`, generates a random `JWT_SECRET`
once in `.env` (gitignored), and runs `docker compose up -d --build`.

## Documentation

- **[CONTEXT.md](./CONTEXT.md)** — Official domain glossary (ubiquitous language).
- **[docs/adr/](./docs/adr/)** — Architecture Decision Records documenting key technical choices.
