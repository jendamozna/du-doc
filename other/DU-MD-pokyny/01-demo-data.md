# 01 · Demo data (mock vrstva)

Tato data tvoří **výchozí obsah in-memory store** demo aplikace (viz `00-projekt-a-design.md`). Jsou **fiktivní** — jména, e-maily (`@example.cz`), telefony i čísla účtů jsou smyšlené a nepatří žádným skutečným osobám. IDčka jsou **stabilní** a specifikace obrazovek na ně odkazují — neměň je. Aktuální „dnešek“ dema: **Středa 26.8.** (rok 2026 — v UI se rok u letošních dat nezobrazuje).

## 1. Oddíly

| ID       | Název                  | Město   | Region                 | Pozn.                                                                                         |
| -------- | ---------------------- | ------- | ---------------------- | --------------------------------------------------------------------------------------------- |
| oddil-01 | Oddíl Severka          | Liberec | Region Sever           | hlavní demo oddíl — plocha B ukazuje jeho data                                                |
| oddil-02 | Oddíl Jestřábi         | Olomouc | Region Morava          | jen pro pestrost portálu                                                                      |
| oddil-03 | Oddíl Bobři            | Písek   | Region Jih             | bez akcí v demu                                                                               |
| oddil-04 | Oddíl Poutníci         | Brno    | — (bez regionu)        | plocha C: založen Pondělí 17.8., bez členů a akcí; pozvánka HVO nedoručena (pozvanka-01, § 9) |
| oddil-90 | Ústředí Dorostové unie | Praha   | — (nepatří do regionů) | speciální oddíl ústředí (`is_hq`); bez registrovaných členů, slouží celostátním akcím         |

**Typy a IČO oddílů (plocha C):** oddil-01 pobočný spolek s vlastním IČO (`04512774`) · oddil-02 pobočný spolek s vlastním IČO (`06923119`) · oddil-03 kolektivní člen (`08812331`) · oddil-04 pobočný spolek s vlastním IČO (`22903330`) · oddil-90 IČO ústředí (`00571245`).

**Bankovní účet oddílu Severka:** `2900123456/2010` (demo, Fio), synchronizace „naposledy Středa 26.8. 6:00“. **Nastavení oddílu Severka:** lhůta schválení zástupcem 7 dní; nabídka náhradníkovi 48 hodin; vypršení nezaplacených přihlášek vypnuto; moduly: družiny ✔, závody (Stezka) ✔, dobrovolnické hodiny ✔.

## 2. Osoby

Osoba ≠ účet: účet mají jen dospělí označení ✔ ve sloupci Účet. Děti a hosté účet nemají. Telefony a e-maily jen u dospělých.

### 2.1 Tým oddílu Severka (oddil-01)

| ID        | Jméno             | Role                   | E-mail                       | Telefon          | Pozn.                                                                              | Účet |
| --------- | ----------------- | ---------------------- | ---------------------------- | ---------------- | ---------------------------------------------------------------------------------- | ---- |
| osoba-001 | Martin Dvořáček   | HVO                    | martin.dvoracek@example.cz   | +420 601 111 222 | demo persona „HVO“                                                                 | ✔    |
| osoba-002 | Klára Vondrušková | VO, vede družinu Sovy  | klara.vondruskova@example.cz | +420 602 333 444 | per akce: úprava akcí a přihlášek                                                  | ✔    |
| osoba-003 | Tomáš Hruban      | VD, vede družinu Lišky | tomas.hruban@example.cz      | +420 603 555 666 | per akce: zápis docházky                                                           | ✔    |
| osoba-004 | Eliška Rákosová   | RÁD                    | eliska.rakosova@example.cz   | +420 604 123 987 | nar. 12.3.2009 (17) — nezletilá, nesmí vidět citlivá data dětí; jen zápis docházky | ✔    |
| osoba-005 | Ivana Šmídková    | ÚČE                    | ivana.smidkova@example.cz    | +420 605 777 888 | demo persona „účetní“                                                              | ✔    |

### 2.2 Zákonní zástupci (vazby na děti v § 2.3)

