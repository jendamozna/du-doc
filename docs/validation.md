# Validační pravidla a byznys-invarianty

Doplněk k [data-model.md](data-model.md), který popisuje **co je platná hodnota** a **co nesmí nastat**. Stavové přechody řeší lifecycle dokumenty ([registration-lifecycle.md](registration-lifecycle.md), [person-lifecycle.md](person-lifecycle.md), [parent-child-lifecycle.md](parent-child-lifecycle.md), [region-lifecycle.md](region-lifecycle.md)).

## Princip

- **Validace na hranici systému** — všechna pravidla se vynucují na serveru bez ohledu na to, co kontroluje UI. Klientská validace je pohodlí, ne ochrana.
- **Invarianty i v databázi** — unikátnosti a nepřekryvy z tabulek níže patří do schématu jako `UNIQUE` / `EXCLUDE`, ne jen do aplikační logiky. Souběžné požadavky by je jinak obešly.
- **Povinnost je kontextová, ne absolutní** — většina polí osoby je povinná až podle toho, co vyžaduje šablona akce nebo stav osoby (viz **Podmíněná povinnost**).
- Pravidla označená **[K rozhodnutí]** ve specifikaci chybí a jsou zde jako návrh.

## Formáty

| Údaj              | Pravidlo                                                                                                                            | Zdroj                                             |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| Částka            | desetinné číslo v CZK, **nezaokrouhluje se**; porovnává se přesně (rozdíl 1 Kč = nedoplatek/přeplatek)                              | [README.md](../README.md) → Modul párování plateb |
| Datum a čas       | ukládá se v UTC, zobrazuje v `Europe/Prague`; čistě datumové údaje se nepřepočítávají                                               | [non-functional.md](non-functional.md)            |
| E-mail            | **[K rozhodnutí]** návrh: syntaktická kontrola + normalizace na malá písmena; doménu neověřovat DNS dotazem                         | —                                                 |
| IČO               | **[K rozhodnutí]** návrh: 8 číslic včetně kontrolní číslice (modulo 11); povinné u typu `branch` a `collective`, prázdné u `hq_ico` | [README.md](../README.md) → Oddíl                 |
| Křestní jméno     | proti `NAME_WHITELIST`; neshoda se dá povolit výjimkou `NAME_EXCEPTION` v rámci oddílu                                              | [README.md](../README.md) → Deduplikace           |
| Příjmení          | **neověřuje se** proti žádnému seznamu                                                                                              | [README.md](../README.md) → Deduplikace           |
| Adresa            | volný text (`PERSON.address`) — **[K rozhodnutí]**, zda strukturovat na ulici/město/PSČ                                             | [data-model.md](data-model.md)                    |
| GPS souřadnice    | `lat` ∈ ⟨−90; 90⟩, `lng` ∈ ⟨−180; 180⟩                                                                                              | [data-model.md](data-model.md) → LOCATION         |
| Variabilní symbol | číselný, generuje systém při vzniku přihlášky; neposkytuje ho uživatel                                                              | [payment-matching.md](payment-matching.md)        |
| Specifický symbol | číselný, zadává vedoucí u akce                                                                                                      | [README.md](../README.md) → Konfigurace akce      |
| Soubor dokumentu  | max **10 MB**, typ PDF/JPG/PNG/HEIC ověřený podle **obsahu, ne přípony**                                                            | [non-functional.md](non-functional.md)            |

## Podmíněná povinnost polí osoby

Která pole `PERSON` musí být vyplněná, závisí na kontextu:

| Kontext                          | Povinná pole                                                   | Zdroj                                                  |
| -------------------------------- | -------------------------------------------------------------- | ------------------------------------------------------ |
| Host v oddílu                    | jméno **a** příjmení, **nebo** přezdívka                       | [README.md](../README.md) → Hlavní vedoucí             |
| Registrovaný člen                | jméno, příjmení, pohlaví, **datum narození**                   | [README.md](../README.md) → Hlavní vedoucí             |
| Přihláška nezletilého bez rodiče | `birth_date` (jinak nelze vyhodnotit bránu) + `guardian_email` | [registration-lifecycle.md](registration-lifecycle.md) |
| Akce typu „S certifikátem"       | tituly před/za + adresa trvalého bydliště                      | [README.md](../README.md) → Typy a šablony             |
| Člen hlídky na závodě            | `birth_date` (bez něj nelze ověřit složení hlídky)             | [race-patrols.md](race-patrols.md)                     |
| Vlastník účtu                    | `ACCOUNT.login_email`                                          | [data-model.md](data-model.md)                         |

