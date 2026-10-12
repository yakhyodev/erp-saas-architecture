# BACKEND_ARCHITECTURE.md

- **Loyiha:** Sales doc copy — Multi-Tenant ERP SaaS
- **Versiya:** 1.0 (2026-10-11)
- **Holat:** PROPOSED (texnik qarorlar hali tasdiqlanmagan)
- **Mas'uliyat:** Backend Architecture
- **Chegara:** Arxitektura va implementatsiya uchun talablar; kod va SQL yo'q.

## 1. Maqsad va qat'iy talablar

Maqsad — 1000+ kompaniyali, ko'p filialli, tezkor va xavfsiz ERP SaaS backendini loyihalash; o'lchovsiz erta optimallashtirishdan qochish. Biznes bo'yicha **APPROVED**: Hybrid Prepaid SaaS; arzon Seat + Business Usage (hujjat uchun bazaviy narx va hajm chegarasidan keyin kredit) + Resource Usage; 1 kredit = 10 so'm; progressiv chegirma va enterprise individual shartnoma; balans tugaganda haqiqiy Seat va Usage xarajatlari bir kalendar oy minusga yoziladi, taxminiy kredit limiti hisoblanmaydi; kompaniya/filiallar yagona SaaS; uz/en/ru va yashirin yangi o'zbek alifbosi varianti. Bu hujjat ularni o'zgartirmaydi.

**PROPOSED:** Go, PostgreSQL, Next.js, Flutter, modular monolith, shared-DB tenant isolation, REST, transactional outbox, workers, OpenTelemetry. Bularning hech biri texnik APPROVED sifatida talqin qilinmaydi. Yakuniy tenant izolyatsiyasi, payment semantics va infrastruktura modullar bilan muvofiqlashtiriladi.

## 2. Tizimning yuqori darajali qatlamlari

1. **Clients:** web/mobile/public integration; backendni yagona business-rule authority deb biladi.
2. **Ingress:** TLS termination (infrastructure), routing, request size limit, rate limiting, request ID; proxy headerlar faqat ishonchli proksidan.
3. **HTTP transport:** schema validation, versioning, localization negotiation, JSON serialization; domen mantiqi yo'q.
4. **Identity & tenant context:** session/token validatsiyasi, tenant/company/branch membership, scoped authorization.
5. **Application:** use-case orchestration, transaction boundary, idempotency, repository interfaces, event creation.
6. **Domain:** invariantlar, aggregate va biznes qoidalar; framework/transportga bog'lanmaydi.
7. **Infrastructure adapters:** PostgreSQL repositories, queue publisher/consumer, object store, external API, telemetry.
8. **Worker process:** shu kod bazadagi alohida process/deployment role; schedule, outbox relay, tasks, retries.

**Qatlam qoidasi:** dependency transport → application → domain; infrastructure domain/application interfeyslarini implement qiladi. Modulning ichki jadvali va repositorysiga boshqa modul to'g'ridan-to'g'ri yozmaydi. Shared-kernel faqat haqiqatan umumiy primitivlardan iborat, universal `common` paket paydo qilinmaydi.

### 2.1 Konseptual kod tuzilmasi (implementatsiya kodi emas)

- `cmd/api` — HTTP startup va graceful shutdown.
- `cmd/worker` — background worker startup.
- `internal/platform/{config,db,auth,tenancy,observability,queue,clock,ids}` — platform adapterlari.
- `internal/modules/<module>/{domain,application,ports,adapters,transport}` — har bir modulning qatlamlari.
- `internal/contracts/{api,events}` — versionlangan tashqi kontraktlar, generator/output manbalari.
- `internal/bootstrap` — composition root va modullarni ulash.
- `docs/architecture/adr` — ADRlar; `docs/contracts` — API/event contract ta'riflari.
- `tests/{integration,contract,e2e,performance,security}` — test qatlamlari.

Bu **mantiqiy papka spetsifikatsiyasi**, hozir fayl tuzilmasi yaratish buyrug'i emas.

## 3. Domain boundaries va ownership

| Modul | O'zi boshqaradigan konsept | Boshqa modul bilan aloqa |
|---|---|---|
| Identity & Access | User identity, credential/session, membership/role association kontrakti | Tenant'ning joriy a'zolik kontekstini tekshiradi; security modul bilan yakuniy ajratiladi |
| Tenant & Organization | Tenant, company, branch, lifecycle, tenant routing | Barcha command va query uchun verified tenant scope beradi |
| Catalog | Product, SKU, price-list referenslari (egalik tasdiqlanadi) | Sales/Inventory faqat public port orqali |
| Sales | Order, sales document, document lifecycle | Inventory reservation/posting va Finance posting use-caselari |
| Inventory | Warehouse, stock movement, reservation, stock ledger | Sales va Purchasing xabarlari yoki sinxron portlar |
| Purchasing | Purchase order/receipts (scope taklif) | Inventory va Finance |
| Finance & Accounting | Accounting documents, journal entries, AR/AP, reconciliation | Biznes eventlarini auditli postingga aylantiradi |
| Billing & Metering | Subscription, seat snapshot, usage event, wallet, credit ledger, discount, invoice | Har bir billable action uchun authoritative usage; Finance tenant moliyasidan ajratilgan platforma billing domeni |
| CRM & Distribution | Contacts, pipeline, distribution orders | Sales'ga versionlangan biznes kontrakt orqali |
| Integrations | External connector, webhook delivery, mapping | Public API va domain events |
| Notifications | Templates, delivery attempts | Events asosida ishlaydi; source of truth emas |
| Audit | Security/business audit trail | Muhim actionlar uchun immutable audit record |

**Muhim farq:** platformaning SaaS billing/wallet hisobi bilan tenantning kundalik finance/accounting hisobi alohida bounded context. Billing chegirmasi, bir kalendar oy davomida minus yozish, kredit qiymati Billing modulining invariantlari; boshqa modullar bu qoidalarni nusxalamaydi.

**Cross-module qoida:** query uchun versionlangan read-model yoki public query port; command uchun application port; vaqtga bog'liq bo'lmagan reaktsiya uchun domain/integration event. Uzoq ketma-ketliklar uchun explicit orchestration/saga faqat zarur bo'lsa.

## 4. Transactions va consistency

