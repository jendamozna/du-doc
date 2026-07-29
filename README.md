# Registrační systém oddílů DU — specifikace

## Přehled projektu

Systém přihlášek na akce pro oddíly DU. Strukturu tvoří ústředí, regiony a oddíly. Ústředí zastřešuje všechny oddíly, vede společnou členskou databázi a pořádá celostátní akce; jeho centrální agendu (správa oddílů, regiony, deduplikace, reporty, vzdělávání) zajišťují moduly ústředí. Každý oddíl spravuje vlastní akce, přihlášky a účastníky. Člen je nezávislá entita — může patřit do více oddílů současně.

**Rozsah:** Veřejný registrační portál, oddílová správa akcí, správa ústředí, self-management pro registrované.

---

## Přehled architektury

```mermaid
flowchart TD
    subgraph USTREDI["Ústředí (ADM)"]
        direction TB
        MODULY["<b>Moduly ústředí</b><br/>správa oddílů · přiřazování HVO<br/>definice a správa regionů<br/>deduplikace osob · reporty ústředí · vzdělávání"]
        USTODD["<b>Speciální oddíl ústředí</b><br/>celostátní akce · vedoucí · dobrovolníci"]
    end

    subgraph REGIONY["Regiony — verzované seskupení běžných oddílů"]
        ODDILY["<b>Běžné oddíly</b><br/>členové · hosté · družiny · dobrovolníci<br/>akce · platby · bankovní účty · chytré sloupce<br/>role: HVO, VO, VD, RÁD, ÚČE"]
    end

    PORTAL["<b>Veřejný registrační portál</b><br/>(procházet akce, přihlásit se na akci)"]
    OSOBA["<b>Osoba</b> (nezávislá entita)<br/>evidována ve více oddílech<br />1 osoba ⇢ max 1 účet"]

    MODULY --> REGIONY
    USTODD --> PORTAL
    ODDILY --> PORTAL
    PORTAL -. "registrace účtu / přihlášení do systému" .-> OSOBA

    classDef modul fill:#1e3a8a,stroke:#1e293b,stroke-width:2px,color:#ffffff;
    classDef oddil fill:#2563eb,stroke:#1e3a8a,stroke-width:1.5px,color:#ffffff;
    classDef portal fill:#059669,stroke:#065f46,stroke-width:1.5px,color:#ffffff;
    classDef osoba fill:#f1f5f9,stroke:#475569,stroke-width:1.5px,color:#0f172a;

    class MODULY,USTODD modul;
    class ODDILY oddil;
    class PORTAL portal;
    class OSOBA osoba;

    style USTREDI fill:#eef2ff,stroke:#1e3a8a,stroke-width:1px;
    style REGIONY fill:#f5f3ff,stroke:#7c3aed,stroke-width:1px;
```

---

## Požadavky

### Role

- Uživatel může být ve více rolích, např. Administrátor a zároveň jeden z vedoucích oddílu nebo dobrovolník a rodič
- Role Hlavní vedoucí oddílu (HVO), Rádce (RÁD), Vedoucí oddílu (VO), Vedoucí družiny (VD), Administrátor (ADM), Účetní (ÚČE)
- **Rodič není ukládaná role** — postavení zákonného zástupce se **odvozuje z aktivní vazby `PARENT_CHILD`** (osoba je rodič, pokud má alespoň jednu vazbu ve stavu `active`). Rozsah práv je vždy **per dítě**, ne globální; role rodiče se proto nepřiděluje ani neodebírá a nemůže se rozejít se skutečnou vazbou (zrušení vazby, přechod do `readonly_after_adulthood`).
- VO/VD nemají pevná globální práva, oprávnění se přidělují u akce / v rámci družiny.

#### Účetní

- Role, která má přístup jen k přihláškám (úpravy), akcím/cenám/stornům/bankovním účtům (čtení) a k párování/potvrzování plateb a výzvám.

#### Administrátor

- Spravuje oddíly a přiřazuje jim jejich Hlavní vedoucí
- Vytváří účty hlavním vedoucím — systém vygeneruje pozvánku e-mailem

#### Hlavní vedoucí oddílu

- Nastavuje bankovní účty
- Vytváří účty účetním, vedoucím, rádcům — systém vygeneruje pozvánku e-mailem
- Může do systému nahrát pověření od staršovstva
- Může definovat družiny, jejich vedoucí a členy
- Eviduje registrované členy (jméno, příjmení, pohlaví, datum narození)
- Eviduje hosty (min. jméno, příjmení nebo přezdívka)

#### Rádce

- Rádci nevidí citlivá data dětí, nejsou plnoletí

#### Rodič (zákonný zástupce)

- Rodič může zastupovat jedno nebo více nezletilých dětí
- Jedno dítě může být svázáno s více rodiči (oba zákonní zástupci)
- Rodič může své zastupované děti přihlašovat na akce a spravovat jejich přihlášky (přihlášení na akci, storno, platby za dítě) a údaje v systému (adresy, pojišťovny, ...)
- Vazba rodič ↔ dítě vzniká přihlášením dítěte na akci rodičem
- Po dosažení zletilosti se zastoupení rodičem přepne do režimu jen pro čtení. Výjimkou je doplnění kontaktního e-mailu dítěte, pokud chybí — slouží k doručení výzvy k převzetí účtu. Zletilý člen může přístup rodiče kdykoli zcela zrušit.
- Vazbu může zrušit sám rodič (vystoupení), případně HVO na žádost; zrušení se loguje (`AUDIT_LOG`). Zůstane-li nezletilé dítě bez navázaného rodiče, jeho údaje a přihlášky spravuje HVO, dokud se nepřipojí nový zákonný zástupce.
- Oba rodiče mají plná práva, platí poslední zápis.
- Druhého zákonného zástupce přidává stávající rodič nebo HVO pozvánkou (e-mailem). Vazba vznikne přijetím pozvánky druhým rodičem. Nemá-li dítě žádného navázaného rodiče, schvaluje připojení HVO, kde je dítě evidováno.

### Osoba vs. uživatelský účet

- Oddělujeme dvě entity:
  - **Osoba** = datový subjekt / účastník; může existovat bez přihlášení (host, nezletilé dítě spravované rodičem)
  - **Účet (uživatel)** = přihlašovací identita (heslo / OAuth), navázaná právě na jednu osobu
- Jedna osoba má nejvýše jeden účet
- **Údaje osoby** jsou pole entity `PERSON`: jméno, příjmení, přezdívka, tituly před a za jménem, pohlaví, datum narození, kontaktní e-mail, adresa trvalého bydliště a zdravotní pojišťovna. Vyplňují se podle potřeby akce (např. tituly a adresa u akcí s certifikátem); cokoli nad tento rámec patří do **chytrých sloupců** oddílu.
- **Pojmy** (důsledně v celé specifikaci):
  - **Registrace** = založení **účtu v systému** (identita osoby); _přihlášení do systému_ = následné ověření (heslo / OAuth).
  - **Přihláška na akci** = účast na konkrétní akci (entita `REGISTRATION`); _přihlásit se na akci_ = vytvořit přihlášku.
  - Slovo „přihlášení“ samotné se používá jen pro login; účast je vždy „přihláška na akci“.

#### Stav osoby (lifecycle)

- Host / registrovaný člen je **stav jedné osoby** (`PERSON_UNIT.membership_state`: `guest` / `registered_member`), nikoli samostatná entita:
  - `host → registrovaný člen` (`guest → registered_member`; migrace provedená HVO — registrovaný člen má povinné datum narození)
  - `* → neaktivní` (`PERSON_UNIT.record_state = inactive`; osoba opustila oddíl nebo je dlouhodobě bez aktivity — záznam zůstává kvůli historii, ale nezapočítává se do počtu členů a nedostává automatické výzvy)
  - `neaktivní → registrovaný člen / host` (`inactive → active`; reaktivace, pokud se osoba vrátí)
  - `* → archivovaný` (`record_state = archived`; GDPR: po uplynutí retenční doby se osobní a citlivá data anonymizují, zachovají se jen agregované/nepřímo identifikující údaje nutné pro reporting)
