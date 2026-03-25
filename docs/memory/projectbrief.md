# Project Brief — Excalidraw Monorepo (Memory Bank)

## Executive summary

This repository is a **Yarn workspaces monorepo** for **Excalidraw**: a **browser-based whiteboard / diagramming editor** with a hand-drawn look. It serves **end users** via a first-party web app and **product teams** via the publishable React package **`@excalidraw/excalidraw`**.

---

## Core purpose

**Problem:** People need a fast, privacy-friendly way to sketch diagrams, annotate ideas, and collaborate visually—without heavyweight design tools—both standalone and inside other web apps.

**Who it is for**

- **General users & teams**: diagramming, brainstorming, sharing boards (Excalidraw app).
- **Developers & integrators**: embedding the editor in React apps, custom hosting, examples (Next.js, script tag).

---

## High-level technology stack

| Layer | Choices |
|--------|---------|
| **Languages** | **TypeScript** (primary), **JavaScript** where allowed, **SCSS** for styles |
| **UI framework** | **React 19** |
| **App bundling / dev** | **Vite 5**, `@vitejs/plugin-react`, PWA-related plugins, SVGR, EJS/HTML helpers |
| **State** | **Jotai** (`jotai-scope` in library) |
| **Monorepo** | **Yarn 1 workspaces** (`excalidraw-app`, `packages/*`, `examples/*`) |
| **Quality** | **ESLint**, **Prettier**, **Vitest** (+ canvas mocks, coverage), **TypeScript** strict project references / path aliases |
| **Collaboration (app)** | **socket.io-client**; remote persistence **Firebase** (Firestore / Storage patterns in app code) |
| **Observability (app)** | **Sentry** (`@sentry/browser`) |
| **Editor / rendering (library)** | **Canvas**-centric stack: **roughjs**, **perfect-freehand**, **pica**, PNG chunk helpers, **fractional-indexing**, **browser-fs-access** |
| **UI primitives (library)** | **radix-ui**, **clsx**, **CodeMirror 6** (`@codemirror/*`, `@lezer/highlight`) |
| **Diagram import** | **@excalidraw/mermaid-to-excalidraw**; TTD / Mermaid-related UI under library components |
| **i18n (app)** | **i18next-browser-languagedetector**; Crowdin config at repo root |
| **Deploy / ops** | **Dockerfile**, **docker-compose**, **Vercel** (`vercel.json`), Node **≥ 18** |

*Note:* There is **no single root `src/` folder**. Application source lives under `excalidraw-app/`; shared packages use `packages/<name>/src/` (or package roots as defined in each package).

---

## Project structure

| Path | Role |
|------|------|
| **`excalidraw-app/`** | Standalone web app: `App.tsx`, `index.tsx`, Vite config, collaboration (`collab/`), persistence (`data/`, Firebase), app-specific components, tests |
| **`packages/excalidraw/`** | Embeddable **`@excalidraw/excalidraw`** React library: editor UI, scene data, encryption helpers, AI-related types, locales, built `dist/` outputs |
| **`packages/common/`** | **`@excalidraw/common`** — shared constants, utilities, cross-cutting helpers |
| **`packages/element/`** | **`@excalidraw/element`** — element model, geometry, element-level operations |
| **`packages/math/`** | **`@excalidraw/math`** — math utilities for layout / geometry |
| **`packages/utils/`** | **`@excalidraw/utils`** — general helpers consumed by other packages |
| **`examples/with-nextjs/`** | Next.js integration example |
| **`examples/with-script-in-browser/`** | Browser script / Vite example for non-React or simple embedding flows |
| **`scripts/`** | Build, release, locales, and tooling automation |
| **`public/`** | Shared static assets at monorepo level (as used by project conventions) |
| **`firebase-project/`** | Firebase-related project assets / configuration supporting setups documented in-repo |
| **Root configs** | `package.json`, `tsconfig.json` (path aliases to packages), `vitest.config.mts`, `.eslintrc.json`, `.env.*`, `Dockerfile`, CI under `.github/` |

---

## Core requirements

- **Runnable first-party app**: Dev and production builds via Vite; static serving / Docker / Vercel-style deployment paths supported by scripts and config.
- **Publishable library**: `@excalidraw/excalidraw` (and internal `@excalidraw/*` packages) build ESM bundles + TypeScript declarations for consumers.
- **Editor fidelity**: Hand-drawn rendering, shape tools, text, images, import/export, library of shapes, accessibility to the degree implemented in components.
- **Optional real-time collaboration** (app): Presence and syncing over sockets; encrypted scene handling and file upload patterns integrated with Firebase where configured.
- **Extensibility hooks**: AI-related surfaces in app/library; Mermaid / text-to-diagram flows in the library.
- **Engineering hygiene**: Typecheck, lint, format, and test gates defined at monorepo root; strict TypeScript baseline.

---

## Success criteria

- **Build graph**: `build:packages` produces internal packages; `excalidraw-app` build completes for target environments (including Docker-oriented env flags where used).
- **Quality bar**: `test:typecheck`, `test:code`, `test:app` (Vitest) pass in CI/local workflows expected by maintainers.
- **Consumability**: External apps can depend on `@excalidraw/excalidraw` with documented React peer versions; examples demonstrate integration patterns.
- **Operational readiness**: Environment-driven configuration for analytics/Sentry, Firebase, and collaboration endpoints; no hard dependency on undeclared runtime for core offline editing paths where applicable.

---

## Main features (product / architecture view)

- **Interactive canvas editor**: Creation and manipulation of elements, selection, undo/redo patterns, export formats, and local/remote file flows.
- **Embeddable React component**: Same core experience consumable from host applications with CSS bundles exposed for styling integration.
- **Collaboration**: Shared sessions via `socket.io-client`; encrypted payloads and Firebase Storage usage for shared assets in collaborative scenarios.
- **Rich text / code editing**: CodeMirror-powered experiences where the editor requires structured text input.
- **Mermaid / diagram generation**: Conversion and UI flows leveraging `@excalidraw/mermaid-to-excalidraw` and TTD-related dialogs.
- **PWA & offline-oriented tooling**: Vite PWA plugin at workspace level; service worker behavior tied to app build configuration.
- **Internationalization**: Locale coverage tracked via scripts; Crowdin workflow for translations.

---

## Boundary conditions (non-goals in-repo)

- This repo is **front-end–centric**: server behavior for collaboration, AI providers, and Firebase is assumed to exist or be configured externally; specifics live in deployment env and service setup—not fully replicated as a backend monolith here.

---

## Orientation map (where to start reading)

- **App shell**: `excalidraw-app/App.tsx`
- **Collaboration**: `excalidraw-app/collab/` (e.g. `Collab.tsx`, `Portal.tsx`)
- **Remote storage**: `excalidraw-app/data/firebase.ts`
- **Library public API**: `packages/excalidraw/index.tsx`
- **Editor composition**: `packages/excalidraw/components/App.tsx`