| ID        | Jméno              | E-mail                        | Telefon          | Děti                                  | Účet                             |
| --------- | ------------------ | ----------------------------- | ---------------- | ------------------------------------- | -------------------------------- |
| osoba-010 | Pavla Konvalinková | pavla.konvalinkova@example.cz | +420 606 101 010 | osoba-020, osoba-021                  | ✔ — demo persona „Rodič s účtem“ |
| osoba-011 | Radek Konvalinka   | radek.konvalinka@example.cz   | +420 607 202 020 | osoba-020, osoba-021 (druhý zástupce) | –                                |
| osoba-012 | Monika Šálková     | monika.salkova@example.cz     | +420 608 303 030 | osoba-022, osoba-023                  | –                                |
| osoba-013 | Jiří Brázdil       | jiri.brazdil@example.cz       | +420 609 404 040 | osoba-024                             | –                                |
| osoba-014 | Lenka Hrbáčková    | lenka.hrbackova@example.cz    | +420 601 505 050 | osoba-025, osoba-026                  | –                                |
| osoba-015 | Ondřej Kučeravý    | ondrej.kuceravy@example.cz    | +420 602 606 060 | osoba-027                             | –                                |
| osoba-016 | Dita Peštová       | dita.pestova@example.cz       | +420 603 707 070 | osoba-028, osoba-029                  | –                                |
| osoba-017 | Zuzana Vrabcová    | zuzana.vrabcova@example.cz    | +420 604 808 080 | osoba-030                             | –                                |
| osoba-018 | Roman Slanina      | roman.slanina@example.cz      | +420 605 909 090 | osoba-031                             | –                                |
| osoba-019 | Alena Doudová      | alena.doudova@example.cz      | +420 606 111 999 | osoba-032                             | –                                |

### 2.3 Děti — registrovaní členové oddílu Severka

Věk = k 31.12. letošního roku (výchozí `age_at_year_end`).

| ID        | Jméno               | Narození   | Věk | Družina | Zástupci             |
| --------- | ------------------- | ---------- | --- | ------- | -------------------- |
| osoba-020 | Anežka Konvalinková | 14.5.2013  | 13  | Sovy    | osoba-010, osoba-011 |
| osoba-021 | Vojtěch Konvalinka  | 2.9.2016   | 10  | Lišky   | osoba-010, osoba-011 |
| osoba-022 | Šimon Šálek         | 21.1.2012  | 14  | Sovy    | osoba-012            |
| osoba-023 | Rozálie Šálková     | 8.11.2014  | 12  | Lišky   | osoba-012            |
| osoba-024 | Matyáš Brázdil      | 30.6.2011  | 15  | Sovy    | osoba-013            |
| osoba-025 | Johana Hrbáčková    | 17.4.2015  | 11  | Lišky   | osoba-014            |
| osoba-026 | Kryštof Hrbáček     | 3.12.2018  | 8   | Lišky   | osoba-014            |
| osoba-027 | Teodor Kučeravý     | 25.7.2010  | 16  | Sovy    | osoba-015            |
| osoba-028 | Amálie Peštová      | 9.2.2013   | 13  | Sovy    | osoba-016            |
| osoba-029 | Norbert Pešta       | 19.10.2017 | 9   | Lišky   | osoba-016            |
| osoba-030 | Klaudie Vrabcová    | 5.3.2012   | 14  | Sovy    | osoba-017            |
| osoba-031 | Hubert Slanina      | 28.8.2014  | 12  | Lišky   | osoba-018            |
| osoba-032 | Melichar Douda      | 11.6.2009  | 17  | Sovy    | osoba-019            |

### 2.4 Hosté a osoby mimo Severku

| ID        | Jméno            | Typ                                  | Pozn.                                                                            |
| --------- | ---------------- | ------------------------------------ | -------------------------------------------------------------------------------- |
| osoba-040 | Břetislav Okurka | host (dospělý), oddil-01             | dobrovolník; bretislav.okurka@example.cz, +420 607 121 212                       |
| osoba-041 | Sára Mlžná       | host (dítě), oddil-01                | **datum narození neuvedeno** — blokuje vyhodnocení přihlášky (viz prihlaska-209) |
| osoba-042 | Karolína Mlžná   | zástupkyně osoby-041                 | karolina.mlzna@example.cz, +420 608 232 323; bez vazby na oddíl                  |
| osoba-050 | Věra Kropáčková  | HVO oddil-02                         | vera.kropackova@example.cz                                                       |
| osoba-051 | Denis Kropáček   | člen oddil-02, nar. 6.6.2014 (12)    |                                                                                  |
| osoba-052 | Stela Vydrová    | členka oddil-02, nar. 1.10.2012 (14) |                                                                                  |
| osoba-053 | Bohumil Ježek    | HVO oddil-03                         | bohumil.jezek@example.cz                                                         |
| osoba-054 | Nikol Ježková    | členka oddil-03, nar. 15.2.2013 (13) |                                                                                  |

### 2.5 Osoby pro plochy C a D

