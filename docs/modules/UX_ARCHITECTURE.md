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

## 16. AR-012 UX Quality Gate — MR-16/MR-17/MR-20

**Status: PROPOSED, NOT APPROVED.** This is an acceptance specification, not a measured success result or approved MVP list. BIZ-001…010 user-approved business constraints preserved. Outstanding ownership and scope decisions remain OPEN. References: [DECISIONS](../../DECISIONS.md), [OPEN QUESTIONS](../../OPEN_QUESTIONS.md), [REGISTRY](../../MODULE_REGISTRY.md), [REVIEW](../reviews/ARCHITECTURE_REVIEW.md).

### 16.1 Candidate persona × top-20 workflow acceptance matrix (MR-16)

Personas are examples, not an approved RBAC grant. All candidate tasks require desktop and mobile review; mobile capability/offline availability is not implicitly promised. Discovery/launch goal is at most 2–3 deliberate interactions from Today, dashboard or global search, NOT completion of multi-step business workflows. Domain owners define actual transitions. Every workflow needs permitted and denied test cases; this matrix has not passed usability testing.

| ID | Candidate persona | Business workflow | Entry/path | Observable acceptance / forbidden case | Domain owners |

|---|---|---|---|---|---|

| UX-T01 | Sotuvchi | Yangi sotuv | Today > Yangi sotuv > mijoz/mahsulot > review | Draft/final farqlansin; narx xatosi finalni bloklaydi | Sales, CRM |
| UX-T02 | Sotuvchi | Buyurtma topish | Global search > order > detail | Faqat vakolatli natija va detal; foreign ID ko‘rinmasin | Sales, Security |
| UX-T03 | Sotuvchi | Qaytarish/bekor qilish | Buyurtma > actions > sabab | Faqat ruxsatli transition; reversal Finance/Sales tasdiqlaydi | Sales, Finance |
| UX-T04 | Omborchi | Kirim qabul | Today > kirim > inspect/post | Miqdor, ombor va xato ko‘rinsin; noto‘g‘ri posting bloklansin | Inventory |
| UX-T05 | Omborchi | Omborlararo transfer | Stock > Transfer > destination | Cross-tenant destination ko‘rinmasin; filial huquqi qayta tekshirilsin | Inventory, Security |
| UX-T06 | Omborchi | Inventarizatsiya | Task > count > discrepancy | Farq, approval va audit ko‘rinsin; silent adjust yo‘q | Inventory |
| UX-T07 | Omborchi | Kam qoldiq | Alert > SKU > action | Filialga tegishli SKU/foydali keyingi amal, stale alert belgisi | Inventory |
| UX-T08 | Sotuvchi | Mijozni topish/yaratish | Search > mijoz > create/edit | Dublikat va scope aniq; AR-008 identity OPEN | CRM |
| UX-T09 | Agent | Mijoz tashrifi | Today > route > customer > visit | GPS/evidence statuslari, offline/pending aniq | CRM, Mobile |
| UX-T10 | Agent | Offline buyurtma | Customer > order draft > sync queue | Local saved server accepted degani emas; retry ko‘rinsin | Mobile, Sales |
| UX-T11 | Ekspeditor | Yetkazish dalili | Manifest > stop > foto/imzo | Upload pending va server receipt farqlansin | Mobile, Inventory |
| UX-T12 | Buxgalter | Debitor to‘lovi | AR > customer > receipt | Draft/settled va allocation; erta GL posting yo‘q | Finance |
| UX-T13 | Buxgalter | Kreditor to‘lovi | AP > bill > approval | Tasdiq, partial payment va reversal tushunarli | Finance |
| UX-T14 | Buxgalter | Bank reconciliation | Bank > unmatched > match | Mismatch, source, audit va qaytarish oqimi | Finance |
| UX-T15 | Buxgalter | Moliyaviy hisobot | Dashboard > reports > P&L/AR | As-of date, scope va export huquqi | Finance, Security |
| UX-T16 | Direktor | Filiallar kesimi KPI | Dashboard > branch context > KPI | All-branch faqat explicit ruxsat bilan | Tenant, Security |
| UX-T17 | Direktor | Savdo exception | Alert > exception > next action | Revoked permissiondan keyin action ishlamasin | Sales, Security |
| UX-T18 | Administrator | Xodim ruxsati | Settings > member > role | Grant/revoke authorized, audit, stale-refresh | Security, Tenant |
| UX-T19 | Administrator | SaaS balans/usage | Billing > ledger > charge | Seat/Business/Resource va 1 kredit=10 UZS; haqiqiy minus xarajat, taxminiy limit emas | Billing |
| UX-T20 | Administrator | Balans to‘ldirish | Billing alert > top-up > status | Pending!=settled; duplicate va payment failed ko‘rinsin | Billing |

