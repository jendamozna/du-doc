# TODO — hodnocení specifikace a co dopsat

Hodnocení dokumentu z pohledu „může podle toho AI/tým naprogramovat aplikaci".

## Hodnocení po sekcích

| Sekce                              | Stav        | Poznámka                                                                                                                        |
| ---------------------------------- | ----------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Datový model (ER)                  | 🟢 Ready    | [docs/data-model.md](docs/data-model.md) — ~50 entit, pole, enumy, FK. Nejsilnější část.                                        |
| Výběrové číselníky (`EVENT_FIELD`) | 🟢 Ready    | [docs/event-fields.md](docs/event-fields.md) — režimy, kapacita, fáze, ceny. Dá se kódovat 1:1.                                 |
| Hlídky Stezky + stanoviště         | 🟢 Ready    | [docs/race-patrols.md](docs/race-patrols.md) — kategorie, věková pravidla, konzistenční funkce.                                 |
| Typy a šablony akcí                | 🟢 Ready    | Tabulka typů + obsah šablony.                                                                                                   |
| Retence a GDPR                     | 🟢 Ready    | Tabulka lhůt; joby popsané. Auditní log v [docs/audit-log.md](docs/audit-log.md).                                               |
| Bankovní synchronizace (Fio)       | 🟢 Ready    | [docs/fio-sync.md](docs/fio-sync.md) — token, kurzor, idempotence, rate limit, chybové stavy.                                   |
| Splatnost a stav úhrady            | 🟢 Ready    | Relativní nebo absolutní splatnost na akci; stav se počítá ze součtu alokací.                                                   |
| Lifecycle osoby                    | 🟢 Ready    | [docs/person-lifecycle.md](docs/person-lifecycle.md) — dvě osy, matice kombinací, guardy, dopady na vazby.                      |
| Vazba rodič ↔ dítě                 | 🟢 Ready    | [docs/parent-child-lifecycle.md](docs/parent-child-lifecycle.md) — vznik, schvalování, práva podle stavu.                       |
| Regiony                            | 🟢 Ready    | [docs/region-lifecycle.md](docs/region-lifecycle.md) — stavy, slučování, verzovaná příslušnost, snapshot.                       |
| Přihlašování na akce               | 🟢 Ready    | [docs/registration-lifecycle.md](docs/registration-lifecycle.md) — brány, události, guardy, lhůty.                              |
| Modul párování plateb              | 🟢 Ready    | [docs/payment-matching.md](docs/payment-matching.md) — pořadí pravidel, víc kandidátů, přeplatek a vratka, ruční režim bez API. |
| Deduplikace / merge                | 🟢 Ready    | [docs/person-merge.md](docs/person-merge.md) — schvalování, konflikty polí, kolize vazeb, revert.                               |
| Role a oprávnění                   | 🟢 Ready    | [docs/authorization.md](docs/authorization.md) — tři vrstvy, matice po doménách, citlivá data.                                  |
| Reporty                            | 🟢 Ready    | [docs/reports.md](docs/reports.md) — metriky, parametry, scope, hrany, požadavky na model.                                      |
| Notifikace / e-maily               | 🟢 Ready    | [docs/notifications.md](docs/notifications.md) — katalog událost → příjemce → šablona → načasování.                             |
| Validace / invarianty              | 🟢 Ready    | [docs/validation.md](docs/validation.md) — formáty, podmíněná povinnost, unikátnosti, invarianty.                               |
| Hranice modulů a doménové události | 🟢 Ready    | [docs/modules.md](docs/modules.md) — 15 modulů, vlastník každé entity, katalog událostí, pravidla doručování.                   |
| API kontrakt (endpointy/DTO)       | 🟡 Optional | Nutné pouze pokud má být API explicitně specifikováno pro frontend nebo externí integrace.                                      |
| UI / obrazovky / toky              | 🔴 Gap      | Bez wireframů/flow; portál i admin.                                                                                             |
| Nefunkční požadavky                | 🟢 Ready    | [docs/non-functional.md](docs/non-functional.md) — technologický stack, OAuth, úložiště, šifrování, e-maily, ...                |

---

V `du-doc-ux-pruvodce.md` je hotový inventář 35 flow (plochy A–D), šablona zpracování a u většiny flow i specifikace obrazovek. Co chybí, není „napsat UI“, ale **uzavřít rozhodnutí, doplnit navigační vrstvu a texty, a napojit to na build**.

## Fáze 0 · Srovnat stav (nejlevnější, udělat hned)

- Přepsat řádek „UI / obrazovky / toky“ z 🔴 Gap na 🟡 Partial s odkazem na UX průvodce; popsat, co konkrétně zbývá (rozhodnutí, IA, texty, wireframy).
- Opravit zastaralé odkazy uvnitř průvodce: cituje TODO #1/#3/#4 jako mezery, ale mezitím vznikly `authorization.md`, `notifications.md`, `validation.md`. Týká se to mj. sekcí „Známá mezera“, „Výslovně otevřené“ a rozhodnutí D7/D10.
- Zvážit rozpad 2 500řádkového souboru na `docs/ux/00-orientace.md`, `01-inventar-a-rozhodnuti.md`, `a-portal.md`, `b-oddil.md`, `c-ustredi.md`, `d-self.md` — jinak bude každá revize bolet (a HTML se generuje stejně dobře z více souborů).

