<!-- Source: Caregiver_Platform_IT_Architecture.md | Sections: 9, 10, 11 -->

## 9. Recommended Technology Stack

| Layer | Technology | Version | Why This Choice |
|-------|-----------|---------|----------------|
| Frontend | Next.js (App Router) | 15.x | SSR for SEO (family acquisition), React ecosystem, Vercel deployment simplicity |
| UI Components | shadcn/ui + Tailwind CSS | Latest | Accessible components, consistent design system, no runtime overhead |
| Backend | NestJS | 11.x | Modular architecture support, TypeScript-native, decorators for clean domain code |
| Runtime | Node.js | 22 LTS | TypeScript across stack, async I/O for marketplace workloads, team skill alignment |
| Primary Database | PostgreSQL | 16 | JSONB for flexible care needs, PostGIS for geo queries, mature, reliable |
| ORM | Drizzle ORM | Latest | Type-safe queries, lightweight, excellent PostgreSQL support, better than Prisma for complex queries |
| Auth Provider | Supabase Auth | v2 | Self-hostable, OIDC-compliant, built-in MFA, phone auth, generous free tier |
| Object Storage | GCP Cloud Storage | N/A | CMEK encryption, signed URLs, lifecycle policies, tight GCP integration |
| Queue | GCP Cloud Tasks + Pub/Sub | N/A | Managed, pay-per-use, at-least-once delivery, dead letter queues |
| Cache | Redis (Memorystore) | 7.x | Session store, rate limiting, matching result cache |
| Search | PostgreSQL pg_trgm + full-text | Built-in | Sufficient for MVP; upgrade to Typesense when >10K caregivers |
| Email | Resend | N/A | Developer-friendly API, React email templates, good deliverability in EU |
| SMS | Twilio | N/A | Reliable phone verification in Poland, EU coverage |
| Push Notifications | Firebase Cloud Messaging | N/A | Free, reliable, cross-platform |
| Payments | Stripe Connect (Standard) | N/A | EU marketplace payments, split payments, KYC built-in, DAC7 reporting support |
| KYC / Identity | Onfido | N/A | EU document coverage, liveness detection, API-first, GDPR-compliant |
| Analytics / BI | Metabase + BigQuery | N/A | Self-hosted Metabase (free), BigQuery for scalable analytics warehouse |
| Monitoring | Grafana Cloud + GCP Cloud Logging | N/A | Free tier sufficient for MVP, dashboards, alerting |
| Tracing | OpenTelemetry + Grafana Tempo | N/A | Vendor-neutral, standard instrumentation |
| Infrastructure as Code | Terraform | 1.9+ | GCP provider, state management, environment parity |
| CI/CD | GitHub Actions | N/A | Free for private repos, mature GCP deployment actions |
| Validation | Zod | 3.x | Runtime type validation, shared schemas between frontend and backend |
| API Documentation | Swagger/OpenAPI via NestJS | 3.x | Auto-generated from decorators, interactive testing |

---

## 10. Repository Structure

Monorepo using Turborepo for workspace management. Single repository ensures atomic changes across frontend and backend while maintaining clear module boundaries.

