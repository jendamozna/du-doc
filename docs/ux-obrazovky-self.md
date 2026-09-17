# UX — Obrazovky plochy D (self-management)

Detailní specifikace obrazovek sekce **Můj účet** (`/muj-ucet/...`). Navigaci, routy, breadcrumbs a operační matici osoby/zástupce definuje [ux-navigace.md](ux-navigace.md) § 2.4; sdílené stavy a texty [ux-texty-stavy.md](ux-texty-stavy.md); rozcestník přihlášky a tokenové stránky [ux-obrazovky-verejny.md](ux-obrazovky-verejny.md); vazby zástupce a zletilost [parent-child-lifecycle.md](parent-child-lifecycle.md); slučování duplicit [person-merge.md](person-merge.md); role a rozsah [authorization.md](authorization.md); pravidla polí [validation.md](validation.md).

## 1. Rámec plochy

- **Publikum:** člověk s účtem — typicky rodič spravující děti, občas dospělý účastník.
- **Ergonomie:** mobile-first (jeden sloupec, karty, dotykové plochy ≥ 44 px); navštěvuje se z e-mailových připomínek. Obsah max ~720 px na střed i na desktopu.
- **Navigace:** `/muj-ucet` přesměruje na `/muj-ucet/prihlasky`. Na mobilu M3 navigation bar dole, na desktopu tytéž položky jako rail vlevo: **Přihlášky** (`/muj-ucet/prihlasky`) · **Děti** (`/muj-ucet/deti`) · **Údaje** (`/muj-ucet/udaje`) · **Účet** (`/muj-ucet/ucet`). Mimo shell je veřejná tokenová stránka `/pozvanka/:token` (D-05). Badge na položce Přihlášky = počet přihlášek vyžadujících krok uživatele.

**Návrhový princip — zrcadlo tokenového rozcestníku:** správa přihlášky tokenovým odkazem je plnohodnotný režim, ne nouzovka. Detail přihlášky v „Moje přihlášky“ je proto **tatáž obrazovka jako rozcestník `/stav/:token`** ([ux-obrazovky-verejny.md](ux-obrazovky-verejny.md) § 7) — jedna specifikace, dvě přístupové cesty. Účet přidává jen to, co token nedá: seznam **všech** přihlášek vlastních i dětí. Tento soubor rozcestník nekopíruje — popisuje jen odchylky.

## 2. D-01 · Moje přihlášky — `/muj-ucet/prihlasky`

**Účel a publikum:** jedna odpověď na otázku „co je s našimi přihláškami a co je teď na mně“ — bez hledání tokenových e-mailů.

**Layout a komponenty:** filter chips nahoře: **Aktivní** (výchozí) · **Vše**, dále chip za osobu (jednotlivé děti · Já). Pod nimi karty přihlášek (elevated) seskupené nadpisy **„Je to na vás“ · „Nadcházející“ · „Proběhlé“**. Primární akce jako filled button přímo na kartě.

**Obsah a pole:** karta = účastník (avatar s iniciálami + „za: [jméno]“), akce, termín, stavový chip s blokující bránou, kategorie (účastník / dobrovolník / náhradník), u platebních stavů zbývající částka a splatnost. Řazení uvnitř skupin: nejbližší lhůta první. Terminální přihlášky (`Canceled`/`Expired`) se zobrazují ve „Vše“ šedě, jen ke čtení. CTA dle stavu vede rovnou do relevantní sekce detailu (Zaplatit, Nahrát dokument, Odpovědět na nabídku).

**Stavy:**

- **Prázdný (účet bez přihlášek):** ikona `assignment`, „Zatím žádné přihlášky“, „Až se na akci přihlásíte vy nebo vaše děti, uvidíte tu stav, platby a dokumenty na jednom místě.“, CTA „Prohlédnout akce“ → `/`.
- **Prázdný po filtru:** ikona `search_off`, „Žádné přihlášky pro tento filtr“, CTA „Zrušit filtr“.
- **Vše hotovo (skupina „Je to na vás“ prázdná):** místo skupiny klidný řádek s ikonou `check_circle` — „Nic nečeká na váš krok. Přihlášky níže vyřizují jiní, nebo jsou hotové.“ — žádná falešná urgence.
- **Načítání:** skeleton karet (stabilní výška); **Chyba:** [ux-texty-stavy.md](ux-texty-stavy.md) § 5.

**Interakce a validace:** klepnutí na kartu → D-02; CTA na kartě vede na relevantní sekci detailu; žádné mutace přímo ze seznamu.

