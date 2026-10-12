# Codex Handoff — Remaining external evidence and architecture gates

- **State:** handoff docs in progress; `DESIGN_SELECTED_UNDER_DELEGATION` does not equal statutorily approved finance or passed tests.
- **Last historical independent audit:** [Round 3](../reviews/ARCHITECTURE_REVIEW_ROUND_3.md), frozen source SHA `34fbb99db8c71a01dbb50ef71a6f7555a3310e91`; the new ADR-010/BILL-011…014 require a delta review, no automatic `READY`.
- **User instruction:** choose good design options without repeated questions, prepare Codex handoff; **product code not authorized in this architecture-only project**.

## Gate classes
- `A` = user-approved original invariants or explicitly approved scoped business choices, with provenance.
- `D` = `DESIGN_SELECTED_UNDER_DELEGATION`; coherent candidate frozen for planning, technically reviewable, no legal/price claims.
- `E` = **EXTERNAL_INPUT_REQUIRED** (real law/contract/account/provider/SLO input cannot be inferred).
- `T` = **TEST_DEFINED_NOT_RUN** (requires implementation environment).
- `S` = technical sign-off of a candidate ADR by owner specialists; not an unverified simulation of testing.

## MR-01…MR-20 canonical current handoff status

| MR | Resolved direction | Remaining actual gate |
|---|---|---|
| 01 Company/legal/payer | **A:** Company distinct LegalEntity, BillingAccount, Wallet; authorized 3rd-party funder != debtor; ADR-BILL-011 | E: legal identity, payer authorization provider evidence |
| 02 Negative month | **D:** anniversary calendar month + clamp, source real debit crossing <0; no estimated limit A; ADR-BILL-013 | E: contract/policy for rearm and expiry service handling; S: time/late correction |
| 03 Seat | **A:** user×Company one, no Branch/device dup; **D:** activation/day proration ADR-BILL-012 | E: actual price, exact status/cutoff, pause/refund and cycle |
| 04 Business/Resource Usage | **D:** finalized docs configured only, metered resource with one responsible Company A; ADR-BILL-014 | E: chargeable tariff, eligible units, price, volumes, refund and tax |
| 05 Discounts | **A:** progressive+enterprise contracts; **D:** marginal tiers and no implicit stack | E: thresholds/tiers/net price and contract priority per customer |
| 06 Finance legal GL | **D:** invoice AR/revenue on legally configured policy, dispatch COGS, append-only balanced GL | E: jurisdiction, accounting standards, functional currency, taxes, chart, close, policy owner |
| 07 Sales/Stock/Finance | **D:** owner-local ACID+outbox, invoice no second movement, dispatch once | S: versioned schema/GL triggers; T: double-post/replay tests |
| 08 Inventory policy | **D:** nonnegative stock, reserved quantity/availability, moving average candidate | E: stock count cutoff, reservation expiry and legal valuation selection; T: concurrency |
| 09 Party master | **D:** one tenant-scoped shared Party context, CRM relationship only, Finance immutable legal debtor | S: merge/privacy authorization; E: legal counterparty verification |
| 10 API/event | **D:** typed versioned REST/OpenAPI and source event/occurrence ID separation | S: API contracts/evolution; T: idempotency/compatibility |
| 11 Isolation/security | **D:** shared schema + tenant composite FK + conditional force RLS, server guard | S: roles/privileges/pool setup; T: RLS, export, search, cache, jobs, tenant negative tests |
| 12 Mobile offline | **D:** encrypted scoped outbox, stable operation ID, next-contact revoke, UNKNOWN distinct | E: device data retention & offline TTL; S/T: policy & reauth |
| 13 GPS/media privacy | **D:** default no tracking until legal basis/consent/retention config | E: lawful purpose, GPS permissions, retention/privacy legal sign-off |
| 14 HA/DR/SLO | **D:** Stage A internal pilot recoverability, Stage B before contractual HA | E: initial paid region/provider, SLO/RPO/RTO, storage retention/cost; T: restore/failover |
| 15 Performance | **D:** measured resource budgeting, no early Kafka/sharding | E: workload, cost envelope; T: p95/p99, hot tenant, month end, sync bursts |
| 16 UX-T01…T20 | **D:** prioritize existing documented workflows with persona/devices and error states | E: pilot participant/user research access; T: task timing/error/a11y outcomes |
| 17 Authorized search | **D:** tenant/branch scoped index/results/counts/facets/recents/cache | S/T: security and locale-switch reindex negative tests |
| 18 MVP domain scope | **D:** core Sales/Inventory/Billing/CRM/Purchasing/Finance subset, defer full HR/Manufacturing | E: actual pilot/customer scope and delivery promise |
| 19 SalesDoc V2 | **D:** read-first disabled adapter; documented provider matrix exists | E: authorized credentials/sandbox/cutover permissions; T: LIVE_VERIFIED |
| 20 Accessibility/locale/network | **A:** uz/en/ru and hidden future alphabet; **D:** UX-T01…20/a11y quality matrix | E: baseline devices/network/users; T: pilot, screen reader, locale QA |

## 13 AR history: do NOT claim sign-off closures

AR-001: new limited user-approved subdecisions, legal/implementation open. AR-002/003/005/006/007/011/012/013: documentary gaps improved, test/owner/compliance still open. AR-004 FIN-001 historical label still provenance unverified — cannot invent dated old approval. AR-008/009/010: domain and billing alternatives harmonized, delegated candidate defaults selected, not full economic/legal acceptance. **0 issues newly independently certified closed by this handoff document.**

## Must be resolved BEFORE enabling live financial/stock operations

1. Verifiable legal contracting jurisdiction, tax/GL recognition, debtor evidence, operational currency and valuation policy, effective-dated company legal identity.
2. **Published** Seat/document/resource tariff and discounts with units/cutoffs/effective policy version, enterprise precedence, cancellation/refund and top-up/chargeback signed policy.
3. Approved negative one-calendar-month precise contract (anniversary/clamp, rearm, expired-state user rights, late-month correction) rather than hidden restrictions.
4. Ownership and permission technical sign-off (membership writer, Party/Catalog, branch switch, payment payer consent, tenant isolated RLS and impersonation).
5. Payment provider verified settlement and lawful payer authorization; no write to wallet on pending webhook.
6. Real test environment and evidence from concurrent stock, Billing idempotency, GL balance, cross-tenant/branch negative security, recovery/PITR drill and real user usability.

## PRE-CODE vs PRE-PRODUCTION distinction

The user can later authorize Stage 0/1 code while some production legal/provider evidence is unavailable, **provided all monetary/tax/outbound paths are disabled by explicit fail-closed flags** and technical architecture is reviewed. To actually charge a Wallet, post GL/tax, sync external data or promise uptime, that specific gate requires real evidence and sign-off. Past Round 3 `CODEX BLOCKED` is a historical independent statement, not overwritten by architecture drafting.

## Ready-for-human-review checklist

- [x] ADR-BILL-011 company scope and legal liability business decisions documented and synced
- [x] ADR-BILL-012 Seat lifecycle candidate and original scope separated
- [x] ADR-BILL-013 calendar negative lifecycle candidate with exact examples
- [x] ADR-BILL-014 chargeable documents/overage/marginal tiers candidates with no fictional rates
- [x] ADR-010 coherent technical and MVP scope baseline
- [x] Codex Start and implementation backlog
- [ ] Owner technical ADR sign-offs and independent post-delegation architecture delta audit
- [ ] External legal/commercial/security compliance profile
- [ ] Explicit user authorization for application code
- [ ] Actual tests on implemented software
