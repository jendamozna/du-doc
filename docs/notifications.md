# Katalog notifikací

Tabulka všech e-mailových událostí zmíněných napříč specifikací ([README.md](../README.md), ostatní `docs/`). Doplňuje [non-functional.md](non-functional.md) → **Odchozí e-maily** (fronta, opakování, evidence odeslání) o konkrétní obsah: **co** se posílá, **komu** a **kdy**.

## Princip

- **Jediný kanál je e-mail** — specifikace nikde nezmiňuje SMS ani push notifikace.
- Každé odeslání se **eviduje** (událost, příjemce, čas), aby fronta při opakování neposlala stejnou zprávu dvakrát (README → **Modul párování plateb**, [non-functional.md](non-functional.md) → **Odchozí e-maily**).
- **Aktér nemusí mít účet** — řada příjemců (zákonný zástupce, host, náhradník) dostává jen tokenový odkaz; e-mail je pro ně celé UI.
- Kód šablony (`EMAIL_*`) je pracovní identifikátor pro implementaci, ne text — přesné znění (předmět, kopie) specifikace nedefinuje a je otevřenou otázkou (viz **Otevřené otázky** níže).

## Přihlášky a zákonný zástupce

| Kód                          | Událost (spouštěč)                                     | Příjemce                              | Načasování                  | Opakování / poznámka                                           | Zdroj                                                                                  |
| ---------------------------- | ------------------------------------------------------ | ------------------------------------- | --------------------------- | -------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| `EMAIL_REG_CONFIRM`          | vznik přihlášky (`registration.created`)               | účastník / rodič                      | ihned                       | jednorázově; obsahuje QR a platební údaje, je-li cena > 0      | [README.md](../README.md#L324), [registration-lifecycle.md](registration-lifecycle.md) |
| `EMAIL_GUARDIAN_REQUEST`     | `guardian.requested` (nezletilý bez rodiče)            | zákonný zástupce (zadaný e-mail)      | ihned po vzniku přihlášky   | jednorázově; odkaz platí do vypršení lhůty (7 dní výchozí)     | [registration-lifecycle.md](registration-lifecycle.md#L90)                             |
| `EMAIL_GUARDIAN_EXPIRED`     | `guardian.expired` (lhůta uplynula)                    | účastník                              | okamžitě po expiraci (job)  | jednorázově                                                    | [registration-lifecycle.md](registration-lifecycle.md#L91)                             |
| `EMAIL_DOCUMENT_REJECTED`    | `document.rejected`                                    | účastník / rodič                      | ihned po posouzení vedoucím | jednorázově na zamítnutí; nese komentář vedoucího              | [README.md](../README.md#L322)                                                         |
| `EMAIL_DOCUMENT_REMINDER`    | chybí povinné dokumenty (job)                          | účastník / rodič                      | podle nastavení oddílu      | opakovaně, dokud nejsou všechny dokumenty schváleny            | [README.md](../README.md#L322) („Nahrání lze vyžádat i připomínkou")                   |
| `EMAIL_PAYMENT_REMINDER`     | nezaplacená/částečně zaplacená přihláška po splatnosti | účastník / rodič                      | podle nastavení oddílu      | opakovaně; četnost per oddíl (README → Nastavení oddílu)       | [README.md](../README.md#L325)                                                         |
| `EMAIL_PAYMENT_CONFIRMATION` | `payment.allocated` (i částečná alokace)               | účastník / rodič                      | ihned po napárování         | jednou na alokaci — eviduje se, aby se neposílalo dvakrát      | [README.md](../README.md#L367)                                                         |
| `EMAIL_SUBSTITUTE_OFFER`     | `substitute.offer` (uvolnění místa)                    | náhradník                             | ihned po výběru vedoucím    | nabídka platí 48 h (výchozí); po vypršení propadá              | [README.md](../README.md#L296), [registration-lifecycle.md](registration-lifecycle.md) |
| `EMAIL_EVENT_CANCELED`       | `event.canceled` (hromadné storno akce)                | všichni účastníci s nekoncovým stavem | ihned                       | jednorázově; informuje o stornu bez poplatku a případné vratce | [registration-lifecycle.md](registration-lifecycle.md#L101)                            |
| `EMAIL_MENTOR_REQUEST`       | přihláška na akci typu „S doporučením" odeslána        | kontakt mentora/vedoucího             | ihned po odeslání přihlášky | jednorázově; žádost o doplnění očekávání a potvrzení           | [README.md](../README.md#L275) (Typy a šablony akcí)                                   |

## Platby a účetnictví

| Kód                       | Událost (spouštěč)                          | Příjemce               | Načasování                  | Opakování / poznámka                                                                    | Zdroj                                      |
| ------------------------- | ------------------------------------------- | ---------------------- | --------------------------- | --------------------------------------------------------------------------------------- | ------------------------------------------ |
| `EMAIL_PAYMENT_REMINDER`  | nezaplacená přihláška po splatnosti (job)   | účastník / rodič       | podle nastavení oddílu      | **jen u účtu s bankovním API** (`provider = 'fio'`); u ruční evidence se neposílá vůbec | [payment-matching.md](payment-matching.md) |
| `EMAIL_FIO_SYNC_FAILURE`  | opakované selhání importu transakcí z banky | Účetní oddílu a HVO    | po opakovaném selhání (job) | dokud selhání trvá — přesná frekvence není specifikovaná                                | [fio-sync.md](fio-sync.md#L33)             |
| `EMAIL_DU_FEE_BATCH_QR`   | uzamčení dávky příspěvků DU                 | HVO (odesílatel dávky) | ihned                       | jednorázově; QR + platební údaje účtu ústředí                                           | [README.md](../README.md) → **Člen DU**    |
| `EMAIL_DU_FEE_BATCH_PAID` | spárování platby dávky účetní ústředí       | HVO                    | po spárování                | jednorázově; seznam osob, kterým vzniklo členství, a přeskočené položky                 | [README.md](../README.md) → **Člen DU**    |

## Rodič ↔ dítě

| Kód                                   | Událost (spouštěč)                            | Příjemce                     | Načasování                | Opakování / poznámka                | Zdroj                                                                                  |
| ------------------------------------- | --------------------------------------------- | ---------------------------- | ------------------------- | ----------------------------------- | -------------------------------------------------------------------------------------- |
| `EMAIL_SECOND_GUARDIAN_INVITE`        | pozvánka druhému zákonnému zástupci           | pozvaný druhý rodič          | ihned                     | jednorázově; vazba vznikne přijetím | [README.md](../README.md#L116), [parent-child-lifecycle.md](parent-child-lifecycle.md) |
| `EMAIL_PARENT_CHILD_PENDING_APPROVAL` | vazba `→ pending` (dítě už má rodiče)         | stávající rodič nebo HVO     | ihned po vzniku žádosti   | jednorázově; lhůta 14 dní (výchozí) | [parent-child-lifecycle.md](parent-child-lifecycle.md#L58-L63)                         |
| `EMAIL_PARENT_CHILD_ADULTHOOD`        | `active → readonly_after_adulthood` (job)     | rodič i dítě (má-li kontakt) | v den dosažení zletilosti | jednorázově                         | [parent-child-lifecycle.md](parent-child-lifecycle.md#L36)                             |
| `EMAIL_ACCOUNT_CLAIM_INVITE`          | doplnění kontaktního e-mailu zletilého dítěte | zletilý člen                 | po doplnění e-mailu       | jednorázově; výzva k převzetí účtu  | [README.md](../README.md#L113)                                                         |

## Role a účty

| Kód                              | Událost (spouštěč)                                      | Příjemce                      | Načasování                   | Opakování / poznámka                            | Zdroj                                           |
| -------------------------------- | ------------------------------------------------------- | ----------------------------- | ---------------------------- | ----------------------------------------------- | ----------------------------------------------- |
| `EMAIL_HVO_INVITE`               | Administrátor vytvoří účet HVO                          | nový HVO                      | ihned                        | jednorázově                                     | [README.md](../README.md#L89)                   |
| `EMAIL_ROLE_INVITE`              | HVO vytvoří účet účetní/vedoucí/rádci                   | nový uživatel role            | ihned                        | jednorázově                                     | [README.md](../README.md#L95)                   |
| `EMAIL_ACCOUNT_LOCKED`           | throttling — série neúspěšných přihlášení               | vlastník účtu                 | ihned                        | jednorázově na zamknutí                         | [non-functional.md](non-functional.md#L45)      |
| `EMAIL_ACCOUNT_INACTIVE_WARNING` | blížící se retenční smazání účtu (24 měsíců nečinnosti) | vlastník účtu (má-li kontakt) | před smazáním (job)          | jednorázově; přesný předstih není specifikovaný | [README.md](../README.md#L155) (Retence a GDPR) |
| `EMAIL_PERSON_INACTIVE_WARNING`  | blížící se automatická deaktivace osoby (bez aktivity)  | osoba (má-li kontakt) a HVO   | 30 dní před deaktivací (job) | jednorázově                                     | [person-lifecycle.md](person-lifecycle.md#L44)  |

## Deduplikace a hlídky

| Kód                      | Událost (spouštěč)                              | Příjemce                                                 | Načasování            | Opakování / poznámka                               | Zdroj                                  |
| ------------------------ | ----------------------------------------------- | -------------------------------------------------------- | --------------------- | -------------------------------------------------- | -------------------------------------- |
| `EMAIL_MERGE_REQUEST`    | žádost o sloučení osob                          | iniciátor, HVO druhého oddílu, účet kandidáta (má-li ho) | ihned                 | jednorázově; opakuje se při doplnění dalších stran | [README.md](../README.md#L171)         |
| `EMAIL_PATROL_REMINDER`  | N dní před akcí, závodník bez hlídky            | vedoucí                                                  | N dní před akcí (job) | jednorázově                                        | [race-patrols.md](race-patrols.md#L27) |
| `EMAIL_PATROL_DISSOLVED` | hlídka se rozpustí po porušení pravidel složení | vlastník přihlášky (zakladatel hlídky)                   | ihned                 | jednorázově; nese důvod rozpuštění                 | [race-patrols.md](race-patrols.md#L26) |

## Systém a organizace

| Kód                         | Událost (spouštěč)                           | Příjemce                      | Načasování                     | Opakování / poznámka                 | Zdroj                                      |
| --------------------------- | -------------------------------------------- | ----------------------------- | ------------------------------ | ------------------------------------ | ------------------------------------------ |
| `EMAIL_UNDELIVERABLE_ALERT` | trvale neodeslaný e-mail po vyčerpání pokusů | vedoucí (kontext dané zprávy) | po posledním neúspěšném pokusu | jednorázově; nesmí zůstat jen v logu | [non-functional.md](non-functional.md#L46) |

Interní upozornění bez e-mailu (jen UI/log): zrušení regionu s aktivními oddíly upozorní ADM v rozhraní ([region-lifecycle.md](region-lifecycle.md#L54)); nemá smysl posílat e-mail administrátorovi, který operaci sám právě provedl.

## Otevřené otázky

Specifikace definuje **kdy** se posílá a **komu**, ale ne přesný **obsah**:

- Žádná zpráva nemá definovaný předmět ani text šablony — jen účel.
- Není řečeno, zda si oddíl může šablony e-mailů upravovat, nebo jsou pevné (na rozdíl od šablony potvrzení o platbě, kterou účetní nahrává — README → **Modul Potvrzení o platbě**).
- U `EMAIL_ACCOUNT_INACTIVE_WARNING` a `EMAIL_PERSON_INACTIVE_WARNING` není určen přesný předstih u účtu (jen u osoby je explicitně 30 dní).
- Frekvence `EMAIL_PAYMENT_REMINDER` a `EMAIL_FIO_SYNC_FAILURE` je „podle nastavení oddílu" bez výchozí hodnoty.
