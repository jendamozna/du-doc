# 05 · Plocha D — Self-management

Specifikace obrazovek sekce **Můj účet** (`/muj-ucet`) pro DEMO. Rámec, M3 tokeny, konvence prázdných/loading/chybových stavů a životní cyklus přihlášky definuje `00-projekt-a-design.md` (**00**); data pocházejí z mock store dle `01-demo-data.md` (**01** — entity plochy D zejména v § 2.5–2.7, § 11 a § 12). **Žádný backend, žádná reálná auth** — každá mutace mění store, UI se přepočítá (vč. `evaluate()`, 00 § 6).

**Publikum a ergonomie:** člověk s účtem — typicky rodič spravující děti, občas dospělý účastník. **Mobile-first** (stejná ergonomie jako plocha A: jeden sloupec, karty, dotykové plochy ≥ 44 px), navštěvuje se z e-mailových připomínek.

**Návrhový princip — zrcadlo tokenového rozcestníku:** správa přihlášky tokenovým odkazem (plocha A) je plnohodnotný režim, ne nouzovka. Detail přihlášky v „Moje přihlášky“ je proto **tatáž obrazovka jako rozcestník `/stav/:token`** (`02-verejny-portal.md` § 2.6, dále **02 § 2.6**) — jedna specifikace, dvě přístupové cesty. Účet přidává jen to, co token nedá: seznam **všech** přihlášek vlastních i dětí (token je vázán na jedinou přihlášku). Tento soubor rozcestník **nekopíruje** — popisuje jen odchylky.

**Demo persona:** přepínač role (00 § 3.3) položka **„Pavla Konvalinková — rodič s účtem“ (osoba-010)** — účet `ucet-010` (heslo + Google), děti Anežka (osoba-020), Vojtěch (osoba-021) a zletilá Barbora (osoba-033, vazba jen pro čtení). Přihlášky dětí pokrývají `Paid`, `PendingPayment`, náhradníka s běžící nabídkou i proběhlou akci. Přepnutí role jen změní personu ve store — žádné heslo; vysvětlivka „Demo režim — role se přepíná bez přihlášení“.

**Navigace a routy:** `/muj-ucet` přesměruje na `/muj-ucet/prihlasky`. Na mobilu **M3 navigation bar** (dole) se čtyřmi položkami, na desktopu tytéž položky jako navigation rail vlevo; obsah max ~720 px na střed:

| Položka | Ikona | Routa |
|---|---|---|
| Přihlášky | assignment | `/muj-ucet/prihlasky` · detail `/muj-ucet/prihlasky/:pid` |
| Děti | family_restroom | `/muj-ucet/deti` · detail `/muj-ucet/deti/:id` |
| Údaje | badge | `/muj-ucet/udaje` |
| Účet | lock | `/muj-ucet/ucet` · sloučení `/muj-ucet/ucet/slouceni` |

Mimo shell existuje veřejná tokenová stránka `/pozvanka/:token` (přijetí pozvánky druhého zástupce, D-05) — patří logicky sem, technicky se chová jako tokenové stránky plochy A. Top app bar: titulek sekce, vpravo přepínač role. Badge na položce Přihlášky = počet přihlášek vyžadujících akci uživatele (výchozí **2**: prihlaska-226, prihlaska-202).

---

## D-01 · Moje přihlášky — `/muj-ucet/prihlasky`

**Účel:** Jedna odpověď na otázku „co je s našimi přihláškami a co je teď na mně“ — bez hledání tokenových e-mailů.

**Layout a M3 komponenty:** filter chips nahoře: **Aktivní** (výchozí) · **Vše**, dále chip za osobu (**Anežka · Vojtěch · Já**). Pod nimi karty přihlášek (elevated) seskupené nadpisy (title-medium): **„Je to na vás“ · „Nadcházející“ · „Proběhlé“**. Primární akce jako filled button přímo na kartě.

**Obsah a pole (z 01 § 5 a § 2.7):** karta = účastník (avatar s iniciálami + „za: [jméno]“), akce, termín, **stavový chip** (00 § 4.2) s blokující bránou, kategorie (účastník/dobrovolník/náhradník), u platebních stavů **zbývající částka a splatnost**. Výchozí obsah pro personu:

1. **Je to na vás:** prihlaska-226 — Vojtěch, Závod Stezka — Podzimní stopa, Sobota 10.10., chip `Nová` + badge „Náhradník č. 1“, zvýrazněný řádek „**Nabídka místa platí do Čtvrtek 27.8. 14:00**“, CTA **„Odpovědět na nabídku“** → `/nabidka/tok-nabidka-226` (02 § 2.7). · prihlaska-202 — Vojtěch, Podzimní víkendovka Skalní mlýn, 25.–27.9., chip `Čeká na platbu`, „Zbývá 970 Kč · splatnost Pátek 4.9.“, CTA **„Zaplatit“** → D-02 (platební sekce).
2. **Nadcházející:** prihlaska-201 — Anežka, víkendovka, `Zaplaceno`. · prihlaska-220 — Anežka, Závod Stezka, `Zaplaceno`.
3. **Proběhlé** (jen ve filtru Vše): prihlaska-240 — Anežka, Letní tábor Stříbrná zátoka, 11.–25.7., `Zaplaceno`.

Řazení uvnitř skupin: nejbližší lhůta první. Terminální přihlášky (`Canceled`/`Expired`) se zobrazují ve „Vše“ šedě, jen ke čtení.

**Stavy:**
- **Prázdný (účet bez přihlášek):** ikona `assignment` v `secondary-container` kruhu, nadpis **„Zatím žádné přihlášky“**, text „Až se na akci přihlásíte vy nebo vaše děti, uvidíte tu stav, platby a dokumenty na jednom místě.“, CTA **„Prohlédnout akce“** → `/`.
- **Prázdný po filtru** (chip „Já“ — Pavla vlastní přihlášku nemá): ikona `search_off`, nadpis **„Žádné vlastní přihlášky“**, text „Vy sama zatím na žádnou akci přihlášená nejste. Přihlášky dětí najdete pod jejich jmény.“, CTA **„Zrušit filtr“**.
- **Vše hotovo** (skupina „Je to na vás“ prázdná, jiné přihlášky existují): místo skupiny klidný řádek s ikonou `check_circle` — „Nic nečeká na váš krok. Přihlášky níže vyřizují jiní, nebo jsou hotové.“ — žádná falešná urgence.
- **Načítání:** skeleton karet (stabilní výška, bez skoků). **Chyba:** vzor 00 § 5.

**Interakce a validace:** klepnutí na kartu → D-02; CTA na kartě vede rovnou na relevantní sekci detailu; žádné mutace přímo ze seznamu.

**Mobil/desktop:** mobil karty na celou šíři; desktop tentýž jeden sloupec (max ~720 px) — hustota se nemění.

---

## D-02 · Detail přihlášky — `/muj-ucet/prihlasky/:pid` (zrcadlo rozcestníku)

**Účel:** Aktuální stav, blokující brána, další krok — **identické s rozcestníkem 02 § 2.6** (checklist bran, dokumenty, platba s QR/VS/SS, storno s náhledem poplatku). Zde jen odchylky pro přihlášeného uživatele.

**Layout a M3 komponenty:** stejná komponenta jako `/stav/:token`, vykreslená uvnitř shellu Můj účet (top app bar se šipkou zpět na D-01, navigation bar zůstává).

**Obsah a pole — odchylky oproti tokenové cestě:**
- Nad hlavičkou banner (tonal, `secondary-container`): **„Jednáte za: Vojtěch Konvalinka“** — rodičovská práva jsou per dítě a UI to připomíná. U vlastní přihlášky banner není.
- Navigace zpět na seznam a **přepínač mezi přihláškami téhož účtu** (šipky ‹ › v hlavičce, pořadí dle D-01).
- **Žádná výzva k založení účtu** — účet už existuje (demo ji ostatně nezobrazuje ani na tokenové cestě, 02 § 3).
- Referenční data: `prihlaska-202` (brána platby: QR, účet 2900123456/2010, 970 Kč, VS 26102202, SS 2026102, splatnost Pátek 4.9.), `prihlaska-226` (náhradník — brány se zámkem „Odemkne se po přijetí nabídky místa“ + odkaz na `/nabidka/tok-nabidka-226`), `prihlaska-201` (vše zelené).

