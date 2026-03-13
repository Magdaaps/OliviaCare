# Sprint 3 — Caregiver & Family Profiles, Document Pipeline
**Duration:** 2 weeks | **Est. effort:** 15 days | **Goal:** Complete profile management for both user types, document upload with virus scanning, admin verification queue, immutable audit log

## Reference Documents (load before implementing)
- `Architecture/docs/arch-domain-data.md` — section 5.2 (caregiver domain), 5.3 (family domain), 5.5 (document domain), section 6 (full data schema — JSONB fields, PII vault schema)
- `Architecture/docs/arch-security.md` — section 7.3 (document storage, signed URLs), 7.5 (threat model — document fraud)
- `Architecture/docs/arch-roadmap.md` — ADR-006 (JSONB for care needs), ADR-002 (PII separation)

## Tasks

| # | Task | Est. Days | Dependency | Branch |
|---|------|-----------|------------|--------|
| 10 | Caregiver profile module: skills[], availability (JSONB), care capabilities (JSONB), hourly rate, location (PostGIS), verification status | 3 | Sprint 2 done | `feature/caregiver-profile` |
| 11 | Family profile module: care needs form (JSONB), health context → PII vault, budget range, location, preferred schedule | 3 | #7, #8 | `feature/family-profile` |
| 12 | Document upload pipeline: signed GCS URLs, upload handler, ClamAV async scanning, status tracking | 4 | #8 | `feature/document-upload` |
| 13 | Document verification queue: admin review UI, approve/reject/request-more actions, status webhooks | 3 | #12 | `feature/document-queue` |
| 14 | Audit logging module: EventEmitter-based, append-only audit_log table, correlation IDs, actor + action + resource + before/after diff | 2 | Sprint 1 done | `feature/audit-module` |

## Definition of Done
- [ ] Caregiver completes profile with all JSONB fields validated by Zod
- [ ] Family care needs stored with health context encrypted in PII vault
- [ ] Document upload: signed URL → client uploads to GCS → ClamAV scans async → status updated
- [ ] Admin can review, approve, reject documents with reason logged
- [ ] Every state-changing operation emits audit event

## Key Architectural Constraints
- JSONB fields: always validated via Zod schema at service layer — database does not enforce (ADR-006)
- Documents: signed URLs expire in 15 min; ClamAV must complete before status = 'verified'
- Health context (dementia stage, diagnoses): PII vault only, never in core DB (ADR-002)
- Audit log: immutable — no UPDATE/DELETE allowed, append-only
