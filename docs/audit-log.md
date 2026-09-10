# Auditní log — model

Implementační detail k [README.md](../README.md) → **Retence a GDPR**, kde je uvedená retenční lhůta a rozsah logovaných událostí. Schéma viz [data-model.md](data-model.md).

## Struktura záznamu

Změnové události napříč systémem se zapisují do jediné tabulky `AUDIT_LOG`:

| Pole                        | Význam                                                                              |
| --------------------------- | ----------------------------------------------------------------------------------- |
| `entity_type` + `entity_id` | cíl operace; odkaz je **polymorfní**, tedy bez cizího klíče                         |
| `action`                    | `create` / `update` / `delete` / `join` / `leave` / `approve` / `reject` / `cancel` |
| `unit_id`                   | oddíl, kvůli izolaci a mazání per oddíl                                             |
| `actor_type`                | zdroj aktéra: `account` / `token` / `system`                                        |
| `actor_account_id`          | povinné právě při `actor_type = account`; jinak `NULL`                              |
| `actor_email`               | volitelný e-mail při `actor_type = token`; jinak `NULL`                             |
| `detail`                    | JSON s tím, co se změnilo, případně důvodem                                         |
| `created_at`                | čas                                                                                 |

- Doporučené indexy: `(entity_type, entity_id)` pro historii jednoho záznamu a `(unit_id, created_at)` pro výpis a retenční job.
- Polymorfní odkaz je vědomý kompromis — cenou za jednu tabulku je chybějící referenční integrita na cíl.
- Kombinace aktéra je závazná: `account` má vyplněné jen `actor_account_id`, `token` nemá účet a může nést `actor_email`, `system` nemá vyplněný ani účet, ani e-mail. Token bez známého e-mailu se proto vždy odliší od systémového jobu hodnotou `actor_type`.

## Struktura `detail`

`detail` je vždy JSON **objekt**, klíče `snake_case`. Pevná sada klíčů napříč celým logem, ať zápisy z různých modulů čte stejný kód:

| Klíč              | Kdy                                 | Obsah                                                                                 |
| ----------------- | ----------------------------------- | ------------------------------------------------------------------------------------- |
| `domain_event_id` | zápis vznikl z doménové události    | id události ([modules.md](modules.md)) — idempotence a dohledatelnost k jejímu zdroji |
| `changes`         | `action = update`                   | mapa `pole → {"from": …, "to": …}`, jen skutečně změněná pole                         |
| `snapshot`        | `create` / `delete`                 | klíčová pole vzniklého či zaniklého záznamu, ne celý objekt                           |
| `reason`          | u operací, kde to vyžaduje pravidlo | důvod zadaný člověkem                                                                 |
| `refs`            | volitelné                           | id souvisejících entit (`event_id`, `registration_id`, `person_id`, …)                |
| `pii`             | jen když je to nutné (viz níže)     | osobní údaje                                                                          |

`refs.event_id` je vždy cizí klíč na `EVENT` (akci) — `domain_event_id` používá jiné jméno záměrně, aby se nepletl s identifikátorem doménové události.

**Pravidla:**

- **Nikdy žádná citlivá data** (`PERSON_SENSITIVE_DATA`). Retence citlivých dat je do 30 dnů po akci, auditní log 3 roky — kopie hodnoty v `detail` by purge přežila. Loguje se jen fakt změny: `{"changes": {"content": {"redacted": true}}}`.
- **Nikdy obsah souborů či dokumentů** — jen `refs` na id, případně `review_note`.
- **Osobní údaje jen pod `detail.pii`.** Některé zápisy je nutně nesou (změna `contact_email`, odesílatel u ručního zápisu platby). Globální anonymizace osoby musí vyprázdnit `contact_email`/`guardian_email` i na starých přihláškách (README → **Retence a GDPR**) — když osobní hodnoty žijí jen pod `detail.pii`, má anonymizační job jedno místo k vyprázdnění; zbytek `detail` (struktura, id, časy) zůstane jako důkaz, že a kdo operaci provedl.
- **`detail` je důkaz, ne dotazovací plocha.** Filtruje a agreguje se přes sloupce (`entity_type`, `action`, `unit_id`, `created_at`), ne uvnitř JSONu — report Platby čte čas storna z `action = 'cancel'`, ne z obsahu `detail`.
- **Strop velikosti** (návrh 8 kB) — v souladu s principem `modules.md` „identifikátory dotčených entit, ne celé objekty".

