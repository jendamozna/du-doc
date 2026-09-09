# Validační pravidla a byznys-invarianty

Doplněk k [data-model.md](data-model.md), který popisuje **co je platná hodnota** a **co nesmí nastat**. Stavové přechody řeší lifecycle dokumenty ([registration-lifecycle.md](registration-lifecycle.md), [person-lifecycle.md](person-lifecycle.md), [parent-child-lifecycle.md](parent-child-lifecycle.md), [region-lifecycle.md](region-lifecycle.md)).

## Princip

- **Validace na hranici systému** — všechna pravidla se vynucují na serveru bez ohledu na to, co kontroluje UI. Klientská validace je pohodlí, ne ochrana.
- **Invarianty i v databázi** — unikátnosti a nepřekryvy z tabulek níže patří do schématu jako `UNIQUE` / `EXCLUDE`, ne jen do aplikační logiky. Souběžné požadavky by je jinak obešly.
- **Povinnost je kontextová, ne absolutní** — většina polí osoby je povinná až podle toho, co vyžaduje šablona akce nebo stav osoby (viz **Podmíněná povinnost**).
- U normalizovaných hodnot se před validací odstraní okolní mezery; mezery uvnitř hodnoty se odstraňují nebo zachovávají podle pravidla konkrétního údaje. Heslo je výjimka: okolní i vnitřní mezery mohou být součástí přístupové fráze a nesmí se tiše měnit.
- Pravidla v tomto dokumentu jsou závazná pro serverovou validaci; klientská validace je pouze pomocná.

## Formáty

| Údaj              | Pravidlo                                                                                                                                                                                                                                                                                               | Zdroj                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------- |
| Částka            | desetinné číslo v CZK, **nezaokrouhluje se**; porovnává se přesně (rozdíl 1 Kč = nedoplatek/přeplatek)                                                                                                                                                                                                 | [README.md](../README.md) → Modul párování plateb |
| Datum a čas       | ukládá se v UTC, zobrazuje v `Europe/Prague`; čistě datumové údaje se nepřepočítávají                                                                                                                                                                                                                  | [non-functional.md](non-functional.md)            |
| E-mail            | Po odstranění okolních mezer nejvýše 254 znaků, syntakticky platný podle běžného e-mailového parseru; celá adresa se ukládá malými písmeny, DNS se neověřuje. `ACCOUNT.login_email` je po normalizaci unikátní a `PERSON.email` unikátní není                                                          | —                                                 |
| IČO               | Po odstranění okolních mezer přesně 8 ASCII číslic včetně kontrolní číslice: první 7 číslic se násobí vahami 8 až 2, součet se modulo 11 převede na poslední číslici (`0` pro zbytek 0 nebo 1, jinak `11 - zbytek`); povinné u typu `branch` a `collective`, prázdné u `hq_ico`                        | [README.md](../README.md) → Oddíl                 |
| Telefon           | Po odstranění okolních mezer volitelný kontaktní telefon; vstup může obsahovat mezery, závorky a spojovníky, ale ukládá se normalizovaný v E.164 (`+` a 8–15 číslic). Devítimístné české číslo bez předvolby se uloží s `+420`; jiné národní formáty se bez předvolby země nepřijímají                 | [data-model.md](data-model.md)                    |
| Křestní jméno     | proti centrálnímu systémovému číselníku `NAME_WHITELIST`, který spravuje ADM; neshoda se dá povolit výjimkou `NAME_EXCEPTION` v rámci oddílu schválenou HVO                                                                                                                                            | [README.md](../README.md) → Deduplikace           |
| Příjmení          | **neověřuje se** proti žádnému seznamu                                                                                                                                                                                                                                                                 | [README.md](../README.md) → Deduplikace           |
| Adresa            | strukturovaná pole `PERSON.street`, `PERSON.house_number`, `PERSON.postal_code`, `PERSON.city` a `PERSON.country`; `country` se ukládá jako kód ISO 3166-1 alpha-2. Jednotlivá pole jsou volitelná podle šablony akce, ale pokud je adresa povinná, musí být vyplněna minimálně `city` a `postal_code` | [data-model.md](data-model.md)                    |
| GPS souřadnice    | `lat` ∈ ⟨−90; 90⟩, `lng` ∈ ⟨−180; 180⟩                                                                                                                                                                                                                                                                 | [data-model.md](data-model.md) → LOCATION         |
| Variabilní symbol | číselný, generuje systém při vzniku přihlášky; neposkytuje ho uživatel                                                                                                                                                                                                                                 | [payment-matching.md](payment-matching.md)        |
| Specifický symbol | číselný, zadává vedoucí u akce                                                                                                                                                                                                                                                                         | [README.md](../README.md) → Konfigurace akce      |
| Soubor dokumentu  | max **10 MB**, typ PDF/JPG/PNG/HEIC ověřený podle **obsahu, ne přípony**                                                                                                                                                                                                                               | [non-functional.md](non-functional.md)            |

