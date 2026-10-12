# ADR-INFRA-001: Bosqichma-bosqich deployment

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Owner:** Infrastructure & Scaling
- **Related module:** [INFRA_ARCHITECTURE.md](../modules/INFRA_ARCHITECTURE.md)

## Muammo
Small initial budget versus future horizontal scaling.

## Ko‘rib chiqilgan variantlar
VM plus containers; Kubernetes from day one; serverless; dedicated bare metal.

## Tanlangan yechim — PROPOSED
Start with simple container host/minimal VMs; move to load balancer and N stateless app instances when measured.

## Sabab
Less operational overhead and portable releases.

## Xavflar va cheklovlar
First stage has single point of failure and manual operational burden. Numeric SLO/RPO/RTO and vendor selection are not approved.

## Boshqa modullarga ta’siri
Backend, Security, Frontend, Operations.

## Tasdiqlash sharti
Cross-module review and explicit user approval; until then not an approved infrastructure decision.

## Round 2 MR-15 scaling and resource budget — 2026-10-12 (PROPOSED)

Budget app compute from measured request CPU/RSS and headroom; bound sum of instance/worker connection pools by actual safe PostgreSQL concurrency; measure outbox enqueue/service/backlog, cache entries/misses, search index/write amplification, background-job CPU/IO and storage/egress/version growth. Compare single host, separate workers/DB, multi-app LB and optional enterprise placement with documented baseline/peak/incident headroom and operating cost. Load tests include hot tenant, billing month-end, concurrent stock writes, mobile reconnection burst, search/report contention and soak/spikes. Instrument p95/p99, failures, wait times, retries, ledger/stock correctness and cost; **no numerical targets or passed measurements claimed**. Kubernetes, Kafka, microservices and sharding not mandatory; migration only via measured saturation/approved need. See [Infra §19](../modules/INFRA_ARCHITECTURE.md).
