<!-- Source: Caregiver_Platform_IT_Architecture.md | Sections: 5, 6 -->

## 5. Domain Architecture

Each domain module is a self-contained unit with explicit boundaries, its own database schema, and a typed public API. Modules communicate through an in-process event bus using typed events.

| Domain Module | Responsibilities | Key Entities | External Dependencies |
|--------------|-----------------|-------------|----------------------|
| User & Identity | Account creation, authentication, role assignment, profile basics, phone verification, MFA management | User, Role, Session, PhoneVerification | Supabase Auth |
| Caregiver Profile | Professional profile, skills, certifications, availability calendar, language progression tracking, credential wallet | CaregiverProfile, Skill, Certification, Availability, LanguageLevel | None (internal) |
| Family Profile | Care needs (JSONB), senior health context, location, budget ranges, preferences | FamilyProfile, CareNeeds, SeniorProfile, BudgetConfig | None (internal) |
| Matching Engine | Discovery, filtering, scoring, ranking, recommendation signals, standby pool management | MatchRequest, MatchResult, MatchScore, StandbyPool | None (extractable) |
| Scheduling | Booking lifecycle, calendar management, shift management, replacement coordination | Booking, Shift, Schedule, ReplacementRequest | None (internal) |
| Document Verification | Upload pipeline, malware scanning, encrypted storage, verification queue, status tracking, retention policies | Document, VerificationRequest, VerificationResult | Onfido, GCP Cloud Storage |
| Payments | Family billing, escrow management, caregiver payouts, platform fees, refunds, IBAN collection | Transaction, Payout, Escrow, PlatformFee, Refund | Stripe Connect |
| Tax Reporting (DAC7) | Total Consideration tracking, XML/JSON export to KAS, IBAN/NIP/PESEL collection, tax residency management | TaxRecord, DAC7Report, SellerData | KAS (Polish tax authority) |
| Continuity of Care | Breakdown risk prediction, standby pool activation, SLA monitoring, care continuity planning, escalation workflows | ContinuityPlan, RiskFlag, EscalationEvent | None (internal) |
| Messaging | Secure in-app messaging between matched parties, message encryption, moderation hooks | Conversation, Message, MessageAttachment | None (internal) |
| Notifications | Multi-channel delivery (push, email, SMS), template management, delivery tracking, preference management | Notification, Template, DeliveryLog, UserPreference | Resend, Twilio, FCM |
| Admin Operations | Coordinator panel, verification queue management, user management, dispute resolution, platform configuration | AdminAction, DisputeCase, PlatformConfig | None (internal) |
| Audit & Compliance | Immutable event logging, GDPR data requests, data retention enforcement, compliance reporting | AuditEvent, DataRequest, RetentionPolicy | BigQuery (for long-term storage) |

### Module Communication Pattern

Modules communicate exclusively through two mechanisms:

- **Synchronous:** Typed service interfaces (for queries requiring immediate response)
- **Asynchronous:** Domain events via in-process EventEmitter (for notifications, audit logging, analytics)

**Example domain events:**

- `CaregiverVerified` → triggers matching pool update, notification to caregiver
- `BookingCreated` → triggers payment initiation, scheduling update, audit log
- `RiskFlagRaised` → triggers continuity planning, coordinator notification
- `PaymentCompleted` → triggers payout scheduling, DAC7 record update, receipt notification

---

## 6. Data Architecture

### 6.1 Core Entities and Relationships

The data model is split across two PostgreSQL instances for security isolation. The core database handles transactional data; the PII vault stores sensitive personal information with separate access credentials.

#### Core Database Schema

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| users | id (UUID), email, role (enum), status, created_at, last_login | 1:1 caregiver_profiles, 1:1 family_profiles |
| caregiver_profiles | user_id (FK), skills (JSONB), availability (JSONB), location (PostGIS), hourly_rate, retention_score, verification_status | N:M certifications, 1:N bookings |
| family_profiles | user_id (FK), care_needs (JSONB), location (PostGIS), budget_range, care_type (enum) | 1:N care_recipients, 1:N bookings |
| care_recipients | id, family_id (FK), health_context (JSONB — encrypted), mobility_level, dementia_stage | N:1 family_profiles |
| bookings | id, caregiver_id, family_id, status (enum), shift_start, shift_end, rate_agreed | 1:1 transactions, 1:N shifts |
| transactions | id, booking_id (FK), amount, currency, stripe_id, status, platform_fee | 1:1 payouts, 1:1 escrow_records |
| match_results | id, family_id, caregiver_id, score, factors (JSONB), created_at | N:1 families, N:1 caregivers |
| documents | id, user_id, type (enum), storage_path (encrypted), verification_status, expires_at | N:1 users |
| audit_events | id, actor_id, action, resource_type, resource_id, metadata (JSONB), timestamp | Append-only, immutable |
| conversations | id, booking_id, participants (array), created_at | 1:N messages |
| notifications | id, user_id, channel, template_id, status, sent_at | N:1 users |

#### PII Vault Schema (Separate Instance)

| Entity | Fields | Access Control |
|--------|--------|---------------|
| pii_records | user_id (FK), first_name (encrypted), last_name (encrypted), date_of_birth (encrypted), nationality | API-only, no direct DB access from application modules |
| identity_documents | user_id, document_type, document_number (encrypted), issuing_country, expiry_date | Read: verification module only; Write: upload pipeline only |
| tax_identifiers | user_id, nip (encrypted), pesel (encrypted), iban (encrypted), tax_residency | Read: payment + DAC7 modules only |
| address_records | user_id, street (encrypted), city, postal_code, country | Read: matching (city-level only) + admin |

### 6.2 Sensitive Data Handling

- All PII fields use **AES-256-GCM encryption** at the application layer (not just at-rest database encryption)
- Encryption keys managed via **GCP Secret Manager** with automatic rotation every 90 days
- Health-related JSONB fields (Art. 9 RODO) encrypted with **separate key hierarchy**
- **Time-limited access:** caregiver sees senior health context only during active booking, revoked automatically on booking end
- **Tokenization:** internal references use opaque UUIDs; real identifiers resolved only at the PII vault boundary

### 6.3 Audit Logging Model

- Append-only `audit_events` table with **no UPDATE/DELETE permissions** granted to any application role
- Every write operation on sensitive entities generates an audit event with actor, action, resource, and timestamp
- Audit logs replicated to **BigQuery daily** for long-term retention (7 years for financial records per DAC7)
- Tamper detection: **SHA-256 hash chain** linking consecutive audit events

### 6.4 Document Storage Model

- Documents stored in **GCP Cloud Storage** with server-side encryption (CMEK via Cloud KMS)
- Access exclusively via **signed URLs with 15-minute expiry** and single-use tokens
- Malware scanning via **ClamAV in Cloud Run job** before storage
- Retention policies: identity documents retained for duration of account + 2 years; financial documents for 7 years; automatic deletion thereafter

### 6.5 Analytics Separation

Operational and analytical workloads are strictly separated:

- Core PostgreSQL serves only transactional queries (OLTP)
- Nightly ETL pipeline (Cloud Functions) exports anonymized/aggregated data to BigQuery
- Metabase connects only to BigQuery for dashboards and reporting
- No direct analyst access to production database
- **PII never enters the analytics pipeline** — only pseudonymized identifiers and aggregated metrics

---

