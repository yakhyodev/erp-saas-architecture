# CRM_DISTRIBUTION.md — CRM & Distribution Architecture

- **Version:** 0.2 (architecture consolidation, 2026-10-11)
- **Domain owner:** CRM & Distribution
- **Status:** PROPOSED for all new module architecture, separate from APPROVED platform constraints
- **Scope:** Customer, Counterparty, Outlet, Field Agent, Supervisor, Territory, Route, Visit, GPS, photo evidence, Distribution, KPI and SalesDoc adapter.
- **Implementation:** no application code.

## 1. APPROVED platform requirements (inherited; not CRM decisions)

Hybrid Prepaid SaaS = inexpensive Seat + Business Usage (base charge per document and additional credits beyond volume threshold) + Resource Usage; **1 credit = 10 UZS**. Progressive usage discounts and enterprise individual agreements. When the balance is exhausted, **actual** seat and usage costs accrue as a negative balance for **one calendar month**, without estimating a credit limit. A unified SaaS platform handles companies and their branches. UI: Uzbek, English, Russian plus hidden future Uzbek alphabet option.

Go, PostgreSQL, Next.js and Flutter remain **recommended, NOT APPROVED**. Modular monolith and multi-tenant architecture are under review; multi-server, sharding and enterprise isolation are future considerations.

## 2. Bounded contexts and ownership (PROPOSED)

| Context | Owns | Outside its authority |
|---|---|---|
| CRM | Counterparty profile / customer relationship, outlet, contacts, dedupe lineage | AR ledger, sales orders, staff login |
| Field Force | Operational agent profile, supervisor association | Global identity and HR payroll |
| Territory | Hierarchy, geographic/list-based coverage, effective-dated assignments | Tenant memberships |
| Route / Visit | Plan, stop, check-in/out, outcome, evidence references | Product stock and sales order posting |
| Distribution | Delivery assignment, expeditor dispatch, proof/exception coordination | Financial posting and inventory movement |
| Performance | KPI rule versions, results, bonus proposals | Payroll execution |
| SalesDoc Adapter | External ID mapping, sync cursors, reconciliation | Canonical ERP domain ownership |

Customer is a commercial relation to a Counterparty, not a second disconnected legal identity. **Open conflict:** whether Counterparty master ultimately belongs to CRM or a shared Party service serving Purchasing/Finance. Validate tenant **and** branch/resource scope for every read, write and cross-domain reference.

## 3. Logical entities (PROPOSED, not SQL)

- **Counterparty:** immutable opaque ID, person/organization, legal name, jurisdiction-specific identifiers, contacts, status, audit, tenant-scoped merge/dedupe lineage.
- **Customer:** Counterparty ID, relationship state, segment, account owner; no mutable ledger balance.
- **SalesOutlet:** customer ID, physical/virtual point, code, address, optional coordinates, contact, status; one customer can have many outlets.
- **Contact:** contact channels, party/outlet link, consent/preference and PII controls.
- **FieldAgent:** Identity/HR reference, role/team/branch eligibility, active dates.
- **SupervisorAssignment:** supervisor-to-agent relationship with effective dates and audit.
- **Territory:** named hierarchy, optional geometry/polygon, lifecycle and effective date; geo polygons not required at MVP.
- **TerritoryAssignment:** territory/agent/team/outlet association, effective dates, overlap/conflict controls.
- **RoutePlan / RouteStop:** date and business timezone, agent, version/state; ordered stops, outlet and planned visit windows.
- **Visit:** agent/outlet, route reference optional, workflow state, purpose, check-in/out and server/device timestamps.
- **LocationEvidence:** purpose, coordinates only if authorized, accuracy/source, device time/server receipt, integrity flags.
- **PhotoReport:** visit reference, type, private object key, checksum, captured/uploaded times, retention/moderation.
- **DeliveryAssignment:** authoritative Sales fulfillment reference, expeditor, route, delivery window, operational state.
- **ProofOfDelivery:** delivery reference, accepted quantity, recipient evidence, exceptions, timestamps and sync status.
- **KPIDefinition:** versioned formula metadata, targets, source, applicability, period/timezone/effective dates.
- **KPIResult / BonusProposal:** rule version, source lineage, amount/score, review/approval, corrections; not payroll/finance posting.
- **ExternalMapping / SyncJob:** tenant-scoped vendor ID to canonical ID, sync cursor, retry, errors, mapping version.

