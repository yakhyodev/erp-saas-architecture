# UX_ARCHITECTURE.md — ERP SaaS UX Architecture

- **Status:** PROPOSED (mazkur hujjatdagi yangi UX qarorlari tasdiqlanmagan)
- **Sana:** 2026-10-11
- **Mas'ul domen:** UX/UI, navigatsiya, axborot arxitekturasi, foydalanuvchi oqimlari va frontend tajribasi
- **Aloqador hujjat:** `UX_UI_ARCHITECTURE.md` (oldingi konsept; mazkur hujjat kodsiz kengaytirilgan spetsifikatsiya)
- **Maqsad:** Sales Doctor'ga mustaqil, funksional jihatdan boy, biroq vazifalarni topish va bajarish ancha oson ERP SaaS.

> **APPROVED — mavjud platforma cheklovlari:** Hybrid Prepaid SaaS; Seat + Business Usage + Resource Usage; Business Usage uchun hujjat narxi va hajmdan keyingi qo'shimcha kredit; 1 kredit = 10 so'm; progressiv chegirmalar va enterprise shartnomalar; balans tugagach haqiqiy Seat/Usage xarajatlarini bir kalendar oy minusga yozish, taxminiy kredit limiti yo'q; kompaniya va filiallar yagona SaaS; uz/en/ru va kelajakdagi o'zbek alifbosi uchun yashirin variant. **Go/PostgreSQL/Next.js/Flutter — PROPOSED**, tanlangan texnologiyalar emas. Ushbu hujjat mavjud biznes qarorlarini o'zgartirmaydi.

## 1. UX missiyasi va mezonlar

**Asosiy prinsip:** foydalanuvchi modul tuzilishini bilmasdan biznes ishini bajaradi. Funksiyalar yashirilmaydi; tegishli jarayon, qidiruv va kontekstda ochiladi. Maxsus amallar uchun doim topiladigan alternativ kirish nuqtasi bor.

### Taklif etilgan qabul mezonlari (PROPOSED)

1. Eng ko'p ishlatiladigan 20 ta vazifaning kamida 90%ini bosh ekrandan **2–3 navigatsion harakatda topish** (jarayon ichidagi majburiy ma'lumot kiritish bosqichlari hisoblanmaydi). Bu hamma funksiyalar uchun universal kafolat emas.
2. Birinchi marta kirgan foydalanuvchi eng muhim rolga xos vazifani yo'naltirilgan onboarding bilan, tashqi qo'llanmasiz bajarishi; sinovdagi completion rate maqsadi ≥ 90%.
3. Pilot usability testlarda asosiy vazifalarda muvaffaqiyat ≥ 95%, noto'g'ri menyuga kirishlar ulushi ≤ 10%; bazaviy o'lchov Sales Doctor bilan bir xil ssenariyda olinadi, uning hozirgi real ko'rsatkichlari taxmin qilinmaydi.
4. Bir xil biznes atamalari web/mobile va uz/en/ru interfeyslarida izchil qo'llanadi.
5. KPI va kritik ogohlantirishlardan bevosita tegishli hujjat yoki yechim workflow'iga o'tish mumkin.
6. Permission tekshiruvi doimo serverda; UI yashirish xavfsizlik mexanizmi emas.

**O'lchovlar:** time-to-find, task completion, time-on-task, misclick, search zero-result, workflow abandonment, backtrack rate, p75 UI performance, accessibility audit. Telemetriyada mijozning shaxsiy/moliyaviy qiymatlarini yig'maslik va tenant chegarasini saqlash.

## 2. Domen chegarasi va bog'liqlik

**UX egaligi:** information architecture, ro'lga mos ko'rinishlar, ekranning states (loading/empty/error/success), global navigatsiya, design tokens, moslashuv, foydalanish mezonlari va ekrandagi workflow affordance.

**UX egalik qilmaydi:** RBAC siyosati, tenant isolation, qidiruv indeksining infrastruktura tanlovi, accounting posting, Sales/Inventory document state machine, pricing/billing, offline conflict resolution. UX boshqa modullarning authoritative holatini aks ettiradi.

Bog'liqliklar: Multi-Tenant (company/branch scope), Security (permissions), Backend (query/API, idempotency), Sales/Inventory (transaction workflows), Finance (invoice/payment), CRM (counterparty/visit), Billing (wallet), Mobile/Offline (sync), Localization (translations), Infrastructure (latency), Database (search/filter scalability).

