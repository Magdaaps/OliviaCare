# Sprint 7 — E2E Tests, Observability, GDPR
**Duration:** 2 weeks | **Est. effort:** 10 days | **Goal:** Critical flows covered by E2E tests, Grafana dashboards operational, GDPR data controls implemented

## Reference Documents (load before implementing)
- `Architecture/docs/arch-infrastructure.md` — section 12 (full observability: logging, metrics, tracing, alerting, dashboards)
- `Architecture/docs/arch-security.md` — section 7.7 (GDPR: data export, deletion, consent, retention policies), section 7.3 (PII logging prohibition)
- `Architecture/docs/arch-stack-workflow.md` — section 11.3 (testing strategy: Playwright E2E, Testcontainers integration, Vitest/Jest unit), section 11.1 (CI stages)

## Tasks

| # | Task | Est. Days | Dependency | Branch |
|---|------|-----------|------------|--------|
| 25 | E2E test suite (Playwright): registration, verification, matching, booking, payment flows; CI integration on PR to main | 4 | Sprint 6 done | `feature/e2e-tests` |
| 26 | Observability: pino structured logging + PII scrubber middleware, OpenTelemetry tracing, custom business counters, Grafana Cloud dashboards | 3 | Sprint 1 done | `feature/observability` |
| 27 | GDPR: data export endpoint (full JSON package, 30-day SLA), right-to-erasure workflow (soft delete + PII vault purge), consent management, retention cron jobs | 3 | Sprint 2 done | `feature/gdpr` |

## Definition of Done
- [ ] Playwright E2E covers all 5 critical user flows and runs in CI
- [ ] PII scrubber removes email, phone, PESEL, IBAN from all log lines — verified by test
- [ ] Correlation ID propagates: HTTP request → DB queries → Cloud Tasks jobs
- [ ] Grafana: Platform Health + Business KPIs dashboards live with real data
- [ ] Data export returns complete user data package (JSON)
- [ ] Erasure: account soft-deleted, PII vault purged, audit log retained (legal obligation)

## Key Architectural Constraints
- Logging: ZERO PII in logs ever — PII scrubber is middleware, not opt-in (section 12.1)
- Audit log must NOT be deleted even on erasure request — legal retention requirement
- E2E tests: run against staging with real DB (Testcontainers for integration; ADR forbids mock DB)
- GDPR Art. 17 erasure: 30-day window; financial records retained 5 years per DAC7 (ADR-007)
