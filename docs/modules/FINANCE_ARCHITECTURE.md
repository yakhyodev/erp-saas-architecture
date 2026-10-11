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
