# Autorizační matice

Kdo smí co, v jakém rozsahu. Doplňuje [README.md](../README.md) → **Role** o vynutitelná pravidla. Stavové podmínky (např. že storno lze jen z nekoncového stavu) řeší lifecycle dokumenty, tady jde výhradně o **oprávnění**.

## Princip: tři vrstvy oprávnění

Oprávnění nevzniká z jednoho zdroje — skládá se ze tří nezávislých vrstev:

1. **Role v oddílu** (`USER_ROLE`) — role je vždy vázaná na konkrétní oddíl (`unit_id`), nikdy globálně. Výjimkou je `ADM`, který působí napříč všemi oddíly.
2. **Základní čtení v oddílu** — aktivní VO a RÁD vidí detail akcí a základní seznam jejich přihlášených v oddílu, ke kterému je jejich role v `USER_ROLE` vázaná. Toto právo nezávisí na `EVENT_ASSIGNMENT`.
3. **Tým akce** (`EVENT_ASSIGNMENT`) — tvoří jej výhradně VO a RÁD z pořádajícího oddílu. Člen týmu může mít per-akční roli `event_leader` (Vedoucí akce). Přiřazení zakládá týmový vztah a může udělit čtyři příznaky pro zápis; samo o sobě už není podmínkou základního čtení akce.
4. **Odvozená oprávnění** — nevznikají přidělením, ale existencí vazby: zákonný zástupce (aktivní `PARENT_CHILD`), vlastník přihlášky, osoba sama nad svými údaji, držitel tokenu.
5. **Delegace HVO** (`PERMISSION_DELEGATION`) — HVO může dočasně předat konkrétní oprávnění jinému účtu v rámci svého oddílu.

**Vyhodnocení:** výchozí stav je **zákaz**. Uživatel s více rolemi má sjednocení jejich práv (README → _Uživatel může být ve více rolích_). Rádce nečte finanční údaje s jedinou výjimkou: je-li `event_leader`, vidí předepsanou, uhrazenou a zbývající částku přihlášek své akce (viz **Finanční údaje**).

**Přiřazení k akci je verzované.** Odebrání přístupu stávající `EVENT_ASSIGNMENT` jen uzavře (`revoked_at`, `revoked_by_account_id`), nemaze ho; změna rozsahu příznaků uzavře starý záznam a založí nový. Kontrola oprávnění pracuje výhradně se záznamy `revoked_at IS NULL`; uzavřené slouží jen k zodpovězení otázky „kdo měl k akci přístup v dubnu 2027" z intervalu `assigned_at`–`revoked_at`. Retence této historie je 10 let od skončení akce (README → **Retence a GDPR**), ne 3 roky jako auditní log.

## Aktéři

| Aktér                           | Zdroj oprávnění                                                  | Rozsah                                                                                               |
| ------------------------------- | ---------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| **ADM** Administrátor           | `USER_ROLE`                                                      | napříč všemi oddíly                                                                                  |
| **HVO** Hlavní vedoucí          | `USER_ROLE` + `unit_id`                                          | jeden oddíl, plná správa                                                                             |
| **VO** Vedoucí oddílu           | `USER_ROLE` + `EVENT.unit_id`                                    | akce a základní seznamy přihlášených vlastního oddílu; týmová práva podle přiřazení                  |
| **RÁD** Rádce (vedoucí družiny) | `USER_ROLE` + `EVENT.unit_id` + družina                          | akce a základní seznamy přihlášených vlastního oddílu; údaje družiny a zvýšená práva podle přiřazení |
| **Vedoucí akce**                | `EVENT_ASSIGNMENT.team_role = 'event_leader'`                    | přiřazená akce; předepsaná, uhrazená a zbývající částka jejích přihlášek                             |
| **ÚČE** Účetní oddílu           | `USER_ROLE` + `unit_id`                                          | celý oddíl, jen platební agenda                                                                      |
| **Zákonný zástupce**            | aktivní `PARENT_CHILD`                                           | **per dítě**, ne globálně                                                                            |
| **Vlastník přihlášky**          | token, `submitted_by_account_id` nebo zákonný zástupce účastníka | jedna přihláška a její dílčí přihlášky                                                               |
| **Osoba (self)**                | `ACCOUNT.person_id`                                              | vlastní údaje a přihlášky                                                                            |
| **Anonym**                      | —                                                                | veřejný výpis akcí, seznam názvů klubů na veřejné akci, sdílecí odkaz                                |

