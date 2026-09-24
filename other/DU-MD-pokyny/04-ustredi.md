# 04 · Plocha C — Správa ústředí

Specifikace obrazovek administrace ústředí pro DEMO. Rámec, navigaci, M3 tokeny a konvence prázdných/loading/chybových stavů definuje `00-projekt-a-design.md` (**00**), data mock store `01-demo-data.md` (**01** — entity plochy C zejména v § 1, § 2.5, § 5.4, § 9 a § 10). Žádný backend, žádná auth; mutace mění in-memory store a UI se přepočítá.

**Publikum a ergonomie:** několik expertních uživatelů s rolí **ADM (administrátor ústředí)**; **desktop-first** — tabulky scrollují ve vlastním kontejneru, na mobilu se hroutí do karet. Rozsah dat: **vše, napříč oddíly, s dimenzí region**.

**Persona (přepínač role, 00 § 3.3):** položka „Bohdana Krejcárková — administrátorka ústředí“ přepne na osobu **osoba-070** a přistane na `/ustredi`; v top app baru zobrazena jako **„Ústředí (ADM)“**.

**Navigace (`/ustredi/...`, M3 rail dle 00 § 3.2):** **Přehled** · **Oddíly** (`/ustredi/oddily`) · **Regiony** (`/ustredi/regiony`) · **Slučování osob** (`/ustredi/slucovani`, badge = žádosti vyžadující zásah, výchozí **1**: merge-901) · **Reporty** (`/ustredi/reporty`) · **Šablony a whitelist** (`/ustredi/sablony`) · **Vzdělávání** (`/ustredi/vzdelavani`) · **Audit log** (`/ustredi/audit`).

**Zásady (vynucuj v UI):** 1) kandidáti na sloučení se **jen navrhují, nikdy neslučují automaticky**; 2) **regiony se nemažou** — jen se označí jako _sloučený_ / _zrušený_, historie se nepřepisuje; 3) **reporty jsou jen ke čtení** a region akce je **snapshot z okamžiku její první publikace**; 4) **reportovací sloučení ≠ skutečné sloučení osob** — nemění žádná data, počítá se jen v R9.

---

## C-01 · Přehled ústředí (dashboard) — `/ustredi`

**Účel:** Kokpit ADM: velikost organizace v číslech a vše, co čeká na zásah ústředí — proklikem o úroveň hlouběji.

**Layout a M3 komponenty:** Top app bar „Přehled — Správa ústředí“. Řádek čtyř **KPI karet** (filled, číslo display-small + popisek), pod ním **„Vyžaduje pozornost“** (M3 list, leading ikona v tonálním kruhu, trailing šipka) a karta **„Organizace“**.

**Obsah a pole (z 01):**

- KPI: **Oddíly 4** (+ 1 speciální oddíl ústředí) · **Regiony 3 aktivní** · **Unikátní děti 2026: 14** (R9 → C-06) · **Žádosti o sloučení: 2 běžící**.
- Vyžaduje pozornost (dle urgence): 1. „**Pozvánka HVO se nepodařila doručit** — Oddíl Poutníci“ → C-02 detail; 2. „**Sloučení připraveno k provedení** — Nikol Ježková (schváleno 2 ze 2)“ → C-05; 3. „**Žádost o sloučení čeká na rodiče** — Sára Mlžná, propadne Čtvrtek 17.9.“ → C-05; 4. „**3 kandidáti na reportovací sloučení** — report Unikátní děti“ → C-06/R9; 5. „**Oddíl bez regionu** — Oddíl Poutníci“ → C-03.
- Organizace: Region Sever (1 oddíl) · Morava (1) · Jih (1) · bez regionu (1); drobný text „Ústředí do regionů nepatří.“

**Stavy:**

- **Prázdný — „Vyžaduje pozornost“:** ikona `done_all` v `secondary-container` kruhu, nadpis **„Vše vyřízeno“**, text „Žádná žádost, pozvánka ani kandidát nečeká na zásah ústředí.“, bez CTA.
- **Načítání:** skeletony; **Chyba:** vzor 00 § 5.

**Interakce a validace:** vše jen proklik, žádné mutace z dashboardu.

