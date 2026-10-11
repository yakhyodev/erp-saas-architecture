# Infrastructure & Scaling Architecture

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Module:** Cloud / Infrastructure & Scaling
- **Scope:** server topology, deployment, scale, load balancing, DB HA, monitoring, backup, DR, tenant placement.
- **Non-goal:** implementation code, final cloud vendor, final instance sizes/pricing.

## 1. Approved project constraints (not new infra approvals)
- Multi-company and multi-branch ERP SaaS; Hybrid Prepaid SaaS.
- Seat + Business Usage + Resource Usage; 1 credit = 10 UZS.
- Document-based business usage, progressive discounts and enterprise contracts.
- Actual seat/usage costs accrue to negative balance for one calendar month when prepaid balance expires; no estimated negative credit limit.
- Performance, resource efficiency, security, scalability and ease of use are priorities.
- Go, PostgreSQL, Next.js and Flutter remain *suggested*, not approved.
- Multi-tenant and modular-monolith deployment details remain subject to cross-module approval.

## 2. Design principles (PROPOSED)
- Start simple; do not introduce Kubernetes, Redis, message brokers or read replicas until measurements justify them.
- Stateless app instances: sessions, mutations, jobs and idempotency must not depend on a process's memory.
- No false HA claims for a single node. Recovery is distinct from availability.
- Protect tenant boundaries in DB, cache, queues, object keys, observability and restoration.
- Version, observe and roll back every release; secrets never enter repository or image.
- All numbers for budgets, SLO/SLA, RPO, RTO and retention require explicit approval.

## 3. Initial server architecture (PROPOSED)

```text
Browser / Mobile
      |
    DNS, TLS
      |
HTTPS reverse proxy
      |
Web UI / stateless API (single app host initially)
      |            |
durable DB jobs    background worker
      |            |
PostgreSQL primary + encrypted off-site backups / PITR
      |
private object storage (files, images; metadata in DB)
      |
central logs / metrics / alerts
```

- Reverse proxy, frontend, API and worker may initially share an application host.
- Prefer managed or separate PostgreSQL where economically justified; if colocated, explicitly record single point of failure.
- Use durable DB-backed outbox/jobs initially; avoid authoritative process-memory queues.
- Redis is optional, introduced for measured cache/job needs, not assumed.
- Blob files belong in private object storage; authorization comes from app policy.
- CDN may serve public immutable assets; do **not** cache authenticated ERP responses.

## 4. Evolved architecture (PROPOSED)

```text
Clients -> TLS / WAF / Load Balancer -> app-1, app-2, ... app-N
                                            |          |
                                    worker pool   shared state
                                            |
                                     PostgreSQL writer
                                            |
                                   standby / WAL archive
                           object storage / optional CDN
                   logs + metrics + traces + alerts
```

- Health-aware LB uses separate liveness and readiness checks, connection draining and upstream timeouts.
- No sticky session dependency. Rate-limit by user, tenant and IP with NAT users in mind.
- Replicas may serve explicitly stale-tolerant reads; payment, billing, stock and finance posting read from strongly consistent source.
- Monitor total connection pool budget across instances. Read replicas and HA are not backups.

## 5. Deployment and operations (PROPOSED)
- Environments: development, staging, production; production-like restore/migration tests in staging.
- Immutable versioned image, pinned digest, documented runtime config and secret injection.
- Delivery gates: automated tests, security checks, backward-compatible migration, smoke test, approval.
- Migration strategy: expand -> deploy -> contract; destructive changes require separate coordination.
- Rollback to previous image/config; DB compatibility or forward-fix instead of assuming DB rollback.
- Rolling rollout only when multiple app nodes support it; single-host maintenance may require downtime.
- Declarative infrastructure recommended; provider and orchestration remain open.
- HTTPS endpoints public; DB, queues, workers and admin interfaces private.

## 6. Scaling and performance (PROPOSED)
Measure: p50/p95/p99 latency, throughput, errors, CPU/RAM, DB connection saturation, IOPS, locks, slow queries, storage, replication lag, job age and peak transaction concurrency.
1. Profile/index/query-optimize before adding servers.
2. Scale stateless app instances horizontally behind LB.
3. Separate heavy workers from request-serving instances.
4. Introduce shared cache and dedicated broker **only when justified** by measurements; keep correctness outside cache.
5. Move to PostgreSQL HA, sharding or dedicated enterprise DB based on availability, performance, contractual or compliance requirements.
6. Track cost per 1,000 operations and worst-case tenant bursts, not only count of tenants.

## 7. PostgreSQL availability (PROPOSED)
- **Stage A:** single primary, automated encrypted base backups, continuous WAL archive/PITR, verified restore. This is not HA.
- **Stage B:** monitored standby, stable writer endpoint, documented failover and fencing to prevent split-brain.
- **Stage C:** tested automatic failover, pooling and replica routing where justified.
- Consider synchronous replication only after evaluating write latency and availability trade-offs.
- Preserve tenant isolation on replicas, during migration and in recovered data.

## 8. Monitoring, alerting, SLO (PROPOSED)
- Edge/API: requests, p95/p99, 5xx, availability, saturation.
- Backend: retries, failed jobs, oldest job, outbox lag.
- DB: locks, disk/WAL, replication lag, connections, long queries.
- Billing/Finance: idempotency violations, usage ledger reconciliation and missing/delayed events.
- Operations: CPU, memory, disk, TLS certificate expiry, secret expiry, backup success and restore drill status.
- Structured logs with trace/request IDs; do not log passwords, tokens or sensitive personal data. Avoid unbounded tenant_id metric cardinality.
- Document alert ownership, on-call, incident severity, escalation and postmortems.
- Numeric SLO, SLA, RPO and RTO remain **OPEN QUESTIONS**.

