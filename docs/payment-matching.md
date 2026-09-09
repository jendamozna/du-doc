# Párování plateb — pravidla

Implementační detail k modulu párování plateb ([README.md](../README.md) → **Modul párování plateb**). Stahování transakcí z banky popisuje [fio-sync.md](fio-sync.md), schéma [data-model.md](data-model.md).

## Alokace

- Vazba transakce ↔ platební cíl je **M:N** a nese částku (`PAYMENT_ALLOCATION.amount`) — cílem je přihláška, oddílový členský předpis nebo dávka příspěvků DU. Jedna platba se tak dá rozdělit mezi více cílů a jeden cíl posbírat z více plateb.
- Stav úhrady přihlášky se **počítá ze součtu alokací** vůči ceně, neukládá se jako samostatné číslo:
  - součet < cena → `PartialPaid`,
  - součet = cena → `Paid`,
  - součet > cena → `Overpayment`.
- Cena přihlášky = **základní cena zafixovaná při podání** (`REGISTRATION.base_price`, odvozená z `EVENT_PRICE` platné k `created_at` a typu účastníka) + součet příplatků aktuálně zvolených položek číselníků (viz [event-fields.md](event-fields.md)). Pozdější změna ceníku ani vznik členství DU už podanou přihlášku nepřeceňuje — základní cenu může změnit jen vedoucí ručně.
- **Splatnost je vlastnost akce** a zadává se jedním ze dvou způsobů: relativně (`EVENT.payment_due_days`, např. 14 dní od podání přihlášky), nebo absolutně (`EVENT.payment_due_date`, pevné datum pro celou akci). Vyplňuje se právě jedno z polí; výchozí hodnota přichází ze šablony akce, fallback je 14 dní. Termín přihlášky se pak počítá:
  - relativně → `MIN(REGISTRATION.created_at + payment_due_days, EVENT.starts_at)`,
  - absolutně → `payment_due_date` (u přihlášek podávaných po tomto datu platí splatnost ihned).
    Stejný výpočet používají výzvy k platbě, připomínky i report Platby ([reports.md](reports.md)).
- U každé alokace se eviduje `matched_by` (`auto` / `manual`), `match_method`, čas spárování a čas odeslání potvrzení.

## Oddílové členské příspěvky

Oddíl může od registrovaných členů a jejich zákonných zástupců vybírat roční členské příspěvky na svůj účet. Každý předpis (`UNIT_MEMBER_FEE`) je vázaný na osobu, oddíl a rok a obsahuje neměnný rozpad:

- **složka DU** = aktuální sazba `DU_FEE_RATE` pro rok; je nulová, pokud osoba již má platné `DU_MEMBERSHIP` pro stejný rok,
- **lokální složka** = částka nastavená HVO pro provoz oddílu v `UNIT_MEMBER_FEE_RATE`,
- **celkem** = součet obou složek, s vlastním VS a QR kódem na bankovní účet oddílu.

Předpis se vytváří jen pro aktivního registrovaného člena oddílu. Zaplatit jej může člen, případně jeho aktivní zákonný zástupce; příjemce výzvy a potvrzení se určí stejným způsobem jako u přihlášky.

- Alokace platby na předpis používá stejný mechanismus jako přihláška, ale cílí na `UNIT_MEMBER_FEE`.
- Součet alokací nejdříve kryje složku DU a až potom lokální složku. `du_collected = MIN(SUM(alokací), du_amount)` je výpočetní hodnota, ne samostatně uložený stav.
- Po uhrazení celé složky DU může HVO osobu zařadit do `DU_FEE_BATCH`; lokální složka se do dávky nikdy nezařazuje a zůstává příjmem oddílu.
- Členství DU nevzniká individuální platbou na účet oddílu. Vznikne až po spárování celé hromadné dávky s platbou na účet ústředí.
- Změna lokální sazby nemění existující předpisy. Oprava již uhrazeného předpisu se provádí novým opravným zápisem nebo vratkou, nikdy tichým přepsáním snapshotu.

## Způsoby spárování (`match_method`)

Pravidla tvoří **seřazený seznam**. Vyhodnocují se shora dolů a vyhrává první, které vrátí právě jednoho kandidáta:

