# CODEX 05 — Stage 0 readiness va blockerlar hisoboti

Sana: 2026-10-12. Source: `main@0b0b086b40fa7492aa4ca26bf3f1562f2a86ef50`; [manbalar va status lug‘ati](README.md). Bu scoped arxitektura delta review; original 14 Architecture Review o‘rnidan yangi mustaqil to‘liq audit yoki approval berilmaydi.

## 1. Stage 0 verdict

| Natija | Joriy holat | Dalil / keyingi exit |
|---|---|---|
| Stage 0 dokumentatsiya paketi | **PREPARED_FOR_REVIEW / PROPOSED** | Ownership, repository spec, IC-01…09, PR/gates va Stage 1 ketma-ketligi mavjud |
| Yangi besh-chat ownership va contract adoption | **OPEN / DELTA_REVIEW_REQUIRED** | Original head: 01/02/03 CHANGES_REQUIRED, 04 ACCEPTED_WITH_CONDITIONS; 12 P1→9 canonical docs corrected, yangi head actual adoption pending; [06 registry](06_REVIEW_FINDINGS.md) |
| Stage 1 application implementation authorization | **OPEN / NOT AUTHORIZED** | Alohida user instruction + aniq target application repository kerak |
| Stage 1 xavfsiz disabled-skeleton specification | **PROPOSED** | [Boshlash ketma-ketligi](05_STAGE1_START_SEQUENCE.md), no economic enable |
| Whole-ERP independent architecture readiness | **BLOCKED — unresolved sign-offs** | Round 3 tarixiy BLOCKED saqlangan; yangi delta review va egalar sign-off’i yo‘q |
| Production/legal/commercial readiness | **OPEN / NOT READY** | Price, law, provider, security/DR/performance/UX gate’lari qolmoqda |
| Mahsulot tests/build/runtime/API integration/deployment | **NOT_TESTED** | Bu topshiriqda application kodi, DB, to‘lov, real API, migration yoki deploy ishlatilmagan |

Hujjat tayyorligi bilan owner adoption, coding permission va production readiness to‘rtta turli daraja. Stage 0 hujjatlarini ko‘rib chiqish uchun legal tariffni uydirish kerak emas. Alohida user authorization va technical pre-code sign-off’dan keyin disabled skeleton boshlanishi mumkin; legal/provider/tariff gate’lari esa tegishli feature enable vaqtigacha ochiq qolishi mumkin.

## 2. APPROVED / selected / proposed / open / not-tested xaritasi

| Qism | APPROVED | DESIGN_SELECTED_UNDER_DELEGATION | PROPOSED / OPEN | NOT_TESTED |
|---|---|---|---|---|
| SaaS biznes | Hybrid Prepaid, Seat+Business+Resource Usage, document base+overage, 1 credit=10 UZS, progressive/enterprise, actual one-calendar-month negative, no estimated credit ceiling | Configured finalized doc triggers, marginal tiers, anniversary-clamp | Exact prices/units/thresholds/cutoffs/refund/rearm/expiry/tax OPEN | Real ledger/concurrency/provider tests |
| Company scope | BIZ-011…014: distinct Company LegalEntity/Account/Wallet, liable entity ≠ funder, user×Company Seat, one responsible shared-resource Company | Two-step accepted future responsibility mapping candidate | Role/same-person assignment policy, legal verification, meter segmentation OPEN | Wrong-wallet/Company isolation, payer acceptance |
| Stack/transactions | Stackga blanket APPROVED yo‘q | Go/Postgres/Next.js/Flutter; modular monolith; owner-local ACID+outbox; Stage A internal pilot | Owner technical sign-off, executable contracts, RLS/runtime privilege details OPEN | Pool/RLS/effect replay/restore |
| Domain writers | Biznes qoidalari writerni blanket tasdiqlamaydi | Identity→Security, Membership→Tenancy, grants→Permissions; Party shared, Catalog/Pricing distinct | Beshta development chat/directory mapping va IC fields yangi PROPOSED; adoption OPEN | Cross-owner runtime consistency |
| ERP Finance/stock | FIN-001 original label faqat tarixiy, provenance OPEN | Typed stock/GL trigger boundary; nonnegative stock va weighted-average candidate | Fiscal jurisdiction/currency/recognition/valuation/chart/close/legal debtor evidence OPEN | Concurrent stock, balanced GL, returns/close tests |
| Mobile/external | Offline device clientga financial authority berilmagan | Server reauth; encrypted scoped queue; disabled read-first adapter | Offline TTL/retention/privacy/device policy, LIVE/cutover authorization OPEN | Mobile actual device/replay; SalesDoc LIVE_VERIFIED yo‘q |
| UX/localization | Tushunarli UX maqsadi, public uz/en/ru va future alphabet hidden | UX-T01…20 delivery/acceptance yo‘nalishi | Numeric/device/pilot targets, comparative baseline OPEN | User usability, a11y, locale/search performance |

