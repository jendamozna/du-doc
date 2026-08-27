# TODO — hodnocení specifikace a co dopsat

Hodnocení dokumentu z pohledu „může podle toho AI/tým naprogramovat aplikaci".

## Hodnocení po sekcích

| Sekce                              | Stav     | Poznámka                                                                                                   |
| ---------------------------------- | -------- | ---------------------------------------------------------------------------------------------------------- |
| Datový model (ER)                  | 🟢 Ready | [docs/data-model.md](docs/data-model.md) — ~50 entit, pole, enumy, FK. Nejsilnější část.                   |
| Výběrové číselníky (`EVENT_FIELD`) | 🟢 Ready | [docs/event-fields.md](docs/event-fields.md) — režimy, kapacita, fáze, ceny. Dá se kódovat 1:1.            |
| Hlídky Stezky + stanoviště         | 🟢 Ready | [docs/race-patrols.md](docs/race-patrols.md) — kategorie, věková pravidla, konzistenční funkce.            |
| Typy a šablony akcí                | 🟢 Ready | Tabulka typů + obsah šablony.                                                                              |
| Retence a GDPR                     | 🟢 Ready | Tabulka lhůt; joby popsané. Auditní log v [docs/audit-log.md](docs/audit-log.md).                          |
| Bankovní synchronizace (Fio)       | 🟢 Ready | [docs/fio-sync.md](docs/fio-sync.md) — token, kurzor, idempotence, rate limit, chybové stavy.              |
| Splatnost a stav úhrady            | 🟢 Ready | Relativní nebo absolutní splatnost na akci; stav se počítá ze součtu alokací.                              |
| Lifecycle osoby                    | 🟢 Ready | [docs/person-lifecycle.md](docs/person-lifecycle.md) — dvě osy, matice kombinací, guardy, dopady na vazby. |
| Vazba rodič ↔ dítě                 | 🟢 Ready | [docs/parent-child-lifecycle.md](docs/parent-child-lifecycle.md) — vznik, schvalování, práva podle stavu.  |
| Regiony                            | 🟢 Ready | [docs/region-lifecycle.md](docs/region-lifecycle.md) — stavy, slučování, verzovaná příslušnost, snapshot.  |
| Přihlašování na akce               | 🟢 Ready | [docs/registration-lifecycle.md](docs/registration-lifecycle.md) — brány, události, guardy, lhůty.         |
| Modul párování plateb              | 🟢 Ready | [docs/payment-matching.md](docs/payment-matching.md) — pořadí pravidel, víc kandidátů, přeplatek a vratka. |
| Deduplikace / merge                | 🟢 Ready | [docs/person-merge.md](docs/person-merge.md) — schvalování, konflikty polí, kolize vazeb, revert.          |
| Role a oprávnění                   | 🟢 Ready | [docs/authorization.md](docs/authorization.md) — tři vrstvy, matice po doménách, citlivá data.             |
| Reporty                            | 🟢 Ready | [docs/reports.md](docs/reports.md) — metriky, parametry, scope, hrany, požadavky na model.                 |
| Notifikace / e-maily               | 🟢 Ready | [docs/notifications.md](docs/notifications.md) — katalog událost → příjemce → šablona → načasování.        |
| Validace / invarianty              | 🟢 Ready | [docs/validation.md](docs/validation.md) — formáty, podmíněná povinnost, unikátnosti, invarianty.          |
| API / hranice modulů               | 🔴 Gap   | Žádný kontrakt (endpointy/DTO) — pokud má AI dělat i backend API.                                          |
| UI / obrazovky / toky              | 🔴 Gap   | Bez wireframů/flow; portál i admin.                                                                        |
| Nefunkční požadavky                | 🟢 Ready | [docs/non-functional.md](docs/non-functional.md) — OAuth, úložiště, šifrování, e-maily, ...                |

## Co dopsat (podle priority pro AI)

1. **API kontrakt / hranice modulů** — pokud AI staví i server (endpointy nebo aspoň use-case seznam).
