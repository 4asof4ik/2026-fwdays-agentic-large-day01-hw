# Product Requirements Document (PRD) — Reverse Engineered

This document describes the **Excalidraw** product as implemented in this monorepo. Source layout differs from a flat `src/` tree: UI and behavior live primarily under `packages/excalidraw/components/`, `packages/excalidraw/actions/`, and the hosted shell under `excalidraw-app/`. Requirements below are **evidence-based**; each capability is traceable to those locations unless a different path is cited.

---

## 1. Executive Summary

- **Product name**: **excalidraw-monorepo** (workspace root in `package.json`); the end-user application package is **excalidraw-app** (`excalidraw-app/package.json`). The editor library is exposed as **`@excalidraw/excalidraw`** (`packages/excalidraw/`).
- **One-line positioning**: A **browser-based virtual whiteboard** that renders diagrams with a **hand-drawn (sketch) aesthetic**, supports **infinite pan/zoom**, **structured diagramming** (frames, arrows, text, media embeds), **import/export**, optional **real-time collaboration** with **encrypted scene sync**, and **progressive-web-app** delivery for offline-friendly use.
- **Vision (the “why”)**: Lower the friction between **thinking in rough sketches** and **sharing clear diagrams**—without forcing users into rigid vector tools. The product optimizes for **speed**, **readability of informal drawings**, **privacy-aware collaboration** (client-side encryption for shared payloads where implemented), and **embedding** in websites and workflows.

---

## 2. Target Audience

| Persona | Primary jobs-to-be-done (inferred from shipped features) |
| -------- | -------------------------------------------------------- |
| **Software / systems engineers** | Whiteboard architecture (boxes, arrows, frames), paste structured inputs (e.g. charts), export to **PNG/SVG** or **JSON**, share **collaboration links**, use **Mermaid → canvas** and **text → diagram** flows (`TTDDialog`, `PasteChartDialog`, `ImageExportDialog`, `JSONExportDialog`). |
| **Product managers & facilitators** | Fast sketching in meetings, **laser pointer** and **live collaboration** presence, **follow mode**, **view mode** for read-only review, **Zen mode** for minimal chrome (`LaserPointerButton`, `UserList`, `FollowMode`, collaboration stack under `excalidraw-app/collab/`). |
| **Designers & UX practitioners** | **Wireframe-level** shapes, **images**, **text** with font controls, **color picker** with **eye dropper**, alignment/distribution, grouping, z-order, **library** of reusable components (`LibraryMenu`, `PublishLibrary`, property actions in `actionProperties.tsx`). |
| **Educators & content authors** | **Embeddable** media elements, **hyperlinks** on objects, **hand-drawn** style for approachable visuals, **export** for slides/docs (`actionEmbeddable.ts`, `hyperlink/`, `actionLink.tsx`). |
| **Integrators & host applications** | Consume the editor as a **React package** with configurable **UI options** (e.g. which canvas actions appear), custom render slots (`LayerUI.tsx` props such as `renderTopLeftUI`, `UIOptions` gating of export entries). |

---

## 3. Functional Requirements (Features)

### 3.1 Core canvas engine

- **Dual-layer HTML canvas rendering**: A **static** canvas draws the bulk of the scene; an **interactive** canvas handles selection, handles, and transient UI. Implemented in `packages/excalidraw/components/canvases/StaticCanvas.tsx` and `InteractiveCanvas.tsx`, driven by `renderStaticScene` / `renderInteractiveScene`.
- **Rough.js “hand-drawn” stroke rendering**: The editor constructs a **RoughCanvas** (`rough.canvas`) alongside the bitmap canvas (`packages/excalidraw/components/App.tsx` constructor), producing deliberately imperfect strokes aligned with the product’s sketch aesthetic.
- **Pan, zoom, and scroll**: Zoom actions (`actionZoomIn`, `actionZoomOut`, `actionResetZoom`, `actionZoomToFit`, `actionZoomToFitSelection`, `actionZoomToFitSelectionInViewport` in `actionCanvas.tsx`) and scroll/wheel behavior are central to the **infinite canvas** experience.
- **Scene model & history**: Elements live in a **scene store** with undo/redo (`History`, `Store` integration in `App.tsx`; `createUndoAction` / `createRedoAction` in `actionHistory.tsx`).
- **View modes**: **Zen mode** (`actionToggleZenMode.tsx`), **grid overlay** (`actionToggleGridMode.tsx`), **view-only mode** (`actionToggleViewMode.tsx`), and **stats overlay** for numeric editing (`actionToggleStats.tsx`, `Stats/`).
- **Theme & canvas background**: Toggle **light/dark** (`actionToggleTheme` in `actionCanvas.tsx`); change **view background color** (`actionChangeViewBackgroundColor`). Main menu exposes **canvas background** and theme (`DefaultItems.tsx`).
- **Render throttling hook**: When enabled, rendering can be **throttled to animation-frame boundaries** via `withBatchedUpdatesThrottled` / `isRenderThrottlingEnabled` (`packages/excalidraw/reactUtils.ts`). The hosted app sets `window.EXCALIDRAW_THROTTLE_RENDER = true` (`excalidraw-app/App.tsx`).