Source evidence S03/S06 repositoryda recorded provenance’ni tasdiqlaydi; eski chatlarda yangi tasdiq olindi degan da’vo yo‘q. Ushbu report official legacy ADR statusini ko‘tarmaydi yoki tushirmaydi.

## 3. Round 3 → joriy HEAD delta

Round 3 audited source `34fbb99db8c71a01dbb50ef71a6f7555a3310e91`; o‘sha snapshotda **12 BLOCKED / 2 READY_WITH_CONDITIONS / 0 READY**. Bu sonlar bugungi qayta auditi emas. Current HEADda 105 ADR fayli sanaldi; auditning 100 ADR soni historical.

| Area | Round 3dagi gap | Current-main dalil | Qolgan haqiqiy gate |
|---|---|---|---|
| AR-001 / MR-01 | Wallet cardinality/legal payer unresolved | BILL-011 va registries v0.4da scoped business approval | Legal evidence, funder authorization/refund/novation va mechanism sign-off |
| MR-03 | Seat counting va lifecycle unclear | User×Company scope APPROVED; BILL-012 detailed effective-history candidate | Price/status/suspend/cutoff/rounding/cadence/refund va economic approval |
| MR-02/04/05 | Calendar, billable trigger, discounts unresolved | BILL-013/014 selected candidate, no fee values | Renewal/expiry terms; tariffs/units/thresholds/enterprise precedence/correction |
| AR-002/008/013 | Catalog, Party, Membership writer alternatives | ADR-010/registry v0.6 single preferred writers | Owners technical sign-off, Party merge/verification/privileged guards |
| AR-005 / MR-07 | Cross-domain transaction ambiguity | Owner-local ACID+outbox selected; historical global ACID clearly non-default | Typed effect schemas, orchestration/recognition/correction sign-off, runtime crash/replay |
| AR-003 / MR-14 | Stage A/B and HA ambiguity | Stage A internal pilot selected; Stage B later gates | Target provider/region, actual RPO/RTO/SLO, backup retention, verified restore/HA |
| MR-18 | MVP scope undecided | ADR-010 core shortlist; full HR/Manufacturing/advanced BI deferred candidate | Actual pilot promise/task/domain selection acknowledgement |
| AR-004 | FIN-001 approval provenance missing | Unchanged historical label, OPEN provenance | Original dated approval link yoki fresh exact scoped reconfirmation |
| AR-011/012 | External/UX evidence missing | Docs and acceptance matrices present | Actual authorized provider/security/UX tests, no inferable pass |

Yangi scoped business approvals real documentary delta. Ular barcha AR/MR gate’larini yopmaydi. Ushbu docs paketidan **0 ta runtime pass**, **0 ta mustaqil owner sign-off**, **0 ta LIVE_VERIFIED provider natijasi** hosil bo‘ldi.

## 4. Pre-code blockerlar va exit dalili

