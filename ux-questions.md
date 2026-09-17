# Otevřená rozhodnutí — co ještě čeká na zadavatele

Zdroj: [docs/du-doc-ux-pruvodce.md](docs/du-doc-ux-pruvodce.md), všech 123 výskytů badge `[K rozhodnutí]`.
Každý bod je ověřený proti aktuálnímu stavu `docs/` (10. 9. 2026); vyřešené body jsou z tohoto seznamu odstraněné.

Zůstává **63 otevřených rozhodnutí**: 2 průřezová (D2, D3) a 61 lokálních u konkrétních obrazovek.

**→ Návrh** u každé otázky je doporučení k odsouhlasení nebo odmítnutí, ne hotové rozhodnutí. Kde návrh vyžaduje změnu specifikace (nové pole, událost, entita), je to výslovně uvedeno — takové body je potřeba propsat zpět do `docs/`, ne je nechat žít jen v UX vrstvě.

Legenda dopadu:
**🔴 blokuje skelet** — bez odpovědi nelze navrhnout obrazovku ·
**🟠 mění chování** — obrazovka se navrhne, ale bude se předělávat ·
**🟡 text a tón** — dá se dopsat později bez přepracování.

---

## 1 · Průřezová rozhodnutí D1–D10

Tato určují skelet napříč plochami — mají přednost před vším ostatním.

### D2 · Prožitek schválení zákonným zástupcem — na obou stranách 🔴

- Co vidí nezletilý při čekání, jak sdělit „vaše místo ještě není rezervované“, viditelnost odpočtu 7 dnů.
- Jak vysvětlit, že schválením vzniká **trvalá** zástupcovská vazba, ne jen souhlas s jednou akcí.

**→ Návrh:** čekací stránka není samostatná obrazovka — je to **rozcestník S6 v režimu „čeká na zákonného zástupce“**: maskovaný e-mail zástupce (`jan****@seznam.cz`), absolutní datum vypršení a doplňkově „zbývá 5 dní“, tlačítka _Poslat odkaz znovu_ a _Opravit e-mail_ (Q-A8).
Formulace stavu: „Poslali jsme žádost o schválení. Přihláška začne platit, až ji zástupce potvrdí — do té doby ti místo nedržíme.“ Nikdy neslibovat rezervaci, protože `PendingGuardian` se nezapočítává do kapacity.
Na straně zástupce: jedna stránka, nahoře co se schvaluje (dítě, akce, cena), pod tím **výslovně** „Schválením potvrzujete, že jste zákonný zástupce. Vznikne tím trvalá vazba — budete moci spravovat přihlášky a údaje dítěte i do budoucna.“ Primární tlačítko _Schválit_, sekundární _Odmítnout_ (mechanika už rozhodnutá).

### D3 · Váha tokenového odkazu vs. tlak na založení účtu 🔴

- Škála od „token je hlavní cesta, účet jen šeptáme“ po „mezistránka s pobídkou k účtu po každé přihlášce“.
- Jak varovat, že **e-mail _je_ klíč**, a co zažívá člověk se třemi tokenovými odkazy a žádným účtem.

**→ Návrh:** **token je hlavní cesta, účet se nabízí jednou a nevtíravě.** Žádný interstitial — ten by z každé přihlášky udělal registrační zeď a odradil přesně ty rodiče, kvůli kterým tokenová cesta existuje.
Konkrétně: v potvrzovacím e-mailu sekundární CTA pod platebními údaji, v rozcestníku trvalý nenápadný pruh „Mít všechny přihlášky na jednom místě“.
Dvě věci ale povinně: (1) v rozcestníku i v e-mailu věta **„Tento odkaz je klíč k přihlášce — nepřeposílejte ho“**, (2) vždy dostupné _Ztratili jste odkaz?_ (mechanika je popsaná v [non-functional.md](docs/non-functional.md) → Tokeny).
Jediná výjimka z nevtíravosti: přijde-li z téhož e-mailu **druhá a další přihláška**, nabídku účtu zvýraznit — tři odkazy v poště jsou přesně ta bolest, kterou účet řeší.

---

## 2 · Plocha A — veřejný portál

### Q-A6 · Kapacita vyčerpaná během vyplňování 🔴

- Někdo vzal poslední lůžko, zatímco rodič vyplňoval. Spec ten okamžik neřeší.

**→ Návrh:** guard na serveru už existuje — doplnit **chování při jeho selhání**: formulář se nezahazuje, uživatel dostane stránku „Poslední místo bylo právě obsazeno“ s vyplněnými daty a dvěma tlačítky _Přihlásit jako náhradníka_ / _Zrušit_. U skupinové přihlášky totéž po účastnících (viz Q-A27).
Vyžaduje doplnění do [docs/registration-lifecycle.md](docs/registration-lifecycle.md) — dnes je popsaný jen guard, ne cesta ven.

### Q-A8 · Znovuodeslání a **oprava překlepu** v e-mailu zástupce 🟠

- Znovuposlání je vyřešené, ale výhradně na původní adresu. Překlep dnes znamená tiché propadnutí po 7 dnech.