### 3.2 Tools & elements

Toolbar tools are enumerated in `packages/excalidraw/components/shapes.tsx` (`SHAPES`):

- **Hand tool** (pan): `hand`, shortcut **H**.
- **Selection**: rectangle selection; optional **lasso** selection mode (`getToolbarTools` swaps first tool to `lasso` when `preferredSelectionTool.type === "lasso"`).
- **Basic shapes**: **rectangle**, **diamond**, **ellipse** with fill/stroke styling.
- **Connectors**: **arrow** and **line** tools; advanced linear editing (**polygon** toggle, linear point editor) via `actionToggleLinearEditor`, `actionTogglePolygon` (`actionLinearEditor.tsx`).
- **Freehand**: **freedraw** (pencil), **eraser**, optional **pen mode** UI (`PenModeButton.tsx`).
- **Text**: in-canvas **WYSIWYG** text editing (`wysiwyg/textWysiwyg.tsx`); font family/size, alignment, vertical alignment, auto-resize (`actionProperties.tsx`, `actionTextAutoResize.ts`, `actionBoundText.tsx`).
- **Images**: placement and decoding with **size and byte limits** enforced in `App.tsx` (e.g. `MAX_ALLOWED_FILE_BYTES`, `DEFAULT_MAX_IMAGE_WIDTH_OR_HEIGHT` from `@excalidraw/common`).
- **Laser pointer** (presentation-style): `laser` tool entry in `SHAPES` and `LaserPointerButton.tsx`.
- **Frames & organization**: Frame tool and frame management actions (`actionFrame.ts`): select all in frame, remove from frame, wrap selection in frame, update frame rendering, set frame as active tool.
- **Embeddables / iframes**: **Embeddable** tool (`actionEmbeddable.ts`) with validation helpers (`embeddableURLValidator`, iframe message handling for **YouTube/Vimeo** in `App.tsx`).
- **Flowchart assistance**: Flowchart-related element logic is present in the element layer (`FlowChartCreator`, `FlowChartNavigator` imports in `App.tsx`).
- **Charts from clipboard**: Dedicated **paste chart** flow (`PasteChartDialog.tsx`) for importing chart data into the canvas.
- **Crop editor**: Toggle image crop UI (`actionToggleCropEditor` in `actionCropEditor.tsx`).
- **Shape switching**: Toggle alternate shape behaviors where applicable (`actionToggleShapeSwitch.tsx`).
- **Hyperlinks & element links**: Link action (`actionLink.tsx`), **copy/link-to-element** flows (`actionElementLink.ts`), dedicated dialogs (`ElementLinkDialog.tsx`, `ShareableLinkDialog.tsx` in app wiring).
- **Locking**: Per-element lock and unlock-all (`actionElementLock.ts`).
- **Arrow binding & snapping**: Toggle **arrow binding** to shapes (`actionToggleArrowBinding.tsx`), **midpoint snapping** (`actionToggleMidpointSnapping.tsx`), global **object snap** (`actionToggleObjectsSnapMode.tsx`).

### 3.3 Styling & property editing

Implemented mainly in `actionProperties.tsx` and surfaced through `SelectedShapeActions` / `PropertiesPopover` (see `LayerUI.tsx`, `Actions.tsx`):