| ID | Blocker | Phase / holat | Egasi | Exit dalili |
|---|---|---|---|---|
| S0-B01 | Application repository va implementation uchun alohida ruxsat yo‘q | **PRE-CODE / OPEN** | User + 05 | Named repo/path, allowed stage va no-live-effect scope’i bilan explicit instruction |
| S0-B02 | Yangi 01…05 ownership/contract responsibility qabul qilinmagan | **PRE-CODE / OPEN** | 05 + 01…04 | Docs PR producer/consumer reviews, entity/directory writer acceptance, adoption SHA |
| S0-B03 | Identity/Membership/Permissions, RLS runtime role/pool va domain transaction teknik sign-off’i yo‘q | **PRE-CODE TECH / OPEN** | 01 + 02/03 consumers + 05 | S07/S08/S09 candidate technical review, commit-time auth spec, negative-suite obligations; tests keyingi implementationda |
| S0-B04 | IC event payloads/effect-slot/correction/version adoption yakunlanmagan | **PRE-CODE CONTRACT / OPEN** | Source+consumer owner + 05 | IC-01…09 accepted spec, stable economic dedup guard, backward-compatibility fixtures plan |
| S0-B05 | Repository maintainer/team mapping va branch protection enforcement noma’lum | **BEFORE MULTI-CHAT CODE MERGE / OPEN** | 01 + authorized maintainer + 05 | Real owner GitHub identities, protection/required-review status; no guessed accounts |

FIN-001 provenance, tariffs, fiscal profile yoki SalesDoc credentials yo‘qligi xavfsiz disabled auth skeletonning barcha non-economic ishlarini avtomatik to‘xtatmaydi. Lekin ular tegishli enabled financial/external release uchun haqiqiy blocker bo‘lib qoladi. PRE-CODE sign-off bilan runtime passni talab qilish circular holat yaratmasin: avval approved specification va test obligations, keyin ruxsatli implementation, undan keyin executed evidence.

## 5. MR-01…20 qolgan real gate’lar

Gate tur: **E** haqiqiy external/business/legal input, **S** actual owner technical review, **T** executed test. Status har rowda OPEN/NOT_TESTED; existing approved/selected subqaror [REMAINING_GATES](../handoff/REMAINING_GATES.md)da saqlanadi.