**Common per-row test contract (PROPOSED):** persona and actual membership, tenant/company/branch scope, desktop/mobile device, uz/en/ru locale, online/degraded/offline state, initial domain state, expected server-accepted outcome, blocked/unauthorized outcome, UI status and recovery, clicks/time/errors, accessible interaction, evidence link, and domain-owner signoff. A draft/queued result must never be reported as posted/settled. Signed-off MVP device and role coverage remains OPEN.

### 16.2 Active context and cross-tenant/branch safeguards

**PROPOSED:** persistent current tenant/company/branch label, explicitly authorized All-branches mode only. On switch, server revalidates membership and action; scope-bound search, suggestions, recents, counts, cards, notification previews, pending requests, download links and cached results are hidden/invalidated before showing new context. In-flight response from prior scope cannot repaint current screen. URL/deep link is an untrusted locator, not authority; inaccessible objects reveal neither title nor existence. Prompt before abandoning unsaved draft; offline queued commands stay irrevocably associated with original scope, never reassign on switch. Tenant/Company/BillingAccount cardinality AR-001 and identity ownership AR-013 are OPEN. Negative tests: revoked membership mid-session, switched branch during request, bookmarked foreign record, shared-device user switch, stale offline queue, All-branches without grant, same-number document in different tenants.

### 16.3 Authorized global search — MR-17

**PROPOSED:** a single entry point for product, customer, document, order, payment, employee and separate labeled commands. Authorization filters apply to query results AND autosuggest, facets/counts, autocomplete, snippets, recent items, history, saved queries, notifications, exports and detail navigation. No unauthorized item identifiers, existence hints, aggregate counts or thumbnails leak; detail/action always rechecks server scope. Cache keys include tenant, membership/scope and authorization version; revocation invalidates stale results. Index records carry scope and deletion/version information. Reindex is resumable, tenant-scoped, auditable and must suppress stale/revoked records; explicit freshness/partial-results states when lag matters. Test typo/apostrophe and language/script aliases without altering originals. **OPEN:** index ownership, index-lag budget, reindex capacity, permission side-channel policy and search Resource Usage classification (Database/Security/Localization/Billing owners).

### 16.4 Onboarding and workflow-first error recovery

**PROPOSED:** first-run role-relevant Today checklist, contextual help and searchable All functions catalogue. Basic/advanced mode is presentation only, not permission. Every guided flow shows task purpose, current step, back/exit, validation, optional draft resume and authoritative confirmation. Define distinct loading, empty, no-permission, offline, queued, syncing, accepted, rejected, conflict, retryable timeout and unknown server outcome states. Offline new order/visit/evidence is only local pending until server validates; no implied offline final stock reservation, GL posting, settlement or SaaS charge. Retry stable operation identity; show server conflict with domain-approved resolution. Device logout/revocation blocks sending pending work until reauthorization. Billing charge boundary AR-010 and offline rating month AR-009 remain OPEN. See Mobile and Billing.

### 16.5 Localization, accessibility and quantitative measurement — MR-20

**APPROVED:** uz/en/ru public interface and internal-only hidden future Uzbek alphabet. **PROPOSED test protocol:** each prioritized flow validated in three public languages, varied content lengths, number/currency/date/time-zone independence, Uzbek apostrophe/search alias, original record preservation; hidden alphabet only flagged internal preview, not a public option. WCAG 2.2 AA target for core workflow: keyboard-only, screen reader manual walkthrough, labeled form validation, focus/dialog/announcements, contrast/color independence, zoom/reflow, minimum usable touch targets, reduced motion, status readable without color. Automated audit alone does not prove compliance.