## 3. Axborot arxitekturasi — Navigation

### 3.1 Global shell (PROPOSED)

- Yuqori qatlam: kompaniya/filial konteksti; universal qidiruv; xabarnomalar; yordam; profil/til.
- Chap rail: `Ish maydoni`, `Savdo`, `Ombor`, `Moliya`, `Mijozlar/CRM`, `Hisobotlar`, `Boshqaruv` (faqat ruxsat va o'rnatilgan modullar asosida). Bu nomlar taklif va tasdiqlangan modul reyestri emas.
- Rolga mos `Mening vazifalarim` va `Tezkor amallar` navigatsiyaning eng yuqori nuqtasida.
- Har sahifada tushunarli sarlavha, current scope, breadcrumbs (zaruratda), asosiy amal, izlash/filter va o'ng kontekst paneli.
- Maksimal ikki darajali ko'rinarli yon menyu maqsad qilinadi; chuqur hujjatlar menyuga ko'milmaydi, entity sahifalari va kontekstual action orqali ochiladi.
- `All functions / Funksiyalar katalogi`: vakolat bilan mavjud barcha amallarni qidiriladigan, biznes vazifalari bo'yicha indekslangan ko'rinishda chiqaradi. Progressive complexity hech qachon topish mumkin bo'lgan funksiyani yo'q qilmaydi.
- Pinned/favorites, recents va yaqinda bajarilgan vazifalar — foydalanuvchi shaxsiy sozlamalari. Tuzilma o'zgarsa, deep-linklar barqaror qoladi.

### 3.2 2–3 harakat qoidasi

`Ish maydoni → Yangi sotuv → Sotuv wizard` (2 harakat); `Global search → Hujjat natijasi` (2); `Ogohlantirish → To'lov tafsiloti` (1). Ma'lumot kiritish, validatsiya va tasdiqlashlar xavfsizlik uchun olib tashlanmaydi.

### 3.3 Filiallararo ish

Company/branch selector aniq ko'rinadi; aktyor ruxsatiga bog'langan scope ko'rsatiladi (`Faqat shu filial`, `Ruxsat etilgan filiallar`). Filial almashtirishda saqlanmagan ma'lumotlar haqida ogohlantirish; boshqa filialdagi mos kelmaydigan deep-link jim o'zgarmaydi, aniqlashtirish talab etiladi.

## 4. Role-based UI va Dashboard

**Persona emas, permission authoritative.** Quyidagilar boshlang'ich arxetiplar; haqiqiy rollar Security moduli bilan kelishiladi. Bir foydalanuvchi bir nechta rolga ega bo'lishi mumkin, UI ishchi kontekstini tanlashni ta'minlaydi.

| Arxetip | Bosh sahifadagi signal | Tezkor amal | Muhim drill-down |
|---|---|---|---|
| Direktor | Savdo, pul oqimi, qarzdorlik, risk | Hisobotni ochish | Filiallar kesimi, KPI manbasi |
| Sotuvchi | Bugungi buyurtmalar, mijozlar, reja | Yangi sotuv | Mijoz/katalog/buyurtma |
| Omborchi | Yig'ish navbati, qoldiq, qabul/topshirish | Tovar qabul qilish | Ombor harakati |
| Buxgalter | To'lovlar, muddatli qarzlar, solishtirish | To'lov kiritish | Invoice/payment/ledger |
| Sales agent | Tashriflar, marshrut, mijoz buyurtmalari | Tashrifni boshlash | Visit/customer (mobile-first) |
| Administrator | A'zolik, xizmat holati, billing xabarlari | Xodim taklif qilish | Permission, audit, wallet |

**Dashboard tuzilishi:** `Bugun bajarish kerak` → `Kritik ogohlantirishlar` → `Tezkor amallar` → `Rol KPI` → `Yaqinda ishlangan obyektlar`. KPI kartasida davr, scope, hisoblangan vaqt va manba/hisobotga havola. Aksiyalar ruxsatsiz ko'rsatilmaydi; bu serverda tekshiruvdan ozod qilmaydi.

**Billing UX:** wallet balansi va haqiqiy sarflangan kreditlar ko'rsatiladi. Nol/musbat/manfiy balansning aniq holati beriladi; minus davr bir kalendar oy ekanligi tushuntiriladi. **Taxminiy kredit limiti yoki avvaldan belgilangan taxminiy sarf limiti ko'rsatilmaydi.** Ushbu xabarlar biznes qoidalarini qayta hisoblamaydi.

### 4.1 Personalizatsiya

Yangi foydalanuvchi uchun `Sodda`, tajribali foydalanuvchi uchun `Kengaytirilgan` ko'rinish taklif etiladi. Ikkalasida ham bir xil permission va biznes amal mavjud, farq zichlik, qo'shimcha ustunlar, bulk action va pro-shortcutlarda. `Barcha funksiyalar` har ikkala ko'rinishda ishlaydi. Ko'rinish tanlovi foydalanuvchiga tegishli, majburiy rol emas.

## 5. Universal search

### 5.1 Qamrov

Mahsulot, mijoz/counterparty, savdo hujjati, buyurtma, to'lov, xodim — yagona qidiruvdan topiladi. Kelajakdagi modullar uchun kengaytiriladigan `search provider` shartnomasi taklif etiladi.

### 5.2 Xulq-atvor

- Har qanday ekranda qidiruv tugmasi va (desktopda) `Ctrl/⌘+K`; mobil ekranda yuqori darajadagi qidiruv.
- Natijalar turga ajratiladi: hujjat raqami, mahsulot kodi/nomi, mijoz nomi, ruxsat bo'lsa telefon va boshqa mos kalitlar; har natijada turi, nomi, kontekst (filial, status) va xabarsiz action.
- Vazifalar (`Yangi buyurtma`, `To'lov kiritish`) ham command-palette orqali topilishi mumkin, lekin `data result` va `command` ko'rinishi alohida.
- Search backend majburiy tenant/company/branch va row permission filterlarini **natijani qaytarishdan avval** bajaradi; autocomplete, count va recent ham shu qoida ostida.
- Search no-result: xato imlo uchun maslahat, scope ko'rsatish, filtrni tozalash, `Yangi yaratish` (ruxsat bo'lsa). Natija bo'lmasa mavjud bo'lmagan obyektni ochish tavsiya etilmaydi.
- Debounce, request cancel, pagination va typed filters; qidiruv so'rovining logging/analytics xavfsizlik siyosati bilan mos.