**→ Návrh:** doplnit do rozcestníku akci **„Opravit e-mail zástupce“**, dostupnou jen ze stavu `PendingGuardian` a jen držiteli tokenu přihlášky. Oprava zneplatní starý token, vydá nový a **lhůta 7 dnů běží znovu od nuly** — na rozdíl od prostého znovuposlání, kde se lhůta záměrně neposouvá (jiná adresa = jiná žádost, ne prodloužení té staré).
Limit 3 opravy na přihlášku, každá do `AUDIT_LOG`. Vyžaduje doplnění do [docs/registration-lifecycle.md](docs/registration-lifecycle.md) a [docs/non-functional.md](docs/non-functional.md) → Tokeny.

### Q-A10 · Co následuje po schválení zástupcem 🟡

**→ Návrh:** poděkování + primární odkaz do rozcestníku přihlášky (zástupce chce vidět, co bude dál a kolik platit), a **jedna** sekundární nabídka účtu s konkrétním užitkem: „Příště přihlásíte dítě na dvě klepnutí.“ Slepá ulička ne — zástupce je od téhle chvíle rodič v systému a bude platit.

### Q-A12 · Tření a formulace u potvrzení storna 🟡

**→ Návrh:** dvoukrokové potvrzení s **vyčíslením k dnešnímu dni**: „Storno k 10. 9.: poplatek 500 Kč z 2 000 Kč. Vrátíme vám 1 500 Kč — vratku posílá oddíl ručně, obvykle do 14 dnů.“
Vratka je mimo systém, takže ji nikdy neformulovat jako automatickou. Očekávání se nastavuje tady, jinak dorazí telefonát za tři dny.

### Q-A13 · Míra tření u storna **zaplacené** přihlášky 🟡

**→ Návrh:** **stejné dvoukrokové potvrzení jako u nezaplacené** — zvláštní tření nepřidávat. Důvod jako **nepovinné** pole s rychlými volbami (nemoc · jiný program · rodinné důvody · jiné), zobrazené vedoucímu u přihlášky.
Povinný důvod by byl bariéra u operace, na kterou má rodič právo; nepovinný s předvolbami vyplní většina lidí a vedoucímu to pomůže při výběru náhradníka.

### Q-A16 · Vzniká vazba zákonný zástupce ↔ dítě u tokenové přihlášky bez účtu? 🔴

**→ Návrh:** **držitel tokenu není rodič.** Přidá-li anonymní držitel odkazu dalšího nezletilého účastníka, vazba nevzniká a dítě prochází standardní bránou — buď má aktivní vazbu (schvaluje stávající zástupce), nebo se vyžádá e-mail zástupce.
Prohlášení o zastoupení, které dnes zakládá vazbu přímo, smí učinit **jen přihlášený účet** — jinak by kdokoli s přeposlaným odkazem získal trvalá práva k cizímu dítěti.
Vyžaduje doplnění do [docs/parent-child-lifecycle.md](docs/parent-child-lifecycle.md).

### Q-A17 · Jedno souhrnné potvrzení, nebo e-mail per dílčí přihláška? 🟠

**→ Návrh:** **jeden souhrnný e-mail** za odeslání, s tabulkou účastník / VS / částka a jedním odkazem do rozcestníku skupiny.
Technicky: notifikaci navázat na kořenovou přihlášku a u dílčích s vyplněným `parent_registration_id` potlačit. Tři e-maily za tři sourozence znamenají tři QR v poště a jistotu, že rodič jeden přehlédne.
Doplnit do [docs/notifications.md](docs/notifications.md) jako pravidlo slučování.

### Q-A19 · Smí účastník nahradit už **schválený** dokument? 🟠

**→ Návrh:** **ano, povolit** — dokumenty prošlé platnosti nebo omylem nahrané cizí soubory jsou reálné případy. Nahrazení zakládá nový záznam, starý zůstává kvůli auditu, a přihláška se vrací do „čeká na dokumenty“ s explicitním varováním **před** potvrzením: „Nahrazením se dokument bude posuzovat znovu a přihláška se vrátí mezi čekající.“
Doplnit přechod do [docs/registration-lifecycle.md](docs/registration-lifecycle.md).

### Q-A20 · Návod na focení dokumentu 🟡

**→ Návrh:** tři ikonky s příklady přímo nad tlačítkem nahrání (celý dokument v záběru · ostré · bez odlesku), plus klientská kontrola: je-li delší strana pod ~1000 px, varovat ještě před odesláním. Nejlevnější prevence zamítnutí, jakou máme — každé zamítnutí stojí kolo e-mailů a práci vedoucího.

### Q-A21 · Rámování komentáře vedoucího k zamítnutí 🟡

**→ Návrh:** komentář **citovat doslova**, ale zarámovat strukturou: nadpis „Potřebujeme dokument znovu“, pod ním blok „Poznámka vedoucího: …“, pod ním neutrální instrukce a tlačítko.
Tón se řeší u zdroje předvolbami důvodů (D8), ne přepisováním toho, co vedoucí napsal — obalování cizího textu šablonou vytváří falešný dojem, že to říká systém.

### Q-A24 · Připomínka **před** vypršením nabídky náhradníkovi 🟠

**→ Návrh:** doplnit **`EMAIL_SUBSTITUTE_OFFER_REMINDER`** po 24 hodinách, jen pokud je nabídka stále nepřijatá. Jedno odeslání, žádná eskalace.
48hodinové okno začínající v pátek večer je jinak snadné prošvihnout a propadlá nabídka stojí místo jak náhradníka, tak oddílu. Zapsat do [docs/notifications.md](docs/notifications.md).

### Q-A25 · Volí dobrovolník výběrové číselníky? 🟠