```
caregiver-platform/
├── apps/
│   ├── web/                    # Next.js frontend (family + caregiver + admin)
│   │   ├── app/               # App Router pages
│   │   │   ├── (family)/      # Family-facing routes
│   │   │   ├── (caregiver)/   # Caregiver portal routes
│   │   │   ├── (admin)/       # Admin panel routes (RBAC-gated)
│   │   │   └── api/           # Next.js API routes (BFF layer)
│   │   ├── components/        # Shared UI components
│   │   ├── lib/               # Client-side utilities
│   │   └── public/            # Static assets
│   └── api/                    # NestJS backend (modular monolith)
│       ├── src/
│       │   ├── modules/
│       │   │   ├── auth/          # Authentication module
│       │   │   ├── user/          # User & Identity
│       │   │   ├── caregiver/     # Caregiver Profile
│       │   │   ├── family/        # Family Profile
│       │   │   ├── matching/      # Matching Engine
│       │   │   ├── scheduling/    # Scheduling
│       │   │   ├── documents/     # Document Verification
│       │   │   ├── payments/      # Payments + Stripe
│       │   │   ├── tax/           # DAC7 / Tax Reporting
│       │   │   ├── continuity/    # Continuity of Care
│       │   │   ├── messaging/     # Messaging
│       │   │   ├── notifications/ # Notifications
│       │   │   ├── admin/         # Admin Operations
│       │   │   └── audit/         # Audit & Compliance
│       │   ├── common/            # Shared guards, pipes, interceptors
│       │   ├── database/          # Drizzle schemas, migrations
│       │   ├── events/            # Domain event definitions
│       │   └── main.ts
│       ├── test/              # Integration + e2e tests
│       └── drizzle.config.ts
├── packages/
│   ├── shared/                 # Shared types, Zod schemas, constants
│   ├── ui/                     # Shared UI component library
│   └── config/                 # Shared ESLint, TypeScript configs
├── infrastructure/
│   ├── terraform/
│   │   ├── modules/           # Reusable TF modules
│   │   ├── environments/
│   │   │   ├── staging/
│   │   │   └── production/
│   │   └── main.tf
│   └── docker/
│       ├── docker-compose.yml     # Local development
│       └── Dockerfile.api
├── docs/
│   ├── adr/                    # Architecture Decision Records
│   ├── api/                    # API documentation
│   └── runbooks/               # Operational runbooks
├── turbo.json
├── package.json
└── .github/
    └── workflows/
        ├── ci.yml
        ├── deploy-staging.yml
        └── deploy-production.yml
```

---

## 11. Development Workflow

### 11.1 CI/CD Pipeline

GitHub Actions orchestrates all CI/CD, with Turborepo caching for fast builds:

| Stage | Trigger | Actions | Duration Target |
|-------|---------|---------|----------------|
| Lint + Type Check | Every push | ESLint, TypeScript compilation, Zod schema validation | <2 min |
| Unit Tests | Every push | Vitest for frontend, Jest for backend, coverage threshold 80% | <3 min |
| Integration Tests | PR to main | Testcontainers (PostgreSQL, Redis), API contract tests | <5 min |
| Security Scan | PR to main | npm audit, Trivy container scan, SAST (Semgrep) | <3 min |
| Build | Merge to main | Docker build, push to Artifact Registry | <3 min |
| Deploy Staging | Merge to main | Cloud Run deployment, smoke tests | <5 min |
| Deploy Production | Manual approval | Cloud Run deployment, health checks, canary (10% → 50% → 100%) | <10 min |

### 11.2 Branching Strategy

Trunk-based development with short-lived feature branches:

- **main** — always deployable, auto-deploys to staging
- **feature/*** — branch from main, PR within 1–2 days, squash merge
- **hotfix/*** — branch from main, fast-track review, deploy directly to production
- No long-lived branches. No develop branch. No release branches.

### 11.3 Testing Strategy

- **Unit tests:** domain logic, validators, utility functions (Vitest/Jest)
- **Integration tests:** API endpoints with real database (Testcontainers)
- **E2E tests:** critical user flows (Playwright) — registration, matching, booking, payment
- **Contract tests:** Stripe webhook signatures, Onfido callback schemas
- **Load tests:** k6 scripts for matching endpoint and search (run monthly)

### 11.4 Database Migrations

- Drizzle Kit for migration generation and execution
- Migrations run as a pre-deployment step in CI/CD (before new code deploys)
- All migrations must be backwards-compatible (expand-contract pattern)
- PII vault migrations require separate approval workflow

### 11.5 Feature Flags

Lightweight feature flags via database-backed configuration (no external service for MVP):

- JSON configuration table in core database
- Cached in Redis with 60-second TTL
- Supports: boolean toggles, percentage rollouts, user-segment targeting
- Used for: new matching algorithm variants, payment flow changes, UI experiments

### 11.6 Release Strategy

- Continuous deployment to staging (every merge to main)
- Production releases: manual trigger with approval gate
- Canary deployments: 10% traffic for 30 min → 50% for 15 min → 100%
- Automated rollback on error rate spike (>1% 5xx responses)
- Release notes auto-generated from conventional commits

---