| MR | Qolgan gate (hech biri guessed default bilan yopilmaydi) | Type / tegishli enable phase | New owner(s) |
|---|---|---|---|
| 01 | Company legal identity, funder authorization/provider settlement, refund/novation | E/S/T / real provisioning/settlement | 01+02 |
| 02 | **G-MR02/C04:** approved anniversary/clamp/timezone/cycle/recovery/rearm/expiry rights/late correction; published tariffning o‘zi live debitga yetmaydi, debit/enforcement disabled | E/S/T / any live Seat/usage debit; F-C04A | 02+01 |
| 03 | Seat price, ACTIVE/suspend/revoke exact cutoff, daily-vs-month posting, rounding/refund | E/S/T / Seat debit | 02+01+04 |
| 04 | Eligible document/resource types, units, base/overage, measurement/threshold, shared attribution | E/S/T / usage debit | 02+03+01 |
| 05 | Enterprise contract priority, tier window/values/stacking and cancellation/net price | E/S/T / rating | 02 |
| 06 | Jurisdiction, tax, functional currency/chart, fiscal invoice and GL/AR/AP recognition/close | E/S/T / real Finance posting | 02+03 |
| 07 | Typed source/stock/GL effects, owner-local schemas, reversal/reconciliation | S/T / physical/economic effect | 03+02+01+05 |
| 08 | Reservation TTL/count cutoff/valuation business choice; concurrent stock and over-return | E/S/T / physical stock/valuation | 03+02 |
| 09 | Shared Party merge/privacy/verification and immutable financial debtor | E/S/T / identity/Finance | 03+02+01 |
| 10 | Full namespaced SourceOccurrenceRef, stable ExternalEntityKey vs observations, completed receipt ordering va resumable pending/ACK; executable schema/retention adoption | S/T / C03/C05/C06/C07 implementation & release | 05+producer/consumer+01 |
| 11 | Runtime privileges, RLS/pooling/security sign-off; SEC-R2-01…13 actual evidence | S/T / any multi-tenant enabled path | 01+all domains |
| 12 | Atomic local intent/body/key/outbox commit, authenticated-user partition; allowlist, TTL/revoke/resync/retention, C06 prior receipt and C09 original scope switch | E/S/T / C08 actual local queue/replay, later than Stage 1 | 04+01+03+02 |
| 13 | GPS/media lawful purpose/consent, retention/legal hold/privacy | E/S/T / GPS/media capture/upload | 04+03+01 |
| 14 | Region/provider/budget/RPO/RTO; **C02 quarantine/resume** manifest for full/tenant logical restore: current auth/member/grant authority, stale session/context invalidation, missing post-target provider/domain effects owner reconciliation, unknown state blocked | E/S/T / F-C02A…C actual isolated PITR drill, real pilot/paid uptime; archive retention not enough | 01+02+03+05 |
| 15 | Approved measured workload/cost envelope, pool/hot tenant/month-end/sync benchmarks | E/T / measured pilot/release | 01+02+03+04 |
| 16 | UX-T01…20 actual priority/persona/customer access and usability outcomes | E/T / pilot claims | 04+domain owners+05 |
| 17 | Search projection/index freshness; C09 clear-before-new-context + generation fence across results/counts/prefetch/notifications/Wallet/back/resume/multi-tab | S/T / F-C09A…D protected shell and search/report exposure | 01+04+source domains |
| 18 | Actual pilot scope/delivery promise; basic Purchasing/reporting shortlist acknowledgement | E/S / scoped implementation promise | 05+03+02+04 |
| 19 | SalesDoc authorized environment/credentials, live compatibility, reconciliation/cutover permission | E/S/T / external reads/writes/cutover | 03+01+02 |
| 20 | Actual baseline devices/network/locales/a11y/pilot and comparison evidence | E/T / UX acceptance/superiority claim | 04+01+domain owners |

FIN-001 AR-004 approval provenance alohida `OPEN`; old statusni qayta yozish yoki boshqa tasdiq bilan yashirin yopish mumkin emas. Round 3 R3-DOC-01 root historical link hygiene alohida follow-up; old registry original owneriga tegishli, ushbu paket uni o‘zboshimchalik bilan patch qilmaydi.

## 6. Round 3 G1…G9ga current disposition

| Gate | Joriy hujjat bilan aniqlashgan qism | Remaining disposition |
|---|---|---|
| G1 Approval/provenance | BIZ-011…014 source + selected-design vocabulary | FIN provenance, legal/commercial owner sign-off OPEN |
| G2 Ownership/cardinality | Approved Company scope, ADR-010 single writer, proposed five-chat map | Writer/directory/Party/legal contract adoption OPEN |
| G3 Isolation/security | 01 guard/RLS/pool spec + PG-SEC/TEN | Technical adoption OPEN; runtime NOT_TESTED |
| G4 Billing | BILL-011…014, IC-03/05/08/09 | Monetary policy/provider/grace terms OPEN; runtime NOT_TESTED |
| G5 Stock/GL/Finance | IC-04 split facts/local commits/corrections | Valuation/recognition/schema sign-off OPEN; runtime NOT_TESTED |
| G6 API/offline/external | IC v1/UNKNOWN/version schema/replay spec | Device/compatibility/retention/provider OPEN; runtime NOT_TESTED |
| G7 Infrastructure/DR | Stage A internal pilot selected, future stage boundaries | Provider/SLO/RPO/RTO/region OPEN; restores/benchmarks NOT_TESTED |
| G8 UX/localization | UX acceptance spec va 04 owner; source conditional spec saqlangan | User priorities/baselines OPEN; actual UX/a11y NOT_TESTED |
| G9 Governance/traceability | Paket, source SHA, PR/gate protocol | Docs adoption, branch protection, signed evidence OPEN |