**Quantitative targets PROPOSED only, not measured:** discover/start frequent task in <=3 deliberate user interactions; LCP p75 <=2.5 s; INP p75 <=200 ms; CLS p75 <=0.1; typical authorized search/filter p75 <=1 s under defined network/dataset. Report per persona/device/locale the completion rate, p50/p75/p95 time, steps, wrong turns, abandonment, retry recovery and inaccessible conditions. Validate against genuinely comparable tasks and consented observations; do not claim Sales Doctor superiority without measured baseline.

**OPEN:** top-20 MVP ordering, role/desktop-mobile feature coverage; low-end Android/desktop spec, browser/screen-reader profile, offline duration, 3G/4G/Wi-Fi profile, dataset/tenant sizes, concurrency, statistical sample, exact acceptance thresholds and instrumentation privacy. UX, Mobile, Localization, Security, Infra and Master must sign off profiles before metrics become binding.

### 16.6 UX Quality Gate exit checklist and cross-module ownership

| Gate | PROPOSED evidence | Owner / unresolved |

|---|---|---|

| G0 Task scope | User/domain-approved top-20 and role/device matrix | Master, Sales, Finance, Billing, CRM — OPEN MR-16 |

| G1 Safe context/search | Auth denial tests for switching, cached results, search counts, recents, deeplinks | Tenant, Security, UX — OPEN AR-013/MR-17 |

| G2 Valid business flow | 20 positive and forbidden workflows with actual server state and error recovery | Sales/Inventory, Finance, Billing, CRM — OPEN AR-005/008/010 |

| G3 Mobile recovery | Offline, sync conflict, device revoke, stale scope and partial uploads | Mobile, Security — OPEN AR-009 |

| G4 Inclusive UX | uz/en/ru, desktop/mobile, a11y manual/automated runs | Localization, Mobile, UX — OPEN MR-20 |

| G5 Evidence | Per-task observed metrics, device/network profile and reproducible evidence | UX, Infra, QA — OPEN MR-16/20 |

**No gate is marked passed:** these are PROPOSED test contracts. Implementation readiness remains BLOCKED per architecture review. Shared registries were NOT edited. Proposed future registry delta: record AR-012 matrix/quality gate as PROPOSED; keep MR-16/17/20, AR-001/005/008/009/010/013 OPEN. Domain references: [Sales](SALES_INVENTORY.md), [Finance](FINANCE_ARCHITECTURE.md), [Billing](BILLING_ARCHITECTURE.md), [Security](SECURITY_ARCHITECTURE.md), [Mobile](MOBILE_ARCHITECTURE.md), [Localization](LOCALIZATION_ARCHITECTURE.md).

## 17. ROUND 2 UX Acceptance Contract — AR-012 / MR-16, MR-17, MR-20 (2026-10-12)

**Status: PROPOSED — no usability tests run and no competitor superiority claim measured.** The prior UX-T01…T20 candidate workflow list (§16) is preserved; this section is its detailed acceptance addendum, not an MVP approval. **APPROVED requirement:** ERP UX must be substantially more understandable, easier to discover and quicker to learn than Sales Doctor. Quantified budgets remain PROPOSED. Sales, Inventory, CRM, ERP Finance, SaaS Billing, Security, Mobile and Localization retain ownership over business invariants, security, offline and translation contracts. Review: [Round 2 §15](../reviews/ARCHITECTURE_REVIEW_ROUND_2.md).

### 17.1 Test fixture and UI-state vocabulary

**Fixture namespace PROPOSED:** synthetic T-A with C-A1/B-A1/B-A2 and C-A2/B-A3; foreign T-B with C-B1/B-B1; one user per persona plus a dual-branch director, revoked member, cross-tenant impostor, stale membership version, duplicate external ID, 20k-order dataset and 100k-product dataset (sizes are test candidate values, not production scale commitments). Every test uses synthetic prices and ledger balances, never changes approved billing tariff. Explicit states: **E** empty (helpful CTA when permitted; otherwise no unauthorized prompt), **L** loading (nonblocking skeleton/progress without data from previous scope), **X** error (validation/authorization/timeout/conflict, no false success), **O** offline/degraded (online-only write disabled; local draft and queued != accepted). For all rows E/L/X/O are required; applicability differences described in §17.2. Each must support uz/en/ru, screen reader/keyboard where applicable, desktop/mobile responsive evaluation with feature availability explicitly marked.

**Test recording:** test ID, actor/membership, active tenant/company/branch, exact domain state, screen trace, server response, before/after record count, screenshot/video subject to privacy permission, pass/fail outcome, network/device/locale, screen-reader notes, number of discovery actions, recovery time, owner sign-off. No pass inferred from documentation.

