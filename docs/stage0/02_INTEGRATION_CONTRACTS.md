# CODEX 05 — cross-workstream integration contract matrix

Status: **PROPOSED spetsifikatsiya**, executable API/schema yoki test natijasi emas. Baseline: [README S02/S06–S11](README.md). Writerlar: [ownership matrix](01_OWNERSHIP_AND_REPOSITORY.md). Barcha `.v1` nomlar contract candidate; mavjud Backend nomlari saqlangan, yangi nomlar ushbu hujjatning taklifidir.

## 1. Umumiy v1 kontrakt

| Qism | Required fields / qoida | Authority |
|---|---|---|
| Command | `command_type`, `schema_version`, `command_id`, `idempotency_key`, normalized `payload_fingerprint`, `expected_revision` (state transitionda), `correlation_id`; verified tenant/company/branch context | Domain transport resolves authenticated scope; client selector authority emas |
| Server guard context | Principal/session/device, credential version, active membership/version, Company/Branch provenance, grant/policy version, action, owner resource state, assurance | 01 Identity→Tenancy→Permissions; domain critical commit boundary’da enforce |
| Event envelope | `event_id`, `event_type`, `schema_version`, `producer`, `tenant_id`, `source_company_id` (tegishli bo‘lsa), `source_branch_id?`, `aggregate_type/id/revision`, `business_occurrence_id`, `occurrence_type`, `occurred_at`, `server_committed_at`, `correlation_id`, `causation_id?`, `caused_by_business_occurrence_id?` | Factni durable commit qilgan source owner |
| Economic time | `domain_effective_at` server tasdiqlaydi; `server_received_at`, `server_committed_at`, `usage_period`, `posting_period`, `policy_effective_at` alohida | Client clock Billing oyini yoki Finance periodini belgilamaydi |
| Correction lineage | New correction occurrence, `original_occurrence_id`, `original_effect_id`, `correction_reason`, permitted delta/quantity va authorization | Original effect owner; original yozuv UPDATE/DELETE qilinmaydi |
| Public error | `code`, `message_key`, `request_id`, `correlation_id`, `retryable`, safe field errors/current revision | No secrets, foreign resource existence yoki to‘liq sensitive payload |
| Async result | Command `ACCEPTED` reference + per-effect `PENDING/COMPLETED/FAILED/PENDING_POLICY/PENDING_REVIEW`; read-model `as_of` | Source acceptance downstream stock/GL/charge completion emas |

Source Branch o‘sha Companyga, Company o‘sha Tenantga tegishli ekanligi serverda tekshiriladi. Tenant-shared meterda `source_company_id` bo‘lmasligi mumkin, lekin scope, resource owner, interval va accepted responsibility versioni majburiy. Billing keyin `responsible_company_id`, account, wallet, liable legal entity va price snapshotni o‘zi resolve qiladi. Client eventdagi Wallet ID’dan posting authority hosil qilinmaydi.

Raw token/session secret, GPS coordinates va shaxsiy hujjatlarni umumiy eventga joylashtirish shart emas. Consumerga minimal owner-approved identity reference beriladi; kerak bo‘lsa alohida scoped query orqali evidence olinadi.

### 1.1 IDlarning ma’nosi va uniqueness

| ID / key | Kim beradi | Ma’no va dedup |
|---|---|---|
| `operation_id` | Offline client, server verified namespace | Bitta offline intent; retries ayni ID va payload. New intent yangi ID, old ID link |
| `command_id` / API `idempotency_key` | Client/integration request, owner persists | Unique `(tenant, command_type, authenticated actor-or-integration scope, key)`; Company/Branch canonical payload ichida |
| `business_occurrence_id` | Source owner | Bitta semantic transition/physical leg. Unique `(tenant, occurrence_type, id)`, transportdan mustaqil, durable source recordda saqlanadi |
| `event_id` | Source outbox producer | Bitta immutable envelope. Relay retry ayni ID; upcast/republication yangi ID bo‘lishi mumkin, source occurrence saqlanadi |
| Downstream effect occurrence | Consumer owner | Originalga `caused_by_business_occurrence_id` bilan bog‘langan yangi semantic effect; source va effect ID aralashtirilmaydi |
| `correlation_id` / `causation_id` | Request chain / producing command-event | Trace, uniqueness yoki economic dedup kaliti emas |
| External key | Provider adapter | `(tenant, provider, provider_account/server, entity_type, external_id_namespace, external_id, revision)`; stable canonical mapping |

