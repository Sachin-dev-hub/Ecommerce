# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Build a real-world e-commerce website selling handmade diyas, and take it through a complete DevOps lifecycle from local development to a production deployment on the user's purchased domain. The user's role framing: act as Senior Full Stack Developer, DevOps Engineer, Kubernetes Engineer, and AWS Architect — guide step by step, one phase at a time.

The single source of truth is the `requirement` file at the repo root. Re-read it before starting a new phase.

## Current State

- **No application code exists yet.** The repo contains only `requirement` (project spec), `CLAUDE.md`, and `README.md`.
- Phase 1 (application development) has not been started.
- **Immediate next step (the spec's "First Task"):** present the Phase 1 architecture — application architecture, proposed folder structure, database design (schema + relationships), and implementation steps — then **stop and wait for the user's confirmation before generating any application code**.

## Agreed Tech Stack

- **Frontend:** React.js (responsive e-commerce UI: product listing, product details, image gallery, shopping cart, admin login + dashboard)
- **Backend:** Node.js + Express.js (REST APIs: CRUD products, product image upload, admin auth/authorization, order management)
- **Database:** PostgreSQL with Prisma ORM (products, images, customers, orders with proper relationships)

### Planned for later phases (not yet built)

Docker + Docker Compose, Kubernetes (user has a practice cluster), Jenkins CI/CD (declarative pipeline, Trivy scanning, rollback), AWS EKS provisioned with Terraform (VPC, ECR, RDS PostgreSQL, S3 for images, CloudWatch), domain DNS + ACM + ALB + HTTPS, Prometheus/Grafana monitoring, autoscaling, payment gateway.

## Phased Roadmap

Status markers: `[next]` = currently in progress / next to start · `[done]` = completed and user-confirmed · `[pending]` = not started. **Update these markers only when a phase is completed and the user confirms it — never advance a phase without that confirmation.** Re-read `requirement` before starting each new phase.

1. `[next]` Application development (frontend + backend + database) — must work locally before proceeding
2. `[pending]` Git & GitHub (repo, branching strategy, PRs, docs)
3. `[pending]` Docker & Kubernetes POC (Dockerfiles, Compose, namespace, Deployments/Services, ConfigMaps/Secrets, storage, Ingress, health checks, troubleshooting)
4. `[pending]` Jenkins CI/CD (GitHub integration, webhooks, declarative Jenkinsfile, builds, tests, Docker images, Trivy, registry push, k8s deploy, verification, rollback)
5. `[pending]` AWS EKS deployment via Terraform (VPC, IAM, EKS, worker nodes, ECR, RDS, S3, Jenkins↔AWS, CloudWatch)
6. `[pending]` Domain configuration & TLS (DNS records, ACM, ALB, HTTPS, HTTP→HTTPS redirect, point domain at app)
7. `[pending]` Production readiness & launch (security, backups, Prometheus/Grafana, centralized logging, alerts, autoscaling, load testing, DR, rollback, payment gateway, full purchase-journey test)

## Working Rules (from the spec's "Important Instructions")

These are hard constraints on how to work in this repo:

1. Work on **one phase at a time**. Never skip a phase.
2. **Do not proceed to the next phase until the user confirms the current phase is complete.**
3. Explain the architecture **before** implementing; wait for confirmation before generating code for a new phase.
4. Provide complete code and folder structures, and explain the purpose of every important configuration.
5. Give exact commands and execution instructions, plus how to test/validate each implementation.
6. Explain real-world production scenarios and common issues.
7. Help troubleshoot root causes, not workarounds.
8. Follow security best practices and maintain documentation throughout.

### DevOps interview prep

For every phase, also explain: why the technology, how it works, how it integrates with other components, what happens in a real production environment, common issues + how DevOps engineers troubleshoot them, and likely interview questions.

## Commands

No build/lint/test commands exist yet — the repo contains no application code. Commands will be introduced phase-by-phase as the stack is built. Do not invent tooling that has not been established.