## 9. Backup and restore (PROPOSED)
- PostgreSQL base backup + archived WAL with integrity checks; encrypt in transit/at rest.
- Keep protected off-site backups logically separate from production credentials and host.
- Include object storage versioning/retention, DB/object consistency, deployment config, metadata and recoverable encryption keys.
- Regularly restore to an isolated environment and validate ledger, stock, tenant isolation and file references.
- Suggested initial rehearsal: monthly; frequency and retention are **not approved**.
- A single tenant's recovery inside shared DB is not equivalent to restoring the whole DB; design a scoped logical recovery procedure with cross-tenant integrity checks.

## 10. Disaster recovery (PROPOSED)
- **App host loss:** redeploy pinned artifact and configs; validate API and jobs.
- **DB corruption:** isolated PITR, verify ledger/stock, controlled cutover.
- **Site loss:** secondary site restore or tested standby promotion; verify fencing and DNS/LB cutover.
- **Object loss:** restore versions/replicas; match DB object metadata.
- **Compromise:** revoke credentials, preserve evidence, rotate secrets and restore clean environment.
- **Queue incident:** replay durable work with idempotency; never double-charge or double-post inventory.
- DR runbook must identify incident commander, communication, validation and post-incident review.
- Do not claim achieved RPO/RTO before drills.

## 11. Tenant placement and enterprise isolation (PROPOSED)
- Shared-first DB placement only if Security/Database/Multi-Tenant architecture agrees.
- Design abstract tenant_id -> database/cluster placement registry; no hard-coded DB endpoint in business logic.
- Dedicated tenant DB and shard migrations use snapshot, change catch-up, controlled cutover, verification and rollback.
- Coordinate identity, billing wallet and global reporting boundaries before cross-database migration.

## 12. Load test plan (PROPOSED)
- Workloads: concurrent sales, inventory posting, payments, mobile sync bursts, uploads, reports, imports and monthly billing.
- Baseline, peak, spike, soak, fault-injection, DB failover and restore trials.
- Validate no tenant leakage, no duplicate financial effect and no lost durable events.
- Report throughput, latency, DB contention, worker lag, cost and safe capacity envelope.

## 13. Decision records
- [ADR-INFRA-001](../adr/ADR-INFRA-001.md): staged deployment — PROPOSED.
- [ADR-INFRA-002](../adr/ADR-INFRA-002.md): PostgreSQL resilience evolution — PROPOSED.
- [ADR-INFRA-003](../adr/ADR-INFRA-003.md): queue and cache introduction — PROPOSED.
- [ADR-INFRA-004](../adr/ADR-INFRA-004.md): backup and DR from initial production — PROPOSED.
- [ADR-INFRA-005](../adr/ADR-INFRA-005.md): tenant placement abstraction — PROPOSED.

## 14. Cross-module impacts and potential conflicts
- Backend: process-local sessions and authoritative jobs conflict with horizontal scaling.
- Database: RLS/tenant separation, backups, read consistency, migration and connection pooling need coordination.
- Multi-Tenant: shared-first placement is conditional, not an approved isolation model.
- Billing/Finance/Inventory: replay must never double-charge or double-post; critical reads must not silently use stale replicas.
- Security: network boundary, secrets, backup access, incident response and data residency policies.
- Mobile: offline burst/sync uploads included in load test.
- UX: availability and latency targets must map to critical workflows.

## 15. Open questions
Cloud vendor/region; data residency; peak concurrency and largest tenant; cost budget; acceptable downtime; SLO/SLA; RPO/RTO; managed DB choice; backup/object retention; enterprise isolation triggers; on-call and incident ownership.

## 16. Proposed synchronization (do not edit shared documents automatically)
- `DECISIONS.md`: add ADR-INFRA-001..005 with status **PROPOSED**.
- `MASTER_ARCHITECTURE.md`: include staged app / DB HA / tenant scaling evolution as proposal.
- `REQUIREMENTS.md`: add TLS, private DB, reliable restore, metrics, audited deployments and durable processing requirements as proposals.
- `MODULE_REGISTRY.md`: Infra owns topology, deployment, capacity, observability and DR interfaces; depends on Backend, Database, Multi-Tenant, Security, Billing, Finance, Inventory and Mobile.
- `OPEN_QUESTIONS.md`: include §15.
- `CHANGELOG.md`: 2026-10-11 initial INFRA proposal and five proposed ADRs.

## 17. Codex deployment specification (SPECIFICATION ONLY — no implementation yet)
**Goal:** portable and reproducible deployment with minimal operations, future horizontal app scale, DB HA and tenant isolation.

**Future deliverables:** environment/topology diagrams; versioned infra definitions; config/secrets contract; TLS/routing/health probes; artifact build/release/rollback design; compatible migrations; DB WAL/PITR/off-site encrypted backup and isolated restore runbook; privacy-safe dashboards/alerts; performance/fault/soak test plans; incident/DR runbooks; tenant placement integration and cost decision matrix.

**Acceptance criteria:**
1. Replacing an app instance does not destroy sessions or durable jobs.
2. Application release is auditably reversible without assuming destructive DB rollback.
3. Database is not public; recovery copies are encrypted and independent.
4. Isolated restore passes tenant and financial/stock integrity checks.
5. Retry/replay does not cause duplicate billing, payment or stock posting.
6. Multi-instance topology requires no sticky sessions.
7. Cache and object access enforce tenant boundaries.
8. Monitoring detects application and backup failure without leaking secrets.
9. Scaling decision uses measured service limits; Kubernetes, Redis and replicas are not default prerequisites.

**Approval gate:** all five infra ADRs require cross-module review and explicit approval before becoming binding technical decisions.