- Stroke and fill **colors**, **stroke width**, **opacity**, **fill style**, **stroke style**, **sloppiness/roughness**, **roundness**.
- **Arrowheads**, **arrow type** (including specialized types such as **elbow** arrows in the element package), consolidated **arrow properties** editor.
- **Font** family (with `FontPicker/`), size, increase/decrease shortcuts, text alignment, vertical alignment.
- **Style clipboard**: **copy/paste styles** (`actionStyles.ts`).
- **Color picking**: Full **ColorPicker** component tree under `components/ColorPicker/` plus **EyeDropper** (`EyeDropper.tsx`).

### 3.4 Selection editing & layout operations

- **Clipboard**: Copy, cut, paste, **copy as PNG**, **copy as SVG**, **copy text** from selection (`actionClipboard.tsx`).
- **Duplicate & delete**: `actionDuplicateSelection.tsx`, `actionDeleteSelected.tsx`.
- **Z-order**: Send backward/forward, to back/front (`actionZindex.tsx`).
- **Align & distribute**: Six align actions + horizontal/vertical distribute (`actionAlign.tsx`, `actionDistribute.tsx`).
- **Flip**: Horizontal/vertical (`actionFlip.ts`).
- **Group / ungroup**: `actionGroup.tsx`.
- **Select all**: `actionSelectAll.ts`.
- **Text-container relationships**: Bind/unbind text, wrap text in container (`actionBoundText.tsx`).

### 3.5 Library & reusable assets

- **In-editor library UI**: `LibraryMenu.tsx`, `LibraryMenuItems.tsx`, browse/insert controls (`LibraryMenuBrowseButton.tsx`, `LibraryMenuControlButtons.tsx`).
- **Library persistence**: Library data types and atoms in `data/library.ts`; hosted app migrates to **IndexedDB** (`LibraryIndexedDBAdapter`, keys like `IDB_LIBRARY` in `excalidraw-app/app_constants.ts`).
- **Publish library**: Authoring workflow to publish a library with preview generation (`PublishLibrary.tsx`).
- **Add selection to library**: `actionAddToLibrary.ts`.

### 3.6 Import, export & file interchange

- **JSON / scene files**: Load and save via `actionLoadScene`, `actionSaveToActiveFile`, `actionSaveFileToDisk` (`actionExport.tsx`) with **overwrite confirmation** patterns (`OverwriteConfirm/`).
- **JSON export dialog**: `JSONExportDialog.tsx` (gated by `UIOptions.canvasActions.export` in `LayerUI.tsx`).
- **Raster export**: `ImageExportDialog.tsx` supports **scale** presets (`EXPORT_SCALES`), **background** toggle, **embed scene** metadata (`actionChangeExportEmbedScene`), **export with dark mode**, selection vs whole scene, integration hook `onExportImage`.
- **SVG export path**: Used inside publish/preview flows (`PublishLibrary.tsx` imports `exportToSvg` from `@excalidraw/utils/export`).
- **Encoding & optional encryption**: `data/encode.ts` compresses and may attach **AES-GCM** encryption metadata; `data/encryption.ts` implements **Web Crypto** encrypt/decrypt helpers used by export/share pipelines.

### 3.7 Command palette, search & help

- **Command palette**: `CommandPalette/CommandPalette.tsx` executes registered **actions** with source `"commandPalette"`; opened from main menu (`DefaultItems.tsx`) and keyboard shortcuts (`shortcuts.ts` integration).
- **Search menu**: `SearchMenu.tsx` + `actionToggleSearchMenu.ts` for in-canvas element discovery.
- **Shortcuts / help dialog**: `actionShortcuts` (`actionMenu.tsx`) opens `HelpDialog.tsx` with documented shortcuts (includes command palette hints).

### 3.8 Text-to-diagram (TTD) & Mermaid

- **TTD dialog shell**: `TTDDialog/TTDDialog.tsx` opens when `appState.openDialog.name === "ttd"`, with tabs for **text-to-diagram** and **Mermaid** (`TTDDialogTabs`, `TextToDiagram.tsx`, `MermaidToExcalidraw.tsx`).
- **Lazy-loaded conversion**: Dynamic `import("@excalidraw/mermaid-to-excalidraw")` in `TTDDialog.tsx`; Vite `manualChunks` isolates `mermaid-to-excalidraw` (`excalidraw-app/vite.config.mts`).
- **Code editor chunk**: CodeMirror/Lezer split into `codemirror.chunk` for lazy loading (`vite.config.mts`).
- **Streaming & chat utilities**: `TTDStreamFetch.ts`, chat hooks (`useChatManagement.ts`, `useTextGeneration.ts`, `TTDChatPanel.tsx`) support assisted diagram generation workflows.
- **Persistence key surface**: `IDB_TTD_CHATS` constant (`excalidraw-app/app_constants.ts`) indicates chat/history persistence integration in the hosted app.