**Mobil/desktop:** KPI 2×2, seznam pozornosti první.

---

## C-02 · Oddíly a pozvánka HVO — `/ustredi/oddily` (+ detail `:id`)

**Účel:** Celá organizace na jednom místě — které oddíly existují, kdo je vede, kam patří — a předání oddílu hlavnímu vedoucímu pozvánkou e-mailem.

**Layout a M3 komponenty:** datová tabulka (00 § 4.4) + tonal button **„Založit oddíl“**; filter chips `Bez HVO` · `Bez regionu`; vyhledávání. Detail oddílu (klik na řádek) se třemi kartami-sekcemi: **Základ** · **Hlavní vedoucí** · **Region**.

**Obsah a pole:**

- Tabulka: **Název · Typ · IČO · Region · HVO** (jméno, nebo chip stavu pozvánky) **· Členů**. Řádky: oddil-01 Severka (Region Sever, Martin Dvořáček, 21) · oddil-02 Jestřábi (Region Morava, Věra Kropáčková, 7) · oddil-03 Bobři (Region Jih, Bohumil Ježek, 3) · **oddil-04 Poutníci** (bez regionu, chip **„Pozvánka nedoručena“** v `error-container` paletě, 0) · **oddil-90 Ústředí Dorostové unie** — odlišený řádek (chip `tertiary` „Speciální oddíl ústředí“, tooltip „Nemá registrované členy, slouží celostátním akcím.“). Typy a IČO viz 01 § 1; sloupec Členů = počet všech evidovaných osob oddílu (vč. hostů a neaktivních záznamů).
- **Detail — Základ:** název, typ, IČO, město; editace dialogem.
- **Detail — Hlavní vedoucí (referenční: oddil-04):** historie pozvánky (pozvanka-01): „Odesláno Pondělí 17.8. 10:02 na `viktor.rovny@exmaple.cz` — **trvale nedoručeno**“ (chip `Canceled` paleta) + banner v `error-container`: „E-mail se nepodařilo doručit. Zkontrolujte adresu a pošlete pozvánku znovu.“ Tlačítko **„Poslat pozvánku znovu“** — dialog s opravitelným e-mailem; odeslání zapíše řádek historie, snackbar „Demo: e-mail odeslán — pozvánka HVO“, chip „Odesláno, čeká na přijetí (platí 14 dní)“. Vysvětlivka: „Přijetím pozvánky vznikne účet s rolí HVO; má-li pozvaný už účet, role se přidá k němu.“ Přijímací stránku demo nestaví. U oddílů 01–03 jen jméno a e-mail HVO.
- **Detail — Region:** aktuální příslušnost (u oddil-04 „Bez regionu“) + akce „Zařadit / přesunout do regionu“ (dialog: „Přesun uzavře stávající příslušnost a otevře novou od dneška. Historické reporty se nemění.“) + odkaz „Historie příslušností“ → C-03. Oddil-90 sekci Region ani pozvánku nemá.

**Stavy:**

- **Prázdný (seznam; v demu nenastane):** ikona `apartment`, nadpis **„Zatím žádné oddíly“**, text „Založte první oddíl a předejte ho hlavnímu vedoucímu pozvánkou e-mailem.“, CTA **„Založit oddíl“**.
- **Prázdný — hlavní vedoucí:** ikona `person_add`, nadpis **„Oddíl nemá hlavního vedoucího“**, text „Bez HVO nelze zakládat akce ani spravovat členy. Pošlete pozvánku e-mailem.“, CTA **„Poslat pozvánku“**.
- **Úspěch:** po odeslání pozvánky potvrzení v sekci („Pozvánka odeslána na … — další krok je mimo systém“). **Načítání / Chyba:** vzor 00 § 5.

**Interakce a validace:** „Založit oddíl“ = dialog: Název* · Typ* (segmented: IČO ústředí / pobočný spolek s vlastním IČO / kolektivní člen) · IČO\* (modulo 11) · Region (volitelný). Dle spec: u typu „kolektivní člen“ nesmí název obsahovat „DU“. E-mail pozvánky validní formát; přesun regionu jen „od teď“. Oddíl nemá v demu zánik — badge **„K rozhodnutí“** s tooltipem „Životní cyklus oddílu není ve specifikaci rozhodnut.“