### UX pomoc pro jméno

- U jména se po zadání alespoň 2 znaků zobrazí našeptávané hodnoty z centrálního seznamu `NAME_WHITELIST`. Našeptávač je pouze pomůcka; uživatel může ponechat vlastní zadaný text a výběr návrhu není povinný.
- U příjmení se při odchodu z pole automaticky nastaví pohlaví na ženské pouze tehdy, je-li zvoleno „neuvedeno“ a příjmení končí na `á`. Jde jen o předvyplnění; explicitně zvolené pohlaví se nepřepisuje a uživatel je může změnit.
- U adresy se po zadání alespoň 3 znaků po prodlevě 300 ms načtou z API Mapy.cz nejvýše 5 návrhů typu `regional.address`. Po výběru se vyplní strukturovaná pole `street`, `house_number`, `postal_code`, `city` a `country`; pokud API návrh nenajde nebo není dostupné, uživatel může pokračovat ručním vyplněním polí.

## Podmíněná povinnost polí osoby

Která pole `PERSON` musí být vyplněná, závisí na kontextu:

| Kontext                                      | Povinná pole                                                                                                                                        | Zdroj                                                  |
| -------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| Host v oddílu                                | jméno **a** příjmení, **nebo** přezdívka                                                                                                            | [README.md](../README.md) → Hlavní vedoucí             |
| Registrovaný člen                            | jméno, příjmení, pohlaví, **datum narození**                                                                                                        | [README.md](../README.md) → Hlavní vedoucí             |
| Přihláška nezletilého bez zákonného zástupce | `birth_date` (jinak nelze vyhodnotit bránu) + `guardian_email`                                                                                      | [registration-lifecycle.md](registration-lifecycle.md) |
| Akce typu „Mentor a doporučení"              | volitelně jméno + e-mail mentora; bez aktivního `PERSON_UNIT` je e-mail hlavního vedoucího povinný, jinak se odvodí z aktivní role HVO vazby oddílu | [README.md](../README.md) → Přihlašování na akce       |
| Akce typu „S certifikátem"                   | tituly před/za + adresa trvalého bydliště                                                                                                           | [README.md](../README.md) → Typy a šablony             |
| Člen hlídky na závodě                        | `birth_date` (bez něj nelze ověřit složení hlídky)                                                                                                  | [race-patrols.md](race-patrols.md)                     |
| Vlastník účtu                                | `ACCOUNT.login_email`                                                                                                                               | [data-model.md](data-model.md)                         |

Ostatní pole (`nickname`, `insurance_company` a jednotlivá pole adresy) jsou povinná jen tehdy, označí-li je tak šablona akce nebo `EVENT_CUSTOM_FIELD.required`.

## Unikátnosti

| Entita                 | Klíč                                          | Poznámka                                                                   |
| ---------------------- | --------------------------------------------- | -------------------------------------------------------------------------- |
| `ACCOUNT`              | `login_email`                                 | přihlašovací e-mail; `PERSON.email` unikátní **není**                      |
| `ACCOUNT`              | `person_id`                                   | jedna osoba má nejvýše jeden účet                                          |
| `OAUTH_IDENTITY`       | `provider` + `provider_user_id`               | jedna externí identita patří jednomu účtu                                  |
| `USER_ROLE`            | `account_id` + `unit_id` + `role`             | tatáž role se v oddílu nepřiděluje dvakrát                                 |
| `DU_MEMBERSHIP`        | `person_id` + `year`                          | **`unit_id` do klíče nepatří** — jedno členství DU na osobu a rok globálně |
| `DU_FEE_RATE`          | `year`                                        | jedna sazba příspěvku na rok                                               |
| `UNIT_MEMBER_FEE_RATE` | `unit_id` + `year`                            | oddíl má pro rok jednu aktivní lokální sazbu                               |
| `UNIT_MEMBER_FEE`      | `unit_id` + `person_id` + `year`              | jedna osoba má v oddílu jeden roční členský předpis                        |
| `DU_FEE_BATCH`         | `vs`                                          | variabilní symbol musí dávku jednoznačně identifikovat                     |
| `DU_FEE_BATCH_ITEM`    | `batch_id` + `person_id`                      | osoba je v jedné dávce nejvýše jednou                                      |
| `ATTENDANCE_RECORD`    | `event_id` + `person_id`                      | nejvýše jeden docházkový záznam na osobu a akci                            |
| `EVENT_ASSIGNMENT`     | `event_id` + `account_id` (otevřený záznam)   | jedno **aktivní** přiřazení na účet a akci; uzavřených může být víc        |
| `EVENT_INVITATION`     | `event_id` + `person_id`                      | osoba dostane na danou akci nejvýše jednu pozvánku                         |
| `PERSON_DOCUMENT`      | `person_id` + `document_type` + otevřený stav | pro daný typ osoby existuje nejvýše jeden aktuální dokument                |
| `BANK_TRANSACTION`     | `bank_account_id` + `external_id`             | idempotentní zápis — opakované stažení ani nahrání výpisu platbu nezdvojí  |
| `RACE_PATROL`          | `event_id` + `name`                           | název hlídky je unikátní v rámci akce                                      |
| `RACE_PATROL_MEMBER`   | `person_id` + `event_id` (přes hlídku)        | osoba je nejvýše v jedné hlídce téže akce                                  |
| `EVENT`                | `share_slug`                                  | sdílecí odkaz je globálně unikátní a nepředvídatelný                       |
| `REGISTRATION`         | `vs`                                          | variabilní symbol musí párování jednoznačně identifikovat                  |
| `REGISTRATION`         | `contact_email_confirmation_token`            | jednorázový token potvrzení kontaktu musí být náhodný a globálně unikátní  |
| `RECOMMENDATION`       | `token`                                       | jednorázový token musí být náhodný a globálně unikátní                     |
| `PERSON_UNIT`          | `person_id` + `unit_id` (otevřený záznam)     | osoba má v oddílu nejvýše jeden platný záznam                              |

