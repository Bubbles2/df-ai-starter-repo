---
name: react-web-express-starter
description: Bootstrap a new React (web) + TypeScript + Vite project, with an optional Express backend and an optional Sequelize/PostgreSQL data layer. Use when the user wants to start a fresh web project with these conventions, or scaffold the layout from scratch.
version: 2.1.0
last_updated: 2026-06-16
---

# React Web + Express Starter

This skill bootstraps a new **React + TypeScript + Vite** web project. A backend (Express) and a database layer (Sequelize/PostgreSQL) are **optional** and chosen up front in Step 0.

> Only invoke when starting a **new** project or recreating this layout. For day-to-day work in an existing repo, follow that repo's `CLAUDE.md` / `AGENTS.md` instead — they reflect the actual code.

---

## When NOT to use

- For incremental work in an existing project — the rules in `CLAUDE.md` / `AGENTS.md` already cover that.
- When the target stack differs fundamentally (e.g. Next.js, Bun, Fastify, Drizzle, Prisma, TanStack Query). Adapt this skill or pick a different starter.

---

## Step 0 — Ask the user

Before writing any code, ask the following. Questions 1–2 can be asked together; ask question 3 only if the answer to 2 is yes.

> 1. **Where should the project be created?**
>    - **Current folder** — scaffold directly into the working directory
>    - **Sub-folder** — create a new directory (prompt for the name, e.g. `my-app`)
>
> 2. **Should this project include an Express backend? (y / n)**
>    - **No** → frontend-only React app (talks to an external/mock API, or none yet)
>    - **Yes** → adds an `server/` Express app served by the same dev workflow
>
> 3. *(only if backend = yes)* **Should the backend use Sequelize + PostgreSQL? (y / n)**
>    - **Yes** → Sequelize 6 over `pg`, lazy model accessors, decimal coercion
>    - **No** → no ORM. Connect to PostgreSQL with the `pg` driver directly (thin `server/config/db.js` `query()` helper) **and** include an `axios` instance (`server/config/http.js`) for calling external/upstream REST APIs. Swap the driver if the user names a different DB.

Record the three answers (location, backend, sequelize) — every later section branches on them. State the chosen configuration back to the user before scaffolding.

---

## Cross-skill rules (always active)

These existing skills govern all code written during bootstrapping:

| Skill | Applies to |
|---|---|
| `clean-typescript` | All `.ts` / `.tsx` files — `strict: true`, no `any`, no `!` assertions |
| `modern-react-components` | All React components — no `useEffect` for derived state, named function declarations |
| `modern-browser-apis` | Prefer native APIs; `Intl.NumberFormat` for formatting over libraries |
| `web-security` | Trust boundaries — validate inputs, generic error envelopes, no secrets committed |
| `api-structure` | Backend domain layout (**only if backend chosen**) — per-domain controllers, routers, and (if Sequelize) lazy models |
| `code-review` | Run before declaring the bootstrap done |

---

## Stack

**Always:**

- **Package manager**: npm.
- **Single package** (or workspace root if you later split), ES modules, Node 20.19+ (or 22+) — required by Vite 8 / ESLint 10 and `node --env-file-if-exists`.
- **Frontend**: React 19, TypeScript (`strict: true`), Vite 8, Redux Toolkit + RTK Query.
- **Tooling**: ESLint (flat config), Prettier, Vitest 4 (jsdom), Playwright, `tsc --noEmit`.

### Version policy — React 19 is the anchor

These versions were verified mutually compatible against npm as of **2026-06**. React 19 is the pin everything else aligns to. The compatibility-critical relationships:

| Package | Version | Constraint it satisfies |
|---|---|---|
| `react` / `react-dom` | `^19.2.0` | the anchor |
| `react-redux` | `^9.3.0` | needs `react ^18 \|\| ^19` (v8 does **not** support React 19) |
| `@reduxjs/toolkit` | `^2.12.0` | needs `react-redux ^9` |
| `@types/react` / `@types/react-dom` | `^19.2.0` | must match React major |
| `@vitejs/plugin-react` | `^6.0.0` | requires **Vite 8** |
| `vite` | `^8.0.0` | — |
| `vitest` | `^4.1.0` | supports Vite 8 |
| `@testing-library/react` | `^16.3.0` | supports React 19; **requires `@testing-library/dom` peer** |
| `@testing-library/dom` | `^10.4.0` | explicit peer of TL-react v16 |

