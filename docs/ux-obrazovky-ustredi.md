# UX — Obrazovky plochy C (správa ústředí)

Detailní specifikace obrazovek administrace ústředí (`/ustredi/...`). Navigaci, routy, breadcrumbs a operační matici ADM definuje [ux-navigace.md](ux-navigace.md) § 2.3; sdílené stavy a texty [ux-texty-stavy.md](ux-texty-stavy.md); role a rozsah [authorization.md](authorization.md) → Agenda ústředí; životní cyklus regionů [region-lifecycle.md](region-lifecycle.md); slučování osob [person-merge.md](person-merge.md); reporty [reports.md](reports.md); pravidla polí [validation.md](validation.md).

## 1. Rámec plochy

- **Publikum:** několik expertních uživatelů s rolí **ADM (administrátor ústředí)**.
- **Ergonomie:** desktop-first — tabulky scrollují ve vlastním kontejneru, na mobilu se hroutí do karet. Rozsah dat: **vše, napříč oddíly, s dimenzí region**.
- **Navigace:** M3 rail — Přehled · Oddíly (`/ustredi/oddily`) · Regiony (`/ustredi/regiony`) · Slučování osob (`/ustredi/slucovani`, badge = žádosti vyžadující zásah) · Reporty (`/ustredi/reporty`) · Šablony a whitelist (`/ustredi/sablony`) · Vzdělávání (`/ustredi/vzdelavani`) · Audit log (`/ustredi/audit`).

**Zásady (vynucuj v UI):**

1. **Kandidáti na sloučení se jen navrhují, nikdy neslučují automaticky** — sloučení vzniká zdola (uživatel/rodič) a schvalují je dotčené strany; ústředí dohlíží, povoluje potlačené dvojice a jako jediné smí sloučení vrátit ([person-merge.md](person-merge.md)).
2. **Regiony se nemažou** — jen se označí jako _sloučený_ / _zrušený_, historie se nepřepisuje ([region-lifecycle.md](region-lifecycle.md)).
3. **Reporty jsou jen ke čtení** a region akce je **snapshot z okamžiku jejího vzniku** ([reports.md](reports.md)).
4. **Reportovací sloučení ≠ skutečné sloučení osob** — nemění žádná data, počítá se jen v reportu Unikátní děti (R9).
5. **ADM je napříč oddíly čtenář** — nevidí obsah dokumentů ani citlivá/zdravotní data mimo celostátní akce ústředí ([authorization.md](authorization.md)).

## 2. C-01 · Přehled ústředí — `/ustredi`

**Účel a publikum:** kokpit ADM — velikost organizace v číslech a vše, co čeká na zásah ústředí, proklikem o úroveň hlouběji.

**Layout a komponenty:** top app bar „Přehled — Správa ústředí“; řádek KPI karet, pod ním „Vyžaduje pozornost“ (M3 list) a karta „Organizace“.

**Obsah a pole:** KPI (počet oddílů, aktivní regiony, unikátní děti za rok → R9, běžící žádosti o sloučení). „Vyžaduje pozornost“ dle urgence: nedoručená pozvánka HVO → C-02; sloučení připravené k provedení → C-05; žádost o sloučení čekající na strany → C-05; kandidáti na reportovací sloučení → C-06/R9; oddíl bez regionu → C-03. Karta Organizace: rozpad oddílů po regionech + poznámka „Ústředí do regionů nepatří.“

**Stavy:**

- **Prázdný — „Vyžaduje pozornost“:** ikona `done_all`, „Vše vyřízeno“, „Žádná žádost, pozvánka ani kandidát nečeká na zásah ústředí.“ Bez CTA.
- **Načítání:** skeletony; **Chyba:** [ux-texty-stavy.md](ux-texty-stavy.md) § 5.

**Interakce a validace:** jen proklik, žádné mutace z dashboardu.

**Oprávnění:** ADM.

**Mobil/desktop:** KPI 2×2, seznam pozornosti první.

**Notifikace:** žádné z přehledu.

## 3. C-02 · Oddíly a pozvánka HVO — `/ustredi/oddily` (+ detail `:id`)

**Účel a publikum:** celá organizace na jednom místě — které oddíly existují, kdo je vede, kam patří — a předání oddílu hlavnímu vedoucímu pozvánkou e-mailem.

**Layout a komponenty:** datová tabulka + tlačítko „Založit oddíl“; filter chips `Bez HVO` · `Bez regionu`; vyhledávání. Detail oddílu (klik na řádek) se sekcemi **Základ · Hlavní vedoucí · Region**.

