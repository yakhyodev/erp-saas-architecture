# CODEX 05 — Stage 1 uchun xavfsiz boshlash ketma-ketligi

Status: `PROPOSED — FUTURE EXECUTION ONLY`. Source: [handoff backlog](../handoff/MVP_IMPLEMENTATION_BACKLOG.md) va [readiness](04_STAGE0_READINESS.md). Quyidagi ishlar hozir bajarilmaydi. Stage 1 nomi baseline’dagi platform skeleton; Commerce/stock/Billing/Finance keyingi Stage 2…7ga tegishli bo‘lib qoladi.

## 1. Stage 0 → Stage 1 exit contract

1. CODEX 05 docs paketini GitHub docs PR orqali review/adopt qilish; explicit producer/consumer writer va directory ownership qaydi.
2. 01 Identity/Tenancy/Permissions/DB/pool/local-transaction contractlariga real technical review; 02/03 consumers scope/effect-key rulesiga rozilik, 04 error/UNKNOWN/compatibility oracle review.
3. Userning alohida **Stage 1 application implementation** ruxsati va haqiqiy target repository/path. Architecture planning delegation ruxsatni almashtirmaydi.
4. Authorized maintainer, repository owner identities, branch/review protections va expected CI environmentni aniqlash. Repository yaratish yoki provisioning vakolati mavjud deb taxmin qilinmaydi.
5. Stage 1 non-economic scope’ini freeze qilish; barcha real money/tax/outbound/offline mutation flags fail closed. Test fixtures’ synthetic policy va customer production tariff alohida.

Narx/jurisdiction/provider yetishmasa skeletonda payable amount yoki legal terms default to‘ldirilmaydi. Ko‘rsatilgan paths disabled qoladi. Shuning uchun faqat skeleton uchun kerak bo‘lmagan barcha production inputni oldindan so‘rab ishni to‘xtatish shart emas; named enable gate keyinchalik yopiladi.

## 2. Dependency-ordered Stage 1 PRlar

| Navbat | Lead / kelajak PR | Input / predecessor | Output va exit oracle |
|---|---|---|---|
| S1-00 | 05 contract freeze, faqat docs | Stage 0 adoption va application authorization | App SHA+architecture SHA, named scope, IC v1 spec, PG/IG obligations; no guessed approval |
| S1-01 | 01 repository/runtime/test-harness skeleton | S1-00, real repo owner | Modular monolith boundaries/composition; domain ports; separate runtime/migration identities; DB tx/outbox primitives specga mos; ownership checks |
| S1-02 | 01 Identity→Tenancy→Permissions vertical skeleton | S1-01, IC-01 technical sign-off | Principal vs Membership vs Grants alohida writer; server scoped read/command; commit-time guard, no economic effect |
| S1-03 | 01 tenant/pool/local-transaction acceptance | S1-02, real authorized local/test Postgres | A/B and branches negative fixtures; pooled rollback/retry/context test; composite FK/runtime privilege checks; actual PG-SEC/TEN/TXN artifacts |
| S1-04 | 01 relay + domain-consumer compatibility harness | S1-01/03, IC envelope | Source commit/publish/ACK crash replay; inbox+semantic effect no-op; same-key conflict; new schema envelope no duplicate; service scope authorization |
| S1-05 | 04 Next.js/Flutter read-only shells + uz/en/ru | S1-02 API/error contract; parallel client sketch docs may start earlier | Scoped login/Company/Branch switch; E/L/X/O/UNKNOWN states; keyboard/a11y; protected context and scoped caches; hidden alphabet OFF |
| S1-06 | 02/03 consumer boundary acceptance fixtures, no real business posting | S1-03/04; accepted owner ports | Membership Seat-subject projection example; Company/legal refs; source/effect lineage fixtures; actual money/stock/GL paths disabled |
| S1-07 | 05 integration evidence review + 01/04 fixes in their own directories | S1-03…06 actual artifacts | Candidate merge/head SHA bo‘yicha PG/IG disposition, remaining OPEN/T evidence; Stage 2 handoff PR with precise disabled paths |

Parallelizm faqat predecessor talab qilmaydigan scoped ishlar uchun: 04 UX spec/read-only mock contract, 02/03 domain contract fixture design. Boshqa chat implementatsiyasini oldindan mavjud deb taxmin qilib patch qilinmaydi. Bu jadval delegated agent ishini ishga tushirish yoki boshqa chatga message yuborish emas.

S1-06 tijoriy Seat hisoblash yoki stock posting emas; accepted kontraktlarni synthetic/non-economic fixture’da tekshirishdir. Haqiqiy Sales/Catalog/Inventory implementation baseline Stage 2/3da, Wallet debits Stage 4da, ERP GL Stage 5da alohida scope bilan keladi.

## 3. Feature enable matrix

