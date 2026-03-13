# Sprint 4 — Matching Engine, Discovery UI, Stripe Connect
**Duration:** 2 weeks | **Est. effort:** 13 days | **Goal:** Families can find matched caregivers via scored search, Stripe Connect accounts onboarded

## Reference Documents (load before implementing)
- `Architecture/docs/arch-domain-data.md` — section 5.4 (matching domain), section 6.3 (matching schema, scoring algorithm)
- `Architecture/docs/arch-stack-workflow.md` — section 9 (PostgreSQL pg_trgm, PostGIS, Stripe Connect, Redis)
- `Architecture/docs/arch-roadmap.md` — ADR-004 (Stripe Connect), ADR-011 (Platform Work Directive — caregiver autonomy), ADR-012 (PostgreSQL full-text search for MVP)

## Tasks

| # | Task | Est. Days | Dependency | Branch |
|---|------|-----------|------------|--------|
| 15 | Basic matching engine: geo-filter (PostGIS), availability overlap, skills match, multi-factor scoring (0–100), result caching (Redis 5 min TTL) | 5 | Sprint 3 done | `feature/matching-engine` |
| 16 | Caregiver discovery UI: search page, filters (location, skills, availability, price), profile cards, pagination, match score display | 4 | #9, #15 | `feature/discovery-ui` |
| 17 | Stripe Connect integration: caregiver account onboarding flow, IBAN collection → PII vault, account status webhooks | 4 | Sprint 2 #7 | `feature/stripe-connect` |

## Definition of Done
- [ ] Matching returns ranked list in <500ms for 1,000 profiles
- [ ] Geo-filter uses PostGIS ST_DWithin (not in-app distance calculation)
- [ ] Match results cached in Redis; cache invalidated on profile update
- [ ] Caregiver can complete Stripe Connect onboarding from profile page
- [ ] IBAN stored encrypted in PII vault, not core DB
- [ ] Discovery UI shows filters + sorted results + profile preview card

## Key Architectural Constraints
- Matching: presents options, caregivers choose freely — NO algorithmic penalties for declining (ADR-011, EU Platform Work Directive)
- Scoring must be deterministic and auditable (log score components)
- Search: PostgreSQL pg_trgm + full-text for MVP; migrate to Typesense at >10K profiles (ADR-012)
- Stripe Connect Standard accounts: caregivers handle their own KYC with Stripe (ADR-004)
