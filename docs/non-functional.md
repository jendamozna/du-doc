# Nefunkční požadavky

Implementační detail k [README.md](../README.md) → **Požadavky**. Schéma viz [data-model.md](data-model.md).

## Architektura

Systém je implementován jako modulární monolit.

- jedna deployovatelná aplikace
- jedna databáze
- moduly sdílejí proces i databázi
- moduly nejsou nasazovány samostatně
- změny se oznamují událostmi, data se čtou přes rozhraní vlastníka

## Technologický stack

| Vrstva    | Volba                                               |
| --------- | --------------------------------------------------- |
| Frontend  | TypeScript, React, Vite, TailwindCSS                |
| Backend   | PHP 8.4, Nette 4                                    |
| Databáze  | MariaDB 10                                          |
| API       | REST, JSON                                          |
| Testování | PHPUnit (jednotkové a integrační), Playwright (E2E) |
| Vývoj     | Docker                                              |
| Nasazení  | GitHub, phinx                                       |

Co z toho plyne pro zbytek specifikace:

- **Oddělený frontend a backend.** Nette neservíruje HTML aplikace — vystavuje **REST JSON API**, React SPA je samostatný artefakt sestavený Vite. Šablony (Latte) zůstávají jen pro **odchozí e-maily** a PDF potvrzení. Autorizace API podle [authorization.md](authorization.md) tedy běží výhradně na serveru; skrytí prvku v UI není ochrana.
- **Mobile-first jako build target** — viz **Rozhraní a zařízení**; Tailwind breakpointy se používají vzestupně (základ = telefon).
- **MariaDB 10, `utf8mb4`** s českou kolací (`utf8mb4_czech_ci`) — třídění jmen musí respektovat české znaky a diakritiku.
- **Částky jsou `DECIMAL(10,2)`**, nikdy `FLOAT` — součty alokací a stav úhrady se porovnávají na haléř ([payment-matching.md](payment-matching.md)).
- **Časy jako `DATETIME` v UTC**, převod do `Europe/Prague` až při zobrazení (viz **Lokalizace a formáty**).
- **Binární obsah souborů** je `LONGBLOB` (viz **Úložiště souborů**) — `max_allowed_packet` musí pokrýt limit 10 MB na soubor s rezervou na šifrovací režii.
- **Plánované úlohy** (viz níže) běží jako Nette CLI příkazy spouštěné cronem, ne jako HTTP endpointy — jinak by šly vyvolat zvenku.
- **Fronta e-mailů a transakční outbox** jsou tabulky v databázi zpracovávané workerem; samostatný broker se pro daný rozsah nezavádí ([modules.md](modules.md) → **Pravidla pro doručování**).
- **Tajemství** (šifrovací klíč, OAuth `client_secret`, přístup k DB) se předávají **proměnnými prostředí kontejneru**, nikdy v obrazu ani v repozitáři (viz **Šifrování a hesla**).
- **Testy:** PHPUnit pokrývá stavové automaty a výpočty (`evaluate` přihlášky, pořadí párovacích pravidel, věková pravidla hlídek), Playwright flow priority P1 z UX průvodce včetně mobilního viewportu a tokenových odkazů bez přihlášení.
- **CI/CD na GitHubu** — stejný Docker obraz pro CI i produkci; migrace DB (phinx) jsou součástí nasazení a musí být zpětně kompatibilní (nejdřív přidat sloupec, pak přepnout kód).

## Lokalizace a formáty

- **Měna:** výhradně CZK. Všechny částky (ceny, storna, platby, alokace) jsou v korunách; zobrazují se s oddělovačem tisíců a symbolem, např. `1 250 Kč`, desetinná čárka.
- **Časové pásmo:** `Europe/Prague`. Časy se ukládají v UTC a zobrazují v místním čase včetně přechodu na letní/zimní čas; čistě datumové údaje bez času se pásmem nepřepočítávají.
- **Formát data a času:** česky, den v týdnu s velkým počátečním písmenem, den a měsíc bez úvodních nul — `Středa 29.7. 14:19`. **Rok se zobrazuje jen tehdy, liší-li se od aktuálního** (`Středa 29.7.2025 14:19`).
- **Jazyk:** čeština (rozhraní i e-maily).
- **DPH systém neřeší** — oddíly jsou neplátci, ceny akcí jsou konečné. Jediným dopadem DPH je delší retenční lhůta u dokladů, které ji obsahují.