**Stavy:** shodné s 02 § 2.6 (všech devět stavů vč. pádu zpět po zamítnutém dokumentu) — **neduplikovat, jedna specifikace, dva vstupy**. Navíc: **cizí přihláška v URL** (pid mimo účet a vazby) → chybový vzor 00 § 5 s textem „Tato přihláška nepatří k vašemu účtu.“ bez dalších detailů.

**Interakce a validace:** shodné s 02 § 2.6 (mutace → `evaluate()` → snackbar). Simulace platby a nahrání dokumentu fungují stejně jako z tokenu — je to táž obrazovka.

**Mobil/desktop:** dle 02 § 2.6; sticky CTA na mobilu zachováno, navigation bar pod ním.

---

## D-03 · Moje děti — `/muj-ucet/deti`

**Účel:** Rozcestník po dětech: kdo má co rozdělaného, koho se týká pozvánka druhého zástupce či zletilost.

**Layout a M3 komponenty:** karty dětí (filled) pod sebou; na kartě avatar s iniciálami, jméno, věk, počet přihlášek, řádek **druhého zástupce**, stavové chipy.

**Obsah a pole (z 01 § 2.5–2.7 a § 11.1):**
- **Anežka Konvalinková (13)** — „3 přihlášky · 2 nadcházející“; druhý zástupce: **Radek Konvalinka — navázán**.
- **Vojtěch Konvalinka (10)** — „2 přihlášky · 2 čekají na váš krok“; zástupci: **Radek Konvalinka — navázán** · chip `tertiary` **„Pozvánka odeslána: jitka.konvalinkova@example.cz“** (pozvanka-901, platí do Pondělí 7.9.).
- **Barbora Konvalinková (18)** — chip **„Jen pro čtení — zletilá“**; podtext „Zastoupení se po 18. narozeninách přepnulo do režimu jen pro čtení.“
- Pod kartami drobná vysvětlivka: „Vazba na dítě vzniká přihlášením na akci nebo schválením jeho přihlášky e-mailem.“

**Stavy:**
- **Prázdný (účet bez vazeb):** ikona `family_restroom`, nadpis **„Zatím tu nejsou žádné děti“**, text „Vazba na dítě vzniká přihlášením dítěte na akci nebo schválením jeho přihlášky — ne ručním přidáním.“, CTA **„Prohlédnout akce“** → `/`. Žádné tlačítko „přidat dítě“ neexistuje.
- **Načítání:** skeleton karet. **Chyba:** vzor 00 § 5.

**Interakce a validace:** klepnutí na kartu → D-04. Pozvat dalšího zástupce a vystoupit z vazby se dělá až v detailu — destruktivní akce mimo dosah palce na přehledu.

**Mobil/desktop:** karty v jednom sloupci vždy.

---

## D-04 · Detail dítěte — `/muj-ucet/deti/:id`

**Účel:** Spravovat údaje a přihlášky jednoho dítěte; obsloužit pozvánku druhého zástupce a hranu zletilosti bez zmatku.

**Layout a M3 komponenty:** svislé karty-sekce: **Údaje** (formulář, outlined text fields) · **Zástupci** (M3 list) · **Přihlášky** (karty jako D-01) · dole zóna destruktivní akce.

