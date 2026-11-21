# AliAuction (ThinkCode Full-Stack Template) — AI Agent & Developer Guidelines

## 1. Scope and Responsibilities

When operating on this repository:
- Act with the standards of a Principal Software Engineer, Laravel Architect, React Architect, Mobile Engineer, and DevOps Engineer.
- This codebase is a reusable enterprise starter template. Avoid client-specific business logic or premature multi-tenancy unless explicitly scoped.
- Preserve existing configurations, folder structures, and linting rules.
- Keep backend, frontend, and mobile consistent: the Laravel API is the single source of truth for both clients.

---

## 2. Directory Layout & Rules

```
aliauction/
├── apps/
│   ├── backend/          # Laravel 13 API (PHP 8.4-FPM)
│   ├── frontend/         # React 19 SPA (Vite + TypeScript)
│   └── mobile/           # Mobile client (consumes the backend API)
├── docs/                 # System documentation & specifications
├── infra/                # Docker Compose & Coolify deployment infrastructure
│   ├── coolify/          # Coolify deployment guides, Dockerfiles & env template
│   │   ├── docker/       # Dockerfiles / container build assets for Coolify
│   │   ├── .env.example  # Environment template for Coolify deployments
│   │   └── README.md     # Coolify deployment guide
│   ├── .dockerignore     # Docker build-context exclusions for infra builds
│   ├── docker-compose.yml     # Production & Coolify base stack
│   └── docker-compose.dev.yml # Local development port overrides
├── .dockerignore         # Root Docker build-context exclusions
├── .env.example          # Root environment template (copy to .env)
├── .gitignore
├── Makefile              # Canonical developer commands
└── README.md             # Project overview & operator instructions
```

### General Rules
1. Never commit real `.env` files. Any new environment variable must be added to the relevant `.env.example` (root and/or `infra/coolify/.env.example`) with a safe placeholder value.
2. Keep `.dockerignore` files updated when adding new heavy or sensitive directories (e.g. `node_modules`, `vendor`, build output, local env files).
3. Put architecture notes, API contracts, and specs in `docs/`, not in code comments or the README.

### Backend Guidelines (`apps/backend/`)
1. **Framework:** Laravel 13.x. Use native Laravel features (Form Requests, API Resources, Gates, Notification channels).
2. **Authentication:**
   - Prefer Sanctum cookie SPA sessions for browser requests. Always verify CSRF token and stateful domain configuration.
   - Use Personal Access Tokens (`HasApiTokens`) only when `device_name` is explicitly provided (this is the path used by the mobile app).
3. **Database:** PostgreSQL is the primary database. SQLite in-memory is permitted only in isolated PHPUnit test configurations.
4. **Queue Workers:** Redis is the queue driver. Never run blocking operations synchronously in HTTP request controllers.
5. **Exception Handling:** Always return consistent JSON for `/api/*` endpoints. Never leak database credentials or stack traces in production.
6. **API Stability:** The API serves both web and mobile. Avoid breaking response shapes; version or deprecate endpoints deliberately, and document changes in `docs/`.

### Frontend Guidelines (`apps/frontend/`)
1. **Framework:** React 19 with strict TypeScript (`tsconfig.app.json`).
2. **Server State:** Use TanStack Query for all remote HTTP state. Do not invent custom global state stores for cached API resources.
3. **Form Handling:** Use React Hook Form with Zod schemas. Display field-level validation errors clearly.
4. **Styling:** Use Tailwind CSS with clean utility composition via `cn()` (`lib/utils.ts`).
5. **No Embedded Secrets:** Never store sensitive API secrets in `VITE_` variables; they are compiled into the client bundle and accessible in the browser.

### Mobile Guidelines (`apps/mobile/`)
> TODO: confirm the mobile stack (e.g. React Native / Expo, Flutter) and replace the generic notes below with stack-specific rules.

1. **API Only:** The mobile app talks to the Laravel API exclusively. No direct database or Redis access, and no business logic that should live on the server.
2. **Authentication:** Use Sanctum Personal Access Tokens with a `device_name`. Store tokens only in secure storage (iOS Keychain / Android Keystore), never in plain AsyncStorage/SharedPreferences or logs.
3. **No Embedded Secrets:** Anything shipped in the app bundle can be extracted. Never include server secrets or private API keys; use public/publishable keys only.
4. **Configuration:** The API base URL and environment flags come from environment config, not hard-coded values. Add new variables to the mobile `.env.example` if one exists.
5. **Consistency:** Follow the same validation rules and error shapes as the web frontend; handle `422` field errors, `401` (token expired/revoked), and network failures explicitly.
6. **Preserve Tooling:** Keep the existing lint, formatting, and TypeScript/analyzer configuration in `apps/mobile/` intact.

### Infrastructure Guidelines (`infra/`)
1. `docker-compose.yml` is the production/Coolify base stack; `docker-compose.dev.yml` only overrides ports and dev-only settings. Do not put production secrets in either file.
2. Coolify-specific Dockerfiles live in `infra/coolify/docker/`. Keep them in sync with the deployment guide in `infra/coolify/README.md`.
3. Environment variables required for deployment must be listed in `infra/coolify/.env.example`.

---

## 3. Mandatory Verification Checklist

Before committing any feature or submitting work:
1. `make test-backend` passes (PHPUnit feature and unit tests).
2. `make test-frontend` passes (Vitest unit and interaction tests).
3. `cd apps/frontend && npm run lint` passes with 0 errors.
4. `cd apps/frontend && npm run build` completes cleanly.
5. `docker compose --env-file .env -f infra/docker-compose.yml config` validates syntax.
6. If `apps/mobile/` was changed: run its lint and test commands (see its README/`package.json`) and confirm they pass.
7. If any environment variable was added or renamed: the matching `.env.example` file(s) are updated.
8. Never claim a command passes without executing it and verifying its exit code.