**Mobil/desktop:** tabulka → karty; detail v jednom sloupci, dialogy jako bottom sheet.

---

## C-03 · Regiony — `/ustredi/regiony` (+ detail s historií)

**Účel:** Živá i historická struktura regionů — sloučené regiony nikdy nemizí — a odpověď na „kam oddíl patřil k datu X“.

**Layout a M3 komponenty:** karty regionů + sekce **„Oddíly bez regionu“**; tonal button **„Založit region“**. Detail regionu (klik): aktuální oddíly + časová osa příslušností.

**Obsah a pole (z 01 § 9.1):**

- Karty: **Region Sever** (aktivní, 1 oddíl) · **Region Morava** (aktivní, 1 oddíl, „vznikl 12.1.2025 sloučením“) · **Region Jih** (aktivní, 1 oddíl) · **Region Východ** a **Region Haná** (chip „Sloučený“, „→ Region Morava“, 0). Přepínač „Zobrazit i sloučené a zrušené“.
- Oddíly bez regionu: oddil-04 Poutníci + akce „Zařadit do regionu“; ústředí se tu nezobrazuje.
- **Detail (např. Region Morava):** aktuální oddíly (oddil-02, od 12.1.2025) s akcí „Přesunout do jiného regionu“; **časová osa příslušností** — oddíl · od–do · kam odešel (oddil-02: Region Haná 1.1.2019–12.1.2025 → Morava). Sloučený region jen ke čtení + odkaz na nástupce.
- **Sloučení regionů:** dvoukrokový dialog — výběr ≥ 2 zdrojů, název **nového** nástupnického regionu, náhled dotčených oddílů a stálá vysvětlivka: **„Historické reporty se nemění — region akce je snapshot z okamžiku její první publikace.“** Rozdělení a zrušení jen neaktivní položky menu s tooltipem „Ve specifikaci zatím nerozhodnuto“.

**Stavy:**

- **Prázdný (seznam):** ikona `map`, nadpis **„Zatím žádné regiony“**, text „Regiony jsou volitelná vrstva mezi ústředím a oddíly — slouží hlavně jako dimenze reportů.“, CTA **„Založit region“**.
- **Prázdný — oddíly bez regionu:** inline řádek „Všechny oddíly jsou zařazené.“
- **Úspěch (sloučení):** souhrn změn, zdroje → chip „Sloučený“, snackbar; operace atomická (jedna mutace store). **Načítání / Chyba:** vzor 00 § 5.

**Interakce a validace:** název regionu povinný a unikátní; přiřazení oddílu uzavře stávající příslušnost (oddíl je nejvýše v jednom regionu).

**Mobil/desktop:** karty v jednom sloupci; časová osa scrolluje.

---

## C-04 · Slučování osob — fronta žádostí — `/ustredi/slucovani`

**Účel:** Dohled ústředí nad deduplikací: běžící, dokončené i potlačené žádosti — a jediné místo, kde lze potlačenou dvojici znovu povolit.

**Layout a M3 komponenty:** datová tabulka + filter chips (`Běžící` výchozí · `Dokončené` · `Zamítnuté / potlačené`); pod tabulkou sekce **„Potlačené dvojice“**.

**Obsah a pole (z 01 § 10.1):** sloupce **Osoby · Druh** („dítě“/„osoba“) **· Stav** (chip, palety 00 § 4.2: `pending` „Čeká na strany“ = PendingGuardian · `ready` „Připraveno k provedení“ = PendingPayment · `completed` „Dokončeno“ = Paid · `rejected` „Zamítnuto“ = Canceled · `reverted` „Vráceno“ = Expired) **· Rozhodnuté strany** („2 ze 2“) **· Propadne** (30 dní od založení). Řádky: **merge-901** Nikol Ježková × Nikol Ježková — Připraveno k provedení — 2 ze 2 — Pátek 18.9. · **merge-902** Sára Mlžná × Sára Mlžná — Čeká na strany — 1 ze 2 — Čtvrtek 17.9. Potlačené dvojice: **merge-903** Denis Kropáček × Denis Kropáček (zamítl Bohumil Ježek, Čtvrtek 6.8.) s akcí **„Znovu povolit nabízení“** (potvrzovací dialog, snackbar). Trvalá vysvětlivka: „Žádosti zakládají uživatelé a rodiče a schvalují je dotčené strany — ústředí jen dohlíží, povoluje potlačené dvojice a jako jediné smí sloučení vrátit.“

