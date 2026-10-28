# IA Bank — Regulatory AI Copilot (POC)

A 5-week POC for ABIAKA: a decision-support tool that analyses a new regulation, identifies the
internal procedures it may impact, highlights possible gaps with supporting evidence, and lets a
Head of Compliance validate each finding.

> This file is for humans. For the context meant for a Claude Code agent, see
> [`CLAUDE.md`](./CLAUDE.md) — it is read automatically when a session opens.

## Team

- **Giang (Dev A, full time)** — Frontend + integration + demo. See [`frontend/`](./frontend).
- **Thư (Dev B, part time)** — Backend + AI pipeline (extraction, retrieval, comparison).
  See [`backend/`](./backend).

## Quick start

- Full product context: `../Abiaka_Regulatory_AI_POC_Full_Project_Guide_EN.docx` (parent folder)
- Demo corpus (fictional procedures + regulations): `IABank_*.docx` files in the parent
  folder
- Current project status: [`PROGRESS.md`](./PROGRESS.md)
- Plan by phase: [`docs/phases/`](./docs/phases)
- Frontend ↔ backend API contract: [`docs/api-contract.md`](./docs/api-contract.md)

## Target deployment

Frontend → Vercel · Backend → Railway/Render (final choice depends on Thư's stack).