- **Členství DU není stav osoby** — odvozuje se z existence záznamu `DU_MEMBERSHIP` (viz **Člen DU**), takže `membership_state` žádnou hodnotu pro členy DU nemá.
- Stavy `inactive` a `archived` jsou **kolmé** na členský stav výše — členský stav drží `membership_state`, životnost záznamu `record_state` (`active` / `inactive` / `archived`).
- U každého je evidována historie — změny, přihlášky, pod jakým oddílem.

### Retence a GDPR

- Citlivá data jsou izolovaná per oddíl, každý oddíl proto maže/anonymizuje jen svoji verzi
- Administrátor může spustit výmaz napříč všemi oddíly.

> Lhůty jsou orientační a je nutné je potvrdit s DPO/právníkem (odvíjejí se od dotačních pravidel a interních směrnic).

| Kategorie dat                                               | Lhůta                                                         | Důvod / právní základ                                   |
| ----------------------------------------------------------- | ------------------------------------------------------------- | ------------------------------------------------------- |
| Citlivá data (zdravotní, alergie, léky, stravovací omezení) | smazat do 30 dnů po skončení akce                             | nutná jen pro průběh akce; minimalizace                 |
| Údaje hosta / jednorázového účastníka (nečlen)              | 12 měsíců od poslední aktivity, pak anonymizace               | žádný trvající vztah                                    |
| Členská evidence (registrovaný člen, člen DU)               | po dobu členství + 10 let                                     | doložitelnost pro dotace/kontroly (MŠMT obvykle 10 let) |
| Docházka, dobrovolnické hodiny                              | 10 let                                                        | výkaznictví k dotacím                                   |
| Účetní doklady (platby, párování, vratky)                   | 5 let (běžné), 10 let u dokladů s DPH                         | zákon o účetnictví / zákon o DPH                        |
| Souhlasy se zpracováním (GDPR)                              | po dobu zpracování + 4 roky po odvolání                       | doložení souhlasu, promlčecí doba                       |
| Úrazy / pojistné události nezletilých                       | do zletilosti dítěte + 4 roky                                 | promlčecí lhůty nároků nezletilých                      |
| Auditní logy (merge, bezpečnostní, změny)                   | 3 roky                                                        | bezpečnost, řešení sporů o spojení osob                 |
| Přihlašovací účet (neaktivní)                               | smazat/anonymizovat po 24 měsících nečinnosti (po upozornění) | minimalizace                                            |

- **Anonymizace, ne mazání** u záznamů potřebných pro agregovaný reporting (návaznost na stav `archivovaný`) — zachovají se jen nepřímo identifikující údaje (rok narození, oddíl, region-snapshot).
- **Nejdelší lhůta vyhrává:** je-li osoba zároveň člen i účastník akce s platbou, řídí se výmaz nejdelší relevantní lhůtou pro daný typ dat (citlivá data se ale mažou samostatně dřív).
- **Automatické joby:** systém periodicky označuje záznamy po expiraci a spouští anonymizaci; citlivá data mají vlastní (kratší) job.

#### Auditní log

- Změnové události napříč systémem se zapisují do jedné společné tabulky `AUDIT_LOG` — mutace hlídek, zrušení vazby rodič ↔ dítě, změny přiřazení vedoucích k akci, úpravy akce a přihlášky, posouzení dokumentů.
- Záznam nese **cíl** (`entity_type` + `entity_id`), **operaci** (`action`), **aktéra** a `detail` s tím, co se změnilo, případně důvodem. Odkaz na cílový záznam je polymorfní, tedy bez cizího klíče.
- **Aktér nemusí mít účet** — přihlášku i hlídku lze spravovat přes token, proto se eviduje `actor_account_id` (`NULL` = systémový job nebo aktér bez účtu) i `actor_email`.
- `unit_id` u záznamu umožňuje mazat a anonymizovat logy per oddíl a naplnit lhůtu **3 roky** z tabulky výše.
- Mimo tuto tabulku zůstávají tři evidence, které nejsou jen auditem: `MERGE_LOG` (nese `snapshot` pro revert sloučení), `PERSON_UNIT_HISTORY` (typované přechody stavů, čtou je reporty) a `GDPR_AUDIT` (doklad o výmazu s vlastním okruhem čtenářů).

### Deduplikace osob, merge

- Systém ověřuje správnost českých **křestních jmen** podle seznamu (spravovaného administrátorem), nabízí možnost přidání výjimky HVO v rámci oddílu. Příjmení se proti seznamu neověřují.
- Osobě s účtem se zobrazí možný kandidát na propojení (z jiného oddílu). Účet zadá Žádost o sloučení. Systém rozešle emailem žádost - iniciátorovi, HVO druhého oddílu a případně i účtu kandidáta na propojení. Po odsouhlasení všemi stranami (HVO se zobrazí pro porovnání náhled obou osob) může uživatel pokračovat se spojením: Záznamy obou osob se spojí do jedné osoby, konflikt základních polí se řeší volbou A/B, účet se naváže na sjednocenou osobu, pokud obě osoby mají účet, pak druhý účet se zruší (uživatel vybere), citlivá data zůstávají per oddíl, OAuth identity se přenesou pod ponechaný účet.
- Podobně se zpracuje duplicitní dítě, které se zobrazí rodiči s tím, že další strana je rodič dítěte kandidáta a výsledek nespojí účty rodičů do jednoho, jen osobu dítěte. Nemá-li dítě žádného navázaného rodiče, schvaluje připojení HVO, kde je dítě evidováno.
- Systém loguje, kdo kdy které osoby spojil, je možné zrušit merge pro nápravu chybného spojení.

### Člen DU

- **Členství DU je samostatný záznam, ne stav osoby** (`DU_MEMBERSHIP`): osoba je členem DU v oddílu _X_ pro rok _R_ právě tehdy, existuje-li záznam `DU_MEMBERSHIP` s touto osobou, oddílem a rokem. `PERSON_UNIT.membership_state` proto hodnotu pro člena DU nemá.
- Osoba se může stát členem DU od ledna následujícího roku po zaplacení příspěvku do listopadu — **systém platbu příspěvku neřeší**; členství pro daný rok zakládá HVO
- Členství DU trvá: leden–prosinec (kalendářní rok). **Vyprší tím, že pro nový rok záznam nevznikne** — není potřeba žádný přechod stavu ani úklidová úloha k 31. 12.
- Osoba je členem DU vždy pod konkrétním oddílem
- Kombinace osoba + rok je unikátní (jedno členství DU na osobu a rok)
- Kde se členství vyhodnocuje (cena podle typu účastníka, podmínka způsobilosti u číselníku, reporty), rozhoduje se vždy **k roku dané akce**, ne podle aktuálního data

### Region

- Region je vrstva mezi ústředím a běžnými oddíly: `Ústředí → Region → Oddíl`. Ústředí (celostátní) do regionů nepatří.
- Regiony definuje a spravuje ADM a přiřazuje do nich běžné oddíly. Oddíl je v daném okamžiku nejvýše v jednom regionu.
- **Příslušnost oddílu k regionu je verzovaná** (platnost od/do) — díky tomu lze určit, do jakého regionu oddíl patřil k libovolnému datu.
- Regiony se v čase mění:
  - **Vznik** – ADM založí nový region.
  - **Přesun oddílu** – uzavře se stávající příslušnost a založí nová do jiného regionu (bez přepisování historie).
  - **Sloučení (A + B → C)** – zdrojové regiony se označí jako _sloučené_ s odkazem na nástupnický region; všem oddílům z A i B se uzavře příslušnost a otevře nová na C.
  - **Rozdělení** – opačná operace ke sloučení.
