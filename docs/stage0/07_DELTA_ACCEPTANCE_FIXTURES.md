# CODEX 05 — corrected contract acceptance fixture katalogi

Status: `PROPOSED / TEST_DEFINED / NOT_TESTED`. Bu faqat Markdown acceptance oracle; product tests, migration, provider call yoki restore drill yaratilmagan/bajarilmagan. Source review/head va original 12 P1 traceability [06 registry](06_REVIEW_FINDINGS.md)da. Contractlar [02](02_INTEGRATION_CONTRACTS.md), parent IG/PG va Stage dependencies [03 §4.1](03_PR_AND_INTEGRATION_GATES.md#ic-fixture-stage-matrix)da.

## Umumiy fixture sharti

Synthetic Tenant T-A/T-B; T-Ada Companies C-A1/C-A2, branches B-A1/B-A2; canonical same user U; revoked user/device; same raw external/source ID `42`; original command expected revision `7`, successful revision `8`; transient crashes/barrier-controlled races. Price/policy/quantity faqat synthetic test profile. Provider simulated receipt va real provider evidence alohida; mock hech qachon `LIVE_VERIFIED` emas. “0 leak/1 effect/eventual APPLIED” expected exit oracle, **observed PASS emas**.

Har row actual execution oldidan owner test command/environment/build SHA’si bilan bog‘lanadi. “Controlled crash” tester tomonidan keyingi ruxsatli fixture’da fault injection degani; hozir recovery amali bajarilmaydi. Ekonomik effects uchun Stage 1 synthetic counter/receipt ishlatadi, real stock/Wallet/journal emas.

## C01 — raw measurement / Billing attribution

| Fixture ID | Setup va trigger | Required oracle | Owners / IG / PG |
|---|---|---|---|
| F-C01A | Mavjud shared resource valid actual meter, assignment missing/ambiguous | One immutable source sample retained; Billing PENDING_ATTRIBUTION with safe scoped pending reference/count; no Wallet selection/amount/debit, usage coverage “zero” emas | 01+02+03+04; IG-02/05/09; TEN/TXN/UX/FIN/REG |
| F-C01B | Accepted A→B future cutover, late pre-cutover A sample; interval crosses cutover | Authoritative historical A assignment yoki explicit pending review; no current B guess. Crossing interval owner-verified measured segmentation yoki pending, no guessed proportional charge | 01+02+03; IG-02/05/09; TXN/FIN/REG |
| F-C01C | Same unresolved measurement yangi event ID bilan; keyin lawful owner-approved resolution | Same full source ref/sample; zero effect pending; enabled approved policy bo‘lsa at most one original effect, otherwise stays actionable pending. Prospective assignment no unapproved retro-charge | 01+02+03+04; IG-02/05/09; TXN/UX/FIN/REG |

## C02 — restore quarantine / resume

| Fixture ID | Setup va trigger | Required oracle | Owners / IG / PG |
|---|---|---|---|
| F-C02A | Known session/member/grant revoked after restore target; old snapshot/full or tenant logical restore | Isolated network/credentials, normal login/workers OFF. Old access/refresh rejected; fresh login old restored grantga access bermaydi until current authority reconstructed. Single writer/context generation verified | 01; IG-01/09; SEC/TEN/REG |
| F-C02B | Source/outbox at T1; irreversible provider effect+local receipt at T2; restore T1 removes receipt | No blind replay. 02/03 owner provider idempotency/lookup evidence establishes effect, owner receipt/ledger reconciles once. Missing evidence RECOVERY_PENDING blocks affected resume; no new economic key/no second external effect | 01+02+03; IG-08/09; TXN/FIN/REG |
| F-C02C | Restore has unresolved authority/effect class; stale routing writer or tenant recovery generation | Resume manifest explicitly blocked for unresolved class/scope; stale workers cannot replay/ACK. Only accepted scoped manifest+owner evidence permits affected traffic/replay; no direct cross-owner ledger SQL | 01+02+03+05; IG-09; SEC/TEN/TXN/REG |

Synthetic restore fixtures define mechanism behavior; paid pilot **actual isolated restore drill** with real permitted recovery artifacts remains MR-14 `OPEN/NOT_TESTED` even if synthetic suite later passes.

## C03 — canonical economic namespace

| Fixture ID | Setup va trigger | Required oracle | Owners / IG / PG |
|---|---|---|---|
| F-C03A | Same Tenant/account: `{sales,order_confirmed,42}` vs `{purchasing,receipt_posted,42}` | Full issuer/type/ID ref means two legitimate semantic facts, separate synthetic effects; raw-ID false conflict/dedup yo‘q. Finance different issuer journal source ham separate | 01+02+03; IG-04/05/09; TXN/FIN/REG |
| F-C03B | Each source same ref with new event/schema/upcast/policy/producer implementation name; original-linked correction | Ordinary replay prior receipt/effect; new policy/version no fresh base/journal. Changed meaning same ref conflict. Authorized correction new namespaced correction ref, original full ref retained | 01+02+03; IG-03/05/09; TXN/FIN/REG |

## C04 — approved precise negative policy gate

| Fixture ID | Setup va trigger | Required oracle | Owners / IG / PG |
|---|---|---|---|
| F-C04A | Published tariff+technical IC review configured; G-MR02 precise commercial grace/rearm/expiry/correction still OPEN | Live Seat/usage debit OFF; expiry enforcement OFF without its policy. No silent zero-balance stop/forecast cap; disabled non-economic skeleton can continue. Synthetic simulated policy does not satisfy real customer approval | 02+01+04; IG-03/05; SEC/TXN/UX/FIN/REG |

## C05 — external identity vs observation

| Fixture ID | Setup va trigger | Required oracle | Owners / IG / PG |
|---|---|---|---|
| F-C05A | Same server ID `42`, same namespace; F1 vs F2; two connections; filial-free method | K includes connection/server/explicit filial scope; distinct source keys resolve separately. Filial-free `SOURCE_GLOBAL` explicit; omitted/guessed filial reject/pending source validation. No local-name merge | 03+01; IG-07/09; TEN/TXN/REG |
| F-C05B | Same K with source revision r1→r2 and observation hash; retry both observations | Same canonical entity mapping, per-observation import dedup. Revision does not create new master. Owner-normalized transition source ref unchanged on replay | 03+02; IG-07/09; TXN/FIN/REG |
| F-C05C | Existing K→canonical P1; new payload attempts K→P2 | Durable conflict quarantine, existing binding/history unchanged; no silent retarget. Owner-authorized identity correction explicit lineage, Finance fiscal debtor immutable | 03+02; IG-07/09; TEN/TXN/FIN/REG |

## C06 — completed command replay ordering

| Fixture ID | Setup va trigger | Required oracle | Owners / IG / PG |
|---|---|---|---|
| F-C06A | Confirm expected rev7 commits rev8, ACK lost; original immutable body/key/rev7 repeats via API/operation alias | Fresh result/object AuthZ → scoped receipt+body comparison before new state/revision checks → same prior accepted receipt/source ref, one effect. Race loser returns winner receipt | 01+03+04; IG-06/09; SEC/TXN/UX/REG |
| F-C06B | Original same key changed submitted body; separately NEW key/intent still expected rev7 | Changed existing body deterministic payload conflict/no mutation; genuinely new stale command revision conflict/no effect; no reclassification of old successful command as new stale intent | 01+03+02; IG-06/09; TXN/REG |
| F-C06C | Old receipt exists; requesting user revoked or other Company/Branch/user submits key or status query | Current result/object permission denies sensitive receipt/body/existence leak; no retarget. Service-authorized prior accepted fact remains for reconciliation; user denial does not DELETE effect | 01+02+03+04; IG-01/06/09; SEC/TEN/UX/REG |

## C07 — resumable dependency inbox

| Fixture ID | Setup va trigger | Required oracle | Owners / IG / PG |
|---|---|---|---|
| F-C07A | Valid return arrives before dispatch, or stock posting before valuation; prerequisite later verified | Durable WAITING_DEPENDENCY + unconsumed APPLIED slot → prerequisite wakeup/sweep → same semantic work one effect; invalid/forged original quarantine separately | 01+02+03; IG-04/09; TXN/PERF/FIN/REG |
| F-C07B | Crash after durable pending+deferred handoff ACK; duplicate/new envelope; lost dependency signal then restart | Deferred work receipt/lease/retry persisted atomically; ACK not APPLIED. Re-envelope resumes same pending, scheduled sweep covers lost wakeup. Retry exhaustion actionable durable operator work, repair resumable; no silent lost job | 01+02+03; IG-04/09; TXN/PERF/REG |
| F-C07C | Prerequisite ready, concurrent lease workers; crash after effect commit before ACK | One atomic effect+guard+APPLIED/outbox; stale lease cannot finalize; replay prior terminal outcome. No second effect or permanent pending after verified successful commit | 01+02+03; IG-04/09; TXN/FIN/REG |

Liveness bound/scheduler budget values owner-defined **OPEN**. Runtime fixture must record waiting age and recovery completion against selected budget, not replace availability/fair scheduling assumptions with an exactly-once delivery claim.

## C08 — local atomic intent/outbox

| Fixture ID | Setup va trigger | Required oracle | Owners / IG / PG |
|---|---|---|---|
| F-C08A | Local save of reviewed intent; crash/storage-full between intent/body/key/outbox writes | One transaction: both complete intent+immutable body/key/queue or neither; UI saved/QUEUED only after durable commit; draft separate from submitted payload | 04+01+03; IG-06/09; TXN/UX/REG |
| F-C08B | Restart with UNKNOWN or rejected submitted operation; retry/edit | Same immutable body/identity on retry; no overwritten rejected/UNKNOWN intent. User correction new operation+old link atomically; no double submit/no false server-accepted badge | 04+01+03; IG-06/09; TXN/UX/REG |
| F-C08C | User U1 logs out, U2 enters same Tenant/device; Company switch | Authenticated-user encrypted partition distinct from BillingAccount; U2 cannot open/submit U1 queue; original user/Company/Branch ownership locked/quarantined by policy, no scope migration | 04+01; IG-01/06/09; SEC/TEN/UX/REG |

Actual local business outbox implementation requires later offline policy/feature authorization; Stage 1 only contract/state fixtures and read-only shells, mutation replay OFF.

## C09 — Company/Branch switch handshake and response fencing

| Fixture ID | Setup va trigger | Required oracle | Owners / IG / PG |
|---|---|---|---|
| F-C09A | Same Tenant C-A1→C-A2 / B-A1→B-A2; delayed search/Wallet/count/history/prefetch/notification | Unsaved intent decision old scope; old view clear/lock BEFORE new server authorization; bind new context then load. Old context+generation response never paints new presentation; selected Wallet reset | 01+04+02+03; IG-01/06; SEC/TEN/UX/REG |
| F-C09B | New context denied; Back/app resume; unset/null branch or unauthorized all-branches | Empty/locked safe denial; no fallback repaint without fresh old-scope authorization; explicit branch-set only. Back/resume generation rechecked, no cached cross-scope exposure | 01+04; IG-01; SEC/TEN/UX/REG |
| F-C09C | Revoke during switch; old form in second tab; delayed subscription | Server current context/grant fence denies stale rights; client generations fence old replies/subscriptions; one tab cannot retarget another tab’s operation silently | 01+04+03; IG-01/06; SEC/TEN/TXN/UX/REG |
| F-C09D | Client aborts old-scope in-flight command, switches Company, server accepted command | Abort not rollback/non-acceptance proof. Original receipt/outbox remains original scope; current authorized original-scope status lookup shows truthful result separately, never relabeled/new Wallet | 01+04+02+03; IG-01/06/09; SEC/TEN/TXN/UX/REG |

## Reviewdagi owner implementation obligations — yangi P1 soniga qo‘shilmaydi

| Fixture ID | Contract / oracle | Phase / owner |
|---|---|---|
| F-O01A | Barrier races: revoke commits before operation fence→deny; operation fences first→finish before revoke completes; after revoke ACK no new old-rights commit. Grant inserts/scope changes participate; cross-owner public fence no private business write | Stage 1 enabled guard; 01+source domain; IG-01, PG-SEC/TEN/TXN/PERF |
| F-O01B | Scoped tenant discovery, per-tenant relay claim, multi-worker per-consumer lease generation; stale worker cannot ACK new lease, no ordinary BYPASSRLS global outbox payload scan | Stage 1 relay; 01+consumer domains; IG-09, PG-TEN/TXN/PERF/REG |

02 future billing tier accumulator, bounded cumulative refund/chargeback and period-close allocation invariants; 03 stock leg/valuation/cancel/return capacity; 04 typed errors/unknown enum/projection completeness/locale contracts reviewlarda owner conditions sifatida qoladi. Ular Stage 1 real economicsni yoqish yoki qo‘shimcha P1 resolution authority emas.