**Obsah a pole (referenční dítě: Vojtěch, osoba-021):**
1. **Údaje:** jméno*, příjmení*, přezdívka, pohlaví, **datum narození 2.9.2016 — jen ke čtení** s poznámkou „Změnu data narození vyřídí vedoucí oddílu — má dopad na schvalování zástupcem a věková pravidla závodů.“ (zvolený default), kontaktní e-mail, adresa trvalého bydliště (Jabloňová 412, 460 01 Liberec), zdravotní pojišťovna (211). Řádek „Oddílová evidence (chytré sloupce)“ se rodiči **nezobrazuje** — v editoru jen šedý řádek s badge **„K rozhodnutí“** (00 § 7): zda rodič dědí viditelnost citlivých sloupců po vlastníkovi účtu dítěte, musí rozhodnout zadavatel; demo volí nezobrazovat.
2. **Zástupci:** „Pavla Konvalinková — vy“ · „Radek Konvalinka — navázán“ · řádek pozvánky (pozvanka-901): chip „Odeslána Pondělí 24.8. 10:12 · platí do Pondělí 7.9.“ + akce **„Poslat znovu“** (snackbar „Demo: e-mail odeslán — pozvánka zástupce“) a **„Zrušit pozvánku“** (dialog; token přestane platit). Tlačítko **„Pozvat dalšího zástupce“** (tonal) → dialog: e-mail*, jméno, text „Vazba vznikne až přijetím pozvánky. Oba zástupci pak mají plná práva; platí poslední zápis.“ Odeslání založí pozvánku ve store (platnost 14 dní — zvolený default), snackbar + odkaz **„Otevřít pozvánku (simulace)“** → D-05.
3. **Přihlášky:** karty prihlaska-202 a prihlaska-226 (obsah jako D-01), detail → D-02. Tlačítko **„Přihlásit na akci“** → portál `/` (formulář plochy A pak předvyplní údaje dítěte).
4. **Vystoupení:** outlined error button **„Zrušit moje zastoupení“** → dialog s dopadem: „Přestanete spravovat údaje a přihlášky Vojtěcha. Zůstane-li bez navázaného zástupce, spravuje ho vedoucí oddílu, dokud se nepřipojí nový. Akce se zaznamená.“ + povinné zaškrtnutí „Rozumím důsledkům“ (zvolený default tření).

**Varianta „jen pro čtení“ (Barbora, osoba-033):** banner `tertiary` „Barbora je zletilá — zastoupení je jen pro čtení. Přístup vám může kdykoli zcela zrušit.“ Všechna pole disabled; **jediné aktivní pole je chybějící kontaktní e-mail** s tlačítkem **„Uložit a poslat výzvu k převzetí účtu“** (zvolený default: výzvu odesílá rodič tlačítkem) → snackbar „Demo: e-mail odeslán — výzva k převzetí účtu“, pole se zamkne. Sekce Přihlášky prázdná (viz níže), vystoupení dostupné.

**Stavy:**
- **Prázdný — přihlášky dítěte:** ikona `assignment`, nadpis **„Žádné přihlášky“**, text „Barbora zatím nemá žádnou přihlášku na akci.“, CTA **„Přihlásit na akci“** (jen u aktivní vazby; u readonly bez CTA).
- **Úspěch:** uložení údajů → snackbar „Údaje uloženy“ (s akcí Zpět); vystoupení → snackbar, dítě zmizí z D-03, přihlášky dítěte zmizí z D-01.
- **Načítání:** skeleton formuláře. **Chyba:** vzor 00 § 5; formulářové chyby inline.

**Interakce a validace:** křestní jméno proti seznamu českých jmen — neznámé jméno neblokuje, jen informační hláška „Jméno není v seznamu českých jmen — výjimku potvrdí vedoucí oddílu.“; e-mail formát; adresa s autofill atributy. Každá mutace → store + snackbar.

**Mobil/desktop:** formulář s viditelnými popisky a správnými klávesnicemi (e-mail, číslo pojišťovny); na mobilu dialogy jako bottom sheet.

---

## D-05 · Pozvánka druhého zástupce — `/pozvanka/:token` (veřejná stránka)

**Účel:** Druhý rodič — často bez účtu, na telefonu — pochopí, co přijímá, a přijme na jeden krok. Stejná „studená“ situace jako schvalující zástupce (02 § 2.5).

**Layout a M3 komponenty:** jednosloupcová tokenová stránka bez navigace shellu: karta s kontextem, karta s vysvětlením práv, formulář, sticky CTA **„Přijmout pozvánku“**.

**Obsah a pole (referenční token `tok-pozvanka-901`):** „**Pavla Konvalinková** vás zve jako zákonného zástupce dítěte **Vojtěch Konvalinka (10)**, Oddíl Severka.“ Vysvětlení: „Přijetím vznikne vazba zástupce — budete spravovat údaje a přihlášky dítěte stejně jako zvoucí rodič. Platí poslední zápis.“ Formulář: jméno a příjmení* (předvyplněno „Jitka Konvalinková“), e-mail (jen ke čtení), checkbox* „Jsem zákonný zástupce tohoto dítěte“. Platnost do Pondělí 7.9.