### 3.9 Collaboration, sharing & presence (hosted app)

The open-source web app layers collaboration on top of the package:

- **Live collaboration entry points**: `LiveCollaborationTrigger.tsx` (package) wired in `excalidraw-app/App.tsx` alongside `Collab` (`excalidraw-app/collab/Collab.tsx`, `Portal.tsx`).
- **Realtime transport**: `socket.io-client` dependency (`excalidraw-app/package.json`); websocket subtypes for **scene init/update**, **pointer/laser**, **idle state**, **visible bounds** (`excalidraw-app/data/index.ts`).
- **Encrypted room payloads**: Collaboration export path uses **compress/encrypt** helpers (`generateEncryptionKey`, `decryptData`, `compressData` in `excalidraw-app/data/index.ts`). **AES-GCM** implementation: `packages/excalidraw/data/encryption.ts`. User-facing copy enforces a **22-character key** expectation for live collaboration (`packages/excalidraw/locales/en.json` keys such as `invalidEncryptionKey`).
- **Backend blob endpoints**: Configurable `VITE_APP_BACKEND_V2_GET_URL` / `POST_URL` (`excalidraw-app/data/index.ts`).
- **Firebase file storage**: `firebase` dependency and `saveFilesToFirebase` / `loadFilesFromFirebase` (`excalidraw-app/data/firebase.ts`, storage prefixes in `app_constants.ts`).
- **Share dialogs**: `ShareableLinkDialog.tsx`, app-level `ShareDialog` (`excalidraw-app/share/ShareDialog.tsx`), **export to backend** helpers (`exportToBackend`, `importFromBackend` from `excalidraw-app/data`).
- **Presence UI**: `UserList.tsx`, **follow mode** (`FollowMode/FollowMode.tsx`), collaborator navigation (`actionGoToCollaborator` in `actionNavigate.tsx`).
- **Cursor sync timing**: `CURSOR_SYNC_TIMEOUT = 33` ms (~**30 Hz**) in `excalidraw-app/app_constants.ts` documents the expected pointer update cadence.

### 3.10 Application shell, onboarding & platform UX

- **Welcome screen**: `welcome-screen/WelcomeScreen.tsx` and hints (`WelcomeScreen.Hints.tsx`); app-specific wrapper `excalidraw-app/components/AppWelcomeScreen.tsx`.
- **Main menu & footer**: `main-menu/MainMenu.tsx`, `footer/Footer.tsx`; app overrides via `AppMainMenu`, `AppFooter`.
- **Mobile adaptation**: `MobileMenu.tsx`, `MobileToolBar.tsx`, editor interface / form factor detection via `@excalidraw/common` (`EditorInterface` usage in `LayerUI.tsx`, `ActionManager`).
- **Sidebar layout**: `Sidebar/`, `DefaultSidebar.tsx`, docked state atom (`isSidebarDockedAtom`); **fixed side containers** (`FixedSideContainer.tsx`) structure the **island**-based UI (`Island.tsx`).
- **Internationalization**: `i18n` integration with locale chunks cached by the service worker (`vite.config.mts` `locales/` runtime caching).
- **Error & confirm dialogs**: `ErrorDialog.tsx`, `ConfirmDialog.tsx`, `ActiveConfirmDialog.tsx`, collaboration error surface (`excalidraw-app/collab/CollabError.tsx`).
- **Telemetry (hosted)**: `@sentry/browser` in `excalidraw-app` (`package.json`, `excalidraw-app/sentry` import from `index.tsx`).
- **Analytics hooks**: `trackEvent` used across actions and UI (`packages/excalidraw/analytics.ts`, e.g. `ActionManager`).

### 3.11 Excalidraw+ & extensibility hooks (hosted)

The app repository includes integration points for **Excalidraw+** and related flows (promotional and export paths), e.g. `ExportToExcalidrawPlus`, `ExcalidrawPlusPromoBanner`, `ExcalidrawPlusIframeExport` under `excalidraw-app/`. These are **product surface areas** in this codebase even if the commercial service is external.

