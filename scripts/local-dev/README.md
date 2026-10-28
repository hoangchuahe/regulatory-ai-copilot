# Running Thư's backend locally, without PostgreSQL

`backend/QUICKSTART.md` assumes PostgreSQL 18 is installed and an `alembic/` folder that, in
practice, was never committed (`alembic/versions/*.py` is gitignored on the backend side).
Following that guide as-is fails on a machine that has neither.

These scripts work around both problems **without changing a single line under
`backend/`** (read-only zone, see `../../CLAUDE.md` § 2):

- they rely on the fact, checked by reading `backend/app/models/`, that the backend
  **has no PostgreSQL-specific column type** — it runs unchanged on
  SQLite;
- they use `backend/reference_database/reference/abiaka_regulatory_demo.sqlite`,
  which **is committed** and already holds the 1,262 demo rows (32 documents,
  44 requirements, 54 requirement × procedure pairs…).

## Getting started

```bash
# Once: creates .venv-backend/, copies the reference database into .local/, adds the
# `users` table (missing from the reference database) and 4 demo accounts.
./scripts/local-dev/setup-backend.sh

# Every session:
./scripts/local-dev/run-backend.sh          # http://localhost:8000/docs — Neon cloud database (default)
DB_TARGET=local ./scripts/local-dev/run-backend.sh   # SQLite database .local/backend-dev.sqlite
cd frontend && pnpm dev                     # http://localhost:3000 (mock mode by default)
```

`DB_TARGET` (`cloud` by default, or `local`) also applies to `run-public.sh`; the logic
is in `db-env.sh`. In `cloud` mode, the URL is read from `backend/.env` (never copied):
writes land in the Neon database **shared with Thư**.

> **Status 2026-10-29:** `backend/.env` is not in the repository, so without it use
> `DB_TARGET=local`. The AI routes also read `OPENROUTER_API_KEY` (extraction and analysis) and
> `CHAT_BOT_KEY` (Copilot) from it: without them, those routes answer 400 and 503.

Sign-in (same as the accounts of the frontend's MSW corpus):
`marie.lefevre@iabank.fr` / `demo1234`.

Prerequisite: Python **>= 3.10** (`brew install python@3.12` if the machine only has the
Python 3.9 shipped with macOS — the case checked on this machine).

**Pitfall met in practice (2026-09-11): another project may already hold port 8000.** If
another repository on the machine also starts a Python service on 8000 (e.g. a
`uvicorn main:app --port 8000` from a completely different project), `curl localhost:8000/health`
can answer `200` by coincidence (if that other service also has a
`/health` route) although it is not Thư's backend at all — sign-in then fails
with a generic error. Check before concluding that the backend is
"broken":
```bash
lsof -i :8000 -sTCP:LISTEN        # which process is really listening on this port?
curl -s localhost:8000/openapi.json | python3 -c "import json,sys; print(list(json.load(sys.stdin)['paths']))"
# must list /api/auth/signin, /api/documents, /api/mappings/*… — nothing else.
```
If the port is taken by another project, start this one on a different port
rather than stopping the other service (which is not ours):
```bash
PORT=8010 ./scripts/local-dev/run-backend.sh
# then in frontend/.env.local: NEXT_PUBLIC_BACKEND_URL=http://localhost:8010
# and restart `pnpm dev` (Next.js only rereads .env.local at startup).
```

## Connecting the frontend to this backend

By default, the frontend stays 100% on MSW (`NEXT_PUBLIC_BACKEND_URL` empty). To
connect the endpoints the backend already covers:

```bash
# frontend/.env.local (gitignored file, to create)
NEXT_PUBLIC_BACKEND_URL=http://localhost:8000
```

Then restart `pnpm dev`. What this changes in practice (see
`frontend/src/lib/api/backend/config.ts`):

| Screen / data | Source with `NEXT_PUBLIC_BACKEND_URL` set |
|---|---|
| Sign-in | real backend — real JWT, `POST /api/auth/signin` |
| Regulation list | real backend — documents `category=EXTERNAL` |
| Internal procedure list | real backend — documents `category=INTERNAL` |
| Regulation detail + source text | real backend |
| Requirements of a regulation | real backend, `domain` filter passed to the server |
| Findings, dashboard, upload, human validation, user list | **still MSW** — the backend does not expose them yet |

The two sources coexist without conflict: the contract (`docs/api-contract.md`) goes out as
relative requests intercepted by MSW, the real backend goes out as absolute requests to
`localhost:8000`.

> **Status 2026-10-29:** the variable is now `NEXT_PUBLIC_API_BASE_URL` (see
> `frontend/.env.example`); `NEXT_PUBLIC_BACKEND_URL` is ignored. The MSW demo corpus was
> removed (commit `4f56230`), so mock mode shows empty screens. Findings, dashboard, upload, human
> validation and the user list (`GET /api/users`) now come from the real backend.

## What remains a workaround, to settle with Thư

- **`frontend/src/lib/api/backend/current-version-id.ts`** rebuilds the version
  identifier from a convention observed in the reference database
  (`VER-{document_id}-{version_no}`), because no endpoint returns the current
  version of a document (open question no. 1 of `docs/backend-integration.md`). If the
  convention changes, the « Texte source » [Source text] tab fails loudly (`BackendGapError`)
  rather than showing an empty text.
- A document's status (`NOT_ANALYZED` / `ANALYZED`) is **always** returned as
  `NOT_ANALYZED` in real backend mode: the backend says nothing about the progress of the
  analysis or about findings — claiming it would go against `docs/ui-guardrails.md`.
- `GET /api/users` does not exist on the backend: assignment and escalation stay on
  the MSW list whatever the mode.

## Resetting

```bash
rm -rf .local .venv-backend
./scripts/local-dev/setup-backend.sh
```

The committed reference database (`backend/reference_database/`) is never modified:
`setup-backend.sh` makes a **copy** of it in `.local/backend-dev.sqlite` before
adding the `users` table.
