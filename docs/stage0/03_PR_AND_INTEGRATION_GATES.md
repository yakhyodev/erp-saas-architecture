# CODEX 05 — yagona branch, PR va integration gate protokoli

Status: `PROPOSED`. Hozir documentation-only. Quyidagi product-code/CI/migration talablari faqat keyingi alohida ruxsatli implementation bosqichiga tegishli. Source: [README S01/S07–S12](README.md).

## 1. GitHub umumiy xotira vazifasini bajaradi

Chatga berilgan prompt yoki uning xotirasi contract source emas. Har chat ish boshida tegishli repository `main` eng yangi SHA’sini oladi, accepted contract/ADR/gate linklarini o‘qiydi va PRga baseline SHA yozadi. Architecture va application repo alohida bo‘lsa, **ikkala SHA** qayd etiladi. Latest HEADdan o‘qish bilan reproducible review uchun aniq SHA freeze qilish bir-birini to‘ldiradi.

Handoff faqat GitHub issue, immutable source/contract link, PR diff, recorded reviews va merge SHA orqali. Boshqa chatlar avtomatik xabardor deb qabul qilinmaydi; keyingi chat uchun README/issue’da aniq “read these merged contracts first” blok bo‘ladi. Boshqa chatga messaging yoki yashirin memory transfer bu protokolning sharti emas.

## 2. Branch va PR protokoli

| Qadam | Qoida | Dalil / mas’ul |
|---|---|---|
| 1 — scope | Issue’da workstream, domain writer, touched directories, source ADR/IC, stage authorization va expected invariant yoziladi | Lead chat, 05 review |
| 2 — latest main | Read/fetch eng yangi `main`; local dirty user filesga tegmaslik; PR base SHA va architecture contract SHA capture | Lead chat |
| 3 — branch | `codex/01-platform/<issue>-<slug>`, `codex/02-billing-finance/...`, `codex/03-commerce/...`, `codex/04-clients/...`, `codex/05-architecture/...` | Bir PR — bitta aniq owner/use-case; `main`ga direct push yo‘q |
| 4 — contract-first | Shared contract o‘zgarsa avval schema/spec PR va producer+consumer review; business APPROVED rule o‘zgarsa user provenance | 05 coordination, producer writer |
| 5 — implementation | Faqat o‘z directory/aggregate; dependencies uchun merged public contract yoki aniq blocked dependency | Domain chat; boshqa owner source’ini “tez fix” qilish yo‘q |
| 6 — validation | Targeted tests va ta’sirlangan suite; gate status/artifacts yoziladi; test runner SHA candidate head bilan mos | Lead owner + impacted consumer |
| 7 — reviews | Source owner va affected consumers recorded review; sensitive auth/RLSda 01, money/GLda 02, shared contractda 05 | Real review URL, approver identity va reviewed SHA |
| 8 — merge | Latest main bilan compatibility; required checks candidate merge/head SHA’da; CODEX 05 unresolved dependenciesni tekshiradi | Authorized maintainer merges; auto-merge authority taxmin qilinmaydi |
| 9 — handoff | Merge SHA, effective contract version, disabled features, remaining gates, next dependent PR | GitHub issue/doc update, consumer keyingi run’da o‘qiydi |

Bu branchlar hozir yaratilmagan. GitHub branch protection/CODEOWNERS/required review konfiguratsiyasi **OPEN**; qoidani hujjatlashtirish remote enforcement borligini bildirmaydi. Application repo/teams aniqlangach maintainer uni yoqadi va negative merge attempt dalili bilan tekshiradi.

### Multi-owner PR va shared file tartibi

1. Default bir owner PR; producer contract, consumer implementation, platform wiring alohida dependency PRlar.
2. Agar atomik source change zarur bo‘lsa, PRda har touched directory uchun writer roziligi va exact diff scope’i **oldindan recorded**. Boshqa chatning faylini yakka lead o‘z xohishi bilan o‘zgartirmaydi; kerakli owner o‘z branch/commitini beradi.
3. Conflict kelib chiqsa, conflicting domain owner semantikani tanlaydi; 05 dependencyni koordinatsiya qiladi. “Ours/theirs” bilan contract yoki ledger invariant overwrite qilinmaydi.
4. Composition/CI/root lockfile/migration manifest change’ni 01 qiladi; 02/03/04 o‘z requested port yoki dependency patchini beradi. Common envelope spec’ni 05, producer schema’ni producer qiladi.
5. Approval yangi commit bilan touched invariant o‘zgarsa qayta olinadi. Eski review va test results yangi candidate SHAga avtomatik ko‘chmaydi; unaffected evidence reuse’i aniq asos bilan ko‘rsatiladi.

### Compatibility release tartibi

