# Synchronizace bankovních transakcí (Fio)

Implementační detail k modulu párování plateb ([README.md](../README.md) → **Modul párování plateb**). Popisuje, jak se transakce dostanou z banky do `BANK_TRANSACTION`; samotné párování na přihlášky zůstává v README.

Synchronizace je **volitelná vrstva**. Oddíl bez bankovního API plní `BANK_TRANSACTION` nahráním výpisu nebo ručním zápisem — viz [payment-matching.md](payment-matching.md) → **Oddíl bez bankovního API**.

## Rozsah

- Synchronizuje se **samostatně za každý bankovní účet** (`BANK_ACCOUNT`) s uloženým tokenem v oddílu, který má modul aktivní. Token patří účtu, ne oddílu — oddíl s více účty se synchronizuje nezávisle.
- Zpracovávají se **příchozí i odchozí pohyby** — obojí se ukládá do `BANK_TRANSACTION`. Příchozí platby (kladná částka) vstupují do běžného párování (SS/VS/jméno); odchozí pohyby (záporná částka) do něj nevstupují, ale automaticky se párují s čekajícím `REFUND_REQUEST` jako vratka (`match_method = 'refund'`, viz [payment-matching.md](payment-matching.md) → **Přeplatek a vratka**).
- `vs` a `ss` se před uložením do `BANK_TRANSACTION` **vždy zbaví počátečních nul** (banka je někdy doplňuje na pevnou délku) — bez normalizace by shoda s `REGISTRATION.vs` (uloženým bez nul) nenaskočila. Pravidlo platí **při zápisu do `BANK_TRANSACTION` bez ohledu na zdroj** ([payment-matching.md](payment-matching.md) → **Oddíl bez bankovního API**), ne jen u Fio API — jinak by stejná platba stažená přes API a nahraná výpisem z jiné banky dostala různý `external_id` a mohla se spárovat jinak.
- Po importu se rovnou spustí automatické párování — jen nad **nově uloženými** transakcemi, ne nad těmi, které přinesl překryv stahovaného okna a už v evidenci byly.

## Token

- Používá se **read-only token** Fio (nesmí umožňovat zadávat platby).
- Ukládá se šifrovaný (`api_token_enc`, stejný mechanismus jako u `smtp_password_enc`); nikde se nezobrazuje ani nevypisuje do logů a chybových hlášek.
- **O zapnutí synchronizace rozhoduje `provider`, token je jeho pověření.** Invariant platí obou směrech: `provider = 'fio'` vyžaduje vyplněný token, `provider = 'manual'` token i synchronizační pole vylučuje ([validation.md](validation.md) → **Platby**). „Odebrat token" proto není samostatný úkon — **vypnout synchronizaci znamená přepnout `provider` na `manual`**, což token smaže v téže operaci; zapnout znamená přepnout na `fio` a zároveň token zadat. Jediný přepínač, atomicky. Bez toho by šel sestavit stav `provider = 'fio'` bez tokenu, kdy se nic nestahuje, ale upomínky plateb běží dál ([notifications.md](notifications.md) → `EMAIL_PAYMENT_REMINDER`) — systém by urgoval platby, o kterých se nikdy nedozví, že dorazily.
- **`provider` je režim, `sync_state` je zdraví** — dvě různé otázky. Vypršelý či odvolaný token na straně banky je `sync_state = 'error'`, **ne** přepnutí na `manual`: účet zůstává `fio`, joby dál zkoušejí, odejde alert a upomínky běží dál, protože jde o opravitelný výpadek, ne o změnu režimu. Přepsání tokenu (rotace) je běžná změna uvnitř `provider = 'fio'`.
- Změnu `provider` provádí účetní nebo HVO. Mění tok peněz a maže pověření, proto se **zapisuje do `AUDIT_LOG`** ([audit-log.md](audit-log.md) → **Bankovní účet a synchronizace**). Přepnutí `fio → manual` se **neblokuje** ani u účtu s nespárovanými transakcemi — párování na režimu nezávisí ([payment-matching.md](payment-matching.md) → **Oddíl bez bankovního API**) a účetní může mít pádný důvod, třeba zrušení API na straně banky. Stažené transakce zůstávají; jejich původ drží `BANK_TRANSACTION.source`, takže po vyprázdnění `last_sync_at` se historie neztrácí.

