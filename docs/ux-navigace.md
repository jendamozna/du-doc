# UX — Navigace, informační architektura a vizuální identita

## 1. Plochy a jejich ergonomika

Aplikace má čtyři plochy ve dvou ergonomických světech:

| Plocha                | Publikum                           | Ergonomika                    | Kořenová routa  |
| --------------------- | ---------------------------------- | ----------------------------- | --------------- |
| **A** Veřejný portál  | anonym, držitelé tokenových odkazů | mobile-first, bez přihlášení  | `/`             |
| **B** Oddílová správa | HVO, VO, RÁD, ÚČE                  | desktop-first                 | `/oddil/...`    |
| **C** Správa ústředí  | ADM                                | desktop-first                 | `/ustredi/...`  |
| **D** Self-management | zákonný zástupce, osoba (self)     | mobile-first, přihlášený účet | `/muj-ucet/...` |

Mobile-first plochy (A, D): jednosloupcový layout, karty, sticky CTA, dotykové plochy min. 44 px, žádný horizontální scroll stránky. Desktop-first plochy (B, C): hustá data, tabulky s filtry, méně kliků na opakovanou akci.

## 2. Sitemap a routy

### 2.1 Plocha A — Veřejný portál

- `/` — výpis veřejných akcí s otevřeným nebo budoucím přihlašovacím oknem
- `/akce/:slug` — detail akce
- `/akce/:slug/prihlaska` — registrační formulář (vícečlenná přihláška)
- `/akce/:slug/potvrzeni` — potvrzení po odeslání
- `/stav/:token` — tokenový rozcestník správy přihlášky (stav, dokumenty, platba, storno)
- `/schvaleni/:token` — schválení přihlášky zákonným zástupcem
- `/nabidka/:token` — nabídka místa náhradníkovi

Aktér je **anonym** (veřejné stránky) nebo **držitel tokenu** (správa jedné přihlášky). Token opravňuje výhradně k operacím nad danou přihláškou — nikdy nezpřístupní seznam osob ani jiné akce.

| Operace                                       | Aktér                        | Chování bez práva                                                                            |
| --------------------------------------------- | ---------------------------- | -------------------------------------------------------------------------------------------- |
| Číst veřejný výpis akcí (`/`)                 | anonym                       | — (vždy dostupné)                                                                            |
| Číst detail veřejné akce (`/akce/:slug`)      | anonym                       | veřejná akce vždy; neveřejná jen přes `share_slug`, jinak 404 „Akci jsme nenašli“            |
| Podat přihlášku (`/akce/:slug/prihlaska`)     | anonym                       | mimo přihlašovací okno → prázdný stav „Přihlašování není otevřené“                           |
| Číst stav přihlášky (`/stav/:token`)          | držitel tokenu               | neplatný token → „Odkaz není platný“; po konci akce → „Platnost odkazu skončila“             |
| Nahrát nebo nahradit dokument                 | držitel tokenu               | jen dokumenty této přihlášky; jiný token 403                                                 |
| Zaplatit / zobrazit QR                        | držitel tokenu               | QR vždy na zbývající částku                                                                  |
| Stornovat přihlášku                           | držitel tokenu               | jen tato přihláška; koncové stavy storno nenabízejí                                          |
| Opravit `contact_email` přihlášky             | držitel tokenu               | bezpečnostní operace; zapíše se do auditu                                                    |
| Přidat dalšího účastníka (dílčí přihláška)    | držitel tokenu               | jen táž akce, jedna úroveň zanoření                                                          |
| Schválit přihlášku (`/schvaleni/:token`)      | držitel schvalovacího tokenu | token opravňuje výhradně ke schválení jedné přihlášky; po vypršení lhůty přihláška `Expired` |
| Přijmout místo náhradníka (`/nabidka/:token`) | držitel tokenu nabídky       | platnost 48 h; po vypršení „Platnost nabídky vypršela“                                       |

Držitel tokenu **není** zákonný zástupce: přidá-li anonymní držitel dalšího nezletilého účastníka, vazba nevzniká a dítě prochází standardní bránou zástupce.

### 2.2 Plocha B — Oddílová správa