## Přihlašování přes OAuth

- Podporovaní poskytovatelé: **Google a Facebook**. `client_id` a `client_secret` jsou v konfiguraci prostředí, **nikdy v databázi** ani v repozitáři; callback URL je registrovaná u poskytovatele a musí se shodovat přesně.
- Rozsah oprávnění je jen `email` + `profile` — systém od poskytovatele nic dalšího nepotřebuje.
- `OAUTH_IDENTITY` drží `provider` a `subject` (stabilní identifikátor od poskytovatele), **ne access ani refresh token**. Přihlašování je jednorázové ověření, ne trvalý přístup k účtu jinde.
- Identita se páruje na účet přes **ověřený** e-mail. Vrátí-li poskytovatel e-mail bez příznaku ověření, přihlášení se odmítne.
- Existuje-li už účet se stejným e-mailem a heslem, propojení se nabídne **až po úspěšném přihlášení heslem**. Automatické propojení by znamenalo převzetí účtu přes podvržený e-mail.
- Odpojit identitu lze jen tehdy, zbývá-li účtu jiný způsob přihlášení (heslo nebo druhá identita).
- Sloučení osob přenese OAuth identity pod ponechaný účet (viz README → **Deduplikace**).

## Úložiště souborů

Systém ukládá nahrané soubory ve třech entitách: **dokumenty přihlášek** (`REGISTRATION_DOCUMENT` — potvrzení od lékaře, bezinfekčnost, citlivá data), **trvalé dokumenty osoby** (`PERSON_DOCUMENT` — kartička pojišťovny, zdravotní způsobilost) a **skeny pověření** (`MANDATE`). Pravidla níže platí pro všechny tři stejně.

- **Obsah souboru se ukládá přímo v databázi** (`content`, `LONGBLOB`), ne v externím objektovém úložišti — při daném rozsahu (limit 10 MB/soubor, řádově stovky dokumentů na akci, viz **Rozsah a výkon**) to zjednodušuje zálohy, retenci i GDPR výmaz na jediné místo.
- Obsah je šifrovaný stejným mechanismem jako pole s příponou `_enc` (libsodium secretbox, vlastní nonce na záznam) — viz **Šifrování a hesla**.
- Žádný soubor není veřejně dostupný. Stažení jde výhradně přes aplikační endpoint, který ověří oprávnění na konkrétní přihlášku a obsah streamuje (`Content-Disposition`); trvalá ani sdílená URL na soubor neexistuje.
- Limit velikosti **10 MB** na soubor, whitelist typů PDF/JPG/PNG/HEIC ověřený podle **skutečného obsahu souboru, ne podle přípony**; ověřený typ se uloží do `mime_type` a původní název do `filename` pro `Content-Disposition` při stahování.
- Retenční a GDPR mazání smaže celý řádek (obsah i metadata) v jedné transakci — nehrozí osamocený soubor bez záznamu nebo záznam bez obsahu.
- Zálohy databáze podléhají stejným retenčním lhůtám jako zbytek dat; žádné oddělené úložiště se zvláštním režimem zálohování není potřeba.
- **Škálování:** při přechodu na víc aplikačních instancí nebo výrazně větší soubory lze později přejít na sdílený souborový systém nebo objektové úložiště (S3) beze změny API — DB sloupec s obsahem stačí nahradit klíčem na externí úložiště.

## Šifrování a hesla

