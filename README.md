# Registrační systém oddílů DU — specifikace

## Přehled projektu

Systém registrací na akce pro oddíly DU. Struktura: Organizace → oddíly. Organizace sdružuje oddíly, spravuje členskou databázi a může pořádat vlastní akce. Každý oddíl spravuje vlastní akce, registrace a účastníky. Člen je nezávislá entita — může patřit do více oddílů v rámci organizace současně.

**Rozsah:** Veřejný registrační portál, oddílová správa akcí, organizační správa, self-management pro registrované.

---

## Přehled architektury

```
┌─────────────────────────────────────────────────┐
│                   Organizace                    │
│  (oddíly, členové DU, vedoucí, přesuny,         │
│  akce, platby, reporting)                       │
└──────────────────────┬──────────────────────────┘
                       │
       ┌───────────────┼────────────────┐
       │               │                │
  ┌────▼─────────┐ ┌───▼─────────┐ ┌────▼───┐
  │oddíl A       │ │oddíl B      │ │oddíl C │
  │akce, platby, │ │...          │ │...     │
  │chytré sloupce│ │             │ │        │
  |členové, hosté│ │             │ │        │
  └───┬──────────┘ └───┬─────────┘ └────┬───┘
      │                │                │
  ┌───▼────────────────▼────────────────▼───┐
  │       Veřejný registrační portál        │
  │  (procházet akce, registrovat se)       │
  └─────────────────────────────────────────┘

┌────────────────────────────────────────────┐
│          Osoba (nezávislá entita)          │
│  - Může být evidován ve více oddílech      │
│  - Může být člen, host, rodič, dobrovolník │
└────────────────────────────────────────────┘

```

---

## Požadavky

### Role

- Uživatel může být ve více rolích, např. admin a zároveň jeden z vedoucích oddílu nebo dobrovolník a rodič
- Role Hlavní vedoucí oddílu, Rádce, Vedoucí oddílu, Vedoucí družiny, Admin (organizace) / Účetní (organizace), Koordinátor (konkrétní akce v organizaci nebo oddíle)
- Role Rodič (zákonný zástupce) — zastupuje jedno či více nezletilých dětí; přihlašuje je na akce a sám se může akcí účastnit, dítě může mít více rodinných zástupců

### Matice oprávnění

Legenda: **C** = plná správa (CRUD + konfigurace), **W** = správa záznamů (bez konfigurace), **R** = jen čtení, **—** = žádný přístup. Rozsah upřesňují poznámky pod tabulkou.

Zkratky rolí: **ORG-A** = Admin (organizace), **ORG-Ú** = Účetní (organizace), **HVO** = Hlavní vedoucí oddílu, **VO** = Vedoucí oddílu, **VD** = Vedoucí družiny, **RÁD** = Rádce, **KOO** = Koordinátor, **ROD** = Rodič.

