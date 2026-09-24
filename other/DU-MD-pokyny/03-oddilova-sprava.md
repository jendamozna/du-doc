# 03 · Plocha B — Oddílová správa

Specifikace obrazovek oddílové administrace pro DEMO. Rámec, navigaci, M3 tokeny a konvence prázdných/loading/chybových stavů definuje `00-projekt-a-design.md` (**00**); data pocházejí z mock store dle `01-demo-data.md` (**01** — plochu B ukazuje **oddil-01 Oddíl Severka** a jeho akce, přihlášky, dokumenty, platby, družiny a docházka). **Žádný backend, žádná reálná auth** — každá mutace mění in-memory store a UI se přepočítá (vč. `evaluate()`, 00 § 6).

**Publikum a ergonomie:** vracející se vedoucí — **HVO, VO, RÁD, ÚČE**; **desktop-first** — husté tabulky s filtry scrollují ve vlastním kontejneru, na mobilu se hroutí do karet. Zápis docházky je výjimka: **výhradně mobilní** návrh (tábořiště, jedna ruka).

**Persony (přepínač role, 00 § 3.3):**

- **„Martin Dvořáček — HVO oddílu Severka“ (osoba-001, `ucet-001`)** — plný rozsah, přistane na `/oddil`; v top app baru „Oddíl Severka (HVO)“. Hlavní persona plochy B.
- **„Ivana Šmídková — účetní“ (osoba-005, `ucet-005`)** — omezený rozsah: Přehled, Akce jen ke čtení, **Platby**; přistane na `/oddil/platby`.

**Navigace (`/oddil/...`, M3 rail dle 00 § 3.2):** **Přehled** (`/oddil`) · **Akce** (`/oddil/akce`, detail `/oddil/akce/:id`) · **Platby** (`/oddil/platby`, jen HVO a ÚČE) · **Osoby** (`/oddil/osoby`) · **Družiny a závody** (`/oddil/druziny`) · **Reporty** (`/oddil/reporty`) · **Nastavení oddílu** (`/oddil/nastaveni`). Badge na položce **Platby** = počet věcí k zásahu ÚČE (výchozí **3**: 2 nespárované transakce trans-306/307 + 1 přeplatek prihlaska-243). Badge na položce **Akce** není — fronty jsou uvnitř detailu akce.

**Zásady (vynucuj v UI):**

1. **Oprávnění se přidělují per akce** — VO/RÁD vidí a mění přihlášky jen tam, kde jsou přiřazení (`EVENT_ASSIGNMENT`); základní čtení detailu akce a seznamu přihlášených plyne z role ve vlastním oddílu ([authorization.md](../docs/authorization.md)).
2. **Stav přihlášky se nikdy nenastavuje ručně** — vedoucí mění fakta (schválí dokument, alokuje platbu, vybere náhradníka) a stav přepočítá `evaluate()`. V UI není žádný ovladač „nastavit stav“.
3. **Platební atributy se maskují dle role** — RÁD mimo funkci Vedoucího akce nevidí částky; ÚČE vidí platby celého oddílu, ale k akcím se nepřiřazuje.
4. **Chování bez oprávnění:** položka mimo rozsah role se **skryje** (navigace, akce), údaj mimo rozsah (částky pro RÁD) se **maskuje** čárkami `———`, přímý vstup na URL bez práva → obrazovka 403 (ikona `lock`, „Sem nemáte přístup“, CTA „Zpět na přehled“).

---

## B-01 · Přehled oddílu — `/oddil`

**Účel:** Kokpit vedoucího: co se v oddílu děje teď a co čeká na jeho zásah — proklikem o úroveň hlouběji.

**Layout a M3 komponenty:** Top app bar „Přehled — Oddíl Severka“. Řádek **KPI karet** (filled, číslo display-small + popisek), pod ním **„Vyžaduje pozornost“** (M3 list, leading ikona v tonálním kruhu, trailing šipka) a karta **„Nadcházející akce“**.

**Obsah a pole (z 01):**

- KPI: **Členů 21** · **Otevřené akce 2** (akce-102, akce-105 není Severky — tedy jen akce-102 a akce-103 běží; zobraz „2 s otevřeným/plným během“) · **Přihlášek k vyřízení 4** · **Nespárované platby 2**.
- Vyžaduje pozornost (dle urgence): 1. „**Dokument čeká na posouzení** — 2 posudky (akce-102)“ → detail akce-102, tab Dokumenty; 2. „**Nabídka náhradníkovi vyprší zítra** — Vojtěch Konvalinka (akce-103)“ → tab Náhradníci; 3. „**Přeplatek k rozhodnutí** — Teodor Kučeravý +100 Kč (Letní tábor)“ → Platby; 4. „**2 nespárované platby** — celkem 1 370 Kč“ → Platby; 5. „**Přihláška blokovaná chybějícím datem narození** — Sára Mlžná (akce-102)“ → detail přihlášky.
- Nadcházející akce: akce-102 Podzimní víkendovka (25.–27.9., otevřená, 8/24) · akce-103 Závod Stezka (10.10., plno, náhradníci) · akce-106 Vánoční dílny (5.12., otevře se 1.11.).