| ID        | Jméno                | Typ / oddíl                   | Narození      | Pozn.                                                                                                                                                                          |
| --------- | -------------------- | ----------------------------- | ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| osoba-033 | Barbora Konvalinková | bývalá členka oddil-01        | 3.4.2008 (18) | záznam neaktivní (mimo výchozí filtr evidence osob); bez družiny a přihlášek; kontaktní e-mail **neuveden**; zástupci osoba-010 a osoba-011 — vazby `readonly_after_adulthood` |
| osoba-034 | Jitka Konvalinková   | zástupkyně (babička)          | —             | **vzniká až přijetím pozvánky** pozvanka-901 (rezervované ID); jitka.konvalinkova@example.cz                                                                                   |
| osoba-060 | Nikol Ježková        | členka oddil-02               | 15.2.2013     | duplicita osoby-054; přezdívka „Niki“; adresa Sadová 8, Olomouc; pojišťovna 205                                                                                                |
| osoba-061 | Sára Mlžná           | členka oddil-02               | 3.6.2015      | slabý kandidát k osobě-041 (ta je bez data narození)                                                                                                                           |
| osoba-062 | Denis Kropáček       | host oddil-03                 | 6.6.2014      | duplicita osoby-051; dvojice potlačena (merge-903)                                                                                                                             |
| osoba-063 | Amálie Peštová       | členka oddil-02               | 9.2.2013      | duplicita osoby-028 — pár pro reportovací sloučení (report R9)                                                                                                                 |
| osoba-064 | Pavla Konvalinková   | host (dospělá), oddil-02      | 7.9.1985      | duplicitní kandidát k osobě-010; adresa Wolkerova 18, 779 00 Olomouc; e-mail pavla.k@example.cz; bez účtu, bez přihlášek                                                       |
| osoba-070 | Bohdana Krejcárková  | administrátorka ústředí (ADM) | —             | bohdana.krejcarkova@example.cz; účet ✔; demo persona „Administrátorka ústředí“                                                                                                 |

Vazby a rozšířená pole: **osoba-053 je zákonný zástupce osoby-054**; osoba-054 má adresu Budějovická 14, Písek a pojišťovnu 111. Osoby 060–063 nemají aktivního rodiče (za dítě jedná HVO oddílu). Pro plochu D navíc: **osoba-010** nar. 7.9.1985, adresa Jabloňová 412, 460 01 Liberec, pojišťovna 111, pohlaví žena; **osoba-020** táž adresa, pojišťovna 111; **osoba-021** táž adresa, pojišťovna 211.

### 2.6 Uživatelské účty

Účty existují právě pro osoby označené ✔ ve sloupci Účet.

| ID                  | Osoba                               | Přihlašovací e-mail            | Metody přihlášení                                 |
| ------------------- | ----------------------------------- | ------------------------------ | ------------------------------------------------- |
| ucet-001 … ucet-005 | osoba-001 … osoba-005 (tým Severky) | e-maily dle § 2.1              | heslo                                             |
| ucet-010            | osoba-010 Pavla Konvalinková        | pavla.konvalinkova@example.cz  | heslo + Google — **demo persona „Rodič s účtem“** |
| ucet-070            | osoba-070 Bohdana Krejcárková       | bohdana.krejcarkova@example.cz | heslo — demo persona ADM                          |

### 2.7 Vazby rodič–dítě (plocha D)

Stavy vazby: `active` / `cancelled` / `readonly_after_adulthood`. Vazby zde neuvedené (ostatní rodiny z § 2.2) jsou všechny `active`; vazby vznikající za běhu dema (schválení `tok-schvaleni-205`, přijetí pozvanka-901) se zakládají `active`.

| ID        | Rodič     | Dítě      | Stav                     |
| --------- | --------- | --------- | ------------------------ |
| vazba-801 | osoba-010 | osoba-020 | active                   |
| vazba-802 | osoba-010 | osoba-021 | active                   |
| vazba-803 | osoba-011 | osoba-020 | active                   |
| vazba-804 | osoba-011 | osoba-021 | active                   |
| vazba-805 | osoba-010 | osoba-033 | readonly_after_adulthood |
| vazba-806 | osoba-011 | osoba-033 | readonly_after_adulthood |

## 3. Šablony akcí

| ID          | Název              | Typ                       | Původ               | Přednastavuje                                                                                                               |
| ----------- | ------------------ | ------------------------- | ------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| sablona-401 | Letní tábor        | Víkendovky / jednoosobové | systémová           | dokumenty: posudek + souhlas s fotografováním; splatnost 14 dní; storna −30/−7 dní (50 %/100 %); kapacita 30 + 5 náhradníků |
| sablona-402 | Víkendovka Severky | Víkendovky / jednoosobové | oddílová (oddil-01) | posudek; splatnost 14 dní; kapacita 24 + 4                                                                                  |
| sablona-403 | Závod Stezka       | Stezka                    | systémová           | hlídky zapnuté (kategorie Stezka, Pěšinka); bez dokumentů; jednotná cena                                                    |
| sablona-404 | Jednorázová akce   | Jednorázové akce          | systémová           | bez přihlášek, bez ceny; docházka                                                                                           |
| sablona-405 | Pravidelné schůzky | Pravidelné schůzky        | systémová           | bez přihlášek; opakovaná docházka                                                                                           |

