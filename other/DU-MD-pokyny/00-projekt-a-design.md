# 00 · Projekt a design systém — DEMO registračního systému Dorostové unie

## 1. Co stavíme

Postav **klikatelné DEMO** registračního systému **Dorostové unie (DU)** — organizace zastřešující regiony a dětské oddíly. Systém řeší přihlašování osob (hlavně dětí) na akce oddílů: přihlášku, schválení zákonným zástupcem, povinné dokumenty, platbu QR převodem a správu toho všeho vedoucími oddílu.

**Toto je demo bez backendu.** Zásadní technická pravidla:

- **Žádná reálná databáze, žádná reálná autentizace, žádné reálné e-maily ani platby.** Celá aplikace běží nad **in-memory mock vrstvou** naplněnou daty ze souboru `01-demo-data.md`. Nestavěj Supabase/DB/auth — jen typovaný in-memory store (např. React context / Zustand).
- Mock vrstva se chová **realisticky**: akce (odeslání přihlášky, schválení dokumentu, alokace platby, storno) mění data ve store a UI se okamžitě přepočítá — včetně stavu přihlášky podle životního cyklu (viz § 6). Změny žijí jen po dobu session (refresh = reset na výchozí demo data).
- Kde by systém poslal e-mail, demo zobrazí **snackbar** „Demo: e-mail odeslán — [název šablony]“ a e-mailové obrazovky (schválení zástupcem, nabídka náhradníkovi…) jsou dostupné jako běžné stránky přes tokenové odkazy z demo dat.
- Simuluj krátké latence (200–500 ms) kvůli loading stavům, ale ne delší.

**Jazyk a formáty (závazné):** celé UI výhradně **česky**. Částky `1 250 Kč` (mezera jako oddělovač tisíců, desetinná čárka). Datum a čas `Středa 29.7. 14:19` — den v týdnu s velkým písmenem, bez úvodních nul, **rok jen když se liší od aktuálního** (`Středa 29.7.2025`). Měna jen CZK, časové pásmo Europe/Prague.

**Terminologie (nezaměňovat):**
- **registrace** = založení uživatelského účtu; **přihlášení** = login do systému; **přihláška na akci** = záznam účasti osoby na akci (vlastní životní cyklus). „Přihlásit se na akci“ = vytvořit přihlášku.
- **osoba ≠ účet** — děti a hosté běžně existují bez účtu; přihláška bez účtu se spravuje **tokenovým odkazem** z e-mailu (normální cesta, ne nouzová).
- **oddíl** (základní jednotka), **ústředí** (centrála DU), **HVO** (hlavní vedoucí oddílu), **VO/VD** (vedoucí oddílu/družiny), **RÁD** (rádce), **ÚČE** (účetní), **zákonný zástupce** (odvozeno z vazby rodič–dítě, ne přidělovaná role).

## 2. Rozsah dema

Demo pokrývá **plnou aplikaci — všechny čtyři plochy**:

| Plocha | Rozsah v demu |
|---|---|
| **A · Veřejný portál** | Plně — výpis akcí, detail akce, registrační formulář, potvrzení, schválení zástupcem, tokenový rozcestník přihlášky, dokumenty, platba QR, nabídka náhradníkovi |
| **B · Oddílová správa** | Plně — akce a šablony, konfigurace, tabulka a detail přihlášek, fronta dokumentů, náhradníci, párování plateb (ÚČE), docházka, evidence osob, družiny a hlídky, reporty, nastavení oddílu |
| **C · Správa ústředí** | Plně — přehled, oddíly a pozvánky HVO, regiony s historií, slučování osob, reporty R1–R9, šablony a whitelist jmen, vzdělávání, audit log |
| **D · Self-management** | Plně — moje přihlášky se sdíleným detailem, děti a pozvánka druhého zástupce, údaje a souhlasy, účet a zabezpečení, sloučení duplicit |

Detailní specifikace obrazovek ploch A–D jsou v navazujících souborech; tento soubor definuje rámec, navigaci a design systém.

## 3. Informační architektura a navigace