**Stavy:**

- **Prázdný — „Vyžaduje pozornost“:** ikona `done_all` v `secondary-container` kruhu, nadpis **„Vše vyřízeno“**, text „Žádná přihláška, dokument ani platba nečeká na zásah.“, bez CTA.
- **Persona ÚČE:** KPI a pozornost jen k platbám (nespárované, přeplatek); sekce dokumentů a náhradníků se skryjí.
- **Načítání:** skeletony; **Chyba:** vzor 00 § 5.

**Interakce a validace:** jen proklik, žádné mutace z přehledu.

**Mobil/desktop:** KPI 2×2, seznam pozornosti první.

---

## B-02 · Akce — seznam — `/oddil/akce`

**Účel:** Všechny akce oddílu na jednom místě — stav, obsazenost, co potřebuje pozornost — a založení nové akce ze šablony.

**Layout a M3 komponenty:** datová tabulka (00 § 4.4) + **extended FAB „Nová akce“** (ikona `add`); filter chips `Otevřené` (výchozí) · `Připravované` · `Proběhlé` · `Zrušené`; vyhledávání podle názvu.

**Obsah a pole (z 01 § 4, jen akce oddil-01):**

- Tabulka: **Název · Typ · Termín · Přihlašování · Obsazenost · Stav** (chip). Řádky: **akce-102** Podzimní víkendovka — Víkendovky — 25.–27.9. — 20.8.–18.9. — 8/24 (+ fronta dok/plateb) — chip „Otevřená“ (`Paid` paleta) · **akce-103** Závod Stezka — Stezka — 10.10. — 10.8.–30.9. — 6/6 + 2 náhr. — chip „Plno“ (`PendingDocuments` paleta) · **akce-104** Výlet na Kozí vrch — Jednorázová — 22.8. — bez přihlášek — docházka — chip „Proběhlá“ (`New` paleta) · **akce-106** Vánoční dílny — Víkendovky — 5.12. — od 1.11. — 0/20 — chip „Otevře se 1.11.“ (`New` paleta) · **akce-101** Letní tábor — Víkendovky — 11.–25.7. — proběhlá, dořešení plateb — chip „Proběhlá“.
- Sloupec Obsazenost: „počítáno/kapacita“ (do kapacity jen `PendingDocuments`–`Overpayment`, 00 § 6) + drobně „+N náhr.“. Trailing badge u řádku s frontou (dokumenty k posouzení, nespárované platby).

**Stavy:**

- **Prázdný (žádná akce ve filtru):** ikona `event`, nadpis **„Zatím žádné akce“**, text „Založte první akci ze šablony — přednastaví dokumenty, ceny i kapacitu.“, CTA **„Nová akce“**.
- **Persona ÚČE:** tabulka jen ke čtení, FAB skrytý, sloupec Obsazenost bez fronty dokumentů.
- **Načítání:** skeleton řádků; **Chyba:** vzor 00 § 5.

**Interakce a validace:** klik na řádek → `/oddil/akce/:id` (tab Přihlášky jako výchozí u otevřené, Docházka u proběhlé bez přihlášek). **„Nová akce“** → dialog: výběr šablony (systémové sablona-401/403/404/405 + oddílová sablona-402) → název* · termín* · přihlašovací okno · kapacita — pole přednastaví šablona jako snapshot ([validation.md](../docs/validation.md) → Akce). Placenou akci nelze publikovat bez bankovního účtu — u konceptu bez účtu disabled „Publikovat“ s tooltipem.

**Mobil/desktop:** tabulka → karty (název, chip, obsazenost); FAB plovoucí.

---

## B-03 · Detail akce — `/oddil/akce/:id`

**Účel:** Řídicí panel jedné akce — vše, co k ní patří, v pěti tabech. Referenční akce: **akce-102** (`01` § 4).

**Layout a M3 komponenty:** Top app bar se šipkou zpět, názvem akce a chipem stavu; pod ním **M3 tabs**: **Nastavení · Přihlášky · Dokumenty · Náhradníci · Docházka**. Nad taby řádek rychlých metrik (přihlášeno / volno / k posouzení / nespárováno). Tab se propisuje do URL (`?tab=prihlasky`), aby proklik z přehledu mířil přesně.

**Viditelnost tabů dle typu akce a role:**

- Akce bez přihlášek (akce-104, typ jednorázová): jen tab **Docházka**; ostatní taby skryté.
- Akce se závody (akce-103): tab **Náhradníci** navíc obsahuje sekci hlídek (viz B-08); docházka je u této akce nahrazena výsledky (v demu jen docházka).
- RÁD bez funkce Vedoucího akce: tab **Nastavení** skrytý, v tabu Přihlášky maskované částky.
- ÚČE: vidí jen tab **Přihlášky** ke čtení; platby řeší z `/oddil/platby`.