| `CUSTOM_FIELD` | `unit_id` + `unit_patrol_id` + `name` | název je unikátní v daném rozsahu |
| `CUSTOM_FIELD_VALUE` | `custom_field_id` + `person_id` | osoba má pro sloupec nejvýše jednu hodnotu |
| `CUSTOM_FIELD_OPTION` | `custom_field_id` + `value` | hodnota volby je v rámci sloupce unikátní |
| `PERMISSION_DELEGATION` | `unit_id` + `delegate_account_id` + `permission` + otevřený interval | jedna aktivní delegace stejného oprávnění v témže rozsahu |
| `COURSE_REQUIREMENT` | `course_id` + `role` + otevřený interval | pro kurz a roli existuje nejvýše jeden aktivní požadavek |

### Vzdělávání a kvalifikace

- `MANDATE` je oddílový dokument navázaný na `subject_account_id` a pověřenou `role`, nikoli osobní kvalifikace. Pověřený účet musí mít vazbu na stejný oddíl jako mandát. `valid_from <= valid_to`, pokud je `valid_to` vyplněno; odvolané pověření má `revoked_at` a nesmí se používat jako platný mandát. Nahrání nebo platnost pověření samo nezaloží ani nezmění `USER_ROLE`.
- `PERSON_COURSE` je osobní kvalifikační záznam. Ověřený a časově platný kurz může splnit `COURSE_REQUIREMENT`, ale absolvování kurzu samo nezaloží `USER_ROLE` ani `MANDATE`.
- Je-li pro výkon role nastaven požadavek na pověření i kvalifikaci, role je považována za způsobilou až po splnění obou nezávislých podmínek; chybějící nebo prošlé pověření nelze nahradit kurzem a naopak.
- `COURSE` v centrálním katalogu zakládá a archivuje pouze ADM; archivovaný kurz nelze nově přiřadit, ale historické `PERSON_COURSE` záznamy zůstávají platné pro historii.
- `COURSE_REQUIREMENT` může vyžadovat kurz pro roli `HVO`, `VO` nebo `RAD`. Požadavek je účinný jen v intervalu `valid_from`–`valid_to`; překrývající se aktivní požadavky stejného kurzu a role nejsou povolené.
- Povinný kurz je splněný pouze tehdy, pokud osoba má `PERSON_COURSE` s `verification_state = 'verified'`, `completed_on <= aktuální datum` a `valid_to IS NULL OR valid_to >= aktuální datum`.
- Kurz udělený absolvováním vzdělávací akce ústředí může systém ověřit automaticky; ručně nahraný zdravotnický kurz, ŠHVT nebo jiný doklad zůstává ve stavu `pending`, dokud jej neschválí ADM nebo pověřený HVO.
- Zamítnutý doklad (`verification_state = 'rejected'`) nesplňuje požadavek. Změna kurzu, dokladu, platnosti nebo stavu ověření se zapisuje do `AUDIT_LOG`.
- Obsah `certificate_file` je kvalifikační podklad, nikoli obecná zdravotní dokumentace. Přístup k němu se řídí autorizačními pravidly modulu vzdělávání a po skončení retenční lhůty se odstraní nebo anonymizuje; stav kvalifikace může zůstat v agregovaném reportu.

### Trvalé dokumenty osoby

