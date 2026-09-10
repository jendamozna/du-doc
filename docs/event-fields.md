# Výběrové číselníky akce — model

Implementační detail k [README.md](../README.md) → **Výběrové číselníky akce**. Schéma viz [data-model.md](data-model.md).

Obecný, znovupoužitelný mechanismus: vedoucí u libovolné akce nadefinuje libovolný počet číselníků (`EVENT_FIELD`), z nichž si účastník při přihlášení vybírá předdefinované hodnoty (`EVENT_FIELD_OPTION`). Stejným modelem se pokryje ubytování, strava, doprava, trika, role i stanoviště na závodě.

## `EVENT_FIELD`

| Pole             | Význam                                                                                                                                                         |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`           | název číselníku                                                                                                                                                |
| `comment`        | veřejný popis / instrukce pro účastníka                                                                                                                        |
| `internal_note`  | neveřejná poznámka jen pro vedoucí                                                                                                                             |
| `assigned_by`    | `self` (vybírá účastník při přihlášení) / `leader` (přiřazuje vedoucí až po přihlášení)                                                                        |
| `max_select`     | počet voleb — `1` = jednovýběrový, `> 1` = vícevýběrový s limitem, `NULL` = vícevýběrový bez limitu                                                            |
| `required_phase` | `NULL` = nepovinný; `on_submit` = volba nutná už při odeslání přihlášky (výchozí — výzva k platbě odchází ihned po podání, takže není žádná pozdější chvíle „před výzvou k platbě"), `before_event` = kdykoli před konáním akce |
| `condition`      | podmínka způsobilosti (`NULL` = všichni) — číselník se zobrazí jen účastníkům, kteří ji splňují (věk, členství DU, role); ostatním se skryje                   |

- U **náhradníka** se povinný výběr (stejně jako dokumenty) vynucuje až po přijetí nabídky z náhradnického místa.
- Číselník, kde smí položku zvolit jen jeden účastník (např. konkrétní lůžko), se modeluje nastavením `capacity = 1` na každé jeho položce — samostatné pole pro tento režim číselník nemá, řídí ho výhradně `EVENT_FIELD_OPTION.capacity`.

## `EVENT_FIELD_OPTION`

| Pole             | Význam                                                                                                  |
| ---------------- | ------------------------------------------------------------------------------------------------------- |
| `value`          | nabízená hodnota                                                                                        |
| `capacity`       | max počet účastníků na položku — `1` = unikátní (lůžko, stanoviště), `> 1` = limit, `NULL` = bez limitu |
| `price_modifier` | příplatek k základní ceně; může být `0` i záporný                                                       |

- Po naplnění kapacity se položka přestane nabízet.
- **Celková cena přihlášky = základní cena zafixovaná při podání (`REGISTRATION.base_price`) + součet zafixovaných příplatků zvolených položek.** Příplatek se stejně jako `base_price` **zafixuje při volbě** do `REGISTRATION_FIELD_VALUE.price_modifier` — pozdější úprava `EVENT_FIELD_OPTION.price_modifier` (oprava ceníku vedoucím) už zvolenou položku nepřeceňuje, platí jen pro volby učiněné od té chvíle.
- Cena se přepočte jen tehdy, **změní-li účastník volbu** (jiné ubytování, strava) — mění se objednaná služba, ne ceník; nová volba dostane novou zafixovanou hodnotu k okamžiku výběru.

## Výběr účastníka

- Volba je vazba přihláška ↔ položka (`REGISTRATION_FIELD_VALUE`) se zafixovaným příplatkem; u vícevýběrového číselníku vznikne více vazeb.

## Příklady

- **Ubytování** — jednovýběrový číselník `budova / stan`, kde „budova" nese vyšší `price_modifier`.
- **Strava** — vícevýběrový číselník `snídaně / oběd / večeře`, každá položka s vlastní cenou.
- **Stanoviště na závodě** — `assigned_by = self` (dospělý nezávodící účastník si stanoviště volí sám při přihlášení; vedoucí s `can_edit_registrations` i vlastník přihlášky ho pak můžou v editaci přihlášky změnit), `max_select = 1`, běžné stanoviště `capacity = 1`, pseudo-stanoviště „Jakékoliv" `capacity = NULL`; viz [race-patrols.md](race-patrols.md).
