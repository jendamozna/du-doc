# Párování plateb — pravidla

Implementační detail k modulu párování plateb ([README.md](../README.md) → **Modul párování plateb**). Stahování transakcí z banky popisuje [fio-sync.md](fio-sync.md), schéma [data-model.md](data-model.md).

## Alokace

- Vazba transakce ↔ přihláška je **M:N** a nese částku (`PAYMENT_ALLOCATION.amount`) — jedna platba se dá rozdělit mezi více přihlášek a jedna přihláška posbírat z více plateb.
- Stav úhrady přihlášky se **počítá ze součtu alokací** vůči ceně, neukládá se jako samostatné číslo:
  - součet < cena → `PartialPaid`,
  - součet = cena → `Paid`,
  - součet > cena → `Overpayment`.
- Cena přihlášky = základní cena podle typu účastníka (`EVENT_PRICE`) + součet příplatků zvolených položek číselníků (viz [event-fields.md](event-fields.md)).
- **Splatnost je vlastnost akce** a zadává se jedním ze dvou způsobů: relativně (`EVENT.payment_due_days`, např. 14 dní od podání přihlášky), nebo absolutně (`EVENT.payment_due_date`, pevné datum pro celou akci). Vyplňuje se právě jedno z polí; výchozí hodnota přichází ze šablony akce, fallback je 14 dní. Termín přihlášky se pak počítá:
  - relativně → `MIN(REGISTRATION.created_at + payment_due_days, EVENT.starts_at)`,
  - absolutně → `payment_due_date` (u přihlášek podávaných po tomto datu platí splatnost ihned).
    Stejný výpočet používají výzvy k platbě, připomínky i report Platby ([reports.md](reports.md)).
- U každé alokace se eviduje `matched_by` (`auto` / `manual`), `match_method`, čas spárování a čas odeslání potvrzení.

## Způsoby spárování (`match_method`)

| Hodnota               | Shoda                                                                                 |
| --------------------- | ------------------------------------------------------------------------------------- | --- | -------- | ---------------------------------------------- |
| `ss_vs_amount`        | SS, VS i částka                                                                       |
| `ss_vs_partial`       | SS, VS a částečná úhrada                                                              |
| `ss_vs_overpayment`   | SS, VS a přeplatek                                                                    |
| `vs_exact_name`       | VS, částka a jméno odesílatele = vlastník přihlášky nebo poznámka platby = název akce |
| `ss_exact_name`       | SS, částka a jméno odesílatele = vlastník přihlášky                                   |
| `vs_partial_name`     | VS, částečná úhrada a shoda jména odesílatele / poznámky platby                       |
| `vs_overpayment_name` | VS, přeplatek a shoda jména odesílatele / poznámky platby                             |
| `manual`              | ruční rozdělení účetní                                                                |     | `refund` | vratka nebo převod přeplatku — záporná alokace |

- SS identifikuje akci, VS přihlášku.
- **Částky se porovnávají přesně, žádná tolerance se neuplatňuje.** Rozdíl o korunu není shoda — je to nedoplatek (`PartialPaid`), nebo přeplatek (`Overpayment`). Zaokrouhlovací pásmo by zavádělo tichou ztrátu penez a v účetnictví se hledá hůř než viditelný rozdíl.
- Automatické párování běží hned po importu nových transakcí; ruční alokace lze kdykoli opravit.

## Pořadí pravidel

Pravidla tvoří **seřazený seznam**. Vyhodnocují se shora dolů a vyhrává první, které vrátí právě jednoho kandidáta:

| #   | Pravidlo                                      | Výsledný `match_method`                   | Alokace     |
| --- | --------------------------------------------- | ----------------------------------------- | ----------- |
| 1   | SS + VS + částka = zbývající cena             | `ss_vs_amount`                            | automaticky |
| 2   | SS + VS, částka menší / větší                 | `ss_vs_partial` / `ss_vs_overpayment`     | automaticky |
| 3   | VS + částka + jméno odesílatele nebo poznámka | `vs_exact_name`                           | automaticky |
| 4   | SS + částka + jméno odesílatele               | `ss_exact_name`                           | návrh       |
| 5   | VS + jméno, částka menší / větší              | `vs_partial_name` / `vs_overpayment_name` | návrh       |

