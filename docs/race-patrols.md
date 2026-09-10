# Hlídky na závodních akcích — model

Implementační detail k [README.md](../README.md) → **Hlídky na závodních akcích (Stezka)**, kde jsou popsané kategorie a pravidla složení. Schéma viz [data-model.md](data-model.md).

## Registration scope

- Hlídky se skládají z osob **vlastní přihlášky a jejích dílčích přihlášek** (podregistrací), jejichž stav není `Canceled`, `Expired` ani `New`. Tato množina se dál označuje jako **registration scope**.

## Vlastnictví a členství

- Hlídku vlastní přihláška, která ji založila (`RACE_PATROL.owner_registration_id`); upravovat a mazat ji smí jen vlastník. Název je unikátní v rámci akce.
- Každá osoba je nejvýše v jedné hlídce (`RACE_PATROL_MEMBER`).
- Kapitána (`role = leader`) lze zvolit jen v kategoriích Stezka a Pěšinka, kde je povinný; automaticky se jím stane první přidaný člen (`joined_at`) a lze ho později změnit. V ostatních kategoriích jsou všichni členové `member`.

## Výpočet věku

- Referenční datum řídí konfigurační volba akce `age_at_year_end`:
  - zapnuto (výchozí) — věk ke **konci aktuálního roku**: `věk = rok(31. 12. letošního roku) − rok(datum narození)`,
  - vypnuto — věk **k datu konání akce**.
- Rozdíl se počítá v letech (date diff). Chybí-li datum narození, člena nelze plně ověřit a kontrola konzistence to hlásí.

## Kontrola konzistence

- Pravidla složení (počty členů, způsobilost, věkové limity podle kategorie) ověřuje **jediná čistá funkce**.
- Poruší-li hlídka pravidla po změně relevantního údaje člena (věk, příznak závodníka, kategorie), **hlídka se rozpustí** — všichni členové se odpojí, hlídka se smaže a vlastník dostane informaci s důvodem.
- **Odchod ze scope se řeší stejnou cestou.** Skončí-li přihláška člena v `Canceled` nebo `Expired`, osoba z hlídky vypadne a kontrola se spustí znovu — zbylá hlídka buď dál vyhovuje (Šerpa s dětmi ze 4 na 3), nebo se rozpustí (Stezka ze 3 na 2). Sama ztráta člena tedy hlídku neruší, ruší ji až porušené pravidlo.
- **Odejde-li kapitán**, stane se jím automaticky nejdéle přítomný ze zbylých členů (`RACE_PATROL_MEMBER.joined_at`) — teprve pak se ověřuje složení, aby hlídka nepadla jen kvůli dočasně chybějícímu kapitánovi.

## Připomínka

- Job N dní před akcí upozorní **vedoucí přiřazené k akci** (`EVENT_ASSIGNMENT`) na závodníky bez hlídky; přeskočí ty, kdo už dnes připomínku dostali ([notifications.md](notifications.md) → `EMAIL_PATROL_REMINDER`).

## Logování

- Každá mutace hlídky (založení, vstup, odchod, úprava, smazání) se zapisuje do `AUDIT_LOG` — viz [audit-log.md](audit-log.md). Aktérem je **účet vlastníka přihlášky**, který hlídku spravuje, při správě přes token jeho e-mail; u rozpuštění kontrolou konzistence je aktérem systém.

## Stanoviště a rozhodčí

Stanoviště jsou modelována jako výběrový číselník akce (viz [event-fields.md](event-fields.md)):

- `EVENT_FIELD` s `assigned_by = self` a `max_select = 1`; jednotlivá stanoviště jsou jeho `EVENT_FIELD_OPTION`, přiřazení rozhodčího je `REGISTRATION_FIELD_VALUE`.
- **Volba je samoobslužná:** dospělý nezávodící účastník si stanoviště zvolí sám v rámci vlastní přihlášky, stejně jako u kteréhokoli jiného číselníku s `assigned_by = self`. Vedoucí s `can_edit_registrations` i vlastník přihlášky ve svém self-managementu můžou volbu dodatečně upravit — stejným oprávněním, jakým se edituje zbytek přihlášky, žádný zvláštní mechanismus navíc.
- **Způsobilost** (`condition`): dospělá osoba (≥ 16), která není závodník ani šerpa a není v žádné hlídce.
- **Kapacita**: běžné stanoviště `capacity = 1`, pseudo-stanoviště „Jakékoliv" `capacity = NULL`.
- Přiřazení je **upsert** (nejvýše jedno na osobu a akci); stanoviště s `capacity = 1` nelze obsadit, je-li už zabrané.
- Přiřazení ke stanovišti je vzájemně výlučné s členstvím v hlídce.
