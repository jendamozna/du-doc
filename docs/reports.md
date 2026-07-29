# Reporty — implementační specifikace

Rozpis reportů popsaných v [../README.md](../README.md) do podoby, ze které lze přímo implementovat dotazy a API. Business popis (co report říká a komu) zůstává v README, zde je **jak se počítá**.

---

## Společná pravidla

### Rozsah dat (scope)

| Role                 | Vidí                                         |
| -------------------- | -------------------------------------------- |
| Rádce (RAD)          | nic (reporty nemá)                           |
| Vedoucí družiny (VD) | jen osoby své družiny (`UNIT_PATROL_MEMBER`) |
| Vedoucí oddílu (VO)  | svůj oddíl                                   |
| Hlavní vedoucí (HVO) | svůj oddíl                                   |
| Účetní (UCE)         | jen report Platby, svůj oddíl                |
| Administrátor (ADM)  | vše (napříč oddíly, s dimenzí region)        |

Scope se vždy aplikuje jako filtr `unit_id` odvozený z `USER_ROLE`, ne z parametru requestu — parametr `unit_id` se validuje proti povoleným oddílům volajícího.

### Společné parametry

| Parametr     | Typ                            | Default              | Poznámka                                          |
| ------------ | ------------------------------ | -------------------- | ------------------------------------------------- |
| `unit_id`    | int / list                     | oddíly volajícího    | ADM může `null` = všechny                         |
| `from`, `to` | date                           | posledních 12 měsíců | uzavřený interval, v `Europe/Prague`              |
| `bucket`     | `month` \| `quarter` \| `year` | `month`              | granularita časové osy                            |
| `region_id`  | int                            | null                 | jen ADM; filtruje přes `EVENT.region_id_snapshot` |
| `event_type` | enum `EVENT.type`              | null                 | filtr na typ akce                                 |

### Konvence výpočtu

- **Časová osa má prázdné koše.** Období bez dat se vrací s nulou (ne vynechané), aby graf nelhal o trendu.
- **Časové pásmo:** bucketing podle `Europe/Prague`; datetime sloupce jsou v UTC, převod v dotazu, ne v aplikaci.
- **Členství DU se vyhodnocuje k roku akce** — `DU_MEMBERSHIP(person_id, year = YEAR(EVENT.starts_at))`, nikdy podle aktuálního data.
- **Aktivní přihláška** = `REGISTRATION.state NOT IN ('Canceled', 'Expired')`. Storna se počítají jen v reportu Platby.
- **Anonymizované osoby** (`PERSON.anonymized_at IS NOT NULL`) se do agregací počítají (počty musí sedět historicky), ale nikdy se nevypisují jmenovitě — detailní řádky je vynechávají.
- **Region** je vždy snapshot z akce (`EVENT.region_id_snapshot`), nikdy aktuální zařazení oddílu.
- **Duplicity osob:** agregace „počet unikátních osob" respektuje `REPORT_MERGE` (viz R9); ostatní reporty počítají osoby podle `PERSON.id`.

### Výstup a API

- Jeden endpoint na report: `GET /api/reports/{code}` s parametry výše, odpověď `{ meta: {...parametry, generated_at}, series: [...], totals: {...} }`.
- Každý report podporuje `format=csv` (stejná data, plochá tabulka) pro export do tabulkového procesoru.
- Číselné hodnoty vždy zaokrouhlené na výstupu, ne v mezivýpočtu; peníze v CZK na 2 desetinná místa.
- Reporty jsou **read-only a idempotentní** — žádný report nesmí zapisovat.
- Cachovat na úrovni odpovědi (klíč = role + parametry) s krátkou platností; není potřeba materializovaný snapshot.

---

## R1 — Seznam akcí a docházky

**Kód:** `events` · **Kdo:** VO, HVO, ADM

Provozní přehled: jeden řádek na akci, s rozpadem účastníků podle typu.

