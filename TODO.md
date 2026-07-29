# TODO — hodnocení specifikace a co dopsat

Hodnocení dokumentu z pohledu „může podle toho AI/tým naprogramovat aplikaci".

## Celkový verdikt

Dokument je **silný na úrovni domény + datového modelu** (PRD/spec), ne na úrovni implementačního zadání. AI podle něj **spolehlivě postaví DB schéma a CRUD** a **dobře zvládne detailně popsané subsystémy** (číselníky, hlídky Stezky, párování plateb, reporty). Na **one-shot celé aplikace to ale nestačí** — chybí autorizační matice, formální stavové automaty, katalog notifikací, validační pravidla a API/UI kontrakt. AI by musela dělat hodně vlastních rozhodnutí.

Orientačně: **~75 % hotovo** pro backend/datovou vrstvu, **~45 %** pro kompletní funkční aplikaci vč. autorizace, UI a integrací.

Business popis žije v [README.md](README.md), implementační detaily v `docs/` — viz rozcestník na konci README.

## Hodnocení po sekcích

| Sekce                              | Stav            | Poznámka                                                                                                      |
| ---------------------------------- | --------------- | ------------------------------------------------------------------------------------------------------------- |
| Datový model (ER)                  | 🟢 Ready        | [docs/data-model.md](docs/data-model.md) — ~50 entit, pole, enumy, FK. Nejsilnější část.                      |
| Výběrové číselníky (`EVENT_FIELD`) | 🟢 Ready        | [docs/event-fields.md](docs/event-fields.md) — režimy, kapacita, fáze, ceny. Dá se kódovat 1:1.               |
| Hlídky Stezky + stanoviště         | 🟢 Ready        | [docs/race-patrols.md](docs/race-patrols.md) — kategorie, věková pravidla, konzistenční funkce.               |
| Typy a šablony akcí                | 🟢 Ready        | Tabulka typů + obsah šablony.                                                                                 |
| Retence a GDPR                     | 🟢 Ready        | Tabulka lhůt; joby popsané. Auditní log v [docs/audit-log.md](docs/audit-log.md).                             |
| Bankovní synchronizace (Fio)       | 🟢 Ready        | [docs/fio-sync.md](docs/fio-sync.md) — token, kurzor, idempotence, rate limit, chybové stavy.                 |
| Splatnost a stav úhrady            | 🟢 Ready        | Relativní nebo absolutní splatnost na akci; stav se počítá ze součtu alokací.                                 |
| Lifecycle osoby                    | 🟡 OK / dořešit | Stavy jsou, ale přechody bez guardů/triggerů.                                                                 |
| Přihlašování na akce               | 🟡 OK / dořešit | Stavy popsané, chybí formální state machine (co přesně `New→PendingPayment→…`).                               |
| Modul párování plateb              | 🟡 Dořešit      | [docs/payment-matching.md](docs/payment-matching.md); chybí tolerance částky, pořadí pravidel, víc kandidátů. |
| Deduplikace / merge                | 🟡 Dořešit      | Flow ano; řešení konfliktů pole-po-poli a revert jen rámcově.                                                 |
| Role a oprávnění                   | 🔴 Gap          | Největší mezera — chybí matice akce × role. „Práva se přidělují u akce" není specifikace.                     |
| Reporty                            | 🟢 Ready        | [docs/reports.md](docs/reports.md) — metriky, parametry, scope, hrany, požadavky na model.                    |
| Notifikace / e-maily               | 🔴 Gap          | Spousta „systém pošle e-mail", ale žádný katalog (událost → šablona → příjemce → načasování).                 |
| Validace / invarianty              | 🔴 Gap          | Rozpuštěno v próze; chybí pravidla po polích (formáty, povinnosti dle stavu).                                 |
| API / hranice modulů               | 🔴 Gap          | Žádný kontrakt (endpointy/DTO) — pokud má AI dělat i backend API.                                             |
| UI / obrazovky / toky              | 🔴 Gap          | Bez wireframů/flow; portál i admin.                                                                           |
| Nefunkční požadavky                | 🟡 Dořešit      | Měna, timezone a formáty hotové (README → Lokalizace); chybí OAuth config, úložiště souborů, i18n, DPH.       |

## Co dopsat (podle priority pro AI)

1. **Autorizační matice** (akce × role × scope) — bez toho AI hádá práva. Nejvyšší priorita.
2. **Stavové automaty** pro `REGISTRATION`, `PARENT_CHILD`, `MERGE_REQUEST`, `REGION` — stavy, přechody, spouštěče, guardy.
3. **Katalog notifikací** — tabulka událost → příjemce → šablona → načasování/opakování.
4. **Validační pravidla a byznys-invarianty** — po polích (formát e-mailu, kdy je `birth_date` povinné, IČO, unikátnosti).
5. **Algoritmus párování plateb** — základ hotový ([docs/payment-matching.md](docs/payment-matching.md)); dopsat toleranci částky, pořadí vyhodnocování pravidel a chování při více kandidátech.
6. ~~**Definice reportů**~~ — hotovo, viz [docs/reports.md](docs/reports.md).
7. **Nefunkční** — lokalizace hotová (měna, timezone, formáty); dopsat OAuth (Google/Facebook) config, šifrování (`*_enc`), úložiště dokumentů a retenční joby.
8. **API kontrakt / hranice modulů** — pokud AI staví i server (endpointy nebo aspoň use-case seznam).

## Jaká úroveň „stačí" pro AI

- **Pro scaffolding (DB, entity, základní CRUD, dobře popsané subsystémy):** současná úroveň **stačí**.
- **Pro implementaci bez hádání (autorizace, platby, notifikace, reporty, UI):** potřeba doplnit **body 1–4** jako minimum; ideálně i 5–7.

Doporučené pořadí: začít **autorizační maticí** a **stavovým automatem přihlášky** — mají největší dopad a nejlíp se z nich generuje kód.