Legenda v maticích: **RW** = čtení i zápis · **R** = jen čtení · **A** = podle příznaku v `EVENT_ASSIGNMENT` · **—** = žádný přístup

## Akce a jejich konfigurace

| Operace                                     | ADM            | HVO           | VO                  | RÁD                 | ÚČE | Zákonný zástupce / účastník |
| ------------------------------------------- | -------------- | ------------- | ------------------- | ------------------- | --- | --------------------------- |
| Založit akci                                | RW             | RW            | —                   | —                   | —   | —                           |
| Upravit akci                                | RW             | RW            | A `can_edit_event`  | A `can_edit_event`  | —   | —                           |
| Číst detail akce                            | R              | R             | R (vlastní oddíl)   | R (vlastní oddíl)   | R   | R (dle viditelnosti)        |
| Nastavit ceny a storno pravidla             | RW             | RW            | A `can_edit_prices` | A `can_edit_prices` | R   | —                           |
| Nastavit výběrové číselníky a dokumenty     | RW             | RW            | A `can_edit_event`  | A `can_edit_event`  | —   | —                           |
| Pozvat členy oddílu a naplánovat připomínku | RW             | RW            | A `can_edit_event`  | —                   | —   | —                           |
| Spravovat tým akce (jen VO/RÁD)             | RW             | RW            | —                   | —                   | —   | —                           |
| Zrušit akci (hromadné storno)               | RW             | RW            | A `can_edit_event`  | —                   | —   | —                           |
| Spravovat šablony akcí                      | RW (systémové) | RW (oddílové) | —                   | —                   | —   | —                           |
| Založit klubovou přihlášku                  | —              | RW            | RW (vlastní oddíl)  | —                   | —   | —                           |
| Spravovat / uzavřít klubovou přihlášku      | —              | RW            | RW (vlastní oddíl)  | —                   | —   | —                           |
| Číst veřejný seznam klubů                   | R              | R             | R                   | R                   | R   | R                           |

## Přihlášky

| Operace                                    | ADM | HVO | VO                                             | RÁD                                                   | ÚČE                       | Zákonný zástupce / vlastník           |
| ------------------------------------------ | --- | --- | ---------------------------------------------- | ----------------------------------------------------- | ------------------------- | ------------------------------------- |
| Číst přihlášky akce                        | R   | R   | **R (vlastní oddíl, bez platebních atributů)** | **R (vlastní oddíl, bez platebních atributů)**        | **R (celý oddíl)**        | R (vlastní / svých dětí)              |
| Číst stav a částky úhrady přihlášek akce   | R   | R   | R (`event_leader`)                             | **R (`event_leader`; předepsáno / uhrazeno / zbývá)** | R (celý oddíl)            | R (vlastní / svých dětí)              |
| Upravit přihlášku                          | RW  | RW  | A `can_edit_registrations`                     | A `can_edit_registrations`                            | **jen platební atributy** | RW (vlastní / svých dětí)             |
| Podat přihlášku                            | —   | RW  | A `can_edit_registrations`                     | **RW (jen sám za sebe, bez schválení zástupcem)**     | —                         | RW                                    |
| Připojit dítě k vybranému klubu            | —   | RW  | RW (vlastní oddíl)                             | —                                                     | —                         | RW (vlastní / svých dětí)             |
| Stornovat přihlášku                        | RW  | RW  | A `can_edit_registrations`                     | —                                                     | —                         | RW (vlastní / svých dětí)             |
| Posoudit dokument (schválit / zamítnout)   | RW  | RW  | A `can_edit_registrations`                     | —                                                     | —                         | —                                     |
| Číst obsah nahraného dokumentu             | R   | R   | R (přiřazené akce)                             | **R (přiřazené akce, v rozsahu Rádce)**               | —                         | R (vlastní)                           |
| Spravovat trvalé dokumenty osoby           | RW  | RW  | —                                              | —                                                     | —                         | RW (vlastní / dítě při aktivní vazbě) |
| Použít platný trvalý dokument v přihlášce  | RW  | RW  | A `can_edit_registrations`                     | —                                                     | —                         | RW (vlastní / dítě při aktivní vazbě) |
| Vybrat náhradníka                          | RW  | RW  | A `can_edit_registrations`                     | —                                                     | —                         | —                                     |
| Přiřadit číselník s `assigned_by = leader` | RW  | RW  | A `can_edit_registrations`                     | A `can_edit_registrations`                            | —                         | —                                     |