**Obsah a pole:**

- Tabulka: Název · Typ · IČO · Region · HVO (jméno, nebo chip stavu pozvánky) · Členů. Speciální oddíl ústředí je odlišený řádek (chip „Speciální oddíl ústředí“, tooltip „Nemá registrované členy, slouží celostátním akcím.“); nemá sekci Region ani pozvánku HVO.
- Detail — Základ: název, typ, IČO, město; editace dialogem.
- Detail — Hlavní vedoucí: historie pozvánky (kdy, na jakou adresu, stav doručení). Při trvalém nedoručení chip „Pozvánka nedoručena“ (`error-container`) + banner „E-mail se nepodařilo doručit. Zkontrolujte adresu a pošlete pozvánku znovu.“ + tlačítko „Poslat pozvánku znovu“ (dialog s opravitelným e-mailem; platí 14 dní). Vysvětlivka: „Přijetím pozvánky vznikne účet s rolí HVO; má-li pozvaný už účet, role se přidá k němu.“
- Detail — Region: aktuální příslušnost + akce „Zařadit / přesunout do regionu“ (dialog: „Přesun uzavře stávající příslušnost a otevře novou od dneška. Historické reporty se nemění.“) + odkaz „Historie příslušností“ → C-03.

**Stavy:**

- **Prázdný (seznam):** ikona `apartment`, „Zatím žádné oddíly“, „Založte první oddíl a předejte ho hlavnímu vedoucímu pozvánkou e-mailem.“, CTA „Založit oddíl“.
- **Prázdný — hlavní vedoucí:** ikona `person_add`, „Oddíl nemá hlavního vedoucího“, „Bez HVO nelze zakládat akce ani spravovat členy. Pošlete pozvánku e-mailem.“, CTA „Poslat pozvánku“.
- **Úspěch:** po odeslání pozvánky potvrzení v sekci + chip „Odesláno, čeká na přijetí (platí 14 dní)“.
- **Načítání / Chyba:** § 5.

**Interakce a validace:** „Založit oddíl“ = dialog: Název* · Typ* (IČO ústředí / pobočný spolek s vlastním IČO / kolektivní člen) · IČO\* (modulo 11) · Region (volitelný). U typu „kolektivní člen“ nesmí název obsahovat „DU“. E-mail pozvánky validní formát; přesun regionu jen „od teď“.

**Oprávnění:** ADM.

**Mobil/desktop:** tabulka → karty; detail v jednom sloupci, dialogy jako bottom sheet.

**Notifikace:** pozvánka → `EMAIL_HVO_INVITE` ([notifications.md](notifications.md)).

## 4. C-03 · Regiony — `/ustredi/regiony` (+ detail s historií)

**Účel a publikum:** živá i historická struktura regionů — sloučené regiony nikdy nemizí — a odpověď na „kam oddíl patřil k datu X“.

**Layout a komponenty:** karty regionů + sekce „Oddíly bez regionu“; tlačítko „Založit region“. Detail regionu (klik): aktuální oddíly + časová osa příslušností.

**Obsah a pole:**

- Karty: název, stav (aktivní / sloučený → nástupce / zrušený), počet oddílů. Přepínač „Zobrazit i sloučené a zrušené“.
- Oddíly bez regionu + akce „Zařadit do regionu“; ústředí se tu nezobrazuje.
- Detail: aktuální oddíly s akcí „Přesunout do jiného regionu“; časová osa příslušností — oddíl · od–do · kam odešel. Sloučený region jen ke čtení + odkaz na nástupce.
- **Sloučení regionů:** dvoukrokový dialog — výběr ≥ 2 zdrojů, název **nového** nástupnického regionu, náhled dotčených oddílů a stálá vysvětlivka: „Historické reporty se nemění — region akce je snapshot z okamžiku jejího vzniku.“

**Stavy:**

- **Prázdný (seznam):** ikona `map`, „Zatím žádné regiony“, „Regiony jsou volitelná vrstva mezi ústředím a oddíly — slouží hlavně jako dimenze reportů.“, CTA „Založit region“.
- **Prázdný — oddíly bez regionu:** inline řádek „Všechny oddíly jsou zařazené.“
- **Úspěch (sloučení):** souhrn změn, zdroje → chip „Sloučený“, snackbar; operace atomická.
- **Načítání / Chyba:** § 5.