## Přírůstkové stahování

- Job stahuje **rozsah dat, ne kurzor**: `date_from = den(last_sync_at) − 7 dní`, `date_to = dnes`. Sedmidenní překryv kryje pohyby, které banka zaúčtuje se starším datem, než je den stažení (víkendová dávka, datum valuty) — bez něj by takový pohyb propadl natrvalo.
- Překryv nic nestojí: už známé pohyby zahodí unikát `účet + external_id` (viz **Idempotence**). Stahování je tím **bezstavové** — spadne-li job uprostřed zpracování, další běh stáhne totéž znovu a žádná dávka se neztratí. Endpoint s kurzorem drženým na straně banky se proto nepoužívá: ten posouvá pozici už při stažení, tedy dřív, než je zpracování potvrzené.
- `last_sync_at` se posouvá **jen po úspěšně dokončeném běhu**, i když nepřinesl žádný pohyb. Při výpadku tak okno samo zůstává otevřené přesně po dobu výpadku a nic se nepřeskočí.
- **První běh** (`last_sync_at IS NULL`): `date_from` = den vložení tokenu, nejvýše však 90 dní zpět. U dlouho existujícího účtu se tím nestahuje celá historie, která stejně nemá co párovat.
- **Interval** se nastavuje u bankovního účtu (`sync_interval_minutes`, výchozí 60 min).
- **Ruční stažení není druhá cesta** — účetní nebo HVO jím jen spouští **tentýž job okamžitě**. Stejné okno, stejné zpracování, stejné posunutí `last_sync_at`; žádný režim „stáhni všechno od začátku" neexistuje, od toho je změna nastavení, ne tlačítko. Jedna cesta kódem znamená, že ruční doběh po výpadku se chová prokazatelně stejně jako plánovaný běh.

## Souběh a rate limit

Fio omezuje volání tokenem (řádově jedno za 30 s). Plánovaný job i ruční spuštění o tentýž účet soupeří, proto:

- **Zámek se drží na bankovní účet** a znamená **„právo volat Fio za tento účet"**, ne jen „právě běží job". Držitel ho **neuvolní dřív než 30 s po posledním volání API**, i když zpracování skončilo dávno předtím. Rate limit je tím vynucený samotným zámkem a nepotřebuje vlastní evidenci času posledního volání; sériovost běhů na účet je jeho důsledek, ne samostatné pravidlo.
- **Souběh se neřeší frontou, ale připojením.** Běží-li stahování a účetní zmáčkne tlačítko, druhý požadavek nevznikne — rozhraní se napojí na probíhající běh a ukáže jeho výsledek. Fronta ručních požadavků by z pěti kliknutí udělala pět volání s třicetisekundovými pauzami, tedy dvě a půl minuty nereagujícího tlačítka. Držení zámku je zároveň celá ochrana proti spamu.
- **Držení zámku je vidět v rozhraní** jako stav „probíhá synchronizace" — jinak účetní nechápe, proč tlačítko nic nedělá.
- **Zámek vyprší sám** (TTL řádově 15 min, bezpečně nad nejdelší reálnou dobu běhu). Bez TTL by pád procesu mezi zabráním a uvolněním zastavil synchronizaci účtu natrvalo, a navíc tiše: alert by naskočil až po třech intervalech a ukazoval by na nedostupné API místo na uvázlý zámek.
- Odmítne-li Fio volání i tak, běh se opakuje s prodlevou.

## Idempotence