## 6. Workflow-first — ish oqimlari

### 6.1 Yangi sotuv (asosiy namunaviy flow)

`Dashboarddagi Yangi sotuv` → `Mijozni tanlash/yangi mijoz` → `Mahsulotni qo'shish, son/narx` → `Tekshirish` → `Tasdiqlash (vakolat bo'yicha)` → `Natija/hujjat`. Tugunlar ekran, wizard qadami yoki inline panel bo'lishi mumkin. Tanlov Sales & Inventory state machine va vakolatlar bilan boshqariladi; UI qonuniy tekshiruvlarni aylanib o'tmaydi.

- Har qadamda keyingi kutilayotgan amal, validatsiya va saqlanish holati aniq.
- Foydalanuvchi `Draft` bo'lsa qayta davom ettira oladi; draft saqlash qonuni domain tomonidan belgilanadi.
- Serverdagi `success` tasdiqlangach hujjat raqami ko'rsatiladi; optimistik UX hujjat posting yakunlandi deb yolg'on xabar bermaydi.
- Kerakli mahsulot topilmasa `Mahsulot yaratish` yon vazifasi (ruxsat bo'lsa) kontekst saqlagan holda ochiladi.

### 6.2 Omborga qabul

`Bugungi qabul navbati` → `Hujjatni ochish` → `Tekshirilgan miqdorlar` → `Farqlarni hal qilish` → `Qabulni tasdiqlash` → `Ombor harakati natijasi`. Hisob va qoldiq o'zgarishi Inventory modulining tasdig'iga bog'liq.

### 6.3 To'lovni qayd etish

`Muddati o'tgan qarzdorlik kartasi` → `Invoice/customer` → `To'lovni kiritish` → `Valyuta/summa/metod tekshirish` → `Tasdiqlash` → `Reconciliation holati`. ERP moliyasi bilan SaaS abonent to'lovi ikki alohida domen ekanligi interfeys nomlashida aniq.

### 6.4 Xatolar va retry

Network timeoutda status noma'lum bo'lishi mumkin: `Tekshirilmoqda` + qayta ko'rish, ko'r-ko'rona qayta posting yo'q. Kiritilgan form qiymatlari qayta urinishda imkon qadar saqlanadi. Destructive/bulk amallar uchun aniq ta'sir doirasi va qaytarish mumkinligi ko'rsatiladi.

## 7. Table design — katta hajmli ERP ma'lumotlari

- `Title + result count (if authorized) + Search + primary action`.
- Ko'rinadigan tez filtrlar (sana, filial, status) + qo'shimcha filtr drawer; aktiv filter chiplarini bir bosishda olib tashlash.
- Server-side filter/sort/page va barqaror sort key; cheksiz client-side 100k row yuklash taqiqlanadi.
- Column chooser, column pinning, resize, row density, saved view, eksport (serverda ruxsat va scope bilan).
- Jami summa/son faqat to'g'ri hisoblash scope bilan belgilanadi; faqat joriy sahifa hisobiga tayangan umumiy jami ko'rsatilmaydi.
- Bulk action oldidan `tanlangan N element`, filial/huquq cheklovlari, tasdiqlash va qisman muvaffaqiyatsizlik hisoboti.
- Empty, no results, offline/stale, partial data, loading, unauthorized va failure uchun alohida state.
- Sticky column/headers, keyboard focus, screen-reader column label, accessible sort status. Virtualizatsiya faqat profiling va a11y sinovidan o'tgach.

## 8. Mobile UX va responsive

**Bir xil mantiq, har xil joylashuv.** Desktopda sidebar + ko'p ustunli jadval; mobilda bottom navigation (3–5 tez-tez ishlatiladigan yo'nalish) + `Barchasi` katalogi, qidiruv va kontekstual action. Barcha modullar bottom navga tiqilmaydi.