Jednotlivé taby jsou popsané v B-04 až B-08.

**Stavy / Načítání / Chyba:** neexistující `:id` → chybový vzor, nadpis „Akci jsme nenašli“, CTA „Zpět na seznam akcí“. Ostatní vzor 00 § 5.

**Mobil/desktop:** taby jako scrollovatelný řádek; metriky pod taby v jednom řádku (na mobilu 2×2).

---

## B-04 · Tab Nastavení akce — `/oddil/akce/:id?tab=nastaveni`

**Účel:** Úplná konfigurace akce — od základních údajů po ceník, číselníky, dokumenty a storno pravidla — se stejnou rodinou formulářů, jakou používají systémové šablony (04 · C-07).

**Layout a M3 komponenty:** sbalitelné sekce (M3 expansion), každá s vlastním tlačítkem „Uložit“: ① Základ · ② Přihlašování a kapacita · ③ Ceník · ④ Výběrové číselníky · ⑤ Povinné dokumenty · ⑥ Storno pravidla · ⑦ Publikace.

**Obsah a pole (referenční akce-102):**

- ① Základ: název, typ (ze šablony, needitovatelný), místo konání (výběr z lokací oddílu), popis, SS `2026102` (needitovatelný, generuje systém).
- ② Přihlašování a kapacita: okno od–do, kapacita 24 + 4 náhradníci, viditelnost (veřejná / interní / přes odkaz), sdílecí odkaz s kopírováním.
- ③ Ceník: tabulka typ účastníka × období platnosti (člen DU 850 · bez DU 950 · vedoucí/dobrovolník 400); `non_DU` je základní cena a musí pokrýt celé okno ([validation.md](../docs/validation.md) → Ceny a storna). Přidat období = segmented button.
- ④ Výběrové číselníky: _Strava_ (běžná +0 · bezlepková +50), _Doprava_ (autobus +120 · vlastní +0); u každé položky kapacita a fáze (`on_submit` / `before_event`). Položka s `required_phase = before_event` musí mít nulový příplatek (guard v UI).
- ⑤ Povinné dokumenty: Posudek o zdravotní způsobilosti (PDF/JPG/PNG/HEIC, max 10 MB) + přepínač „umožnit trvalý dokument osoby“.
- ⑥ Storno pravidla: termínovaná procenta (do 11.9. zdarma · 12.–18.9. 50 % · později 100 %); `percent` ∈ ⟨0; 100⟩.
- ⑦ Publikace: přepínač koncept ↔ publikováno; **placenou akci nelze publikovat bez bankovního účtu** — bez účtu disabled s vysvětlením.

**Stavy:**

- **Prázdný:** nová akce z konceptu má sekce předvyplněné ze šablony; číselníky lze mít prázdné.
- **Úspěch:** po uložení sekce snackbar „Uloženo“ + poznámka u ceníku „Změna ceny nepřecení už podané přihlášky.“ ([validation.md](../docs/validation.md)).
- **Chyba (validace):** inline pod polem; publikace bez účtu → banner v `error-container`.
- **Bez práva (VO/RÁD bez `can_edit_event`):** sekce jen ke čtení, tlačítka „Uložit“ skrytá.

**Interakce a validace:** pravidla akce dle [validation.md](../docs/validation.md) → **Akce**, **Ceny a storna**, **Výběrové číselníky**. Snížit kapacitu pod počet započítaných přihlášek nelze; snížit kapacitu položky číselníku pod počet voleb nelze.

**Mobil/desktop:** sekce jako karty pod sebou; ceníková tabulka scrolluje ve vlastním kontejneru.

---

## B-05 · Tab Přihlášky — `/oddil/akce/:id?tab=prihlasky`

**Účel:** Seznam všech přihlášek akce se stavem a stavem úhrady — a rychlá cesta k jednotlivé přihlášce i k hromadným úkonům.

**Layout a M3 komponenty:** datová tabulka s **checkbox výběrem** pro hromadné akce; filter chips podle stavu (`Vše` · `Čeká na akci vedoucího` · `Zaplaceno` · `Náhradníci`); vyhledávání podle jména; nad tabulkou při výběru **kontextová lišta hromadných akcí**.

**Obsah a pole (akce-102, z 01 § 5.1):**