**Stavy:**

- **Prázdný (fronta):** ikona `join`, nadpis **„Žádné žádosti o sloučení“**, text „Žádosti vznikají zdola od uživatelů — prázdná fronta je normální stav, ne chyba.“, bez CTA. Prázdné potlačené dvojice: inline řádek.
- **Načítání:** skeleton řádků; **Chyba:** vzor 00 § 5.

**Interakce a validace:** klik na řádek → C-05. Potlačení dvojice je v demu **trvalé** (povolí jen ADM) — u akce „Znovu povolit nabízení“ badge **„K rozhodnutí“** s tooltipem „Trvalé vs. dočasné potlačení není ve specifikaci rozhodnuto.“

**Mobil/desktop:** tabulka → karty (osoby, chip stavu, propadnutí).

---

## C-05 · Porovnání osob, konflikty a provedení — `/ustredi/slucovani/:id`

**Účel:** Obě osoby vedle sebe, rozhodnutí každého konfliktu volbou A/B a provedení — poslední krok před operací vratnou jen ADM revertem.

**Layout a M3 komponenty:** hlavička (obě jména, chip stavu, druh); karta **„Schválení stran“** (M3 list — strana, rozhodnutí, kdy); **porovnávací tabulka dvou sloupců** (osoba A | osoba B), řádek za slučované pole; sticky lišta s primární akcí.

**Obsah a pole (referenční: merge-901, osoba-054 = A/cíl, osoba-060 = B/zdroj):**

- Schválení stran: „Bohumil Ježek (rodič, za Nikol — Bobři) — schválil Čtvrtek 20.8. 18:12“ · „Věra Kropáčková (HVO Jestřábi, dítě bez aktivního rodiče) — schválila Pátek 21.8. 09:30“. Vysvětlivka: „Sloučení spojí jen osobu dítěte, účty rodičů se nespojují.“
- Porovnání — **jen základní slučovaná pole** (jméno, příjmení, přezdívka, tituly, pohlaví, datum narození, e-mail, adresa, pojišťovna); nikdy citlivá data z cizího oddílu (vysvětlivka s ikonou `lock`). Řádky merge-901: jméno, příjmení, datum narození — **shodné, zamčené**; přezdívka (— × „Niki“) — **automaticky vyřešeno** („vyplněná strana vyhrává“), zobrazené, ale zamčené; **adresa** („Budějovická 14, Písek“ × „Sadová 8, Olomouc“) a **pojišťovna** (111 × 205) — **skutečné konflikty s povinnou volbou A/B** (radio v každém sloupci). Ruční přepis se nenabízí — text: „Vybírá se vždy hodnota jedné z osob, aby šlo sloučení věrně vrátit. Opravit hodnotu lze až po sloučení editací.“
- Souhrn: „Přenese se automaticky: členství v oddílech, přihlášky (1 — Drakiáda), docházka, členství DU. Citlivá data a dokumenty zůstávají per oddíl.“
- Sticky lišta: **„Provést sloučení“** (filled, aktivní až po rozhodnutí všech konfliktů) + text „V reálném systému provádí iniciátor; v demu krok spouští ADM z fronty.“

**Stavy:**

