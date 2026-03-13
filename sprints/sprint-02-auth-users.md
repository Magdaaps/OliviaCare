# Sprint 2 — Authentication & User Core
**Duration:** 2 weeks | **Est. effort:** 11 days | **Goal:** Full auth flow (register, login, phone verification, RBAC), PII vault operational, Next.js shell with auth context

## Reference Documents (load before implementing)
- `Architecture/docs/arch-security.md` — sections 7.1 (auth), 7.2 (RBAC/ABAC), 7.3 (PII encryption), 7.4 (audit)
- `Architecture/docs/arch-stack-workflow.md` — section 9 (Supabase Auth, Next.js, Zod)
- `Architecture/docs/arch-domain-data.md` — section 5 (user domain), section 6.1–6.2 (core schema, PII vault schema)
- `Architecture/docs/arch-roadmap.md` — ADR-002 (PII vault), ADR-005 (Supabase Auth), ADR-009 (app-layer encryption)

## Tasks

| # | Task | Est. Days | Dependency | Branch |
|---|------|-----------|------------|--------|
| 6 | Supabase Auth integration: JWT guard, phone OTP, social login hooks, token refresh | 3 | Sprint 1 done | `feature/supabase-auth` |
| 7 | User module: UserService, UserRepository, role assignment (family/caregiver/coordinator/admin), profile basics | 2 | #5, #6 | `feature/user-module` |
| 8 | PII vault service: AES-256-GCM encrypt/decrypt, tokenized access, PiiService with audited methods, rate limiting | 3 | #5 | `feature/pii-vault` |
| 9 | Next.js frontend scaffold: App Router layout, auth context (Supabase JS), shadcn/ui setup, Tailwind, protected routes | 3 | #6 | `feature/nextjs-scaffold` |

## Definition of Done
- [ ] User can register with email + phone OTP verification
- [ ] JWT guard protects all API routes correctly
- [ ] RBAC: family, caregiver, coordinator, admin roles enforced
- [ ] PII stored encrypted (AES-256-GCM), only accessible via PiiService
- [ ] PII access logged to audit trail with correlation ID
- [ ] Next.js renders protected dashboard for logged-in users

## Key Architectural Constraints
- Phone verification mandatory for all accounts — not optional (section 7.1)
- PII fields: never in core DB, never in logs, never in error messages (ADR-002)
- AES-256-GCM with keys from GCP Secret Manager, 90-day rotation (ADR-009)
- Access tokens: 15 min expiry; refresh tokens: 7 days (section 7.1)