- Asosiy amal bir qo'l bilan yetadigan hududda; katta touch targets, kamroq majburiy typing, barcode/camera/GPS imkonlari faqat tegishli permission va device availability bilan.
- Stol jadvallari mobil ekranda entity list/card bilan almashtiriladi; muhim status, miqdor, mijoz, sana va CTA yo'qolmaydi; qolgan field detail sahifasida.
- Multi-step form mobilda ham xuddi shu biznes ketma-ketligida; step indicator va saqlangan qiymatlar.
- Offline (`Mobile & Offline` moduli hal qiladigan tanlov): `Qurilmada saqlandi`, `Sinxronlash kutilmoqda`, `Server tasdiqladi`, `Konflikt` alohida statuslar; offline draft serverda tasdiqlangan deb berilmaydi.
- Filial almashish va balans/manfiy holat mobil ekranda ham tushunarli.

## 9. Accessibility, localization, shortcut

**Maqsad:** WCAG 2.2 AA darajasiga moslikni audit orqali tekshirish (PROPOSED, hali sertifikatlangan emas). To'liq keyboard navigation, ko'rinadigan focus, semantik sarlavha/label, screen-reader status, matnni kattalashtirish, 4.5:1 normal text contrast, 3:1 large text/UI grafik belgilari; animation kamaytirish (`prefers-reduced-motion`), rangga yolg'iz tayanmaslik, touch target, xato tavsifi va tuzatish yo'li.

- `Ctrl/⌘+K` global qidiruv; `?` shortcut yordam oynasi (inputda yozilayotganda ishlamaydi); boshqa shortcutlar collision/test asosida belgilanadi.
- Har shortcutning vizual ekvivalenti bor; shortcutni o'zgartirish imkoniyati keyingi bosqichda baholanadi.
- uz/en/ru, uzun tarjimalar bilan layout sinovi, nomlar lug'ati va raqam/sana/valyuta formats; yashirin yangi o'zbek alifbosi foydalanuvchiga yoqilmaguncha chiqmaydi.
- Serverdan kelgan error code localized message bilan moslanadi; tarjima bo'lmasa xavfsiz fallback.

## 10. Design system va wireframe contract

### 10.1 Asosiy komponentlar

