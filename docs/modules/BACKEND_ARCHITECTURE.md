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