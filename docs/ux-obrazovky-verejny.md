# UX — Obrazovky plochy A (veřejný portál)

Detailní specifikace obrazovek veřejného portálu a tokenových stránek. Navigaci, routy a mapu oprávnění definuje [ux-navigace.md](ux-navigace.md); sdílené stavy, prázdné obrazovky a validační texty [ux-texty-stavy.md](ux-texty-stavy.md); životní cyklus přihlášky [registration-lifecycle.md](registration-lifecycle.md); pravidla polí [validation.md](validation.md); entity [data-model.md](data-model.md).

## 1. Rámec plochy

- **Publikum:** anonym a držitelé tokenových odkazů; bez přihlášení.
- **Ergonomie:** mobile-first — jednosloupcový layout, karty, sticky CTA, dotykové plochy min. 44 px, žádný horizontální scroll. Na desktopu (≥ 905 px) obsah centrovaný, max. šířka 720 px; výpis akcí smí použít mřížku karet 2–3 sloupce.
- **Horní lišta:** center-aligned top app bar s názvem „Dorostová unie · akce“ a přepínačem světlého/tmavého režimu.
- **Routy:** `/` · `/akce/:slug` · `/akce/:slug/prihlaska` · `/akce/:slug/potvrzeni` · `/stav/:token` · `/schvaleni/:token` · `/nabidka/:token`.
- **Token je klíč k jedné přihlášce**, ne nouzová varianta — je rovnocenný přihlášenému účtu ([ux-navigace.md](ux-navigace.md) § 3). Tokenové stránky nemají breadcrumbs ani shell.
- Každá mutace (odeslání přihlášky, schválení zástupcem, nahrání dokumentu, platba, storno) přepočítá stav přihlášky funkcí `evaluate()` v pořadí bran zástupce → dokumenty → platba ([registration-lifecycle.md](registration-lifecycle.md)).

## 2. A-01 · Výpis akcí — `/`

**Účel a publikum:** anonym najde veřejnou akci a získá důvěru v systém během několika vteřin.

**Layout a komponenty:** pod horní lištou hero (nadpis + jedna věta „Přihlašování bez účtu — vše vyřídíte odkazem z e-mailu.“), řada filter chips, seznam elevated karet. Mobil 1 sloupec, desktop mřížka 2–3 sloupce.

**Obsah a pole (napojení na datový model):** zobrazují se jen **veřejné akce (`EVENT.visibility = public`) s otevřeným nebo budoucím přihlašovacím oknem**; proběhlé akce a akce bez přihlašování se nezobrazují. Karta akce: název · pořádající oddíl a město · termín · typ akce (chip) · cena „od …“ nebo „Zdarma“ · přihlašovací okno · **kapacitní chip** (Volno / Plno — lze jako náhradník / Otevíráme dd.mm.). Filtry: oddíl a typ akce; kombinují se AND, aplikují se okamžitě.

**Stavy:**

- **Prázdný:** žádná zobrazitelná akce → prázdný stav „Výpis akcí (portál)“ z [ux-texty-stavy.md](ux-texty-stavy.md) § 4. Filtr bez výsledku → tentýž vzor s CTA „Zrušit filtry“.
- **Načítání:** skeleton 3 karet bez skoku layoutu.
- **Chyba:** obecný chybový vzor [ux-texty-stavy.md](ux-texty-stavy.md) § 5.
- **Úspěch:** klepnutí na kartu → `/akce/:slug`.

**Interakce a validace:** celá karta je dotyková plocha; bez hledání a stránkování (malý počet akcí).

**Oprávnění:** veřejné, bez omezení ([ux-navigace.md](ux-navigace.md) § 2.1).

**Mobil/desktop:** mobil karty přes celou šířku, filter chips ve vodorovném scrollu; desktop mřížka.

**Notifikace:** žádné odchozí ani příchozí.

## 3. A-02 · Detail akce — `/akce/:slug`

**Účel a publikum:** anonym zjistí „co, kdy, za kolik a co k tomu potřebuje“ dřív, než formulář cokoli vyžaduje.