Every mutable entity has tenant scope, version, lifecycle status and timestamps; branch access is separately authorized. Preserve historical assignments; don't silently delete transaction-referenced parties.

## 4. Main workflows (PROPOSED)

**Customer onboarding:** search duplicates → validate counterparty/contact → create customer/outlet → approve if policy requires → effective-dated agent/territory assignment → publish changed-entity event. Merge needs permission, review and lineage.

**Agent/supervisor:** Identity owns authentication; CRM stores only operational profiles and effective-dated roles/teams/coverage. Reassignment must preserve prior sales/visit/KPI attribution. A supervisor only views permitted team data.

**Territory/route:** area may be customer-list-based or geographic. Route suggested states: DRAFT → PUBLISHED → IN_PROGRESS → COMPLETED, with CANCELLED/REVISED branches and audit. Manual order first; route optimization optional and not approved. Publish validates branch, agent, outlet, schedule and conflicts. Revision to an offline device requires explicit conflict handling.

**Visit/GPS:** suggested PLANNED or AD_HOC → CHECKED_IN → CHECKED_OUT → SUBMITTED → VERIFIED, plus REJECTED/NEEDS_REVIEW/CANCELLED. GPS capture only for lawful, disclosed work purposes; default excludes off-duty continuous tracking. Differentiate missing GPS, inaccurate GPS, out-of-radius and suspected spoofing—these are not automatic fraud determinations. No geofence distance, accuracy threshold, legal basis, retention schedule or required-photo policy is yet approved.

**Mobile offline:** propose encrypted device queue, generated idempotent command IDs, ordered visit mutation sync, server transition and permission revalidation, visible pending/conflict state and private resumable media uploads. Do not let offline acceptance bypass Sales credit/stock decisions.

**Distribution:** Sales creates fulfillment-ready request → dispatcher assigns expeditor / optional delivery route → delivery evidence/exception submitted → Sales confirms quantities, returns and stock → Finance handles authorized collections/AR. Operational states suggested: READY → ASSIGNED → IN_TRANSIT → DELIVERED, with PARTIAL/FAILED/RETURN_REQUESTED/CANCELLED. Fleet management and cash-on-delivery module boundary are open.

**KPI/bonus:** potential metrics include planned/productive visits, coverage, verified visits, net accepted sales, new active outlets, successful deliveries, returns and collection quality. Formulas, targets, exclusions and payout rates remain OPEN. Preserve KPI policy version, source event lineage, provisional vs finalized periods. Bonus proposal → review → approval → HR/Finance handoff → payment owner confirmation. Corrections use append-only adjustments, not silent overwrites.

## 5. Inter-module data flows (PROPOSED)

| From → To | Information |
|---|---|
| Tenant / Identity → CRM | tenant membership, branch access and actor permissions |
| CRM → Sales / Distribution | CustomerChanged, OutletChanged with canonical IDs |
| Visit → Sales | CreateSalesOrder request with idempotency key; Sales owns result |
| Sales → Distribution | FulfillmentReady, OrderChanged, DeliveryCancelled |
| Distribution → Sales | DeliveryEvidenceSubmitted, DeliveryExceptionReported |
| Sales / Finance → KPI | authoritative net sales, returns, AR/collection snapshots |
| Visit / Distribution → KPI | verified visits, routes and delivery outcomes |
| KPI → HR / Finance | approved bonus proposal / adjustment for acceptance |
| CRM / Distribution → SaaS Billing | deduplicated *candidate* metering event; Billing owns rate, wallet, discount and negative-balance rules |
| External SalesDoc ↔ Integration | mapped IDs, import/export, cursor, errors, reconciliation |

Recommend tenant-scoped outbox/inbox, event schema version, event ID, aggregate version, occurred-at UTC, correlation ID, actor and minimal payload. Never broadcast raw GPS, photos or tax IDs. Support retries/reconciliation without duplicate business documents or duplicate SaaS charges.

