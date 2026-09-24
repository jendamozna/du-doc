# 02 · Plocha A — Veřejný portál

Specifikace obrazovek veřejného portálu demo aplikace. Rámec, design tokeny, konvence prázdných/loading/chybových stavů a životní cyklus přihlášky definuje `00-projekt-a-design.md`; veškerá data (ID akcí, osob, přihlášek, tokenů) definuje `01-demo-data.md` — na obojí se tu jen odkazuje.

## 1. Rámec plochy A

- **Mobile-first, bez přihlášení.** Jednosloupcový layout, karty, sticky CTA, dotykové plochy min. 44 px, stránka nikdy nescrolluje horizontálně. Na desktopu (≥ 905 px) obsah centrovaný, max. šířka 720 px (výpis akcí smí použít mřížku karet 2–3 sloupce).
- **Horní lišta** (center-aligned top app bar): logo/název „Dorostová unie · akce“, vpravo přepínač role a toggle tmavého režimu (viz `00-projekt-a-design.md` § 3.3, § 4).
- **Routy:** `/` · `/akce/:slug` · `/akce/:slug/prihlaska` · `/akce/:slug/potvrzeni` · `/stav/:token` · `/schvaleni/:token` · `/nabidka/:token`.
- **Slugy akcí** (mapuj na ID z `01-demo-data.md` § 4): `akce-101` → `letni-tabor-stribrna-zatoka` · `akce-102` → `podzimni-vikendovka-skalni-mlyn` · `akce-103` → `zavod-stezka-podzimni-stopa` · `akce-104` → bez slugu (bez přihlášek, na portálu se nezobrazuje) · `akce-105` → `drakiada-na-kopci-vetrnik` · `akce-106` → `vanocni-dilny`.
- **Demo bez backendu:** všechny akce nad in-memory store; odeslání přihlášky, schválení zástupcem, nahrání dokumentu, simulace platby i storno mění store a stav přihlášky se přepočítá funkcí `evaluate()` (brány zástupce → dokumenty → platba, viz `00` § 6). Kde by šel e-mail, zobraz snackbar „Demo: e-mail odeslán — [šablona]“ a odkaz na příslušnou tokenovou stránku zpřístupni přímo v UI (panel „Demo odkazy“, viz § 2.4).
- **Tokenové odkazy jsou normální cesta správy přihlášky**, ne nouzový režim. Předpřipravené tokeny dema jsou v `01-demo-data.md` § 12; pro přihlášky vytvořené za běhu generuj tokeny `tok-runtime-1`, `tok-runtime-2`, …
- „Dnešek“ dema je **Středa 26.8.** — všechny výpočty lhůt, splatností a storno poplatků počítej k tomuto datu.

## 2. Obrazovky

### 2.1 `/` — Výpis akcí

**Účel:** Najít akci, o které rodiči někdo řekl, a získat důvěru v systém do 90 vteřin.

**Layout a M3 komponenty:** Pod horní lištou hero (display-small „Akce oddílů Dorostové unie“, body-large „Přihlašování na tábory, víkendovky a závody. Bez účtu — vše vyřídíte odkazem z e-mailu.“), řada **filter chips**, seznam **elevated karet**; mobil 1 sloupec, desktop mřížka 2 sloupce.

**Obsah a pole:**
- Zobrazuj jen **veřejné akce s otevřeným nebo budoucím přihlašovacím oknem**; proběhlé akce a akce bez přihlášek se nezobrazují. Ve výchozích demo datech tedy 4 karty v pořadí dle začátku akce: **akce-102, akce-103, akce-105, akce-106**.
- Karta akce: název (title-large) · pořádající oddíl + město · termín (`25.–27.9.`, u jednodenních `Sobota 17.10.`) · typ akce (assist chip, `secondary-container`) · „cena od 850 Kč“ (u akce-105 „Zdarma“) · přihlašovací okno („Přihlašování do Pátek 18.9.“) · **kapacitní chip**: „Volno“ (paleta `Paid`), „Plno — lze se hlásit jako náhradník“ (paleta `PendingDocuments`), „Otevíráme Neděle 1.11.“ (paleta `New`). Akce-103 má „Plno…“, akce-106 „Otevíráme…“.
- Filter chips: **Oddíl** (Severka / Jestřábi) a **Typ** (Víkendovky / Stezka) + vpravo outlined chip **„K rozhodnutí“** (`tertiary`) s tooltipem „Rozsah filtrů výpisu není finálně rozhodnut.“

