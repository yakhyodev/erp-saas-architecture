# CODEX 05 — Stage 0 arxitektura, integratsiya va QA paketi

- Sana: 2026-10-12, Asia/Tashkent.
- Repository: `yakhyodev/erp-saas-architecture`, branch: `main`.
- O‘qilgan va tekshirilgan HEAD: [`0b0b086b40fa7492aa4ca26bf3f1562f2a86ef50`](https://github.com/yakhyodev/erp-saas-architecture/commit/0b0b086b40fa7492aa4ca26bf3f1562f2a86ef50).
- Paket holati: `PROPOSED` — implementatsiya uchun kontrakt va acceptance spetsifikatsiyasi.
- `PRODUCT_CODE_AUTHORIZED=NO`; `PRODUCTION_READY=NO`; bajarilgan mahsulot testlari: `NOT_TESTED`.
- Round 3 tarixiy verdict: `BLOCKED`; ushbu paket uni o‘zgartirmaydi.

## Natija va foydalanish tartibi

To‘rtta development workstream va beshinchi koordinatsiya chati bitta GitHub kontraktlar majmuasidan ishlashi uchun quyidagi hujjatlar tayyorlandi:

| Hujjat | Natija |
|---|---|
| [01 — Ownership va application repository](01_OWNERSHIP_AND_REPOSITORY.md) | Besh chat, entity writer/reader, directory ownership, legacy yo‘nalishlar xaritasi va tavsiya etilgan papkalar |
| [02 — Integratsiya kontraktlari](02_INTEGRATION_CONTRACTS.md) | Yetti talab qilingan chain, umumiy envelope/ID, outbox/inbox, replay, correction va recovery |
| [03 — PR va integration gate](03_PR_AND_INTEGRATION_GATES.md) | Yagona branch/PR protokoli, cross-owner review, yetti acceptance gate, PR shabloni |
| [04 — Stage 0 readiness](04_STAGE0_READINESS.md) | Statuslar, Round 3 delta, real blockerlar, MR-01…20 javobgarligi |
| [05 — Stage 1 boshlash ketma-ketligi](05_STAGE1_START_SEQUENCE.md) | Faqat alohida code authorizationdan keyin bajariladigan dependency-ordered ishlar |
| [06 — Review finding registry](06_REVIEW_FINDINGS.md) | 12 original P1→9 canonical issue, comment ID/source/new-line traceability va delta-review disposition |
| [07 — Delta acceptance fixtures](07_DELTA_ACCEPTANCE_FIXTURES.md) | Har corrected contract uchun named expected fixtures, IC/IG/PG/Stage dependency mapping; NOT_TESTED |

Hozir faqat Markdown spetsifikatsiyalari yaratilgan. Application repository, mahsulot source, SQL migration, CI konfiguratsiyasi, infratuzilma va deployment yaratilmagan. Paket [docs-only draft PR #2](https://github.com/yakhyodev/erp-saas-architecture/pull/2)da review uchun. Chatlar uchun authoritative bo‘lishi docs PR merge qilinganidan keyin; correction/review request boshqa ownerning sign-off’i emas.

## Review correction revision

01 top-level report va 02/03/04 formal COMMENT reviewlar, barcha 10 inline comment o‘qildi: reviewed head `5c9096eeef262ed8e5a5634935435dfc990f7dd4`, 12 P1 (2+3+4+3) → 9 canonical. Yangi correction head GitHub PR metadata va correction summary commentda exact SHA bilan qayd etiladi; registry o‘z commit SHA’sini o‘z ichida taxmin qilib yozmaydi. Barcha original commentlar [06](06_REVIEW_FINDINGS.md)da, fixtures [07](07_DELTA_ACCEPTANCE_FIXTURES.md)da.

05 tavsiyasi `READY_FOR_REVIEW` — corrected docs bo‘yicha **delta review uchun**, merge approval emas. 01–04 yangi head sign-off’lari `OPEN/DELTA_REVIEW_REQUIRED`; original scoped verdictlar tarixiy saqlanadi. Mahsulot tests `NOT_TESTED`, Round 3 `BLOCKED`, code permission NO. `APPROVED_BY_USER` biznes subqarori, delegated selected design, yangi `PROPOSED` correction va unresolved `OPEN` bir-biridan farqlanadi.

## Statuslarni qo‘llash

| Status | Aniq ma’no |
|---|---|
| `APPROVED` | Joriy registry/ADRda manbasi ko‘rsatilgan, aniq foydalanuvchi tasdiqlagan biznes subqarori. Butun ADRni tasdiqlamaydi. |
| `DESIGN_SELECTED_UNDER_DELEGATION` | Mavjud ADR-010 va handoff’da delegatsiya asosida afzal tanlangan dizayn; official ADR `PROPOSED` qoladi. |
| `PROPOSED` | Ushbu paketdagi yangi contract, ownership, branch yoki gate spetsifikatsiyasi; egalar review’i va qabul qaydi kerak. |
| `OPEN` | Zarur tashqi ma’lumot, provenance, policy yoki technical sign-off yo‘q. |
| `NOT_TESTED` | Runtime/security/performance/finance/UX/recovery isboti bajarilmagan. Test spetsifikatsiyasi borligi natija emas. |

Bir qarorda statuslar yonma-yon bo‘lishi mumkin: user×Company Seat scope `APPROVED`, kunlik proratsiya `DESIGN_SELECTED_UNDER_DELEGATION`, narx `OPEN`, replay testi `NOT_TESTED`. FIN-001 tarixiy `APPROVED` labeli o‘zgartirilmaydi; uning approval provenance’i `OPEN`, uni yangi tasdiq deb ishlatib bo‘lmaydi.

## Manba ustuvorligi

1. Aniq scoped user-approved BIZ qoidalari va ular haqidagi provenance.
2. `DECISIONS`, `REQUIREMENTS`, `MODULE_REGISTRY`, `OPEN_QUESTIONS` eng yangi scoped addendumlari hamda ADR-010/BILL-011…014.
3. Shu baseline bilan mos domain ADR va module addendumlari.
4. Round 1/2/3 o‘z audit SHA’sidagi tarixiy dalil; eski `OPEN` wording yangi tanlovni bekor qilmaydi, yangi tanlov ham auditni retroaktiv `READY` qilmaydi.
5. Ushbu paket yangi takliflarni aniq belgilaydi; mavjud tasdiq yoki technical sign-off’ni o‘zi yaratmaydi.

Qarama-qarshi APPROVED qoida topilsa, kodlash bilan yechilmaydi: CODEX 05 conflict qaydini, aniq ikki source va qaror egasini PRga kiritadi. Faqat keyingi aniq tasdiq/superseding ADR normative holatni o‘zgartiradi.

## O‘qilgan asosiy dalillar

Quyidagi yo‘llar yuqoridagi frozen HEADga tegishli. GitHub’da manbani ko‘rish uchun `https://github.com/yakhyodev/erp-saas-architecture/blob/0b0b086b40fa7492aa4ca26bf3f1562f2a86ef50/` prefiksidan foydalanish mumkin.

| Dalil ID | Repository fayli / bo‘lim | Paketda ishlatilgan jihat |
|---|---|---|
| S01 | [CODEX_START_HERE](../handoff/CODEX_START_HERE.md), [backlog](../handoff/MVP_IMPLEMENTATION_BACKLOG.md), [gates](../handoff/REMAINING_GATES.md) | No-code chegarasi, stage ketma-ketligi, A/D/E/S/T farqi |
| S02 | [ADR-010](../adr/ADR-010-mvp-delegated-baseline.md) §2–5 | Stack, modular monolith, single writer, owner-local ACID, Stage A, domenlar |
| S03 | [DECISIONS](../../DECISIONS.md) v0.4–0.6, [REQUIREMENTS](../../REQUIREMENTS.md) v0.4–0.6 | BIZ-001…014, yangi design markerlari, FIN-001 provenance |
| S04 | [MODULE_REGISTRY](../../MODULE_REGISTRY.md) v0.6, [OPEN_QUESTIONS](../../OPEN_QUESTIONS.md) v0.4–0.6 | Canonical writerlar va qolgan MR/AR gate’lari |
| S05 | [Round 3](../reviews/ARCHITECTURE_REVIEW_ROUND_3.md) §6–15 | Frozen audit `34fbb99…`, G1…G9, 12 BLOCKED / 2 conditional / 0 READY |
| S06 | [BILL-011](../adr/ADR-BILL-011.md), [BILL-012](../adr/ADR-BILL-012.md), [BILL-013](../adr/ADR-BILL-013.md), [BILL-014](../adr/ADR-BILL-014.md) | Account, debtor/funder, Seat, negative month, tariff eligibility |
| S07 | [DB-003](../adr/ADR-DB-003.md), [DB-004](../adr/ADR-DB-004.md), [DB-005](../adr/ADR-DB-005.md), [DB-009](../adr/ADR-DB-009.md) | Composite tenant keys, RLS/pool, local transaction, immutable ledger, expand/contract |
| S08 | [SEC-001](../adr/ADR-SEC-001.md), [SEC-002](../adr/ADR-SEC-002.md), [SEC-004](../adr/ADR-SEC-004.md), [SEC-009](../adr/ADR-SEC-009.md), [SEC-010](../adr/ADR-SEC-010.md), [Security](../modules/SECURITY_ARCHITECTURE.md) §21 | Identity/member/permission, freshness, SEC-R2-01…13 |
| S09 | [Backend](../modules/BACKEND_ARCHITECTURE.md) §2.1/18, [BE-003](../adr/ADR-BE-003.md), [BE-006](../adr/ADR-BE-006.md), [SI-015](../adr/ADR-SI-015.md) | Command/event names, canonical occurrence, dedup, independent effects |
| S10 | [FIN-001](../adr/FIN-001.md), [FIN-002](../adr/FIN-002.md), [FIN-003](../adr/FIN-003.md) | ERP GL vs SaaS ledger, balanced journals, legal debtor/Party snapshots |
| S11 | [MOB-004](../adr/ADR-MOB-004.md), [MOB-009](../adr/ADR-MOB-009.md), [Integrations](../modules/INTEGRATION_ARCHITECTURE.md) | Offline UNKNOWN/replay/revoke, external mapping and evidence taxonomy |
| S12 | [Master](../modules/MASTER_ARCHITECTURE.md), [UX](../modules/UX_ARCHITECTURE.md) §17–18, [CHANGELOG](../../CHANGELOG.md) v0.6 | Legacy master alignment, UX-T01…20, delegated delta |

Joriy daraxtda 105 ADR fayli borligi qayta sanaldi. Bu barcha 105 ADRning exhaustive qayta auditi emas; ushbu paket ko‘rsatilgan handoff, registry va integratsiyaga taalluqli ADR/bo‘limlarni tahlil qiladi. Mahsulot testlari, real to‘lov, SalesDoc tenant API yoki DBHub demo orqali ERP production dalili olinmadi.