| Zdroj / akce                             | ORG-A | ORG-Ú | HVO | VO  | VD  | RÁD | KOO | ROD |
| ---------------------------------------- | ----- | ----- | --- | --- | --- | --- | --- | --- |
| Oddíly (CRUD)                            | C     | R     | R¹  | —   | —   | —   | —   | —   |
| Uživatelé a role oddílu                  | C     | —     | C¹  | —   | —   | —   | —   | —   |
| Členové oddílu (evidence)                | W²    | R²    | C¹  | W¹  | R³  | R³  | —   | W¹⁰ |
| Citlivá data (zdravotní ap.)             | —⁴    | —⁴    | R⁵  | R⁵  | R⁴  | R⁴  | —⁴  | W¹⁰ |
| Přesun člena mezi oddíly                 | C     | —     | W⁶  | —   | —   | —   | —   | —   |
| Družiny                                  | R     | —     | C¹  | C¹  | W⁷  | R⁷  | —   | —   |
| Vlastní certifikáty / potvrzení          | R     | —     | W⁸  | W⁸  | W⁸  | W⁸  | —   | —   |
| Akce (vytvoření/konfigurace)             | C     | R     | C¹  | W¹  | —   | —   | —   | —   |
| Registrace / přihlášky na akci           | W     | R     | W¹  | W¹  | —   | —   | W⁹  | W¹⁰ |
| Bankovní účty + tokeny                   | C     | R     | C¹  | —   | —   | —   | —   | —   |
| Platby (potvrzení, párování, výzvy)      | W     | W     | W¹  | R¹  | —   | —   | R⁹  | R¹⁰ |
| Chytré sloupce (definice + viditelnost)  | R     | —     | C¹  | R¹  | R¹  | R¹  | —   | —   |
| Komunikační e-mail / token oddílu        | C     | —     | C¹  | —   | —   | —   | —   | —   |
| Reporting (období, docházka, statistiky) | C     | R     | R¹  | R¹  | R⁷  | R⁷  | R⁹  | —   |
| Dobrovolníci + docházka hodin            | R     | —     | W¹  | W¹  | W⁷  | R⁷  | —   | —   |
| Hosté (evidence, povýšení na člena)      | W     | R     | W¹  | W¹  | —   | —   | W⁹  | —   |

Poznámky k rozsahu (scope):

1. Jen vlastní oddíl — HVO/VO, sloupce, akce a platby působí pouze v rámci svého oddílu.
2. ORG-A/ORG-Ú napříč organizací, ale bez citlivých dat (viz pozn. 4).
3. VD/RÁD vidí členy jen ve své družině.
4. Citlivá data jsou vedená **odděleně pro každý oddíl** (každý oddíl si drží vlastní záznam, např. zdravotní kartu) — nevidí je organizace ani jiný oddíl (zdravotní omezení, léky, alergie, církev).
5. Vidí je jen vedoucí oddílu, ve kterém jsou data pořízena; je-li člen ve více oddílech, každý oddíl vidí pouze svoji verzi citlivých dat.
6. HVO může člena přesunout ze svého oddílu; cílový oddíl přesun potvrdí (jinak provádí jen ORG-A). — k rozhodnutí.
7. Jen vlastní družina (VD spravuje, RÁD čte).
8. Každý vedoucí/rádce edituje jen své vlastní certifikáty a potvrzení.
9. Jen přidělená akce (přes assignment); bez přístupu ke konfiguraci akce (ceny/storno) a k jiným akcím.
10. Rodič jen u vlastních zastupovaných dětí (a sebe jako účastníka). Je editorem jejich základních i citlivých dat (zdravotní apod.) a jejich přihlášek; u plateb vidí stav a dostanává výzvy, nepáruje je. Bez přístupu k oddílovým zdrojům a k cizím osobám.

#### Osoba vs. uživatelský účet

- Oddělujeme dvě entity:
  - **Osoba** = datový subjekt / účastník; může existovat bez přihlášení (host, nezletilé dítě spravované rodičem)
  - **Účet (uživatel)** = přihlašovací identita (heslo / OAuth), navázaná právě na jednu osobu
- Jedna osoba má nejvýše jeden účet; jeden účet může mít více propojených OAuth identit (Google, Facebook)
- Host nemá účet — má pouze identifikátor (token), kterým si může účet založit; po založení se účet propojí s existující osobou (nevznikne duplicita)

#### Stav osoby (lifecycle)

- Host / registrovaný člen / člen DU je **stav jedné osoby**, nikoli samostatná entita:
  - `host → registrovaný člen` (migrace provedená hlavním vedoucím)
  - `registrovaný člen → člen DU` (po zaplacení příspěvku do listopadu; platí leden–prosinec)
  - `člen DU → registrovaný člen` (automatický přechod koncem roku, pokud nebyl zaplacen příspěvek na další rok — členství DU vyprší 31. 12.)
  - `* → neaktivní` (osoba opustila oddíl nebo dlouhodobě bez aktivity; záznam zůstává kvůli historii, ale nezapočítává se do počtu členů a nedostává automatické výzvy)
  - `neaktivní → registrovaný člen / host` (reaktivace, pokud se osoba vrátí)
  - `* → archivovaný` (GDPR: po uplynutí retenční doby se osobní a citlivá data anonymizují; zachovají se jen agregované/nepřímo identifikující údaje nutné pro reporting)