- Pole s příponou `_enc` (`api_token_enc`, `smtp_password_enc`) jsou šifrovaná symetricky (libsodium secretbox) s **klíčem z konfigurace prostředí**, nikdy z databáze. Každý záznam má vlastní nonce.
- Šifruje se **jen to, co systém musí přečíst zpět** — přístupové údaje k bance a k odchozí poště a **obsah nahraných dokumentů** (viz **Úložiště souborů**). Nic jiného; běžná osobní data v databázi šifrovaná nejsou, chrání je oprávnění a retence.
- Uživatelská hesla se **hashují** (Argon2id), nešifrují. Reset hesla je jednorázový token s krátkou platností.
- Heslo má **12 až 128 znaků**; musí obsahovat alespoň 3 ze 4 skupin (velká písmena, malá písmena, číslice, speciální znaky), nesmí obsahovat přihlašovací e-mail ani běžné či známé prolomené heslo a může obsahovat mezery jako součást přístupové fráze.
- Šifrované ani hashované hodnoty se nikdy nevypisují do logů, chybových hlášek ani do exportů.
- Podpora **rotace klíče** — každý šifrovaný záznam nese `key_version`, aby šlo přešifrovat postupně bez výpadku. Pole je na `BANK_ACCOUNT`, `UNIT_MAIL_SETTING`, `REGISTRATION_DOCUMENT`, `PERSON_DOCUMENT` a `MANDATE` ([data-model.md](data-model.md)).

## Tokeny a ochrana přístupu

- Přihláška se dá spravovat bez účtu přes odkaz s tokenem (`REGISTRATION.token`) a stejným způsobem schvaluje zákonný zástupce. Token je náhodný (min. 128 bitů), vázaný na jednu přihlášku a **omezeně platný** — u schválení zástupcem lhůtou, u správy přihlášky koncem akce.
- Token opravňuje jen k operacím nad danou přihláškou; nikdy nezpřístupní seznam osob ani jiné akce.
- Přihlašování má **throttling** podle účtu i IP; po sérii neúspěchů dočasné zamknutí a e-mail vlastníkovi účtu.
- Ochrana proti výčtu účtů: chybová hláška u přihlášení i u obnovy hesla je vždy stejná bez ohledu na to, zda účet existuje.

### Znovuposlání ztraceného odkazu

Pro zástupce, náhradníky a hosty je e-mail celé UI ([notifications.md](notifications.md)) — ztracená zpráva je pro ně ztracený přístup. Musí proto existovat cesta „přišel jsem o odkaz".

