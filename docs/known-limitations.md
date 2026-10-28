# Known limitations — Regulatory AI Copilot POC

> Demo/Phase 5 document, meant to be shown or summarised to the client. Write in one sentence what
> each limitation means for the demo; the full technical detail (gaps endpoint by endpoint, open
> questions with Thư) stays in `docs/backend-integration.md`, whose role does not change.
> Updated by every session that touches the backend integration — see `PROGRESS.md`.
>
> English translation, 2026-10-29. Lines starting with **Status 2026-10-29** were added during
> the translation and checked against the code that day; the rest is the original text. French
> strings displayed by the app are kept verbatim, with an English gloss in brackets.

## What works end to end today

Account creation → sign-in → selecting a regulation → structured requirements → Impact analysis
→ side-by-side evidence → human validation (Accept/Reject/Escalate), demonstrable without manual
intervention, in mock mode (MSW) as in real backend mode (`NEXT_PUBLIC_BACKEND_URL`) for
everything the latter covers. The cross-language scenario of the demo script (EBA regulation in
English ↔ internal procedures in French) is covered by the hand-built demo corpus.

## Limitations to announce before the demo

1. **Human validation (Accept/Reject/Escalate) has been persisted on the real server since
   2026-09-09**, via `PUT /api/mappings/:id/human-status` — the decision survives a reload,
   another tab or another computer. What is NOT sent to the backend: `custom_action`
   (the action the reviewer chose) and `reviewer_comment`, for lack of server-side columns — the
   decision (Accepted/Rejected/Escalated) is real, these two details stay local to the browser
   session. In mock mode (MSW), nothing changes: everything stays in `sessionStorage` as
   before.
   **New (2026-09-15)**: **assigning an escalated finding to a specific person no longer has a
   server equivalent.** Thư removed the `assignee` column from
   `requirement_procedure_map` (`3f3f05b`) in favour of assignment at document level only (see
   point 6 below, now real). Choosing an assignee on an escalation is still possible and shows
   immediately, but is no longer saved on the server — it disappears on reload, as before
   2026-09-09.
2. **The History tab (who decided what, and when) only works in mock mode.** The backend can now
   store a decision but does not yet say who made it (`actor_id`) — without that the history
   would stay empty for every real decision. Explicit decision: better to show the tab only with
   consistent demo data than with an empty, confusing history.
3. **The internal evidence is not always the exact passage cited.** In real backend mode, the link
   between a finding (requirement × procedure pair) and a precise passage of the internal document
   does not exist on the server yet (`chunk_id` missing). The excerpt shown is then the beginning
   of the procedure's document, explicitly labelled « passage non ciblé par le mapping » [passage
   not targeted by the mapping] rather than presented as a precise citation — never an invented
   text, only less precise. In mock mode, the internal evidence does cite the exact passage.