- Stavy `neaktivní` a `archivovaný` jsou kolmé na členský stav výše — určují, zda je záznam živý, uspaný, nebo anonymizovaný.
- **Retence a GDPR:** citlivá data (zdravotní apod.) se mažou dříve než základní evidence; konkrétní lhůty retence jsou ke stanovení (TODO). Citlivá data jsou izolovaná per oddíl, každý oddíl proto maže/anonymizuje jen svoji verzi; ORG-A může spustit výmaz napříč všemi oddíly.
- Role **rodič** (a např. dobrovolník) je kolmá na tento stav — osoba může být současně rodič i host/člen

### Hosté (= nečleni, veřejnost)

- Oddíl/organizace eviduje hosty (min jméno příjmení nebo přezdívka)

### Registrovaní členové

- Hlavní vedoucí může zmigrovat data Hosta do Registrovaného člena
- Registrovaný člen má povinné datum narození
- U každého je evidována historie - změny, registrace, pod jakým oddílem

## Deduplikace osob, merge

- Na úrovni organizace systém zobrazí možné kandidáty (jméno, příjmení, datum narození). Systém nabídne "Reportovací sloučení" osob pro účely unikátních počtů, záznamy zůstanou oddělené. Pro účely reportů se neřeší hosty.
- Osobě s účtem se zobrazí možný kandidát na propojení (z jiného oddílu/organizace). Účet zadá Žádost o sloučení. Systém rozešle emailem žádost - iniciátorovi, hlavní vedoucí druhého klubu a případně i účtu kandidáta na propojení. Po odsouhlasení všemi stranami (vedoucímu se zobrazí pro porovnání náhled obou osob) může uživatel pokračovat se spojením: Záznamy obou osob se spojí do jedné osoby, konflikt základních polí se řeší volbou A/B, účet se naváže na sjednocenou osobu, pokud obě osoby mají účet, pak druhý účet se zruší (uživatel vybere), citlivá data zůstávají per oddíl, OAuth identity se přenesou pod ponechaný účet.
- Podobně se zpracuje duplicitní dítě, které se zobrazí rodiči s tím, že další strana je rodič dítěte kandidáta a výsledek nespojí účty rodičů do jednoho, jen osobu dítěte. Pokud dítě nemá navázaného rodiče, schválí merge vedoucí oddílu, kde je dítě evidováno.
- Systém loguje, kdo kdy které osoby spojil, je možné zrušit merge pro nápravu chybného spojení.

### Člen DU

- Člen se může stát členem DU od ledna následujícího roku po zaplacení příspěvku do listopadu
- Členství DU trvá: leden–prosinec (kalendářní rok)

### Rodič (zákonný zástupce)