- Sloupce: **Účastník · Stav** (chip 00 § 4.2) **· Cena · Uhrazeno · Zbývá · Splatnost · Kontakt**. Řádky např.: Anežka K. — Zaplaceno — 970 — 970 — 0 — — · Vojtěch K. — Čeká na platbu — 970 — 0 — 970 — 4.9. · Šimon Š. — Čeká na dokumenty — 850 — 0 — 850 — · Rozálie Š. — Čeká na dokumenty — 900 — 0 — 900 — (posudek zamítnut) · Matyáš B. — Čeká na zástupce — 850 — … · Sára M. — Nová — 850 — … („chybí datum narození“, ikona warning).
- **Maskování dle role:** RÁD mimo Vedoucího akce vidí sloupce Cena/Uhrazeno/Zbývá jako `———`; jako Vedoucí akce vidí předepsáno/uhrazeno/zbývá, ne slevy ani storno poplatky ([authorization.md](../docs/authorization.md)).
- Kontextová lišta při výběru: **„Poslat připomínku platby“** (jen na přihlášky ve stavu čekání na platbu) a **„Exportovat výběr“** (CSV) — jiné hromadné akce demo nenabízí (viz Přijaté defaulty).

**Stavy:**

- **Prázdný:** kanonický **„Tabulka přihlášek akce“** z 00 § 5 (ikona `inbox`, „Zatím žádné přihlášky“, CTA „Zkopírovat sdílecí odkaz“).
- **Úspěch (hromadná připomínka):** snackbar „Demo: odesláno N připomínek platby“; do auditu záznam per přihláška.
- **Načítání:** skeleton řádků; **Chyba:** vzor 00 § 5.

**Interakce a validace:** klik na řádek → **B-06 Detail přihlášky**. Připomínka platby jen pro přihlášky s nenulovou zbývající částkou a nastaveným kontaktem. Export = reálné stažení ploché CSV z mock dat.

**Mobil/desktop:** tabulka → karty (účastník, chip, zbývá, splatnost); hromadné akce jako bottom sheet.

---

## B-06 · Detail přihlášky — `/oddil/akce/:id/prihlaska/:pid`

**Účel:** Jedna přihláška z pohledu vedoucího — týž checklist bran a stavů jako tokenový rozcestník (02 § 2.6), rozšířený o akce, které smí jen vedoucí.

**Návrhový princip — zrcadlo rozcestníku:** stavová část (hlavička, checklist bran zástupce → dokumenty → platba, karty Dokumenty a Platba) je **tatáž sdílená komponenta** jako `/stav/:token` a `/muj-ucet/prihlasky/:pid` ([ux-texty-stavy.md](../docs/ux-texty-stavy.md)). Tento soubor popisuje jen **admin odchylky** — co token ani rodič nemá.

**Obsah a pole — admin odchylky (referenční prihlaska-204, zamítnutý posudek):**

- **Posouzení dokumentu:** u nahraného posudku tlačítka **„Schválit“** a **„Zamítnout“**; zamítnutí = dialog s výběrem důvodu (_nečitelné · chybí druhá strana · prošlá platnost · jiný dokument_) + volný text — po odeslání se stane komentářem viditelným rodiči. Schválení/zamítnutí spustí `evaluate()`; zamítnutí povinného posudku vrátí přihlášku do `PendingDocuments` i ze `Paid` (rozcestník to popisuje klidně).
- **Ruční úprava přihlášky (jen HVO, VO s `can_edit_registrations`):** změna typu účastníka / ceny (`can_edit_prices`, loguje se), oprava kontaktního e-mailu, změna volby číselníku. Text „Změna se zapíše do auditu.“
- **Storno vedoucím:** dialog s výpočtem poplatku k dnešku + nepovinný důvod z rychlých voleb (nemoc · jiný program · kolize termínu · jiné); u prihlaska-229 demonstruje storno vedoucím.
- **Chybějící datum narození (prihlaska-209):** banner „Bránu zástupce nelze vyhodnotit — chybí datum narození. Systém si ho vyžádal e-mailem.“ + tlačítko „Doplnit datum narození“ (jen vedoucí; edituje osobu).
- **Platba:** vedoucí vidí historii alokací; ruční párování se dělá z `/oddil/platby` (B-09), ne odtud — jen odkaz „Otevřít v platbách“.

**Stavy:** všech 9 stavů dědí z rozcestníku; navíc **náhradník** — místo platby sekce „Vybrat jako náhradníka“ vede na B-08.

**Interakce a validace:** posouzení dokumentu a storno jen dle role/přiřazení; RÁD dokument neposuzuje. Každá mutace → snackbar + `evaluate()` + zápis do auditu.

**Mobil/desktop:** stejné jako rozcestník; admin akce v `overflow` menu na mobilu, jako tlačítka na desktopu.

---

## B-07 · Tab Dokumenty — fronta posuzování — `/oddil/akce/:id?tab=dokumenty`

**Účel:** Jedno místo, kde vedoucí odbaví všechny nahrané dokumenty akce — rychle a se stejným tónem zamítnutí.

**Layout a M3 komponenty:** fronta karet per dokument (leading náhled — šedý placeholder rámeček z 01 § 6, název souboru, velikost), filter chips `Čeká na posouzení` (výchozí) · `Schválené` · `Zamítnuté`. Na kartě chip stavu (dle B-03 stavů dokumentu) a akční tlačítka.

