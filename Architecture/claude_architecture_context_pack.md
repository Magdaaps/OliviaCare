# ARCHITECTURE CONTEXT PACK

## Digital Caregiver Marketplace (Poland → EU)

Purpose: Provide constraints and design context for system architecture.

You are designing the architecture for a **digital marketplace
connecting caregivers with families needing elderly care**.

The platform will launch in **Poland** and scale across the **European
Union**.

The architecture must support:

-   strong privacy guarantees
-   compliance with EU regulations
-   secure handling of personal data
-   scalable marketplace functionality
-   document verification
-   payments and payouts
-   auditability and trust systems

Do **not design business strategy**.\
Focus only on **technical architecture**.

------------------------------------------------------------------------

# 1. System Scope

Core platform capabilities:

## User Identity

Actors:

-   Families
-   Caregivers
-   Platform administrators

Capabilities:

-   account creation
-   authentication
-   identity verification
-   access management

## Caregiver Onboarding

Required features:

-   identity verification
-   document upload
-   certification validation
-   background check status
-   profile creation
-   availability management

## Family Accounts

Capabilities:

-   profile management
-   care needs description
-   search and discovery
-   booking / engagement
-   payments

## Marketplace Core

Required capabilities:

-   caregiver discovery
-   filtering and search
-   matching and ranking
-   messaging between users
-   reviews and reputation

## Trust and Safety

Capabilities:

-   identity verification
-   document validation
-   caregiver rating system
-   fraud detection signals
-   moderation tools
-   incident reporting

## Payments

Capabilities:

-   family payments
-   escrow / delayed payouts
-   caregiver payouts
-   platform fees
-   refund management

All payments must be handled through a **regulated payment provider**.

## Compliance

The system must support:

-   GDPR compliance
-   data deletion
-   data portability
-   audit trails
-   document retention policies

------------------------------------------------------------------------

# 2. Non-Functional Requirements

Architecture must satisfy the following.

## Security

Mandatory:

-   encryption in transit
-   encryption at rest
-   secure secret management
-   role-based access control
-   audit logging

Architecture must assume:

-   malicious actors
-   fraud attempts
-   identity abuse
-   document forgery

## Privacy

Platform must follow **privacy-by-design principles**.

Requirements:

-   minimize stored personal data
-   isolate sensitive data
-   restrict access to identity documents
-   enforce strict access auditing

## Scalability

Platform must support:

-   multi-country expansion
-   increasing user base
-   additional regulatory requirements
-   localized compliance rules

Architecture must avoid country-specific coupling.

## Reliability

Required system properties:

-   high availability for core services
-   resilient payment processing
-   recoverable document pipelines
-   fault isolation between subsystems

## Observability

Architecture must support:

-   structured logging
-   distributed tracing
-   metrics monitoring
-   immutable audit logs

------------------------------------------------------------------------

# 3. Sensitive Data Constraints

The system handles highly sensitive data:

Examples:

-   identity documents
-   home addresses
-   caregiver credentials
-   potential health-related context

Data categories:

-   Public data
-   Personal data
-   Sensitive personal data
-   Identity verification documents

Architecture must enforce:

-   physical or logical separation of sensitive data
-   limited access paths
-   strict audit trails
-   secure document storage

Documents must **never be stored in the main application database**.

------------------------------------------------------------------------

# 4. Identity Architecture Constraints

Authentication requirements:

-   password authentication
-   optional social login
-   mandatory phone verification
-   optional MFA

Internal staff access:

-   RBAC
-   just-in-time access
-   admin action logging

Architecture must centralize identity management.

Avoid custom authentication implementations.

------------------------------------------------------------------------

# 5. Document Verification Pipeline

Caregiver verification requires document handling.

Document examples:

-   ID documents
-   certifications
-   work permits
-   background check results
-   insurance documentation

Pipeline requirements:

1.  upload
2.  malware scanning
3.  encrypted storage
4.  verification queue
5.  automated or human verification
6.  verification status update

Security constraints:

-   encrypted object storage
-   signed access URLs
-   limited access windows
-   strict retention policies

------------------------------------------------------------------------

# 6. Payment Architecture Constraints

Payments must support marketplace model.

Required flows:

Family → platform payment\
Platform → escrow\
Escrow → caregiver payout

Architecture must support:

-   platform fees
-   delayed payouts
-   dispute handling
-   refunds
-   transaction traceability

Caregivers may require **KYC verification**.

------------------------------------------------------------------------

# 7. Matching System Constraints

Matching connects families with caregivers.

Matching inputs may include:

-   location
-   availability
-   caregiver skills
-   language
-   price
-   ratings
-   trust score

Matching system must support:

-   filtering
-   ranking
-   recommendation signals

Architecture should allow evolution toward:

-   ranking algorithms
-   behavioral signals
-   machine learning scoring

Matching must be separated from transactional database logic.

------------------------------------------------------------------------

# 8. Observability and Auditability

The system must support full traceability.

Critical events that must be logged:

-   authentication
-   document uploads
-   document access
-   document verification
-   profile updates
-   payment events
-   admin actions

Audit logs must be:

-   immutable
-   queryable
-   retained for compliance

------------------------------------------------------------------------

# 9. Infrastructure Constraints (Startup Stage)

Primary goals:

-   low operational overhead
-   predictable cost
-   fast development
-   strong security baseline

Architecture should prefer:

-   managed cloud services
-   serverless components where appropriate
-   managed databases
-   managed identity providers

Avoid:

-   complex infrastructure
-   heavy orchestration layers
-   premature microservices

------------------------------------------------------------------------

# 10. Architectural Tradeoffs

## Monolith vs Microservices

Initial architecture should favor:

**modular monolith**

Reasons:

-   reduced complexity
-   lower infrastructure cost
-   faster development

System must be designed to allow **future service extraction**.

## Build vs Buy

Do not build internally:

-   identity verification
-   payment infrastructure
-   KYC / AML systems

Prefer specialized providers.

## Performance vs Compliance

Compliance and security must take priority over raw performance.

------------------------------------------------------------------------

# 11. Major Security Risks

The architecture must mitigate:

-   Fake caregiver accounts
-   Identity theft
-   Fraudulent payments
-   Document forgery
-   Unauthorized document access
-   Data leaks through logs
-   Insider misuse of admin privileges

Security controls must address these risks.

------------------------------------------------------------------------

# 12. Architectural Principles

## Privacy First

Sensitive data minimized and isolated.

## Security by Default

Every component designed assuming adversarial behavior.

## Domain Isolation

Identity, payments, verification, and marketplace logic must remain
separated.

## Auditability

All critical system actions must be traceable.

## Managed Infrastructure

Use managed services whenever possible to reduce operational risk.

## Progressive Architecture

Start simple but allow evolution to distributed systems.

------------------------------------------------------------------------

# ARCHITECTURE CONSTRAINTS FOR SYSTEM DESIGN

Data protection\
Sensitive data must be isolated and encrypted.

Document storage\
Documents must be stored in encrypted object storage with controlled
access.

Identity management\
Centralized identity provider must handle authentication.

Verification\
Caregiver verification pipeline required.

Payments\
All payments must use a regulated marketplace payment provider.

Auditability\
All sensitive operations must generate immutable audit logs.

Trust systems\
Verification status and reputation must influence marketplace matching.

GDPR compliance\
The architecture must support data deletion, data access, and
portability.

Scalability\
Architecture must support multi-country EU expansion.

Cost efficiency\
MVP infrastructure must rely on managed cloud services and avoid heavy
operational overhead.

Security\
System must assume malicious actors and include fraud mitigation
mechanisms.