| Sloupec              | Výpočet                                                                                 |
| -------------------- | --------------------------------------------------------------------------------------- |
| akce                 | `EVENT.name`, `type`, `starts_at`, `ends_at`                                            |
| přihlášeno           | počet aktivních `REGISTRATION` s `category = 'participant'`                             |
| dorazilo             | počet `ATTENDANCE_RECORD` s `present = true`                                            |
| nedorazilo           | počet `ATTENDANCE_RECORD` s `present = false`                                           |
| členové DU           | z těch, kdo dorazili, ti s `DU_MEMBERSHIP` pro rok akce                                 |
| registrovaní členové | `PERSON_UNIT.membership_state = 'registered_member'` v oddílu akce, aktivní k datu akce |
| hosté                | zbytek (osoba bez aktivní vazby na pořádající oddíl nebo `membership_state = 'guest'`)  |
| vedoucí / rádci      | osoby s `USER_ROLE` VO/HVO/VD resp. RAD v oddílu akce                                   |
| dobrovolníci         | přihlášky s `category = 'volunteer'`, které mají docházku                               |
| odpracované hodiny   | `SUM(ATTENDANCE_RECORD.volunteer_hours)`                                                |

**Hrany:**

- Osoba může mít víc rolí (rádce i dobrovolník) — kategorie se **nevylučují**, součet sloupců proto nemusí dát počet účastníků. Do UI patří poznámka.
- Akce bez přihlášek (`club`, `one_off`) mají „přihlášeno" prázdné, ne nulu.

---

## R2 — Počty členů v čase

**Kód:** `members` · **Kdo:** VO, HVO, ADM

Vývoj velikosti oddílu. Metrika je **stav ke konci každého období**, ne přírůstek.

- **Registrovaní členové:** počet `PERSON_UNIT` s `membership_state = 'registered_member'`, `record_state = 'active'` a intervalem `valid_from <= konec_období AND (valid_to IS NULL OR valid_to > konec_období)`.
- **Členové DU:** počet `DU_MEMBERSHIP` s `year = rok(konec_období)` a `unit_id` v scope. U měsíčního bucketu je hodnota v rámci roku konstantní — to je správně, členství je roční.
- **Hosté:** totéž co registrovaní členové, ale `membership_state = 'guest'`.
- **Přírůstek / úbytek:** rozdíl proti předchozímu koši, dopočítaný na výstupu.

**Hrany:**

- Osoba ve více oddílech se počítá v každém oddílu zvlášť; při agregaci přes více oddílů (ADM) se nabízí navíc řádek „unikátní osoby" (`COUNT(DISTINCT person_id)`).
- Historii je nutné číst z intervalů `PERSON_UNIT.valid_from/valid_to`, ne z `PERSON_UNIT_HISTORY` — historie slouží k přechodům (R6), ne ke stavu.

---

## R3 — Účast na akcích

**Kód:** `attendance-events` · **Kdo:** VO, HVO, ADM

Časová řada přes akce s přihláškami (`type NOT IN ('club', 'one_off')`), bucket podle `EVENT.starts_at`.

| Metrika              | Výpočet                                                                                |
| -------------------- | -------------------------------------------------------------------------------------- |
| počet akcí           | `COUNT(EVENT)`                                                                         |
| počet účastníků      | aktivní přihlášky `category = 'participant'`                                           |
| unikátní osoby       | `COUNT(DISTINCT REGISTRATION.person_id)` v období                                      |
| naplněnost kapacit   | `SUM(účastníci) / SUM(EVENT.capacity)`; akce bez kapacity se do jmenovatele nezapočítá |
| podíl náhradníků     | přihlášky `category = 'substitute'` / všechny aktivní přihlášky                        |
| úspěšnost náhradníků | `SUBSTITUTE_OFFER.state = 'accepted'` / všechny odeslané nabídky                       |

**Hrany:** akce s `capacity IS NULL` musí být z výpočtu naplněnosti vyloučená úplně (jinak vyjde 0 %); počet takových akcí se vrací v `meta`.

---

## R4 — Docházka pravidelných schůzek

**Kód:** `clubs` · **Kdo:** VD, VO, HVO, ADM

Sezónnost pravidelných schůzek — jen akce `type = 'club'`, bucket podle `EVENT.starts_at`.

| Metrika              | Výpočet                                                           |
| -------------------- | ----------------------------------------------------------------- |
| počet schůzek        | `COUNT(EVENT)`                                                    |
| přítomných celkem    | `COUNT(ATTENDANCE_RECORD WHERE present = true)`                   |
| průměrná návštěvnost | přítomní / počet schůzek, zaokrouhleno na 1 desetinné místo       |
| docházka osoby       | volitelný rozpad: osoba × podíl přítomnosti (`present / schůzky`) |