- **`pending` (merge-902):** porovnání jen ke čtení; karta stran: „Karolína Mlžná (rodička) — zatím nerozhodla“ + banner `tertiary` „Žádost čeká na strany. Propadne bez odezvy Čtvrtek 17.9.“ U osoby-041 chybějící datum narození s warning ikonou („slabý kandidát“).
- **Úspěch (provedení merge-901):** potvrzovací dialog „Sloučit Nikol Ježkovou (Jestřábi) do Nikol Ježkové (Bobři)? Zdrojová osoba zůstane jako náhrobek s přesměrováním — staré odkazy povedou na sjednocenou osobu.“ Po potvrzení: `completed`, prihlaska-253 přejde na osobu-054, osoba-060 se zobrazuje jako „sloučena do osoba-054“, snackbar „Osoby sloučeny“, badge railu klesne na 0.
- **Chyba — blokující kolize** (v demu jen definovaný vzhled): mají-li obě osoby aktivní přihlášku na téže akci, dialog v `error-container`: „Sloučení nelze provést: obě osoby mají aktivní přihlášku na akci [název]. Vyřešit ji musí vedoucí akce — nic nebylo změněno.“
- **`completed` — revert:** detail zobrazí snapshot (stav obou osob před sloučením, rozhodnuté volby, přenesené vazby) a tlačítko **„Vrátit sloučení“** (outlined, error; jen ADM). Dialog revertu ve dvou sloupcích **před potvrzením**: „Co se vrátí“ (snapshot) × „Co zůstane u cílové osoby“ (záznamy vzniklé po sloučení). Revert je jednorázový → `reverted`; nové sloučení = nová žádost.

**Interakce a validace:** bez rozhodnutí všech konfliktů nelze dokončit; volby se drží, dokud je žádost `ready`; různá data narození by vyžadovala zvláštní potvrzení „pravděpodobně nejde o stejnou osobu“ (v demu nenastává).

**Mobil/desktop:** sloupce se hroutí na řádky „pole → hodnota A / B“; rozhodovací tlačítka sticky, mimo dosah omylu.

---

## C-06 · Reporty ústředí — `/ustredi/reporty` (vč. R9)

**Účel:** Jedna obálka pro reporty R1–R9 napříč oddíly s dimenzí region — podklad pro vykazování a dotace.

**Layout a M3 komponenty:** vlevo výběr reportů (M3 list, kód + název), vpravo plocha reportu: parametry · sloupcový graf · tabulka · tonal button **„Export CSV“**. Sdílí vzor s oddílovými reporty (03 · B-11).

**Obsah a pole:**

- Výběr: **R1 Seznam akcí a docházky · R2 Počty členů v čase · R3 Účast na akcích · R4 Docházka schůzek · R5 Dobrovolnické hodiny · R6 Retence a odchody · R7 Platby · R8 Vzdělávání · R9 Unikátní děti**.
- Parametry: Oddíly (multi-select, výchozí „všechny“) · Období od–do (výchozí 12 měsíců) · Granularita (měsíc / kvartál / rok) · **Region** (filtruje přes snapshot akce) · Typ akce. Pod výsledkem metadata „Vygenerováno Středa 26.8. 14:19 · parametry výpočtu“.
- **R1 (s demo čísly)** — tabulka akce · termín · přihlášeno · dorazilo/nedorazilo · dobrovolnické hodiny: Letní tábor — 11.–25.7. — 6 — 6/0 — 52 h · Výlet na Kozí vrch — Sobota 22.8. — bez přihlášek — 7/2 — 0 h · Podzimní víkendovka — 25.–27.9. — 11 — — · Závod Stezka — Sobota 10.10. — 8 — — · Drakiáda — Sobota 17.10. — 4 — —. Poznámka (dle spec): „Kategorie se nevylučují — součet sloupců nemusí dát počet účastníků.“
- **R7 (s demo čísly)** — dlaždice + graf po měsících: **Předepsáno 34 330 Kč · Inkasováno 24 120 Kč · Pohledávky 4 920 Kč · Přeplatky 100 Kč · Nespárované platby 2 (1 370 Kč) · Storna 2**; rozpad po oddílech v tabulce.
- **R9 — Unikátní děti** (vlastní podoba obrazovky): výběr **roku** (segmented, výchozí 2026 — u R9 jen kalendářní roky) a velké číslo výsledku: **„14 unikátních dětí“**; rozpad po regionech Sever 10 · Morava 4 (snapshot z akce). Poznámky: „Hosté cizích oddílů se nepočítají.“ a „Součet po regionech může být vyšší než celkem — dítě mohlo jet do dvou regionů.“ Trvalý banner `tertiary`: **„Reportovací sloučení nemění žádná data — dvě osoby se počítají jako jedna jen v tomto reportu. Skutečné sloučení osob se schvalováním je v sekci Slučování osob.“** Karta **Kandidáti** (shoda jména, příjmení a data narození napříč oddíly; zobrazují se **jen tato tři pole, nic víc**): Amálie Peštová, 9.2.2013 · Nikol Ježková, 15.2.2013 · Denis Kropáček, 6.6.2014; u každého tlačítko **„Sloučit pro report“**. Karta **Aktivní reportovací sloučení**: výchozí prázdná; skupiny (tranzitivně A–B, B–C ⇒ jedna osoba) s akcí **„Zrušit“** (kdykoli — nic nepřepsalo).
- Ostatní reporty (R2–R6, R8) se počítají ze store „best effort“; metriky bez podkladu se **skryjí** (ne „0“, ne pomlčka). Export CSV = reálné stažení ploché tabulky z mock dat.