### 3.12 Structured chart parsing (bar, line, radar)

The editor ships a **charts** submodule under `packages/excalidraw/charts/` (`charts.bar.ts`, `charts.line.ts`, `charts.radar.ts`, `charts.parse.ts`, shared types in `charts.types.ts`). This supports turning structured chart descriptions into drawable elements—aligned with **PasteChartDialog** and data-driven diagramming workflows.

### 3.13 Context menu & secondary UI affordances

- **Context menu**: `ContextMenu.tsx` renders action lists with **predicate filtering** (items hidden when `action.predicate` fails), shortcut labels via `getShortcutFromShortcutName`, and viewport-aware positioning through `Popover` (`fitInViewport`, offsets from `appState`).
- **Default right sidebar**: `DefaultSidebar.tsx` composes **`LibraryMenu`** and **`SearchMenu`** as tabs (`LIBRARY_SIDEBAR_TAB`, `CANVAS_SEARCH_TAB` from `@excalidraw/common`), with docking behavior and a **force-docked** rule while canvas search is active.
- **Tunnels / portal layout**: `TunnelsContext` and tunnel components in `LayerUI.tsx` allow toolbar islands to host triggers that render in coordinated portal locations—supporting a **flexible desktop layout** without prop-drilling every control.
- **Hint overlay**: `HintViewer.tsx` surfaces contextual tips during editing.
- **Convert element type**: `ConvertElementTypePopup.tsx` supports changing an element’s type in place where the model allows.
- **Unlock popup**: `UnlockPopup.tsx` explains or routes unlocking when interacting with locked content.
- **Collaboration triggers (package-level)**: `live-collaboration/LiveCollaborationTrigger.tsx` is the embeddable entry affordance; the **hosted app** supplies live room wiring.

### 3.14 Scene reconciliation & collaboration data model

- **Remote merge helpers**: `reconcileElements` is imported in `excalidraw-app/App.tsx` from `@excalidraw/excalidraw` to merge **remote** element lists with local state during collaboration.
- **Syncable element filter**: `isSyncableElement` / `getSyncableElements` in `excalidraw-app/data/index.ts` omit **invisibly small** shapes and apply a **tombstone window** for recently deleted elements (`DELETED_ELEMENT_TIMEOUT` in `app_constants.ts`), keeping sync payloads meaningful without retaining deleted noise indefinitely.

### 3.15 Debugging & developer overlays

- **Debug canvas**: `excalidraw-app/components/DebugCanvas.tsx` (with `debugRenderer`, `isVisualDebuggerEnabled`) exposes optional **render debugging** for contributors; state can be loaded from saved debug preferences (`loadSavedDebugState`).

### 3.16 AI-related shell (hosted)

- **AI components entry**: `excalidraw-app/components/AI.tsx` exports `AIComponents`, rendered when `excalidrawAPI` is ready (`excalidraw-app/App.tsx`), indicating an **AI-assisted** product surface in the hosted application (exact feature flags and endpoints are configured outside this PRD’s scope but the integration point is present in source).

---

## 4. User Flows

### 4.1 Primary creation flow

1. User lands on the app (PWA or tab) and sees **welcome** content or a blank canvas (`WelcomeScreen`, `InitializeApp.tsx`).
2. User selects a **tool** from the floating toolbar (`ShapesSwitcher` in `LayerUI.tsx` / `shapes.tsx`).
3. User draws shapes, connectors, text, or inserts images/embeds on the **infinite canvas** with pan/zoom as needed.
4. User refines via **selection**, **properties**, **alignment**, **grouping**, and **history** (undo/redo).
5. User saves work to **local file**, **local storage**, or continues into **collaboration** (app-specific).

### 4.2 Export / share flow

1. Open **main menu** → **Export** or **Save as image** (`DefaultMainMenu` in `LayerUI.tsx`).
2. Configure **PNG/SVG** options in `ImageExportDialog.tsx` (scale, background, embed scene, selection scope).
3. Optionally export **JSON** via `JSONExportDialog.tsx` when enabled.
4. For collaboration: start live session, obtain **share link**, with optional **end-to-end encryption** of uploaded ciphertext (`excalidraw-app/data/index.ts`, `encryption.ts`).

### 4.3 Library reuse flow

