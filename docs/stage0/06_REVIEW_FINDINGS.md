# CODEX 05 — PR #2 review finding registry va correction disposition

Sana: 2026-10-12. Source repository `yakhyodev/erp-saas-architecture`; main `0b0b086b40fa7492aa4ca26bf3f1562f2a86ef50`; original reviewed PR head `5c9096eeef262ed8e5a5634935435dfc990f7dd4`. Correction head — ushbu registry’ni o‘z ichiga olgan PR #2 commit (exact SHA GitHub metadata/correction summary’da). Scope faqat `docs/stage0/`, writer 05; domain source/ADR/authoritative business decisions o‘zgarmagan.

## 1. Review inventory va disposition vocabulary

| Owner | Original report/review ID va URL | Original verdict | P1 |
|---|---|---|---:|
| CODEX 01 | [issuecomment 6123437500](https://github.com/yakhyodev/erp-saas-architecture/pull/2#issuecomment-6123437500), PLAT-R1/PLAT-R2 headings | CHANGES_REQUIRED | 2 |
| CODEX 02 | [review 5490148633](https://github.com/yakhyodev/erp-saas-architecture/pull/2#pullrequestreview-5490148633), 3 inline | CHANGES_REQUIRED | 3 |
| CODEX 03 | [review 5490182394](https://github.com/yakhyodev/erp-saas-architecture/pull/2#pullrequestreview-5490182394), 4 inline | CHANGES_REQUIRED | 4 |
| CODEX 04 | [review 5490144993](https://github.com/yakhyodev/erp-saas-architecture/pull/2#pullrequestreview-5490144993), 3 inline | ACCEPTED_WITH_CONDITIONS | 3 |

Hammasi paginated GitHub review/inline/conversation APIs orqali o‘qilgan: **3 formal COMMENT reviews + 1 top-level report + 10 original inline comments = 12 P1**, 0 original P0. CODEX 01da inline emas, bitta original commentning ikki named finding’i; ularni ikkita yangi GitHub comment ID deb uydirmaymiz.

`DOC_CORRECTED / DELTA_REVIEW_REQUIRED` = 05 normative wording va expected oracle’ni tuzatdi, owner review required. `OWNER_ACCEPTED` faqat original owner yangi corrected SHA’da actual review qayd etsa qo‘shiladi; bu registry buni oldindan belgilamaydi. Barcha **27 named fixture rowlari** va mahsulot testlari `NOT_TESTED`; docs validation software PASS emas. Original comments/threads resolved yoki dismissed emas.

## 2. To‘qqiz canonical issue

| Canonical | Birlashtirilgan original findinglar | Corrected contract | Acceptance / review owners |
|---|---|---|---|
| C01 Raw meter vs resolved assignment | PLAT-R1, BF-R2, 03-R2, 04-R01 | Immutable measured source pending saqlanadi; optional assignment hint; Billing accepted effective historical snapshot before charge; no guessed/retro Wallet | F-C01A…C; 01/02/03/04 |
| C02 PITR / logical restore | PLAT-R2 | Archive retention alohida; isolated restore, current authority va external effects owner reconciliation, explicit resume manifest; unknown blocked | F-C02A…C; 01+02+03 (05 coordination) |
| C03 Economic namespace | BF-R1 | Full `{issuer_namespace,occurrence_type,id}` across guard/effect/correction; event/schema/price/policy version namespace emas | F-C03A/B; 01+02+03 |
| C04 Precise negative-policy enable gate | BF-R3 | G-MR02 required before live Seat/usage debit; expiry enforcement separately approved; no zero-balance hard-stop/forecast cap | F-C04A; 02+01+04 |
| C05 External entity vs observation | 03-R1 | Stable scoped K incl. connection/server/filial; revision/hash only observation key; canonical map stable/conflict quarantine | F-C05A…C; 03+01+02 |
| C06 Completed command replay | 03-R3 | Fresh result authorization→canonical prior receipt/body→old outcome; only NEW intent expected revision/state/fence | F-C06A…C; 01+03+02+04 |
| C07 WAITING_DEPENDENCY liveness | 03-R4 | Durable resumable pending≠APPLIED slot; event/sweep wakeup/lease; atomic deferred handoff ACK≠effect, invalid-original quarantine distinct | F-C07A…C; 01+03+02 |
| C08 Atomic local offline outbox | 04-R02 | User-scoped encrypted intent/body/key/outbox one durable transaction; UI after commit; retry immutable/new linked correction | F-C08A…C; 04+01+03 |
| C09 Company/Branch switch | 04-R03 | Unsaved original-scope decision→clear/lock→new authorization→bind/load; generation fencing, explicit branch-set, original receipts/outbox retained | F-C09A…D; 01+04+02/03 consumers |

C01ning 4 original finding’i bitta canonical fixga birlashdi; qolgan 8 finding 8 issue. **12 original → 9 canonical; duplicate traceability yo‘qolmagan.**

## 3. Har original commentning corrected file/line traceability’si

Line locators ushbu correction revision uchun; aniq GitHub head SHA bilan tekshiriladi. Contract/source headings va fixture IDs ham saqlanadi. Keyingi revision line siljitsa registry qayta sync qilinadi. Jadvalning har row disposition’i **DOC_CORRECTED / DELTA_REVIEW_REQUIRED**, runtime **NOT_TESTED**.

| Owner / original finding | Original comment ID / URL | Original head file:line | Canonical / corrected file:line | Yangi contract / fixture |
|---|---|---|---|---|
| 01 PLAT-R1 | [issuecomment:6123437500 / PLAT-R1](https://github.com/yakhyodev/erp-saas-architecture/pull/2#issuecomment-6123437500) | 02_INTEGRATION_CONTRACTS.md:17,147 | C01 — [02_INTEGRATION_CONTRACTS.md](02_INTEGRATION_CONTRACTS.md) L17 | Raw measured fact vs Billing snapshot; F-C01A…C |
| 01 PLAT-R2 | [issuecomment:6123437500 / PLAT-R2](https://github.com/yakhyodev/erp-saas-architecture/pull/2#issuecomment-6123437500) | 02_INTEGRATION_CONTRACTS.md:47; 03:73; 04:82 | C02 — [02_INTEGRATION_CONTRACTS.md](02_INTEGRATION_CONTRACTS.md) L65 | Quarantine/current deny/effect reconciliation/resume; F-C02A…C |
| 02 BF-R1 | [inline:4247542086](https://github.com/yakhyodev/erp-saas-architecture/pull/2#discussion_r4247542086) | 02_INTEGRATION_CONTRACTS.md:33,108 | C03 — [02_INTEGRATION_CONTRACTS.md](02_INTEGRATION_CONTRACTS.md) L27 | Full namespaced economic ref in Finance/Billing guards; F-C03A/B |
| 02 BF-R2 | [inline:4247542097](https://github.com/yakhyodev/erp-saas-architecture/pull/2#discussion_r4247542097) | 02_INTEGRATION_CONTRACTS.md:17 | C01 — [02_INTEGRATION_CONTRACTS.md](02_INTEGRATION_CONTRACTS.md) L17 | Unassigned/late measured sample persistence and lawful resolution; F-C01A…C |
| 02 BF-R3 | [inline:4247542103](https://github.com/yakhyodev/erp-saas-architecture/pull/2#discussion_r4247542103) | 05_STAGE1_START_SEQUENCE.md:38 | C04 — [05_STAGE1_START_SEQUENCE.md](05_STAGE1_START_SEQUENCE.md) L53 | Mandatory G-MR02 and separately gated expiry; F-C04A |
| 03 R1 | [inline:4247567512](https://github.com/yakhyodev/erp-saas-architecture/pull/2#discussion_r4247567512) | 02_INTEGRATION_CONTRACTS.md:31 | C05 — [02_INTEGRATION_CONTRACTS.md](02_INTEGRATION_CONTRACTS.md) L31 | Stable external entity K vs revision/hash observation; F-C05A…C |
| 03 R2 | [inline:4247567533](https://github.com/yakhyodev/erp-saas-architecture/pull/2#discussion_r4247567533) | 02_INTEGRATION_CONTRACTS.md:17 | C01 — [02_INTEGRATION_CONTRACTS.md](02_INTEGRATION_CONTRACTS.md) L17 | Raw producer→pending→resolved same ref; IG-02 now includes 03; F-C01A…C |
| 03 R3 | [inline:4247567540](https://github.com/yakhyodev/erp-saas-architecture/pull/2#discussion_r4247567540) | 02_INTEGRATION_CONTRACTS.md:41 | C06 — [02_INTEGRATION_CONTRACTS.md](02_INTEGRATION_CONTRACTS.md) L41 | Completed receipt before new-revision validation; F-C06A…C |
| 03 R4 | [inline:4247567563](https://github.com/yakhyodev/erp-saas-architecture/pull/2#discussion_r4247567563) | 02_INTEGRATION_CONTRACTS.md:44 | C07 — [02_INTEGRATION_CONTRACTS.md](02_INTEGRATION_CONTRACTS.md) L55 | Resumable pending work/semantic slot/ACK; F-C07A…C |
| 04 R01 | [inline:4247539498](https://github.com/yakhyodev/erp-saas-architecture/pull/2#discussion_r4247539498) | 02_INTEGRATION_CONTRACTS.md:17 | C01 — [02_INTEGRATION_CONTRACTS.md](02_INTEGRATION_CONTRACTS.md) L17 | Valid unassigned fact actionable pending, never fake zero usage; F-C01A…C |
| 04 R02 | [inline:4247539510](https://github.com/yakhyodev/erp-saas-architecture/pull/2#discussion_r4247539510) | 02_INTEGRATION_CONTRACTS.md:128 | C08 — [02_INTEGRATION_CONTRACTS.md](02_INTEGRATION_CONTRACTS.md) L156 | Local intent/body/key/outbox atomic, authenticated-user account; F-C08A…C |
| 04 R03 | [inline:4247539515](https://github.com/yakhyodev/erp-saas-architecture/pull/2#discussion_r4247539515) | 02_INTEGRATION_CONTRACTS.md:73 | C09 — [02_INTEGRATION_CONTRACTS.md](02_INTEGRATION_CONTRACTS.md) L96 | Company/Branch handshake/fenced results and original queue; F-C09A…D |

Cross-cutting corrected gate/fixture/dependency locations: [03 §4.1](03_PR_AND_INTEGRATION_GATES.md#ic-fixture-stage-matrix), [05 §2](05_STAGE1_START_SEQUENCE.md), [07 named fixtures](07_DELTA_ACCEPTANCE_FIXTURES.md). Source baseline ADR-INT-002, UX-002, MOBILE_ARCHITECTURE §Sync va handoff MR-02/MR-14 faqat read qilindi, authoritative owner hujjatlari patch qilinmadi.

## 4. Reviewdagi qo‘shimcha shartlar — yangi P1 emas

- 01: commit/revoke linearization va public fence, scoped relay/per-consumer leases. Common obligations IC-01 §3.1 va F-O01A/B; implementation mechanism 01niki. DB-first containment→Identity→Tenancy→Permissions→relay graph S1 jadvalga sync.
- 02: future tier accumulator atomicity, cumulative refund/chargeback/settlement capacity, close/post serializability, immutable legal/price/time snapshots. Stage 4/5 owner contract/tests gate; huquqiy yoki narx qarori bu patch bilan tasdiqlanmaydi.
- 03: later physical leg/valuation slot, cancel-vs-dispatch fence, cumulative return/reversal/count capacity. Stage 3 owner review; Stage 1 real stock emas.
- 04: localization typed params/error/unknown enum, projection completeness, protected-shell switch. Linguistic alias contract 04da, source index persistence/reindex source owner+01da; writer table o‘zgarishsiz.

## 5. Delta-review request — yuboriladigan qisqa topshiriq

Umumiy matn: **“PR #2 yangi correction HEAD SHA’ni tekshiring. Oldingi 5c9096e review’dan keyingi docs/stage0 delta, 06 original-ID registry, 02 corrected contracts, 03 IC→IG/PG matrix, 05 DB-first dependencies va 07 expected fixturesni o‘qing. Faqat own scope uchun DOC_ACCEPTED / CHANGES_REQUIRED / ACCEPTED_WITH_CONDITIONS verdictini exact SHA bilan PR review/commentga yozing; unresolved original IDs/canonical issues va current contract adoptionni ayting. Mahsulot kodi, merge, business approval yoki runtime PASS yaratmang. Specs PROPOSED, testlar NOT_TESTED; Stage 1 authorization alohida.”**

| Owner | Targeted delta |
|---|---|
| 01 | C01/C02/C03/C06/C07/C08/C09/O01; meter raw schema, PITR current authority/external resume, fencing, per-tenant leases, DB-first dependency |
| 02 | C01/C02/C03/C04/C05/C06/C07/C09; full namespace, effective attribution, G-MR02 live enable, ledger/provider ownership and disabled Stage 1 |
| 03 | C01/C02/C03/C05/C06/C07/C08/C09; stable provider map/observation, existing-receipt order, resumable pending, IG-02/04/06/07/09 |
| 04 | C01/C06/C08/C09 (+C02 consumer recovery); atomic submitted queue, user partition, scoped switch/cache/prefetch/status/Wallet fencing, S1-05 read-only exit |

Shared contractlar GitHub hujjatlari va reviewlar orqali authoritative; chatga yuborilgan request faqat shu head/source linklarini ko‘rsatadi. 05 boshqa chat verdictini oldindan yozmaydi. Delta review qaytishi bu topshiriqning tashqi follow-up gate’i; original commentsni resolve qilish owner acceptance’dan keyingi alohida action.

## 6. Qolgan blocker va tavsiya

**READY_FOR_REVIEW (documentation delta)**: 12 findingning har biriga normative fix va expected fixture bor. **MERGE_ACCEPTANCE=OPEN**: 01–04 yangi SHA verdictlari zarur; original CHANGES_REQUIRED holatlari o‘zi yopilmaydi. Independent Round 3 BLOCKED va FIN-001 provenance OPEN saqlangan; user code authorization, application repo, tariff/grace/legal/provider/pilot/restore/runtime gate’lari [04](04_STAGE0_READINESS.md)da.

Validation record GitHub correction summary’da: exact correction SHA, actual Markdown/local-anchor checks, canonical ID/key/replay invariants prose checks, 12→9 traceability coverage, 09 IC/09 IG/07 PG alignment, domain ownership unchanged and docs-only diff. Bu performed documentation checks; 07 software fixtures hamon **NOT_TESTED**. PR #2 draft/open, merge action bajarilmaydi.

Correction validation performed: **8 Markdown, 81 lokal havola, 3 lokal anchor, 12 corrected numeric file/line locator, 27 unique named fixture row; 0 dokumentatsiya issue**. IC=9, IG=9, PG=7, MR=20; source comment coverage=12 va canonical issues=9. UTF-8/title/status/fence/whitespace/link/anchor/locator/coverage read-only Python check; `git diff --check` clean. Semantic ID/replay/approval/ownership obligations docs review orqali tekshirildi, software execution orqali emas. Checks source/head context exact correction commit bilan PR summary’da saqlanadi.