**→ Návrh:** **ano, se stejnými pravidly jako účastník** — číselník se zobrazí, pokud ho `condition` nevyloučí; kategorie `volunteer` sama o sobě není filtr. Dobrovolník na táboře spí a jí stejně jako ostatní.
Potřebuje-li akce dobrovolníkům nabídnout jiné ubytování, udělá to podmínkou způsobilosti (Q-B4), ne zvláštním pravidlem. Doplnit větu do [docs/event-fields.md](docs/event-fields.md).

### Q-A26 · Platí pro dobrovolníky povinné dokumenty akce? 🟠

**→ Návrh:** doplnit na `EVENT_DOCUMENT` pole **`applies_to`** (`participant` / `volunteer` / `both`, výchozí `participant`).
Bez něj by dospělý dobrovolník musel dodávat souhlas zákonného zástupce, zatímco potvrzení o bezúhonnosti by po něm nešlo vyžádat vůbec. Vyžaduje doplnění do [docs/data-model.md](docs/data-model.md) a [docs/validation.md](docs/validation.md).

### Q-A27 · Kapacita stačí jen pro část skupiny 🔴

**→ Návrh:** **přijmout, co se vejde, zbytek nabídnout jako náhradníky ve stejném kroku** a nechat rodiče potvrdit. Nikdy nerozhodovat za něj a nikdy neodmítat celou skupinu.
Text: „Volná jsou 2 místa ze 3. Přihlásit Terezu jako náhradnici?“ s volbou _Ano, přihlásit všechny_ / _Přihlásit jen ty, na které je místo_ / _Zrušit celou přihlášku_.
Vyžaduje doplnění pravidla do [docs/registration-lifecycle.md](docs/registration-lifecycle.md) — dnes se kapacita vyhodnocuje per přihláška bez ohledu na sourozence.

### Q-A28 · Formulace stavu „dočasně zamčeno“ u přihlášení 🟡

**→ Návrh:** **říct pravdu** — vlastník o zamčení stejně dostal e-mail: „Účet jsme dočasně zamkli po několika neúspěšných pokusech o přihlášení. Zkuste to znovu za 15 minut, nebo si nastavte nové heslo.“
Enumeraci účtů to neprozradí, protože stejná hláška se ukáže i u neexistujícího e-mailu. Nechat projít bezpečnostním review společně s hláškou o resetu hesla.

### Q-A29 · Okamžik volby workshopových běhů 🟠

**→ Návrh:** volba je **součástí přihlášky (S3)**, protože obsazuje kapacitu běhu; změna je pak možná z rozcestníku, dokud běh není plný a akce nezačala.
Formálně: modelovat běhy jako `EVENT_FIELD` s `required_phase = on_submit` a kapacitou na `EVENT_FIELD_OPTION` — pak nevzniká žádný nový mechanismus a platí i pravidlo o nulovém `price_modifier` u pozdních fází.

### Q-A30 · Ukazovat zbývající volná místa běhů? 🟡

**→ Návrh:** **neukazovat počty**, jen tři stavy: _volno · poslední místa · plno_, kde „poslední místa“ nastupuje pod 20 % kapacity. Konzistentní s Q-A2. Konkrétní číslo („zbývají 2 místa“) vytváří tlak, který u dětské akce nechceme, a navíc se v okamžiku zobrazení už může měnit.

### Q-A31 · Jak dopředu komunikovat riziko rozpuštění hlídky 🟡

**→ Návrh:** vysvětlit **jednou při zakládání hlídky**, ne trvalým varovným pruhem: „Hlídka musí splňovat věková pravidla po celou dobu. Změní-li se údaje člena tak, že pravidla přestanou platit, hlídka se rozpustí a budete ji muset složit znovu.“
Podruhé pak v e-mailu při rozpuštění, s konkrétním důvodem (kanál je už rozhodnutý — `EMAIL_PATROL_DISSOLVED`). Permanentní varování lidé přestanou číst dřív, než se stane to, před čím varuje.

### Q-A32 · Lhůta a připomínky pro mentora 🟠

**→ Návrh:** **lhůtu nezavádět** — doporučení stejně neblokuje stav přihlášky, kapacitu ani platbu, takže vypršení by nemělo co způsobit. Místo toho doplnit **`EMAIL_MENTOR_REMINDER`** po 7 dnech, pokud žádost není potvrzená, a stav ukázat účastníkovi v rozcestníku jako informaci („Mentor zatím nepotvrdil“).
Méně lhůt v systému = méně jobů, méně stavů a méně vysvětlování.

---

## 3 · Plocha B — oddíl

### Q-B1 · Lze založit akci _bez_ šablony? 🟠

**→ Návrh:** **šablona povinná**, ale mezi systémovými šablonami mít „Prázdná akce“ jako plnohodnotný záznam. Kód má jednu cestu, data zůstanou konzistentní (`action_template_id` je vždy vyplněné) a úniková cesta zůstává. Výjimka „akce bez šablony“ by znamenala druhou větev všude, kde se ze šablony čtou výchozí hodnoty.

### Q-B2 · Vzniká SS akce automaticky, nebo ho zadává HVO? 🔴

**→ Návrh:** **generovat automaticky a needitovatelně** — číselná řada v rámci bankovního účtu, unikátnost vynucená v databázi.
SS je jediný identifikátor akce při párování plateb. Ruční zadání znamená, že jeden překlep tiše rozbije párování všech plateb akce a projeví se to až upomínkami lidem, kteří zaplatili. Úspora „chci si zvolit hezké číslo“ za to nestojí.
Vyžaduje doplnění unikátnosti `EVENT.ss` v rámci `bank_account_id` do [docs/validation.md](docs/validation.md).

