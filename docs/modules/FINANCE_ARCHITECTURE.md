# FINANCE_ARCHITECTURE.md
Status: PROPOSED (except explicitly approved business requirements)
Scope: ERP Finance & Accounting. Date: 2026-10-11.

## Domain boundaries
ERP Finance accounts for each customer's sales, purchases, cash, bank, AR, AP and financial statements. SaaS Platform Billing independently accounts for seat subscriptions, business usage, resource usage, wallet credits and SaaS charges. **APPROVED requirement:** these ledgers must remain separate. No cross-ledger direct posting.

## Proposed accounting architecture
- Double-entry journal engine: every posted journal balances debit and credit in functional currency; immutable posted entries.
- Chart of accounts configurable per legal entity; accounting periods and controlled closing.
- Source-document references, idempotent posting, transactionally consistent journal header and lines.
- Cash and bank: cashboxes, bank accounts, receipts, disbursements, transfers, bank statement import and matching.
- Receivables: customer open items, aging, partial settlement, advance payments, credit notes, write-offs with approval.
- Payables: supplier open items, due dates, partial settlements, advances and approval-controlled payments.
- Payment lifecycle: draft, authorized, submitted, settled, failed/reversed; allocate only settled funds according to policy.
- Invoice lifecycle: draft, issued, posted, partially paid, paid, cancelled via reversal/credit note. Separate business invoice from SaaS invoice.
- Multi-currency: transaction currency, functional currency, rate source/time, realized/unrealized FX gains and losses.
- Branch dimension: branch-coded entries with legal entity boundary; interbranch settlement requires explicit policy.
- Audit: actor, tenant, legal entity, timestamps, source and reason; no deletion of posted journal records.
- Reconciliation: bank-to-cash, AR/AP-to-GL, inventory-to-GL, suspense accounts and period-end discrepancies.
- Financial reports: trial balance, balance sheet, P&L, cash flow, AR/AP aging, branch reporting.

## Integrations
Sales/Purchase/Inventory own operational documents; Finance owns accounting policy and postings. Tenant/Security owns identity and permissions; Database owns isolation and integrity; Billing owns SaaS wallet and usage ledger. Domain events must be idempotent and replay-safe.

## Invariants (PROPOSED)
1. No cross-tenant postings.
2. Debits equal credits for each posted journal.
3. Posted journals are immutable; reversal and corrective entries retain linkage.
4. Closed periods reject postings unless reopened by authorized workflow.
5. Original currency, conversion rate and functional-currency amounts remain auditable.
6. Every payment allocation refers to a valid open item; allocations cannot exceed permitted outstanding balances.
7. Finance and platform billing ledgers never share balances or entries.

## Open questions
Legal entity versus branch accounting boundaries; Uzbekistan-first or international tax jurisdictions; functional currency; posting triggers; FX rate policy; close/reopen authority; interbranch clearing; bank integration scope.

## Codex architecture specification (no code)
Define modules for accounting policy, journal posting, cash/bank, receivables, payables, payment orchestration, invoices, FX, reconciliation and audit. Specify entities, state machines, domain events, transactional boundaries, tenant-scoped permissions, idempotency, reversal workflows, financial reporting read models and invariant tests before implementation. Do not hardcode unresolved accounting or tax policies.

## ADR links
See docs/adr/FIN-001.md and docs/adr/FIN-002.md. All implementation choices remain PROPOSED pending approval.

## Reconciliation addendum — AR-004 / AR-005 / AR-008 / MR-06 (2026-10-11)

**Status: PROPOSED; no new approval.** Reconciled against current `main` central registries, Architecture Review, Sales/Inventory, CRM and Billing. Preserve historical decisions. [FIN-001](../adr/FIN-001.md) retains its recorded `APPROVED` status, but explicit original approval provenance is **AR-004 OPEN**; this addendum does not authenticate that approval.

### Ownership and separation contract (PROPOSED)