Plocha je desktop-first; husté tabulky s filtry scrollují ve vlastním kontejneru a na mobilu se mění na karty. Výjimkou je docházka navržená mobile-first pro práci v terénu.

| ID   | Routa                            | Obrazovka a hlavní obsah                                                                                                                               |
| ---- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| B-01 | `/oddil`                         | Přehled oddílu: KPI, položky vyžadující pozornost a nadcházející akce. Z přehledu se pouze proklikává, mutace se provádějí až v cílové agendě.         |
| B-02 | `/oddil/akce`                    | Seznam akcí: filtry Otevřené / Připravované / Proběhlé / Zrušené, vyhledávání, obsazenost a extended FAB „Nová akce“.                                  |
| B-03 | `/oddil/akce/:id`                | Detail akce s metrikami a taby Nastavení / Přihlášky / Dokumenty / Náhradníci / Docházka. Aktivní tab je v query `?tab=...`, aby na něj šlo odkazovat. |
| B-04 | `/oddil/akce/:id?tab=nastaveni`  | Nastavení akce v sekcích Základ, Přihlašování a kapacita, Ceník, Výběrové číselníky, Povinné dokumenty, Storno pravidla a Publikace.                   |
| B-05 | `/oddil/akce/:id?tab=prihlasky`  | Tabulka přihlášek s filtry, výběrem řádků a dvěma hromadnými akcemi: připomínka platby a export výběru.                                                |
| B-06 | `/oddil/akce/:id/prihlaska/:pid` | Detail přihlášky: sdílený checklist bran, dokumenty a platba, doplněné o admin akce (posouzení dokumentu, úprava, storno, odkaz do plateb).            |
| B-07 | `/oddil/akce/:id?tab=dokumenty`  | Fronta dokumentů s filtry Čeká na posouzení / Schválené / Zamítnuté a akcemi Schválit / Zamítnout.                                                     |
| B-08 | `/oddil/akce/:id?tab=nahradnici` | Náhradníci, běžící nabídky a u závodní akce také hlídky a stanoviště.                                                                                  |
| B-09 | `/oddil/akce/:id?tab=dochazka`   | Třístavový zápis Přítomen / Nepřítomen / Nezapsáno, filtr družiny a sticky souhrn. Mobile-first.                                                       |
| B-10 | `/oddil/platby`                  | Párovací dvoupanel: fronta transakcí, detail s kandidáty, nahrání výpisu, ruční platba a řešení přeplatků.                                             |
| B-11 | `/oddil/osoby`                   | Evidence členů a hostů, filtry, vyhledávání a detail osoby se základními údaji, vazbami, členstvím a chytrými sloupci.                                 |
| B-12 | `/oddil/druziny`                 | Družiny, jejich vedoucí a členové; u zapnutého modulu závodů také správa stanovišť.                                                                    |
| B-13 | `/oddil/reporty`                 | Oddílové reporty omezené rolí, parametry období/granularity/typu akce a export CSV.                                                                    |
| B-14 | `/oddil/nastaveni`               | Základní údaje, bankovní účet, lhůty, moduly, tým a role a členské příspěvky.                                                                          |

Pravidla plochy B:

1. Oprávnění k zápisu se přidělují per akce přes `EVENT_ASSIGNMENT`; základní čtení detailu akce a seznamu přihlášených plyne z role ve vlastním oddílu.
2. Stav přihlášky se **nikdy nenastavuje ručně**. Vedoucí mění fakta (posoudí dokument, alokuje platbu, vybere náhradníka) a stav přepočítá `evaluate()`.
3. Tab Nastavení se skrývá uživateli bez práva `can_edit_event`. U akce bez přihlášek zůstává jen Docházka. U závodní akce Náhradníci obsahují také hlídky a stanoviště.
4. HVO má plný rozsah. VO a RÁD mají rozsah podle role a `EVENT_ASSIGNMENT`. ÚČE vidí Přehled, Akce ke čtení, Platby a report R7; k akcím se nepřiřazuje.
5. Platební údaje se dle role maskují: RÁD mimo funkci vedoucího akce nevidí částky; vedoucí akce vidí předepsáno, uhrazeno a zbývá, ale ne slevy, storno poplatky, přeplatky, vratky ani transakce.
6. Položka navigace mimo rozsah role se skryje. Nepovolený údaj se maskuje. Přímý vstup na nepovolenou routu vrátí 403 s ikonou `lock`, nadpisem „Sem nemáte přístup“ a CTA „Zpět na přehled“.
7. Badge u Plateb ukazuje součet nespárovaných transakcí a přeplatků čekajících na rozhodnutí. Fronty dokumentů a náhradníků jsou badge uvnitř detailu akce, ne v hlavní navigaci.

