# Synchronizace bankovních transakcí (Fio)

Implementační detail k modulu párování plateb ([README.md](../README.md) → **Modul párování plateb**). Popisuje, jak se transakce dostanou z banky do `BANK_TRANSACTION`; samotné párování na přihlášky zůstává v README.

Synchronizace je **volitelná vrstva**. Oddíl bez bankovního API plní `BANK_TRANSACTION` nahráním výpisu nebo ručním zápisem — viz [payment-matching.md](payment-matching.md) → **Oddíl bez bankovního API**.

## Rozsah

- Synchronizuje se **samostatně za každý bankovní účet** (`BANK_ACCOUNT`) s uloženým tokenem v oddílu, který má modul aktivní. Token patří účtu, ne oddílu — oddíl s více účty se synchronizuje nezávisle.
- Zpracovávají se **příchozí platby** (kladná částka); odchozí pohyby se ukládají také, ale do párování nevstupují.
- Z variabilního čísla lze automaticky odstraňovat počáteční nuly (přidané bankou), které by mohly bránit v úspěšném spárování.
- Po importu se nad nově staženými transakcemi rovnou spustí automatické párování.

## Token

- Používá se **read-only token** Fio (nesmí umožňovat zadávat platby).
- Ukládá se šifrovaný (`api_token_enc`, stejný mechanismus jako u `smtp_password_enc`); nikde se nezobrazuje ani nevypisuje do logů a chybových hlášek.
- **Přítomnost tokenu rozhoduje o zapnutí synchronizace** — účet bez tokenu se nesynchronizuje. Účetní/HVO ho může přepsat nebo odebrat; žádný další příznak zapnuto/vypnuto neexistuje, `sync_state` popisuje jen výsledek posledního běhu.

## Přírůstkové stahování

- Job používá Fio endpoint „od posledního stažení" a po úspěšném zpracování posune kurzor na poslední zpracovaný pohyb (`last_external_id`).
- Při prvním běhu (nebo po ztrátě kurzoru) se stáhne období od aktivace modulu, resp. posledních N dnů.
- **Interval** se nastavuje u bankovního účtu (`sync_interval_minutes`, výchozí 60 min); účetní může vyvolat stažení i ručně.
- Fio omezuje volání tokenem (řádově jedno za 30 s), proto joby běží na účet sériově s odstupem a při odmítnutí se opakují s prodlevou.

## Idempotence

- Identifikátor pohybu z Fio se ukládá jako `BANK_TRANSACTION.external_id` a je **unikátní v rámci bankovního účtu**.
- Opakované stažení stejného pohybu se zahodí, takže výpadek ani ruční doběh nezpůsobí duplicitní platby ani duplicitní alokace.

## Chybové stavy

- Neplatný token, nedostupné API nebo překročený limit se zaznamenají do `sync_state` / `sync_error`; synchronizace se u daného účtu nezastaví natrvalo.
- Po opakovaném selhání systém upozorní účetní a HVO.
- Poslední úspěšné stažení drží `last_sync_at`.

## Doplňková pole `BANK_ACCOUNT`

Nad rámec polí uvedených v [datovém modelu](data-model.md) si integrace drží:

| Pole                    | Význam                                             |
| ----------------------- | -------------------------------------------------- |
| `provider`              | poskytovatel — `fio`, nebo `manual` u účtu bez API |
| `sync_interval_minutes` | perioda stahování                                  |
| `last_external_id`      | kurzor — poslední stažený pohyb                    |
| `sync_error`            | text poslední chyby                                |

## Mapování polí Fio → `BANK_TRANSACTION`

| Fio                   | Pole                                  |
| --------------------- | ------------------------------------- |
| ID pohybu             | `external_id`                         |
| Datum                 | `date`                                |
| Objem                 | `amount`                              |
| Protiúčet / kód banky | `sender_account` / `sender_bank_code` |
| Název protiúčtu       | `sender_name`                         |
| VS / SS               | `vs` / `ss`                           |
| Zpráva pro příjemce   | `message`                             |
| Typ pohybu            | `transaction_type`                    |
| čas zpracování jobem  | `imported_at`                         |
