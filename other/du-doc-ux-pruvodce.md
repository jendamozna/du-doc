*UX průvodce · projekt du-doc · 26. 8. 2026*

# du-doc — UX průvodce

UX specifikace registračního systému **Dorostové unie (DU)** — orientace v projektu, uživatelská flow a specifikace obrazovek. Vychází ze specifikace projektu du-doc.

> **O tomto dokumentu**
> Místa, kde specifikace mlčí a je potřeba rozhodnutí zadavatele, jsou v celém dokumentu označena badgem **[K rozhodnutí]**; každý doménový fakt nese citační štítek jako (zdroj: README.md → Rozsah) a odvozené úsudky štítek *(odvozeno)*.

## 01 · Orientace v projektu

*Co systém je, komu a jak pomáhá a co už je rozhodnuto*

### CO? PROČ? JAK?

#### CO — účel systému

Webový **systém přihlášek na akce a členské evidence pro oddíly Dorostové unie (DU)**. Organizace má tři vrstvy: ústředí (ústředí), regiony a oddíly. Ústředí zastřešuje všechny oddíly, vede společnou členskou databázi a pořádá celostátní akce; každý oddíl si spravuje vlastní akce, přihlášky a účastníky. Osoba je nezávislá entita — může patřit do více oddílů současně. (zdroj: README.md → Přehled projektu)

Rozsah tvoří čtyři věci: **veřejný registrační portál**, **oddílová správa akcí**, **správa ústředí** a **self-management pro registrované**. (zdroj: README.md → Rozsah)

Praktický kontext pro UX: primární cesta peněz i stresu je **rodič, který přihlašuje dítě na akci a platí bankovním převodem s QR kódem**. Platby se párují automaticky proti bankovním transakcím (synchronizace s Fio bankou). Vše je výhradně česky — rozhraní, e-maily, měna (CZK) i formáty data jsou předepsané. (zdroj: docs/non-functional.md → Lokalizace) (zdroj: docs/payment-matching.md)

*(odvozeno)* Spec nedefinuje značku, tón ani vizuální identitu Dorostové unie — zatím neexistují. Struktura organizace (oddíly, družiny, vedoucí, rádci, dobrovolníci, tábory, závody) silně připomíná české skautské či turistické dětské organizace — to je užitečné pro empatii, ale je to můj úsudek, ne fakt ze spec.

#### PROČ — jaký problém komu řeší

> **Poctivé upozornění**
> Spec je psaná jako „**co** systém dělá“, nikoli „**proč** existuje“ — explicitní formulace problému a motivace v ní chybí (sama o sobě říká jen, že popisuje „co systém dělá a proč“ pro zadavatele, hlavní vedoucí, účetní a právníky (zdroj: README.md → Implementační dokumentace)). Motivace níže je proto *(odvozeno)* z mechanismů, které systém staví.

- **Administrativa přihlášek s právními náležitostmi.** Souhlas zákonného zástupce s lhůtou, povinné dokumenty se schvalováním, storno poplatky podle termínů — systém hlídá pořadí a úplnost místo vedoucích, kteří jsou dobrovolníci. (zdroj: docs/registration-lifecycle.md)
- **Ruční párování plateb.** Přesné částky, VS/SS symboly, párování M:N („rodič zaplatil za tři děti jedním převodem“), automatická potvrzení o platbě — celý modul míří na práci, kterou dnes někdo dělá ručně nad bankovním výpisem *(odvozeno)*. (zdroj: docs/payment-matching.md)
- **Doložitelnost pro dotace a kontroly.** Retenční lhůty se výslovně opírají o dotační pravidla (MŠMT, 10 let), zákon o účetnictví a o DPH; reporty ústředí počítají unikátní děti napříč všemi oddíly s dimenzí regionu. (zdroj: README.md → Retence a GDPR) (zdroj: docs/reports.md)
- **GDPR povinnosti.** Anonymizace místo mazání, automatické retenční joby, doklad o výmazu, citlivá data izolovaná per oddíl. (zdroj: README.md → Retence a GDPR)
- **Roztříštěná evidence osob.** Jedna osoba ve více oddílech, deduplikace podle jmen a data narození, vícestranné schvalování sloučení. (zdroj: README.md → Deduplikace osob, merge)

**[K rozhodnutí]** — přesněji: **otázka pro zadavatele**. Spec neříká, *co systém nahrazuje* — papírové přihlášky? sdílené tabulky? starší systém? — ani jaké jsou dnešní největší bolesti jednotlivých rolí. Pro UX research je to první otázka: bez ní nevíme, s čím budou uživatelé nový systém srovnávat a kde je laťka. Možnosti, jak to zjistit: krátké rozhovory se zadavatelem + jedním HVO a jedním účetním, nebo aspoň písemný dotazník před návrhem prvních obrazovek.

#### JAK pomáhá — po skupinách uživatelů

*Rodič / zákonný zástupce*

**Přihlásit dítě a mít klid**

- Přihlásí jedno i více dětí na akci; schválí přihlášku nezletilého **odkazem z e-mailu bez zakládání účtu**. (zdroj: README.md → Přihlašování na akce)
- Zaplatí QR kódem s přesnou částkou a VS; systém platbu sám spáruje a pošle potvrzení. (zdroj: docs/payment-matching.md)
- Sleduje stav přihlášky, nahrává a opravuje dokumenty, provádí storno, spravuje údaje dětí (adresy, pojišťovny). (zdroj: README.md → Rodič)
- Pozve druhého zákonného zástupce; po zletilosti dítěte přechází vazba do režimu jen pro čtení. (zdroj: README.md → Rodič)

*Účastník / host*

**Účast bez povinného účtu**

- Přihlásí se na akci **bez účtu** — dostane tokenizovaný odkaz, kterým přihlášku spravuje (storno, změny, přidání dalších účastníků). (zdroj: README.md → Přihlašování na akce) (zdroj: docs/non-functional.md → Tokeny)
- Kdykoli později si z odkazu založí účet (heslo nebo Google/Facebook OAuth) a účet se propojí s jeho existující osobou. (zdroj: docs/non-functional.md → OAuth)
- Jako náhradník dostane časově omezenou nabídku uvolněného místa. (zdroj: docs/registration-lifecycle.md → substitute.*)

*Vedoucí oddílu (HVO, VO, VD, RÁD)*

**Akce a lidé bez papírování**

- Zakládá akce ze šablon typů; nastaví ceny, storna, číselníky, dokumenty, viditelnost. (zdroj: README.md → Typy a šablony akcí)
- Vidí přihlášky a jejich stavy na jednom místě; schvaluje či zamítá dokumenty; vybírá náhradníky. (zdroj: README.md → Přihlašování na akce)
- Zapisuje docházku vč. dobrovolnických hodin — samostatné oprávnění, které může mít i rádce bez přístupu k citlivým datům. (zdroj: README.md → Docházka)
- Vede evidenci členů, hostů a družin s chytrými sloupci; čte reporty oddílu. (zdroj: README.md → Pomocná evidence) (zdroj: docs/reports.md)

*Účetní (ÚČE)*

**Platby, které se párují samy**

- Transakce se stahují automaticky z Fio banky a párují podle SS (akce) a VS (přihláška). (zdroj: docs/fio-sync.md) (zdroj: docs/payment-matching.md)
- Nespárované platby ručně rozdělí mezi více přihlášek (jedna rodinná platba za víc dětí); u přeplatku volí vrátit / převést / ponechat jako dar. (zdroj: docs/payment-matching.md)
- Systém sám posílá výzvy k platbě, připomínky splatnosti a potvrzení o každé napárované platbě. (zdroj: README.md → Modul párování plateb)

*Ústředí (ADM)*

**Celkový přehled a společná pravidla**

- Spravuje oddíly, jmenuje HVO pozvánkami e-mailem, definuje verzované regiony. (zdroj: README.md → Administrátor, Region)
- Řeší deduplikaci osob napříč oddíly a schvaluje sloučení. (zdroj: docs/person-merge.md)
- Čte reporty ústředí — unikátní počty dětí napříč oddíly, dimenze regionu — pro výkaznictví k dotacím. (zdroj: README.md → Modul reporty ústředí)
- Spravuje modul vzdělávání (kurzy, platnosti, certifikáty vedoucích), systémové šablony akcí a jmenný whitelist. (zdroj: README.md → Modul vzdělávání)

### Slovník, který nás zradí, když ho zaměníme

Spec je v terminologii záměrně přísná — a texty v našem UI musí být taky. (zdroj: README.md → Osoba vs. uživatelský účet)

**DU = Dorostová unie**

Dorostová unie (DU) je organizace, pro kterou systém vzniká — ústředí zastřešující regiony a oddíly. Členství DU je roční záznam členství v organizaci, nezávislý na stavu osoby v oddílu.

**Registrace = založení účtu**

„Registrace“ v tomto systému znamená **založení uživatelského účtu** (přihlašovací identity). Samotné přihlášení znamená výhradně **login do systému** — nic jiného.

**Přihláška na akci = účast**

Účast na akci je vždy **přihláška na akci** — záznam per osoba a akce s vlastním životním cyklem. „Přihlásit se na akci“ = vytvořit přihlášku.

**Osoba ≠ účet**

**Osoba** (datový subjekt — může být dítě nebo host bez loginu) je oddělená od **účtu** (přihlašovací identita). Jedna osoba má nejvýše jeden účet. Děti a hosté běžně existují úplně bez účtu.

**Host vs. registrovaný člen**

Host a registrovaný člen jsou dva stavy jedné osoby v rámci oddílu; kolmo na ně stojí stav záznamu (aktivní / neaktivní / archivovaný). „Členství DU“ je ještě něco jiného — záznam per rok, ne stav osoby. (zdroj: README.md → Stav osoby)

Důsledek pro UX: polovina našich uživatelů bude jednat **za někoho jiného** (rodič za dítě, vedoucí za hosta), a často **bez jediného přihlášení do systému** — přihlášky lze spravovat tokenizovanými odkazy z e-mailu. (zdroj: docs/non-functional.md → Tokeny)

### Kdo je kdo — aktéři a role

Role jsou vázané na oddíl a jeden člověk jich může držet několik najednou (administrátor, který je zároveň vedoucí a rodič). (zdroj: README.md → Role)

| Aktér | Pojem ve spec | Co dělá (dle spec) |
|---|---|---|
| **ADM** — Administrátor | Administrátor | Agenda ústředí: spravuje oddíly a přiřazuje jim hlavní vedoucí (pozvánka e-mailem), definuje regiony, řídí deduplikaci osob, reporty ústředí, katalog kurzů, systémové šablony akcí a jmenný whitelist. |
| **HVO** — Hlavní vedoucí oddílu | Hlavní vedoucí oddílu | Vede oddíl: nastavuje bankovní účty, zve účetní/vedoucí/rádce, definuje družiny, vede evidenci členů a hostů, zakládá akce, nahrává pověření od staršovstva, zapíná volitelné moduly. |
| **VO / VD** — Vedoucí oddílu / Vedoucí družiny | Vedoucí oddílu / Vedoucí družiny | Nemají pevná globální práva — oprávnění se přidělují **per akce** (úprava akce, úprava přihlášek, úprava cen a storen, zápis docházky) nebo v rámci družiny. Samo přiřazení k akci dává čtení jejích přihlášek. |
| **RÁD** — Rádce | Rádce | Nezletilí pomocníci. Výslovně **nesmí** vidět citlivá data dětí. Mohou na akci dostat samostatné oprávnění zápisu docházky, aniž vidí přihlášky nebo platby. |
| **ÚČE** — Účetní | Účetní | Přístup omezený na: úpravy přihlášek, čtení akcí/cen/storen/bankovních účtů a párování & potvrzování plateb a výzvy k platbě. |
| **Rodič / zákonný zástupce** | Rodič / zákonný zástupce | **Není přidělovaná role** — odvozuje se z aktivní vazby rodič ↔ dítě, práva vždy per dítě. Přihlašuje děti, spravuje jejich přihlášky, storna, platby a osobní údaje. Vazba vzniká přihlášením dítěte (nebo schválením zástupcem), mohou ji sdílet oba zástupci a po zletilosti dítěte přechází do režimu jen pro čtení. |
| **Účastníci a hosté** | účastníci / hosté | Lidé, pro které akce jsou. Nemusí mít účet: kdo se přihlásí na akci bez účtu, dostane tokenizovaný odkaz na správu přihlášky (storno, změny, přidání účastníků) a možnost později si založit účet. |

> **Známá mezera**
> Spec sama označuje **role a oprávnění za svou největší mezeru** — chybí matice akce × role; „práva se přidělují u akce“ je zatím celý příběh. Počítejme s tím, že to na konci posune některá UI rozhodnutí. (zdroj: TODO.md → Role a oprávnění (🔴 Gap))

### Čtyři plochy uživatelského rozhraní

*Plocha A · veřejná, bez loginu*

**Veřejný registrační portál**

- Procházení veřejných akcí; vnitřní akce po přihlášení s vazbou na oddíl; neveřejné akce přes sdílecí odkaz, který má každá akce. (zdroj: README.md → Viditelnost akce)
- Přihláška na akci — jádrové flow (vzorové flow A3), včetně schválení zástupcem, dokumentů a QR platby.
- Samostatná stránka pro přihlášení dobrovolníků per akce, je-li zapnutá. (zdroj: README.md → Konfigurace akce)
- Založení účtu z odkazu po přihlášce; login heslem nebo přes Google/Facebook OAuth. (zdroj: docs/non-functional.md → OAuth)

*Plocha B · HVO, VO, VD, RÁD, ÚČE*

**Oddílová správa**

- Zakládání akcí ze šablon typů (šablony); konfigurace cen, storen, výběrových číselníků, povinných dokumentů, viditelnosti, bankovního účtu. (zdroj: README.md → Konfigurace akce)
- Správa přihlášek; posuzování a schvalování/zamítání nahraných dokumentů; výběr náhradníků po uvolnění místa.
- Pracovna párování plateb pro účetní (nespárované transakce, ruční rozdělení, přeplatky). (zdroj: docs/payment-matching.md)
- Docházka vč. dobrovolnických hodin; evidence členů a hostů s chytrými sloupci (chytré sloupce); družiny; hlídky a stanoviště pro závody Stezka; reporty oddílu; nastavení oddílu a moduly.

*Plocha C · ADM*

**Správa ústředí**

- Správa oddílů a přiřazování HVO (pozvánky e-mailem); definice a verzování regionů (vznik, přesun oddílů, sloučení, rozdělení). (zdroj: README.md → Region)
- Deduplikace osob a schvalování sloučení napříč oddíly. (zdroj: docs/person-merge.md)
- Reporty ústředí — unikátní počty dětí napříč oddíly, dimenze regionu. (zdroj: README.md → Modul reporty ústředí)
- Modul vzdělávání (kurzy, platnosti, certifikáty vedoucích); systémové šablony akcí; jmenný whitelist. Navíc speciální oddíl ústředí pro celostátní akce.

*Plocha D · přihlášení uživatelé*

**Self-management**

- Moje přihlášky na akce: stav, platba, dokumenty, storno.
- Moje děti: údaje a přihlášky per dítě, pozvání druhého zákonného zástupce, po zletilosti jen pro čtení. (zdroj: README.md → Rodič)
- Moje osobní údaje (adresa, zdravotní pojišťovna…); souhlasy.
- Účet a zabezpečení: heslo, propojené OAuth identity, žádosti o sloučení, když systém najde možnou duplicitu. (zdroj: README.md → Deduplikace)

### Životní cyklus přihlášky ve zkratce

Nejdůležitější mechanismus celé spec. Přihláška na akci (přihláška) **neprochází pevným průvodcem** — její stav je **čistá funkce podmínek**, přepočítaná po každé relevantní změně: `evaluate(přihláška)` kontroluje tři brány v závazném pořadí — **zástupce → dokumenty → platba** — a první nesplněná brána určí stav. Explicitně se nastavují jen stavy `New`, `Canceled` a `Expired`. (zdroj: docs/registration-lifecycle.md)

Stavy a přechody (textový přepis diagramu):

- `New` → `PendingGuardian`: nezletilý bez navázaného rodiče.
- `PendingGuardian` → `PendingDocuments`: zástupce schválil (odkaz v e-mailu, 7 dní).
- `PendingDocuments` → `PendingPayment`: všechny povinné dokumenty schváleny vedoucím.
- `PendingPayment` → `Paid`: zaplaceno přesně (QR bankovní převod).
- Neuplatněné brány se přeskakují (dospělý s navázaným rodičem, žádné dokumenty…); akce zdarma (cena = 0) jde rovnou do `Paid`.
- `PendingPayment` → `PartialPaid`: částečná úhrada; `PartialPaid` → `Paid`: doplatek uhrazen.
- `PendingPayment` → `Overpayment`: přeplaceno; `Overpayment` → `Paid`: přeplatek vrácen.
- Zamítnutý povinný dokument vrací stav zpět do `PendingDocuments` (i ze `Paid`).
- → `Canceled`: storno — možné z každého nekoncového stavu, i ze `Paid` (terminální).
- `PendingGuardian` → `Expired`: zástupce neschválil ve lhůtě; také z `PendingPayment`, je-li zapnuté volitelné vypršení nezaplacených (výchozí: vypnuto) (terminální).
- Náhradník: zůstává v `New` se zamčenými branami, dokud nepřijme nabídku místa (platnost 48 h); propadlá nabídka jeho stav nemění.

Stavový automat přihlášky na akci dle (zdroj: docs/registration-lifecycle.md). Názvy stavů jsou formální identifikátory ze spec (jejich česká pojmenování dává tabulka níže). Brány, které se neuplatní, se prostě přeskočí. Přechod, který uživatele překvapí: zamítnutý dokument stáhne i zaplacenou přihlášku zpět do `PendingDocuments`. Podobně vratka nebo změna ceny umí vrátit `Paid` do `PartialPaid`. Terminální stavy jsou konečné — stornovaná přihláška se nikdy neoživuje, vzniká nová.

| Stav | Název ve spec (česky) | Význam | Počítá se do kapacity | Terminální |
|---|---|---|---|---|
| `New` | nová | Vznikla; čeká na první vyhodnocení, nebo náhradník čekající na nabídku místa | ne | ne |
| `PendingGuardian` | čeká na zákonného zástupce | Čeká na schválení zástupcem přes odkaz v e-mailu | **ne** | ne |
| `PendingDocuments` | čeká na dokumenty | Povinné dokumenty chybí nebo nejsou schválené | ano | ne |
| `PendingPayment` | čeká na platbu | Zatím nic nezaplaceno | ano | ne |
| `PartialPaid` | částečně zaplaceno | Zaplacena jen část ceny | ano | ne |
| `Paid` | zaplaceno | Uhrazeno přesně (nebo je akce zdarma) | ano | ne |
| `Overpayment` | přeplatek | Zaplaceno víc, než je cena; řeší účetní (vrátit / převést / ponechat jako dar) | ano | ne |
| `Canceled` | stornována | Stornována účastníkem nebo vedoucím; storno poplatek podle pravidel akce | ne | **ano** |
| `Expired` | expirovaná | Propadla, aniž byla dokončena (lhůta zástupce, nebo volitelné vypršení nezaplacených) | ne | **ano** |

#### Výchozí časové lhůty

| Lhůta | Výchozí hodnota | Kde se nastavuje |
|---|---|---|
| Schválení zákonným zástupcem | 7 dní od odeslání žádosti e-mailem | nastavení oddílu |
| Nabídka místa náhradníkovi | 48 hodin | nastavení oddílu |
| Splatnost | 14 dní od podání (nejpozději k začátku akce), nebo pevné datum per akce | nastavení akce |
| Vypršení nezaplacené přihlášky | **vypnuto** — přihlášku ruší vedoucí vědomě, aby systém sám nerušil místa lidem, kteří platí pozdě | nastavení oddílu |

Další fakta ze spec, která stojí za to mít brzy v hlavě: pořadí bran je závazné (nezletilý nesmí nic nahrávat před schválením; nevybírají se peníze za přihlášku, která nemůže projít); přihlášku **bez data narození nelze vyhodnotit** v bráně zástupce — systém si datum vyžádá a přihláška zůstává v `New`; po skončení akce se automatické přepočty zastaví s jedinou výjimkou plateb. (zdroj: docs/registration-lifecycle.md → Guardy a invarianty)

### Co je rozhodnuto vs. co je výslovně otevřené

Spec se hodnotí poctivě sama. Její vlastní verdikt: **~75 % hotovo pro backend a datovou vrstvu, ~45 % pro kompletní aplikaci** — a „UI / obrazovky / flow“ je jedna z jmenovaných červených mezer. Přesně tu mezeru zaplňuje tenhle UX projekt. (zdroj: TODO.md)

**Rozhodnuto — stavíme na tom, neotvíráme znovu**

- Datový model (~50 entit, pole, enumy) (zdroj: docs/data-model.md)
- Stavový automat přihlášky, brány, lhůty (zdroj: docs/registration-lifecycle.md)
- Model výběrových číselníků (číselníky): režimy, kapacity, cenové příplatky, fáze (zdroj: docs/event-fields.md)
- Pravidla párování plateb a výpočet splatnosti (zdroj: docs/payment-matching.md)
- Hlídky na závodech (Stezka): kategorie, věková pravidla, stanoviště (zdroj: docs/race-patrols.md)
- Sloučení osob: schvalování, konflikty polí, revert (zdroj: docs/person-merge.md)
- Reporty: metriky, parametry, rozsah per role (zdroj: docs/reports.md)
- Tabulka typů a šablon akcí; retenční/GDPR lhůty; auditní log (zdroj: README.md)
- Nefunkční požadavky: pouze čeština, CZK, `Europe/Prague`, formát data `Středa 29.7. 14:19`, pravidla OAuth, uploady 10 MB (PDF/JPG/PNG/HEIC), zabezpečení tokenů (zdroj: docs/non-functional.md)

**Výslovně otevřené — říká to spec sama**