### 17.2 Per-task contract — UX-T01…UX-T20

Each row names navigation starting at role-specific Today/dashboard, *illustrative permission keys* (not approved IAM strings), domain-approved success and failure checks, and a concrete synthetic dataset. **Universal state oracle** per row: E shows scoped empty state and permitted new task/help; L shows scoped progress and freezes relevant destructive action; X shows reason/next safe action, preserves entered data and no unconfirmed posting; O labels cache freshness, allows only owner-approved offline draft/queue and never claims final business acceptance. A row-specific O flag below overrides generic drafts when the task must be online.

| ID / persona | Screens (PROPOSED) | Required capability (ILLUSTRATIVE, never client-only) | E/L/X/O special expectations | Success / failure oracle | Dataset fixture |
|---|---|---|---|---|---|
| UX-T01 / Sotuvchi | Today → Yangi sotuv → Customer → Lines → Review → Submit | `sales.order.create; customer.read; product.read; price.read; sales.order.submit (conditional)` | E: authorized scoped empty CTA; L: current-scope skeleton; X: domain error/retry, no optimistic final; O: read-only cached view with timestamp if available; online-only commit disabled | PASS: Valid draft retained, only server-confirmed submission marked successful; stale price/denied permission displays rejected state. FAIL: denied/wrong scope, duplicate effect or false-success display | two customers, 3 SKUs, one outdated price, one denied branch |
| UX-T02 / Sotuvchi | Global Search → Buyurtma natijasi → Order detail | `sales.order.read; search scoped` | E: authorized scoped empty CTA; L: current-scope skeleton; X: domain error/retry, no optimistic final; O: read-only cached view with timestamp if available; online-only commit disabled | PASS: Permitted order opens; forbidden order has no count/title/snippet even for exact ID. FAIL: denied/wrong scope, duplicate effect or false-success display | same order number in two tenants; 1 revoked membership |
| UX-T03 / Sotuvchi | Order detail → Return/Cancel → Reason → Review | `sales.order.return/cancel; finance.refund.read only if granted` | E: authorized scoped empty CTA; L: current-scope skeleton; X: domain error/retry, no optimistic final; O: read-only cached view with timestamp if available; online-only commit disabled | PASS: Domain-authorized transition recorded; already invoiced/fulfilled invalid reversal blocked. FAIL: denied/wrong scope, duplicate effect or false-success display | draft, confirmed, dispatched, invoiced, reversed orders |
| UX-T04 / Omborchi | Today → Kutilayotgan kirim → Receipt → Inspect → Post | `purchase.receipt.read; inventory.receipt.post` | E: authorized scoped empty CTA; L: current-scope skeleton; X: domain error/retry, no optimistic final; O: read-only cached view with timestamp if available; online-only commit disabled | PASS: Posted only on authoritative receipt; negative/overage/rejected quantity blocked or reviewed. FAIL: denied/wrong scope, duplicate effect or false-success display | PO with 2 lines, partial arrival, rejected line |
| UX-T05 / Omborchi | Ombor → Transfer → Source/Destination → Review | `inventory.transfer.create; warehouse.read; branch-scope grant` | E: authorized scoped empty CTA; L: current-scope skeleton; X: domain error/retry, no optimistic final; O: read-only cached view with timestamp if available; online-only commit disabled | PASS: Transfer stays in valid state; foreign tenant warehouse never selectable. FAIL: denied/wrong scope, duplicate effect or false-success display | two branches, 3 warehouses, one unauthorized tenant |
| UX-T06 / Omborchi | Today → Inventory count → Enter counts → Variance review | `inventory.count.read/write; inventory.adjust.approve separate` | E: authorized scoped empty CTA; L: current-scope skeleton; X: domain error/retry, no optimistic final; O: read-only cached view with timestamp if available; online-only commit disabled | PASS: Saved count distinct from posted adjustment; unapproved variance does not affect stock. FAIL: denied/wrong scope, duplicate effect or false-success display | 5 SKUs, two variances, count cutoff snapshot |
| UX-T07 / Omborchi | Dashboard low-stock alert → SKU → Allowed replenishment action | `inventory.stock.read; purchase.replenish.create conditional` | E: authorized scoped empty CTA; L: current-scope skeleton; X: domain error/retry, no optimistic final; O: read-only cached view with timestamp if available; online-only commit disabled | PASS: Correct branch and fresh threshold; unauthorized replenishment denied. FAIL: denied/wrong scope, duplicate effect or false-success display | normal/low/out-of-stock SKUs in two branches |
| UX-T08 / Sotuvchi | Search → Mijoz → Open / New → Duplicate check → Save | `crm.customer.read/create/update; party.lookup conditional` | E: authorized scoped empty CTA; L: current-scope skeleton; X: domain error/retry, no optimistic final; O: read-only cached view with timestamp if available; online-only commit disabled | PASS: Duplicate candidates correctly scoped; invalid legal identity flagged, no silent merge. FAIL: denied/wrong scope, duplicate effect or false-success display | two similar names, distinct Party IDs, same phone, separate companies |
| UX-T09 / Agent/ekspeditor | Today → Route → Outlet → Visit → Record outcome | `crm.route.read; crm.visit.create; assigned outlet scope` | E: authorized scoped empty CTA; L: current-scope skeleton; X: domain error/retry, no optimistic final; O: scope-bound local queue permitted only for supported evidence/drafts | PASS: Pending offline vs accepted online clearly labelled; forbidden outlet absent. FAIL: denied/wrong scope, duplicate effect or false-success display | assigned route with 3 stops, unassigned outlet, GPS denied |
| UX-T10 / Agent/ekspeditor | Outlet → New order → Draft → Queue → Sync status | `sales.order.draft; mobile.sync.submit after reauthorization` | E: authorized scoped empty CTA; L: current-scope skeleton; X: domain error/retry, no optimistic final; O: scope-bound local queue permitted only for supported evidence/drafts | PASS: Queued local draft not confirmed sale; duplicate retry one accepted occurrence. FAIL: denied/wrong scope, duplicate effect or false-success display | offline basket 3 SKUs, server price change, revoked agent |
| UX-T11 / Agent/ekspeditor | Manifest → Delivery stop → Evidence → Upload → Receipt | `delivery.assignment.read; delivery.evidence.submit` | E: authorized scoped empty CTA; L: current-scope skeleton; X: domain error/retry, no optimistic final; O: scope-bound local queue permitted only for supported evidence/drafts | PASS: Uploading ≠ proof accepted; partial upload/reject and retry shown. FAIL: denied/wrong scope, duplicate effect or false-success display | manifest with photo/signature, interrupted upload and invalid format |
| UX-T12 / Buxgalter | Finance → Debtors → Customer → Receipt/Allocation → Review | `finance.ar.read; finance.receipt.create; finance.receipt.post conditional` | E: authorized scoped empty CTA; L: current-scope skeleton; X: domain error/retry, no optimistic final; O: read-only cached view with timestamp if available; online-only commit disabled | PASS: Pending/settled distinction; no GL posting before owner confirmation. FAIL: denied/wrong scope, duplicate effect or false-success display | invoice 100 units, partial receipt 40, advance, duplicate request |
| UX-T13 / Buxgalter | Finance → Payables → Supplier bill → Approve/Pay | `finance.ap.read; finance.payment.create/approve by SoD` | E: authorized scoped empty CTA; L: current-scope skeleton; X: domain error/retry, no optimistic final; O: read-only cached view with timestamp if available; online-only commit disabled | PASS: Approval separated from execution; partial/failed/reversed payments accurate. FAIL: denied/wrong scope, duplicate effect or false-success display | bill, insufficient access, dual approver, partial pay |
| UX-T14 / Buxgalter | Finance → Bank → Unmatched → Match → Review | `finance.bank.read; finance.reconcile.perform` | E: authorized scoped empty CTA; L: current-scope skeleton; X: domain error/retry, no optimistic final; O: read-only cached view with timestamp if available; online-only commit disabled | PASS: Matched only after server posting; mismatched currency/duplicate blocked. FAIL: denied/wrong scope, duplicate effect or false-success display | statement 4 transactions, one FX mismatch, duplicate reference |
| UX-T15 / Buxgalter | Dashboard → Reports → AR Aging / P&L → Filter → Export | `finance.report.read; finance.export conditional` | E: authorized scoped empty CTA; L: current-scope skeleton; X: domain error/retry, no optimistic final; O: read-only cached view with timestamp if available; online-only commit disabled | PASS: As-of, legal entity, branch and data freshness explicit; export denied if grant absent. FAIL: denied/wrong scope, duplicate effect or false-success display | closed/open month, 2 branches, restricted report |
| UX-T16 / Direktor | Director dashboard → Company/Branch picker → KPI detail | `kpi.read for each allowed company/branch; aggregate permission separate` | E: authorized scoped empty CTA; L: current-scope skeleton; X: domain error/retry, no optimistic final; O: read-only cached view with timestamp if available; online-only commit disabled | PASS: No implicit global aggregation; prior-scope KPI cleared immediately on switch. FAIL: denied/wrong scope, duplicate effect or false-success display | 2 companies, 3 branches, one forbidden legal entity |
| UX-T17 / Direktor | Today Exceptions → Sales alert → Detail → Decision | `sales.exception.read; sales.approve conditional` | E: authorized scoped empty CTA; L: current-scope skeleton; X: domain error/retry, no optimistic final; O: read-only cached view with timestamp if available; online-only commit disabled | PASS: Revocation while screen open blocks action; old alert not actionable. FAIL: denied/wrong scope, duplicate effect or false-success display | pending order exception, stale alert, revoked approver |
| UX-T18 / Administrator | Settings → Members → Member → Role review → Save | `membership.read; role.assign/revoke (authoritative)` | E: authorized scoped empty CTA; L: current-scope skeleton; X: domain error/retry, no optimistic final; O: read-only cached view with timestamp if available; online-only commit disabled | PASS: Permission version refresh and audit; no self-elevation or unauthorized assignment. FAIL: denied/wrong scope, duplicate effect or false-success display | two tenant members, one suspended, privileged role request |
| UX-T19 / Administrator | Workspace → SaaS Billing → Balance → Usage ledger | `billing.ledger.read; billing.usage.read for payer scope` | E: authorized scoped empty CTA; L: current-scope skeleton; X: domain error/retry, no optimistic final; O: read-only cached view with timestamp if available; online-only commit disabled | PASS: Shows actual Seat/Business/Resource charges, 1 credit=10 UZS; no invented overdraft limit. FAIL: denied/wrong scope, duplicate effect or false-success display | wallet with positive then negative entries and branch attribution |
| UX-T20 / Administrator | Billing alert → Top-up → Payment state → Ledger receipt | `billing.topup.create; billing.ledger.read` | E: authorized scoped empty CTA; L: current-scope skeleton; X: domain error/retry, no optimistic final; O: read-only cached view with timestamp if available; online-only commit disabled | PASS: Pending vs settled; retry does not double-credit; failures visible. FAIL: denied/wrong scope, duplicate effect or false-success display | settled top-up, pending provider callback, duplicate transaction |

