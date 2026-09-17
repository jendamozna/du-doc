# UX — Katalog stavů, prázdných obrazovek a validačních textů

## 1. Devět stavů přihlášky — sdílená komponenta

Stav se **nikdy nezobrazuje jako jediné sdělení** — hlavní je checklist „co ještě chybí" (§ 2), název stavu je jen doplňkový štítek (M3 assist/filter chip, tonální, ikona vlevo). Jedna sdílená komponenta pro tokenový rozcestník (`/stav/:token`), self-management (`/muj-ucet/prihlasky/:pid`) i detail v oddílové správě.

| Stav (identifikátor) | Česká nálepka              | Ikona            | Kdy nastává                                                                 |
| -------------------- | -------------------------- | ---------------- | --------------------------------------------------------------------------- |
| `New`                | Nová                       | fiber_new        | založena, žádná brána zatím nevyhodnocena / náhradník před přijetím nabídky |
| `PendingGuardian`    | Čeká na zákonného zástupce | family_restroom  | nezletilý bez aktivní/schválené vazby                                       |
| `PendingDocuments`   | Čeká na dokumenty          | description      | chybí nebo je zamítnutý povinný dokument                                    |
| `PendingPayment`     | Čeká na platbu             | account_balance  | dokumenty vyřízené, platba nulová                                           |
| `PartialPaid`        | Částečně zaplaceno         | hourglass_bottom | přijata částečná úhrada                                                     |
| `Paid`               | Zaplaceno                  | check_circle     | uhrazeno přesně (akce zdarma jde rovnou sem)                                |
| `Overpayment`        | Přeplatek                  | trending_up      | přijato víc, než je cena                                                    |
| `Canceled`           | Stornována                 | cancel           | zrušeno účastníkem nebo vedoucím                                            |
| `Expired`            | Expirovaná                 | schedule         | marná lhůta (přeškrtnutý text)                                              |