Ostatní pole (`nickname`, `insurance_company`, `address`) jsou povinná jen tehdy, označí-li je tak šablona akce nebo `EVENT_CUSTOM_FIELD.required`.

## Unikátnosti

| Entita               | Klíč                                        | Poznámka                                                                   |
| -------------------- | ------------------------------------------- | -------------------------------------------------------------------------- |
| `ACCOUNT`            | `login_email`                               | přihlašovací e-mail; `PERSON.email` unikátní **není**                      |
| `ACCOUNT`            | `person_id`                                 | jedna osoba má nejvýše jeden účet                                          |
| `OAUTH_IDENTITY`     | `provider` + `provider_user_id`             | jedna externí identita patří jednomu účtu                                  |
| `USER_ROLE`          | `account_id` + `unit_id` + `role`           | tatáž role se v oddílu nepřiděluje dvakrát                                 |
| `DU_MEMBERSHIP`      | `person_id` + `year`                        | **`unit_id` do klíče nepatří** — jedno členství DU na osobu a rok globálně |
| `DU_FEE_RATE`        | `year`                                      | jedna sazba příspěvku na rok                                               |
| `DU_FEE_BATCH`       | `vs`                                        | variabilní symbol musí dávku jednoznačně identifikovat                     |
| `DU_FEE_BATCH_ITEM`  | `batch_id` + `person_id`                    | osoba je v jedné dávce nejvýše jednou                                      |
| `ATTENDANCE_RECORD`  | `event_id` + `person_id`                    | nejvýše jeden docházkový záznam na osobu a akci                            |
| `EVENT_ASSIGNMENT`   | `event_id` + `account_id` (otevřený záznam) | jedno **aktivní** přiřazení na účet a akci; uzavřených může být víc        |
| `BANK_TRANSACTION`   | `bank_account_id` + `external_id`           | idempotentní zápis — opakované stažení ani nahrání výpisu platbu nezdvojí  |
| `RACE_PATROL`        | `event_id` + `name`                         | název hlídky je unikátní v rámci akce                                      |
| `RACE_PATROL_MEMBER` | `person_id` + `event_id` (přes hlídku)      | osoba je nejvýše v jedné hlídce téže akce                                  |
| `EVENT`              | `share_slug`                                | sdílecí odkaz je globálně unikátní a nepředvídatelný                       |
| `REGISTRATION`       | `vs`                                        | variabilní symbol musí párování jednoznačně identifikovat                  |
| `PERSON_UNIT`        | `person_id` + `unit_id` (otevřený záznam)   | osoba má v oddílu nejvýše jeden platný záznam                              |

## Invarianty po entitách

### Oddíl a region

- **Ústředí (`is_hq`) nemá registrované členy** a nepatří do žádného regionu.
- `UNIT_REGION`: intervaly téhož oddílu se **nesmí překrývat**, nejvýše jeden otevřený (`valid_to = NULL`); díra povolená je ([region-lifecycle.md](region-lifecycle.md)).
- `LOCATION` je viditelná jen v rámci vlastnícího oddílu — akci nelze přiřadit lokaci cizího oddílu.

### Akce