1. Open **library** sidebar (`LibraryMenu`).
2. Insert items onto canvas or **add** current selection via `actionAddToLibrary`.
3. For sharing collections, use **Publish library** (`PublishLibrary.tsx`) to generate previews and structured export.

### 4.4 Text / Mermaid → diagram flow

1. Open **TTD** dialog (`TTDDialogTrigger` from app, dialog in `TTDDialog.tsx`).
2. Choose **text-to-diagram** or **Mermaid** tab.
3. Edit in **CodeMirror**-backed editor; submit to generate elements; iterate with optional **chat** panels.

### 4.5 Discovery / power-user flow

1. Open **command palette** (menu or shortcut) to fuzzy-run actions (`CommandPalette.tsx`).
2. Use **search menu** to jump to elements (`SearchMenu.tsx`).
3. Toggle **stats** or **view mode** for precision edits or presentation (`Stats/`, `actionToggleViewMode`).

---

## 5. Non-Functional Requirements

### 5.1 Performance

- **Animation-frame-aligned updates**: `requestAnimationFrame` appears in hot paths (e.g. `ToolButton.tsx`, `textWysiwyg.tsx`, `App.tsx`, `CommandPalette.tsx`) for smoother UI.
- **Throttled React updates**: `lodash.throttle` for expensive handlers (e.g. image refresh scheduling in `App.tsx`, stats dimension updates in `Stats/index.tsx`).
- **Collaboration cadence**: Pointer synchronization tuned to ~**30 updates/sec** (`CURSOR_SYNC_TIMEOUT` in `excalidraw-app/app_constants.ts`).
- **Large scene safeguards**: Export pipeline can fail when the canvas is **too large** (`canvasError.canvasTooBig` in `packages/excalidraw/data/index.ts` and `ImageExportDialog.tsx` preview error surface), signaling a practical **performance/size constraint** rather than an unbounded guarantee.

### 5.2 UX / UI

- **Minimal chrome with “islands”**: Floating controls grouped in `Island` components; **Zen mode** hides distractions (`actionToggleZenMode.tsx`).
- **Keyboard-first**: Centralized shortcut resolution with conflict detection (`ActionManager.handleKeyDown`).
- **Accessible menu patterns**: ARIA labels on core menu items (`DefaultItems.tsx` uses `aria-label` extensively).
- **Hand-drawn aesthetic**: Rough.js rendering (`App.tsx` rough canvas) differentiates the product from rigid diagram suites.

### 5.3 Security & privacy

- **Client-side encryption**: AES-GCM via **Web Crypto** (`packages/excalidraw/data/encryption.ts`, `encode.ts` metadata).
- **Collaboration privacy messaging**: Locale strings describe **end-to-end encrypted** sessions and uploads (`packages/excalidraw/locales/en.json`).
- **Cookie-gated Plus detection**: `AUTH_STATE_COOKIE` constant (`excalidraw-app/app_constants.ts`) for signed-in state checks—relevant to integrated flows.

### 5.4 Reliability & observability

- **Sentry** integration in the hosted bundle (`excalidraw-app/package.json`, `index.tsx` import).
- **Error boundaries**: `TopErrorBoundary` in `excalidraw-app/components/TopErrorBoundary.tsx`.

---

## 6. Technical Constraints

### 6.1 Platform & runtime

- **Modern browsers only**: `excalidraw-app/package.json` **browserslist** excludes legacy IE, very old Safari/Chrome/Edge, and sets production targets to broadly evergreen browsers.
- **Node**: `engines.node >= 18` at root and app (`package.json`, `excalidraw-app/package.json`).
- **TypeScript**: `strict` mode enabled (`tsconfig.json`); `module`/`target` **ESNext** with DOM libs—assumes modern JS baseline in browsers.
- **React 19** stack in the hosted app (`react`, `react-dom` versions in `excalidraw-app/package.json`).

### 6.2 Build & delivery

- **Vite 5** bundler (`package.json` devDependencies; `excalidraw-app` scripts use `vite`).
- **PWA**: `vite-plugin-pwa` registers a service worker (`excalidraw-app/vite.config.mts`, `virtual:pwa-register` in `excalidraw-app/index.tsx`). Precache excludes heavy lazy chunks (locales, CodeMirror, fonts strategy documented inline in config).
- **Monorepo workspaces**: `yarn` workspaces spanning `excalidraw-app`, `packages/*`, `examples/*` (root `package.json`).

