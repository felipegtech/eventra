# ADR-0001: Stack, Repository Structure, and Architecture Boundaries

- **Date:** 2026-11-25
- **Status:** Accepted
- **Decided by:** Felipe Gómez

## Context

Eventra is an internal operations dashboard for an event production agency managing **Clients**,
**Vendors**, **Users**, and **Events**. The repository needs a clear technical baseline before any
application code is written so that:

- local development requires minimal setup (no external database server to install or run),
- the backend and frontend have strict ownership boundaries (no duplicated domain state),
- security and input validation are enforced uniformly at the API boundary, and
- the repository stays clean and reproducible across machines and Docker builds.

## Decision

1. **Multi-app repository layout (`backend/`, `frontend/`, later `nuxt/`):**
   Keep all apps in a single repository with independent `package.json`, TypeScript, and linter
   configurations so each app builds and deploys cleanly on its own.

2. **Backend — NestJS 12 (ESM) + TypeORM + SQLite (`better-sqlite3`):**
   - Use NestJS 12 with native ES Modules (`"type": "module"`, `module: "nodenext"`) and strict
     TypeScript.
   - Persist data in a local SQLite database via `better-sqlite3` and TypeORM with database-generated
     numeric primary keys (`@PrimaryGeneratedColumn()`).
   - Populate initial demo records through idempotent backend seeders when tables are empty.

3. **Validation & Security at the API Boundary:**
   - Validate and normalize all incoming payloads via DTOs using `class-validator`,
     `class-transformer`, and a global `ValidationPipe` (`whitelist`, `forbidNonWhitelisted`,
     `transform`).
   - Protect every route by default using JWT authentication (`@nestjs/jwt`), `bcrypt` password
     hashing (`select: false`), and a global guard chain (`AuthGuard` → `RolesGuard`), opting routes
     out explicitly with `@Public()` and restricting admin endpoints with `@Roles('admin')`.

4. **Frontend — Vue 3 + Vite + TypeScript (API-first, session-only Pinia):**
   - Build the internal SPA with Vue 3 (`<script setup lang="ts">`), Vite, Vue Router, and Axios.
   - Keep **no domain data** (Clients, Vendors, Events) in Pinia or `localStorage`. All domain reads
     and writes go through the backend API; Pinia holds only the active authentication session
     (`SessionStore`), and a single `StorageService` wraps token persistence.

5. **Repository Hygiene:**
   - Never commit build outputs (`dist/`, `.nuxt/`, `.output/`), local databases (`*.sqlite`), or
     secrets (`.env`). Commit only non-secret local defaults such as `frontend/.env.development`.

## Considered Options

- **Frontend-only with `localStorage` persistence** — rejected: cannot enforce real authentication,
  role-based access control, or relational integrity across entities.
- **PostgreSQL in Docker for local development** — rejected: adds operational friction for local
  onboarding and testing without an immediate scaling requirement.
- **Storing domain entities in Pinia stores** — rejected: creates stale-cache synchronization bugs
  between browser state and the database; keeping the API as the single source of truth is simpler
  and more reliable.
- **Committing `dist/` to Git** — rejected: bloats Git history and causes deployed builds to drift
  out of sync with source code.

## Consequences

### Positive
- Anyone can clone the repo and run the full stack locally with just Node.js—no external DB setup.
- Every endpoint is secure-by-default and rejects malformed or unknown payloads with `400`/`401`/`403`
  instead of failing inside database queries with `500`.
- Clear separation of concerns makes each layer independently testable (Vitest on backend, strict
  `vue-tsc` + Oxlint/ESLint on frontend).

### Trade-offs
- SQLite is single-file and not suited for multi-instance horizontal scaling; moving to PostgreSQL
  later will require updating TypeORM column types (e.g., `simple-enum`) and connection config.
- Because the frontend holds no local domain data, the backend must be running for the SPA to load
  or mutate records.
- ESM in NestJS requires explicit `.js` extensions on relative TypeScript imports.