**Granular state verification:** for every task, run (1) empty permitted dataset and inaccessible-only dataset — these must not be distinguishable through counts, (2) delayed response with scope switch mid-flight, (3) server validation error, explicit 403 and conflict/version mismatch, (4) offline during entry and during server acknowledgment, (5) retry after indeterminate outcome, and (6) revoked access immediately before final commit. Domain-specific write availability in offline mode is determined by Mobile/Sales/Finance owners, not inferred from UI. For online-only finance, subscription and administrative tasks, O is clearly disabled with preserved uncommitted input when safe.

### 17.3 Role dashboard and task-first navigation contract

| Persona | Default first screen | Visible primary action | Alerts / KPI (authorized and scoped) | Secondary discoverability |
|---|---|---|---|---|
| Direktor | Today / Company summary with explicitly labeled branch scope | Review exceptions, drill into KPI | Sales trend, overdue receivables, branch exceptions with as-of date | Search, All Functions, permitted consolidated reports |
| Savdo agenti | Today / route + customer tasks | Start visit / New order | Assigned visits, sync queue, rejected drafts | Customer search, route, Sales actions |
| Omborchi | Today / warehouse queue | Receive goods / Pick / Count | Pending receipt, low stock, discrepancy by warehouse | SKU search, transfer, inventory history |
| Buxgalter | Today / finance tasks | Record receipt / reconcile | AR/AP due, unmatched bank lines, closed-period warnings | Finance reports and document search |
| Administrator | Today / workspace operations | Manage members / Billing balance | Security alerts, actual wallet balance and charge history | Settings, audit, search and full permission-filtered function catalog |