**Stavy:**

- **Prázdný:** kanonický **„Za toto období nejsou data“** z 00 § 5 — jen bez jakékoli akce v rozsahu. **Série s nulami není prázdný stav** — prázdné koše se kreslí jako nuly (graf nesmí lhát o trendu).
- **Prázdný — kandidáti R9:** ikona `verified`, nadpis **„Žádní kandidáti na sloučení“**, text „V datech není shoda jména a data narození napříč oddíly — evidence je čistá.“, bez CTA. Prázdná aktivní sloučení: inline řádek.
- **Úspěch (R9):** sloučení dvojice Amálie Peštová okamžitě přepočítá číslo **14 → 13** (jediný efekt operace); zrušení ho vrátí. Dvojice Nikol a Denis číslo nezmění — UI: „v čísle za 2026 se neprojeví“ (druhá osoba nemá letos aktivní přihlášku).
- **Načítání:** skeleton se zachovanou plochou grafu (500 ms); **Chyba:** vzor 00 § 5. **Úspěch (export):** snackbar „CSV exportováno“.

**Interakce a validace:** reporty jen ke čtení (kromě reportovacího sloučení — zakládá ústředí samo, bez schvalování); změna parametru přepočítá okamžitě; cesta ke skutečnému sloučení se z R9 nenabízí.

**Mobil/desktop:** výběr reportů jako chips; grafy responzivní, tabulky ve scrollu.

---

## C-07 · Šablony akcí ústředí a whitelist jmen — `/ustredi/sablony`

**Účel:** Spravovat systémové šablony, ze kterých oddíly zakládají akce — s jistotou, že úprava nic nerozbije zpětně — a whitelist jmen, aby kontrola chytala překlepy, ne neobvyklá jména.

**Layout a M3 komponenty:** dvě M3 taby: **Systémové šablony** · **Whitelist jmen**.

**Obsah a pole:**

- **Systémové šablony:** karty **sablona-401 Letní tábor · sablona-403 Závod Stezka · sablona-404 Jednorázová akce · sablona-405 Pravidelné schůzky** (z 01 § 3; oddílovou sablona-402 ADM nevidí). Na kartě: název, typ, výchozí hodnoty (sloupec „Přednastavuje“ z 01), přepínač **Aktivní** (neaktivní se nenabízí při zakládání; existující akce nedotčené). Editor = **stejná rodina formulářů jako nastavení akce (03 · B-04) v režimu „výchozí hodnoty“**. Trvalá poznámka: **„Úprava šablony se projeví jen u nově založených akcí — akce si šablonu ukládají jako snapshot.“**
- **Whitelist jmen:** prohledávatelný seznam křestních jmen (v demu vzorek, stránkovaný po 50) + tlačítko **„Přidat jméno“**. Sekce **„Oddílové výjimky“**: **vyjimka-01 „Melichar“ — Oddíl Severka — schválil Martin Dvořáček — Pondělí 2.3.** s akcí **„Povýšit do whitelistu“** (přesune jméno do seznamu, výjimka zmizí, snackbar). Vysvětlivka: „Ověřují se jen křestní jména, příjmení nikdy. Jméno mimo seznam neblokuje — oddíl může založit výjimku.“