### 6.3 Networking & storage

- **Collaboration requires backend services**: WebSocket server implied by `socket.io-client`; HTTP endpoints for encrypted blobs (`BACKEND_V2_*` URLs); **Firebase** for shared file storage in the hosted configuration.
- **Local persistence**: `localStorage` keys (`STORAGE_KEYS` in `excalidraw-app/app_constants.ts`), **IndexedDB** adapters for library (`LibraryIndexedDBAdapter` referenced from `excalidraw-app/App.tsx`).
- **File size limits**: `FILE_UPLOAD_MAX_BYTES` (**4 MiB**) aligned with editor image limits (`excalidraw-app/app_constants.ts`, `MAX_ALLOWED_FILE_BYTES` usage in `App.tsx`).

### 6.4 Cryptographic constraints

- **Web Crypto dependency**: Encryption helpers assume `window.crypto.subtle` availability (`packages/excalidraw/data/encryption.ts`).
- **Key format constraints**: Collaboration UX expects encryption key length constraints (see `invalidEncryptionKey` copy, **22 characters**, in `packages/excalidraw/locales/en.json`).

### 6.5 Embedding & security surface

- **Iframe embeds**: `iframe` allow lists and `postMessage` handlers for specific video providers (`App.tsx` YouTube/Vimeo branches) constrain cross-origin messaging.

---

## 7. Traceability Note (Methodology)

- **Actions** map to user-visible commands: see `packages/excalidraw/actions/*.ts(x)` and the aggregated `actions` array consumed in `App.tsx` (`import { actions } from "../actions/register"`).
- **UI composition** is orchestrated in `packages/excalidraw/components/LayerUI.tsx` (menus, dialogs, sidebars, toolbars).
- **Hosted-app-only** behaviors (collab backends, PWA registration, Sentry, Firebase) appear under `excalidraw-app/` and are not implied for minimal `@excalidraw/excalidraw` embeds unless integrators wire equivalent props/services.

---

## Appendix A — Feature ↔ primary source map (selected)

The following table is a **non-exhaustive** index for reviewers; the editor surface is larger than any single list.