### 2.3 Plocha C — Správa ústředí

- `/ustredi` — přehled (dashboard)
- `/ustredi/oddily`, detail `/ustredi/oddily/:id` — oddíly a pozvánka HVO
- `/ustredi/regiony`, detail s historií
- `/ustredi/slucovani`, detail `/ustredi/slucovani/:id` — fronta žádostí o sloučení osob, porovnání a provedení
- `/ustredi/reporty`
- `/ustredi/sablony` — šablony akcí ústředí a whitelist jmen
- Zkrácené sekce: Vzdělávání, Audit log

Aktér je **ADM**. Čtení napříč oddíly je jen čtecí a nezahrnuje obsah souborů ani zdravotní údaje mimo akce ústředí.

| Operace                                                    | ADM | Poznámka / chování bez práva                               |
| ---------------------------------------------------------- | --- | ---------------------------------------------------------- |
| Číst přehled ústředí (`/ustredi`)                          | RW  | jiná role 403                                              |
| Spravovat oddíly a zakládat oddíl (`/ustredi/oddily`)      | RW  | HVO se přiřazuje pozvánkou                                 |
| Odeslat, znovu odeslat nebo odvolat pozvánku HVO           | RW  | pozvánka platí 14 dní                                      |
| Definovat regiony a přiřazovat oddíly (`/ustredi/regiony`) | RW  | historie se nepřepisuje                                    |
| Řídit frontu sloučení osob (`/ustredi/slucovani`)          | RW  | ADM dohlíží nad žádostmi uživatelů a stran                 |
| Zrušit potlačení zamítnuté dvojice                         | RW  | lze znovu povolit potlačenou dvojici                       |
| Provést sloučení nebo revert (`/ustredi/slucovani/:id`)    | RW  | revert smí jen ADM                                         |
| Číst obsah nahraného dokumentu                             | —   | pouze v akci ústředí nebo v nutném rozsahu pro kvalifikace |
| Reporty napříč oddíly (`/ustredi/reporty`)                 | RW  | report nikdy nevrací citlivé údaje                         |
| Systémové šablony a whitelist jmen (`/ustredi/sablony`)    | RW  | oddílové šablony ADM nevidí                                |
| Katalog kurzů a požadavky kvalifikací                      | RW  | obsah dokladu jen v nutném rozsahu                         |
| Audit log                                                  | R   | pouze čtení                                                |
| Spustit výmaz podle GDPR                                   | RW  | napříč oddíly                                              |

ADM nevidí zdravotní údaje mimo akce ústředí ani obsah dokumentů mimo akce ústředí a kvalifikační podklady.

### 2.4 Plocha D — Self-management

- `/muj-ucet/prihlasky` — moje přihlášky
- `/muj-ucet/prihlasky/:pid` — detail přihlášky (sdílená obrazovka s tokenovým rozcestníkem `/stav/:token`)
- `/muj-ucet/deti`, `/muj-ucet/deti/:id` — moje děti a detail dítěte
- `/muj-ucet/udaje` — moje údaje a souhlasy
- `/muj-ucet/ucet` — účet a zabezpečení
- `/muj-ucet/ucet/slouceni` — žádost o sloučení duplicit vlastního účtu
- `/pozvanka/:token` — veřejná stránka pozvánky druhého zákonného zástupce (mimo přihlášený shell plochy D)

