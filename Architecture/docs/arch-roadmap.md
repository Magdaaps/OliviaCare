<!-- Source: Caregiver_Platform_IT_Architecture.md | Sections: 13, 14, 15, 16 -->

## 13. Cost-Optimized MVP

### MVP Scope (Poland — Warsaw, Kraków, Wrocław)

The MVP focuses on proving core marketplace mechanics with minimal infrastructure spend. Target: **<€150/month** for all cloud infrastructure.

#### What to Build

- Caregiver registration + document upload + basic verification (manual review)
- Family registration + care needs form (JSONB-backed)
- Basic matching: geolocation + availability + skills filtering with simple scoring
- In-app messaging between matched parties
- Stripe Connect integration: family payment → escrow → caregiver payout
- DAC7 data collection from first transaction (IBAN, tax ID, total consideration)
- Admin panel: verification queue, user management, basic analytics

#### What to Defer

- Advanced matching algorithm (ML-based scoring) → Phase 2
- Continuity of care automation (standby pool, risk prediction) → Phase 2
- PD A1 form automation → Phase 2 (DE/AT expansion)
- Multi-language support (beyond PL) → Phase 2
- Mobile native apps → Phase 2 (PWA sufficient for MVP)
- Real-time notifications (WebSocket) → Phase 2 (polling sufficient for MVP)

#### MVP Infrastructure Cost Breakdown

| Service | MVP Configuration | Monthly Cost |
|---------|------------------|-------------|
| Cloud Run | 1 service, 0–2 instances, 1 vCPU / 512MB | €0–€15 |
| Cloud SQL (core) | db-f1-micro, 10GB SSD | €15 |
| Cloud SQL (PII vault) | db-f1-micro, 5GB SSD | €15 |
| Memorystore Redis | Basic 1GB | €30 |
| Cloud Storage | Standard, ~5GB | €1 |
| Cloud Tasks + Pub/Sub | Low volume | €2 |
| Cloud Armor | 1 policy | €5 |
| Secret Manager + KMS | Low usage | €2 |
| Vercel (frontend) | Hobby plan (upgrade to Pro at scale) | €0 |
| External SaaS | Stripe (pay-per-tx), Onfido (pay-per-check), Resend (free tier), Twilio (pay-per-SMS) | €20–€50 |

**Total estimated MVP cost: €90–€150/month (infrastructure) + variable SaaS usage**

#### Cost Optimization Tactics

- Cloud Run minimum instances = 0 (cold starts acceptable for MVP, ~2s)
- Cloud SQL shared-core instances (db-f1-micro) — upgrade to dedicated when >100 concurrent users
- Vercel free tier for frontend hosting (generous for SSR)
- Metabase self-hosted on Cloud Run (not a paid BI tool)
- Redis on Memorystore Basic tier (no replication — acceptable for MVP)
- BigQuery on-demand pricing (first 1TB/month free)

---

## 14. Scaling Roadmap

### Phase 1: MVP — Poland Metropolitan (Months 1–6)

| Area | Configuration | Scaling Trigger |
|------|--------------|----------------|
| Users | 100–1,000 caregivers, 200–2,000 families | N/A (launch) |
| Compute | Cloud Run: 0–2 instances | Upgrade to min 1 instance when p95 latency >3s |
| Database | db-f1-micro (shared core) | Upgrade to db-custom-1-3840 at >1,000 active users |
| Matching | In-process module, PostgreSQL queries | Monitor query time; optimize indexes |
| Search | PostgreSQL full-text + pg_trgm | Sufficient for <10,000 profiles |
| Payments | Stripe Connect Standard | No scaling needed |

### Phase 2: National Scale — All Poland + DE/AT Corridor (Months 6–18)

| Area | Upgrade | Reason |
|------|---------|--------|
| Users | 5,000–20,000 caregivers, 10,000–50,000 families | National expansion + cross-border |
| Compute | Cloud Run: min 2, max 10 instances, 2 vCPU / 1GB | Consistent latency requirements |
| Database | db-custom-2-7680, read replica for analytics | Write volume + analytics separation |
| Matching | Extract to separate Cloud Run service | Independent scaling, ML model integration |
| Search | Typesense cluster (3 nodes) | Fast faceted search at scale, geo-radius queries |
| Cache | Redis Standard tier (HA with replica) | Uptime requirements for production traffic |
| Notifications | WebSocket (Socket.io on Cloud Run) | Real-time matching alerts, chat |
| Localization | i18next with PL, DE, EN | German market entry |
| Compliance | PD A1 automation, Austrian 24h system integration | Cross-border regulatory requirements |

### Phase 3: EU Expansion (Months 18–36)

| Area | Upgrade | Reason |
|------|---------|--------|
| Users | 50,000+ caregivers, 100,000+ families across EU | Multi-country operations |
| Architecture | Service mesh: matching, payments, notifications as independent services | Team size >8, independent deployment needs |
| Database | Regional read replicas (Frankfurt, Vienna) | Latency for DE/AT users |
| Multi-tenancy | Country-specific configuration, tax rules, compliance modules | Regulatory divergence across EU |
| Mobile | React Native apps (shared codebase) | User expectation in mature markets |
| ML | Dedicated matching ML pipeline (Vertex AI) | Behavioral scoring, risk prediction at scale |
| Observability | Full Grafana Enterprise, distributed tracing across services | Operational complexity of multi-service architecture |