Aplikace má **dva ergonomické světy**: mobile-first (plochy A a D) a desktop-first (plochy B a C):

### 3.1 Veřejný portál (plocha A) — mobile-first, bez přihlášení
- Publikum: rodiče na telefonu, hosté, lidé z e-mailového odkazu. Žádný login není potřeba.
- Layout: jednosloupcový, karty, sticky CTA, dotykové plochy min. 44 px, žádný horizontální scroll stránky (tabulky scrollují ve vlastním kontejneru).
- Routy: `/` (výpis akcí) · `/akce/:slug` (detail) · `/akce/:slug/prihlaska` (formulář) · `/akce/:slug/potvrzeni` · `/stav/:token` (tokenový rozcestník přihlášky — stav, dokumenty, platba, storno) · `/schvaleni/:token` (schválení zástupcem) · `/nabidka/:token` (nabídka místa náhradníkovi).
- Horní lišta: logo/název „Dorostová unie · akce“, vpravo přepínač role (§ 3.3).
- Stejnou ergonomii (jeden sloupec, karty, dotykové plochy ≥ 44 px) sdílí **plocha D — Self-management** (`/muj-ucet/...`, M3 navigation bar dole na mobilu / rail na desktopu, obsah max ~720 px) — detail v `05-self-management.md`; mimo její shell patří veřejná tokenová stránka `/pozvanka/:token`.

### 3.2 Oddílová správa (plocha B) — desktop-first
- Publikum: HVO, VO/VD, RÁD, ÚČE — vracející se uživatelé, hustá data, tabulky s filtry, méně kliků.
- Layout: **M3 navigation rail** (≥1240 px expanded drawer, 600–1239 px rail, <600 px modal drawer + bottom app bar). Obsahová plocha s top app bar (titulek sekce, hledání, avatar persony).
- Položky navigace (`/oddil/...`): **Přehled** · **Akce** (`/oddil/akce`, detail `/oddil/akce/:id` s taby Nastavení / Přihlášky / Dokumenty / Náhradníci / Docházka) · **Platby** (`/oddil/platby`, jen persona ÚČE a HVO) · **Osoby** (`/oddil/osoby`) · **Družiny a závody** (`/oddil/druziny`) · **Reporty** (`/oddil/reporty`) · **Nastavení oddílu** (`/oddil/nastaveni`).
- Dvě zásady prostupují vším: **oprávnění se přidělují per akce** a **stav přihlášky se nikdy nenastavuje ručně** — vedoucí mění fakta (schválí dokument, alokuje platbu) a stav se přepočítá.
- Stejný admin vzor (rail, tabulky, top app bar) používá **plocha C — Správa ústředí** (`/ustredi/...`: Přehled · Oddíly · Regiony · Slučování osob · Reporty · Šablony a whitelist · Vzdělávání · Audit log) — detail v `04-ustredi.md`.

### 3.3 Přepínač role (jen demo mechanika)
Demo nemá reálné přihlášení. V horní liště je vždy **menu „Role“** (ikona osoby + jméno persony), kterým se přepíná pohled:

| Persona | Kdo | Kam vede |
|---|---|---|
| Návštěvník | bez přihlášení | Veřejný portál `/` |
| Martin Dvořáček — HVO oddílu Severka | hlavní vedoucí | Oddílová správa `/oddil` (plný rozsah) |
| Ivana Šmídková — účetní | ÚČE | `/oddil/platby` (omezený rozsah: Přehled, Akce jen ke čtení, Platby) |
| Pavla Konvalinková — rodič s účtem | zákonná zástupkyně | Self-management `/muj-ucet` (plocha D) |
| Bohdana Krejcárková — administrátorka ústředí | ADM | Správa ústředí `/ustredi` (plocha C) |

Přepnutí role jen změní aktivní personu ve store a přesměruje — žádné heslo, žádný OAuth. Do UI napiš drobnou vysvětlivku „Demo režim — role se přepíná bez přihlášení“.

## 4. Design systém — Material 3

