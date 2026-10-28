# Business Glossary — KYC / AML / Regulatory Compliance

Condensed for developers (no need to be a banking expert). Full detail:
`../../Abiaka_Regulatory_AI_POC_Full_Project_Guide_EN.docx` section 3.

| Term | Practical meaning in this POC |
|---|---|
| **Regulation** | External legal/regulatory text that applies to the bank (e.g. an ACPR instruction, an EBA guideline). |
| **Requirement / Obligation** | One atomic obligation extracted from a regulation. A single document often contains 20-50. ID format `REQ-XXX`. |
| **Policy** | High-level internal rule ("the bank must know its customers"). |
| **Procedure** | Detailed internal instructions that implement a policy/obligation (e.g. `KYC-001`). |
| **Control** | Check that the procedure is actually applied. |
| **Evidence** | Exact source passage that supports a finding, traceable to a document + section/page. |
| **Regulatory Impact Assessment** | Determines what a new regulation impacts in the bank. |
| **Gap Analysis** | Compares an external requirement with internal content to find missing/partial coverage. |
| **KYC** | Know Your Customer — identify/verify a customer's identity and profile. |
| **AML/CFT** | Anti-Money Laundering / Combating the Financing of Terrorism. |
| **UBO / Bénéficiaire effectif** | Ultimate Beneficial Owner: the natural person who actually owns/controls the customer. |
| **PEP / PPE** | Politically Exposed Person / Personne Politiquement Exposée (the French term). |
| **STR / Déclaration de soupçon (DS)** | Suspicious Transaction Report to the competent authority (Tracfin in France). |
| **Human-in-the-loop** | The AI assists and prioritises; the Head of Compliance takes the final decision. |

## Key functional chain

```
Regulation → Obligation → Policy → Procedure → Control → Evidence
```

## The 5 assessment statuses — fixed vocabulary, do not invent others

| Value (code) | Meaning | Suggested colour |
|---|---|---|
| `COVERED` | The internal procedure covers the requirement satisfactorily | green |
| `PARTIAL` | The procedure covers part of the requirement, not all of it | amber/yellow |
| `POTENTIAL_GAP` | An important element of the requirement seems to be missing from the procedure | red |
| `NO_RELEVANT_PROCEDURE` | No internal procedure matches this requirement | grey/neutral |
| `EXPERT_REVIEW` | Ambiguous, needs a human's interpretation (do not decide automatically) | purple/blue |

## Human validation statuses (`human_status`)

`PENDING` → `ACCEPTED` | `REJECTED` | `ESCALATED` (with an optional comment).