**Obsah a pole (akce-102, z 01 § 6):**

- Fronta „čeká na posouzení“: **dok-701** posudek Šimon (posudek-simon.pdf, 412 kB, nahráno 24.8. 19:42) · **dok-705** posudek Kryštof (IMG_2481.jpg, 3,4 MB, 25.8. 21:17) — badge „2“.
- Schválené: dok-703 Anežka, dok-704 Vojtěch. Zamítnuté: **dok-702** Rozálie (posudek-rozalie.jpg, 2,1 MB) s viditelným důvodem „Sken je nečitelný a chybí razítko lékaře…“.
- Akce na kartě čekajícího: **„Schválit“** (filled tonal) · **„Zamítnout“** (text, error) → dialog s předvolbami důvodu + volný text. Klik na náhled → větší placeholder v dialogu.

**Stavy:**

- **Prázdný:** kanonický **„Fronta dokumentů“** z 00 § 5 (ikona `done_all`, „Vše posouzeno“).
- **Úspěch:** po posouzení karta zmizí z fronty „čeká“, badge klesne, snackbar „Dokument schválen“ / „Dokument zamítnut — účastník dostane e-mail“; `evaluate()` přepočítá přihlášku.
- **Načítání:** skeleton karet; **Chyba:** vzor 00 § 5.

**Interakce a validace:** posuzuje jen HVO nebo VO s `can_edit_registrations`; RÁD sem nemá zápis. Zamítnutí bez důvodu nelze odeslat. HEIC náhled se v demu nahradí placeholderem (viz Přijaté defaulty).

**Mobil/desktop:** karty v jednom sloupci; na mobilu je primární akcí velké tlačítko, dialog důvodu jako bottom sheet.

---

## B-08 · Tab Náhradníci a hlídky — `/oddil/akce/:id?tab=nahradnici`

**Účel:** Řídit čekací listinu a nabídky uvolněných míst — a u závodní akce sestavené hlídky a stanoviště.

**Layout a M3 komponenty:** sekce **„Náhradníci“** (M3 list v pořadí podání) + u závodní akce sekce **„Hlídky“** a **„Stanoviště“** (karty). U náhradníka chip stavu nabídky.

**Obsah a pole (akce-103, z 01 § 5.2 a § 8):**

- Náhradníci v pořadí: **1. Vojtěch Konvalinka** (prihlaska-226) — chip „Nabídka běží, platí do Čt 27.8. 14:00“ (`PendingPayment` paleta) + akce „Zrušit nabídku“ · **2. Kryštof Hrbáček** (prihlaska-227) — chip „Čeká v pořadí“ (`New` paleta) + akce **„Nabídnout místo“** (aktivní jen když je volné místo). Vysvětlivka: „Pořadí je podle času podání, výběr je na vedoucím — pořadí se účastníkům neukazuje.“
- Guard: současně běžících nabídek nejvýše tolik, kolik je volných míst (v demu 1 volné → druhou nabídku nelze spustit, dokud první neskončí).
- **Hlídky (akce-103):** hlidka-501 Rysové (Stezka, kapitán Matyáš, členové Matyáš/Šimon/Anežka) · hlidka-502 Vydry (Pěšinka, kapitán Amálie, členové Amálie/Klaudie). Konzistence složení dle [race-patrols.md](../docs/race-patrols.md) — věková kategorie, unikátní název v akci. Vedoucí je jen ke čtení upravuje (sestavuje vlastník přihlášky), může řešit nekonzistenci.
- **Stanoviště:** stanoviste-01 Uzly (rozhodčí Klára) · stanoviste-02 Mapa a buzola (Tomáš) · stanoviste-03 První pomoc (**prázdný slot** — „Rozhodčí nepřiřazen“, akce „Přiřadit“).

**Stavy:**

- **Prázdný — náhradníci:** kanonický **„Náhradníci akce“** z 00 § 5 (ikona `group_off`).
- **Prázdný — hlídky:** kanonický **„Hlídky závodu“** z 00 § 5 (ikona `flag`).
- **Úspěch:** po „Nabídnout místo“ → náhradník dostane nabídku (token), chip „Nabídka běží 48 h“, snackbar „Demo: nabídka odeslána“. Po vypršení nabídky se místo vrátí do fronty.
- **Načítání / Chyba:** vzor 00 § 5.

**Interakce a validace:** nabídku spouští jen HVO/VO s `can_edit_registrations`; přijetí/odmítnutí řeší náhradník na `/nabidka/:token` (02 § 2.7). Prázdný slot rozhodčího je platný stav, ne chyba.

**Mobil/desktop:** seznamy v jednom sloupci; hlídky jako karty s kapitánem nahoře.

---

## B-09 · Tab Docházka — `/oddil/akce/:id?tab=dochazka`

**Účel:** Zapsat, kdo dorazil — na tábořišti, na telefonu, jednou rukou. Jediná **výhradně mobilní** obrazovka plochy B.