### Q-B3 · Úprava a smazání položky číselníku, kterou už si účastníci zvolili 🟠

**→ Návrh:** **smazání blokovat**, dokud na položku ukazuje aspoň jedna `REGISTRATION_FIELD_VALUE`; místo toho nabídnout **„skrýt z nabídky“** — položka se přestane nabízet novým, existující volby zůstanou platné.
Změnu `price_modifier` naopak **povolit bez potvrzovacího dialogu**: zafixovaná cena u existujících voleb se stejně nepřeceňuje, takže dotčené přihlášky žádné nejsou.
Vyžaduje doplnit `EVENT_FIELD_OPTION.hidden_at` do [docs/data-model.md](docs/data-model.md).

### Q-B4 · Jak se zadává podmínka způsobilosti číselníku 🟠

**→ Návrh:** **skládačka s pevnou gramatikou**, uložená do `condition` jako JSON: `{věk od–do}` AND `{členství DU: ano / ne / nerozhoduje}` AND `{kategorie: účastník / dobrovolník / nerozhoduje}`.
Volný text nemá kdo vyhodnotit — číselník by musel filtrovat člověk, což je přesně to, čemu se pole `condition` snaží předejít. Tři operandy pokryjí všechny příklady, které spec uvádí.

### Q-B5 · Hromadné operace nad přihláškami 🟠

**→ Návrh:** doplnit do spec **dvě** hromadné akce a nic víc: _poslat připomínku platby_ a _exportovat výběr_ (Q-B6).
Hromadné storno nezavádět — zrušení celé akce spec už umí a je auditovatelné, zatímco „vybrat 40 řádků a stornovat“ je operace, kde jeden omyl znamená 40 e-mailů rodičům a nevratné storno poplatky.

### Q-B6 · Export přihlášek do tabulky 🟠

**→ Návrh:** CSV export tabulky, který respektuje **filtry i sloupce, jaké má daná role vidět** — tedy i maskování platebních atributů pro Rádce, a to na serveru, ne skrytím sloupce v UI.
Doplnit „Exportovat seznam přihlášek“ jako samostatnou operaci do matice v [docs/authorization.md](docs/authorization.md); export citlivých sloupců je jiná operace než jejich zobrazení na obrazovce.

### Q-B7 · Fronta dokumentů per akce, nebo společná schránka? 🟠

**→ Návrh:** **společná schránka** přes všechny akce, ke kterým je vedoucí přiřazený, s filtrem na akci a výchozím řazením podle nejdéle čekajících. Fronta u akce zůstane jako předfiltrovaná varianta téže obrazovky.
Vedoucí posuzuje dokumenty v dávce („mám deset minut“), ne po akcích — nutit ho obcházet akce znamená, že na některou zapomene a rodič čeká.

### Q-B8 · Zobrazení HEIC v prohlížeči 🟠

**→ Návrh:** **konverze náhledu na serveru při nahrání** — vedle originálu uložit odvozený JPEG náhled, originál se stahuje beze změny.
Bez toho je fronta B4 na iPhonových fotkách nepoužitelná: vedoucí by musel každý soubor stáhnout a otevřít mimo prohlížeč, což je přesně ten typ tření, po kterém dokumenty zůstanou neposouzené. HEIC je mezi povolenými formáty právě proto, že rodiče fotí telefonem — pak to musíme umět zobrazit.
Doplnit do [docs/non-functional.md](docs/non-functional.md) → Úložiště souborů.

### Q-B9 · Smí běžet víc paralelních nabídek náhradníkům? 🟠

**→ Návrh:** **povolit paralelní nabídky, ale nikdy víc nabídek než volných míst** (guard: počet aktivních nabídek ≤ počet volných míst).
Tím zmizí riziko přeslibu, kvůli kterému by někdo dostal „místo je vaše“ a vzápětí „bohužel“, a zároveň se neztrácí rychlost — u pěti uvolněných míst nemá smysl obesílat po jednom a čekat 5× 48 hodin.
Doplnit do [docs/registration-lifecycle.md](docs/registration-lifecycle.md).

### Q-B10 · Přehled nedávných **automatických** alokací 🟡

**→ Návrh:** **přidat** jako záložku „Nedávno spárováno“ (posledních 7 dní): datum, částka, cíl, `match_method`, `matched_by` a akce _rozpárovat_. Data už se evidují, takže jde jen o obrazovku.
Bez ní účetní automatu nevěří a kontroluje ručně — čímž se celá úspora z automatického párování ztratí. Je to nejlevnější důvěryhodnostní prvek v celém modulu.

### Q-B11 · AI návrh nejpravděpodobnější přihlášky při selhání SS/VS 🟠

**→ Návrh:** **v první verzi nezavádět.** Deterministická pravidla plus ruční dvoupanel pokrývají všechny případy; AI návrh by potřeboval vlastní vizuální jazyk, vlastní auditní stopu a odpověď na otázku, kdo nese odpovědnost za špatně spárovanou platbu.
Zavede-li se později: odlišná barva a ikona, **vždy s uvedeným důvodem shody** („jméno odesílatele odpovídá vlastníkovi, částka sedí“) a **nikdy jako předvybraná volba**. Aktuální [docs/payment-matching.md](docs/payment-matching.md) žádné AI pravidlo neobsahuje, takže „nezavádět“ je i levnější cesta k souladu spec a UI.