Účetní má **širší čtení** (celý oddíl bez ohledu na přiřazení k akci), ale **užší zápis** než vedoucí — párování je operace nad bankovním účtem oddílu a jedna platba může pokrýt přihlášky z více akcí, proto se k akcím nepřiřazuje (README → **Účetní oddílu**).

Platební operace může provést jedna ÚČE bez schválení druhou osobou. Odpovědnost a dohled zajišťuje auditní log, který zaznamenává aktéra, změnu a čas operace.

Rádce má přesně opačné omezení než Účetní: přihlášku vidí včetně údajů o dítěti a zdravotních údajů, ale **platební atributy se mu maskují**. Je-li však Rádce Vedoucím akce, vidí pro přihlášky této akce předepsanou, uhrazenou a zbývající částku, aby mohl řídit účast; nevidí slevy, storno poplatky, přeplatky, dary, bankovní účet ani transakce (README → **Rádce**). Podání vlastní přihlášky RÁD na akci pořádajícího oddílu je odvozené právo osoby nad sebou samým; **brána schválení zákonným zástupcem se v tomto případě nepoužije**, i když je Rádce nezletilý. RÁD tím nezískává právo podávat přihlášky za jiné osoby.

## Platby

| Operace                                     | ADM | HVO | VO  | RÁD | ÚČE          | Ostatní                  |
| ------------------------------------------- | --- | --- | --- | --- | ------------ | ------------------------ |
| Nastavit bankovní účet a token              | —   | RW  | —   | —   | R            | —                        |
| Číst bankovní transakce                     | —   | R   | —   | —   | R            | —                        |
| Nahrát výpis / ručně zapsat platbu          | —   | RW  | —   | —   | RW           | —                        |
| Párovat platby, ruční rozdělení             | —   | RW  | —   | —   | RW           | —                        |
| Řešit přeplatek (vratka / převod / dar)     | —   | RW  | —   | —   | RW           | —                        |
| Odeslat výzvu k platbě                      | —   | RW  | —   | —   | RW           | —                        |
| Vygenerovat potvrzení o platbě              | —   | RW  | —   | —   | RW           | R (vlastní)              |
| Nastavit lokální složku členského příspěvku | —   | RW  | —   | —   | R            | —                        |
| Vystavit oddílový členský předpis a výzvu   | —   | RW  | —   | —   | RW           | R (vlastní / svých dětí) |
| Sestavit a odeslat dávku příspěvků DU       | R   | RW  | —   | —   | R            | —                        |
| Párovat platbu dávky příspěvků DU           | R   | —   | —   | —   | RW (ústředí) | —                        |
| Spravovat sazbu příspěvku DU                | RW  | —   | —   | —   | —            | —                        |