**Stavy:**
- **Prázdný:** žádná zobrazitelná akce → kanonický stav „Výpis akcí (portál)“ z `00` § 5. Filtr bez výsledku → kanonický stav „Výpis akcí — filtr“ s CTA „Zrušit filtry“.
- **Načítání:** skeleton 3 karet (obdélník + 3 řádky), žádný skok layoutu.
- **Chyba:** obecný chybový vzor z `00` § 5 (v demu prakticky nenastane).
- **Úspěch:** klepnutí na kartu → `/akce/:slug`.

**Interakce a validace:** Celá karta je dotyková plocha. Filtry se kombinují (AND), aplikují se okamžitě. Žádné hledání ani stránkování (zvolený default — malý počet akcí).

**Mobil/desktop:** Mobil: karty přes celou šířku, filter chips ve vodorovném scrollu. Desktop: mřížka, hero užší.

### 2.2 `/akce/:slug` — Detail akce

**Účel:** Odpovědět „co, kdy, za kolik a co k tomu budu potřebovat“ dřív, než formulář cokoli chce. Referenční obsah: **akce-102** (`01-demo-data.md` § 4).

**Layout a M3 komponenty:** Top app bar s šipkou zpět a názvem akce. Sekce jako filled karty pod sebou: ① hlavička, ② ceník, ③ volitelné položky, ④ povinné dokumenty, ⑤ storno podmínky, ⑥ o oddílu. Dole **sticky spodní lišta** (elevace 2) s celkovým CTA.

**Obsah a pole:**
- ① Hlavička: název (headline-small), kapacitní chip (jako 2.1), termín, místo („Skalní mlýn, Jizerské hory“ — jen název bez mapy, zvolený default), pořádající oddíl, přihlašovací okno, kapacita („24 míst + 4 náhradníci“).
- ② Ceník: tabulka typ účastníka × období, částky `850 Kč`. U akce-102 jedno období (člen DU 850 · bez DU 950 · vedoucí/dobrovolník 400); u akce-101 dvě období (segmented button „do 31.5. / od 1.6.“); u akce-105 jen řádek „Zdarma“. Pod tabulkou body-medium: „Výsledná cena = cena dle typu účastníka + příplatky zvolených položek. Členství v DU se posuzuje k roku akce.“
- ③ Volitelné položky (číselníky s příplatky, jen náhled): *Strava* — běžná +0 Kč · bezlepková +50 Kč; *Doprava* — společný autobus +120 Kč · vlastní +0 Kč.
- ④ Povinné dokumenty: list s ikonou `description` — u akce-102 „Posudek o zdravotní způsobilosti (PDF/JPG/PNG/HEIC, max 10 MB)“; text „Dokumenty nahrajete po odeslání přihlášky, schvaluje je vedoucí.“
- ⑤ Storno podmínky: „do Pátek 11.9. zdarma · 12.–18.9. 50 % ceny · později 100 %“ + outlined chip **„K rozhodnutí“** (umístění storno podmínek před závazkem není finální; v demu se zobrazují zde — zvolený default).
- ⑥ O oddílu: název, město, region.

**Stavy:**
- **Prázdný:** akce bez povinných dokumentů (akce-103, 105) → sekci ④ nahraď řádkem s ikonou `task_alt`: „Tato akce nevyžaduje žádné dokumenty.“ Akce bez volitelných položek → sekci ③ skryj.
- **Načítání:** skeleton hlavičky + 2 karet.
- **Chyba:** neexistující slug → chybový vzor, nadpis „Akci jsme nenašli“, text „Odkaz je neplatný, nebo akce už není dostupná.“, CTA „Zpět na výpis akcí“.
- **Varianty CTA (stavy okna/kapacity):** otevřeno + volno → filled „Přihlásit na akci“; **plno** (akce-103) → filled „Přihlásit se jako náhradník“ + text „Kapacita je plná. Náhradníci dostávají nabídku e-mailem, platí 48 hodin.“; **ještě neotevřeno** (akce-106) → disabled tlačítko „Přihlašování otevřeme Neděle 1.11.“; **po uzávěrce / proběhlá** → disabled „Přihlašování skončilo“.

