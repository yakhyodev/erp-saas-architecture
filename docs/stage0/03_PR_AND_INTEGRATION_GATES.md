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
| PG-SEC Security | Har entrypointda fresh server guard, receipt-result authorization, commit/revoke fence; restore resume faqat current authority bilan | Revoke-first/commit-first F-O01A, restored revoked credential/new login F-C02A, denied replay F-C06C, context switch revoke F-C09C; SEC-R2-03/10/11 | 01 + domain |
| PG-TEN Tenant isolation | Composite tenant FK, explicit Company/branch-set, pool/RLS, scoped restore/relay/cache | Pool reuse/privileged negative cases; F-C09A…D same-Tenant stale views, F-C08C cross-user queue partition, F-C05A filial map, F-O01B scoped relay leases; SEC-R2-01/02/04…09/13 | 01 + 05 + source owner |
| PG-TXN Transaction/effect integrity | Server and local intent/outbox atomic; full namespaced effect guard; PENDING vs APPLIED, durable handoff; no global ACID | F-C03A/B namespace and replay; F-C06A/B terminal lookup; F-C07A…C resumable dependency/ACK; F-C08A/B local commit; F-C02B/C restore evidence | Source+consumer owners; 01 harness; 05 IC cross-check |
| PG-PERF Performance/resource | Measured workload/budget; bounded queues and resumable work, backpressure | Hot SKU/tenant/month-end/sync/pool metrics; pending wakeup/sweep, lease contention and commit-fence load F-C07/F-O01; numeric liveness/retry/latency budgets OPEN | 01 + changed domain; 04 UX flow |
| PG-UX UX/localization/a11y | UX-T01…20/E/L/X/O/UNKNOWN; clear-before-verify Company/Branch switch, pending usage != 0 coverage, saved only after local commit | F-C01A pending meter, F-C06A/C result/no leak, F-C08A/B no false saved, F-C09A…D delayed/denied/back/resume/multi-tab; locale/a11y obligations | 04 + business oracle owner + 01 security |
| PG-FIN Finance correctness | Exact money, separate ledgers, full SourceOccurrenceRef, historical attribution, G-MR02 prerequisite | Existing debit=credit/Seat/return/concurrency checks; F-C01B/C no guessed payer, F-C03A/B no collision/re-rate, F-C04A MR-02 OPEN no debit/enforcement; F-C02B effect evidence; no forecast cap | 02 + 03 stock/valuation + 01 isolation |
| PG-REG Regression/compatibility/recovery | Oldest supported API/events, stable namespaces and separate observation mapping, restore manifest and flags | F-C03B upcast, F-C05B/C revised mapping, F-C06A receipt aliases, F-C07A…C resume, F-C02A…C quarantine/resume, F-C04A disabled paths | Lead writer + consumers + 05 |

Har PR **barcha yetti gate’ni disposition qiladi**. Docs-only PRda software tests `NOT_TESTED` bo‘lib qoladi, doc-link/contract consistency tekshiruvi alohida qayd etiladi; bu docs PR merge’ini to‘smaydi, product readiness ham bermaydi. Gate’dagi N/A masalan front-end style PRning GL postingiga ta’siri yo‘qligi bilan asoslanadi; tenant/UX/security ta’siri bo‘lsa uni N/A deyish mumkin emas.

Auth/stock/money change’da tegishli security/tenant/transaction/finance gate FAIL yoki NOT_TESTED bo‘lsa feature enabled holatda merge/release qilinmaydi. Explicit fail-closed disabled skeleton PR affected acceptance spec va scope sign-off bilan merge bo‘lishi mumkin; qolgan release gate BLOCKED qayd etiladi. No fabricated tests yoki “docs pass → product pass”.

Sonli SLO, retry budget, UX task-time, RPO/RTO, retention va tariff bu paketdan uydirilmaydi. Approved pilot workload/budget yetishmasa quantitative gate `BLOCKED`/`NOT_TESTED`; “tez” degan assertion qo‘yilmaydi. Correctness oracles (masalan foreign write = 0, one economic effect) test exit sharti, kuzatilgan natija emas.

## 4. Integration acceptance scenario katalogi