**Ikki dedup qatlami:** transport inbox `(consumer, tenant, event_id)` va biznes effect `(tenant, owner_effect_kind, source_occurrence, effect_slot)` unique. Aynan bir payload/resultga parallel retry bitta effect beradi. Yangi event yoki schema version eski effectni qaytarmaydi.

Billing/Finance ADRlaridagi policy/component versionli effect kalitlari muhim review nuqtasi: yangi tarif yoki accounting policy versionini qo‘shib, eski sourcega yana oddiy posting qilib bo‘lmaydi. **Taklif etilgan qo‘shimcha guard:** policy versiondan mustaqil stable source/effect-slot registry original applied postingni saqlaydi; policy snapshot immutable. Rerating faqat original-linked correction intent bo‘lib, o‘z correction ID’sini oladi. Bu key layout 02+01+05 sign-off’i **OPEN**; old ADR rasmiymi degan label o‘zgartirilmaydi.

**Fingerprint normalizatsiyasi:** contract versionga mos canonical field representation; pul fixed precision string/integer, vaqt UTC instant + zarur IANA timezone, aniq unit; transport `event_id`, retry counter va delivery timestamp semantik fingerprintdan chiqariladi. Optional field va schema upcast qanday solishtirilishi compatibility fixture’da belgilanadi. Same semantic key + changed meaning → `409 IDEMPOTENCY_PAYLOAD_CONFLICT` yoki quarantined event; silent overwrite/merge yo‘q.

### 1.2 Local transaction, delivery va recovery

1. Command owner: current authorization + domain invariant → local transactionda aggregate/ledger, terminal idempotency result va outbox. Commitdan oldingi crash hech qanday applied effect bermaydi.
2. Consumer: producer/type/version/scope validation → o‘z service identity va domain policy → inbox claim + semantic effect uniqueness + state/ledger + result/outbox bitta owner transactionda. Effect commitdan oldin ACK yo‘q; failed transaction inbox claimni ham rollback qiladi.
3. Relay kamida bir marta yetkazadi. Publish/ACK yo‘qolsa ayni event qayta yuboriladi; economic effect qaytalanmaydi. Global FIFO va exactly-once network delivery kafolati yo‘q.
4. Aggregate revision bo‘yicha ordering; missing predecessor `WAITING_DEPENDENCY`, bounded retry yoki source-owner snapshot reconciliation. Stale event yangiroq holatni orqaga qaytarmaydi. Effective history kerak bo‘lgan Seat/charge consumer eski intervalni yo‘qotmaydi.
5. Transient failure bounded backoff/jitter; numeric retry budget/SLO **OPEN**. Unknown schema, changed fingerprint, orphan reference, wrong tenant yoki policy conflict durable quarantine/DLQ; operator ticket va owner remediation kerak. Authorization rejection avtomatik transient retry emas.
6. Reconciliation original fact→outbox→inbox→effect→projectionni tekshiradi. Hech qachon boshqa owner ledgeriga to‘g‘ridan-to‘g‘ri fix SQL yozilmaydi; mavjud occurrence retry yoki yangi authorized correction command ishlatiladi.
7. Dedupe registry/terminal result retention **OPEN**; offline compatibility/replay/import/restore oynasidan kam bo‘lishi mumkin emas. Monthly partition detach yoki schema upgrade financial occurrence guard’ini yo‘qotmasin; nonpartitioned guard candidate. Archive/PITRdan keyin ham duplicate payment/effect saqlanadi.

