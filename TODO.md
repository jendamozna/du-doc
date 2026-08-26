# TODO — hodnocení specifikace a co dopsat

Hodnocení dokumentu z pohledu „může podle toho AI/tým naprogramovat aplikaci".

## Hodnocení po sekcích

| Sekce                              | Stav            | Poznámka                                                                                                   |
| ---------------------------------- | --------------- | ---------------------------------------------------------------------------------------------------------- |
| Datový model (ER)                  | 🟢 Ready        | [docs/data-model.md](docs/data-model.md) — ~50 entit, pole, enumy, FK. Nejsilnější část.                   |
| Výběrové číselníky (`EVENT_FIELD`) | 🟢 Ready        | [docs/event-fields.md](docs/event-fields.md) — režimy, kapacita, fáze, ceny. Dá se kódovat 1:1.            |
| Hlídky Stezky + stanoviště         | 🟢 Ready        | [docs/race-patrols.md](docs/race-patrols.md) — kategorie, věková pravidla, konzistenční funkce.            |
| Typy a šablony akcí                | 🟢 Ready        | Tabulka typů + obsah šablony.                                                                              |
| Retence a GDPR                     | 🟢 Ready        | Tabulka lhůt; joby popsané. Auditní log v [docs/audit-log.md](docs/audit-log.md).                          |
| Bankovní synchronizace (Fio)       | 🟢 Ready        | [docs/fio-sync.md](docs/fio-sync.md) — token, kurzor, idempotence, rate limit, chybové stavy.              |
| Splatnost a stav úhrady            | 🟢 Ready        | Relativní nebo absolutní splatnost na akci; stav se počítá ze součtu alokací.                              |
| Lifecycle osoby                    | 🟡 OK / dořešit | Stavy jsou, ale přechody bez guardů/triggerů.                                                              |
| Přihlašování na akce               | 🟢 Ready        | [docs/registration-lifecycle.md](docs/registration-lifecycle.md) — brány, události, guardy, lhůty.         |
| Modul párování plateb              | 🟢 Ready        | [docs/payment-matching.md](docs/payment-matching.md) — pořadí pravidel, víc kandidátů, přeplatek a vratka. |
| Deduplikace / merge                | 🟢 Ready        | [docs/person-merge.md](docs/person-merge.md) — schvalování, konflikty polí, kolize vazeb, revert.          |
| Role a oprávnění                   | 🔴 Gap          | Největší mezera — chybí matice akce × role. „Práva se přidělují u akce" není specifikace.                  |
| Reporty                            | 🟢 Ready        | [docs/reports.md](docs/reports.md) — metriky, parametry, scope, hrany, požadavky na model.                 |
| Notifikace / e-maily               | 🔴 Gap          | Spousta „systém pošle e-mail", ale žádný katalog (událost → šablona → příjemce → načasování).              |
| Validace / invarianty              | 🔴 Gap          | Rozpuštěno v próze; chybí pravidla po polích (formáty, povinnosti dle stavu).                              |
| API / hranice modulů               | 🔴 Gap          | Žádný kontrakt (endpointy/DTO) — pokud má AI dělat i backend API.                                          |
| UI / obrazovky / toky              | 🔴 Gap          | Bez wireframů/flow; portál i admin.                                                                        |
| Nefunkční požadavky                | 🟢 Ready        | [docs/non-functional.md](docs/non-functional.md) — OAuth, úložiště, šifrování, e-maily, ...                |

## Co dopsat (podle priority pro AI)