## Ruční evidence plateb

Zápis platby bez bankovního API (`BANK_TRANSACTION` se `source != 'import'`, viz [payment-matching.md](payment-matching.md)) se loguje vždy — chybí bankovní protistrana, takže auditní log je jediné krytí toho, kdo prohlásil přihlášku za zaplacenou.

| `entity_type` / `action`      | Kdy                                     | `detail`                                                         |
| ----------------------------- | --------------------------------------- | ---------------------------------------------------------------- |
| `BANK_TRANSACTION` / `create` | ruční zápis nebo řádek nahraného výpisu | `snapshot: {source, amount, date, vs, ss}`, `pii: {sender_name}` |
| `BANK_TRANSACTION` / `cancel` | storno ručního zápisu (`voided_at`)     | `reason`                                                         |
| `REFUND_REQUEST` / `create`   | účetní rozhodla o vratce                | `snapshot: {amount, reason}`, `refs: {registration_id}`          |
| `REFUND_REQUEST` / `cancel`   | vratka zrušena před provedením          | `reason`                                                         |

Tím se oddělí od `import`, který se neloguje — automatické stažení není lidské rozhodnutí a jeho stopu drží `last_sync_at` a `external_id`.

## Znovuposlání tokenového odkazu

Znovuposlání je **vydání nového přístupu k osobním údajům** — starý token se zneplatní a vznikne nový ([non-functional.md](non-functional.md) → **Znovuposlání ztraceného odkazu**). Loguje se proto vždy, oběma cestami.

| `entity_type` / `action`   | Kdy                                       | `detail`                                                                    |
| -------------------------- | ----------------------------------------- | --------------------------------------------------------------------------- |
| `REGISTRATION` / `update`  | znovuposlání odkazu na správu či schválení | `changes: {token: {"from": "redacted", "to": "redacted"}}`, `refs: {event_id}` |

Aktér rozlišuje obě cesty: `actor_type = 'token'` s `actor_email` u samoobslužné žádosti, `account` u vedoucího, který odkaz poslal z detailu přihlášky. **Hodnota tokenu se do `detail` nikdy nezapisuje** — je to přístupové pověření, stejně jako `api_token_enc`.

## Bankovní účet a synchronizace

Přepnutí `BANK_ACCOUNT.provider` mění tok peněz a maže uložené pověření, proto se loguje vždy. Samotný token se do `detail` nikdy nedostane.

| `entity_type` / `action`  | Kdy              | `detail`                          |
| ------------------------- | ---------------- | --------------------------------- |
| `BANK_ACCOUNT` / `update` | změna `provider` | `changes: {provider: {from, to}}` |

Vložení nebo rotace `api_token_enc` beze změny `provider` se nezapisuje — token sám je citlivé pověření, ne obsah rozhodnutí, a jeho platnost/neplatnost je vidět v `sync_state`, ne v auditu.

## Evidenční oddíl členství DU

`DU_MEMBERSHIP` sám historii nenese, ale evidenční oddíl rozhoduje o výkazu členské základny (README → **Člen DU**), takže se jeho převod musí dát dohledat.

| `entity_type` / `action`   | Kdy                       | `detail`                                                                      |
| -------------------------- | ------------------------- | ----------------------------------------------------------------------------- |
| `DU_MEMBERSHIP` / `update` | převod evidenčního oddílu | `changes: {unit_id: {from, to}}`, `refs: {confirmed_by_account_id}`, `reason` |

Záznam se zapisuje **oběma oddílům** (`unit_id` původní i nový) — jinak by po smazání logu jedné strany zmizela polovina stopy.

## Co se neloguje sem

Čtyři evidence zůstávají oddělené, protože nejsou jen auditem:

- `MERGE_LOG` — nese `snapshot` pro revert sloučení osob,
- `PERSON_UNIT_HISTORY` — typované přechody stavů, které čtou reporty,
- `EVENT_ASSIGNMENT` s `revoked_at` — historie přístupu vedoucích k akci; je to primární evidence s vlastní retencí 10 let od skončení akce, kterou by 3letý auditní log neunesl,
- `GDPR_AUDIT` — doklad o lokálním výmazu citlivých dat nebo globální anonymizaci osoby, s vlastní retencí a okruhem čtenářů.
