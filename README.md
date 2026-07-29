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
        ODDILY["<b>Běžné oddíly</b><br/>členové · hosté · družiny · dobrovolníci<br/>akce · platby · bankovní účty · chytré sloupce<br/>role: HVO, VO, VD, RÁD, ÚČE, ROD"]
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
- Role Hlavní vedoucí oddílu (HVO), Rádce (RÁD), Vedoucí oddílu (VO), Vedoucí družiny (VD), Administrátor (ADM), Účetní (ÚČE), Rodič (ROD)
- VO/VD nemají pevná globální práva, oprávnění se přidělují u akce / v rámci družiny.

#### Účetní

- Role, která má přístup jen k přihláškám (úpravy), akcím/cenám/stornům/bankovním účtům (čtení) a k párování/potvrzování plateb a výzvám.

#### Administrátor

- Spravuje oddíly a přiřazuje jim jejich Hlavní vedoucí
- Vytváří účty hlavním vedoucím - system vygeneruje pozvánku emailem

#### Hlavní vedoucí oddílu

- Nastavuje bankovní účty
- Vytváří účty účetním, vedoucím, rádcům - system vygeneruje pozvánku emailem
- Může do systému nahrát pověření od staršovstva
- Může definovat družiny, jejich vedoucí a členy
- Eviduje registrované členy (jméno, příjmení, pohlaví, datum narození)
- Eviduje hosty (min. jméno, příjmení nebo přezdívka)

#### Rádce

- Rádci nevidí citlivá data dětí, nejsou plnoletí

#### Rodič (zákonný zástupce)

- Rodič je osoba, která má vazbu na alespoň jedno dítě (typicky nezletilé)
- Rodič může zastupovat jedno nebo více nezletilých dětí
- Jedno dítě může být svázáno s více rodiči (oba zákonní zástupci)
- Rodič může své zastupované děti přihlašovat na akce a spravovat jejich přihlášky (přihlášení na akci, storno, platby za dítě) a údaje v systému (adresy, pojišťovny, ...)
- Vazba rodič ↔ dítě vzniká přihlášením dítěte na akci rodičem
- Po dosažení zletilosti se zastoupení rodičem přepne do režimu jen pro čtení. Výjimkou je doplnění kontaktního e-mailu dítěte, pokud chybí — slouží k doručení výzvy k převzetí účtu. Zletilý člen může přístup rodiče kdykoli zcela zrušit.
- Vazbu může zrušit sám rodič (vystoupení), případně HVO na žádost; zrušení se loguje. Zůstane-li nezletilé dítě bez navázaného rodiče, jeho údaje a přihlášky spravuje HVO, dokud se nepřipojí nový zákonný zástupce.
- Oba rodiče mají plná práva, platí poslední zápis.
- Druhého zákonného zástupce přidává stávající rodič nebo HVO pozvánkou (e-mailem). Vazba vznikne přijetím pozvánky druhým rodičem. Nemá-li dítě žádného navázaného rodiče, schvaluje připojení HVO, kde je dítě evidováno.

### Osoba vs. uživatelský účet

- Oddělujeme dvě entity:
  - **Osoba** = datový subjekt / účastník; může existovat bez přihlášení (host, nezletilé dítě spravované rodičem)
  - **Účet (uživatel)** = přihlašovací identita (heslo / OAuth), navázaná právě na jednu osobu
- Jedna osoba má nejvýše jeden účet
- **Pojmy** (důsledně v celé specifikaci):
  - **Registrace** = založení **účtu v systému** (identita osoby); _přihlášení do systému_ = následné ověření (heslo / OAuth).
  - **Přihláška na akci** = účast na konkrétní akci (entita `REGISTRATION`); _přihlásit se na akci_ = vytvořit přihlášku.
  - Slovo „přihlášení“ samotné se používá jen pro login; účast je vždy „přihláška na akci“.

#### Stav osoby (lifecycle)