## 4. Akce

SS identifikuje akci, VS přihlášku (párování plateb). Rok dat: aktuální (2026), nezobrazuje se.

| ID       | Název                           | Oddíl    | Šablona     | Termín        | Přihlašování      | Kapacita     | Stav dema                                                                                    |
| -------- | ------------------------------- | -------- | ----------- | ------------- | ----------------- | ------------ | -------------------------------------------------------------------------------------------- |
| akce-101 | Letní tábor Stříbrná zátoka     | oddil-01 | sablona-401 | 11.–25.7.     | 1.3.–15.6.        | 30 + 5 náhr. | **proběhlá, po uzávěrce**; docházka zapsaná; zbývá dořešit platby (PartialPaid, Overpayment) |
| akce-102 | Podzimní víkendovka Skalní mlýn | oddil-01 | sablona-402 | 25.–27.9.     | 20.8.–18.9.       | 24 + 4       | **otevřená registrace** — hlavní živá akce dema                                              |
| akce-103 | Závod Stezka — Podzimní stopa   | oddil-01 | sablona-403 | Sobota 10.10. | 10.8.–30.9.       | 6 + 3        | **plná kapacita, náhradníci čekají**                                                         |
| akce-104 | Výlet na Kozí vrch              | oddil-01 | sablona-404 | Sobota 22.8.  | — (bez přihlášek) | —            | proběhlá jednodenní akce; jen docházka                                                       |
| akce-105 | Drakiáda na kopci Větrník       | oddil-02 | sablona-401 | Sobota 17.10. | 15.8.–15.10.      | 40 + 0       | otevřená, **zdarma** (přihláška jde rovnou do Paid)                                          |
| akce-106 | Vánoční dílny                   | oddil-01 | sablona-402 | Sobota 5.12.  | od 1.11.          | 20 + 0       | **přihlašování ještě neotevřeno** (zobrazit datum otevření)                                  |

**Detail akce-102 (referenční):** SS `2026102`; viditelnost veřejná; místo „Skalní mlýn, Jizerské hory“; ceny: člen DU 850 Kč · bez DU 950 Kč · vedoucí/dobrovolník 400 Kč; splatnost 14 dní od podání; storno: do 11.9. zdarma, 12.–18.9. 50 %, později 100 %; povinný dokument: **Posudek o zdravotní způsobilosti** (PDF/JPG/PNG/HEIC, max 10 MB); výběrové číselníky: _Strava_ (běžná +0 Kč · bezlepková +50 Kč) a _Doprava_ (společný autobus +120 Kč · vlastní +0 Kč). Výsledná cena = základní cena dle typu účastníka + příplatky zvolených položek.

**Detail akce-103:** SS `2026103`; jednotná cena 150 Kč; bez dokumentů; hlídky dle § 8. **Detail akce-101:** SS `2026101`; ceny do 31.5.: člen DU 3 900 Kč / bez DU 4 400 Kč; poté 4 200 / 4 700 Kč; dokumenty: posudek + souhlas s fotografováním.

## 5. Přihlášky

Stavy přesně dle životního cyklu: `New` (nová) · `PendingGuardian` (čeká na zákonného zástupce) · `PendingDocuments` (čeká na dokumenty) · `PendingPayment` (čeká na platbu) · `PartialPaid` (částečně zaplaceno) · `Paid` (zaplaceno) · `Overpayment` (přeplatek) · `Canceled` (stornována) · `Expired` (expirovaná). VS = variabilní symbol přihlášky.

### 5.1 Akce-102 · Podzimní víkendovka (otevřená)