- Rodič je osoba, která má vazbu na alespoň jedno dítě (typicky nezletilé)
- Rodič může zastupovat jedno nebo více nezletilých dětí (vazba rodič ↔ dítě, typu 1:N)
- Jedno dítě může být svázáno s více rodiči (oba zákonní zástupci) — vazba je M:N
- Rodič může své zastupované děti přihlašovat na akce a spravovat jejich přihlášky (registrace, storno, platby za dítě)
- Rodič se sám může akcí účastnit jako účastník (vystupuje pak zároveň jako účastník i jako zástupce dětí)
- Rodič vidí a edituje pouze údaje a přihlášky vlastních dětí
- Vazba rodič ↔ dítě vzniká registraci dítěte rodičem
- Po dosažení zletilosti se zastoupení rodičem přepne do režimu jen pro čtení. Výjimkou je doplnění kontaktního e-mailu dítěte, pokud chybí — slouží k doručení výzvy k převzetí účtu. Zletilý člen může přístup rodiče kdykoli zcela zrušit.
- Vazbu může zrušit sám rodič (vystoupení), případně vedoucí oddílu na žádost; zrušení se loguje. Zůstane-li nezletilé dítě bez navázaného rodiče, jeho údaje a přihlášky spravuje vedoucí oddílu, dokud se nepřipojí nový zákonný zástupce.
- Oba rodiče mají plná práva, platí poslední zápis.
- Druhého zákonného zástupce přidává stávající rodič nebo hlavní vedoucí pozvánkou (e-mailem). Vazba vznikne přijetím pozvánky druhým rodičem. Nemá-li dítě žádného navázaného rodiče, schvaluje připojení vedoucí oddílu, kde je dítě evidováno.

### Organizace

- Administrátor spravuje oddíly a přiřazuje jim jejich Hlavní vedoucí
- Vidí seznam členů všech oddílů (kromě jejich citlivých dat a hostů)
- Hosté se nepočítají do počtu členů oddílu

### Oddíl

- Hlavní vedoucí můžou do systému nahrát pověření od staršovstva
- Typy oddílů: IČO ústředí, Pobočný spolek (vlastní IČO), kolektivní člen (bez DU v názvu, vlastní IČO)
- Registrace - Je možné vytvořit formulář (na úrovni oddílu) určený k registraci s možností zvolit políčka k vyplnění o členovi (lze vybírat z chytré tabulky)
- Vedoucí může definovat družiny a do nich přiřadit členy
- Admin a Hlavní vedoucí oddílu můžou do systému zadat bankovní účty
- Rádci nevidí citlivá data oddílových dětí, nejsou plnoletí

### Družina

- Má své Vedoucí, Rádce a členy

### Dobrovolníci

- Typy: krátkodobí a dlouhodobí

### Přihlašování na oddílové/celoČR akce

- Hlavní vedoucí i Admin můžou vytvářet Přihlášky na akce
- Každá akce je svazána s maximálně jedním účtem
- systém posílá výzvu k zaplacení (QR kód + platební údaje)
- systém připomíná nezaplacené platby
- Akce může být veřejná nebo neveřejná (dostupná přes odkaz)
- Při registraci na akci je hostům vygenerován identifikátor, díky kterému je možné si založit účet

### Generování reportu na konci období (roku)

- seznam akcí/výprav, docházka členů/nečlenů/vedoucích/rádců
- Unikátni počet dětí v rámci všech akcí (počítá se jednou, ikdyž bylo na více akcích)
- Trendy - TODO

### Docházka

- Vedoucí můžou vytvářet události - např. páteční kluby
- Vedoucí můžou vybirat účastníky ze seznamu osob z oddílu
- Při evidenci dobrovolníku je možné zadat počet hodin

### Platební modul

- ke každému bankovnímu účtu je možné doplnit API token
- systém automaticky páruje bankovní transakce s přihláškami podle SS=akce a VS=přihláška
- systém automaticky posílá potvrzení o platbě

### Přihlašování do systému

- Administrátoři vytvářejí účty hlavním vedoucím, hlavní vedoucí vytvářejí účty vedoucím a rádcům
- Každý uživatel si může v systému změnit heslo ()
- Každý si může vytvořit účet v systému a v něm editovat svojí identitu, kterou může použít při dalších registracích na akce
- Neregistrovaným uživatelům je umožněno přes token spravovat jejich registrace (storno, měnit nebo přidávat další účastníky)
- Pro přihlášení do aplikace půjde použít účet Google nebo Facebook (OAuth)
- Jeden uživatel může mít více propojených identit (Google i Facebook).

### Konfigurace akce