App Shell, Tenant/Branch Switcher, Navigation Rail, Search Palette, Breadcrumb, Dashboard Card, Alert Banner, Data Table, Filter Builder, Entity Picker, Stepper, Draft/Saved Indicator, Confirm Dialog, Toast, Drawer, Permission State, Empty State, Offline/Sync Badge, Audit Detail, Locale Switcher. Komponentlar o'zining error/loading/disabled/focus holatlariga ega.

### 10.2 Wireframe (matnli)

**Desktop home:** `Top: tenant/branch | universal search | notifications | help | profile` / `Left: Workspace; Sales; Inventory; Finance; CRM; Reports; Settings` / `Main: Today's tasks; urgent alerts; primary actions; role KPIs; recent entities` / `Context panel: instructions, activity, related docs`.

**Sotuv wizard:** `Heading: Yangi sotuv, filial, saved state` / `Stepper: Customer → Items → Review → Confirm` / `Main: current step form` / `Right: compact totals and customer context` / `Footer: Back, Save draft (allowed), Continue/Confirm`.

**Document list:** `Heading + New` / `Search + quick filters + saved views` / `Count + columns + selection` / `Server-side table` / `Pagination` / `Selected action bar (conditional)`.

**Mobile home:** `Top context + Search` / `Today's primary action` / `Task feed and alerts` / `KPI compact carousel or stack` / `Bottom: Home, Tasks, Search, relevant domain, All`.

### 10.3 UX state matrix

Har muhim ekranga minimal states: `initial`, `loading`, `ready`, `empty`, `no-results`, `validation-error`, `unauthorized`, `network-error`, `saving`, `saved-draft`, `submitted`, `offline-pending` (agar tegishli). Dialoglar biznes operatsiya yakunini faqat backend tasdiqlaganda e'lon qiladi.

## 11. UX performance budget (PROPOSED)

| Metrika | Maqsad | Eslatma |
|---|---|---|
| Click feedback | < 100 ms | visual response, operatsiyaning yakunlanishi emas |
| Search first useful result | p75 < 1 s | normal connectivity/typical dataset; backend SLO bilan tekshiriladi |
| Common navigation usable | p75 < 1 s | cached route, realistic device |
| LCP | p75 ≤ 2.5 s | Core Web Vitals target |
| INP | p75 ≤ 200 ms | interaktivlik |
| CLS | p75 ≤ 0.1 | layout stability |

Loading skeleton mazmun shaklini saqlaydi; kechikishda progress va aniq xato/retry ko'rsatiladi. Barcha sonlar pilot instrumentatsiya bilan tekshiriladi, tasdiqlangan biznes SLA emas.

## 12. Architecture Decision Records

### ADR-UX-001 — Task-first, ko'rinarli navigatsiya
- **Status:** PROPOSED
- **Muammo:** chuqur menyular foydalanuvchiga funksiyani topishni qiyinlashtiradi.
- **Variantlar:** A) modul-first deep menu; B) faqat command palette; C) task-first dashboard + soddalashtirilgan modullar + qidiriladigan barcha funksiyalar katalogi.
- **Tanlov:** C.
- **Sabab:** tajribasiz foydalanuvchiga tez kirish, professionallarga to'liq funksionallik.
- **Xavf:** dashboard tartibsizlashishi; rol personalizatsiyasi murakkablashishi.
- **Ta'sir:** Security permissions, module registry, deep-links, localization.

### ADR-UX-002 — Role-aware UI, server-authorized actions
- **Status:** PROPOSED
- **Muammo:** hamma rolga bir xil murakkab ekran berish va UI permission bilan authorizationni adashtirish.
- **Variantlar:** statik ekran; frontend-only role hide; permission-aware server-authorized presentation.
- **Tanlov:** permission-aware personalizatsiya, barcha huquq serverda tekshiriladi.
- **Sabab:** moslik va himoya.
- **Xavf:** permission tizimiga kuchli bog'liqlik, cache eskirishi.
- **Ta'sir:** Auth/RBAC, tenant scope, search, offline.

### ADR-UX-003 — Universal search + commands
- **Status:** PROPOSED
- **Muammo:** ma'lumot turiga qarab turli sahifalarda izlash kerak.
- **Variantlar:** har modulda alohida qidiruv; yagona search natijalari; search + command palette.
- **Tanlov:** ruxsatga asoslangan global search + alohida belgilangan commands.
- **Sabab:** ma'lumot yoki amalni kirish nuqtasidan mustaqil topish.
- **Xavf:** indeks kechikishi, permission leak, qidiruv xarajati.
- **Ta'sir:** Backend, Database, Security, audit, Resource Usage (pricing doirasini billing moduli belgilaydi).