| ID            | Osoba                    | Stav               | VS       | Cena                         | Pozn.                                                                                                                    |
| ------------- | ------------------------ | ------------------ | -------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| prihlaska-201 | osoba-020 Anežka K.      | `Paid`             | 26102201 | 970 Kč (850 + autobus 120)   | podáno Pátek 21.8. 08:15; zástupce schválil, posudek schválen, zaplaceno — **tokenový odkaz `tok-demo-rodic`**           |
| prihlaska-202 | osoba-021 Vojtěch K.     | `PendingPayment`   | 26102202 | 970 Kč                       | posudek schválen; splatnost Pátek 4.9.                                                                                   |
| prihlaska-203 | osoba-022 Šimon Š.       | `PendingDocuments` | 26102203 | 850 Kč                       | posudek nahrán, čeká na posouzení (dok-701)                                                                              |
| prihlaska-204 | osoba-023 Rozálie Š.     | `PendingDocuments` | 26102204 | 900 Kč (850 + bezlepková 50) | posudek **zamítnut** (dok-702), čeká na nové nahrání — token `tok-stav-204`                                              |
| prihlaska-205 | osoba-024 Matyáš B.      | `PendingGuardian`  | 26102205 | 850 Kč                       | žádost zástupci odeslána Pondělí 24.8. 17:03, lhůta do Pondělí 31.8. — schvalovací token `tok-schvaleni-205` (osoba-013) |
| prihlaska-206 | osoba-025 Johana H.      | `PendingGuardian`  | 26102206 | 970 Kč                       | odeslána Čtvrtek 20.8.; lhůta do Čtvrtek 27.8. — **zítra vyprší**; token `tok-schvaleni-206` (osoba-014)                 |
| prihlaska-207 | osoba-026 Kryštof H.     | `PendingDocuments` | 26102207 | 970 Kč                       | posudek čeká na posouzení (dok-705)                                                                                      |
| prihlaska-208 | osoba-027 Teodor K.      | `Paid`             | 26102208 | 950 Kč                       | bez DU                                                                                                                   |
| prihlaska-209 | osoba-041 Sára M. (host) | `New`              | 26102209 | 850 Kč                       | **chybí datum narození** — bránu zástupce nelze vyhodnotit; systém si datum vyžádal e-mailem                             |
| prihlaska-210 | osoba-028 Amálie P.      | `PendingPayment`   | 26102210 | 850 Kč                       | splatnost Úterý 8.9.                                                                                                     |
| prihlaska-211 | osoba-030 Klaudie V.     | `Canceled`         | 26102211 | 850 Kč                       | storno zástupkyní Sobota 22.8. — před 11.9., poplatek 0 Kč                                                               |
| prihlaska-212 | osoba-040 Břetislav O.   | `PendingPayment`   | 26102212 | 400 Kč (dobrovolník)         | platba dorazila **bez VS** — čeká na ruční spárování (trans-307)                                                         |

Do kapacity (24) se počítá 8 přihlášek (stavy PendingDocuments–Paid) → **volno**.

### 5.2 Akce-103 · Závod Stezka (plná kapacita)

| ID            | Osoba                | Stav                   | VS       | Pozn.                                                                                                         |
| ------------- | -------------------- | ---------------------- | -------- | ------------------------------------------------------------------------------------------------------------- |
| prihlaska-220 | osoba-020 Anežka K.  | `Paid`                 | 26103220 |                                                                                                               |
| prihlaska-221 | osoba-022 Šimon Š.   | `Paid`                 | 26103221 |                                                                                                               |
| prihlaska-222 | osoba-024 Matyáš B.  | `Paid`                 | 26103222 |                                                                                                               |
| prihlaska-223 | osoba-027 Teodor K.  | `PendingPayment`       | 26103223 | splatnost Středa 9.9.                                                                                         |
| prihlaska-224 | osoba-028 Amálie P.  | `PendingPayment`       | 26103224 |                                                                                                               |
| prihlaska-225 | osoba-030 Klaudie V. | `Paid`                 | 26103225 |                                                                                                               |
| prihlaska-226 | osoba-021 Vojtěch K. | `New` — náhradník č. 1 | 26103226 | **aktivní nabídka místa**: odeslána Úterý 25.8. 14:00, platí do Čtvrtek 27.8. 14:00 — token `tok-nabidka-226` |
| prihlaska-227 | osoba-026 Kryštof H. | `New` — náhradník č. 2 | 26103227 | čeká v pořadí; brány zamčené                                                                                  |
| prihlaska-228 | osoba-031 Hubert S.  | `Expired`              | 26103228 | zástupce neschválil do 7 dnů                                                                                  |
| prihlaska-229 | osoba-029 Norbert P. | `Canceled`             | 26103229 | storno vedoucím (kolize termínu)                                                                              |

Do kapacity (6) se počítá 6 → **plno**; CTA na portálu „Přihlásit se jako náhradník“.

### 5.3 Akce-101 · Letní tábor (proběhlá — dořešení plateb)

| ID            | Osoba               | Stav          | VS       | Cena     | Pozn.                                                                                                    |
| ------------- | ------------------- | ------------- | -------- | -------- | -------------------------------------------------------------------------------------------------------- |
| prihlaska-240 | osoba-020 Anežka K. | `Paid`        | 26101240 | 3 900 Kč |                                                                                                          |
| prihlaska-241 | osoba-022 Šimon Š.  | `Paid`        | 26101241 | 3 900 Kč |                                                                                                          |
| prihlaska-242 | osoba-024 Matyáš B. | `PartialPaid` | 26101242 | 4 400 Kč | zaplaceno 2 000 Kč, **zbývá 2 400 Kč**                                                                   |
| prihlaska-243 | osoba-027 Teodor K. | `Overpayment` | 26101243 | 3 900 Kč | zaplaceno 4 000 Kč → **přeplatek 100 Kč**, čeká na rozhodnutí ÚČE (vrátit / převést / ponechat jako dar) |
| prihlaska-244 | osoba-025 Johana H. | `Paid`        | 26101244 | 3 900 Kč |                                                                                                          |
| prihlaska-245 | osoba-028 Amálie P. | `Paid`        | 26101245 | 3 900 Kč |                                                                                                          |

