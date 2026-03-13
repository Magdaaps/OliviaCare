<!-- Source: Caregiver_Platform_IT_Architecture.md | Sections: 8, 12 -->

## 8. Infrastructure Architecture

### 8.1 Cloud Setup (GCP — europe-central2, Warsaw)

GCP selected over AWS/Azure for: (1) superior startup credits program, (2) Warsaw region for Polish data residency, (3) BigQuery for cost-effective analytics, (4) Cloud Run's generous free tier.

| Layer | Service | Configuration | Est. Monthly Cost (MVP) |
|-------|---------|--------------|------------------------|
| Compute | Cloud Run | 1 service, min 0 / max 4 instances, 1 vCPU, 512MB RAM | €0–€30 |
| Primary Database | Cloud SQL PostgreSQL 16 | db-f1-micro (shared), 10GB SSD, automated backups | €15–€25 |
| PII Vault Database | Cloud SQL PostgreSQL 16 | db-f1-micro (shared), 5GB SSD, private IP only | €15–€25 |
| Cache | Memorystore Redis | Basic tier, 1GB | €30 |
| Object Storage | Cloud Storage | Standard tier, ~10GB initially | €1–€3 |
| Queue / Tasks | Cloud Tasks + Pub/Sub | Pay-per-use | €0–€5 |
| CDN | Cloud CDN (via Load Balancer) | Caching static assets | €5–€10 |
| WAF | Cloud Armor | Standard tier, managed rules | €5/policy + €0.75/M requests |
| Secrets | Secret Manager | Pay-per-access | <€1 |
| KMS | Cloud KMS | Key management for encryption | €1–€3 |
| Monitoring | Cloud Logging + Grafana Cloud (free tier) | Structured logs, metrics | €0–€10 |
| Analytics | BigQuery | Pay-per-query, ~10GB scanned/month initially | €0–€5 |
| CI/CD | GitHub Actions | Free tier for private repos (2000 min/month) | €0 |

**Estimated total MVP infrastructure cost: €80–€150/month**

### 8.2 Environment Separation

| Environment | Purpose | Infrastructure | Data |
|-------------|---------|---------------|------|
| Development | Local development | Docker Compose (PostgreSQL, Redis) | Synthetic test data only |
| Staging | Pre-production testing, QA | Minimal Cloud Run + shared Cloud SQL | Anonymized production subset |
| Production | Live platform | Full Cloud Run + dedicated Cloud SQL instances | Real data, encrypted, audited |

### 8.3 Network Architecture

- VPC with private subnets for database instances (no public IP on Cloud SQL)
- Cloud Run connects to Cloud SQL via VPC connector (Serverless VPC Access)
- Cloud Armor WAF at the load balancer edge
- All internal communication over private network; no public endpoints for databases or cache
- Egress restricted to known external service IPs (Stripe, Onfido, Resend, Twilio)

---

## 12. Observability and Operations

### 12.1 Logging

- Structured JSON logging (pino logger in NestJS)
- Log levels: ERROR, WARN, INFO, DEBUG (DEBUG only in dev/staging)
- **PII scrubbing middleware:** automatically redacts email, phone, PESEL, IBAN from logs
- **Correlation IDs:** every request gets a unique trace ID propagated across all log entries
- Log routing: GCP Cloud Logging with 30-day retention; critical logs forwarded to BigQuery for long-term

### 12.2 Metrics

| Metric Category | Examples | Tool |
|----------------|----------|------|
| Infrastructure | CPU utilization, memory usage, instance count, request latency p50/p95/p99 | GCP Cloud Monitoring |
| Application | Request rate, error rate, response time by endpoint, active sessions | OpenTelemetry → Grafana |
| Business | Registrations/day, verifications completed, matches made, bookings created, payment volume | Custom counters → Grafana |
| Security | Failed login attempts, rate limit hits, document access frequency, admin actions | Custom counters → Grafana + alerts |

### 12.3 Tracing

- OpenTelemetry SDK instrumented in NestJS (auto-instrumentation for HTTP, PostgreSQL, Redis)
- Trace propagation across async jobs via Cloud Tasks metadata
- Grafana Tempo for trace storage and visualization
- Sampled at 10% in production (100% for error traces)

### 12.4 Alerting

| Alert | Condition | Channel | Severity |
|-------|-----------|---------|----------|
| API Error Rate | >2% 5xx responses for 5 min | Slack + PagerDuty | Critical |
| Database Connection Pool | >80% utilization for 10 min | Slack | Warning |
| Payment Failures | >3 consecutive failures | Slack + PagerDuty | Critical |
| Document Verification Queue | >50 pending for 2 hours | Slack | Warning |
| Rate Limit Spikes | >100 blocked requests/min | Slack | Warning |
| Certificate Expiry | <14 days until expiry | Email | Warning |
| Disk Usage | >80% on any Cloud SQL instance | Slack | Warning |
| Matching Latency | p95 >5 seconds for 10 min | Slack | Warning |

### 12.5 Operational Dashboards

- **Platform Health:** request rate, error rate, latency, active instances
- **Business KPIs:** daily registrations, active matches, booking conversion, GMV
- **Verification Pipeline:** queue depth, processing time, approval/rejection rates
- **Security:** failed logins, rate limit events, suspicious activity patterns
- **Infrastructure Cost:** daily spend tracking with alerts at 120% of budget

---