| Hodnota               | Shoda                                                                                 | Alokace     |
| --------------------- | ------------------------------------------------------------------------------------- | ----------- |
| `ss_vs_amount`        | SS, VS i částka                                                                       | automaticky |
| `ss_vs_partial`       | SS, VS a částečná úhrada                                                              | automaticky |
| `ss_vs_overpayment`   | SS, VS a přeplatek                                                                    | automaticky |
| `vs_exact_name`       | VS, částka a jméno odesílatele = vlastník přihlášky nebo poznámka platby = název akce | automaticky |
| `ss_exact_name`       | SS, částka a jméno odesílatele = vlastník přihlášky                                   | automaticky |
| `vs_exact`            | VS, částka                                                                            | automaticky |
| `vs_partial_name`     | VS, částečná úhrada a shoda jména odesílatele / poznámky platby                       | návrh       |
| `vs_overpayment_name` | VS, přeplatek a shoda jména odesílatele / poznámky platby                             | návrh       |
| `manual`              | ruční rozdělení účetní                                                                |             |
| `refund`              | vratka nebo převod přeplatku — záporná alokace                                        |             |

- SS identifikuje akci, VS přihlášku.
- **Částky se porovnávají přesně, žádná tolerance se neuplatňuje.** Rozdíl o korunu není shoda — je to nedoplatek (`PartialPaid`), nebo přeplatek (`Overpayment`). Zaokrouhlovací pásmo by zavádělo tichou ztrátu penez a v účetnictví se hledá hůř než viditelný rozdíl.
- Automatické párování běží hned po importu nových transakcí; ruční alokace lze kdykoli opravit.
- **Automaticky** znamená, že alokace vznikne bez zásahu člověka; **návrh** znamená, že se transakce zobrazí účetní s předvyplněným rozdělením, které potvrdí nebo upraví. Hranice mezi oběma sloupci je konfigurace, ne konstanta v kódu.
- Shoda jména je normalizovaná (bez diakritiky, malá písmena, pořadí jméno/příjmení nerozhoduje) a porovnává se proti **vlastníkovi přihlášky, účastníkovi i jeho zákonným zástupcům** — zákonný zástupce platí za dítě pod svým jménem, takže jméno účastníka na výpisu často vůbec není.
- Žádné pravidlo nesmí alokovat víc, než kolik na transakci zbývá nerozděleného.
- Pořadí je závazné kvůli testovatelnosti — sada vstupních transakcí má dávat předem daný `match_method`, jinak nelze chování regresně ověřit.
- Párování se spouští nejen po importu transakcí, ale i při vzniku přihlášky — platba může dorazit dřív, než se člověk přihlásí.

## Více kandidátů

Nejčastější reálný případ: zákonný zástupce pošle jednou platbou za tři děti, VS je jeho vlastní nebo žádný.

- Vrátí-li pravidlo víc než jednoho kandidáta, **nikdy se nealokuje automaticky**. Vznikne návrh rozdělení, který účetní potvrdí nebo upraví.
- Sedí-li součet zbývajících cen všech kandidátů přesně na částku platby, nabídne se rozpad 1:N s již předvyplněnými částkami; jinak se nabídne seznam kandidátů s prázdnými částkami.
- Návrh se **neukládá** — počítá se při otevření transakce, aby nezastaral, když mezitím přibude přihláška nebo se změní cena.
- **Stav transakce se počítá** ze součtu alokací vůči její částce: `unmatched` (nic) → `partially_allocated` (něco zbývá) → `allocated` (rozděleno beze zbytku). Navíc lze transakci označit jako `ignored` (příspěvek, refundace, platba mimo systém) — to je jediný ručně nastavený příznak.
- **Nerozdělený zbytek** (`amount − Σ alokací`) je hlavní pracovní fronta účetní; jeho výše a stáří jsou vidět v přehledu.

Po každém běhu párovacího automatu vzniká událost `payment.reconciliation_completed`. Nese `transaction_id`, `account_id`, výsledek (`allocated`, `partially_allocated`, `unmatched` nebo `ambiguous`), `allocated_amount`, `unmatched_amount` a případné `candidates[]`. Událost slouží jako vstup pro notifikaci účetnímu oddílu; ruční potvrzení nebo oprava návrhu je samostatná operace a tento automatický výstup se jí nemění.

## Přeplatek a vratka

- Přeplatek přihlášky = `Σ alokací − cena` a **počítá se**, neukládá se jako záznam. Nikdy se nevrací automaticky.
- Systém nabídne účetní tři řešení: **vrátit** odesílateli, **převést** na jinou přihlášku též osoby, nebo **ponechat** (dar) s poznámkou. Do rozhodnutí zůstává přihláška ve stavu `Overpayment` a je vidět v reportu Platby.
- Vratka se eviduje jako **záporná alokace** na původní transakci (`match_method = 'refund'`), nikoli mazaním nebo úpravou původní alokace — historie plateb zůstává dohledatelná a stav přihlášky se přepočte sám.
- Převod na jinou přihlášku je dvojice záporná + kladná alokace též transakce, takže součet alokací transakce zůstává roven její částce.
- Účetní provádí **výplatu vratky mimo systém** — aplikace nedrží odchozí platební příkazy, jen eviduje, že vratka byla vypořádána.
- Stejný postup se použije při stornu přihlášky ([registration-lifecycle.md](registration-lifecycle.md)); liší se jen výše vratky, kterou určují storno pravidla akce.