---

## 15. Technical Implementation Roadmap

First 30 engineering tasks, ordered by priority and dependency:

| # | Task | Sprint | Dependency | Est. Days |
|---|------|--------|------------|-----------|
| 1 | Repository setup: Turborepo, TypeScript, ESLint, CI pipeline | 1 | None | 2 |
| 2 | Terraform: GCP project, Cloud SQL, Cloud Storage, Secret Manager, VPC | 1 | None | 3 |
| 3 | Docker Compose local dev environment (PostgreSQL, Redis) | 1 | #1 | 1 |
| 4 | NestJS project scaffold with module structure | 1 | #1 | 2 |
| 5 | Drizzle ORM setup: core schema + PII vault schema + migrations | 1 | #4 | 3 |
| 6 | Supabase Auth integration: registration, login, phone verification | 2 | #4 | 3 |
| 7 | User module: CRUD, role assignment, profile basics | 2 | #5, #6 | 2 |
| 8 | PII vault service: encrypted storage, tokenized access | 2 | #5 | 3 |
| 9 | Next.js frontend scaffold: routing, auth context, UI component library | 2 | #6 | 3 |
| 10 | Caregiver profile module: skills, availability, JSONB care capabilities | 3 | #7 | 3 |
| 11 | Family profile module: care needs form, JSONB health context, budget | 3 | #7, #8 | 3 |
| 12 | Document upload pipeline: signed URLs, Cloud Storage, ClamAV scanning | 3 | #8 | 4 |
| 13 | Document verification queue: admin review UI, status management | 3 | #12 | 3 |
| 14 | Audit logging module: event capture, immutable append, correlation IDs | 3 | #4 | 2 |
| 15 | Basic matching engine: geo-filter, availability, skills, scoring algorithm | 4 | #10, #11 | 5 |
| 16 | Caregiver discovery UI: search, filters, profile cards, pagination | 4 | #9, #15 | 4 |
| 17 | Stripe Connect integration: account onboarding, IBAN collection | 4 | #7 | 4 |
| 18 | Payment flow: family payment, escrow, caregiver payout, platform fee | 5 | #17 | 5 |
| 19 | DAC7 data collection: tax identifiers, total consideration tracking | 5 | #18, #8 | 3 |
| 20 | In-app messaging: conversations, message storage, basic UI | 5 | #7 | 4 |
| 21 | Notification module: email templates (Resend), SMS (Twilio) | 5 | #7 | 3 |
| 22 | Booking flow: request, accept, schedule, shift management | 6 | #15, #18 | 5 |
| 23 | Admin panel: user management, verification queue, basic dashboard | 6 | #9, #13 | 4 |
| 24 | Rate limiting + Cloud Armor WAF configuration | 6 | #4 | 2 |
| 25 | E2E test suite: registration, matching, booking, payment flows | 7 | #22 | 4 |
| 26 | Observability: structured logging, metrics, Grafana dashboards | 7 | #4 | 3 |
| 27 | GDPR: data export endpoint, data deletion workflow, consent management | 7 | #7, #8 | 3 |
| 28 | Security hardening: SAST pipeline, dependency audit, penetration test prep | 8 | All | 3 |
| 29 | Performance optimization: query analysis, index tuning, caching strategy | 8 | #15, #22 | 2 |
| 30 | Production deployment: Terraform apply, DNS, SSL, monitoring, runbooks | 8 | All | 3 |

**Estimated total: 8 sprints (≈16 weeks) for a team of 3–4 engineers**

---

## 16. Architecture Decision Records (ADR)

### ADR-001: Use Modular Monolith as Initial Architecture Style

**Context:** Team size is 3–5 engineers. Microservices require dedicated platform engineering effort. The platform handles sensitive data requiring domain isolation.

**Decision:** Deploy a single NestJS application with strict module boundaries, typed inter-module APIs, and separate database schemas for sensitive data domains.

**Consequences:** Lower operational overhead. Risk of module coupling if boundaries are not enforced. Clear extraction path when scaling triggers are met.

---

### ADR-002: Separate PII Vault from Core Database

**Context:** Platform handles GDPR Art. 9 health data, identity documents, and financial identifiers. A single database creates risk of accidental PII exposure through queries, logs, or backups.

**Decision:** Use a physically separate Cloud SQL PostgreSQL instance for all PII. Application accesses PII only through a dedicated PiiService with audited, rate-limited methods.

**Consequences:** No accidental JOINs with PII. Higher infrastructure cost (€15/month). Slightly more complex data access patterns. Stronger GDPR compliance posture.

---

### ADR-003: Choose GCP over AWS and Azure

**Context:** Need EU data residency (Warsaw region), startup-friendly pricing, managed services for small team, and strong analytics stack (BigQuery).