Dávky příspěvků páruje **účetní ústředí** — `ÚČE` se `unit_id` ústředí. Nejde o novou roli: příspěvky chodí na účet ústředí, takže platí stejné pravidlo jako u akcí („účetní páruje platby svého oddílu"). Účetní běžného oddílu do dávek nevidí, HVO vidí jen dávky vlastního oddílu.

Oddílový členský předpis je finanční agenda oddílu. HVO nastavuje lokální složku a vystavuje předpisy; ÚČE je smí vystavit, párovat a opravovat ve svém oddílu. Člen nebo jeho aktivní zákonný zástupce vidí pouze vlastní předpis a jeho stav úhrady. RÁD k předpisům ani jejich částkám přístup nemá.

## Osoby, družiny a docházka

| Operace                                    | ADM      | HVO                     | VO                        | RÁD                           | ÚČE | Osoba / zákonný zástupce |
| ------------------------------------------ | -------- | ----------------------- | ------------------------- | ----------------------------- | --- | ------------------------ |
| Evidovat členy a hosty                     | R        | RW                      | R                         | R (svá družina)               | —   | R (sebe / dětí)          |
| Měnit stav osoby (host → člen, deaktivace) | —        | RW                      | —                         | —                             | —   | —                        |
| Upravit údaje osoby                        | —        | RW                      | —                         | —                             | —   | RW (sebe / dětí)         |
| Definovat družiny a jejich členy           | —        | RW                      | —                         | —                             | —   | —                        |
| Zapsat docházku                            | —        | RW                      | A `can_record_attendance` | **A `can_record_attendance`** | —   | —                        |
| Založit `DU_MEMBERSHIP`                    | —        | RW                      | —                         | —                             | —   | —                        |
| Převést evidenční oddíl členství           | RW       | RW (žádost + potvrzení) | —                         | —                             | —   | —                        |
| Vytvořit účty rolí (pozvánka)              | RW (HVO) | RW (VO/RÁD/ÚČE)         | —                         | —                             | —   | —                        |

Zápis docházky je **samostatné oprávnění** — může ho mít i Rádce, který nemá přístup k platbám (README → **Docházka**).

## Chytré sloupce (pomocná evidence)

Přístup neurčuje role přímo, ale dvě úrovně přístupu na `CUSTOM_FIELD`:

| Pole             | Koho se týká                                                  | Hodnoty                  |
| ---------------- | ------------------------------------------------------------- | ------------------------ |
| `owner_access`   | osoba nad vlastní hodnotou, případně aktivní zákonný zástupce | `none` / `view` / `edit` |
| `advisor_access` | Rádce v povoleném rozsahu                                     | `none` / `view` / `edit` |

HVO má ke sloupcům svého oddílu vždy plný přístup; VO/RÁD podle rozsahu své družiny.

### Delegace HVO

- HVO může delegovat jen konkrétní oprávnění, které sám má, a pouze v rámci svého oddílu.
- Delegace může být omezena na celý oddíl nebo konkrétní družinu a vždy má `valid_from`; volitelně má `valid_to`.
- Delegovaný účet nesmí delegované oprávnění dále předat a delegace sama nezakládá roli ani přístup k jiným oblastem.
- Delegaci může HVO kdykoli odvolat; odvolání uzavře záznam (`revoked_at`, `revoked_by_account_id`) a nemění historii předchozího přístupu.
- Založení, změna i odvolání delegace se zapisuje do `AUDIT_LOG`.

Pravidla pro `owner_access`:

- osoba s účtem vidí a mění jen vlastní hodnoty a pouze podle nastavené úrovně;
- aktivní zákonný zástupce dědí stejné právo nad hodnotou dítěte;
- vazba `readonly_after_adulthood` umožňuje pouze čtení, i když je `owner_access = edit`;
- token k přihlášce ani vlastnictví přihlášky samo o sobě nezakládá přístup k ostatním hodnotám osoby.

Pravidla pro `advisor_access`:

- přístup platí jen pro osoby v rozsahu družiny nebo akce, ke které je Rádce přiřazen;
- `edit` opravňuje ke změně hodnoty, nikoli k úpravě definice sloupce;
- vlastní hodnota Rádce se posuzuje z jeho práva osoby nad sebou samým, ne z `advisor_access`;
- `CUSTOM_FIELD` nesmí zpřístupnit data, která patří do `PERSON_SENSITIVE_DATA`.

## Agenda ústředí

ADM může číst osobní údaje a přihlášky napříč všemi oddíly, včetně údajů potřebných pro podporu a kontrolu systému. Toto oprávnění je pouze čtecí; změny provádí ADM jen tam, kde je to výslovně uvedeno v matici. Na archivované osoby se vztahuje zákaz čtení po anonymizaci.

| Operace                                      | ADM | HVO               | Ostatní |
| -------------------------------------------- | --- | ----------------- | ------- |
| Spravovat oddíly, přiřazovat HVO             | RW  | —                 | —       |
| Definovat regiony, přiřazovat oddíly         | RW  | —                 | —       |
| Řídit deduplikaci a schvalovat sloučení      | RW  | RW (svého oddílu) | —       |
| Reporty ústředí (napříč oddíly)              | RW  | —                 | —       |
| Katalog kurzů a požadavky kvalifikací        | RW  | R (svého oddílu)  | —       |
| Ověřovat kvalifikace vedoucích napříč oddíly | RW  | —                 | —       |
| Systémové šablony, jmenný whitelist          | RW  | —                 | —       |
| Spustit výmaz podle GDPR napříč oddíly       | RW  | RW (svého oddílu) | —       |

## Reporty

Rozsah je definovaný v [reports.md](reports.md) a shoduje se s rolí:

| Role | Vidí                                |
| ---- | ----------------------------------- |
| RÁD  | jen osoby své družiny               |
| VO   | svůj oddíl                          |
| HVO  | svůj oddíl                          |
| ÚČE  | jen report Platby, svůj oddíl       |
| ADM  | vše napříč oddíly, s dimenzí region |

Scope se aplikuje jako **filtr odvozený z `USER_ROLE`**, ne z parametru requestu — `unit_id` v požadavku se proti povoleným oddílům validuje.

## Vzdělávání a kvalifikace

- **Pověření od staršovstva** je oddílový mandát evidovaný v `MANDATE`.
- **Vzdělávání** je osobní evidence `PERSON_COURSE`. Dokládá odbornou způsobilost a může splnit `COURSE_REQUIREMENT`, ale samo nezakládá pověření ani `USER_ROLE`.
- **ADM** spravuje centrální katalog kurzů a požadavky na role; může určit, že kvalifikace je pro danou roli povinná a do kdy musí být platná.
- **HVO** čte kvalifikace vedoucích a rádců svého oddílu a může kontrolovat splnění požadavků. **VO a RÁD** spravují vlastní kvalifikační záznamy v rozsahu povoleném katalogem.
- ADM může napříč oddíly číst stav kvalifikace, datum platnosti, zdrojový kurz a stav ověření dokladu. Přístup k obsahu dokladu je omezen na podklady nutné k ověření kvalifikace; nezpřístupňuje jiné zdravotní údaje osoby.
- Platnost se vyhodnocuje k aktuálnímu datu: chybějící, zamítnutý nebo prošlý doklad nesplňuje povinný požadavek. Automaticky udělený kurz z akce ústředí se považuje za ověřený podle výsledku akce.
- Každé založení, změna, schválení nebo zamítnutí kvalifikace a požadavku se zapisuje do `AUDIT_LOG`; osobní doklady mají vlastní retenční pravidla.

## Odvozená oprávnění

### Trvalé dokumenty osoby

- `PERSON_DOCUMENT` patří osobě, nikoli konkrétní přihlášce. Osoba může spravovat své dokumenty; aktivní zákonný zástupce dědí právo dokument dítěte nahrát, obnovit, odvolat a použít v jeho přihlášce.
- Po dosažení zletilosti přechází práva zákonného zástupce k dokumentu do režimu jen pro čtení stejně jako ostatní práva z `PARENT_CHILD`.
- VO/RÁD bez příslušného oprávnění nevidí obsah trvalého dokumentu. Vedoucí smí obsah posoudit pouze v rozsahu oprávnění k dané přihlášce; základní čtení seznamu přihlášených obsah dokumentů nezpřístupňuje.

### Zákonný zástupce

- Práva jsou **per dítě**, ne globální, a plynou z existence `PARENT_CHILD` ve stavu `active`.
- Rozsah podle stavu vazby (plná tabulka v [parent-child-lifecycle.md](parent-child-lifecycle.md)): `pending` nedává nic, `active` plná práva k dítěti, `readonly_after_adulthood` jen čtení + doplnění chybějícího kontaktního e-mailu.
- Role zákonného zástupce se **nepřiděluje ani neodebírá** — nemůže se proto rozejít se skutečným stavem vazby.

### Vlastník přihlášky a token

- **Vlastník není `REGISTRATION.person_id`** — to je účastník. Vlastníkem je držitel tokenu, účet v `submitted_by_account_id`, nebo zákonný zástupce účastníka podle aktivní `PARENT_CHILD`. U přihlášky, kterou si zletilý podal sám, jsou to tytéž osoby; u dítěte ne.
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

Zdravotní údaje, alergie, léky a stravovací omezení (`PERSON_SENSITIVE_DATA`) mají **vlastní pravidlo**, které přebíjí matice výše:

- **Rádce je vidí** v rámci svého rozsahu — svá družina a akce, ke kterým je přiřazený. K práci rádce jsou nezbytné (README → **Rádce**). Mimo svůj rozsah je nevidí.
- **Účetní je nevidí** — platební agenda je nepotřebuje.
- Data jsou **izolovaná per oddíl** — oddíl A nevidí citlivá data téže osoby zapsaná v oddílu B.
- **Žádný report nevrací citlivé údaje**, bez ohledu na roli volajícího ([reports.md](reports.md)).

## Finanční údaje

Stav a výše plateb, slevy, storno poplatky, přeplatky, vratky, dary a bankovní účty mají pro Rádce zákaz čtení s úzkou výjimkou pro Vedoucího akce:

- Rádce, který není Vedoucím akce, je nevidí nikdy — ani u akcí, ke kterým je přiřazený, ani sjednocením s jinou rolí.
- **Vedoucí akce s rolí RÁD** vidí u přihlášek své akce předepsanou, uhrazenou a zbývající částku. Nesmí vidět datum, zdroj platby, VS/SS, bankovní účet, jednotlivé transakce, slevy, storno poplatky, přeplatky, vratky ani dary.
- Platební atributy přihlášky se Rádci **maskují na serveru**, ne skrývají v UI — nesmí odejít v odpovědi API.
- Výjimkou je **vlastní přihláška Rádce**, u které platební údaje vidí z odvozeného práva osoby nad sebou samou.

## Pravidla vyhodnocení

- **Deny by default** — chybí-li explicitní pravidlo, přístup se odepře.
- **Sjednocení rolí** — uživatel s více rolemi dostane sjednocení práv; výjimkou je zákaz finančních údajů pro Rádce mimo úzce vymezené částky přihlášek akce, jejímž je Vedoucím akce.
- **Scope se nikdy nebere z požadavku** — `unit_id` i `event_id` z parametrů se validují proti tomu, co aktérovi náleží.
- **Přiřazení k akci není nutná podmínka** pro základní čtení VO/RÁD — aktivní role v pořádajícím oddílu jim dává čtení detailu akce a seznamu přihlášených. Přiřazení zůstává nutné pro týmový vztah, zápis podle příznaků, přístup k dokumentům a zdravotním údajům v rozsahu akce a pro roli Vedoucí akce.
- Aplikace nesmí rozhodovat pouze podle role uživatele. Správná kontrola musí vždy zahrnovat i rozsah oprávnění. Každé oprávnění je vyhodnocováno nad:
  - subject = uživatel
  - role = HVO, VO, ROD, KOO...
  - scope = organizace, oddíl, družina, akce
  - resource = konkrétní člen, registrace, platba...
- **Archivovaná osoba** (`record_state = archived`) nemá čitelné osobní údaje pro nikoho — anonymizace je nevratná ([person-lifecycle.md](person-lifecycle.md)).
- Každá operace měnící data se zapisuje do auditního logu s aktérem ([audit-log.md](audit-log.md)); u přístupu přes token je aktérem e-mail, ne účet.