**Stavy:**
- **Platná pozvánka:** výchozí obsah.
- **Úspěch:** latence ~300 ms → ikona `check_circle`, „Přijato — děkujeme“, text „Vazba zástupce byla vytvořena. Údaje a přihlášky dítěte teď spravujete i vy.“; ve store vznikne osoba-034 + vazba `active`; snackbar „Demo: e-mail odeslán — potvrzení přijetí“. Chip na D-03/D-04 se změní na „navázán“.
- **Již přijato:** idempotentně a přátelsky — „Tuto pozvánku jste už přijali. Vše je nastaveno.“ (žádná chyba).
- **Vypršelá / zrušená:** ikona `schedule`, nadpis **„Pozvánka už neplatí“**, text „Platnost vypršela, nebo byla pozvánka zrušena. Požádejte zvoucího rodiče o novou.“, bez CTA.
- **Neplatný token:** chybový vzor 00 § 5 bez detailů.

**Interakce a validace:** checkbox povinný; jméno povinné. Přijetí je jediná mutace.

**Mobil/desktop:** mobile-first jako celá plocha A; na desktopu úzký sloupec na střed.

---

## D-06 · Moje údaje a souhlasy — `/muj-ucet/udaje`

**Účel:** Jedno místo pro údaje vlastní osoby a přehled souhlasů se zpracováním — navštěvované zřídka, typicky po změně bydliště či pojišťovny.

**Layout a M3 komponenty:** dvě karty-sekce: **Moje údaje** (formulář) a **Souhlasy** (M3 list).

**Obsah a pole (osoba-010 dle 01 § 2.5, souhlasy dle 01 § 11.2):**
1. **Moje údaje:** jméno* Pavla, příjmení* Konvalinková, přezdívka —, tituly před/za —, pohlaví žena, **datum narození 7.9.1985 — jen ke čtení** (stejný default a poznámka jako D-04), kontaktní e-mail pavla.konvalinkova@example.cz, adresa Jabloňová 412, 460 01 Liberec, zdravotní pojišťovna 111. Drobný text: „Vyplňuje se jen to, co akce potřebují.“
2. **Souhlasy:** řádek = typ, účel, uděleno kdy, stav. souhlas-921 „Zpracování osobních údajů — členská evidence a přihlášky · uděleno 12.3.2024 · platí“ (chip `Paid` paleta „Platí“). souhlas-922 „Zasílání novinek e-mailem · uděleno 12.3.2024 · **odvolán 5.1.**“ (chip `Canceled` paleta „Odvolán“) + podtext „Záznam se uchovává ještě 4 roky po odvolání.“ U platného souhlasu akce **„Odvolat“** (zvolený default: self-service) → dialog s dopadem a potvrzením; odvolání zapíše `revoked_at` do store, snackbar. **Žádost o výmaz** demo neřeší tlačítkem — pod seznamem jen text „O výmaz údajů (GDPR) požádejte vedoucího oddílu.“ (zvolený default).

**Stavy:**
- **Prázdný — souhlasy** (v demu nenastane, definovaný vzhled): ikona `fact_check`, nadpis **„Žádné souhlasy“**, text „Souhlasy se objeví po udělení — například při přihlášce na akci.“, bez CTA.
- **Úspěch:** nenápadný snackbar „Údaje uloženy“ (s akcí Zpět). **Chyba:** inline u pole; vzor 00 § 5 pro celou stránku.

**Interakce a validace:** stejná pravidla jmen jako D-04; e-mail formát; autofill adresy. Uložení nemění nic jiného ve store (žádné přepočty přihlášek).

**Mobil/desktop:** formulářová pravidla ploch A/D; na desktopu stále jeden sloupec.

---

## D-07 · Účet a zabezpečení — `/muj-ucet/ucet`

**Účel:** Přihlašovací metody pod kontrolou uživatele; nic tu nesmí umožnit zamknout sám sebe ven. V demu **vše mock** — heslo ani OAuth nejsou reálné.

**Layout a M3 komponenty:** karty-sekce: **Přihlášení** · **Propojené účty** · **Sloučení duplicit** (vstup do D-08). Nahoře nenápadný banner „Demo režim — přihlášení se pouze simuluje.“

