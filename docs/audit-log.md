# Auditní log — model

Implementační detail k [README.md](../README.md) → **Retence a GDPR**, kde je uvedená retenční lhůta a rozsah logovaných událostí. Schéma viz [data-model.md](data-model.md).

## Struktura záznamu

Změnové události napříč systémem se zapisují do jediné tabulky `AUDIT_LOG`:

| Pole                              | Význam                                                                              |
| --------------------------------- | ----------------------------------------------------------------------------------- |
| `entity_type` + `entity_id`       | cíl operace; odkaz je **polymorfní**, tedy bez cizího klíče                         |
| `action`                          | `create` / `update` / `delete` / `join` / `leave` / `approve` / `reject` / `cancel` |
| `unit_id`                         | oddíl, kvůli izolaci a mazání per oddíl                                             |
| `actor_account_id`, `actor_email` | aktér; `NULL` u účtu = systémový job nebo aktér bez účtu (správa přes token)        |
| `detail`                          | JSON s tím, co se změnilo, případně důvodem                                         |
| `created_at`                      | čas                                                                                 |

- Doporučené indexy: `(entity_type, entity_id)` pro historii jednoho záznamu a `(unit_id, created_at)` pro výpis a retenční job.
- Polymorfní odkaz je vědomý kompromis — cenou za jednu tabulku je chybějící referenční integrita na cíl.

## Ruční evidence plateb

Zápis platby bez bankovního API (`BANK_TRANSACTION` se `source != 'import'`, viz [payment-matching.md](payment-matching.md)) se loguje vždy — chybí bankovní protistrana, takže auditní log je jediné krytí toho, kdo prohlásil přihlášku za zaplacenou.

| `entity_type` / `action`      | Kdy                                     | `detail`                                   |
| ----------------------------- | --------------------------------------- | ------------------------------------------ |
| `BANK_TRANSACTION` / `create` | ruční zápis nebo řádek nahraného výpisu | `source`, částka, datum, VS/SS, odesílatel |
| `BANK_TRANSACTION` / `cancel` | storno ručního zápisu (`voided_at`)     | důvod                                      |

Tím se oddělí od `import`, který se neloguje — automatické stažení není lidské rozhodnutí a jeho stopu drží `last_sync_at` a `external_id`.

## Evidenční oddíl členství DU

`DU_MEMBERSHIP` sám historii nenese, ale evidenční oddíl rozhoduje o výkazu členské základny (README → **Člen DU**), takže se jeho převod musí dát dohledat.

| `entity_type` / `action`   | Kdy                       | `detail`                                      |
| -------------------------- | ------------------------- | --------------------------------------------- |
| `DU_MEMBERSHIP` / `update` | převod evidenčního oddílu | původní a nový `unit_id`, kdo potvrdil, důvod |

Záznam se zapisuje **oběma oddílům** (`unit_id` původní i nový) — jinak by po smazání logu jedné strany zmizela polovina stopy.

## Co se neloguje sem

Tři evidence zůstávají oddělené, protože nejsou jen auditem:

- `MERGE_LOG` — nese `snapshot` pro revert sloučení osob,
- `PERSON_UNIT_HISTORY` — typované přechody stavů, které čtou reporty,
- `GDPR_AUDIT` — doklad o výmazu s vlastní retencí a okruhem čtenářů.