Navigation = persistent active company/branch selector + Today + global search + 4–6 task groups adjusted for real entitlements + searchable All Functions catalog; contextual step breadcrumbs and related next action are always visible. **Function discoverability is not permission grant:** use server capabilities, not fixed role labels. Deep actions may require domain confirmation beyond 3 discovery clicks. For dual-role user, offer pinned tasks (PROPOSED), not silently merged permission scopes.

### 17.4 Context-switch state machine and leakage gate

**PROPOSED states:** ACTIVE(scope/version) → SWITCH_REQUESTED → UNSAVED_WORK_PROMPT → CLEAR_SCOPE_BOUND_VIEWS → SERVER_REAUTHORIZE → LOAD_NEW_SCOPE → ACTIVE(new scope/version); on denial → DENIED_SAFE with no previous tenant data shown. Request generation token/context version gates all in-flight results; stale promises, notifications, analytics, query facets/counts, document previews, thumbnails, downloads and clipboard-sensitive actions cannot paint new-scope UI. Persist only preference/approved drafts tied to original scope; no moving drafts or offline queue across scope. Deep links resolve only after server authorization and use indistinguishable safe denied/absent response. Logout/revoke clears visible privileged data immediately. **Negative fixture:** T-A→T-B foreign deep link, C-A1→C-A2 switch with same document number, stale websocket alert, slow search returning after switch, offline device then membership revoked. Cross-company aggregate and Billing payer cardinality remain OPEN with Tenant/Billing owners.

