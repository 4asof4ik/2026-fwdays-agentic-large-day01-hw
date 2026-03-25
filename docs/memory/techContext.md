# Technical Context

## Core Tech Stack

* **Frontend (primary app)**: React **19.0.0** + React DOM **19.0.0**, **Vite 5.0.12** SPA (`excalidraw-app/`). Client-side only; no App Router for the main product.
* **Example integration**: **Next.js 14.1** in `examples/with-nextjs` (React 19, separate dev/build scripts).
* **Language**: **TypeScript 5.9.3** — **Strict mode: Yes** (`"strict": true` in root `tsconfig.json` and `packages/tsconfig.base.json`; `noFallthroughCasesInSwitch`, `forceConsistentCasingInFileNames`).
* **Styling**: **Sass 1.51.0** (`packages/excalidraw`), **clsx**, **Radix UI 1.4.3**. **Tailwind CSS / shadcn**: not used in this repo (no Tailwind dependency in `package.json` files).
* **State management**: **Jotai 2.11.0** (app + library); ESLint forbids importing `"jotai"` directly — use app-specific modules (`editor-jotai` / `app-jotai`).
* **Backend/API**: No first-party server in-repo for the main app. Integrations via env-configured HTTP/WebSocket endpoints, **Firebase 11.3.1**, **Socket.io client 4.7.2**, optional **Sentry 9.0.1**. Library build outputs are static ESM under `packages/*/dist`.
* **Notable UI/editor dependencies** (library): **CodeMirror 6**, **@lezer/**, **roughjs**, **perfect-freehand**, **@excalidraw/mermaid-to-excalidraw 2.1.1**.

## Monorepo layout

* **Package manager**: **Yarn 1** — `packageManager: "yarn@1.22.22"`, workspaces: `excalidraw-app`, `packages/*`, `examples/*`.
* **Workspace packages** (version **0.18.0** where published): `@excalidraw/common`, `@excalidraw/element`, `@excalidraw/math`, `@excalidraw/excalidraw`, `@excalidraw/utils`.
* **Path aliases**: `@excalidraw/*` mapped in root `tsconfig.json` to `packages/*/src` or package entry points; mirrored in Vite and Vitest resolve config.

## Development Workflow

* **Build tool**: **Vite** with `@vitejs/plugin-react 3.1.0`, `vite-plugin-checker` (TypeScript + ESLint in dev), `vite-plugin-pwa`, `vite-plugin-svgr`, `vite-plugin-ejs`, custom **woff2** browser plugin, `vite-plugin-html`, sitemap plugin.
* **Linting / formatting**: **ESLint** (extends `@excalidraw/eslint-config`, `eslint-config-react-app`; **max warnings = 0** in `test:code`). **Prettier 2.6.2** with `@excalidraw/prettier-config`. **lint-staged** (`.lintstagedrc.js`) runs ESLint with `--fix` on staged JS/TS/TSX and Prettier on CSS/SCSS/JSON/MD/HTML/YML.
* **Git hooks**: `prepare` runs `husky install` (`.husky/` is present; pre-commit wiring may vary locally).
* **Key commands** (root `package.json`):
  * **`start`**: `yarn --cwd ./excalidraw-app start` — Vite dev server (port from `VITE_APP_PORT` or **3000** per `vite.config.mts`).
  * **`build`**: app build via `excalidraw-app` (`vite build` + version script).
  * **`build:packages`**: ESM builds for `common` → `math` → `element` → `excalidraw`.
  * **`test`**: **Vitest 3.0.6** (root config `vitest.config.mts`, **jsdom**, `setupTests.ts`, `vitest-canvas-mock`, `@testing-library/*`).
  * **`test:all`**: `tsc` + ESLint + Prettier `--list-different` + Vitest `--watch=false`.
  * **`test:typecheck`**: `tsc` (root project, `noEmit: true`).
  * **`test:code`**: ESLint on `.js,.ts,.tsx`.
  * **`test:other`**: Prettier check.
  * **`test:coverage` / `test:ui`**: Vitest coverage / UI mode.
  * **`fix`**: Prettier write + ESLint `--fix`.
  * **`start:production`**: build then static serve (`http-server` on port **5001** in app scripts).
  * **`build:preview`**: build + `vite preview` on **5000**.
  * **`clean-install`**, **`rm:build`**, **`rm:node_modules`**: maintenance scripts.

## Technical Constraints & Decisions

* **SSR / ISR / Server Actions**: Not used for the main Excalidraw app (static/Vite SPA). Next.js sample is illustrative only.
* **TypeScript**: Strict; `allowJs: true` at repo root; `skipLibCheck: true`; `jsx`: `react-jsx`; `module`/`moduleResolution`: `ESNext` / `node`.
* **PWA**: `vite-plugin-pwa` — Workbox caching for fonts, locales, lazy chunks; dev PWA gated by **`VITE_APP_ENABLE_PWA`**.
* **ESLint policy highlights** (`/.eslintrc.json`): `@typescript-eslint/consistent-type-imports` (separate type imports); ordered imports with `@excalidraw/**` grouping; **`react/jsx-no-target-blank`** with `allowReferrer`; in `packages/excalidraw` (non-test), restricted imports (no barrel `index.tsx` self-imports; restricted relative paths).
* **Testing**: Vitest coverage thresholds — lines/statements **60%**, branches **70%**, functions **63%** (`vitest.config.mts`).
* **Library packaging**: `@excalidraw/excalidraw` ships **dev/prod** ESM + CSS exports (`index.css`); build via `scripts/buildPackage.js` + declaration emit.

## Environment Variables (Vite — `excalidraw-app/vite-env.d.ts`)

All are `VITE_*` (inlined at build time). Documented in typings:

* **`VITE_APP_PORT`**, **`VITE_APP_BACKEND_V2_GET_URL`**, **`VITE_APP_BACKEND_V2_POST_URL`**
* **`VITE_APP_WS_SERVER_URL`**, **`VITE_APP_PORTAL_URL`**, **`VITE_APP_AI_BACKEND`**
* **`VITE_APP_FIREBASE_CONFIG`** (JSON string)
* **`VITE_APP_DEV_DISABLE_LIVE_RELOAD`**, **`VITE_APP_DISABLE_SENTRY`**, **`VITE_APP_COLLAPSE_OVERLAY`**, **`VITE_APP_ENABLE_ESLINT`**, **`VITE_APP_ENABLE_PWA`**
* **`VITE_APP_PLUS_LP`**, **`VITE_APP_PLUS_APP`**, **`VITE_APP_GIT_SHA`** (e.g. from **`VERCEL_GIT_COMMIT_SHA`** in `build:app`)

Additional flags used in code / builds include **`VITE_APP_ENABLE_TRACKING`** (`build:app` sets it `true`), **`VITE_APP_DISABLE_PREVENT_UNLOAD`**, and **`VITE_APP_PLUS_EXPORT_PUBLIC_KEY`**. **`packages/excalidraw/vite-env.d.ts`** declares further optional vars (e.g. library/Matomo/debug) beyond `excalidraw-app/vite-env.d.ts`.

## Deployment & Infrastructure

* **Hosting (upstream-aligned)**: **`vercel.json`** at repo root — **`outputDirectory`: `excalidraw-app/build`**, `installCommand`: `yarn install`, security/CORS-style headers and redirects for production domains.
* **Container**: **Docker** multi-stage build — **Node 18** image for `yarn install` + **`yarn build:app:docker`** (Sentry disabled via env), static assets copied to **nginx 1.27-alpine** (`/usr/share/nginx/html`).
* **CI/CD**: **GitHub Actions** under `.github/workflows/` (directory includes e.g. test, lint, Docker build/publish, autorelease, locales coverage, size-limit, Sentry production — exact job names match filenames there).
* **Database**: No primary application database in this codebase; persistence patterns include **Firebase**, **IndexedDB** (`idb-keyval`), and collaborator-facing backends configured by URLs above.

## Runtime requirements

* **Node.js**: `>=18.0.0` (root and `excalidraw-app` `package.json` `engines`).