1. **Autorizační matice** (akce × role × scope) — bez toho AI hádá práva. Nejvyšší priorita.
2. **Stavové automaty** pro `PARENT_CHILD` a `REGION` — stavy, přechody, spouštěče, guardy. Hotové jsou `REGISTRATION` ([docs/registration-lifecycle.md](docs/registration-lifecycle.md)) a `MERGE_REQUEST` ([docs/person-merge.md](docs/person-merge.md)); lifecycle osoby chybí.
3. **Katalog notifikací** — tabulka událost → příjemce → šablona → načasování/opakování.
4. **Validační pravidla a byznys-invarianty** — po polích (formát e-mailu, kdy je `birth_date` povinné, IČO, unikátnosti).
5. **API kontrakt / hranice modulů** — pokud AI staví i server (endpointy nebo aspoň use-case seznam).
6. **Hromadné přihlášení vedoucím (mimo portál)** — celá kapitola „Přihlašování na akce" popisuje jen samoobslužný tok přes veřejný portál (účastník nebo rodič si podává přihlášku sám). U docházky je explicitně řešeno, že vedoucí může akci založit zpětně a rovnou vybrat libovolné účastníky ze seznamu osob oddílu — obdobný hromadný zápis pro **přihlášky** ale specifikace nezmiňuje. Chybí odpověď na to, jestli HVO/Vedoucí smí za skupinu existujících členů (např. celou družinu) založit přihlášky najednou bez toho, aby si je každý podával sám, jaký stav taková přihláška dostane (rovnou čeká na platbu?) a jak se to promítne do kapacity a pořadí náhradníků.
7. **Pořadí podání přihlášky vs. založení `DU_MEMBERSHIP`** — cena a způsobilost se vyhodnocují k roku dané akce (README → **Člen DU**), ne podle aktuálního data, takže účastník přihlášený v prosinci na lednovou akci může platit cenu pro DU, pokud HVO členství pro nový rok už založil. Není ale řečeno, zda se cena přihlášky **zafixuje v okamžiku podání**, nebo se **přepočítává** až do splatnosti/platby — výsledek se liší podle toho, kdy HVO členství pro nový rok stihne založit vůči podání přihlášky.
8. **Nezletilý Rádce** — text jen konstatuje, že „Rádci nevidí citlivá data dětí, nejsou plnoletí" (README → **Rádce**), ale účet Rádce zakládá HVO pozvánkou stejně jako dospělým rolím, bez zmínky o zastoupení zákonným zástupcem. Chybí, zda pozvánku/založení role musí schválit rodič, jak se role promítá do existující vazby rodič ↔ dítě a kdo právně odpovídá za činy nezletilého Rádce v systému (zápis docházky, úprava chytrých sloupců).
9. **Členský příspěvek DU vs. více oddílů** — `DU_MEMBERSHIP` má unikátní klíč jen osoba + rok ([data-model.md](docs/data-model.md#L406-L410)), `unit_id` do klíče nepatří, takže osoba může mít max. jedno členství DU za rok v celém systému, přestože může být evidovaná ve víc oddílech současně (README → **Osoba vs. uživatelský účet**). Není řečeno, který oddíl smí členství založit, když je osoba aktivní ve víc oddílech, zda cena DU platí i na akcích jiného oddílu než toho, co členství založil, a co se stane při přesunu osoby mezi oddíly v průběhu roku.
10. **Výběr a párování členského příspěvku DU** — „systém platbu příspěvku neřeší" (README → **Člen DU**), ale příspěvek není akce, takže chybí, jak se vybírá a páruje, a kde se platí (účet organizace, nebo oddílu) — navazuje na bod 9. Návrh k vložení do sekce „Člen DU":
    - Členský příspěvek DU se vybírá na úrovni organizace jako zvláštní platební položka s vlastním SS, párovaná stejným mechanismem jako akce (VS = osoba/přihláška příspěvku).
    - Příspěvek je jednou za osobu a kalendářní rok — zaplacení povýší osobu na „člena DU" globálně, bez ohledu na počet oddílů, kde je evidována.
    - Stav „člen DU" a evidenci plateb příspěvku spravuje a vidí organizace (ORG-A/ORG-Ú); oddíl vidí jen výsledný stav členství.

## Jaká úroveň „stačí" pro AI

- **Pro scaffolding (DB, entity, základní CRUD, dobře popsané subsystémy):** současná úroveň **stačí**.
- **Pro implementaci bez hádání (autorizace, platby, notifikace, reporty, UI):** potřeba doplnit **body 1–4** jako minimum; ideálně i 5–7.