### 17.5 Universal search exposure contract — MR-17

**PROPOSED search resource categories:** product, customer/party/outlet, Sales documents and orders, ERP payments, employees (only if permitted), SaaS Billing ledger (separate scope). Commands are clearly distinguished from records. Return only authorized records and authorized *metadata*, including total counts, autocomplete, snippets, highlights, facets, recents, history, saved searches, ranking signals, related objects and exports. Scope intersection = trusted user membership × tenant/company/branch grants × resource/action grants × record-level access; backend authoritative and rechecked at result open and command execution. **Never** hide unauthorized rows only in frontend. Cache/index partition by tenant, permission version and branch filter; revocation, deletion or migration must suppress stale results and recents even if index is lagging. Index freshness visible when relevant; explicit stale/partial state, no ungrounded promise of instant reindex. Search history storage/retention and telemetry privacy OPEN. Alias normalization for uz/en/ru must preserve original IDs and official document text; future Uzbek alphabet flag OFF by default. Test identical prefixes across T-A/T-B, forbidden item in facets/counts, cleared recents after revoke, paging and autocomplete races, offline stale index, deleted/moved object and search overlay during tenant switch.

### 17.6 Workflow-first progressive disclosure and recovery

**Beginner mode (PROPOSED):** Today 1–3 prominent domain-allowed actions, plain-language next step, example-only onboarding dataset marked synthetic, accessible stepper, review before committing, clear outcome receipt, help and exit. **Professional mode (PROPOSED):** customizable/pinned shortcuts, keyboard command palette, batch actions with preview and undo where legally/domain allowed, saved views and advanced filters. The feature catalog stays searchable regardless of mode; basic/advanced preferences never change authorization, pricing, posting or audit. Conversion of beginner into professional profile requires no migration of business data.

**Minimal discovery flows (not total commit steps):** sale Today→New Sale (customer→SKU→review); warehouse Today→Receive (PO→quantity→post); finance Today→Record payment (counterparty→allocation→review); visit Today→Next Outlet (check in→outcome→sync); SaaS Wallet Billing summary→Actual usage/Top-up (ledger source, negative month explicitly factual only); onboarding First workspace→Choose task→guided sample/draft. For errors: stable operation ID delegated to domain, retain draft, show authoritative pending/accepted/rejected/conflict; payment/provider pending does not mean settled, dispatch and invoice stock/GL lifecycle follows Sales/Finance owners, mobile local queue is not server confirmation.

### 17.7 Desktop × Mobile × Locale × Accessibility × Network acceptance matrix

