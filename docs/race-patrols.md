# Hlídky na závodních akcích — model

Implementační detail k [README.md](../README.md) → **Hlídky na závodních akcích (Stezka)**, kde jsou popsané kategorie a pravidla složení. Schéma viz [data-model.md](data-model.md).

## Club scope

- Hlídky se skládají z osob **vlastní přihlášky a jejích dílčích přihlášek** (podregistrací), jejichž stav není `Canceled`, `Expired` ani `New`. Tato množina se dál označuje jako **club scope** a platí i pro přiřazování rozhodčích.

## Vlastnictví a členství

- Hlídku vlastní přihláška, která ji založila (`RACE_PATROL.owner_registration_id`); upravovat a mazat ji smí jen vlastník. Název je unikátní v rámci akce.
- Každá osoba je nejvýše v jedné hlídce (`RACE_PATROL_MEMBER`).
- Kapitána (`role = leader`) lze zvolit jen v kategoriích Stezka a Pěšinka, kde je povinný; automaticky se jím stane první přidaný člen a lze ho později změnit. V ostatních kategoriích jsou všichni členové `member`.

## Výpočet věku

- Referenční datum řídí konfigurační volba akce `age_at_year_end`:
  - zapnuto (výchozí) — věk ke **konci aktuálního roku**: `věk = rok(31. 12. letošního roku) − rok(datum narození)`,
  - vypnuto — věk **k datu konání akce**.
- Rozdíl se počítá v letech (date diff). Chybí-li datum narození, člena nelze plně ověřit a kontrola konzistence to hlásí.

## Kontrola konzistence

- Pravidla složení (počty členů, způsobilost, věkové limity podle kategorie) ověřuje **jediná čistá funkce**.
- Poruší-li hlídka pravidla po změně relevantního údaje člena (věk, příznak závodníka, kategorie), **hlídka se rozpustí** — všichni členové se odpojí, hlídka se smaže a vlastník dostane informaci s důvodem.

## Připomínka

- Job N dní před akcí upozorní vedoucí na závodníky bez hlídky; přeskočí ty, kdo už dnes připomínku dostali.

## Logování

- Každá mutace hlídky (založení, vstup, odchod, úprava, smazání) se zapisuje do `AUDIT_LOG` — viz [audit-log.md](audit-log.md). Aktérem je účet vedoucího, při správě přes token jeho e-mail.

## Stanoviště a rozhodčí

Stanoviště jsou modelována jako výběrový číselník akce (viz [event-fields.md](event-fields.md)):

- `EVENT_FIELD` s `assigned_by = leader` a `max_select = 1`; jednotlivá stanoviště jsou jeho `EVENT_FIELD_OPTION`, přiřazení rozhodčího je `REGISTRATION_FIELD_VALUE`.
- **Způsobilost** (`condition`): osoba z club scope, dospělá (≥ 16), která není závodník ani šerpa a není v žádné hlídce.
- **Kapacita**: běžné stanoviště `capacity = 1`, pseudo-stanoviště „Jakékoliv" `capacity = NULL`.
- Přiřazení je **upsert** (nejvýše jedno na osobu a akci); stanoviště s `capacity = 1` nelze obsadit, je-li už zabrané.
- Přiřazení ke stanovišti je vzájemně výlučné s členstvím v hlídce.
