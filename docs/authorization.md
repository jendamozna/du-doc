# Autorizační matice

Kdo smí co, v jakém rozsahu. Doplňuje [README.md](../README.md) → **Role** o vynutitelná pravidla. Stavové podmínky (např. že storno lze jen z nekoncového stavu) řeší lifecycle dokumenty, tady jde výhradně o **oprávnění**.

## Princip: tři vrstvy oprávnění

Oprávnění nevzniká z jednoho zdroje — skládá se ze tří nezávislých vrstev:

1. **Role v oddílu** (`USER_ROLE`) — role je vždy vázaná na konkrétní oddíl (`unit_id`), nikdy globálně. Výjimkou je `ADM`, který působí napříč všemi oddíly.
2. **Přiřazení k akci** (`EVENT_ASSIGNMENT`) — u VO/VD a Rádce nedává role sama žádná práva k akcím; ta se přidělují **per akci** čtyřmi příznaky. Samo přiřazení dává čtení přihlášek akce.
3. **Odvozená oprávnění** — nevznikají přidělením, ale existencí vazby: rodič (aktivní `PARENT_CHILD`), vlastník přihlášky, osoba sama nad svými údaji, držitel tokenu.

**Vyhodnocení:** výchozí stav je **zákaz**. Uživatel s více rolemi má sjednocení jejich práv (README → _Uživatel může být ve více rolích_). Zákaz čtení citlivých dat je ale **absolutní** a sjednocením se nepřebíjí (viz **Citlivá data**).

**Přiřazení k akci je verzované.** Odebrání přístupu stávající `EVENT_ASSIGNMENT` jen uzavře (`revoked_at`, `revoked_by_account_id`), nemaze ho; změna rozsahu příznaků uzavře starý záznam a založí nový. Kontrola oprávnění pracuje výhradně se záznamy `revoked_at IS NULL`; uzavřené slouží jen k zodpovězení otázky „kdo měl k akci přístup v dubnu 2027" z intervalu `assigned_at`–`revoked_at`. Retence této historie je 10 let od skončení akce (README → **Retence a GDPR**), ne 3 roky jako auditní log.

## Aktéři

| Aktér                  | Zdroj oprávnění                                       | Rozsah                                    |
| ---------------------- | ----------------------------------------------------- | ----------------------------------------- |
| **ADM** Administrátor  | `USER_ROLE`                                           | napříč všemi oddíly                       |
| **HVO** Hlavní vedoucí | `USER_ROLE` + `unit_id`                               | jeden oddíl, plná správa                  |
| **VO** Vedoucí oddílu  | `USER_ROLE` + `EVENT_ASSIGNMENT`                      | jen akce, ke kterým je přiřazený          |
| **VD** Vedoucí družiny | `USER_ROLE` + družina + `EVENT_ASSIGNMENT`            | svá družina + přiřazené akce              |
| **RÁD** Rádce          | `USER_ROLE` + `EVENT_ASSIGNMENT`                      | jen přiřazené akce, **bez citlivých dat** |
| **ÚČE** Účetní oddílu  | `USER_ROLE` + `unit_id`                               | celý oddíl, jen platební agenda           |
| **Rodič**              | aktivní `PARENT_CHILD`                                | **per dítě**, ne globálně                 |
| **Vlastník přihlášky** | token, `submitted_by_account_id` nebo rodič účastníka | jedna přihláška a její dílčí přihlášky    |
| **Osoba (self)**       | `ACCOUNT.person_id`                                   | vlastní údaje a přihlášky                 |
| **Anonym**             | —                                                     | veřejný výpis akcí, sdílecí odkaz         |

Legenda v maticích: **RW** = čtení i zápis · **R** = jen čtení · **A** = podle příznaku v `EVENT_ASSIGNMENT` · **—** = žádný přístup

## Akce a jejich konfigurace