If you bump React, re-check `react-redux` (≥9), the React types major, and `@testing-library/react` (≥16) first — those are the ones that break.

**If backend = yes:**

- **Backend**: Express 4 in plain JavaScript ESM (typechecked via `allowJs`).
- `concurrently` to run client + server together in `dev`.

**If sequelize = yes:**

- **Database**: PostgreSQL via Sequelize 6 over `pg`.

**If backend = yes but sequelize = no:**

- **Database access**: `pg` driver directly, exposed through a thin `server/config/db.js` `query()` helper. Swap for the user's chosen driver if specified.
- **External APIs**: an `axios` instance in `server/config/http.js` (base URL + interceptors) for calling upstream/third-party REST services.

---

## Layout

### Layout A — Frontend only (backend = no)

```
.
├── src/
│   ├── main.tsx
│   ├── styles.css
│   └── client/
│       ├── app/
│       ├── components/
│       │   └── <domain>-domain/
│       ├── features/<domain>/
│       └── store/
│           ├── api/                # RTK Query services (baseUrl → external API)
│           ├── slices/
│           ├── hooks.ts
│           ├── index.ts
│           └── provider.tsx
├── .env.example
├── index.html
├── vite.config.js
├── tsconfig.client.json
├── eslint.config.js
├── package.json
├── SPEC.md
├── AGENTS.md
├── CLAUDE.md
└── README.md
```

### Layout B — Frontend + Express backend (backend = yes)

Same as Layout A, plus a `server/` tree and `tsconfig.server.json`:

```
├── server/
│   ├── index.js                 # listen on API_PORT
│   ├── app.js                   # mount /api, serve dist/
│   ├── api/
│   │   ├── index.js
│   │   ├── health/              # always scaffold a health domain first
│   │   │   ├── healthController.js
│   │   │   └── index.js
│   │   └── <domain>/
│   │       ├── <domain>Controller.js
│   │       └── index.js
│   ├── config/
│   │   ├── database.js          # Sequelize instance               (sequelize = yes)
│   │   │   └── ─ OR ─
│   │   ├── db.js                # pg pool + query() helper          (sequelize = no)
│   │   └── http.js              # axios instance for external APIs  (sequelize = no)
│   ├── models/                  # only if sequelize = yes
│   │   ├── index.js
│   │   └── <domain>.js
│   └── utils/
└── tsconfig.server.json
```

> The `models/` directory and `config/database.js` exist **only when Sequelize is chosen**. Without Sequelize, use `config/db.js` and query inside controllers directly.

---

## `package.json` scripts (canonical)

Include only the lines relevant to the chosen configuration.

```jsonc
{
  "scripts": {
    // --- always ---
    "dev:client": "vite --host 127.0.0.1",
    "build": "vite build",
    "preview": "vite preview --host 127.0.0.1",
    "lint": "eslint src --ext .js,.jsx,.ts,.tsx",
    "format": "prettier --write \"src/**/*.{js,jsx,ts,tsx,css}\" \"*.{js,md,json,html}\"",
    "typecheck": "tsc -p tsconfig.client.json",
    "test": "vitest run --passWithNoTests --environment jsdom",
    "test:e2e": "playwright test --pass-with-no-tests",

    // --- backend = yes (replaces "dev", extends lint/format/typecheck) ---
    "dev": "concurrently \"npm run dev:client\" \"npm run dev:server\"",
    "dev:server": "node --env-file-if-exists=.env server/index.js",
    "start": "node --env-file-if-exists=.env server/index.js",
    "lint:full": "eslint src server --ext .js,.jsx,.ts,.tsx",
    "typecheck:server": "tsc -p tsconfig.server.json"
    // (when backend = yes, fold server globs into lint/format/typecheck rather than keeping :full variants)
  }
}
```

> When **backend = no**, set `"dev": "npm run dev:client"` and drop every server-related script.
> `--env-file-if-exists` is preferred over `--env-file` so the server still boots when `.env` is absent.

---

## Conventions

**Always:**

- **TypeScript on the client** (`strict: true`). Server JS is ESM, typechecked via `allowJs`.
- **RTK Query** for all client server-state. One `createApi` per domain under `src/client/store/api/`.
- **Generic error envelope**: `{ error: "<message>" }`. Show generic text in the UI; log the full error server-side only.
- **kebab-case filenames**, **PascalCase** components, **camelCase** identifiers.
- **Number formatting** via `Intl.NumberFormat` — no `numeral.js` or similar.

**If backend = yes:**