- `PERSON_DOCUMENT.person_id` musí být platná osoba a `document_type` je jedna z hodnot `insurance_card`, `medical_fitness` nebo `other`.
- `valid_from <= valid_to`, pokud je `valid_to` vyplněno. Potvrzení o lékařské způsobilosti k účasti na letním táboře může mít `valid_to IS NULL` a platí do odvolání; „trvalý“ dokument neznamená, že jej nelze nahradit novější verzí nebo odvolat.
- Nový dokument se stává použitelným až ve stavu `valid`; zamítnutý, prošlý nebo odvolaný dokument nelze použít ke splnění požadavku akce. Pro konkrétní akci je dokument platný právě tehdy, když `valid_from <= EVENT.starts_at` a `valid_to IS NULL OR valid_to >= EVENT.ends_at`; platnost se tedy posuzuje pro celou dobu akce, ne jen v okamžiku podání přihlášky.
- `EVENT_DOCUMENT.accepts_person_document = true` dovolí splnit požadavek odkazem na platný `PERSON_DOCUMENT` stejné osoby a stejného `document_type`. Systém takový dokument automaticky přiřadí vytvořením `REGISTRATION_DOCUMENT.person_document_id`; jiný typ dokumentu ani dokument jiné osoby se nepřijme.
- Pokud pro požadovaný typ neexistuje platný osobní dokument, nebo jeho platnost skončí před `EVENT.ends_at`, systém zobrazí při podání přihlášky varování aktivnímu zákonnému zástupci nezletilého účastníka, případně zletilému účastníkovi. Varování nenahrazuje povinnost dokumentu: dokud není dokument nahrán a schválen nebo automaticky přiřazen, přihláška zůstává ve stavu čekání na dokumenty.
- `REGISTRATION_DOCUMENT` může mít vyplněné právě jedno z `person_document_id` a `file`: odkaz na trvalý dokument, nebo nově nahranou kopii. Při použití trvalého dokumentu se soubor nekopíruje do přihlášky; zachová se odkaz a výsledek posouzení.
- Aktivní zákonný zástupce smí dokument dítěte nahrát, obnovit a vybrat pro jeho přihlášku. Operace se zapíše s účtem zákonného zástupce jako aktérem; po zletilosti lze dokument použít, ale zákonný zástupce jej už nesmí měnit.

### Oddílové členské příspěvky

- `UNIT_MEMBER_FEE` lze založit jen pro osobu s aktivním `PERSON_UNIT` ve stavu `registered_member` v témže oddílu. Historický předpis zůstává zachován i po změně stavu osoby.
- `total_amount = du_amount + local_amount`; všechny tři hodnoty jsou nezáporné a jsou snapshotem při vystavení předpisu. Změna `UNIT_MEMBER_FEE_RATE` nebo `DU_FEE_RATE` existující předpis nesmí přepsat.
- `du_amount` odpovídá sazbě DU pro daný rok, nebo je 0, pokud již existuje `DU_MEMBERSHIP(person_id, year)`. Předpis s nulovou složkou DU nelze zařadit do `DU_FEE_BATCH`.
- `PAYMENT_ALLOCATION` cílí právě na jednu z entit `REGISTRATION`, `DU_FEE_BATCH` nebo `UNIT_MEMBER_FEE`; bankovní účet transakce musí patřit témuž oddílu jako cíl předpisu.
- Položka `DU_FEE_BATCH_ITEM` musí odkazovat na předpis téhož oddílu a osoby, jehož součet alokací kryje alespoň `du_amount`. Jedna složka DU smí být nejvýše v jedné neuzavřené dávce.
- Vratka nebo oprava nesmí snížit krytí složky DU pod `du_amount`, pokud je předpis v uzamčené nebo zaplacené dávce; nejdřív se provede oprava odpovídající dávky u ústředí.

## Invarianty po entitách

### Mentor a doporučení

- `RECOMMENDATION` smí vzniknout jen pro `REGISTRATION` akce s `EVENT.type = 'mentor_recommendation'`; pro každou kombinaci `registration_id` + `type` existuje nejvýše jedna aktuální žádost ve stavu `requested` nebo `confirmed`.
- `REGISTRATION.contact_email_confirmation_token` smí být vyplněn jen pro akci typu `mentor_recommendation` před potvrzením kontaktu. Po úspěšném potvrzení se zneplatní; použitý nebo neplatný token nesmí nic změnit.
- `type = 'mentor'` vyžaduje neprázdné `contact_name` a `contact_email`. U `type = 'head_leader'` účastník bez aktivního `PERSON_UNIT` povinně zadá `contact_email` a `source_unit_id` je `NULL`; u účastníka s aktivním oddílem musí `source_unit_id` odkazovat na jeho aktivní `PERSON_UNIT` a `contact_email` se uloží jako snapshot e-mailu aktivního HVO tohoto oddílu. Má-li osoba více aktivních vazeb, výběr `source_unit_id` je povinný. Po potvrzení jsou v obou případech povinné neprázdné odpovědi `reason_leader` a `reason_participant`, každá nejvýše 600 znaků.
- Zvolený `source_unit_id` musí mít aktivní roli HVO s přihlašovacím e-mailem. Chybí-li, žádost o doporučení se nevytvoří a systém oznámí konfigurační chybu pořadateli; nesmí nabídnout ruční náhradu e-mailu HVO účastníkovi, který vazbu na oddíl má.
- Token platí jen pro žádost ve stavu `requested`. Úspěšné potvrzení jej jednou provždy zneplatní (`token_used_at`); neplatný, použitý nebo nahrazený token nesmí měnit data.
- Změna mentora vytvoří novou žádost s novým tokenem a předchozí označí `superseded`; předchozí potvrzení se nepřenáší. Vznik, potvrzení i nahrazení žádosti se zapisují do auditního logu.
- Stav `RECOMMENDATION` není podmínkou `evaluate(registration)` a nesmí ovlivnit stav přihlášky, kapacitu, dokumenty ani platbu.

