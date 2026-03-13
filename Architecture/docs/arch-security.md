<!-- Source: Caregiver_Platform_IT_Architecture.md | Sections: 7, 17 -->

## 7. Security Architecture

### 7.1 Authentication

- **Supabase Auth** as centralized identity provider — no custom auth implementation
- JWT tokens with short expiry (15 min access, 7 day refresh)
- Mandatory phone verification for all accounts (SMS OTP via Twilio)
- Optional MFA via TOTP (recommended for caregivers with verified status)
- Social login: Google, Apple (optional convenience; phone verification still required)
- Session management: single active session by default; concurrent session requires explicit approval

### 7.2 Authorization (RBAC + ABAC Hybrid)

| Role | Base Permissions | Attribute-Based Restrictions |
|------|-----------------|------------------------------|
| family | View caregiver profiles, create bookings, manage own profile, send messages | Can only view caregivers in matched/active state; budget-filtered results |
| caregiver | Manage own profile, respond to bookings, view active client info, upload documents | Health context visible only during active booking; location shared at city level until matched |
| coordinator | View verification queue, manage disputes, view reports, moderate content | Cannot access PII vault directly; actions logged with justification field |
| admin | Full platform configuration, user management, financial overview | JIT access with time-limited elevation; all actions require MFA re-authentication |
| system | Automated processes: matching, notifications, ETL, audit | Service accounts with narrowest possible IAM roles; no interactive login |

### 7.3 PII Segregation

- Separate PostgreSQL instance for PII vault with **distinct Cloud SQL credentials**
- Application accesses PII vault through a dedicated `PiiService` with rate-limited, audited methods
- **No JOINs possible** between core DB and PII vault (enforced by separate instances)
- PII resolution happens at the API response boundary — internal modules work with tokenized references

### 7.4 Document Security

- **Upload pipeline:** client → signed upload URL → Cloud Storage → ClamAV scan → move to verified bucket
- **Storage:** CMEK encryption via Cloud KMS, bucket-level access control, no public access
- **Access:** signed URLs with 15-min expiry, generated on-demand with audit log entry
- **Retention:** automated lifecycle policies delete expired documents; manual deletion blocked

### 7.5 Secrets Management

- All secrets stored in **GCP Secret Manager** (API keys, DB credentials, encryption keys)
- Application reads secrets at startup via IAM-authenticated API calls
- **No secrets in environment variables, code, or configuration files**
- Automatic key rotation: encryption keys every 90 days, API keys every 180 days
- Separate secret namespaces per environment (dev/staging/prod)

### 7.6 API Security

- **GCP Cloud Armor WAF** in front of all public endpoints
- Rate limiting: 100 req/min per authenticated user, 20 req/min for unauthenticated, 5 req/min for auth endpoints
- Input validation: **Zod schemas** on all API endpoints with strict type checking
- CORS: allowlist of known frontend origins only
- CSRF: SameSite cookie policy + CSRF tokens for state-changing operations
- Request size limits: 10MB for document uploads, 1MB for all other endpoints

### 7.7 Bot Protection

- reCAPTCHA v3 on registration and login flows
- Device fingerprinting for fraud signal collection
- Automated blocking of known bot user agents and suspicious patterns
- Honeypot fields on registration forms

### 7.8 Admin Access Security

- **Just-in-time (JIT)** privilege elevation with configurable time window (default: 4 hours)
- All admin actions require re-authentication via MFA
- Admin action log with **mandatory justification field**
- Quarterly access reviews with automatic deprovisioning of unused admin accounts
- Separate admin authentication flow with IP allowlisting for sensitive operations

### 7.9 Threat Model

| Threat | Impact | Likelihood | Mitigation |
|--------|--------|------------|------------|
| Fake caregiver accounts | High — safety risk to families | High | Mandatory KYC (Onfido), phone verification, document verification pipeline, manual review queue |
| Identity theft / impersonation | Critical — regulatory + safety | Medium | Multi-factor verification, liveness detection (Onfido), background check integration |
| Document forgery | High — trust undermined | Medium | Onfido automated document verification, checksum validation, manual review for flagged docs |
| Unauthorized PII access | Critical — GDPR breach | Low | Separate PII vault, application-layer encryption, audit logging, least-privilege IAM |
| Caregiver database scraping | High — competitive risk | Medium | Rate limiting, bot protection, no bulk export APIs, pagination limits |
| Payment fraud | High — financial loss | Medium | Stripe Radar, escrow model, velocity checks, manual review for unusual patterns |
| Insider threat (admin abuse) | High — data breach | Low | JIT access, MFA, audit logging, justification fields, quarterly reviews |
| DDoS attack | Medium — availability | Medium | Cloud Armor DDoS protection, Cloud Run auto-scaling, rate limiting |
| Data leak through logs | High — GDPR breach | Medium | PII scrubbing in log pipeline, structured logging with field classification, no sensitive data in error messages |
| Session hijacking | Medium — account takeover | Low | Short JWT expiry, secure cookie flags, session binding to device fingerprint |