## 6. Security, privacy, scale (PROPOSED)

Enforce role + tenant + branch + assignment + purpose across APIs, offline sync, exports, evidence and events. Encrypt device cache, transport and private object data; signed time-limited media access, checksum, malware/EXIF handling, audit high-risk access. Collect minimal GPS only while authorized; implement legally required notice, retention and access/delete controls. Retention, media quotas and exact policies remain OPEN.

Prefer module boundaries compatible with a modular monolith if platform approves. Scalable sync/media/integration/KPI workers; no local-only persistent state. Index access by tenant/branch/outlet/agent/status/time; Database team owns storage decisions. Store photo bytes outside transactional rows; location precision and archival policy require review. Monitor event lag, sync conflicts, route completion, uploads, delivery exceptions and KPI recomputation without leaking sensitive coordinates.

## 7. Architectural Decision Records (all PROPOSED)

- [ADR-CRM-001 — Domain ownership](../adr/ADR-CRM-001-domain-ownership.md)
- [ADR-CRM-002 — Offline visits, GPS and privacy](../adr/ADR-CRM-002-offline-gps-privacy.md)
- [ADR-CRM-003 — SalesDoc anti-corruption adapter](../adr/ADR-CRM-003-salesdoc-adapter.md)
- [ADR-CRM-004 — Effective-dated assignments](../adr/ADR-CRM-004-effective-dated-assignments.md)
- [ADR-CRM-005 — KPI and bonus separation](../adr/ADR-CRM-005-kpi-bonus-separation.md)

## 8. OPEN QUESTIONS

1. Counterparty canonical master owner (CRM vs common Party) with Finance/Purchasing?
2. Overlapping outlet assignments to branches, agents, territories; precedence?
3. Counterparty dedupe/merge, approval and cross-branch visibility policy?
4. Route publish/reorder authority and offline revision conflicts?
5. Visit verification, geofence accuracy/radius, GPS lawful basis/retention and work hours?
6. Delivery vs standalone Logistics boundary; fleet, cash on delivery and returns?
7. KPI formulas, bonus eligibility, reversals, approvers and period locking?
8. SalesDoc API auth, supported endpoints, rate limits, versions and data ownership—**unverified**.
9. Usage metering taxonomy, photo quotas, privacy/SLO/performance limits and corporate timezone?

## 9. Shared documentation changes — SUGGESTED ONLY, NOT APPLIED

After approval, register ADRs in `DECISIONS.md`, domain ownership in `MASTER_ARCHITECTURE.md`, accepted functional/privacy rules in `REQUIREMENTS.md`, interfaces in `MODULE_REGISTRY.md`, unknowns in `OPEN_QUESTIONS.md` and dated changes in `CHANGELOG.md`. Do not overwrite other chats' documentation.

## 10. CODEX SPECIFICATION — FUTURE IMPLEMENTATION ONLY

No code now. After explicit approvals Codex should deliver:

1. Reviewed entities, state transitions and versioned contracts; stable IDs and tenant/branch-aware authorization.
2. CRUD/command/query interfaces for party/outlet, agent assignments, territories, routes, visits, photo/GPS evidence, dispatch/POD, KPI/bonus approvals, sync/reconciliation; cursor pagination and field permissions.
3. Reliable outbox/inbox event delivery with retries, idempotency, dead-letter/quarantine, audit and correlation.
4. Encrypted, revocable mobile offline cache and sync queue; conflicting revisions go to review, not silent acceptance.
5. Protected geospatial/media storage, minimum collection, limited-time object access, retention and redaction.
6. Versioned KPI engine, period snapshots, reversible adjustments, authority-respecting HR/Finance bonus handoff.
7. SalesDoc adapter only after verified API contract, with external ID mappings, backfill/replay and reconciliation.
8. Cross-module contract tests and load/security/recovery tests. Block cross-tenant/branch access; no duplicate visits/orders/POD/usage charges; never directly post ledger or stock from CRM; historical attribution remains immutable.

**Definition of Ready:** ADR approvals, Party owner, access/branch model, route/visit/offline/GPS policy, Delivery vs Logistics owner, KPI/bonus rules, SalesDoc contract and Billing metering categories. This specification is NOT authorization to implement.