- Host / registrovaný člen / člen DU je **stav jedné osoby**, nikoli samostatná entita:
  - `host → registrovaný člen` (migrace provedená HVO - Registrovaný člen má povinné datum narození)
  - `registrovaný člen → člen DU`
  - `člen DU → registrovaný člen` (automatický přechod koncem roku, pokud nebyl zaplacen příspěvek na další rok — členství DU vyprší 31. 12.)
  - `* → neaktivní` (osoba opustila oddíl nebo dlouhodobě bez aktivity; záznam zůstává kvůli historii, ale nezapočítává se do počtu členů a nedostává automatické výzvy)
  - `neaktivní → registrovaný člen / host` (reaktivace, pokud se osoba vrátí)
  - `* → archivovaný` (GDPR: po uplynutí retenční doby se osobní a citlivá data anonymizují; zachovají se jen agregované/nepřímo identifikující údaje nutné pro reporting)
- Stavy `neaktivní` a `archivovaný` jsou kolmé na členský stav výše — určují, zda je záznam živý, uspaný, nebo anonymizovaný.
- U každého je evidována historie - změny, přihlášky, pod jakým oddílem

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

### Deduplikace osob, merge

- system oveřuje správnost českých jmen podle seznamu (spravovaného administrátorem), nabízí možnost přidáni vyjímky HVO v rámci oddílu.
- Osobě s účtem se zobrazí možný kandidát na propojení (z jiného oddílu). Účet zadá Žádost o sloučení. Systém rozešle emailem žádost - iniciátorovi, HVO druhého oddílu a případně i účtu kandidáta na propojení. Po odsouhlasení všemi stranami (HVO se zobrazí pro porovnání náhled obou osob) může uživatel pokračovat se spojením: Záznamy obou osob se spojí do jedné osoby, konflikt základních polí se řeší volbou A/B, účet se naváže na sjednocenou osobu, pokud obě osoby mají účet, pak druhý účet se zruší (uživatel vybere), citlivá data zůstávají per oddíl, OAuth identity se přenesou pod ponechaný účet.
- Podobně se zpracuje duplicitní dítě, které se zobrazí rodiči s tím, že další strana je rodič dítěte kandidáta a výsledek nespojí účty rodičů do jednoho, jen osobu dítěte. Nemá-li dítě žádného navázaného rodiče, schvaluje připojení HVO, kde je dítě evidováno.
- Systém loguje, kdo kdy které osoby spojil, je možné zrušit merge pro nápravu chybného spojení.

### Člen DU

- Je vlastnost osoby
- Osoba se může stát členem DU od ledna následujícího roku po zaplacení příspěvku do listopadu
- Členství DU trvá: leden–prosinec (kalendářní rok)
- Osoba je členem DU vždy pod konkrétním oddílem
- Kombinace osoba + rok je unikátní (jedno členství DU na osobu a rok)

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
- Registrace - chytré sloupce oddílu (viz níže) lze **zařadit do přihlášky na akci** jako volitelná nebo povinná pole; vyplněná hodnota se uloží k osobě
- Oddíl si vede vlastní seznam lokací (GPS souřadnice a volitelně adresa), které jsou viditelné jen v rámci klubu; lze je přiřadit jako sídlo oddílu i jako místo konání akce

#### Družina

- Členy družiny mohou být členové, hosté, Vedoucí a Rádci z oddílu
- Družina může mít vlastní chytré sloupce nad rámec sloupců oddílu

### Přihlašování do systému

- Každý uživatel si může v systému změnit heslo
- Každý si může vytvořit účet v systému a v něm editovat svojí identitu, kterou může použít při dalších přihláškách na akce
- Pro přihlášení do aplikace půjde použít účet Google nebo Facebook (OAuth)
- jeden účet může mít více propojených OAuth identit (Google, Facebook)

### Konfigurace akce