- **Žádost odkaz neurčuje, jen ho odemyká.** Nový odkaz se pošle **výhradně na adresu, na kterou už token jednou odešel**; adresa z požadavku se s ní jen porovná a nikdy se nepoužije jako cíl. Odpověď je vždy stejná („pokud k adrese existuje čekající odkaz, poslali jsme ho") bez ohledu na výsledek — táž ochrana proti výčtu jako u obnovy hesla.
- **Znovuposlání token vymění a starý zneplatní.** Jedno pravidlo pro všechny typy: tyhle odkazy dávají přístup k osobním údajům dítěte a původní e-mail může být přeposlaný nebo ve sdílené schránce.
- **Lhůta se rotací neposouvá** — nový token dědí původní `expires_at`. Jinak by šlo resendem prodlužovat lhůtu schválení donekonečna a expirační job by nikdy nedoběhl; u 48hodinové nabídky náhradníkovi by se stejným způsobem dala držet obsazená kapacita libovolně dlouho. Je-li lhůta pryč, neodešle se nic a vrátí se táž neutrální hláška.
- **Dvě vstupní cesty:** veřejný formulář bez přihlášení (viz výše) a tlačítko „poslat odkaz znovu" na detailu přihlášky pro vedoucího s `can_edit_registrations` a pro HVO. Druhá cesta je v praxi častější a jako autentizovaná nemá enumerační problém; adresa se ani tam nezadává, bere se z přihlášky.
- **Throttling podle adresy i IP** a krátká prodleva mezi dvěma žádostmi (`resent_at`) — bez ní je z tlačítka nástroj na zahlcení cizí schránky.
- **Zapisuje se do `AUDIT_LOG`** — jde o vydání nového přístupu k osobním údajům. `actor_type = 'token'` s `actor_email` u samoobslužné cesty, `account` u vedoucího ([audit-log.md](audit-log.md)).
- **Neposílá se nová šablona** — odejde tentýž `EMAIL_*` jako poprvé (`EMAIL_GUARDIAN_REQUEST`, `EMAIL_REG_CONFIRM`, `EMAIL_SUBSTITUTE_OFFER` …). Katalog notifikací se tím nemění.
- **`UNIT_REGISTRATION.share_token` je z toho vyňatý.** Není osobní — vedoucí ho má v rozhraní a rozesílá sám, takže znovuposlání nedává smysl a rotace by rozbila odkazy už rozeslané rodičům.

## Odchozí e-maily

- Oddíl si může nastavit **vlastní SMTP** (`smtp_email`, `smtp_password_enc`); není-li nastavené, použije se systémové odesílání.
- E-maily se odesílají **z fronty**, ne synchronně v požadavku — selhání odeslání nesmí shodit registraci ani spárování platby.
- Opakování při chybě s exponenciálním odstupem a konečným počtem pokusů; trvale neodeslaný e-mail se zobrazí v rozhraní HVO oddílu, ne jen do logu, a odejde jako `EMAIL_UNDELIVERABLE_ALERT` ([notifications.md](notifications.md)) — nikdy však na adresu, která právě selhala.
- Odražené a odmítnuté adresy se označují u osoby, aby se na neplatnou adresu nezkoušelo posílat donekonečna. Adresa z `REGISTRATION.contact_email` k žádné osobě přiřadit nemusí — příznak proto musí umět sednout i na samotnou přihlášku.
- Každé odeslání se eviduje (událost, příjemce, čas) — bez toho nejde doložit, že výzva k platbě nebo žádost zástupci opravdu odešla.

## Plánované úlohy

| Úloha                            | Frekvence          | Poznámka                                                                                                                  |
| -------------------------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------- |
| import bankovních transakcí      | dle nastavení účtu | idempotentní podle `external_id` ([fio-sync.md](fio-sync.md))                                                             |
| párování nových transakcí        | po každém importu  | i po vzniku přihlášky ([payment-matching.md](payment-matching.md))                                                        |
| expirace schválení zástupcem     | denně              | stav `PendingGuardian` po lhůtě (7 dní) → `Expired` ([registration-lifecycle.md](registration-lifecycle.md))                                                                               |
| expirace schválení vazby zástupce | denně              | `PARENT_CHILD` v `pending` po lhůtě (14 dní) → `canceled` ([parent-child-lifecycle.md](parent-child-lifecycle.md)) |
| překlopení vazby po zletilosti   | denně              | `active` → `readonly_after_adulthood` v den 18. narozenin ([parent-child-lifecycle.md](parent-child-lifecycle.md)) |
| propadnutí nabídky náhradníkovi  | hodinově           | nabídka propadá, stav přihlášky se nemění                                                                                 |
| výzvy a připomínky splatnosti    | denně              | termín podle nastavení akce                                                                                               |
| vypršení nezaplacené přihlášky   | denně              | jen u akcí se zapnutým vypršením → `Expired` a uvolnění kapacity ([registration-lifecycle.md](registration-lifecycle.md)) |
| propadnutí žádosti o sloučení    | denně              | `MERGE_REQUEST.expires_at` po 30 dnech bez odezvy; týž běh rozesílá připomínku 7 dní předem ([person-merge.md](person-merge.md)) |
| připomínka závodníkům bez hlídky | denně              | N dní před akcí, přeskočí ty, kdo už dnes připomínku dostali ([race-patrols.md](race-patrols.md))                         |
| připomínka chybějících dokumentů | denně              | podle nastavení oddílu, dokud nejsou všechny schváleny ([notifications.md](notifications.md)) |
| odeslání naplánovaných pozvánek | hodinově           | `EVENT_INVITATION.scheduled_at` a `reminder_scheduled_at` ([notifications.md](notifications.md)) |
| upomínky členských předpisů     | denně              | po splatnosti, jen u účtu s bankovním API ([payment-matching.md](payment-matching.md)) |
| varování před deaktivací osoby  | denně              | 30 dní před automatickou deaktivací ([person-lifecycle.md](person-lifecycle.md)) |
| retenční mazání a anonymizace    | denně              | podle tabulky lhůt v README → **Retence a GDPR**                                                                          |
| vyřízení žádostí o výmaz (GDPR)  | denně              | s dokladem o výmazu                                                                                                       |
| čištění auditního logu           | měsíčně            | podle retence v [audit-log.md](audit-log.md)                                                                              |

- Všechny úlohy jsou **idempotentní** — opakovaný běh nesmí nic zdvojit ani smazat víc.
- Každá úloha má **zámek proti souběhu** a eviduje běh (začátek, konec, počet zpracovaných záznamů, chyba). Rozsah zámku je ten nejužší, který dává smysl — u bankovního importu je to **jeden bankovní účet**, ne celá úloha, aby se oddíl se dvěma účty nesynchronizoval zbytečně sériově ([fio-sync.md](fio-sync.md) → **Souběh a rate limit**).
- Časy se vyhodnocují v `Europe/Prague`, aby „denně" znamenalo místní den včetně přechodu času.
- Selhání úlohy se hlásí **tomu, kdo s ní může něco udělat**, ne paušálně administrátorovi: selhání bankovní synchronizace jde účetní a HVO oddílu, protože token opravuje oddíl ([fio-sync.md](fio-sync.md) → **Chybové stavy**), zatímco selhání systémových úloh (retence, čištění auditu) jde ADM. Tichý výpadek importu plateb by se jinak projevil až upomínkami zaplaceným lidem.

## Rozhraní a zařízení

- Návrh je **mobile-first**: rozhraní se kreslí nejdřív pro telefon a teprve pak rozšiřuje na tablet a desktop. Zákonní zástupci podají přihlášku a zaplatí z mobilu, vedoucí zapisuje docházku na schůzce a v terénu — desktop je menšinový scénář, typicky účetní při párování plateb a správa ústředí.
- **Responzivní webová aplikace**, ne nativní apka — odpadá distribuce přes obchody a odkazy z e-mailů (token přihlášky, schválení zástupcem, pozvánka) se otevírají přímo.
- Každou operaci, kterou dělá účastník nebo zákonný zástupce, musí jít dokončit na telefonu — včetně **nahrání dokumentů fotoaparátem** (proto je mezi povolenými typy HEIC) a načtení QR k platbě v bankovní aplikaci.
- Husté tabulky (seznam přihlášek, párování plateb, reporty) mají na úzkém displeji **alternativní zobrazení** místo horizontálního scrollování; export do tabulky zůstává doménou desktopu.
- Ovládací prvky jsou dimenzované na dotyk a formuláře používají správné typy vstupů (e-mail, telefon, datum), aby se na mobilu nabídla odpovídající klávesnice.
- Rozhraní musí být použitelné i na **pomalém mobilním připojení** — těžké agregace se dopočítávají až na vyžádání, ne při prvním načtení stránky.

## Rozsah a výkon

- Řádově tisíce osob a stovky akcí ročně na oddíl — návrh nemusí řešit horizontální škálování, ale **musí zvládnout reporty ústředí nad všemi oddíly** ([reports.md](reports.md)).
- Reporty jsou read-only a cachovatelné na úrovni odpovědi; těžké agregace nesmí blokovat běžnou práci s přihláškami.
- Import transakcí respektuje rate limit banky ([fio-sync.md](fio-sync.md)).

## Co systém záměrně neřeší

- **Odchozí platební příkazy** — vratky vyplácí účetní ve své bance, systém je jen eviduje ([payment-matching.md](payment-matching.md)).
- **Účetnictví** — systém eviduje platby a jejich přiřazení, není účetní software.