**Interakce a validace:** CTA vede na `/akce/:slug/prihlaska` (u plné akce s příznakem náhradníka). Sdílecí odkaz (ikona `share` v top app baru) zkopíruje URL detailu + snackbar „Odkaz zkopírován“.

**Mobil/desktop:** Ceník scrolluje uvnitř vlastního kontejneru. Sticky CTA vždy viditelné bez scrollování.

### 2.3 `/akce/:slug/prihlaska` — Registrační formulář (vícečlenná přihláška)

**Účel:** Sebrat přesně to, co akce žádá, spočítat skutečnou cenu před odesláním a zvládnout přihlásit i sourozence najednou.

**Layout a M3 komponenty:** Jedna stránka s **opakovatelnými bloky účastníků** (outlined karty, sbalitelné) — zvolený default místo stepperu. Blok se po vyplnění jména přejmenuje na jméno účastníka. Pod bloky tonal button „Přidat dalšího účastníka“. Nahoře pole společného kontaktu, dole **sticky souhrn ceny** (surface-container, elevace 2): cena per účastník + „Celkem 1 940 Kč“ + filled button „Odeslat přihlášku“.

**Obsah a pole:**
- Společné: **Kontaktní e-mail** (outlined text field, `type=email`) — na něj „chodí“ potvrzení a tokenový odkaz.
- Per účastník (outlined text fields, popisky nad poli): Jméno* · Příjmení* · **Datum narození*** (datumové pole) · **Typ účastníka*** (radio dle ceníku: člen DU / bez DU / vedoucí či dobrovolník) · číselníky akce jako radio skupiny s příplatkem v popisku („Bezlepková (+50 Kč)“) — u akce-102 *Strava* a *Doprava* povinné před odesláním · **E-mail zákonného zástupce*** — jen když z data narození vyplývá nezletilost; supporting text: „Zástupce musí přihlášku potvrdit e-mailem do 7 dnů. Do potvrzení místo není rezervované.“
- Cena účastníka se přepočítává živě při každé volbě.

**Stavy:**
- **Prázdný:** formulář má vždy 1 blok — prázdný stav nastává jen při zavřeném přihlašovacím okně (přímý vstup na URL): místo formuláře vzor prázdného stavu, ikona `event_busy`, nadpis „Přihlašování není otevřené“, text „U této akce teď nejde podat přihlášku. Podívejte se na detail akce, kdy se otevře.“, CTA „Zpět na detail akce“.
- **Načítání:** skeleton polí prvního bloku.
- **Chyba:** validační chyby inline pod polem (supporting text v `error`); při odeslání s více chybami souhrn nahoře + scroll k první chybě; u vícečlenné přihlášky chyba vždy jmenuje účastníka („Vojtěch: chybí datum narození“).
- **Úspěch/odesílání:** tlačítko se zamkne + circular progress („Odesílám…“), latence ~400 ms, pak redirect na potvrzení. Dvojité odeslání nesmí být možné.

**Interakce a validace:** Povinná pole dle hvězdiček; e-maily formátem; datum narození povinné (zvolený default — scénář „chybí datum narození“ demonstruje prihlaska-209 z demo dat, ne formulář); whitelist křestních jmen se v demu nevaliduje (zvolený default). Odebrat účastníka lze ikonou `delete` v hlavičce bloku (min. 1 blok zůstává). **Odeslání (mock):** pro každého účastníka vznikne ve store samostatná přihláška s vlastním VS (pokračuj řadou `26102213`, `26102214`, …), vlastním stavem přes `evaluate()` a společným tokenem `tok-runtime-N`; u plné akce (akce-103) vzniká jako náhradník ve stavu `New` se zamčenými branami. Snackbar „Demo: e-mail odeslán — Potvrzení přihlášky“.

**Mobil/desktop:** Správné klávesnice (`type=email`, datumové pole, `inputmode=numeric`), zapnutý autofill, 44px dotykové cíle u radio položek. Na mobilu otevřený vždy jen jeden blok účastníka.

### 2.4 `/akce/:slug/potvrzeni` — Potvrzení po odeslání

**Účel:** Přeložit vypočtený stav do krátkého poctivého seznamu „co bude dál“ a předat odkaz pro správu přihlášky.