Bu jadval G1…G9ni yangi READY deb belgilamaydi. Keyingi mustaqil architecture delta audit accepted docs/ADRs va actual sign-off linksni tekshirib alohida verdict beradi; undan keyin ham coding permission alohida.

## 7. Dastlabki head’da tekshirilgan narsalar — historical

- Remote `refs/heads/main` SHA olingan, shu `main` lokal checkoutida talab qilingan handoff/registry va muhim ADRlar tahlil qilingan.
- 105 ADR fayli directory inventory bo‘yicha sanalgan; barcha ADR bo‘yicha exhaustive recertification da’vosi yo‘q.
- Yangi fayllar faqat `docs/stage0/*.md`; manba registries va tarixiy auditlar o‘zgartirilmagan.
- Lokal Markdown havolalari, IC/PG/IG coverage, writer/directory mapping va docs-only diff tekshirildi. Bu dokumentatsiya QA, application runtime testi emas.
- Boshqa chat reviews, GitHub merge, application build/test, real PostgreSQL tenant isolation, finance/payment, SalesDoc API, restore yoki deployment bu topshiriqda bajarilmagan.

| Dokumentatsiya tekshiruvi | Bajarilgan natija |
|---|---|
| UTF-8 Markdown title/status va fence checks | Original `5c9096e` revision: 6 hujjat, 0 issue |
| Lokal relative links mavjudligi | 51 havola, 0 broken link |
| Kontrakt va gate coverage | IC-01…09 = 9; IG-01…09 = 9; PG = 7; MR-01…20 = 20 |
| Existing tracked source o‘zgarishi | None; yangi fayllar faqat `docs/stage0/` |
| Remote HEAD qayta tekshiruvi | `0b0b086b40fa7492aa4ca26bf3f1562f2a86ef50`, baseline bilan bir xil |

Bu natijalar lokal Python read-only Markdown validator va Git status/diff/ls-remote inspectiondan olingan. Semantic integration acceptance IG testlari hali software’da bajarilmagan. Draft PR review/merge va independent owner sign-off alohida pending.

## 8. P1 correction readiness

[06 finding registry](06_REVIEW_FINDINGS.md): 12 original P1, 9 canonical corrections; har biri `DOC_CORRECTED / DELTA_REVIEW_REQUIRED`, acceptance fixtures `NOT_TESTED`. 01/02/03/04 initial reviews’ recorded ownership direction saqlanadi, yangi head technical adoption hali OPEN. `READY_FOR_REVIEW` tavsiyasi corrected documentationga tegishli, merge readiness yoki software PASS emas.

Qolgan real blockerlar: (a) 01–04 new-head technical delta reviews va accepted schemas/fence/restore details; (b) target application repo + separate named Stage 1 authorization + repository review enforcement; (c) enabled economics uchun G-MR02, exact rates/legal/accounting/payer/provider approvals; (d) real security/transaction/stock/finance/UX/performance/offline/restore evidence. Technical specsni user tariff/huquqiy shart deb yoki documented fixture’ni executed test deb talqin qilinmaydi.

Corrected dependency: containment S1-03 enabled authority S1-02A/B/Cdan oldin; S1-05 protected read-only shells C09 adoption/evidencega tayanadi; S1-04/06 resumable server pending/replay synthetic fixtures; C08 actual local business queue va real Stage A restore/key management keyingi authorized phases. P1lar source owner transferini talab qilmagan: [01 writer matrix](01_OWNERSHIP_AND_REPOSITORY.md) o‘zgarishsiz.

**Stage 0 xulosasi:** Correction package delta review uchun tayyor; PR merge qilinmagan. Original findingsni reviewers yopmagan, software tests NOT_TESTED. Round 3 BLOCKED va production ready NO; APPROVED_BY_USER/selected design/PROPOSED/OPEN statuslari saqlangan.