### Družiny a role RÁD

- `USER_ROLE.role = 'RAD'` je systémová role účtu v oddílu; sama o sobě neurčuje, že osoba vede konkrétní družinu.
- Funkce vedoucího a zástupce vedoucího družiny se eviduje přes `UNIT_PATROL_MEMBER.role = 'leader'` nebo `role = 'deputy'` na osobě propojené s účtem přes `ACCOUNT.person_id`.
- Jedna družina může mít nejvýše jednu osobu s funkcí `leader` a nejvýše jednu osobu s funkcí `deputy`; ostatní osoby mají `role = 'member'`. Jedna osoba může být `leader` nebo `deputy` ve více družinách stejného oddílu. RÁD může mít i nulový počet těchto vazeb.
- Vazba `leader` nebo `deputy` nesmí odkazovat na družinu jiného oddílu než `USER_ROLE.unit_id`; bez žádné z těchto vazeb nemá RÁD přístup k družinovým údajům.

### Pomocná evidence

- `field_type` je jedna z hodnot `text`, `number`, `date`, `boolean`, `choice`; uložená hodnota musí odpovídat typu.
- `owner_access` a `advisor_access` jsou každá samostatně jedna z hodnot `none`, `view`, `edit`; výchozí hodnota při založení je `none`.
- `unit_patrol_id` musí patřit stejnému `unit_id` jako sloupec. Sloupec oddílu bez družiny je dostupný v celém oddílu; družinový sloupec jen osobám v dané družině.
- Hodnotu lze uložit jen osobě, která má v oddílu platný záznam `PERSON_UNIT`; archivovaná osoba ji nemůže číst ani měnit.
- `required` se vyhodnocuje při použití sloupce v `EVENT_CUSTOM_FIELD`, ne jako absolutní povinnost pro každou osobu v oddílu.
- `choice` vyžaduje definovaný seznam povolených hodnot; tento seznam se při změně nesmí změnit tak, aby zneplatnil existující hodnoty bez migrace.
- `CUSTOM_FIELD` nesmí sloužit jako náhrada `PERSON_SENSITIVE_DATA` ani zpřístupnit zdravotní údaje přes méně přísné oprávnění.
- HVO může měnit definici i hodnoty sloupců svého oddílu. Vedoucí nebo zástupce družiny může měnit definici i hodnoty družinových sloupců své družiny, ale ne sloupce oddílu ani jiné družiny. VO může měnit hodnoty u osob v rámci svého oddílu, ale bez této funkce nemůže měnit definici sloupce.
- Změna definice sloupce a změna nebo smazání jeho hodnoty se zapisuje do `AUDIT_LOG`; při změně oprávnění se zachovává předchozí hodnota a aktér změny.

### Delegace oprávnění

- `delegator_account_id` musí mít v daném oddílu roli HVO; delegovat nelze oprávnění ADM ani oprávnění, které HVO sám nemá.
- `delegate_account_id` musí být účet s vazbou na stejný oddíl; účet nesmí delegaci dále řetězit.
- `valid_from < valid_to`, pokud je `valid_to` vyplněno; aktivní delegace stejného oprávnění a rozsahu se nesmí překrývat.
- Odvolaná nebo časově neplatná delegace se při vyhodnocení oprávnění ignoruje.

### Oddíl a region

- **Ústředí (`is_hq`) nemá registrované členy** a nepatří do žádného regionu.
- `UNIT_REGION`: intervaly téhož oddílu se **nesmí překrývat**, nejvýše jeden otevřený (`valid_to = NULL`); díra povolená je ([region-lifecycle.md](region-lifecycle.md)).
- `LOCATION` je viditelná jen v rámci vlastnícího oddílu — akci nelze přiřadit lokaci cizího oddílu.

### Akce