### Q-B12 · Předvyplňovat převod přeplatku na jinou přihlášku téže osoby? 🟠

**→ Návrh:** **nabízet, nepředvyplňovat.** V detailu přeplatku ukázat seznam otevřených přihlášek téže osoby s dlužnými částkami a tlačítkem _Převést sem_; předvybrané nic není.
Systém tím ušetří účetní hledání (což je ta pracná část), ale rozhodnutí, jestli se peníze vrací nebo převádí, zůstane na člověku — je to rozhodnutí o cizích penězích.

### Q-B13 · Offline režim zápisu docházky 🔴

**→ Návrh:** **fronta zápisů v local storage se synchronizací** — střední varianta, ne plná PWA.
Docházka je jediná obrazovka, která offline potřebuje, a je to jednoduchý zápis bez konfliktů (klíč osoba + akce + datum, poslední zápis vyhrává). Plná PWA by znamenala offline vrstvu pro celou aplikaci včetně autorizace a citlivých dat — nepoměrný náklad i riziko.
Rozhodnout hned: znamená to, že frontend musí od začátku umět servisní vrstvu s frontou, což se dodatečně dolepuje draho. Doplnit do [docs/non-functional.md](docs/non-functional.md).

### Q-B14 · Zobrazení archivovaných (anonymizovaných) osob v tabulce 🟡

**→ Návrh:** v tabulce osob **nezobrazovat vůbec** — anonymizovaná osoba nemá jméno ani údaje, takže není co ukázat a řádek by jen mátl.
V historických kontextech, kde by vznikla díra (seznam účastníků staré akce, docházka), ponechat řádek **„Anonymizovaná osoba“** bez odkazu. Součty a počty brát z reportů, které s `anonymized_at` pracují.

### Q-B15 · Odebrání role ÚČE / VO / RÁD 🔴

**→ Návrh:** role **nemazat, ale uzavírat** — doplnit na `USER_ROLE` pole `revoked_at` a `revoked_by_account_id`, přesně jako to má `EVENT_ASSIGNMENT`. Kontrola oprávnění pracuje jen se záznamy `revoked_at IS NULL`.
Odebrání role automaticky uzavře všechna otevřená `EVENT_ASSIGNMENT` toho účtu v daném oddílu (jinak by přiřazení k akci přežilo roli, na které stojí) a zapíše se do auditu. Poslední HVO oddílu odebrat nelze (Q-C4).
Vyžaduje doplnění do [docs/data-model.md](docs/data-model.md) a [docs/authorization.md](docs/authorization.md).

### Q-B16 · Vizuální jazyk reportů 🟡

**→ Návrh:** ponechat na dataviz fázi, ale **zafixovat teď tři věci**, aby se B11 a C4 nerozešly: jeden sdílený set komponent pro obě plochy · žádné koláčové grafy (podíly řešit skládaným vodorovným pruhem) · mapování formy na otázku — časová řada = čára, srovnání kategorií = vodorovný pruh, jedno číslo = velké číslo s trendem.

---

## 4 · Plocha C — ústředí

### Q-C3 · Platnost pozvánky role a chování po vypršení 🟠

**→ Návrh:** **14 dní**, stejně jako u vazby zákonného zástupce — držet počet různých lhůt v systému co nejnižší, protože každá je vlastní job, vlastní text a vlastní zdroj překvapení.
Po vypršení stránka „Pozvánka vypršela“ s tlačítkem _Požádat o novou_, které pošle notifikaci **zvoucímu** (HVO/ADM), ne obecně administrátorovi.

### Q-C4 · Výměna hlavního vedoucího 🔴

**→ Návrh:** oddíl **smí mít víc HVO současně** a **nesmí zůstat bez HVO**.
Překryv je normální provoz (předávání trvá týdny), takže omezení na jednoho by lidi nutilo k nebezpečným obchvatům — sdílení hesla. Naopak odebrání poslední role HVO zablokovat s hláškou „Nejdřív jmenujte nového hlavního vedoucího“, protože HVO je jediný schvalovatel pro osiřelé děti, výjimky jmen a pozvánky.
Odcházejícímu se role odebere běžnou cestou (Q-B15). Zapsat do [docs/authorization.md](docs/authorization.md).

### Q-C5 · Životní cyklus oddílu 🟠

**→ Návrh:** doplnit **`UNIT.state`** (`active` / `suspended` / `closed`). `closed` nepřijímá nové akce ani přihlášky, existující dobíhají; osoby, historie a členská evidence zůstávají kvůli retenci (členství + 10 let) a reportům ústředí.
Skrývání ne — report unikátních dětí za rok 2024 musí sedět i poté, co oddíl v roce 2026 skončil. Vyžaduje doplnění do [docs/data-model.md](docs/data-model.md) a krátkou kapitolu v `region-lifecycle.md` (nebo vlastní `unit-lifecycle.md`).

### Q-C6 · Zpětné datum přesunu oddílu mezi regiony 🟠

**→ Návrh:** povolit **jen „od teď“** (`valid_from` = dnes), zpětné datum nepodporovat.
Akce nesou `region_id_snapshot` pořízený při založení; zpětná změna příslušnosti by rozešla snapshoty s historií a umožnila zpětně přepsat už odevzdané regionální výkazy. Opravu chyby řešit zásahem ADM s povinným důvodem a auditem, ne běžným UI.

