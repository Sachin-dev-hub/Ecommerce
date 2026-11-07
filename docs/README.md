# Project Documentation Index

All guidance, architecture, and implementation notes for this project are maintained **in files under `docs/`**, organized per phase. This is the single place to look for explanations — the chat is only for conversation; the files are the source of documentation.

## Conventions

- One file per topic/response, numbered in reading order within a phase.
- `docs/phaseN/` holds everything for phase N.
- This index (`docs/README.md`) links all files so you can navigate by phase.
- As the project progresses, each phase adds its own folder and files.

## Phase 1 — Application Development

| # | File | What it covers |
|---|------|----------------|
| 1 | [01-architecture.md](phase1/01-architecture.md) | High-level application architecture, diagram, data flow, why this stack |
| 2 | [02-folder-structure.md](phase1/02-folder-structure.md) | Proposed repo folder structure and purpose of each part |
| 3 | [03-database-design.md](phase1/03-database-design.md) | Database design: ERD, Prisma schema, relationships, design decisions |
| 4 | [04-api-design.md](phase1/04-api-design.md) | REST API design: endpoints, auth, sample request/response shapes |
| 5 | [05-implementation-steps.md](phase1/05-implementation-steps.md) | Step-by-step implementation plan, environment setup, validation checklist |
| 6 | [06-security-best-practices.md](phase1/06-security-best-practices.md) | Security best practices applied from day one |
| 7 | [07-technology-stack.md](phase1/07-technology-stack.md) | **Every technology we use, and what we use it for** (single reference file) |
| 8 | [08-backend-setup-on-poc.md](phase1/08-backend-setup-on-poc.md) | Backend environment setup on the user's machine (POC): corrected commands, `.env` file, Postgres check |

## Later phases

Added as each phase begins — Phase 2 (Git & GitHub), Phase 3 (Docker & Kubernetes), Phase 4 (Jenkins CI/CD), Phase 5 (AWS EKS + Terraform), Phase 6 (Domain & TLS), Phase 7 (Production readiness).

---

*Status markers live in the repo root `CLAUDE.md`. A phase is only marked done after the user confirms it.*