Quyidagi flag nomlari existing handoff’dagilar va qo‘shimcha proposal labels. Flag config/schema hozir yaratilmagan. Backend use-case ham fail closed enforce qiladi; faqat UI button yashirish yetarli emas.

| Feature / Stage 1 default | Keyin enable qilish uchun exit |
|---|---|
| `billing_posting=false` | 02 approved published prices/units/thresholds/timezone/cycle, customer contract, IC-03/05/08/09 adopted, PG-FIN/TXN/TEN evidence |
| `finance_gl_posting=false` | 02 fiscal/accounting profile, debtor evidence, recognition/valuation/period controls; balanced journal/reversal/reconciliation actual tests |
| `external_writes=false` | Provider LIVE evidence, direction/single-writer, owning domain cutover acceptance, separate outbound authorization |
| `offline_mutation_sync=false` | 01+04 auth/device/TTL/retention spec, 03 allowed-intent catalog, IC-06 replay/conflict/UNKNOWN tests |
| Verified top-up credit disabled (proposal) | Provider authorization+settlement verified, target Wallet evidence, duplicate webhook/chargeback policy tests; pending never credited |
| GPS/media upload disabled (proposal) | Lawful purpose/consent/privacy/retention, scoped storage/security/device tests |
| Hidden future Uzbek alphabet OFF | Approved future standard/rollout, compatible locale/search/a11y verification |
| Physical stock mutation unavailable in Stage 1 (proposal) | Stage 3 permission + reservation/count/negative/valuation policy and actual concurrency/dispatch/return tests |

Config missing yoki flag storage unavailable bo‘lsa financial/outbound/offline side effectga allow fallback bo‘lmaydi. Flag OFF flow’da eligibility/price “pending”ni tushunarli ko‘rsatadi; real debitni taxminiy 0 yoki paid-success deb ko‘rsatmaydi. Test fixture’da enabled simulated policy bo‘lsa bu production authorization emas.

## 4. Stage 1 exit

- Latest main/contract dependency yozilgan scoped PRlar merged, writerlar va named scope review dalili bor.
- Auth/tenant/pool/outbox retry/fingerprint/compatibility uchun real tests executed; observed failures unresolved bo‘lsa Stage 2ga open dependency qaydi.
- 04 read-only client smoke/task/scoping/a11y/locale tests tested build SHA’da; timeout va revoked state truthful.
- Economic/external/offline mutation flags backend’da disabled bo‘lishi negative test bilan ko‘rsatilgan.
- Runtime role va test environment dalili captured; DBHub demo natijasi real ERP Postgres/RLS evidence sifatida ishlatilmagan.
- Stage A real pilot environment provisioning/restore uchun alohida authorized scope va MR-14 inputlari kerak. Stage 1 local CI success Stage A restore yoki production HA isboti emas.
- Stage 2 kickoff issue: canonical Party/Catalog/Pricing, legal mapping va MVP customer scope; Stage 3 physical effects; Stage 4 rates/provider; Stage 5 law/accounting — har biri named gate bilan.

## 5. Qachon to‘xtatish va kim tuzatishi kerak

| Trigger | Harakat |
|---|---|
| Other domain directory/entityga o‘zgartirish zarur | Owner change request + IC diff; writer review/commit kelmaguncha dependent patch yo‘q |
| Same occurrence yangi policy/event versionda ikkinchi effect yaratadi | Release gate FAIL; 02 yoki 03 owner guard/correction contractni tuzatadi, 05 consumer reviewni koordinatsiya qiladi |
| Cross-tenant/branch/pool leak, unauthorized effect | Tegishli gate FAIL; flag disabled; 01+source owner regression fix, reproducible evidence |
| Legal/tariff/provider ma’lumot yetishmaydi | O‘sha financial/external feature OPEN va disabled; vakolatli skeleton/non-economic ishlar davom etishi mumkin |
| Requirement APPROVED-vs-APPROVED contradiction | 05 source-linkli conflict qaydi; source/user authority resolution talab, taxminiy qoida bilan code fix yo‘q |
| Test infrastructure yo‘q yoki check bajarilmadi | NOT_TESTED/BLOCKED va aniq owner; passed deb ko‘rsatmaslik |
| Application authorization berilmadi | Architecture docs-only holat saqlanadi; source, migration, runtime provisioning va deployment boshlanmaydi |

**Kelajak kickoff uchun short contract:** eng yangi main va frozen architecture contractni o‘qing; named Stage 1 authorizationni tekshiring; o‘z directory/owner scope’ida bitta kichik vertical PR qiling; real PG/IG evidence’ni yozing; monetary/legal/external/offline pathsni ularning gate’i yopilmaguncha disabled saqlang. Boshqa chat natijalarini faqat GitHub merged docs/PR dalili orqali iste’mol qiling.
