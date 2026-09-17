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

### 2.2 Plocha B — Oddílová správa

- `/oddil` — přehled
- `/oddil/akce`, `/oddil/akce/:id` (taby Nastavení / Přihlášky / Dokumenty / Náhradníci / Docházka)
- `/oddil/platby` — jen persony s přístupem k platební agendě (HVO, ÚČE)
- `/oddil/osoby` — evidence osob oddílu
- `/oddil/druziny` — družiny a závodní hlídky
- `/oddil/reporty`
- `/oddil/nastaveni`

Dvě zásady platí napříč celou plochou B: oprávnění se přidělují **per akce** (`EVENT_ASSIGNMENT`), a stav přihlášky se **nikdy nenastavuje ručně** — vedoucí mění fakta (schválí dokument, alokuje platbu), stav se přepočítá funkcí `evaluate()`.

### 2.3 Plocha C — Správa ústředí

- `/ustredi` — přehled (dashboard)
- `/ustredi/oddily`, detail `/ustredi/oddily/:id` — oddíly a pozvánka HVO
- `/ustredi/regiony`, detail s historií
- `/ustredi/slucovani`, detail `/ustredi/slucovani/:id` — fronta žádostí o sloučení osob, porovnání a provedení
- `/ustredi/reporty`
- `/ustredi/sablony` — šablony akcí ústředí a whitelist jmen
- Zkrácené sekce: Vzdělávání, Audit log

### 2.4 Plocha D — Self-management

- `/muj-ucet/prihlasky` — moje přihlášky
- `/muj-ucet/prihlasky/:pid` — detail přihlášky (sdílená obrazovka s tokenovým rozcestníkem `/stav/:token`)
- `/muj-ucet/deti`, `/muj-ucet/deti/:id` — moje děti a detail dítěte
- `/muj-ucet/udaje` — moje údaje a souhlasy
- `/muj-ucet/ucet` — účet a zabezpečení
- `/muj-ucet/ucet/slouceni` — žádost o sloučení duplicit vlastního účtu
- `/pozvanka/:token` — veřejná stránka pozvánky druhého zákonného zástupce (mimo přihlášený shell plochy D)

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

## 5. Obrazovka × role (mapa oprávnění)

Role vycházejí z [authorization.md](authorization.md#aktéři) — ADM, HVO, VO, RÁD, ÚČE, zákonný zástupce (odvozený z aktivní `PARENT_CHILD`, ne přiřaditelná role), vlastník přihlášky (token nebo účet), osoba (self), anonym.

| Plocha / routa                                         | Přístup                                                                     | Poznámka                                                           |
| ------------------------------------------------------ | --------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| `/`, `/akce/:slug`, `/akce/:slug/prihlaska`            | anonym                                                                      | veřejný výpis a podání přihlášky                                   |
| `/stav/:token`, `/schvaleni/:token`, `/nabidka/:token` | držitel tokenu                                                              | vlastník přihlášky bez nutnosti účtu                               |
| `/oddil/...`                                           | HVO (plný rozsah), VO/RÁD (akce a přihlášky vlastního oddílu dle přiřazení) | viz [authorization.md](authorization.md#akce-a-jejich-konfigurace) |
| `/oddil/platby`                                        | HVO, ÚČE                                                                    | ostatní role bez přístupu k platební agendě                        |
| `/ustredi/...`                                         | ADM                                                                         | napříč všemi oddíly                                                |
| `/muj-ucet/...`                                        | osoba (self), zákonný zástupce                                              | per dítě, ne globálně                                              |

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