**Authorization va durable fact farqi:** yangi/queued user command acceptance vaqtida joriy membership/device/grant tekshiriladi. Avval durable accepted bo‘lgan dispatch yoki settled payment faktini consumer qayta ishlaganda o‘z service scope’i, fact provenance’i va posting policy’si tekshiriladi. Keyingi user revocation legitimate committed factni yo‘qotish yoki posted ledgerni o‘chirish sababi emas. Hali qabul qilinmagan deferred user intent uchun execution-time authorization yana kerak. Bu uch holat 01+02+03 review’i va gate fixture’da alohida ko‘rsatiladi.

### 1.3 Version evolution

REST `/api/v1`; typed commands/events `.v1`; envelope version, domain payload version va price/legal policy version alohida. Additive optional field compatible; field meaning/required field/unknown enum xavfsizligi o‘zgarsa yangi major. Reader avval yangi optional field/unknown variantni safely handle qiladi, producer keyin emit qiladi. Breaking v2 parallel contract bilan, har consumer sign-off va published deprecation jadvali bilan keladi; offline client support horizon **OPEN**. Unsupported variant no side effect bilan `UPGRADE_REQUIRED/PENDING_COMPATIBILITY`ga tushadi; guessed fallback yo‘q. Ko‘rsatilgan yangi error names ham v1 catalog review candidate.

## 2. Talab qilingan integratsiya matrix’i

| ID / chain | Authoritative writer → reader | API / event candidate v1 | Economic/semantic identity | Required output va gate |
|---|---|---|---|---|
| IC-01 Identity→Membership→Authorization | 01 Identity→01 Tenancy→01 Permissions→all domain guards | `ResolvePrincipal`, `GetMembership`, `EvaluatePermission`; `MembershipChanged`, `GrantChanged`, `SessionRevoked` | Membership transition occurrence + source version; event ID alohida, no charge | Fresh scoped guard result; SEC-R2-01…13, revoke/tenant-switch tests |
| IC-02 Company/LegalEntity→BillingAccount→Wallet | 01 Tenancy + 02 legal attestation→02 Billing | `CompanyLegalLinkVerified`, `ProvisionCompanyBilling`; `BillingAccountProvisioned` | One provision occurrence per Company; effective legal link snapshot | Company:LegalEntity:Account:Wallet 1:1:1:1; no branch/pooled Wallet |
| IC-03 Membership→Seat charge | 01 Tenancy→02 Billing | `MembershipChanged`; `QualifySeat`, `PostSaaSCharge`; `SaaSChargePosted` | user×Company + policy-defined accrual slot; membership/event ID emas | One Company Seat, priced-policy gate, source interval history |
| IC-04 Catalog/Pricing→Sales→Inventory→Finance | 03 Catalog/Pricing→03 Sales→03 Inventory→02 Finance | `ConfirmSalesOrder`, `ReserveStock`, `DispatchRequested`, `StockMovementPosted`, `DispatchCompleted`, `InvoiceIssued`, `GLJournalPosted` | Order transition, dispatch/receipt leg, invoice/journal effect IDs distinct | Source/stock/GL reconcile; no confirm/invoice second stock debit |
| IC-05 Domain occurrence→Billing charge | 01/03 actual source, yoki 02 Finance source→02 Billing | Domain fact→`RecordSaaSUsage`→`PostSaaSCharge`; `SaaSUsageAccepted`, `SaaSChargePosted` | Logical document lineage + eligible transition/effect slot; source occurrence and charge distinct | Published type/unit/rate/policy; no guessed tariff/payer |
| IC-06 Offline queue→server AuthZ→economic event | 04 local outbox→01 sync/guard→03 domain (02 high-risk online-only)→02 effects | Scoped `SubmitOperation`, `GetOperationStatus`; accepted domain event v1 | Stable operation→stored command/source occurrence mapping | UNKNOWN recovery, rejected revoke, no local stock/GL/Wallet effects |
| IC-07 Party/CRM→Finance legal debtor→external | 03 Party/CRM→02 Finance; 03 non-payment adapter/02 settlement adapter | `PartyIdentityChanged`, `ResolveLegalCounterparty`, `ImportMappedFact`, `AllocateVerifiedPayment` | Canonical Party + financial debtor snapshot; provider-scoped external key | Outlet/funder/remitter/debtor distinct, authorized mapping, no live claim |