| Dimension | Proposed coverage | Observable pass / failure |
|---|---|---|
| Desktop web | 1280×800 and 1920×1080 candidate viewports; keyboard, mouse, 200% zoom | Navigation stays discoverable; focus after dialog/branch switch; no hidden primary task |
| Mobile web/app | 360×800 and tablet 768×1024 candidate viewport; touch + device assistive tech | Task semantics same as desktop, adaptive screen not clipped; no phantom offline posting |
| Locale | Public uz/en/ru every core workflow; future Uzbek alphabet INTERNAL/OFF | Translated labels/errors/date/money readable; identifier stable, fallback visible to QA; forbidden new alphabet not public |
| Accessibility | WCAG 2.2 AA target; keyboard, screen reader, reduced motion, contrast, zoom/reflow, accessible errors | Manual assistive-technology run + automated scan; neither alone implies full compliance |
| Online | Normal and constrained candidate 4G/Wi-Fi, high RTT | Correct skeleton → data and measured p50/p75/p95 time; no stale scope flash |
| Offline/degraded | No network, intermittent response, resumed connection, expired grants | Queued/accepted separated, idempotent retry, revoked action rejected without cross-tenant data |
| Error/recovery | 403, 404 privacy-safe, 409 conflict, 429, 5xx, request timeout, duplicate retry | Draft preserved where safe, clear retry/resolve, no duplicate money/stock/charge |
| Large data | Candidate 20k orders and 100k products, 2 tenants/3 branches | Server paging/filter and permission-safe counts, no huge full client table assumption |

**Performance goals (all PROPOSED, not empirical):** discovery/start of frequent workflow ≤3 deliberate actions at proposed p75; search/filter p75 ≤1s given an **OPEN** realistic dataset/network load profile; LCP p75 ≤2.5s, INP p75 ≤200ms, CLS p75 ≤0.1. Define p95 and acceptable completion/error rates after pilot design; no blanket desktop/mobile equivalent latency claim. No claims of passed accessibility tests or proven superiority to Sales Doctor.

### 17.8 Business approvals, pilot protocol and gate

**OPEN user decision packet:** (1) rank UX-T01…T20 into MVP must-have / later, including owner of HR/employee search; (2) approve persona×branch scopes and mobile/offline eligible operations; (3) approve real pilot organizations/users, task recruitment and consent/privacy baseline; (4) choose representative devices/networks/locale/a11y profiles and task success thresholds; (5) agree source of comparative Sales Doctor baseline, same task definition and valid observation permissions. Until approval, current T01…T20 is candidate only.

**PROPOSED pilot design:** recruited new and expert users for each priority persona, randomized task order and controlled initial datasets, no real customer confidential records, first-attempt discovery/completion, independent observer logs and consented recordings; report task completion, median/p75 discovery actions and time, errors/backtracks, assistance needed, SUS or comparable measure if approved, mobile offline recovery and accessibility findings. Establish baseline and predefined sample size before results. **No participant testing has been done.**

**Release gate statuses:** G0 scope/role approval = OPEN; G1 authorization and cross-scope negative test = UNTESTED; G2 domain correctness and retry = UNTESTED; G3 mobile offline/revoke = UNTESTED; G4 language/a11y = UNTESTED; G5 pilot and performance evidence = UNTESTED. Audit Round 2’s “documentation fixed” is NOT closure of AR-012 or implementation approval. Global DECISIONS/OPEN_QUESTIONS/REQUIREMENTS/MASTER/CHANGELOG changes are suggested only; none made here.

---

## 18. Delegated MVP UX delivery priorities and verification scope (2026-10-12)

[ADR-010](../adr/ADR-010-mvp-delegated-baseline.md) selects **PROPOSED** task-first role dashboards and phased UX-T01…UX-T20 acceptance fixtures for MVP. Prioritize sale/create/confirm/dispatch, receipt/transfer, Customer/Outlet, payment/AR, company Wallet/verified third-party funding and shared-resource responsibility. Always show source Company/Branch, responsible LegalEntity/Wallet, actual charge vs unpriced candidate, posting vs queued/UNKNOWN offline result; never label provisional Wallet effect as posted. Company switching must clear stale search/count/cache and any previously selected Wallet. Public uz/en/ru; hidden alphabet remains OFF. Better-than-Sales-Doctor is a user-APPROVED quality *goal*, not measured benchmark. Pilot tasks, accessibility WCAG candidate and actual test outcomes NOT RUN; see [MVP backlog](../handoff/MVP_IMPLEMENTATION_BACKLOG.md).