## Potvrzení

- Za každou napárovanou platbu (i částečnou) se odesílá potvrzení; odeslání se eviduje na alokaci (`confirmation_sent_at`), aby se e-mail neposlal dvakrát.

## Oddíl bez bankovního API

Modul má dvě nezávislé vrstvy: **evidence plateb** (vše v tomto dokumentu) běží vždy, **synchronizace z API** ([fio-sync.md](fio-sync.md)) jen při přítomnosti tokenu. Oddíl s jinou bankou než Fio si `BANK_TRANSACTION` plní sám (`BANK_ACCOUNT.provider = 'manual'`).

Výpočet stavu úhrady, pořadí párovacích pravidel, přeplatky i vratky se tím **nemění vůbec** — `evaluate()` čte součet alokací, ne banku. Liší se jen zdroj transakcí.

### Jak transakce vznikne

| Způsob (`source`)  | Vstup                                                                 | Po uložení                                 |
| ------------------ | --------------------------------------------------------------------- | ------------------------------------------ |
| `import`           | Fio API                                                               | automatické párování                       |
| `statement_import` | výpis z internetbankingu (CSV / ABO / MT940) nahraný účetní           | automatické párování, stejná sada pravidel |
| `manual_entry`     | ruční zápis jedné platby (datum, částka, odesílatel, volitelně VS/SS) | automatické párování, stejná sada pravidel |

- Import výpisu je pro oddíl bez API hlavní pracovní režim — nahrání jednou týdně nahradí stažení z API.
- Zkratka **„označit jako zaplacené“** na seznamu přihlášek založí ruční transakci ve výši zbývající ceny a rovnou ji alokuje (`matched_by = 'manual'`).
- **VS se u ručního zápisu nevyžaduje.** Bez něj se pravidla opřená o VS/SS přeskočí a párování spadne na shodu jména odesílatele, případně na ruční rozdělení. Vynucovat VS by nemělo smysl — v nahraném výpisu bude často chybět také.
- K ručně zapsané transakci lze připojit poznámku a přílohu (sken výpisu).

### Idempotence importu výpisu

Výpis nenese ID pohybu, proto se `external_id` odvodí z otisku řádku:

```
fingerprint = hash(bank_account_id, date, amount, vs, ss, sender_account, message, pořadí_mezi_identickými_řádky)
external_id = "stmt:" + fingerprint
```

- Pořadové číslo ve skupině shodných řádků kryje legální případ dvou stejných částek bez VS ve stejný den.
- Opakované nahrání téhož nebo překrývajícího se výpisu narazí na stávající unikát `účet + external_id` a nic nezdvojí; před potvrzením se ukáže rozpad „nové / už známé“ řádky.
- Ruční zápis dostane `manual:<uuid>`. `external_id` tím zůstává povinné a unikátní pro všechny zdroje — v kódu nevzniká větev „transakce bez identifikátoru“.

### Oprava chybného zápisu

- Transakci se `source != 'import'` lze **stornovat** (`voided_at`), ale jen když nemá žádnou alokaci — účetní nejdřív dealokuje, pak stornuje. Historie plateb u přihlášky se tím neztratí.
- Špatná částka už rozdělená na přihlášky se řeší **zápornou alokací** stejně jako vratka výše, ne úpravou původní alokace.
- Stornovaná transakce mizí z fronty k párování a zůstává v auditním logu.

### Důsledky pro notifikace a lhůty

- **Připomínky nezaplacených plateb se neposílají.** Bez API systém nezná stav úhrady v reálném čase a urgoval by i ty, kdo zaplatili před nahráním výpisu. Vedoucí může výzvu poslat ručně ze seznamu přihlášek po splatnosti.
- **Automatické vypršení nezaplacené přihlášky** ([registration-lifecycle.md](registration-lifecycle.md)) nelze u ručního účtu zapnout — systém by rušil místa na základě neúplné informace.
- Výzva k platbě při podání přihlášky, QR kód i potvrzení o platbě fungují beze změny — nezávisí na znalosti stavu úhrady.
- Ruční zápis a import výpisu smí **ÚČE a HVO** ([authorization.md](authorization.md)) a logují se do auditního logu — v ručním režimu chybí bankovní protistrana, takže audit je jediné krytí.

### Přechod na API

Doplní-li oddíl později token, nic se nemigruje — ruční i stažené transakce koexistují na témž účtu a připomínky se zapnou. Při importu se hledá ruční transakce se shodným účtem, datem a částkou; shoda se nabídne ke sloučení místo tichého zdvojení.