**Oprávnění:** vlastní přihlášky + přihlášky dětí z aktivní vazby zástupce ([authorization.md](authorization.md)).

**Mobil/desktop:** mobil karty na celou šíři; desktop tentýž jeden sloupec — hustota se nemění.

**Notifikace:** žádné ze seznamu; připomínky plateb a dokumentů přicházejí e-mailem ([notifications.md](notifications.md)).

## 3. D-02 · Detail přihlášky — `/muj-ucet/prihlasky/:pid` (zrcadlo rozcestníku)

**Účel a publikum:** aktuální stav, blokující brána, další krok — **identické s rozcestníkem** ([ux-obrazovky-verejny.md](ux-obrazovky-verejny.md) § 7): checklist bran, dokumenty, platba s QR/VS/SS, storno s náhledem poplatku. Zde jen odchylky pro přihlášeného uživatele.

**Layout a komponenty:** stejná komponenta jako `/stav/:token`, vykreslená uvnitř shellu Můj účet (top app bar se šipkou zpět na D-01, navigation bar zůstává).

**Obsah a pole — odchylky oproti tokenové cestě:**

- Nad hlavičkou banner (tonal, `secondary-container`): **„Jednáte za: [jméno dítěte]“** — rodičovská práva jsou per dítě a UI to připomíná. U vlastní přihlášky banner není.
- Navigace zpět na seznam a **přepínač mezi přihláškami téhož účtu** (šipky ‹ › v hlavičce, pořadí dle D-01).
- Žádná výzva k založení účtu — účet už existuje.

**Stavy:** shodné s rozcestníkem (všech devět stavů vč. pádu zpět po zamítnutém dokumentu) — neduplikovat, jedna specifikace, dva vstupy. Navíc: **cizí přihláška v URL** (pid mimo účet a vazby) → chybový vzor § 5 s textem „Tato přihláška nepatří k vašemu účtu.“ bez dalších detailů.

**Interakce a validace:** shodné s rozcestníkem (mutace → `evaluate()` → snackbar). Platba i nahrání dokumentu fungují stejně jako z tokenu — je to táž obrazovka.

**Oprávnění:** vlastní přihláška, nebo přihláška dítěte z aktivní vazby.

**Mobil/desktop:** dle rozcestníku; sticky CTA na mobilu zachováno, navigation bar pod ním.

**Notifikace:** dle akcí ([notifications.md](notifications.md)).

## 4. D-03 · Moje děti — `/muj-ucet/deti`

**Účel a publikum:** rozcestník po dětech — kdo má co rozdělaného, koho se týká pozvánka druhého zástupce či zletilost.

**Layout a komponenty:** karty dětí (filled) pod sebou; na kartě avatar s iniciálami, jméno, věk, počet přihlášek, řádek druhého zástupce, stavové chipy.

**Obsah a pole:** u dítěte počet přihlášek a kolik čeká na krok rodiče; stav druhého zástupce (navázán / chip „Pozvánka odeslána: [e-mail]“ s platností). Zletilé dítě: chip „Jen pro čtení — zletilá“ + podtext „Zastoupení se po 18. narozeninách přepnulo do režimu jen pro čtení.“ ([parent-child-lifecycle.md](parent-child-lifecycle.md)). Pod kartami vysvětlivka: „Vazba na dítě vzniká přihlášením na akci nebo schválením jeho přihlášky e-mailem.“

**Stavy:**

- **Prázdný (účet bez vazeb):** ikona `family_restroom`, „Zatím tu nejsou žádné děti“, „Vazba na dítě vzniká přihlášením dítěte na akci nebo schválením jeho přihlášky — ne ručním přidáním.“, CTA „Prohlédnout akce“ → `/`. Žádné tlačítko „přidat dítě“ neexistuje.
- **Načítání:** skeleton karet; **Chyba:** § 5.

**Interakce a validace:** klepnutí na kartu → D-04. Pozvat dalšího zástupce a vystoupit z vazby se dělá až v detailu — destruktivní akce mimo dosah palce na přehledu.

**Oprávnění:** děti z aktivní vazby zástupce (`readonly_after_adulthood` = jen ke čtení).

**Mobil/desktop:** karty v jednom sloupci vždy.

**Notifikace:** žádné z přehledu.

## 5. D-04 · Detail dítěte — `/muj-ucet/deti/:id`

**Účel a publikum:** spravovat údaje a přihlášky jednoho dítěte; obsloužit pozvánku druhého zástupce a hranu zletilosti bez zmatku.