Barvy a přesné hex hodnoty viz [ux-navigace.md](ux-navigace.md#62-stavové-barvy-přihlášek-a-dokumentů). Pohyb zpět (např. zamítnutý dokument vrátí `Paid` → `PendingDocuments`) **není regrese** — text vysvětluje důvod („Vedoucí potřebuje kopii posudku znovu"), progress bar se nepoužívá nikde, protože stav je počítaný a nelineární.

## 2. Checklist bran „co ještě chybí"

Namísto progress baru: vertikální seznam max **tří** bran v pevném pořadí **zákonný zástupce → dokumenty → platba**, každá ve stavu:

- ✅ **hotovo** (`check_circle`)
- ● **čeká na nás** — systém/vedoucí musí něco udělat (např. posoudit dokument)
- ○ **čeká na vás** — uživatel musí něco udělat (schválit, nahrát, zaplatit); aktuální brána (`radio_button_unchecked`)
- 🔒 **zamčeno** (`lock`) — brána za aktuální, zatím nevyhodnocená

Neuplatněné brány (např. zletilý účastník nemá bránu zástupce) se **vynechávají celé**, ne jen odškrtnou. Vrácený krok se odškrtne zpět a dostane jednořádkový důvod, ne jen zmizí.

## 3. Stavy dokumentu (`PERSON_DOCUMENT` / `REGISTRATION_DOCUMENT`)

| Stav                      | Nálepka           | Barva (dle § 1)           |
| ------------------------- | ----------------- | ------------------------- |
| čeká na nahrání           | Čeká na nahrání   | paleta `New`              |
| nahráno, čeká na kontrolu | Čeká na posouzení | paleta `PendingDocuments` |
| schváleno                 | Schválen          | paleta `Paid`             |
| zamítnuto                 | Zamítnut          | paleta `Canceled`         |

Zamítnutí vždy nese **komentář posuzovatele** viditelný uživateli (v `error-container`), vybraný z předpřipravených důvodů (_nečitelné · neúplný dokument · prošlá platnost · jiný dokument_) s možností volného textu. Rozdíl „nahráno" ≠ „schváleno" musí být v UI vždy vidět jako dva různé stavy, ne jeden „odesláno".

## 4. Katalog prázdných stavů

Vzor (M3): ikona (64 px, outlined Material Symbol v `secondary-container` kruhu) → nadpis (title-medium, věcný, bez omluv) → text (body-medium, max 2 řádky, proč je prázdno a co s tím) → CTA (jen pokud existuje smysluplná akce).

| Kde                        | Kdy nastane                                    | Ikona           | Nadpis                         | Text                                                                                                                 | CTA                      |
| -------------------------- | ---------------------------------------------- | --------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------- | ------------------------ |
| Výpis akcí (portál)        | žádná veřejná akce s otevřeným/budoucím oknem  | event           | Zrovna žádné veřejné akce      | Oddíly zveřejňují akce průběžně. Máte-li od oddílu sdílecí odkaz, otevřete ho — funguje i pro akce mimo tento výpis. | —                        |
| Výpis akcí — filtr         | filtr nic nenašel                              | search_off      | Nic neodpovídá filtru          | Zkuste filtr zrušit nebo změnit.                                                                                     | Zrušit filtry            |
| Registrační formulář       | přihlašovací okno zavřené (přímý vstup na URL) | event_busy      | Přihlašování není otevřené     | U této akce teď nejde podat přihláška. Podívejte se na detail akce, kdy se otevře.                                   | Zpět na detail akce      |
| Potvrzení po odeslání      | přímý vstup bez čerstvého odeslání             | inbox           | Tady nic není                  | Potvrzení se zobrazí hned po odeslání přihlášky.                                                                     | Zpět na detail akce      |
| Přehled oddílu — pozornost | žádná položka nečeká na zásah                  | done_all        | Vše vyřízeno                   | Žádná přihláška, dokument ani platba nečeká na zásah.                                                                | —                        |
| Seznam akcí oddílu         | žádná akce nebo žádná akce ve filtru           | event           | Zatím žádné akce               | Založte první akci ze šablony — přednastaví dokumenty, ceny i kapacitu.                                              | Nová akce                |
| Rozcestník — dokumenty     | akce nevyžaduje žádné dokumenty                | task_alt        | Žádné dokumenty nejsou potřeba | Tato akce nevyžaduje nahrání žádných dokumentů.                                                                      | —                        |
| Rozcestník — platby        | žádná přijatá platba                           | —               | —                              | Zatím žádná platba nedorazila.                                                                                       | —                        |
| Tabulka přihlášek akce     | akce zatím bez přihlášek                       | inbox           | Zatím žádné přihlášky          | Přihlašování je otevřené — sdílejte odkaz na akci, ať se lidé mohou hlásit.                                          | Zkopírovat sdílecí odkaz |
| Fronta dokumentů           | nic nečeká na posouzení                        | done_all        | Vše posouzeno                  | Žádný dokument nečeká na schválení. Nové položky se tu objeví hned po nahrání.                                       | —                        |
| Náhradníci akce            | žádní náhradníci                               | group_off       | Žádní náhradníci               | Až se kapacita naplní, noví zájemci se zařadí sem a uvidíte je v pořadí podání.                                      | —                        |
| Fronta transakcí (ÚČE)     | žádné nespárované transakce                    | account_balance | Žádné nespárované platby       | Všechny příchozí transakce jsou spárované. Nové se objeví po synchronizaci s bankou.                                 | —                        |
| Přeplatky                  | žádný přeplatek nečeká na rozhodnutí           | task_alt        | Žádné přeplatky k vyřízení     | Žádný přeplatek nečeká na rozhodnutí.                                                                                | —                        |
| Evidence osob              | prázdný oddíl / filtr                          | person_search   | Nikdo tu není                  | Přidejte první osobu, nebo upravte filtr.                                                                            | Přidat osobu             |
| Docházka                   | akce bez účastníků k zápisu                    | checklist       | Není koho zapsat               | Docházka se zapisuje účastníkům akce. Tato akce zatím žádné nemá.                                                    | —                        |
| Reporty                    | zvolené období bez dat                         | monitoring      | Za toto období nejsou data     | Zkuste jiné období nebo jinou akci.                                                                                  | —                        |
| Družiny                    | oddíl bez družin                               | groups          | Zatím žádné družiny            | Družiny pomáhají členit oddíl a zapisovat docházku po skupinách.                                                     | Založit družinu          |
| Hlídky závodu              | zatím nesestavené hlídky                       | flag            | Zatím žádné hlídky             | Hlídky sestavují vlastníci přihlášek po potvrzení účasti.                                                            | —                        |

Neplatný nebo prošlý token (sdílené pro `/stav/:token`, `/schvaleni/:token`, `/nabidka/:token`):

| Kde                                  | Ikona    | Nadpis                   | Text                                                                                                      |
| ------------------------------------ | -------- | ------------------------ | --------------------------------------------------------------------------------------------------------- |
| Neplatný token                       | link_off | Odkaz není platný        | Zkontrolujte, že jste odkaz z e-mailu zkopírovali celý. Pokud problém trvá, ozvěte se vedoucímu oddílu.   |
| Token po konci akce (`/stav/:token`) | history  | Platnost odkazu skončila | Odkaz pro správu přihlášky platí do konce akce. Akce už proběhla — s dotazy se obraťte na vedoucí oddílu. |

## 5. Loading a chybové stavy

- **Loading:** skeleton ve tvaru cílového obsahu (karty, řádky tabulky) — žádný skok layoutu; nad tabulkami při refetchi `linear progress`.
- **Chyba (systémová):** stejný vzor jako prázdný stav, ikona `error` v `error-container`, nadpis „Něco se nepovedlo", text „Zkuste to prosím znovu.", CTA „Zkusit znovu"; **nikdy technické detaily** (stack trace, kódy) v uživatelském UI.
- **Formulářová chyba:** inline pod polem (supporting text v barvě `error`); při odeslání s víc chybami navíc souhrn nahoře s odkazy na jednotlivá pole (nutné na mobilu, kde chyba může být mimo obrazovku). U vícečlenné přihlášky chyba vždy jmenuje účastníka („Vojtěch: chybí datum narození").

## 6. Validace — princip a texty

Tři pravidla bez výjimky; kanonické texty chyb jsou v [validation.md](validation.md) → **UI texty validačních chyb**:

1. **Nikdy nevalidovat při psaní** — jen při opuštění pole (`blur`) a znovu při odeslání.
2. Chyba **pod polem** (viz § 5); u odeslání navíc souhrn nahoře.
3. Text říká **co udělat**, ne co je špatně: „Zadejte PSČ ve tvaru 123 45", ne „Neplatné PSČ".

Klientská validace je jen pohodlí — server vynucuje totéž podle [validation.md](validation.md) bez ohledu na to, co UI zvládlo odchytit dřív (princip „validace na hranici systému").

### 6.1 Cross-check formulářových polí proti data-model.md / validation.md

Pole veřejného registračního formuláře (`/akce/:slug/prihlaska`) mají protějšek v existující specifikaci — nic nechybí, jde jen o to psát UI texty ze stejného zdroje:

| Pole ve formuláři         | Entita / sloupec                     | Pravidlo                                                                                                                                                                                                                                |
| ------------------------- | ------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Kontaktní e-mail          | `REGISTRATION.contact_email`         | povinný jen bez účtu; formát e-mailu — [validation.md](validation.md#přihláška)                                                                                                                                                         |
| Jméno, Příjmení           | `PERSON.first_name/last_name`        | podmíněná povinnost dle kontextu — [validation.md](validation.md#podmíněná-povinnost-polí-osoby)                                                                                                                                        |
| Datum narození            | `PERSON.birth_date`                  | povinné, určuje nezletilost a bránu zástupce — [validation.md](validation.md#podmíněná-povinnost-polí-osoby)                                                                                                                            |
| Typ účastníka             | odvozeno, ne pole uživatele          | `membership_type` se **určuje systémem** dle pořadí pravidel, nevybírá si ho účastník volně — [validation.md](validation.md#ceny-a-storna). UI text pro typ účastníka nesmí naznačovat volnou volbu tam, kde pravidlo řadí automaticky. |
| Číselníky s příplatkem    | `EVENT_FIELD` / `EVENT_FIELD_OPTION` | kapacita, `required_phase`, `condition` — [validation.md](validation.md#výběrové-číselníky)                                                                                                                                             |
| E-mail zákonného zástupce | `REGISTRATION.guardian_email`        | povinný jen u nezletilého bez aktivní vazby — [validation.md](validation.md#přihláška)                                                                                                                                                  |

Kanonické znění textů (formáty i byznys pravidla) je v [validation.md](validation.md) → **UI texty validačních chyb**, klíčované stabilním kódem chyby — frontend i backend tak hlásí doslova totéž. Tabulky v této kapitole popisují, kde a jak se text zobrazí; slova se neduplikují, berou se odtud.

Ostatní obrazovky plochy A jsou **potvrzovací**, ne datové — nemají volný vstup polí kromě jednoho zaškrtnutí:

| Obrazovka / operace                        | Vstup a guard                                                                                         | Text uživateli                                                                              |
| ------------------------------------------ | ----------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| Schválení zástupcem (`/schvaleni/:token`)  | Bez polí; jediná akce „Schválit“. Platný `guardian_approval_token`, zamítnutí neexistuje.             | „Potvrzením schvalujete účast a získáváte zástupcovský přístup k přihláškám tohoto dítěte.“ |
| Nabídka náhradníkovi (`/nabidka/:token`)   | Bez polí; akce „Přijmout místo“. Platný token, nabídka nesmí být po lhůtě.                            | „Když nabídku nepřijmete, nic se neruší — zůstáváte na čekací listině.“                     |
| Storno přihlášky (`/stav/:token`)          | Potvrzovací dialog s náhledem poplatku k dnešku; povinné zaškrtnutí „Rozumím, že storno je nevratné“. | „Storno je nevratné. Poplatek k dnešku je X Kč.“                                            |
| Přidání dalšího účastníka (`/stav/:token`) | Otevře formulář A-03 pro tutéž akci — platí cross-check výše.                                         | —                                                                                           |

### 6.2 Validační scénáře oddílové správy

| Obrazovka / operace        | Guard a chování UI                                                                                                              | Text uživateli                                                                              |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| Založení akce              | Akce vzniká výběrem existující šablony; její hodnoty jsou snapshot.                                                             | „Vyberte šablonu akce.“                                                                     |
| Publikace placené akce     | Bez bankovního účtu je „Publikovat“ disabled; guard platí i při odebrání účtu nebo přidání ceny publikované akci.               | „Placenou akci nelze publikovat bez bankovního účtu.“                                       |
| Přihlašovací okno          | `registration_from < registration_to`; začátek a konec akce musí být ve správném pořadí.                                        | „Konec přihlašování musí být po jeho začátku.“ / „Konec akce musí být po jejím začátku.“    |
| Kapacita akce              | Nelze snížit pod počet přihlášek, které se do ní už počítají.                                                                   | „Kapacitu nelze snížit pod aktuálně obsazený počet X.“                                      |
| Kapacita položky číselníku | Nelze snížit pod počet existujících voleb; kontrola nové volby je atomická.                                                     | „Limit nelze snížit pod X již zvolených míst.“                                              |
| Číselník `before_event`    | Položka musí mít příplatek 0 Kč.                                                                                                | „Volba dokončovaná před akcí nemůže měnit cenu.“                                            |
| Základní cena              | Placená akce vyžaduje `non_DU` cenu pokrývající celé přihlašovací okno.                                                         | „Doplňte základní cenu pro celé období přihlašování.“                                       |
| Změna ceny                 | Existující přihlášky se automaticky nepřecení.                                                                                  | „Změna ceny se projeví jen u nových přihlášek.“                                             |
| Hromadná připomínka        | Lze vybrat jen přihlášky s nenulovou zbývající částkou a dostupným kontaktem. Nezpůsobilé řádky se přeskočí a uvedou v souhrnu. | „Připomínku lze poslat jen přihláškám s dlužnou částkou a kontaktním e-mailem.“             |
| Posouzení dokumentu        | Zamítnutí vyžaduje důvod; schválení i zamítnutí spustí `evaluate()`.                                                            | „Vyberte důvod zamítnutí nebo napište vlastní.“                                             |
| Chybějící datum narození   | Bránu zákonného zástupce nelze vyhodnotit; doplnit smí vedoucí.                                                                 | „Bránu zákonného zástupce nelze vyhodnotit — chybí datum narození.“                         |
| Nabídka náhradníkovi       | Běžících nabídek smí být nejvýše tolik, kolik je volných míst.                                                                  | „Další nabídku nelze poslat, dokud se neuvolní místo nebo neskončí některá běžící nabídka.“ |
| Docházka                   | Přítomen / Nepřítomen / Nezapsáno jsou tři různé stavy; nejvýše jeden záznam na osobu a akci.                                   | Bez validační chyby; změna se ukládá okamžitě.                                              |
| Párování platby            | Součet alokací nesmí překročit částku transakce; částky se porovnávají přesně.                                                  | „Rozdělená částka nesmí překročit částku transakce.“                                        |
| Vratka                     | Záporná alokace nesmí stáhnout součet přihlášky pod nulu.                                                                       | „Vrácená částka je vyšší než dosud uhrazená částka.“                                        |
| Přeplatek                  | Nabídnout Vrátit / Převést na jinou přihlášku téže osoby / Ponechat jako dar.                                                   | „Vyberte, jak naložit s přeplatkem X Kč.“                                                   |
| Družina                    | Nejvýše jeden vedoucí a jeden zástupce vedoucího na družinu.                                                                    | „Družina už vedoucího má. Nejprve změňte nebo odeberte stávajícího.“                        |
| Odebrání role              | Uzavře otevřená `EVENT_ASSIGNMENT`; posledního HVO nelze odebrat.                                                               | „Posledního hlavního vedoucího oddílu nelze odebrat.“                                       |

### 6.3 Výsledky mutací v oddílové správě

Každá úspěšná mutace se zapíše do auditu, překreslí dotčené badge a zobrazí snackbar. Pokud mění fakta přihlášky, vždy následně spustí `evaluate()`.

| Operace                                | Snackbar / potvrzení                                              |
| -------------------------------------- | ----------------------------------------------------------------- |
| Uložení konfigurace, osoby nebo oddílu | „Uloženo“                                                         |
| Hromadná připomínka                    | „Odesláno N připomínek platby“                                    |
| Export                                 | „CSV exportováno“                                                 |
| Schválení dokumentu                    | „Dokument schválen“                                               |
| Zamítnutí dokumentu                    | „Dokument zamítnut — účastník dostane e-mail“                     |
| Odeslání nabídky náhradníkovi          | „Nabídka odeslána“; chip zobrazí absolutní konec 48hodinové lhůty |
| Uložení docházky                       | „Uloženo“; souhrn přítomných se přepočítá                         |
| Spárování platby                       | „Platba spárována“                                                |
| Vyřešení přeplatku                     | „Přeplatek vyřešen“                                               |
| Založení nebo změna družiny            | „Družina uložena“                                                 |

### 6.4 Role a prezentace nepřístupných údajů

- Akce, tab nebo ovládací prvek mimo rozsah role se skryje; neukazuje se jako disabled, pokud uživatel nemůže oprávnění sám získat.
- Částka, na jejíž existenci uživatel právo má, ale nesmí znát hodnotu, se maskuje `———` s tooltipem „Platební údaje nejsou pro tuto roli dostupné.“
- Read-only režim ponechá hodnoty viditelné a skryje ukládací/destruktivní akce.
- Přímý vstup na nepovolenou URL zobrazí 403: ikona `lock`, nadpis „Sem nemáte přístup“, text „Pro tuto část nemáte potřebné oprávnění.“, CTA „Zpět na přehled“.
- Nenalezená akce zobrazí nadpis „Akci jsme nenašli“ a CTA „Zpět na seznam akcí“.

### 6.5 Cross-check formulářových polí plochy C (ústředí)

Formulářová pole administrace ústředí a jejich protějšek ve specifikaci — UI text se píše ze stejného zdroje:

| Pole ve formuláři            | Entita / sloupec                      | Pravidlo a UI text                                                                                                                                                                               |
| ---------------------------- | ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Založit oddíl — Název        | `UNIT.name`                           | povinný; u typu `collective` nesmí obsahovat „DU“ — [validation.md](validation.md#oddíl-a-region). „Název kolektivního člena nesmí obsahovat „DU“.“                                              |
| Založit oddíl — Typ          | `UNIT.type`                           | povinný (`hq_ico` / `branch` / `collective`); určuje povinnost IČO. „Vyberte typ oddílu.“                                                                                                        |
| Založit oddíl — IČO          | `UNIT.ico`                            | 8 číslic, kontrolní číslice modulo 11; povinné u `branch`/`collective`, prázdné u `hq_ico` — [validation.md](validation.md#formáty). „Zadejte IČO jako 8 číslic.“                                |
| Zařazení / přesun do regionu | `UNIT_REGION`                         | volitelný; intervaly se nepřekrývají, přesun uzavře stávající příslušnost od dneška — [validation.md](validation.md#oddíl-a-region). „Přesun uzavře stávající zařazení a otevře nové od dneška.“ |
| E-mail pozvánky HVO          | pozvánka na roli (`EMAIL_HVO_INVITE`) | formát e-mailu — [validation.md](validation.md#formáty); pozvánka platí 14 dní. „Zadejte e-mail ve tvaru jmeno@domena.cz.“                                                                       |
| Založit region — Název       | `REGION.name`                         | povinný a unikátní — [region-lifecycle.md](region-lifecycle.md). „Region s tímto názvem už existuje.“                                                                                            |
| Sloučení regionů — Nástupce  | `REGION.name` (nový)                  | ≥ 2 zdroje, nástupce je vždy **nový** region s unikátním názvem. „Vyberte alespoň dva regiony a zadejte název nového.“                                                                           |
| Whitelist — Přidat jméno     | `NAME_WHITELIST`                      | unikátní bez ohledu na diakritiku a velikost písmen — [validation.md](validation.md#unikátnosti). „Toto jméno už v seznamu je.“                                                                  |
| Kurz — Platnost              | `COURSE` (platnost v měsících)        | kladné celé měsíce, nebo prázdné = „trvalý“ — [validation.md](validation.md#vzdělávání-a-kvalifikace). „Zadejte platnost v celých měsících, nebo nechte prázdné pro trvalý kurz.“                |

Guardy operací ústředí bez klasického formuláře:

| Operace                          | Guard a chování UI                                                                                                                    | Text uživateli                                                                                      |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| Sloučení osob — volba pole       | Každý konflikt má povinnou volbu A/B; ruční přepis se nenabízí (kvůli věrnému revertu). Provést lze až po rozhodnutí všech konfliktů. | „Rozhodněte, která hodnota se přenese.“                                                             |
| Sloučení osob — blokující kolize | Obě osoby mají aktivní přihlášku na téže akci → sloučení nelze provést, nic se nemění.                                                | „Sloučení nelze provést: obě osoby mají aktivní přihlášku na akci [název]. Vyřeší ji vedoucí akce.“ |
| Sloučení osob — revert           | Vrátit sloučení smí jen ADM; operace je jednorázová.                                                                                  | „Sloučení může vrátit jen administrátor ústředí.“                                                   |
| Reportovací sloučení (R9)        | Nemění žádná data — počítá se jen v reportu Unikátní děti.                                                                            | „Reportovací sloučení nemění žádná data — dvě osoby se počítají jako jedna jen v tomto reportu.“    |
| Znovu povolit potlačenou dvojici | Potlačenou dvojici smí znovu nabídnout jen ADM.                                                                                       | „Dvojici lze znovu nabízet.“                                                                        |

### 6.6 Cross-check formulářových polí plochy D (self-management)

Pole sekce Můj účet sdílí pravidla s registračním formulářem (§ 6.1) a s osobními poli oddílové správy:

| Pole ve formuláři               | Entita / sloupec                              | Pravidlo a UI text                                                                                                                                                           |
| ------------------------------- | --------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Jméno (moje / dítě)             | `PERSON.first_name`                           | whitelist; neshoda **neblokuje** — [validation.md](validation.md#ux-pomoc-pro-jméno). „Jméno není v seznamu českých jmen — výjimku potvrdí vedoucí oddílu.“                  |
| Příjmení                        | `PERSON.last_name`                            | neověřuje se proti žádnému seznamu — [validation.md](validation.md#formáty).                                                                                                 |
| Datum narození                  | `PERSON.birth_date`                           | **jen ke čtení**; mění vedoucí oddílu — [validation.md](validation.md#osoba-a-vazby). „Změnu data narození vyřídí vedoucí oddílu.“                                           |
| Kontaktní e-mail                | `PERSON.email`                                | formát e-mailu; u zletilého dítěte v režimu jen pro čtení je to jediné doplnitelné pole — [validation.md](validation.md#formáty). „Zadejte e-mail ve tvaru jmeno@domena.cz.“ |
| Adresa                          | `PERSON.street/house_number/postal_code/city` | PSČ 5 číslic, mezera volitelná — [validation.md](validation.md#formáty). „Zadejte PSČ ve tvaru 123 45.“                                                                      |
| Zdravotní pojišťovna            | `PERSON.insurance_company`                    | volitelná, není-li vyžádána šablonou akce.                                                                                                                                   |
| Pozvat zástupce — E-mail        | vazba `PARENT_CHILD` (pozvánka)               | formát e-mailu; vazba vznikne až přijetím, pozvánka platí 14 dní — [parent-child-lifecycle.md](parent-child-lifecycle.md). „Zadejte e-mail ve tvaru jmeno@domena.cz.“        |
| Pozvánka 2. zástupce — Checkbox | —                                             | povinné potvrzení „Jsem zákonný zástupce tohoto dítěte“. „Bez potvrzení nelze pozvánku přijmout.“                                                                            |
| Změna hesla — Nové heslo        | `ACCOUNT`                                     | min. 8 znaků, dvakrát shodně. „Heslo musí mít alespoň 8 znaků.“ / „Hesla se neshodují.“                                                                                      |
| Odvolání souhlasu               | souhlas (self-service)                        | zapíše `revoked_at`; záznam se uchovává ještě 4 roky. „Souhlas odvolán.“                                                                                                     |

Guardy operací self-managementu:

| Operace                        | Guard a chování UI                                                                                                                    | Text uživateli                                                                                        |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| Odpojení přihlašovací metody   | Poslední zbývající způsob přihlášení nelze odpojit; tlačítko disabled + ikona `lock` — [authorization.md](authorization.md).          | „Jediný způsob přihlášení nelze odpojit — nejdřív nastavte heslo nebo připojte jiný účet.“            |
| Zrušení zastoupení dítěte      | Dialog s dopadem + povinné zaškrtnutí „Rozumím důsledkům“; akce se zaznamená.                                                         | „Přestanete spravovat údaje a přihlášky dítěte. Zůstane-li bez zástupce, spravuje ho vedoucí oddílu.“ |
| Zletilé dítě (readonly)        | Všechna pole disabled kromě chybějícího kontaktního e-mailu — [parent-child-lifecycle.md](parent-child-lifecycle.md).                 | „[Jméno] je zletilá — zastoupení je jen pro čtení.“                                                   |
| Cizí přihláška v URL           | `:pid` mimo účet a vazby → chybová stránka bez detailů.                                                                               | „Tato přihláška nepatří k vašemu účtu.“                                                               |
| Sloučení duplicit — volba pole | Povinná volba A/B u každého konfliktu, žádný ruční přepis; dokončit lze jen po rozhodnutí všech — [person-merge.md](person-merge.md). | „Rozhodněte, která hodnota se přenese.“                                                               |
| Odmítnutí návrhu sloučení      | „Toto nejsem já“ dvojici trvale potlačí; znovu povolí jen ADM.                                                                        | „Dvojice se trvale potlačí a návrh se už nezobrazí. Povolit ji může jen administrátor ústředí.“       |