### Q-C7 · Váha zamítnutí žádosti o sloučení 🟡

**→ Návrh:** **jeden potvrzovací modal**, ne dvoukrokový průvodce. Text musí říct důsledek: „Dvojici už systém znovu nenabídne. Odblokovat ji může jen administrátor s uvedením důvodu.“
Zamítnutí je legitimní a časté rozhodnutí (jmenovci existují) — přidávat tření by ho zpomalilo víc, než kolik ušetří na omylech. Riziko řeší informace, ne počet kliknutí.

### Q-C8 · Má být směr sloučení viditelný a volitelný? 🟠

**→ Návrh:** **ukázat, ale nedat volit.** V porovnání jasně označit „Zůstane tato osoba“ u cílové a „Bude přesměrována“ u zdrojové, s vysvětlením, že všechny vazby se přenesou a zdrojová osoba zůstane jako přesměrování.
Volba směru by otevřela otázku, co dělat, když ji dva schvalovatelé vidí různě, a nepřinesla by nic — po sloučení se hodnoty polí stejně vybírají jednotlivě.

### Q-C9 · Omezit období reportu R10 na kalendářní roky? 🟡

**→ Návrh:** **předvolit kalendářní rok** jako výběr ze seznamu (2025, 2026, …); obecný interval schovat pod „vlastní období“ s poznámkou „Vykazovací hodnota je za kalendářní rok“.
Nezakazovat — jiný interval má legitimní analytické použití — ale výchozí cesta musí být ta, jejíž výsledek se smí poslat do výkazu.

### Q-C10 · Nabídnout z reportovacího sloučení cestu ke skutečnému? 🟠

**→ Návrh:** **nabídnout, ale jako sekundární odkaz s vysvětlením**, ne jako tlačítko vedle „sloučit pro report“: „Jde o tutéž osobu i v datech? Založit žádost o sloučení záznamů“ + jedna věta o rozdílu (report nemění data, sloučení ano a je vratné jen administrátorem).
Skrýt cestu úplně by znamenalo, že duplicity v datech zůstanou neřešené, přestože je ústředí právě našlo. Riziko záměny se dá zvládnout textem a odlišnou vizuální váhou.

### Q-C11 · Co přesně znamená „absolvování“ akce 🔴

**→ Návrh:** **docházkový záznam se stavem přítomnosti** (`on_time` nebo `late`) na akci s vyplněným `course_id`, vyhodnocený po skončení akce.
Bez docházkového záznamu se kurz neudělí automaticky a vedoucí ho může přidělit ručně (což je stejně potřeba pro doklady mimo systém). Alternativa „konec akce = absolvoval“ by udělovala kvalifikaci lidem, kteří nepřijeli.
Nutné potvrdit se zadavatelem a zapsat do spec — stojí na tom celá automatika C5.

### Q-C12 · Smí ADM otevřít soubor certifikátu? 🟠

**→ Návrh:** **ano, ale se zápisem do auditu** jako čtení citlivého obsahu — táž mechanika jako u zdravotních údajů.
Bez přístupu k souboru nelze kvalifikaci ověřit, což je celý účel modulu vzdělávání; zákaz by z ADM udělal razítkovač. Doplnit řádek „Číst soubor certifikátu“ do matice v [docs/authorization.md](docs/authorization.md) — dnes tam kapitola _Vzdělávání a kvalifikace_ přístup k `certificate_file` nerozepisuje.

### Q-C13 · Opakovaná výjimka jména napříč oddíly 🟡

**→ Návrh:** **žádná automatika**, jen signál: v přehledu výjimek sloupec „Schváleno výjimkou v N oddílech“ s řazením podle N a prahem zvýraznění při N ≥ 3, plus akce _Přidat do whitelistu_ na jedno kliknutí.
Automatické přidávání by z lokálního omylu tří oddílů udělalo celostátní pravidlo. Rozhodnutí zůstane na člověku, systém jen přestane spoléhat na to, že si toho někdo všimne sám.

### Q-C14 · Vidí ADM oddílové šablony akcí? 🟡

**→ Návrh:** **jen ke čtení a jen z detailu oddílu**, ne v hlavním seznamu šablon.
Bez čtení nelze oddílu pomoct, když se ptá „proč se nám akce zakládá s divnými cenami“; s editací by ADM zasahoval do oddílové autonomie, kterou spec jinde důsledně chrání.

### Q-C15 · Počáteční naplnění whitelistu křestních jmen 🔴

**→ Návrh:** spustit modul v **režimu našeptávače bez blokace**: whitelist se naplní importem veřejného seznamu jmen (matriční číselník) a do doby, než ho ADM prohlásí za dostatečný, se neshoda **jen loguje**, nic neblokuje. Blokaci zapíná ADM jedním přepínačem.
Prázdný whitelist by první den provozu označil každé jméno a modul by musel být vypnutý — což je horší stav než postupné plnění, protože se na něj zapomene. Vyžaduje doplnit systémový přepínač do spec.

---

## 5 · Plocha D — samoobsluha přihlášeného uživatele

### Q-D1 · Segmentace seznamu „Moje přihlášky“ 🟠

**→ Návrh:** dvě záložky — **Aktivní** (výchozí) a **Historie** — a uvnitř seskupení po osobách, jakmile má uživatel víc než jedno dítě. Řadit podle nejbližšího termínu akce. Filtry nezavádět; rodič má jednotky přihlášek, ne desítky.