| Test contract ID | Minimal scenario | Expected oracle | Dependents |
|---|---|---|---|
| IG-01 / IC-01 | Fresh authority/commit fence va same-Tenant Company/Branch switch; F-C09A…D/F-O01A | Revoke-first deny, commit-first finish-before-revoke; clear-before-authorize, delayed generations no render, no outbox retarget | 01+02+03+04 |
| IG-02 / IC-02,08 | Company provision/shared assignment; F-C01A…C unassigned/late/resolved source | Distinct wallets/legal refs; retain meter pending no debit; effective historical assignment only; authorized same sample resolution once; no pooling | 01+02+03+04 |
| IG-03 / IC-03 | Seat duplicate/branch/device/history/price-version; F-C03B/F-C04A | One subject/eligible slot; full canonical source ref replay; no day/month double debit; MR-02 OPEN live debit OFF, no hidden zero-balance stop | 01+02+04 |
| IG-04 / IC-04 | Typed stock/GL flow, concurrent SKU/return; F-C07A…C prerequisite order/valuation/crash | No confirm/invoice second stock effect; pending consumes no APPLIED slot; ready prerequisite resumes same work to one effect or actionable exception | 01+02+03+04 |
| IG-05 / IC-05 | Logical order base/partial/import; F-C01A…C/F-C03A/B/F-C04A | Valid pending raw usage retained, no guessed amount/Wallet, distinct source namespaces separate effects, re-envelope same effect; G-MR02 mandatory | 01+02+03+04 |
| IG-06 / IC-06 | Offline ACK/status/revoke; F-C06A…C/F-C08A…C/F-C09D | Existing rev7 receipt before revision recheck; new stale/changed body reject; local intent+outbox both/neither, cross-user deny; aborted client not server rollback | 01+02+03+04 |
| IG-07 / IC-07 | Debtor/Party merge; F-C05A…C external connection/server/filial+revision observations | Historical debtor unchanged; F1/F2 same ID scoped maps; revised same entity same canonical ID; conflicting map quarantine | 01+02+03+04 |
| IG-08 / IC-09 | Provider pending→settled; different webhook IDs same payment; crash/chargeback | Pending no credit; settled once target Wallet; linked correction not double credit | 01+02+04 |
| IG-09 / all | F-C02A…C PITR/logical restore, F-C03/05/06/07/08 replay+pending+local crash, F-O01B leases | Archive guard retention alohida; restore current auth/effect evidence bo‘lmaguncha traffic/replay OFF; pending is resumable; no duplicate stock/GL/Wallet/external effect | 01+02+03+04+05 |

Fixture’da synthetic Tenant A/B, Company A1/A2, branches, canonical same-user, foreign IDs, revoked member/device, pending provider va source revisions bo‘ladi. Synthetic price/quantity faqat test config; customer tariff yoki production policy deb belgilanmaydi. Finance enabled-path testsga haqiqiy jurisdiction contractdan alohida deterministic fixture tax-policy profile kerak; bu compliance approval emas.

<a id="ic-fixture-stage-matrix"></a>

### 4.1 Corrected IC → fixture → PG → Stage 1 matrix

Named fixtures: [07_DELTA_ACCEPTANCE_FIXTURES](07_DELTA_ACCEPTANCE_FIXTURES.md), finding/comment traceability: [06_REVIEW_FINDINGS](06_REVIEW_FINDINGS.md). Bular **expected acceptance specifications, NOT_TESTED**. To‘qqiz IG group va yettita PG gate saqlangan; yangi fixture’lar ularga subcase sifatida bog‘langan.

| IC | Delta contract / fixture family | IG | PG | Stage 1 predecessor / keyingi enable |
|---|---|---|---|---|
| IC-01 | C06 receipt auth, C09 switch; C02 restored authority; O01 fence/relay | 01,06,09 | SEC,TEN,TXN,UX,REG | S1-03 containment → S1-02A/B/C authority → S1-05 client; restore real drill separately |
| IC-02 | C01 responsibility vs org; C02 resume guards | 02,09 | SEC,TEN,TXN,FIN,REG | S1-02B/C verified refs → S1-06 synthetic provision consumers; real legal evidence later |
| IC-03 | C03 namespaced semantic slot, C04 G-MR02 | 03,05,09 | SEC,TEN,TXN,FIN,REG | S1-06 interval/effect fixtures only; real priced debit Stage 4 |
| IC-04 | C03 source/effect namespace, C06 replay, C07 dependency liveness, C02 effects | 04,06,09 | SEC,TEN,TXN,PERF,UX,FIN,REG | S1-03/04 + S1-06 synthetic stock/GL boundaries; real effects Stage 3/5 |
| IC-05 | C01 raw/pending snapshot, C03 effect key, C04 price+MR-02 | 02,05,09 | SEC,TEN,TXN,PERF,UX,FIN,REG | S1-06 non-economic measured facts; live debit policy Stage 4 |
| IC-06 | C06 receipt lookup, C07 deferred effect, C08 local atomic save, C09 original scope | 01,06,09 | SEC,TEN,TXN,UX,REG | S1-04/06 server fixtures; S1-05 read-only consumer; actual local queue later |
| IC-07 | C05 stable entity vs observation, C03 normalized occurrence, C02 recovery | 07,09 | SEC,TEN,TXN,FIN,REG | S1-06 synthetic external mapping, no provider call; later authorized live/cutover |
| IC-08 | C01 raw capture/accepted attribution separation + C09 pending Wallet scope | 02,05,09 | SEC,TEN,TXN,UX,FIN,REG | S1-06 source/consumer schemas; actual assignment roles/interval/price gate later |
| IC-09 | C02 external receipt recovery, C03 full provider economic ref, C09 no relabeled Wallet | 08,09,01 | SEC,TEN,TXN,UX,FIN,REG | Stage 1 status fixtures only; actual verified provider settlement later |

Performance adoption remains measured-workload/budget **OPEN** and **NOT_TESTED**; C07 backlog/lease and O01 fence contention are PG-PERF obligations, no numeric target or achieved result. Completed-command/PENDING/restore/spec QA bir-birining runtime isboti emas.

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
