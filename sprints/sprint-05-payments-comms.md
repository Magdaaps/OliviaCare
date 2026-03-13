# Sprint 5 — Payments, DAC7, Messaging, Notifications
**Duration:** 2 weeks | **Est. effort:** 15 days | **Goal:** Full payment flow with escrow, DAC7 data captured from transaction #1, in-app messaging, email/SMS notifications

## Reference Documents (load before implementing)
- `Architecture/docs/arch-domain-data.md` — section 5.6 (payments domain), 5.7 (tax/DAC7 domain), 5.8 (messaging domain), section 6.4 (payments schema)
- `Architecture/docs/arch-security.md` — section 7.6 (payment security, Stripe Radar, velocity checks), 7.3 (IBAN in PII vault)
- `Architecture/docs/arch-roadmap.md` — ADR-004 (Stripe Connect escrow model), ADR-007 (DAC7 from tx #1)
- `Architecture/docs/arch-stack-workflow.md` — section 9 (Resend, Twilio, Cloud Tasks + Pub/Sub)

## Tasks

| # | Task | Est. Days | Dependency | Branch |
|---|------|-----------|------------|--------|
| 18 | Payment flow: Stripe PaymentIntent, escrow hold, release on booking completion, platform fee deduction, payout trigger | 5 | Sprint 4 #17 | `feature/payment-flow` |
| 19 | DAC7 compliance: collect NIP/PESEL + tax residency at onboarding, track total_consideration per caregiver per year, XML export endpoint | 3 | #18, Sprint 2 #8 | `feature/dac7` |
| 20 | In-app messaging: conversations table, message storage, REST polling API (MVP), basic chat UI | 4 | Sprint 2 done | `feature/messaging` |
| 21 | Notification module: email templates (Resend + React Email), SMS (Twilio), Cloud Tasks async delivery, notification preferences | 3 | Sprint 2 done | `feature/notifications` |

## Definition of Done
- [ ] Family pays → funds held in Stripe escrow → released on booking end → caregiver payout minus platform fee
- [ ] Stripe webhooks handled idempotently (idempotency keys, duplicate event protection)
- [ ] DAC7: NIP/PESEL in PII vault; total_consideration tracked from transaction #1
- [ ] Matched parties can exchange messages via polling API (10s interval)
- [ ] Email/SMS sent on: registration, verification status change, booking request, payment confirmation

## Key Architectural Constraints
- Idempotency keys on ALL Stripe API calls — never process a webhook event twice
- DAC7: tax identifiers in PII vault (AES-256-GCM); total_consideration in core DB (not PII) (ADR-007)
- Cloud Tasks: at-least-once delivery — notification handlers must be idempotent
- No WebSocket for MVP — polling acceptable (section 13, deferred to Phase 2)