- Regiony se **nemažou**, jen označí stavem _sloučený / zrušený_ — kvůli zachování historie.
- **Reporty (snapshot):** region oddílu/akce se zaznamenává jako **snapshot na akci v okamžiku jejího vzniku** (`region_id` se uloží k akci). Pozdější přesun oddílu nebo sloučení regionu **nemění už existující reporty**; nové akce počítají podle aktuálního zařazení. _Modul reporty ústředí_ tím získá dimenzi „region".

### Oddíl

- Typy oddílů: IČO ústředí, Pobočný spolek (vlastní IČO), kolektivní člen (bez DU v názvu, vlastní IČO)
- Ústředí je **speciální typ oddílu** určený pro celostátní akce. **Nemá registrované členy**.
- Registrace — chytré sloupce oddílu (viz níže) lze **zařadit do přihlášky na akci** jako volitelná nebo povinná pole; vyplněná hodnota se uloží k osobě
- Oddíl si vede vlastní seznam lokací (GPS souřadnice a volitelně adresa), které jsou viditelné jen v rámci oddílu; lze je přiřadit jako sídlo oddílu i jako místo konání akce

#### Družina

- Členy družiny mohou být členové, hosté, Vedoucí a Rádci z oddílu
- Družina může mít vlastní chytré sloupce nad rámec sloupců oddílu

### Přihlašování do systému

- Každý uživatel si může v systému změnit heslo
- Pro přihlášení do aplikace půjde použít účet Google nebo Facebook (OAuth)
- Jeden účet může mít více propojených OAuth identit (Google, Facebook)

### Konfigurace akce

- Hlavní vedoucí vytváří akce
- **Přiřazení vedoucích k akci** (`EVENT_ASSIGNMENT`): HVO přiřadí k akci účty (Vedoucí, Rádce, Účetní) a každému nastaví rozsah oprávnění — úprava akce (`can_edit_event`), úprava přihlášek (`can_edit_registrations`), úprava cen a storen (`can_edit_prices`), zápis docházky (`can_record_attendance`). Samo přiřazení dává **čtení přihlášek** akce; bez přiřazení k akci vedoucí přístup nemá. Na účet a akci existuje nejvýše jedno přiřazení a eviduje se, kdo a kdy je založil.
- Každá akce může být svazána s maximálně jedním bankovním účtem
- Každá akce může mít místo konání vybrané z lokací oddílu (GPS)
- Název, SS, max kapacita, počet náhradníků, ceny pro členy DU i ostatní, začátek a konec akce, začátek a konec přihlašování, termíny pro storno podmínky
- **Evidence dobrovolníků (volitelná, per akce):** je-li u akce zapnutá (`volunteers_enabled`), systém nabídne **samostatnou stránku pro přihlášení dobrovolníků** s vlastní cenou a začátkem/koncem přihlašování. Dobrovolníci se **evidují odděleně od účastníků** — mají vlastní kategorii přihlášky (`volunteer`), **nezapočítávají se do kapacity ani do počtu náhradníků** akce a vedou se ve zvláštním seznamu. Bez zapnutí se dobrovolnická stránka nenabízí.
- Náhradníci — po uvolnění místa jsou informováni vedoucí akce; po výběru náhradníka dostane náhradník časově omezenou nabídku, po vypršení propadá a vedoucí znovu vybírá.
- Akce může mít libovolný počet **výběrových číselníků** (např. ubytování, strava, doprava, stanoviště) — viz **Výběrové číselníky akce (obecný model)**
- **Viditelnost akce** (`visibility`) má tři vzájemně výlučné úrovně:
  - `public` (**veřejná**) — zobrazuje se ve veřejném výpisu portálu, přihlásit se může kdokoli,
  - `internal` (**vnitřní**) — ve výpisu není; vidí ji jen přihlášená osoba s aktivní vazbou na pořádající oddíl (u akce ústředí všichni registrovaní členové),
  - `private` (**neveřejná**) — dostupná výhradně přes sdílecí odkaz.
    Sdílecí odkaz (`share_slug`) má **každá** akce bez ohledu na úroveň — je to přístupová cesta, ne publikace ve výpisu.
- Akce může definovat **povinné dokumenty** k přihlášce; seznam a povinnost se přebírají ze **šablony akce** jako výchozí a lze je přepsat v nastavení konkrétní akce — detail a schvalovací flow viz **Přihlašování na akce**

#### Výběrové číselníky akce (obecný model)

Obecný, znovupoužitelný mechanismus: vedoucí u libovolné akce nadefinuje **libovolný počet číselníků** (`EVENT_FIELD`), z nichž si účastník při přihlášení vybírá předdefinované hodnoty. Model je společný pro různé účely (např. výběr lůžka/pokoje, turnusu, dopravy, trika, role).

- **Číselník** (`EVENT_FIELD`) patří akci a má název a množinu **položek** (`EVENT_FIELD_OPTION`) — předdefinovaných hodnot. Volitelně nese **veřejný komentář** (`comment`, popis/instrukce pro účastníka) i **neveřejnou poznámku** (`internal_note`, jen pro vedoucí).
- **Režim výběru** (`selection_mode`):
  - **exkluzivní** — položku může zvolit **nejvýše jeden účastník** (1:1, např. konkrétní lůžko); po výběru se ostatním přestane nabízet,
  - **sdílený** — stejnou položku může zvolit **více účastníků**; počet omezuje **kapacita položky** (viz níže).
- **Kapacita položky** (`EVENT_FIELD_OPTION.capacity`): max počet účastníků, kteří mohou danou položku zvolit — `1` = **unikátní** (např. konkrétní stanoviště nebo lůžko), `> 1` = **více** (např. ubytovací kapacita budovy), `NULL` = bez limitu. Po naplnění se položka přestane nabízet; exkluzivní režim odpovídá kapacitě `1` u všech položek.
- **Podmínka způsobilosti** (`condition`, `NULL` = všichni): číselník se zobrazuje a hodnoty vybírají jen účastníci, kteří podmínku splňují (např. věk, členství DU, role); ostatním se skryje.
- **Počet voleb** (`max_select`): číselník je buď **jednovýběrový** (`max_select = 1`, např. typ ubytování), nebo **vícevýběrový** (`max_select > 1` / `NULL` = bez limitu, např. výběr jídel).
- **Povinnost a fáze** (`required_phase`, `NULL` = nepovinný): u povinného číselníku určuje, kdy nejpozději musí účastník volbu provést:
  - `on_submit` — už **při odeslání přihlášky** (bez volby nelze přihlášku odeslat; výchozí),
  - `before_payment` — nejpozději **před výzvou k platbě / úhradou**,
  - `before_event` — nejpozději **před konáním akce**.
  - U **náhradníka** se povinný výběr (stejně jako dokumenty) vynucuje až **po schválení přihlášky**.
- **Cenový příplatek položky** (`EVENT_FIELD_OPTION.price_modifier`, může být `0` i záporný): výběr položky upraví cenu přihlášky. **Celková cena = základní cena podle typu účastníka (`EVENT_PRICE`) + součet příplatků zvolených položek.**
- **Výběr účastníka** je vazba přihláška ↔ položka (`REGISTRATION_FIELD_VALUE`); u vícevýběrového číselníku vznikne více vazeb.

Tím se stejným modelem pokryjí oba případy z otázky: **ubytování** (jednovýběrový číselník budova/stan, kde „budova“ nese vyšší příplatek) i **strava** (vícevýběrový číselník snídaně/oběd/večeře, každá položka s vlastní cenou).

#### Ceny a storna na akcích