**Interakce a validace:** název regionu povinný a unikátní; přiřazení oddílu uzavře stávající příslušnost (oddíl je nejvýše v jednom regionu) — dle [region-lifecycle.md](region-lifecycle.md).

**Oprávnění:** ADM.

**Mobil/desktop:** karty v jednom sloupci; časová osa scrolluje.

**Notifikace:** žádné.

## 5. C-04 · Slučování osob — fronta žádostí — `/ustredi/slucovani`

**Účel a publikum:** dohled ústředí nad deduplikací — běžící, dokončené i potlačené žádosti — a jediné místo, kde lze potlačenou dvojici znovu povolit.

**Layout a komponenty:** datová tabulka + filter chips (`Běžící` výchozí · `Dokončené` · `Zamítnuté / potlačené`); pod tabulkou sekce „Potlačené dvojice“.

**Obsah a pole:** sloupce Osoby · Druh (dítě / osoba) · Stav (chip: Čeká na strany · Připraveno k provedení · Dokončeno · Zamítnuto · Vráceno) · Rozhodnuté strany („2 ze 2“) · Propadne (30 dní od založení). Potlačené dvojice s akcí „Znovu povolit nabízení“ (potvrzovací dialog, snackbar). Trvalá vysvětlivka: „Žádosti zakládají uživatelé a rodiče a schvalují je dotčené strany — ústředí jen dohlíží, povoluje potlačené dvojice a jako jediné smí sloučení vrátit.“

**Stavy:**

- **Prázdný (fronta):** ikona `join`, „Žádné žádosti o sloučení“, „Žádosti vznikají zdola od uživatelů — prázdná fronta je normální stav, ne chyba.“ Bez CTA. Prázdné potlačené dvojice: inline řádek.
- **Načítání:** skeleton řádků; **Chyba:** § 5.

**Interakce a validace:** klik na řádek → C-05. Potlačenou dvojici znovu povolit smí jen ADM ([person-merge.md](person-merge.md)).

**Oprávnění:** ADM.

**Mobil/desktop:** tabulka → karty (osoby, chip stavu, propadnutí).

**Notifikace:** připomínky a vypršení žádostí → `EMAIL_MERGE_REQUEST_REMINDER`, `EMAIL_MERGE_REQUEST_EXPIRED` ([notifications.md](notifications.md)).

## 6. C-05 · Porovnání, konflikty a provedení — `/ustredi/slucovani/:id`

**Účel a publikum:** obě osoby vedle sebe, rozhodnutí každého konfliktu volbou A/B a provedení — poslední krok před operací vratnou jen ADM revertem.

**Layout a komponenty:** hlavička (obě jména, chip stavu, druh); karta „Schválení stran“ (M3 list — strana, rozhodnutí, kdy); porovnávací tabulka dvou sloupců (osoba A | osoba B), řádek za slučované pole; sticky lišta s primární akcí.

**Obsah a pole:**

- Schválení stran: kdo a kdy schválil; vysvětlivka „Sloučení spojí jen osobu dítěte, účty rodičů se nespojují.“
- Porovnání — **jen základní slučovaná pole** (jméno, příjmení, přezdívka, tituly, pohlaví, datum narození, e-mail, adresa, pojišťovna); nikdy citlivá data z cizího oddílu (vysvětlivka s ikonou `lock`). Shodná pole zamčená; automaticky vyřešená (vyplněná strana vyhrává) zobrazená a zamčená; skutečné konflikty s povinnou volbou A/B (radio v každém sloupci). Ruční přepis se nenabízí: „Vybírá se vždy hodnota jedné z osob, aby šlo sloučení věrně vrátit. Opravit hodnotu lze až po sloučení editací.“
- Souhrn přenosu: „Přenese se automaticky: členství v oddílech, přihlášky, docházka, členství DU. Citlivá data a dokumenty zůstávají per oddíl.“
- Sticky lišta: „Provést sloučení“ (aktivní až po rozhodnutí všech konfliktů).

**Stavy:**