---

## 17. Security Checklist for Launch Readiness

All items must be verified before production launch. Each item requires sign-off from the responsible engineer.

### Authentication & Access Control

| # | Check | Status | Owner |
|---|-------|--------|-------|
| 1.1 | Supabase Auth configured with phone verification mandatory | ☐ | |
| 1.2 | JWT token expiry set to 15 min (access) / 7 days (refresh) | ☐ | |
| 1.3 | MFA available and recommended for caregiver accounts | ☐ | |
| 1.4 | RBAC roles defined and enforced on all API endpoints | ☐ | |
| 1.5 | Admin accounts require MFA re-authentication for sensitive actions | ☐ | |
| 1.6 | JIT privilege elevation configured for admin roles (4h window) | ☐ | |
| 1.7 | Session management: concurrent session detection implemented | ☐ | |

### Data Protection

| # | Check | Status | Owner |
|---|-------|--------|-------|
| 2.1 | PII vault on separate Cloud SQL instance with distinct credentials | ☐ | |
| 2.2 | Application-layer AES-256-GCM encryption on all PII fields | ☐ | |
| 2.3 | Health data (Art. 9 RODO) encrypted with separate key hierarchy | ☐ | |
| 2.4 | Encryption keys in Secret Manager with 90-day auto-rotation | ☐ | |
| 2.5 | No PII in application logs (scrubbing middleware verified) | ☐ | |
| 2.6 | Database backups encrypted and access-restricted | ☐ | |
| 2.7 | GDPR data export and deletion endpoints functional | ☐ | |

### Document Security

| # | Check | Status | Owner |
|---|-------|--------|-------|
| 3.1 | Documents stored in encrypted Cloud Storage (CMEK) | ☐ | |
| 3.2 | Signed URLs with 15-min expiry for document access | ☐ | |
| 3.3 | Malware scanning (ClamAV) on all uploads before storage | ☐ | |
| 3.4 | Document retention policies configured and automated | ☐ | |
| 3.5 | No public access to document storage buckets | ☐ | |

### API & Network Security

| # | Check | Status | Owner |
|---|-------|--------|-------|
| 4.1 | Cloud Armor WAF deployed with managed rule sets | ☐ | |
| 4.2 | Rate limiting configured (100/min auth, 20/min unauth, 5/min login) | ☐ | |
| 4.3 | CORS allowlist restricted to known frontend origins | ☐ | |
| 4.4 | CSRF protection on all state-changing endpoints | ☐ | |
| 4.5 | Input validation (Zod) on all API endpoints | ☐ | |
| 4.6 | Request size limits enforced (10MB uploads, 1MB general) | ☐ | |
| 4.7 | reCAPTCHA v3 on registration and login | ☐ | |
| 4.8 | Database instances have no public IP | ☐ | |
| 4.9 | TLS 1.3 enforced on all endpoints | ☐ | |

### Payment Security

| # | Check | Status | Owner |
|---|-------|--------|-------|
| 5.1 | Stripe webhook signatures verified on all payment callbacks | ☐ | |
| 5.2 | No card data stored on platform (Stripe handles PCI DSS) | ☐ | |
| 5.3 | Escrow flow tested: payment → hold → payout → refund | ☐ | |
| 5.4 | DAC7 data collection verified for all caregiver payouts | ☐ | |
| 5.5 | Velocity checks on payment amounts and frequency | ☐ | |

### Monitoring & Incident Response

| # | Check | Status | Owner |
|---|-------|--------|-------|
| 6.1 | Structured logging deployed with correlation IDs | ☐ | |
| 6.2 | Audit logging covers all critical operations (immutable, append-only) | ☐ | |
| 6.3 | Alerting configured for error rate, payment failures, security events | ☐ | |
| 6.4 | Incident response runbook documented | ☐ | |
| 6.5 | Automated rollback configured for deployment failures | ☐ | |
| 6.6 | Database connection pool monitoring with alerts at 80% utilization | ☐ | |

### Compliance

| # | Check | Status | Owner |
|---|-------|--------|-------|
| 7.1 | Anti-control measures verified (no algorithmic penalties, free choice) | ☐ | |
| 7.2 | DAC7 XML export pipeline tested with sample data | ☐ | |
| 7.3 | RODO privacy policy published and consent flow implemented | ☐ | |
| 7.4 | Data Processing Agreement (DPA) in place with all sub-processors | ☐ | |
| 7.5 | Cookie consent banner with granular controls | ☐ | |
| 7.6 | Right to data portability endpoint returns machine-readable format | ☐ | |
| 7.7 | Audit log retention meets DAC7 requirement (7 years for financial) | ☐ | |

---

*End of Document — Caregiver Matching Platform IT Architecture v1.0*