Používej **Material 3** (M3) komponenty a tokeny. Decentní, důvěryhodný vzhled pro mládežnickou organizaci: klidná lesní zelená, hodně bílé plochy, zaoblené tvary.

### 4.1 Barevné tokeny

**Primární barva: lesní zelená `#2E6B4F`.** Světlý i tmavý režim (přepínání dle systému + ruční toggle v horní liště):

| Token | Světlý | Tmavý |
|---|---|---|
| primary | `#2E6B4F` | `#95D5B0` |
| on-primary | `#FFFFFF` | `#00391F` |
| primary-container | `#B2F1CC` | `#0F5237` |
| on-primary-container | `#002112` | `#B2F1CC` |
| secondary | `#4D6357` | `#B4CCBB` |
| secondary-container | `#CFE9D8` | `#364B3F` |
| on-secondary-container | `#0A1F14` | `#CFE9D8` |
| tertiary | `#3C6472` | `#A4CDDD` |
| tertiary-container | `#BFE9FA` | `#234C59` |
| surface | `#F6FBF4` | `#0F1512` |
| surface-container | `#EAEFE8` | `#1B211D` |
| on-surface | `#171D19` | `#DEE4DD` |
| outline | `#707972` | `#8A938B` |
| error | `#BA1A1A` | `#FFB4AB` |
| error-container | `#FFDAD6` | `#93000A` |

Pozadí `body` vždy `surface`. Kontrast textu min. 4,5:1.

### 4.2 Stavové barvy přihlášek
Stav přihlášky se všude zobrazuje jako **M3 assist/filter chip** (tonální, bez obrysu, ikona vlevo). Barvy (světlý režim `pozadí/text`, tmavý `pozadí/text`):

| Stav (identifikátor) | Česká nálepka | Světlý | Tmavý | Ikona |
|---|---|---|---|---|
| `New` | Nová | `#E1E3E0`/`#44483F` | `#44483F`/`#E1E3E0` | fiber_new |
| `PendingGuardian` | Čeká na zákonného zástupce | `#EADDFF`/`#4F378B` | `#4F378B`/`#EADDFF` | family_restroom |
| `PendingDocuments` | Čeká na dokumenty | `#FFDF9E`/`#5F4300` | `#5F4300`/`#FFDF9E` | description |
| `PendingPayment` | Čeká na platbu | `#D8E2FF`/`#00458F` | `#00458F`/`#D8E2FF` | account_balance |
| `PartialPaid` | Částečně zaplaceno | `#C2E8FF`/`#004C68` | `#004C68`/`#C2E8FF` | hourglass_bottom |
| `Paid` | Zaplaceno | `#B2F1CC`/`#005230` | `#005230`/`#B2F1CC` | check_circle |
| `Overpayment` | Přeplatek | `#BFE9FA`/`#1F4C5A` | `#1F4C5A`/`#BFE9FA` | trending_up |
| `Canceled` | Stornována | `#FFDAD6`/`#93000A` | `#93000A`/`#FFDAD6` | cancel |
| `Expired` | Expirovaná | `#E1E3E0`/`#5C5F5A` + přeškrtnutý text | `#3A3F3B`/`#AEB3AD` | schedule |

Stejné barvy platí pro stavy dokumentů: čeká na posouzení = `PendingDocuments` paleta, schválen = `Paid` paleta, zamítnut = `Canceled` paleta.

### 4.3 Typografie, tvary, elevace
- **Roboto** (Google Fonts) s fallbackem `system-ui, sans-serif`. M3 type scale: display-small 36/44 (hero portálu), headline-small 24/32 (nadpisy stránek), title-large 22 (karty akcí), title-medium 16/500 (sekce), body-large 16 (formuláře, portál), body-medium 14 (admin, tabulky), label-large 14/500 (tlačítka, čipy).
- Tvary: karty a dialogy 12 px, tlačítka plně zaoblená (pill), čipy 8 px, bottom sheets 28 px horní rohy, textová pole outlined 4 px.
- Elevace střídmě: karty level 1, sticky lišty level 2, dialogy/menu level 3. Rozlišuj plochami (`surface-container`), ne stíny.