Aktér je **osoba (self)** nad vlastními údaji a přihláškami a **zákonný zástupce** nad dítětem. Práva zástupce jsou per dítě a plynou z aktivní `PARENT_CHILD`; `readonly_after_adulthood` dává jen čtení a doplnění chybějícího kontaktního e-mailu.

| Operace                                                   | Osoba (self) | Zákonný zástupce         | Poznámka / chování bez práva                   |
| --------------------------------------------------------- | ------------ | ------------------------ | ---------------------------------------------- |
| Číst „Moje přihlášky“ (`/muj-ucet/prihlasky`)             | RW (vlastní) | RW (dětí)                | jen vlastní nebo navázané děti                 |
| Detail přihlášky (`/muj-ucet/prihlasky/:pid`)             | RW (vlastní) | RW (aktivní vazba)       | po zletilosti dítěte jen čtení                 |
| Nahrát nebo nahradit dokument dítěte                      | —            | RW (aktivní vazba)       | po zletilosti dítěte jen čtení                 |
| Stornovat přihlášku                                       | RW (vlastní) | RW (dětí, aktivní vazba) | koncové stavy storno nenabízejí                |
| Číst „Moje děti“ (`/muj-ucet/deti`)                       | —            | R/RW (per dítě)          | cizí dítě 403                                  |
| Upravit údaje dítěte (`/muj-ucet/deti/:id`)               | —            | RW (aktivní vazba)       | datum narození edituje jen vedoucí oddílu      |
| Zrušit vazbu na dítě                                      | —            | RW (vlastní vazba)       | změna se notifikuje druhému zástupci a HVO     |
| Pozvat druhého zástupce                                   | —            | RW (aktivní vazba)       | pozvánka platí 14 dní                          |
| Číst a měnit vlastní údaje a souhlasy (`/muj-ucet/udaje`) | RW           | RW (vlastní)             | odvolání souhlasu je self-service              |
| Účet a zabezpečení (`/muj-ucet/ucet`)                     | RW           | RW (vlastní)             | OAuth odpojit jen při jiném způsobu přihlášení |
| Žádost o sloučení duplicit (`/muj-ucet/ucet/slouceni`)    | RW (vlastní) | —                        | sloučení schvalují dotčené strany              |
| Přijmout pozvánku (`/pozvanka/:token`)                    | —            | držitel tokenu pozvánky  | přijetím vzniká vazba                          |

## 3. Vstupní body z e-mailů a tokenů

Pro zástupce, náhradníky a anonymní žadatele je e-mail první třídou vstupu do systému, ne okrajová cesta:

- `/schvaleni/:token` — schválení přihlášky zákonným zástupcem (z `EMAIL_REG_CONFIRM_MINOR` apod.)
- `/nabidka/:token` — nabídka místa náhradníkovi, platnost 48 h
- `/stav/:token` — rozcestník přihlášky, hlavní cesta bez účtu (token je klíč — nepřeposílat)
- `/pozvanka/:token` — pozvánka druhého zákonného zástupce

Token a přihlášený účet jsou dvě rovnocenné cesty ke stejnému obsahu (`/stav/:token` ≈ `/muj-ucet/prihlasky/:pid`), ne nouzová varianta jedna druhé.

## 4. Navigační komponenty

| Plocha | Horní lišta                                      | Hlavní navigace                                                                                          | Poznámka                          |
| ------ | ------------------------------------------------ | -------------------------------------------------------------------------------------------------------- | --------------------------------- |
| A      | logo/název „Dorostová unie · akce"               | žádná — lineární flow + zpět                                                                             | žádný login                       |
| B, C   | top app bar s titulkem sekce, hledáním, avatarem | M3 navigation rail (≥ 1240 px expanded drawer, 600–1239 px rail, < 600 px modal drawer + bottom app bar) | položky navigace viz sitemap výše |
| D      | top app bar                                      | M3 navigation bar dole na mobilu / rail na desktopu, obsah max ~720 px                                   | mobile-first jako plocha A        |

### 4.1 Breadcrumbs

Breadcrumbs orientují uživatele v hierarchii desktopových ploch a nabízejí skok o úroveň výš:

- **Plochy B a C** mají breadcrumbs vždy, pod top app barem. Kořen je název sekce z navigace, poslední článek je aktuální stránka (není odkaz). Taby detailu akce se do breadcrumbs **nepromítají** — jsou to podřízené záložky, ne úrovně cesty.
- **Plocha D** breadcrumbs nemá — je mobile-first a plochá; návrat řeší M3 navigation bar a tlačítko zpět.
- **Plocha A** breadcrumbs nemá — je lineární flow; návrat řeší šipka zpět v top app baru. Tokenové stránky (`/stav/:token`, `/schvaleni/:token`, `/nabidka/:token`, `/pozvanka/:token`) breadcrumbs nikdy nemají — nejsou zasazené do žádného shellu.
- Článek breadcrumbs, na který uživatel nemá právo, se vynechá; cesta se nikdy nezobrazí jako mrtvý odkaz.
- Dynamický název (akce, oddíl, osoba) je uveden **snapshotem názvu**, ne ID.

Vzory cest:

| Routa                            | Breadcrumbs                             |
| -------------------------------- | --------------------------------------- |
| `/oddil/akce/:id?tab=prihlasky`  | Akce → _název akce_                     |
| `/oddil/akce/:id/prihlaska/:pid` | Akce → _název akce_ → _jméno účastníka_ |
| `/oddil/osoby` (detail)          | Osoby → _jméno osoby_                   |
| `/ustredi/oddily/:id`            | Oddíly → _název oddílu_                 |
| `/ustredi/regiony` (detail)      | Regiony → _název regionu_               |
| `/ustredi/slucovani/:id`         | Slučování osob → _dvojice jmen_         |

## 5. Společná pravidla oprávnění

