# Párování plateb — pravidla

Implementační detail k modulu párování plateb ([README.md](../README.md) → **Modul párování plateb**). Stahování transakcí z banky popisuje [fio-sync.md](fio-sync.md), schéma [data-model.md](data-model.md).

## Alokace

- Vazba transakce ↔ přihláška je **M:N** a nese částku (`PAYMENT_ALLOCATION.amount`) — jedna platba se dá rozdělit mezi více přihlášek a jedna přihláška posbírat z více plateb.
- Stav úhrady přihlášky se **počítá ze součtu alokací** vůči ceně, neukládá se jako samostatné číslo:
  - součet < cena → `PartialPaid`,
  - součet = cena → `Paid`,
  - součet > cena → `Overpayment`.
- Cena přihlášky = základní cena podle typu účastníka (`EVENT_PRICE`) + součet příplatků zvolených položek číselníků (viz [event-fields.md](event-fields.md)).
- **Splatnost** se neukládá, počítá se z přihlášky: `MIN(REGISTRATION.created_at + lhůta, EVENT.starts_at)`, kde lhůta je klíč v nastavení oddílu s defaultem 14 dní. Používají ji shodně výzvy k platbě, připomínky i report Platby ([reports.md](reports.md)).
- U každé alokace se eviduje `matched_by` (`auto` / `manual`), `match_method`, čas spárování a čas odeslání potvrzení.

## Způsoby spárování (`match_method`)

| Hodnota               | Shoda                                                                                 |
| --------------------- | ------------------------------------------------------------------------------------- |
| `ss_vs_amount`        | SS, VS i částka                                                                       |
| `ss_vs_partial`       | SS, VS a částečná úhrada                                                              |
| `ss_vs_overpayment`   | SS, VS a přeplatek                                                                    |
| `vs_exact_name`       | VS, částka a jméno odesílatele = vlastník přihlášky nebo poznámka platby = název akce |
| `ss_exact_name`       | SS, částka a jméno odesílatele = vlastník přihlášky                                   |
| `vs_partial_name`     | VS, částečná úhrada a shoda jména odesílatele / poznámky platby                       |
| `vs_overpayment_name` | VS, přeplatek a shoda jména odesílatele / poznámky platby                             |
| `manual`              | ruční rozdělení účetní                                                                |

- SS identifikuje akci, VS přihlášku. Nesedí-li částka jediné přihlášce, systém párování nenavrhne automaticky a nechá účetní rozdělit částku ručně.
- Automatické párování běží hned po importu nových transakcí; ruční alokace lze kdykoli opravit.

## Potvrzení

- Za každou napárovanou platbu (i částečnou) se odesílá potvrzení; odeslání se eviduje na alokaci (`confirmation_sent_at`), aby se e-mail neposlal dvakrát.