| Capability | Primary evidence (paths under repo root) |
| ---------- | ---------------------------------------- |
| Tool palette & shortcuts | `packages/excalidraw/components/shapes.tsx`, `Actions.tsx`, `MobileToolBar.tsx` |
| Canvas static/interactive render | `packages/excalidraw/components/canvases/StaticCanvas.tsx`, `InteractiveCanvas.tsx`, `renderer/staticScene`, `renderer/interactiveScene` |
| Rough sketch rendering | `packages/excalidraw/components/App.tsx` (`rough.canvas`) |
| Undo / redo | `packages/excalidraw/actions/actionHistory.tsx`, `App.tsx` registration |
| Zoom & pan | `packages/excalidraw/actions/actionCanvas.tsx` |
| Clipboard & image/svg copy | `packages/excalidraw/actions/actionClipboard.tsx`, `packages/excalidraw/clipboard.ts` |
| Style & property panels | `packages/excalidraw/actions/actionProperties.tsx`, `ColorPicker/`, `FontPicker/` |
| Align, distribute, flip, z-order | `actionAlign.tsx`, `actionDistribute.tsx`, `actionFlip.ts`, `actionZindex.tsx` |
| Grouping | `actionGroup.tsx` |
| Frames | `actionFrame.ts` |
| Embeds | `actionEmbeddable.ts`, iframe handlers in `App.tsx` |
| Text ↔ container | `actionBoundText.tsx`, `wysiwyg/textWysiwyg.tsx` |
| Linear editing | `actionLinearEditor.tsx` |
| Crop | `actionCropEditor.tsx` |
| Locking | `actionElementLock.ts` |
| Hyperlinks & deep links | `actionLink.tsx`, `actionElementLink.ts`, `hyperlink/`, `ElementLinkDialog.tsx` |
| Library | `LibraryMenu.tsx`, `data/library.ts`, `actionAddToLibrary.ts`, `PublishLibrary.tsx` |
| Publish library previews | `PublishLibrary.tsx` (`exportToCanvas`, `exportToSvg`) |
| JSON / file IO | `actionExport.tsx`, `JSONExportDialog.tsx`, `data/json` |
| Raster export | `ImageExportDialog.tsx`, `@excalidraw/utils/export` |
| Encode / encrypt | `packages/excalidraw/data/encode.ts`, `packages/excalidraw/data/encryption.ts` |
| Command palette | `CommandPalette/CommandPalette.tsx`, `defaultCommandPaletteItems.ts` |
| Search | `SearchMenu.tsx`, `actionToggleSearchMenu.ts` |
| Help / shortcuts | `HelpDialog.tsx`, `actionMenu.tsx` |
| Stats / numeric edits | `Stats/index.tsx`, `actionToggleStats.tsx` |
| TTD / Mermaid | `TTDDialog/*`, `@excalidraw/mermaid-to-excalidraw` dynamic import |
| TTD chat & streaming | `TTDDialog/Chat/*`, `TTDStreamFetch.ts`, `useChatManagement.ts` |
| Welcome | `welcome-screen/*`, `excalidraw-app/components/AppWelcomeScreen.tsx` |
| Main menu defaults | `main-menu/DefaultItems.tsx`, `LayerUI.tsx` (`DefaultMainMenu`) |
| Context menu | `ContextMenu.tsx` |
| Default sidebar | `DefaultSidebar.tsx`, `Sidebar/Sidebar.tsx` |
| Collaboration UI | `UserList.tsx`, `FollowMode/FollowMode.tsx`, `LiveCollaborationTrigger.tsx` |
| Collab transport & crypto | `excalidraw-app/collab/*`, `excalidraw-app/data/index.ts` |
| Firebase files | `excalidraw-app/data/firebase.ts` |
| Local persistence | `excalidraw-app/data/localStorage.ts`, `LocalData.ts`, `app_constants.ts` keys |
| Tab sync | `excalidraw-app/data/tabSync.ts`, `SYNC_BROWSER_TABS_TIMEOUT` |
| PWA | `excalidraw-app/vite.config.mts` (`VitePWA`), `excalidraw-app/index.tsx` (`registerSW`) |
| Error reporting | `excalidraw-app/sentry`, `@sentry/browser` dependency |
| Charts parsing | `packages/excalidraw/charts/*`, `PasteChartDialog.tsx` |
| Flowchart helpers | `FlowChartCreator`, `FlowChartNavigator` (imported from `@excalidraw/element` in `App.tsx`) |
| Custom stats slot | `excalidraw-app/CustomStats.tsx` |
| AI-assisted UI shell | `excalidraw-app/components/AI.tsx` (`AIComponents` in `App.tsx`) |
| Language detection | `excalidraw-app/app-language/language-detector.ts`, locale bundles under `packages/excalidraw/locales/` |

---

## Appendix B — Engineering quality gates (from root tooling)

These are **build/test constraints** for the codebase, not end-user features, but they shape what can ship:

- **Typecheck**: `tsc` with `strict` settings (`tsconfig.json`, script `test:typecheck` in root `package.json`).
- **Unit tests**: `vitest` with canvas mocking (`vitest-canvas-mock` in root devDependencies; script `test:app`).
- **Lint**: ESLint with zero-warning policy in CI script (`test:code` uses `--max-warnings=0`).
- **Formatting**: Prettier over configured globs (`fix:other`, `test:other`).
- **Path aliases**: `@excalidraw/*` packages resolved in `tsconfig.json` for consistent imports across workspaces.

---

## Appendix C — UX strategy inferred from layout code

- **Floating “islands”** (`Island.tsx`, `LayerUI.scss`, `Toolbar.scss`) reduce permanent chrome and keep focus on the canvas—consistent with a **distraction-free whiteboard**.
- **Responsive density**: `LayerUI.tsx` switches spacing tokens when `stylesPanelMode === "compact"` (derived from editor interface), tightening controls on smaller viewports.
- **Progressive disclosure**: Advanced capabilities (export options, encryption details, library publish) live in **dialogs** rather than the always-visible toolbar.
- **Dual sidebar model**: Library and search are **tabs** inside one dockable sidebar (`DefaultSidebar.tsx`), so users switch between **reuse** and **navigation** without two competing panels.
- **Collaboration affordances** are grouped near presence (`UserList`) and triggers, separate from file operations in the main menu—mirroring a **session-based** mental model.

---

*Verified against source: Yes*