- **Per-domain folders** under `server/api/<domain>/` with `<domain>Controller.js` + `index.js` (see `api-structure`).
- **Named exports only** in controllers; never `module.exports`.
- **`Promise.all`** for parallel independent reads.
- Decide read-only vs read-write **with the user** — do not assume either. (The original ERP project was read-only; a general project usually is not.)

**If sequelize = yes:**

- **Lazy Sequelize models** via `getXxxModels()` accessors (avoids instantiating Sequelize at module load).
- **Decimal coercion** in a single `toNumber()` util — `pg` serialises `DECIMAL` as strings, which break client arithmetic.
- If you interpolate dynamic identifiers (schema/table names) into raw SQL, **validate them against a strict allowlist regex** (`^[a-zA-Z_][a-zA-Z0-9_]*$`) via a helper — never string-concatenate user input into SQL (see `web-security`).

**If sequelize = no:**

- Centralise the `pg` pool in `server/config/db.js`; export a `query(text, params)` helper. **Always use parameterised queries** (`$1, $2`) — never string interpolation.
- Centralise the `axios` instance in `server/config/http.js` (one `axios.create({ baseURL })`); add interceptors for auth headers / error normalisation there, not in each controller. Never log secrets; keep base URLs and keys in `.env`.

---

## Bootstrap order

1. `npm init -y`, set `"type": "module"` in `package.json`.
2. **Install dependencies** (this skill is self-contained — do not reference any other project's `package.json`):

   ```bash
   # Always — frontend runtime (React 19 anchor)
   npm install react@^19.2.0 react-dom@^19.2.0 @reduxjs/toolkit@^2.12.0 react-redux@^9.3.0

   # Always — tooling (dev). Vite 8 + plugin-react 6 + vitest 4 move together.
   npm install -D \
     vite@^8.0.0 @vitejs/plugin-react@^6.0.0 typescript@~5.9.2 \
     eslint@^9.0.0 prettier@^3.8.0 \
     vitest@^4.1.0 jsdom@^29.0.0 \
     @testing-library/react@^16.3.0 @testing-library/dom@^10.4.0 @testing-library/jest-dom@^6.9.0 \
     @playwright/test@^1.61.0 concurrently@^10.0.0 \
     @types/react@^19.2.0 @types/react-dom@^19.2.0 @types/node@^20.19.0

   # If backend = yes  (Express 4 — see note; v5 is current latest if you opt in)
   npm install express@^4.21.0

   # If sequelize = yes
   npm install sequelize@^6.37.0 pg@^8.21.0 pg-hstore@^2.3.0

   # If backend = yes and sequelize = no (pg driver + axios for external APIs)
   npm install pg@^8.21.0 axios@^1.18.0
   ```

   > **TypeScript is pinned to `~5.9.2`** (matching the mobile starter), not the latest 6.x — TS 6 is new and brings no benefit needed here. **Express stays on 4** to match the templates in `api-structure`; npm's current `express` latest is **5.x** (breaking routing changes) — only move to it deliberately, not via an unpinned install. `sequelize` stays on **6** (v7 is not yet stable). None of these touch React, so they're unaffected by the React 19 bump.

3. Add `tsconfig.client.json` (strict, `jsx: "react-jsx"`, `moduleResolution: "Bundler"`). **If backend = yes**, also add `tsconfig.server.json` (NodeNext, `allowJs`, `checkJs: false`).
4. Add `vite.config.js`. **If backend = yes**, add the `/api` proxy to the Express port.
5. Add `eslint.config.js` (flat) with separate blocks for `src/**/*.{ts,tsx}` and `src/**/*.{js,jsx}` (and `server/**/*.js` if backend = yes).
6. Scaffold `src/main.tsx`, `src/client/app/`, `src/client/store/` (provider, hooks, index, slices, api).
7. **If backend = yes**: scaffold `server/index.js`, `server/app.js`, the DB layer (`config/database.js` for Sequelize or `config/db.js` for `pg`), and the first domain (`server/api/health/`) per `api-structure`.
8. Add `.env.example` (placeholders only; `.env` in `.gitignore`).
9. Add `SPEC.md`, `AGENTS.md`, `CLAUDE.md`.
10. Run the full validation suite below — all green before committing.

---

## Validation

```powershell
npm run lint
npm run typecheck       # runs client; and server too when backend = yes
npm test
npm run build
```

If you can't run these, report it explicitly — don't claim the bootstrap is done. Then run `code-review` on the scaffolded output.