**Stavy:**

- **Prázdný — šablony (v demu nenastane):** ikona `library_books`, nadpis **„Zatím žádné systémové šablony“**, text „Šablony přednastavují akce všem oddílům.“, CTA **„Vytvořit šablonu“**.
- **Prázdný — výjimky:** ikona `fact_check`, nadpis **„Žádné oddílové výjimky“**, text „Když oddíl schválí jméno mimo seznam, objeví se tady k případnému povýšení do whitelistu.“, bez CTA.
- **Úspěch:** snackbar „Šablona uložena — projeví se u nových akcí“ / „Jméno přidáno“. **Načítání / Chyba:** vzor 00 § 5.

**Interakce a validace:** jméno unikátní (bez ohledu na diakritiku a velikost písmen); editace jen ADM.

**Mobil/desktop:** karty v jednom sloupci; vyhledávání jmen jako primární vstup.

---

## C-08 · Zkrácené sekce — Vzdělávání a Audit log

Tyto části má plocha C v demu jen v minimální podobě — jedna obrazovka, bez detailů:

- **Vzdělávání — `/ustredi/vzdelavani`** — katalog kurzů ústředí: tabulka kurz · platnost v měsících (prázdná = slovo **„trvalý“**, ne prázdná buňka) · držitelů; tlačítko „Založit kurz“ (dialog: název\*, platnost — kladné celé měsíce, nebo „trvalý“). Kurzy se vážou na vzdělávací akce ústředí; po absolvování vzniká účastníkům záznam s platností. Přehled expirací (R8) demo nenabízí — jen odkaz na C-06. **Prázdný stav (výchozí):** ikona `school`, nadpis **„Zatím žádné kurzy“**, text „Kurzy evidují kvalifikace vedoucích a jejich platnost. Založte první kurz.“, CTA **„Založit kurz“**.
- **Audit log — `/ustredi/audit`** — tabulka změnových událostí: čas · aktér (u systémových „Systém“) · akce (založení / úprava / schválení / zamítnutí / storno) · cíl · oddíl. Plní se **mutacemi aktuální session** (mock store zapisuje každou mutaci ze všech ploch); filtr podle oddílu. **Prázdný stav (výchozí po startu):** ikona `history`, nadpis **„Zatím žádné události“**, text „Auditní log se plní změnami provedenými v této demo session. Proveďte změnu a vraťte se.“, bez CTA.

**Mimo rozsah dema (nestavět):** přijímací stránka pozvánky HVO, rozdělení a zrušení regionu, iniciace žádosti o sloučení uživatelem (plocha D), GDPR audit, katalog notifikací.

---

## Přijaté defaulty

Kde je spec otevřená, demo volí bez přepínačů: pozvánka HVO platí **14 dní**; přijetí pozvánky existujícím účtem přidá roli; přesun oddílu mezi regiony jen „od teď“; nástupce sloučení regionů je vždy **nový** region; rozdělení a zrušení regionu se nestaví; potlačení zamítnuté dvojice je **trvalé**; **revert smí jen ADM**; směr sloučení: cílem je **starší záznam**; období R9 jen kalendářní roky; ADM vidí jen **systémové** šablony; whitelist porovnává bez diakritiky a velikosti písmen. Badge **„K rozhodnutí“** (00 § 7) zobraz pouze na dvou místech: životní cyklus oddílu (C-02) a trvalost potlačení dvojice (C-04) — jinde defaulty nekomentuj.

---

## Doplnění demo dat

Demo entity plochy C jsou sloučené přímo v `01-demo-data.md`: oddíly oddil-04 a oddil-90 vč. typů a IČO (§ 1) · osoby osoba-060 až osoba-063 a osoba-070 (§ 2.5) · přihlášky prihlaska-252 a prihlaska-253 (§ 5.4) · regiony region-801 až region-805, pozvanka-01 a vyjimka-01 (§ 9) · žádosti merge-901 až merge-903 a podklad reportu R9 (§ 10).