**Layout a M3 komponenty:** Ikona `check_circle` v `primary-container` kruhu, headline-small „Přihláška odeslána“. Pod tím karta per účastník, pak karta „Správa přihlášky“ a panel „Demo odkazy“.

**Obsah a pole:**
- Karta účastníka: jméno · stavový chip (`00` § 4.2) · rekapitulace ceny s rozpadem („850 Kč + autobus 120 Kč = 970 Kč“) · VS · splatnost („do Středa 9.9.“) · **text dle stavu**: `PendingGuardian` → „Zákonnému zástupci jsme poslali e-mail na …. Musí přihlášku potvrdit do [datum]. Do potvrzení místo není rezervované.“; `PendingDocuments` → „Nahrajte posudek o zdravotní způsobilosti — hned, nebo později odkazem níže.“; `PendingPayment` → „Zaplaťte přesnou částku QR platbou — údaje najdete v odkazu níže i v e-mailu.“; `Paid` (akce zdarma) → „Hotovo, místo je potvrzené. Akce je zdarma.“; náhradník → „Jste na čekací listině. Uvolní-li se místo, dostanete e-mailem nabídku platnou 48 hodin.“ Pořadí ve frontě náhradníků se **neukazuje** (zvolený default — vedoucí vybírá ručně, žádný slib pořadí).
- Karta „Správa přihlášky“: „Přihlášku spravujete tímto odkazem — uložte si ho.“ + readonly pole s URL `/stav/tok-runtime-N` + ikona kopírování. Text: „Odkaz platí do konce akce. Přes něj nahrajete dokumenty, zaplatíte i stornujete.“
- **Panel „Demo odkazy“** (outlined karta, chip „Demo“): tlačítka „Otevřít správu přihlášky“ → `/stav/tok-runtime-N` a — je-li některý účastník `PendingGuardian` — „Otevřít e-mail zástupce (simulace)“ → `/schvaleni/tok-runtime-N-z`. Vysvětlivka: „V ostrém provozu chodí tyto odkazy e-mailem.“

**Stavy:**
- **Prázdný:** přímý vstup bez čerstvého odeslání → ikona `inbox`, nadpis „Tady nic není“, text „Potvrzení se zobrazí hned po odeslání přihlášky.“, CTA „Zpět na detail akce“.
- **Načítání:** není (data už jsou ve store). **Chyba:** nenastává. **Úspěch:** je podstatou obrazovky.

**Interakce a validace:** Kopírování odkazu → snackbar „Odkaz zkopírován“. Nabídka založení účtu se v demu nezobrazuje (mimo rozsah, viz § 3).

**Mobil/desktop:** Vše v jednom sloupci; URL pole se zalamuje, nikdy nerozbíjí šířku stránky.

### 2.5 `/schvaleni/:token` — Schválení zákonným zástupcem

**Účel:** Umožnit rodiči, který o systému nikdy neslyšel, ověřit a schválit přihlášku dítěte do minuty — a pochopit, že schválením vzniká jeho trvalá vazba na dítě.

**Layout a M3 komponenty:** Jedna karta na výšku jedné obrazovky telefonu: hlavička, souhrn, jedno filled tlačítko. Žádná navigace, žádné přihlašování.

**Obsah a pole (referenční data `tok-schvaleni-205`, `01-demo-data.md` § 12):**
- „Matyáš Brázdil se přihlásil na akci **Podzimní víkendovka Skalní mlýn** (25.–27.9., Oddíl Severka, cena 850 Kč).“ Kdo přihlášku podal; lhůta jako konkrétní datum: „Potvrďte prosím do **Pondělí 31.8.**“
- Vysvětlení důsledku (body-medium): „Potvrzením schvalujete účast a získáváte zástupcovský přístup k přihláškám tohoto dítěte.“
- CTA: filled „Schválit přihlášku“. Akce „zamítnout“ neexistuje (zvolený default — žádost zástupce buď schválí, nebo propadne).