## 3. IC-01 — Identity, Membership, Authorization

- **Scope:** global principal foydalanuvchini aniqlaydi; Tenancy lifecycle va tenant/company/branch membershipni; Permissions role/grants/action evaluationni boshqaradi. Domain resource attribute va biznes invariantni o‘zi tekshiradi.
- **Payload:** authoritative principal/session/device validity; membership ID/version/state, company scope version, grant/policy version, object tenant/company/branch/status, action. Forged tenant/company selectors reject.
- **Occurrence/outbox:** membership/grant revoke yoki scope change alohida owner transition + revision/outbox. Consumer cache eviction inbox orqali version-aware; duplicate/out-of-order invalidation safe.
- **Idempotency/replay:** membership lifecycle command key+expected revision; rejected/stale payload silent rejoin bermaydi. Cache yoki eski JWT authoritative current state o‘rnini bosmaydi.
- **Correction/recovery:** role xatosi auditli yangi grant/revoke occurrence; eski approved action tarixi o‘chirilmaydi. Authority unavailable sensitive mutationga fail closed; source health qaytganda qayta authorize. Tenant switch eski cache/cursor/subscription/branchni bekor qiladi.
- **Acceptance:** cross-tenant object/list/count/search/export/file, old grant, revoked device, pool reuse va commit-time revocation race. Scope/lock/freshness mekanizmi 01 sign-off talab qiladi; numeric cache TTL/revoke SLA `OPEN`.

## 4. IC-02 — Company, LegalEntity, BillingAccount, Wallet

- **Approved scope:** BIZ-011/012 va BILL-011; alohida Company→alohida LegalEntity→alohida Account→alohida Wallet, Branch Wallet yo‘q. SaaS debtor Company LegalEntity; authorized funder uni almashtirmaydi.
- **API:** Tenancy public legal-link query va effective version; Finance legal-verification attestation; Billing idempotent provisioning. Fields: tenant/company/legal entity IDs, org revision, verification reference, effective range, requesting principal; Billing o‘zi Account/Wallet IDs beradi.
- **Occurrence/outbox:** Company legal link commit source occurrence; Billing provision effect o‘z IDsi bilan originalga link. Provision inbox + Company uniqueness + Account/Wallet+outbox owner-local transaction.
- **Replay:** bir Company uchun parallel provision bitta hisob; boshqa payload/foreign legal entity conflict. Company record mavjudligi Billing provision completed degani emas; projection `BILLING_PROVISION_PENDING` bo‘lishi mumkin.
- **Correction/recovery:** legal reorganization/identity change eski liabilityni backfill overwrite qilmaydi. Future-effective link hamda original invoices/charge debtor snapshot saqlanadi; legal novation/customer policy `OPEN`, shu yo‘l disabled. Orphan provision owner query orqali reconcile; wrong-wallet transfer auto-fix emas.
- **Acceptance:** wrong tenant/Company FK, unverified legal profile, same provision event yangi event ID bilan, Company A/B wallets, funder change debtor unchanged.

## 5. IC-03 — Membership va Seat charge

- **Approved identity:** `(tenant_id, canonical_user_id, company_id)`; branch/role/device yoki duplicate membership soni qo‘shimcha Seat emas. Different Company alohida Seat.
- **Source payload:** membership/version va company-scope transition, effective_at/committed_at, canonical user, source transition occurrence. Billing active interval historyni saqlaydi; faqat eng yangi statusga qarab o‘tgan haqiqiy kunlarni yo‘qotmaydi.
- **Qualification:** ACTIVE/day-proration existing selected candidate. SUSPENDED paid-state, activation/revoke day, timezone/cycle, price, rounding/refund `OPEN`; unconfigured statega fee taxmin qilinmaydi.
- **Economic identity:** Billing stable Seat subject + policy-defined accrual day/interval/price segment. Daily accrual va monthly summary bir xarajatni ikki marta post qilmaydi; summary projection yoki alternative posting model, ikkalasi parallel debit emas.
- **Transaction:** source membership+outbox; Billing inbox/effective interval→qualification; faqat published approved policy bilan immutable rated charge+ledger+wallet summary+outbox. Stable component registry policy version o‘zgarganda re-charge’ni to‘sadi.
- **Correction/recovery:** late/reordered revoke activation history replay’i old chargesni overwrite qilmaydi; original-linked delta, original usage month va current open posting period. Negative grace alohida clock; correctionning grace ta’siri sign-off kutadi.
- **Acceptance:** duplicate branch/device, same user ikki Company, activation/revoke/suspend same day, out-of-order backdate, 28/29/30/31/leap/DST, daily vs monthly double debit, unpriced policy. All `NOT_TESTED`.