- Producer schema/spec→consumer tolerant reader+fixtures→producer optional emission→contract integration gate→feature enable, keyin future contract removal. Breaking version parallel v2, migration window va offline unsupported-client recovery bilan.
- Database evolve: expand→compatible application/consumer→bounded backfill→verify→contract. Domain owner migration, 01 ordering/isolation review; drop/retype uchun old clients/workers/backups support gate.
- Dedupe/economic occurrence registryni reset qilish compatibility strategiyasi emas. Application rollback old eventni bilmasa consumer safe pending qiladi; posted money/stock business correction bilan tiklanadi, silent DELETE bilan emas.

## 3. Har PR uchun yetti acceptance gate

Gate statuses: `PASS`, `FAIL`, `BLOCKED`, `NOT_TESTED`, yoki aniq asoslangan `NOT_APPLICABLE`. Bu runtime test verdictlari; decision statuses `APPROVED/DESIGN_SELECTED/PROPOSED/OPEN` bilan aralashtirilmaydi. Hozir barcha mahsulot testlari `NOT_TESTED`.

| Gate | Acceptance va ta’sir scope’i | Required adversarial oracle / evidence | Review owner |
|---|---|---|---|
| PG-SEC Security | Har API/query/command/job/sync/file/exportda server identity+fresh membership+role/object guard; no client privilege | Revoked user/device, forged scope, stale grants, self-escalation, commit-time revoke; SEC-R2-03/10/11 | 01 + domain |
| PG-TEN Tenant isolation | Composite tenant FK, branch/company bounds; fail-closed pool/RLS/runtime roles; cache/search/metadata/storage scope | A→B same pooled connection after commit/rollback/timeout/prepared reuse; guessed IDs, counts/facets/export leak, runtime BYPASSRLS attempt; SEC-R2-01/02/04…09/13 | 01 + 05 + source owner |
| PG-TXN Transaction/effect integrity | Local state+idempotency/inbox+outbox atomic; per-effect uniqueness; no global ACID claim | Crash before/after commit/publish/consumer ACK; new event ID same occurrence; same-key changed payload; ordering/partition/restore replay | Source+consumer owners; 01 harness; 05 IC cross-check |
| PG-PERF Performance/resource | Realistic pilot workload envelope va approved budgetga taqqoslash; bounded queries/queues/payload, backpressure | Hot SKU/tenant, billing month-end, offline burst, pool saturation, slow provider; p50/p95/p99/throughput/query-plan/resources old/new at same dataset/env | 01 + changed domain; 04 UX flow |
| PG-UX UX/localization/a11y | Affected UX-T01…20; E/L/X/O va UNKNOWN/PENDING/ACCEPTED/POSTED distinction; uz/en/ru, keyboard/screen-reader/context switch | No false-success payment/order; foreign scope stale cache yo‘q; error safe next action/preserved input; locale IDs/amounts mutate bo‘lmasin | 04 + business oracle owner + 01 security |
| PG-FIN Finance correctness | Exact money/credits/UoM, independent SaaS and ERP ledgers; source lineage, correct debtor/wallet, no unpriced debit | debit=credit, balance rebuild, concurrent funding/debit, Seat branch/device dedup, partial/return/closed-period/price-version replay; 1 credit=10 UZS; no forecast credit ceiling | 02 + 03 stock/valuation + 01 isolation |
| PG-REG Regression/compatibility/recovery | Affected suites va producer/consumer fixtures; oldest supported clients/events; migration/restore path and feature flags | Old/new contract fixtures, immutable semantic IDs across upgrades/rollback, reconcile projections, disabled money/outbound/offline paths actually deny | Lead writer + consumers + 05 |

Har PR **barcha yetti gate’ni disposition qiladi**. Docs-only PRda software tests `NOT_TESTED` bo‘lib qoladi, doc-link/contract consistency tekshiruvi alohida qayd etiladi; bu docs PR merge’ini to‘smaydi, product readiness ham bermaydi. Gate’dagi N/A masalan front-end style PRning GL postingiga ta’siri yo‘qligi bilan asoslanadi; tenant/UX/security ta’siri bo‘lsa uni N/A deyish mumkin emas.

Auth/stock/money change’da tegishli security/tenant/transaction/finance gate FAIL yoki NOT_TESTED bo‘lsa feature enabled holatda merge/release qilinmaydi. Explicit fail-closed disabled skeleton PR affected acceptance spec va scope sign-off bilan merge bo‘lishi mumkin; qolgan release gate BLOCKED qayd etiladi. No fabricated tests yoki “docs pass → product pass”.

Sonli SLO, retry budget, UX task-time, RPO/RTO, retention va tariff bu paketdan uydirilmaydi. Approved pilot workload/budget yetishmasa quantitative gate `BLOCKED`/`NOT_TESTED`; “tez” degan assertion qo‘yilmaydi. Correctness oracles (masalan foreign write = 0, one economic effect) test exit sharti, kuzatilgan natija emas.

## 4. Integration acceptance scenario katalogi

