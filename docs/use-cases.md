# Business use cases (UC01–UC10) — acceptance criteria

Source: Francis's email of 2026-09-11. Replaces the need for the "6 user journeys" (unreadable
ChatGPT link, see `PROGRESS.md`) — this table is more precise and actionable. **P0 = the first
6, mandatory for the demo; P1 = the next 4, if time allows.**

Use it as a test grid before any demo, and as a non-regression checklist after every coding
session that touches analysis or validation.

| # | Use case | User action | Expected Abiaka behaviour | Priority | Corpus case |
|---|---|---|---|---|---|
| UC01 | Analyse a new regulation | Compliance uploads/selects a regulation | Extract the requirements → find the impacted Bank procedures → identify coverage/gaps → actions | P0 | CASE-01 to CASE-08 (existing corpus) |
| UC02 | Analyse a new/modified Bank procedure | Uploads/selects a procedure | Search the Bank Compliance KB → applicable requirements → assess coverage | P0 | **Needs the Procedure screen — see Phase 7 § Jour 0 (Day 0)** |
| UC03 | Extend a procedure's analysis to Europe | Selects "Bank + European Sources" | Bank KB first → then EU search → additional candidate EU requirements | P0 / **wow moment** | CASE-09 (new, see below) |
| UC04 | Investigate a potential gap | Clicks a finding | Regulatory evidence vs internal evidence, missing element, explanation, recommended action | P0 | Already covered (`FindingDetailDialog`) |
| UC05 | Human validation | Compliance reviews an AI finding | Accept / Reject / Escalate / Comment, with an audit trail | P0 | Already covered |
| UC06 | Ambiguous requirement | The AI meets non-specific regulatory wording | Return "Human interpretation required", never invent a precise rule | P0 | CASE-07 ("Ambiguous Requirement", existing corpus) |
| UC07 | No internal coverage | A requirement has no matching procedure | "No relevant procedure found" + recommend an investigation (never "the bank is non-compliant") | P1 | CASE-08 |
| UC08 | Cross-language analysis | EU requirement in English vs Bank procedure in French | Correct semantic match, original evidence kept in its source language (never translated in place of the original) | P1 | Existing EBA (EN) corpus / FR procedures |
| UC09 | Query the Compliance Copilot | "What are my high-priority gaps?" | Answer only from existing findings/evidence, always with a citation | P1 | Copilot screen (currently P2/placeholder — see `docs/phases/phase-5-client-readiness.md`) |
| UC10 | Operational action plan | "Generate an operational action plan" | Generate prioritised actions, impacted teams, validation status | P1 | New — overlaps the Report/Export screen mentioned in Francis's previous email |

## Detail of the most structuring cases

### UC05 — The Compliance Officer disagrees with the AI

Important for credibility with a bank. The AI proposes `POTENTIAL_GAP`, the human chooses
`REJECTED` and writes a reason ("Covered by PROC-KYC-012 §7.2, which was not retrieved"). The
finding keeps **both** pieces of information side by side: the original AI assessment AND the
human decision — never overwrite one with the other. Already supported by the current model
(`assessment` + `human_status` + `reviewer_comment` are separate fields) — check that the UI
displays both at the same time on a rejected finding, not only the latest state.

### UC06 — The AI does not know (safe behaviour)

Use CASE-07 from the existing corpus. A word such as "promptly" or "proportionate to risk" must
never become "the Bank must remediate within 30 days" — the displayed text must stay at the
`EXPERT_REVIEW` level, with an explanation that says explicitly that the source passage sets no
precise deadline. Check this text in `docs/glossary.md`/`docs/ui-guardrails.md` — this is a
non-regression test of the product guardrails, not just a functional test.

### CASE-09 — European Extended Search (new golden scenario)

- **Procedure**: `PROC-ICT-017` — ICT Third-Party Risk Management (synthetic, to be created)
- **Run 1 — Bank KB only**: 11 requirements found (7 covered / 3 partial / 1 potential gap)
- **Run 2 — Bank + European Regulatory Sources**:
  - Bank KB: 11 requirements (unchanged)
  - EU candidates: 9
  - Already represented in the Bank KB: 6
  - New EU candidates: 3
  - Example of a new finding: *ICT Exit Strategy Testing*, source DORA — Regulation (EU)
    2022/2554, procedure `PROC-ICT-017 §8.4`, assessment `PARTIALLY_COVERED`, applicability
    `TO_BE_CONFIRMED`, priority `HIGH`
- This scenario is THE end-to-end test of Phase 7 (milestone D5 + final DoD)

## Recommended demo storyline (do not walk through the 10 UCs one by one)

1. New/modified procedure
2. Analyse against the Bank KB
3. Show a potential gap
4. Extend the search to Europe
5. Discover an additional EU candidate
6. Open the evidence (EU regulation ↔ Bank procedure)
7. The AI meets an ambiguity → Human interpretation required
8. The Compliance Officer validates
9. Generate the operational action plan

Closing message (Francis's wording, to be reused as-is in the demo): *"Abiaka first
tells you what your own Compliance knowledge base already knows. With one selection, the
Compliance Officer can extend the same analysis to authoritative European regulatory sources,
identify additional candidate requirements, understand exactly why they may impact the procedure,
and decide what action to take — with the final decision remaining under human control."*