- Hlavní vedoucí vytváří akce
- Hlavní vedoucí přiřazuje k akcím Vedoucí - získají přístup k přihláškám, nastaví jim uroveň oprávnění, zda můžou editovat akci, přihlášky, ceny, atd.
- Každá akce může být svazána s maximálně jedním bankovním účtem
- Každá akce může mít místo konání vybrané z lokací oddílu (GPS)
- Název, SS, max kapacita, počet náhradníků, ceny pro členy DU i ostatní, začátek a konec akce, začátek a konec přihlašování, termíny pro storno podmínky
- **Evidence dobrovolníků (volitelná, per akce):** je-li u akce zapnutá (`volunteers_enabled`), systém nabídne **samostatnou stránku pro přihlášení dobrovolníků** s vlastní cenou a začátkem/koncem přihlašování. Dobrovolníci se **evidují odděleně od účastníků** — mají vlastní kategorii přihlášky (`volunteer`), **nezapočítávají se do kapacity ani do počtu náhradníků** akce a vedou se ve zvláštním seznamu. Bez zapnutí se dobrovolnická stránka nenabízí.
- Náhradníci - po uvolnění místa jsou informováni vedoucí akce, po výběru náhradníka, náhradník dostane časově omezenou nabídku, po vypršení propadá a vedoucí znovu vybírá.
- Akce může mít libovolný počet **výběrových číselníků** s předdefinovanými hodnotami, ze kterých si účastníci za daných podmínek vybírají (exkluzivně = hodnotu zvolí jen jeden účastník, nebo sdíleně = stejnou hodnotu více účastníků); položka může nést cenový příplatek — viz **Výběrové číselníky akce (obecný model)**
- Akce může být veřejná, vnitřní nebo neveřejná. Každá akce má neveřejnou adresu (sdílecí odkaz), kterou lze sdílet a přihlašovat se přes ni, aniž by se akce publikovala ve veřejném výpisu. Veřejná akce se navíc zobrazuje ve veřejném výpisu portálu, interní akce je viditelná pouze po prihlášeným osobám, kteří jsou členy klubu.
- Akce může definovat **povinné dokumenty**, které musí účastník k přihlášce nahrát (např. potvrzení o lékařské způsobilosti); u každého dokumentu se určí název a zda je povinný. Seznam a povinnost se přebírají z **šablony akce (ActionTemplate)** jako výchozí a lze je **přepsat v nastavení konkrétní akce**. U náhradníka se upload zpřístupní až po schválení přihlášky.

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
- Systém umožnuje definovat storno poplatky procentuálně v různých termínech
- Vratky systém neřeší

#### Typy a šablony akcí

- **Typ** (`type`) je klasifikace akce (`club` / `one_off` / `weekend` / `course` / `certificate` / `recommendation` / `group` / `race` / `workshop`) — řídí větvení logiky, zapnuté subsystémy, filtrování a reporty.
- **Šablona (`ActionTemplate`)** je přednastavená konfigurace daného typu, ze které HVO/Vedoucí zakládá konkrétní akci, aby se vše nemuselo nastavovat ručně. **K jednomu typu může existovat více šablon** (systémová i vlastní oddílové s odlišnými výchozími hodnotami).
- Akce si při vzniku uloží **odkaz na šablonu (`action_template_id`) i vlastní `type`** (snapshot); pozdější úprava šablony už založené akce nemění.

**Co jednotlivé typy zapínají / vyžadují** (výchozí obsah šablony):

| Typ                             | Kód              | Zapíná / vyžaduje                                                                                                                                                                                                                               |
| ------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Pravidelné kluby                | `club`           | zápis docházky; akce bez přihlášek (neotevírá registraci, neřeší cenu ani platbu) — každá schůzka je samostatná datovaná akce                                                                                                                    |
| Jednorázové akce                | `one_off`        | bez potřeby přihlášky                                                                                                                                                                                                                           |
| Víkendovky / jednoosobové       | `weekend`        | obecná přihláška                                                                                                                                                                                                                                |
| Kurz                            | `course`         | vazba na nabízené kurzy ústředí                                                                                                                                                                                                                 |
| S certifikátem                  | `certificate`    | v přihlašovacím formuláři navíc tituly (před/za) a povinná adresa trvalého bydliště                                                                                                                                                             |
| S doporučením mentora/vedoucího | `recommendation` | pole na kontakty; systém osloví mentory/vedoucí o doplnění očekávání a potvrzení přihlášky                                                                                                                                                      |
| Skupinové                       | `group`          | v přihlašovacím formuláři více účastníků vč. zákonných zástupců najednou                                                                                                                                                                        |
| Stezka                          | `race`           | po přihlášení sestavení **hlídek** pro závod (jméno, kapitán, rozhodčí na stanovištích) — viz **Hlídky na závodních akcích**                                                                                                                    |
| Workshopové                     | `workshop`       | akce má několik časových **bloků**; workshopy/semináře (2 typy: `workshop` / `seminar`) se nabízejí jako **běhy** v blocích a mohou se **opakovat ve více blocích**; každý běh má vlastní kapacitu; účastník si v každém bloku vybere jeden běh |