- Identifikátor pohybu z Fio se ukládá jako `BANK_TRANSACTION.external_id` a je **unikátní v rámci bankovního účtu**.
- Opakované stažení stejného pohybu se zahodí, takže překryv okna, výpadek ani ruční doběh nezpůsobí duplicitní platby ani duplicitní alokace. Na tomto unikátu stojí celý bezstavový model stahování.

## Chybové stavy

- Neplatný token, nedostupné API nebo překročený limit se zaznamenají do `sync_state` / `sync_error`; synchronizace se u daného účtu nezastaví natrvalo.
- Práh alertu je **žádný úspěšný běh po dobu tří intervalů**: `now − COALESCE(last_sync_at, token_set_at) > 3 × sync_interval_minutes`. Protože `last_sync_at` se posouvá jen po úspěšně dokončeném běhu, je to totéž jako „tři po sobě jdoucí selhání" — jen bez počitadla, které by bylo dalším stavem k udržování. Úspěšný běh, i vyvolaný ručně, práh vynuluje sám tím, že `last_sync_at` posune. `token_set_at` kryje účet, který od vložení tokenu ještě nikdy neuspěl.
- Po dosažení prahu systém vytvoří událost `bank_account.sync_failed` a upozorní účetní a HVO šablonou `EMAIL_FIO_SYNC_FAILURE`. Událost obsahuje `bank_account_id`, název účtu, `last_sync_at`, čas posledního neúspěšného běhu a bezpečně zkrácenou poslední chybu; **nikdy neobsahuje token**.
- Během trvání problému se upozornění odešle nejvýše **jednou za 24 hodin**. Podmínku vyhodnotí evidence odeslaných e-mailů ([non-functional.md](non-functional.md) → **Odchozí e-maily**) — poslední `EMAIL_FIO_SYNC_FAILURE` pro daný účet; žádné pole na `BANK_ACCOUNT` k tomu netřeba. Po úspěšném běhu a novém dosažení prahu vznikne nový alert.
- Běhy synchronizace se **nezapisují do `AUDIT_LOG`** — automatické stažení není lidské rozhodnutí ([audit-log.md](audit-log.md) → **Ruční evidence plateb**). Jeho stopu drží `last_sync_at`, `sync_state`, `sync_error` a `external_id` stažených pohybů.

## Doplňková pole `BANK_ACCOUNT`

Nad rámec polí uvedených v [datovém modelu](data-model.md) si integrace drží:

| Pole                    | Význam                                        |
| ----------------------- | --------------------------------------------- |
| `sync_interval_minutes` | perioda stahování                             |
| `token_set_at`          | den vložení tokenu — `date_from` prvního běhu |
| `sync_error`            | text poslední chyby                           |

`provider`, `api_token_enc`, `last_sync_at` a `sync_state` jsou naopak součástí [datového modelu](data-model.md) — jejich význam se definuje tam, ne zde.

## Mapování polí Fio → `BANK_TRANSACTION`

| Fio                   | Pole                                  | Poznámka                                     |
| --------------------- | ------------------------------------- | -------------------------------------------- |
| ID pohybu             | `external_id`                         |                                              |
| Datum                 | `date`                                |                                              |
| Objem                 | `amount`                              | kladné = příchozí, záporné = odchozí         |
| Protiúčet / kód banky | `sender_account` / `sender_bank_code` | protistrana bez ohledu na směr pohybu        |
| Název protiúčtu       | `sender_name`                         | totéž — u vratky jde o příjemce              |
| VS / SS               | `vs` / `ss`                           | ukládá se bez počátečních nul (viz **Rozsah**) |
| Zpráva pro příjemce   | `message`                             |                                              |
| Typ pohybu            | `transaction_type`                    |                                              |
| —                     | `bank_account_id`                     | účet, za který job běží                      |
| —                     | `source`                              | konstanta `import`                           |
| —                     | `imported_at`                         | čas zpracování jobem                         |

Prefix `sender_` je historický — sloupce drží **protistranu**, tedy u příchozí platby odesílatele a u odchozí vratky příjemce.