## Fáze 1 · Uzavřít rozhodnutí (blokuje všechno ostatní)

- Vytáhnout **všechna** `[K rozhodnutí]` (D1–D10 v sekci 2.3 + desítky lokálních u jednotlivých obrazovek) do jednoho `docs/ux-decisions.md` ve formátu ADR: kontext → varianty → rozhodnutí → dopad na flow.
- Prioritizovat ta, která mění skelet obrazovek: **D1** (struktura skupinové přihlášky), **D3** (váha tokenového odkazu), **D4** (QR per přihláška vs. rodinné), **D5** (české popisky stavů), **D7** (strategie validací), **D9** (priority zařízení).
- Část rozhodnutí je vlastně **změna spec, ne UX** — vyčlenit je zvlášť a poslat do příslušných doc souborů: chybějící akce „odmítnout“ u zástupce, token pro `RECOMMENDATION`, chování akce s cenou > 0 bez bankovního účtu, chybějící cena pro typ účastníka, znovuposlání ztraceného tokenu.

## Fáze 2 · Navigační vrstva (jediná úplně chybějící úroveň)

Průvodce popisuje flow a obrazovky, ale ne systém jako celek. Doplnit `docs/ux/navigace.md`:

- **Sitemap per plocha** + URL/route schéma (veřejné, tokenové `/p/{token}`, admin `/oddil/{id}/...`).
- Globální navigace, přepínač oddílu/role (uživatel může být HVO i rodič současně), breadcrumbs, chování po přihlášení.
- **Vstupní body z e-mailů a tokenů** jako první třída — pro zástupce a náhradníky je e-mail celé UI.
- Mapa „obrazovka × role“ ověřená proti `authorization.md` — každá obrazovka musí mít doložené oprávnění a definované chování při jeho absenci (skrýt vs. read-only vs. 403).

## Fáze 3 · Katalog textů, stavů a validací

- Finální **české popisky 9 stavů přihlášky** + varianta „checklist co chybí“ (rozhodnutí D5), jednotně použitá v S6, D1 a B3.
- Katalog chybových hlášek a prázdných stavů; texty pak zpětně zapsat do `validation.md`, ať UI a backend validují stejně.
- Ověřit, že každé pole formulářů z průvodce má protějšek v `data-model.md` a pravidlo ve `validation.md` — tenhle křížový průchod odhalí zbylé mezery rychleji než cokoli jiného.

## Fáze 4 · E-maily jako obrazovky

Sloučit `notifications.md` s UX vrstvou: pro každou notifikaci předmět, tělo, CTA, odesílatel (oddílové SMTP vs. systém), a odkaz na cílovou obrazovku. Bez toho zůstávají flow A4, A7, A8, B4 useknuté v půli.

## Fáze 5 · Vizuál — až po fázích 1–3

- Rozhodnout formát: **wireframy (Figma)** vs. **rovnou HTML/komponentový prototyp**. Vzhledem k poznámce v průvodci („aplikaci generuje AI z těchto dokumentů“) dává větší smysl přeskočit wireframy a jít na **inventář komponent + design tokeny** (mobile-first dle `non-functional.md`).
- Minimální set komponent, který pokryje 80 % obrazovek: stavový checklist, karta akce, formulářový blok účastníka, platební panel s QR, fronta ke schválení, párovací dvoupanel, datová tabulka s filtry.

## Fáze 6 · Napojení na druhou mezeru (API)

Specifikace obrazovek jsou nejlepší dostupný zdroj pro **use-case seznam** — každý krok flow = jedna operace. Doporučuji API kontrakt psát rovnou z hotových flow (nejdřív P1 sada), ne nezávisle; jinak vzniknou dvě neslučitelné pravdy.

## Doporučené pořadí práce (vertikální řez, ne po plochách)

1. Fáze 0 + extrakce rozhodnutí (Fáze 1).
2. Uzavřít D1, D3, D4, D5, D7, D9.
3. P1 řez „rodič přihlásí dítě a zaplatí“: A1 → A2 → A3 → A5 → A6 → A7 a jejich admin protějšky B1, B2, B3, B4, B6 — dotáhnout do stavu „AI podle toho postaví obrazovku“ včetně navigace, textů a e-mailů.
4. Teprve pak P2/P3 (A8–A14, B5–B11, D1–D4) a nakonec P4 (C5, C6, D3).

## Definice hotovo pro každou obrazovku

Účel · publikum · pole napojená na entity datového modelu · stavy (prázdný/načítání/chyba/úspěch) · validace s texty · oprávnění a chování bez něj · mobilní chování · route · notifikace, které z ní vedou nebo do ní míří · nula nevyřešených `[K rozhodnutí]`.

Chceš, abych některou fázi rozpracoval detailněji — třeba připravil strukturu `docs/ux-decisions.md` se seznamem všech nalezených `[K rozhodnutí]`?

Created 10 todos