**Layout a komponenty:** top app bar s šipkou zpět a názvem akce; sekce jako karty pod sebou: ① hlavička, ② ceník, ③ volitelné položky, ④ povinné dokumenty, ⑤ storno podmínky, ⑥ o oddílu. Dole sticky lišta s celkovým CTA.

**Obsah a pole:**

- ① Hlavička: název, kapacitní chip, termín, místo, pořádající oddíl, přihlašovací okno, kapacita („N míst + M náhradníků“).
- ② Ceník: tabulka typ účastníka × období platnosti; základní cena `non_DU` musí pokrýt celé okno ([validation.md](validation.md)). Poznámka „Výsledná cena = cena dle typu účastníka + příplatky zvolených položek. Členství v DU se posuzuje k roku akce.“
- ③ Volitelné položky (`EVENT_FIELD`) s příplatky, jen náhled.
- ④ Povinné dokumenty: seznam s formátem a limitem velikosti; text „Dokumenty nahrajete po odeslání přihlášky, schvaluje je vedoucí.“
- ⑤ Storno podmínky: termínovaná procenta.
- ⑥ O oddílu: název, město, region.

**Stavy:**

- **Prázdný:** akce bez povinných dokumentů → sekci ④ nahradí řádek „Tato akce nevyžaduje žádné dokumenty.“ Akce bez volitelných položek → sekce ③ skrytá.
- **Načítání:** skeleton hlavičky + 2 karet.
- **Chyba:** neexistující slug → „Akci jsme nenašli“, CTA „Zpět na výpis akcí“.
- **Úspěch / varianty CTA:** otevřeno a volno → „Přihlásit na akci“; plno → „Přihlásit se jako náhradník“; ještě neotevřeno → disabled „Přihlašování otevřeme dd.mm.“; po uzávěrce → disabled „Přihlašování skončilo“.

**Interakce a validace:** CTA vede na `/akce/:slug/prihlaska` (u plné akce s příznakem náhradníka). Sdílení kopíruje URL detailu + snackbar „Odkaz zkopírován“.

**Oprávnění:** veřejná akce vždy; neveřejná jen přes platný `share_slug`, jinak 404 ([ux-navigace.md](ux-navigace.md) § 2.1).

**Mobil/desktop:** ceník scrolluje ve vlastním kontejneru; sticky CTA vždy viditelné.

**Notifikace:** žádné.

## 4. A-03 · Registrační formulář — `/akce/:slug/prihlaska`

**Účel a publikum:** sebrat přesně to, co akce žádá, spočítat skutečnou cenu před odesláním a umožnit přihlásit i sourozence najednou.

**Layout a komponenty:** stránka s opakovatelnými bloky účastníků (sbalitelné karty); pod bloky tlačítko „Přidat dalšího účastníka“; nahoře společný kontakt, dole sticky souhrn ceny s tlačítkem „Odeslat přihlášku“.

**Obsah a pole:**

