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

## 18. Cross-module reconciliation: AR-003 / MR-14 / MR-15 (PROPOSED, 2026-10-12)

**Matched with [Database Architecture §17](DATABASE_ARCHITECTURE.md) and [ADR-DB-007](../adr/ADR-DB-007.md):** Production stage *choice* is still OPEN pending user requirements; an early initial-standby suggestion is retained as an optional alternative, not a mandatory APPROVED policy.

| Stage | Topology proposal | Recovery / availability claim | Advance gate |
|---|---|---|---|
| A — cost-conscious starting option | One PostgreSQL writer; isolated encrypted offsite base backups + continuous WAL archive/PITR; recurring restore drills | Recovery-capable **but not high availability**; outage during writer failure | Only after user accepts measured restore downtime / availability trade-off |
| B — stronger availability option | Writer + physical standby, health checks, controlled promotion, stable writer endpoint, fencing | Potentially faster service recovery; must measure actual failover, replication lag and no split-brain | Approved SLO/RPO/RTO, verified promotion/rollback and budget |
| C — later evolution | Automated supervised failover, read replicas where stale reads permitted, bounded pools and optional tenant placement | Explicit failover tests and capacity data required | Sustained measured load and operational staffing |

**Non-negotiable proposed tests before production:** protect and periodically restore WAL chain offsite; perform simulated primary loss, test rollback and split-brain fencing if Stage B; measure recovery time/data loss and compare *only to approved objectives*; verify ledger consistency, stock reconciliation and tenant isolation after recovery; establish alert ownership, restore authorization, credential separation, incident process and retention/legal hold policy. None of these tests are represented as already executed.

**MR-14/MR-15 OPEN:** acceptable downtime, RPO/RTO, availability SLO, restore frequency, retention, residency, expected peak tenants/transactions and connection budget. No numerical target or provider is approved by this document.

**Impact:** 03 Database, 04 Backend, 02 Multi-Tenant, 10 Security, Billing/Finance/Inventory and 00 Master. The prior AR-003 *textual* disagreement is reconciled at the PROPOSED stage; selection of launch topology remains an explicit user/business decision.


## 19. ROUND 2 CORRECTION — AR-003 / MR-14 / MR-15 (2026-10-12; PROPOSED, no tests executed)

**Evidence:** [Round 2 audit](../reviews/ARCHITECTURE_REVIEW_ROUND_2.md) §12 / §20, [Database](DATABASE_ARCHITECTURE.md) §9 / §18.5, [Backend](BACKEND_ARCHITECTURE.md) §7 / §18, [Security](SECURITY_ARCHITECTURE.md) §19, [Billing](BILLING_ARCHITECTURE.md) §20, [Multi-Tenant](MULTI_TENANT_ARCHITECTURE.md) §19. Existing §7 / §18 is preserved; this section specifies operational gates, not a new approval. **No measurements, restore drills or failover exercises have been performed or claimed.**

### 19.1 Canonical availability stages and migration gate

| Stage | Architecture | Guarantee / trade-off | Admission gate — evidence required |
|---|---|---|---|
| **A — recovery-first** | Single PostgreSQL primary; encrypted base backup + continuous off-site WAL archive / PITR + verified isolated restore | **NOT HA**; primary host failure causes downtime; cost/operations lower than standby | Business accepts downtime risk; backup encryption, complete WAL chain, isolated PITR, restore verification, alert/runbook/keys and region compliance tested **before production** |
| **B — controlled HA** | Primary + streaming standby; monitored lag; **single writable endpoint, fencing, promotion and tested failback** | Potential lower downtime but additional compute/storage/network/operator cost; async replication may lose unshipped commits | Explicit availability objectives and budget; Stage A recovery tests remain mandatory; DB/Security verify lockout of old writer, no dual-primary, promotion/rejoin/rollback, application reconnect, stale read policy |
| **C — measured automation** | Automated failover orchestration, quorum/witness where needed; optional read routing based on measured load | More moving parts, split-brain risk, operational complexity; not an MVP obligation | Repeated successful B drills, incident/volume evidence, approved error budget/cost and automatic fencing & reconciliation test |

