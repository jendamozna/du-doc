# Vazba zákonný zástupce ↔ dítě — stavový automat

Formální model `PARENT_CHILD` ([README.md](../README.md) → **Zákonný zástupce**). Schéma viz [data-model.md](data-model.md), navazující tok přihlášky [registration-lifecycle.md](registration-lifecycle.md).

## Princip: vazba je zdrojem práv, ne role

Zákonné zastoupení se **nepřiděluje jako role** — postavení zákonného zástupce plyne výhradně z existence vazby ve stavu `active`. Rozsah práv je proto vždy **per dítě**, ne globální, a nemůže se rozejít se skutečným stavem vazby.

Vazba je **asymetrická dvojice osob** (`parent_person_id`, `child_person_id`), ne vazba mezi účty — dítě zpravidla účet nemá. Zákonný zástupce účet mít musí, protože jinak nemá jak práva vykonávat (výjimkou je schválení přihlášky odkazem z e-mailu, které účet nevyžaduje). Pole `relationship_type` popisuje vztah zástupce k dítěti: `mother`, `father`, `guardian` nebo `other`; nemění stav vazby ani rozsah odvozených oprávnění.

## Založení účtu zákonného zástupce

Pokud zákonný zástupce ještě nemá účet, systém mu odešle e-mail s jednorázovým tokenem. Otevřením odkazu a dokončením registrace si zákonný zástupce založí účet; jeho uživatelským jménem je e-mailová adresa, na kterou byla pozvánka odeslána. Po úspěšném založení účtu systém účet spáruje s dítětem vytvořením nebo aktivací vazby `PARENT_CHILD`. Token se po použití okamžitě zneplatní a nelze jej použít znovu.

Tento token slouží k založení účtu a propojení s dítětem. Není totožný s tokenem pro jednorázové schválení přihlášky na akci.

## Stavy

| Stav                       | Význam                                                                                     | Dává práva | Terminální |
| -------------------------- | ------------------------------------------------------------------------------------------ | ---------- | ---------- |
| `pending`                  | čeká na rozhodnutí člověka — dítě už má jiného zákonného zástupce, nebo chybí `birth_date` | ne         | ne         |
| `active`                   | platná vazba, zákonný zástupce má plná práva k dítěti                                      | ano        | ne         |
| `readonly_after_adulthood` | dítě dosáhlo zletilosti, přístup zůstává jen pro čtení                                     | jen čtení  | ne         |
| `canceled`                 | vazba zrušena zákonným zástupcem, HVO nebo zletilým dítětem                                | ne         | ano        |

`canceled` je **terminální** — obnovit vazbu nelze, vzniká nová (původní zůstává pro auditní stopu).

## Vznik vazby

Tři cesty, všechny ústí do `pending` nebo `active` podle toho, zda dítě už nějakého zákonného zástupce má:

| Cesta                                       | Dítě bez zákonného zástupce             | Dítě už má zákonného zástupce                              |
| ------------------------------------------- | --------------------------------------- | ---------------------------------------------------------- |
| Zákonný zástupce přihlásí dítě na akci      | → `active` (s prohlášením o zastoupení) | → `pending`, schvaluje stávající zákonný zástupce nebo HVO |
| Nezletilý se přihlásí sám, zástupce schválí | → `active`                              | → `pending`                                                |
| Pozvánka druhému zákonnému zástupci         | → `pending`, schvaluje HVO              | → `pending`, schvaluje stávající zákonný zástupce          |