## 6. IC-04 — Catalog, Pricing, Sales, Inventory, Finance

| Step | Source transaction / downstream | Payload va occurrence | Failure/recovery |
|---|---|---|---|
| Quote/order confirm | Catalog/Pricing read; Sales confirmation+outbox | Product/SKU/UoM refs, fixed-precision price/discount/tax-policy reference snapshot, order revision, confirmation occurrence | Stale quote/revision review; confirm stock/GL yozmaydi |
| Reservation | Inventory reservation/availability+outbox | Source order occurrence, warehouse/SKU/dimension, quantity/UoM, reservation attempt/ID | Reject/partial policy explicit; no on-hand decrement; TTL OPEN |
| Dispatch | Sales commercial dispatch request→Inventory movement/balance/reservation+outbox→Sales completion | Dispatch ID/physical leg ID, reservation ref, actual quantity, source Company/Branch, expected revision; source/effect occurrences distinct | Inventory rejection Sales pending/failed; worker timeout same leg retry; no premature COMPLETED |
| Receipt/transfer/return | Purchasing receipt or transfer intent→Inventory accepted physical legs | Canonical receipt/logical transfer, per-leg ID, original movement+bounded returned quantity | Partial legs distinct; final transfer status hamma required legsga tayangan; oversell/over-return reject |
| Fiscal invoice | Finance invoice issuance/local outbox, Sales reads reference | Commercial order lineage, issuer Company LegalEntity, legal debtor snapshot, exact currency/amount, accounting policy ref | Invoice stock kamaytirmaydi; legal/tax/recognition missing `PENDING_ACCOUNTING_POLICY` |
| GL | Finance inbox+balanced journal/lines/open-item effects+outbox | Source occurrence, journal-kind/effect slot, accounting policy snapshot, original-linked correction | Valuation/dependency/closed period pending; no half journal, no hidden source rollback |

Inventory on-hand/reserved projection bir owner local transactionda moslashadi; hot-SKU writes deterministic lock/conditional update orqali tekshiriladi. Sales orchestration progress faqat Salesga tegishli; global Sales+Stock+GL ACID va global rollback yo‘q. Source qabul qilinib Inventory rad etsa, Billingdagi confirmed-order candidate charge/refund siyosati alohida; downstream failure avtomatik refund emas.

**Replay uniqueness:** `(tenant, warehouse, physical_leg_id, movement_kind)` / bounded dimensions va semantic source registry; Finance `(tenant, legal_entity, source_occurrence, journal_kind/effect_slot)` + immutable policy snapshot. Invoice/GL ikkinchi stock effect bermaydi; policy version orqali oddiy double journal bloklanadi.

**Correction:** consumed reservation/dispatchni o‘chirish emas; authorized cancel of unposted intent yoki linked physical return, Finance credit/reversal va Billing adjustment alohida owner transactionlari. Actual quantitydan ortiq reversal/return mumkin emas. Reconciliation: requested dispatch↔posted leg↔fulfillment, accepted receipt↔stock increment, invoice↔AR/GL, stock valuation↔COGS candidate. Accounting/valuation approval `OPEN`.

## 7. IC-05 — Domain occurrence, usage, charge