## 11. Cross-module reconciliation: AR-002, AR-008, AR-011 (PROPOSED; 2026-10-12)

**Authority boundary:** Quyidagi jadval yangi biznes qoida yoki `APPROVED` ADR emas. Ushbu hujjatning avvalgi bo‘limlaridagi keng talqinlar bilan farq bo‘lsa, amaliy implementatsiyada masala `OPEN_QUESTIONS.md` va yakuniy user approval orqali hal qilinadi.

| Entity / fact | Candidate canonical writer | Other modules may do | Explicit OPEN question |
|---|---|---|---|
| Product, SKU, UoM, product hierarchy | Separate logical Catalog boundary in modular monolith | Sales references SKU; Inventory owns physical stock; CRM owns assortment/market relations; Purchasing references product | Tenant-global vs company-specific catalog, Catalog vs Sales container owner |
| PriceList, applied price snapshot | Pricing boundary (proposal) | Sales snapshots price at order transition; CRM owns customer segment context | Pricing rules and company/branch scope |
| SalesOrder and document lifecycle | Sales | CRM captures demand/visits; fulfillment coordinates Inventory | Approval vs posting/dispatch consequences |
| StockMovement/StockReservation | Inventory | CRM shows authorized read-only availability | Stock policy and reservation TTL |
| Counterparty/Party identity | **OPEN:** CRM-managed canonical Party registry versus independent shared Party context | Customer/Supplier/Outlet/Payer are role records and relationships; Finance owns AR/AP, not Party identity | Authoritative master, deduplication, legal identifiers, company visibility |
| Customer/Outlet/Route/Visit | CRM & Distribution | Sales receives immutable references; Finance may map payer/receivable owner | Outlet legal entity versus delivery location; payer vs customer |
| Payer, ReceivableOwner | **OPEN:** canonical Party role/reference mapping | Finance owns receivable and allocation ledger, Integration maps external payer ID | Inter-company payer and allocation policy |

**Proposed Party contract:** `party_id` (immutable tenant-scoped) + `party_role` (customer, supplier, payer) + `outlet_id` (physical trade/sale point, not automatically a legal payer); external mapping uses `source_system + external_type + external_id -> canonical_id`, unique per tenant with audited merges and alias history. Never silently merge payable/receivable identities. Dedupe candidates require scoped review and preserve immutable sales and finance document references.

**Integration evidence taxonomy:** `DOCUMENTED` means public provider documentation describes the feature, **not** that a live server was tested; `LIVE_VERIFIED` requires recorded endpoint/environment/test date/result; `INFERRED` is interpretation; `UNVERIFIED` has no sufficient evidence. The CRM adapter may read `DOCUMENTED` mappings without representing them as production-tested. No claim of live third-party API tests is made here.

**Contract examples (PROPOSED):** `PartyLinkedToCustomer`, `OutletAssignedToAgent` and `ExternalCounterpartyMapped` events include tenant scope, immutable IDs, event version, idempotency key and source time. Finance creates its own ledger postings; CRM does not update AR/AP balances. Every authorized search or merge operation enforces tenant/company/branch grants.

**Approval gate:** domain owners 00/04/05/06/07/12 must agree on one canonical Party and Catalog owner and versioned interfaces; user must approve any business consequence. AR-002 and AR-008 remain **OPEN**, AR-011's terminology is reconciled here as **PROPOSED**.


## 12. ROUND 2 CORRECTION — AR-002 / AR-008 / AR-011; MR-09 / MR-13 (2026-10-12)

**Status: PROPOSED; architectural reconciliation only.** §§1–11 historical material remains intact. This section specifies a single-writer recommendation, not an approved policy. Source cross-check: Round 2 review; Sales §11; Backend §17–18; Finance Accounting identity/settlement; Integration AR-008/011; UX §16; Mobile offline. Where preceding text assumes CRM owns Counterparty, that assumption is **superseded as PROPOSED**, pending owner selection; no old approval was changed.

### 12.1 Canonical identity and role distinctions