- `EVENT.status` a `EVENT.visibility` jsou dvě nezávislé osy. `status` řídí životní cyklus, zatímco `visibility` publikum publikované akce; hodnoty těchto polí se nikdy nesmějí vzájemně zaměňovat.
- `EVENT.status` je jedna z hodnot `draft`, `published`, `hidden` nebo `cancelled`; nový záznam vzniká ve stavu `draft`. Jen `published` přijímá nové přihlášky v otevřeném přihlašovacím okně. `draft` (koncept) ani `hidden` (skrytá) nepřijímají přihlášky a neposílají pozvánky či připomínky; `cancelled` nepřijímá nové přihlášky a jeho existující přihlášky se řeší podle storno pravidel.
- `EVENT.visibility` je právě jedna z hodnot `public`, `internal` nebo `private`. U stavu `published` určuje publikum: `public` patří do veřejného výpisu portálu, `internal` je pro osoby s vazbou na pořádající oddíl a `private` je dostupná pouze přes sdílecí odkaz.
- **Splatnost je výlučná** — vyplněno buď `payment_due_days`, nebo `payment_due_date`, nikdy obojí ani nic.
- `meeting_at` a `return_at` jsou-li vyplněné, musí ležet v pořadí `meeting_at <= starts_at < ends_at <= return_at`; místo srazu a návratu musí patřit témuž oddílu jako akce. `destination` může být prázdný u akcí bez přesunu.
- `registration_from < registration_to`, `starts_at < ends_at`; přihlašovací okno smí přesahovat začátek akce.
- `visibility` má tři **vzájemně výlučné** hodnoty; `share_slug` má **každá** akce bez ohledu na viditelnost.
- `capacity ≥ 0`, `substitute_count ≥ 0`. Kapacitu nelze snížit pod počet přihlášek, které se do ní už počítají.
- Dobrovolnická pole (`volunteer_registration_*`) dávají smysl jen při `volunteers_enabled = true`.
- Akce bez přihlášek (typ `club`, `one_off`) nesmí mít ceny, storno pravidla ani otevřenou registraci.
- `bank_account_id` musí patřit **témuž oddílu** jako akce.
- `EVENT_ASSIGNMENT.account_id` musí mít v pořádajícím oddílu akce aktivní roli `VO` nebo `RAD`; aktivní přiřazení těchto účtů tvoří tým akce a řídí jejich zvýšená/týmová oprávnění. Základní čtení detailu akce a seznamu přihlášených pro VO/RÁD plyne z aktivní role v témže oddílu a na přiřazení není vázané. Účet s rolí HVO, ÚČE, ADM nebo bez role se do týmu akce nezařazuje.
- `EVENT_ASSIGNMENT.team_role` je `member` nebo `event_leader`; roli `event_leader` lze přiřadit jen aktivnímu členu týmu. Jeden tým může mít více vedoucích akce.
- `EVENT_ASSIGNMENT` se **nemaže** — odebrání přístupu vyplní `revoked_at` a `revoked_by_account_id`; `revoked_at >= assigned_at` a uzavřený záznam už nelze měnit. Změna rozsahu příznaků uzavře starý záznam a založí nový.
- Kontrola oprávnění pracuje výhradně se záznamy `revoked_at IS NULL`; uzavřené jsou doklad o minulém přístupu ([authorization.md](authorization.md)).
- `EVENT_INVITATION.person_id` musí mít při založení aktivní `PERSON_UNIT` v pořádajícím oddílu. Výběr celé skupiny se při naplánování materializuje do jednotlivých pozvánek; pozdější změna družiny, věku, pohlaví nebo role osoby seznam pozvaných nemění.
- Na osobu a akci existuje nejvýše jedna pozvánka. `scheduled_at` nesmí ležet po `registration_to`; volitelný `reminder_scheduled_at` musí být po `scheduled_at` a nejpozději v `registration_to`.
- `response` je `pending`, `accepted` nebo `declined`. Volba **Přihlásit** založí standardní `REGISTRATION` a nastaví `accepted`; volba **Omluvit** nastaví `declined` a přihlášku nezaloží. `registration_id` smí být vyplněné jen při `accepted` a musí odkazovat na přihlášku téže osoby a akce.
- Job odešle pozvánku právě jednou po dosažení `scheduled_at`. Připomínku odešle právě jednou po dosažení `reminder_scheduled_at` jen tehdy, když pozvánka už byla odeslána, `response = 'pending'` a neexistuje přihláška této osoby na akci; pro akci, jejíž `status` není `published`, se pozvánky ani připomínky neposílají.

### Ceny a storna

- `EVENT_PRICE`: intervaly platnosti pro **tutéž** `membership_type` se nesmí překrývat.
- `REGISTRATION.base_price` a `price_id` se určí **při podání** z ceníku platného k `created_at` a od té chvíle se samy nemění — přepsat je smí jen vedoucí s `can_edit_prices` (loguje se). Změna `EVENT_PRICE` ani nové `DU_MEMBERSHIP` už podanou přihlášku nepřeceňuje.
- `CANCELLATION_RULE.percent` ∈ ⟨0; 100⟩.
- **Výsledná cena může být záporná?** Ne — součet základní ceny a příplatků (`price_modifier` může být záporný) se ošetří na minimum 0.

### Výběrové číselníky

- `EVENT_FIELD_OPTION.capacity ≥ 1` nebo `NULL` (bez limitu); `selection_mode = exclusive` odpovídá kapacitě 1.
- Počet voleb v jednovýběrovém číselníku je 1; ve vícevýběrovém nejvýše `max_select`.
- Volbu nelze uložit, je-li položka **plná** — kontrola kapacity musí být atomická, jinak dvě souběžné přihlášky obsadí totéž lůžko.
- Číselník s `assigned_by = leader` nesmí vyplnit účastník.
- Nesplňuje-li osoba `condition`, číselník se jí nenabízí a volba se odmítne i při přímém požadavku.