- Systém umožňuje definovat více cen platných v různých termínech pro různé typy účastníků - DU, bez DU, dobrovolníky, oddílové vedoucí i děti oddílových vedoucích a sponzorské ceny
- Volitelné příplatky (ubytování, strava apod.) se modelují přes **výběrové číselníky** — každá položka může nést cenový příplatek; výsledná cena přihlášky = základní cena + součet příplatků zvolených položek (viz **Výběrové číselníky akce**)
- Systém umožňuje definovat storno poplatky procentuálně v různých termínech
- Vratky systém neřeší

#### Typy a šablony akcí

- **Typ** (`type`) je klasifikace akce (`club` / `one_off` / `weekend` / `course` / `certificate` / `recommendation` / `group` / `race` / `workshop`) — řídí větvení logiky, zapnuté subsystémy, filtrování a reporty.
- **Šablona (`ActionTemplate`)** je přednastavená konfigurace daného typu, ze které HVO/Vedoucí zakládá konkrétní akci, aby se vše nemuselo nastavovat ručně. **K jednomu typu může existovat více šablon** (systémová i vlastní oddílové s odlišnými výchozími hodnotami).
- Akce si při vzniku uloží **odkaz na šablonu (`action_template_id`) i vlastní `type`** (snapshot); pozdější úprava šablony už založené akce nemění.

**Co jednotlivé typy zapínají / vyžadují** (výchozí obsah šablony):

| Typ                             | Kód              | Zapíná / vyžaduje                                                                                                                                                                                                                               |
| ------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Pravidelné schůzky              | `club`           | **opakující se** oddílová činnost — každá schůzka je samostatná datovaná akce; zápis docházky, bez přihlášek (neotevírá registraci, neřeší cenu ani platbu)                                                                                     |
| Jednorázové akce                | `one_off`        | **neopakující se** samostatná akce (výlet, brigáda, oddílová akce) — bez přihlášek; docházku lze zapsat, cenu a platbu neřeší                                                                                                                   |
| Víkendovky / jednoosobové       | `weekend`        | obecná přihláška                                                                                                                                                                                                                                |
| Kurz                            | `course`         | vazba na nabízené kurzy ústředí                                                                                                                                                                                                                 |
| S certifikátem                  | `certificate`    | v přihlašovacím formuláři navíc tituly (před/za) a povinná adresa trvalého bydliště                                                                                                                                                             |
| S doporučením mentora/vedoucího | `recommendation` | pole na kontakty; systém osloví mentory/vedoucí o doplnění očekávání a potvrzení přihlášky                                                                                                                                                      |
| Skupinové                       | `group`          | v přihlašovacím formuláři více účastníků vč. zákonných zástupců najednou                                                                                                                                                                        |
| Stezka                          | `race`           | po přihlášení sestavení **hlídek** pro závod (jméno, kapitán, rozhodčí na stanovištích) — viz **Hlídky na závodních akcích**                                                                                                                    |
| Workshopové                     | `workshop`       | akce má několik časových **bloků**; workshopy/semináře (2 typy: `workshop` / `seminar`) se nabízejí jako **běhy** v blocích a mohou se **opakovat ve více blocích**; každý běh má vlastní kapacitu; účastník si v každém bloku vybere jeden běh |

**Šablona dále definuje:**

- **povinná a nabízená pole** přihlašovacího formuláře — která pole osoby jsou u daného typu povinná (např. tituly a trvalé bydliště u certifikátu) a **zařazení chytrých sloupců oddílu** do přihlášky jako volitelných/povinných (např. kontakty na mentora u doporučení),
- **zapnuté subsystémy** (hlídky Stezky, workshopy, doporučení mentora, více účastníků u skupinových, vazba na kurz ústředí),
- **výchozí povinné dokumenty** (např. potvrzení o lékařské způsobilosti),
- **výchozí hodnoty** cen podle typu účastníka, storno termínů, kapacity a počtu náhradníků, podpory dobrovolníků, referenčního data pro výpočet věku (**věk ke konci roku** vs. k datu akce).

- **Rozsah šablony:** systémové šablony spravuje ADM (ústředí) a jsou dostupné všem oddílům; oddíl si může nad jejich rámec založit vlastní (unit-scoped) šablony.
- Šablony jsou vstupem pro AI návrh nové akce (viz `AI_support.md`) — předvyplní název, termíny a storno podle typu.

#### Hlídky na závodních akcích (Stezka)

Akce typu **Stezka** umožní z přihlášených osob sestavit **hlídky** (družstva) pro závod. V jedné přihlášce může být přihlášeno **více osob** (účastníků); vlastník přihlášky skládá hlídky z **osob** své přihlášky a potvrzených dílčích přihlášek (**club scope** = vlastní přihláška + dílčí přihlášky/podregistrace, jejichž stav není `Canceled` / `Expired` / `New`). Hlídka se skládá z těchto osob a jedna z nich je jejím **kapitánem**.

- **Vlastnictví:** hlídku vlastní přihláška, která ji založila (`owner_registration_id`); upravovat/smazat ji smí jen vlastník. Název hlídky je v rámci akce unikátní.
- **Členství:** každá osoba je nejvýše v jedné hlídce. Jeden člen (osoba) je kapitán (`role = leader`), ostatní `member`. Při vstupu do prázdné hlídky kategorie Stezka/Pěšinka se první člen stane kapitánem automaticky; u ostatních kategorií se kapitán volí ručně.
- **Výpočet věku:** referenční datum pro výpočet věku řídí konfigurační volba akce **„věk ke konci roku"** (`age_at_year_end`). Je-li zapnutá (výchozí), věk se počítá ke **konci aktuálního roku** (31. 12.): `věk = rok(31. 12. letošního roku) − rok(datum narození)`; je-li vypnutá, počítá se k **datu konání akce**. Rozdíl let přes date diff. Chybí-li datum narození, člena nelze plně ověřit a kontrola konzistence to hlásí.
- **Kategorie a pravidla složení:**

| Kategorie         | Počet členů | Způsobilost                  | Věková pravidla                                    |
| ----------------- | ----------- | ---------------------------- | -------------------------------------------------- |
| **Stezka**        | přesně 3    | každý závodník               | nejstarší ≤ 16; součet věků ≤ 42                   |
| **Pěšinka**       | přesně 3    | každý závodník               | nejstarší ≤ 12                                     |
| **Šerpa s dětmi** | 3–4         | závodník / šerpa / dítě < 16 | právě 1 doprovod ≥ 16; 1–3 děti (věk < 4 nebo > 8) |
| **Pocestní**      | 2–3         | každý závodník               | nejmladší ≥ 9                                      |

- **Kontrola konzistence:** pravidla složení ověřuje jediná čistá funkce. Poruší-li hlídka pravidla po změně relevantního pole člena (věk, příznak závodníka, kategorie), **hlídka se rozpustí** — všichni členové se odpojí, hlídka se smaže a vlastník je informován s důvodem.
- **Připomínka (job):** N dní před akcí systém upozorní vedoucí na závodníky bez hlídky (přeskočí, pokud už dnes připomínku dostali).
- Každá mutace hlídky se **loguje** do `AUDIT_LOG` (založení, vstup, odchod, úprava, smazání; aktér = účet vedoucího, u správy přes token jeho e-mail).

##### Stanoviště a rozhodčí

Na závodních akcích se **dospělí pomocníci** (rozhodčí) přiřazují ke **stanovištím**. Stanoviště jsou modelována jako **výběrový číselník akce** (`EVENT_FIELD` s `assigned_by = leader`): jednotlivá stanoviště jsou jeho **položky** (`EVENT_FIELD_OPTION`) a přiřazení rozhodčího je `REGISTRATION_FIELD_VALUE`. Přiřazení ke stanovišti je vzájemně výlučné s členstvím v hlídce — účastník je buď závodník v hlídce, nebo dospělý na stanovišti, nikdy obojí.