**Obsah a pole (ucet-010):**
1. **Přihlášení:** přihlašovací e-mail pavla.konvalinkova@example.cz — jen ke čtení, podtext „Změnu e-mailu vyřídí podpora.“ (zvolený default). Tlačítko **„Změnit heslo“** → dialog: staré heslo*, nové heslo* 2× (min. 8 znaků — zvolený default politiky); v demu projde libovolné „staré“ heslo, jen validace nového. Úspěch: snackbar „Heslo změněno“ + „Demo: e-mail odeslán — potvrzení změny hesla“.
2. **Propojené účty:** řádek **Google — propojen** (pavla.konvalinkova@example.cz) s akcí **„Odpojit“**; řádek **Facebook — nepropojen** s akcí **„Propojit“** → dialog „Demo režim — propojení OAuth se simuluje“ → potvrzení změní stav ve store + snackbar. **Pravidlo poslední cesty:** odpojit lze, jen zbývá-li jiný způsob přihlášení — u poslední metody je tlačítko zamčené (disabled + ikona `lock`) s vysvětlením „Jediný způsob přihlášení nelze odpojit — nejdřív nastavte heslo nebo připojte jiný účet.“ Poznámka pod sekcí: „Propojení přes e-mail se nabízí až po přihlášení heslem — automatické propojení by umožnilo převzetí účtu.“ (jen text, v demu nenastává).
3. **Sloučení duplicit:** karta s obsahem dle stavu — výchozí: **nabídka kandidáta** „Nejste to vy? V Oddílu Jestřábi je evidována Pavla Konvalinková se shodným datem narození.“ + tlačítko **„Zobrazit návrh“** → D-08. Po odmítnutí/dokončení viz prázdný stav níže.

Sekce „Zrušení přístupu rodiče“ (pro zletilé) se u persony nezobrazuje — Pavla žádného rodiče navázaného nemá; pravidlo připomíná jen text u Barbory v D-04.

**Stavy:**
- **Prázdný — sloučení duplicit** (po dokončení či odmítnutí návrhu): ikona `person_search`, nadpis **„Žádný návrh na sloučení“**, text „Nenašli jsme jiný záznam, který by mohl být váš. Kdyby se objevil, uvidíte ho tady — nic se nesloučí bez vašeho souhlasu.“, bez CTA.
- **Úspěch:** snackbary u každé mutace. **Chyba:** vzor 00 § 5; formulář hesla inline (neshoda nových hesel, málo znaků).

**Interakce a validace:** viz výše; všechny akce dostupné jen personě Rodič s účtem. Throttling, zamykání účtu a jednotné chybové hlášky přihlášení demo nezobrazuje (není login) — nestavět.

**Mobil/desktop:** podpora správců hesel (autocomplete atributy); přepínače a tlačítka se 44px cíli.

---

## D-08 · Žádost o sloučení duplicit — `/muj-ucet/ucet/slouceni`

**Účel:** Delikátní moment souhlasu: srozumitelně nabídnout „tohle jste možná vy v jiném oddíle“, provést žádostí a volbou polí — a nic neslíbit dřív, než souhlasí všechny strany. Kandidáti se **jen navrhují, nikdy neslučují automaticky**.

**Layout a M3 komponenty:** krokový obsah řízený stavem žádosti: (1) nabídka kandidáta — dvě porovnávací karty, (2) stav žádosti — M3 list stran se stavovými ikonami, (3) volba polí A/B — pole po poli dvě porovnatelné karty (ne tabulka), (4) shrnutí a dokončení.