### 5.4 Akce-105 · Drakiáda (zdarma, oddil-02)

| ID            | Osoba                            | Stav   | Pozn.                                                 |
| ------------- | -------------------------------- | ------ | ----------------------------------------------------- |
| prihlaska-250 | osoba-051 Denis K.               | `Paid` | cena 0 Kč → rovnou `Paid`                             |
| prihlaska-251 | osoba-052 Stela V.               | `Paid` | dtto                                                  |
| prihlaska-252 | osoba-063 Amálie P. (duplicitní) | `Paid` | drží pár pro reportovací sloučení v reportu R9 (§ 10) |
| prihlaska-253 | osoba-060 Nikol J. (duplicitní)  | `Paid` | při provedení merge-901 přechází na osobu-054 (§ 10)  |

## 6. Dokumenty (fronta posuzování — akce-102)

| ID      | Přihláška     | Dokument                         | Soubor                       | Stav              | Detail                                                                                           |
| ------- | ------------- | -------------------------------- | ---------------------------- | ----------------- | ------------------------------------------------------------------------------------------------ |
| dok-701 | prihlaska-203 | Posudek o zdravotní způsobilosti | posudek-simon.pdf (412 kB)   | čeká na posouzení | nahráno Pondělí 24.8. 19:42                                                                      |
| dok-702 | prihlaska-204 | Posudek o zdravotní způsobilosti | posudek-rozalie.jpg (2,1 MB) | zamítnut          | Neděle 23.8., důvod: „Sken je nečitelný a chybí razítko lékaře. Nahrajte prosím čitelnou kopii.“ |
| dok-703 | prihlaska-201 | Posudek o zdravotní způsobilosti | posudek-anezka.pdf (388 kB)  | schválen          | Pátek 21.8. 10:02 (osoba-001)                                                                    |
| dok-704 | prihlaska-202 | Posudek o zdravotní způsobilosti | posudek-vojtech.pdf (395 kB) | schválen          | Pátek 21.8. 10:05                                                                                |
| dok-705 | prihlaska-207 | Posudek o zdravotní způsobilosti | IMG_2481.jpg (3,4 MB)        | čeká na posouzení | nahráno Úterý 25.8. 21:17                                                                        |

Fronta „čeká na posouzení“ má tedy 2 položky (badge na navigaci). Náhledy souborů v demu nahraď šedým placeholder rámečkem s názvem souboru.

## 7. Platby — transakce na účtu oddílu Severka (pracoviště ÚČE)

| ID        | Datum               | Částka   | VS       | SS      | Odesílatel / zpráva                    | Párování                                                                                  |
| --------- | ------------------- | -------- | -------- | ------- | -------------------------------------- | ----------------------------------------------------------------------------------------- |
| trans-301 | Pátek 21.8. 11:32   | 970 Kč   | 26102201 | 2026102 | KONVALINKOVA PAVLA                     | automaticky `vs_exact` → prihlaska-201 (970 Kč)                                           |
| trans-302 | Sobota 22.8. 09:10  | 950 Kč   | 26102208 | 2026102 | KUCERAVY ONDREJ                        | automaticky `vs_exact` → prihlaska-208                                                    |
| trans-303 | Pondělí 24.8. 08:41 | 150 Kč   | 26103220 | 2026103 | KONVALINKOVA PAVLA                     | automaticky `vs_exact` → prihlaska-220                                                    |
| trans-304 | Úterý 2.6. 14:05    | 2 000 Kč | 26101242 | 2026101 | BRAZDIL JIRI / „záloha tábor Matyáš“   | návrh `vs_partial_name`, potvrzeno → prihlaska-242 (částečná úhrada)                      |
| trans-305 | Pátek 5.6. 7:58     | 4 000 Kč | 26101243 | 2026101 | KUCERAVY ONDREJ                        | návrh `vs_overpayment_name`, potvrzeno → prihlaska-243; **přeplatek 100 Kč k rozhodnutí** |
| trans-306 | Úterý 25.8. 16:20   | 970 Kč   | 99999999 | —       | HRBACKOVA LENKA / „vikendovka Krystof“ | **nespárovaná** — VS neexistuje; kandidát dle jména: prihlaska-207                        |
| trans-307 | Úterý 25.8. 18:44   | 400 Kč   | —        | —       | OKURKA BRETISLAV                       | **nespárovaná** — bez VS; kandidát dle jména: prihlaska-212                               |
| trans-308 | Pondělí 24.8. 12:15 | 150 Kč   | 26103225 | 2026103 | VRABCOVA ZUZANA                        | automaticky `ss_exact_name` → prihlaska-225                                               |

