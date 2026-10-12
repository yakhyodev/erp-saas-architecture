# CODEX 05 — ownership va application repository spetsifikatsiyasi

Status: `PROPOSED`. Source baseline va status lug‘ati: [Stage 0 README](README.md). Mavjud writer tanlovi S02/S04’dan `DESIGN_SELECTED_UNDER_DELEGATION`; yangi besh-chat taqsimoti technical adoption kutadi.

## 1. Besh chatning ownership matrix’i

Bu yangi development chat raqamlari. Ular tarixiy 00…14 architecture owner raqamlarini almashtiruvchi tasdiq emas. Masalan, yangi CODEX 01 Billing emas; tarixiy 01 Billing yangi CODEX 02ga xaritalanadi.

| Chat | Authoritative writer / mas’uliyat | Readers / tashqi dependency | Expected outputs | Directory ownership |
|---|---|---|---|---|
| **01 Platform Core & Security** | Identity/session/device; Tenant/Company/Branch/Membership; Role/Policy; platform DB/pool, audit, outbox relay, infra plumbing | 02 legal evidence/Billing constraints; 03 resource attributes; 04 auth/sync UX; 05 common contract spec | Versioned auth/org ports, current authorization guard, tenant/branch negative-suite design, domain-local transaction primitives, observability va DR runbook | `cmd/`, `internal/bootstrap/`, `internal/platform/`, modules `identity,tenancy,permissions,audit,notifications`; runtime/CI/ops; common technical test harness |
| **02 Billing & Finance** | Alohida BillingAccount/Wallet/Seat/Usage/charge ledger; alohida ERP fiscal document/GL/AR/AP/settlement/period; legal verification evidence | 01 Company/Membership/Policy; 03 domain facts, Party, valuation; 04 money state UI | Billing va Finance ports/events, priced-policy gates, liability snapshots, occurrence→ledger reconciliation, finance golden/hostile acceptance contracts | Modules `billing,finance`; ularga tegishli contracts/migrations/tests/docs; payment/fiscal adapterlari |
| **03 Commerce & Operations** | Shared Party, Catalog, Pricing, CRM relationships/assignments, Sales, Inventory, Purchasing; non-payment external adapter mapping/staging | 01 authoritative scopes; 02 legal debtor/recognition/Billing eligibility; 04 task/offline clients | Har domenning state machine’i, quote snapshot, dispatch/receipt/return contracts, reservation/stock invariants, adapter evidence ledger | Modules `party,catalog,pricing,crm,sales,inventory,purchasing,integrations`; ularga tegishli contracts/migrations/tests/docs |
| **04 Web, Mobile & UX** | Web/mobile presentation, local encrypted scoped queue/cache, UX contracts, translation bundles; client generated artifacts | 01 auth/current context; 02 money/status projections; 03 commercial/stock contracts; 05 envelope/version/error rules | Task-first UX-T01…20 acceptance, E/L/X/O states, scoped context switch, uz/en/ru, a11y, UNKNOWN recovery | `apps/web/`, `apps/mobile/`, `packages/design-system/`, `packages/localization/`, `packages/client-contracts/`, UX/e2e docs va client tests |
| **05 Architecture, Integration & QA** | Architecture/spec writer; umumiy envelope/error/versioning spec; ownership/compatibility/gate registry; integratsiya review koordinatsiyasi | Barcha domain producer/consumer sign-offlari; original 00 Master governance va 14 audit dalillari | Ushbu paket, contract change qarori, dependency graph, integration gate/evidence index, blocker disposition | Architecture repository `docs/stage0/`; kelajak application `docs/architecture/`, `docs/contracts/common/`, `docs/qa/`, `docs/ownership/`; root governance o‘zgarishi faqat alohida docs PR va original owner review bilan |

CODEX 05 biznes entity, ledger, application runtime, boshqa chatning migration yoki source fayliga writer emas. QA review title’i boshqa chat nomidan tasdiq yoki test pass yozishga huquq bermaydi.

Tarixiy yo‘nalishlar mapping’i: 02 Multi-Tenant / 03 Database / 04 Backend / 10 Security / 11 Infrastructure → yangi 01; 01 Billing / 06 Finance → yangi 02; 05 Sales-Inventory / 07 CRM / 12 Integrations → yangi 03; 08 UX / 09 Mobile / 13 Localization → yangi 04; 00 Master / 14 Review koordinatsiya dalillari → yangi 05. Domenlarga tegishli Backend kontrakt va testlar o‘sha domen writerida qoladi.