**Layout a komponenty:** svislé karty-sekce: **Údaje** (formulář) · **Zástupci** (M3 list) · **Přihlášky** (karty jako D-01) · dole zóna destruktivní akce.

**Obsah a pole:**

1. **Údaje:** jméno*, příjmení*, přezdívka, pohlaví, **datum narození — jen ke čtení** s poznámkou „Změnu data narození vyřídí vedoucí oddílu — má dopad na schvalování zástupcem a věková pravidla závodů.“, kontaktní e-mail, adresa trvalého bydliště, zdravotní pojišťovna. Oddílovou evidenci (chytré sloupce) rodič nevidí.
2. **Zástupci:** seznam navázaných zástupců + řádek běžící pozvánky (chip „Odeslána [datum] · platí do [datum]“) s akcemi „Poslat znovu“ a „Zrušit pozvánku“ (dialog; token přestane platit). Tlačítko „Pozvat dalšího zástupce“ → dialog: e-mail\*, jméno, text „Vazba vznikne až přijetím pozvánky. Oba zástupci pak mají plná práva; platí poslední zápis.“ Odeslání založí pozvánku (platnost 14 dní), snackbar.
3. **Přihlášky:** karty přihlášek dítěte (obsah jako D-01), detail → D-02. Tlačítko „Přihlásit na akci“ → portál `/` (formulář pak předvyplní údaje dítěte).
4. **Vystoupení:** outlined error button „Zrušit moje zastoupení“ → dialog s dopadem: „Přestanete spravovat údaje a přihlášky [dítě]. Zůstane-li bez navázaného zástupce, spravuje ho vedoucí oddílu, dokud se nepřipojí nový. Akce se zaznamená.“ + povinné zaškrtnutí „Rozumím důsledkům“.

**Varianta „jen pro čtení“ (zletilé dítě):** banner `tertiary` „[Jméno] je zletilá — zastoupení je jen pro čtení. Přístup vám může kdykoli zcela zrušit.“ Všechna pole disabled; **jediné aktivní pole je chybějící kontaktní e-mail** s tlačítkem „Uložit a poslat výzvu k převzetí účtu“ → snackbar, pole se zamkne ([parent-child-lifecycle.md](parent-child-lifecycle.md) → readonly_after_adulthood). Vystoupení zůstává dostupné.

**Stavy:**

- **Prázdný — přihlášky dítěte:** ikona `assignment`, „Žádné přihlášky“, „[Jméno] zatím nemá žádnou přihlášku na akci.“, CTA „Přihlásit na akci“ (jen u aktivní vazby; u readonly bez CTA).
- **Úspěch:** uložení údajů → snackbar „Údaje uloženy“; vystoupení → snackbar, dítě zmizí z D-03 i jeho přihlášky z D-01.
- **Načítání:** skeleton formuláře; **Chyba:** § 5; formulářové chyby inline.

**Interakce a validace:** křestní jméno proti whitelistu — neznámé jméno neblokuje, jen informační hláška „Jméno není v seznamu českých jmen — výjimku potvrdí vedoucí oddílu.“ ([validation.md](validation.md)); e-mail formát; adresa s autofill atributy. Každá mutace → snackbar.

**Oprávnění:** aktivní vazba = zápis; zletilé dítě = jen čtení + doplnění chybějícího kontaktu.

**Mobil/desktop:** formulář s viditelnými popisky a správnými klávesnicemi; na mobilu dialogy jako bottom sheet.

**Notifikace:** pozvánka zástupce → `EMAIL_SECOND_GUARDIAN_INVITE`; výzva k převzetí účtu → `EMAIL_ACCOUNT_CLAIM_INVITE` ([notifications.md](notifications.md)).

## 6. D-05 · Pozvánka druhého zástupce — `/pozvanka/:token` (veřejná stránka)

**Účel a publikum:** druhý rodič — často bez účtu, na telefonu — pochopí, co přijímá, a přijme na jeden krok. Stejná „studená“ situace jako schvalující zástupce ([ux-obrazovky-verejny.md](ux-obrazovky-verejny.md) § 6).

**Layout a komponenty:** jednosloupcová tokenová stránka bez navigace shellu: karta s kontextem, karta s vysvětlením práv, formulář, sticky CTA „Přijmout pozvánku“.

**Obsah a pole:** kontext „[Zvoucí] vás zve jako zákonného zástupce dítěte [dítě] ([věk]), [oddíl].“ Vysvětlení: „Přijetím vznikne vazba zástupce — budete spravovat údaje a přihlášky dítěte stejně jako zvoucí rodič. Platí poslední zápis.“ Formulář: jméno a příjmení* (předvyplněno z pozvánky), e-mail (jen ke čtení), checkbox* „Jsem zákonný zástupce tohoto dítěte“. Zobrazená platnost tokenu.