Fronta ÚČE tedy obsahuje: **2 nespárované transakce** (trans-306, trans-307) a **1 přeplatek k rozhodnutí** (prihlaska-243). Částky se párují **přesně, bez tolerance**. Pro ostatní přihlášky ve stavu `Paid`, které v tabulce nemají transakci, vygeneruj při inicializaci store odpovídající spárovanou transakci (částka = cena, VS = VS přihlášky, metoda `vs_exact`, datum před dneškem). Vratka/převod přeplatku se eviduje jako záporná alokace (`refund`) na původní transakci.

## 8. Družiny, hlídky, docházka

**Družiny (oddil-01):** druzina-601 **Lišky** (vede osoba-003; členové: osoba-021, 023, 025, 026, 029, 031) · druzina-602 **Sovy** (vede osoba-002; členové: osoba-020, 022, 024, 027, 028, 030, 032).

**Hlídky akce-103** (skládá vlastník přihlášky z osob svých potvrzených přihlášek; název unikátní v rámci akce):

| ID         | Název  | Kategorie | Kapitán   | Členové                         |
| ---------- | ------ | --------- | --------- | ------------------------------- |
| hlidka-501 | Rysové | Stezka    | osoba-024 | osoba-024, osoba-022, osoba-020 |
| hlidka-502 | Vydry  | Pěšinka   | osoba-028 | osoba-028, osoba-030            |

Stanoviště závodu: stanoviste-01 „Uzly“ (rozhodčí osoba-002) · stanoviste-02 „Mapa a buzola“ (osoba-003) · stanoviste-03 „První pomoc“ (rozhodčí zatím nepřiřazen — prázdný slot v UI).

**Docházka akce-104 (Výlet na Kozí vrch, Sobota 22.8.):** přítomni osoba-020, 021, 022, 025, 026, 028, 029; nepřítomni osoba-024 a osoba-027; ostatní bez záznamu (nezapsáni). Vedl osoba-003, zapsáno Sobota 22.8. 17:40. **Dobrovolnické hodiny akce-101:** osoba-040 Břetislav Okurka — 32 h (kuchyně), osoba-004 Eliška Rákosová — 20 h (program).

## 9. Struktura ústředí — regiony, pozvánka HVO, whitelist (plocha C)

### 9.1 Regiony a příslušnosti

| ID         | Název         | Stav                 | Nástupce   | Aktuální oddíly         |
| ---------- | ------------- | -------------------- | ---------- | ----------------------- |
| region-801 | Region Sever  | aktivní              | —          | oddil-01 (od 1.1.2019)  |
| region-802 | Region Morava | aktivní              | —          | oddil-02 (od 12.1.2025) |
| region-803 | Region Jih    | aktivní              | —          | oddil-03 (od 1.1.2019)  |
| region-804 | Region Východ | sloučený (12.1.2025) | region-802 | —                       |
| region-805 | Region Haná   | sloučený (12.1.2025) | region-802 | —                       |

Historie příslušností: oddil-02 — Region Haná 1.1.2019–12.1.2025, poté Morava. Snapshot regionu na akcích (region akce = snapshot z okamžiku její první publikace): akce oddílu-01 → Sever; akce-105 → Morava.

### 9.2 Pozvánka HVO a whitelist jmen

| ID          | Co                          | Detail                                                                                                                                         |
| ----------- | --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| pozvanka-01 | pozvánka HVO pro oddil-04   | Pondělí 17.8. 10:02 na `viktor.rovny@exmaple.cz` (překlep záměrně) → **trvale nedoručeno**; správně `viktor.rovny@example.cz`; platnost 14 dní |
| vyjimka-01  | oddílová výjimka whitelistu | jméno „Melichar“, oddil-01, schválil osoba-001, Pondělí 2.3.                                                                                   |

Whitelist křestních jmen: naplnit vzorkem běžných českých jmen (min. všechna křestní jména osob z § 2 kromě „Melichar“, který je výjimkou).

## 10. Slučování osob a reportovací sloučení (plocha C)

### 10.1 Žádosti o sloučení osob