- **Číselník „Stanoviště“** patří akci; přiřazení stanoviště platí jen v rámci ní.
- **Přiřazuje vedoucí** (`assigned_by = leader`) **až po přihlášení na akci** — vybírá se pouze z osob z club scope (vlastní přihláška + potvrzené dílčí přihlášky).
- **Způsobilost rozhodčího** (`condition`): osoba z club scope, dospělá (≥ 16), která není závodník ani šerpa a není v žádné hlídce.
- **Kapacita položky**: běžné stanoviště má `capacity = 1` (nejvýše jeden rozhodčí); pseudo-stanoviště „Jakékoliv“ má `capacity = NULL` (více rozhodčích). `max_select = 1` — rozhodčí je nejvýše na jednom stanovišti.
- Přiřazení je **upsert** (nejvýše jedno na osobu a akci); stanoviště s `capacity = 1` nelze obsadit, je-li už zabrané jiným rozhodčím.

#### Přihlašování na akce

- Účastník, který nemá účet získá přihláškou identifikátor (token), kterým si může účet založit (po založení se účet propojí s existující osobou) a spravovat své přihlášky (storno, měnit nebo přidávat další účastníky)
- **Nezletilý účastník (< 18 let):** věk se odvozuje z pole `datum narození` (`birth_date`). Přihlašuje-li se nezletilý sám (nemá navázaného rodiče, který přihlášku provádí), musí v přihlášce zadat **e-mail zákonného zástupce**. Systém pošle zástupci žádost o schválení; přihláška zůstává ve stavu `PendingGuardian` a nezapočítává se do kapacity, dokud zástupce neschválí (odkazem v e-mailu). Po schválení přihláška pokračuje standardním tokem (výzva k platbě apod.); neschválí-li zástupce do vypršení, přihláška expiruje. Schválením vzniká vazba rodič ↔ dítě. Chybí-li datum narození, přihlášku nelze vyhodnotit a systém e-mail zástupce vyžádá.
- **Povinné dokumenty:** akce může vyžadovat nahrání dokumentů (např. **potvrzení o lékařské způsobilosti**, souhlas zákonného zástupce, kopie kartičky pojišťovny). Po vytvoření přihlášky je každý požadovaný dokument ve stavu `pending` (očekává se nahrání). Účastník je může nahrávat **postupně nebo najednou**; dokud nejsou nahrané všechny povinné dokumenty, přihláška je ve stavu `PendingDocuments`. **Náhradník** dokumenty nahrává až **po schválení přihlášky** (po přijetí nabídky z náhradnického místa) — do té doby je upload skrytý/uzamčený.
- **Schvalovací flow dokumentů** (`pending` → `uploaded` → `approved` / `rejected`): vedoucí u každého nahraného dokumentu vidí stav a dokument buď **schválí**, nebo **zamítne s komentářem** (`review_note`) s důvodem (např. nečitelný, prošlý, nesprávný dokument). Zamítnutí přepne dokument do `rejected`, zaznamená kdo a kdy posoudil (`reviewed_by_account_id`, `reviewed_at`) a **e-mailem vyzve účastníka k opětovnému nahrání**. Přihláška zůstává (příp. se vrátí) do stavu `PendingDocuments`, dokud nejsou všechny povinné dokumenty ve stavu `approved`. Nahrání lze vyžádat i připomínkou.
- Systém posílá potvrzení přihlášky s výzvou k zaplacení (QR kód + platební údaje, pokud je stanovena cena akce)
- Systém připomíná nezaplacené platby — četnost lze upravit v Nastavení oddílu
- Systém kategorizuje přihlášky: Účastník, Dobrovolník, Náhradník
- Stavy přihlášky (pořadí podle životního cyklu): `New`, `PendingGuardian`, `PendingDocuments`, `PendingPayment`, `PartialPaid`, `Paid`, `Overpayment`, `Canceled`, `Expired`

### Docházka

- Docházka se vede **přímo na akci** (`EVENT`) — samostatná docházková událost neexistuje. Každý zápis je `ATTENDANCE_RECORD` (akce + osoba), unikátní v rámci akce.
- Vedoucí můžou vytvářet akce i zpětně — např. pravidelné kluby (typ `club`) a rovnou vybrat libovolné účastníky ze seznamu osob z oddílu
- **Klubová schůzka je akce bez přihlášek** — nemá otevřenou registraci (`registration_from`/`registration_to` prázdné, `visibility = private`), takže se u ní nespouští potvrzení přihlášky, výzvy k platbě ani připomínky. Účast se eviduje jen docházkovým záznamem.
- U akce s přihláškami jsou obě evidence odlišené: `REGISTRATION` = kdo se přihlásil, `ATTENDANCE_RECORD` = kdo se skutečně zúčastnil a kolik odpracoval.
- Záznam nese `present`, takže se rozliší **nepřítomnost** (zapsán, nedorazil) od **nezapsaného** (žádný záznam).
- Při evidenci dobrovolníku je možné zadat počet hodin — vždy na `ATTENDANCE_RECORD` téže akce
- Systém rozděluje Krátkodobé dobrovolníky (pod 50hod.) a dlouhodobé (nad 50hod.)
- Zápis docházky je **samostatné oprávnění na akci** (`EVENT_ASSIGNMENT.can_record_attendance`) — může ho mít i Rádce, který nemá přístup k přihláškám a platbám.

#### Reporty

- Seznam akcí/schůzek, docházka členů/nečlenů/vedoucích/rádců/dobrovolníků
- Počty členů v čase — vývoj registrovaných členů / členů DU / hostů po měsících nebo letech (růst/úbytek oddílu).
- Účast na akcích — kolik lidí chodí na akce v jednotlivých obdobích, naplněnost kapacit, podíl náhradníků.
- Docházka — průměrná návštěvnost pravidelných schůzek v průběhu roku (sezónní výkyvy).
- Dobrovolnické hodiny — vývoj odpracovaných hodin, poměr krátkodobých/dlouhodobých dobrovolníků.
- Retence / odchody — kolik osob přechází do neaktivní, míra reaktivací.
- Platby — vývoj inkasa, podíl včas/pozdě zaplacených, storna.
- Vzdělávání — kolik vedoucích má platné kurzy v čase, blížící se expirace.

### Volitelné moduly

- Povoluje Hlavní vedoucí pro svůj oddíl v Nastavení oddílu

#### Pomocná evidence

- Vedoucí může pro svůj oddíl nebo družinu definovat nové sloupce (do tabulky hostů/členů)
- Sloupcům lze nastavit viditelnost — zda je vlastník účtu může vidět nebo upravovat
- Sloupcům lze nastavit oprávnění — zda Rádci můžou vidět nebo upravovat

#### Modul párování plateb

- Aktivuje se doplněním tokenu k bankovnímu účtu
- Párování je M:N — jedna bankovní transakce může pokrýt více přihlášek (např. rodič platí za více dětí jednou platbou) a jedna přihláška může být uhrazena více platbami (postupné / částečné platby)
- Systém automaticky navrhuje párování podle SS=akce a VS=přihláška; když částka neodpovídá jediné přihlášce, umožní ruční rozdělení (alokaci) částky mezi více přihlášek
- U každé alokace se eviduje **způsob spárování** (`match_method`):
  - `ss_vs_amount` - shoda SS, VS i částky,
  - `ss_vs_partial` - shoda SS, VS a částečná úhrada,
  - `ss_vs_overpayment` - shoda SS, VS a přeplatek,
  - `vs_exact_name` - shoda VS, částky a jména odesílatele platby s vlastníkem přihlášky nebo poznámky platby s názvem akce,
  - `ss_exact_name` - shoda SS, částky a jména odesílatele platby s vlastníkem přihlášky,
  - `vs_partial_name` - shoda VS, částečná úhrada a shoda jména odesílatele platby s vlastníkem přihlášky nebo poznámky platby s názvem akce,
  - `vs_overpayment_name` - shoda VS, přeplatek a shoda jména odesílatele platby s vlastníkem přihlášky nebo poznámky platby s názvem akce,
  - `manual` - ruční