**Stavy:**

- **Platná pozvánka:** výchozí obsah.
- **Úspěch:** ikona `check_circle`, „Přijato — děkujeme“, „Vazba zástupce byla vytvořena. Údaje a přihlášky dítěte teď spravujete i vy.“; chip na D-03/D-04 se změní na „navázán“.
- **Již přijato:** idempotentně a přátelsky — „Tuto pozvánku jste už přijali. Vše je nastaveno.“ (žádná chyba).
- **Vypršelá / zrušená:** ikona `schedule`, „Pozvánka už neplatí“, „Platnost vypršela, nebo byla pozvánka zrušena. Požádejte zvoucího rodiče o novou.“ Bez CTA.
- **Neplatný token:** chybový vzor § 5 bez detailů.

**Interakce a validace:** checkbox povinný; jméno povinné. Přijetí je jediná mutace.

**Oprávnění:** veřejná tokenová stránka (bez přihlášení).

**Mobil/desktop:** mobile-first; na desktopu úzký sloupec na střed.

**Notifikace:** potvrzení přijetí dle [notifications.md](notifications.md).

## 7. D-06 · Moje údaje a souhlasy — `/muj-ucet/udaje`

**Účel a publikum:** jedno místo pro údaje vlastní osoby a přehled souhlasů se zpracováním — navštěvované zřídka, typicky po změně bydliště či pojišťovny.

**Layout a komponenty:** dvě karty-sekce: **Moje údaje** (formulář) a **Souhlasy** (M3 list).

**Obsah a pole:**

1. **Moje údaje:** jméno*, příjmení*, přezdívka, tituly před/za, pohlaví, **datum narození — jen ke čtení** (stejná poznámka jako D-04), kontaktní e-mail, adresa, zdravotní pojišťovna. Drobný text „Vyplňuje se jen to, co akce potřebují.“
2. **Souhlasy:** řádek = typ, účel, uděleno kdy, stav (chip „Platí“ / „Odvolán“ + podtext „Záznam se uchovává ještě 4 roky po odvolání.“). U platného souhlasu akce „Odvolat“ → dialog s dopadem a potvrzením. Pod seznamem text „O výmaz údajů (GDPR) požádejte vedoucího oddílu.“

**Stavy:**

- **Prázdný — souhlasy:** ikona `fact_check`, „Žádné souhlasy“, „Souhlasy se objeví po udělení — například při přihlášce na akci.“ Bez CTA.
- **Úspěch:** snackbar „Údaje uloženy“; **Chyba:** inline u pole; vzor § 5 pro celou stránku.

**Interakce a validace:** stejná pravidla jmen jako D-04; e-mail formát; autofill adresy. Uložení nemění nic jiného (žádné přepočty přihlášek).

**Oprávnění:** vlastní osoba (self).

**Mobil/desktop:** na desktopu stále jeden sloupec.

**Notifikace:** odvolání souhlasu se zaznamenává; e-mail dle [notifications.md](notifications.md).

## 8. D-07 · Účet a zabezpečení — `/muj-ucet/ucet`

**Účel a publikum:** přihlašovací metody pod kontrolou uživatele; nic tu nesmí umožnit zamknout sám sebe ven.

**Layout a komponenty:** karty-sekce: **Přihlášení** · **Propojené účty** · **Sloučení duplicit** (vstup do D-08).

**Obsah a pole:**

1. **Přihlášení:** přihlašovací e-mail — jen ke čtení, podtext „Změnu e-mailu vyřídí podpora.“ Tlačítko „Změnit heslo“ → dialog: staré heslo*, nové heslo* 2× (min. 8 znaků). Úspěch: snackbar „Heslo změněno“.
2. **Propojené účty:** řádek za každou OAuth metodu (propojen / nepropojen) s akcí „Odpojit“ / „Propojit“. **Pravidlo poslední cesty:** odpojit lze, jen zbývá-li jiný způsob přihlášení — u poslední metody je tlačítko zamčené (disabled + ikona `lock`) s vysvětlením „Jediný způsob přihlášení nelze odpojit — nejdřív nastavte heslo nebo připojte jiný účet.“ Poznámka: „Propojení přes e-mail se nabízí až po přihlášení heslem — automatické propojení by umožnilo převzetí účtu.“
3. **Sloučení duplicit:** karta s obsahem dle stavu — při nalezeném kandidátovi nabídka „Nejste to vy? V [oddíl] je evidována [jméno] se shodným datem narození.“ + tlačítko „Zobrazit návrh“ → D-08. Po odmítnutí/dokončení prázdný stav níže.