**Layout a M3 komponenty:** seznam účastníků (nebo členů družiny) s velkými přepínači **Přítomen / Nepřítomen / (nezapsáno)**; segmented button pro filtr podle družiny; sticky souhrn „Přítomno 7 / 9“. Na desktopu tentýž seznam v užším sloupci — návrh se ale optimalizuje pro dotyk.

**Obsah a pole (akce-104 Výlet na Kozí vrch, z 01 § 8):**

- Přítomni: osoba-020, 021, 022, 025, 026, 028, 029. Nepřítomni: osoba-024, 027. Ostatní bez záznamu. Vedl osoba-003, zapsáno So 22.8. 17:40.
- Každý řádek: jméno, družina (Lišky/Sovy), třístavový přepínač (výchozí „nezapsáno“ — šedý). Nezapsaný ≠ nepřítomný.

**Stavy:**

- **Prázdný:** kanonický **„Docházka“** z 00 § 5 (ikona `checklist`, „Není koho zapsat“).
- **Úspěch:** změna přepínače se ukládá okamžitě (optimisticky), tichý snackbar „Uloženo“; souhrn se přepočítá.
- **Offline (K rozhodnutí):** badge **„K rozhodnutí“** (00 § 7) u indikátoru připojení — offline fronta zápisu docházky není ve specifikaci rozhodnuta (viz [questions.md](../questions.md) Q-B13); v demu se zapisuje jen online.
- **Načítání / Chyba:** vzor 00 § 5.

**Interakce a validace:** zápis smí HVO nebo kdokoli s `can_record_attendance` (i RÁD) — samostatné oprávnění, nezávislé na platbách. Nejvýše jeden záznam na osobu a akci.

**Mobil/desktop:** primárně mobil; desktop zobrazí týž seznam vlevo, vpravo souhrn.

---

## B-10 · Platby — párování — `/oddil/platby`

**Účel:** Pracoviště účetní: srovnat příchozí platby s přihláškami, dořešit nespárované a rozhodnout přeplatky — přesně na korunu.

**Layout a M3 komponenty:** **dvoupanel** (desktop): vlevo fronta **transakcí**, vpravo detail vybrané transakce s **návrhy párování**; nad frontou filter chips `Nespárované` (výchozí) · `Spárované` · `Přeplatky` · `Vratky`. Tonal buttony **„Nahrát výpis“** a **„Zapsat platbu ručně“**. Sekce **„Přeplatky k rozhodnutí“**.

**Obsah a pole (účet Severky, z 01 § 7):**

- Fronta nespárovaných: **trans-306** (25.8. 16:20, 970 Kč, VS 99999999 — neexistuje, „HRBACKOVA LENKA / vikendovka Krystof“) · **trans-307** (25.8. 18:44, 400 Kč, bez VS, „OKURKA BRETISLAV“).
- Detail trans-306: návrh dle jména **prihlaska-207 Kryštof H.** (`vs_partial_name` — VS nesedí, jméno a částka ano) s tlačítkem **„Potvrdit párování“**; párování je přesné bez tolerance ([payment-matching.md](../docs/payment-matching.md)). Detail trans-307: návrh **prihlaska-212 Břetislav O.** (bez VS, shoda jména a částky 400 Kč).
- Přeplatky k rozhodnutí: **prihlaska-243 Teodor K.** — zaplaceno 4 000, cena 3 900, **přeplatek 100 Kč** — tři akce: **Vrátit** (záporná alokace `refund`) · **Převést na jinou přihlášku** téže osoby · **Ponechat jako dar**.
- Spárované (ke čtení): trans-301…305, 308 s metodou párování a cílovou přihláškou.

**Stavy:**

- **Prázdný — nespárované:** kanonický **„Fronta transakcí (ÚČE)“** z 00 § 5 (ikona `account_balance`, „Žádné nespárované platby“).
- **Prázdný — přeplatky:** inline řádek „Žádný přeplatek nečeká na rozhodnutí.“
- **Úspěch:** po potvrzení párování transakce zmizí z fronty, přihláška projde `evaluate()` (např. na `Paid`), snackbar „Platba spárována“, badge Platby klesne. Přeplatek po rozhodnutí zmizí ze sekce.
- **M:N (definovaný vzhled):** jedna platba za víc přihlášek → návrh s víc cíli a ručním potvrzením; naznač jako sekundární scénář.
- **Načítání / Chyba:** vzor 00 § 5.

**Interakce a validace:** párovat a řešit přeplatky smí HVO a ÚČE ([authorization.md](../docs/authorization.md) → Platby). Součet alokací transakce nesmí překročit její částku; vratka nesmí stáhnout součet přihlášky pod nulu. Ruční zápis platby generuje `external_id` (`manual:<uuid>`). VS/SS se ukládají bez počátečních nul.