- Každá alokace eviduje napárovanou částku; stav přihlášky se počítá ze součtu alokací vůči ceně
- Systém automaticky posílá potvrzení za každou napárovanou platbu (i částečnou); odeslání se eviduje, aby se neposílalo dvakrát

#### Modul Potvrzení o platbě

- Vyžaduje aktivní Platební modul
- Účetní nahraje šablonu potvrzení s razítkem/podpisem v nastavení modulu — šablona je záležitost **aplikační vrstvy** (soubor šablony + konfigurace), v datovém modelu pro ni není samostatná entita
- Systém automaticky připraví potvrzení o platbě ke stažení

#### Modul reporty ústředí

- Počítá unikátní počet dětí v rámci všech akcí všech oddílů (počítá se jednou, i když bylo na více akcích)
- Lze filtrovat a agregovat podle **regionu** (region akce = snapshot uložený při vzniku akce)
- Zobrazí možné kandidáty (jméno, příjmení, datum narození). Systém nabídne "Reportovací sloučení" osob pro účely unikátních počtů, záznamy zůstanou oddělené
- nepočítá hosty ostatních oddílů

#### Modul vzdělávání

- Administrátor definuje jaké kurzy ústředí lze použít pro vzdělání vedoucích - eviduje se i doba platnosti
- Vzdělávací akce ústředí může být provázána s kurzem; po absolvování vznikne každému účastníkovi vazba s odkazem na zdrojovou akci
- Hlavní vedoucí, Vedoucí a Rádci můžou sobě přiřadit kurzy z nabídky
- Systém automaticky přiřadí kurz ústředí všem účastníkům po jeho absolvování
- Všichni Vedoucí a Rádci mají možnost vložit do systému svoje certifikáty, potvrzení od doktora a jiné absolvované kurzy
- Modul vzdělání zobrazuje Administrátorovi, jaké kurzy absolvovali jednotliví vedoucí v oddílech

---

## Datový model (ER diagram)

> Návrh schématu odvozený ze specifikace. Spojovací (M:N) a historizační tabulky jsou uvedeny zvlášť.
>
> **Aktérské cizí klíče se v diagramu nekreslí.** Pole typu `*_by_account_id`, `actor_account_id` a `initiator_account_id` zaznamenávají, kdo operaci provedl; v databázi jsou to cizí klíče na `ACCOUNT`, ale relace se nekreslí, aby diagram nezahltila hvězda čar kolem `ACCOUNT`. Nakreslené vazby na `ACCOUNT` proto značí, že je účet **předmětem** záznamu (např. `EVENT_ASSIGNMENT`), ne jeho původcem.