**Hrany:**

- Schůzka **bez jediného docházkového záznamu** se do průměru počítá jako nula pouze tehdy, pokud proběhla (`starts_at < now`); budoucí schůzky se vylučují.
- Filtr `unit_patrol_id` (družina) se aplikuje přes `UNIT_PATROL_MEMBER` osoby — VD ho má vynucený.

---

## R5 — Dobrovolnické hodiny

**Kód:** `volunteers` · **Kdo:** VO, HVO, ADM

- **Hodiny v období:** `SUM(ATTENDANCE_RECORD.volunteer_hours)` přes akce v koši.
- **Klasifikace osoby:** součet hodin osoby **za kalendářní rok**; `< 50` = krátkodobý, `>= 50` = dlouhodobý dobrovolník. Hranice je konfigurovatelná v nastavení oddílu.
- **Výstup:** hodiny na časové ose + počet krátkodobých/dlouhodobých k závěru každého roku + jmenný seznam s hodinami (jen pro scope volajícího).

**Hrany:**

- Klasifikace je **roční, ne za zvolené období** — jinak by se stejná osoba měnila podle filtru. Když zvolený interval nekryje celý rok, vrací se klasifikace za dotčené roky a `meta.note` to uvádí.
- Hodiny se evidují jen na docházkovém záznamu; přihláška s `category = 'volunteer'` bez docházky = 0 hodin, ne chybějící údaj.

---

## R6 — Retence a odchody

**Kód:** `retention` · **Kdo:** HVO, ADM

Zdrojem jsou přechody v `PERSON_UNIT_HISTORY` (proto se tato historie nesmí slučovat do obecného auditního logu).

| Metrika         | Výpočet                                                               |
| --------------- | --------------------------------------------------------------------- |
| odchody         | přechody `from_record = 'active' AND to_record = 'inactive'` v období |
| reaktivace      | přechody `from_record = 'inactive' AND to_record = 'active'`          |
| archivace       | přechody `to_record = 'archived'` (GDPR retence)                      |
| míra reaktivace | reaktivace / odchody za stejné období                                 |
| retence         | osoby aktivní na začátku i na konci období / osoby aktivní na začátku |

**Hrany:**

- Osoba může odejít a vrátit se ve stejném období — počítají se **přechody**, ne osoby; unikátní počet osob se vrací zvlášť.
- Automatická deaktivace (job) i ruční změna se logují stejně; rozlišit lze podle `changed_by_account_id IS NULL` (systém).

---

## R7 — Platby

**Kód:** `payments` · **Kdo:** UCE, HVO, ADM

Bucket podle data akce (`EVENT.starts_at`), varianta „cash-flow" podle `BANK_TRANSACTION.date`.

| Metrika              | Výpočet                                                                                             |
| -------------------- | --------------------------------------------------------------------------------------------------- |
| předepsáno           | `SUM(cena přihlášky)` přes aktivní přihlášky (cena = `EVENT_PRICE` + příplatky číselníků)           |
| inkasováno           | `SUM(PAYMENT_ALLOCATION.amount)` k těmto přihláškám                                                 |
| pohledávky           | předepsáno − inkasováno, jen `state IN ('PendingPayment', 'PartialPaid')`                           |
| přeplatky            | `SUM(alokace − cena)` u `state = 'Overpayment'`                                                     |
| nespárované platby   | příchozí `BANK_TRANSACTION` bez alokace, se stářím (0–7 / 8–30 / 30+ dní)                           |
| storna               | přihlášky `state = 'Canceled'`, počet + předepsaná částka + storno poplatek dle `CANCELLATION_RULE` |
| zaplaceno včas/pozdě | podíl přihlášek, kde datum poslední alokace ≤ termín splatnosti (viz níže)                          |

**Termín splatnosti** se nikde neukladá, počítá se relačně: `REGISTRATION.created_at + 14 dní` (lhůta je klíč v nastavení oddílu, default 14). Když akce začíná dřív, platí dřější z obou dat — `MIN(created_at + lhůta, EVENT.starts_at)`. Stejný výpočet používají výzvy k platbě a připomínky, aby report a notifikace nemohly dát různý výsledek.

**Hrany:**