**Šablona dále definuje:**

- **povinná a nabízená pole** přihlašovacího formuláře (**zařazení chytrých sloupců oddílu** do přihlášky jako volitelné/povinné — např. tituly a trvalé bydliště u certifikátu, kontakty na mentora u doporučení),
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
- Každá mutace hlídky se **loguje** (založení, vstup, odchod, úprava, smazání; aktér = e-mail vedoucího).

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
- **Povinné dokumenty:** akce může vyžadovat nahrání dokumentů (např. **potvrzení o lékařské způsobilosti**, souhlas zákonného zástupce, kopie kartičky pojišťovny). Účastník je může nahrávat **postupně nebo najednou**; dokud nejsou nahrané všechny povinné dokumenty, přihláška je ve stavu `PendingDocuments` (čeká na nahrání povinných dokumentů). **Náhradník** dokumenty nahrává až **po schválení přihlášky** (po přijetí nabídky z náhradnického místa) — do té doby je upload skrytý/uzamčený.
- **Schvalovací flow dokumentů:** vedoucí u každého nahraného dokumentu vidí stav (`uploaded` / `approved` / `rejected`) a dokument buď **schválí**, nebo **zamítne s komentářem** (`review_note`) s důvodem (např. nečitelný, prošlý, nesprávný dokument). Zamítnutí přepne dokument do `rejected`, zaznamená kdo a kdy posoudil (`reviewed_by_account_id`, `reviewed_at`) a **e-mailem vyzve účastníka k opětovnému nahrání**. Přihláška zůstává (příp. se vrátí) do stavu `PendingDocuments`, dokud nejsou všechny povinné dokumenty ve stavu `approved`. Nahrání lze vyžádat i připomínkou.
- Systém posílá potvrzení přihlášky s výzvou k zaplacení (QR kód + platební údaje, pokud je stanovena cena akce)
- Systém připomíná nezaplacené platby - četnost lze upravit v Nastavení oddílu
- Systém kategorizuje přihlášky: Učastník, Dobrovolník, Náhradník
- Stavy přihlášky: Paid, New, Canceled, PartialPaid, Overpayment, PendingPayment, PendingGuardian, PendingDocuments, Expired

### Docházka

- Docházka se vede **přímo na akci** (`EVENT`) — samostatná docházková událost neexistuje. Každý zápis je `ATTENDANCE_RECORD` (akce + osoba), unikátní v rámci akce.
- Vedoucí můžou vytvářet akce i zpětně - např. pravidelné kluby (typ `club`) a rovnou vybrat libovolné účastníky ze seznamu osob z oddílu
- **Klubová schůzka je akce bez přihlášek** — nemá otevřenou registraci (`registration_from`/`registration_to` prázdné, `public = false`), takže se u ní nespouští potvrzení přihlášky, výzvy k platbě ani připomínky. Účast se eviduje jen docházkovým záznamem.
- U akce s přihláškami jsou obě evidence odlišené: `REGISTRATION` = kdo se přihlásil, `ATTENDANCE_RECORD` = kdo se skutečně zúčastnil a kolik odpracoval.
- Záznam nese `present`, takže se rozliší **nepřítomnost** (zapsán, nedorazil) od **nezapsaného** (žádný záznam).
- Při evidenci dobrovolníku je možné zadat počet hodin — vždy na `ATTENDANCE_RECORD` téže akce, takže hodiny z klubů i z víkendovek tečou jedním kanálem a report je sčítá z jednoho místa.
- Systém rozděluje Krátkodobé dobrovolníky (pod 50hod.) a dlouhodobé (nad 50hod.)
- Zápis docházky je **samostatné oprávnění na akci** — může ho mít i Rádce, který nemá přístup k přihláškám a platbám.

#### Reporty