- **Čeká na strany:** porovnání jen ke čtení; banner „Žádost čeká na strany. Propadne bez odezvy [datum].“ U slabého kandidáta (chybějící datum narození) warning ikona.
- **Úspěch (provedení):** potvrzovací dialog „Sloučit [zdroj] do [cíl]? Zdrojová osoba zůstane jako náhrobek s přesměrováním — staré odkazy povedou na sjednocenou osobu.“ Po potvrzení: stav `Dokončeno`, přihlášky přejdou na cíl, snackbar „Osoby sloučeny“, badge railu klesne.
- **Chyba — blokující kolize:** mají-li obě osoby aktivní přihlášku na téže akci, dialog v `error-container`: „Sloučení nelze provést: obě osoby mají aktivní přihlášku na akci [název]. Vyřešit ji musí vedoucí akce — nic nebylo změněno.“
- **Dokončeno — revert:** detail zobrazí snapshot (stav obou osob před sloučením, rozhodnuté volby, přenesené vazby) + tlačítko „Vrátit sloučení“ (jen ADM). Dialog revertu ve dvou sloupcích: „Co se vrátí“ × „Co zůstane u cílové osoby“ (záznamy vzniklé po sloučení). Revert je jednorázový → `Vráceno`; nové sloučení = nová žádost.

**Interakce a validace:** bez rozhodnutí všech konfliktů nelze dokončit; různá data narození vyžadují zvláštní potvrzení „pravděpodobně nejde o stejnou osobu“. Směr sloučení a pravidla dědění polí dle [person-merge.md](person-merge.md).

**Oprávnění:** provedení a revert jen ADM.

**Mobil/desktop:** sloupce se hroutí na řádky „pole → hodnota A / B“; rozhodovací tlačítka sticky, mimo dosah omylu.

**Notifikace:** dle [notifications.md](notifications.md) (žádosti a jejich schvalování).

## 7. C-06 · Reporty ústředí — `/ustredi/reporty` (vč. R9)

**Účel a publikum:** jedna obálka pro reporty R1–R9 napříč oddíly s dimenzí region — podklad pro vykazování a dotace.

**Layout a komponenty:** vlevo výběr reportů (M3 list, kód + název), vpravo plocha reportu: parametry · graf · tabulka · tlačítko „Export CSV“. Sdílí vzor s oddílovými reporty ([ux-obrazovky-oddil.md](ux-obrazovky-oddil.md) § 14).

**Obsah a pole:**

- Výběr: R1 Seznam akcí a docházky · R2 Počty členů v čase · R3 Účast na akcích · R4 Docházka schůzek · R5 Dobrovolnické hodiny · R6 Retence a odchody · R7 Platby · R8 Vzdělávání · R9 Unikátní děti ([reports.md](reports.md)).
- Parametry: Oddíly (multi-select, výchozí „všechny“) · Období od–do (výchozí 12 měsíců) · Granularita (měsíc / kvartál / rok) · Region (filtruje přes snapshot akce) · Typ akce. Pod výsledkem metadata generování.
- **R9 — Unikátní děti** (vlastní podoba obrazovky): výběr **roku** (segmented, u R9 jen kalendářní roky) a velké číslo výsledku „[N] unikátních dětí“; rozpad po regionech (snapshot z akce). Poznámky: „Hosté cizích oddílů se nepočítají.“ a „Součet po regionech může být vyšší než celkem — dítě mohlo jet do dvou regionů.“ Trvalý banner `tertiary`: „Reportovací sloučení nemění žádná data — dvě osoby se počítají jako jedna jen v tomto reportu. Skutečné sloučení osob se schvalováním je v sekci Slučování osob.“ Karta **Kandidáti** (shoda jména, příjmení a data narození napříč oddíly; zobrazují se **jen tato tři pole**) s tlačítkem „Sloučit pro report“. Karta **Aktivní reportovací sloučení** (tranzitivní skupiny A–B, B–C ⇒ jedna osoba) s akcí „Zrušit“ (kdykoli — nic se nepřepsalo).
- Ostatní reporty se počítají „best effort“; metriky bez podkladu se **skryjí** (ne „0“, ne pomlčka). Export CSV = stažení ploché tabulky.

**Stavy:**

- **Prázdný:** kanonický „Za toto období nejsou data“ ([ux-texty-stavy.md](ux-texty-stavy.md) § 4). **Série s nulami není prázdný stav** — prázdné koše se kreslí jako nuly.
- **Prázdný — kandidáti R9:** ikona `verified`, „Žádní kandidáti na sloučení“, „V datech není shoda jména a data narození napříč oddíly — evidence je čistá.“ Bez CTA.
- **Úspěch (R9):** sloučení dvojice okamžitě přepočítá číslo; zrušení ho vrátí. Dvojice bez aktivní přihlášky za daný rok číslo nezmění — UI to hlásí („v čísle za rok se neprojeví“).
- **Načítání:** skeleton se zachovanou plochou grafu; **Chyba:** § 5. **Úspěch (export):** snackbar „CSV exportováno“.