**Stavy:**
- **Platný token** — výše. **Urgentní** (`tok-schvaleni-206`): navíc banner v `error-container` s ikonou `schedule`: „Pozor, lhůta vyprší už zítra (Čtvrtek 27.8.).“
- **Úspěch:** po klepnutí latence ~300 ms → obrazovka se změní na potvrzení: ikona `check_circle`, „Schváleno — děkujeme“, text „Vazba zákonného zástupce byla vytvořena. Přihláška pokračuje: [další brána — např. čeká na dokumenty].“ + tlačítko „Zobrazit stav přihlášky“ (→ rozcestník). Snackbar „Demo: e-mail odeslán — Potvrzení schválení“. Ve store: `guardian_approved_at`, vazba rodič↔dítě, `evaluate()`.
- **Už schváleno** (opakované otevření): idempotentní, přátelské — „Tato přihláška už je schválená. Není potřeba nic dělat.“ + odkaz na stav.
- **Lhůta vypršela:** přihláška `Expired` — „Lhůta pro schválení uplynula a přihláška vypršela. Dokud je přihlašování otevřené, můžete dítě přihlásit znovu.“ CTA „Otevřít detail akce“.
- **Přihláška mezitím stornována:** „Tuto přihlášku už není co schvalovat — byla stornována.“ Bez CTA. Žádná vazba nevzniká.
- **Prázdný / neplatný token:** sdílený vzor § 2.8.
- **Načítání:** skeleton karty.

**Interakce a validace:** Jediná akce; po úspěchu tlačítko zmizí (žádné dvojité schválení).

**Mobil/desktop:** Celý obsah na jednu obrazovku telefonu bez scrollu.

### 2.6 `/stav/:token` — Rozcestník správy přihlášky

**Účel:** Jedna stránka, která vždy odpoví: jaký je stav, která brána blokuje a co je další krok. Centrální obrazovka plochy A.

**Layout a M3 komponenty:** Hlavička (kdo + jaká akce — rodič se třemi otevřenými tokeny je nesmí zaměnit): jméno účastníka (headline-small), akce + termín, **stavový chip** dle `00` § 4.2. Pod ní **checklist bran** (vertikální M3 list, zástupce → dokumenty → platba; ikony `check_circle` splněno / `radio_button_unchecked` aktuální / `lock` zamčeno; neuplatněné brány se vynechávají). Dále karty: **Účastníci** (jen u vícečlenné přihlášky), **Dokumenty**, **Platba**, **Akce s přihláškou**. Referenční data: `tok-demo-rodic` (prihlaska-201, vše hotovo), `tok-stav-204` (prihlaska-204, zamítnutý dokument).

**Karta Účastníci** (vícečlenná přihláška z jednoho tokenu): řádek per dílčí přihláška — jméno, stavový chip, zbývající platba; klepnutí přepne rozcestník na daného účastníka (segmented look). Stavy sourozenců se legálně rozcházejí — žádný souhrnný stav skupiny se nepředstírá.

**Karta Dokumenty:** řádek per povinný dokument: název, stavový chip („Čeká na nahrání“ paleta `New` / „Čeká na posouzení“ paleta `PendingDocuments` / „Schválen“ paleta `Paid` / „Zamítnut“ paleta `Canceled`), název souboru + velikost, náhled jako šedý placeholder rámeček (`01` § 6). U zamítnutého viditelný komentář posuzovatele v `error-container` (u `tok-stav-204`: „Sken je nečitelný a chybí razítko lékaře. Nahrajte prosím čitelnou kopii.“) + filled tonal „Nahrát znovu“. Nahrání: file picker (accept PDF/JPG/PNG/HEIC, na mobilu i fotoaparát), kontrola 10 MB před odesláním, progress, pak stav „Čeká na posouzení“ + snackbar „Dokument nahrán — posoudí ho vedoucí“. Rozdíl „nahráno“ ≠ „schváleno“ musí být vidět.

**Karta Platba:** cena s rozpadem · seznam přijatých plateb (datum, částka) · **„Zbývá zaplatit: 970 Kč“** (title-medium) · splatnost · **QR blok**: statický placeholder QR (šedý čtverec s popiskem „QR platba“, rezervované místo — žádný skok layoutu) + údaje s ikonou kopírování po řádcích: účet `2900123456/2010`, částka, VS, SS, zpráva „DU – Podzimní víkendovka Skalní mlýn – Vojtěch Konvalinka“. Upozornění (body-medium): „Plaťte prosím přesnou částku — platby se párují na korunu přesně.“ Po částečné platbě nese QR **zbývající částku** (zvolený default). **Demo panel** (outlined, chip „Demo“): tlačítka „Simulovat přesnou platbu“ a „Simulovat částečnou platbu (50 %)“ — vytvoří ve store spárovanou transakci, `evaluate()`, snackbar „Demo: platba přijata a spárována“.