- **PROPOSED:** bitta DBdagi kuchli bog'langan use-case uchun qisqa ACID transaction; persistent biznes state + idempotency result + outbox record atomar yoziladi.
- Transactionni application layer ochadi/yopadi; nested transaction yashirin yaratilmaydi. Bir biznes use-case bir transaction ownershipiga ega.
- Tenant/branch scope barcha read/write'larda qo'llanadi; cross-tenant transaction taqiqlanadi (platform-level operatsiyalar mustasno va alohida audit qilinadi).
- Inventory negative-stock, reservation va quantity invariantlari race conditionga qarshi concurrency control bilan himoyalanadi; optimal locking strategiyasi Database/Sales modullarida belgilanadi.
- Financial posting: balansdan foydalanish, credit ledger, invoice va accounting postinglarida double entry/invariantlar hamda deterministic reversal; operatsiyani o'chirish bilan tuzatish emas.
- **Idempotency:** tegishli mutating endpointlarda (`payments`, `sales posting`, `billing debit`, `stock posting`, external webhook processing) tenant + operation scope bo'yicha idempotency key; payload fingerprint tekshirilsin; bir xil key + boshqa payload → conflict; saqlanadigan natija va retention siyosati alohida hal qilinsin.
- Tashqi provider chaqiruvi DB transaction ichida bloklovchi I/O sifatida bajarilmaydi; intent/outbox, provider reference, reconciliation bilan boshqariladi.
- Taqsimlangan operatsiyalarda exactly-once delivery deb va'da berilmaydi: at-least-once processing + idempotent consumer + retry + compensating action.
- Katta hisobotlar transactionni uzoq band qilmasin; read models/projections taklif qilinadi.

## 5. API arxitekturasi

**PROPOSED:** web/mobile/external uchun versionlangan HTTPS JSON REST API; internal portlar esa in-process. gRPC faqat tarmoq chegarasi yoki yuqori throughput talabi o'lchansa qo'shiladi.

- Contract-first OpenAPI (versiya, schema, response, error, auth, pagination, limitlar va deprecation policy).
- Global URI versiyalash misoli: `/api/v1/...` (shakli PROPOSED); major breaking change yangi major versiyada.
- Resources: tenant/company/branch context; foydalanuvchining ruxsati serverda qayta tekshiriladi. Client yuborgan tenant ID o'zi bilan huquq bermaydi.
- Command va querylarni HTTP method semantikasi bilan ajratish; side-effectli commandlar explicit endpoint va idempotency bilan.
- Cursor pagination katta ro'yxatlar uchun afzal, barqaror sort; query filter whitelist; response field visibility permissionga mos.
- Offset pagination faqat kichik/administrativ ro'yxatda.
- Standard: request ID, trace context, locale, validation errors, UTC timestamps (locale UI qatlamida), decimal/money aniq birliklari (float emas); API schema narx/kredit birliklarini belgilaydi.
- File upload signed upload flow va scan/authorization bilan; ishchi jarayonlar fayl kontentini tekshiradi.
- Public API: scoped credentials, quotas, webhook signing, replay defense, retry/redelivery va audit; rate limit plan va tenant siyosati bilan muvofiqlashadi.
- API schema va domain event schema alohida: HTTP contract va event contract bir-biriga ko'r-ko'rona tenglashtirilmaydi.

## 6. Authentication/authorization flow

1. Login boshlanadi (credential/SSO variantlari Security Architecture bilan tasdiqlanadi); TLS va brute-force/rate controls.
2. Identity provider/session service foydalanuvchini tekshiradi; MFA uchun yuqori xavfli amallar bo'yicha siyosat belgilanadi.
3. Backend short-lived access contextni va session/refresh lifecycle'ni nazorat qiladi; token storing, rotation, revocation, CSRF/CORS siyosati client turiga qarab Security Architecture'da belgilanadi.
4. Request kelganda imzo/session, expiry, issuer/audience va revoke/permission policy tekshiriladi.
5. Tenant/company/branch tanlovi verified membership bilan moslashtiriladi; active tenant backend tomonidan resolve qilinadi; clientning tenant headeriga ko'r-ko'rona ishonilmaydi.
6. Har bir use-case permission (role + resource scope + branch scope + special restrictions) bilan tekshiriladi; faqat UI elementini yashirish yetarli emas.
7. DB transaction contextda tenant isolation qo'llanadi; RLS tanlansa SET LOCAL kabi transaction-scoped approach Database/Multi-Tenant modullari bilan kelishiladi, connection pool orqali leak bo'lmasligi test qilinadi.
8. Impersonation va platform-admin bypass faqat explicit elevatsiya, qisqa amal muddati, sabab va audit bilan; default deny.

## 7. Background workers va queue

**PROPOSED initial:** queue uchun PostgreSQL-backed durable job table yoki tayyor DB-backed queue adapteri; ulkan throughput talabida Redis Streams/RabbitMQ/Kafka alohida baholanadi. Tanlov texnologik tasdiq emas. API va worker bir kod bazadan, alohida scale qilinadi.

**Worker turlari:** billing aggregation/seat reconciliation; subscription cycle boundary; invoice generation; webhook delivery; notification; data import/export; materialized projection; audit export; cleanup; recurring schedule; external sync.

**Job contract:** job_id, tenant_id, job_type, contract_version, payload/reference, deduplication_key, priority, available_at, attempt_count, timeout, trace_id, correlation_id; sensitive payload o'rniga reference.

**Delivery:** durable enqueue, lease/visibility timeout, heartbeat/lease expiry requeue, bounded retry (exponential backoff+jitter), permanent failure → dead-letter, operator replay; max attempts va retention siyosati keyin tasdiqlanadi. Consumerlar idempotent va tenant-aware. Ordering faqat kerakli aggregate/partition ichida kafolatlashga uriniladi; global ordering va exactly-once va'da qilinmaydi.

**Scheduler:** bir nechta instance bo'lsa leader/lease yoki DB lock; recurring joblar deterministic interval key bilan dedup; vaqt zonasi va calendar-month boundaries billingda explicit. Month-end billingni tashqi worker kechikishi natijani ikki marta yozmasligi kerak.

## 8. Event architecture