### Q-D2 · Vidí rodič přihlášky ve stavu `PendingGuardian`, které sám nepodal? 🟠

**→ Návrh:** **ano, vidí všechny přihlášky dítěte** bez ohledu na to, kdo je podal, včetně těch čekajících na schválení.
Opak by znamenal, že přihláška podaná dítětem je pro rodiče neviditelná až do chvíle, kdy po 7 dnech propadne — a rodič se o ní dozví jen z e-mailu, který mohl přehlédnout. Aktivní vazba dává právo na údaje dítěte, tohle je jeho přirozený rozsah. Doplnit do [docs/authorization.md](docs/authorization.md).

### Q-D3 · Ukázat druhému rodiči, kdo přihlášku naposledy změnil? 🟡

**→ Návrh:** **ano**, jedna řádka pod hlavičkou přihlášky: „Naposledy upravila Jana Nováková, 3. 9. v 18:20.“ Data má auditní log, takže jde jen o zobrazení.
Řeší nejčastější zmatek u dvou rodičů s plnými právy a pravidlem „platí poslední zápis“ — bez toho vypadá tichá změna jako chyba systému. Jméno druhého zástupce je informace, kterou rodič už zná.

### Q-D4 · Tření a notifikace u zrušení vazby na dítě 🟠

**→ Návrh:** **dvoukrokové potvrzení s výčtem dopadu**, ne prostý dialog: „Ztratíte přístup k přihláškám a údajům dítěte. Přihlášky zůstanou v platnosti a přejdou pod druhého zákonného zástupce (nebo hlavního vedoucího oddílu).“
Plus **notifikace druhému zástupci a HVO oddílu** — u posledního zástupce dítěte přechází odpovědnost na HVO, takže se to nesmí dozvědět náhodou. Zrušení nelze blokovat (nelze držet zástupce proti jeho vůli), takže tření má být informační, ne překážkové.
Doplnit notifikaci do [docs/notifications.md](docs/notifications.md).

### Q-D5 · Smí rodič měnit datum narození dítěte? 🔴

**→ Návrh:** rodič smí, **dokud dítě nemá žádnou přihlášku v nekoncovém stavu**; jinak jen HVO. Změna vždy s varováním o dopadech (věkové brány akcí, složení hlídek) a zápisem do auditu.
Volná editace kdykoli by dovolila obejít věkovou podmínku akce po přihlášení; úplný zákaz by naopak nutil rodiče volat vedoucímu kvůli běžnému překlepu při registraci.

### Q-D6 · Editace vlastního data narození 🟠

**→ Návrh:** **stejný guard jako u dítěte** (Q-D5) — bez otevřené přihlášky volně, jinak přes HVO. Konzistence pravidla je tady důležitější než pohodlí; dvě různá pravidla pro tutéž hodnotu se nedají vysvětlit ani otestovat.

### Q-D7 · Kdo a kdy posílá výzvu k převzetí účtu po zletilosti 🟠

**→ Návrh:** **automaticky v den 18. narozenin**, navěšeno na job, který už tak jako tak překlápí vazbu do `readonly_after_adulthood`. E-mail jde na kontaktní adresu osoby, existuje-li; rodič má v detailu dítěte tlačítko _Poslat výzvu znovu_ (a možnost chybějící e-mail doplnit — výjimka v právech je přesně na tohle).
Nechat to na tlačítku rodiče by znamenalo, že se výzva u většiny dětí neodešle nikdy. Doplnit `EMAIL_ACCOUNT_TAKEOVER` do [docs/notifications.md](docs/notifications.md).

### Q-D8 · Odvolání pozvánky druhého zástupce zvoucím rodičem 🟡

**→ Návrh:** **doplnit** — zvoucí smí pozvánku ve stavu `pending` odvolat týmž tlačítkem, kterým ji poslal (→ `canceled`, důvod „odvoláno zvoucím“, token se zneplatní).
Bez toho zůstane pozvánka poslaná na špatnou adresu viset 14 dní a jedinou cestou ven je čekat. Vyžaduje doplnit přechod do [docs/parent-child-lifecycle.md](docs/parent-child-lifecycle.md).

### Q-D9 · Odvolání souhlasu — self-service, nebo žádost? 🟠

**→ Návrh:** **self-service tlačítko** u každého odvolatelného souhlasu, s okamžitým účinkem a jednou větou o důsledku („Bez souhlasu s fotografováním vás nebudeme fotit na akcích“).
Souhlas, který nejde odvolat vlastní silou, není souhlas — a žádost vyřizovaná HVO by z toho udělala měsíc čekání. Doplnit do spec, že odvolání zapisuje `revoked_at` a aktéra do auditu.

### Q-D10 · Kde se podává žádost o výmaz (GDPR) 🔴

**→ Návrh:** tlačítko **„Požádat o výmaz údajů“** v D3-S1, které založí záznam nové entity **`ERASURE_REQUEST`** (`person_id`, `requested_at`, `state`, `resolved_by_account_id`, `note`); denní job ho zpracuje a HVO/ADM potvrdí.
Řešit to „přes podporu“ nejde: GDPR má lhůty a systém potřebuje doklad o vyřízení — což je mimochodem přesně to, co už denní job „vyřízení žádostí o výmaz“ předpokládá, jen mu chybí vstup.
Vyžaduje doplnění do [docs/data-model.md](docs/data-model.md) a [docs/non-functional.md](docs/non-functional.md).