### ADR-UX-004 — Workflow-first business actions
- **Status:** PROPOSED
- **Muammo:** foydalanuvchi hujjat turini tushunmaguncha amaliyotni boshlay olmaydi.
- **Variantlar:** generic CRUD form; wizard; kontekstga mos guided flow.
- **Tanlov:** yuqori chastotali task uchun guided flow, power-user alternativasi.
- **Sabab:** onboarding va xatoni kamaytirish.
- **Xavf:** murakkab biznes senariylarida qadam ko'payishi.
- **Ta'sir:** Sales/Inventory, Finance, CRM state machine va validators.

### ADR-UX-005 — Server-driven data views
- **Status:** PROPOSED
- **Muammo:** katta qatorlar brauzer xotirasi va latencyga zarar qiladi.
- **Variantlar:** full client table; infinite scroll; server query + paginated views.
- **Tanlov:** server-driven filtering/sorting/pagination; virtualization optional.
- **Sabab:** predictable resurs, tez ochilish, tenant-permission nazorati.
- **Xavf:** qiyin aggregate, filtr semantikasi, eksport consistency.
- **Ta'sir:** Backend, Database, Reporting, Security.

### ADR-UX-006 — Responsive parity va a11y
- **Status:** PROPOSED
- **Muammo:** mobil va desktop turli, topilishi qiyin ish usuliga aylanadi.
- **Variantlar:** desktop-first shrink; alohida navigatsiya logikasi; bir xil IA/business flows, device-specific presentation.
- **Tanlov:** shared task taxonomy, device-specific layout, WCAG 2.2 AA audit target.
- **Sabab:** xodimning qurilma almashtirganda qayta o'rganmasligi.
- **Xavf:** mobile murakkab jadval, offline/assistive tech edge case.
- **Ta'sir:** Mobile/Offline, Localization, Frontend, QA.

## 13. OPEN QUESTIONS

1. Ruxsatlar modeli foydalanuvchi-kompaniya-filial va multi-role holatlarni qanday birlashtiradi? (Security)
2. MVP uchun `top-20 task` ro'yxati, ularning biznes ustuvorligi va benchmark auditoriyasi kim? (Product + barcha modullar)
3. Universal search qaysi fieldlarni ko'rsatishi/indekslashiga xavfsizlik cheklovlari qanday? (Security, Backend)
4. Saqlangan filter/column view'lar shaxsiymi, jamoaviymi? (Tenant/Security)
5. Mobile offline yozish, konflikt hal qilish va GPS UX qoidalari qaysi workflows uchun mavjud? (Mobile/CRM)
6. Real role taxonomy va module naming qachon tasdiqlanadi? (Master/Module Registry)
7. UX test uchun qaysi qurilma, tarmoq, til, ma'lumot hajmi baseline bo'ladi? (Infrastructure/QA)
8. Keyboard shortcut konfliktlari va foydalanuvchi konfiguratsiyasi MVP doirasidami? (Frontend/Product)

## 14. Hujjatlar sinxronizatsiyasi uchun tayyor yozuvlar

**DECISIONS.md:** `ADR-UX-001..006 | PROPOSED | UX_ARCHITECTURE.md` (APPROVED deb kiritilmasin).

**MASTER_ARCHITECTURE.md:** UI presentation layer vazifa-first navigatsiya, role-aware, permission-backed search, workflow-first interaction va responsive parity taklifi sifatida havola.

**REQUIREMENTS.md:** funksiya topilishi (`top-20` 2–3 action), universal search object coverage, WCAG 2.2 AA target, p75 performance target — PROPOSED nofunksional talablar.

**MODULE_REGISTRY.md:** UX umumiy cross-cutting presentation domeni; Security/Backend/Multi-Tenant/Billing/Sales/Inventory/Finance/CRM/Localization/Mobile bog'lanishlari.

**OPEN_QUESTIONS.md:** yuqoridagi 8 savolni qo'shish (takror yozilmasin, havola ishlatilsin).