- Splatnost se **nepřepočítává**, když se změní cena přihlášky nebo lhůta v nastavení oddílu — rozhoduje `created_at` přihlášky. Historické reporty tím zůstávají stabilní.
- Přihláška podaná později než 14 dní před akcí je splatná k začátku akce; přihláška podaná v den akce je splatná ihned a do „pozdě" spadne jen při úhradě po skončení dne.
- U částečně zaplacených přihlášek rozhoduje **poslední** alokace, která dorovnala cenu; nedoplacená přihláška po splatnosti se počítá do pohledávek po splatnosti, ne do „pozdě zaplacených".
- Odchozí transakce (`amount < 0`) se do inkasa nezapočítávají.
- Vratky se evidují jako záporná alokace — do „inkasováno" vstupují se znaménkem, aby souhlasil zůstatek.

---

## R8 — Vzdělávání

**Kód:** `education` · **Kdo:** HVO, ADM

- **Platné kurzy k datu:** `PERSON_COURSE` s `completed_on <= datum AND (valid_to IS NULL OR valid_to >= datum)`; na časové ose stav ke konci každého koše.
- **Blížící se expirace:** `valid_to` v intervalu `<now, now + N dní>`, default `N = 90`, parametr `expiring_in_days`.
- **Pokrytí:** podíl osob s rolí VO/HVO/VD/RAD v oddílu, které mají platný daný kurz — jmenovatel jsou aktivní vedoucí, ne všechny osoby.
- **Rozpad:** kurz × počet platných × počet expirujících × počet propadlých.

**Hrany:** kurz s `validity_months = NULL` je trvalý — nikdy neexpiruje a do „expirujících" nepatří.

---

## R9 — Unikátní děti (modul reporty ústředí)

**Kód:** `unique-children` · **Kdo:** ADM

Nejcitlivější report — vstupuje do vykazování ústředí, proto je definice závazná.

1. **Základ:** osoby s aspoň jednou aktivní přihláškou na akci v období.
2. **Vyloučení:** osoby, které jsou hostem cizího oddílu ve všech svých vazbách (`PERSON_UNIT.membership_state = 'guest'`) a nemají vazbu na pořádající oddíl.
3. **Deduplikace:** dvojice v `REPORT_MERGE` se považují za jednu osobu. Implementace je **union-find** nad všemi dvojicemi (tranzitivně: A–B, B–C ⇒ jedna osoba). Záznamy osob zůstávají oddělené, sloučení je jen výpočetní.
4. **Agregace:** `COUNT` výsledných skupin, volitelně po `EVENT.region_id_snapshot`.

**Kandidáti na sloučení:** shoda `LOWER(first_name)`, `LOWER(last_name)` a `birth_date` napříč oddíly u neanonymizovaných osob → nabídka ke schválení. Vrací se jméno, příjmení a datum narození, nic dalšího.

**Hrany:**

- Osoba se v jednom období počítá jednou, i když byla na deseti akcích deseti oddílů.
- Report se počítá **za kalendářní rok** (vykazovací období), i když UI dovolí jiný interval.

---

## Požadavky na datový model

Reporty výše lze postavit nad stávajícím modelem s těmito výjimkami:

| Chybí                                     | Potřebuje report    | Návrh                                                                      |
| ----------------------------------------- | ------------------ | -------------------------------------------------------------------------- |
| čas vzniku přihlášky                      | R7 (splatnost)     | `REGISTRATION.created_at` (datetime) — splatnost = `created_at + 14 dní`   |
| čas přechodu stavu přihlášky              | R7 (storna v čase) | `REGISTRATION.state_changed_at` nebo čtení z `AUDIT_LOG` (action `cancel`) |
| lhůta splatnosti a hranice dobrovolníka   | R7, R5             | klíče v nastavení oddílu, default 14 dní a 50 hodin                        |

Bez těchto polí se příslušné metriky nevrací (ne odhadují) a UI je skryje.

---

## Co reporty záměrně neřeší

- **Žádný report nevrací citlivé údaje** (zdravotní informace, dokumenty, adresy) — ani v detailním rozpadu.
- **Žádná predikce.** Reporty popisují minulost; odhady a doporučení patří do [../AI_support.md](../AI_support.md).
- **Žádné vlastní sestavy uživatele** v první fázi — sada reportů je pevná, rozšiřuje se vývojem.