## 2. Entity writer va reader matrix’i

| Canonical entity / fact | Yagona writer (chat / context) | Reader / port / event | O‘zgarmas chegara |
|---|---|---|---|
| Global Principal, credentials, MFA, session, device security | 01 / Identity | Tenancy references; all server guards | Global identity tenant membership emas |
| Tenant, Company, Branch, Company→LegalEntity tashkiliy link | 01 / Tenancy | 02 verify legal evidence; all scoped ports | Tenant/Company/LegalEntity ID tenglashtirilmaydi; legal link effective-dated |
| Company legal verification attestation | 02 / Finance legal validation | Tenancy/Billing read verified reference | Attestation yangi Company yoki Party master yaratmaydi |
| TenantMembership status/version/company scope | 01 / Tenancy | Permissions current read; Billing effective facts | Permissions va Billing ikkinchi membership writer emas |
| RoleAssignment, PolicyDefinition, grant version | 01 / Permissions | Domain guards and authorized queries | Client role flag authority emas |
| PermissionEvaluation | 01 / Permissions evaluator | Domain executes guarded use-case | Mustaqil writable permission truth saqlanmaydi |
| Canonical tenant Party / identity aliases / merge lineage | 03 / Party | CRM, Sales, Purchasing, Finance by public port | Company organizational LegalEntity linkidan alohida rol; bitta Party master |
| Customer, Outlet, territory/agent assignment, visit | 03 / CRM | Sales/mobile scoped reads | Outlet yoki relationship avtomatik legal debtor emas; GPS privacy gate |
| Product, SKU, UoM | 03 / Catalog | Pricing/Sales/Inventory/Purchasing/CRM | Qoldiq va SaaS tariff Catalogga tegishli emas |
| PriceList, commercial quote | 03 / Pricing | Sales immutable price snapshot | SaaS pricing = Billing; quote qoldiq kafolati emas |
| SalesOrder, commercial dispatch intent, fulfillment projection | 03 / Sales | Inventory commands/facts, Finance invoice refs, Billing facts | Confirm physical on-hand’ni kamaytirmaydi |
| Reservation, StockMovement, StockBalance, valuation source | 03 / Inventory | Sales, Purchasing, Finance typed facts | Har posting leg bir marta; invoice ikkinchi movement emas |
| PurchaseOrder, commercial GoodsReceipt | 03 / Purchasing | Inventory accepted receipt, Finance | Purchasing StockBalance yozmaydi |
| Fiscal invoice/credit note, legal debtor AR/AP, GL journal, settlement, close | 02 / Finance | Sales/Purchasing/report projections | Fiscal invoice writer Finance candidate; Sales faqat commercial refs. Recognition/legal owner sign-off OPEN |
| SaaS BillingAccount, Wallet, Seat qualification, tariff/rating, SaaS statement | 02 / Billing | Scoped web/mobile summaries and Finance org refs | ERP GL/AR/AP’dan alohida ledger |
| Tenant-shared responsibility assignment | 02 / Billing | 01 org/permission validates; meter reads effective assignment | Old debtor yoki balance ko‘chirilmaydi; bitta accepted responsible Company |
| Actual resource meter fact | Resursning mavjud owner’i: 01 platform resource / 03 domain resource | 02 Billing ingest | Source narx va Wallet tanlamaydi; meter interval/unit/provenance kerak |
| Verified provider payment receipt/mapping | 02 / Billing payment adapter | Billing funding command | Provider pending webhook Wallet credit emas |
| Non-payment external IDs/provider mapping/staging | 03 / Integrations | Domain-owned normalized import commands | CS_id/SD_id/code_1C canonical ID emas; GL/Walletga direct write yo‘q |
| Encrypted offline queue/cache/cursor | 04 / Mobile | 01 sync transport; 03 domain acceptance; 02 effects | Lokal queued server accepted/posted emas |
| Translation bundles/search aliases | 04 / Localization | Web/mobile/domain read models | Canonical SKU, INN, IDs va money o‘zgarmaydi |
| Search/report domain projection | O‘sha canonical source domeni | 04 query client; 01 authorization/index plumbing | Har domain index write’i o‘z projection portidan; common search dispatcher 01; count/facet/export ham scoped |
| Consumer inbox/effect/outbox | Tegishli consumer domeni | 01 relay/harness; 05 reconciliation review | Common relay business state yozmaydi |