- **Domain events** modul ichida, muvaffaqiyatli biznes tranzaksiyani ifodalaydi; **integration events** boshqa modul/external consumer uchun barqaror kontrakt.
- Transactional outbox biznes holati bilan bir transaction ichida persisted; relay commitdan keyin queue'ga yuboradi; at-least-once delivery.
- Event metadata: event_id, event_type, schema_version, tenant_id, company/branch context (tegishli bo'lsa), aggregate_id, occurred_at, causation_id, correlation_id, trace_id.
- Consumer inbox/dedup store; duplicate delivery oddiy holat sifatida qabul qilinadi. Event ordering va replay siyosati aggregate kesimida belgilanadi.
- Event name biznes faktini bildiradi (`SalePosted`, `StockMovementRecorded`, `UsageRecorded` kabi konseptual misollar). Buyruq (`DoBilling`) va hodisa (`UsageRecorded`) farqlanadi.
- Bir modulning event payloadiga boshqa modul ichki jadvalini qo'shib bog'lanish yaratmaydi. Schema backward compatibility va deprecation policy yuritiladi.
- Billing: chargeable action identifikatori bilan source module event → usage ingestion → billing calculation/ledger; hujjat commit bo'lishi va usage billing o'rtasida yo'qolmaslik kafolatlanadi; SaaS walletni har bir event consumer mustaqil hisoblamaydi.

## 9. Error handling va resilience

**Error taxonomy:** VALIDATION, UNAUTHENTICATED, PERMISSION_DENIED, TENANT_SCOPE_VIOLATION, NOT_FOUND, CONFLICT, IDEMPOTENCY_CONFLICT, BUSINESS_RULE_VIOLATION, RATE_LIMITED, DEPENDENCY_UNAVAILABLE, TIMEOUT, INTERNAL_ERROR. Public response: stable code, localized-safe message key, request_id, field errors, retryable flag faqat kerakli holatda. Internal stack traces va tenant sensitive data mijozga chiqmaydi.

- Validation 4xx; unexpected 5xx, log+trace; response/status mapping contract bilan boshqariladi.
- Retry faqat vaqtinchalik xatolar va idempotent operatsiyalar uchun. Deadline, bounded concurrency, backpressure, circuit breaker (zarurat bo'lsa) va graceful shutdown.
- Dependency failureda kritik write default fail-closed; noaniq natijada reconciliation workflow.
- Rate limit: identity/IP/tenant/key bo'yicha qatlamli; tenant noisy-neighbor himoyasi.
- User-facing xabarlar uz/en/ru; yashirin yangi o'zbek alifbosi availability flag bilan; error code lokalizatsiyadan mustaqil.

## 10. Logging, monitoring, audit, observability

- Structured JSON logs: UTC timestamp, level, service, module, env, request_id, trace_id, span_id, tenant pseudonymous identifier, operation, error_code, latency; PII/secrets/token/payment info maskalanadi.
- Logs va audit alohida: log operatsion diagnostika; audit kim/nima/qachon/qaysi tenant/sabab/oldingi-keyingi biznes holatning zarur minimal metadata sini o'zgartirishdan himoyalangan shaklda saqlaydi. Retention huquqiy talablarga qarab aniqlanadi.
- OTel uslubidagi tracing+metrics; REST, DB, queue, worker, external call spanlari; high-cardinality tenant ID metric label bo'lmaydi.
- Golden signals: latency (p50/p95/p99), throughput, error ratio, saturation. Qo'shimcha: DB pool wait, slow queries, lock waits, queue lag, retry/DLQ, outbox backlog, billing unprocessed usage, wallet-ledger reconciliation failures, tenant isolation incidents.
- Health: liveness, readiness, dependency health alohida; dashboards va severity-based alerting. SLO va thresholdlar production o'lchovlardan keyin APPROVED qilinadi.
- Data retention, log access RBAC, encryption, data residency / compliance Security va Infrastructure tomonidan kelishiladi.

## 11. Scale strategiyasi va extraction criteria

**Bosqich 1:** modular monolith; stateless API replicas + worker replicas; DB connection pool; indexes va query profiling; optional cache faqat o'lchangan hot-path uchun. Har bir endpoint ma'lumotlar chegarasini qat'iy saqlaydi.

**Bosqich 2:** read replica, cache, materialized reads, queue isolation, autoscaling, tenant-aware workloads; DB partitioning Database Architecture bilan boshqariladi.

**Bosqich 3:** enterprise isolation/sharding variantlari uchun tenant routing abstraction; schema/database ajratish migration va dual-write xavflarini oldindan baholash.

**Microservice extraction mezonlari (birgalikda baholanadi):** mustaqil resource profile yoki maxsus scaling zarur; aniq ownership va barqaror public contract; alohida release/deployment ehtiyoji; failure isolation qiymati; cross-service consistency xarajati qabul qilinadi; infra/observability/security tayyor; migration/rollback plan va testlar mavjud. Mikroservisga ajratish kompaniya soni bittagina thresholdga yetgani uchun avtomatik amalga oshmaydi.

**Ajratish ketma-ketligi taklifi:** avval side-effecti kamroq notifications/integrations yoki heavy reporting; Billing/Inventory/Accounting kabi kuchli transaction bog'liqligi bo'lgan modullar keyin, data ownership/reconciliation aniq bo'lganda. Strangler pattern, API/event compatibility, data migration, cutover, rollback zarur.

## 12. Risklar va ochiq masalalar

| ID | Masala | Holat | Bog'liq modul |
|---|---|---|---|
| OQ-BE-01 | Shared DB + RLS yoki boshqa tenant isolation yakuniy modeli | OPEN QUESTION | Multi-Tenant, Database, Security |
| OQ-BE-02 | Identity/session modeli, MFA, SSO, refresh token siyosati | OPEN QUESTION | Security, UI/Mobile |
| OQ-BE-03 | Sale posting + stock + finance cross-module atomicity chegarasi | OPEN QUESTION | Sales, Inventory, Finance, DB |
| OQ-BE-04 | Billing chargeable event, reversal, backfill va negative-month closure semantics | OPEN QUESTION | Billing, Database |
| OQ-BE-05 | Queue texnologiyasi, delivery throughput, SLA, retry/retention qiymatlari | OPEN QUESTION | Infrastructure |
| OQ-BE-06 | API standard, pagination limitlari, rate limit va public API scope | OPEN QUESTION | Integrations, UX, Security |
| OQ-BE-07 | Performance SLO, peak load profile, DB sizing va budget | OPEN QUESTION | Infrastructure, Database |
| OQ-BE-08 | Owner of Product Catalog, Purchasing va CRM boundary | OPEN QUESTION | Master, Sales, CRM |

**Potensial konfliktlar:**
1. Agar tenant isolation `database-per-tenant` bo'lsa, umumiy transaction assumptions va connection routing qayta ko'riladi.
2. Agar Sales/Finance ikkita service'ga ajratilsa, bir DB transactionda posting qilish mavjud yondashuvi o'zgartirilishi kerak; explicit saga/reconciliation zarur.
3. Billingda balans tugashi *faqat* bir kalendar oy haqiqiy xarajat minus yozilishini anglatadi; backend taxminiy limit yoki qo'shimcha avtomatik blok siyosatini ixtiro qilmaydi. Oydan keyingi holat alohida biznes savol.
4. Seat billingning snapshot yoki pro-rata hisoblash usuli hali tasdiqlanmagan; worker bu xatti-harakatni mustaqil belgilamaydi.

## 13. ADRlar (hammasi PROPOSED)

### ADR-BE-001 — Go backend candidate
- **Muammo:** katta ERP backendida performance, dev tezligi va operational cost balansi.
- **Variantlar:** Go; Java/Spring; .NET; Node.js/TypeScript.
- **Tanlov (PROPOSED):** Go asosiy nomzod sifatida baholanadi, hali APPROVED emas.
- **Sabab:** kompilyatsiyalangan binary, concurrency va ixcham deploy modeli; ops soddaligi potentsiali. Real load test shart.
- **Xavf:** kuchli domain conventions va libraries tanlovi uchun intizom, hiring/skill gap.
- **Ta'sir:** Infrastructure, Database, Security, frontend contractlar.

### ADR-BE-002 — Modular monolith boundaries
- **Muammo:** monolit kompleksligi va microservice operatsion xarajati.
- **Variantlar:** qatlamli monolit; modular monolith; microservices first.
- **Tanlov (PROPOSED):** bounded contextli modular monolith.
- **Sabab:** transactionlar sodda va deploy operatsiyasi kam, kelajakda extraction mumkin.
- **Xavf:** boundary erosion, tightly coupled queries.
- **Ta'sir:** barcha biznes modullar, Database, Security.

### ADR-BE-003 — Transaction & idempotency
- **Muammo:** duplicate charge, stock race, half-commit.
- **Variantlar:** best-effort; ACID + outbox + idempotent consumers; distributed 2PC.
- **Tanlov (PROPOSED):** lokal ACID, transactional outbox, inbox/dedup va reversal.
- **Sabab:** auditability va real delivery failurelarga chidamlilik.
- **Xavf:** eventual consistency va outbox backlog; strict transaction boundary.
- **Ta'sir:** Billing, Sales, Inventory, Finance, Database.

### ADR-BE-004 — API-first versioned REST
- **Muammo:** web/mobile/integration evolyutsiyasi.
- **Variantlar:** REST/OpenAPI; GraphQL; gRPC public.
- **Tanlov (PROPOSED):** REST/OpenAPI; gRPC internal extraction payti qayta baholanadi.
- **Sabab:** contract clarity, client/tooling compatibility.
- **Xavf:** chatti endpoints, version compatibility burden.
- **Ta'sir:** Web, Mobile, Integrations, Security.

### ADR-BE-005 — Worker & queue architecture
- **Muammo:** reliable deferred processing, burst control.
- **Variantlar:** synchronous-only; DB-backed queue; Redis Streams; RabbitMQ; Kafka.
- **Tanlov (PROPOSED):** initial DB-backed durable jobs + independent workers; load bo'yicha upgrade.
- **Sabab:** boshida kam infratuzilma, DB bilan transaction yaqinligi.
- **Xavf:** job polling DB bosimi, throughput chegarasi.
- **Ta'sir:** Billing, Notifications, Integrations, Infrastructure.

### ADR-BE-006 — Event contracts and outbox
- **Muammo:** event publish va DB commit orasida data loss.
- **Variantlar:** direct publish; polling; outbox + relay.
- **Tanlov (PROPOSED):** transactional outbox + versioned events + idempotent consumer.
- **Sabab:** publish failurelarga chidamlilik.
- **Xavf:** duplicate, ordering, schema governance va replay murakkabligi.
- **Ta'sir:** barcha event-producing modullar.

### ADR-BE-007 — Authentication and tenant authorization
- **Muammo:** cross-tenant data exposure, privilege escalation.
- **Variantlar:** client-trusted IDs; app-only checks; verified authn/authz + DB defense-in-depth.
- **Tanlov (PROPOSED):** server-resolved tenant/membership; use-case authorization; DB isolation defense-in-depth.
- **Sabab:** bir necha qatlamda tenant breach oldini olish.
- **Xavf:** RLS/context leak, permission caching stale bo'lishi.
- **Ta'sir:** Multi-Tenant, Security, Database, UX.

### ADR-BE-008 — Observability and controlled extraction
- **Muammo:** ko'p tenantda production muammo tashxisi va scale.
- **Variantlar:** ad hoc logs; structured observability + SLO; microservices immediately.
- **Tanlov (PROPOSED):** logs+metrics+traces+audit, migration mezonlari bilan bosqichma-bosqich extraction.
- **Sabab:** qarorlarni real dalillarga tayash.
- **Xavf:** telemetry xarajatlari, PII leakage, metric cardinality.
- **Ta'sir:** Infrastructure, Security, Master Architecture.

## 14. Codex uchun BACKEND SPECIFICATION (kod yozish topshirig'i emas)

> Quyidagi bo'lim keyinchalik Codexga beriladigan implementatsiya cheklovlari bo'lib, hozir dastur yoki SQL yaratishni buyurmaydi. `PROPOSED` qismlar faqat tegishli ADR tasdiqlangandan keyin implementatsiya qilinadi.

**Architecture constraints**
1. Backend candidate — Go; language/framework tanlovini APPROVED sifatida qabul qilma.
2. Modul ichida domain/application/ports/adapters/transport mas'uliyatlarini ajrat; repositoryga faqat owner modul yozadi.
3. Global user identity, tenant/company/branch/membership kontekstini mustaqil tekshir; har bir read/write uchun scoped access, default deny.
4. Idempotency talab qiluvchi operationlarda key uniqueness va result reuse hamda conflicting payload rejectionni tavsifla.
5. Moliyaviy qiymatlarni aniq decimal/minor-unit representation bilan ifodala; 1 kredit=10 so'm qoidasini billingning bitta authoritative joyida saqla.
6. Local transactionda biznes mutatsiyasi+outbox event atomar; external calls transaction tashqarisida; event consumers idempotent.
7. Seat/Business Usage/Resource Usage independent source metering, central pricing calculation va credit ledger eventlarini ajrat.
8. Balans 0 bo'lganda haqiqiy Seat+Usage charge minus ledgerga o'tishini va bir kalendar oy chegarasini buzma; taxminiy minus limit hisoblama. Oydan keyingi policy ochiq qoladi.
9. API OpenAPI kontrakti, versioned JSON response/error taxonomy, stable machine error code, request_id, locale, pagination; clientlar faqat contractga bog'lanadi.
10. Worker lease/retry/DLQ, correlation trace, job dedup, monthly scheduler drift protection; reprocessing bilan duplicate debit yoki posting yuz bermasin.
11. Logs structured; secret/PII redaction; tenant ID yuqori cardinality metric labeliga qo'yilmaydi; audit mustaqil.
12. Graceful shutdown, request deadlines, health probes, pool caps, bounded concurrency, rate limiting, tenant fairness va load-test acceptance gates bo'lsin.
13. Cross-tenant access, duplicate webhook, concurrent sale posting, partial payment, queue crash/replay, stale role, ledger reconciliation, worker retry uchun test scenarios hujjatlashtirilsin.
14. Microservice extractionga qadar bir modulning ichki schema/adapterini boshqa modul import qilmasin; public ports va versioned events orqali dependency saqlansin.
15. Tasdiqlanmagan seat formula, usage unit prices, credit volumes, request thresholds, SLAs, DB isolation turi yoki service splitni o'zboshimchalik bilan tanlama; OPEN QUESTION sifatida qayd et.

**Acceptance criteria**
- Har bir mutating business use-case uchun tenant+permission+idempotency+transaction/event boundary xaritasi mavjud.
- Credit ledger/stock ledgerda double applicationga qarshi mexanizm tavsiflangan.
- Bir node va ko'p node worker retry/crash testlari uchun expected behavior belgilangan.
- OpenAPI/event contract evolution va backward compatibility mexanizmi ko'rsatilgan.
- Monitoringda request, DB, job, outbox, usage, reconcile, tenant isolation critical signal ro'yxati mavjud.
- SLO, isolation, queue vendor, auth/session model yechilmaguncha ularni APPROVED deb ko'rsatmaydi.

## 15. Sinxronizatsiya uchun taklif qilingan Markdown yozuvlari

Fayllar bevosita o'zgartirilmagan; boshqa hujjatlarga qo'shishdan oldin ular ichidagi mavjud ADRlarni tekshirish kerak.

### DECISIONS.md — proposed additions
- `ADR-BE-001` Go candidate — `PROPOSED`
- `ADR-BE-002` Modular monolith — `PROPOSED`
- `ADR-BE-003` Transactions and idempotency — `PROPOSED`
- `ADR-BE-004` REST API contracts — `PROPOSED`
- `ADR-BE-005` DB-backed workers — `PROPOSED`
- `ADR-BE-006` Transactional outbox events — `PROPOSED`
- `ADR-BE-007` Verified tenant auth — `PROPOSED`
- `ADR-BE-008` Observability and extraction — `PROPOSED`

### MASTER_ARCHITECTURE.md — proposed reference
- Backend ko'p instancega mos modular monolith bo'lishi mumkin; bounded contextlar public port/event contracts bilan ajratiladi. Qaror `BACKEND_ARCHITECTURE.md` va `ADR-BE-002`ga bog'liq, `PROPOSED`.

### REQUIREMENTS.md — proposed cross-functional requirements
- Tenant isolation defense-in-depth; idempotent critical writes; durable background processing; auditable credit/stock/finance mutation; versioned API/event contracts; structured observability; graceful shutdown, consistent localization — texnik yechimlar tasdiqlanmagan.

### MODULE_REGISTRY.md — proposed references
- Identity, Tenant, Sales, Inventory, Finance, Billing, CRM, Integration, Notifications, Audit modullarining ownership va public portlari `BACKEND_ARCHITECTURE.md` 3-bo'limga bog'langan. Catalog/Purchasing boundary `OPEN QUESTION`.

### OPEN_QUESTIONS.md — proposed additions
- `OQ-BE-01` ... `OQ-BE-08` yuqoridagi 12-bo'limdagi savollar.

### CHANGELOG.md — proposed entry
- `2026-10-11` — Backend architecture v1.0 proposal: Go candidate, modular boundaries, API/security, local ACID/outbox, jobs, telemetry, extraction criteria; **no technical APPROVED decisions**.

## 16. Reference notes

Texnologik imkoniyatlarni tekshirish uchun rasmiy qo'llanmalar: Go `database/sql` transactions and context cancellation (https://go.dev/doc/database/), OpenTelemetry Go documentation (https://opentelemetry.io/docs/languages/go/). Reference mavjudligi tanlovni APPROVED qilmaydi.

## 17. Cross-module ownership and command/event reconciliation — PROPOSED (2026-10-12)

This section clarifies AR-002, AR-005, AR-013 and MR-10 by aligning with the latest Catalog/CRM/Sales/Finance/Database/Security proposals; it does not independently approve an unapproved business or platform decision.

| Fact / aggregate | Candidate authoritative write boundary | Read/reference consumers | Approval question |
|---|---|---|---|
| Product/SKU/UoM | Separate logical Catalog boundary (within modular monolith); Pricing separately owns PriceList | Sales, Inventory, Purchasing, CRM, Integrations | Tenant-global vs company catalog and final module container OPEN |
| SalesOrder and commercial document | Sales | Inventory, Finance, Billing, CRM | Billable lifecycle boundary and invoice transition OPEN |
| StockMovement and StockReservation | Inventory | Sales orchestration, Purchasing, Finance valuation | Negative-stock and reservation policy OPEN |
| Finance Journal/AR/AP | Finance | Sales uses outcome references, CRM reads authorized balances | GL posting on dispatch vs invoice based on agreed policy OPEN |
| Party/Counterparty, Outlet, Payer | Party canonical master owner **OPEN** (CRM vs shared Party context); CRM owns customer relationships/outlets | Finance owns receivables and payer allocations; Integrations external mapping | Merge/legal entity/customer role mapping OPEN |
| User credentials and session | Identity | All request contexts | MFA/SSO details OPEN |
| TenantMembership and Company/Branch membership context | Tenant & Organization (candidate) | Security policy evaluation, Backend context resolution | Exact membership and role cardinality OPEN |
| Role grants, policy and permission evaluation | Security (candidate) | Backend always enforces on server, UX displays hints only | Scope inheritance and revoke timing OPEN |
| Subscription, UsageEvent and SaaS Wallet | Billing | Domain services emit source facts but never debit wallet | BillingAccount cardinality and usage month OPEN |

**Commands vs facts (PROPOSED):** `ConfirmSalesOrder` writes a commercial state and may request StockReservation, not a physical stock debit. `PostDispatch` writes an idempotent Sales dispatch fact, Inventory posts exactly one stock movement; `IssueInvoice` writes immutable invoice fact and Finance posts at the explicit, still OPEN recognition trigger. `RecordUsage` is a Billing-owned command caused by a versioned chargeable domain fact, never by UI button clicks alone. Cancellation/return generates a linked compensating fact; original ledgers are append-only.

**Contracts:** Commands require `tenant_id`, authorized `company_id/branch_id` as applicable, `actor_id`, `expected_revision`, `idempotency_key`, `request_fingerprint`, `causation_id`, `correlation_id` and validated domain input. Events carry `event_id`, `type`, `schema_version`, tenant scope, canonical aggregate IDs, UTC occurrence/commit timestamps and immutable effect references. Separate outbox and consumer dedup/ordering protect retries; each domain persists authoritative effects atomically within its own owned transaction. Cross-domain workflows needing strict atomicity must share an explicitly designed transaction owner; asynchronous paths require reconciliation and do not promise impossible distributed exactly-once execution.

**Authorization invariant:** Identity verifies principal; Tenant authenticates active membership and verified organization context; Security evaluates versioned grants; application commands reject stale/revoked scope; database RLS is proposed defense-in-depth, not a substitute for application authorization. Off-device sync must re-authorize upon server reception.

**Open approvals:** real financial posting transitions, wallet/account cardinality, canonical Catalog/Party ownership, permission inheritance and event-schema compatibility/retention. These remain marked OPEN in the central registry and are never upgraded to APPROVED here.

## 18. ROUND 2 CORRECTION — AR-002 / AR-005 / AR-013; MR-07 / MR-10 / MR-11 (2026-10-12)

**Status: PROPOSED technical contracts; OPEN domain/business approval.** This addendum refines earlier §§3–8 and §17, but does **not** approve Go, modular monolith, a Catalog location, accounting recognition, payer mapping, or role-grant owner. Existing APPROVED business invariants remain unchanged: Hybrid Prepaid; inexpensive Seat + Business Usage + Resource Usage; per-document base and threshold credits; 1 credit = 10 UZS; progressive discounts/enterprise contracts; only actual Seat/Usage expenses accrue negative for one calendar month, **no estimated credit limit**; unified companies/branches; uz/en/ru plus hidden future alphabet. Read with [Round 2 audit](../reviews/ARCHITECTURE_REVIEW_ROUND_2.md), [Database §6/§17](DATABASE_ARCHITECTURE.md), [Sales §11](SALES_INVENTORY.md), [CRM §11](CRM_DISTRIBUTION.md), [Finance](FINANCE_ARCHITECTURE.md), [Billing §20](BILLING_ARCHITECTURE.md), [Multi-Tenant §16–17](MULTI_TENANT_ARCHITECTURE.md), [Security §17–19](SECURITY_ARCHITECTURE.md) and [Integrations](INTEGRATION_ARCHITECTURE.md).

### 18.1 AR-013 — identity, membership and permission authority

A *writer* alone may mutate canonical state; *readers* consume immutable references or public versioned queries/events; *enforcement* is required at every server execution boundary. The following is a **candidate split, not APPROVED**.

| Concept | Candidate authoritative writer | Authoritative readers / derived consumers | Backend enforcement and change notification | OPEN dispute |
|---|---|---|---|---|
| IdentityAuth (UserCredentials, MFA, sessions, device trust) | Identity/Auth bounded context (Security's identity capability) | Authentication middleware; TenantMembership only stable \`user_id\`, not credentials | Verify issuer/audience/expiry/signature/session status; reject revoked principal/device; emit \`PrincipalRevoked.v1\` / \`SessionRevoked.v1\` | Identity ownership packaging and SSO/MFA policy |
| Tenant/Company/Branch graph | Multi-Tenant | Security evaluator, Billing attribution, every domain's tenant resolver | Resolve actor's active context and resource hierarchy **server-side**; reject cross-tenant/branch read/write | LegalEntity/payer/cardinality and delegated company memberships |
| TenantMembership (\`membership_id\`, \`user_id\`, \`tenant_id\`, status/version) | **Multi-Tenant proposed sole membership-state writer** | Security PermissionEvaluation and session-context resolver; Billing seat attribution reads transitions only | Validate active membership and requested company/branch against current server state **on each command**; changes issue \`MembershipChanged.v1\` with revision | Security/Identity older wording says it owns memberships: unresolved AR-013, no silent override |
| RoleAssignment / grants, policy versions | **Security proposed sole role/grant writer** | PermissionEvaluation; tenant provisioning and UX projections read-only | Check actor, resource, action, scope, policy/grant revision; revoke invalidates authorizations | Security vs Multi-Tenant packaging and role inheritance OPEN |
| PermissionEvaluation (decision service, not authoritative membership store) | Security owns policy/evaluation behavior; no separate membership/credential writer | API handlers, workers, mobile sync, export, search, scheduled jobs | **Backend is mandatory policy-enforcement point**: fail closed, no client-side permission authority | Cached decision freshness/revocation SLA OPEN |

Flow: authenticate principal → resolve tenant/company/branch **from trusted server data** → verify active membership/version → evaluate Security permission + object/branch grants → open tenant-scoped Unit of Work → persist owned command + outbox → audit decision. Tokens can convey candidate claims, **never alone authorize** stale membership, grants, branch switch or cross-tenant object ID. Worker/service principal must carry scoped initiating actor or explicit auditable system authorization; delayed mobile sync always rechecks current permission and revision at replay; revocation rejects uncommitted tasks (post-commit reversal is separate). RLS is defense-in-depth, not a replacement for evaluator. With transaction-pooled connections, tenant RLS context must be set transaction-locally, proven reset on reuse, with negative cross-tenant tests for API, worker, read models, exports, search and sync; a privileged bypass requires explicit separately audited boundary. MR-11 test outcomes are **NOT VERIFIED**.

### 18.2 AR-002 / MR-07 — disjoint authoritative business writers

| Bounded context | Exclusive canonical mutable facts (candidate) | References/consumers | Must not own | Status / unanswered choice |
|---|---|---|---|---|
| Catalog | Product identity, SKU/variant, taxonomy, barcode, UoM/conversion | Sales, Pricing, Inventory, Purchasing, CRM | Price rules, SalesOrder, StockBalance | PROPOSED; tenant-global vs company scoped, actual module container OPEN |
| Pricing | PriceList/rule, applicability calculation, price quote/version | Sales snapshots resolved price + currency, UoM and tax basis | Catalog SKU identity, SaaS subscription rating | PROPOSED; precedence/company/branch scopes OPEN |
| Sales | SalesOrder, commercial confirmation, fulfillment intent, Sales dispatch command acceptance/status | Inventory, Finance, CRM, Billing subscribe to typed commercial facts | Physical stock ledger, GL, wallet, canonical Product | PROPOSED; dispatch workflow authority vs physical posting boundary OPEN |
| Inventory | StockReservation, StockMovement/Balance, warehouse/transfer/stock count | Sales, Purchasing, Finance valuation | Sales document, invoice, price quote, Finance journal | PROPOSED; negative stock, reservations, valuation OPEN |
| Purchasing | PurchaseOrder and supplier receipt *intent*/purchase lifecycle | Inventory posts physical receipt; Finance owns AP | Stock ledger, canonical Party, GL | PROPOSED; no-PO receipt and tolerances OPEN |
| Party/CRM | **Party master writer OPEN**: CRM-owned Party registry or separate shared Party context; CRM owns CustomerRole, Outlet, routes/visits, relationships | Purchasing SupplierRole, Sales customer ref, Finance debtor ref, Integrations external IDs | SalesOrder, stock, legal receivable ledger, credentials | PROPOSED split; payer ≠ customer ≠ outlet ≠ legal debtor by default; merge/legal mapping OPEN |
| Finance ERP Accounting | GL journal and lines, AR/AP, fiscal invoice/credit note as agreed | Sales/Inventory/CRM source business facts | SaaS credit wallet, usage charges | PROPOSED; revenue/COGS posting trigger, invoice creator/tax ownership OPEN |
| SaaS Billing | Usage ingestion, rating, unique charge occurrence, SaaS credit ledger, wallet/billing invoice | Source domains emit **non-monetary** lifecycle facts; Billing resolves approved chargeable trigger | Tenant ERP GL and stock | APPROVED monetization invariants; event trigger/account cardinality/period policy details OPEN |

Product ID is not SKU ID. Sales lines store immutable selected SKU/UoM/price quote snapshot. References use owner public contracts, scoped IDs, and versioned event payloads rather than writing another module's tables. Previous "Sales owns products/inventory" phrase in CRM/Sales documentation is a **disputed historical broad statement**; this bounded-context separation is a PROPOSED reconciliation, **not** authority to mark CRM/Sales ADRs APPROVED or alter them here. Party canonical identity must have one owner before implementing cross-module dedupe; immutable external mappings use \`tenant_id + source_system + external_type + external_id\` and preserve merge aliases/history.

### 18.3 MR-10 / AR-005 — typed command/event ownership and consequences

Command schemas are versioned contracts (suggest \`command_type.v1\`); events are **facts after source-owner commit**, not commands for listeners to assume billing/GL policy. Every command carries \`command_id\`, \`idempotency_key\`, payload hash, \`tenant_id\`, server-validated \`company_id\`, optional \`branch_id\`, principal/membership, expected aggregate revision, correlation/causation and occurred-at if offline (server received-at separately). Every event carries \`event_id\`, \`event_type\`, \`schema_version\`, \`business_occurrence_id\`, \`aggregate_id/revision\`, \`tenant/company/branch\`, \`occurred_at\`, \`committed_at\`, correlation/causation, producer and contract payload. Event IDs and occurrence IDs are **different identifiers**.

| Transition command / owner | Owner-specific atomic state + outbox | Versioned owner event | Downstream authorized command/effect (separate unit) | Business rule pending |
|---|---|---|---|---|
| \`ConfirmSalesOrder.v1\` / Sales | Order CONFIRMED, revision, unique confirmation occurrence and outbox; **no stock decrement** | \`SalesOrderConfirmed.v1\` | Inventory may execute \`ReserveStock.v1\` with Sales occurrence ref; no automatic GL | Whether reservation required/allowed and Billing chargeability OPEN |
| \`ReserveStock.v1\` / Inventory | Reservation and available-stock conditional update; outbox | \`StockReserved.v1\` / \`ReservationRejected.v1\` | Sales updates derived fulfillment/projection; no Sales direct stock write | TTL/partial reserve/negative stock OPEN |
| \`PostDispatch.v1\` / Sales commercial boundary; Inventory physical posting owner | Sales dispatch/fulfillment intent and outbox **separate** from Inventory atomic \`PostStockMovement.v1\` movement+balance+outbox | \`DispatchRequested.v1\` → \`StockMovementPosted.v1\` → \`DispatchCompleted.v1\` (Sales after confirmed stock effect) | Finance may receive fulfillment fact and independently post if accounting policy approved; no double stock effect | Dispatch owner/orchestration, revenue/COGS timing OPEN |
| \`IssueInvoice.v1\` / Finance (candidate fiscal document owner) | Invoice immutable issuance fact, uniqueness and outbox; **no inventory movement** | \`InvoiceIssued.v1\` | Finance may \`PostGLJournal.v1\` if approved trigger; Sales consumes invoice ref | Sales vs Finance invoice document split, taxes, GL trigger OPEN |
| \`PostGLJournal.v1\` / Finance | Balanced journal+lines, accounting period lock, unique posting occurrence and outbox | \`GLJournalPosted.v1\` | Sales/reporting may project posted status; never direct finance table mutation | AR/COGS/revenue recognition and reversal policy OPEN |
| \`RecordSaaSUsage.v1\` and \`PostSaaSCharge.v1\` / Billing | Ingest source business occurrence, dedupe; when approved eligibility/rating exists atomically persist unique actual rated charge+credit ledger+wallet summary+outbox | \`SaaSUsageAccepted.v1\`, \`SaaSChargePosted.v1\` | Analytics/notifications projections; **no tenant ERP GL posting** | Exact billable transition, attribution, late/offline month and negative-balance lifecycle OPEN |

A failed reservation/dispatch/invoice/GL/charge must not leave the source committed fact "undone" invisibly: persist pending/failed status and compensating/reversal facts as appropriate, retry, and reconcile. Confirm does not physically ship goods; invoice must never re-decrement stock; Finance and Billing have **distinct money ledgers**.

### 18.4 Canonical occurrences, idempotency and delivery invariants

- \`event_id\`: unique immutable **envelope/publication instance**; an event may be retried/redelivered with identical ID; new schema projection/republication can have new event ID and still refer to the **same** \`business_occurrence_id\`.
- \`business_occurrence_id\`: stable, source-owner-assigned identifier for **one semantic business action** (confirmed revision, reservation attempt, dispatch/stock posting, invoice issuance, GL posting, approved SaaS chargeable occurrence); never derive from message transport delivery ID or billing month. Include business occurrence type and tenant scope in canonical uniqueness; retried commands retain same ID. Source fact and independently created downstream effect each receive distinct occurrence IDs linked by \`caused_by_business_occurrence_id\`. Never reuse a single occurrence ID across distinct actual charges unless Billing's billable identity contract explicitly does so.
- API idempotency uniqueness candidate: \`(tenant_id, command_type, actor-or-integration-scope, idempotency_key)\`; store canonical payload fingerprint and immutable terminal response/reference in same owner transaction. **Same key + same payload** returns prior result/status (not new mutation); **same key + different normalized payload** yields deterministic \`409 IDEMPOTENCY_PAYLOAD_CONFLICT\` without effect. Concurrent in-flight same-key requests serialize or return \`409 OPERATION_IN_PROGRESS\` with safe retry hints. Retention horizon and normalization rules OPEN.
- Outbox: owner-owned aggregate state, command dedupe/result and pending event persist in **one local ACID commit**; relay publishes asynchronously with bounded retry/backoff+jitter and poison quarantine. Broker at-least-once means duplications expected; never assert exactly-once transport. On publish success update delivery state; if ack lost consumer inbox still protects effects.
- Inbox: unique \`(consumer_name, tenant_id, event_id)\` for transport replay plus unique **business-effect key** (e.g. \`(tenant_id, effect_type, source_business_occurrence_id)\`) to guard re-enveloped events and late replay. Inbox claim and owner effect/write/outbox persist atomically; failed transaction rolls back inbox claim. Duplicate same occurrence+same immutable payload = no-op; same business-effect key but different payload = quarantine/conflict, no merge or silent overwrite.
- Ordering: guaranteed only within **one aggregate revision stream** where producer sequence/revision is monotonic; no assumption of global ordering. Out-of-order events held/retried until predecessor, or reconciled against authoritative owner snapshot; stale events cannot regress state. Concurrent operations use expected revision/ETag and conditional updates.
- Partial failures: source committed but downstream unavailable → durable outbox backlog + consumer retry; poisoned messages → DLQ/manual remediation; changes of business intent use new compensating facts linked to original IDs, not destructive rewrite. Daily/periodic reconciliation compares Sales dispatch↔Inventory postings, Invoice↔Finance journal, Billing eligible source occurrences↔unique charges, and owner state↔read projections. Reconciliation cadence/SLO OPEN.
- Cross-tenant guard: all dedupe and occurrence uniqueness keys tenant-scoped, while external global provider IDs require separately scoped provider/source uniqueness and permission checks. Timestamp \`occurred_at\` must not alone choose SaaS billing period: late offline attribution remains Billing OPEN.

### 18.5 DB §6 versus §17 — transaction boundary decision packet

| Design candidate | Atomic scope / ownership | Partial failure and retry | Double posting, reversal and reconciliation | Implications |
|---|---|---|---|---|
| **A: one cross-domain ACID Sales+Inventory+Finance transaction** (historical DB §6 PROPOSED wording) | Shared DB and orchestrator writes SalesDocument+StockMovement+GL together | Full rollback if any owner fails; retries require cross-owner locks/ordering; fails with future DB split or asynchronous policy | Unique idempotency keys can avoid duplicates; reversal crosses contexts; reconciliation simpler only while tightly coupled | Tight ownership coupling, long locks, no independent dispatch/invoice accounting timing; not recommended default |
| **B: owner-specific atomic transaction + outbox** (**recommended PROPOSED**, DB §17 compatible) | Sales order+outbox; Inventory reservation/movement+outbox; Finance journal+outbox; Billing charge ledger+outbox in separate owner commits | Source remains committed when downstream fails; durable retry/DLQ, status projection and explicit compensation | Per-owner occurrence uniqueness + inbox/effect key prevents double posting; linked immutable reversals; periodic cross-owner reconciliation required | Eventual consistency and operational observability; works with future extraction/shards |
| C: distributed 2PC | Global commit coordinator across DBs/services | Can block under partitions; coordinator recovery required | Strong global atomicity where participants support XA, but costly/fragile | Avoid by default; no approval |

**Recommendation:** B for all cross-domain workflows; local ACID within **each authoritative writer** only. It is a *technical recommendation*, not approved accounting timing. Earlier Backend §4 wording "bitta DBdagi kuchli bog'langan use-case" must be read as **one owner-specific** aggregate/unit of work, not permission to write another owner's Sales/Inventory/Finance ledger together. Database §6's combined posting is an **alternative/historical PROPOSED interpretation**, not executable MVP mandate; Database §17 is the compatible candidate. Only Database owning chat may edit §6 to formally resolve textual contradiction; current cross-document discrepancy remains explicitly OPEN until that sync. A cross-domain synchronously invoked owner command remains independently committed, not magically one ACID transaction.

### 18.6 Versioned API, schema evolution and predictable contracts — MR-10

| Concern | PROPOSED contract | Compatibility / rejection behavior |
|---|---|---|
| Transport/version | Public HTTPS REST + published OpenAPI, \`/api/v1\` resource path; internal ports may be in-process; gRPC later only by evidence | Contract-first review; major breaking changes new \`v2\`, coexistence/migration window OPEN |
| Command concurrency | \`Idempotency-Key\` for externally retried mutations; \`If-Match\`/resource revision for update/transition | Missing required precondition \`428 PRECONDITION_REQUIRED\`, stale revision \`412 PRECONDITION_FAILED\`; duplicate key conflict 409 |
| Query pagination | Bounded \`limit\`, opaque cursor/keyset (stable sort by created timestamp + immutable ID), tenant-scope/filter hash in cursor | Invalid/expired cursor \`400 INVALID_CURSOR\`; avoid unbounded offsets and cross-tenant cursors |
| Schema evolution | Additive optional fields, preserve field meaning and enum-safe unknown handling; event envelope schema version independent of API version | Never repurpose field meanings or delete mandatory fields within v1; consumers ignore unknown optional fields; incompatible change new major |
| Lifecycle | Deprecation notice, telemetry of client version, documented migration path, defined sunset/compatibility dates **OPEN** | No abrupt client breaking change, especially offline mobile clients |
| Errors | Uniform \`code\`, \`message_key\`, \`request_id\`, \`correlation_id\`, \`retryable\`, optional \`field_errors\`/\`current_revision\`, HTTP status; no PII internals | 400 bad request; 401 unauthenticated; 403 forbidden; 404 nonexistent/not-visible; 409 duplicate/conflict; 412 stale; 422 domain invariant; 429 throttled; 503 dependency unavailable, Retry-After where applicable |
| Search | Scoped \`/api/v1/search\` read model, filter by effective membership/grants per result, tenant/company/branch projection version | No autocomplete leakage/cross-tenant counts; stale projection marked \`as_of\`, authorization failures never masked by cached rights |
| UX read models | Lightweight owner-published materialized summary of order/stock/balance status (read-only), bounded fields, indexed queries, read-your-write via owner endpoint when needed | Projection lag and timestamp explicitly disclosed; no financial command from projection |
| Observability | Correlation/request/causation IDs trace API→outbox→consumer→effect, structured logs without secrets, latency/error/queue age metrics | Limit tenant-ID high-cardinality metrics; operational SLO/thresholds require approval |

**Authorization pipeline (all entrypoints):** inbound API, external webhook, worker, scheduler, mobile offline/sync, imports and admin tooling verify machine or human principal → server-resolved tenant/branch context → fresh membership and Security grants → resource owner+revision check → permitted command; absence/uncertainty **fail closed**. UI availability, signed token claims, cursor or cache do not confer permission. Search authorization must re-check before returning rows, including historical access after membership revocation. Enforce tenant RLS using transaction-local context when selected; verify connection pool cleanup.

### 18.7 Reconciliation tests and OPEN decisions (specification, not executed)

**Proposed negative/contract acceptance matrix:** 1) duplicate confirmations produce one transition; 2) same idempotency key/different payload returns 409 with no effects; 3) lost relay acknowledgment triggers harmless replay; 4) re-envelope with new event_id but same occurrence never double debits stock/GL/wallet; 5) out-of-order reservation/dispatch events cannot regress status; 6) delayed billing remains unrated until approved chargeability and period rules; 7) revoked user or device fails API/worker/offline replay; 8) tenant A cannot read/write/search/export tenant B even after pooled connection reuse; 9) invoice never triggers second stock movement; 10) Finance GL and SaaS Billing ledgers are independently reconciled; 11) backward-compatible v1 client accepts additive event/API fields; 12) read models mark staleness and never substitute for source owner. **These tests have not been executed.**

**Still OPEN, user/domain approval needed:** catalog namespace/placement; canonical Party writer and legal payer mapping; membership writer vs older Security/Billing wording; RoleAssignment ownership and revoke freshness; Sales-vs-Inventory dispatch orchestration; fiscal invoice creator; GL/AR/COGS/revenue recognition trigger; reserve TTL/negative stock; exactly which lifecycle transitions are billable; offline source timestamp vs billing calendar month, negative-one-calendar-month interpretation and payer account cardinality; event/idempotency retention windows; API support/deprecation windows; RLS/pooling validation and service placement; queue broker and measured SLOs. **No implicit approval or implementation authorization.**

**ADR linkage (owned only):** [ADR-BE-002](../adr/ADR-BE-002.md), [ADR-BE-003](../adr/ADR-BE-003.md), [ADR-BE-004](../adr/ADR-BE-004.md), [ADR-BE-006](../adr/ADR-BE-006.md), [ADR-BE-007](../adr/ADR-BE-007.md), [ADR-BE-008](../adr/ADR-BE-008.md). Shared registries **not edited**; proposed central sync: record AR-002/005/013 open authority decisions and MR-07/10/11 contracts, link this section, and retain Database §6 as historical/alternative PROPOSED until its owner reconciles.
