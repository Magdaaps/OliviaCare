# Sprint 6 — Booking Flow, Admin Panel, Rate Limiting
**Duration:** 2 weeks | **Est. effort:** 11 days | **Goal:** End-to-end booking lifecycle, admin operations panel, WAF + rate limiting in place

## Reference Documents (load before implementing)
- `Architecture/docs/arch-domain-data.md` — section 5.1 (booking/scheduling domain), section 6.5 (bookings schema, state machine)
- `Architecture/docs/arch-security.md` — section 7.2 (RBAC — coordinator role permissions), 7.4 (rate limiting), 7.5 (threat model)
- `Architecture/docs/arch-infrastructure.md` — section 8.1 (Cloud Armor WAF config)
- `Architecture/docs/arch-roadmap.md` — ADR-010 (feature flags), ADR-011 (caregiver autonomy — no decline penalties)

## Tasks

| # | Task | Est. Days | Dependency | Branch |
|---|------|-----------|------------|--------|
| 22 | Booking flow: request → accept/decline → scheduled → active → completed/cancelled, shift management, event-sourced status history | 5 | Sprint 4 #15, Sprint 5 #18 | `feature/booking-flow` |
| 23 | Admin panel: user management (activate/deactivate/ban + justification), verification queue integration, analytics dashboard (registrations, bookings, GMV) | 4 | Sprint 2 #9, Sprint 3 #13 | `feature/admin-panel` |
| 24 | Rate limiting (NestJS ThrottlerModule + Redis store), Cloud Armor OWASP managed ruleset, auth endpoint brute-force protection | 2 | Sprint 1 done | `feature/security-rate-limiting` |

## Definition of Done
- [ ] Full booking state machine with all transitions covered by unit tests
- [ ] Caregiver can decline freely — no penalty fields stored (ADR-011)
- [ ] Admin can manage users with justification logged to audit trail
- [ ] Rate limiting: 100 req/min per IP; 5 failed logins → 15-min lockout
- [ ] Cloud Armor OWASP managed ruleset active in staging environment

## Key Architectural Constraints
- Booking state: event-sourced (append-only history), not direct column UPDATE
- Admin actions: justification field required; logged with coordinator ID (section 7.2)
- RBAC: coordinator role cannot access PII vault directly (section 7.2)
- Feature flags (DB-backed, Redis 60s TTL) used to gate booking flow changes (ADR-010)