- **Candidate types (BILL-014):** SalesOrder CONFIRMED; accepted GoodsReceipt POSTED; logical StockTransfer COMPLETED. Published tariff bo‘lmasa `NOT_CHARGEABLE_PENDING_POLICY`, Wallet debit yo‘q. SalesInvoice second base, PurchaseOrder, Return/Correction, count adjustment va historical import default charge disabled; bu zero-rate approval degani emas.
- **Payload:** document type/canonical ID/logical lineage, transition/source revision/occurrence, source Company/Branch, actual metric quantity/unit, server effective time, import-origin marker, original link. Resource uchun unique meter sample/interval va measured quantity.
- **Mapping:** Billing accepted account/legal link va effective policy bilan actual eligible componentni resolve qiladi; source eventdagi amount/Wallet client authority emas. One logical order re-confirm/amend event boshqa occurrence olsa ham base-lineage guard yangi base charge bermaydi. Genuine distinct receipt uchun identity va tarif classification review kerak.
- **Economic identity:** eligible source/logical lineage + stable charge component/effect slot, account scope; `charge_id` va charge occurrence source/eventdan mustaqil. Source revisionning har o‘zgarishi yangi base degani emas.
- **Transaction:** ingest inbox/usage dedup owner-local; rating ready bo‘lganda actual charge/immutable liability+price snapshot/ledger/summary/outbox atomik. Failureda source biznes fact saqlanadi, Billing pending/exception ko‘rinadi.
- **Replay/correction:** yangi event ID, policy version yoki closed-month transport retry qayta debit emas. Cancellation before trigger no charge; after charge refund faqat signed policy. Correction source/policy/original charge bilan yangi bounded delta; old ledger re-rate UPDATE yo‘q.
- **Negative period:** actual debit durable balance `>=0`dan `<0`ga tushganda BILL-013 selected anniversary-clamp candidate. Real one-calendar-month/no estimated limit `APPROVED`; rearm/expiry restriction va late correction `OPEN`. Calendar/month clockni rollout bilan default tasdiqlab bo‘lmaydi.
- **Recovery/gate:** unexplained source→charge mismatch owner review; unpriced/unattributed/unsupported policy hech qanday Wallet fallback bermaydi. Tier boundary/enterprise precedence, strict-above threshold, conversion 1 credit=10 UZS exact arithmetic, partial/import/replay gate zarur.

## 8. IC-06 — Offline operation, server authorization, economic event

- **Local writer:** 04 encrypted queue `(account, tenant, Company/Branch, device)` scope’da; `operation_id`, payload hash, expected revision, client occurred_at, local dependency refs. QUEUED/SENDING/UNKNOWN server acceptance yoki settlement emas.
- **Server pipeline:** validate session/device/revoke→current membership→Company/Branch scope→Permissions→domain revision/price/stock invariant→owner acceptance. Offline batchdagi har command va media upload alohida tekshiriladi. Scope auto-retarget yo‘q.
- **Result:** accepted command reference, canonical aggregate/source occurrence IDs va downstream effect statuses. Timeout `UNKNOWN`; status query/retry same operation/hash. Status read ham current permission bilan scoped; revoked clientga historical sensitive result leak berilmaydi.
- **Outbox/inbox:** local queue device durable; server command dedup/source state/outbox bitta domain commit; server facts keyin stock/GL/Billingga IC-04/05 bilan yetkaziladi. ACK source acceptance; effect completion alohida.
- **Replay/correction:** stale revision explicit conflict, changed intent yangi operation ID oldiga link. Revoked membership/device queued command reject; already committed facts service-authorized reconciliation orqali davom etadi. Client clock price period/grace authority emas.
- **Recovery:** cursor+delta+tombstone lokal atomic apply; invalid cursor scoped resync, pending/UNKNOWN queue saqlanadi. Next-contact revoke lock/quarantine; disconnected remote wipe kafolati yo‘q. TTL/retention/legal hold `OPEN`; taxminiy kunlar yozilmaydi.
- **Gate:** offline mutation feature disabled Stage 1da. Stock/finance/payment intents default online-authoritative; allowed offline intent catalog keyingi owner policy bilan ochiladi. Duplicate/ACK loss/scope switch/stolen token/out-of-order local dependency tests `NOT_TESTED`.