Role vycházejí z [authorization.md](authorization.md#aktéři) — ADM, HVO, VO, RÁD, ÚČE, zákonný zástupce (odvozený z aktivní `PARENT_CHILD`, ne přiřaditelná role), vlastník přihlášky (token nebo účet), osoba (self), anonym.

Chování při absenci oprávnění je jednotné napříč plochami: prvek mimo rozsah role se **skryje**; údaj, na jehož existenci má uživatel právo, ale nesmí znát hodnotu, se **maskuje** (`———`); read-only režim ponechá hodnoty a skryje zápisové akce; přímý vstup na nepovolenou routu vrátí **403** (ikona `lock`, „Sem nemáte přístup", CTA „Zpět na přehled"). Scope se nikdy nebere z parametru requestu — validuje se proti tomu, co aktérovi náleží ([authorization.md](authorization.md#pravidla-vyhodnocení)).

## 6. Vizuální identita — design tokeny

### 6.1 Barevné tokeny

Primární barva: lesní zelená `#2E6B4F`. Světlý i tmavý režim (dle systému + ruční přepínač v horní liště):

| Token                  | Světlý    | Tmavý     |
| ---------------------- | --------- | --------- |
| primary                | `#2E6B4F` | `#95D5B0` |
| on-primary             | `#FFFFFF` | `#00391F` |
| primary-container      | `#B2F1CC` | `#0F5237` |
| on-primary-container   | `#002112` | `#B2F1CC` |
| secondary              | `#4D6357` | `#B4CCBB` |
| secondary-container    | `#CFE9D8` | `#364B3F` |
| on-secondary-container | `#0A1F14` | `#CFE9D8` |
| tertiary               | `#3C6472` | `#A4CDDD` |
| tertiary-container     | `#BFE9FA` | `#234C59` |
| surface                | `#F6FBF4` | `#0F1512` |
| surface-container      | `#EAEFE8` | `#1B211D` |
| on-surface             | `#171D19` | `#DEE4DD` |
| outline                | `#707972` | `#8A938B` |
| error                  | `#BA1A1A` | `#FFB4AB` |
| error-container        | `#FFDAD6` | `#93000A` |

Pozadí `body` vždy `surface`. Kontrast textu min. 4,5:1.

### 6.2 Stavové barvy přihlášek a dokumentů

Stav přihlášky se všude zobrazuje jako M3 assist/filter chip (tonální, bez obrysu, ikona vlevo):

| Stav (identifikátor) | Česká nálepka              | Světlý (pozadí/text)                   | Tmavý (pozadí/text) | Ikona            |
| -------------------- | -------------------------- | -------------------------------------- | ------------------- | ---------------- |
| `New`                | Nová                       | `#E1E3E0`/`#44483F`                    | `#44483F`/`#E1E3E0` | fiber_new        |
| `PendingGuardian`    | Čeká na zákonného zástupce | `#EADDFF`/`#4F378B`                    | `#4F378B`/`#EADDFF` | family_restroom  |
| `PendingDocuments`   | Čeká na dokumenty          | `#FFDF9E`/`#5F4300`                    | `#5F4300`/`#FFDF9E` | description      |
| `PendingPayment`     | Čeká na platbu             | `#D8E2FF`/`#00458F`                    | `#00458F`/`#D8E2FF` | account_balance  |
| `PartialPaid`        | Částečně zaplaceno         | `#C2E8FF`/`#004C68`                    | `#004C68`/`#C2E8FF` | hourglass_bottom |
| `Paid`               | Zaplaceno                  | `#B2F1CC`/`#005230`                    | `#005230`/`#B2F1CC` | check_circle     |
| `Overpayment`        | Přeplatek                  | `#BFE9FA`/`#1F4C5A`                    | `#1F4C5A`/`#BFE9FA` | trending_up      |
| `Canceled`           | Stornována                 | `#FFDAD6`/`#93000A`                    | `#93000A`/`#FFDAD6` | cancel           |
| `Expired`            | Expirovaná                 | `#E1E3E0`/`#5C5F5A` (přeškrtnutý text) | `#3A3F3B`/`#AEB3AD` | schedule         |

Stejné barvy platí pro stavy dokumentů: čeká na posouzení = paleta `PendingDocuments`, schválen = paleta `Paid`, zamítnut = paleta `Canceled`.

### 6.3 Typografie, tvary, elevace

- **Roboto** (Google Fonts), fallback `system-ui, sans-serif`. M3 type scale: display-small 36/44 (hero portálu) · headline-small 24/32 (nadpisy stránek) · title-large 22 (karty akcí) · title-medium 16/500 (sekce) · body-large 16 (formuláře, portál) · body-medium 14 (admin, tabulky) · label-large 14/500 (tlačítka, čipy).
- Tvary: karty a dialogy 12 px, tlačítka plně zaoblená (pill), čipy 8 px, bottom sheets 28 px horní rohy, textová pole outlined 4 px.
- Elevace střídmě: karty level 1, sticky lišty level 2, dialogy/menu level 3. Rozlišovat plochami (`surface-container`), ne stíny.

### 6.4 Mapování komponent

- **Top app bar** — portál: center-aligned/small; admin: small s titulkem sekce a akcemi.
- **Navigation rail / drawer** — jen plochy B a C; portál a self-management navigaci v tomto smyslu nemají (lineární flow + zpět, resp. bottom bar/rail dle § 4).
- **FAB** — jen admin: „Nová akce" na seznamu akcí (extended FAB s ikonou add).
- **Karty (elevated/filled)** — akce na portálu, souhrny, karty přihlášek a dětí (plocha D), KPI karty (plocha C).
- **Čipy** — filter chips nad tabulkami (stav, akce, typ osoby), assist chips pro stavy (§ 6.2), input chips pro vybrané položky číselníků.
- **Datové tabulky** — admin: hustá M3 tabulka, řádek 52 px, sticky hlavička, checkbox výběr pro hromadné úkony, řazení klikem na hlavičku; vždy scroll ve vlastním kontejneru.
- **Dialogy** — potvrzení destruktivních akcí (storno, zamítnutí dokumentu — vždy s polem důvodu), náhledy „e-mailů".
- **Snackbary** — výsledek každé mutace, s akcí Zpět tam, kde jde vrátit.
- **Bottom sheets** — na mobilu místo dialogů/menu: filtry tabulek, detail platby, výběr položky číselníku.
- **Ostatní** — outlined text fields, segmented buttons (přepínač období ceníku), linear progress + skeletony pro načítání, badge s počtem na položkách rail (fronta dokumentů, nespárované platby).
