# UI Guardrails — Allowed / Forbidden Wording

**Non-negotiable product rule**: the tool must NEVER state an autonomous compliance conclusion.
It proposes findings; a human (Head of Compliance) validates. Every string displayed to the user
(label, message, tooltip, button text, Copilot text) must follow this table. Source: Full Project
Guide section 14.1.

The French phrases below are the product's reference wording (the French UI strings in
`frontend/src/messages/fr.json` follow it): they are kept as written, with an English gloss in
brackets.

| ❌ Never display | ✅ Use instead |
|---|---|
| "The Bank is non-compliant" / "La banque n'est pas conforme" | "Écart potentiel détecté — revue de conformité requise" [Potential gap detected — compliance review required] |
| "The AI validated compliance" / "L'IA a validé la conformité" | "Preuve identifiée — validation par un expert requise" [Evidence identified — expert validation required] |
| "This procedure violates the regulation" / "Cette procédure viole la régulation" | "Cette procédure ne semble pas couvrir intégralement l'exigence REQ-XXX" [This procedure does not appear to fully cover requirement REQ-XXX] |
| "No rule exists" / "Aucune règle n'existe" | "Aucune procédure interne pertinente trouvée dans le corpus indexé" [No relevant internal procedure found in the indexed corpus] |
| Copilot answer without a citation | Always cite the evidence, or say explicitly that no evidence was found |

## Other wording rules

- The confidence score (if displayed) is NOT a legal probability — label it as a decision-support
  indicator ("force de la preuve" [evidence strength]), never as a certainty.
- Always display the source evidence (excerpt + document/section reference) next to any material
  finding — never a "bare" finding without traceability.
- Displayed text must stay in French by default (see `docs/ui-guidelines.md` — French-first).

## Colour code of the 5 assessment statuses

See `docs/glossary.md` for the full table — follow it strictly for visual consistency (do not
reinvent a palette per screen).
