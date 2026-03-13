# Sprint 8 — Security Hardening, Performance, Production Launch
**Duration:** 2 weeks | **Est. effort:** 8 days | **Goal:** Platform hardened and security-audited, production infrastructure live, canary deployment tested, runbooks complete

## Reference Documents (load before implementing)
- `Architecture/docs/arch-security.md` — section 17 (full security checklist — all 17 items must be ✅ before launch)
- `Architecture/docs/arch-infrastructure.md` — section 8.1 (Cloud Run prod config), 8.2 (env separation), 8.3 (network architecture), 12.4 (alerting rules)
- `Architecture/docs/arch-roadmap.md` — ADR-003 (GCP prod setup), ADR-009 (encryption verification), ADR-010 (canary deployment)
- `Architecture/docs/arch-stack-workflow.md` — section 11.6 (release strategy: canary 10%→50%→100%, automated rollback)

## Tasks

| # | Task | Est. Days | Dependency | Branch |
|---|------|-----------|------------|--------|
| 28 | Security hardening: SAST (Semgrep) in CI, npm audit gate, Trivy container scan, secrets rotation verification, pen-test prep checklist | 3 | All sprints done | `feature/security-hardening` |
| 29 | Performance: EXPLAIN ANALYZE on matching + search queries, composite index tuning, Redis caching audit, Cloud Run concurrency config | 2 | Sprint 4 #15, Sprint 6 #22 | `feature/performance` |
| 30 | Production deployment: Terraform apply (prod env), DNS + SSL (Cloud Load Balancer managed certs), Secret Manager prod secrets, PagerDuty alerts, canary deploy, operational runbooks | 3 | All | `feature/production-deploy` |

## Definition of Done
- [ ] All 17 items in section 17 security checklist: ✅
- [ ] SAST + npm audit + Trivy all pass in CI pipeline
- [ ] Matching endpoint: p95 <500ms under 100 concurrent simulated users
- [ ] Production Cloud Run: min-instances=1 (no cold starts for users)
- [ ] PagerDuty alerts active: API error rate, payment failures, DB pool >80%, cert expiry <14 days
- [ ] Canary deploy tested: 10%→50%→100% with automated rollback at >1% 5xx
- [ ] Runbooks written: incident response, DB failover, payment dispute, GDPR erasure request

## Key Architectural Constraints
- Canary: automated rollback if error rate >1% 5xx for 5 min (section 11.6)
- Production secrets: Secret Manager only — never in env vars, never in code (ADR-009)
- SSL: auto-renewed via Cloud Load Balancer managed certificates
- Production: min-instances=1 vs staging: min-instances=0 (cost vs UX tradeoff — section 8.2)
- Cloud SQL prod: private IP only, VPC connector (section 8.3)