## 9. IC-07 — Party, CRM, legal debtor, external integration

- **Party/CRM:** 03 canonical tenant Party identity+alias/merge; CRM Customer/Outlet/agent assignmentni alohida yozadi. Finance 02 legal debtor/receivable owner va invoice-time immutable legal snapshotni yozadi. Outlet/visit/free-text remitter invoice qarzdori deb qabul qilinmaydi.
- **API payload:** canonical Party ID/version, customer/outlet reference, debtor legal evidence/role, issuer legal entity, effective mapping and merge lineage. Third-party payer/remitter settlement/evidence/authorized allocation alohida fields; SaaS funder va ERP AR debtor alohida.
- **External boundary:** provider adapter→normalization→namespace mapping→staging→owner import command→reconciliation. External server/account, namespace CS_id/SD_id/code_1C, entity type/revision/hash, canonical ref va adapter contract version. Same external key different meaning conflict, no ID collision fallback.
- **Occurrence/outbox:** owner-accepted import source occurrence; source mapping replay uni saqlaydi. Mapping owner aggregate yozmaydi; Finance/Billing/Inventory public commandsgina effect yaratadi. Import history origin BILL-014 eligibility’da explicit excluded candidate.
- **Correction:** Party merge future resolution aliasesga o‘tadi, old fiscal debtor snapshot/journal o‘zgarmaydi. Split/identity dispute owner-authorized mapping correction, original links; unmatched funds suspense/unapplied. Cross-company settlement/novation approval `OPEN`, cross-tenant allocation deny.
- **Recovery:** provider failure checkpoint/cursor va staging saqlanadi; authorized bounded retry; source checkpoint/count/quantity/amount reconciliation. Cutover writes paused, rollback point va old/new source writer aniq; irreversible financial data restore orqali silent undo qilinmaydi.
- **Evidence:** SalesDoc `DOCUMENTED`, live auth/schema/rate-limit/tenant behavior `NOT_TESTED`, `LIVE_VERIFIED` yo‘q. API method katalogi bu paketda qayta research qilinmagan. Production outbound sync disabled; credential/cutover/jurisdiction ni userdan alohida ruxsatsiz ishga tushirib bo‘lmaydi.

## 10. Qo‘shimcha muhim Billing chegaralari

| Contract | Writer/readers va v1 fields | Replay/correction/recovery va gate |
|---|---|---|
| IC-08 Tenant-shared resource responsibility | 02 Billing assignment; 01 org/role guard; 01/03 meters. Tenant/resource scope, assignment version, proposed/accepted by roles, effective range, responsible Company | One accepted responsible Company; prospective change; exact metering segment source mapping. Missing mapping `PENDING_ATTRIBUTION`; old debt/Wallet never transfers. Two-role/same-person/cutoff policy OPEN |
| IC-09 Verified SaaS top-up | 02 payment adapter/Billing; provider account/payment ID, verified settled revision/evidence, destination Account/Wallet, authorized funder, amount/currency, provider occurrence | Dedup provider business payment ID across differing webhook event IDs; pending/success screen credit bermaydi. Settlement→credit once; refund/chargeback new linked correction; provider proof and policy OPEN |

IC-08 va IC-09 ham §1 envelope/local ACID/inbox/idempotency/version/recovery talablarini bajaradi. Responsible assignment revision yoki provider webhook event ID o‘zi yangi debit/credit uchun yetarli economic identity emas.

## 11. Contract adoption check

Har IC uchun producer+consumer review commentlari: payload fieldlari, ID issuer, semantic dedup key, scope provenance, authorization-at-acceptance vs durable-service effect, state/effect oracle, version compatibility, correction owner, retry/DLQ/restore rehearsal va stage enable gate. Aniq imzo/review URLlari [PR qoidalari](03_PR_AND_INTEGRATION_GATES.md) bo‘yicha GitHub’da qayd etiladi. Barcha acceptance runtime holati hozir `NOT_TESTED`; owner technical adoption `OPEN`.