**Karta Akce s přihláškou:** tonal button „Přidat dalšího účastníka“ (→ formulář § 2.3 pro tutéž akci; nová dílčí přihláška se objeví v kartě Účastníci) · text button v barvě `error` „Stornovat přihlášku“ → **dialog storna**: náhled poplatku z termínovaných pravidel k dnešku („Storno je nyní zdarma — jste před Pátkem 11.9.“ / „Storno poplatek: 50 % = 485 Kč, případná vratka 485 Kč“), text „Vratku vyplácí účetní oddílu mimo systém. Stornovanou přihlášku nelze obnovit.“, checkbox „Rozumím, že storno je nevratné“ odemyká error-filled tlačítko „Stornovat“. Úspěch: stav `Canceled`, snackbar, karta jen ke čtení + CTA „Přihlásit znovu“ (dokud je okno otevřené).

**Stavy (obrazovka musí zvládnout všech 9 stavů životního cyklu):**
- `Paid` (`tok-demo-rodic`): checklist celý zelený, platba „Zaplaceno — historie plateb“, žádná dlužná částka.
- `PendingGuardian`: dokumenty i platba se zámkem a textem „Odemkne se po schválení zástupcem.“; řádek zástupce ukazuje lhůtu.
- `PendingDocuments` (`tok-stav-204`): blokuje dokument; platba viditelná, ale sekundární.
- `PartialPaid`: „Zaplaceno 500 Kč · zbývá 470 Kč“ — normální mezistav, ne chyba. Po splatnosti zvýrazni termín (`error` text); přihláška se sama neruší.
- `Overpayment`: „Přišlo nám víc, než je cena (přeplatek X Kč). Ozve se vám účetní oddílu — vy nic dělat nemusíte.“
- `Canceled`/`Expired`: jen ke čtení, chip + vysvětlení + „Přihlásit znovu“, dokud je okno otevřené.
- **Náhradník:** banner „Jste na čekací listině“ + brány viditelné, ale zamčené: „Dokumenty a platba se odemknou po přijetí nabídky místa.“
- **Prázdný (sekce):** akce bez dokumentů → kanonický stav „Rozcestník — dokumenty“ z `00` § 5; žádná přijatá platba → v seznamu plateb řádek „Zatím žádná platba nedorazila.“
- **Načítání:** skeleton hlavičky, checklistu a 2 karet. **Chyba / neplatný token / token po konci akce:** § 2.8.

**Interakce a validace:** Každá mutace → snackbar; každá mutace → `evaluate()` a překreslení chipu i checklistu. Zamítnutí dokumentu vedoucím (v ploše B) vrací přihlášku do `PendingDocuments` i ze `Paid` — rozcestník to popisuje klidně: „Platba se nikam neztratila, jen je znovu potřeba dokument.“

**Mobil/desktop:** Kopírovací tlačítka u každého platebního údaje (na jednom zařízení QR nenaskenujete). Nahrávání dokumentů rovnou z fotoaparátu (`capture`). Desktop: karty ve dvou sloupcích (dokumenty + platba vedle sebe).

### 2.7 `/nabidka/:token` — Nabídka místa náhradníkovi

**Účel:** Proměnit uvolněné místo v potvrzeného účastníka dřív, než 48hodinová nabídka propadne.

**Layout a M3 komponenty:** Jedna karta jako u § 2.5: souhrn, termín platnosti, jedno filled tlačítko.

**Obsah a pole (referenční data `tok-nabidka-226`):**
- „Uvolnilo se místo na akci **Závod Stezka — Podzimní stopa** (Sobota 10.10.) pro účastníka **Vojtěch Konvalinka**.“
- Platnost výrazně: chip s ikonou `schedule` „Nabídka platí do **Čtvrtek 27.8. 14:00**“ (konkrétní datum a čas, žádný odpočet — zvolený default).
- Co přijetí odemkne: „Přijetím se stanete účastníkem — odemkne se platba 150 Kč (splatnost může být ihned) a případné dokumenty.“
- Poctivá věta o nečinnosti: „Když nabídku nepřijmete, nic se neruší — zůstáváte na čekací listině.“
- CTA: filled „Přijmout místo“. Akce „odmítnout“ se nenabízí (zvolený default).