| Concept | Precise meaning | Proposed record/reference | Authoritative write candidate | Must NOT imply |
|---|---|---|---|---|
| Party | Canonical tenant-scoped person/organization identity; stable `party_id` | `PartyIdentity` + legal identity versions + aliases | shared logical Party context **recommended**, CRM-owned alternative OPEN | Legal debtor, SaaS subscriber, payer or global cross-tenant identity |
| Counterparty | Business relationship/role of Party toward a legal entity/company; may buy/supply/settle | `CounterpartyRelation(party_id,company_id,role,effective_from/to)` | Party relationship writer candidate (shared) vs CRM-owned; OPEN | Necessarily same as Customer or AR debtor |
| Customer | CRM-managed selling relationship, status/segment/coverage | `CustomerRelationship(party_id,company_id?,branch_scope?)` | **CRM** for customer relationship only | Ownership of legal identity or GL |
| Supplier | Purchasing counterpart role | `SupplierRelationship(party_id,legal_entity_id)` | Purchasing (role only) | CRM customer status |
| Outlet / SalesOutlet | Physical/virtual commercial service/visit/delivery location, optionally operated by another Party | `Outlet(outlet_id,customer_relation_id,operator_party_id?,delivery_address_id?)` | **CRM / Distribution** | Legal debtor or Payer; company Branch is a different tenant organization object |
| Payer | Party **actually remitting** settlement; link to obligation and remittance, can be third party | `PayerInstruction/PaymentRemitter(payer_party_id,invoice_id,authorization_ref)` | **Finance** for ERP settlement; **Billing** for SaaS subscription obligation | Invoice legal debtor, credit liability or SaaS wallet holder |
| ReceivableOwner | Legal debtor obligated on posted invoice/open item, legal entity and snapshot immutable | `ARObligation(debtor_party_id,legal_entity_id,invoice_id,legal_snapshot)` | **Finance** alone | Outlet/customer label or payment sender |
| External ID | Provider-specific opaque ID, not canonical `party_id` | `ExternalIdentityMap` | **Integration** adapter mapping authority | Authorization or identity proof |

**Concrete distinction:** one Party may be Customer to company A, Supplier to company B, operate 3 outlets, pay another party's invoice with authorization, while a separate debtor remains `ReceivableOwner`. Company's customer relation and branch/outlet assignment do not grant cross-company or cross-tenant data access. Tenant/company/legal entity/branch/BillingAccount IDs are distinct from Party identity.

### 12.2 Party writer decision packet (OPEN — AR-008, MR-09)

| Option | Single authoritative writer | Benefits | Costs / failure modes | Status |
|---|---|---|---|---|
| A — Shared logical Party context **RECOMMENDED** | PartyIdentity service *within* modular monolith initially, one write API and event source; no requirement for independent microservice | Consistent CRM/Purchasing/Finance legal identity references; role-neutral merge and audit | New bounded context, arbitration between legal entities, complex scoped search | PROPOSED / user approval OPEN |
| B — CRM-owned Party master | CRM Party writer exclusively; Finance and Purchasing consume versioned contracts | Fewer initial modules, simpler onboarding | CRM becomes cross-business identity authority; high coupling and privilege concentration | PROPOSED alternative / user approval OPEN |
| C — Independent masters with reconciliation | Separate CRM/Finance/Purchasing writers | Local autonomy | Ambiguous identity, broken dedup and debtor mapping; **not recommended** | Alternative, not APPROVED |

**Invariant (PROPOSED):** whatever choice, exactly one authoritative `PartyIdentity` writer per tenant scope; consumers hold references + immutable legal snapshots, never independently mutate master. Shared logical Party is a design boundary, not automatic deployment approval. Legal ownership of Party, cross-company visibility and tenant migration semantics remain **OPEN**.

### 12.3 Identity lifecycle, matching, merge/split and evidence — PROPOSED