**Prohlášení o zastoupení** u první cesty je povinný explicitní souhlas („jsem zákonný zástupce tohoto dítěte") — zapisuje se do auditního logu s časem a aktérem. Bez něj vazba nevznikne.

## Diagram

```mermaid
stateDiagram-v2
    [*] --> pending : přihláška / pozvánka (dítě už má zákonného zástupce)
    [*] --> active : přihláška s prohlášením (dítě bez zákonného zástupce)

    pending --> active : schválil stávající zákonný zástupce nebo HVO
    pending --> canceled : zamítnuto nebo lhůta uplynula

    active --> readonly_after_adulthood : dítě dosáhlo 18 let (job)
    active --> canceled : zákonný zástupce vystoupil / HVO na žádost

    readonly_after_adulthood --> canceled : zletilý zrušil přístup zákonného zástupce

    canceled --> [*]
```

## Přechody

| Přechod                               | Spouštěč                                         | Guard                                                                      | Efekt                                                                   |
| ------------------------------------- | ------------------------------------------------ | -------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| `→ pending`                           | přihláška, schválení zástupcem, pozvánka         | dítě už má aspoň jednu vazbu v `active`, **nebo** nemá `birth_date`        | e-mail schvalovateli, nastavení lhůty                                   |
| `→ active` (přímo)                    | přihláška s prohlášením                          | dítě je nezletilé a **nemá** žádnou vazbu v `active`; prohlášení potvrzeno | `valid_from`, zápis prohlášení do auditního logu                        |
| `pending → active`                    | odkaz v e-mailu / rozhraní                       | schvaluje stávající zákonný zástupce v `active`, nebo HVO oddílu dítěte    | `approved_by_account_id`, `valid_from`, notifikace žadateli             |
| `pending → canceled`                  | schvalovatel, nebo job po lhůtě                  | —                                                                          | `valid_to`, `EMAIL_PARENT_CHILD_REJECTED` žadateli s důvodem            |
| `active → readonly_after_adulthood`   | job (denně)                                      | dítě dosáhlo 18 let                                                        | práva se omezí na čtení, notifikace oběma stranám                       |
| `active → canceled`                   | zákonný zástupce (vystoupení) nebo HVO na žádost | —                                                                          | `valid_to`, zápis do auditního logu, kontrola osiření dítěte (viz níže) |
| `readonly_after_adulthood → canceled` | zletilé dítě                                     | dítě má vlastní účet                                                       | `valid_to`, zákonný zástupce ztrácí i čtení                             |

Zrušení se **vždy loguje** (README → **Auditní log**); u systémových přechodů (job) je aktérem systém.

## Práva podle stavu

| Operace                                     | `pending` | `active` | `readonly_after_adulthood` |
| ------------------------------------------- | --------- | -------- | -------------------------- |
| Číst údaje a přihlášky dítěte               | ne        | ano      | ano                        |
| Přihlásit dítě na akci, stornovat, platit   | ne        | ano      | ne                         |
| Upravovat údaje dítěte (adresa, pojišťovna) | ne        | ano      | ne                         |
| Doplnit chybějící kontaktní e-mail dítěte   | ne        | ano      | **ano** (výjimka)          |

Výjimka u kontaktního e-mailu existuje proto, aby šlo zletilému doručit výzvu k převzetí účtu, když e-mail chybí.

## Guardy a invarianty

- **Oba zákonní zástupci mají plná práva**, mezi vazbami není hierarchie; při souběžné úpravě platí poslední zápis.
- **Vazba nevzniká k zletilé osobě.** Je-li osoba v okamžiku pokusu už zletilá, vazba se nezaloží vůbec — nelze obejít omezení tím, že se založí a hned překlopí do `readonly`.
- **Chybí-li `birth_date`**, nelze zletilost vyhodnotit; systém datum vyžádá a vazba zůstává v `pending`. Schvalovatelem je v tomto případě **HVO oddílu dítěte** — stávající zákonný zástupce nemusí existovat, a přesto někdo rozhodnout musí. Je to druhý důvod, proč se vazba ocitne v `pending`; samostatný stav pro něj nevzniká, protože chování i cesty ven jsou totožné.
- **Osiření dítěte:** zruší-li se poslední vazba v `active`, údaje a přihlášky nezletilého spravuje HVO oddílu, kde je dítě evidované, dokud se nepřipojí nový zástupce. Zrušení se tím **neblokuje** — nelze držet zákonného zástupce proti jeho vůli.
- **Zrušení vazby nemění existující přihlášky ani platby.** Přihlášky zůstávají v platnosti a přechází pod správu HVO (nebo druhého zákonného zástupce); už provedené platby a alokace se nedotýkají.
- **Zrušení nelze provést, dokud je vazba v `pending`** — nejdřív se musí rozhodnout o schválení; zamítnutí je samo přechodem do `canceled`.
- **Sloučení osob** ([person-merge.md](person-merge.md)) přenáší vazby pod sjednocenou osobu; duplicitní vazba (stejný zákonný zástupce i dítě) se sloučí do jedné.
- `record_state` osoby (viz [person-lifecycle.md](person-lifecycle.md)) **vazbu neovlivňuje** — deaktivace osoby v oddílu vazbu neruší.

## Časové lhůty

| Lhůta                                  | Výchozí                    | Kde se nastavuje                                                          |
| -------------------------------------- | -------------------------- | ------------------------------------------------------------------------- |
| schválení vazby (`pending`)            | 14 dní od odeslání žádosti | nastavení oddílu                                                          |
| schválení přihlášky zákonným zástupcem | 7 dní                      | nastavení oddílu ([registration-lifecycle.md](registration-lifecycle.md)) |

Lhůta pro schválení vazby je delší než pro schválení přihlášky — vazba je trvalé rozhodnutí, přihláška má vlastní kapacitní tlak.