**Stavy:**

- **Prázdný — sloučení duplicit:** ikona `person_search`, „Žádný návrh na sloučení“, „Nenašli jsme jiný záznam, který by mohl být váš. Kdyby se objevil, uvidíte ho tady — nic se nesloučí bez vašeho souhlasu.“ Bez CTA.
- **Úspěch:** snackbary u každé mutace; **Chyba:** vzor § 5; formulář hesla inline (neshoda hesel, málo znaků).

**Interakce a validace:** viz výše; ochrana poslední přihlašovací cesty je tvrdý guard v UI i na serveru.

**Oprávnění:** vlastník účtu.

**Mobil/desktop:** podpora správců hesel (autocomplete atributy); přepínače a tlačítka se 44px cíli.

**Notifikace:** změna hesla → `EMAIL_ACCOUNT_LOCKED`/potvrzení dle [notifications.md](notifications.md).

## 9. D-08 · Žádost o sloučení duplicit — `/muj-ucet/ucet/slouceni`

**Účel a publikum:** delikátní moment souhlasu — srozumitelně nabídnout „tohle jste možná vy v jiném oddíle“, provést žádostí a volbou polí, a nic neslíbit dřív, než souhlasí všechny strany. Kandidáti se **jen navrhují, nikdy neslučují automaticky** ([person-merge.md](person-merge.md)).

**Layout a komponenty:** krokový obsah řízený stavem žádosti: (1) nabídka kandidáta — dvě porovnávací karty, (2) stav žádosti — M3 list stran se stavovými ikonami, (3) volba polí A/B — pole po poli dvě porovnatelné karty, (4) shrnutí a dokončení.

**Obsah a pole:**

- **Nabídka:** karta „Váš záznam“ vedle karty „Nalezený záznam“ s důvodem shody („shodné jméno a datum narození“). Dopady: „Sloučením se přenesou všechny vazby (přihlášky, docházka, členství). Citlivá data zůstávají per oddíl — vedoucí druhého oddílu nezíská přístup k vašim datům.“ Akce: „Požádat o sloučení“ a „Toto nejsem já“ — před potvrzením odmítnutí dialog „Dvojice se trvale potlačí a návrh se už nezobrazí. Povolit ji může jen administrátor ústředí.“
- **Po podání — čeká na strany:** seznam stran s jejich rozhodnutím (u kandidáta bez účtu rozhoduje HVO jeho oddílu); text „Bez odezvy žádost propadá po 30 dnech.“
- **Připraveno — volba polí A/B:** konfliktní pole po jednom; u každého dvě karty, výběr právě jedné, **žádný ruční přepis** (kvůli věrnému revertu — oprava až po sloučení běžnou editací). Prázdné pole prohrává automaticky. Různá data narození vyvolají varování.
- **Dokončení:** shrnutí → „Dokončit sloučení“ → zvolené hodnoty se zapíší, zdrojová osoba zanikne (náhrobek s přesměrováním). Text „Sloučení může vrátit jen administrátor ústředí.“

**Stavy:**

- **Čeká na strany:** ukázat, na koho se čeká. **Zamítnuto:** kdokoli zamítl nebo 30 dní bez odezvy — klidné sdělení, dvojice potlačena, bez CTA. **Dokončeno / Vráceno:** jen ke čtení.
- **Zablokováno kolizí:** banner `error-container` „Oba záznamy mají aktivní přihlášku na téže akci. Nejdřív to vyřeší vedoucí akce — zatím se nic nemění.“
- **Prázdný (bez kandidáta):** shodný s prázdným stavem sekce v D-07.
- **Načítání:** skeleton karet; **Chyba:** vzor § 5.

**Interakce a validace:** dokončit lze jen ze stavu „Připraveno“ a jen když jsou všechny konflikty rozhodnuté; celé sloučení je jedna transakce (selže-li část, nezmění se nic). Odmítnutí i dokončení přepnou sekci v D-07 do prázdného stavu.

**Oprávnění:** vlastník účtu iniciuje; provedení dle pravidel [person-merge.md](person-merge.md).

**Mobil/desktop:** na úzkém displeji jedno konfliktní pole na obrazovku s postupem (krok X z N); na desktopu pole pod sebou.

**Notifikace:** žádost, připomínky a potvrzení → `EMAIL_MERGE_REQUEST*` ([notifications.md](notifications.md)).