**Decision:** Deploy all infrastructure on GCP europe-central2 (Warsaw). Use Cloud Run, Cloud SQL, Cloud Storage, and BigQuery as primary services.

**Consequences:** Good startup credits. Fewer managed service options than AWS. Warsaw region ensures Polish data residency. BigQuery provides cost-effective analytics separation.

---

### ADR-004: Use Stripe Connect for Marketplace Payments

**Context:** Platform requires split payments (family → platform fee + caregiver payout), escrow, KYC, IBAN collection, and DAC7-compatible reporting across EU.

**Decision:** Integrate Stripe Connect (Standard accounts) for all payment processing. Caregivers onboard directly with Stripe for KYC.

**Consequences:** Faster time-to-market. Higher per-transaction cost than custom solution. Built-in KYC eliminates need for separate KYC provider for payment-related verification. DAC7 data collection simplified.

---

### ADR-005: Use Supabase Auth Instead of Custom Authentication

**Context:** Authentication is a high-risk component. Custom implementations introduce security vulnerabilities. Team should focus on business logic, not auth infrastructure.

**Decision:** Use Supabase Auth (self-hostable, OIDC-compliant) for all authentication: passwords, social login, phone verification, MFA.

**Consequences:** Reduced security risk. Dependency on Supabase (mitigated by self-hosting option). Standard JWT-based flow integrates cleanly with NestJS guards.

---

### ADR-006: Use JSONB for Flexible Care Needs and Health Context

**Context:** Care needs vary significantly across countries and cases (dementia stages, mobility levels, incontinence care, etc.). Rigid schema would require frequent migrations as platform expands to new markets.

**Decision:** Store care_needs and health_context as JSONB columns in PostgreSQL with Zod validation at the application layer.

**Consequences:** Flexible schema evolution without migrations. GIN indexes on JSONB for query performance. Validation enforced at application layer (not database). Risk: schema drift if validation is not strictly maintained.

---

### ADR-007: DAC7 Compliance from First Transaction

**Context:** EU DAC7 directive requires platforms to collect and report seller (caregiver) tax data. Retrofitting this after launch creates massive technical debt and compliance risk.

**Decision:** Collect IBAN, NIP/PESEL, tax residency, and track Total Consideration from the first transaction. Build automated XML export pipeline for KAS.

**Consequences:** Higher initial development cost. Zero technical debt for tax compliance. Platform can operate legally from day one. Enables EU expansion without compliance rearchitecture.

---

### ADR-008: Use Drizzle ORM over Prisma

**Context:** ORM must support PostgreSQL JSONB queries, PostGIS extensions, multiple schema connections (core + PII vault), and generate type-safe queries.

**Decision:** Use Drizzle ORM for all database access. Drizzle provides SQL-like query builder with full TypeScript inference, better performance than Prisma for complex queries, and native JSONB support.

**Consequences:** Steeper learning curve than Prisma. Better query control and performance. Excellent PostgreSQL extension support. Smaller community than Prisma but growing rapidly.

---

### ADR-009: Application-Layer Encryption for PII (Not Just At-Rest)

**Context:** Database-level encryption at rest protects against physical disk theft but not against SQL injection, backup exposure, or unauthorized DB access.

**Decision:** Implement AES-256-GCM encryption at the application layer for all PII fields. Encryption keys managed via GCP Secret Manager with 90-day rotation.

**Consequences:** PII unreadable even if database is compromised. Performance overhead for encrypt/decrypt operations (acceptable for PII access patterns). Key management complexity. Cannot use database-level search on encrypted fields (use tokenized indexes).

---

### ADR-010: Trunk-Based Development with Feature Flags

**Context:** Small team needs fast iteration without merge conflicts. Long-lived branches create integration pain. Feature flags enable progressive rollout.

**Decision:** Use trunk-based development (short-lived feature branches, squash merge to main). Database-backed feature flags for progressive rollout. No develop or release branches.

**Consequences:** Faster delivery cycle. Requires discipline in keeping main always deployable. Feature flags add small complexity but enable safe rollouts and A/B testing.

---

### ADR-011: Anti-Control Measures for Platform Work Directive Compliance

**Context:** EU Platform Work Directive creates presumption of employment relationship if platform controls work conditions. System must not algorithmically penalize or unilaterally assign work.

**Decision:** Matching engine presents ranked options; caregivers choose freely. No penalties for declining bookings. No algorithmic schedule enforcement. System logs prove caregiver autonomy.

**Consequences:** Slightly lower matching efficiency (caregivers may decline). Platform avoids employment relationship classification. Audit trail proves compliance with directive.

---

### ADR-012: PostgreSQL Full-Text Search for MVP (Defer Typesense)

**Context:** Search is critical for caregiver discovery but dedicated search engines add operational complexity. PostgreSQL offers pg_trgm and full-text search capabilities.

**Decision:** Use PostgreSQL full-text search with pg_trgm extension for MVP. Migrate to Typesense when profile count exceeds 10,000 or search latency exceeds 200ms p95.

**Consequences:** Zero additional infrastructure for MVP. Adequate performance for initial scale. Clear migration trigger defined. Typesense migration is straightforward (index from PostgreSQL).

---