**Stavy:**
- **Platná** — výše. **Úspěch:** přijetí (guard: místo stále volné) → kategorie účastník, odemčení bran, `evaluate()`, redirect na `/stav/:token` se snackbarem „Místo je vaše — zbývá zaplatit 150 Kč“.
- **Propadlá:** „Platnost nabídky vypršela. Zůstáváte na čekací listině — uvolní-li se místo znovu, můžete dostat další nabídku.“ Bez CTA.
- **Místo už není volné:** „Místo mezitím obsadil někdo jiný. Zůstáváte na čekací listině.“
- **Už přijato:** idempotentní — „Nabídku jste už přijali.“ + CTA „Zobrazit stav přihlášky“.
- **Prázdný / neplatný token:** § 2.8. **Načítání:** skeleton karty.

**Interakce a validace:** Jediná akce, po úspěchu nedostupná znovu.

**Mobil/desktop:** Jedna obrazovka telefonu bez scrollu.

### 2.8 Sdílené stavové stránky tokenů

Jedna komponenta pro všechny tokenové routy, vzor prázdného/chybového stavu z `00` § 5:

- **Neplatný token** (ikona `link_off`): nadpis „Odkaz není platný“, text „Zkontrolujte, že jste odkaz z e-mailu zkopírovali celý. Pokud problém trvá, ozvěte se vedoucímu oddílu.“ Bez CTA.
- **Token po konci akce** (ikona `history`, jen `/stav/:token`): nadpis „Platnost odkazu skončila“, text „Odkaz pro správu přihlášky platí do konce akce. Akce už proběhla — s dotazy se obraťte na vedoucí oddílu.“ Bez CTA.
- Ztracený odkaz demo neřeší (bez funkce „znovu poslat odkaz“ — zvolený default; hlavní supportní scénář budoucna).

## 3. Flow plochy A mimo rozsah dema

Tyto části na portálu existují jen zkráceně, nebo vůbec — demo na ně nesmí stavět navigaci navíc:

- **Dobrovolnická přihláška (samostatná stránka):** v demu není; typ účastníka „vedoucí či dobrovolník“ se volí přímo v registračním formuláři (odtud dobrovolník Břetislav Okurka, prihlaska-212 za 400 Kč). Odkaz na detailu akce se nezobrazuje.
- **Skupinové akce (typ „Skupinové“):** v demo datech žádná není; vícečlennost pokrývá formulář § 2.3 a karta Účastníci v rozcestníku. Samostatný skupinový formulář se nestaví.
- **Založení účtu a přihlášení (vč. OAuth):** demo nemá reálnou autentizaci — nahrazuje ji přepínač role (`00` § 3.3). Registrace účtu, login ani obnova hesla se nestaví; nabídky „založte si účet“ se nezobrazují.
- **Výběr běhů workshopů** a **potvrzení doporučení mentora e-mailem:** nejsou v rozsahu dema.
- **Sestavení závodních hlídek účastníkem (portál):** není v rozsahu dema; hlídky akce-103 spravuje plocha B. Prázdná sekce hlídek by užila kanonický stav „Hlídky závodu“ z `00` § 5.

## 4. Kontrolní scénáře k proklikání

Po sestavení musí projít bez zásahu do kódu:

1. `/` → akce-102 → formulář: 2 účastníci (jeden nezletilý s e-mailem zástupce) → potvrzení se 2 kartami různých stavů → „Otevřít e-mail zástupce (simulace)“ → schválit → rozcestník: nahrát posudek → simulovat přesnou platbu — stav správně zůstává `PendingDocuments`, dokud posudek neschválí plocha B.
2. `tok-stav-204`: zamítnutý posudek s komentářem → nahrát znovu → „Čeká na posouzení“.
3. `tok-schvaleni-205` schválit · `tok-schvaleni-206` urgentní banner.
4. Akce-103 (plno) → přihlásit jako náhradník → text čekací listiny; `tok-nabidka-226` → přijmout → rozcestník s odemčenou platbou 150 Kč.
5. `tok-demo-rodic` → storno dialog (zdarma, před 11.9.) → `Canceled` → „Přihlásit znovu“.
6. `/stav/nesmysl` → „Odkaz není platný“.