| Operation | Preconditions and authority | Effects / safeguards | OPEN decision |
|---|---|---|---|
| Create | Verified tenant + company context; permission, normalized legal name/IDs where applicable; candidate search | Mint immutable ID; possible duplicates flagged; `created_from` trace; no public global directory | Mandatory legal ID / verification by jurisdiction |
| Match | Scoped exact identifier match; normalized phone/name/address fuzzy suggestions are **candidates only** | `POSSIBLE_DUPLICATE` not automatic join; evidence/provenance and confidence source recorded; no false legal assertions | Auto-match thresholds, human reviewer role |
| Merge | Review of legal identity, role history, invoice obligations, account access and source evidence; two-record revision lock; authorized reviewer | Surviving `party_id`, immutable alias `superseded_party_id → surviving_party_id`, audit/reason; external maps re-resolved; no deletion of original invoices/AR legal debtor snapshots | Legal-entity or cross-company merge permission, reversibility window |
| Split / mistaken merge | Explicit privileged correction workflow; separate newly minted identity and scoped affected references | Append `PartySplit` correction lineage, replay/reconcile only non-posted references where safe; Finance reallocation only via Finance-controlled compensating process | Who may reclassify posted docs and what evidence suffices |
| Cross-company relationship | Same Party can have independent company-scoped customer/supplier/credit roles | Distinct agreements, taxes, AR and visibility checks; links never imply data sharing | Which legal entities may reuse Party master within tenant |
| Cross-tenant duplicate | Search and compare only inside authorized tenant boundary | No implicit cross-tenant match/merge, inference via search counts or shared evidence | Enterprise isolation/migration special process |
| External mapping | unique `(tenant_id,provider,connection_id,external_entity_type,external_id)` + optional company scope, canonical ID and mapping version | Append correction/audit, 1:N alias history, quarantine one external ID → conflicting canonical IDs; detect duplicate import by source occurrence | External ID reuse and cutover backfill strategy |

Legal identity is versioned with `jurisdiction`, `identifier_type`, `identifier_value` protected appropriately, source, `verified_at` and `verification_method`; an integration import or matching score is **not** legal verification. Preserve original issuer/recipient/debtor/tax identification on Finance posted documents even after Party merge. Exports and global search require row-level tenant/company authorization and redaction.

**Contract proposal:** `PartyCreated.v1`, `PartyIdentityCorrected.v1`, `PartyMerged.v1`, `PartySplit.v1`, `CustomerRelationshipChanged.v1`, `OutletAssignmentChanged.v1`, `ExternalPartyMapped.v1`. Envelope: `event_id`, `business_occurrence_id` (distinct), `schema_version`, `tenant_id`, `company_id?`, `party_id`, `aggregate_version`, `occurred_at`, `committed_at`, `correlation_id`, `payload_digest`. Local writer transaction + outbox, consumer inbox/dedup and same-key/different-payload conflict review; no distributed ACID assumption. Finance authoritative debtor snapshots stay unchanged.

### 12.4 Strict Catalog/Pricing/Inventory boundary — AR-002

| Master/fact | Single logical writer **PROPOSED** | CRM access |
|---|---|---|
| Category, Product, SKU/variant, UoM, conversions | Catalog (logical context; 05 container vs standalone OPEN) | Read-only catalog reference; CRM may own outlet-specific assortment relationships but not SKU |
| PriceList, pricing rules, resolved quote | Pricing | Display versioned quote/snapshot returned through Pricing/Sales; no CRM price authoring |
| SalesOrder/commercial confirmation | Sales | Creates request with customer/outlet/agent/visit IDs and idempotency key; displays authoritative server result |
| Stock reservation, StockMovement, inventory valuation | Inventory | Scoped availability projection only; no CRM stock decrement |
| Supplier PurchaseOrder | Purchasing | Supplier role reference only |
| ERP GL, AR, payer allocations, credit notes | Finance | Read authorized projections; no ledger changes |
| SaaS seat/usage charge, wallet | Billing | Emits candidate usage facts only; no rate/credit computation |

No phrase “Sales owns products/inventory/prices” should be interpreted as one aggregate writer. Catalog namespace and Product-vs-SKU identity remain OPEN in Sales/Backend.

### 12.5 SalesDoc evidence boundary — AR-011