### 4.4 Mapování komponent
- **Top app bar** — portál: center-aligned/small; admin: small s titulkem sekce a akcemi.
- **Navigation rail / drawer** — jen admin (§ 3.2); portál navigaci nemá (lineární flow + zpět).
- **FAB** — jen admin: „Nová akce“ na seznamu akcí (extended FAB s ikonou add).
- **Karty (elevated/filled)** — akce na portálu, souhrny, karty přihlášek a dětí (plocha D), KPI karty (plocha C).
- **Čipy** — filter chips nad tabulkami (stav, akce, typ osoby), assist chips pro stavy (§ 4.2), input chips pro vybrané položky číselníků.
- **Datové tabulky** — admin: hustá M3 tabulka, řádek 52 px, sticky hlavička, checkbox výběr pro hromadné úkony, řazení klikem na hlavičku; vždy scroll ve vlastním kontejneru.
- **Dialogy** — potvrzení destruktivních akcí (storno, zamítnutí dokumentu — vždy s polem důvodu), náhledy „e-mailů“.
- **Snackbary** — výsledek každé mutace („Přihláška odeslána“, „Dokument schválen“, „Demo: e-mail odeslán“), s akcí Zpět tam, kde jde vrátit.
- **Bottom sheets** — na mobilu místo dialogů/menu: filtry tabulek, detail platby, výběr položky číselníku.
- **Ostatní** — outlined text fields, segmented buttons (přepínač období ceníku), linear progress + skeletony pro načítání, badge s počtem na položkách rail (fronta dokumentů, nespárované platby).

## 5. Konvence prázdných, loading a chybových stavů

**Každá obrazovka a každá tabulka/sekce má definovaný prázdný stav.** Vzor (M3):

1. **Ilustrace/ikona** — velká outlined Material Symbol (64 px) v `secondary-container` kruhu; žádné stock fotky.
2. **Nadpis** (title-medium) — věcný, bez omluv.
3. **Text** (body-medium, max 2 řádky) — proč je prázdno a co s tím.
4. **CTA** (filled/tonal button) — jen pokud existuje smysluplná akce; jinak bez tlačítka.

Kanonické prázdné stavy (přesná copy; obrazovkové specifikace na ně odkazují):

| Kde | Kdy nastane | Ikona | Nadpis | Text | CTA |
|---|---|---|---|---|---|
| Výpis akcí (portál) | žádná veřejná akce s otevřeným/budoucím oknem | event | Zrovna žádné veřejné akce | Oddíly zveřejňují akce průběžně. Máte-li od oddílu sdílecí odkaz, otevřete ho — funguje i pro akce mimo tento výpis. | — |
| Výpis akcí — filtr | filtr nic nenašel | search_off | Nic neodpovídá filtru | Zkuste filtr zrušit nebo změnit. | Zrušit filtry |
| Rozcestník — dokumenty | akce nevyžaduje žádné dokumenty | task_alt | Žádné dokumenty nejsou potřeba | Tato akce nevyžaduje nahrání žádných dokumentů. | — |
| Tabulka přihlášek akce | akce zatím bez přihlášek | inbox | Zatím žádné přihlášky | Přihlašování je otevřené — sdílejte odkaz na akci, ať se lidé mohou hlásit. | Zkopírovat sdílecí odkaz |
| Fronta dokumentů | nic nečeká na posouzení | done_all | Vše posouzeno | Žádný dokument nečeká na schválení. Nové položky se tu objeví hned po nahrání. | — |
| Náhradníci akce | žádní náhradníci | group_off | Žádní náhradníci | Až se kapacita naplní, noví zájemci se zařadí sem a uvidíte je v pořadí podání. | — |
| Fronta transakcí (ÚČE) | žádné nespárované transakce | account_balance | Žádné nespárované platby | Všechny příchozí transakce jsou spárované. Nové se objeví po synchronizaci s bankou. | — |
| Evidence osob | prázdný oddíl / filtr | person_search | Nikdo tu není | Přidejte první osobu, nebo upravte filtr. | Přidat osobu |
| Docházka | akce bez účastníků k zápisu | checklist | Není koho zapsat | Docházka se zapisuje účastníkům akce. Tato akce zatím žádné nemá. | — |
| Reporty | zvolené období bez dat | monitoring | Za toto období nejsou data | Zkuste jiné období nebo jinou akci. | — |
| Družiny | oddíl bez družin | groups | Zatím žádné družiny | Družiny pomáhají členit oddíl a zapisovat docházku po skupinách. | Založit družinu |
| Hlídky závodu | zatím nesestavené hlídky | flag | Zatím žádné hlídky | Hlídky sestavují vlastníci přihlášek po potvrzení účasti. | — |