| Concept | Candidate authoritative owner | Finance contract | Decision |
|---|---|---|---|
| Tenant / Company / Branch / LegalEntity identity | Multi-Tenant | Verify scoped IDs and effective legal entity; never infer company=tenant | OPEN: entity cardinality |
| Party / Counterparty legal identity | CRM or shared Party master | Store immutable scoped reference and legal snapshot on posted document; no independent duplicate master | AR-008 OPEN |
| Customer relationship / Outlet | CRM | Reference commercial customer/outlet; outlet is not automatically legal debtor | AR-008 OPEN |
| Payer | Party-role mapping, owner TBD | Payment remitter may differ from invoice debtor; preserve payer ID and allocation authorization | AR-008 OPEN |
| ReceivableOwner / legal debtor | Finance owns open item and debtor obligation; canonical party ID externally resolved | Invoice debtor snapshot fixed on posting; reassignment via audited correction, never silent customer merge | AR-008 OPEN |
| SalesOrder / dispatch commercial state | Sales | Consume canonical committed facts; do not mutate Sales state | AR-005 OPEN |
| Physical StockMovement | Inventory | No physical quantity writes from Finance; consume valuation fact separately | AR-005 OPEN |
| ERP invoice / AR / GL / allocation | Finance | Sole owner of customer ERP accounting effects | PROPOSED |
| SaaS BillingAccount / Wallet / usage charge / SaaS invoice | Billing | No direct GL balance sharing, no wallet posting; optional explicit integration only | AR-001/AR-010 OPEN |

**SaaS Billing vs tenant ERP GL:** distinct identity of debtor, economic event, journal, ledger, invoice numbering, reconciliation and access boundary. Billing's 1 credit = 10 UZS, actual negative Seat/Usage for one calendar month, and absence of estimated credit limit are existing approved SaaS business rules; **none** creates ERP cash/AR or permits ERP negative stock. SaaS billing charge eligibility remains AR-010 OPEN. FIN-001 approval provenance remains AR-004 OPEN.

### Proposed Sales lifecycle → stock / AR / GL effect matrix (AR-005, MR-06)

| Transition / authoritative source | Stock effect | AR effect | ERP GL effect | Reversal/exception |
|---|---|---|---|---|
| Order DRAFT / SUBMITTED (Sales) | None | None | None | Discard/withdraw without financial posting |
| Order CONFIRMED (Sales) | No physical movement; optional reservation reduces available, not on-hand | None by default | None by default | Release reservation on cancellation |
| DISPATCH POSTED (Inventory fact from Sales fulfillment) | Once-only physical decrease; reservation consumed | No invoice AR by default | **OPEN:** inventory/COGS recognition timing and valuation policy | Compensating stock movement linked to original |
| SALES INVOICE POSTED (Finance after authorized source validation) | None; never dispatch twice | Proposed new open receivable for legal debtor | Proposed AR debit / revenue and tax credits; tax recognition **OPEN** | Credit note / linked journal reversal, no edit of posted journal |
| PAYMENT SETTLED and ALLOCATED (Finance / verified payment evidence) | None | Reduce allocated open item; support partial payment and advance | Proposed cash/bank debit and AR credit; cash recognition policy **OPEN** | Refund/chargeback creates linked compensating entries |
| CUSTOMER RETURN RECEIVED/POSTED (Inventory) | Once-only accepted quantity to inspected bucket | None until Finance credit note / refund | **OPEN:** cost/valuation reversal recognition | Original dispatch line + return qty cap; no implicit SaaS credit |
| CREDIT NOTE / INVOICE REVERSAL (Finance) | None | Reduce or reverse original open item as permitted | Linked correction of original financial effects | No physical restock without Inventory fact |
| CANCEL BEFORE POSTING (Sales/Finance as applicable) | Release reservation only | None unless already posted | None unless already posted | If posted, explicit compensation, never destructive cancel |

These are **candidate** effects, not approved accounting policy. Shipment-vs-invoice revenue recognition, tax point, cost method, split dispatch, deposits, interbranch obligations and return eligibility require user/domain sign-off. Billing sees only stable non-monetary document occurrences; it alone determines if/when a charge applies under an approved tariff.

