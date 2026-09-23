# cyber-wiki-front

Frontend for **CyberWiki** — a Git-backed wiki / documentation tool that turns
a Bitbucket Server / GitHub repository into a browsable, editable,
review-friendly knowledge base. Pairs with [`cyber-wiki-back`](../cyber-wiki-back)
(Django) on `http://localhost:8888`.

Built on **React 19 + Vite 6 + TypeScript 5** and the in-house
[`@cyberfabric/react`](https://github.com/constructorfabric/frontx) framework
(event-driven Flux: Action → Event → Effect → Slice).

## Quick start

Prereqs: Node `>=25.1.0`, npm `>=10`. Backend (`cyber-wiki-back`) running on
`localhost:8888`.

```bash
npm install
npm run dev           # → http://localhost:5173
```

Vite proxies `/api/*` to the backend (`vite.config.ts:server.proxy`). Sign in
on the LoginPage with credentials provisioned by the backend.

## Scripts

| Script | What it does |
|---|---|
| `npm run dev` | Vite dev server with HMR. |
| `npm run dev:all` | Dev server + auxiliary watchers. |
| `npm run build` | Production build → `dist/`. |
| `npm run preview` | Preview a built bundle locally. |
| `npm run lint` | ESLint, treats warnings as errors. |
| `npm run lint:check-disables` | Fails when new `eslint-disable` directives appear. |
| `npm run type-check` | `tsc --noEmit`. |
| `npm run arch:check` | Architecture rules + type-check (run before pushing). |
| `npm run arch:deps` | Dependency-cruiser graph check. |
| `npm test` | Vitest, single run. |
| `npm run test:watch` | Vitest watch mode. |

## Project layout

Everything user-facing lives under `src/app/` (rule:
`.ai/rules/cyberwiki-front.md`):

```
src/app/
├── pages/           # Top-level routes (Dashboard, Spaces, SpaceView, ...)
├── api/             # API services (extend BaseApiService) + types + plugins
├── actions/         # Pure event-emitters (no async, no state read)
├── effects/         # eventBus.on(...) handlers — call API, dispatch slices
├── events/          # EventPayloadMap augmentation (typed event keys)
├── components/      # Domain-grouped UI (file/, space/, enrichments/, ...)
│   └── primitives/  # shadcn-style primitives (Modal, Avatar, CodeEditor, ...)
├── layout/          # App shell — Header / Sidebar / Layout
├── themes/          # Theme tokens
├── icons/           # Icon components
├── lib/             # Pure helpers — formatDate, notify, httpStatus, i18n, ...
└── locales/         # i18n JSON (en.json is source of truth)
```

### Pages

`Dashboard`, `Spaces`, `SpaceView` (file tree + viewer + enrichments panel),
`SpaceConfiguration`, `Profile`, `Tokens`, `Comments`, `Changes`, `PRs`,
`Logs` (debug-mode only), `Login`.

### Component domains

- `file/` — FileViewer, FileTree, FileRenderer, MdRenderer, FileViewerHeader,
  CreateFileModal, BlameView, FileStatus.
- `file-mapping/` — file-path → display-name configuration UI.
- `space/` — Space CRUD, SpaceTree.
- `changes/` — DraftDiffView, PRBanner, RecentCommitsPanel.
- `enrichments/` — Comments / Diffs / PRs panel, ConflictResolution.
- `loading/` — SmartLoadingIndicator, ViewLoadingFallback.
- `primitives/` — Modal, ConfirmDialog, DropdownMenu, ContextMenu, Avatar,
  CodeBlock, **CodeEditor / CodeViewer** (Monaco), Skeleton, Sonner, sidebar.

## Editor

- **Markdown** — WYSIWYG via Milkdown (`MdRenderer`); `Source` toggle shows
  raw markdown in a line-numbered table for per-line commenting.
- **Non-markdown** (TOML, JSON, YAML, code, …) — Monaco editor (`CodeEditor`)
  in edit mode, Monaco read-only viewer (`CodeViewer`) for read mode, with
  syntax highlighting, folding, glyph margins for comments, and decorations
  for unsaved-draft change markers. Languages outside Monaco's built-in set
  (TOML and friends) are registered in
  `components/primitives/monacoSetup.ts`.

## Architecture rules (one-pager)

The full ruleset is in `.ai/rules/cyberwiki-front.md`. Highlights:

- **Event-driven flow**: components call actions → actions emit events →
  effects handle events → slices update. Components never touch
  `apiRegistry` or `eventBus.emit` directly.
- **No literal-value unions** — define an enum (`HttpStatus`, `ModalSize`,
  `FileViewMode`, …) and use its members.
- **i18n** — every user-facing string goes through `t('key')` and lives in
  `src/app/locales/en.json`. Effects use `t()` from `@/app/lib/i18n` (not
  `useTranslation`). Listed files are lint-locked.
- **Dates** — render via `formatDate / formatDateTime / formatTime` from
  `@/app/lib/formatDate`. Inline `toLocale*String()` is banned.
- **Toasts / logs** — go through `@/app/lib/notify` (`notify.success / info /
  warn / error`). `console.*` is banned outside `lib/notify.ts` and
  `lib/performanceTracker.ts`.
- **Modals** — `<Modal>` from `components/primitives/Modal.tsx`. No
  hand-rolled `fixed inset-0 ... bg-black/...` overlays or `createPortal`
  outside primitives.
- **Styling** — Tailwind only, theme tokens via CSS variables, rem-based
  sizes (px allowed only for borders). Inline styles allowed only inside
  `components/primitives/`.
- **Page size budget** — `pages/*.tsx` are capped at 700 non-blank /
  non-comment lines (lint-enforced).

## Pre-push checklist

```bash
npm run arch:check         # ESLint + tsc
npm run lint:check-disables # zero new eslint-disable directives
npm test
```

## Where to read more

- `CLAUDE.md` (root) — pointer to the rules file used by AI assistants.
- `.ai/rules/cyberwiki-front.md` — full ruleset (SOLID/DRY checklist,
  file-placement rules, banned patterns, pre-diff checklist).
- `.ai/commands/review-and-commit.md` — review-and-commit slash command.
- Backend repo: `cyber-wiki-back`.