- Seznam akcí/výprav, docházka členů/nečlenů/vedoucích/rádců/dobrovolníků
- Počty členů v čase — vývoj registrovaných členů / členů DU / hostů po měsících nebo letech (růst/úbytek oddílu).
- Účast na akcích — kolik lidí chodí na akce v jednotlivých obdobích, naplněnost kapacit, podíl náhradníků.
- Docházka klubů — průměrná návštěvnost pravidelných klubů v průběhu roku (sezónní výkyvy).
- Dobrovolnické hodiny — vývoj odpracovaných hodin, poměr krátkodobých/dlouhodobých dobrovolníků.
- Retence / odchody — kolik osob přechází do neaktivní, míra reaktivací.
- Platby — vývoj inkasa, podíl včas/pozdě zaplacených, storna.
- Vzdělávání — kolik vedoucích má platné kurzy v čase, blížící se expirace.

### Volitelné moduly

- Povoluje Hlavní vedoucí pro svůj oddíl v Nastavení oddílu

#### Pomocná evidence

- Vedoucí může pro svůj oddíl nebo družinu definovat nové sloupce (do tabulky hostů/členů)
- Sloupcům lze nastavit viditelnost - zda je vlastník účtu může vidět nebo upravovat
- Sloupcům lze nastavit oprávnění - zda Rádci můžou vidět nebo upravovat

#### Modul párování plateb

- Aktivuje se doplněním tokenu k bankovnímu účtu
- Párování je M:N — jedna bankovní transakce může pokrýt více přihlášek (např. rodič platí za více dětí jednou platbou) a jedna přihláška může být uhrazena více platbami (postupné / částečné platby)
- Systém automaticky navrhuje párování podle SS=akce a VS=přihláška; když částka neodpovídá jediné přihlášce, umožní ruční rozdělení (alokaci) částky mezi více přihlášek
- U každé alokace se eviduje **způsob spárování** (`match_method`):
  - `ss_vs_amount` - shoda SS, VS i částky,
  - `ss_vs_partial` - shoda SS, VS a čáštečná úhrada,
  - `ss_vs_overpayment` - shoda SS, VS a přeplatek,
  - `vs_exact_name` - shoda VS, částky a jména odesílatele platby s vlastníkem přihlášky nebo poznámky platby s názvem akce,
  - `ss_exact_name` - shoda SS, částky a jména odesílatele platby s vlastníkem přihlášky,
  - `vs_partial_name` - shoda VS, čáštečná úhrady a shoda jména odesílatele platby s vlastníkem přihlášky nebo poznámky platby s názvem akce,
  - `vs_overpayment_name` - shoda VS, přeplatek a shoda jména odesílatele platby s vlastníkem přihlášky nebo poznámky platby s názvem akce,
  - `manual` - ruční
- Každá alokace eviduje napárovanou částku; stav přihlášky se počítá ze součtu alokací vůči ceně
- Systém automaticky posílá potvrzení za každou napárovanou platbu (i částečnou); odeslání se eviduje, aby se neposílalo dvakrát

#### Modul Potvrzení o platbě

- Vyžaduje aktivní Platební modul
- Účetní do systému zadá systému šablonu s razítkem/podpisem
- Systém automaticky připraví potvrzení o platbě ke stažení

#### Modul reporty ústředí