| Evidence label | What it actually proves | Required metadata | Forbidden interpretation |
|---|---|---|---|
| `DOCUMENTED` | Public provider documentation describes schema/behavior at source revision | URL/source title, accessed date, relevant document revision/quote, applicable endpoint/version | Not a successful API execution |
| `LIVE_VERIFIED` | Specific authenticated endpoint exercised in a named environment with result | Env (sandbox/production), URL/path, test time, request/response sanitization, status/result, verifier, scope | Not verification of all API versions/accounts |
| `INFERRED` | Analyst assumption based on indirect evidence | Rationale/source and uncertainty | Not a provider guarantee |
| `UNVERIFIED` | Evidence missing, stale, unauthorized or untested | Blocker and verification owner | Not deployable as proven integration |

**Current CRM claim:** no SalesDoc endpoint, production authentication, scope, quota or behavior has been live-verified in this correction. `ExternalIdentityMap` is a proposed contract, not evidence of live sync. Integration is evidence and adapter owner; CRM owns business role meanings. Imported external documents have source IDs and history; Sales/Billing decide any billable occurrence under their OPEN lifecycle policies, not CRM.

### 12.6 Field-worker GPS, visit and photo privacy decision packet — MR-13

| Topic | Options considered | Recommendation (PROPOSED) | OPEN/legal gate |
|---|---|---|---|
| Legal basis/consent | Explicit opt-in; other jurisdiction-authorized employment/business basis with clear notice; strict refusal of collection | Determine lawful basis per country; explicit transparent notice and scope; do **not** assume consent always sufficient or required | Legal/privacy review, withdrawal handling, local labor law |
| Tracking window | Check-in/out only; approved on-duty intervals; continuous background | Check-in/out purpose-bound by default; no off-duty or silent background tracking | Need for route monitoring, worker representation |
| Accuracy/geofence | Strict threshold/block; confidence band + review; manual location-free exception | Capture accuracy and source; low accuracy/spoof/out-of-geofence → review, not automatic fraud verdict | Numeric tolerance, fallback approver |
| Photos/media | Mandatory every visit; exception-based photo; optional capture | Purpose/category scoped minimal capture, no default mandatory human-face photography; private access, checksum, EXIF minimization | Which visit/delivery types require evidence and consent |
| Retention | Fixed global TTL; purpose/regional schedule; indefinite | Purpose+jurisdiction retention schedule; separate GPS/media/audit retention | Exact durations, legal holds, data-residency scope |
| Erasure | Immediate purge all; scoped delete/anonymize; legal hold exception | Subject request intake, erase/anonymize when lawful, audit minimally, preserve restricted legal holds and posted Finance evidence as required | Controller roles, appeal/export SLA, legal exceptions |
| Mobile offline | Local plaintext; encrypted queue with bounded TTL | Encrypt cache, require device/user authorization on replay, cleanup after ack under approved retention; no promise of immediate offline remote wipe | Offline TTL, revoked-device pending evidence processing |

**Minimal evidence fields:** `evidence_id`, `visit_id`, `tenant/company/branch`, `purpose`, `captured_at_device`, `received_at_server`, `accuracy_m?`, `source`, `integrity_status`, `consent_or_legal_basis_ref?`, `retention_policy_version`, `media_object_ref?`, `access_audit`. Never fabricate missing GPS; separate `NOT_COLLECTED`, `PERMISSION_DENIED`, `LOW_ACCURACY`, `SUSPECTED_SPOOF`, `VERIFIED`. Do not place raw coordinates or biometric content in shared events, KPI exports or observability logs.

### 12.7 UX-T01…T20 and Mobile alignment (MR-13)

UX's twenty-task quality gate is **PROPOSED, not tested**. CRM owns the following workflow segments; other UX tasks remain with their listed owners.