**Loading:** skeletony ve tvaru cílového obsahu (karty, řádky tabulky) — žádný skok layoutu; nad tabulkami linear progress při refetchi. **Chyba:** stejný vzor jako prázdný stav s ikonou `error` v `error-container`, nadpis „Něco se nepovedlo“, text „Zkuste to prosím znovu.“, CTA „Zkusit znovu“; nikdy technické detaily. **Formulářové chyby:** inline pod polem (supporting text v `error`), souhrn nahoře jen při odeslání s více chybami.

## 6. Životní cyklus přihlášky (mock logika)

Stav přihlášky je **čistá funkce podmínek** — po každé změně přepočítej `evaluate(přihláška)` přes tři brány v pořadí **zástupce → dokumenty → platba**; první nesplněná brána určí stav. Ručně se nastavují jen `New`, `Canceled`, `Expired`.

- `New` → `PendingGuardian` (nezletilý bez schváleného zástupce) → `PendingDocuments` (chybí/neschválené povinné dokumenty) → `PendingPayment` → `Paid` (uhrazeno **přesně**; akce zdarma jde rovnou do `Paid`). Neuplatněné brány se přeskakují.
- `PendingPayment` → `PartialPaid` (částečná úhrada) → `Paid`; `PendingPayment`/`Paid` → `Overpayment` (přeplaceno; řeší účetní: vrátit / převést / ponechat jako dar).
- Zamítnutý povinný dokument vrací přihlášku do `PendingDocuments` — **i ze `Paid`**.
- `Canceled` (storno, možné z každého nekoncového stavu i ze `Paid`) a `Expired` (marná lhůta zástupce 7 dní; vypršení nezaplacených je ve výchozím stavu vypnuto) jsou **terminální**.
- Náhradník zůstává v `New` se zamčenými branami, dokud nepřijme nabídku místa (platnost 48 h). Do kapacity se počítají jen `PendingDocuments`, `PendingPayment`, `PartialPaid`, `Paid`, `Overpayment`.
- Částky se párují **přesně, bez tolerance** — rozdíl o korunu je nedoplatek nebo přeplatek.

## 7. Otevřené body

Nerozhodnuté detaily (filtry výpisu, zobrazení storno podmínek na detailu, matice rolí) označ v UI decentně jen tam, kde to specifikace obrazovek výslovně žádá, badgem **„K rozhodnutí“** (outlined chip, `tertiary`). V demu zvol vždy uvedenou výchozí variantu.

## 8. Navazující specifikace obrazovek

Obrazovky jednotlivých ploch definují samostatné soubory — každá obrazovka v nich má strukturu Účel · Layout a M3 komponenty · Obsah a pole · Stavy (prázdný, načítání, chyba, úspěch) · Interakce a validace · Mobil/desktop:

- **`02-verejny-portal.md`** — plocha A: Veřejný portál (`/`, `/akce/...`, tokenové stránky).
- **`03-oddilova-sprava.md`** — plocha B: Oddílová správa (`/oddil/...`).
- **`04-ustredi.md`** — plocha C: Správa ústředí (`/ustredi/...`), persona Bohdana Krejcárková (ADM).
- **`05-self-management.md`** — plocha D: Self-management (`/muj-ucet/...`, `/pozvanka/:token`), persona Pavla Konvalinková.