```mermaid
%%{init: {
  "theme": "base",
  "layout": "elk",
  "elk": {
    "nodePlacementStrategy": "LINEAR_SEGMENTS"
  },
  "themeVariables": {
    "background": "#ffffff",
    "primaryColor": "#eff6ff",
    "primaryBorderColor": "#1e3a8a",
    "primaryTextColor": "#0f172a",
    "secondaryColor": "#f5f3ff",
    "secondaryBorderColor": "#7c3aed",
    "tertiaryColor": "#ecfeff",
    "tertiaryBorderColor": "#0f766e",
    "lineColor": "#475569"
  }
}}%%

erDiagram
    REGION ||--o{ UNIT_REGION : includes
    UNIT ||--o{ UNIT_REGION : "membership (versioned)"
    UNIT ||--o{ EVENT : organizes
    UNIT ||--o{ ACTION_TEMPLATE : defines
    UNIT ||--o{ BANK_ACCOUNT : has
    UNIT ||--o{ UNIT_PATROL : has
    UNIT ||--o{ PERSON_UNIT : tracks
    UNIT ||--o{ USER_ROLE : "scoped to"
    UNIT ||--o{ CUSTOM_FIELD : defines
    UNIT ||--o{ LOCATION : defines
    UNIT ||--o{ DU_MEMBERSHIP : "du members"
    UNIT }o--o| LOCATION : "based at"
    EVENT }o--o| LOCATION : "held at"

    PERSON ||--o| ACCOUNT : has
    PERSON ||--o{ PERSON_UNIT : "tracked in"
    PERSON_UNIT ||--o{ PERSON_UNIT_HISTORY : "state changes"
    PERSON ||--o{ PARENT_CHILD : "as parent"
    PERSON ||--o{ PARENT_CHILD : "as child"
    PERSON ||--o{ UNIT_PATROL_MEMBER : is
    PERSON ||--o{ REGISTRATION : submits
    PERSON ||--o{ DU_MEMBERSHIP : has
    PERSON ||--o{ ATTENDANCE_RECORD : attends
    PERSON ||--o{ CUSTOM_FIELD_VALUE : has
    PERSON ||--o{ PERSON_COURSE : completes

    ACCOUNT ||--o{ OAUTH_IDENTITY : has
    ACCOUNT ||--o{ USER_ROLE : has

    UNIT_PATROL ||--o{ UNIT_PATROL_MEMBER : contains
    UNIT_PATROL ||--o{ CUSTOM_FIELD : scopes

    ACTION_TEMPLATE ||--o{ EVENT : "instantiated as"
    EVENT ||--o{ EVENT_ASSIGNMENT : delegates
    ACCOUNT ||--o{ EVENT_ASSIGNMENT : "assigned to"
    EVENT ||--o{ EVENT_PRICE : has
    EVENT_PRICE ||--o{ REGISTRATION : "priced by"
    EVENT ||--o{ CANCELLATION_RULE : has
    EVENT ||--o{ EVENT_FIELD : has
    EVENT ||--o{ REGISTRATION : contains
    REGISTRATION ||--o{ REGISTRATION : "sub-registrations"
    EVENT ||--o{ EVENT_DOCUMENT : requires
    EVENT ||--o{ ATTENDANCE_RECORD : attendance
    EVENT }o--o| BANK_ACCOUNT : "linked to"
    EVENT }o--o| REGION : "region snapshot"

    EVENT_FIELD ||--o{ EVENT_FIELD_OPTION : offers
    EVENT_FIELD_OPTION ||--o{ REGISTRATION_FIELD_VALUE : "chosen by"
    REGISTRATION ||--o{ REGISTRATION_FIELD_VALUE : selects
    EVENT_DOCUMENT ||--o{ REGISTRATION_DOCUMENT : "fulfilled by"
    REGISTRATION ||--o{ REGISTRATION_DOCUMENT : uploads

    EVENT ||--o{ RACE_PATROL : "race patrols"
    REGISTRATION ||--o{ RACE_PATROL : owns
    RACE_PATROL ||--o{ RACE_PATROL_MEMBER : has
    PERSON ||--o{ RACE_PATROL_MEMBER : "in patrol"
    EVENT ||--o{ WORKSHOP_BLOCK : "has blocks"
    EVENT ||--o{ WORKSHOP : offers
    WORKSHOP }o--o| LOCATION : "held at"
    WORKSHOP_BLOCK ||--o{ WORKSHOP_OFFERING : contains
    WORKSHOP ||--o{ WORKSHOP_OFFERING : "scheduled as"
    WORKSHOP_OFFERING ||--o{ WORKSHOP_REGISTRATION : has
    REGISTRATION ||--o{ WORKSHOP_REGISTRATION : "enrolled via"
    PERSON ||--o{ WORKSHOP_REGISTRATION : attends

    REGISTRATION ||--o{ PAYMENT_ALLOCATION : "paid by"
    BANK_ACCOUNT ||--o{ BANK_TRANSACTION : records
    BANK_TRANSACTION ||--o{ PAYMENT_ALLOCATION : "split into"

    CUSTOM_FIELD ||--o{ CUSTOM_FIELD_VALUE : has
    COURSE ||--o{ PERSON_COURSE : "offered as"
    COURSE ||--o{ EVENT : "awarded by"
    EVENT ||--o{ PERSON_COURSE : "completed at"

    PERSON ||--o{ CONSENT : grants
    PERSON ||--o{ GDPR_AUDIT : "subject of"
    PERSON ||--o{ PERSON_SENSITIVE_DATA : has
    UNIT ||--o{ PERSON_SENSITIVE_DATA : owns
    EVENT ||--o{ PERSON_SENSITIVE_DATA : "context of"
    PERSON ||--o{ PARENT_INVITATION : "guardian invite"
    PERSON ||--o{ RECOMMENDATION : "as mentor"
    PERSON ||--o{ REPORT_MERGE : "candidate A"
    PERSON ||--o{ REPORT_MERGE : "candidate B"
    PERSON ||--o{ MERGE_REQUEST : "as source"
    PERSON ||--o{ MERGE_REQUEST : "as target"
    MERGE_REQUEST ||--o{ MERGE_APPROVAL : "approved by"
    MERGE_REQUEST ||--o{ MERGE_LOG : "logged as"
    UNIT ||--o{ NAME_EXCEPTION : approves
    UNIT ||--o{ AUDIT_LOG : logs
    REGISTRATION ||--o{ SUBSTITUTE_OFFER : offers
    REGISTRATION ||--o{ RECOMMENDATION : requires
    UNIT ||--o{ UNIT_MODULE : enables
    UNIT ||--o{ UNIT_SETTING : has
    UNIT ||--o{ MANDATE : has
    UNIT ||--o| UNIT_MAIL_SETTING : "mail config"
    EVENT ||--o{ EVENT_CUSTOM_FIELD : collects
    CUSTOM_FIELD ||--o{ EVENT_CUSTOM_FIELD : "included in"

    REGION {
        int id PK
        string name
        string state "active / merged / cancelled"
        date valid_from
        date valid_to "NULL = aktivni"
        int merged_into_region_id FK "nastupnicky region"
    }
    UNIT {
        int id PK
        string name
        string type "hq_ico / branch / collective"
        string ico
        bool is_hq
        int location_id FK "sidlo (volitelne)"
    }
    UNIT_REGION {
        int id PK
        int unit_id FK
        int region_id FK
        date valid_from
        date valid_to "NULL = aktualni"
    }
    PERSON {
        int id PK
        string first_name
        string last_name
        string nickname
        string title_before
        string title_after
        string gender "male / female / other"
        date birth_date "povinne u registrovaneho clena"
        string email "kontaktni e-mail (nemusi byt unikatni)"
        string address "trvale bydliste"
        string insurance_company
    }
    PERSON_UNIT {
        int id PK
        int person_id FK
        int unit_id FK
        string membership_state "guest / registered_member"
        string record_state "active / inactive / archived"
        datetime valid_from
        datetime valid_to
    }
    ACCOUNT {
        int id PK
        int person_id FK "1:1"
        string login_email "prihlasovaci e-mail (unikatni)"
        string password_hash
    }
    OAUTH_IDENTITY {
        int id PK
        int account_id FK
        string provider "google / facebook"
        string provider_user_id
        bool email_verified
    }
    USER_ROLE {
        int id PK
        int account_id FK
        int unit_id FK "role scope"
        string role "HVO / VO / VD / RAD / ADM / UCE"
    }
    PARENT_CHILD {
        int id PK
        int parent_person_id FK
        int child_person_id FK
        string state "active / cancelled / readonly_after_adulthood"
        datetime valid_from
        datetime valid_to
    }
    UNIT_PATROL {
        int id PK
        int unit_id FK
        string name
    }
    UNIT_PATROL_MEMBER {
        int id PK
        int unit_patrol_id FK
        int person_id FK
        string role "leader / advisor / member"
    }
    EVENT {
        int id PK
        int unit_id FK
        int bank_account_id FK
        int region_id_snapshot FK "region pri zalozeni akce"
        int location_id FK "misto konani (volitelne)"
        int action_template_id FK "sablona (snapshot)"
        string name
        string ss "specific symbol"
        string type "club / one_off / weekend / course / certificate / recommendation / group / race / workshop"
        int course_id FK "udeluje kurz po absolvovani"
        int capacity
        int substitute_count
        string visibility "public / internal / private"
        string share_slug "neverejny sdileci odkaz"
        datetime starts_at
        datetime ends_at
        datetime registration_from
        datetime registration_to
        bool volunteers_enabled
        datetime volunteer_registration_from
        datetime volunteer_registration_to
        bool age_at_year_end "vek pocitan ke konci roku (jinak k datu akce)"
    }
    EVENT_PRICE {
        int id PK
        int event_id FK
        string membership_type "DU / non_DU / volunteer / leader / leader_child / sponsor"
        decimal amount
        date valid_from
        date valid_to
    }
    CANCELLATION_RULE {
        int id PK
        int event_id FK
        decimal percent
        date valid_until
    }
    EVENT_ASSIGNMENT {
        int id PK
        int event_id FK,UK "unikat: akce + ucet"
        int account_id FK,UK
        bool can_edit_event
        bool can_edit_registrations
        bool can_edit_prices
        bool can_record_attendance
        int assigned_by_account_id FK
        datetime assigned_at
    }
    EVENT_FIELD {
        int id PK
        int event_id FK
        string name "nazev ciselniku"
        string comment "verejny popis pro ucastnika"
        string internal_note "neverejna poznamka pro vedouci"
        string selection_mode "exclusive / shared"
        string assigned_by "self / leader"
        int max_select "max poctu voleb (NULL = bez limitu)"
        string required_phase "on_submit / before_payment / before_event (NULL = nepovinny)"
        string condition "podminka zpusobilosti (NULL = vsichni)"
    }
    EVENT_FIELD_OPTION {
        int id PK
        int event_field_id FK
        string value
        int capacity "max ucastniku na polozku (1 = unikatni; NULL = bez limitu)"
        decimal price_modifier "priplatek k zakladni cene (muze byt zaporny)"
    }
    REGISTRATION_FIELD_VALUE {
        int id PK
        int registration_id FK
        int event_field_option_id FK
    }
    EVENT_DOCUMENT {
        int id PK
        int event_id FK
        string name
        bool required
    }
    REGISTRATION_DOCUMENT {
        int id PK
        int registration_id FK
        int event_document_id FK "ktery pozadavek plni"
        string file
        string state "pending / uploaded / approved / rejected"
        string review_note "duvod zamitnuti"
        int reviewed_by_account_id FK "kdo posoudil"
        datetime uploaded_at
        datetime reviewed_at "NULL = neposouzeno"
    }
    ACTION_TEMPLATE {
        int id PK
        int unit_id FK "NULL = systemova sablona"
        string type "club / one_off / weekend / course / certificate / recommendation / group / race / workshop"
        string name
        json config "vychozi nastaveni akce"
        bool active
    }
    RACE_PATROL {
        int id PK
        int event_id FK
        int owner_registration_id FK "prihlaska, ktera hlidku zalozila"
        string name "unikatni v ramci akce"
        string category "Stezka / Pesinka / Serpa_s_detmi / Pocestni"
    }
    RACE_PATROL_MEMBER {
        int id PK
        int race_patrol_id FK
        int person_id FK "osoba z club scope"
        string role "leader (kapitan) / member"
    }
    WORKSHOP_BLOCK {
        int id PK
        int event_id FK
        string name "casovy blok akce"
        datetime starts_at
        datetime ends_at
    }
    WORKSHOP {
        int id PK
        int event_id FK
        int location_id FK "misto konani (volitelne)"
        string type "workshop / seminar"
        string name
        string description
        string instructor "lektor"
        int min_age
        string requirements "potreby"
        int capacity "max ucastniku na beh"
    }
    WORKSHOP_OFFERING {
        int id PK
        int workshop_block_id FK "casovy blok"
        int workshop_id FK "workshop / seminar"
    }
    WORKSHOP_REGISTRATION {
        int id PK
        int workshop_offering_id FK
        int registration_id FK "prihlaska (club scope)"
        int person_id FK "ucastnik z club scope"
    }
    REGISTRATION {
        int id PK
        int event_id FK
        int person_id FK
        int parent_registration_id FK "nadrazena prihlaska (NULL = hlavni); definuje club scope"
        int price_id FK
        string vs "variable symbol"
        string category "participant / volunteer / substitute"
        string state "New / PendingGuardian / PendingDocuments / PendingPayment / PartialPaid / Paid / Overpayment / Canceled / Expired"
        string guardian_email "e-mail zak. zastupce (nezletily bez rodice)"
        string guardian_approval_token
        datetime guardian_approved_at "NULL = neschvaleno"
        string token "sprava prihlasky bez uctu"
    }
    PAYMENT_ALLOCATION {
        int id PK
        int bank_transaction_id FK
        int registration_id FK
        decimal amount "alokovana cast platby"
        string matched_by "auto / manual"
        string match_method "ss_vs_amount / ss_vs_partial / ss_vs_overpayment / vs_exact_name / ss_exact_name / vs_partial_name / vs_overpayment_name / manual"
        datetime matched_at
        datetime confirmation_sent_at "NULL = neodeslano"
    }
    BANK_ACCOUNT {
        int id PK
        int unit_id FK
        string name
        string account_number
        string bank_code
        string api_token_enc
    }
    BANK_TRANSACTION {
        int id PK
        int bank_account_id FK
        string external_id UK "id transakce z banky (idempotentni import)"
        string ss
        string vs
        decimal amount
        string sender_name
        string sender_account
        string sender_bank_code
        string message
        string transaction_type
        date date "datum transakce dle banky"
        datetime imported_at "kdy ji stahl import"
    }
    DU_MEMBERSHIP {
        int id PK
        int person_id FK,UK "unikat: osoba + rok"
        int unit_id FK "oddil clenstvi"
        int year UK
    }
    ATTENDANCE_RECORD {
        int id PK
        int event_id FK,UK "unikat: akce + osoba"
        int person_id FK,UK
        bool present "false = zapsan, nedorazil"
        decimal volunteer_hours "odpracovane hodiny"
    }
    CUSTOM_FIELD {
        int id PK
        int unit_id FK
        int unit_patrol_id FK "druzina (volitelne)"
        string name
        string visibility "none / view / edit (vlastnik uctu)"
        string permission "none / view / edit (radce)"
    }
    CUSTOM_FIELD_VALUE {
        int id PK
        int custom_field_id FK
        int person_id FK
        string value
    }
    COURSE {
        int id PK
        string name
        int validity_months
    }
    LOCATION {
        int id PK
        int unit_id FK "vlastnik"
        string name
        decimal lat
        decimal lng
        string address "volitelne"
    }
    PERSON_COURSE {
        int id PK
        int person_id FK
        int course_id FK
        int source_event_id FK "vzdelavaci akce (volitelne)"
        date completed_on
        date valid_to "cache: completed_on + validity_months"
        string certificate_file
    }
    CONSENT {
        int id PK
        int person_id FK
        string type "processing / photo / health / ..."
        string purpose
        datetime granted_at
        datetime revoked_at "NULL = platny"
        date retention_until
    }
    SUBSTITUTE_OFFER {
        int id PK
        int registration_id FK
        string token
        datetime offered_at
        datetime expires_at
        string state "offered / accepted / expired"
    }
    RECOMMENDATION {
        int id PK
        int registration_id FK
        int mentor_person_id FK "NULL = jen e-mail"
        string mentor_email
        string type "mentor / leader"
        string expectation
        string state "requested / confirmed / rejected"
        datetime confirmed_at
    }
    EVENT_CUSTOM_FIELD {
        int id PK
        int event_id FK
        int custom_field_id FK "chytry sloupec zarazeny do prihlasky"
        bool required "povinne pole prihlasky"
    }
    MANDATE {
        int id PK
        int unit_id FK
        string file
        date valid_from
        date valid_to
    }
    UNIT_MODULE {
        int id PK
        int unit_id FK
        string code "payment_matching / payment_confirmation / training / custom_fields / reports"
        bool active
        json config "nastaveni modulu"
        datetime activated_at
    }
    UNIT_SETTING {
        int id PK
        int unit_id FK
        string key "napr. reminder_frequency_days"
        string value
    }
    UNIT_MAIL_SETTING {
        int id PK
        int unit_id FK
        string from_email "odesilatel (volitelne)"
        string smtp_email "e-mail pro odchozi postu"
        string smtp_password_enc "sifrovane heslo (libsodium)"
    }
    GDPR_AUDIT {
        int id PK
        string action "anonymize / purge / ..."
        int person_id FK "dotcena osoba (NULL = hromadne)"
        string scope "person / unit / guests / sensitive"
        int by_account_id FK "kdo (NULL = system)"
        string detail
        datetime created_at
    }
    AUDIT_LOG {
        int id PK
        string entity_type "race_patrol / race_patrol_member / registration / registration_document / event / event_assignment / parent_child / ..."
        int entity_id "bez FK (polymorfni)"
        string action "create / update / delete / join / leave / approve / reject / cancel"
        int unit_id FK "izolace a mazani per oddil"
        int actor_account_id FK "NULL = system nebo akter bez uctu"
        string actor_email "akter bez uctu (token)"
        json detail "co se zmenilo / duvod"
        datetime created_at
    }
    PERSON_SENSITIVE_DATA {
        int id PK
        int person_id FK
        int unit_id FK "vlastnici oddil"
        int event_id FK "kontext akce (volitelne)"
        string category "health / allergy / medication / diet"
        string content
        datetime created_at
    }
    PARENT_INVITATION {
        int id PK
        int child_person_id FK
        string email
        string token
        datetime expires
        int invited_by_account_id FK
        datetime accepted_at "NULL = nevyrizena"
        datetime created_at
    }
    NAME_WHITELIST {
        int id PK
        string name "krestni jmeno"
    }
    NAME_EXCEPTION {
        int id PK
        int unit_id FK
        string name "krestni jmeno mimo whitelist"
        int approved_by_account_id FK "schvalil HVO"
        datetime created_at
    }
    MERGE_REQUEST {
        int id PK
        string kind "person / child"
        int source_person_id FK "tombstone po slouceni"
        int target_person_id FK "vysledna osoba"
        int initiator_account_id FK
        int keep_account_id FK "ktery ucet zustava"
        string state "pending / ready / rejected / completed / reverted"
        datetime created_at
        datetime completed_at
    }
    MERGE_APPROVAL {
        int id PK
        int merge_request_id FK
        string party "initiator / candidate / hvo / parent"
        int account_id FK
        bool approved "NULL = nerozhodnuto"
        datetime decided_at
    }
    MERGE_LOG {
        int id PK
        int merge_request_id FK
        int source_person_id FK
        int target_person_id FK
        json snapshot "pro revert"
        int merged_by_account_id FK
        datetime created_at
        datetime reverted_at "NULL = platne"
    }
    REPORT_MERGE {
        int id PK
        int person_a_id FK
        int person_b_id FK
        string reason "duvod"
        datetime created_at
    }
    PERSON_UNIT_HISTORY {
        int id PK
        int person_unit_id FK
        string from_membership
        string to_membership
        string from_record
        string to_record
        string note
        int changed_by_account_id FK
        datetime changed_at
    }
```