4. **The dashboard (portfolio and per regulation) is recomputed on the client, not served by
   the backend.** `/api/dashboard/overview`, `/summary` and `/map` do not exist on the server: the
   frontend reloads the requirements and findings of each regulation and then aggregates locally —
   same figures, but a network cost that would grow with a corpus much larger than the current 8
   demo regulations.
   **Status 2026-10-29: resolved.** The dashboard now loads from one backend endpoint,
   `GET /api/dashboard/portfolio` (backend PR #8, deployed on 2026-10-07; `backend/API.md` § 5.5).
5. **Uploading a new regulation is still entirely simulated (MSW).** The backend exposes an
   update of an existing document (`POST /api/documents/{id}/update`) but not yet creation with
   automatic requirement extraction — importing a `.docx` in real backend mode would therefore
   have no observable effect beyond the screen.
   **Status 2026-10-29: resolved.** Uploads go to the real ingestion routes,
   `POST /api/documents/regulation-ingest` and `POST /api/procedures/ingest`
   (`frontend/src/lib/api/upload.ts`); requirements are then extracted when the user clicks
   « Analyser » [Analyze].
6. **✅ Resolved on 2026-09-15 — the « Personne en charge » [Assigned to] assignment of a
   regulation is now persisted on the real server.** Thư added an `assignee` field on `Document`
   and `PUT /api/documents/:id/assignee` (`4ea5611`): the assignment survives a reload, another
   tab or another computer, like human validation. Replaces the browser-only local fix that had
   existed since 2026-09-09 (removed the same day, `regulation-assignee-overrides.ts` deleted).
7. **Backend-generated texts (explanation, recommended action) are shown in French when the
   backend provides a variant.** Since 2026-09-09, the screen displays the French variant
   (`explanation_fr`/`recommended_action_fr`) in the FR interface, as soon as the backend provides
   it. Nuance: this translation only exists for requirement × procedure pairs created or updated
   after this backend change — older pairs of the corpus stay in English until Thư regenerates
   them. The evidence itself (`excerpt`) always stays, as a product principle, in its source
   language — this point does not concern it.
8. **A finding's `priority` is provisionally derived from the requirement's `risk_level`** in
   real backend mode (same LOW/MEDIUM/HIGH values) — a reasonable approximation, not yet
   confirmed with Thư as a definitive equivalence.
9. **Demo authentication is not real access control.** An account creation screen now exists
   (since 2026-09-09), working in mock mode as in real backend mode — but in real backend mode, no
   fine-grained authorisation by role is applied on the frontend beyond the sign-in screen; in
   mock mode, authentication stays entirely simulated (session in `sessionStorage`).
10. **An account's role can be changed (« Utilisateurs » [Users] screen), but without access
    control.** When an account is created, the role is set automatically
    (`Responsable Conformité` [Compliance Officer] in mock mode, `COMPLIANCE_OFFICER` in real
    backend mode); a dedicated screen then lets you change it. Accepted limitation: any signed-in
    person can change the role of any other account, not only their own — announced directly in
    the screen rather than hidden.
    **Status 2026-10-29: no longer true on the server.** `PUT /api/users/{user_id}/role` requires
    an administrator role (`COMPLIANCE_ADMIN` or `Admin Base de Connaissances`) and answers 403
    otherwise (`backend/app/core/deps.py`, backend PR #7).
11. **The Copilot remains a presentation screen (P2)** for the very first client demo — it
    must not become the main screen of the demo script (`docs/phases/phase-5-client-readiness.md`).
    **Clarification of 2026-09-11**: Francis himself classed it as one of the "5 official screens"
    of the application (not an optional 6th screen) in his answer on the navigation structure
    (see `docs/phases/phase-6-francis-feedback.md` §2.2) — he confirms however that Dashboard and
    Copilot "add polish and commercial impact" and are not on the critical path of the 1st
    demo (Analyze / Impact Analysis / Evidence & Validation are). So: still not a priority to
    build now, but no longer to be presented as a mere side feature in the long run — the Copilot
    is meant to answer real questions (UC09/UC10, `docs/use-cases.md`), not only to exist as a
    showcase.
12. **An expired session (JWT) in real backend mode shows a generic error instead of sending the
    user back to sign-in.** The screen still shows the person as signed in (name at the top
    right) but every data screen shows « Le chargement a échoué » [Loading failed] without
    explaining why. Workaround until then, for a demo: signing out and back in is enough to start
    again with a fresh token. **Token lifetime extended on 2026-09-10 on the backend**
    (`access_token_expire_minutes`, Thư's commit `462fd58`): 60 min → 1,440 min (24 h) — the
    symptom should no longer be seen in a normal demo, only on a session left open for more than a
    day. The behaviour itself (generic error instead of a redirect) is not fixed, just much less
    likely to trigger.
    **Status 2026-10-29: resolved.** A 401 now clears the token and the stored user
    (`frontend/src/lib/api/token.ts`), and the session guard redirects to sign-in. Checked that
    day on the hosted app with an invalid token: the page landed on `/fr/login`.

13. **The dark-mode categorical palette has never been validated by the
    `validate_palette.js` script (`dataviz` skill).** Fixed on 2026-09-11: slots 2 and 8
    (red/orange) were replaced in light AND dark (see `docs/ui-guidelines.md`), but when the
    remaining 6 dark colours were run through the validator, **the lightness band already fails
    for 5 of the 6** (`--mode dark` expects L 0.48–0.67 OKLCH, current values go up to 0.826) — a
    pre-existing problem, independent of Francis's request about red/orange, never detected before
    because the script had not been run. Not blocking for the demo (light mode by default), but to
    be redone entirely (8 slots) with the validator before considering dark mode presentable to
    the client.

14. **The 4 access profiles (Head of Compliance / Compliance Officer / Auditeur / Admin Base de
    Connaissances, `frontend/src/lib/access-profile.ts`) are a demo-experience layer, not real
    access control.** The profile is derived only from the `role` label (a free string, never
    validated by the backend — see point 10 above) already present in the browser session:
    changing your own role from the « Utilisateurs » [Users] screen instantly changes what you see
    (sidebar, landing screen, decision buttons), without going through the server. Scoping
    decision taken by Giang on 2026-09-10
    (`docs/phases/phase-6-francis-feedback.md` § 1): reproduce what each profile would see in the
    demo, not build real server-side authorisation.

15. **« Ouvrir dans un nouvel onglet » [Open in a new tab] (procedure) fails silently if the
    browser blocks pop-ups.** `lib/open-in-new-tab.ts` opens an empty window
    (`window.open("", "_blank")`) so that it can copy the session into it before navigating it —
    if the browser blocks that call (no direct user gesture, strict settings), nothing happens and
    no error message is shown. In normal use the click always comes from a real user click, so it
    is not reproduced in practice, but keep it in mind if this click is ever automated.
    The Print button that comes with it (`canPrint`) is the same experience guardrail as the
    access profiles (point 14) — not real access control.

16. **The search for European texts (Knowledge Base) only covers titles, and depends on a public
    service with no published rate limit.** `GET /api/eu-search` queries the CELLAR SPARQL
    endpoint: the keyword is searched in the title (not the full text) — an exact CELEX number
    typed in the same field is recognised and searched as such —, the EuroVoc subjects are a
    closed list of 8 banking subjects. No official limit is published (a burst of 30 requests
    without refusal on 2026-09-15); each request is cached for 1 h and a CELLAR failure shows as
    an error with « Réessayer » [Retry]. **Even in mock mode, this search calls the real CELLAR**
    (internet connection required for the demo). The Vercel function runs in the default region
    `iad1`: latency to Luxembourg not measured in production. This is a consultation: no link is
    made with the procedures, and applicability remains to be confirmed by Compliance (notice
    shown under the table).
    The `/api/eu-search` route is not authenticated: anyone who knows the URL can trigger CELLAR
    requests (each different keyword bypasses the cache) — acceptable for a demo, plan a rate
    limiting rule (Vercel Firewall) if the URL is shared widely.
    **Reading and .docx (2026-09-17)**: clicking a title opens the full CELLAR text in the
    app (`GET /api/eu-document`, same lack of authentication). The original images and style
    sheets are not displayed (links relative to the EUR-Lex site, blocked). The .docx is not
    published by the EU: it is the official HTML wrapped up, which Word or LibreOffice converts on
    opening — Google Docs and quick previews show an empty document. Very large acts
    (CRR ≈ 6.6 MB of HTML): not checked from Vercel.

## Product guardrail, reminder

None of these limitations calls into question the non-negotiable rule of `docs/ui-guardrails.md`:
the tool never states an autonomous compliance conclusion, and a finding is never displayed
without its evidence — even when that evidence is, as in point 3, honestly flagged as less
precise than it could be.