| UX task | Role/action path | CRM and other owner boundary | Mobile/offline/error acceptance (PROPOSED) |
|---|---|---|---|
| UX-T01 | Seller: Today → new sale → customer/outlet → items → review | CRM resolves authorized customer/outlet; Catalog resolves SKU; Pricing/Sales resolves price and submits order | Clear draft vs server-accepted; stale price blocks final; duplicate request same operation no duplicate order |
| UX-T02 | Seller: global search → order | CRM exposes scoped party/outlet labels; Sales owns order search | No unauthorized party/order hit, count, cached recent or deep-link leak |
| UX-T08 | Seller: search or create customer/outlet | Party proposed canonical lookup; CRM relationship/outlet create; duplicate candidate is review item | Scoped search, duplicate warning with no disclosure; offline provisional ID mapped on sync; rejects revoked rights |
| UX-T09 | Field agent: Today → route → outlet → check-in → visit → check-out | CRM owns route/visit/evidence, Mobile owns queue; Security rechecks context | Show GPS unavailable/accuracy/review; pending ≠ verified; stale route or scope → conflict; no fabricated GPS |
| UX-T10 | Field agent: visit → order draft → sync queue | CRM context IDs; Sales owns order acceptance; Mobile owns offline queue | `operation_id`, `base_version`, `client_created_at`; server reprices/validates stock; no offline GL/stock/Billing charge |
| UX-T11 | Expediter: manifest → delivery → POD/photo/signature | Distribution assignment, Sales/Inventory physical reconciliation, Mobile media upload | Pending upload distinguished from accepted delivery; checksum and retry; one POD occurrence; missing media remains pending |
| UX-T12/T15 | Accountant: customer AR/financial report | CRM label only, Finance owns debtor/AR and snapshot | Payer is not automatically debtor; authorized scope + snapshot as-of timestamp |
| UX-T16…T20 | UX/admin/other roles per UX canonical matrix | CRM participates only for permitted party/outlet links and role-filtered search | No assumption of ownership of unrelated tasks; locale/accessibility and context quality gate applies |

**Mobile contract:** scoped cache `tenant > company > branch > user > assignment > device`; atomic local command+queue; stable `operation_id`, versioned payload, device/source time plus server receipt; bounded encrypted storage; retry/backoff, cursor tombstones, reauthorization and visible accepted/rejected/conflict states. Rejected rights cannot be overridden by local role claims. Offline visit/order is never an authoritative Sales confirmation, Stock posting, Finance posting or SaaS Billing charge.

**Testing specification, not executed:** wrong-tenant search denied including result counts; party merge under open invoice preserves Finance legal snapshot; cross-company assignment blocked without grant; duplicate external mapping quarantined; ID replay same/different payload; stale offline visit review; revoked device reject; GPS missing/low accuracy displayed distinctly; orphan photo uploads cleaned under defined policy; UX-T01/08/09/10/11 with uz/en/ru and keyboard/mobile accessibility. Numeric usability/performance targets from UX §16 remain PROPOSED/unmeasured.

### 12.8 Approval and sync packet (NOT APPLIED)

**OPEN user choices:** Party shared vs CRM writer; Party identity legal fields and merge/split reviewer; cross-company reuse; external ID reassignment; Outlet↔Customer/Party cardinality; employer/worker lawful GPS basis and consent, geofence tolerance, retention/erasure/exception; mandatory photo scenarios; route/visit verification; UX task acceptance thresholds; SalesDoc live API proof. Coordinate with Finance (AR debtor), Integration (ID/evidence), Sales/Backend (Catalog, Pricing, Inventory), Security/Mobile (authorization/offline) and UX (T01–T20).

Central `DECISIONS.md`, `MASTER_ARCHITECTURE.md`, `REQUIREMENTS.md`, `MODULE_REGISTRY.md`, `OPEN_QUESTIONS.md`, `CHANGELOG.md` require **separately proposed** changes after approval; not modified here. No code; no production tests claimed.

---

## 13. Delegated Party master and MVP CRM boundary (2026-10-12)

[ADR-010](../adr/ADR-010-mvp-delegated-baseline.md) selects **PROPOSED** single tenant-scoped *shared Party logical bounded context* (not CRM Sales/Product/GL/Wallet dual ownership). CRM owns Customer/Outlet/territory/visit/agent relationship and consumes canonical Party; Finance owns immutably snapshotted LegalDebtor/AR obligation; Integrations stores scoped CS_id/SD_id/code_1C external links only. SaaS Billing Company LegalEntity debt vs third-party funder from **user APPROVED** ADR-BILL-011 is distinct from customer payer/remitter in ERP Finance. Party merge/split is audited, cross-Company legal settlement requires law/authorization; GPS/media default disabled without lawful basis, retention and device policy. SalesDoc documented API is not LIVE_VERIFIED. Party ownership needs joint technical sign-off before schema writer. [Handoff](../handoff/REMAINING_GATES.md).