| ID        | Druh | Osoby (cíl × zdroj)   | Stav                           | Strany a rozhodnutí                                                                    | Založena     | Propadne      |
| --------- | ---- | --------------------- | ------------------------------ | -------------------------------------------------------------------------------------- | ------------ | ------------- |
| merge-901 | dítě | osoba-054 × osoba-060 | `ready`                        | osoba-053 (rodič) ✔ Čtvrtek 20.8. 18:12 · osoba-050 (HVO oddil-02) ✔ Pátek 21.8. 09:30 | Středa 19.8. | Pátek 18.9.   |
| merge-902 | dítě | osoba-041 × osoba-061 | `pending`                      | osoba-050 (HVO oddil-02) ✔ Sobota 22.8. 10:15 · osoba-042 (rodička) nerozhodla         | Úterý 18.8.  | Čtvrtek 17.9. |
| merge-903 | dítě | osoba-051 × osoba-062 | `rejected` — potlačená dvojice | zamítl osoba-053 (HVO oddil-03) Čtvrtek 6.8.                                           | Pondělí 3.8. | —             |

Konflikty merge-901: jméno, příjmení a datum narození shodné; přezdívka auto (vyplněná strana vyhrává — jen zdroj); **adresa** a **pojišťovna** volba A/B (hodnoty viz § 2.5 a pozn. pod ní). Žádná blokující kolize přihlášek.

### 10.2 Report R9 — Unikátní děti

R9 za rok 2026: základ **14 unikátních dětí** (Severka 10: osoby 020–028 a 030; Jestřábi 4: 051, 052, 060, 063); po reportovacím sloučení 028 × 063 → **13**. Výchozí stav: žádné aktivní reportovací sloučení, 3 kandidáti (Amálie Peštová 9.2.2013 · Nikol Ježková 15.2.2013 · Denis Kropáček 6.6.2014).

## 11. Pozvánky zástupců, souhlasy a sloučení duplicit (plocha D)

### 11.1 Pozvánky druhého zástupce

| ID           | Dítě                 | Zve       | E-mail pozvaného              | Odeslána            | Platí do     | Stav                                       |
| ------------ | -------------------- | --------- | ----------------------------- | ------------------- | ------------ | ------------------------------------------ |
| pozvanka-901 | osoba-021 Vojtěch K. | osoba-010 | jitka.konvalinkova@example.cz | Pondělí 24.8. 10:12 | Pondělí 7.9. | čeká na přijetí — token `tok-pozvanka-901` |

### 11.2 Souhlasy (osoba-010)

| ID          | Typ / účel                                               | Uděleno   | Stav                                          |
| ----------- | -------------------------------------------------------- | --------- | --------------------------------------------- |
| souhlas-921 | zpracování osobních údajů — členská evidence a přihlášky | 12.3.2024 | platí                                         |
| souhlas-922 | zasílání novinek e-mailem                                | 12.3.2024 | odvolán 5.1. (uchovává se 4 roky po odvolání) |

### 11.3 Sloučení duplicit vlastního účtu

Nabídka kandidáta: dvojice **osoba-010 × osoba-064**, důvod „shodné jméno a datum narození“, výchozí stav **nabídnuto** (žádost zatím nepodána). Rezervované ID žádosti: **zadost-941**; strany po podání: iniciátorka osoba-010 (souhlas okamžitě) a HVO oddil-02 osoba-050 Věra Kropáčková (kandidát účet nemá). Konfliktní pole pro volbu A/B: adresa, kontaktní e-mail; pojišťovna bez konfliktu (prázdné pole prohrává — bere se 111).

## 12. Tokenové odkazy dema

| Token               | Vede na                                           | Kontext                                                  |
| ------------------- | ------------------------------------------------- | -------------------------------------------------------- |
| `tok-demo-rodic`    | `/stav/tok-demo-rodic` — rozcestník prihlaska-201 | šťastná cesta: vše hotovo, stav Zaplaceno                |
| `tok-stav-204`      | rozcestník prihlaska-204                          | zamítnutý dokument → nové nahrání                        |
| `tok-schvaleni-205` | `/schvaleni/tok-schvaleni-205`                    | zástupce osoba-013 schvaluje přihlášku Matyáše           |
| `tok-schvaleni-206` | schvalovací stránka prihlaska-206                 | lhůta vyprší zítra (urgentní stav)                       |
| `tok-nabidka-226`   | `/nabidka/tok-nabidka-226`                        | nabídka místa náhradníkovi, platí do Čtvrtek 27.8. 14:00 |
| `tok-pozvanka-901`  | `/pozvanka/tok-pozvanka-901`                      | přijetí pozvánky druhého zástupce (Jitka K. → Vojtěch)   |

Platební sekce rozcestníků zobrazuje QR kód platby (v demu statický placeholder QR obrázek) s údaji: účet `2900123456/2010`, částka, VS přihlášky, SS akce, zpráva „DU – [název akce] – [jméno]“.