- Společné: **kontaktní e-mail** (`REGISTRATION.contact_email`) — na něj chodí potvrzení a tokenový odkaz. U plné akce je nad souhrnem výrazně uvedeno: „Místo není jisté. Uvolní-li se, vedoucí vybere náhradníka a pošleme vám nabídku na 48 hodin.“
- Per účastník: jméno* · příjmení* · **datum narození\*** · volitelné číselníky akce s příplatkem v popisku · **e-mail zákonného zástupce\*** jen když z data narození plyne nezletilost, s textem „Zástupce musí přihlášku potvrdit e-mailem. Do potvrzení není místo rezervované.“ **Typ účastníka se nezadává** — systém ho odvodí podle pořadí pravidel ([validation.md](validation.md#ceny-a-storna): role v pořádajícím oddílu, vazba na vedoucího, DU členství, vazba na oddíl) a rovnou dopočítá výslednou cenu; formulář ho zobrazí jen jako výsledek v souhrnu ceny, nikdy jako volbu k výběru.
- Cena účastníka se přepočítává živě při každé volbě.

**Stavy:**

- **Prázdný:** přímý vstup mimo přihlašovací okno → prázdný stav, ikona `event_busy`, „Přihlašování není otevřené“, CTA „Zpět na detail akce“.
- **Načítání:** skeleton polí prvního bloku.
- **Chyba:** validační chyby inline pod polem; při odeslání souhrn nahoře + scroll k první chybě; u vícečlenné přihlášky chyba jmenuje účastníka.
- **Úspěch/odesílání:** tlačítko se zamkne + progress, pak redirect na potvrzení; dvojité odeslání není možné.

**Interakce a validace:** povinná pole dle hvězdiček; e-maily formátem ([validation.md](validation.md)); datum narození povinné. Odebrat účastníka lze (min. 1 blok zůstává). Při odeslání vzniká pro každého účastníka samostatná přihláška s vlastním VS a společným tokenem; u plné akce jako náhradník ve stavu `New` se zamčenými branami.

**Oprávnění:** anonym; podáním se stává držitelem tokenu vzniklých přihlášek.

**Mobil/desktop:** správné klávesnice a autofill; na mobilu otevřený vždy jen jeden blok účastníka.

**Notifikace:** po odeslání `EMAIL_REG_CONFIRM` (potvrzení); u nezletilého bez zákonného zástupce navíc zákonnému zástupci `EMAIL_GUARDIAN_REQUEST`; nezletilému `EMAIL_REG_CONFIRM_MINOR`, má-li doručovací kontakt ([notifications.md](notifications.md)).

## 5. A-04 · Potvrzení po odeslání — `/akce/:slug/potvrzeni`

**Účel a publikum:** přeložit vypočtený stav do krátkého seznamu „co bude dál“ a předat odkaz pro správu přihlášky.

**Layout a komponenty:** ikona `check_circle`, nadpis „Přihláška odeslána“, karta per účastník, karta „Správa přihlášky“.

**Obsah a pole:** karta účastníka: jméno · stavový chip · rozpad ceny · VS · splatnost · **text dle stavu** (`PendingGuardian` / `PendingDocuments` / `PendingPayment` / `Paid` / náhradník) dle [ux-texty-stavy.md](ux-texty-stavy.md) § 1. Karta „Správa přihlášky“: readonly URL `/stav/:token` s kopírováním a textem „Odkaz platí do konce akce. Přes něj nahrajete dokumenty, zaplatíte i stornujete.“ Pořadí ve frontě náhradníků se neukazuje.

**Stavy:**

- **Prázdný:** přímý vstup bez čerstvého odeslání → ikona `inbox`, „Tady nic není“, CTA „Zpět na detail akce“.
- **Načítání / chyba:** nevzniká. **Úspěch:** podstata obrazovky.

**Interakce a validace:** kopírování odkazu → snackbar „Odkaz zkopírován“.

**Oprávnění:** držitel právě vzniklého tokenu.

**Mobil/desktop:** jeden sloupec; URL pole se zalamuje.

**Notifikace:** navazuje na e-maily odeslané z A-03.

## 6. A-05 · Schválení zákonným zástupcem — `/schvaleni/:token`

**Účel a publikum:** rodič ověří a schválí přihlášku dítěte do minuty a pochopí, že schválením vzniká jeho trvalá vazba na dítě.

**Layout a komponenty:** jedna karta na výšku obrazovky telefonu: hlavička, souhrn, primární filled tlačítko schválení a méně výrazná akce zamítnutí (text/outlined). Bez navigace a přihlášení.

**Obsah a pole:** kdo se přihlásil, na jakou akci a za kolik; kdo přihlášku podal; lhůta jako konkrétní datum. Text „Potvrzením schvalujete účast a získáváte zástupcovský přístup k přihláškám tohoto dítěte.“ Primární CTA „Schválit přihlášku“, sekundární „Zamítnout přihlášku“. Zamítnutí je vědomé rozhodnutí zástupce — přihláška skončí `Canceled` (ne `Expired`) a vazba na dítě nevzniká ([parent-child-lifecycle.md](parent-child-lifecycle.md), [registration-lifecycle.md](registration-lifecycle.md)).

**Stavy:**

- **Platný token:** výše; blíží-li se konec lhůty, banner v `error-container` s upozorněním.
- **Úspěch:** potvrzení „Schváleno — děkujeme“, vznik vazby zástupce a `guardian_approved_at`, `evaluate()`, tlačítko „Zobrazit stav přihlášky“.
- **Zamítnutí:** akce „Zamítnout přihlášku“ otevře potvrzení s **volitelným** polem důvodu (krátký text); po potvrzení stav `Canceled`, `guardian_rejected_at` a případný důvod, token se zneplatní, vazba nevzniká; hláška „Přihlášku jste zamítli.“ Bez CTA ke schválení.
- **Už schváleno:** idempotentně „Tato přihláška už je schválená.“ + odkaz na stav.
- **Už zamítnuto:** idempotentně „Tuto přihlášku jste už zamítli.“ Bez CTA.
- **Lhůta vypršela:** přihláška `Expired` — „Lhůta pro schválení uplynula.“ CTA „Otevřít detail akce“.
- **Mezitím stornováno:** „Tuto přihlášku už není co schvalovat.“ Bez CTA; vazba nevzniká.
- **Neplatný token:** sdílený vzor § 9. **Načítání:** skeleton karty.

**Interakce a validace:** dvě akce — schválení na jeden klik, zamítnutí přes potvrzovací krok s volitelným důvodem (guard proti omylu). Obojí lze jen ve stavu `PendingGuardian` a v rámci lhůty; po kterékoli akci se token zneplatní a tlačítka zmizí.

**Oprávnění:** výhradně držitel schvalovacího tokenu; token opravňuje jen ke schválení nebo zamítnutí jedné přihlášky ([ux-navigace.md](ux-navigace.md) § 2.1).

**Mobil/desktop:** celý obsah na jednu obrazovku bez scrollu.

**Notifikace:** po schválení potvrzení schválení, po zamítnutí notifikace účastníkovi (`EMAIL_GUARDIAN_REJECTED`) ([notifications.md](notifications.md)).

## 7. A-06 · Rozcestník správy přihlášky — `/stav/:token`

**Účel a publikum:** jedna stránka, která vždy odpoví: jaký je stav, která brána blokuje a co je další krok. Centrální obrazovka plochy A; **tatáž sdílená komponenta** jako `/muj-ucet/prihlasky/:pid` a admin detail přihlášky ([ux-texty-stavy.md](ux-texty-stavy.md)).

**Layout a komponenty:** hlavička (jméno účastníka, akce, termín, stavový chip); checklist bran (zástupce → dokumenty → platba) u běžící přihlášky; u náhradníka pouze informace, že se dokumenty a platba odemknou po přijetí nabídky místa; karty **Účastníci** (jen u vícečlenné přihlášky), **Dokumenty**, **Platba**, **Akce s přihláškou**.

**Obsah a pole:**

- **Účastníci:** řádek per dílčí přihláška z jednoho tokenu; klepnutí přepne rozcestník na účastníka. Stavy sourozenců se legálně rozcházejí — žádný souhrnný stav skupiny.
- **Dokumenty:** řádek per povinný dokument se stavem (Čeká na nahrání / Čeká na posouzení / Schválen / Zamítnut, [ux-texty-stavy.md](ux-texty-stavy.md) § 3); u zamítnutého komentář posuzovatele a „Nahrát znovu“. Kontrola formátu a velikosti před odesláním; „nahráno“ ≠ „schváleno“.
- **Platba:** cena s rozpadem, přijaté platby, „Zbývá zaplatit“, splatnost, **QR na zbývající částku** + údaje (účet, částka, VS, SS, zpráva) s kopírováním po řádcích. Upozornění „Plaťte přesnou částku — platby se párují na korunu přesně“ ([payment-matching.md](payment-matching.md)).
- **Akce s přihláškou:** „Přidat dalšího účastníka“ (→ formulář A-03 pro tutéž akci) · „Stornovat přihlášku“ → dialog storna s náhledem poplatku k dnešku, textem o nevratnosti a povinným potvrzením.

**Stavy:** obrazovka zvládá všech devět stavů životního cyklu ([ux-texty-stavy.md](ux-texty-stavy.md) § 1): `Paid`, `PendingGuardian` (dokumenty a platba zamčené), `PendingDocuments`, `PendingPayment`, `PartialPaid`, `Overpayment`, `Canceled`/`Expired` (jen ke čtení + „Přihlásit znovu“, dokud je okno otevřené), náhradník (dokumenty a platba se zobrazí jako odemknutelné až po přijetí nabídky). Prázdné/načítací/chybové vzory dle [ux-texty-stavy.md](ux-texty-stavy.md) § 4–5.

**Interakce a validace:** každá mutace → snackbar + `evaluate()` + překreslení chipu a checklistu. Zamítnutí dokumentu vrací přihlášku do `PendingDocuments` i ze `Paid`; obrazovka to vysvětlí „Platba se nikam neztratila, jen je znovu potřeba dokument.“

**Oprávnění:** držitel tokenu jen nad touto přihláškou; jiný token → 403.

**Mobil/desktop:** kopírovací tlačítka u každého platebního údaje; nahrávání dokumentů z fotoaparátu; desktop karty ve dvou sloupcích.

**Notifikace:** připomínky platby `EMAIL_PAYMENT_REMINDER`, potvrzení platby `EMAIL_PAYMENT_CONFIRMATION`, storno `EMAIL_REGISTRATION_CANCELED` ([notifications.md](notifications.md)).

## 8. A-07 · Nabídka místa náhradníkovi — `/nabidka/:token`

**Účel a publikum:** proměnit uvolněné místo v potvrzeného účastníka dřív, než 48hodinová nabídka propadne.

**Layout a komponenty:** jedna karta: souhrn, termín platnosti, jedno filled tlačítko.

**Obsah a pole:** která akce a účastník; platnost výrazně jako konkrétní datum a čas; co přijetí odemkne (platba, případné dokumenty); věta o nečinnosti „Když nabídku nepřijmete, nic se neruší — zůstáváte na čekací listině.“ CTA „Přijmout místo“.

**Stavy:**

- **Platná:** výše. **Úspěch:** guard „místo stále volné“ → kategorie účastník, odemčení bran, `evaluate()`, redirect na `/stav/:token` se snackbarem o zbývající platbě.
- **Propadlá:** „Platnost nabídky vypršela — zůstáváte na čekací listině.“ Bez CTA.
- **Místo už není volné:** „Místo mezitím obsadil někdo jiný.“
- **Už přijato:** idempotentně + CTA „Zobrazit stav přihlášky“.
- **Neplatný token:** § 9. **Načítání:** skeleton karty.

**Interakce a validace:** jediná akce, po úspěchu nedostupná znovu.

**Oprávnění:** držitel tokenu nabídky.

**Mobil/desktop:** jedna obrazovka bez scrollu.

**Notifikace:** přijetí → potvrzení a odemčená platba; navazuje na nabídku náhradníkovi a `EMAIL_SUBSTITUTE_OFFER_DECLINED` při odmítnutí ([notifications.md](notifications.md)).

## 9. A-08 · Sdílené stavové stránky tokenů

Jedna komponenta pro všechny tokenové routy ([ux-texty-stavy.md](ux-texty-stavy.md) § 4–5):

- **Neplatný token** (ikona `link_off`): „Odkaz není platný“ — „Zkontrolujte, že jste odkaz z e-mailu zkopírovali celý. Pokud problém trvá, ozvěte se vedoucímu oddílu.“ Bez CTA.
- **Token po konci akce** (ikona `history`, jen `/stav/:token`): „Platnost odkazu skončila“ — „Odkaz platí do konce akce. Akce už proběhla.“ Bez CTA.
