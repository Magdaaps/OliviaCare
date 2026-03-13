# Sprint 1 — Foundation & Infrastructure
**Duration:** 2 weeks | **Est. effort:** 8 days | **Goal:** Working local dev environment, GCP infrastructure provisioned, NestJS + Drizzle skeleton running

## Reference Documents (load before implementing)
- `Architecture/docs/arch-stack-workflow.md` — sections 9 (tech stack), 10 (repo structure), 11.1 (CI/CD), 11.4 (migrations)
- `Architecture/docs/arch-infrastructure.md` — sections 8.1 (GCP setup), 8.2 (env separation), 8.3 (network)
- `Architecture/docs/arch-roadmap.md` — ADR-001 (modular monolith), ADR-003 (GCP), ADR-008 (Drizzle ORM), ADR-010 (trunk-based dev)

## Tasks

| # | Task | Est. Days | Dependency | Branch |
|---|------|-----------|------------|--------|
| 1 | Turborepo monorepo: TypeScript strict, ESLint, path aliases, workspace packages | 2 | — | `feature/turbo-setup` |
| 2 | Terraform: GCP project, Cloud SQL (core + PII vault), Cloud Storage, Secret Manager, VPC, Cloud Run skeleton | 3 | — | `feature/terraform-gcp` |
| 3 | Docker Compose: PostgreSQL 16 + Redis 7, .env.example, dev startup script | 1 | #1 | `feature/docker-local` |
| 4 | NestJS app scaffold: AppModule, ConfigModule, health endpoint, request logging (pino), correlation IDs | 2 | #1 | `feature/nestjs-scaffold` |
| 5 | Drizzle ORM: core schema (users, roles), PII vault schema (pii_records), drizzle.config.ts, first migration, seed script | 3 | #4 | `feature/drizzle-setup` |

## Definition of Done
- [ ] `npm run dev` starts both frontend stub and NestJS with hot reload
- [ ] `GET /health` returns 200
- [ ] Drizzle migrations run cleanly on local Docker PostgreSQL
- [ ] GitHub Actions CI passes (lint + type-check)
- [ ] Terraform plan produces no errors for staging environment

## Key Architectural Constraints
- Two separate PostgreSQL instances from day 1: core DB + PII vault (ADR-002)
- Node.js 22 LTS, NestJS 11.x, Next.js 15.x, Drizzle ORM — not Prisma (ADR-008)
- GCP region: europe-central2 (Warsaw) (ADR-003)
- All secrets via Secret Manager — no hardcoded credentials
