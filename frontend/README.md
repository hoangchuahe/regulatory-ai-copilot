# Frontend — IA Bank Regulatory AI Copilot

Next.js (App Router) + TypeScript. Working rules: `CLAUDE.md` (this folder) and `../CLAUDE.md`.

## Getting started

```bash
corepack enable pnpm       # pnpm 9, once per machine
pnpm install
cp .env.example .env.local # optional: the default values are enough (mock mode)
pnpm dev                   # open the URL shown (3000 if the port is free)
```

Demo account: `marie.lefevre@iabank.fr` / `demo1234` (repeated under the form).

Node 22 recommended (`.nvmrc`). On Node 20 everything works, but Vitest and jsdom
fall outside their `engines` range: `jsdom` is deliberately pinned to 26.

> **Status 2026-10-29:** the demo account is no longer repeated under the sign-in form, and it
> only works against a backend seeded by `../scripts/local-dev/setup-backend.sh` (see "Data"
> below).

## Scripts

| Command | Role |
|---|---|
| `pnpm dev` | development server |
| `pnpm build` / `pnpm start` | production build and serving |
| `pnpm lint` | ESLint (must stay error-free) |
| `pnpm typecheck` | `tsc --noEmit` |
| `pnpm test` | Vitest (unit + component + API contract) |
| `pnpm screenshot` | Playwright screenshots in `screenshots/` — mandatory visual check |

The screenshot script signs in by itself with the demo account and walks through the screens
(FR + EN, tabs, highlight by link, procedure window). It **fails if a console error
appears**.

`pnpm screenshot` assumes `pnpm dev` is already running. If the port differs:
`SCREENSHOT_BASE_URL=http://localhost:3001 pnpm screenshot`.

## Data: two sources, one switch

Thư's backend exists (FastAPI, see `../backend/`) but exposes a data model different from the
contract and does not cover everything yet (see `../docs/backend-integration.md`). The frontend
talks to both at once, endpoint by endpoint:

- **Mock mode (default)** — `NEXT_PUBLIC_BACKEND_URL` empty in `.env.local`. All the requests of
  `../docs/api-contract.md` are served by **MSW** in the browser, from the fictional demo corpus
  (`src/lib/mocks/data/`) — the corpus hand-built for the client demo (9 KYC/AML
  gaps). Validation decisions are kept in `sessionStorage` for the life of the tab.
- **Real backend mode** — set `NEXT_PUBLIC_BACKEND_URL=http://localhost:8000` (or the URL of
  Thư's server) in `.env.local`. Automatic switch, endpoint by endpoint, in
  `src/lib/api/*.ts`: authentication, regulations/procedures, requirements and findings go to
  the real backend (`src/lib/api/backend/`, with its own adaptation layer and Zod schemas);
  dashboard (recomputed on the client from the real data), upload, human validation
  and history stay on MSW — the backend does not expose them.

> **Status 2026-10-29:** the MSW demo corpus was removed (commit `4f56230`), so mock mode now
> shows empty screens: run the backend locally (below) to see data. The backend URL variable is
> now `NEXT_PUBLIC_API_BASE_URL` (see `.env.example`); `NEXT_PUBLIC_BACKEND_URL` is ignored. The
> dashboard (`GET /api/dashboard/portfolio`), upload, human validation and the user list now use
> the real backend.

**Running Thư's backend locally** (no PostgreSQL needed — see
`../scripts/local-dev/README.md`):

```bash
../scripts/local-dev/setup-backend.sh   # once
../scripts/local-dev/run-backend.sh     # every session → http://localhost:8000
```

In both modes, responses are validated by Zod (`src/types/api.ts`): a response that does not match
the contract raises an `ApiContractError` and shows an explicit error state rather than
silently corrupting the screen — never a loosened schema to "let" a response through.

## Layout

```
src/
├── app/[locale]/login/    # sign-in (outside the app shell)
├── app/[locale]/(app)/    # shell with sidebar: dashboard, regulations (+ detail), copilot
├── components/
│   ├── ui/                # generated shadcn/ui primitives
│   ├── layout/             # sidebar, top bar, FR/EN toggle, user menu
│   ├── features/          # business components (including regulation-mindmap, regulation-history-tab,
│   │                       #   markdown-line, pagination-controls)
│   └── providers/          # MSW, TanStack Query, session
├── i18n/                  # next-intl routing (+ src/proxy.ts for the middleware)
├── lib/api/               # typed client, one file per resource
│   └── backend/           # switch + adaptation to Thư's real backend
├── lib/mocks/             # MSW handlers + demo corpus + aggregates (summary.ts)
├── lib/evidence-match.ts  # locates an excerpt in a document (pure function)
├── lib/mindmap-layout.ts  # layout + filtering of the impact map (pure function)
├── lib/simple-markdown.ts # minimal Markdown rendering of the source text (pure function)
├── messages/              # fr.json / en.json — always in sync
├── types/api.ts           # mirror of docs/api-contract.md (Zod schemas + types)
└── scripts/screenshot.ts  # visual check loop
```

## Vercel deployment

`vercel.json` sets the framework and the commands. In the Vercel dashboard,
**Root Directory must be set to `frontend`** (the repository also contains `backend/`).
**Not deployed yet** — see `../docs/phases/phase-5-client-readiness.md`.

> **Status 2026-10-29:** deployed at https://ia-bank-demo-one.vercel.app from the `production`
> branch, with the backend on Railway.