**Interakce a validace:** reporty jen ke čtení (kromě reportovacího sloučení — zakládá ústředí samo, bez schvalování); změna parametru přepočítá okamžitě; cesta ke skutečnému sloučení se z R9 nenabízí. Scope se odvozuje z role, ne z parametru requestu.

**Oprávnění:** ADM (celoorganizační rozsah).

**Mobil/desktop:** výběr reportů jako chips; grafy responzivní, tabulky ve scrollu.

**Notifikace:** žádné.

## 8. C-07 · Šablony akcí a whitelist jmen — `/ustredi/sablony`

**Účel a publikum:** spravovat systémové šablony, ze kterých oddíly zakládají akce — s jistotou, že úprava nic nerozbije zpětně — a whitelist jmen, aby kontrola chytala překlepy, ne neobvyklá jména.

**Layout a komponenty:** dvě M3 taby: **Systémové šablony** · **Whitelist jmen**.

**Obsah a pole:**

- **Systémové šablony:** karty šablon (název, typ, výchozí hodnoty, přepínač Aktivní — neaktivní se nenabízí při zakládání, existující akce nedotčené). Editor = stejná rodina formulářů jako nastavení akce ([ux-obrazovky-oddil.md](ux-obrazovky-oddil.md) § 5) v režimu „výchozí hodnoty“. Trvalá poznámka: „Úprava šablony se projeví jen u nově založených akcí — akce si šablonu ukládají jako snapshot.“ (ADM vidí jen systémové šablony, ne oddílové.)
- **Whitelist jmen:** prohledávatelný seznam křestních jmen + tlačítko „Přidat jméno“. Sekce „Oddílové výjimky“ (jméno, oddíl, kdo a kdy schválil) s akcí „Povýšit do whitelistu“ (přesune jméno do seznamu, výjimka zmizí, snackbar). Vysvětlivka: „Ověřují se jen křestní jména, příjmení nikdy. Jméno mimo seznam neblokuje — oddíl může založit výjimku.“

**Stavy:**

- **Prázdný — šablony:** ikona `library_books`, „Zatím žádné systémové šablony“, „Šablony přednastavují akce všem oddílům.“, CTA „Vytvořit šablonu“.
- **Prázdný — výjimky:** ikona `fact_check`, „Žádné oddílové výjimky“, „Když oddíl schválí jméno mimo seznam, objeví se tady k případnému povýšení do whitelistu.“ Bez CTA.
- **Úspěch:** snackbar „Šablona uložena — projeví se u nových akcí“ / „Jméno přidáno“. **Načítání / Chyba:** § 5.

**Interakce a validace:** jméno unikátní bez ohledu na diakritiku a velikost písmen ([validation.md](validation.md)); editace jen ADM.

**Oprávnění:** ADM.

**Mobil/desktop:** karty v jednom sloupci; vyhledávání jmen jako primární vstup.

**Notifikace:** žádné.

## 9. C-08 · Zkrácené sekce — Vzdělávání a Audit log

- **Vzdělávání — `/ustredi/vzdelavani`** — katalog kurzů ústředí: tabulka kurz · platnost v měsících (prázdná = slovo „trvalý“) · držitelů; tlačítko „Založit kurz“ (dialog: název\*, platnost — kladné celé měsíce, nebo „trvalý“). Kurzy se vážou na vzdělávací akce ústředí; po absolvování vzniká účastníkům záznam s platností. Přehled expirací = R8 v C-06. **Prázdný stav:** ikona `school`, „Zatím žádné kurzy“, „Kurzy evidují kvalifikace vedoucích a jejich platnost. Založte první kurz.“, CTA „Založit kurz“.
- **Audit log — `/ustredi/audit`** — tabulka změnových událostí: čas · aktér (u systémových „Systém“) · akce (založení / úprava / schválení / zamítnutí / storno) · cíl · oddíl; filtr podle oddílu ([audit-log.md](audit-log.md)). **Prázdný stav:** ikona `history`, „Zatím žádné události“, text dle katalogu ([ux-texty-stavy.md](ux-texty-stavy.md) § 4). Bez CTA.

**Oprávnění:** ADM.

**Mobil/desktop:** tabulky → karty.

**Notifikace:** kurzy vážou expirace na R8; audit log sám nenotifikuje.