- **Splatnost je výlučná** — vyplněno buď `payment_due_days`, nebo `payment_due_date`, nikdy obojí ani nic.
- `registration_from < registration_to`, `starts_at < ends_at`; přihlašovací okno smí přesahovat začátek akce.
- `visibility` má tři **vzájemně výlučné** hodnoty; `share_slug` má **každá** akce bez ohledu na viditelnost.
- `capacity ≥ 0`, `substitute_count ≥ 0`. Kapacitu nelze snížit pod počet přihlášek, které se do ní už počítají.
- Dobrovolnická pole (`volunteer_registration_*`) dávají smysl jen při `volunteers_enabled = true`.
- Akce bez přihlášek (typ `club`, `one_off`) nesmí mít ceny, storno pravidla ani otevřenou registraci.
- `bank_account_id` musí patřit **témuž oddílu** jako akce.
- `EVENT_ASSIGNMENT` se **nemaže** — odebrání přístupu vyplní `revoked_at` a `revoked_by_account_id`; `revoked_at >= assigned_at` a uzavřený záznam už nelze měnit. Změna rozsahu příznaků uzavře starý záznam a založí nový.
- Kontrola oprávnění pracuje výhradně se záznamy `revoked_at IS NULL`; uzavřené jsou doklad o minulém přístupu ([authorization.md](authorization.md)).

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
- Dílčí přihláška (`parent_registration_id`) musí patřit **téže akci** jako nadřazená a nesmí mít vlastní dílčí přihlášky (zanoření jen jedna úroveň).
- `guardian_email` má smysl jen u nezletilého bez aktivní vazby na rodiče; jinak zůstává prázdný.
- Přihlášku nelze podat mimo přihlašovací okno ani nad kapacitu (kromě náhradnických míst).
- Dokumenty a povinné číselníky **náhradníka** jsou uzamčené, dokud nepřijme nabídku.

### Platby

- `PAYMENT_ALLOCATION`: součet alokací jedné transakce **nesmí překročit** její částku (v absolutní hodnotě).
- Do párování vstupují **jen příchozí** platby.
- Záporná alokace (`refund`) nesmí stáhnout součet u přihlášky pod nulu.
- Alokace musí odkazovat na přihlášku akce **téhož oddílu**, jako je bankovní účet transakce (u dávky příspěvků na účet ústředí).
- `external_id` je povinné u **všech** zdrojů — u ručního zápisu se generuje (`manual:<uuid>`), u importu výpisu odvodí z otisku řádku (`stmt:<hash>`).
- **VS ani SS nejsou u transakce povinné** — v nahraném výpisu i u ručního zápisu často chybí; příslušná párovací pravidla se pak jen přeskočí.
- `voided_at` lze nastavit **jen** u transakce se `source != 'import'` a **jen** když nemá žádnou alokaci.
- `api_token_enc` smí být vyplněný jen při `provider = 'fio'`; `provider = 'manual'` vylučuje synchronizační pole (`last_sync_at`, `sync_state`, `last_external_id`).
- `PAYMENT_ALLOCATION` má vyplněné **právě jedno** z `registration_id` / `fee_batch_id` — alokace míří buď na přihlášku, nebo na dávku příspěvků.

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

- Vazba rodič ↔ dítě: `parent_person_id ≠ child_person_id`; dítě musí být v okamžiku vzniku nezletilé ([parent-child-lifecycle.md](parent-child-lifecycle.md)).
- `DU_MEMBERSHIP.year` — rozsah rozumných let (např. ⟨2000; aktuální + 1⟩), aby překlep nezaložil členství na rok 20250.
- `DU_MEMBERSHIP.unit_id` je **evidenční oddíl** — musí to být oddíl, kde je osoba v okamžiku založení evidovaná (`PERSON_UNIT`). Do vyhodnocování ceny a způsobilosti **nevstupuje**; ověřuje se jen existence záznamu pro osobu a rok.
- Kolize při založení členství **není chyba validace, ale stav k zobrazení** — porušení unikátu `person_id + year` se přeloží na hlášku „členství pro rok _R_ už založil oddíl _X_", ne na obecné „nelze uložit".
- Přepsání `unit_id` (převod evidenčního oddílu) je přípustné jen na oddíl, kde je osoba evidovaná, a jen po potvrzení druhou stranou ([authorization.md](authorization.md)).
- `ATTENDANCE_RECORD.volunteer_hours ≥ 0`; hodiny dávají smysl jen u dobrovolníka.
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

## Otevřené otázky

- **Formát e-mailu, IČO a adresy** — specifikace je nedefinuje vůbec (návrhy výše jsou označené **[K rozhodnutí]**).
- **Telefonní číslo** v datovém modelu neexistuje, přestože u dětských akcí bývá kontakt na rodiče provozně nutný.
- **Minimální délka a složitost hesla** není specifikovaná; [non-functional.md](non-functional.md) řeší jen hashování (Argon2id).
- **Horní věková hranice** u `birth_date` (kontrola překlepu v roce) není nikde uvedená.