**Mobil/desktop:** dvoupanel se na mobilu hroutí — fronta jako seznam, detail transakce jako **bottom sheet** s návrhy.

---

## B-11 · Osoby — `/oddil/osoby`

**Účel:** Evidence členů a hostů oddílu — kdo je kdo, ve které družině, v jakém stavu — a úprava jejich údajů.

**Layout a M3 komponenty:** datová tabulka + tonal button **„Přidat osobu“**; filter chips `Členové` (výchozí) · `Hosté` · `Neaktivní` · podle družiny; vyhledávání. Detail osoby (klik) jako pravý panel / bottom sheet.

**Obsah a pole (oddil-01, z 01 § 2):**

- Sloupce: **Jméno · Typ** (člen / host) **· Věk · Družina · Zástupci · Stav**. Řádky děti osoba-020…032 (Sovy/Lišky), tým osoba-001…005, hosté osoba-040 (dobrovolník), osoba-041 (host bez data narození, warning). Neaktivní **osoba-033 Barbora** je mimo výchozí filtr (chip „Neaktivní“).
- Detail — Základ: jméno (našeptávač whitelistu, 00 a [validation.md](../docs/validation.md)), příjmení, datum narození, pohlaví, adresa (Mapy.cz našeptávač), pojišťovna, kontakt. Sekce **Vazby** (zástupci u dítěte), **Členství** (družina, stav), **Chytré sloupce** (dle přístupu).
- **Citlivá / zdravotní data** se zobrazují jen podle role — RÁD nezletilý (osoba-004) je nevidí; maskováno s ikonou `lock`.

**Stavy:**

- **Prázdný:** kanonický **„Evidence osob“** z 00 § 5 (ikona `person_search`, CTA „Přidat osobu“).
- **Úspěch:** úprava → snackbar „Uloženo“ + audit. Změna stavu host → člen dialogem.
- **Bez práva:** VO/RÁD vidí evidenci ke čtení (RÁD jen svou družinu); editace jen HVO.
- **Načítání / Chyba:** vzor 00 § 5.

**Interakce a validace:** upravit údaje osoby a měnit stav smí jen HVO ([authorization.md](../docs/authorization.md) → Osoby). Podmíněná povinnost polí dle kontextu ([validation.md](../docs/validation.md)). Datum narození dítěte edituje jen vedoucí oddílu (viz plocha D).

**Mobil/desktop:** tabulka → karty; detail jako bottom sheet.

---

## B-12 · Družiny a závody — `/oddil/druziny`

**Účel:** Členění oddílu do družin (pro docházku i chytré sloupce) a příprava závodních stanovišť.

**Layout a M3 komponenty:** karty družin (název, vedoucí, počet členů) + tonal button **„Založit družinu“**; u oddílu se zapnutým modulem závodů sekce **„Stanoviště“**.

**Obsah a pole (z 01 § 8):**

- Družiny: **druzina-601 Lišky** (vede Tomáš Hruban, 6 členů) · **druzina-602 Sovy** (vede Klára Vondrušková, 7 členů). Klik → seznam členů s možností přeřadit; funkce `leader`/`deputy` nejvýše jedna na družinu ([validation.md](../docs/validation.md) → Družiny a role RÁD).
- Stanoviště (jsou-li závody): viz B-08; zde jejich správa napříč akcemi oddílu.

**Stavy:**

- **Prázdný:** kanonický **„Družiny“** z 00 § 5 (ikona `groups`, CTA „Založit družinu“).
- **Úspěch:** založení/přeřazení → snackbar + audit.
- **Načítání / Chyba:** vzor 00 § 5.

**Interakce a validace:** definovat družiny a jejich členy smí jen HVO. Osoba může být `leader`/`deputy` ve víc družinách téhož oddílu; nejvýše jeden `leader` a jeden `deputy` na družinu.

**Mobil/desktop:** karty v jednom sloupci; seznam členů jako bottom sheet.

---

## B-13 · Reporty oddílu — `/oddil/reporty`

**Účel:** Oddílové reporty — členská základna, účast, platby, docházka — jako podklad pro vedení a vykazování.

**Layout a M3 komponenty:** vlevo výběr reportů (M3 list, kód + název), vpravo plocha reportu: parametry · graf · tabulka · tonal button **„Export CSV“**. Sdílí vzor s reporty ústředí (04 · C-06), ale **scope je vlastní oddíl** ([reports.md](../docs/reports.md), [authorization.md](../docs/authorization.md) → Reporty).

**Obsah a pole:**

- Výběr (rozsah dle role): **R1 Akce a docházka · R2 Členové v čase · R3 Účast na akcích · R4 Docházka schůzek · R5 Dobrovolnické hodiny · R7 Platby**. ÚČE vidí jen **R7 Platby**; RÁD jen osoby své družiny; VO/HVO celý oddíl.
- Parametry: Období od–do (výchozí 12 měsíců) · Granularita · Typ akce. Pod výsledkem metadata „Vygenerováno … · parametry“.
- Demo čísla čerpají z akcí a plateb Severky (R5 dobrovolnické hodiny: Břetislav 32 h, Eliška 20 h; R7 z transakcí § 7).