### Proposed cross-module idempotent posting contract

| Producer → consumer | Immutable source fact | Required identity and correlation | Consumer outcome |
|---|---|---|---|
| Sales → Finance | SalesInvoiceAuthorized / SalesOrder state fact | tenant_id, company_id, legal_entity_id?, branch_id?, source_document_id, revision, transition_id, source_fact_id, occurred_at, committed_at, correlation_id, schema_version | Validate legal debtor and policy; **never** post on generic order event without authorized trigger |
| Inventory → Finance | StockDispatchPosted / ReturnStockPosted / GoodsReceiptPosted | tenant_id, stock_movement_id, source_fact_id, original_fact_id?, line_id, quantities, valuation reference/version?, committed_at | At most one GL economic effect per canonical fact + journal_kind + policy_version; valuation/timing OPEN |
| CRM / Party → Finance | PartyReferenceChanged / PartyMerged | tenant_id, party_id, canonical_version, alias lineage, effective_at | Resolve for future documents; historical debtor snapshots immutable; mapping/merge approval OPEN |
| Finance → Sales / CRM | InvoicePosted / ReceivableChanged / PaymentAllocated / CreditNotePosted | tenant_id, invoice_id, receivable_id, debtor_party_id, payment_id?, source_fact_id, version, correlation_id | Read-only operational projection; no second Finance posting |
| Sales / Finance → Billing | SourceDocumentTransitionCommitted (candidate only) | tenant_id, company_id, document_type/id/revision, transition_id, canonical occurrence_id, original_occurrence_id?, accepted_at | Billing alone rates/charges under AR-010; no ERP journal write |
| Payment provider / integration → Finance | VerifiedSettlementReceived | tenant_id, provider_reference, settlement_id, amount, currency, verified_at, payload_digest | Durable dedup; allocation and bank reconciliation; no credit on unverified notification |

**Common proposal:** inbox/outbox at-least-once delivery; stable `source_fact_id` distinct from transport `event_id`; unique accounting key `(tenant_id, legal_entity_id, source_fact_id, journal_kind, accounting_policy_version)`; same key+same payload returns prior outcome, different payload is conflict. Journal header, lines and inbox decision atomically committed within Finance's own boundary. Consumer retry never duplicates AR, GL, stock or SaaS charge. Cross-context atomicity vs eventual Finance posting remains OPEN with Backend/Database. No distributed transaction asserted. Reversal uses unique reversal fact and original posting reference; never delete history.

### Explicit unresolved business and technical choices

| ID | Decision required | Options / coordination |
|---|---|---|
| AR-004 | FIN-001 original explicit approval source | Provide dated user approval or governance resolution; do not change status unilaterally |
| AR-005 / MR-06 | GL revenue, COGS and AR trigger | Dispatch vs invoice vs delivery/acceptance; Finance + Sales + Inventory + Backend |
| AR-008 | Canonical Party owner; Payer vs debtor vs outlet | Shared Party vs CRM master; cross-party payment, merge lineage; CRM + Sales + Finance |
| FIN-OPEN-TAX | Tax jurisdiction, tax point, tax account mapping | Uzbekistan-only vs multi-jurisdiction; legal review/business consent |
| FIN-OPEN-FX | Functional/reporting currency and rate source | Per legal entity vs company, rate timestamp and rounding; business consent |
| FIN-OPEN-PERIOD | Fiscal calendar, close/reopen, late corrections | Legal-entity calendar, posting date and controls; business consent |
| FIN-OPEN-ENTITY | Company/LegalEntity/Branch accounting books | Single legal book with branch dimension vs separate legal books; Multi-Tenant sign-off |
| FIN-OPEN-ATOMICITY | Cross-domain posting consistency | Synchronous same-DB orchestration vs durable outbox/inbox eventual posting; Backend/DB review |

**Codex gate:** documentation-only until AR-004/005/008 and fiscal policy decisions are resolved and explicit approval is recorded. No code or guessed defaults.

## Round 2 correction — AR-001/004/005/008; MR-06/07/09 (2026-10-12)