### Přihláška

- `person_id` musí být platná osoba (`merged_into_person_id IS NULL`) — na tombstone po sloučení nelze zakládat.
- `EVENT.club_registration_enabled` lze nastavit jen u akcí pořádaných ústředím. `CLUB_REGISTRATION.event_id` musí odkazovat na takovou akci a `CLUB_REGISTRATION.unit_id` musí být oddíl zakladatele.
- `CLUB_REGISTRATION.created_by_account_id` musí být účet s aktivní rolí HVO nebo VO v `CLUB_REGISTRATION.unit_id`; `state = 'closed'` vyžaduje `closed_at` a uzavřený kontejner nepřijímá nové přihlášky.
- Kombinace `CLUB_REGISTRATION.event_id` + `CLUB_REGISTRATION.unit_id` je unikátní; jeden oddíl má na jedné akci nejvýše jeden klubový záznam. `public_name` nesmí být prázdný.
- `CLUB_REGISTRATION.share_token` je náhodný, jedinečný a opravňuje pouze k založení nebo dokončení přihlášky dítěte vázané na stejný `event_id` a `unit_id`; token nezpřístupňuje jiné akce ani seznam osob oddílu.
- `CLUB_REGISTRATION.public_name` je snapshot názvu oddílu v okamžiku založení; veřejně se zobrazí pouze tehdy, když `EVENT.status = 'published'`, `EVENT.visibility = 'public'`, akce je ústředí, klubový režim je zapnutý a kontejner je otevřený. Veřejný výpis nesmí obsahovat vedoucího, účastníky ani jejich počet.
- Veřejné připojení k existujícímu klubu musí použít `CLUB_REGISTRATION.id` z aktuálního seznamu nebo platný `share_token`; pod stejnou akcí může vzniknout nejvýše jedna přihláška osoby bez ohledu na vstupní cestu.
- `REGISTRATION.club_registration_id` smí být vyplněné jen při zapnutém režimu na stejné akci. Dítě musí mít aktivní `PERSON_UNIT` v oddílu klubové přihlášky a jedna osoba může mít v dané akci nejvýše jednu individuální přihlášku.
- Klubová přihláška nemá vlastní cenu, stav úhrady ani kapacitní místo. Kapacitu akce zvyšují pouze individuální přihlášky pod ní, které splní stejné podmínky jako ostatní účastnické přihlášky; jejich schválení zákonným zástupcem se vyhodnocuje samostatně.
- `person_id` je **účastník**, právě jeden na přihlášku; kdo přihlášku podal, drží `submitted_by_account_id` (NULL u podání tokenem).
- `contact_email` je **doručovací adresa přihlášky, ne kontakt osoby**. Povinný — a musí projít formátem e-mailu — právě tehdy, když `submitted_by_account_id IS NULL`; jinak zůstává prázdný a adresa se bere z účtu podavatele.
- Dílčí přihláška dědí `contact_email` z nadřazené, dokud nemá vlastní hodnotu.
- Dílčí přihláška (`parent_registration_id`) musí patřit **téže akci** jako nadřazená a nesmí mít vlastní dílčí přihlášky (zanoření jen jedna úroveň).
- `guardian_email` má smysl jen u nezletilého bez aktivní vazby na zákonného zástupce; jinak zůstává prázdný.
- Přihlášku nelze podat mimo přihlašovací okno ani nad kapacitu (kromě náhradnických míst).
- Dokumenty a povinné číselníky **náhradníka** jsou uzamčené, dokud nepřijme nabídku.

### Platby

- `PAYMENT_ALLOCATION`: součet alokací jedné transakce **nesmí překročit** její částku (v absolutní hodnotě).
- Do párování vstupují **jen příchozí** platby.
- Záporná alokace (`refund`) nesmí stáhnout součet u přihlášky pod nulu.
- Alokace musí odkazovat na přihlášku akce nebo oddílový členský předpis **téhož oddílu**, jako je bankovní účet transakce (u dávky příspěvků na účet ústředí).
- `external_id` je povinné u **všech** zdrojů — u ručního zápisu se generuje (`manual:<uuid>`), u importu výpisu odvodí z otisku řádku (`stmt:<hash>`).
- **VS ani SS nejsou u transakce povinné** — v nahraném výpisu i u ručního zápisu často chybí; příslušná párovací pravidla se pak jen přeskočí.
- `voided_at` lze nastavit **jen** u transakce se `source != 'import'` a **jen** když nemá žádnou alokaci.
- `api_token_enc` smí být vyplněný jen při `provider = 'fio'`; `provider = 'manual'` vylučuje synchronizační pole (`last_sync_at`, `sync_state`, `last_external_id`).
- `PAYMENT_ALLOCATION` má vyplněné **právě jedno** z `registration_id` / `fee_batch_id` / `unit_member_fee_id` — alokace míří na přihlášku, dávku příspěvků DU nebo oddílový členský předpis.