- **Matice rolí a oprávnění** — největší deklarovaná mezera (zdroj: TODO.md #1)
- **Katalog notifikací** — mnoho „systém pošle e-mail“, ale žádná tabulka událost → šablona → příjemce → načasování (zdroj: TODO.md #3)
- **Validační pravidla a invarianty** po polích (zdroj: TODO.md #4)
- **UI / obrazovky / flow** — žádné wireframy pro portál ani admin (zdroj: TODO.md (UI Gap))
- **API kontrakt / hranice modulů** (zdroj: TODO.md #8)
- **Lifecycle osoby** — stavy existují, ale je otevřený devítibodový seznam „TODO — dořešit“: chybí matice přechodů pro dvě kolmé osy, chybí guardy a aktéři, nedefinovaná „dlouhodobá neaktivita“, rozpor archivace per oddíl vs. globální anonymizace, dopady deaktivace na vazby a login, obsah historie, interakce s členstvím DU (zdroj: README.md → Stav osoby → TODO - dořešit)
- Dvě otevřené otázky lifecyclu přihlášky: má zamítnutý dokument vracet přihlášku zpět i po skončení přihlašování? Má se převod přeplatku na jinou přihlášku nabízet automaticky, nebo vždy jen jako návrh účetní? (zdroj: docs/registration-lifecycle.md → Otevřené otázky)

> **Proč na tom UX procesu záleží**
> Aplikace bude **generovaná AI přímo z těchto dokumentů** a lidský vývojář ji bude jen revidovat. Cokoli napíšeme do dokumentů flow a specifikací obrazovek se tak fakticky stává vstupem buildu. Naše výstupy proto píšeme na stejnou laťku jako (zdroj: docs/registration-lifecycle.md): explicitní stavy, explicitní texty, explicitní hraniční případy — próza, kterou AI nemůže špatně přečíst a člověk ji umí porovnat se spec.

## 02 · UX proces a inventář flow

*Inventář 35 flow s prioritami, schválená šablona a rozhodnutí, která je potřeba učinit*

### 2.1 · Inventář flow s prioritami

Každé samostatné uživatelské flow, které ze spec plyne, seskupené po plochách. Navržené pořadí: **nejdřív veřejný portál** — nejvyšší sázky (peníze, data dětí, zákonné zastoupení) a nejméně zkušení uživatelé (rodiče na telefonu, bez zaškolení, bez účtu); pak oddílová správa (protějšek portálu — flow na portálu není hotové, dokud nemá specifikovanou i svou admin polovinu); pak ústředí; nakonec self-management, který přebírá vzory z portálu. Priority jsou návrh k přeskládání.

> **Pozor na číslování**
> Identifikátory flow D1–D4 (skupina D · Self-management) nejsou totéž co UX rozhodnutí D1–D10 v sekci 2.3 — jde o dvě nezávislé číselné řady. V textu proto vždy říkáme „flow D1“ vs. „rozhodnutí D1“.

| # | Flow | Priorita | Zdůvodnění jednou větou | Zdroj |
|---|---|---|---|---|
| **A · Veřejný registrační portál** | | | | |
| A1 | Objevení akcí (veřejný výpis; sdílecí odkazy; vnitřní akce po přihlášení) | P1 | Vstupní brána; tři úrovně viditelnosti musí působit srozumitelně i na návštěvníka bez kontextu. | (zdroj: README.md → Viditelnost akce) |
| A2 | Detail akce a náhled ceny | P1 | Tady se rodič rozhoduje; cena se liší podle typu účastníka, termínu a příplatků číselníků. | (zdroj: README.md → Ceny) (zdroj: docs/event-fields.md) |
| A3 | **Přihláška na akci** (jeden účastník) — vzorové flow | P1 | Jádrové flow peněz i důvěry; všechno ostatní se na něj váže. | (zdroj: docs/registration-lifecycle.md) |
| A4 | Schválení zákonným zástupcem přes odkaz v e-mailu | P1 | Právně významné, na jeden pokus, a provádí ho člověk, který o systém nikdy nežádal. | (zdroj: docs/registration-lifecycle.md → guardian.*) |
| A5 | Správa přihlášky bez účtu (rozcestník z tokenového odkazu: stav, dokumenty, storno, přidání účastníků) | P1 | Spec dělá z režimu bez účtu normální případ, ne nouzovku; ztráta odkazu = ztráta přístupu. | (zdroj: README.md → Přihlašování na akce) (zdroj: docs/non-functional.md → Tokeny) |
| A6 | Nahrání povinných dokumentů; opětovné nahrání po zamítnutí | P1 | Zamítnutý dokument umí stáhnout i zaplacenou přihlášku zpět — flow to musí vysvětlit v klidu. | (zdroj: docs/registration-lifecycle.md) |
| A7 | Platba přes QR + výzva k platbě a připomínky | P1 | Párování na přesnou částku (bez tolerance zaokrouhlení) dělá ze samotných platebních instrukcí kritické UX. | (zdroj: docs/payment-matching.md) |
| A8 | Nabídka místa náhradníkovi — přijetí do 48 h | P2 | Časově omezený e-mailový moment; špatně formulovaný pálí reálná místa. | (zdroj: docs/registration-lifecycle.md → substitute.*) |
| A9 | Stránka pro přihlášení dobrovolníků (per akce, je-li zapnutá) | P2 | Jiné publikum, vlastní cena a vlastní okno přihlašování, mimo kapacitu akce. | (zdroj: README.md → Konfigurace akce) |
| A10 | Skupinová přihláška — více účastníků vč. zákonných zástupců v jednom formuláři | P2 | Formulářově nejsložitější flow (dílčí přihlášky); závisí na rozhodnutí D1 níže. | (zdroj: README.md → Typy (Skupinové)) (zdroj: docs/data-model.md) |
| A11 | Založení účtu z odkazu po přihlášce; login vč. Google/Facebook OAuth | P2 | Konvertuje tokenové uživatele na účty; pravidla propojení OAuth už jsou daná. | (zdroj: docs/non-functional.md → OAuth) |
| A12 | Výběr běhu workshopu po časových blocích | P3 | Jen jeden typ akce; výběr s vlastní kapacitou per běh. | (zdroj: README.md → Typy (Workshopové)) |
| A13 | Sestavení závodní hlídky vlastníkem přihlášky (Stezka) | P3 | Bohatá pravidla, malé publikum; pravidla složení jsou plně specifikovaná. | (zdroj: docs/race-patrols.md) |
| A14 | Potvrzení doporučení mentorem / vedoucím (e-mail) | P3 | Jeden typ akce; další e-mailový moment pro publikum bez kontextu. | (zdroj: README.md → Typy (S doporučením)) |
| **B · Oddílová správa** | | | | |
| B1 | Založení akce ze šablony typu (šablona) | P1 | Určuje všechno, co portál zobrazí; šablony nesou výchozí hodnoty. | (zdroj: README.md → Typy a šablony akcí) |
| B2 | Konfigurace akce: ceny a jejich platnost, storno pravidla, výběrové číselníky, dokumenty, viditelnost, bankovní účet, splatnost | P1 | Nejhustší rodina nastavovacích obrazovek v celém systému. | (zdroj: README.md → Konfigurace akce) (zdroj: docs/event-fields.md) |
| B3 | Přehled a detail přihlášek (per akce) | P1 | Denní chleba vedoucích; stavy, kategorie a kapacita na jeden pohled. | (zdroj: docs/registration-lifecycle.md) |
| B4 | Fronta schvalování dokumentů (schválit / zamítnout s komentářem) | P1 | Admin polovina flow A6; text zamítnutí putuje rodiči do e-mailové schránky. | (zdroj: README.md → Schvalování dokumentů) |
| B5 | Výběr náhradníka po uvolnění místa | P2 | Admin polovina flow A8; po každé propadlé nabídce vybírá vedoucí znovu. | (zdroj: README.md → Náhradníci) |
| B6 | Pracovna párování plateb (ÚČE): fronta nespárovaných, ruční rozdělení, řešení přeplatků | P1 | Nejreálnější denní bolest: „rodič zaplatil za tři děti jedním převodem“. | (zdroj: docs/payment-matching.md) |
| B7 | Zápis docházky vč. dobrovolnických hodin | P2 | Nejspíš se dělá na telefonu přímo na akci; oprávnění, které může mít i rádce samostatně. | (zdroj: README.md → Docházka) |
| B8 | Evidence osob: členové/hosté, stavy osoby, chytré sloupce | P2 | Základní tabulka HVO; lifecycle osoby je zčásti otevřený (viz Rozhodnuto vs. otevřeno). | (zdroj: README.md → Stav osoby, Pomocná evidence) |
| B9 | Družiny; dohled nad hlídkami, stanoviště a rozhodčí | P3 | Strukturované, ale okrajové; konzistenční kontroly jsou specifikované. | (zdroj: docs/race-patrols.md) |
| B10 | Nastavení oddílu: bankovní účty, moduly, lokace, pozvánky rolí, nastavení pošty | P2 | Nastavuje se jednou; podmiňuje řadu dalších flow (platby potřebují bankovní token). | (zdroj: README.md → Volitelné moduly) (zdroj: docs/fio-sync.md) |
| B11 | Reporty oddílu | P3 | Metriky i rozsah per role plně definované; převážně datavizová práce. | (zdroj: docs/reports.md) |
| **C · Správa ústředí** | | | | |
| C1 | Správa oddílů a přiřazování HVO (pozvánky e-mailem) | P3 | Několik expertních uživatelů; onboardingový řetěz celé organizace. | (zdroj: README.md → Administrátor) |
| C2 | Životní cyklus regionů: vznik, přesun oddílů, sloučení, rozdělení (verzované) | P3 | Vzácné operace, ale sémantika zachování historie musí být vidět. | (zdroj: README.md → Region) |
| C3 | Deduplikace osob a schvalování sloučení (vícestranné) | P3 | Vícestranné schvalování e-mailem + konflikt polí volbou A/B; mechanika je plně specifikovaná. | (zdroj: docs/person-merge.md) |
| C4 | Reporty ústředí (unikátní počty dětí, dimenze regionu, reportovací sloučení) | P3 | Páteř výkaznictví k dotacím; jen pro čtení. | (zdroj: README.md → Modul reporty ústředí) (zdroj: docs/reports.md) |
| C5 | Modul vzdělávání (kurzy, platnost, certifikáty) | P4 | Samostatný volitelný modul. | (zdroj: README.md → Modul vzdělávání) |
| C6 | Systémové šablony a jmenný whitelist | P4 | Vzácná administrátorská konfigurace. | (zdroj: README.md → Deduplikace, Šablony) |
| **D · Self-management (přihlášený uživatel)** | | | | |
| D1 | Moje přihlášky — rozcestník (stav, platba, dokumenty, storno) | P2 | Účtové zrcadlo flow A5 — mělo by beze zbytku převzít jeho návrh. | (zdroj: README.md → Rozsah) |
| D2 | Moje děti: údaje a přihlášky per dítě, pozvání druhého zástupce, přechod do čtení po zletilosti | P2 | Vazba rodič ↔ dítě má reálné hrany životního cyklu (18. narozeniny, zrušení vazby), na které uživatelé narazí. | (zdroj: README.md → Rodič) |
| D3 | Moje osobní údaje (adresa, pojišťovna, souhlasy) | P4 | Prosté CRUD, jakmile budou existovat pravidla polí. | (zdroj: README.md → Osoba) |
| D4 | Účet a zabezpečení: heslo, propojení OAuth, žádosti o sloučení duplicitních osob | P3 | Návrh sloučení („nejste to vy v jiném oddíle?“) je citlivý souhlasový moment. | (zdroj: README.md → Deduplikace) (zdroj: docs/non-functional.md → OAuth) |

*(odvozeno)* Priority i seskupení jsou návrh. Jeden poctivý protiargument proti „nejdřív portál“: na portálu nic neexistuje, dokud flow B1/B2 neumí akci založit — proto jsem obě přitáhl do P1 vedle portálové sady.

### 2.2 · Schválená šablona flow

Vzorové zpracování flow A3 slouží jako šablona pro všechna flow v inventáři. Každé flow se zpracovává jednotně:

*Úroveň flow*

**Hlavička + Průběh**

- **Název flow, priorita a citace zdrojů** — dohledatelnost každého tvrzení ve spec.
- **Průběh** — číslované kroky od spouštěče po koncové stavy; odbočky uvozené „**Větev:**“ (schválení zástupcem, dokumenty, náhradník…), mapované na stavy životního cyklu, kde dávají smysl.

*Úroveň obrazovky*

**Obrazovky**

Pro každou obrazovku flow:

- **Účel** — jediná práce obrazovky
- **Publikum** — zkušenost, zařízení, rozpoložení
- **Obsah a pole** — z datového modelu; povinné/volitelné; záměr textů
- **Stavy** — prázdný, načítání, chyba, úspěch — jen kde dávají smysl
- **Validace** — dle spec vs. navržené (označené)
- **Mobil** — klávesnice, dotykové cíle, ovládání jednou rukou
- **Otevřené UX otázky** — s badgem a možnostmi

U jednodušších administrativních flow, kde spec definuje málo detailu, se šablona uplatní úsporněji — méně obrazovek a kratší specifikace; hloubka se řídí tím, co spec skutečně říká. Záměrně mimo rozsah zatím zůstává design systém, knihovna komponent a vizuální identita — dokumenty jsou nezávislé na nástroji (lze je provést ve Figmě, nebo zapsat rovnou do build promptu pro AI).

### 2.3 · UX rozhodnutí k učinění (D1–D10)

Místa, kde spec mlčí — nebo definuje mechaniku, ale ne prožitek. U každého: co spec říká, co neříká, a prostor možností, jak ho vidím.

#### D1 Vícečlenná přihláška: jeden dlouhý formulář, stepper, nebo „přidat dalšího účastníka“? **[K rozhodnutí]**

Spec říká: typ akce „Skupinové“ dává do jednoho přihlašovacího formuláře „více účastníků vč. zákonných zástupců najednou“; datový model podporuje dílčí přihlášky (`parent_registration_id`), každou s vlastním nezávislým stavem. (zdroj: README.md → Typy) (zdroj: docs/data-model.md) Spec neříká: nic o struktuře formuláře — a nezávislé stavy znamenají, že se sourozenci mohou rozjet (jeden čeká na dokumenty, druhý je zaplacený).

- Jeden formulář s opakovatelnými bloky účastníků — nejrychlejší pro 2–3 děti, těžkopádné na mobilu.
- Stepper (účastníci → volby → souhrn) — krotí složitost, ale skrývá celek.
- Po jednom, s tlačítkem „přidat dalšího účastníka“ po odeslání (spec výslovně umožňuje přidávat účastníky později přes odkaz pro správu) — nejjednodušší obrazovky, riziko odpadnutí uživatele.

#### D2 Jak se prožívá schválení zákonným zástupcem — na obou stranách **[K rozhodnutí]**

Spec říká: nezletilý bez navázaného rodiče musí zadat e-mail zástupce; systém pošle schvalovací odkaz; výchozí lhůta 7 dní; do schválení se přihláška nepočítá do kapacity; schválením vzniká vazba rodič ↔ dítě; propadnutí ⇒ `Expired`. (zdroj: docs/registration-lifecycle.md) Spec neříká: co vidí nezletilý při čekání, jestli jde odkaz poslat znovu nebo opravit překlep v e-mailu, jestli zástupce může přihlášku *odmítnout* (existuje jen schválit-nebo-nechat-vypršet), ani jak sdělit „vaše místo ještě není rezervované“.

- Rozhodnout: obsah čekací stránky, možnost znovu odeslat / opravit e-mail, explicitní odmítnutí vs. tiché vypršení, viditelnost odpočtu — a jak vysvětlit, že schválením vzniká trvalá zástupcovská vazba.

#### D3 Správa bez účtu: jakou váhu dát tokenovému odkazu **[K rozhodnutí]**

Spec říká: přihláška založená bez účtu dostane odkaz na svou správu (storno, změny, přidání účastníků) a na pozdější založení účtu; token platí pro jednu přihlášku a končí s koncem akce. (zdroj: README.md → Přihlašování na akce) (zdroj: docs/non-functional.md → Tokeny) Spec neříká: jak uživatele varovat, že e-mail *je* klíč, kdy a jak naléhavě pobízet k založení účtu, ani co zažívá člověk se třemi tokenovými odkazy a žádným účtem.

- Možnosti od „token je hlavní cesta, účet jen šeptáme“ po „mezistránka s pobídkou k účtu po každé přihlášce“. Ovlivní zátěž supportu víc než cokoli jiného, co rozhodneme.

#### D4 Prezentace platby — QR, přesné částky a rodiny **[K rozhodnutí]**

Spec říká: potvrzovací e-mail nese QR + platební údaje; částky se párují *přesně*, koruna rozdílu = nedoplatek/přeplatek; párování zvládá jednu rodičovskou platbu za víc přihlášek (M:N) s automatickým rozdělením, když součty sedí. (zdroj: docs/payment-matching.md) Spec neříká: jestli má UI kombinovanou rodinnou platbu někdy *nabízet*, nebo vždy ukázat jedno QR na přihlášku, jak zobrazit zbývající částku po částečné platbě, ani kde kromě e-mailu platební údaje žijí.

- Pouze QR per přihláška (nejbezpečnější pro párování) vs. souhrnné QR „zaplatit za všechny moje děti“ (nejlepší UX, opírá se o vícekandidátní párování s potvrzením účetní). Tohle si zaslouží rozhovor s personou účetního.

#### D5 Lidská řeč pro strojové stavy — a pro pohyb zpět **[K rozhodnutí]**

Spec říká: devět interních stavů, počítaných, s legálními návraty zpět (zamítnutý dokument stáhne `Paid` do `PendingDocuments`); v próze README existují česká pojmenování stavů (nová → čeká na zákonného zástupce → … → zaplaceno / přeplatek). UI je výhradně česky. (zdroj: docs/registration-lifecycle.md) (zdroj: README.md → Přihlašování na akce) (zdroj: docs/non-functional.md) Spec neříká: zda jsou prozaická pojmenování zároveň uživatelskými popisky, ani jak nelineární, přepočítávaný stav zobrazit, aby nepůsobil jako rozbitý progress bar.

- Rozhodnout: finální sadu českých popisků, zda místo názvu stavu ukázat checklist („co ještě chybí“), a tón momentu návratu zpět („potřebujeme jeden dokument znovu“ vs. děsivá regrese stavu).

#### D6 Plná / uzavřená / ještě neotevřená akce — a jak se stát náhradníkem **[K rozhodnutí]**

Spec říká: kapacita a počet náhradníků per akce; náhradník sedí v `New` se vším zamčeným; uvolněné místo → vedoucí vybere → nabídka na 48 h; propadlá nabídka ho tiše vrátí do čekání. (zdroj: docs/registration-lifecycle.md) Spec neříká: zda se zobrazuje pořadí ve frontě, jak formulovat „jste náhradník“ už při přihlášení, ani co říká stránka akce před otevřením a po uzavření přihlašování.

#### D7 Strategie validací — spec zatím žádnou nemá **[K rozhodnutí]**

Spec říká: validační pravidla jsou deklarovaná mezera; existují jen fragmenty (jmenný whitelist s výjimkami per oddíl, datum narození nutné pro bránu zástupce, přesné povinné fáze u číselníků). (zdroj: TODO.md #4) Spec neříká: formáty polí, kdy se validace spouští, ani texty chyb.

- Návrh: naše specifikace obrazovek definují UX vrstvu (inline při opuštění pole, chyba pod polem) a zároveň plní chybějící validační katalog zpět do spec. Nepsali bychom jen UI, ale i kus spec.

#### D8 Nahrávání dokumentů a smyčka zamítnutí **[K rozhodnutí]**

Spec říká: nahrávat lze postupně nebo najednou; 10 MB, PDF/JPG/PNG/HEIC ověřované podle obsahu; vedoucí schválí, nebo zamítne s komentářem; zamítnutí vyzve účastníka e-mailem k novému nahrání. (zdroj: README.md → Povinné dokumenty) (zdroj: docs/non-functional.md → Úložiště) Spec neříká: návod na focení telefonem, prezentaci stavu per dokument, ani jak zobrazit komentář k zamítnutí (píše ho dobrovolný vedoucí, čte ho rodič).

#### D9 Priority zařízení po plochách **[K rozhodnutí]**

Spec říká: o zařízeních nikde nic. Spec neříká — takže je to celé na nás. *(odvozeno)* Rozumné výchozí nastavení: portál a zástupcovské/e-mailové momenty mobile-first; zápis docházky (flow B7) primárně telefon na tábořišti; pracovna párování pro účetní a konfigurace akce primárně desktopové tabulky. Před jakoukoli prací na layoutu to chce potvrzení.

#### D10 Notifikační dotykové body — načasování a tón **[K rozhodnutí]**

Spec říká: e-mailů existuje spousta (potvrzení + výzva k platbě, žádost zástupci, zamítnutí dokumentu, potvrzení plateb, připomínky s frekvencí nastavitelnou per oddíl, nabídky náhradníkům) a každé odeslání se eviduje; samotný katalog je deklarovaná mezera. (zdroj: TODO.md #3) (zdroj: docs/non-functional.md → Odchozí e-maily) Spec neříká: obsah, předměty, výchozí kadence, ani které momenty si zaslouží i zobrazení v aplikaci. Pro zástupce a náhradníky *je* e-mail celé UI — v našich specifikacích s ním zacházíme jako s plnohodnotnou obrazovkou.

## A · Plocha A — Veřejný registrační portál

*14 flow · bez přihlášení, rodiče na mobilu, tokenové odkazy z e-mailu jako normální cesta*

Portál je jediná plocha, kterou vidí lidé **bez zaškolení a často bez účtu**: rodiče přihlašující děti, hosté, zákonní zástupci a náhradníci, kteří dostali e-mail. Účastník bez účtu dostává k přihlášce **tokenový odkaz**, kterým ji spravuje (storno, změny, přidání účastníků) — to je podle spec normální cesta, ne nouzový režim. (zdroj: README.md → Přihlašování na akce) (zdroj: docs/non-functional.md → Tokeny) Všechno je česky, částky ve formátu `1 250 Kč`, data ve formátu `Středa 29.7. 14:19` (rok jen když se liší od aktuálního). (zdroj: docs/non-functional.md → Lokalizace) Každé flow níže proto počítá s telefonem jako výchozím zařízením a s e-mailem jako plnohodnotnou obrazovkou.

Flow A3 je referenční vzor šablony; obrazovky S1–S7 z něj sdílí i flow A1, A2, A4, A5, A7 a A8 — kde se obrazovka opakuje, odkazujeme a doplňujeme jen to, co dané flow přidává.

### A1 · Objevení akcí P1

Zdroje: (zdroj: README.md → Viditelnost akce) (zdroj: docs/data-model.md → EVENT) (zdroj: docs/non-functional.md → Lokalizace)

**Cíl a spouštěč.** „Najít akci, o které mi řekli na schůzce nebo v oddílovém chatu, a ověřit si, že je to ona.“ Spouštěč: odkaz sdílený oddílem, nebo přímé otevření portálu. **Aktéři:** studený návštěvník (rodič, host), přihlášený člen. **Vstupní body:** URL portálu, sdílecí odkaz akce, výpis po přihlášení.

#### Průběh

1. **Otevření portálu — veřejný výpis** — Výpis zobrazuje jen akce s viditelností **veřejná**; přihlásit se na ně může kdokoli. (zdroj: README.md → Viditelnost akce)
2. **Větev: přihlášený uživatel — vnitřní akce** — **Vnitřní** akce ve výpisu nejsou pro veřejnost; vidí je jen přihlášená osoba s aktivní vazbou na pořádající oddíl. U vnitřní akce ústředí je vidí všichni registrovaní členové. (zdroj: README.md → Viditelnost akce)
3. **Větev: sdílecí odkaz — přímý vstup na detail** — **Každá** akce má sdílecí odkaz (`share_slug`) bez ohledu na viditelnost — je to přístupová cesta, ne publikace ve výpisu. **Neveřejná** akce je dostupná výhradně tudy; odkaz otevře rovnou detail akce (flow A2). (zdroj: README.md → Viditelnost akce) (zdroj: docs/data-model.md → EVENT.share_slug)
4. **Výběr akce → detail** — Klepnutí na akci vede na detail s náhledem ceny (flow A2) a odtud na formulář (flow A3).

**Data:** čtení `EVENT` (viditelnost, termíny, přihlašovací okno), `EVENT_PRICE` pro „cena od“. (zdroj: docs/data-model.md) **Notifikace:** žádné. **Admin protějšek:** B2 (nastavení viditelnosti akce).

#### Obrazovky

#### A1-S1 · Výpis akcí (domovská stránka portálu)

**Účel:** Najít akci, o které mi někdo řekl, a získat důvěru v systém za devadesát vteřin. Totožná obrazovka jako S1 schváleného vzoru A3 (viz níže) — zde jako vlastník flow s větvemi viditelnosti.

**Publikum:** Studení návštěvníci, převážně rodiče, převážně na telefonu.

**Obsah a pole:**

- Jen veřejné akce; po přihlášení navíc vnitřní akce oddílů s aktivní vazbou (u ústředí všichni registrovaní členové). *(odvozeno)* Vnitřní akce by měly být ve výpisu vizuálně označené („jen pro oddíl“), aby člen pochopil, proč je kamarád nevidí. (zdroj: README.md → Viditelnost akce)
- Na kartu akce *(odvozeno z polí EVENT)*: název, typ, termín od–do, pořádající oddíl, přihlašovací okno (`registration_from/to`), „cena od“ z `EVENT_PRICE`. Datum ve formátu `Středa 29.7.`, rok jen když se liší. (zdroj: docs/data-model.md → EVENT) (zdroj: docs/non-functional.md → Lokalizace)

**Stavy:**

- **Prázdný:** žádné veřejné akce — pro oddíly, které jedou vše vnitřně, je to normální stav; prázdná stránka má vysvětlit sdílecí odkazy a přihlášení, ne se omlouvat.
- **Načítání:** skeleton karet s rezervovaným místem — žádný skok layoutu.
- **Chyba:** srozumitelná hláška s možností opakovat; žádné technické detaily.

**Mobil:** Karty v jednom sloupci, celá karta jako dotyková plocha (min. 44 px), žádné horizontální posouvání. Sdílecí odkaz musí fungovat i jako cíl z QR kódu na papírové pozvánce. *(odvozeno)*

**Otevřené UX otázky:**

- **[K rozhodnutí]** Spec nedefinuje žádné filtrování, hledání ani řazení výpisu (region? oddíl? datum? typ?). Návrh: začít bez filtrů, řadit podle začátku akce; filtry doplnit až podle reálného počtu akcí.
- **[K rozhodnutí]** Nedefinováno, zda a jak dlouho jsou vidět skončené či uzavřené akce a zda výpis ukazuje stav „plno / uzavřeno / brzy otevřeme“ už na kartě.

### A2 · Detail akce a náhled ceny P1

Zdroje: (zdroj: README.md → Konfigurace akce, Ceny a storna) (zdroj: docs/event-fields.md) (zdroj: docs/data-model.md → EVENT, EVENT_PRICE, EVENT_DOCUMENT, CANCELLATION_RULE)

**Cíl a spouštěč.** „Rozhodnout se, jestli sem dám dítě a peníze — a co všechno k tomu budu potřebovat.“ Spouštěč: klepnutí ve výpisu (A1), sdílecí odkaz. **Aktéři:** rodič, dospělý účastník, nezletilý. **Předpoklady:** žádné — detail je čitelný i před otevřením přihlašování i přes sdílecí odkaz u neveřejné akce.

#### Průběh

1. **Základní údaje akce** — Název, termín od–do, přihlašovací okno, kapacita + počet náhradníků, pořádající oddíl, případně místo konání z lokací oddílu. (zdroj: docs/data-model.md → EVENT) (zdroj: README.md → Konfigurace akce)
2. **Náhled ceny** — Ceny platné v různých termínech pro různé typy účastníků (DU / bez DU / dobrovolník / vedoucí / dítě vedoucího / sponzorská). **Výsledná cena = základní cena podle typu účastníka + součet příplatků zvolených položek číselníků.** Členství DU se vyhodnocuje k roku akce, ne k dnešku. (zdroj: README.md → Ceny a storna) (zdroj: docs/event-fields.md) (zdroj: README.md → Člen DU)
3. **Výběrové číselníky a povinné dokumenty** — Číselníky s veřejným popisem (`comment`) a příplatky; položky s naplněnou kapacitou se přestanou nabízet. Číselníky s podmínkou způsobilosti se nezpůsobilým skryjí. Výčet povinných dokumentů akce (např. potvrzení o lékařské způsobilosti). (zdroj: docs/event-fields.md) (zdroj: README.md → Povinné dokumenty)
4. **Větev: dobrovolníci** — Má-li akce zapnutou evidenci dobrovolníků, detail nabídne odkaz na samostatnou dobrovolnickou stránku s vlastní cenou a vlastním oknem přihlašování (flow A9). Bez zapnutí se stránka nenabízí. (zdroj: README.md → Konfigurace akce)
5. **Větev: plná kapacita** — Je-li kapacita plná a akce má místa náhradníků, CTA se mění na „přihlásit se jako náhradník“ (flow A3, větev S; nabídky pak flow A8). (zdroj: docs/registration-lifecycle.md)
6. **Přechod na formulář** — CTA vede na registrační formulář (flow A3) — jen v otevřeném přihlašovacím okně.

**Data:** čtení `EVENT`, `EVENT_PRICE`, `EVENT_FIELD(_OPTION)`, `EVENT_DOCUMENT`, `CANCELLATION_RULE`. **Notifikace:** žádné. **Admin protějšek:** B2 (vše, co je tu vidět, se tam nastavuje).

#### Obrazovky

#### A2-S1 · Detail akce

**Účel:** Odpovědět „co, kdy, za kolik a co budu potřebovat“ dřív, než formulář cokoli chce. Totožná obrazovka jako S2 schváleného vzoru A3 (viz níže).

**Publikum:** Rodič rozhodující se o penězích; často přichází sdílecím odkazem bez jakéhokoli kontextu o systému.

**Obsah a pole:**

- Vše z kroků 1–3 průběhu; ceník jako tabulka typ účastníka × období platnosti, částky `1 250 Kč`. (zdroj: docs/non-functional.md → Lokalizace)
- Storno podmínky: procenta v termínech (`CANCELLATION_RULE`) existují v datech — zobrazení před závazkem je otázka důvěry (viz níže). (zdroj: README.md → Ceny a storna)
- Kapacitní stav: volno / plno + náhradnická místa / uzavřeno / ještě neotevřeno (s datem otevření). (zdroj: docs/data-model.md → EVENT)

**Stavy:**

- Před otevřením přihlašování (zobrazit `registration_from`) · otevřeno · **plno → „přihlásit se jako náhradník“** · po uzávěrce · akce skončila · otevřeno přes sdílecí odkaz u neveřejné akce (stránka funguje, jen není ve výpisu).

**Mobil:** CTA „Přihlásit na akci“ dostupné bez dlouhého scrollování (sticky spodní lišta *(odvozeno)*); ceník jako tabulka smí scrollovat uvnitř vlastního kontejneru, ne celá stránka do šířky.

**Otevřené UX otázky:**

- **[K rozhodnutí]** Typ účastníka návštěvníka je před formulářem neznámý — ukázat celý ceník, nebo „od X Kč“ s tabulkou za klepnutím?
- **[K rozhodnutí]** Lokace oddílu jsou „viditelné jen v rámci oddílu“, ale akce může lokaci mít jako místo konání — co vidí veřejnost: název + GPS, jen název, nic? Rozpor spec k vyjasnění se zadavatelem. (zdroj: README.md → Oddíl)
- **[K rozhodnutí]** Zda storno podmínky zobrazovat už zde (transparentnost) nebo až v potvrzení přihlášky.

### A3 · Přihlášení na veřejnou akci (jeden účastník) P1

Zdroje: (zdroj: docs/registration-lifecycle.md) (zdroj: README.md → Přihlašování na akce, Viditelnost akce) (zdroj: docs/event-fields.md) (zdroj: docs/payment-matching.md)

**Cíl a spouštěč.** „Přihlásit své dítě (nebo sebe) na tuhle akci a vědět, že mám místo jisté.“ Spouštěč: odkaz sdílený v oddílovém chatu, nebo procházení portálu. **Aktéři:** dospělý účastník, rodič jednající za dítě, nebo nezletilý přihlašující se sám; později zákonný zástupce (e-mail), vedoucí (dokumenty), banka (platba). **Vstupní body:** veřejný výpis, sdílecí odkaz (`share_slug` — má ho každá akce bez ohledu na viditelnost), vnitřní výpis po přihlášení. (zdroj: README.md → Viditelnost akce)

**Předpoklady / guardy.** Otevřené přihlašovací okno (`registration_from/to`); volná kapacita nebo místo náhradníka; u závodní/skupinové/workshopové akce se navíc uplatní subsystémy daného typu. (zdroj: docs/registration-lifecycle.md → registration.created)

#### Průběh

1. **Objevení akce** — Veřejný výpis zobrazuje jen **veřejné** akce; **vnitřní** akce se ukážou přihlášeným lidem s aktivní vazbou na pořádající oddíl; **neveřejné** akce se otevírají jen sdílecím odkazem. (zdroj: README.md → Viditelnost akce)
2. **Přečtení detailu akce, skutečná cena** — Název, termíny, přihlašovací okno, kapacita, ceny podle typu účastníka a období, povinné dokumenty, výběrové číselníky s veřejnými popisy a příplatky. Výsledná cena = základní cena podle typu účastníka + součet příplatků zvolených položek. (zdroj: docs/event-fields.md)
3. **Vyplnění přihlášky** — Pole osoby podle šablony akce (např. tituly + trvalé bydliště jen u akcí s certifikátem), chytré sloupce oddílu označené jako povinné/volitelné, číselníky s `required_phase = on_submit`, a — je-li účastník nezletilý bez navázaného rodiče — **e-mail zákonného zástupce**. (zdroj: README.md → Šablony, Přihlašování na akce)
4. **Odeslání → přihláška existuje, stav se počítá** — Vznikne `REGISTRATION` s variabilním symbolem (VS) a tokenem pro správu; proběhne `evaluate()` a první nesplněná brána určí stav. Odchází potvrzovací e-mail (s výzvou k platbě + QR, je-li stanovena cena). (zdroj: docs/registration-lifecycle.md) (zdroj: README.md → Přihlašování na akce)
5. **Větev Z — schválení zákonným zástupcem `PendingGuardian`** — Nezletilý bez navázaného rodiče: zástupce dostane e-mail s odkazem; do schválení se přihláška **nepočítá do kapacity** a nezletilý nesmí nic nahrávat. Schválení zapíše `guardian_approved_at`, **vytvoří vazbu rodič↔dítě** a přepočítá stav. Bez schválení do 7 dnů (výchozí) → `Expired`, účastník dostane zprávu. Chybí datum narození? Bránu nelze vyhodnotit — systém si ho vyžádá a přihláška zůstává `New`. (zdroj: docs/registration-lifecycle.md → guardian.*)
6. **Větev D — povinné dokumenty `PendingDocuments`** — Nahrávání postupně nebo najednou (10 MB; PDF/JPG/PNG/HEIC ověřené podle obsahu). Vedoucí každý dokument schválí, nebo zamítne s komentářem — zamítnutí e-mailem vyzve k novému nahrání a přihláška tu zůstává (nebo sem spadne zpět!), dokud nejsou všechny povinné dokumenty *schválené*. (zdroj: README.md → Povinné dokumenty, Schvalování dokumentů) (zdroj: docs/non-functional.md → Úložiště)
7. **Platba `PendingPayment` → `Paid`** — QR + účet, částka, VS (identifikuje přihlášku) a SS (identifikuje akci). Splatnost: relativní (výchozích 14 dní od podání, nejpozději k začátku akce), nebo pevné datum pro celou akci; pozdě podaná přihláška je splatná ihned. Připomínky v četnosti dle nastavení oddílu. Částečná platba → `PartialPaid`; přesná → `Paid` (potvrzení za každou napárovanou platbu); vyšší → `Overpayment`, který řeší účetní (vratka / převod na jinou přihlášku / ponechat jako dar). (zdroj: docs/payment-matching.md)
8. **Větev S — akce plná: náhradník `New`** — Přihláška dostane kategorii *náhradník* a čeká se všemi branami zamčenými. Když se uvolní místo, vedoucí vybere náhradníka, který dostane **48hodinovou nabídku**; přijetí ho překlopí na účastníka a odemkne dokumenty/platbu; propadlá nabídka ho beze změny vrátí do čekání. (zdroj: docs/registration-lifecycle.md → substitute.*)
9. **Hotovo — a východy, které jsou otevřené vždy** — **Výstupní stavy:** `Paid` (šťastný konec), `Canceled` (storno je možné z každého nekoncového stavu, i ze zaplaceno; poplatek dle termínovaných storno pravidel akce; vratku vyplácí účetní mimo systém), `Expired` (lhůta zástupce, nebo volitelné vypršení nezaplacených — ve výchozím stavu vypnuté). Terminální je terminální: oživení = nová přihláška. (zdroj: docs/registration-lifecycle.md → Guardy a invarianty)

**Data:** `REGISTRATION`, `PERSON` (vznik nebo dohledání), `REGISTRATION_FIELD_VALUE`, `REGISTRATION_DOCUMENT`, `PARENT_CHILD` (při schválení zástupcem), `PAYMENT_ALLOCATION`. (zdroj: docs/data-model.md) **Notifikace:** potvrzení + výzva k platbě, žádost zástupci, zamítnutí dokumentu, potvrzení plateb, připomínky, nabídka náhradníkovi, zpráva o expiraci. **Admin protějšky:** B3 (seznam přihlášek), B4 (posuzování dokumentů), B5 (výběr náhradníka), B6 (párování).

#### Obrazovky

Flow pokrývá sedm obrazovek. „Obrazovkou“ jsou i dva e-mailové momenty — pro zástupce a náhradníky *je* e-mail produktem.

#### S1 · Výpis akcí (domovská stránka portálu)

**Účel:** Najít akci, o které vám někdo řekl; získat důvěru za devadesát vteřin.

**Publikum:** Studení návštěvníci, převážně rodiče, převážně telefony.

**Obsah a pole:**

- Jen veřejné akce; přihlášení uživatelé s vazbou na oddíl vidí i jeho vnitřní akce (vnitřní akce ústředí: všichni registrovaní členové). (zdroj: README.md → Viditelnost akce)
- Na akci *(odvozeno z polí EVENT)*: název, typ, termín od–do, pořádající oddíl, přihlašovací okno, cena od. Data ve formátu `Středa 29.7.`, rok jen liší-li se od aktuálního. (zdroj: docs/non-functional.md → Lokalizace)

**Stavy:**

- **Prázdný:** žádné veřejné akce — spec připouští, že to je normální stav oddílů, které jedou vše vnitřně; prázdný stav má vysvětlit sdílecí odkazy, ne se omlouvat.
- **Načítání:** skeleton karet s rezervovaným místem — žádný skok layoutu.

**Otevřené UX otázky:** **[K rozhodnutí]** Spec nedefinuje žádné filtry, hledání ani řazení výpisu (region? oddíl? datum? typ?). Nedefinováno i to, zda zůstávají vidět skončené akce.

#### S2 · Detail akce

**Účel:** Odpovědět „co, kdy, za kolik, co budu potřebovat“ dřív, než formulář o cokoli požádá.

**Publikum:** Rodič rozhodující se, zda vloží peníze.

**Obsah a pole:**

- Z `EVENT`: název, termíny, přihlašovací okno, kapacita + počet náhradníků, místo konání, je-li nastaveno. (zdroj: docs/data-model.md)
- **Ceny podle typu účastníka a období platnosti** (člen DU / bez DU / dobrovolník / vedoucí / dítě vedoucího / sponzorská), částky jako `1 250 Kč`. (zdroj: README.md → Ceny) (zdroj: docs/non-functional.md)
- Seznam povinných dokumentů; číselníky s veřejným popisem (`comment`) a příplatky — položky s vyčerpanou kapacitou se přestanou nabízet. (zdroj: docs/event-fields.md)
- Odkaz na dobrovolnickou stránku, je-li zapnutá (vlastní okno a cena, vlastní stránka). (zdroj: README.md → Konfigurace akce)

**Stavy:**

- Ještě neotevřeno (zobrazit datum otevření) · otevřeno · **plno → „přihlásit se jako náhradník“** · uzavřeno · otevřeno sdílecím odkazem u neveřejné akce (stránka funguje, jen není ve výpisu).

**Otevřené UX otázky:**

- **[K rozhodnutí]** Typ účastníka je před formulářem neznámý — ukázat celou cenovou tabulku, nebo „od X Kč“ s tabulkou za klepnutím? (Otázka tónu z rozhodnutí D5 v malém.)
- **[K rozhodnutí]** Lokace oddílu jsou „viditelné jen v rámci oddílu“, přitom akce může lokaci mít jako místo konání — co vidí veřejnost: název + GPS, jen název, nic? Rozpor spec k vyřešení se zadavatelem. (zdroj: README.md → Oddíl)
- **[K rozhodnutí]** Zda zde zobrazovat storno podmínky (termínovaná procenta) — v datech existují a jejich ukázání před závazkem je otázka důvěry. (zdroj: README.md → Ceny a storna)

#### S3 · Registrační formulář

**Účel:** Sebrat přesně to, co žádá šablona této akce — nic navíc — a spočítat skutečnou cenu ještě před odesláním.

**Publikum:** Jeden palec, jedna cesta vlakem, možná patnáctiletý.

**Obsah a pole:**

- Základ osoby: jméno, příjmení, přezdívka, pohlaví, datum narození, kontaktní e-mail; tituly před/za a trvalé bydliště *jen* když je šablona vyžaduje (akce s certifikátem); pojišťovna dle potřeb akce. (zdroj: docs/data-model.md → PERSON) (zdroj: README.md → Osoba)
- Chytré sloupce oddílu vložené jako povinná/volitelná pole (hodnoty se ukládají k osobě). (zdroj: README.md → Oddíl)
- Číselníky s `required_phase = on_submit`; jedno- vs. vícevýběr podle `max_select`; podmínky způsobilosti skryjí číselníky nezpůsobilým; každá volba živě upravuje celkovou cenu. (zdroj: docs/event-fields.md)
- E-mail zákonného zástupce — **podmíněně:** jen u nezletilého bez aktivní vazby na rodiče. (zdroj: docs/registration-lifecycle.md)

**Validace:**

- **Dané spec:** česká křestní jména proti whitelistu (existují oddílové výjimky — chyba musí nabídnout cestu, ne zeď); datum narození je nutné k vyhodnocení brány zástupce — bez něj přihláška parkuje v `New` a systém si vyžádá e-mail zástupce; povinné on-submit číselníky blokují odeslání; plné položky se přestanou nabízet. (zdroj: README.md → Deduplikace) (zdroj: docs/registration-lifecycle.md)
- **Vše ostatní je deklarovaná mezera validací** — formáty, pravidla e-mailu, struktura adresy jsou na nás (rozhodnutí D7). (zdroj: TODO.md #4)

**Stavy:**

- Odesílání: zamčené tlačítko + průběh — dvojité odeslání zde znamená duplicitní přihlášku dítěte.
- Kapacita položky vyčerpaná *během vyplňování* (někdo vzal poslední lůžko): potřebuje vlídný moment nové volby, spec mlčí. **[K rozhodnutí]**

**Mobil:** Viditelné popisky nad poli, text chyby přímo pod chybným polem s `aria-describedby`; správné typy a klávesnice (`type=email`, datumové pole, `inputmode=numeric`); zapnutý autofill — rodiče píší stejnou adresu mnohokrát ročně; 44px dotykové cíle u výběru položek.

**Otevřené UX otázky:**

- **[K rozhodnutí]** Jedna stránka vs. stepper — a tentýž skelet se musí natáhnout na skupinové akce (rozhodnutí D1) i výběr workshopových bloků.
- **[K rozhodnutí]** Vysvětlení důsledku e-mailu zástupce *před* odesláním („rodič musí potvrdit do 7 dnů, do té doby místo není držené“) — formulace s vysokou sázkou, spec mlčí.

#### S4 · Potvrzení po odeslání („co bude dál“)

**Účel:** Přeložit vypočtený stav do krátkého, poctivého seznamu úkolů; předat odkaz pro správu přihlášky.

**Publikum:** Moment, kdy se důvěra vyhrává, nebo prohrává.

**Obsah a pole:**

- Varianta podle výsledného stavu: `PendingGuardian` („schvalovací e-mail odešel na …; místo není rezervované, dokud zástupce neschválí; 7 dní“), `PendingDocuments` (nahrát teď / později odkazem), `PendingPayment` (QR přímo tady + je i v e-mailu), `Paid` (akce zdarma — hotovo), náhradník („jste na čekací listině; nabídky chodí e-mailem a platí 48 h“). (zdroj: docs/registration-lifecycle.md)
- Rekapitulace: účastník, akce, rozpad ceny (základ + příplatky), VS, splatnost. (zdroj: docs/payment-matching.md)
- Tokenový odkaz pro správu, rámovaný podle rozhodnutí D3; nabídka založení účtu. (zdroj: README.md → Přihlašování na akce)

**Otevřené UX otázky:** **[K rozhodnutí]** Zda náhradníkům ukazovat pořadí ve frontě (spec pořadí nikde viditelně neukládá; vedoucí vybírá ručně — z čehož plyne, že *žádný* slib pořadí bychom dávat neměli). (zdroj: docs/registration-lifecycle.md → substitute.*)

#### S5 · Schvalovací stránka zástupce (z e-mailu)

**Účel:** Umožnit zástupci ověřit a schválit do minuty — a pochopit, že schválením zároveň vzniká jeho trvalá vazba na dítě v systému.

**Publikum:** Rodič, který o používání tohoto systému nikdy nežádal, na telefonu, uprostřed dne.

**Obsah a pole:**

- Jméno dítěte, souhrn akce (název, termíny, oddíl, cena), kdo přihlášku podal; jediná schvalovací akce. Token platí po dobu schvalovací lhůty (výchozích 7 dní), rozsah jedna přihláška. (zdroj: docs/registration-lifecycle.md → guardian.approved) (zdroj: docs/non-functional.md → Tokeny)
- Důsledek srozumitelně: „schválením získáváte zástupcovský přístup k přihláškám tohoto dítěte.“ (zdroj: README.md → Rodič)

**Stavy:**

- Platný · už schváleno (idempotentní, přátelské) · **token vypršel** → přihláška je `Expired`; stránka má říct, co dělat (přihlásit znovu), ne jen selhat.

**Otevřené UX otázky:**

- **[K rozhodnutí]** Ve spec neexistuje akce „zamítnout“ — jen schválit, nebo nechat propadnout. Nabídnout explicitní zamítnutí (laskavější, končí rychleji), nebo zůstat spec-minimální? Přidání by se propsalo do stavového automatu.
- **[K rozhodnutí]** Po schválení: slepá ulička s poděkováním vs. pozvánka k založení účtu (zástupce je teď v systému rodičem a příště bude platit).

#### S6 · Rozcestník stavu přihlášky (tokenový odkaz nebo účet)

**Účel:** Jedna stránka, která vždy odpoví na aktuální stav, blokující bránu a další krok. Sdílený návrh se self-managementem D1.

**Publikum:** Vracející se uživatelé s jedinou otázkou: „co chybí?“

**Obsah a pole:**

- Stav jako lidský popisek + checklist bran (zástupce → dokumenty → platba), zrcadlící závazné pořadí bran. (zdroj: docs/registration-lifecycle.md)
- **Sekce dokumentů:** každý povinný dokument se stavem (`pending / uploaded / approved / rejected`) a komentářem posuzovatele; nové nahrání na místě; limity 10 MB, PDF/JPG/PNG/HEIC. (zdroj: docs/data-model.md → REGISTRATION_DOCUMENT) (zdroj: docs/non-functional.md)
- **Platební sekce:** QR, účet, *zbývající* částka, VS/SS, splatnost; dosud přijaté platby (částečné platby jsou první třída). (zdroj: docs/payment-matching.md)
- Akce: storno s poplatkem spočteným z termínovaných pravidel akce k dnešnímu datu; změna/přidání účastníků. (zdroj: README.md → Ceny a storna, Přihlašování na akce)

**Stavy:**

- Vykreslení potřebuje všech devět stavů životního cyklu — včetně těch nepříjemných: `Overpayment` („přišlo nám víc než cena; ozve se vám účetní“ — řešení je její, ne uživatelovo), pád zpět do `PendingDocuments` po zamítnutí, terminální `Canceled`/`Expired` (jen ke čtení, s „přihlásit znovu“, dokud je okno otevřené). (zdroj: docs/payment-matching.md → Přeplatek)
- Zamčená varianta pro náhradníky: brány viditelné, ale výslovně zamčené do přijetí nabídky. (zdroj: docs/registration-lifecycle.md)

**Mobil:** QR blok nesmí poskakovat při načítání dat (rezervovat místo); nahrávání dokumentů rovnou z fotoaparátu; změny stavu hlášené čtečkám obrazovky.

**Otevřené UX otázky:**

- **[K rozhodnutí]** Znázornění postupu u stavu, který se legálně umí vrátit zpět (rozhodnutí D5) — checklist, časová osa, nebo prostý stavový řádek.
- **[K rozhodnutí]** Potvrzení storna: formulace náhledu poplatku a zda „i zaplacená jde stornovat“ zaslouží extra tření. Vratka je ruční a mimo systém — nastavení očekávání je čisté copywriting. (zdroj: docs/payment-matching.md → Přeplatek a vratka)

#### S7 · Stránka nabídky náhradníkovi (z e-mailu)

**Účel:** Proměnit uvolněné místo v potvrzeného účastníka dřív, než nabídka propadne.

**Publikum:** Někdo, kdo napůl zapomněl, že se hlásil — a teď běží 48hodinové odpočítávání.

**Obsah a pole:**

- Rekapitulace akce, výslovný termín (`expires_at`), co přijetí odemkne (dokumenty k nahrání, splatnost platby — běží od původního podání nebo je pevným datem, takže může být „ihned“). (zdroj: docs/data-model.md → SUBSTITUTE_OFFER) (zdroj: docs/payment-matching.md → Splatnost)
- Akce přijmout; guard: místo musí být v okamžiku přijetí stále volné. (zdroj: docs/registration-lifecycle.md → substitute.offer.accepted)

**Stavy:**

- Platná · **propadlá** (nabídka vypršela; zůstáváte na čekací listině — spec výslovně říká, že se stav nemění) · místo už není volné · už přijato.

**Otevřené UX otázky:** **[K rozhodnutí]** Ve spec chybí akce „odmítnout“ — odmítnutí by vedoucímu uvolnilo ruce k novému výběru hned, místo čekání 48 h. Stojí za návrh jako změna spec. A dále: odpočet vs. prosté datum termínu.

### A4 · Schválení zákonným zástupcem (odkaz z e-mailu) P1

Zdroje: (zdroj: docs/registration-lifecycle.md → guardian.*) (zdroj: README.md → Přihlašování na akce, Rodič) (zdroj: docs/non-functional.md → Tokeny, Odchozí e-maily, Plánované úlohy)

**Cíl a spouštěč.** „Podívat se, na co se moje dítě přihlásilo, a potvrdit to jedním klepnutím.“ Spouštěč: nezletilý (< 18 let) se přihlásil sám — nemá navázaného rodiče, který by přihlášku provedl — a zadal e-mail zákonného zástupce. **Aktér:** zástupce bez účtu a bez jakéhokoli předchozího kontaktu se systémem; jediný vstupní bod je e-mail. Právně významný moment: schválením vzniká trvalá vazba rodič↔dítě. (zdroj: README.md → Přihlašování na akce)

#### Průběh

1. **Vznik žádosti** — `evaluate()` vrátí `PendingGuardian` (osoba je nezletilá bez aktivní vazby na rodiče); systém pošle zástupci e-mail s odkazem a nastaví lhůtu — výchozích 7 dní, konfiguruje se v nastavení oddílu. Do schválení se přihláška **nepočítá do kapacity** a nezletilý nesmí nahrávat dokumenty. (zdroj: docs/registration-lifecycle.md → guardian.requested)
2. **Větev: chybí datum narození** — Bránu zástupce nelze vyhodnotit — systém si vyžádá doplnění (resp. e-mail zástupce) a přihláška zůstává ve stavu `New`. (zdroj: docs/registration-lifecycle.md → Guardy a invarianty)
3. **Zástupce otevře odkaz** — Token je náhodný (min. 128 bitů), vázaný na jednu přihlášku a platný jen po schvalovací lhůtu; opravňuje výhradně k operacím nad touto přihláškou. (zdroj: docs/non-functional.md → Tokeny)
4. **Schválení** — Guard: token platný, lhůta neuplynula. Efekt: zapíše se `guardian_approved_at`, **vznikne vazba rodič↔dítě** a přihláška se přepočítá — pokračuje standardním flow (dokumenty, výzva k platbě). (zdroj: docs/registration-lifecycle.md → guardian.approved)
5. **Větev: lhůta uplyne** — Denní job převede přihlášku do `Expired` a notifikuje účastníka. Terminální stav — oživení znamená novou přihlášku. (zdroj: docs/registration-lifecycle.md → guardian.expired) (zdroj: docs/non-functional.md → Plánované úlohy)

**Data:** `REGISTRATION.guardian_email / guardian_approval_token / guardian_approved_at`, `PARENT_CHILD` (vznik při schválení). (zdroj: docs/data-model.md) **Notifikace:** žádost zástupci; zpráva účastníkovi při expiraci. Každé odeslání se eviduje. (zdroj: docs/non-functional.md → Odchozí e-maily) **Admin protějšek:** B3 (vedoucí vidí přihlášky ve stavu čekání na zástupce).

#### Obrazovky

#### A4-S1 · E-mail se žádostí o schválení

**Účel:** Přimět rodiče, který o systému nikdy neslyšel, aby klikl na odkaz — a nepovažoval zprávu za phishing.

**Publikum:** Zástupce v mobilním e-mailovém klientu, bez kontextu; dost možná první kontakt celé rodiny se systémem.

**Obsah a pole:**

- Spec definuje jen mechaniku: „e-mail zástupci s odkazem, nastavení lhůty“. (zdroj: docs/registration-lifecycle.md → guardian.requested) *(odvozeno)* Obsah navrhujeme: kdo se přihlásil (dítě), na co (akce, termín, oddíl, cena), dokdy je třeba schválit, jediné výrazné CTA na schvalovací stránku.
- Lhůta formulovaná jako konkrétní datum ve formátu spec (`Středa 29.7.`), ne „za 7 dní“. (zdroj: docs/non-functional.md → Lokalizace)

**Stavy:**

- Doručeno · odraženo — odražené a odmítnuté adresy se u osoby označují, aby se na neplatnou adresu neposílalo donekonečna; jenže zástupce nemusí být osobou v systému a dítě se o odražení jinak nedozví. (zdroj: docs/non-functional.md → Odchozí e-maily)

**Otevřené UX otázky:**

- **[K rozhodnutí]** Spec nedává nezletilému možnost opravit překlep v e-mailu zástupce ani žádost znovu odeslat — bez toho žádost tiše propadne po 7 dnech. Navrhnout „znovu odeslat / opravit e-mail“ v rozcestníku S6?
- **[K rozhodnutí]** Odesílatel a předmět: oddíl může mít vlastní SMTP — má e-mail nést jméno oddílu, nebo systému? Katalog notifikací je deklarovaná mezera. (zdroj: TODO.md #3)

#### A4-S2 · Schvalovací stránka

**Účel:** Plná specifikace je schválená obrazovka S5 flow A3 — ověřit a schválit do minuty, s vysvětlením vzniku trvalé vazby. Zde jen doplnění za flow A4.

**Stavy:**

- Navíc k S5: **přihláška mezitím stornována** (storno je možné i ze stavu `PendingGuardian`) — stránka má říct, že není co schvalovat, a nevytvářet vazbu. (zdroj: docs/registration-lifecycle.md) *(odvozeno)*

**Mobil:** Celá stránka na jednu obrazovku telefonu: souhrn + jedno tlačítko; žádné přihlašování, žádná registrace účtu před schválením.

**Otevřené UX otázky:** Viz S5 (chybějící „zamítnout“, krok po schválení). Obojí je rozhodnutí D2.

### A5 · Správa přihlášky bez účtu (tokenový rozcestník) P1

Zdroje: (zdroj: README.md → Přihlašování na akce) (zdroj: docs/non-functional.md → Tokeny) (zdroj: docs/registration-lifecycle.md → Guardy a invarianty) (zdroj: docs/payment-matching.md → Přeplatek a vratka)

**Cíl a spouštěč.** „Zjistit, co mé přihlášce chybí, a vyřídit to — bez hesla a bez účtu.“ Spouštěč: odkaz z potvrzovacího e-mailu. Token je vázaný na jednu přihlášku, platí do konce akce a nikdy nezpřístupní seznam osob ani jiné akce; e-mail s odkazem je tedy klíč. (zdroj: docs/non-functional.md → Tokeny) Přes odkaz lze přihlášku stornovat, měnit a přidávat další účastníky, i založit účet. (zdroj: README.md → Přihlašování na akce)

#### Průběh

1. **Otevření odkazu → rozcestník** — Stránka S6: lidský popisek stavu, checklist bran (zástupce → dokumenty → platba), další krok. Aktér bez účtu se do auditního logu zapisuje e-mailem. (zdroj: docs/audit-log.md) (zdroj: README.md → Auditní log)
2. **Vyřízení blokující brány** — Podle stavu: sledovat schválení zástupcem (A4), nahrát dokumenty (A6), zaplatit (A7).
3. **Větev: storno** — Možné z každého nekoncového stavu, i ze zaplaceno. Poplatek dle termínovaných storno pravidel akce k datu storna; vratka se eviduje jako záporná alokace a vyplácí ji účetní mimo systém. Stornovanou přihlášku nelze oživit. (zdroj: docs/registration-lifecycle.md → Guardy a invarianty) (zdroj: docs/payment-matching.md → Přeplatek a vratka)
4. **Větev: přidání dalšího účastníka / změna** — Přidaný účastník vzniká jako dílčí přihláška (`parent_registration_id`) s vlastním stavem a VS; vyhodnocuje se nezávisle. (zdroj: README.md → Přihlašování na akce) (zdroj: docs/data-model.md → REGISTRATION) (zdroj: docs/registration-lifecycle.md → Dílčí přihlášky)
5. **Větev: založení účtu** — Z odkazu lze založit účet; po založení se propojí s existující osobou (flow A11). (zdroj: README.md → Přihlašování na akce)
6. **Konec platnosti tokenu** — Token pro správu přihlášky platí do konce akce. (zdroj: docs/non-functional.md → Tokeny) Co se ztraceným odkazem, spec neřeší — viz otevřené otázky.

**Data:** `REGISTRATION` (+ dílčí), `REGISTRATION_DOCUMENT`, `PAYMENT_ALLOCATION`, `AUDIT_LOG` (aktér e-mailem). **Notifikace:** podle vyvolaných akcí (viz A6, A7). **Admin protějšky:** B3–B6.

#### Obrazovky

#### A5-S1 · Rozcestník stavu přihlášky

**Účel:** Plně specifikováno jako schválená obrazovka S6 flow A3; toto flow je jejím vlastníkem pro přístup tokenem. Navíc: hlavička musí jasně říkat, čí přihláška a jaká akce — jeden rodič může mít otevřené tři tokenové odkazy pro tři děti a nesmí je zaměnit (rozhodnutí D3).

**Stavy:**

- Navíc k S6: **token po konci akce** — platnost skončila; stránka má vysvětlit proč a odkázat na založení účtu, ne vrátit holou 404. (zdroj: docs/non-functional.md → Tokeny) *(odvozeno)*

**Otevřené UX otázky:** **[K rozhodnutí]** Ztracený e-mail s odkazem spec neřeší: nabídnout „znovu poslat odkaz na e-mail přihlášky“, nebo odkázat jen na vedoucího? Bez odpovědi je to hlavní budoucí supportní scénář.

#### A5-S2 · Potvrzení storna

**Účel:** Zabránit omylnému nevratnému kroku a poctivě říct, kolik bude storno stát.

**Publikum:** Rodič rušící účast, často ve stresu (nemoc dítěte); nesmí odejít s falešným očekáváním vratky.

**Obsah a pole:**

- Náhled poplatku: procento z termínovaného pravidla platného k dnešnímu datu + výsledná částka a případná vratka. (zdroj: README.md → Ceny a storna) (zdroj: docs/data-model.md → CANCELLATION_RULE)
- Očekávání: vratku vyplácí účetní ve své bance mimo systém — žádné „peníze do 3 dnů“. (zdroj: docs/payment-matching.md → Přeplatek a vratka)
- Nevratnost: stornovanou přihlášku nelze obnovit; nové přihlášení = nová přihláška (dokud je okno otevřené). (zdroj: docs/registration-lifecycle.md → Guardy a invarianty)

**Stavy:**

- Poplatek 0 Kč (nic zaplaceno, žádné pravidlo) · poplatek > 0 · storno zaplacené přihlášky (zvýrazněná rekapitulace vratky) · úspěch: potvrzení + e-mail. *(odvozeno)* (notifikaci o stornu spec výslovně nejmenuje — patří do chybějícího katalogu notifikací, (zdroj: TODO.md #3))

**Mobil:** Destruktivní tlačítko vizuálně odlišené a mimo dosah omylu (ne pod palcem hned po scrollu); potvrzení dvoukrokové.

**Otevřené UX otázky:** **[K rozhodnutí]** Míra tření u storna zaplacené přihlášky (rozhodnutí D5/S6): stačí dvoukrokové potvrzení, nebo vyžádat důvod (pomohl by vedoucímu vybrat náhradníka)?

#### A5-S3 · Přidání dalšího účastníka

**Účel:** Přihlásit sourozence bez nového hledání akce; vzniká dílčí přihláška s vlastním životním cyklem.

**Obsah a pole:**

- Stejný formulář jako S3 pro novou osobu; výsledek se objeví v rozcestníku jako samostatná karta s vlastním stavem, cenou a VS. (zdroj: docs/data-model.md → REGISTRATION)
- Nezletilý přidaný rodičem: vazba rodič↔dítě vzniká přihlášením dítěte rodičem, brána zástupce se tedy neotevírá. (zdroj: README.md → Rodič) *(odvozeno)* — u tokenového přístupu bez účtu ale systém nemusí vědět, že přidávající *je* rodič; viz otázka níže.

**Validace:** Shodné se S3 (whitelist jmen, datum narození, povinné číselníky); navíc kapacita akce v okamžiku přidání — plná akce nabídne místo náhradníka. (zdroj: docs/registration-lifecycle.md → registration.created)

**Otevřené UX otázky:** **[K rozhodnutí]** Kdo je „rodič“ u tokenové přihlášky bez účtu: vzniká vazba rodič↔dítě i tehdy, když dítě přidává anonymní držitel tokenu? Spec definuje vznik vazby jen „přihlášením dítěte rodičem“ a schválením zástupce — hraniční případ k vyjasnění. (zdroj: README.md → Rodič)

### A6 · Nahrání povinných dokumentů a nové nahrání po zamítnutí P1

Zdroje: (zdroj: README.md → Povinné dokumenty, Schvalování dokumentů) (zdroj: docs/non-functional.md → Úložiště souborů) (zdroj: docs/registration-lifecycle.md → document.*) (zdroj: docs/data-model.md → EVENT_DOCUMENT, REGISTRATION_DOCUMENT)

**Cíl a spouštěč.** „Vyfotit potvrzení od doktora telefonem, nahrát ho a mít jistotu, že je to vyřízené.“ Spouštěč: přihláška ve stavu `PendingDocuments`, nebo e-mail o zamítnutí. **Aktéři:** účastník/rodič (nahrává), vedoucí (posuzuje — flow B4). **Guardy:** přihláška není terminální; náhradník nahrává až po přijetí nabídky. (zdroj: docs/registration-lifecycle.md → document.uploaded)

#### Průběh

1. **Co akce vyžaduje** — Seznam povinných dokumentů definuje akce (výchozí ze šablony, přepsatelné) — např. potvrzení o lékařské způsobilosti, souhlas zákonného zástupce, kopie kartičky pojišťovny. (zdroj: README.md → Konfigurace akce, Povinné dokumenty)
2. **Nahrání** — Postupně nebo najednou; limit 10 MB na soubor, whitelist PDF/JPG/PNG/HEIC ověřený podle skutečného obsahu, ne přípony. Soubor jde do objektového úložiště pod náhodným klíčem; stahuje se krátce platnou podepsanou URL po ověření oprávnění. (zdroj: docs/non-functional.md → Úložiště souborů)
3. **Posouzení vedoucím** — Vedoucí dokument schválí, nebo zamítne s komentářem (nečitelný, prošlý, nesprávný); eviduje se kdo a kdy posoudil. Zamítnutí e-mailem vyzve účastníka k novému nahrání. (zdroj: README.md → Schvalování dokumentů)
4. **Větev: zamítnutí po zaplacení** — Přihláška se vrátí do `PendingDocuments` i ze stavu `Paid` — moment, který musí UI vysvětlit klidně (rozhodnutí D5). Otevřená otázka spec: má se to dít i po skončení přihlašování? (zdroj: docs/registration-lifecycle.md → Otevřené otázky)
5. **Vše schváleno → další brána** — Jakmile jsou všechny povinné dokumenty schválené, `evaluate()` pustí přihlášku k platbě. Po skončení akce už dokumenty stav nemění. (zdroj: docs/registration-lifecycle.md → Guardy a invarianty)

**Data:** `REGISTRATION_DOCUMENT` (stav `pending / uploaded / approved / rejected`, `review_note`). Dokumenty přihlášek jsou citlivá data; zdravotní citlivá data se mažou do 30 dnů po skončení akce. (zdroj: docs/non-functional.md → Úložiště souborů) (zdroj: README.md → Retence a GDPR) **Notifikace:** výzva při zamítnutí; nahrání lze vyžádat i připomínkou. **Admin protějšek:** B4 (fronta posuzování).

#### Obrazovky

#### A6-S1 · Sekce dokumentů v rozcestníku

**Účel:** Ukázat u každého požadovaného dokumentu, v jakém je stavu a co s ním má uživatel udělat; nahrání na místě. (Součást S6, zde v plné hloubce.)

**Publikum:** Rodič s telefonem v ruce a papírem od doktora na stole.

**Obsah a pole:**

- Řádek na každý požadovaný dokument (`EVENT_DOCUMENT.name`) se stavem `pending / uploaded / approved / rejected`; u zamítnutého viditelný komentář posuzovatele (`review_note`) a tlačítko nového nahrání. (zdroj: docs/data-model.md → REGISTRATION_DOCUMENT)
- Náhled/stažení vlastního nahraného souboru přes podepsanou URL. (zdroj: docs/non-functional.md → Úložiště souborů)
- U náhradníka je celá sekce zamčená s vysvětlením („nahrávání se odemkne po přijetí nabídky místa“). (zdroj: README.md → Povinné dokumenty)

**Stavy:**

- **Prázdný:** nic nenahráno — seznam požadavků s jasným prvním krokem.
- **Nahrávání:** ukazatel průběhu; velikost nad 10 MB zachytit před odesláním.
- **Chyba:** typ souboru se ověřuje podle obsahu na serveru — chyba musí říct „soubor není platné PDF/JPG/PNG/HEIC“, ne jen „nahrání selhalo“. (zdroj: docs/non-functional.md → Úložiště souborů)
- **Úspěch:** stav `uploaded` = „čeká na posouzení vedoucím“ — nikoli hotovo; rozdíl mezi „nahráno“ a „schváleno“ je jádro téhle obrazovky.

**Validace:** **Dané spec:** 10 MB, PDF/JPG/PNG/HEIC podle obsahu. (zdroj: docs/non-functional.md) Vše ostatní (počet souborů na dokument, minimální čitelnost) spec nedefinuje.

**Mobil:** Nahrání rovnou z fotoaparátu (`capture`); HEIC ve whitelistu je přesně kvůli fotkám z iPhonu — nenutit konverzi. Velké dotykové cíle na řádcích dokumentů.

**Otevřené UX otázky:**

- **[K rozhodnutí]** Vícestránkový dokument = více souborů k jednomu požadavku? Model má 1 soubor na `REGISTRATION_DOCUMENT`; návod „vyfoťte obě strany do jednoho PDF“ vs. povolit více záznamů. (zdroj: docs/data-model.md)
- **[K rozhodnutí]** Smí účastník nahradit už schválený dokument? Spec mlčí; má to dopad na pád stavu.
- **[K rozhodnutí]** Návod na focení (ostrost, celý papír v záběru) — levná prevence zamítnutí, spec neřeší.

#### A6-S2 · E-mail o zamítnutí dokumentu

**Účel:** Dostat nové nahrání bez paniky — zvlášť když už bylo zaplaceno.

**Publikum:** Rodič, který si myslel, že má hotovo. Komentář píše dobrovolník-vedoucí a čte rodič — tón zprostředkovává systém.

**Obsah a pole:**

- Který dokument, komentář posuzovatele (`review_note`), odkaz do rozcestníku k novému nahrání. (zdroj: README.md → Schvalování dokumentů)
- Pokud přihláška spadla zpět (i z `Paid`): vysvětlit, že platba se nikam neztratila, jen chybí dokument. (zdroj: docs/registration-lifecycle.md) *(odvozeno)* (formulace je naše, mechanika spec)

**Otevřené UX otázky:** **[K rozhodnutí]** Rámování komentáře vedoucího: citovat doslova, nebo obalit šablonou, která tón změkčí? (Rozhodnutí D8.)

### A7 · Platba přes QR, výzva k platbě a připomínky P1

Zdroje: (zdroj: docs/payment-matching.md) (zdroj: README.md → Přihlašování na akce, Modul párování plateb) (zdroj: docs/non-functional.md → Plánované úlohy)

**Cíl a spouštěč.** „Zaplatit správnou částku napoprvé — a dostat potvrzení, že dorazila.“ Spouštěč: přihláška prošla branami do `PendingPayment`; výzva přichází v potvrzovacím e-mailu. **Aktéři:** plátce (rodič), banka (Fio sync), účetní (nejasné případy — flow B6). Klíčové omezení: **částky se párují přesně, bez tolerance** — koruna vedle znamená nedoplatek nebo přeplatek. (zdroj: docs/payment-matching.md)

#### Průběh

1. **Výzva k platbě** — Potvrzení přihlášky nese QR kód + platební údaje (účet, částka, VS = přihláška, SS = akce), je-li stanovena cena. (zdroj: README.md → Přihlašování na akce)
2. **Splatnost** — Relativní: `MIN(podání + payment_due_days, začátek akce)`, výchozích 14 dní; nebo absolutní pevné datum — přihlášky podané po něm jsou splatné ihned. Změna nastavení akce nemění splatnost už podaných přihlášek. (zdroj: docs/payment-matching.md → Splatnost) (zdroj: README.md → Přihlašování na akce)
3. **Platba a automatické párování** — Transakce se stahují z banky a párují hned po importu podle seřazených pravidel (SS+VS+částka → … → VS+částka); párování běží i při vzniku přihlášky — platba smí dorazit dřív. Za každou napárovanou platbu (i částečnou) odchází potvrzení, právě jednou. (zdroj: docs/payment-matching.md → Způsoby spárování)
4. **Větev: částečná platba** — `PartialPaid` — rozcestník ukazuje zaplaceno / zbývá; doplatek vede na `Paid`. (zdroj: docs/payment-matching.md → Alokace)
5. **Větev: přeplatek** — `Overpayment` — nevrací se automaticky; účetní volí vrátit / převést na jinou přihlášku téže osoby / ponechat jako dar. Pro uživatele: „ozve se vám účetní“, žádná akce na jeho straně. (zdroj: docs/payment-matching.md → Přeplatek a vratka)
6. **Větev: rodinná platba za víc dětí** — Jedna transakce může pokrýt více přihlášek (M:N). Víc kandidátů se nikdy nealokuje automaticky — vznikne návrh pro účetní; sedí-li součet cen přesně, návrh je předvyplněný. Pro plátce to znamená prodlevu do potvrzení. (zdroj: docs/payment-matching.md → Více kandidátů)
7. **Připomínky** — Systém připomíná nezaplacené platby denním jobem, četnost dle nastavení oddílu (`reminder_frequency_days`). Vypršení nezaplacené přihlášky je ve výchozím stavu vypnuté — nikoho automaticky nevyhazujeme. (zdroj: README.md → Přihlašování na akce) (zdroj: docs/non-functional.md → Plánované úlohy) (zdroj: docs/registration-lifecycle.md → Časové lhůty)

**Data:** `PAYMENT_ALLOCATION`, `BANK_TRANSACTION` (čtení). **Notifikace:** výzva k platbě, potvrzení za každou alokaci, připomínky splatnosti. **Admin protějšek:** B6 (pracovna párování).

#### Obrazovky

#### A7-S1 · E-mail: potvrzení přihlášky s výzvou k platbě

**Účel:** Doručit platební údaje tak, aby platba proběhla napoprvé přesně — QR je hlavní cesta, ruční přepis záloha.

**Publikum:** Rodič, který zaplatí z mobilního bankovnictví na témže telefonu, kde čte e-mail.

**Obsah a pole:**

- QR kód + účet, částka (`1 250 Kč`), VS, SS, splatnost jako konkrétní datum; rozpad ceny (základ + příplatky). (zdroj: README.md → Přihlašování na akce) (zdroj: docs/payment-matching.md)
- Tokenový odkaz na rozcestník (tady bydlí i QR pro pozdější zaplacení). (zdroj: README.md → Přihlašování na akce)
- *(odvozeno)* Upozornění „plaťte přesnou částku“ — přesné párování je vlastnost systému a jeden z hlavních zdrojů budoucích dotazů. (zdroj: docs/payment-matching.md)

**Mobil:** QR čitelné i v tmavém režimu klienta (světlý podklad pod kódem); částka/VS/SS kopírovatelné jednotlivě — na jednom zařízení se QR nenaskenuje a údaje se přepisují.

**Otevřené UX otázky:** **[K rozhodnutí]** Nabádat rodiny k jedné společné platbě (pohodlné, ale končí u účetní jako ruční návrh), nebo důsledně 1 QR = 1 přihláška? Rozhodnutí D4 — probrat s personou účetní.

#### A7-S2 · Platební sekce rozcestníku

**Účel:** Vždy aktuální odpověď „kolik ještě dlužím a jak to zaplatím“. (Součást S6, zde v plné hloubce.)

**Obsah a pole:**

- Cena, dosud přijaté platby (jednotlivé alokace s daty), *zbývající* částka, splatnost, QR + údaje. (zdroj: docs/payment-matching.md → Alokace)
- Po částečné platbě se zbývající částka přepočítá; stav `PartialPaid` je normální mezistav, ne chyba.

**Stavy:**

- `PendingPayment` (nic) · `PartialPaid` (zaplaceno / zbývá) · `Paid` (hotovo, historie plateb) · `Overpayment` („přišlo víc — vyřeší účetní“) · po splatnosti (zvýraznění termínu; přihláška se sama neruší). (zdroj: docs/registration-lifecycle.md → Časové lhůty)

**Mobil:** Rezervované místo pro QR (žádný skok), tlačítka „kopírovat číslo účtu / VS / částku“.

**Otevřené UX otázky:** **[K rozhodnutí]** Jakou částku nese QR po částečné platbě? Zbývající částka je jediná, která dopáruje přesně — spec ale generování QR pro zbytek nedefinuje. Návrh: QR vždy na zbývající částku.

#### A7-S3 · E-maily: potvrzení platby a připomínka splatnosti

**Účel:** Potvrzení uzavírá smyčku důvěry („peníze došly“); připomínka musí umět rozlišit „nezačal platit“ od „zbývá doplatek“.

**Obsah a pole:**

- **Potvrzení:** za každou napárovanou platbu (i částečnou), právě jednou (`confirmation_sent_at`); částka alokace + nový stav + případný zbytek. (zdroj: docs/payment-matching.md → Potvrzení)
- **Připomínka:** četnost dle nastavení oddílu; obsah spec nedefinuje — *(odvozeno)*: zbývající částka + QR + splatnost, tón laskavý (vedoucí přihlášky sami neruší). (zdroj: README.md → Přihlašování na akce)

**Otevřené UX otázky:** **[K rozhodnutí]** Kadence a eskalace připomínek (jemná → důraznější?) a zda po splatnosti informovat i vedoucího — katalog notifikací je deklarovaná mezera. (zdroj: TODO.md #3)

### A8 · Nabídka místa náhradníkovi (přijetí do 48 h) P2

Zdroje: (zdroj: docs/registration-lifecycle.md → substitute.*) (zdroj: docs/data-model.md → SUBSTITUTE_OFFER) (zdroj: README.md → Náhradníci) (zdroj: docs/event-fields.md)

**Cíl a spouštěč.** „Uvolnilo se místo — chci ho, než propadne.“ Spouštěč: e-mail s nabídkou poté, co vedoucí vybral náhradníka po uvolnění místa (storno/expirace jiné přihlášky). **Aktér:** náhradník, který se hlásil třeba před týdny; token z e-mailu, žádný účet.

#### Průběh

1. **Čekání** — Náhradník (`category = substitute`) sedí v `New` se zamčenými branami: nepočítá se do kapacity, nemůže nahrávat dokumenty ani platit; povinné číselníky se vynucují až po schválení. (zdroj: docs/registration-lifecycle.md → Guardy a invarianty) (zdroj: docs/event-fields.md)
2. **Uvolnění místa → výběr → nabídka** — Uvolnění kapacity spustí informaci vedoucím; vedoucí vybere náhradníka (flow B5) a tomu odejde časově omezená nabídka (`SUBSTITUTE_OFFER`, výchozích 48 hodin, nastavení oddílu). (zdroj: README.md → Náhradníci) (zdroj: docs/registration-lifecycle.md → Časové lhůty)
3. **Přijetí** — Guard: nabídka platná a kapacita stále volná. Efekt: `category → participant`, odemknou se dokumenty i povinné číselníky, `evaluate()`. Pozor: relativní splatnost běží od *původního podání* — může být „ihned“. (zdroj: docs/registration-lifecycle.md → substitute.offer.accepted) (zdroj: docs/payment-matching.md → Splatnost)
4. **Větev: propadnutí** — Hodinový job nabídku nechá propadnout; **přihláška zůstává náhradníkem v `New`** beze změny a vedoucí vybírá znovu. (zdroj: docs/registration-lifecycle.md → substitute.offer.expired) (zdroj: docs/non-functional.md → Plánované úlohy)

**Data:** `SUBSTITUTE_OFFER` (token, `offered_at/expires_at`, stav), `REGISTRATION.category`. **Notifikace:** nabídka náhradníkovi; informace vedoucím při uvolnění místa. **Admin protějšek:** B5.

#### Obrazovky

#### A8-S1 · E-mail s nabídkou místa

**Účel:** Vyburcovat k rozhodnutí během dvou dnů — jasný termín, jasný důsledek nečinnosti.

**Publikum:** Někdo, kdo napůl zapomněl, že se hlásil; e-mail musí nejdřív připomenout kontext.

**Obsah a pole:**

- Akce, účastník, termín platnosti nabídky jako konkrétní datum a čas (`expires_at`, formát `Středa 29.7. 14:19`), CTA na stránku nabídky. (zdroj: docs/data-model.md → SUBSTITUTE_OFFER) (zdroj: docs/non-functional.md → Lokalizace)
- Poctivá věta o důsledku: „když nabídku nepřijmete, zůstáváte na čekací listině“ — propadnutí nic neruší. (zdroj: docs/registration-lifecycle.md → substitute.offer.expired)

**Otevřené UX otázky:** **[K rozhodnutí]** Poslat i připomínku před vypršením nabídky (např. po 24 h)? Spec definuje jen jedno odeslání; 48hodinové okno přes víkend je snadné prošvihnout.

#### A8-S2 · Stránka nabídky

**Účel:** Plně specifikováno jako schválená obrazovka S7 flow A3. Doplnění za flow A8: po přijetí přesměrovat rovnou do rozcestníku S6 s odemčenými branami — to, co teď chybí (dokumenty, povinné volby, platba se splatností možná „ihned“), je nejdůležitější sdělení hned po úspěchu.

**Stavy:** Viz S7: platná · propadlá · místo už není volné · už přijato.

### A9 · Dobrovolnická přihláška (samostatná stránka akce) P2

Zdroje: (zdroj: README.md → Konfigurace akce (Evidence dobrovolníků), Docházka) (zdroj: docs/data-model.md → EVENT, EVENT_PRICE, REGISTRATION)

**Cíl a spouštěč.** „Chci na akci pomáhat, ne jet jako účastník.“ Spouštěč: odkaz z detailu akce — nabízí se jen u akcí se zapnutou evidencí dobrovolníků. **Aktér:** dobrovolník (často rodič nebo bývalý člen). Dobrovolníci se evidují **odděleně od účastníků**, nezapočítávají se do kapacity ani do počtu náhradníků a mají vlastní cenu a vlastní okno přihlašování. (zdroj: README.md → Konfigurace akce)

#### Průběh

1. **Vstup na dobrovolnickou stránku** — Samostatná stránka per akce; dostupná jen v dobrovolnickém okně (`volunteer_registration_from/to`). (zdroj: docs/data-model.md → EVENT)
2. **Vyplnění a odeslání** — Vzniká `REGISTRATION` s `category = volunteer` a cenou typu dobrovolník; vede se ve zvláštním seznamu. (zdroj: docs/data-model.md → REGISTRATION, EVENT_PRICE) *(odvozeno)* Dál běží standardní životní cyklus přihlášky (brány, platba dobrovolnické ceny, tokenový odkaz) — spec odchylky pro dobrovolníky nedefinuje, jen je vyjímá z kapacity.
3. **Po akci: hodiny** — Odpracované hodiny se zapisují na docházkovém záznamu akce (flow B7); systém rozlišuje krátkodobé (do 50 h) a dlouhodobé dobrovolníky. Dobrovolníka se to na portálu netýká — jen ať ví, že hodiny eviduje oddíl. (zdroj: README.md → Docházka)

**Admin protějšek:** B2 (zapnutí evidence, okno, cena), B7 (hodiny).

#### Obrazovky

#### A9-S1 · Dobrovolnická stránka s formulářem

**Účel:** Oddělit dobrovolnickou cestu od účastnické tak, aby se rodič omylem nepřihlásil jako dobrovolník (a naopak).

**Publikum:** Dospělí ochotní pomoct; přicházejí odkazem z detailu akce nebo přímo od vedoucího.

**Obsah a pole:**

- Zřetelné označení „dobrovolnická přihláška“ + souhrn akce; dobrovolnická cena; vlastní přihlašovací okno. (zdroj: README.md → Konfigurace akce)
- Formulář zrcadlí S3 (pole osoby, podmíněný e-mail zástupce u nezletilého — brána zástupce je vlastnost osoby, ne kategorie *(odvozeno)* z (zdroj: docs/registration-lifecycle.md)).

**Stavy:** Dobrovolnické okno ještě neotevřeno · otevřeno · uzavřeno — nezávisle na účastnickém okně. Kapacita se nehlídá (dobrovolníci se do ní nepočítají). (zdroj: README.md → Konfigurace akce)

**Mobil:** Shodné se S3; stránka musí fungovat jako samostatný cíl sdíleného odkazu („pošli to mamce“).

**Otevřené UX otázky:**

- **[K rozhodnutí]** Volí dobrovolník výběrové číselníky (strava, ubytování)? Číselníky patří akci a podmínka způsobilosti umí cílit na roli — výchozí chování pro dobrovolníky spec neurčuje. (zdroj: docs/event-fields.md)
- **[K rozhodnutí]** Platí pro dobrovolníky povinné dokumenty akce? Spec je váže na přihlášku, kategorii nerozlišuje.

### A10 · Skupinová přihláška (více účastníků v jednom formuláři) P2

Zdroje: (zdroj: README.md → Typy a šablony (Skupinové), Rodič) (zdroj: docs/data-model.md → REGISTRATION.parent_registration_id) (zdroj: docs/registration-lifecycle.md → Dílčí přihlášky) (zdroj: docs/payment-matching.md → Více kandidátů)

**Cíl a spouštěč.** „Přihlásit celou rodinu (nebo družinu) najednou.“ Spouštěč: akce typu *Skupinové* — formulář přijímá více účastníků včetně zákonných zástupců najednou. (zdroj: README.md → Typy a šablony) Nejsložitější formulář portálu; struktura je rozhodnutí D1.

#### Průběh

1. **Vyplnění více účastníků** — Jeden formulář, více osob; za každou osobu pole dle šablony + její volby číselníků (cena se počítá per účastník: základ dle typu + příplatky). (zdroj: README.md → Typy a šablony) (zdroj: docs/event-fields.md)
2. **Odeslání → hlavní + dílčí přihlášky** — Vznikne hlavní přihláška a dílčí přihlášky (`parent_registration_id`); **každá má vlastní stav, cenu a VS a vyhodnocuje se nezávisle** — sourozenci se legálně rozejdou (jeden zaplaceno, druhý čeká na dokumenty). Hlavní přihláška definuje club scope (hlídky, workshopy). (zdroj: docs/data-model.md) (zdroj: docs/registration-lifecycle.md → Dílčí přihlášky)
3. **Větev: nezletilí ve skupině** — Přihlašuje-li děti jejich rodič, vazba rodič↔dítě vzniká právě tímto přihlášením — brána zástupce se neotevře. Nezletilý bez navázaného rodiče ve skupině cizí osoby spustí bránu zástupce na své dílčí přihlášce. (zdroj: README.md → Rodič) (zdroj: docs/registration-lifecycle.md)
4. **Větev: jedna platba za všechny** — Rodič zaplatí jednou částkou: párování je M:N; víc kandidátů se nikdy nepáruje automaticky — sedí-li součet přesně, účetní potvrdí předvyplněný rozpad. (zdroj: docs/payment-matching.md → Více kandidátů)
5. **Správa skupiny** — Tokenem hlavní přihlášky (rozcestník ukazuje všechny dílčí); další účastníky lze přidávat i později (flow A5, větev přidání). (zdroj: README.md → Přihlašování na akce)

**Data:** `REGISTRATION` (strom), `PARENT_CHILD`, `REGISTRATION_FIELD_VALUE` per účastník. **Notifikace:** **[K rozhodnutí]** jedno souhrnné potvrzení, nebo e-mail per dílčí přihláška? Spec mlčí; souhrn s rozpadem VS je náš návrh. **Admin protějšek:** B3 (dílčí přihlášky v seznamu), B6.

#### Obrazovky

#### A10-S1 · Skupinový registrační formulář

**Účel:** Zvládnout 2–5 osob na telefonu bez ztráty přehledu, čí pole zrovna vyplňuji.

**Publikum:** Rodič se třemi dětmi, jedna cesta vlakem; nejtěžší formulářový úkol celého systému.

**Obsah a pole:**

- Opakovatelné bloky účastníků: pole osoby dle šablony + číselníky per osoba; podmíněný e-mail zástupce jen u nezletilého, kterého nepřihlašuje jeho rodič. (zdroj: README.md → Typy a šablony) (zdroj: docs/registration-lifecycle.md)
- Živý součet: cena per účastník + celkem za skupinu. (zdroj: docs/event-fields.md)

**Stavy:**

- Kapacita akce stačí jen pro část skupiny: spec neřeší, zda zbytek spadne mezi náhradníky — potřebné pravidlo i formulace. **[K rozhodnutí]**
- Odesílání: zamčené tlačítko; dvojité odeslání by zdvojilo celou skupinu.

**Validace:** Shodná se S3, per účastník; chyby musí ukazovat, u *koho* jsou (jméno účastníka v hlášce, ne jen „pole 3“).

**Mobil:** Sbalitelné bloky účastníků (otevřený vždy jen jeden), pojmenované jménem hned po vyplnění; „přidat účastníka“ na konci, ne v hlavičce.

**Otevřené UX otázky:** **[K rozhodnutí]** Rozhodnutí D1: jeden dlouhý formulář s bloky vs. stepper vs. „přidat dalšího po odeslání“ (spec dovoluje přidávat později). Volba určuje skelet S3 pro všechny typy akcí.

#### A10-S2 · Rozcestník skupiny (souhrn dílčích přihlášek)

**Účel:** Odpovědět „komu z nás co chybí“ — stavy dílčích přihlášek se rozcházejí a jeden souhrnný stav skupiny neexistuje (a nemá se předstírat).

**Obsah a pole:**

- Karta per účastník: stav, blokující brána, zbývající platba, vlastní VS; akce (dokumenty, storno) per dílčí přihláška — storno jedné neruší ostatní. (zdroj: docs/registration-lifecycle.md → Dílčí přihlášky) *(odvozeno)*
- Souhrn plateb skupiny: zbývá celkem (s rozpadem per VS). (zdroj: docs/payment-matching.md)

**Stavy:** Všechny kombinace stavů dílčích přihlášek; prázdný stav neexistuje (minimálně hlavní přihláška je vždy).

**Mobil:** Karty pod sebou, stav barevně + textově; „zaplatit za všechny“ jen pokud padne rozhodnutí D4 ve prospěch společné platby.

**Otevřené UX otázky:** **[K rozhodnutí]** Zobrazit jeden QR na součet skupiny (pohodlí vs. ruční potvrzování u účetní — D4), nebo QR per účastník?

### A11 · Založení účtu z odkazu po přihlášce; přihlášení vč. OAuth P2

Zdroje: (zdroj: README.md → Přihlašování na akce, Osoba vs. uživatelský účet, Přihlašování do systému) (zdroj: docs/non-functional.md → Přihlašování přes OAuth, Tokeny a ochrana přístupu)

**Cíl a spouštěč.** „Už nechci hlídat e-maily s odkazy — chci se prostě přihlásit.“ Spouštěč: odkaz získaný přihláškou na akci; po založení se účet propojí s existující osobou. (zdroj: README.md → Přihlašování na akce) Terminologie je závazná: **registrace = založení účtu**, „přihlášení“ samotné = login; účast na akci je vždy „přihláška na akci“. (zdroj: README.md → Osoba vs. uživatelský účet) Jedna osoba má nejvýše jeden účet.

#### Průběh

1. **Založení účtu z odkazu** — Z tokenového odkazu přihlášky; účet (přihlašovací e-mail + heslo, nebo OAuth) se naváže právě na existující osobu — její přihlášky se objeví v self-managementu (plocha D). (zdroj: README.md → Přihlašování na akce) (zdroj: docs/data-model.md → ACCOUNT)
2. **Přihlášení** — Heslem, nebo přes Google/Facebook (OAuth, rozsah jen `email` + `profile`). Jeden účet může mít víc propojených OAuth identit. (zdroj: README.md → Přihlašování do systému) (zdroj: docs/non-functional.md → OAuth)
3. **Větev: OAuth e-mail bez ověření** — Vrátí-li poskytovatel e-mail bez příznaku ověření, přihlášení se odmítne. (zdroj: docs/non-functional.md → OAuth)
4. **Větev: existující účet se stejným e-mailem** — Propojení OAuth identity se nabídne **až po úspěšném přihlášení heslem** — automatické propojení by umožnilo převzetí účtu přes podvržený e-mail. (zdroj: docs/non-functional.md → OAuth)
5. **Větev: zapomenuté heslo** — Reset jednorázovým tokenem s krátkou platností; chybová hláška u přihlášení i obnovy je vždy stejná bez ohledu na existenci účtu (ochrana proti výčtu účtů). (zdroj: docs/non-functional.md → Šifrování a hesla, Tokeny a ochrana přístupu)
6. **Ochrany** — Throttling podle účtu i IP; po sérii neúspěchů dočasné zamknutí a e-mail vlastníkovi. Odpojení OAuth identity jen, zbývá-li jiný způsob přihlášení. (zdroj: docs/non-functional.md → Tokeny a ochrana přístupu, OAuth)

**Data:** `ACCOUNT`, `OAUTH_IDENTITY`, `PERSON` (1:1 vazba). **Notifikace:** e-mail při zamknutí účtu; reset hesla. **Admin protějšek:** žádný (self-service).

#### Obrazovky

#### A11-S1 · Založení účtu (z odkazu po přihlášce)

**Účel:** Proměnit držitele tokenů v účet — a vysvětlit, co tím získá (všechny přihlášky na jednom místě, žádné hledání e-mailů).

**Publikum:** Rodič, který právě dokončil přihlášku; motivace je čerstvá, trpělivost nízká.

**Obsah a pole:**

- Přihlašovací e-mail (unikátní; předvyplnit kontaktním e-mailem osoby *(odvozeno)*), heslo — nebo tlačítka Google/Facebook. (zdroj: docs/data-model.md → ACCOUNT) (zdroj: docs/non-functional.md → OAuth)
- Věta o propojení: „účet se naváže na osobu Jana Nováková a její přihlášky“. (zdroj: README.md → Přihlašování na akce)

**Validace:** Pravidla hesla spec nedefinuje (hash Argon2id je backend) — součást chybějícího katalogu validací. (zdroj: TODO.md #4) **[K rozhodnutí]** Navrhnout minimální délku + ukazatel síly, bez povinných speciálních znaků.

**Stavy:** Úspěch → self-management (plocha D) · e-mail už má účet → nabídnout přihlášení (jednotná hláška, žádné „tento e-mail existuje“ mimo tento autentizovaný kontext *(odvozeno)* z ochrany proti výčtu) · OAuth odmítnut (neověřený e-mail) s vysvětlením.

**Mobil:** `type=email`, autofill/správce hesel (`autocomplete=new-password`), OAuth tlačítka v systémovém vzhledu poskytovatelů.

**Otevřené UX otázky:** **[K rozhodnutí]** Kdy a jak naléhavě účet nabízet (rozhodnutí D3): tichý odkaz v S4/S6, nebo interstitial po každé přihlášce?

#### A11-S2 · Přihlášení do systému

**Účel:** Login bez překvapení; bezpečnostní pravidla spec jsou tu přísnější než zvyk.

**Obsah a pole:**

- E-mail + heslo; tlačítka Google/Facebook; odkaz na obnovu hesla. (zdroj: README.md → Přihlašování do systému)
- Chybová hláška vždy stejná, ať účet existuje, nebo ne — **dané spec, nepřepisovat na „vstřícnější“**. (zdroj: docs/non-functional.md → Tokeny a ochrana přístupu)

**Stavy:**

- Chybné údaje (jednotná hláška) · dočasně zamčeno po sérii neúspěchů (+ e-mail vlastníkovi) · OAuth s neověřeným e-mailem odmítnut · nabídka propojení OAuth po úspěšném přihlášení heslem. (zdroj: docs/non-functional.md)

**Mobil:** Správce hesel (`autocomplete=current-password`); OAuth jako plnohodnotná primární cesta, ne drobný odkaz.

**Otevřené UX otázky:** **[K rozhodnutí]** Formulace stavu „dočasně zamčeno“: jednotnost hlášky vs. srozumitelnost pro legitimního vlastníka (který dostal e-mail). Bezpečnostní review formulací.

#### A11-S3 · Obnova hesla

**Účel:** Jednorázový tokenový reset — stručná obrazovka, spec definuje jen mechaniku.

**Obsah a pole:** Zadání e-mailu → vždy stejná odpověď („pokud účet existuje, poslali jsme odkaz“); stránka z odkazu: nové heslo, token s krátkou platností. (zdroj: docs/non-functional.md → Šifrování a hesla, Tokeny a ochrana přístupu)

**Stavy:** Token propadlý → nabídnout nové odeslání, ne slepou chybu.

### A12 · Výběr běhů workshopů po časových blocích P3

Zdroje: (zdroj: README.md → Typy a šablony (Workshopové)) (zdroj: docs/data-model.md → WORKSHOP, WORKSHOP_BLOCK, WORKSHOP_OFFERING, WORKSHOP_REGISTRATION)

**Cíl a spouštěč.** „Vybrat si, co budu v každém bloku dělat.“ Jen akce typu *Workshopové*: akce má časové bloky; workshopy a semináře se nabízejí jako **běhy** v blocích a mohou se opakovat ve více blocích; každý běh má vlastní kapacitu; **účastník si v každém bloku vybere jeden běh**. (zdroj: README.md → Typy a šablony) Spec definuje jen tolik — flow je proto úsporné.

#### Průběh

1. **Zobrazení nabídky** — Bloky (`WORKSHOP_BLOCK`: název, od–do) × běhy (`WORKSHOP_OFFERING`); u workshopu název, popis, lektor, minimální věk, potřeby, kapacita. (zdroj: docs/data-model.md)
2. **Volba jednoho běhu v každém bloku** — Volba se ukládá jako `WORKSHOP_REGISTRATION` (per osoba z club scope). Plný běh se přestane nabízet *(odvozeno)* z kapacity běhu. (zdroj: docs/data-model.md → WORKSHOP.capacity)

#### Obrazovky

#### A12-S1 · Výběr běhů

**Účel:** V každém bloku právě jedna volba, s dostatkem informací pro rozhodnutí (popis, lektor, věk, potřeby).

**Publikum:** Účastníci vybírající program; často děti/mladiství na telefonu.

**Obsah a pole:** Sekce per blok (název + čas), v ní karty běhů: název, typ (workshop/seminář), popis, lektor, min. věk, potřeby; tentýž workshop se smí objevit ve více blocích. (zdroj: docs/data-model.md)

**Stavy:** Běh plný (nenabízí se / zašedlý s „obsazeno“) · nevybraný blok (zvýrazněná nedodělka) · vše vybráno.

**Validace:** **Dané spec:** jeden běh na blok; min. věk workshopu. (zdroj: README.md → Typy a šablony) (zdroj: docs/data-model.md → WORKSHOP.min_age) Kdy je volba povinná (při odeslání vs. později), spec neurčuje.

**Mobil:** Bloky pod sebou, jeden rozbalený; volba radio-kartou, ne dropdownem — popisy je potřeba číst.

**Otevřené UX otázky:**

- **[K rozhodnutí]** Okamžik volby: součást přihlášky (S3), nebo dodatečně z rozcestníku? (Analogicky `required_phase` číselníků, ale workshopy tenhle mechanismus výslovně nemají.)
- **[K rozhodnutí]** Ukazovat zbývající volná místa běhů (tlak na rychlost vs. klid)?

### A13 · Sestavení závodní hlídky vlastníkem přihlášky (Stezka) P3

Zdroje: (zdroj: docs/race-patrols.md) (zdroj: README.md → Hlídky na závodních akcích (Stezka))

**Cíl a spouštěč.** „Poskládat z našich přihlášených dětí platné hlídky na závod.“ Jen akce typu *Stezka*, po přihlášení. **Aktér:** vlastník přihlášky (vedoucí výpravy, i přes token — mutace se logují s e-mailem aktéra). Hlídky se skládají z **club scope**: osoby vlastní přihlášky a jejích dílčích přihlášek, jejichž stav není `Canceled`, `Expired` ani `New`. (zdroj: docs/race-patrols.md → Club scope)

#### Průběh

1. **Založení hlídky** — Název (unikátní v rámci akce) + kategorie (Stezka / Pěšinka / Šerpa s dětmi / Pocestní). Hlídku vlastní zakládající přihláška; upravovat a mazat ji smí jen vlastník. (zdroj: docs/race-patrols.md → Vlastnictví a členství)
2. **Přidání členů** — Jen osoby z club scope; každá osoba nejvýše v jedné hlídce. Kapitán jen u Stezky/Pěšinky (tam povinný): automaticky první přidaný člen, lze změnit. (zdroj: docs/race-patrols.md)
3. **Kontrola složení** — Pravidla per kategorie (počty členů, způsobilost, věkové limity — např. Stezka: přesně 3, nejstarší ≤ 16, součet věků ≤ 42) ověřuje jediná čistá funkce; věk se počítá ke konci roku (výchozí) nebo k datu akce dle nastavení. Chybí-li datum narození, člena nelze plně ověřit a kontrola to hlásí. (zdroj: README.md → Hlídky (tabulka kategorií)) (zdroj: docs/race-patrols.md → Výpočet věku, Kontrola konzistence)
4. **Větev: pozdější porušení pravidel → rozpuštění** — Poruší-li hlídka pravidla po změně údajů člena (věk, příznak závodníka, kategorie), **hlídka se rozpustí** — členové se odpojí, hlídka zanikne a vlastník dostane informaci s důvodem. (zdroj: docs/race-patrols.md → Kontrola konzistence)
5. **Připomínka** — N dní před akcí systém upozorní vedoucí na závodníky bez hlídky. Rozhodčí na stanoviště přiřazuje vedoucí (flow B9), nikoli vlastník — přiřazení je výlučné s členstvím v hlídce. (zdroj: docs/race-patrols.md → Připomínka, Stanoviště a rozhodčí)

**Data:** `RACE_PATROL`, `RACE_PATROL_MEMBER`, `AUDIT_LOG` (každá mutace, aktér i e-mailem). **Notifikace:** informace o rozpuštění s důvodem; připomínka vedoucím. **Admin protějšek:** B9 (dohled, stanoviště a rozhodčí).

#### Obrazovky

#### A13-S1 · Správa hlídek (seznam + založení)

**Účel:** Přehled: kdo z club scope už je v hlídce, kdo zbývá, které hlídky jsou platné.

**Publikum:** Vedoucí výpravy s tokenovým odkazem, večer po přihlášení dvaceti dětí.

**Obsah a pole:**

- Seznam hlídek (název, kategorie, členové, stav kontroly složení) + nepřiřazené osoby z club scope; osoby z terminálních přihlášek a v `New` se nenabízejí. (zdroj: docs/race-patrols.md → Club scope)
- Založení: název (kontrola unikátnosti v akci) + kategorie s viditelnými pravidly složení (tabulka z README). (zdroj: README.md → Hlídky)

**Stavy:** Prázdný (žádná hlídka — vysvětlit pravidla kategorie) · hlídka nekompletní/neplatná (co přesně porušuje) · člen bez data narození („nelze plně ověřit“).

**Validace:** **Dané spec:** unikátní název; osoba max v jedné hlídce; pravidla složení per kategorie; způsobilost rozhodčího výlučná s hlídkou. (zdroj: docs/race-patrols.md)

**Mobil:** Skládání drag-and-drop nahradit na telefonu výběrem „přidat do hlídky“ ze seznamu; kontrolní stav hlídky viditelný bez rozkliku.

**Otevřené UX otázky:** **[K rozhodnutí]** Jak dopředu komunikovat riziko rozpuštění („změní-li se věk nebo údaje člena, hlídka se automaticky rozpustí“) — tvrdé pravidlo spec, které bez vysvětlení působí jako chyba systému.

#### A13-S2 · Detail hlídky

**Účel:** Složení dotáhnout do platného stavu — živá kontrola pravidel po každé změně (táž čistá funkce jako na serveru).

**Obsah a pole:**

- Členové s věkem dle referenčního data akce; volba kapitána (jen Stezka/Pěšinka); přidání/odebrání z club scope. (zdroj: docs/race-patrols.md → Výpočet věku)
- Vyhodnocení pravidel: počet členů, nejstarší, součet věků, způsobilost — po každé změně, s konkrétním porušením („součet věků 44 > 42“). *(odvozeno)* (živé UI; pravidla sama jsou spec)

**Stavy:** Platná · neplatná (výčet porušení) · rozpuštěná (historický záznam s důvodem — kanál informace vlastníkovi spec neurčuje, viz níže).

**Otevřené UX otázky:** **[K rozhodnutí]** „Vlastník je informován s důvodem“ — jakým kanálem? E-mail na adresu přihlášky je jediný jistý; in-app notifikace neexistují. Doplnit do katalogu notifikací. (zdroj: TODO.md #3)

### A14 · Potvrzení doporučení mentora / vedoucího (e-mail) P3

Zdroje: (zdroj: README.md → Typy a šablony (S doporučením)) (zdroj: docs/data-model.md → RECOMMENDATION)

**Cíl a spouštěč.** „Potvrdit, že tohohle člověka na kurz doporučuji — a napsat, co od něj čekám.“ Jen akce typu *S doporučením mentora/vedoucího*: formulář přihlášky má pole na kontakty a **systém osloví mentory/vedoucí o doplnění očekávání a potvrzení přihlášky**. (zdroj: README.md → Typy a šablony) **Aktér:** mentor/vedoucí — může, ale nemusí být osobou v systému (`mentor_person_id` NULL = jen e-mail). Spec definuje jen datový základ; flow je proto úsporné.

#### Průběh

1. **Uchazeč zadá kontakty** — V přihlášce (S3) pole na kontakty mentorů/vedoucích; vzniká `RECOMMENDATION` ve stavu `requested` (typ `mentor` / `leader`). (zdroj: docs/data-model.md → RECOMMENDATION)
2. **Systém osloví mentora e-mailem** — Žádost o doplnění očekávání (`expectation`) a potvrzení přihlášky. (zdroj: README.md → Typy a šablony)
3. **Větev: potvrzení / odmítnutí** — Stav `confirmed` (s `confirmed_at`), nebo `rejected`. Co odmítnutí udělá s přihláškou, spec neříká — stavový automat přihlášky bránu doporučení nemá. (zdroj: docs/data-model.md → RECOMMENDATION) (zdroj: docs/registration-lifecycle.md)

#### Obrazovky

#### A14-S1 · E-mail a potvrzovací stránka mentora

**Účel:** Další moment pro „studené“ publikum z e-mailu: mentor má za minutu pochopit, kdo ho žádá o co, napsat očekávání a rozhodnout.

**Publikum:** Mentor/vedoucí bez povinnosti mít účet; stejná vlídnost jako u zástupce (A4).

**Obsah a pole:**

- Kdo se hlásí, na jakou akci/kurz; textové pole „očekávání“ (`expectation`); akce potvrdit / odmítnout (stavy `confirmed` / `rejected`). (zdroj: docs/data-model.md → RECOMMENDATION)

**Stavy:** Čeká na vyjádření · potvrzeno (idempotentní) · odmítnuto · přihláška mezitím stornována.

**Mobil:** Jedno textové pole + dvě tlačítka; psaní delšího textu na telefonu — automaticky rostoucí textarea, průběžné ukládání konceptu netřeba (jedna obrazovka).

**Otevřené UX otázky:**

- **[K rozhodnutí]** `RECOMMENDATION` nemá vlastní token (na rozdíl od zástupce a náhradníka) — jak se mentor bezpečně dostane na stránku? Chybějící kus modelu k doplnění do spec. (zdroj: docs/data-model.md)
- **[K rozhodnutí]** Blokuje nepotvrzené/odmítnuté doporučení přihlášku (nová brána?), nebo je jen informací pro vedoucí? Bez odpovědi nelze navrhnout stav přihlášky v S6.
- **[K rozhodnutí]** Lhůta a připomínky pro mentora — spec žádnou nezná (na rozdíl od 7 dnů zástupce).

> **Sdílené obrazovky plochy A**
> Aby portál držel pohromadě, sdílí flow obrazovky záměrně: **S1** (výpis) vlastní A1, **S2** (detail) vlastní A2, **S3** (formulář) je skelet i pro A9/A10/A12, **S5** (schválení zástupce) vlastní A4, **S6** (rozcestník) vlastní A5 a jeho sekce rozpracovávají A6/A7, **S7** (nabídka) vlastní A8. Rozhodnutí D1 (struktura formuláře) a D3 (váha tokenového odkazu) je proto potřeba uzavřít před návrhem čehokoli dalšího — sahají do většiny flow výše.

## B · Plocha B — Oddílová správa

*11 flow pro vedoucí a účetní — pracovní nástroje oddílu*

> **Publikum a zásady celé plochy**
> Uživatelé plochy B jsou **HVO, VO/VD, RÁD a ÚČE** — lidé, kteří se do systému vracejí denně nebo týdně a pracují s desítkami až stovkami záznamů najednou. *(odvozeno)* Primární zařízení je **desktop** a primární vzor je **hustá tabulka s filtry**, ne karta; výjimkou je docházka (B7), která se zapisuje na akci z telefonu. Efektivita práce (méně kliků, hromadné úkony, klávesnice) tu váží víc než vysvětlující onboarding — na rozdíl od plochy A uživatel systém zná.
> Dvě systémové skutečnosti prostupují všemi flow: **oprávnění se přidělují per akce** (samo přiřazení k akci dává čtení přihlášek; čtyři samostatné příznaky: úprava akce, úprava přihlášek, úprava cen a storen, zápis docházky) (zdroj: README.md → Konfigurace akce), a **stav přihlášky se počítá, nikdy nenastavuje ručně** — vedoucí mění fakta (schválí dokument, alokuje platbu), nikdy stav (zdroj: docs/registration-lifecycle.md). Role ÚČE má přesně vymezený rozsah: přihlášky (úpravy), akce/ceny/storna/bankovní účty (čtení), párování a potvrzování plateb a výzvy (zdroj: README.md → Účetní).
> **Známé riziko**
> Matice rolí × akcí je největší přiznaná mezera specifikace (zdroj: TODO.md → Role a oprávnění) — u každé obrazovky níže proto uvádíme, z čeho přístup odvozujeme, a sporná místa značíme badgem.

*Flow B1 · Priorita P1 · Zdroje: (zdroj: README.md → Typy a šablony akcí) (zdroj: README.md → Konfigurace akce) (zdroj: AI_support.md)*

### B1 · Založení akce ze šablony

**Cíl:** „Založit tábor tak, jak ho děláme každý rok — za dvě minuty, ne za hodinu nastavování.“ **Aktéři:** HVO (README v sekci Konfigurace akce říká „Hlavní vedoucí vytváří akce“, sekce Šablony ale mluví o „HVO/Vedoucí“ — rozpor viz níže). **Vstupní bod:** tlačítko Nová akce v administraci oddílu.

#### Průběh

1. **Výběr šablony, ne typu** — Akce se zakládá ze **šablony** — přednastavené konfigurace daného typu. K jednomu typu může existovat víc šablon: systémové (spravuje ADM, dostupné všem) i vlastní oddílové. (zdroj: README.md → Typy a šablony akcí)
2. **Šablona předvyplní vše podstatné** — Povinná a nabízená pole formuláře (vč. zařazení chytrých sloupců), zapnuté subsystémy (hlídky, workshopy, doporučení, více účastníků, kurz), výchozí povinné dokumenty, výchozí ceny podle typu účastníka, splatnost, storno termíny, kapacitu a počet náhradníků, podporu dobrovolníků a referenční datum pro výpočet věku (konec roku vs. datum akce). (zdroj: README.md → Typy a šablony akcí)
3. **Větev: AI návrh akce** — Systém může navrhnout název, datum a popis podle toho, co vedoucí naposledy vytvářel, a storno termíny s procenty podle data konání. Vždy jako *návrh k úpravě*, ne automatický zápis. (zdroj: AI_support.md → Nová událost/akce)
4. **Doplnění základu a uložení** — Název, SS, začátek a konec akce, okno přihlašování, kapacita, počet náhradníků. Uložením vznikne `EVENT` se **snapshotem odkazu na šablonu i typ** a se snapshotem regionu — pozdější úprava šablony už založenou akci nemění. (zdroj: README.md → Typy a šablony akcí) (zdroj: README.md → Region)
5. **Pokračování do konfigurace (B2)** — Typy bez přihlášek (Pravidelné schůzky, Jednorázové akce) neotevírají registraci a neřeší cenu ani platbu — konfigurační kroky kolem cen, dokumentů a viditelnosti se u nich vůbec nenabízejí. (zdroj: README.md → Typy a šablony akcí)

**Výstupy:** nová akce ve stavu „nastavuje se“ *(odvozeno)* (spec žádný draft stav akce nedefinuje — viz otázka u B1-S2). **Dotčená data:** `EVENT`, `ACTION_TEMPLATE` (jen čtení). (zdroj: docs/data-model.md)

#### Obrazovky

#### B1-S1 · Výběr šablony

*Publikum: HVO, desktop*

**Účel:** Vybrat správnou šablonu tak, aby se ručně nastavovalo co nejméně — šablona je hlavní páka efektivity celého flow.

**Obsah a pole:**

- Šablony seskupené podle **typu akce** (9 typů), u každého typu stručně co zapíná/vyžaduje — přeloženo z tabulky typů. (zdroj: README.md → Typy a šablony akcí)
- Rozlišení **systémová** (ADM) vs. **oddílová** šablona; neaktivní šablony (`ACTION_TEMPLATE.active = false`) se nenabízejí. (zdroj: docs/data-model.md → ACTION_TEMPLATE)
- *(odvozeno)* U šablony náhled klíčových výchozích hodnot (ceny, splatnost, dokumenty), aby vedoucí nevybíral naslepo.

**Stavy:**

- **Prázdný:** oddíl bez vlastních šablon vidí jen systémové — to je normální stav, ne chyba.

**Otevřené UX otázky:**

- **[K rozhodnutí]** **Kdo smí akci založit?** README říká na jednom místě „Hlavní vedoucí vytváří akce“, na druhém „ze které HVO/Vedoucí zakládá konkrétní akci“ — rozpor k vyjasnění se zadavatelem dřív, než se rozhodne, komu tlačítko Nová akce zobrazit. (zdroj: README.md → Konfigurace akce, Typy a šablony akcí)
- **[K rozhodnutí]** Lze založit akci *bez* šablony (prázdnou)? Spec zakládání popisuje výhradně ze šablony. Možnosti: šablona povinná (jednodušší, konzistentní data) vs. „prázdná šablona“ jako úniková cesta.

#### B1-S2 · Založení akce — základní údaje

*Publikum: HVO, desktop*

**Účel:** Dotáhnout předvyplněnou akci do uložitelné podoby; vše ze šablony je viditelně předvyplněné a editovatelné.

**Obsah a pole:**

- Název, SS, začátek/konec akce, začátek/konec přihlašování, max. kapacita, počet náhradníků. (zdroj: README.md → Konfigurace akce)
- Tlačítko *Navrhnout pomocí AI* (název, datum, popis, storna) — výsledek se vloží do polí k úpravě. (zdroj: AI_support.md)
- Souhrn toho, co šablona nastavila „za mě“ (ceny, dokumenty, subsystémy), s odkazem do B2.

**Validace:**

- *(odvozeno)* Pořadí termínů (konec ≥ začátek, přihlašování končí nejpozději začátkem akce) spec nedefinuje — patří do validačního katalogu, který je přiznaná mezera. (zdroj: TODO.md #4)

**Otevřené UX otázky:**

- **[K rozhodnutí]** **SS akce:** generovat automaticky, nebo zadávat ručně? SS identifikuje akci při párování plateb — kolize dvou akcí se stejným SS by párování rozbila; spec o vzniku SS mlčí. (zdroj: docs/payment-matching.md)
- **[K rozhodnutí]** Spec nezná stav „rozpracovaná akce“ — je akce viditelná (dle nastavené viditelnosti) hned po uložení, nebo existuje explicitní publikace? Návrh: koncept do doby, než HVO otevře přihlašování; vyžaduje doplnění spec.

*Flow B2 · Priorita P1 · Zdroje: (zdroj: README.md → Konfigurace akce, Ceny a storna) (zdroj: docs/event-fields.md) (zdroj: docs/payment-matching.md → Splatnost)*

### B2 · Konfigurace akce

**Cíl:** „Nastavit vše, co určí, co portál ukáže a kolik kdo zaplatí.“ Nejhustší rodina nastavení v systému. **Aktéři:** HVO; vedoucí s oprávněním *úprava akce* resp. *úprava cen a storen*. (zdroj: README.md → Konfigurace akce)

#### Průběh

1. **Ceny podle typu účastníka a termínů** — Více cen platných v různých termínech pro typy: člen DU, bez DU, dobrovolník, oddílový vedoucí, dítě oddílového vedoucího, sponzorská (`EVENT_PRICE`: typ, částka, platnost od–do). Členství DU se u ceny vyhodnocuje **k roku akce**, ne k dnešku. (zdroj: README.md → Ceny a storna) (zdroj: README.md → Člen DU)
2. **Storno pravidla** — Procentní poplatky v různých termínech (`CANCELLATION_RULE`: procento, platí do). Vratky systém neřeší — jen je eviduje. (zdroj: README.md → Ceny a storna)
3. **Splatnost — výlučná volba** — Buď **relativní** (počet dní od podání, výchozích 14, nejpozději k začátku akce), nebo **absolutní** (pevné datum pro celou akci); později podaná přihláška je splatná ihned. Změna nastavení **nemění splatnost už podaných přihlášek** (u relativní varianty). (zdroj: README.md → Konfigurace akce, Přihlašování na akce) (zdroj: docs/payment-matching.md → Splatnost)
4. **Výběrové číselníky** — Libovolný počet číselníků (ubytování, strava, doprava, stanoviště…): jedno-/vícevýběrové, kapacity položek, cenové příplatky (i nulové a záporné), podmínka způsobilosti, povinná fáze (při odeslání / před výzvou k platbě / před akcí), vyplňuje účastník nebo přiřazuje vedoucí. (zdroj: docs/event-fields.md)
5. **Povinné dokumenty, viditelnost, dobrovolníci** — Seznam dokumentů se přebírá ze šablony a lze ho u akce přepsat. Viditelnost: veřejná / vnitřní / neveřejná — sdílecí odkaz má každá akce bez ohledu na úroveň. Volitelně samostatná dobrovolnická stránka s vlastní cenou a oknem; dobrovolníci mimo kapacitu. (zdroj: README.md → Konfigurace akce)
6. **Bankovní účet a místo konání** — Akce má nejvýše jeden bankovní účet a volitelně místo z lokací oddílu (GPS). (zdroj: README.md → Konfigurace akce)
7. **Větev: přiřazení vedoucích a oprávnění** — HVO přiřadí k akci konkrétní lidi (Vedoucí, Rádce, Účetní) a každému nastaví rozsah: úprava akce, úprava přihlášek, úprava cen a storen, zápis docházky. Samo přiřazení dává čtení přihlášek; eviduje se kdo a kdy přiřadil. (zdroj: README.md → Konfigurace akce) (zdroj: docs/data-model.md → EVENT_ASSIGNMENT)
8. **Větev: změna ceny za běhu** — Změna ceny nebo volby v číselníku spouští přepočet stavu (jen dokud akce neskončila) — může legálně vrátit `Paid` → `PartialPaid`. UI musí před uložením ukázat, kolika přihlášek se změna dotkne. (zdroj: docs/registration-lifecycle.md → price.changed)

**Dotčená data:** `EVENT`, `EVENT_PRICE`, `CANCELLATION_RULE`, `EVENT_FIELD(_OPTION)`, `EVENT_DOCUMENT`, `EVENT_CUSTOM_FIELD`, `EVENT_ASSIGNMENT`. (zdroj: docs/data-model.md)

#### Obrazovky

#### B2-S1 · Nastavení akce — přehled

*Publikum: HVO / vedoucí s úpravou akce, desktop*

**Účel:** Rozcestník sekcí nastavení s viditelným stavem úplnosti — vedoucí musí na jeden pohled vidět, co ještě brání otevření přihlašování.

**Obsah a pole:**

- Sekce: Základ (B1-S2 pole) · Ceny a storna · Splatnost · Číselníky · Dokumenty · Viditelnost a sdílecí odkaz · Dobrovolníci · Bankovní účet a místo · Tým akce. Sekce, které daný typ nezapíná, se nezobrazují (schůzky bez přihlášek nemají ceny ani dokumenty). (zdroj: README.md → Typy a šablony akcí)
- Sdílecí odkaz (`share_slug`) s tlačítkem kopírovat — existuje u každé akce, je to hlavní distribuční kanál i pro vnitřní a neveřejné akce. (zdroj: README.md → Viditelnost akce)

**Stavy:**

- *(odvozeno)* Upozornění „akce má cenu, ale nemá bankovní účet“ — výzva k platbě s QR pak nemá odkud vzít platební údaje; spec tuto kombinaci neřeší.

**Otevřené UX otázky:**

- **[K rozhodnutí]** Co přesně se stane u akce s cenou > 0 bez svázaného bankovního účtu (blokovat otevření přihlašování / dovolit a výzvy neodesílat)? Spec mlčí; obě varianty mají zastánce — doporučujeme blokaci s jasnou hláškou.

#### B2-S2 · Ceny, storna a splatnost

*Publikum: HVO / vedoucí s úpravou cen a storen; ÚČE jen čtení*

**Účel:** Zadat cenovou matici a storno křivku tak, aby výsledek odpovídal tomu, co uvidí rodič na portálu (A2) a co použije párování (B6).

**Obsah a pole:**

- Tabulka cen: řádek = typ účastníka (6 typů) × platnost od–do × částka v Kč. (zdroj: docs/data-model.md → EVENT_PRICE)
- Storno pravidla: procento × platí do (datum); řazeno chronologicky, aby křivka byla čitelná. (zdroj: docs/data-model.md → CANCELLATION_RULE)
- Splatnost: přepínač relativní (počet dní, výchozí 14) / absolutní (datum) — vyplňuje se právě jedno pole. (zdroj: docs/payment-matching.md → Splatnost)

**Stavy:**

- **Úspěch s dopadem:** při uložení změny ceny u akce s existujícími přihláškami zobrazit počet přihlášek, kterým se přepočte stav (vč. možného pádu z `Paid`), a vyžádat potvrzení. (zdroj: docs/registration-lifecycle.md → price.changed)

**Validace:**

- **Dle spec:** výlučnost relativní × absolutní splatnosti; částky přesně v Kč (systém nezaokrouhluje — o korunu jiná platba je nedoplatek/přeplatek, cena proto musí být zadaná na korunu přesně). (zdroj: docs/payment-matching.md)
- *(odvozeno)* Procento storna 0–100, nepřekrývající se platnosti cen téhož typu — spec nedefinuje (validační mezera). (zdroj: TODO.md #4)

**Otevřené UX otázky:**

- **[K rozhodnutí]** Chybí-li cena pro některý typ účastníka (např. sponzor), znamená to „zdarma“, nebo „tento typ se nemůže přihlásit“? Spec neurčuje a rozdíl je zásadní pro portálový formulář.

#### B2-S3 · Editor výběrového číselníku

*Publikum: HVO / vedoucí s úpravou akce, desktop*

**Účel:** Nadefinovat číselník a jeho položky tak, aby jedno univerzální UI pokrylo lůžka, stravu, dopravu, trika i stanoviště.

**Obsah a pole:**

- Hlavička: název · **veřejný popis** (uvidí účastník) · **neveřejná poznámka** (jen vedoucí) — obě pole musí být vizuálně nezaměnitelná, jedno „prosakuje“ na portál · režim `exclusive`/`shared` · vyplňuje účastník/vedoucí · max. počet voleb · povinná fáze (`on_submit` výchozí / `before_payment` / `before_event` / nepovinný) · podmínka způsobilosti (věk, členství DU, role). (zdroj: docs/event-fields.md → EVENT_FIELD)
- Položky: hodnota · kapacita (1 = unikátní, N = limit, prázdné = bez limitu) · cenový příplatek (i 0 a záporný). Naplněná položka se přestane nabízet. (zdroj: docs/event-fields.md → EVENT_FIELD_OPTION)
- *(odvozeno)* Živý náhled „jak to uvidí účastník“ vedle editoru — nejlevnější prevence chyb v popisu a příplatcích.

**Validace:**

- **Dle spec:** režim `exclusive` odpovídá kapacitě 1 u všech položek (UI kapacity zamkne); u náhradníka se povinná volba vynucuje až po přijetí nabídky — editor to nemusí řešit, jen o tom informovat. (zdroj: docs/event-fields.md)

**Otevřené UX otázky:**

- **[K rozhodnutí]** Úprava/smazání položky, kterou už si účastníci zvolili (mění cenu → `price.changed` a přepočet stavů; smazání volby spec vůbec neřeší). Návrh: smazání blokovat, dokud existují vazby; změnu příplatku potvrzovat s výčtem dotčených přihlášek.
- **[K rozhodnutí]** Jak zadávat *podmínku způsobilosti* — volný výraz, nebo skládačka (věk / členství DU / role)? Datový model má jen textové pole `condition`. (zdroj: docs/data-model.md → EVENT_FIELD)

#### B2-S4 · Dokumenty a tým akce

*Publikum: HVO, desktop*

**Účel:** Určit, co účastníci dokládají, a kdo z oddílu s akcí smí pracovat a v jakém rozsahu.

**Obsah a pole:**

- Dokumenty: název + povinný ano/ne; výchozí seznam přijde ze šablony a lze ho přepsat. (zdroj: README.md → Konfigurace akce) (zdroj: docs/data-model.md → EVENT_DOCUMENT)
- Tým akce: osoba + čtyři zaškrtávací oprávnění (úprava akce / úprava přihlášek / úprava cen a storen / zápis docházky); u řádku kdo a kdy přiřadil. (zdroj: docs/data-model.md → EVENT_ASSIGNMENT)
- Vysvětlující řádek: samo přiřazení = čtení přihlášek; Rádce může dostat samotný zápis docházky bez přístupu k přihláškám a platbám. (zdroj: README.md → Konfigurace akce, Docházka)

**Otevřené UX otázky:**

- **[K rozhodnutí]** Rádci „nevidí citlivá data dětí“, ale přiřazení k akci dává čtení přihlášek — co přesně Rádce v přihláškách vidí (jen jména? bez dokumentů a plateb?) spec nedefinuje. Přímý důsledek mezery v matici rolí. (zdroj: README.md → Rádce) (zdroj: TODO.md → Role a oprávnění)

*Flow B3 · Priorita P1 · Zdroje: (zdroj: docs/registration-lifecycle.md) (zdroj: README.md → Přihlašování na akce) (zdroj: docs/data-model.md)*

### B3 · Přehled a detail přihlášek akce

**Cíl:** „Vidět na jednu obrazovku, kdo jede, kdo co dluží a kde se co zaseklo — a zasáhnout bez telefonování.“ Denní nástroj vedoucích. **Aktéři:** každý přiřazený k akci (čtení); zásahy dle oprávnění *úprava přihlášek*; ÚČE (úpravy přihlášek z titulu role). (zdroj: README.md → Konfigurace akce, Účetní)

#### Průběh

1. **Otevření přehledu přihlášek akce** — Tabulka všech přihlášek s kategoriemi Účastník / Dobrovolník / Náhradník a devíti stavy životního cyklu. Do kapacity se počítají jen přihlášky `category = participant` v nekoncovém stavu mimo `New` a `PendingGuardian` — čítač kapacity musí počítat stejně. (zdroj: docs/registration-lifecycle.md → Stavy)
2. **Detail přihlášky** — Vyplněná pole a volby číselníků, dokumenty se stavy, platby (alokace) a splatnost, historie změn stavu s časem a původcem. (zdroj: docs/data-model.md → REGISTRATION) (zdroj: docs/registration-lifecycle.md → Události)
3. **Zásah = změna faktu, nikdy stavu** — Vedoucí schvaluje dokumenty (B4), přiřazuje leader-číselníky (stanoviště), ÚČE alokuje platby (B6) — po každé změně se stav přepočte funkcí `evaluate()`. UI záměrně nenabízí žádný „přepnout stav“. (zdroj: docs/registration-lifecycle.md → Princip)
4. **Větev: storno vedoucím** — Storno je možné z každého nekoncového stavu, i ze zaplaceno; poplatek podle storno pravidel akce k datu storna, vratka se eviduje jako záporná alokace. Terminální stav je konečný — obnovení = nová přihláška. (zdroj: docs/registration-lifecycle.md → Guardy a invarianty)
5. **Větev: zrušení celé akce** — Hromadné `Canceled` s nulovým storno poplatkem a vratkami. Destruktivní operace zasluhující potvrzení s výčtem dopadů. (zdroj: docs/registration-lifecycle.md → event.canceled)
6. **Po skončení akce** — Automatické přepočty se zastaví s výjimkou plateb (účetní dopárovává i zpětně); dokumenty ani zástupce už stav nemění. (zdroj: docs/registration-lifecycle.md → Guardy a invarianty)

#### Obrazovky

#### B3-S1 · Tabulka přihlášek

*Publikum: vedoucí a ÚČE, desktop, denní použití*

**Účel:** Provozní kokpit akce: stavová a platební situace všech přihlášek na jedné husté obrazovce.

**Obsah a pole:**

- Hlavička: kapacita X/Y (počítáno dle pravidla výše) + počet náhradníků; okno přihlašování. (zdroj: docs/registration-lifecycle.md)
- Sloupce *(odvozeno z políREGISTRATIONa výpočtu úhrady)*: osoba, kategorie, stav (český štítek + barva, viz rozhodnutí D5 v sekci 2.3), cena, uhrazeno, splatnost (po splatnosti zvýrazněně), VS, dokumenty *schváleno/celkem*, podáno. Částky `1 250 Kč`, data `Středa 29.7.`. (zdroj: docs/non-functional.md → Lokalizace)
- Filtry: stav, kategorie, „po splatnosti“, „čeká na můj zásah“ *(odvozeno)*; rychlé hledání podle jména a VS.
- **Efektivita:** řádek = odkaz do detailu; sloupce řaditelné; výchozí řazení „vyžaduje pozornost první“ *(odvozeno)*.

**Stavy:**

- **Prázdný:** žádné přihlášky — nabídnout zkopírování sdílecího odkazu (distribuční kanál každé akce). (zdroj: README.md → Viditelnost akce)
- **Načítání:** skeleton řádků, stabilní šířky sloupců.
- Sekce Náhradníci oddělená od účastníků (jiná pravidla, zamčené brány). (zdroj: docs/registration-lifecycle.md)

**Mobil:** Sekundární: tabulka se na úzkém displeji hroutí do karet s třemi klíčovými údaji (osoba, stav, uhrazeno/cena); plná tabulka horizontálně skroluje ve vlastním kontejneru.

**Otevřené UX otázky:**

- **[K rozhodnutí]** Hromadné operace (vybrat N přihlášek → připomínka platby / storno) spec nezná; pro tábor s 80 dětmi jsou ale rozdíl mezi hodinou a minutou práce. Navrhnout jako doplněk spec.
- **[K rozhodnutí]** Export přihlášek do tabulky — spec ho garantuje jen u reportů; vedoucí ho u prezenčky reálně potřebují. (zdroj: docs/reports.md → Výstup a API)

#### B3-S2 · Detail přihlášky (pohled vedoucího)

*Publikum: vedoucí s úpravou přihlášek, ÚČE; desktop*

**Účel:** Odpovědět „proč je přihláška v tomto stavu a co s tím“ — zrcadlo účastnického stavového hubu (A5/S6), rozšířené o zásahy.

**Obsah a pole:**

- Stav + kontrolní seznam bran v závazném pořadí zástupce → dokumenty → platba; u nesplněné brány rovnou akce vedoucího. (zdroj: docs/registration-lifecycle.md → Princip)
- Osoba a vyplněná pole vč. hodnot chytrých sloupců; volby číselníků s příplatky a výslednou cenou (základ + součet příplatků). (zdroj: docs/event-fields.md)
- Dokumenty se stavy a komentáři posouzení (proklik do B4); platby: alokace s částkou, `match_method` a časem; splatnost dle výpočtu akce. (zdroj: docs/data-model.md → REGISTRATION_DOCUMENT, PAYMENT_ALLOCATION)
- Historie: každá změna stavu s časem, původcem a událostí (systémové změny bez původce). (zdroj: docs/registration-lifecycle.md → Události)
- Akce: storno s náhledem poplatku k dnešnímu datu; úprava polí; u leader-číselníků přiřazení hodnoty.

**Stavy:**

- **Chyba/hrana:** terminální přihláška je jen ke čtení s vysvětlením „obnovení = nová přihláška“; náhradník má brány zobrazené jako zamčené. (zdroj: docs/registration-lifecycle.md → Guardy a invarianty)

**Validace:**

- **Dle spec:** storno jen z nekoncového stavu; zásahy jen s oprávněním *úprava přihlášek*; žádná ruční změna stavu neexistuje. (zdroj: docs/registration-lifecycle.md)

**Otevřené UX otázky:**

- **[K rozhodnutí]** Vidí vedoucí citlivá data (zdravotní údaje per oddíl/akce) přímo v detailu přihlášky, nebo ve zvláštním režimu s auditem? Spec izoluje citlivá data per oddíl, ale nepopisuje čtecí UI. (zdroj: README.md → Retence a GDPR) (zdroj: docs/data-model.md → PERSON_SENSITIVE_DATA)

*Flow B4 · Priorita P1 · Zdroje: (zdroj: README.md → Přihlašování na akce (Schvalování dokumentů)) (zdroj: docs/registration-lifecycle.md → document.*) (zdroj: docs/non-functional.md → Úložiště souborů)*

### B4 · Fronta schvalování dokumentů

**Cíl:** „Projít nahrané papíry za jeden večer a nepustit dál nic prošlého nebo nečitelného.“ Administrativní polovina účastnického flow A6 — text zamítnutí odchází e-mailem rodiči, je to tedy i copywriterská úloha. **Aktéři:** vedoucí s oprávněním *úprava přihlášek* na akci. (zdroj: docs/registration-lifecycle.md → document.approved/rejected)

#### Průběh

1. **Dokument čeká na posouzení** — Účastník nahrává postupně nebo najednou (10 MB, PDF/JPG/PNG/HEIC ověřené podle obsahu); vedoucí u každého nahraného dokumentu vidí stav. (zdroj: README.md → Přihlašování na akce) (zdroj: docs/non-functional.md → Úložiště souborů)
2. **Náhled a rozhodnutí** — Soubor se stahuje přes krátce platnou podepsanou URL vydanou po ověření oprávnění. Vedoucí **schválí**, nebo **zamítne s komentářem** (nečitelný, prošlý, nesprávný dokument); eviduje se kdo a kdy posoudil. (zdroj: docs/non-functional.md → Úložiště souborů) (zdroj: README.md → Schvalování dokumentů)
3. **Větev: zamítnutí** — E-mail vyzve účastníka k opětovnému nahrání; přihláška zůstává — nebo se **vrátí, i ze zaplaceno** — do `PendingDocuments`, dokud nejsou všechny povinné dokumenty schválené. (zdroj: docs/registration-lifecycle.md)
4. **Vše schváleno → brána se otevírá** — `evaluate()` posune přihlášku dál (typicky do `PendingPayment`). Nahrání lze vyžádat i připomínkou. (zdroj: README.md → Schvalování dokumentů)

> **Otevřená otázka spec**
> Má zamítnutý dokument po skončení přihlašování ještě vracet přihlášku do `PendingDocuments`, nebo už jen upozornit vedoucího? Otázku si spec klade sama — rozhodnutí ovlivní kopii i chování této fronty. (zdroj: docs/registration-lifecycle.md → Otevřené otázky)

#### Obrazovky

#### B4-S1 · Fronta dokumentů s náhledem

*Publikum: vedoucí (dobrovolník po večerech), desktop*

**Účel:** Posoudit dávku dokumentů co nejrychleji: náhled + dvě tlačítka + další v řadě.

**Obsah a pole:**

- Seznam čekajících (stav `uploaded`) s osobou, typem dokumentu a časem nahrání; vpravo velký náhled souboru. (zdroj: docs/data-model.md → REGISTRATION_DOCUMENT)
- Akce: Schválit · Zamítnout s komentářem (komentář povinný — spec zamítnutí bez komentáře nezná). U zamítnutí nápověda tónu: text čte rodič v e-mailu; nabídnout předpřipravené důvody (nečitelný / prošlý / nesprávný dokument) s možností doplnění. (zdroj: README.md → Schvalování dokumentů)
- Po rozhodnutí automaticky další dokument; **Efektivita:** klávesové zkratky schválit/zamítnout/další *(odvozeno)*.
- Záložka „posouzené“ s auditem (kdo, kdy, komentář). (zdroj: docs/data-model.md → REGISTRATION_DOCUMENT)

**Stavy:**

- **Prázdný:** „vše posouzeno“ — pozitivní nulová fronta s počtem přihlášek, kterým dokumenty ještě chybí (stav `pending`, nic nenahráno).
- **Chyba:** náhled se nepodařilo načíst (podepsaná URL expirovala) → tlačítko obnovit; nikdy neukazovat surovou chybu úložiště.

**Validace:**

- **Dle spec:** posouzení jen s oprávněním *úprava přihlášek*; zamítnutí zapisuje kdo/kdy/komentář a odesílá e-mail. (zdroj: docs/registration-lifecycle.md → document.approved/rejected)

**Otevřené UX otázky:**

- **[K rozhodnutí]** Fronta per akce, nebo společná schránka přes všechny akce vedoucího? Spec popisuje jen pohled u akce; společná schránka šetří obcházení akcí.
- **[K rozhodnutí]** HEIC (z iPhonu) je povolený formát uploadu, ale prohlížeče ho nativně nezobrazí — konverze náhledu na serveru, nebo „stáhnout a otevřít“? (zdroj: docs/non-functional.md → Úložiště souborů)

*Flow B5 · Priorita P2 · Zdroje: (zdroj: README.md → Konfigurace akce (Náhradníci)) (zdroj: docs/registration-lifecycle.md → substitute.*)*

### B5 · Výběr náhradníka po uvolnění místa

**Cíl:** „Uvolnilo se místo — obsadit ho dřív, než vychladne.“ Administrativní polovina účastnického flow A8. **Aktéři:** vedoucí akce (spec neupřesňuje, které oprávnění výběr provádí — *(odvozeno)* *úprava přihlášek*).

#### Průběh

1. **Uvolnění místa spouští výběr** — Storno nebo expirace přihlášky uvolní kapacitu a informuje vedoucí akce. (zdroj: docs/registration-lifecycle.md → Guardy a invarianty) (zdroj: README.md → Náhradníci)
2. **Vedoucí vybere náhradníka** — Ze seznamu přihlášek `category = substitute` (čekají v `New` se zamčenými branami). Výběr je ruční — systém pořadí neslibuje. (zdroj: docs/registration-lifecycle.md → Stavy)
3. **Odchází časově omezená nabídka** — Výchozí platnost 48 hodin (nastavení oddílu); náhradník ji přijímá odkazem (A8/S7). (zdroj: docs/registration-lifecycle.md → Časové lhůty)
4. **Větev: přijetí** — Guard: nabídka platná a kapacita stále volná → kategorie se mění na účastníka, odemknou se dokumenty i povinné výběry, přepočet stavu. (zdroj: docs/registration-lifecycle.md → substitute.offer.accepted)
5. **Větev: propadnutí** — Nabídka propadá, **stav přihlášky náhradníka se nemění** — zůstává v čekání a vedoucí vybírá znovu. (zdroj: docs/registration-lifecycle.md → substitute.offer.expired)

#### Obrazovky

#### B5-S1 · Náhradníci a nabídky

*Publikum: vedoucí akce, desktop*

**Účel:** Vybrat náhradníka a sledovat běžící nabídku, aniž by vedoucí musel počítat lhůty z hlavy.

**Obsah a pole:**

- Volná místa: kolik jich právě je (kapacita − započítané přihlášky). (zdroj: docs/registration-lifecycle.md)
- Seznam náhradníků: osoba, čas podání přihlášky (informativně — spec žádné pořadí negarantuje), tlačítko *Nabídnout místo*. (zdroj: docs/registration-lifecycle.md)
- Běžící nabídky: komu, odesláno, `expires_at`, stav `offered / accepted / expired`. (zdroj: docs/data-model.md → SUBSTITUTE_OFFER)

**Stavy:**

- **Prázdný:** žádní náhradníci — sekce se skryje, na přehledu jen počet 0.
- **Hrana:** nabídka propadla → řádek se vrací mezi čekající s poznámkou „nabídka z DD.MM. propadla“, vedoucí vybírá znovu. (zdroj: docs/registration-lifecycle.md → substitute.offer.expired)

**Otevřené UX otázky:**

- **[K rozhodnutí]** Více volných míst najednou: smí běžet víc paralelních nabídek? Spec popisuje jen „po výběru náhradníka dostane náhradník nabídku“ v jednotném čísle. Paralelní nabídky obsazují rychleji, ale riskují přeslib při souběhu — guard „kapacita stále volná“ souběh sice jistí, zážitek odmítnutého náhradníka je ale špatný.

*Flow B6 · Priorita P1 · Zdroje: (zdroj: docs/payment-matching.md) (zdroj: docs/fio-sync.md) (zdroj: README.md → Modul párování plateb)*

### B6 · Pracoviště párování plateb (ÚČE)

**Cíl:** „Doparovat, co automat nezvládl — typicky rodiče, který poslal jednou platbou za tři děti — a mít frontu prázdnou.“ Nejreálnější každotýdenní bolest; pracoviště účetní je nejhustší obrazovka systému. **Aktéři:** ÚČE (párování a potvrzování plateb je jádro její role); HVO. (zdroj: README.md → Účetní)

#### Průběh

1. **Import transakcí z banky** — Samostatně za každý bankovní účet s tokenem; přírůstkově, idempotentně (`external_id`), výchozí interval 60 min, ruční stažení možné. Do párování vstupují jen příchozí platby. (zdroj: docs/fio-sync.md)
2. **Automatické párování** — Běží hned po importu *i při vzniku přihlášky* (platba může dorazit dřív než přihláška). Pravidla tvoří závazně seřazený seznam (SS+VS+částka → … → VS+částka); vyhrává první, které vrátí právě jednoho kandidáta. Částky se porovnávají **přesně, bez tolerance**. (zdroj: docs/payment-matching.md → Způsoby spárování)
3. **Fronta účetní = nerozdělený zbytek** — Hlavní pracovní fronta je `amount − Σ alokací` každé transakce; její výše a stáří jsou vidět v přehledu (pásma 0–7 / 8–30 / 30+ dní). (zdroj: docs/payment-matching.md → Více kandidátů) (zdroj: docs/reports.md → R7)
4. **Návrhy u více kandidátů** — Vrátí-li pravidlo víc kandidátů, **nikdy se nealokuje automaticky** — vznikne návrh rozdělení. Sedí-li součet zbývajících cen kandidátů přesně na částku, předvyplní se rozpad 1:N; jinak seznam kandidátů s prázdnými částkami. Návrh se neukládá — počítá se při otevření transakce, aby nezastaral. (zdroj: docs/payment-matching.md → Více kandidátů)
5. **Ruční rozdělení M:N** — Účetní rozdělí částku mezi přihlášky; žádné pravidlo nesmí alokovat víc, než na transakci zbývá. U každé části se eviduje, jak vznikla (`matched_by`, `match_method`). Za každou napárovanou platbu (i částečnou) odchází potvrzení, právě jednou. (zdroj: docs/payment-matching.md → Alokace, Potvrzení)
6. **Větev: ignorovaná transakce** — Jediný ručně nastavovaný příznak: `ignored` pro příspěvky, refundace a platby mimo systém — transakce opustí frontu, ale zůstane dohledatelná. (zdroj: docs/payment-matching.md → Více kandidátů)
7. **Větev: přeplatek — trojí volba** — Přeplatek se nikdy nevrací automaticky. Účetní volí: **vrátit** odesílateli, **převést** na jinou přihlášku téže osoby, nebo **ponechat jako dar** s poznámkou. Vratka = záporná alokace (`match_method = refund`); převod = dvojice záporná + kladná alokace téže transakce; samotnou výplatu provádí účetní ve své bance mimo systém. (zdroj: docs/payment-matching.md → Přeplatek a vratka)
8. **Větev: chyba synchronizace** — Neplatný token, nedostupné API či limit se zapíšou do `sync_state`/`sync_error`; po opakovaném selhání systém upozorní účetní a HVO. Tichý výpadek importu by se projevil až upomínkami zaplaceným lidem. (zdroj: docs/fio-sync.md → Chybové stavy) (zdroj: docs/non-functional.md → Plánované úlohy)

#### Obrazovky

#### B6-S1 · Fronta transakcí

*Publikum: ÚČE, desktop, práce v dávkách*

**Účel:** Dovést nerozdělený zbytek k nule — „inbox zero“ účetní; automaticky spárované transakce překážejí co nejméně.

**Obsah a pole:**

- Záložky per bankovní účet (synchronizují se nezávisle) s `last_sync_at` a tlačítkem ručního stažení. (zdroj: docs/fio-sync.md)
- Sloupce: datum, částka, **nerozdělený zbytek** (dominantní sloupec), odesílatel, VS/SS, zpráva pro příjemce, stav (`unmatched` / `partially_allocated` / `allocated` / `ignored`), stáří v pásmech 0–7 / 8–30 / 30+ dní. (zdroj: docs/payment-matching.md) (zdroj: docs/data-model.md → BANK_TRANSACTION)
- Výchozí filtr: jen transakce se zbytkem > 0; přepínač „zobrazit vše“. **Efektivita:** Enter otevírá detail, po vyřízení skok na další; akce *Ignorovat* dostupná přímo z řádku. *(odvozeno)*

**Stavy:**

- **Prázdný:** „vše rozděleno“ + čas posledního stažení — jistota, že prázdno ≠ výpadek importu.
- **Chyba:** banner chyby synchronizace s textem `sync_error` (token se nikdy nevypisuje) a odkazem do B10-S2. (zdroj: docs/fio-sync.md → Token)
- **Načítání:** ruční stažení běží — tlačítko deaktivované; Fio limituje volání (~1 za 30 s), opakované klikání nesmí frontovat požadavky. (zdroj: docs/fio-sync.md → Přírůstkové stahování)

**Otevřené UX otázky:**

- **[K rozhodnutí]** Přehled nedávných *automatických* alokací (log s `match_method`) — spec ho nevyžaduje, ale bez něj účetní automatu nevěří a kontroluje ručně, čímž se úspora ztrácí. Navrhujeme jako záložku.

#### B6-S2 · Detail transakce — rozdělení platby

*Publikum: ÚČE, desktop*

**Účel:** Rozdělit jednu platbu mezi přihlášky s minimem psaní — případ „rodič poslal za tři děti“ na tři kliknutí.

**Obsah a pole:**

- Hlavička transakce: částka, zbytek, odesílatel (jméno, účet), VS/SS, zpráva, datum. (zdroj: docs/data-model.md → BANK_TRANSACTION)
- **Návrh rozdělení** počítaný při otevření: kandidáti se zbývající cenou; sedí-li součet přesně, částky předvyplněné (rozpad 1:N), jinak prázdné. Shoda jména je normalizovaná a porovnává se i proti zákonným zástupcům vlastníka přihlášky. (zdroj: docs/payment-matching.md → Více kandidátů, Způsoby spárování)
- Ruční přidání řádku: hledání přihlášky *(odvozeno)* podle jména, VS a akce (spec vyhledávání nepopisuje — nutná schopnost, jinak nelze „manual“ provést).
- Existující alokace s původem (`auto` + metoda / `manual` / `refund`), časem a stavem odeslání potvrzení; ruční alokace lze kdykoli opravit. (zdroj: docs/payment-matching.md → Alokace)

**Stavy:**

- **Úspěch:** po uložení souhrn „odesláno N potvrzení, stav přihlášek přepočten“; zbytek 0 → transakce mizí z výchozího filtru fronty.
- **Hrana:** mezi otevřením a uložením přibyla alokace jinde → přepočítat a upozornit, neukládat slepě (návrh se záměrně neukládá právě kvůli zastarání). (zdroj: docs/payment-matching.md → Více kandidátů)

**Validace:**

- **Dle spec:** součet alokací ≤ částka transakce (nesmí se alokovat víc, než zbývá); částky přesně na koruny, bez zaokrouhlení; alokace nad cenu přihlášky je legální a vede k `Overpayment` — UI ji dovolí, ale zvýrazní. (zdroj: docs/payment-matching.md)

**Otevřené UX otázky:**

- **[K rozhodnutí]** AI návrh nejpravděpodobnější přihlášky při selhání SS/VS (podle částky, jména, času) je ve spec zmíněn — kde přesně ho zobrazit a jak odlišit od deterministických návrhů, aby účetní věděla, čemu věří? (zdroj: AI_support.md → Párování plateb)

#### B6-S3 · Řešení přeplatku

*Publikum: ÚČE, desktop*

**Účel:** Vyřídit přeplatek jednou ze tří cest a nechat po sobě dohledatelnou stopu.

**Obsah a pole:**

- Seznam přihlášek ve stavu `Overpayment` s výší přeplatku (`Σ alokací − cena`, počítá se, neukládá). (zdroj: docs/payment-matching.md → Přeplatek a vratka)
- Tři akce s vysvětlením důsledku: **Vrátit odesílateli** (vznikne záporná alokace; výplatu provedete ve své bance — systém ji jen eviduje) · **Převést na jinou přihlášku** (výběr omezen na přihlášky *téže osoby*; dvojice alokací, součet transakce zůstává) · **Ponechat jako dar** (povinná poznámka). (zdroj: docs/payment-matching.md → Přeplatek a vratka)

**Stavy:**

- **Prázdný:** žádné přeplatky — skrytá sekce, jen počítadlo 0 v přehledu.
- **Úspěch:** stav přihlášky se přepočte sám (typicky na `Paid`); do vyřízení zůstává v `Overpayment` a je vidět v reportu Platby. (zdroj: docs/payment-matching.md)

**Otevřené UX otázky:**

- **[K rozhodnutí]** Spec si sama klade otázku, zda převod na jinou přihlášku téže osoby nabízet automaticky, nebo vždy jen jako návrh účetní — rozhodnutí určí, jestli tato obrazovka převod předvyplňuje. (zdroj: docs/registration-lifecycle.md → Otevřené otázky)

*Flow B7 · Priorita P2 · Zdroje: (zdroj: README.md → Docházka) (zdroj: docs/data-model.md → ATTENDANCE_RECORD)*

### B7 · Zápis docházky a dobrovolnických hodin

**Cíl:** „Odškrtat, kdo dorazil, přímo na místě — klidně rádcem, klidně bez signálu na chatě.“ **Aktéři:** kdokoli s oprávněním *zápis docházky* na akci — samostatné oprávnění, které může mít i Rádce bez přístupu k přihláškám a platbám. (zdroj: README.md → Docházka)

#### Průběh

1. **Docházka se vede přímo na akci** — Samostatná docházková událost neexistuje; každá osoba má na akci nejvýše jeden záznam. Záznam rozlišuje **nepřítomnost** (zapsaný, nedorazil) od **nezapsaného** (žádný záznam). (zdroj: README.md → Docházka)
2. **Odškrtání účastníků** — U akce s přihláškami jsou obě evidence odlišené: přihláška = kdo se přihlásil, docházka = kdo skutečně přišel a kolik odpracoval. (zdroj: README.md → Docházka)
3. **Větev: dobrovolnické hodiny** — Počet hodin se zadává vždy na docházkovém záznamu téže akce; klasifikaci krátkodobý (< 50 h/rok) vs. dlouhodobý dobrovolník počítají reporty, hranice je nastavitelná v oddílu. (zdroj: README.md → Docházka) (zdroj: docs/reports.md → R5)
4. **Větev: zpětné akce a klubové schůzky** — Vedoucí může akci (např. pravidelnou schůzku) vytvořit i zpětně a rovnou vybrat účastníky ze seznamu osob oddílu. Klubová schůzka je akce bez přihlášek — docházkový záznam je jediná evidence účasti. (zdroj: README.md → Docházka)

#### Obrazovky

#### B7-S1 · Zápis docházky

*Publikum: vedoucí/rádce na akci, telefon v terénu*

**Účel:** Nejrychlejší možné odškrtávání jmen jednou rukou — jediná obrazovka plochy B navržená mobile-first. *(odvozeno)* (spec zařízení neurčuje; kontext „na akci“ telefon silně implikuje — viz rozhodnutí D9 v sekci 2.3).

**Obsah a pole:**

- Soupiska: u akce s přihláškami z aktivních přihlášek, u schůzky/zpětné akce výběr ze seznamu osob oddílu. (zdroj: README.md → Docházka)
- Trojstavový prvek na řádku: **přítomen** / **nepřítomen** (zapsán, nedorazil) / **nezapsán** (výchozí, žádný záznam) — tři stavy jsou datový fakt, ne vizuální detail. (zdroj: docs/data-model.md → ATTENDANCE_RECORD)
- U dobrovolníků pole hodin (numerická klávesnice).
- Rádce vidí jen jména — žádná citlivá data, přihlášky ani platby. (zdroj: README.md → Rádce, Docházka)

**Stavy:**

- **Úspěch:** průběžné ukládání po každém odškrtnutí (žádné velké tlačítko Uložit, které se v terénu zapomene); záznam je upsert — akce + osoba unikátní. (zdroj: docs/data-model.md → ATTENDANCE_RECORD)

**Mobil:** Cíle dotyku min. 44 px na celém řádku; abecední seznam s rychlým hledáním; vysoký kontrast pro čtení venku.

**Otevřené UX otázky:**

- **[K rozhodnutí]** Offline režim: tábořiště bez signálu je reálný scénář a spec ho neřeší. Možnosti: žádný offline (zapsat večer), fronta zápisů ve storage s pozdější synchronizací, plná PWA. Nákladové rozpětí je obrovské — rozhodnout brzy.

*Flow B8 · Priorita P2 · Zdroje: (zdroj: README.md → Hlavní vedoucí oddílu, Stav osoby, Pomocná evidence, Člen DU)*

### B8 · Evidence osob oddílu

**Cíl:** „Mít pořádek v tom, kdo je člen, kdo host, kdo je letos člen DU — a vidět to všechno v jedné tabulce.“ Základní tabulka HVO. **Aktéři:** HVO (evidence a migrace), Rádci dle oprávnění sloupců. **Pozor:** lifecycle osoby má ve spec otevřený devítibodový seznam „dořešit“ — návrh musí být defenzivní a počítat se změnami. (zdroj: README.md → Stav osoby → TODO - dořešit)

#### Průběh

1. **Evidence členů a hostů** — Registrovaný člen: jméno, příjmení, pohlaví, datum narození (povinné). Host: minimálně jméno, příjmení nebo přezdívka. Dva stavy jedné osoby v rámci oddílu, ne dvě entity. (zdroj: README.md → Hlavní vedoucí oddílu, Stav osoby)
2. **Dvě kolmé osy stavu** — Členský stav (host / registrovaný člen) × stav záznamu (aktivní / neaktivní / archivovaný). Neaktivní se nezapočítává do počtu členů a nedostává automatické výzvy; archivace = GDPR anonymizace po retenční době. (zdroj: README.md → Stav osoby)
3. **Větev: migrace host → registrovaný člen** — Provádí HVO; vyžaduje povinné datum narození. Opačný směr a guardy ostatních přechodů spec nedefinuje (TODO bod 2). (zdroj: README.md → Stav osoby)
4. **Větev: členství DU pro rok** — Samostatný záznam osoba + oddíl + rok (unikátní na osobu a rok), zakládá HVO; platbu příspěvku systém neřeší; vyprší tím, že pro nový rok záznam nevznikne. (zdroj: README.md → Člen DU)
5. **Větev: nové křestní jméno mimo whitelist** — Systém ověřuje česká křestní jména proti seznamu; HVO může v rámci oddílu přidat výjimku. Chybová hláška musí nabídnout cestu (výjimku), ne zeď. (zdroj: README.md → Deduplikace osob, merge)
6. **Hodnoty chytrých sloupců** — Vlastní sloupce oddílu/družiny s viditelností pro vlastníka účtu (nic / čtení / úprava) a oprávněním pro Rádce (nic / čtení / úprava). Hodnota vyplněná v přihlášce se ukládá k osobě. (zdroj: README.md → Pomocná evidence, Oddíl) (zdroj: docs/data-model.md → CUSTOM_FIELD)

#### Obrazovky

#### B8-S1 · Tabulka osob

*Publikum: HVO, desktop; Rádci s omezenými sloupci*

**Účel:** Jedna tabulka pro obě osy stavu, členství DU i chytré sloupce — s viditelností sloupců řízenou rolí prohlížejícího.

**Obsah a pole:**

- Sloupce: jméno, příjmení, přezdívka, datum narození, členský stav, stav záznamu, člen DU (aktuální rok ✓/–), družina, chytré sloupce oddílu. (zdroj: docs/data-model.md → PERSON, PERSON_UNIT, DU_MEMBERSHIP)
- Filtry po obou osách; výchozí pohled: jen aktivní záznamy *(odvozeno)*.
- Rádcům se chytré sloupce zobrazují/skrývají dle per-sloupcového oprávnění; citlivá data dětí nikdy. (zdroj: README.md → Pomocná evidence, Rádce)
- **Efektivita:** inline editace hodnot chytrých sloupců přímo v tabulce *(odvozeno — návrh)*; hromadné založení členství DU pro vybrané osoby na nový rok (jinak HVO kliká po jednom) *(odvozeno — návrh)*.

**Stavy:**

- **Prázdný:** nový oddíl bez osob — CTA přidat osobu.
- **Hrana:** archivované (anonymizované) osoby — reporty je počítají, ale nevypisují jmenovitě; jak je zobrazit v evidenci spec neřeší (viz otázka níže). (zdroj: docs/reports.md → Konvence výpočtu)

**Validace:**

- **Dle spec:** křestní jméno proti whitelistu s možností oddílové výjimky; datum narození povinné u registrovaného člena. (zdroj: README.md → Deduplikace, Stav osoby)

**Otevřené UX otázky:**

- **[K rozhodnutí]** Zobrazení archivovaných záznamů: skrýt úplně, nebo řádek „anonymizovaná osoba“ kvůli sedícím historickým počtům? Naráží na nevyřešený rozpor per-oddílového stavu vs. globální anonymizace. (zdroj: README.md → Stav osoby → TODO bod 4)

#### B8-S2 · Karta osoby

*Publikum: HVO, desktop*

**Účel:** Celý vztah osoby k oddílu na jednom místě: údaje, stavy, historie, přihlášky, docházka, vazby.

**Obsah a pole:**

- Údaje osoby (jméno, příjmení, přezdívka, tituly, pohlaví, datum narození, kontaktní e-mail, adresa, pojišťovna) + hodnoty chytrých sloupců. (zdroj: README.md → Osoba vs. uživatelský účet)
- Stavy s historií změn (obě osy, kdo/kdy — čte ji report Retence). (zdroj: README.md → Stav osoby) (zdroj: docs/data-model.md → PERSON_UNIT_HISTORY)
- Členství DU po letech s akcí „založit pro rok N“; vazby rodič ↔ dítě; přihlášky a docházka v oddílu; udělené souhlasy. (zdroj: README.md → Člen DU, Rodič) (zdroj: docs/data-model.md → CONSENT)
- Akce: migrovat hosta na člena (vynutí datum narození) · deaktivovat / reaktivovat · pozvat druhého zákonného zástupce (u dítěte bez rodiče schvaluje připojení HVO). (zdroj: README.md → Stav osoby, Rodič)

**Stavy:**

- **Hrana:** osoba evidovaná i v jiných oddílech — citlivá data jsou izolovaná per oddíl, karta ukazuje jen data svého oddílu. (zdroj: README.md → Retence a GDPR)

**Otevřené UX otázky:**

- **[K rozhodnutí]** Guardy deaktivace/archivace (osoba s otevřenou nezaplacenou přihláškou? s pohledávkou?) spec výslovně neřeší — do vyjasnění navrhnout konzervativní blokaci s vysvětlením. (zdroj: README.md → Stav osoby → TODO bod 2)

*Flow B9 · Priorita P3 · Zdroje: (zdroj: README.md → Družina, Hlídky na závodních akcích) (zdroj: docs/race-patrols.md)*

### B9 · Družiny a závodní agenda (hlídky, stanoviště)

**Cíl:** „Udržet strukturu oddílu (družiny) a na závodech dohlédnout, aby každá hlídka splňovala pravidla a každé stanoviště mělo rozhodčího.“ **Aktéři:** HVO (družiny), vedoucí závodní akce (dozor hlídek, přiřazování rozhodčích). Hlídky *staví* vlastníci přihlášek na ploše A (flow A13) — vedoucí je nevlastní a needituje. (zdroj: docs/race-patrols.md → Vlastnictví)

#### Průběh

1. **Družiny oddílu** — HVO definuje družiny, jejich vedoucí a členy (členové, hosté, Vedoucí i Rádci z oddílu); družina může mít vlastní chytré sloupce nad rámec oddílových. (zdroj: README.md → Družina)
2. **Dozor hlídek na akci typu Stezka** — Vedoucí vidí hlídky po kategoriích (Stezka, Pěšinka, Šerpa s dětmi, Pocestní) s výsledkem kontroly složení — počty členů, způsobilost, věkové limity podle referenčního data akce (konec roku výchozí / datum akce). (zdroj: README.md → Hlídky na závodních akcích) (zdroj: docs/race-patrols.md → Výpočet věku)
3. **Větev: porušení pravidel po změně údajů** — Hlídka se automaticky **rozpustí** — členové se odpojí, hlídka zanikne, vlastník dostane informaci s důvodem. Vedoucí to nevyvolává ani nezastaví; jeho obrazovka to musí srozumitelně zobrazit jako fakt. (zdroj: docs/race-patrols.md → Kontrola konzistence)
4. **Větev: závodníci bez hlídky** — Job N dní před akcí upozorní vedoucí na závodníky bez hlídky; chybějící datum narození brání plnému ověření a kontrola to hlásí. (zdroj: docs/race-patrols.md → Připomínka, Výpočet věku)
5. **Stanoviště a rozhodčí** — Stanoviště jsou leader-číselník (`assigned_by = leader`, `max_select = 1`); rozhodčí = dospělá osoba ≥ 16 z club scope, která není závodník ani šerpa a není v hlídce. Běžné stanoviště max. 1 rozhodčí, pseudo-stanoviště „Jakékoliv“ bez limitu; přiřazení je upsert a je výlučné s členstvím v hlídce. (zdroj: docs/race-patrols.md → Stanoviště a rozhodčí)

#### Obrazovky

#### B9-S1 · Družiny

*Publikum: HVO, desktop*

**Účel:** Jednoduchá správa struktur: družina → vedoucí, rádci, členové.

**Obsah a pole:**

- Seznam družin s počty; detail: členové s rolí (vedoucí / rádce / člen), přidávání z evidence osob oddílu; družinové chytré sloupce (stejný editor jako B10, jen s jiným rozsahem). (zdroj: docs/data-model.md → UNIT_PATROL(_MEMBER)) (zdroj: README.md → Družina)

**Stavy:**

- **Prázdný:** oddíl bez družin — družiny jsou volitelná struktura, prázdno není chyba.

#### B9-S2 · Závodní přehled — hlídky a stanoviště

*Publikum: vedoucí závodní akce, desktop*

**Účel:** Předzávodní kontrolní věž: platnost všech hlídek, závodníci bez hlídky, neobsazená stanoviště.

**Obsah a pole:**

- Hlídky po kategoriích: název (unikátní v akci), kapitán, členové s věkem k referenčnímu datu, výsledek kontroly složení (OK / co je porušené / koho nelze ověřit bez data narození). (zdroj: docs/race-patrols.md)
- Seznam závodníků bez hlídky (tentýž výpočet, který živí připomínku). (zdroj: docs/race-patrols.md → Připomínka)
- Stanoviště: mřížka stanoviště × rozhodčí; přiřazování jen z osob club scope splňujících způsobilost; obsazené stanoviště s kapacitou 1 nelze přebrat. (zdroj: docs/race-patrols.md → Stanoviště a rozhodčí)
- Historie mutací hlídek z auditního logu (založení, vstupy, odchody, rozpuštění s důvodem). (zdroj: docs/race-patrols.md → Logování)

**Stavy:**

- **Hrana:** rozpuštěná hlídka — zobrazit jako událost s důvodem („hlídka Lišky rozpuštěna: nejstarší člen překročil 16 let“), ne jen tichým zmizením ze seznamu. (zdroj: docs/race-patrols.md → Kontrola konzistence)

**Otevřené UX otázky:**

- **[K rozhodnutí]** Kdo přesně přiřazuje rozhodčí: spec říká „přiřazuje vedoucí … vybírá jen z osob vlastní přihlášky a jejích potvrzených dílčích přihlášek“ — formulace míchá vedoucího akce a vlastníka přihlášky. Vyjasnit se zadavatelem; ovlivní, zda je tato mřížka jen pro vedoucí, nebo i na ploše A. (zdroj: README.md → Stanoviště a rozhodčí)
- **[K rozhodnutí]** Může vedoucí hlídku upravit „za vlastníka“ (např. na místě před startem)? Spec: upravovat smí jen vlastník — tvrdé pravidlo, které se v den závodu potká s realitou. Navrhnout postup (správa přes token vlastníka? výjimka pro vedoucí?) jako doplněk spec. (zdroj: docs/race-patrols.md → Vlastnictví)

*Flow B10 · Priorita P2 · Zdroje: (zdroj: README.md → Hlavní vedoucí oddílu, Volitelné moduly) (zdroj: docs/fio-sync.md) (zdroj: docs/non-functional.md → Odchozí e-maily, Šifrování)*

### B10 · Nastavení oddílu

**Cíl:** „Jednorázově nastavit oddíl tak, aby všechno ostatní fungovalo samo.“ Setup-once plocha, která ale *gatuje* jiná flow: bez bankovního tokenu není párování (B6), bez pozvánek není tým. **Aktéři:** HVO; token bankovního účtu smí přepsat i ÚČE. (zdroj: docs/fio-sync.md → Token)

#### Průběh

1. **Bankovní účty** — Název, číslo účtu, kód banky; doplněním read-only Fio tokenu se aktivuje modul párování a synchronizace (přítomnost tokenu = zapnuto, žádný další přepínač neexistuje). (zdroj: README.md → Modul párování plateb) (zdroj: docs/fio-sync.md → Token)
2. **Moduly** — Párování plateb · Potvrzení o platbě (vyžaduje aktivní platební modul; ÚČE nahrává šablonu s razítkem/podpisem) · Pomocná evidence (chytré sloupce) · Vzdělávání · Reporty. Povoluje HVO v Nastavení oddílu. (zdroj: README.md → Volitelné moduly)
3. **Pozvánky rolí** — HVO vytváří účty účetním, vedoucím a rádcům — systém generuje pozvánku e-mailem (stejný mechanismus, jakým ADM zve HVO). (zdroj: README.md → Hlavní vedoucí oddílu)
4. **Lokace, pověření, chytré sloupce** — Lokace (GPS + volitelná adresa, viditelné jen v rámci oddílu, použitelné jako sídlo i místo akce); nahrání pověření od staršovstva (platnost od–do); definice chytrých sloupců s viditelností pro vlastníka a oprávněním pro Rádce. (zdroj: README.md → Oddíl, Hlavní vedoucí oddílu, Pomocná evidence)
5. **Lhůty a e-maily** — Nastavení oddílu drží: lhůtu schválení zástupcem (výchozí 7 dní), platnost nabídky náhradníkovi (48 h), četnost připomínek plateb, vypršení nezaplacených přihlášek (záměrně vypnuto), hranici dobrovolníka (50 h). Volitelně vlastní SMTP pro odchozí poštu. (zdroj: docs/registration-lifecycle.md → Časové lhůty) (zdroj: docs/reports.md → R5) (zdroj: docs/non-functional.md → Odchozí e-maily)

#### Obrazovky

#### B10-S1 · Nastavení oddílu — moduly, lhůty, tým, evidence

*Publikum: HVO, desktop, použití jednou za čas*

**Účel:** Vše „jednou nastav a zapomeň“ pohromadě, s výchozími hodnotami viditelnými i nezměněné.

**Obsah a pole:**

- Moduly s přepínači a závislostí (Potvrzení o platbě aktivovatelné jen s platebním modulem; u něj upload šablony potvrzení). (zdroj: README.md → Modul Potvrzení o platbě)
- Lhůty s výchozími hodnotami a vysvětlením dopadu; u „vypršení nezaplacených“ výslovná výstraha, proč je výchozí stav vypnuto (systém nemá sám rušit místa lidem, kteří platí pozdě). (zdroj: docs/registration-lifecycle.md → Časové lhůty)
- Tým: seznam rolí v oddílu + odeslané pozvánky se stavem; lokace; pověření (soubor + platnost); editor chytrých sloupců (název, rozsah oddíl/družina, viditelnost vlastníka, oprávnění Rádce). (zdroj: docs/data-model.md → USER_ROLE, LOCATION, MANDATE, CUSTOM_FIELD)
- Vlastní SMTP (e-mail + heslo, šifrované); trvale neodeslané e-maily se zobrazují vedoucímu, ne jen do logu. (zdroj: docs/non-functional.md → Odchozí e-maily)

**Validace:**

- **Dle spec:** šifrovaná pole (SMTP heslo) se nikdy nevypisují zpět; formulář po uložení ukazuje jen „nastaveno“. (zdroj: docs/non-functional.md → Šifrování a hesla)

**Otevřené UX otázky:**

- **[K rozhodnutí]** Odebrání role (ÚČE/VO/RÁD) spec nepopisuje — jen vytváření pozvánkou. Co s existujícími přiřazeními k akcím při odebrání role? Souvisí s mezerou v matici rolí. (zdroj: TODO.md → Role a oprávnění)

#### B10-S2 · Bankovní účty a synchronizace

*Publikum: HVO a ÚČE, desktop*

**Účel:** Napojit účet na Fio a dát jistotu, že synchronizace běží — zdraví importu je vidět tady, ne v logu.

**Obsah a pole:**

- Per účet: název, číslo, kód banky; **token** jako write-only pole (nikdy se nezobrazuje ani nevypisuje do chyb — jen „nastaven / nenastaven“, akce přepsat / odebrat); interval stahování (`sync_interval_minutes`, výchozí 60). (zdroj: docs/fio-sync.md → Token, Přírůstkové stahování)
- Zdraví synchronizace: `last_sync_at`, `sync_state` (ok / error) a text `sync_error`; tlačítko ručního stažení. (zdroj: docs/fio-sync.md → Chybové stavy)
- Instrukce u tokenu: musí být **read-only** (nesmí umožňovat zadávat platby) — systém to nemůže ověřit za uživatele, kopie ano. (zdroj: docs/fio-sync.md → Token)

**Stavy:**

- **Úspěch:** po uložení tokenu proběhne první stažení (od aktivace modulu, resp. posledních N dnů) — zobrazit průběh a výsledek jako potvrzení, že token funguje. (zdroj: docs/fio-sync.md → Přírůstkové stahování)
- **Chyba:** neplatný token → `sync_error` u účtu + notifikace ÚČE a HVO po opakovaném selhání; synchronizace se nezastaví natrvalo. (zdroj: docs/fio-sync.md → Chybové stavy)
- **Načítání:** ruční stažení respektuje rate limit Fio (~1 volání za 30 s) — tlačítko s cooldownem. (zdroj: docs/fio-sync.md)

*Flow B11 · Priorita P3 · Zdroje: (zdroj: docs/reports.md) (zdroj: README.md → Reporty)*

### B11 · Reporty oddílu

**Cíl:** „Podklady pro výroční zprávu a dotace bez ručního počítání.“ Metriky, parametry i rozsah dat jsou plně definované — tady je práce hlavně vizualizační, proto úsporný zápis. **Aktéři a rozsah:** VD jen svou družinu, VO/HVO svůj oddíl, ÚČE jen report Platby; Rádce reporty nemá. Rozsah se vynucuje na serveru z role, ne z parametru. (zdroj: docs/reports.md → Rozsah dat)

#### Průběh

1. **Výběr reportu** — Pevná sada (žádné vlastní sestavy v první fázi): Seznam akcí a docházky, Počty členů v čase, Účast na akcích, Docházka schůzek, Dobrovolnické hodiny, Retence a odchody, Platby, Vzdělávání — nabídka filtrovaná rolí. (zdroj: docs/reports.md → R1–R8, Co reporty záměrně neřeší)
2. **Parametry a zobrazení** — Období (výchozí posledních 12 měsíců), granularita měsíc/kvartál/rok, typ akce. Časová osa vrací prázdné koše jako nuly, aby graf nelhal o trendu. (zdroj: docs/reports.md → Společné parametry, Konvence výpočtu)
3. **Export** — Každý report jde exportovat do tabulky (CSV, stejná data v ploché podobě). Žádný report nevrací citlivé údaje; anonymizované osoby se počítají, ale nevypisují jmenovitě. (zdroj: docs/reports.md → Výstup a API, Konvence výpočtu)

#### Obrazovky

#### B11-S1 · Zobrazení reportu

*Publikum: HVO/VO/VD/ÚČE dle rozsahu, desktop*

**Účel:** Jeden vzor pro všech osm reportů: parametry nahoře, graf + tabulka pod nimi, export vpravo.

**Obsah a pole:**

- Parametry dle společné tabulky (období, bucket, typ akce; družinový filtr má VD vynucený). (zdroj: docs/reports.md → Společné parametry, R4)
- Časová řada + souhrny (`totals`); peníze v Kč na 2 desetinná místa, zaokrouhlení až na výstupu. (zdroj: docs/reports.md → Výstup a API)
- U R1 povinná vysvětlivka přímo v UI: kategorie osob se nevylučují, součet sloupců nemusí dát počet účastníků — spec ji výslovně požaduje. (zdroj: docs/reports.md → R1 Hrany)
- Metriky, pro které chybí data v modelu, se skrývají — neodhadují. (zdroj: docs/reports.md → Požadavky na datový model)

**Stavy:**

- **Prázdný:** období bez dat = graf nul (ne prázdná plocha) — záměr spec. (zdroj: docs/reports.md → Konvence výpočtu)
- **Načítání:** odpovědi se cachují s krátkou platností; skeleton grafu bez skoku layoutu.

**Otevřené UX otázky:**

- **[K rozhodnutí]** Typy grafů a vizuální jazyk reportů spec neurčuje (definuje jen data) — patří do pozdější dataviz fáze společné pro B11 a reporty ústředí (C4), aby obě plochy sdílely jeden vizuální systém.

> **Shrnutí plochy B**
> 11 flow, 21 obrazovek. Tři svorníky, které drží plochu pohromadě: **tabulka s filtry** jako základní vzor (B3, B6, B8), **stav se počítá z faktů** — žádné UI nesmí nabízet ruční přepnutí stavu přihlášky (zdroj: docs/registration-lifecycle.md), a **fronty s nulovým cílem** (dokumenty B4, nespárované platby B6) jako denní rytmus vedoucího i účetní. Největší návrhová rizika zůstávají mimo UX: matice rolí (dotýká se B2-S4, B3, B10) a lifecycle osoby (B8) — obě spec sama označuje za nedořešené. (zdroj: TODO.md) (zdroj: README.md → Stav osoby → TODO - dořešit)

## C · Plocha C — Správa ústředí

*Šest flow administrátora (ADM): oddíly, regiony, slučování osob, reporty, vzdělávání, šablony a whitelist*

Plocha C je agenda **ústředí**: správa oddílů a přiřazování HVO, definice a správa regionů, deduplikace osob, reporty ústředí a vzdělávání — zajišťují ji *moduly ústředí*. (zdroj: README.md → Přehled projektu) Vedle modulů existuje ještě **speciální oddíl ústředí** pro celostátní akce (nemá registrované členy) — jeho akce se ale spravují stejnými flow jako u běžného oddílu (plocha B), proto je zde neopakujeme. (zdroj: README.md → Oddíl)

Publikum celé plochy: **několik málo expertních uživatelů** s rolí ADM, kteří systém znají a pracují v něm opakovaně; rozsah dat mají definovaný jako „vše, napříč oddíly, s dimenzí region“. (zdroj: docs/reports.md → Rozsah dat) *(odvozeno)* Ve shodě s návrhem D9 počítáme u celé plochy s **desktop-first** režimem; mobilní poznámky u obrazovek jsou proto stručné — tabulky vždy scrollují ve vlastním kontejneru, akce zůstávají dosažitelné.

> **Společná nejistota plochy**
> Chybějící matice rolí a oprávnění (největší deklarovaná mezera spec) dopadá i sem: u řady operací níže spec neříká, zda je smí provést *jen* ADM, nebo i někdo jiný. Kde na tom závisí návrh obrazovky, je to označeno. (zdroj: TODO.md → Role a oprávnění (🔴 Gap))

### C1 · Správa oddílů a přiřazení HVO P3

Zdroje: (zdroj: README.md → Administrátor) (zdroj: README.md → Oddíl) (zdroj: docs/data-model.md → UNIT) (zdroj: docs/non-functional.md → Odchozí e-maily)

**Cíl a spouštěč.** „Založit nový oddíl a předat ho jeho hlavnímu vedoucímu.“ Flow je vstupní branou celé organizace: bez oddílu s aktivním HVO neexistují akce, členové ani platby. ADM spravuje oddíly a přiřazuje jim Hlavní vedoucí; účet HVO vzniká **pozvánkou e-mailem**, kterou generuje systém. (zdroj: README.md → Administrátor)

#### Průběh

1. **ADM otevře seznam oddílů** — Přehled všech oddílů v systému včetně speciálního oddílu ústředí (`is_hq`). (zdroj: docs/data-model.md → UNIT)
2. **Založení oddílu** — Název, typ (**IČO ústředí / Pobočný spolek s vlastním IČO / kolektivní člen** — bez „DU“ v názvu, vlastní IČO) a IČO. (zdroj: README.md → Oddíl) Sídlo (lokace) je volitelné a vybírá se z lokací oddílu — ty ale definuje až oddíl sám, takže při založení zůstává prázdné. (zdroj: docs/data-model.md → UNIT, LOCATION)
3. **Zařazení do regionu (volitelné)** — Oddíl je v daném okamžiku nejvýše v jednom regionu; ústředí do regionů nepatří. Detailně flow C2. (zdroj: README.md → Region)
4. **Pozvánka HVO** — ADM zadá e-mail budoucího HVO, systém vygeneruje pozvánku a odešle ji z fronty; každé odeslání se eviduje, trvale neodeslaný e-mail se zobrazí (nezmizí jen do logu). (zdroj: README.md → Administrátor) (zdroj: docs/non-functional.md → Odchozí e-maily)
5. **Větev: pozvaný už má účet** — **[K rozhodnutí]** Spec popisuje jen „vytváří účty hlavním vedoucím“. Co když pozvaný e-mail už účet má (typicky vedoucí jiného oddílu)? *(odvozeno)* Přijetí pozvánky by mělo přidat roli HVO existujícímu účtu místo zakládání druhého (jedna osoba = max. jeden účet). Datový model navíc entitu pozvánky pro role vůbec nemá (existuje jen `PARENT_INVITATION`) — platnost, opětovné odeslání a odvolání pozvánky jsou nedefinované. (zdroj: docs/data-model.md)
6. **HVO přijme pozvánku a přebírá oddíl** — Vznikne účet (heslo / OAuth) a role HVO vázaná na oddíl (`USER_ROLE`). HVO pokračuje na ploše B: bankovní účty, pozvánky ÚČE/VO/RÁD, družiny, pověření od staršovstva, moduly. (zdroj: README.md → Hlavní vedoucí oddílu)

**Dotčená data:** `UNIT`, `USER_ROLE`, `UNIT_REGION`. **Notifikace:** pozvánka HVO (obsah, platnost a připomenutí nejsou ve spec — patří do chybějícího katalogu notifikací (zdroj: TODO.md #3)).

#### Obrazovky

#### C1·S1 · Seznam oddílů

*ADM; občasná, ale klíčová práce; desktop*

**Účel:** Jediné místo, odkud ADM vidí celou organizaci: které oddíly existují, kdo je vede a kam patří.

**Obsah a pole:**

- Za oddíl: název, typ, IČO, aktuální region, HVO (přiřazen / pozvánka čeká). (zdroj: docs/data-model.md → UNIT, UNIT_REGION) *(odvozeno)* — sloupce jsou odvozené z polí entit, spec žádný výpis nedefinuje.
- Speciální oddíl ústředí zřetelně odlišený (nemá registrované členy, slouží celostátním akcím). (zdroj: README.md → Oddíl)
- Akce: založit oddíl, otevřít detail.

**Stavy:**

- **Prázdný:** čerstvá instalace — první krok onboardingu celé organizace; prázdný stav má vést rovnou k „založit první oddíl“.
- **Chyba doručení pozvánky:** trvale neodeslaný e-mail se musí zobrazit u oddílu, ne schovat do logu. (zdroj: docs/non-functional.md → Odchozí e-maily)

**Mobil:** Desktop-first *(odvozeno)*; tabulka v horizontálním scrollu, na úzkém displeji karty oddílů.

**Otevřené UX otázky:** **[K rozhodnutí]** Oddíl nemá ve spec žádný životní cyklus (zánik, pozastavení) — `UNIT` žádný stav nenese. Co s oddílem, který skončil? Skrývání vs. archivace je potřeba vyjasnit se zadavatelem. (zdroj: docs/data-model.md → UNIT)

#### C1·S2 · Detail / založení oddílu

*ADM zakládající strukturu organizace*

**Účel:** Založit či upravit oddíl a spravovat přiřazení HVO na jednom místě.

**Obsah a pole:**

- Název, typ (3 hodnoty), IČO. (zdroj: README.md → Oddíl)
- Blok HVO: aktuální HVO, nebo pole pro e-mail + „poslat pozvánku“; stav pozvánky (odesláno kdy, doručeno/nedoručeno). (zdroj: README.md → Administrátor)
- Blok region: aktuální příslušnost + odkaz na historii (flow C2). (zdroj: README.md → Region)

**Validace:**

- **Dle spec:** u typu „kolektivní člen“ nesmí být „DU“ v názvu. (zdroj: README.md → Oddíl)
- Formát a povinnost IČO spec neurčuje — spadá do deklarované mezery validací; navrhneme (kontrola modulo 11) a vrátíme do spec. (zdroj: TODO.md #4)

**Stavy:**

- **Úspěch:** po odeslání pozvánky jasné potvrzení „pozvánka odeslána na …“ — další krok je mimo systém (HVO musí kliknout v e-mailu).

**Otevřené UX otázky:**

- **[K rozhodnutí]** Výměna HVO: spec neříká, zda oddíl může být bez HVO, zda může mít HVO dva, ani co se děje s rolí odcházejícího. Potřeba doplnit do autorizační matice. (zdroj: TODO.md #1)
- **[K rozhodnutí]** Znovuodeslání a odvolání pozvánky — mechanika chybí v modelu i próze (viz větev v Průběhu).

#### C1·S3 · Pozvánka HVO (e-mail + přijímací stránka)

*budoucí HVO — dobrovolník, který systém vidí poprvé*

**Účel:** Proměnit e-mailovou pozvánku v účet s rolí HVO — a neztratit člověka na prvním kroku. E-mail je zde produktem, stejně jako u schválení zástupcem (obrazovka S5 flow A3).

**Obsah a pole:**

- E-mail: kdo zve, který oddíl, co role HVO obnáší (nastavuje účty, banku, akce); jedno tlačítko. *(odvozeno)* — obsah spec nedefinuje, patří do katalogu notifikací. (zdroj: TODO.md #3)
- Přijímací stránka: založení účtu heslem nebo přes Google/Facebook OAuth. (zdroj: README.md → Přihlašování do systému)

**Stavy:**

- Platná · už přijatá (idempotentně, přátelsky) · **expirovaná** — lhůta ale není ve spec definována. **[K rozhodnutí]** Navrhnout platnost pozvánky a chování po vypršení (požádat ADM o novou).

**Mobil:** E-mailový moment = mobil: jednosloupcový formulář, OAuth tlačítka na plnou šířku, žádné tabulky.

### C2 · Životní cyklus regionů P3

Zdroje: (zdroj: README.md → Region) (zdroj: docs/data-model.md → REGION, UNIT_REGION)

**Cíl a spouštěč.** „Přeorganizovat regiony tak, aby historické reporty zůstaly pravdivé.“ Region je vrstva `Ústředí → Region → Oddíl`; příslušnost oddílu je **verzovaná** (platnost od/do), takže lze určit region oddílu k libovolnému datu. Regiony se **nikdy nemažou** — jen se označí jako *sloučený* nebo *zrušený*. Reporty používají snapshot regionu uložený na akci při jejím vzniku; pozdější přesuny a slučování už existující reporty nemění. (zdroj: README.md → Region)

#### Průběh

1. **Vznik regionu** — ADM založí nový region (název). (zdroj: README.md → Region)
2. **Zařazení a přesun oddílů** — Přiřazení otevře příslušnost (`UNIT_REGION.valid_from`). Přesun **uzavře stávající příslušnost a založí novou** do jiného regionu — historie se nikdy nepřepisuje. (zdroj: README.md → Region)
3. **Sloučení (A + B → C)** — Zdrojové regiony se označí jako *sloučené* s odkazem na nástupnický region (`merged_into_region_id`); všem oddílům z A i B se uzavře příslušnost a otevře nová na C. (zdroj: README.md → Region) (zdroj: docs/data-model.md → REGION)
4. **Větev: rozdělení** — Spec ho definuje jen jako „opačnou operaci ke sloučení“. (zdroj: README.md → Region) **[K rozhodnutí]** Jak se oddíly rozdělí mezi nástupnické regiony, spec neříká — *(odvozeno)* jediná bezpečná interpretace je ruční rozřazení každého oddílu administrátorem v průvodci.
5. **Větev: zrušení regionu** — Stav *zrušený* v modelu i próze existuje („regiony se nemažou, jen označí stavem sloučený / zrušený“), ale operace zrušení popsaná není. **[K rozhodnutí]** Lze zrušit region s aktivními oddíly? Kam se oddíly podějí? Chybí stavový automat regionu — spec to sama uvádí jako úkol. (zdroj: TODO.md #2)
6. **Dopad na reporty — žádný zpětně** — Region akce = snapshot při vzniku akce; nové akce počítají podle aktuálního zařazení. Tuto sémantiku musí UI průvodců výslovně říkat, jinak ADM čeká přepočet historie. (zdroj: README.md → Region) (zdroj: docs/reports.md → Konvence výpočtu)

#### Obrazovky

#### C2·S1 · Seznam regionů

*ADM; vzácná, strukturální operace*

**Účel:** Přehled živé i historické struktury regionů — včetně těch sloučených a zrušených, protože nikdy nemizí.

**Obsah a pole:**

- Za region: název, stav (*aktivní / sloučený / zrušený*), počet aktuálně zařazených oddílů; u sloučeného odkaz na nástupnický region. (zdroj: docs/data-model.md → REGION)
- Sekce „oddíly bez regionu“ — oddíl je v regionu *nejvýše* jednom, tedy může být i v žádném; ústředí do regionů nepatří a v této sekci se nezobrazuje. (zdroj: README.md → Region)

**Stavy:**

- **Prázdný:** regiony jsou volitelná vrstva — prázdný stav vysvětlí, k čemu slouží (dimenze reportů ústředí), místo aby nutil k zakládání.

**Mobil:** Desktop-first *(odvozeno)*; seznam funguje i jako karty.

#### C2·S2 · Detail regionu s historií příslušností

*ADM ověřující „kam oddíl patřil k datu X“*

**Účel:** Ukázat aktuální složení regionu a verzovanou historii tak, aby šlo odpovědět na otázku k libovolnému datu — to je hlavní přislíbená vlastnost modelu. (zdroj: README.md → Region)

**Obsah a pole:**

- Aktuální oddíly (příslušnosti s `valid_to = NULL`) + akce „přesunout do jiného regionu“.
- Časová osa příslušností: oddíl, od–do, kam odešel. (zdroj: docs/data-model.md → UNIT_REGION)
- Akce: sloučit, rozdělit, zrušit (s guardy dle rozhodnutí u větví výše).

**Stavy:**

- **Sloučený / zrušený region:** jen pro čtení, s viditelným odkazem na nástupce; žádné akce kromě prohlížení historie.

**Otevřené UX otázky:** **[K rozhodnutí]** Má přesun oddílu umožnit zpětné datum (`valid_from` v minulosti)? Spec datum přechodu nedefinuje; zpětná změna by kolidovala se snapshoty už založených akcí — *(odvozeno)* bezpečnější je povolit jen „od teď“.

#### C2·S3 · Průvodce sloučením / rozdělením

*ADM při reorganizaci — jednou za pár let, s velkým dopadem*

**Účel:** Provést strukturální změnu bez překvapení: než se cokoli stane, ukázat přesně, kterým oddílům se uzavře a otevře příslušnost a že historie i reporty zůstávají.

**Obsah a pole:**

- Sloučení: výběr zdrojových regionů, název nástupnického regionu, náhled dotčených oddílů. (zdroj: README.md → Region) *(odvozeno)* Spec neříká, zda nástupce C musí být nový region, nebo smí být existující — průvodce zatím předpokládá nový.
- Rozdělení: ruční rozřazení oddílů mezi nástupce (viz větev v Průběhu).
- Stálá vysvětlivka: „Historické reporty se nemění — region akce je snapshot z okamžiku jejího vzniku.“ (zdroj: README.md → Region)

**Stavy:**

- **Úspěch:** souhrn provedených změn (uzavřeno/otevřeno příslušností), stav zdrojů → *sloučený*.
- **Chyba:** operace má být atomická — buď celá, nebo nic. *(odvozeno)* (spec transakčnost u regionů neřeší, na rozdíl od merge osob).

**Validace:** **Dle spec** jen implicitní: oddíl smí mít v jednom okamžiku jedinou příslušnost. Vše ostatní (min. počet zdrojů, prázdný nástupce) je na nás. (zdroj: README.md → Region)

### C3 · Deduplikace a slučování osob P3

Zdroje: (zdroj: docs/person-merge.md) (zdroj: README.md → Deduplikace osob, merge) (zdroj: docs/data-model.md → MERGE_REQUEST, MERGE_APPROVAL, MERGE_LOG)

**Cíl a spouštěč.** „Dvě evidence, jeden člověk — spojit je, aniž by kdokoli přišel o data nebo o kontrolu.“ Nejcitlivější a zároveň nejlépe specifikované flow plochy C: vícestranné schvalování, řešení konfliktů pole po poli a plnohodnotný revert. Kandidáti se **jen navrhují, nikdy neslučují automaticky** — rozhoduje vždy člověk. (zdroj: docs/person-merge.md → Detekce kandidátů) Žádost typicky zakládá uživatel s účtem (plocha D4) nebo rodič u duplicitního dítěte; ústředí do flow vstupuje jako dohled: povoluje potlačené dvojice a jako jediné smí provést revert. (zdroj: docs/person-merge.md → Schvalování, Revert)

#### Průběh

1. **Detekce kandidátů** — **Silný kandidát** = shodné datum narození *a* shodné normalizované jméno i příjmení (bez diakritiky, malá písmena, ořezané mezery); **slabý** = shodné datum narození a podobné příjmení, nebo shodné jméno i příjmení bez data narození. Přezdívka se počítá jako alternativa křestního jména (Pepa/Josef), sama kandidáta nezakládá. Hledá se **napříč oddíly**; duplicity uvnitř oddílu vidí HVO přímo v seznamu osob. (zdroj: docs/person-merge.md → Detekce kandidátů)
2. **Žádost o sloučení** — Strany podle druhu žádosti: `person` → iniciátor, HVO druhého oddílu a kandidát (má-li vlastní účet); `child` → rodiče obou dětí, bez aktivního rodiče nastupuje HVO oddílu, kde je dítě evidováno. Systém žádost rozešle e-mailem. (zdroj: docs/person-merge.md → Schvalování) (zdroj: README.md → Deduplikace osob, merge)
3. **Schvalování všemi stranami** — Žádost je `pending`, dokud všechny strany nerozhodly; souhlas všech → `ready`. Schvalující HVO vidí náhled obou osob k porovnání, ale **nevidí citlivá data z cizího oddílu** — jen základní slučovaná pole. (zdroj: docs/person-merge.md → Schvalování)
4. **Větev: zamítnutí nebo propadnutí** — Jediné zamítnutí → `rejected` (terminální); nerozhodnutá žádost **propadá po 30 dnech** s důvodem „bez odezvy“. Zamítnutá dvojice se **trvale potlačí** — znovu se nenabízí, dokud ji administrátor nepovolí. Ze stavu `ready` může navíc kterákoli strana souhlas odvolat → `rejected`. (zdroj: docs/person-merge.md → Schvalování, Detekce kandidátů)
5. **Volba konfliktních polí a účtu** — Ze stavu `ready` pokračuje **iniciátor**: konflikt se řeší pole po poli — prázdná strana prohrává bez volby, shodné hodnoty beze změny, různé hodnoty **vyžadují explicitní volbu A/B**. Ruční přepis se nenabízí (kvůli věrnému revertu); opravit hodnotu lze až po sloučení běžnou editací. Různá data narození → varování „pravděpodobně nejde o stejnou osobu“, potvrzuje HVO zvlášť. Mají-li obě osoby účet, vybírá se ponechaný účet (`keep_account_id`); OAuth identity se přenesou, při kolizi téhož poskytovatele vyhrává identita ponechaného účtu. (zdroj: docs/person-merge.md → Konflikt základních polí)
6. **Provedení v jedné transakci** — Vazby se **nevolí, přenášejí se všechny** (přihlášky, docházka, členství DU, vzdělání, dokumenty, alokace plateb, historie stavů); kolize unikátů řeší tabulka pravidel (silnější členský stav, aktivnější záznam, přítomnost vyhrává…). Citlivá data zůstávají per oddíl — HVO oddílu A sloučením nezíská data z oddílu B. Zdrojová osoba se nemaže, zůstává jako **tombstone** s přesměrováním. Selže-li cokoli, nezmění se nic. (zdroj: docs/person-merge.md → Přenos vazeb a kolize unikátů)
7. **Větev: blokující kolize přihlášek** — Mají-li obě osoby na téže akci **aktivní** přihlášku, sloučení se zablokuje a musí to nejdřív vyřešit vedoucí akce (je-li jedna terminální, zůstává druhá). Chybová hláška musí jmenovat akci a říct, kdo je na tahu. (zdroj: docs/person-merge.md → Přenos vazeb a kolize unikátů)
8. **Revert (administrátor)** — Jen ze stavu `completed` a jen dokud existuje `MERGE_LOG` (retence 3 roky). Vrátí se **jen to, co je ve snapshotu**; záznamy vzniklé po sloučení zůstávají u cílové osoby a systém je **vypíše před potvrzením, ne až po něm**. Zrušený účet se obnoví, jen není-li jeho e-mail mezitím obsazený — jinak se revert dokončí bez účtu a ADM dostane upozornění. Revert je jednorázový; nové sloučení = nová žádost. (zdroj: docs/person-merge.md → Revert)

**Dotčená data:** `MERGE_REQUEST`, `MERGE_APPROVAL`, `MERGE_LOG` (snapshot pro revert), `PERSON.merged_into_person_id`. **Notifikace:** rozeslání žádosti všem stranám, výsledek (dokončeno/zamítnuto/propadlo) — obsah opět patří do chybějícího katalogu. (zdroj: TODO.md #3) **Protistrany:** iniciace a souhlas kandidáta žijí na ploše D (D4), souhlas HVO na ploše B.

#### Obrazovky

#### C3·S1 · Fronta žádostí o sloučení (dohled ústředí)

*ADM; kontrola, ne exekutiva — slučují strany*

**Účel:** Přehled běžících, dokončených i potlačených žádostí — a jediné místo, kde lze potlačenou dvojici znovu povolit.

**Obsah a pole:**

- Za žádost: druh (`person`/`child`), obě osoby, stav (`pending` / `ready` / `rejected` / `completed` / `reverted`), rozhodnuté strany (např. „2 ze 3“), zbývající čas do propadnutí (`expires_at`, 30 dní). (zdroj: docs/data-model.md → MERGE_REQUEST, MERGE_APPROVAL)
- Sekce potlačených dvojic (`rejected`) s akcí „znovu povolit nabízení“. (zdroj: docs/person-merge.md → Detekce kandidátů)

**Stavy:**

- **Prázdný:** normální stav — žádosti vznikají zdola, od uživatelů; nevolat po akci.

**Mobil:** Desktop-first *(odvozeno)*; tabulka scrolluje.

**Otevřené UX otázky:** **[K rozhodnutí]** Sama spec se ptá: má být potlačení dvojice po zamítnutí trvalé, nebo jen dočasné (lidé si to mohou rozmyslet)? Rozhodnutí určí, zda tato obrazovka potřebuje i „potlačit dočasně“. (zdroj: docs/person-merge.md → Otevřené otázky)

#### C3·S2 · Porovnání osob a rozhodnutí strany

*HVO, rodič nebo kandidát — z e-mailového odkazu, možná na mobilu*

**Účel:** Dát straně jistotu, že jde o téhož člověka — a jedno nevratné rozhodnutí (jediné zamítnutí je terminální pro celou žádost).

**Obsah a pole:**

- Náhled obou osob vedle sebe — **jen základní slučovaná pole** (jméno, příjmení, přezdívka, tituly, pohlaví, datum narození, e-mail, adresa, pojišťovna); nikdy citlivá data z cizího oddílu. (zdroj: docs/person-merge.md → Schvalování, Konflikt základních polí)
- U žádosti typu `child`: vysvětlení, že druhou stranou je rodič druhého dítěte a že sloučení **nespojí účty rodičů**, jen osobu dítěte. (zdroj: docs/person-merge.md → Schvalování)
- Akce: schválit / zamítnout; text u zamítnutí říká, že dvojice se přestane nabízet. (zdroj: docs/person-merge.md → Detekce kandidátů)

**Stavy:**

- Čeká na mě · už rozhodnuto (idempotentně) · **žádost propadla** (30 dní) · žádost zamítnuta jinou stranou.
- Ve stavu `ready` (před provedením) může strana souhlas odvolat. (zdroj: docs/person-merge.md → stavový diagram)

**Mobil:** Porovnání dvou sloupců na úzkém displeji: řádek za řádkem „pole → hodnota A / hodnota B“ místo dvou sloupců vedle sebe; rozhodovací tlačítka mimo dosah omylu.

**Otevřené UX otázky:** **[K rozhodnutí]** Váha zamítnutí: jediné „ne“ ukončí žádost terminálně a dvojici potlačí. Přidat rozmyslový krok („opravdu nejde o tutéž osobu?“), nebo držet jednoklikovou jednoduchost? Spec mechaniku definuje, tón nikoli.

#### C3·S3 · Volba konfliktních polí a provedení

*iniciátor po souhlasu všech; soustředěná, jednorázová práce*

**Účel:** Rozhodnout každý konflikt volbou A/B a spustit sloučení — s vědomím, že jde o poslední krok před nevratnou (jen ADM-revertovatelnou) operací.

**Obsah a pole:**

- Řádek za slučované pole: automaticky vyřešené (jedna strana prázdná → vyplněná vyhrává; shodné → beze změny) zobrazené, ale zamčené; skutečné konflikty s povinnou volbou A/B. **Ruční přepis se nenabízí** — u pole musí jít doložit původ, jinak by revert vracel hodnotu, která v žádné z osob nebyla; text to má říct. (zdroj: docs/person-merge.md → Konflikt základních polí)
- Mají-li obě osoby účet: volba ponechaného účtu; druhý se zruší, jeho přihlašovací e-mail se uvolní, OAuth identity se přenesou. (zdroj: docs/person-merge.md → Konflikt základních polí)
- Souhrn toho, co se přenese automaticky (všechny vazby) a co zůstává per oddíl (citlivá data, dokumenty). (zdroj: docs/person-merge.md → Přenos vazeb)

**Validace:**

- **Dle spec:** bez rozhodnutí všech konfliktů nelze sloučení dokončit; různá data narození vyžadují zvláštní potvrzení HVO. (zdroj: docs/person-merge.md → Konflikt základních polí)

**Stavy:**

- **Chyba — blokující kolize:** obě osoby mají aktivní přihlášku na téže akci → sloučení se zablokuje; hláška jmenuje akci a vedoucího, který to musí vyřešit; díky transakci se nic nezměnilo. (zdroj: docs/person-merge.md → Přenos vazeb a kolize unikátů)
- **Úspěch:** `completed`; potvrzení vysvětlí tombstone („staré odkazy povedou na sjednocenou osobu“). (zdroj: docs/person-merge.md → Přenos vazeb)

**Mobil:** Proveditelné, ale nedoporučené — dlouhá rozhodovací série; rozpracovanost by neměla propadnout (volby držet, dokud je žádost `ready`).

**Otevřené UX otázky:** **[K rozhodnutí]** Směr sloučení (kdo je zdroj a kdo cíl, tj. čí `PERSON.id` přežije) spec z pohledu uživatele nepopisuje — v modelu je dané žádostí. Má ho iniciátor vidět, případně volit? Ovlivňuje to, „čí“ profil vizuálně vyhrává.

#### C3·S4 · Detail sloučení a revert

*ADM řešící stížnost na chybné spojení — pod tlakem, s právním dopadem*

**Účel:** Ukázat přesně, co sloučení provedlo, a umožnit jeho vrácení s úplným náhledem důsledků *před* potvrzením — přesně jak spec vyžaduje.

**Obsah a pole:**

- Ze snapshotu `MERGE_LOG`: stav obou osob před sloučením, rozhodnutá volba u každého konfliktního pole, seznam přenesených vazeb, záznamy zahozené kvůli kolizím, zrušený účet a jeho identity. (zdroj: docs/person-merge.md → Revert)
- Náhled revertu ve dvou sloupcích: **co se vrátí** (obsah snapshotu) vs. **co zůstane u cílové osoby** (záznamy vzniklé po sloučení — nová přihláška, platba, členství; smazaná retenční data se neobnoví, odeslané e-maily se nevrací). (zdroj: docs/person-merge.md → Revert)
- Upozornění na podmíněnou obnovu účtu: je-li e-mail mezitím obsazený, revert doběhne bez účtu a ADM dostane upozornění. (zdroj: docs/person-merge.md → Revert)

**Stavy:**

- Revertovatelné (`completed` + existující `MERGE_LOG`) · **po retenci** (3 roky — log smazán, revert nedostupný, s vysvětlením) · už revertováno (`reverted`, jednorázově; nové sloučení = nová žádost). (zdroj: docs/person-merge.md → Revert) (zdroj: README.md → Retence a GDPR)

**Otevřené UX otázky:** **[K rozhodnutí]** Druhá otevřená otázka spec: smí revert provést i HVO oddílu, kde sloučení vzniklo, nebo výhradně ADM? Určuje, zda tato obrazovka existuje i na ploše B. (zdroj: docs/person-merge.md → Otevřené otázky)

### C4 · Reporty ústředí P3

Zdroje: (zdroj: README.md → Modul reporty ústředí) (zdroj: docs/reports.md) (zdroj: docs/person-merge.md → Reportovací sloučení)

**Cíl a spouštěč.** „Vykázat unikátní počty dětí za organizaci — a obhájit je při kontrole.“ ADM vidí všechny reporty R1–R8 napříč oddíly s dimenzí region a navíc report R9 (unikátní děti), který je jen jeho. Reporty jsou read-only, idempotentní a exportovatelné do CSV; žádný report nevrací citlivé údaje. (zdroj: docs/reports.md → Rozsah dat, Výstup a API, Co reporty záměrně neřeší)

#### Průběh

1. **Výběr reportu a parametrů** — Společné parametry: oddíly (ADM může „všechny“), období od–do (default posledních 12 měsíců, `Europe/Prague`), granularita měsíc/kvartál/rok, region (jen ADM; filtruje přes snapshot akce), typ akce. Scope se vynucuje na serveru z rolí, ne z parametru. (zdroj: docs/reports.md → Společné parametry, Rozsah dat)
2. **Zobrazení a export** — Časová osa vrací i prázdné koše s nulou (graf nesmí lhát o trendu); peníze v CZK na 2 desetinná místa; každý report umí `format=csv`. Anonymizované osoby se do součtů počítají, ale nikdy nevypisují jmenovitě. (zdroj: docs/reports.md → Konvence výpočtu, Výstup a API)
3. **R9 — unikátní děti** — Základ = osoby s aspoň jednou aktivní přihláškou v období; vyloučí se osoby, které jsou všude jen hostem cizího oddílu; dvojice v `REPORT_MERGE` se počítají jako jedna osoba (union-find, tranzitivně); výsledkem je počet skupin, volitelně po regionech. Počítá se **za kalendářní rok** (vykazovací období). (zdroj: docs/reports.md → R9)
4. **Větev: reportovací sloučení kandidátů** — Systém nabídne kandidáty (shoda jména, příjmení a data narození napříč oddíly u neanonymizovaných osob) — vrací se **jen jméno, příjmení a datum narození, nic dalšího**. Reportovací sloučení nemění žádná data ani vazby, zakládá ho ústředí bez schvalování, respektuje ho jen R9 a je kdykoli zrušitelné. (zdroj: docs/reports.md → R9) (zdroj: docs/person-merge.md → Reportovací sloučení)

#### Obrazovky

#### C4·S1 · Přehled reportů a parametry

*ADM připravující podklady pro dotace; přesnost nad rychlostí*

**Účel:** Jedna obálka pro R1–R9: výběr reportu, parametry, graf/tabulka, export. Sdílí návrh s oddílovými reporty (B11) — liší se jen scope a dimenzí region.

**Obsah a pole:**

- Parametry dle spec (oddíly, od–do, koš, region, typ akce); u výsledku metadata `generated_at` a parametry výpočtu — reporty se krátce cachují, uživatel má vidět, k jakému okamžiku data platí. (zdroj: docs/reports.md → Společné parametry, Výstup a API)
- Série + součty; export CSV (stejná data, plochá tabulka). (zdroj: docs/reports.md → Výstup a API)
- Poznámky, které spec u konkrétních reportů výslovně chce v UI: kategorie v R1 se nevylučují (součet sloupců ≠ počet účastníků); akce bez kapacity vyloučené z naplněnosti (R3, počet v `meta`); roční klasifikace dobrovolníků při nekrytém intervalu (R5, `meta.note`). (zdroj: docs/reports.md → R1, R3, R5)

**Stavy:**

- **Načítání:** agregace nad všemi oddíly jsou těžké a nesmí blokovat běžnou práci — skeleton + zachovaná plocha grafu. (zdroj: docs/non-functional.md → Rozsah a výkon)
- **Prázdný:** pozor — série s nulami *není* prázdný stav, je to platný výsledek; skutečně prázdno je jen bez jakýchkoli akcí ve scope.
- **Chybějící pole modelu:** metriky, pro které chybí data (např. čas vzniku přihlášky pro splatnost v R7), se nevrací a **UI je skryje** — ne „0“, ne pomlčka bez vysvětlení. (zdroj: docs/reports.md → Požadavky na datový model)

**Validace:** **Dle spec:** parametr oddílů se validuje proti povoleným oddílům volajícího (server); UI proto nabízí jen povolené hodnoty a chybu scope nikdy neukazuje běžnému uživateli. (zdroj: docs/reports.md → Rozsah dat)

**Mobil:** Desktop-first *(odvozeno)*; grafy responzivní, tabulky ve scrollu; export funguje i z mobilu.

**Otevřené UX otázky:** **[K rozhodnutí]** Formy vizualizace (typ grafu na metriku) spec neurčuje — je to samostatná datavizová kapitola sdílená s B11.

#### C4·S2 · R9 — unikátní děti a reportovací sloučení

*ADM; čísla jdou do oficiálního vykazování*

**Účel:** Spočítat závazné číslo pro vykazování a spravovat reportovací sloučení tak, aby si ho nikdo nespletl se skutečným sloučením osob.

**Obsah a pole:**

- Výsledek: počet unikátních dětí za kalendářní rok, volitelně po regionech (snapshot z akce); hosté cizích oddílů se nepočítají. (zdroj: docs/reports.md → R9) (zdroj: README.md → Modul reporty ústředí)
- Kandidáti na sloučení: **jen jméno, příjmení, datum narození** — obrazovka nesmí ukázat víc, i kdyby to bylo „užitečné“. (zdroj: docs/reports.md → R9)
- Aktivní reportovací sloučení jako skupiny (tranzitivně: A–B, B–C ⇒ jedna osoba) s akcí zrušit — je kdykoli zrušitelné, protože nic nepřepsalo. (zdroj: docs/person-merge.md → Reportovací sloučení)
- Stálá vysvětlivka rozdílu: reportovací sloučení *nemění žádná data*, počítá jen pro tento report; skutečné sloučení osob je flow C3 se schvalováním. (zdroj: docs/person-merge.md → Reportovací sloučení)

**Stavy:**

- **Prázdný (kandidáti):** žádní kandidáti = dobrá zpráva; formulovat jako potvrzení čistoty dat.
- **Úspěch (sloučení/zrušení):** okamžitý přepočet viditelný v čísle reportu — jediný efekt, který operace má.

**Otevřené UX otázky:**

- **[K rozhodnutí]** Spec u R9 říká, že se počítá za kalendářní rok, „i když UI dovolí jiný interval“ — máme tedy u R9 volbu období omezit na roky, nebo nechat obecný interval a výsledek přepočítávat na rok s vysvětlením? (zdroj: docs/reports.md → R9)
- **[K rozhodnutí]** Má obrazovka u kandidáta nabízet i cestu ke *skutečnému* sloučení (založit žádost C3)? Spec obě mechaniky odděluje a sama varuje, že se snadno pletou — nabídka zkratky pomůže datové čistotě, ale zvyšuje riziko záměny. (zdroj: docs/person-merge.md → Reportovací sloučení)

### C5 · Modul vzdělávání P4

Zdroje: (zdroj: README.md → Modul vzdělávání) (zdroj: docs/data-model.md → COURSE, PERSON_COURSE) (zdroj: docs/reports.md → R8)

**Cíl a spouštěč.** „Vědět, kteří vedoucí mají platné kurzy — a komu brzy propadnou.“ Spec definuje datový základ a report, ale málo procesu; flow proto držíme úsporně na dvou obrazovkách.

#### Průběh

1. **ADM definuje kurzy ústředí** — Název a doba platnosti (`validity_months`; `NULL` = trvalý kurz, nikdy neexpiruje). (zdroj: README.md → Modul vzdělávání) (zdroj: docs/reports.md → R8)
2. **Vzdělávací akce ústředí se prováže s kurzem** — Akce nese `course_id`; po absolvování vznikne každému účastníkovi vazba `PERSON_COURSE` s odkazem na zdrojovou akci a spočtenou platností (`valid_to = completed_on + validity_months`) — přiřazení je automatické. (zdroj: README.md → Modul vzdělávání) (zdroj: docs/data-model.md → PERSON_COURSE)
3. **Větev: co je „absolvování“?** — **[K rozhodnutí]** Spec spouštěč nedefinuje — konec akce, docházkový záznam s přítomností, ruční potvrzení vedoucím? *(odvozeno)* Docházka je nejbližší existující mechanismus, ale je to interpretace; nutné potvrdit se zadavatelem, protože na tom stojí automatika kroku 2.
4. **Samoobslužné doplnění (plocha B/D)** — HVO, Vedoucí a Rádci si mohou sami přiřadit kurzy z nabídky a vložit vlastní certifikáty, potvrzení od doktora a jiné absolvované kurzy (`certificate_file`). (zdroj: README.md → Modul vzdělávání)
5. **ADM sleduje pokrytí a expirace** — Modul zobrazuje ADM, jaké kurzy absolvovali jednotliví vedoucí v oddílech; report R8 dodává platné kurzy k datu, expirace do N dní (default 90) a pokrytí — podíl aktivních vedoucích s platným kurzem. (zdroj: README.md → Modul vzdělávání) (zdroj: docs/reports.md → R8)

#### Obrazovky

#### C5·S1 · Katalog kurzů ústředí

*ADM; jednoduchá evidence, mění se zřídka*

**Účel:** Udržovat nabídku kurzů, ze které čerpají vzdělávací akce i samoobslužné přiřazení vedoucích.

**Obsah a pole:**

- Za kurz: název, doba platnosti v měsících (prázdné = trvalý — v UI říct slovem, ne prázdnou buňkou). (zdroj: docs/data-model.md → COURSE)
- *(odvozeno)* Počet držitelů a navázaných akcí jako kontext před úpravami — spec sloupce nedefinuje.

**Stavy:**

- **Prázdný:** modul bez kurzů nemá co nabízet — prázdný stav vede k založení prvního kurzu.

**Validace:** Spec žádné neurčuje (mezera validací (zdroj: TODO.md #4)); navrhneme kladné celé měsíce.

**Otevřené UX otázky:** **[K rozhodnutí]** `COURSE` nemá příznak aktivnosti ani se nikde neřeší vyřazení kurzu, na kterém visí `PERSON_COURSE` záznamy. Smazat, skrýt z nabídky, nebo archivovat? (zdroj: docs/data-model.md → COURSE)

#### C5·S2 · Přehled vzdělání vedoucích

*ADM kontrolující způsobilost před sezónou*

**Účel:** Odpovědět na „kdo smí vést“ dřív, než to bude problém: platné, expirující a propadlé kurzy po oddílech.

**Obsah a pole:**

- Rozpad kurz × platné × expirující × propadlé; expirace do N dní s parametrem (`expiring_in_days`, default 90). (zdroj: docs/reports.md → R8)
- Pokrytí na oddíl: podíl osob s rolí VO/HVO/VD/RÁD s platným kurzem — jmenovatelem jsou aktivní vedoucí, ne všechny osoby. (zdroj: docs/reports.md → R8)
- Detail osoby: absolvované kurzy, zdrojová akce, platnost do. (zdroj: README.md → Modul vzdělávání)

**Stavy:**

- **Prázdný / bez dat:** trvalé kurzy nikdy nepatří mezi expirující — nula v tom sloupci je správný výsledek, ne chyba. (zdroj: docs/reports.md → R8)

**Otevřené UX otázky:**

- **[K rozhodnutí]** Samoobslužně vložené kurzy a certifikáty nikdo neschvaluje (spec žádný schvalovací krok nemá). Má je ADM v přehledu odlišit od kurzů z absolvovaných akcí, případně ověřovat? Dotýká se důvěryhodnosti pokrytí.
- **[K rozhodnutí]** Smí ADM otevřít soubor certifikátu (`certificate_file`)? Reporty citlivé údaje vracet nesmí, ale tento přehled je modul, ne report — hranice není ve spec vytyčena. (zdroj: docs/reports.md → Co reporty záměrně neřeší)

### C6 · Systémové šablony akcí a whitelist jmen P4

Zdroje: (zdroj: README.md → Typy a šablony akcí) (zdroj: README.md → Deduplikace osob, merge) (zdroj: docs/data-model.md → ACTION_TEMPLATE, NAME_WHITELIST, NAME_EXCEPTION)

**Cíl a spouštěč.** Dvě drobné, ale celosystémové konfigurace: **systémové šablony akcí** (spravuje ADM, dostupné všem oddílům; oddíly si nad jejich rámec zakládají vlastní) a **whitelist českých křestních jmen** pro kontrolu při zakládání osob (příjmení se neověřují; výjimky přidává HVO per oddíl). (zdroj: README.md → Typy a šablony akcí, Deduplikace osob, merge)

#### Průběh

1. **Správa systémových šablon** — Šablona je přednastavená konfigurace typu akce; k jednomu typu jich může existovat víc. Definuje povinná a nabízená pole formuláře, zapnuté subsystémy, výchozí povinné dokumenty a výchozí hodnoty cen, splatnosti, storna, kapacity, náhradníků, dobrovolníků a referenčního data pro věk. (zdroj: README.md → Typy a šablony akcí)
2. **Bezpečná editace díky snapshotu** — Akce si při vzniku ukládá odkaz na šablonu i typ jako snapshot — **pozdější úprava šablony už založené akce nemění**. Editor to má říkat, jinak se ADM bude bát šablony měnit. Šablony jsou navíc vstupem pro AI návrh akce. (zdroj: README.md → Typy a šablony akcí) (zdroj: AI_support.md)
3. **Správa whitelistu jmen** — Systém ověřuje česká křestní jména proti seznamu spravovanému administrátorem. Jméno mimo seznam neblokuje natvrdo: HVO může v rámci oddílu přidat výjimku (eviduje se, kdo schválil). (zdroj: README.md → Deduplikace osob, merge) (zdroj: docs/data-model.md → NAME_EXCEPTION)
4. **Větev: výjimka se opakuje napříč oddíly** — *(odvozeno)* Když stejné jméno schválí výjimkou několik oddílů, je zřejmě čas přidat ho do celostátního whitelistu — spec tuto smyčku nepopisuje, navrhujeme ji jako doplněk. **[K rozhodnutí]**

#### Obrazovky

#### C6·S1 · Systémové šablony akcí

*ADM; edituje zřídka, dopad na všechny oddíly*

**Účel:** Spravovat výchozí konfigurace, ze kterých oddíly zakládají akce — s jistotou, že úprava nic nerozbije zpětně.

**Obsah a pole:**

- Seznam šablon seskupený podle devíti typů akcí (pravidelné schůzky … workshopové); za šablonu název, typ, příznak aktivnosti. (zdroj: README.md → Typy a šablony akcí) (zdroj: docs/data-model.md → ACTION_TEMPLATE)
- Editor šablony = stejná rodina formulářů jako konfigurace akce (B2) v režimu „výchozí hodnoty“ — nenavrhovat druhý, odlišný formulář na tatáž pole. *(odvozeno)*
- Trvalá poznámka o snapshotu: „Úprava šablony se projeví jen u nově založených akcí.“ (zdroj: README.md → Typy a šablony akcí)

**Stavy:**

- **Neaktivní šablona:** nenabízí se při zakládání akce; existující akce nedotčené (plyne ze snapshotu). *(odvozeno)*

**Otevřené UX otázky:** **[K rozhodnutí]** Vidí ADM i oddílové šablony (jen ke čtení?), nebo výhradně systémové? Spec odděluje jen správu („systémové spravuje ADM“), viditelnost neřeší. (zdroj: README.md → Typy a šablony akcí)

#### C6·S2 · Whitelist křestních jmen a výjimky

*ADM; údržba číselníku, spouštěná stížnostmi z oddílů*

**Účel:** Udržovat seznam jmen tak, aby kontrola chytala překlepy (smysl deduplikace), a ne blokovala skutečná, jen neobvyklá jména.

**Obsah a pole:**

- Prohledávatelný seznam jmen; přidání a odebrání. (zdroj: docs/data-model.md → NAME_WHITELIST)
- Přehled oddílových výjimek: jméno, oddíl, kdo z HVO schválil, kdy — s akcí „povýšit do whitelistu“ (viz větev; návrh nad rámec spec, *(odvozeno)*). (zdroj: docs/data-model.md → NAME_EXCEPTION)

**Validace:** **Dle spec:** ověřují se jen křestní jména, příjmení nikdy. Pravidla porovnání (diakritika, velikost písmen) spec u whitelistu neurčuje — u kandidátů merge normalizuje, tady nic; sjednotit a doplnit do spec. (zdroj: README.md → Deduplikace osob, merge) (zdroj: docs/person-merge.md → Detekce kandidátů)

**Stavy:**

- **Prázdný:** prázdný whitelist by označil každé jméno — počáteční naplnění seznamu spec neřeší a bez něj modul nelze zapnout. **[K rozhodnutí]** Odkud se seznam vezme (import, postupné budování z výjimek)?

**Mobil:** Desktop-first *(odvozeno)*; vyhledávání jako primární vstup, seznam se stránkuje.

> **Souhrn otevřených bodů plochy C**
> Nejtěžší rozhodnutí této plochy nejsou vizuální: **životní cyklus oddílu** (C1), **stavový automat regionu vč. rozdělení a zrušení** (C2, spec ho sama žádá (zdroj: TODO.md #2)), **trvalost potlačení a právo revertu** (C3, otevřené otázky přímo ve spec (zdroj: docs/person-merge.md → Otevřené otázky)), **oddělení reportovacího a skutečného sloučení v UI** (C4) a **spouštěč „absolvování“ kurzu** (C5). Všechny se dotýkají spec, ne jen obrazovek — doporučujeme je projednat se zadavatelem před návrhem detailních wireframů.

## D · Plocha D — Self-management

*Přihlášený člověk spravuje sebe, své děti, své přihlášky a svůj účet*

Plocha D pokrývá **self-management pro registrované** — čtvrtou položku rozsahu projektu. (zdroj: README.md → Rozsah) Uživatelem je tu člověk s **účtem** (přihlašovací identita navázaná právě na jednu osobu, heslo nebo OAuth) (zdroj: README.md → Osoba vs. uživatelský účet) — typicky rodič spravující jedno či více dětí, dospělý účastník, případně zletilý bývalý „spravovaný" člen, který účet právě převzal.

> **Návrhový princip celé plochy: zrcadlo tokenového rozcestníku**
> *(odvozeno)* Spec dělá správu přihlášky **bez účtu přes tokenový odkaz** normálním, plnohodnotným režimem (storno, změny, dokumenty, přidání účastníků). (zdroj: README.md → Přihlašování na akce) (zdroj: docs/non-functional.md → Tokeny a ochrana přístupu) Navrhujeme proto, aby detail přihlášky v „Moje přihlášky" byl **tatáž obrazovka jako tokenový rozcestník** (obrazovka S6 z plochy A) — jednou navržená, dvě přístupové cesty (token vs. účet). Uživatel, který si účet založí až po několika tokenových přihláškách, uvidí důvěrně známé rozhraní; podpora vysvětluje jeden systém, ne dva. Účet navíc přidává jen to, co token dát nemůže: seznam *všech* přihlášek osoby a zastupovaných dětí (token je vázán vždy na jedinou přihlášku (zdroj: docs/non-functional.md → Tokeny a ochrana přístupu)).

Priorita ploch: D1 a D2 jsou P2 (těží z hotových vzorů plochy A), D4 je P3 (žádost o sloučení je citlivý moment souhlasu), D3 je P4 (prosté CRUD, jakmile existují pravidla polí). Priority dle inventáře v sekci 2.1.

### D1 · Moje přihlášky P2

Zdroje: (zdroj: README.md → Rozsah) (zdroj: README.md → Přihlašování na akce) (zdroj: docs/registration-lifecycle.md) (zdroj: docs/payment-matching.md)

**Cíl a spouštěč.** „Chci vidět, co je s našimi přihláškami — co komu chybí, co mám zaplatit, co nahrát." Spouštěč: e-mailová připomínka (výzva k platbě, zamítnutý dokument), nebo prostě rodičovský návyk zkontrolovat stav. **Aktéři:** držitel účtu za sebe; rodič za zastupované děti (práva vždy per dítě, odvozená z aktivní vazby rodič ↔ dítě). (zdroj: README.md → Role (Rodič))

#### Průběh

1. **Přihlášení do systému** — Heslo nebo Google/Facebook OAuth; jeden účet může mít více propojených identit. (zdroj: README.md → Přihlašování do systému) Po přihlášení vstup do sekce „Moje přihlášky".
2. **Seznam přihlášek — vlastní i dětí** — Účet je navázán na osobu; zobrazují se přihlášky této osoby a přihlášky dětí s aktivní vazbou rodič ↔ dítě (rodič „spravuje jejich přihlášky — přihlášení na akci, storno, platby za dítě"). (zdroj: README.md → Role (Rodič)) Každá položka nese stav ze stavového automatu a blokující bránu. (zdroj: docs/registration-lifecycle.md)
3. **Detail přihlášky = sdílený rozcestník (S6)** — Checklist bran v závazném pořadí zástupce → dokumenty → platba; sekce dokumentů se stavy a komentáři zamítnutí; platební sekce s QR, VS/SS a zbývající částkou. (zdroj: docs/registration-lifecycle.md → Pořadí bran) (zdroj: docs/payment-matching.md) *(odvozeno)* Stejná obrazovka jako tokenový rozcestník, viz princip výše.
4. **Větev: zaplatit `PendingPayment` / `PartialPaid`** — QR + platební údaje; částky se párují **přesně** (koruna rozdílu = nedoplatek/přeplatek), po částečné platbě se zobrazuje zbývající částka; potvrzení chodí za každou napárovanou platbu. (zdroj: docs/payment-matching.md)
5. **Větev: dokumenty `PendingDocuments`** — Nahrávání postupně nebo najednou (10 MB, PDF/JPG/PNG/HEIC dle skutečného obsahu); zamítnutí s komentářem vedoucího vrací přihlášku zpět, dokud nejsou všechny povinné dokumenty schválené. (zdroj: README.md → Přihlašování na akce (Povinné dokumenty, Schvalování dokumentů)) (zdroj: docs/non-functional.md → Úložiště souborů)
6. **Větev: storno** — Možné z každého nekoncového stavu, i ze `Paid`; poplatek dle storno pravidel akce k datu storna; vratku vyplácí účetní mimo systém, systém ji jen eviduje. Terminální stav je konečný — návrat znamená novou přihlášku. (zdroj: docs/registration-lifecycle.md → Guardy a invarianty) (zdroj: README.md → Ceny a storna)
7. **Historie a terminální přihlášky** — `Canceled` a `Expired` zůstávají viditelné jen pro čtení (historie a účetnictví se zachovávají). (zdroj: docs/registration-lifecycle.md → Guardy a invarianty)

**Notifikace ve flow:** výzvy a připomínky splatnosti (denní job, četnost dle nastavení oddílu), e-mail při zamítnutí dokumentu, potvrzení plateb. (zdroj: docs/non-functional.md → Plánované úlohy) **Admin protistrany:** B3 (přehled přihlášek), B4 (posuzování dokumentů), B6 (párování plateb).

#### Obrazovky

#### D1-S1 · Moje přihlášky — seznam

**Účel:** Jedna odpověď na otázku „co je s našimi přihláškami a co je teď na mně".

**Publikum:** Rodič s více dětmi na telefonu, přicházející z e-mailové připomínky; méně často dospělý účastník s jedinou přihláškou.

**Obsah a pole:**

- Karta přihlášky: účastník (já / jméno dítěte), akce, termín, stav (český popisek + blokující brána), kategorie (účastník / dobrovolník / náhradník). (zdroj: README.md → Přihlašování na akce) (zdroj: docs/registration-lifecycle.md → Stavy)
- U platebních stavů zbývající částka a splatnost (u relativní splatnosti 14 dní od podání, nejpozději k začátku akce). (zdroj: docs/payment-matching.md)
- *(odvozeno)* Řazení: nejdřív přihlášky vyžadující akci uživatele (zamítnutý dokument, splatnost), pak čekající na druhé strany, pak hotové a terminální — plyne z toho, že stav vždy jmenuje první nesplněnou bránu.
- Data: `REGISTRATION` osob s vazbou přes `ACCOUNT.person_id` a aktivní `PARENT_CHILD`. (zdroj: docs/data-model.md)

**Stavy:**

- **Prázdný:** účet bez jediné přihlášky — nabídnout vstup do výpisu akcí (plocha A), ne omluvu.
- **Načítání:** skeleton karet, bez skoků layoutu.
- **Vše hotovo:** všechny přihlášky `Paid` — klidná potvrzující varianta, žádná falešná urgence.

**Mobil:** Karty na celou šíři, primární akce („Zaplatit", „Nahrát dokument") jako tlačítko přímo na kartě; stavové změny oznamované čtečkám.

**Otevřené UX otázky:**

- **[K rozhodnutí]** Spec nedefinuje žádné filtrování/segmentaci seznamu. Možnosti: jeden chronologický seznam · záložky „aktivní / historie" · seskupení po dětech. Pro rodiče se třemi dětmi na táboře jde o hlavní rozhodnutí obrazovky.
- **[K rozhodnutí]** Zda seznam ukazovat i přihlášky ve stavu `PendingGuardian` u dítěte, které rodič sám nepodal (podal je nezletilý, rodič zatím jen schválil jinou) — spec vznik vazby řeší, viditelnost napříč přihláškami ne.

#### D1-S2 · Detail přihlášky (sdílený s tokenovým rozcestníkem S6)

**Účel:** Aktuální stav, blokující brána, další krok — identické s S6 (plocha A). Zde jen odchylky pro přihlášeného uživatele.

**Publikum:** Stejné jako S6; navíc uživatel, který mezi přihláškami přepíná (má jich víc).

**Obsah a pole:**

- Vše z S6: checklist bran, dokumenty se stavy a komentáři, platba (QR, VS/SS, zbývá, splatnost), storno s náhledem poplatku, změny a přidání účastníků. (zdroj: docs/registration-lifecycle.md) (zdroj: docs/payment-matching.md) (zdroj: README.md → Přihlašování na akce)
- **Navíc oproti tokenu:** navigace zpět na seznam D1-S1 a mezi přihláškami; žádná výzva k založení účtu (už existuje).
- *(odvozeno)* U přihlášky dítěte viditelně „jednáte za: [dítě]" — rodičovská práva jsou per dítě a UI to má připomínat. (zdroj: README.md → Role (Rodič))

**Stavy:** Shodné s S6 (všech devět stavů životního cyklu vč. zamčené varianty náhradníka a pádu zpět po zamítnutém dokumentu). Neduplikovat — jedna specifikace, dva vstupy.

**Validace:** Shodné s S6 (limity nahrávání dokumentů dle obsahu souboru). (zdroj: docs/non-functional.md → Úložiště souborů)

**Otevřené UX otázky:**

- **[K rozhodnutí]** Oba rodiče mají plná práva a „platí poslední zápis" (zdroj: README.md → Role (Rodič)) — má UI druhému rodiči ukázat, že přihlášku právě změnil ten první (např. „naposledy upravil …")? Auditní log data má (zdroj: README.md → Auditní log), spec o zobrazení mlčí.

### D2 · Moje děti P2

Zdroje: (zdroj: README.md → Role (Rodič / zákonný zástupce)) (zdroj: docs/data-model.md → PARENT_CHILD) (zdroj: docs/registration-lifecycle.md → guardian.approved) (zdroj: README.md → Deduplikace osob, merge)

**Cíl a spouštěč.** „Spravovat údaje a přihlášky svých dětí — a přizvat druhého rodiče, ať to nevisí jen na mně." **Aktéři:** rodič (zákonný zástupce); druhý zástupce (e-mailová pozvánka); HVO (zástup, když dítě nemá navázaného rodiče; schválení připojení); zletilé dítě (převzetí a případné zrušení přístupu). **Klíčový fakt:** rodič **není přidělovaná role** — postavení se odvozuje z aktivní vazby rodič ↔ dítě, práva vždy per dítě. (zdroj: README.md → Role)

#### Průběh

1. **Vazba vzniká přihlášením nebo schválením** — Vazba rodič ↔ dítě vzniká přihlášením dítěte na akci rodičem (zdroj: README.md → Role (Rodič)), nebo schválením přihlášky nezletilého odkazem v e-mailu (`guardian.approved` vazbu zakládá). (zdroj: docs/registration-lifecycle.md → Události) Sekce „Moje děti" tedy typicky vzniká sama, ne registračním formulářem.
2. **Přehled dětí** — Seznam dětí s aktivní vazbou; stav vazby drží `PARENT_CHILD.state`: `active / cancelled / readonly_after_adulthood`. (zdroj: docs/data-model.md → PARENT_CHILD)
3. **Detail dítěte: údaje + přihlášky** — Rodič spravuje údaje dítěte v systému (adresy, pojišťovny, …) a jeho přihlášky (přihlášení na akci, storno, platby za dítě). (zdroj: README.md → Role (Rodič)) Přihlášení na akci vede do portálového flow A3; *(odvozeno)* s předvyplněnými údaji dítěte z osoby.
4. **Větev: pozvat druhého zákonného zástupce** — Druhého zástupce přidává stávající rodič (nebo HVO) **pozvánkou e-mailem**; vazba vzniká až přijetím pozvánky druhým rodičem. Nemá-li dítě žádného navázaného rodiče, připojení schvaluje HVO oddílu, kde je dítě evidováno. Oba rodiče pak mají plná práva, platí poslední zápis. (zdroj: README.md → Role (Rodič))
5. **Větev: zrušení vazby (vystoupení)** — Vazbu může zrušit sám rodič, případně HVO na žádost; zrušení se loguje. Zůstane-li nezletilé dítě bez navázaného rodiče, jeho údaje a přihlášky spravuje HVO, dokud se nepřipojí nový zástupce. (zdroj: README.md → Role (Rodič))
6. **Větev: zletilost dítěte — režim jen pro čtení** — Po dosažení zletilosti se zastoupení přepne do režimu jen pro čtení (`readonly_after_adulthood`). Jediná výjimka: rodič smí doplnit chybějící kontaktní e-mail dítěte — slouží k doručení **výzvy k převzetí účtu**. Zletilý člen může přístup rodiče kdykoli zcela zrušit. (zdroj: README.md → Role (Rodič))
7. **Větev: duplicitní dítě** — Systém zobrazí rodiči kandidáta na duplicitu dítěte; druhou stranou schvalování je rodič dítěte-kandidáta (bez rodiče schvaluje HVO). Sloučení spojí jen osobu dítěte, nikdy účty rodičů. (zdroj: README.md → Deduplikace osob, merge) (zdroj: docs/person-merge.md → Schvalování (kind = child)) Mechanika žádosti je shodná s D4 (níže) a s adminovským flow C3.

**Notifikace ve flow:** pozvánka druhému zástupci; e-maily žádosti o sloučení všem stranám. (zdroj: README.md → Deduplikace osob, merge) **Admin protistrany:** HVO — schválení připojení zástupce u dítěte bez rodiče, zrušení vazby na žádost, správa „osiřelého" dítěte.

#### Obrazovky

#### D2-S1 · Moje děti — přehled

**Účel:** Rozcestník po dětech: kdo má co rozdělaného, koho se týká pozvánka či zletilost.

**Publikum:** Rodič; občas rodič, který vazbu právě získal schválením přihlášky a sekci vidí poprvé.

**Obsah a pole:**

- Karta dítěte: jméno, přezdívka, počet aktivních přihlášek; stav vazby (`active` / `readonly_after_adulthood`). (zdroj: docs/data-model.md → PARENT_CHILD)
- Indikace druhého zástupce: navázán / pozvánka odeslána / žádný. (zdroj: README.md → Role (Rodič))
- Akce: pozvat druhého zástupce · zrušit svou vazbu (vystoupení).

**Stavy:**

- **Prázdný:** účet bez vazeb — vysvětlit, že vazba vzniká přihlášením dítěte na akci nebo schválením jeho přihlášky, s odkazem na výpis akcí; nenabízet „přidat dítě ručně" (spec takový vznik vazby nezná).
- **Po zletilosti:** karta v režimu jen pro čtení s vysvětlením a případnou výzvou „doplňte kontaktní e-mail" — viz D2-S2.

**Mobil:** Karty pod sebou; destruktivní akce (vystoupení) mimo primární dosah palce, až v detailu.

**Otevřené UX otázky:**

- **[K rozhodnutí]** Zrušení vazby: spec ho umožňuje jedním rozhodnutím rodiče, ale důsledek je závažný (dítě bez rodiče spravuje HVO). Kolik tření přidat — potvrzovací dialog s dopadem, nebo dvoukrokové potvrzení? A má se druhému zástupci / HVO poslat notifikace? Spec jmenuje jen logování.

#### D2-S2 · Detail dítěte

**Účel:** Spravovat údaje a přihlášky jednoho dítěte; obsloužit hranu zletilosti bez zmatku.

**Publikum:** Rodič; po 18. narozeninách dítěte tentýž rodič v režimu jen pro čtení.

**Obsah a pole:**

- Údaje osoby dítěte: jméno, příjmení, přezdívka, pohlaví, datum narození, kontaktní e-mail, adresa trvalého bydliště, zdravotní pojišťovna; tituly jen mají-li smysl. (zdroj: README.md → Osoba vs. uživatelský účet) (zdroj: docs/data-model.md → PERSON)
- Přihlášky dítěte — stejné karty jako D1-S1, detail vede na D1-S2.
- Zástupci dítěte: já + druhý zástupce (nebo tlačítko „pozvat"); dítě může mít více rodičů. (zdroj: README.md → Role (Rodič))
- Ve stavu `readonly_after_adulthood`: vše jen pro čtení; jediné aktivní pole je chybějící kontaktní e-mail dítěte (pro doručení výzvy k převzetí účtu). (zdroj: README.md → Role (Rodič))

**Stavy:**

- Aktivní vazba (plná editace) · jen pro čtení po zletilosti (s vysvětlením, proč) · vazba zrušena dítětem — *(odvozeno)* dítě po zrušení zmizí z přehledu, historii pláteb rodič dál vidí u svých vlastních plateb? Spec neříká — viz otázky.

**Validace:**

- Křestní jméno proti seznamu českých jmen s možností výjimky HVO v rámci oddílu (chyba musí nabízet cestu, ne zeď); příjmení se neověřují. (zdroj: README.md → Deduplikace osob, merge)
- Datum narození je povinné u registrovaného člena a nutné pro vyhodnocení brány zástupce a věková pravidla hlídek. (zdroj: docs/data-model.md → PERSON) (zdroj: docs/registration-lifecycle.md → Guardy a invarianty)
- Ostatní formáty polí = deklarovaná mezera validací. (zdroj: TODO.md #4)

**Mobil:** Formulář s viditelnými popisky, správné klávesnice (datum, e-mail); autofill adresy — rodič ji píše opakovaně.

**Otevřené UX otázky:**

- **[K rozhodnutí]** Chytré sloupce: viditelnost/editaci pro *vlastníka účtu* spec definuje (zdroj: README.md → Pomocná evidence), ale o rodiči dítěte mlčí. Zdědí rodič práva vlastníka, nebo sloupce nevidí? Nutné dořešit se zadavatelem — jde o citlivá data dětí.
- **[K rozhodnutí]** Smí rodič měnit datum narození dítěte? Změna může rozpustit hlídku a přepnout věková pravidla (zdroj: docs/race-patrols.md) — možnosti: volná editace s varováním dopadů · editace jen přes HVO.
- **[K rozhodnutí]** „Výzva k převzetí účtu" po zletilosti: spec dává jen mechanismus doplnění e-mailu; kdo a kdy výzvu odesílá (automaticky v den 18. narozenin? tlačítkem rodiče?) definované není.

#### D2-S3 · Pozvánka druhého zástupce (e-mail + přijímací stránka)

**Účel:** Druhý rodič pochopí, co přijímá (plná práva k dítěti v systému), a přijme na jeden krok.

**Publikum:** Druhý rodič — často bez účtu i bez znalosti systému, na telefonu; stejná „studená" situace jako schvalující zástupce v A4.

**Obsah a pole:**

- Kdo zve, které dítě, co přijetí znamená: vazba vznikne přijetím; oba rodiče pak mají plná práva, platí poslední zápis. (zdroj: README.md → Role (Rodič))
- Přijetí = *(odvozeno)* vyžaduje identitu druhého rodiče v systému (osobu, ideálně účet) — spec vznik osoby druhého rodiče při přijetí nepopisuje; nutné dodefinovat se zadavatelem (patrně založení účtu jako součást přijetí, analogicky k převzetí přihlášky tokenem (zdroj: README.md → Přihlašování na akce)).

**Stavy:**

- Platná pozvánka · již přijato (idempotentně, přátelsky) · *(odvozeno)* neplatná/odvolaná.

**Otevřené UX otázky:**

- **[K rozhodnutí]** Platnost pozvánky: spec žádnou lhůtu nedefinuje (na rozdíl od 7 dní u schválení zástupcem a 48 h u náhradníka). Navrhnout lhůtu a chování po vypršení, nebo pozvánku nechat bez expirace?
- **[K rozhodnutí]** Možnost pozvánku odmítnout a možnost zvoucího rodiče ji odvolat — ani jedno spec nezná; obojí stojí za návrh do spec.

### D3 · Moje osobní údaje a souhlasy P4

Zdroje: (zdroj: README.md → Osoba vs. uživatelský účet) (zdroj: docs/data-model.md → PERSON, CONSENT) (zdroj: README.md → Retence a GDPR)

**Cíl a spouštěč.** „Opravit si adresu / pojišťovnu; vědět, k čemu jsem dal souhlas." Prosté CRUD nad vlastní osobou — spec tu definuje málo detailu, proto úsporně: jedna obrazovka. Flow: přihlášení → „Moje údaje" → úprava polí / správa souhlasů.

#### Obrazovky

#### D3-S1 · Moje údaje a souhlasy

**Účel:** Jedno místo pro údaje osoby a přehled souhlasů se zpracováním.

**Publikum:** Držitel účtu; navštěvuje zřídka, typicky po změně bydliště nebo pojišťovny před akcí.

**Obsah a pole:**

- Údaje osoby: jméno, příjmení, přezdívka, tituly před/za, pohlaví, datum narození, kontaktní e-mail, adresa trvalého bydliště, zdravotní pojišťovna. Vyplňují se podle potřeby akce; cokoli nad tento rámec patří do chytrých sloupců oddílu. (zdroj: README.md → Osoba vs. uživatelský účet)
- Chytré sloupce oddílů, u nichž má vlastník účtu nastavenou viditelnost/možnost úprav. (zdroj: README.md → Pomocná evidence)
- Souhlasy: typ (zpracování / foto / zdravotní …), účel, uděleno kdy, případné odvolání (`revoked_at`), retence. (zdroj: docs/data-model.md → CONSENT) Souhlasy se uchovávají po dobu zpracování + 4 roky po odvolání. (zdroj: README.md → Retence a GDPR)

**Validace:** Křestní jméno proti seznamu (s cestou k výjimce přes HVO) (zdroj: README.md → Deduplikace osob, merge); ostatní pravidla polí jsou deklarovaná mezera — naše specifikace je bude zakládat. (zdroj: TODO.md #4)

**Stavy:** **Úspěch:** nenápadné potvrzení uložení. **Chyba:** inline u pole. (Prázdný ani načítací stav tu nejsou zajímavé — data vždy existují.)

**Mobil:** Standardní formulářová pravidla ploch A/D; autofill adresy.

**Otevřené UX otázky:**

- **[K rozhodnutí]** Odvolání souhlasu: model ho zná (`revoked_at`), ale spec nikde neříká, že ho osoba provádí sama v UI (vs. žádost vyřizovaná HVO). Self-service tlačítko, nebo žádost?
- **[K rozhodnutí]** Žádost o výmaz (GDPR): existuje denní job „vyřízení žádostí o výmaz" (zdroj: docs/non-functional.md → Plánované úlohy), ale kdo a kde žádost podává, spec nedefinuje. Patří sem tlačítko „požádat o výmaz", nebo je to proces přes HVO/ADM?
- **[K rozhodnutí]** Vlastní editace data narození — stejná otázka jako u dítěte (dopady na brány a hlídky); možnosti viz D2-S2.

### D4 · Účet a bezpečnost P3

Zdroje: (zdroj: README.md → Přihlašování do systému) (zdroj: docs/non-functional.md → Přihlašování přes OAuth) (zdroj: docs/non-functional.md → Tokeny a ochrana přístupu) (zdroj: README.md → Deduplikace osob, merge) (zdroj: docs/person-merge.md)

**Cíl a spouštěč.** „Změnit si heslo, přidat přihlášení Googlem — a rozhodnout o návrhu ‚nejste to vy v jiném oddíle?'." **Aktéři:** držitel účtu; u sloučení dále HVO druhého oddílu a případně účet kandidáta (schvalují e-mailem) a administrátor (revert). (zdroj: docs/person-merge.md → Schvalování)

#### Průběh

1. **Správa přihlášení** — Změna hesla (hesla se hashují, reset je jednorázový token s krátkou platností); propojení více OAuth identit Google/Facebook s jedním účtem. (zdroj: README.md → Přihlašování do systému) (zdroj: docs/non-functional.md → Šifrování a hesla)
2. **Větev: propojení OAuth s existujícím účtem** — Identita se páruje přes **ověřený** e-mail (neověřený se odmítá); existuje-li účet se stejným e-mailem a heslem, propojení se nabídne **až po úspěšném přihlášení heslem** — automatické propojení by umožnilo převzetí účtu. (zdroj: docs/non-functional.md → Přihlašování přes OAuth)
3. **Větev: odpojení identity** — Odpojit lze jen tehdy, zbývá-li účtu jiný způsob přihlášení (heslo nebo druhá identita) — tlačítko se u poslední cesty zamyká s vysvětlením. (zdroj: docs/non-functional.md → Přihlašování přes OAuth)
4. **Návrh duplicity: „nejste to vy?"** — Osobě s účtem se zobrazí možný kandidát na propojení z jiného oddílu; kandidáti se **jen navrhují, nikdy neslučují automaticky**. (zdroj: README.md → Deduplikace osob, merge) (zdroj: docs/person-merge.md → Detekce kandidátů)
5. **Žádost o sloučení a schvalování** — Účet zadá Žádost o sloučení; systém rozešle e-maily iniciátorovi, HVO druhého oddílu a případně účtu kandidáta. Žádost je `pending`, dokud všechny strany nerozhodly; souhlas všech → `ready`, jediné zamítnutí → `rejected` (terminální); bez odezvy propadá po 30 dnech. (zdroj: docs/person-merge.md → Schvalování)
6. **Dokončení: volba polí A/B** — Ze stavu `ready` spouští sloučení iniciátor: konflikt základních polí se řeší pole po poli, jen volbou A/B (žádný ruční přepis — kvůli věrnému revertu); prázdné pole prohrává s vyplněným; různá data narození vyvolají varování. Zůstává jeden účet, druhý se ruší (uživatel vybere); OAuth identity se přenesou. (zdroj: docs/person-merge.md → Konflikt základních polí) (zdroj: README.md → Deduplikace osob, merge)
7. **Větev: kolize blokuje** — Mají-li obě osoby aktivní přihlášku na téže akci, sloučení se zablokuje a nejdřív to řeší vedoucí akce; sloučení běží v jedné transakci — selže-li část, nezmění se nic. (zdroj: docs/person-merge.md → Přenos vazeb a kolize unikátů)
8. **Bezpečnostní obálka** — Throttling přihlášení podle účtu i IP; po sérii neúspěchů dočasné zamknutí a e-mail vlastníkovi; chybová hláška u přihlášení i obnovy hesla je vždy stejná (ochrana proti výčtu účtů). Neaktivní účet se po 24 měsících (po upozornění) maže/anonymizuje. (zdroj: docs/non-functional.md → Tokeny a ochrana přístupu) (zdroj: README.md → Retence a GDPR)

**Admin protistrany:** C3 (deduplikace a schvalování HVO/ADM, revert sloučení administrátorem). (zdroj: docs/person-merge.md → Revert)

#### Obrazovky

#### D4-S1 · Účet a bezpečnost

**Účel:** Přihlašovací metody pod kontrolou uživatele; nic tu nesmí umožnit zamknout sám sebe ven.

**Publikum:** Držitel účtu, návštěva jednou za rok; občas ve stresu (podezření na zneužití, e-mail o zamknutí účtu).

**Obsah a pole:**

- Změna hesla (staré + nové). (zdroj: README.md → Přihlašování do systému)
- Propojené OAuth identity (Google / Facebook) s akcemi připojit/odpojit; odpojení zamčené, je-li identita poslední cestou k přihlášení — s vysvětlením proč. (zdroj: docs/non-functional.md → Přihlašování přes OAuth)
- Přihlašovací e-mail účtu (`login_email`, unikátní). (zdroj: docs/data-model.md → ACCOUNT)
- Pro zletilého člena: zrušení přístupu rodiče (zletilý může přístup rodiče kdykoli zcela zrušit). (zdroj: README.md → Role (Rodič)) *(odvozeno)* umístění právě sem — jde o kontrolu nad přístupem k vlastní osobě.

**Stavy:**

- **Chyba přihlášení / zamknutí:** po sérii neúspěchů dočasné zamknutí + e-mail vlastníkovi; hlášky nikdy neprozrazují existenci účtu. (zdroj: docs/non-functional.md → Tokeny a ochrana přístupu)
- **Úspěch:** potvrzení změny hesla; *(odvozeno)* bezpečnostní e-mail o změně — spec nezmíněno, návrh do katalogu notifikací (mezera TODO #3).

**Validace:** Pravidla síly hesla spec nedefinuje (Argon2id je uložení, ne politika) — patří do mezery validací. (zdroj: TODO.md #4)

**Mobil:** Podpora správců hesel (autocomplete atributy); přepínače OAuth se 44px cíli.

**Otevřené UX otázky:**

- **[K rozhodnutí]** Změna přihlašovacího e-mailu: model má `login_email`, ale žádné flow změny (ověření nové adresy?) spec nepopisuje. Navrhnout, nebo v první verzi jen přes podporu?
- **[K rozhodnutí]** Zrušení vlastního účtu uživatelem: spec zná jen retenční mazání po 24 měsících nečinnosti. Má existovat self-service „zrušit účet" (a co pak s osobou a vazbami — souvisí s otevřeným lifecycle osoby, (zdroj: README.md → Stav osoby → TODO - dořešit))?

#### D4-S2 · Žádost o sloučení osob (návrh duplicity)

**Účel:** Delikátní moment souhlasu: srozumitelně nabídnout „tohle jste možná vy v jiném oddíle", provést schválením a volbou polí, a nic neslíbit dřív, než souhlasí všechny strany.

**Publikum:** Běžný uživatel bez znalosti pojmu „merge"; jde o jeho osobní data, důvěra je křehká.

**Obsah a pole:**

- **Nabídka kandidáta:** základní údaje kandidáta z jiného oddílu (silný kandidát = shodné datum narození + normalizované jméno a příjmení; přezdívka jen jako alternativa křestního jména). (zdroj: docs/person-merge.md → Detekce kandidátů)
- **Podání žádosti** a přehled stran, které musí souhlasit: já (iniciátor), HVO druhého oddílu, případně účet kandidáta. (zdroj: docs/person-merge.md → Schvalování)
- **Po stavu `ready`: volba konfliktních polí A/B** (jméno, příjmení, přezdívka, tituly, pohlaví, datum narození, e-mail, adresa, pojišťovna) — jen výběr z obou hodnot, oprava až po sloučení běžnou editací; volba ponechaného účtu, má-li účet i kandidát. (zdroj: docs/person-merge.md → Konflikt základních polí)
- Vysvětlení dopadů: vazby (přihlášky, docházka, členství DU, …) se přenášejí všechny; citlivá data zůstávají per oddíl — HVO druhého oddílu nezíská přístup k datům z mého. (zdroj: docs/person-merge.md → Přenos vazeb a kolize unikátů)

**Stavy:**

- `pending` — čeká se na strany (ukázat na koho); `ready` — na tahu je iniciátor (volba polí); `rejected` — kdokoli zamítl nebo 30 dní bez odezvy; `completed`; `reverted` (revert dělá administrátor, uživatel jen vidí výsledek). (zdroj: docs/person-merge.md)
- **Zablokováno kolizí:** obě osoby mají aktivní přihlášku na téže akci — sdělit, že to nejdřív vyřeší vedoucí akce, nic se zatím nemění. (zdroj: docs/person-merge.md → Přenos vazeb a kolize unikátů)
- **Odmítnutí uživatelem:** zamítnutá dvojice se trvale potlačí a znovu se nenabízí (dokud ji ADM nepovolí) — říct to před potvrzením, ne po něm. (zdroj: docs/person-merge.md → Detekce kandidátů)

**Mobil:** Volba A/B jako dvě porovnatelné karty pole po poli, ne tabulka; na úzkém displeji jedno pole na obrazovku s jasným postupem.

**Otevřené UX otázky:**

- **[K rozhodnutí]** Kde a jak nabídku kandidáta zobrazit: nevtíravá sekce zde v D4, jednorázový banner po přihlášení, nebo e-mail? „Nejste to vy?" u osobních dat působí snadno strašidelně — tón je klíčový.
- **[K rozhodnutí]** Trvalost potlačení po zamítnutí je i otevřená otázka spec samotné (natrvalo vs. na určitou dobu) (zdroj: docs/person-merge.md → Otevřené otázky) — UI formulace „už se nezobrazí" na odpovědi závisí.

**Zdroje.** Všechny doménové fakty pocházejí ze specifikace projektu du-doc: [github.com/jendamozna/du-doc](https://github.com/jendamozna/du-doc) (README.md, TODO.md, AI_support.md a dokumenty ve složce `docs/`). Tvrzení označená *(odvozeno)* jsou autorův úsudek, ne fakt ze specifikace; místa označená **[K rozhodnutí]** čekají na rozhodnutí zadavatele.