**Stage A versus B production launch remains OPEN.** Replication, replica, snapshot and automatic failover never replace independently restorable off-site backup. Stage C is not mandatory. DB §18.5 is the controlling peer proposal, and no contradictory Stage B-at-launch requirement is imposed here.

### 19.2 Explicit OPEN business decision packet (user sign-off required)

| Decision | Alternatives / user input | Evidence and impact | State |
|---|---|---|---|
| Launch topology | Stage A recovery-first **or** Stage B primary/standby from day one | Downtime tolerance, recurring infra/ops expense and ability to operate fencing | **OPEN** |
| RPO | Maximum tolerable **data loss duration**, per recovery scenario; no preset number | WAL archival lag, sync/async replication tradeoffs, restore drill observations | **OPEN** |
| RTO | Maximum tolerable **service restoration duration**, by failure class | Detection + restore + verification + cutover timings | **OPEN** |
| Uptime SLO/SLA | Separate internal SLO and contractual SLA, measurement window, exclusions and error budget | Business/service priority, on-call coverage, availability tracking | **OPEN** |
| Backup retention | Full/WAL/object versions, legal hold, immutable copies and deletion/privacy requirements | Storage cost, compliance, tenant-specific recovery capability | **OPEN** |
| Region/residency | Primary, backup, standby, DR region and cross-border policy | Regulation, latency, replication cost and cross-region key custody | **OPEN** |
| Budget & sizing | Monthly baseline/peak cap, cost per workload and availability premium; provider/VM class | Actual load simulation plus vendor quotes, not fabricated tenant count/TPS target | **OPEN** |

**Decision process:** Business/product owner selects objectives and launch Stage A/B; Database + Security + Infra document feasibility, costs and disaster scenarios; approve ADR-DB-007 jointly with ADR-INFRA-002/004; schedule evidenced preproduction drills. If objectives are absent, no numeric commitment and no declaration of production readiness.

### 19.3 Resource budget method — proposed worksheet, values TBD

| Resource | Demand/limit model | Observation and saturation signals | Scale decision |
|---|---|---|---|
| App server / LB | Concurrent active requests × measured CPU-time and memory per request; reserve OS/TLS/rollout headroom; account for worker competition | CPU throttling, RSS/GC, p95/p99, request queue, 5xx | Tune requests/DB first, then stateless replicas; zero sticky-session dependency |
| PostgreSQL connections | `Σ(app_instances × app_pool_max) + Σ(worker_instances × worker_pool_max) + admin/maintenance + replica/HA overhead ≤ measured safe DB capacity` | Active/idle sessions, wait time, lock waits, transaction duration, memory per connection | Bound pools/worker concurrency; admission/backpressure; consider pooler only if tested with transaction-local tenant/RLS context |
| Queue/outbox | Peak enqueue rate versus durable worker completion rate; estimate backlog ≈ arrival–service rate integrated over burst; storage and retry DLQ allowance | Oldest event age, backlog, failed retries, duplicates, DB contention | Increase worker capacity within DB limits; external broker only after measured need; idempotency mandatory |
| Cache/Redis | Hot-key count × measured encoded item size × replication/allocator overhead; TTL and invalidation budget | hit/miss, eviction, stale authorization and tenant key leakage | Start without shared cache; tenant-scoped keys, invalidation/version control; never cache authoritative ledger/permission effect |
| Search | Indexed rows × measured index amplification, ingestion bursts and query concurrency; rebuild space | Query time, indexing lag, disk/IO, hot tenant skew | DB indexes first; separate engine only with workload evidence and tenant-aware search enforcement |
| Background jobs | Worker slots × CPU/RAM/IO/time/job; concurrent month-end jobs versus pool/DB reserve | Job lag, runaway exports/reports, retries and starvation | Priority/fair scheduling, per-tenant concurrency caps, rate limiting and separate pool when justified |
| Object storage/CDN | Upload GB + object count + versioning + backup copies + read/egress GB and request fees | Growth, failed uploads, egress, signed URL expiry, orphan objects | Private, tenant-authorized access; lifecycle/retention per approval; CDN only for safely public immutable assets |