### Příspěvek DU

- Osobu lze zařadit do dávky jen tehdy, je-li **evidovaná v oddílu dávky** a **nemá pro `year` členství** ani položku v jiné dávce ve stavu `draft`/`locked`.
- `total_amount = počet položek × DU_FEE_RATE.amount` pro `year`; hodnota **zamrzne při uzamčení**, pozdější změna sazby ji nemění.
- **Sazbu pro daný rok nelze změnit, jakmile na něj dorazila první platba** — existuje-li k `year` alespoň jedna dávka ve stavu `paid` nebo s libovolnou alokací, je `DU_FEE_RATE.amount` uzamčená (`locked_at`). Jinak by dva oddíly platily za týž rok různě a částka na už rozeslaných QR by přestala sedět.
- Do první platby smí ADM sazbu upravit; úprava přepočte `total_amount` všem dávkám roku ve stavu `draft` a dávky ve stavu `locked` **zruší** — jejich QR nese starou částku, oddíl musí založit novou.
- Ve stavu `locked` a `paid` nelze měnit položky. Oprava = `canceled` + nová dávka.
- **Příznak člena DU se nastaví jen při úplné úhradě** — částečná alokace nechává dávku v `locked` a nezaloží žádné `DU_MEMBERSHIP`.
- Vznikne-li mezi uzamčením a platbou členství osoby jinou dávkou, položka dostane `skipped_at`, členství se nezaloží podruhé a rozdíl se řeší jako přeplatek dávky.
- Dávka se páruje proti bankovnímu účtu **ústředí**, ne oddílu, který ji podal.

### Osoba a vazby

- Vazba zákonný zástupce ↔ dítě: `parent_person_id ≠ child_person_id`; dítě musí být v okamžiku vzniku nezletilé ([parent-child-lifecycle.md](parent-child-lifecycle.md)).
- `DU_MEMBERSHIP.year` — rozsah rozumných let (např. ⟨2000; aktuální + 1⟩), aby překlep nezaložil členství na rok 20250.
- `PERSON.birth_date` nesmí být v budoucnosti a při běžném založení nebo úpravě osoby nesmí být starší než 90 let k aktuálnímu datu. Tato hranice slouží jen jako kontrola zjevné chyby v datu; věkovou způsobilost pro konkrétní akci určuje její vlastní referenční datum a pravidla.
- `DU_MEMBERSHIP.unit_id` je **evidenční oddíl** — musí to být oddíl, kde je osoba v okamžiku založení evidovaná (`PERSON_UNIT`). Do vyhodnocování ceny a způsobilosti **nevstupuje**; ověřuje se jen existence záznamu pro osobu a rok.
- Kolize při založení členství **není chyba validace, ale stav k zobrazení** — porušení unikátu `person_id + year` se přeloží na hlášku „členství pro rok _R_ už založil oddíl _X_", ne na obecné „nelze uložit".
- Přepsání `unit_id` (převod evidenčního oddílu) je přípustné jen na oddíl, kde je osoba evidovaná, a jen po potvrzení druhou stranou ([authorization.md](authorization.md)).
- `ATTENDANCE_RECORD.status` je jedna z hodnot `on_time`, `late`, `absent` nebo `excused_in_advance`. Stav `on_time` i `late` znamená skutečnou účast.
- `absence_reason` je povinný právě pro `status = 'absent'` a je jedna z hodnot `illness`, `family`, `other_activity`, `school`, `studying`, `forgot`, `grounded`, `unmotivated` nebo `other`; pro ostatní stavy musí být prázdný. `excused_in_advance` je samostatný stav bez povinnosti uvádět důvod.
- `ATTENDANCE_RECORD.volunteer_hours ≥ 0`; hodiny dávají smysl jen u dobrovolníka se stavem `on_time` nebo `late`.
- `PERSON_SENSITIVE_DATA` patří vždy konkrétnímu oddílu — citlivá data se nesdílejí mezi oddíly.

### Hlídky a stanoviště

Pravidla složení (počty členů, věkové limity, právě jeden kapitán) jsou v [race-patrols.md](race-patrols.md). Navíc platí:

- Přiřazení ke stanovišti je **vzájemně výlučné** s členstvím v hlídce.
- Rozhodčí je nejvýše na jednom stanovišti; běžné stanoviště obsadí nejvýše jeden rozhodčí (pseudo-stanoviště „Jakékoliv" je bez limitu).
- Hlídku smí měnit jen vlastnící přihláška.

### Workshopy

- Účastník má v jednom `WORKSHOP_BLOCK` nejvýše **jeden** běh.
- `WORKSHOP.capacity` platí na běh (`WORKSHOP_OFFERING`), ne na workshop jako celek.
- Bloky téže akce se nesmí časově překrývat.