**Status: PROPOSED contracts / OPEN business choices.** Round 2 review and Sales, CRM, Billing, Database, UX were consulted. This section refines earlier proposals without changing their historical statuses.

### 1. Strictly separate financial domains

| Boundary | Tenant ERP Finance | SaaS Platform Billing |
|---|---|---|
| Obligation | Customer/supplier legally recognized receivable/payable | Contracted SaaS BillingAccount liability |
| Source | ERP invoice, approved purchase liability, settlement, valuation | Approved Seat, Business Usage, Resource Usage occurrence, verified wallet top-up |
| Writer | Finance for GL, AR/AP, cash/bank, allocations | Billing for wallet, credits, usage rating, SaaS charges/invoices |
| Balance | Functional currency by accounting book/legal entity; tenant-scoped | Credits by BillingAccount/Wallet; tenant/company/branch source attribution |
| Reconciliation | GL↔AR/AP↔cash/bank↔Inventory valuation | Accepted usage↔charges↔wallet↔provider settlement |
| Identity | Legal debtor, creditor and payment remitter snapshots | Billing payer mapping; not automatically ERP debtor |
| Reversal | Finance linked journal/credit note | Billing linked adjustment/refund only under approved SaaS policy |

**Approved SaaS business invariants unchanged:** Hybrid Prepaid; low Seat + Business Usage + Resource Usage; 1 credit=10 UZS; progressive discounts and enterprise contract; actual Seat/Usage negative for one calendar month without estimated credit ceiling. These never imply negative ERP cash, stock or receivable policy. AR-001 BillingAccount/Wallet cardinality and consolidated payer remain OPEN. No Finance journal may mutate Billing wallet and vice versa.

### 2. Authoritative accounting trigger matrix (candidate, not approved)

| Source state/event | Writer / economic fact | Physical stock | AR / AP | GL candidate effect | Status / correction |
|---|---|---|---|---|---|
| Sales draft/submit | Sales | None | None | None | PROPOSED; draft not posted |
| Sales confirm | Sales | None; optional reservation changes available only | None | No default GL | PROPOSED; credit approval policy OPEN |
| Stock dispatch posted | Inventory | Decrement once per movement leg | No automatic AR | Inventory credit + COGS debit **if recognition policy authorizes** | COGS point, valuation OPEN; linked compensating stock on reversal |
| Goods receipt posted | Inventory | Increment accepted stock once | No automatic AP | Inventory debit + GRNI/accrual credit **if policy authorizes** | Valuation/GRNI/tax OPEN |
| Sales invoice issued/posted | Finance | None | Candidate AR open item | AR debit; revenue/tax credits according to jurisdiction | Invoice issue vs final posting recognition OPEN |
| Purchase invoice approved/posted | Finance | None | Candidate AP open item | Expense/inventory/GRNI clearing debit; AP credit, tax policy OPEN | Match/tolerance/recognition OPEN |
| Verified payment settlement + allocation | Finance/payment integration | None | Reduce AR/AP allocated open items once | Cash/bank and AR/AP settlement journal; unapplied funds separately | Authorization, third-party payment and settlement timing OPEN |
| Credit note/debit note posted | Finance | None | Correct original AR/AP open item | Linked correcting journal, tax correction OPEN | Must reference original; no physical return implied |
| Customer return posted | Inventory | Accepted returned qty once into appropriate bucket | No automatic AR correction | Inventory/COGS reversal only under policy | Credit note separate; returned valuation OPEN |
| Supplier return dispatched | Inventory | Decrement once | No automatic AP correction | Inventory/GRNI or cost correction under policy | Supplier debit note separate |
| Reversal after posting | Each original economic-effect owner | Compensating movement only for prior physical movement | Finance compensating open item if original exists | Linked immutable reversal, never delete original | Prevent over-reversal and replay |

**No conflation:** confirm does not physically decrement; dispatch cannot be counted twice; invoice does not create StockMovement; Billing chargeability is an independent AR-010 decision, never inferred from Finance GL.

