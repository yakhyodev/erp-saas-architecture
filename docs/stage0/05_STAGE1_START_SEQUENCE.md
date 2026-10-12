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
| S1-01 | 01 foundation: repository/runtime/contract tooling | S1-00, real repo owner | Bootstrap/config, public ports, redacted errors, fail-closed flags; no enabled business store |
| S1-03 | 01 DB-first synthetic tenant/pool/local-transaction harness | S1-01, real authorized local/test Postgres; **S1-02ga dependency yo‘q** | Composite refs/RLS/non-bypass roles, real pool A→B/rollback; state+receipt+outbox and inbox/effect fault fixtures. Non-public synthetic scope; authdan oldin containment PG-TEN/TXN actual evidence |
| S1-02A | 01 Identity slice | S1-03 containment, Identity/session/device policy spec | Only Identity writer; current credential/session/device/refresh/revoke fixtures; login tenant permission emas |
| S1-02B | 01 Tenancy slice | S1-02A + S1-03; IC-01/02/03 producer adoption | Only org/member writer; effective scopes/versions and composite refs; no grants/Wallet writes |
| S1-02C | 01 Permissions + domain guard slice | S1-02A/B + S1-03; C06/C09/O01 contract freeze | Only grant/policy writer; fresh receipt-result AuthZ, commit/revoke fence interleavings F-O01A, safe scoped switch; enabled auth PG-SEC/TEN evidence |
| S1-04 | 01 relay + domain-consumer compatibility harness | S1-03 + S1-02C service authority + adopted corrected IC envelope | Namespaced guards F-C03, terminal replay F-C06, resumable pending/ACK F-C07, scoped per-consumer leases F-O01B; crash/upcast fixtures. Real external effect yo‘q |
| S1-05 | 04 Next.js/Flutter read-only shells + uz/en/ru | Frozen C09/API/error spec; protected enabled shells S1-02C va S1-03 evidence’dan keyin; sketches earlier | Clear/lock-before-new-authorization + generation fencing; F-C09A…D Company/Branch delayed/denied/back/resume/revoke/multi-tab; truthful pending/UNKNOWN; hidden OFF, **local business queue/replay implementation yo‘q** |
| S1-06 | 02/03 non-economic consumer boundary fixtures | S1-03/04 + S1-02B/C; adopted owner ports and C01/C03/C04/C05/C06/C07 | Raw pending meter, namespaced economic refs, stable external mapping, Seat-subject history, receipt replay and pending liveness fixtures; money/stock/GL/provider disabled |
| S1-07 | 05 integration evidence review + 01/04 fixes in their own directories | S1-03…06 actual artifacts | Candidate merge/head SHA bo‘yicha PG/IG disposition, remaining OPEN/T evidence; Stage 2 handoff PR with precise disabled paths |

Parallelizm faqat predecessor talab qilmaydigan scoped ishlar uchun: 04 UX spec/read-only mock contract, 02/03 domain contract fixture design. Boshqa chat implementatsiyasini oldindan mavjud deb taxmin qilib patch qilinmaydi. Bu jadval delegated agent ishini ishga tushirish yoki boshqa chatga message yuborish emas.

S1-06 tijoriy Seat hisoblash yoki stock posting emas; accepted kontraktlarni synthetic/non-economic fixture’da tekshirishdir. Haqiqiy Sales/Catalog/Inventory implementation baseline Stage 2/3da, Wallet debits Stage 4da, ERP GL Stage 5da alohida scope bilan keladi.

### 2.1 Domain review backloglari bilan mapping

S1-02 eski logical family nomi; corrected graph `S1-01 → S1-03 → S1-02A → S1-02B → S1-02C → S1-04`, undan S1-06/integrated S1-05→S1-07. Numeric IDsni topological order deb o‘qimang. S1-03 synthetic harness authsiz public authority bypass emas. Pooled/RLS containment tekshiruvi enabled authdan keyinga surilmaydi.

| Owner review’dagi local PR IDs | Global dependency / scope |
|---|---|
| 01 PR-001/002/003A/003B/003C/004 | Mos ravishda S1-01/03/02A/02B/02C/04; local IDlar GitHub PR raqami emas |
| 02 S1-BF-01…06 | Ports/ref freeze S1-00/01; legal/membership fixtures S1-02B/C; namespace/replay/policy-pending S1-03/04/06; status client DTO 04 review. Real Wallet/GL/tariff/provider engine emas |
| 03 C03-S1-01…05 | Corrected C01/C03/C05/C06/C07 schemas→S1-02B/C current scope→S1-04 pending/replay→S1-06 cross-owner synthetic→S1-07 evidence; no real physical stock/live SalesDoc |
| 04 04-S1-PR-001…007 | Consumer C09/error/locale freeze→design system→API adapters→S1-02C protected shells→switch generation/cache tests→truthful read-only states→S1-03/04 integration evidence. C08 actual local business queue faqat later-phase policy/implementationda |

09 IC/09 IG/07 PG bilan detailed mapping [03 §4.1](03_PR_AND_INTEGRATION_GATES.md#ic-fixture-stage-matrix), named scenarios [07](07_DELTA_ACCEPTANCE_FIXTURES.md). Owner review backloglari taklif sifatida qayd etilgan; bu mapping user code permission yoki boshqa owner business approval emas.

## 3. Feature enable matrix

Quyidagi flag nomlari existing handoff’dagilar va qo‘shimcha proposal labels. Flag config/schema hozir yaratilmagan. Backend use-case ham fail closed enforce qiladi; faqat UI button yashirish yetarli emas.

| Feature / Stage 1 default | Keyin enable qilish uchun exit |
|---|---|
| `billing_posting=false` | **C04 / G-MR02 mandatory:** 02 published approved prices/units/thresholds + precise approved anniversary/clamp/timezone/cycle/recovery/rearm/expiry rights/late-correction policy; customer terms, IC-03/05/08/09 adoption va actual PG-FIN/TXN/TEN. MR-02 OPEN bo‘lsa live Seat/usage debit OFF, tariff/reviewning o‘zi yetmaydi |
| Expiry service enforcement OFF (proposal) | Exact expiry/rearm/customer rights policy approved bo‘lmaguncha restriction yoqilmaydi; no hidden zero-balance hard-stop/no estimated credit cap. F-C04A fixture |
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
- Auth/tenant/pool/outbox namespace/terminal-replay/pending-liveness/lease/compatibility real fixtures executed; F-C03/C06/C07/O01 candidate SHA’dagi dalili; observed unresolved failures Stage 2ga explicit dependency.
- 04 read-only client smoke/task/scoping/a11y/locale va F-C09 switch-generation fixtures tested build SHA’da; timeout/revoke/unrated pending truthful. C08 future queue spec/qabuldan oldin atomic-save fixtures majburiy, actual offline delivery Stage 1 OFF.
- Economic/external/offline mutation flags backend’da disabled bo‘lishi negative test bilan ko‘rsatilgan.
- Runtime role va test environment dalili captured; DBHub demo natijasi real ERP Postgres/RLS evidence sifatida ishlatilmagan.
- Stage A real pilot provisioning/restore alohida scope/MR-14: C02 isolated recovery, current authority+external-effect reconciliation va resume manifest, F-C02A…C actual drill. Stage 1 synthetic replay/local CI pass Stage A restore/PITR yoki HA isboti emas.
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