| Operace                                 | ADM            | HVO           | VO / VD             | RÁD                 | ÚČE | Rodič / účastník     |
| --------------------------------------- | -------------- | ------------- | ------------------- | ------------------- | --- | -------------------- |
| Založit akci                            | RW             | RW            | —                   | —                   | —   | —                    |
| Upravit akci                            | RW             | RW            | A `can_edit_event`  | A `can_edit_event`  | —   | —                    |
| Číst detail akce                        | R              | R             | R (přiřazené)       | R (přiřazené)       | R   | R (dle viditelnosti) |
| Nastavit ceny a storno pravidla         | RW             | RW            | A `can_edit_prices` | A `can_edit_prices` | R   | —                    |
| Nastavit výběrové číselníky a dokumenty | RW             | RW            | A `can_edit_event`  | A `can_edit_event`  | —   | —                    |
| Přiřadit vedoucí k akci                 | RW             | RW            | —                   | —                   | —   | —                    |
| Zrušit akci (hromadné storno)           | RW             | RW            | A `can_edit_event`  | —                   | —   | —                    |
| Spravovat šablony akcí                  | RW (systémové) | RW (oddílové) | —                   | —                   | —   | —                    |

## Přihlášky

| Operace                                    | ADM | HVO | VO / VD                    | RÁD                        | ÚČE                       | Rodič / vlastník          |
| ------------------------------------------ | --- | --- | -------------------------- | -------------------------- | ------------------------- | ------------------------- |
| Číst přihlášky akce                        | R   | R   | R (přiřazené akce)         | R (přiřazené akce)         | **R (celý oddíl)**        | R (vlastní / svých dětí)  |
| Upravit přihlášku                          | RW  | RW  | A `can_edit_registrations` | A `can_edit_registrations` | **jen platební atributy** | RW (vlastní / svých dětí) |
| Podat přihlášku                            | —   | RW  | A `can_edit_registrations` | —                          | —                         | RW                        |
| Stornovat přihlášku                        | RW  | RW  | A `can_edit_registrations` | —                          | —                         | RW (vlastní / svých dětí) |
| Posoudit dokument (schválit / zamítnout)   | RW  | RW  | A `can_edit_registrations` | —                          | —                         | —                         |
| Číst obsah nahraného dokumentu             | R   | R   | R (přiřazené akce)         | **—**                      | —                         | R (vlastní)               |
| Vybrat náhradníka                          | RW  | RW  | A `can_edit_registrations` | —                          | —                         | —                         |
| Přiřadit číselník s `assigned_by = leader` | RW  | RW  | A `can_edit_registrations` | A `can_edit_registrations` | —                         | —                         |

Účetní má **širší čtení** (celý oddíl bez ohledu na přiřazení k akci), ale **užší zápis** než vedoucí — párování je operace nad bankovním účtem oddílu a jedna platba může pokrýt přihlášky z více akcí, proto se k akcím nepřiřazuje (README → **Účetní oddílu**).

## Platby

| Operace                                 | ADM | HVO | VO / VD | RÁD | ÚČE          | Ostatní     |
| --------------------------------------- | --- | --- | ------- | --- | ------------ | ----------- |
| Nastavit bankovní účet a token          | —   | RW  | —       | —   | R            | —           |
| Číst bankovní transakce                 | —   | R   | —       | —   | R            | —           |
| Nahrát výpis / ručně zapsat platbu      | —   | RW  | —       | —   | RW           | —           |
| Párovat platby, ruční rozdělení         | —   | RW  | —       | —   | RW           | —           |
| Řešit přeplatek (vratka / převod / dar) | —   | RW  | —       | —   | RW           | —           |
| Odeslat výzvu k platbě                  | —   | RW  | —       | —   | RW           | —           |
| Vygenerovat potvrzení o platbě          | —   | RW  | —       | —   | RW           | R (vlastní) |
| Sestavit a odeslat dávku příspěvků DU   | R   | RW  | —       | —   | R            | —           |
| Párovat platbu dávky příspěvků DU       | R   | —   | —       | —   | RW (ústředí) | —           |
| Spravovat sazbu příspěvku DU            | RW  | —   | —       | —   | —            | —           |