- Počítá unikátni počet dětí v rámci všech akcí všech oddílů (počítá se jednou, ikdyž bylo na více akcích)
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
    UNIT ||--o{ PATROL : has
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
    PERSON ||--o{ PATROL_MEMBER : is
    PERSON ||--o{ REGISTRATION : submits
    PERSON ||--o{ DU_MEMBERSHIP : has
    PERSON ||--o{ ATTENDANCE_RECORD : attends
    PERSON ||--o{ CUSTOM_FIELD_VALUE : has
    PERSON ||--o{ PERSON_COURSE : completes

    ACCOUNT ||--o{ OAUTH_IDENTITY : has
    ACCOUNT ||--o{ USER_ROLE : has
    ACCOUNT ||--o{ REGISTRATION_DOCUMENT : reviews

    PATROL ||--o{ PATROL_MEMBER : contains
    PATROL ||--o{ CUSTOM_FIELD : scopes

    ACTION_TEMPLATE ||--o{ EVENT : "instantiated as"
    EVENT ||--o{ EVENT_PRICE : has
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
    EVENT }o--o{ PERSON_SENSITIVE_DATA : "context of"
    PERSON ||--o{ PARENT_INVITATION : "guardian invite"
    PERSON ||--o{ RECOMMENDATION : "as mentor"
    PERSON ||--o{ REPORT_MERGE : "candidate A"
    PERSON ||--o{ REPORT_MERGE : "candidate B"
    PERSON ||--o{ MERGE_REQUEST : "as source"
    PERSON ||--o{ MERGE_REQUEST : "as target"
    MERGE_REQUEST ||--o{ MERGE_APPROVAL : "approved by"
    MERGE_REQUEST ||--o{ MERGE_LOG : "logged as"
    UNIT ||--o{ NAME_EXCEPTION : approves
    ACCOUNT ||--o{ GDPR_AUDIT : "performed by"
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
        date valid_to "NULL = active"
        int merged_into_region_id FK "successor region"
    }
    UNIT {
        int id PK
        string name
        string type "hq_ico / branch / collective"
        string ico
        bool is_hq
        int location_id FK "GPS umisteni (volitelne)"
    }
    UNIT_REGION {
        int id PK
        int unit_id FK
        int region_id FK
        date valid_from
        date valid_to "NULL = current"
    }
    PERSON {
        int id PK
        string first_name
        string last_name
        string nickname
        string gender "male / female / other"
        date birth_date "required for registered member"
        string email "kontaktni e-mail (nemusi byt unikatni)"
    }
    PERSON_UNIT {
        int id PK
        int person_id FK
        int unit_id FK
        string membership_state "guest / registered_member / du_member"
        string record_state "active / inactive / archived"
        datetime valid_from
        datetime valid_to
    }
    ACCOUNT {
        int id PK
        int person_id FK "1:1"
        string login_email "prihlasovaci e-mail (unikatni); nezavisly na kontaktnim PERSON.email"
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
        string role "HVO / VO / VD / RAD / ADM / UCE / ROD"
    }
    PARENT_CHILD {
        int id PK
        int parent_person_id FK
        int child_person_id FK
        string state "active / cancelled / readonly_after_adulthood"
        datetime valid_from
        datetime valid_to
    }
    PATROL {
        int id PK
        int unit_id FK
        string name
    }
    PATROL_MEMBER {
        int id PK
        int patrol_id FK
        int person_id FK
        string role "leader / advisor / member"
    }
    EVENT {
        int id PK
        int unit_id FK
        int bank_account_id FK
        int region_id_snapshot FK "region at event creation"
        int location_id FK "GPS umisteni konani (volitelne)"
        int action_template_id FK "sablona pri zalozeni akce (snapshot)"
        string name
        string ss "specific symbol"
        string type "club / one_off / weekend / course / certificate / recommendation / group / race / workshop"
        int course_id FK "udeluje kurz po absolvovani (jen vzdelavaci akce)"
        int capacity
        int substitute_count
        bool public
        string share_slug "neverejny sdileci odkaz (bez publikace)"
        datetime starts_at
        datetime ends_at "konec akce (rizeni retence citlivych dat, vypocet veku k datu akce)"
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
    EVENT_FIELD {
        int id PK
        int event_id FK
        string name "nazev ciselniku"
        string comment "verejny popis / instrukce pro ucastnika (NULL = zadny)"
        string internal_note "neverejna poznamka pro vedouci (NULL = zadna)"
        string selection_mode "exclusive (1 ucastnik) / shared (vice ucastniku)"
        string assigned_by "self (ucastnik) / leader (vedouci; napr. stanoviste rozhodcich)"
        int max_select "kolik polozek smi ucastnik zvolit (1 = jedna; NULL = bez limitu)"
        string required_phase "kdy je vyber povinny: on_submit / before_payment / before_event (NULL = nepovinny)"
        string condition "podminka zpusobilosti (vek / DU / role; NULL = vsichni)"
    }
    EVENT_FIELD_OPTION {
        int id PK
        int event_field_id FK
        string value "preddefinovana hodnota"
        int capacity "max ucastniku na polozku: 1 = unikatni (napr. stanoviste), >1 = vice (napr. kapacita budovy), NULL = bez limitu"
        decimal price_modifier "priplatek k zakladni cene (0 = bez vlivu; muze byt zaporny)"
    }
    REGISTRATION_FIELD_VALUE {
        int id PK
        int registration_id FK
        int event_field_option_id FK
    }
    EVENT_DOCUMENT {
        int id PK
        int event_id FK
        string name "napr. potvrzeni o lekarske zpusobilosti"
        bool required
    }
    REGISTRATION_DOCUMENT {
        int id PK
        int registration_id FK
        int event_document_id FK "ktery pozadavek plni"
        string file
        string state "pending / uploaded / approved / rejected"
        string review_note "komentar vedouciho pri zamitnuti (duvod; NULL = bez poznamky)"
        int reviewed_by_account_id FK "kdo schvalil/zamitl (NULL = neposouzeno)"
        datetime uploaded_at
        datetime reviewed_at "NULL = neposouzeno"
    }
    ACTION_TEMPLATE {
        int id PK
        int unit_id FK "NULL = systemova sablona (ADM), jinak vlastni oddilu"
        string type "club / one_off / weekend / course / certificate / recommendation / group / race / workshop"
        string name
        json config "povinna pole, zapnute subsystemy, vychozi ceny/storna/kapacita, vek ke konci roku, povinne dokumenty"
        bool active
    }
    RACE_PATROL {
        int id PK
        int event_id FK
        int owner_registration_id FK "vlastnik hlidky = prihlaska, ktera ji zalozila (jen ten smi upravit/smazat)"
        string name "unikatni v ramci akce"
        string category "Stezka / Pesinka / Serpa_s_detmi / Pocestni"
    }
    RACE_PATROL_MEMBER {
        int id PK
        int race_patrol_id FK
        int person_id FK "clen hlidky = osoba z club scope prihlasky vlastnika"
        string role "leader (kapitan) / member"
    }
    WORKSHOP_BLOCK {
        int id PK
        int event_id FK
        string name "casovy blok akce, napr. dopoledni / odpoledni"
        datetime starts_at
        datetime ends_at
    }
    WORKSHOP {
        int id PK
        int event_id FK
        int location_id FK "GPS (volitelne)"
        string type "workshop / seminar"
        string name
        string description
        string instructor "lektor"
        int min_age "min vek"
        string requirements "potreby"
        int capacity "max ucastniku (plati pro kazdy beh workshopu)"
    }
    WORKSHOP_OFFERING {
        int id PK
        int block_id FK "casovy blok, ve kterem se workshop/seminar bezi"
        int workshop_id FK "ktery workshop/seminar (muze se opakovat ve vice blocich)"
    }
    WORKSHOP_REGISTRATION {
        int id PK
        int workshop_offering_id FK
        int registration_id FK "prihlaska (club scope), pres kterou je ucastnik zapsan"
        int person_id FK "konkretni ucastnik z club scope prihlasky"
    }
    REGISTRATION {
        int id PK
        int event_id FK
        int person_id FK
        int parent_registration_id FK "nadrazena prihlaska pro dilci prihlasky/podregistrace (NULL = samostatna/hlavni); definuje club scope"
        int price_id FK
        string vs "variable symbol"
        string category "participant / volunteer / substitute"
        string state "New / PendingGuardian / PendingDocuments / PendingPayment / PartialPaid / Paid / Overpayment / Canceled / Expired"
        string guardian_email "e-mail zak. zastupce (nezletily <18 bez rodice; NULL jinak)"
        string guardian_approval_token "token pro schvaleni zastupcem"
        datetime guardian_approved_at "NULL = neschvaleno"
        string token "management without account"
    }
    PAYMENT_ALLOCATION {
        int id PK
        int bank_transaction_id FK
        int registration_id FK
        decimal amount "alokovana cast platby"
        string matched_by "auto / manual"
        string match_method "ss_vs_amount / ss_vs_partial / ss_vs_overpayment / vs_exact_name / ss_exact_name / vs_partial_name / vs_overpayment_name / manual"
        datetime matched_at
        datetime confirmation_sent_at "potvrzeni prijate sparovane platby odeslano (NULL = neodeslano)"
    }
    BANK_ACCOUNT {
        int id PK
        int unit_id FK
        string name
        string account_number "cislo uctu"
        string bank_code "kod banky"
        string api_token_enc
    }
    BANK_TRANSACTION {
        int id PK
        int bank_account_id FK
        string ss
        string vs
        decimal amount
        string sender_name "nazev odesilatele"
        string sender_account "ucet odesilatele"
        string sender_bank_code "kod banky odesilatele"
        string message "zprava pro prijemce"
        string transaction_type "typ transakce"
        date date
    }
    DU_MEMBERSHIP {
        int id PK
        int person_id FK,UK "unikat: osoba + oddil + rok"
        int unit_id FK "oddil, pres ktery je osoba clenem DU"
        int year UK
    }
    ATTENDANCE_RECORD {
        int id PK
        int event_id FK,UK "unikat: akce + osoba"
        int person_id FK,UK
        bool present "false = zapsan, ale nedorazil; zadny zaznam = nezapsan"
        decimal volunteer_hours "odpracovane hodiny (jen dobrovolnicka ucast)"
    }
    CUSTOM_FIELD {
        int id PK
        int unit_id FK
        int patrol_id FK "optional"
        string name
        string visibility "none / view / edit (vlastnik uctu)"
        string permission "none / view / edit (radce)"
    }
    CUSTOM_FIELD_VALUE {
        int id PK
        int field_id FK
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
        int unit_id FK "vlastnik (klub) - viditelne per klub"
        string name
        decimal lat "zemepisna sirka"
        decimal lng "zemepisna delka"
        string address "volitelne"
    }
    PERSON_COURSE {
        int id PK
        int person_id FK
        int course_id FK
        int source_event_id FK "z jake vzdelavaci akce (volitelne)"
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
        datetime revoked_at "NULL = valid"
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
        int mentor_person_id FK "NULL = email only"
        string mentor_email
        string type "mentor / leader"
        string expectation
        string state "requested / confirmed / rejected"
        datetime confirmed_at
    }
    EVENT_CUSTOM_FIELD {
        int id PK
        int event_id FK
        int custom_field_id FK "chytry sloupec oddilu/druziny zarazeny do prihlasky"
        bool required "true = povinne pole prihlasky, false = volitelne"
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
        json config "module-specific settings"
        datetime activated_at
    }
    UNIT_SETTING {
        int id PK
        int unit_id FK
        string key "e.g. reminder_frequency_days"
        string value
    }
    UNIT_MAIL_SETTING {
        int id PK
        int unit_id FK
        string from_email "odesilatel (volitelne)"
        string smtp_email "Google email pro odchozi komunikaci"
        string smtp_password_enc "sifrovane heslo aplikaci (libsodium)"
    }
    GDPR_AUDIT {
        int id PK
        string action "anonymize / purge / ..."
        int person_id FK "affected person (NULL = bulk)"
        string scope "person / unit / guests / sensitive"
        int by_account_id FK "who (NULL = system)"
        string detail
        datetime created_at
    }
    PERSON_SENSITIVE_DATA {
        int id PK
        int person_id FK
        int unit_id FK "vlastnici oddil (izolace a mazani per oddil)"
        int event_id FK "kontext akce (volitelne); rizeni retence - mazano do 30 dnu po skonceni akce"
        string category "health / allergy / medication / diet"
        string content "obsah (mazano po retencni lhute)"
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
        string name
        string kind "first / last"
    }
    NAME_EXCEPTION {
        int id PK
        int unit_id FK
        string name
        string kind "first / last"
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
        string reason "reporting merge (records stay separate)"
        datetime created_at
    }
    PERSON_UNIT_HISTORY {
        int id PK
        int person_unit_id FK
        string from_membership "predchozi stav clenstvi"
        string to_membership "novy stav clenstvi"
        string from_record "predchozi record_state"
        string to_record "novy record_state"
        string note
        int changed_by_account_id FK
        datetime changed_at
    }
```