**Stavy:**

- **Prázdný:** kanonický **„Reporty“** z 00 § 5 (ikona `monitoring`, „Za toto období nejsou data“). Série s nulami **není** prázdný stav — prázdné koše se kreslí jako nuly.
- **Úspěch (export):** snackbar „CSV exportováno“.
- **Načítání:** skeleton se zachovanou plochou grafu; **Chyba:** vzor 00 § 5.

**Interakce a validace:** reporty jen ke čtení; scope se odvozuje z role, ne z parametru requestu.

**Mobil/desktop:** výběr reportů jako chips; grafy responzivní, tabulky ve scrollu.

---

## B-14 · Nastavení oddílu — `/oddil/nastaveni`

**Účel:** Konfigurace oddílu jako celku — údaje, bankovní účet, lhůty, moduly, tým rolí — na jednom místě.

**Layout a M3 komponenty:** sbalitelné sekce: ① Základ · ② Bankovní účet · ③ Lhůty a pravidla · ④ Moduly · ⑤ Tým a role · ⑥ Členské příspěvky.

**Obsah a pole (Severka, z 01 § 1):**

- ① Základ: název, město (Liberec), typ (pobočný spolek), IČO `04512774` (modulo 11).
- ② Bankovní účet: `2900123456/2010` (Fio), stav synchronizace „naposledy St 26.8. 6:00“; token se nastavuje (maskovaný), párování běží i z nahraného výpisu bez API.
- ③ Lhůty a pravidla: schválení zástupcem **7 dní**, nabídka náhradníkovi **48 hodin**, vypršení nezaplacených přihlášek **vypnuto**.
- ④ Moduly: družiny ✔ · závody (Stezka) ✔ · dobrovolnické hodiny ✔.
- ⑤ Tým a role: seznam účtů s rolemi (Martin HVO, Klára VO, Tomáš VD, Eliška RÁD, Ivana ÚČE) + **„Pozvat do týmu“** (pozvánka na roli VO/RÁD/ÚČE, platí 14 dní). Odebrání role uzavře otevřená přiřazení akcí a zapíše audit; poslední HVO odebrat nelze.
- ⑥ Členské příspěvky: lokální složka příspěvku, výchozí sazba pro rok.

**Stavy:**

- **Úspěch:** uložení sekce → snackbar + audit.
- **Bez práva:** jen HVO má zápis; ÚČE vidí bankovní účet ke čtení; ostatní sekce skryté.
- **Chyba (validace):** IČO modulo 11, pozvánka validní e-mail; inline pod polem.
- **Načítání / Chyba:** vzor 00 § 5.

**Interakce a validace:** nastavení účtu, lhůt a modulů a správu rolí smí jen HVO ([authorization.md](../docs/authorization.md)). Pozvánka na roli dle Pozvánky na role; přijetí existujícím účtem přidá roli k němu.

**Mobil/desktop:** sekce jako karty pod sebou; správa rolí jako tabulka → karty.

---

## Přijaté defaulty

Kde je spec otevřená, demo volí bez přepínačů (viz [questions.md](../questions.md) Q-B\*): akci **nelze** založit bez šablony — vždy se vybírá z existujících (Q-B1); **hromadné akce jsou jen dvě** — připomínka platby a export výběru (Q-B5/Q-B6); export je plochá CSV z mock dat; **HEIC náhled** se v demu nahrazuje šedým placeholderem, ne serverovým převodem (Q-B8); běží **nejvýše tolik nabídek náhradníkům, kolik je volných míst** (Q-B9); přeplatek nabízí tři akce (vrátit / převést / ponechat jako dar); změna ceny nepřecení podané přihlášky; SS je needitovatelné (generuje systém), VS přihlášky rovněž. Badge **„K rozhodnutí“** (00 § 7) zobraz v UI pouze u **offline režimu zápisu docházky (B-09)** — jinde defaulty nekomentuj.

---

## Doplnění demo dat

Plocha B čerpá výhradně z existujících entit `01-demo-data.md` pro **oddil-01 Oddíl Severka** — žádné nové ID nezavádí: akce akce-101 až akce-104, akce-106 (§ 4) · přihlášky prihlaska-201 až 229, 240 až 245 (§ 5) · dokumenty dok-701 až dok-705 (§ 6) · transakce trans-301 až 308 (§ 7) · družiny druzina-601/602, hlídky hlidka-501/502, stanoviště stanoviste-01 až 03, docházka akce-104, dobrovolnické hodiny akce-101 (§ 8) · tým osoba-001 až 005 a účty ucet-001 až 005 (§ 2.1, § 2.6). Persona plochy B: **Martin Dvořáček (HVO)** a **Ivana Šmídková (ÚČE)**.