### 3. Recognition-policy decision packet (OPEN)

| Concern | Alternative A | Alternative B | Recommendation status |
|---|---|---|---|
| Revenue | Invoice issuance/posting | Delivery/acceptance/performance obligation satisfaction | OPEN; legal/accounting jurisdiction first |
| COGS | Dispatch posted valuation | Revenue-matched recognition after delivery/acceptance | OPEN; Inventory costing policy needed |
| Inventory valuation | FIFO | Moving weighted average / other lawful method | OPEN; cost layers, returns, negative stock coordination |
| AR | Invoice posting | Contractual performance/milestone receivable | OPEN; tax/legal terms |
| AP | Supplier invoice posting | Goods receipt accrual then invoice clearing | OPEN; GRNI and matching |
| Tax | Invoice tax point | Supply/payment-based tax point per law | OPEN; no invented jurisdiction |
| Period close | Strict close, later adjustments in next open period | Authorized reopen with audit | OPEN; fiscal calendar and tax corrections |
| FX | Legal-entity functional currency, approved rate at posting | Different lawful rate-date/revaluation basis | OPEN; source/rounding |
| Atomicity | Cross-domain same-DB ACID | Owner-local ACID + transactional outbox/inbox | **Prefer B PROPOSED**, aligns Database §18; A historical alternative, not approved |

### 4. Party, legal debtor and payer identity contract (AR-008/MR-09)

| Identity | Proposed semantic | Authoritative candidate | Financial rule |
|---|---|---|---|
| Party / Counterparty | Tenant-scoped person/legal organization identity | CRM master or shared Party domain — OPEN | Stable ID, merge aliases, legal identity snapshots |
| Customer | Commercial relationship to Party | CRM | Not automatically the legal debtor |
| Outlet | Sales/distribution delivery point | CRM | Not automatically debtor or payer |
| LegalDebtor / ReceivableOwner | Party legally liable for invoice | Finance owns AR obligation; Party identity external | Snapshot debtor party ID, legal name, identifiers and liability terms at posting |
| Payer / Remitter | Party funding a settlement | Payment evidence + Party mapping, owner OPEN | May differ from debtor; verify authorization and trace both IDs |
| SaaS payer | Contracted BillingAccount payer | Billing | Not inferred from ERP invoice debtor |

**Third-party payment proposal:** require tenant/legal-entity scope, payer identity, remittance reference, debtor authorization/evidence and explicit allocation to open item. An unmatched remittance is unapplied/suspense, never silently assigned. Party merge does not rewrite historical invoice debtor, payment payer or tax identity; preserve original snapshot, canonical alias mapping, merge approval, actor and effective time. Cross-tenant allocations prohibited; cross-company/intercompany payments OPEN.

### 5. GL posting, reversal and duplicate contract

| Contract field | Requirement (PROPOSED) |
|---|---|
| Envelope | schema_version, tenant_id, company_id, legal_entity_id, branch_id?, source_domain, source_document_id, source_revision, source_fact_id, event_id, operation_id, journal_kind, policy_version, amount/currency, effective_at, committed_at, correlation_id, payload_digest |
| Economic unique key | (tenant_id, legal_entity_id, source_fact_id, journal_kind, policy_version); reversal uses its own fact and original_journal_id |
| Inbox | Durable received/processed/conflict state; event_id is transport identity, not business occurrence identity |
| Same-key same-payload | Return original posting result; no new journal |
| Same-key changed payload | Reject conflict, operator review; never overwrite |
| Local transaction | Finance journal header+balanced lines+AR/AP open item/allocations+dedup outcome+outbox atomically |
| Cross-domain delivery | Source owner commits local facts/outbox; Finance consumes at least once, retries; no claim of distributed ACID |
| Reversal | Explicit authorization, original link, amount/currency/policy lineage, over-reversal prevention; immutable posted records |
| Recovery | Missing event, poison message, crash after commit-before-ack, out-of-order, period-closed and valuation-not-ready -> exception queue + reconciliation; never blind duplicate post |

