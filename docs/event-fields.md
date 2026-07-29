# Výběrové číselníky akce — model

Implementační detail k [README.md](../README.md) → **Výběrové číselníky akce**. Schéma viz [data-model.md](data-model.md).

Obecný, znovupoužitelný mechanismus: vedoucí u libovolné akce nadefinuje libovolný počet číselníků (`EVENT_FIELD`), z nichž si účastník při přihlášení vybírá předdefinované hodnoty (`EVENT_FIELD_OPTION`). Stejným modelem se pokryje ubytování, strava, doprava, trika, role i stanoviště na závodě.

## `EVENT_FIELD`

| Pole             | Význam                                                                                                                                                         |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`           | název číselníku                                                                                                                                                |
| `comment`        | veřejný popis / instrukce pro účastníka                                                                                                                        |
| `internal_note`  | neveřejná poznámka jen pro vedoucí                                                                                                                             |
| `selection_mode` | `exclusive` (položku zvolí nejvýše jeden účastník, po výběru se ostatním přestane nabízet) / `shared` (více účastníků, omezuje kapacita položky)               |
| `assigned_by`    | `self` (vybírá účastník při přihlášení) / `leader` (přiřazuje vedoucí až po přihlášení)                                                                        |
| `max_select`     | počet voleb — `1` = jednovýběrový, `> 1` nebo `NULL` = vícevýběrový bez limitu                                                                                 |
| `required_phase` | `NULL` = nepovinný; `on_submit` = volba nutná už při odeslání přihlášky (výchozí), `before_payment` = před výzvou k platbě, `before_event` = před konáním akce |
| `condition`      | podmínka způsobilosti (`NULL` = všichni) — číselník se zobrazí jen účastníkům, kteří ji splňují (věk, členství DU, role); ostatním se skryje                   |

- U **náhradníka** se povinný výběr (stejně jako dokumenty) vynucuje až po schválení přihlášky.
- `selection_mode = exclusive` odpovídá kapacitě `1` u všech položek.

## `EVENT_FIELD_OPTION`

| Pole             | Význam                                                                                                  |
| ---------------- | ------------------------------------------------------------------------------------------------------- |
| `value`          | nabízená hodnota                                                                                        |
| `capacity`       | max počet účastníků na položku — `1` = unikátní (lůžko, stanoviště), `> 1` = limit, `NULL` = bez limitu |
| `price_modifier` | příplatek k základní ceně; může být `0` i záporný                                                       |

- Po naplnění kapacity se položka přestane nabízet.
- **Celková cena přihlášky = základní cena podle typu účastníka (`EVENT_PRICE`) + součet příplatků zvolených položek.**

## Výběr účastníka

- Volba je vazba přihláška ↔ položka (`REGISTRATION_FIELD_VALUE`); u vícevýběrového číselníku vznikne více vazeb.

## Příklady

- **Ubytování** — jednovýběrový číselník `budova / stan`, kde „budova" nese vyšší `price_modifier`.
- **Strava** — vícevýběrový číselník `snídaně / oběd / večeře`, každá položka s vlastní cenou.
- **Stanoviště na závodě** — `assigned_by = leader`, `max_select = 1`, běžné stanoviště `capacity = 1`, pseudo-stanoviště „Jakékoliv" `capacity = NULL`; viz [race-patrols.md](race-patrols.md).
