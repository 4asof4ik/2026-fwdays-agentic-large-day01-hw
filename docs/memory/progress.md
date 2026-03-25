# Project Progress

## Current Status

* **Version**: **excalidraw-app** `1.0.0` (private workspace); root **excalidraw-monorepo** unpinned.
* **Overall Completion**: **Product codebase** — high (full Excalidraw editor, app shell, packages, examples present and wired). **Memory Bank / course hygiene** — partial (core docs in flight; see WIP).
* **Last Major Milestone**: Repository bootstrap and **CI/review tooling** (CodeRabbit, PR template); ongoing **Memory Bank** documentation under `docs/memory/`.

## Source inventory note (verified layout)

There is **no single repository-root `src/`**. Application code lives in **`excalidraw-app/`** (flat TSX/TS, no `src/` subfolder). Shared libraries use **`packages/<name>/src/`** for `@excalidraw/common`, `element`, `math`, **`utils`**, while **`packages/excalidraw`** uses the package root (`components/`, `actions/`, `data/`, `renderer/`, etc.). Examples: **`examples/with-nextjs/src/`** (App Router + Pages), **`examples/with-script-in-browser/`** (Vite + `components/`).

## Work in Progress (WIP)

* [ ] **Memory Bank**: Finish canonical set under `docs/memory/` (`projectbrief`, `productContext`, `techContext`, `systemPatterns`, `activeContext`); add **`progress.md`** (this file); reconcile or remove duplicate **`systemPatterns.md`** at repo root.
* [ ] **Repository hygiene**: Decide tracking for **`repomix-compressed.txt`**, **`yarn.lock`** (registry/metadata churn vs. real upgrades), and **`.cursorignore`**.
* [ ] **Assignment clarity**: Confirm whether next milestone is **documentation-only** or **feature work** in Excalidraw (per course brief).

## Completed Features

* [x] **Monorepo & toolchain**: Yarn workspaces, Vite 5, React 19, TypeScript, ESLint/Prettier/Vitest scripts at root.
* [x] **First-party app (`excalidraw-app/`)**: `App.tsx` / `index.tsx` shell; **collaboration** (`collab/Collab.tsx`, `Portal.tsx`, errors); **persistence** (`data/firebase.ts`, `LocalData.ts`, `FileManager.ts`, `TTDStorage.ts`, lockers/tab sync); **share/QR** flows; **i18n** helpers; **Sentry**; app-specific **sidebar**, **welcome**, **footer**, **AI** surface, **Plus** promo/export hooks; **theme** handling.
* [x] **Embeddable library (`packages/excalidraw`)**: Public **`index.tsx`**; large **UI** surface (`components/App.tsx`, `LayerUI`, main menu, sidebars, mobile toolbars, command palette, dialogs, color/font pickers, TTD/Mermaid panels, canvases, stats, follow mode, laser pointer, library UI, etc.); **`actions/`** for editor commands; **`data/`** (blob, restore, library, encryption helpers); **`renderer/`** and **`scene/`**; **snapping**, **history**, **workers**, **fonts** (incl. subset/loaders), **tests** under `tests/`.
* [x] **Shared packages**: **`@excalidraw/common`** (constants, utils, events, bounds, colors, etc.); **`@excalidraw/element`** (scene, types, geometry, arrows, frames, text, image, bindings, z-order, tests); **`@excalidraw/math`**; **`@excalidraw/utils`**.
* [x] **Examples**: **Next.js** wrapper and routes (`src/app/page.tsx`, `src/pages/…`); **script/browser** example with custom chrome and utilities.
* [x] **Deploy & ops artifacts**: Dockerfile, docker-compose, `vercel.json`, PWA-related Vite plugins, Crowdin/locale scripts.
* [x] **Quality gates**: `test:typecheck`, `test:code`, `test:app`, coverage/UI test options documented in root `package.json`.

## Future Roadmap / Backlog

* [ ] **Memory Bank hardening**: Single source of truth for patterns/tech context; link from README if required by course.
* [ ] **Optional product phases** (if in scope for maintainers): deeper **collab/ops** hardening, **performance** passes on large scenes (known hotspot notes in `element`/`frame`), **accessibility** audits, expanded **E2E** beyond Vitest unit coverage.
* [ ] **Examples**: Broader **embed** recipes (framework matrix) if needed for integrators.
* [ ] **Cleanup**: Reduce **TODO/FIXME** backlog in core packages where high-impact (see Known Issues).

## Known Issues & Tech Debt

* **`yarn.lock` noise**: Possible **registry URL** normalization (`yarnpkg` ↔ `npmjs`) inflating diffs — verify before commit (**activeContext**).
* **`packages/common`**: `colors.ts` **FIXME** — circular dependency blocks moving shared helper to `utils`.
* **`packages/element`**: **`store.ts`** — multiple **TODOs** on mutation/snapshot semantics and call-site safety; **`frame.ts`** — performance bottleneck called out for large scenes; **`elbowArrow.ts`** — cleanup when `Scene.getScene()` is removed.
* **`packages/excalidraw`**: **`data/library.ts`** — Jotai scoping **TODO**; **`actionFinalize.tsx`** — undo/redo capture semantics (**#7348**); **`renderer/interactiveScene.ts`** — multiplayer selected group IDs; **`fonts/Fonts.ts`** — custom font numeric model **TODO**; **`utils/export` tests** — **FIXME** SVG filtering of deleted elements.
* **`packages/math`**: **`point.ts`** — tuple migration **TODOs** (API cleanup).
* **`excalidraw-app`**: **`collab/Collab.tsx`** — `ImportedDataState` typing noted as abused.
* **Tests skipped / gaps**: **`packages/element/tests/resize.test.tsx`** — cases disabled pending text flip/resize fixes; **`flip.test.tsx`** — bounding-box **TODOs** for curved elements.
* **Examples**: **`with-script-in-browser/ExampleApp.tsx`** — “TODO fix type”.

---

*Last Updated: 2026-03-26*  
*Verified against source: **Yes** — `excalidraw-app/`, `packages/excalidraw/`, `packages/{common,element,math,utils}/src/`, `examples/` (no repo-root `src/`)*