**Procedure:** measure workload mix and unit cost in an isolated tenant-aware fixture; model baseline, peak and failure/recovery headroom; cost each candidate Stage A/B/C; document limiting resource and uncertainty; remeasure after every topology change. No arbitrary server size, connection count or threshold is approved. Resource Usage billability is Billing-owned and not inferred from these infrastructure estimates.

### 19.4 MR-15 load and failure test plan (not executed)

| Scenario | Workload / fault injection | Required evidence and invariant |
|---|---|---|
| Hot tenant / noisy neighbor | One tenant dominates reads, writes, large uploads, searches while other tenants operate | Per-tenant fairness and isolation, p50/p95/p99 by workload, DB pool wait, throttling and cost; no cross-tenant leakage |
| Billing month-end | Burst of seat/usage events, backdated/late mobile sync and reconciliation | Outbox/worker age, ledger balance invariant, same occurrence billed once, retry/dedup, no guessed payer/month; unresolved rules remain Billing OPEN |
| Concurrent stock updates | Many sales/dispatch/reservation attempts against same SKU/location | Lock/deadlock analysis, inventory movement exactly once, no silent lost update; stock policy stays Sales-owned |
| Offline sync bursts | Large device reconnect wave plus revocations and uploads | Queue saturation, fair retry, authorization at server reconnect, tenant placement epoch safety |
| Reporting and search | Expensive exports/report reads alongside OLTP sales and finance work | Read routing consistency, lag, production query latency protection, index/IO pressure; authoritative financial reads never silently stale |
| Soak and spike | Prolonged normal load then peak and app restart/network impairment | Memory/connection leak, backlog drain, resource cost, 5xx, recovery without local session dependence |
| DB failover / WAL outage | Kill writer under B, stop archiver under A/B, partition old writer | Fail-closed writes, fencing, no split-brain, WAL gap alarm, validated PITR and recovery elapsed/loss observations |

**Test methodology:** record dataset sizes, tenant skew, hardware/provider region, release version, duration, load generator, baseline vs peak, observed concurrency/TPS, p95/p99/error rates, pool waits/locks, queue lag and cost per business transaction. Compare with objectives **only after explicit approval**; no result is labelled achieved or passed before execution. Failure injection only in controlled non-production environments until separately authorized.

### 19.5 Backup, failover and tenant recovery acceptance contract (NOT EXECUTED)

1. **Backup chain:** enumerate encrypted full/base backups and continuous WAL segments, verify checksums/manifest, retention access and complete gap-free replay across target PITR timestamps; simulate a missing WAL segment and require alert/fail-closed recovery.
2. **Isolated restore:** use separate credentials, network, storage and encryption-key access path; restore DB plus version-consistent object blobs, validate transaction cutoffs, operational metadata, immutable Billing ledger, GL/stock reconciliation and protected tenant boundaries.
3. **Primary failure, B:** controlled promotion selects one writer; old primary is fenced before client traffic can resume; detect stale DNS/endpoint, retry storm and duplicate event effects; failback requires re-seeding and reconciliation rather than unsafe dual writable.
4. **Split-brain:** inject partition/quorum loss and stale placement epoch; deny writes where exclusive writer cannot be proven, audit election and operator override; verify no dual primary even under repeated restart.
5. **Tenant-scoped recovery:** cannot simply PITR whole shared database to rescue one tenant; restore into isolated DB, extract/validate tenant-owned data, map references, avoid cross-tenant collateral writes, coordinate Billing/Finance/attachments/security and single-writer epoch fencing; tenant restore RPO/RTO remains OPEN.
6. **DR evidence packet:** dated scenario/test artifact, source backup IDs and WAL ranges, integrity results, elapsed recovery time and observed data loss, approvals, unresolved gaps and remediations; repeat after major schema/placement changes.

**Cross-module gate:** Database owns recovery correctness, Backend owns retries/pooling, Security owns key and access controls, Multi-Tenant owns placement/tenant identity, Billing/Finance/Inventory own ledger/stock reconciliation. No global `DECISIONS.md` / `OPEN_QUESTIONS.md` edits here; propose links and OPEN packet synchronisation only. **Codex deployment remains BLOCKED for production until business decisions, test execution and approval; this is architecture-only specification.**