**CHANGELOG.md:** `2026-10-11 — UX_ARCHITECTURE.md dastlabki PROPOSED dizayn va Codex spetsifikatsiyasi tuzildi; boshqa fayllarga o'zgarish kiritilmagan.`

## 15. Codex uchun frontend architecture specification (KOD YOZMASLIK)

**Status:** SPECIFICATION / PROPOSED; loyiha implementatsiya bosqichiga o'tguncha kod yozilmasin. Codex kerak bo'lsa mavjud skills/MCP orqali dizayn referenslarini va tasdiqlangan modullarni o'rgansin, lekin biznes qarorini mustaqil o'zgartirmasin.

### 15.1 Maqsad va artefaktlar

1. `App Shell` va `Navigation IA`: desktop/sidebar, mobil/bottom nav, tenant/branch switcher, role-based cards, all-functions catalog; ikki/uch harakatli task navigation map.
2. `Information Architecture`: barcha modullar uchun route taxonomy va deep-link contract (aniq URLni texnik bosqichda belgilang).
3. `Role Dashboard`: data contracts (role, allowed actions, scope, widgets, last-updated, loading state) va drill-down.
4. `Universal Search`: entity-type registry, search response contract (type, title, subtitle, route, scope, permissions), command registry, permission filtering and no-results flows.
5. `Workflow Engine UX`: stepper, draft state, validation errors, review/confirm, async/pending outcome, retry-safe UI; domain workflow lifecycle serverning authoritative holatiga asoslanadi.
6. `Large Data Views`: server filtering/sorting/pagination, saved views, column controls, bulk actions, totals scope and export permissions.
7. `Responsive & Mobile`: shared interaction semantics, mobile card/list, prominent main CTA, offline-status indicators.
8. `Accessible Design System`: token specs, component state matrix, accessibility checklist, locale-aware layout, shortcut map.
9. `Analytics & Testing`: funnel events without sensitive payload, top-20 usability plan, cross-role testing, p75 web vitals and large-dataset benchmark.

### 15.2 Integratsiya qoidalari

- UI component authorization emas. Har so'rov va har komanda backenddagi tenant, company, branch, role/permission va object-level policy bilan tasdiqlanadi.
- Posting/payout/sale confirmation faqat authoritative backend success holatida final deb chiqariladi; idempotency va error semantics Backend arxitekturasi bilan kelishiladi.
- Wallet va billing qiymatlari faqat Billing modulidan olinadi; seat/usage summalari UXda taxmin qilinmaydi, minus balans uchun taxminiy limit ishlab chiqarilmaydi.
- Boshqa chatdagi APPROVED domain statuslarini o'zgartirmaslik; kelishmovchilik aniqlansa Architecture Reviewga ochiq savol sifatida yuborish.
- Next.js/Flutter tanlovi hozircha tavsiya. Frontend design contract texnologiyaga bog'lanmagan bo'lsin.

### 15.3 Qabul testlari — specification darajasida

- Yangi sotuv, mahsulot, mijoz, buyurtma, to'lov va ishchi qidiruvi umumiy searchga kiritilgan.
- Har rol dashboardida kamida tasks/alerts/actions/KPI uchun aniq bo'sh/yuklanish/xato holati mavjud.
- 100k+ hujjatlik datasetda butun jadval clientga yuklanmaydi; server paginatsiyasi va filtr ishlaydi.
- Forbidden object search result, autocomplete, count, recent va URL orqali ochilmaydi; backend authorization sinovi alohida.
- Mobile/desktop user task semantikasi mos; ekran o'zgarganda amaldagi draft saqlash qoidasi buzilmaydi.
- Keyboard-only, screen reader, 200% zoom, high contrast, localized long text va reduced-motion regressiya tekshiriladi.
- UZ/EN/RU UI va yashirin kelajak alifbosi flag qoidasi i18n arxitekturasi bilan mos.
- UX testlar real foydalanuvchida `time-to-find`, completion, 2–3 navigation action reachabilityni qayd etadi; boshlang'ich bazaviy qiymatlar uydirilmaydi.

**Codexga topshiriladigan ish:** hozir frontend kodi emas, ushbu spetsifikatsiyadan kelib chiqib foydalanuvchi oqimlari, permission-state matrix, IA va ekran inventarini tekshirish va o'zaro modullar bilan moslashtirish. Implementatsiya faqat alohida ruxsatdan so'ng.