**Obsah a pole (kandidát osoba-064, rezervovaná žádost zadost-941):**
- **Nabídka:** karta „Váš záznam“ (osoba-010: Oddíl Severka, nar. 7.9.1985, Liberec) vedle karty „Nalezený záznam“ (osoba-064: host, Oddíl Jestřábi, nar. 7.9.1985, Olomouc); důvod: „shodné jméno a datum narození“. Dopady: „Sloučením se přenesou všechny vazby (přihlášky, docházka, členství). Citlivá data zůstávají per oddíl — vedoucí Jestřábů nezíská přístup k datům ze Severky.“ Akce: **„Požádat o sloučení“** (filled) a **„Toto nejsem já“** (text button) — před potvrzením odmítnutí dialog: „Dvojice se trvale potlačí a návrh se už nezobrazí. Povolit ji může jen administrátor ústředí.“
- **Po podání — `pending`:** seznam stran: „Vy (iniciátorka) — souhlas ✓“ · „Věra Kropáčková, HVO Oddílu Jestřábi — čeká“ (kandidát účet nemá, rozhoduje za něj HVO); text „Bez odezvy žádost propadá po 30 dnech.“ Snackbary: „Žádost odeslána“ + „Demo: e-mail odeslán — žádost o sloučení (2 příjemci)“. Demo tonal button **„Simulovat souhlas všech stran“** (vysvětlivka „jen v demu“) → `ready`.
- **`ready` — volba polí A/B:** konfliktní pole po jednom — **adresa** (Jabloňová 412, Liberec × Wolkerova 18, Olomouc) a **kontaktní e-mail** (pavla.konvalinkova@example.cz × pavla.k@example.cz); u každého dvě karty, výběr právě jedné, **žádný ruční přepis** (kvůli věrnému revertu — oprava až po sloučení běžnou editací). Prázdné pole prohrává automaticky (pojišťovna — bere se 111, šedě „bez konfliktu“). Různá data narození by vyvolala varování (v demu shodná). Účet má jen jedna strana — volba účtu se nezobrazuje.
- **Dokončení:** shrnutí → **„Dokončit sloučení“** → ve store zanikne osoba-064, zvolené hodnoty se zapíší, stav `completed`; potvrzení „Záznamy jsou sloučeny“ + snackbar „Demo: e-mail odeslán — potvrzení sloučení“. Text: „Sloučení může vrátit jen administrátor ústředí.“

**Stavy:**
- **`pending`:** viz výše (ukázat, na koho se čeká). **`rejected`:** kdokoli zamítl nebo 30 dní bez odezvy — klidné sdělení, dvojice potlačena, bez CTA. **`completed`** / **`reverted`:** jen ke čtení.
- **Zablokováno kolizí** (definovaný vzhled, v demu nenastává): banner `error-container` „Oba záznamy mají aktivní přihlášku na téže akci. Nejdřív to vyřeší vedoucí akce — zatím se nic nemění.“
- **Prázdný (bez kandidáta):** shodný s prázdným stavem sekce v D-07.
- **Načítání:** skeleton karet. **Chyba:** vzor 00 § 5.

**Interakce a validace:** dokončit lze jen ze stavu `ready` a jen když jsou všechny konflikty rozhodnuté; celé sloučení je jedna transakce nad store (selže-li část, nezmění se nic). Odmítnutí i dokončení přepnou sekci v D-07 do prázdného stavu.

**Mobil/desktop:** na úzkém displeji jedno konfliktní pole na obrazovku s jasným postupem (krok X z 2); na desktopu pole pod sebou.

---

## Přijaté defaulty

Kde je spec otevřená, demo volí bez přepínačů: detail přihlášky = sdílená obrazovka s tokenovým rozcestníkem; seznam přihlášek segmentují chipy Aktivní/Vše + chip osoby (žádné záložky); datum narození edituje jen vedoucí oddílu; pozvánka zástupce platí 14 dní, lze ji odeslat znovu i zrušit; vystoupení z vazby chrání dialog s dopadem + „Rozumím“; výzvu k převzetí účtu odesílá rodič tlačítkem; odvolání souhlasu self-service, žádost o výmaz přes vedoucího; změna přihlašovacího e-mailu jen přes podporu; heslo min. 8 znaků; nabídka kandidáta na sloučení nevtíravě v sekci Účet (žádný banner po přihlášení); odmítnutá dvojice se potlačuje trvale. Badge **„K rozhodnutí“** (00 § 7) zobraz v UI pouze u viditelnosti chytrých sloupců pro rodiče (D-04) — jinde defaulty nekomentuj.

---

## Doplnění demo dat

Demo entity plochy D jsou sloučené přímo v `01-demo-data.md`: osoby osoba-033, osoba-034 a osoba-064 vč. rozšířených polí osob 010, 020 a 021 (§ 2.5) · uživatelské účty (§ 2.6) · vazby vazba-801 až vazba-806 (§ 2.7) · pozvanka-901, souhlasy souhlas-921 a souhlas-922, žádost zadost-941 (§ 11) · token `tok-pozvanka-901` (§ 12).
