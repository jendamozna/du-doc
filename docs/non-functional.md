# Nefunkční požadavky

Implementační detail k [README.md](../README.md) → **Požadavky**, kde jsou uvedené měna, časové pásmo a formáty. Tento dokument popisuje zbytek provozních a technických požadavků. Schéma viz [data-model.md](data-model.md).

## Přihlašování přes OAuth

- Podporovaní poskytovatelé: **Google a Facebook**. `client_id` a `client_secret` jsou v konfiguraci prostředí, **nikdy v databázi** ani v repozitáři; callback URL je registrovaná u poskytovatele a musí se shodovat přesně.
- Rozsah oprávnění je jen `email` + `profile` — systém od poskytovatele nic dalšího nepotřebuje.
- `OAUTH_IDENTITY` drží `provider` a `subject` (stabilní identifikátor od poskytovatele), **ne access ani refresh token**. Přihlašování je jednorázové ověření, ne trvalý přístup k účtu jinde.
- Identita se páruje na účet přes **ověřený** e-mail. Vrátí-li poskytovatel e-mail bez příznaku ověření, přihlášení se odmítne.
- Existuje-li už účet se stejným e-mailem a heslem, propojení se nabídne **až po úspěšném přihlášení heslem**. Automatické propojení by znamenalo převzetí účtu přes podvržený e-mail.
- Odpojit identitu lze jen tehdy, zbývá-li účtu jiný způsob přihlášení (heslo nebo druhá identita).
- Sloučení osob přenese OAuth identity pod ponechaný účet (viz README → **Deduplikace**).

## Úložiště souborů

Systém ukládá tři druhy souborů: **dokumenty přihlášek** (potvrzení od lékaře, bezinfekčnost — citlivá data), **šablony potvrzení o platbě** a **loga / přílohy akcí**.

- Objektové úložiště kompatibilní s S3. Do databáze se ukládá **klíč objektu, ne URL** (`REGISTRATION_DOCUMENT.file`) — URL se generuje až při stahování.
- Klíč obsahuje náhodné UUID a **nikdy jméno osoby ani název akce**; z klíče nesmí jít nic odvodit.
- Žádný soubor není veřejně dostupný. Stažení probíhá přes **krátce platnou podepsanou URL** (řádově minuty) vydanou až po ověření oprávnění na konkrétní přihlášku.
- Limit velikosti **10 MB** na soubor, whitelist typů PDF/JPG/PNG/HEIC ověřený podle **skutečného obsahu souboru, ne podle přípony**.
- Retenční a GDPR mazání maže i objekt v úložišti, nejen řádek v databázi. Ověření, že objekt zmizel, je součástí dokladu o výmazu.
- Zálohy úložiště podléhají stejným retenčním lhůtám jako databáze.

## Šifrování a hesla

- Pole s příponou `_enc` (`api_token_enc`, `smtp_password_enc`) jsou šifrovaná symetricky (libsodium secretbox) s **klíčem z konfigurace prostředí**, nikdy z databáze. Každý záznam má vlastní nonce.
- Šifruje se **jen to, co systém musí přečíst zpět** — přístupové údaje k bance a k odchozí poště. Nic jiného.
- Uživatelská hesla se **hashují** (Argon2id), nešifrují. Reset hesla je jednorázový token s krátkou platností.
- Šifrované ani hashované hodnoty se nikdy nevypisují do logů, chybových hlášek ani do exportů.
- Podpora **rotace klíče** — každý záznam nese identifikátor verze klíče, aby šlo přešifrovat postupně bez výpadku.

## Tokeny a ochrana přístupu

- Přihláška se dá spravovat bez účtu přes odkaz s tokenem (`REGISTRATION.token`) a stejným způsobem schvaluje zákonný zástupce. Token je náhodný (min. 128 bitů), vázaný na jednu přihlášku a **omezeně platný** — u schválení zástupcem lhůtou, u správy přihlášky koncem akce.
- Token opravňuje jen k operacím nad danou přihláškou; nikdy nezpřístupní seznam osob ani jiné akce.
- Přihlašování má **throttling** podle účtu i IP; po sérii neúspěchů dočasné zamknutí a e-mail vlastníkovi účtu.
- Ochrana proti výčtu účtů: chybová hláška u přihlášení i u obnovy hesla je vždy stejná bez ohledu na to, zda účet existuje.

## Odchozí e-maily

- Oddíl si může nastavit **vlastní SMTP** (`smtp_email`, `smtp_password_enc`); není-li nastavené, použije se systémové odesílání.
- E-maily se odesílají **z fronty**, ne synchronně v požadavku — selhání odeslání nesmí shodit registraci ani spárování platby.
- Opakování při chybě s exponenciálním odstupem a konečným počtem pokusů; trvale neodeslaný e-mail se zobrazí vedoucímu, ne jen do logu.
- Odražené a odmítnuté adresy se označují u osoby, aby se na neplatnou adresu nezkoušelo posílat donekonečna.
- Každé odeslání se eviduje (událost, příjemce, čas) — bez toho nejde doložit, že výzva k platbě nebo žádost zástupci opravdu odešla.

## Plánované úlohy

| Úloha                           | Frekvence          | Poznámka                                                           |
| ------------------------------- | ------------------ | ------------------------------------------------------------------ |
| import bankovních transakcí     | dle nastavení účtu | idempotentní podle `external_id` ([fio-sync.md](fio-sync.md))      |
| párování nových transakcí       | po každém importu  | i po vzniku přihlášky ([payment-matching.md](payment-matching.md)) |
| expirace schválení zástupcem    | denně              | stav `PendingGuardian` po lhůtě → `Expired`                        |
| propadnutí nabídky náhradníkovi | hodinově           | nabídka propadá, stav přihlášky se nemění                          |
| výzvy a připomínky splatnosti   | denně              | termín podle nastavení akce                                        |
| retenční mazání a anonymizace   | denně              | podle tabulky lhůt v README → **Retence a GDPR**                   |
| vyřízení žádostí o výmaz (GDPR) | denně              | s dokladem o výmazu                                                |
| čištění auditního logu          | měsíčně            | podle retence v [audit-log.md](audit-log.md)                       |

- Všechny úlohy jsou **idempotentní** — opakovaný běh nesmí nic zdvojit ani smazat víc.
- Každá úloha má **zámek proti souběhu** a eviduje běh (začátek, konec, počet zpracovaných záznamů, chyba).
- Časy se vyhodnocují v `Europe/Prague`, aby „denně" znamenalo místní den včetně přechodu času.
- Selhání úlohy se hlásí administrátorovi; tichý výpadek importu plateb by se projevil až upomínkami zaplaceným lidem.

## Rozsah a výkon

- Řádově tisíce osob a stovky akcí ročně na oddíl — návrh nemusí řešit horizontální škálování, ale **musí zvládnout reporty ústředí nad všemi oddíly** ([reports.md](reports.md)).
- Reporty jsou read-only a cachovatelné na úrovni odpovědi; těžké agregace nesmí blokovat běžnou práci s přihláškami.
- Import transakcí respektuje rate limit banky ([fio-sync.md](fio-sync.md)).

## Co systém záměrně neřeší

- **DPH** — oddíly jsou neplátci DPH a akce jsou nezisková činnost.
- **Vícejazyčnost** — rozhraní i e-maily jsou česky.
- **Odchozí platební příkazy** — vratky vyplácí účetní ve své bance, systém je jen eviduje ([payment-matching.md](payment-matching.md)).
- **Účetnictví** — systém eviduje platby a jejich přiřazení, není účetní software.