**Database §6/§17/§18 alignment:** owner-specific local atomic transaction + outbox is the preferred **PROPOSED** candidate. A single Sales+Inventory+Finance ACID boundary is an alternative, not simultaneously mandatory. Finance records pending accounting status until posting confirmed; Sales must not display GL-posted merely because dispatch succeeded. Billing independently deduplicates canonical billable occurrences.

### 6. Finance UX acceptance contract (UX §16)

| Workflow / persona | Required UX behavior | Forbidden behavior / recovery test |
|---|---|---|
| Invoice — accountant | Draft→validate→review debtor/tax/currency/period→authorized post; show server journal ID and AR status | Timeout is UNKNOWN, not posted; retry same operation ID, display prior result, no second invoice/GL |
| AR payment allocation — accountant (UX-T12) | Show payer vs debtor, open items, remaining amounts, partial/unapplied and verified settlement | No early settlement or over-allocation; stale balances refresh and conflict resolution |
| AP payment — accountant (UX-T13) | Approval, beneficiary, settlement state, partial allocation and audit | Denied or revoked role cannot post; failed provider settlement cannot mark paid |
| Bank reconciliation — accountant (UX-T14) | Unmatched items, proposed matches, amount/date/currency differences, source evidence, reviewer decision | Never auto-post balancing entries silently; mismatch stays actionable and reversible |
| Period close — chief accountant | Preview unresolved postings, unmatched AR/AP/GL/stock, FX/tax checks, approval, close outcome | Concurrent late posting denied/queued by policy; close retry idempotent; reopening requires authorization |
| Error recovery — accountant | Explicit PENDING/POSTED/FAILED/CONFLICT/UNKNOWN, correlation ID, safe retry, authorized exception escalation | No optimistic final posting, no duplicate credit note, no cross-tenant/branch leak |

Test all with active tenant/company/branch context, permission revocation mid-flow, uz/en/ru, desktop/mobile layouts, keyboard/screen-reader, latency/offline and stale data. UX §16 targets are PROPOSED/unmeasured; no claim of passed usability tests. Finance remains implementation BLOCKED until policy and security gates approved.

### 7. FIN-001 provenance alternatives (AR-004 OPEN)

- **A: Locate evidence:** find dated explicit user approval (conversation, signed decision, issue or commit-linked approval), record immutable link, approver and date in central governance through Master.
- **B: Explicit re-confirmation:** present exact ledger-boundary text to user; capture fresh approval and date without claiming it retroactively validates original status.
- **C: Formal review:** if no evidence and user does not reconfirm, Master initiates status/provenance review; only authorized governance changes historical status with audit trail.

**Do not change FIN-001's existing APPROVED label.** Technical ledger topology and all new contracts remain PROPOSED.

**OPEN:** AR-001 payer/cardinality; AR-004 evidence; AR-005 recognition/transaction trigger; AR-008 canonical Party; tax jurisdiction/tax point, functional currency/rate source, FIFO vs average, fiscal calendar/close, legal entity vs branch books, third-party/intercompany settlement and authorized UX role scopes. See [FIN-002](../adr/FIN-002.md), [FIN-003](../adr/FIN-003.md).

---

## AR-001 — Scope-limited SaaS debtor/third-party payer approval (2026-10-12)

**APPROVED_BY_USER for SaaS MVP:** every Company corresponds to one distinct LegalEntity, BillingAccount and Wallet; that Company's LegalEntity owes its SaaS service charges even if an authorized third party funds Wallet. Tenant-shared measured Resource Usage is charged to one designated responsible Company, ordinary Company/Branch resources to their owner; Branch no independent SaaS wallet. Details: [ADR-BILL-011](../adr/ADR-BILL-011.md). ERP sales-invoice debtor and GL/AR/AP are never inferred from this SaaS contract; separate ERP GL vs SaaS wallet remains. **PROPOSED** prospective two-role assignment and immutable liability snapshots. **OPEN** legal identity verification, tax jurisdiction, remitter authority, refund/chargeback, intercompany clearing and FIN-001 original approval provenance.