### Q-D11 · Změna přihlašovacího e-mailu 🟠

**→ Návrh:** **ano, už v první verzi, s ověřením nové adresy**: zadání → potvrzovací odkaz na novou adresu → po potvrzení se `login_email` změní a na starou adresu jde informativní zpráva („Přihlašovací e-mail byl změněn. Nebyli jste to vy?“).
Bez toho je ztráta přístupu ke staré schránce neřešitelná bez zásahu ADM do databáze — a u rodičů, kteří mění zaměstnavatele a s ním e-mail, je to běžná situace, ne okrajová.

### Q-D12 · Zrušení vlastního účtu uživatelem 🟠

**→ Návrh:** **ano, ale jako zrušení přihlášení, ne výmaz osoby.** Zruší se `ACCOUNT`; osoba, přihlášky, docházka a členství zůstávají a řídí se retencí.
Text to musí říct předem a doslova, jinak uživatel očekává smazání údajů (od toho je Q-D10) a vznikne stížnost. Guard: poslednímu HVO oddílu zrušení nedovolit (Q-C4), stejně jako uživateli s aktivní vazbou, kde je jediným zástupcem nezletilého — nejdřív se musí vypořádat vazba.

### Q-D13 · Kde a jak nabídnout kandidáta na sloučení osob 🟡

**→ Návrh:** **nevtíravá sekce v D4** — žádný banner po přihlášení, žádný e-mail.
Nadpis neutrální a vysvětlující: **„Našli jsme podobný záznam“** místo „Nejste to vy?“, pod ním jedna věta proč („Aby se vaše údaje a historie neevidovaly dvakrát“) a dvě **rovnocenná** tlačítka _Ano, jsem to já_ / _Ne, to není moje_.
Nabídka u osobních dat působí snadno strašidelně; neutrální rámování a absence tlaku jsou tady důležitější než míra odezvy.

---

## Dodatek · Změny specifikace, které z návrhů plynou

Návrhy výše nejsou jen UX — 26 z nich znamená doplnit něco do `docs/`. Souhrn pro případ, že se odsouhlasí:

| #     | Změna                                                                | Kam                                              |
| ----- | -------------------------------------------------------------------- | ------------------------------------------------ |
| Q-A6  | chování guardu při vyčerpání kapacity během vyplňování               | `registration-lifecycle.md`                      |
| Q-A8  | akce „opravit e-mail zástupce“ (nový token, restart lhůty, limit 3×) | `registration-lifecycle.md`, `non-functional.md` |
| Q-A16 | držitel tokenu nezakládá vazbu; prohlášení jen z účtu                | `parent-child-lifecycle.md`                      |
| Q-A17 | slučování potvrzení u dílčích přihlášek                              | `notifications.md`                               |
| Q-A19 | nahrazení schváleného dokumentu a pád stavu                          | `registration-lifecycle.md`                      |
| Q-A24 | `EMAIL_SUBSTITUTE_OFFER_REMINDER` (24 h)                             | `notifications.md`                               |
| Q-A26 | `EVENT_DOCUMENT.applies_to`                                          | `data-model.md`, `validation.md`                 |
| Q-A27 | částečné přijetí skupiny + zbytek jako náhradníci                    | `registration-lifecycle.md`                      |
| Q-A32 | `EMAIL_MENTOR_REMINDER` (7 dní)                                      | `notifications.md`                               |
| Q-B2  | unikátnost `EVENT.ss` v rámci účtu, generování                       | `validation.md`                                  |
| Q-B3  | `EVENT_FIELD_OPTION.hidden_at` + zákaz mazání s vazbami              | `data-model.md`, `validation.md`                 |
| Q-B6  | export seznamu přihlášek jako samostatná operace                     | `authorization.md`                               |
| Q-B8  | serverový náhled pro HEIC                                            | `non-functional.md`                              |
| Q-B9  | guard „nabídek ≤ volných míst“                                       | `registration-lifecycle.md`                      |
| Q-B13 | offline fronta docházky                                              | `non-functional.md`                              |
| Q-B15 | `USER_ROLE.revoked_at` + kaskáda na `EVENT_ASSIGNMENT`               | `data-model.md`, `authorization.md`              |
| Q-C4  | víc HVO povoleno, poslední nelze odebrat                             | `authorization.md`                               |
| Q-C5  | `UNIT.state`                                                         | `data-model.md`                                  |
| Q-C11 | definice „absolvování“ = docházka na akci s `course_id`              | `validation.md`                                  |
| Q-C12 | čtení `certificate_file` v matici rolí                               | `authorization.md`                               |
| Q-C15 | přepínač blokace whitelistu jmen                                     | `validation.md`                                  |
| Q-D2  | rodič vidí i `PendingGuardian` přihlášky dítěte                      | `authorization.md`                               |
| Q-D4  | notifikace o zrušení vazby druhému zástupci a HVO                    | `notifications.md`                               |
| Q-D7  | `EMAIL_ACCOUNT_TAKEOVER` v den 18. narozenin                         | `notifications.md`                               |
| Q-D8  | odvolání pozvánky zvoucím rodičem                                    | `parent-child-lifecycle.md`                      |
| Q-D10 | entita `ERASURE_REQUEST`                                             | `data-model.md`, `non-functional.md`             |
