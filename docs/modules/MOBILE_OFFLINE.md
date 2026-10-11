# MOBILE_OFFLINE.md

Status: **PROPOSED** | 2026-10-11

Companion to [MOBILE_ARCHITECTURE.md](MOBILE_ARCHITECTURE.md). No separate competing decisions.

## Approved context
APPROVED business rules: Hybrid Prepaid SaaS; low-cost Seat + document-based Business Usage + Resource Usage; 1 credit = 10 UZS; progressive discounts and enterprise contracts; real Seat/Usage costs may create negative balance for one calendar month without estimated credit limit; unified company/branch SaaS; uz/en/ru plus hidden future Uzbek alphabet.

## Offline-first rules
Only assigned scoped data is stored. Local commands and durable outbox are atomic. Lifecycle: draft > queued > sending > accepted/rejected/conflict. Timeouts do not imply acceptance; stable operation_id enables safe retries.

## Delta sync and recovery
Push commands with dependency order; pull paged delta, tombstones and scope-bound cursor; apply changes and checkpoint in one transaction. Expired cursor triggers reseed without losing pending operations. Background execution is not guaranteed.

## Consistency
Stock snapshot is informational. Payment is unposted evidence. Central server authorizes and finalizes inventory, financial ledger and SaaS billing exactly once. Late usage period is unresolved; approved minus-balance rule remains unchanged.

## Security and evidence
Encrypted local DB and media; OS key storage; bounded offline session; server-side revocation. GPS may be absent, never fabricated. Media uses checksums and acknowledgement before cleanup.

## ADRs
See [Mobile ADR registry](MOBILE_ARCHITECTURE.md#adr-registry).