- **Automaticky** znamená, že alokace vznikne bez zásahu člověka; **návrh** znamená, že se transakce zobrazí účetní s předvyplněným rozdělením, které potvrdí nebo upraví. Hranice mezi oběma sloupci je konfigurace, ne konstanta v kódu.
- Pravidla 1–2 stojí na symbolech, které systém sám vygeneroval, proto mají přednost před pravidly, která se opirají o jméno odesílatele nebo text poznámky.
- Shoda jména je normalizovaná (bez diakritiky, malá písmena, pořadí jméno/příjmení nerozhoduje) a porovnává se proti vlastníkovi přihlášky i jeho zákonným zástupcům.
- Žádné pravidlo nesmí alokovat víc, než kolik na transakci zbývá nerozděleného.
- Pořadí je závazné kvůli testovatelnosti — sada vstupních transakcí má dávat předem daný `match_method`, jinak nelze chování regresně ověřit.
- Párování se spouští nejen po importu transakcí, ale i při vzniku přihlášky — platba může dorazit dřív, než se člověk přihlásí.

## Více kandidátů

Nejčastější reálný případ: rodič pošle jednou platbou za tři děti, VS je jeho vlastní nebo žádný.

- Vrátí-li pravidlo víc než jednoho kandidáta, **nikdy se nealokuje automaticky**. Vznikne návrh rozdělení, který účetní potvrdí nebo upraví.
- Sedí-li součet zbývajících cen všech kandidátů přesně na částku platby, nabídne se rozpad 1:N s již předvyplněnými částkami; jinak se nabídne seznam kandidátů s prázdnými částkami.
- Návrh se **neukládá** — počítá se při otevření transakce, aby nezastaral, když mezitím přibude přihláška nebo se změní cena.
- **Stav transakce se počítá** ze součtu alokací vůči její částce: `unmatched` (nic) → `partially_allocated` (něco zbývá) → `allocated` (rozděleno beze zbytku). Navíc lze transakci označit jako `ignored` (příspěvek, refundace, platba mimo systém) — to je jediný ručně nastavený příznak.
- **Nerozdělený zbytek** (`amount − Σ alokací`) je hlavní pracovní fronta účetní; jeho výše a stáří jsou vidět v přehledu.

## Přeplatek a vratka

- Přeplatek přihlášky = `Σ alokací − cena` a **počítá se**, neukládá se jako záznam. Nikdy se nevrací automaticky.
- Systém nabídne účetní tři řešení: **vrátit** odesílateli, **převést** na jinou přihlášku též osoby, nebo **ponechat** (dar) s poznámkou. Do rozhodnutí zůstává přihláška ve stavu `Overpayment` a je vidět v reportu Platby.
- Vratka se eviduje jako **záporná alokace** na původní transakci (`match_method = 'refund'`), nikoli mazaním nebo úpravou původní alokace — historie plateb zůstává dohledatelná a stav přihlášky se přepočte sám.
- Převod na jinou přihlášku je dvojice záporná + kladná alokace též transakce, takže součet alokací transakce zůstává roven její částce.
- Účetní provádí **výplatu vratky mimo systém** — aplikace nedrží odchozí platební příkazy, jen eviduje, že vratka byla vypořádána.
- Stejný postup se použije při stornu přihlášky ([registration-lifecycle.md](registration-lifecycle.md)); liší se jen výše vratky, kterou určují storno pravidla akce.

## Potvrzení

- Za každou napárovanou platbu (i částečnou) se odesílá potvrzení; odeslání se eviduje na alokaci (`confirmation_sent_at`), aby se e-mail neposlal dvakrát.