- Vedoucí/administrátoři můžou vytvářet akce
- název, SS, max kapacita, počet náhradníků, ceny pro členy DU i ostatní, začátek akce, začátek a konec přihlašování, termíny pro storno podmínky
- pokud akce vyžaduje dobrovolníky, lze zadat cenu, začátek a konec přihlašování, system nabídne samostatnou registraci pro dobrovolníky

### Typy akcí

- Vzdělávací - vyžaduje tituly, adresu trvalého bydliště
- S doporučením mentora, vedoucího - system osloví zadané vedoucí/mentory o doplnění očekávání, potvrzení
- Jednoosobové - obecná přihláška
- Klubové - lze přihlásit více účastníků včetně jejich zákonných zástupců

### Vzdělávání

- Organizace nabízí některé akce, které si může vedoucí započítat do svého vzdělání, pokud je absolvuje
- Organizace udržuje tabulku se seznamem kurzů - vlastní nebo jiné s seznamem platnosti
- Hlavní vedoucí, Vedoucí a Rádci můžou sobě přiřadit kurzy z nabídky
- Systém automaticky přiřadí kurz organizace všem účastníkům po jeho absolvování
- Vedoucí a Rádci mají možnost vložit do systému svoje certifikáty, potvrzení od doktora a jiné kurzy
- Organizace vidí, jaké kurzy absolvovali jednotliví vedoucí v oddílech

### Pomocná evidence

- Vedoucí může pro svůj oddíl definovat nové sloupce (do tabulky hostů/členů)
- Sloupcům lze nastavit, zda je vlastník účtu (nebo Rádce) může vidět nebo upravovat
- Hodnoty z pomocné evidenci nejsou přístupné z úrovně organizace

---

## Technologický stack

| Vrstva   | Technologie                               |
| -------- | ----------------------------------------- |
| Backend  | Nette, PHP 8.2+                           |
| Databáze | Nette Database (Explorer), MySQL          |
| Frontend | Vite, React 18+, TypeScript, Tailwind CSS |
| API      | REST, JSON, autentizace přes session      |
| Stav     | React Query pro stav serveru              |

---

## Implementační detaily

### Komunikační modul

- systém ve výchozím stavu komunikuje emailem přes ověřený SMTP účet dorostove unie
- každý oddíl může volitelně definovat svůj email pro odchozí komunikaci nastavením SMTP tokenu
- šablony jsou ve složce emails/

### Ukládání tokenů

Tokeny jsou uloženy v databázi šifrovaně, klíč je uložen na serveru v neveřejné části.
https://github.com/defuse/php-encryption

### Pravidla přihlášení přes OAuth (bezpečné párování)

1. **Existující propojení:** Pokud (`provider`, `provider_user_id`) už existuje
   v `user_oauth_identities`, přihlas příslušného uživatele. (E-mail se
   nepoužívá k vyhledání účtu.)
2. **Nové propojení — jen ověřený e-mail:** Propojení s existujícím účtem podle
   e-mailu je povoleno **pouze** když provider vrátí `email_verified = true`.
   Neověřený e-mail (typicky Facebook) nikdy automaticky nepropojí ani nevytvoří
   účet.
3. **Potvrzení vlastnictví účtu:** I při ověřeném e-mailu se účet automaticky
   nepřebírá. Před propojením musí uživatel prokázat vlastnictví existujícího
   účtu jedním z:
   - přihlášením stávajícím heslem, nebo
   - kliknutím na potvrzovací odkaz odeslaný na e-mail účtu (platnost např. 30 min,
     jednorázový token).
4. **Žádný účet v systému:** Pokud e-mail v systému neexistuje a je ověřený,
   lze založit nový účet a rovnou vytvořit OAuth identitu. Při neověřeném e-mailu
   založení odmítni a vyžádej standardní registraci.
5. **Odpojení:** Uživatel může OAuth identitu odpojit; nesmí ale zůstat bez
   možnosti přihlášení (musí mít heslo nebo alespoň jednu identitu).

---