Dávky příspěvků páruje **účetní ústředí** — `ÚČE` se `unit_id` ústředí. Nejde o novou roli: příspěvky chodí na účet ústředí, takže platí stejné pravidlo jako u akcí („účetní páruje platby svého oddílu"). Účetní běžného oddílu do dávek nevidí, HVO vidí jen dávky vlastního oddílu.

## Osoby, družiny a docházka

| Operace                                    | ADM      | HVO                     | VO                        | VD                        | RÁD                           | ÚČE | Osoba / rodič    |
| ------------------------------------------ | -------- | ----------------------- | ------------------------- | ------------------------- | ----------------------------- | --- | ---------------- |
| Evidovat členy a hosty                     | —        | RW                      | R                         | R (svá družina)           | R (svá družina)               | —   | R (sebe / dětí)  |
| Měnit stav osoby (host → člen, deaktivace) | —        | RW                      | —                         | —                         | —                             | —   | —                |
| Upravit údaje osoby                        | —        | RW                      | —                         | —                         | —                             | —   | RW (sebe / dětí) |
| Definovat družiny a jejich členy           | —        | RW                      | —                         | —                         | —                             | —   | —                |
| Zapsat docházku                            | —        | RW                      | A `can_record_attendance` | A `can_record_attendance` | **A `can_record_attendance`** | —   | —                |
| Založit `DU_MEMBERSHIP`                    | —        | RW                      | —                         | —                         | —                             | —   | —                |
| Převést evidenční oddíl členství           | RW       | RW (žádost + potvrzení) | —                         | —                         | —                             | —   | —                |
| Vytvořit účty rolí (pozvánka)              | RW (HVO) | RW (VO/VD/RÁD/ÚČE)      | —                         | —                         | —                             | —   | —                |

Zápis docházky je **samostatné oprávnění** — může ho mít i Rádce, který nemá přístup k přihláškám ani platbám (README → **Docházka**).

## Chytré sloupce (pomocná evidence)

Přístup neurčuje role přímo, ale dvě pole na `CUSTOM_FIELD`:

| Pole         | Koho se týká          | Hodnoty                  |
| ------------ | --------------------- | ------------------------ |
| `visibility` | vlastník účtu (osoba) | `none` / `view` / `edit` |
| `permission` | Rádce                 | `none` / `view` / `edit` |

HVO má ke sloupcům svého oddílu vždy plný přístup; VO/VD podle rozsahu své družiny.

## Agenda ústředí

| Operace                                            | ADM | HVO               | Ostatní |
| -------------------------------------------------- | --- | ----------------- | ------- |
| Spravovat oddíly, přiřazovat HVO                   | RW  | —                 | —       |
| Definovat regiony, přiřazovat oddíly               | RW  | —                 | —       |
| Řídit deduplikaci a schvalovat sloučení            | RW  | RW (svého oddílu) | —       |
| Reporty ústředí (napříč oddíly)                    | RW  | —                 | —       |
| Katalog kurzů, systémové šablony, jmenný whitelist | RW  | —                 | —       |
| Spustit výmaz podle GDPR napříč oddíly             | RW  | RW (svého oddílu) | —       |

## Reporty

Rozsah je definovaný v [reports.md](reports.md) a shoduje se s rolí:

| Role | Vidí                                |
| ---- | ----------------------------------- |
| RÁD  | nic (reporty nemá)                  |
| VD   | jen osoby své družiny               |
| VO   | svůj oddíl                          |
| HVO  | svůj oddíl                          |
| ÚČE  | jen report Platby, svůj oddíl       |
| ADM  | vše napříč oddíly, s dimenzí region |

Scope se aplikuje jako **filtr odvozený z `USER_ROLE`**, ne z parametru requestu — `unit_id` v požadavku se proti povoleným oddílům validuje.

## Odvozená oprávnění

### Rodič

- Práva jsou **per dítě**, ne globální, a plynou z existence `PARENT_CHILD` ve stavu `active`.
- Rozsah podle stavu vazby (plná tabulka v [parent-child-lifecycle.md](parent-child-lifecycle.md)): `pending` nedává nic, `active` plná práva k dítěti, `readonly_after_adulthood` jen čtení + doplnění chybějícího kontaktního e-mailu.
- Role rodiče se **nepřiděluje ani neodebírá** — nemůže se proto rozejít se skutečným stavem vazby.

### Vlastník přihlášky a token

- **Vlastník není `REGISTRATION.person_id`** — to je účastník. Vlastníkem je držitel tokenu, účet v `submitted_by_account_id`, nebo rodič účastníka podle aktivní `PARENT_CHILD`. U přihlášky, kterou si zletilý podal sám, jsou to tytéž osoby; u dítěte ne.
- Token (`REGISTRATION.token`) opravňuje **jen k operacím nad danou přihláškou** — nikdy nezpřístupní seznam osob ani jiné akce ([non-functional.md](non-functional.md) → **Tokeny**).
- Vlastník přihlášky smí spravovat i její **potvrzené dílčí přihlášky** (skládání hlídek, přidávání účastníků).
- **Změna `contact_email` je bezpečnostní operace** — přesměruje tokenový odkaz, tedy přístup k přihlášce. Smí ji provést vlastník přihlášky nebo HVO oddílu a zapisuje se do auditního logu.
- Schvalovací token zástupce opravňuje **výhradně ke schválení** jedné přihlášky, k ničemu jinému.

### Osoba sama

- Čtení a úprava vlastních údajů, správa vlastních přihlášek, změna hesla, propojení a odpojení OAuth identit, žádost o sloučení duplicit.
- Odpojit OAuth identitu lze jen tehdy, zbývá-li účtu jiný způsob přihlášení.

### Anonym

- Veřejný výpis akcí s viditelností `public`; detail akce přes `share_slug` bez ohledu na viditelnost.
- Podání přihlášky na veřejnou akci. Nic dalšího.

## Citlivá data

Zdravotní údaje, alergie, léky a stravovací omezení (`PERSON_SENSITIVE_DATA`) mají **vlastní, přísnější pravidlo**, které přebíjí matice výše:

- **Rádce je nevidí nikdy** — ani u akcí, ke kterým je přiřazený, ani se sjednocením jiných rolí. Důvodem je, že Rádci nejsou plnoletí (README → **Rádce**).
- **Účetní je nevidí** — platební agenda je nepotřebuje.
- Data jsou **izolovaná per oddíl** — oddíl A nevidí citlivá data téže osoby zapsaná v oddílu B.
- **Žádný report nevrací citlivé údaje**, bez ohledu na roli volajícího ([reports.md](reports.md)).

## Pravidla vyhodnocení

- **Deny by default** — chybí-li explicitní pravidlo, přístup se odepře.
- **Sjednocení rolí** — uživatel s více rolemi dostane sjednocení práv; výjimkou je zákaz citlivých dat, který je absolutní.
- **Scope se nikdy nebere z požadavku** — `unit_id` i `event_id` z parametrů se validují proti tomu, co aktérovi náleží.
- **Přiřazení k akci je nutná podmínka** pro VO/VD/RÁD — bez něj k akci přístup nemají, ani kdyby patřila jejich oddílu.
- **Archivovaná osoba** (`record_state = archived`) nemá čitelné osobní údaje pro nikoho — anonymizace je nevratná ([person-lifecycle.md](person-lifecycle.md)).
- Každá operace měnící data se zapisuje do auditního logu s aktérem ([audit-log.md](audit-log.md)); u přístupu přes token je aktérem e-mail, ne účet.

## Otevřené otázky

- **Může ADM číst přihlášky a osobní údaje konkrétního oddílu?** Matice výše mu dává čtení akcí a přihlášek kvůli podpoře, ale specifikace to nikde neříká — alternativou je přístup jen k agregovaným reportům. **[K rozhodnutí]**
- **Delegace HVO** — není řečeno, zda HVO smí udělit část svých práv jinému účtu bez přiřazení k akci (např. zástup po dobu nepřítomnosti).
- **Čtyřoční princip u plateb** — Účetní sama rozděluje platby, eviduje vratky a vidí bankovní účty; specifikace nepožaduje schválení druhou osobou. Krytí je jen auditním logem.
- **Nezletilý Rádce** — kdo odpovídá za jeho úkony v systému, viz otevřený bod v README → **K diskusi**.