Bir chat ichida ham context chegarasi bor: 03 Sales Inventory private jadvalini; 02 Billing Finance journalini bevosita yozmaydi. Bir commitda faqat owner aggregate/inbox/effect/outbox atomikligi va tegishli local invariantlar kafolatlanadi.

## 3. Application repository papka tavsiyasi

Application repository nomi/manzili **OPEN**. Quyidagi daraxt S09 Backend §2.1ga mos tavsiya; bu papkalar hozir yaratilmaydi:

```text
<application-repository>/
  cmd/{api,worker}/                              # 01
  internal/
    bootstrap/                                  # 01 composition, domain public ports only
    platform/{config,db,auth,tenancy,observability,queue,clock,ids}/ # 01 adapters
    modules/
      {identity,tenancy,permissions,audit,notifications}/ # 01
      {billing,finance}/                         # 02
      {party,catalog,pricing,crm,sales,inventory,purchasing,integrations}/ # 03
        {domain,application,ports,adapters,transport}/
    contracts/
      api/<producer-domain>/v1/                 # tegishli 01/02/03 writer
      events/<producer-domain>/v1/              # tegishli 01/02/03 writer
      common/v1/                                # 01 executable tooling; 05 spec review
  apps/{web,mobile}/                            # 04
  packages/{design-system,localization,client-contracts}/ # 04
  migrations/
    platform/                                   # 01
    domains/{billing,finance}/                   # 02
    domains/<commerce-domain>/                  # 03
    manifest/                                   # 01 ordering; 05 cross-owner review
  tests/
    harness/                                    # 01
    domain/<domain>/                            # domain writer
    contract/<producer-domain>/                 # producer writer + consumer fixtures
    consumer/<consumer-domain>/                 # consumer writer
    {security,performance,recovery}/platform/    # 01 + domain-owned cases
    e2e/{web,mobile}/                           # 04 + domain oracle review
  docs/
    domains/<domain>/                           # domain writer
    contracts/<producer-domain>/                # producer writer
    contracts/common/                           # 05
    {architecture,qa,ownership}/                # 05
    ux/                                         # 04
  ops/                                          # 01, faqat kelajak authorizationdan keyin
  .github/                                      # 01 implementation; ownership/gate spec 05 review
```

Platform `auth/tenancy` papkalari transport/context adapterlari; `modules/identity/tenancy` authoritative business writer. Ikki credential yoki membership store paydo bo‘lmasin. `packages/client-contracts` server contractdan generated client artifact; backend schema source’ini ko‘chirmaydi. Common library’da business price/tax/recognition logic bo‘lmaydi.

Payment/fiscal adapters 02 context adaptersida; SalesDoc/bankdan tashqari commerce connector 03 Integrationsda. Agar provider bir nechta domenni qamrasa, secret/HTTP primitives 01, normalizer provider ownerida, har biznes effekt esa tegishli owner portida. Bank settlement adapter va allocation authority 02da qoladi.

## 4. Directory o‘zgartirish tartibi

- Producer contract uchun yagona writer producer chat; consumer o‘z fixture/client/projectionini yozadi. 05 common protocol specni koordinatsiya qiladi.
- Domain migrationni domain writer tayyorlaydi; 01 tenant/FK/RLS/ordering safety review qiladi. 01 DBA vazifasi domain entity’ga yashirin writer authority bermaydi.
- Root dependency lock, composition root, CI va migration manifest 01 exclusive writer; boshqa chat change request va aniq port/schema diff beradi.
- Ownership xaritasi faqat 05 docs PR bilan, ta’sirlangan eski va yangi ownerlarning recorded review’idan keyin o‘zgaradi. Mavjud approved biznes chegarasini o‘zgartirish uchun alohida user provenance kerak.
- CODEOWNERS keyinchalik shu jadvaldan yaratiladi. GitHub username/team mapping **OPEN**; taxminiy username yozilmaydi. Bitta inson besh chatni ishlatsa ham chatlar review navbati va dalilini GitHub’da qayd etadi; bu mustaqil ikki inson review’i deb ko‘rsatilmaydi.