| Test contract ID | Minimal scenario | Expected oracle | Dependents |
|---|---|---|---|
| IG-01 / IC-01 | Request accepted-boundary vs revoked role/membership/device; tenant switch in-flight | New unauthorized intent denied; existing committed fact audited service processingga yo‘qotilmaydi; stale scope leakage yo‘q | 01+02+03+04 |
| IG-02 / IC-02,08 | Company A/B provision + branch duplicates + shared payer future reassignment | Distinct legal/account/wallet; one responsible Company; original liability immutable; no pooling | 01+02+04 |
| IG-03 / IC-03 | Duplicate membership events new event IDs; multi-branch/device; two Companies; price version change | One economic Seat per user×Company/eligible slot; no day/month double debit; rerating only correction | 01+02+04 |
| IG-04 / IC-04 | Confirm→reserve→partial dispatch→invoice, worker crash/retry, concurrent SKU, bounded return | Confirm no on-hand decrease; each physical leg once; invoice no second debit; balanced GL only allowed policy | 02+03+04 |
| IG-05 / IC-05 | Same logical order re-confirm/amend + partial callbacks + unpriced source + historical import | One configured original base; no guessed tariff; no imported historical charge by default | 02+03 |
| IG-06 / IC-06 | Offline ACK loss, reconnect revoke, wrong branch, stale price/version, closed-month late sync | Same operation retries one source; UNKNOWN no false final; reject unauthorized; signed period adjustment only | 01+02+03+04 |
| IG-07 / IC-07 | Customer/outlet differs debtor; Party merge after invoice; two external namespaces/revisions | Correct debtor snapshot unchanged; mapped single canonical identity; suspense until verified allocation | 02+03+04 |
| IG-08 / IC-09 | Provider pending→settled; different webhook IDs same payment; crash/chargeback | Pending no credit; settled once target Wallet; linked correction not double credit | 01+02+04 |
| IG-09 / all | Upcast/re-envelope, DLQ repair, archive/PITR restore, outbox publish ACK loss | Stable semantic guard across schema/partition/restore; source→effects reconciliation; no duplicate stock/GL/Wallet | All + 05 |

Fixture’da synthetic Tenant A/B, Company A1/A2, branches, canonical same-user, foreign IDs, revoked member/device, pending provider va source revisions bo‘ladi. Synthetic price/quantity faqat test config; customer tariff yoki production policy deb belgilanmaydi. Finance enabled-path testsga haqiqiy jurisdiction contractdan alohida deterministic fixture tax-policy profile kerak; bu compliance approval emas.

## 5. Evidence record va sign-off

Har executed gate: test/IC ID, tested candidate/merge SHA, architecture contract SHA/version, date, runner/environment/runtime/DB roles/pool mode, fixture version, exact command, actual pass/fail/skipped sonlari, observed invariant, artifact URL, limitations va reviewer. Secret/PII/mutlaq production payload logga kiritilmaydi.

Review roles actual repository team/accountga xaritalanishi `OPEN`. 05ning own-doc check’i mustaqil architecture auditor sign-off’i emas. Finance legal/commercial qaror customer/user/accounting authority daliliga; security sign-off actual owner reviewga; test pass executed artifactga tayanadi.

## 6. Copyable PR description shabloni

Quyidagini har chat PR body sifatida to‘ldiradi. Bo‘sh check yoki “assumed passed” qoldirilmaydi:

```markdown
## Muammo va natija
Trigger, oldingi/kutilgan xulq va owner invariant:

## Scope va authority
Workstream/domain writer:
Issue va explicit implementation authorization:
Application main/base SHA:
Architecture main/contract SHA, ADR/IC:
Touched directories va cross-owner reviews:

## Kontrakt va consistency
Producer/consumer/version compatibility:
Canonical occurrence + effect-slot + command/event dedup:
Tenant/Company/Branch va debtor/Wallet provenance:
Local transaction, pending effects, correction/recovery:
Policy/feature flags va enabled/disabled yo‘llar:

## Gate dalili
PG-SEC:
PG-TEN:
PG-TXN:
PG-PERF:
PG-UX:
PG-FIN:
PG-REG:
Har birida status, exact commands, tested SHA, result/artifact yoki N/A sababi.

## Rollout, rollback va remaining gates
Expand/contract va old client/worker support:
Non-destructive rollback/correction/reconciliation:
OPEN/NOT_TESTED blocker, egasi va follow-up issue:

## Recorded reviews va handoff
Source owner review URL/SHA:
Affected consumer review URL/SHA:
Security/Finance/Architecture review (applicable):
Merge SHA va keyingi chat o‘qishi kerak bo‘lgan docs:
```

## 7. Integration gate review ketma-ketligi

Contract spec acceptance→producer/consumer scoped PRlar→real test artifacts→05 cross-contract delta review→authorized maintainer merge→staged flag activation. Flag activate economics/legal/outbound/restricted offline effectni yoqsa o‘sha named gate va authority alohida yopiladi. Merge yoki elapsed time unapproved money pathni avtomatik yoqmaydi.
