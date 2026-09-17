# UX — Obrazovky plochy B (oddílová správa)

Detailní specifikace obrazovek oddílové administrace (`/oddil/...`). Navigaci, routy, breadcrumbs a pravidla oprávnění definuje [ux-navigace.md](ux-navigace.md) § 2.2; sdílené stavy, prázdné obrazovky a validační texty [ux-texty-stavy.md](ux-texty-stavy.md); role a matice [authorization.md](authorization.md); pravidla polí [validation.md](validation.md); entity [data-model.md](data-model.md).

## 1. Rámec plochy

- **Publikum:** vracející se vedoucí — HVO, VO, RÁD, ÚČE.
- **Ergonomie:** desktop-first — husté tabulky s filtry scrollují ve vlastním kontejneru, na mobilu se hroutí do karet. Zápis docházky je výjimka: výhradně mobilní návrh (tábořiště, jedna ruka).
- **Navigace:** M3 rail (Přehled · Akce · Platby · Osoby · Družiny a závody · Reporty · Nastavení oddílu). Badge na položce **Platby** = počet věcí k zásahu ÚČE (nespárované transakce + přeplatky). Fronty dokumentů a náhradníků jsou badge uvnitř detailu akce, ne v hlavní navigaci.

**Zásady (vynucuj v UI):**

1. **Oprávnění k zápisu se přidělují per akce** přes `EVENT_ASSIGNMENT` — VO/RÁD vidí a mění přihlášky jen tam, kde jsou přiřazení; základní čtení detailu akce a seznamu přihlášených plyne z role ve vlastním oddílu ([authorization.md](authorization.md)).
2. **Stav přihlášky se nikdy nenastavuje ručně** — vedoucí mění fakta (schválí dokument, alokuje platbu, vybere náhradníka) a stav přepočítá `evaluate()`. V UI není žádný ovladač „nastavit stav“.
3. **Platební atributy se maskují dle role** — RÁD mimo funkci Vedoucího akce nevidí částky; ÚČE vidí platby celého oddílu, ale k akcím se nepřiřazuje.
4. **Chování bez oprávnění:** položka mimo rozsah role se **skryje**, údaj mimo rozsah se **maskuje** (`———`), přímý vstup na URL bez práva → 403 (ikona `lock`, „Sem nemáte přístup“, CTA „Zpět na přehled“).

## 2. B-01 · Přehled oddílu — `/oddil`

**Účel a publikum:** kokpit vedoucího — co se v oddílu děje teď a co čeká na jeho zásah, proklikem o úroveň hlouběji.

**Layout a komponenty:** top app bar „Přehled — [název oddílu]“; řádek KPI karet, pod ním „Vyžaduje pozornost“ (M3 list) a karta „Nadcházející akce“.

**Obsah a pole:** KPI (členů, otevřené akce, přihlášek k vyřízení, nespárované platby). „Vyžaduje pozornost“ dle urgence: dokumenty k posouzení, vypršující nabídky náhradníkům, přeplatky k rozhodnutí, nespárované platby, přihlášky blokované chybějícím datem narození — každá s proklikem do cílové agendy. Nadcházející akce s obsazeností a stavem.

**Stavy:**

- **Prázdný — „Vyžaduje pozornost“:** ikona `done_all`, „Vše vyřízeno“, „Žádná přihláška, dokument ani platba nečeká na zásah.“ Bez CTA.
- **Persona ÚČE:** KPI a pozornost jen k platbám; dokumenty a náhradníci skryté.
- **Načítání:** skeletony; **Chyba:** [ux-texty-stavy.md](ux-texty-stavy.md) § 5.

**Interakce a validace:** jen proklik, žádné mutace z přehledu.

**Oprávnění:** HVO/VO/RÁD/ÚČE dle role; ÚČE jen platební agenda.

**Mobil/desktop:** KPI 2×2, seznam pozornosti první.

**Notifikace:** žádné z přehledu.

## 3. B-02 · Akce — seznam — `/oddil/akce`

**Účel a publikum:** všechny akce oddílu na jednom místě — stav, obsazenost, co potřebuje pozornost — a založení nové akce ze šablony.

**Layout a komponenty:** datová tabulka + extended FAB „Nová akce“; filter chips `Otevřené` (výchozí) · `Připravované` · `Proběhlé` · `Zrušené`; vyhledávání podle názvu.

**Obsah a pole:** sloupce Název · Typ · Termín · Přihlašování · Obsazenost · Stav (chip). Obsazenost = „počítáno/kapacita“ (do kapacity jen stavy `PendingDocuments`–`Overpayment`) + drobně „+N náhr.“. Trailing badge u řádku s frontou dokumentů nebo nespárovaných plateb.

**Stavy:**

- **Prázdný:** ikona `event`, „Zatím žádné akce“, „Založte první akci ze šablony — přednastaví dokumenty, ceny i kapacitu.“, CTA „Nová akce“.
- **Persona ÚČE:** tabulka jen ke čtení, FAB skrytý.
- **Načítání:** skeleton řádků; **Chyba:** § 5.

**Interakce a validace:** klik na řádek → `/oddil/akce/:id`. „Nová akce“ → dialog: výběr šablony → název* · termín* · přihlašovací okno · kapacita — pole přednastaví šablona jako snapshot ([validation.md](validation.md) → Akce). Placenou akci nelze publikovat bez bankovního účtu — u konceptu bez účtu disabled „Publikovat“ s tooltipem.

**Oprávnění:** zakládat akci smí HVO a VO s příslušným právem; ÚČE jen čtení.

**Mobil/desktop:** tabulka → karty (název, chip, obsazenost); FAB plovoucí.

**Notifikace:** publikace akce nespouští e-mail; pozvánky na akci řeší notifikace přihlášek.

## 4. B-03 · Detail akce — `/oddil/akce/:id`

**Účel a publikum:** řídicí panel jedné akce — vše, co k ní patří, v pěti tabech.

**Layout a komponenty:** top app bar se šipkou zpět, názvem akce a chipem stavu; M3 tabs **Nastavení · Přihlášky · Dokumenty · Náhradníci · Docházka**. Nad taby řádek rychlých metrik. Aktivní tab v query `?tab=...` kvůli přímému prokliku.

**Viditelnost tabů dle typu akce a role:**

- Akce bez přihlášek (jednorázová): jen tab **Docházka**, ostatní skryté.
- Závodní akce: tab **Náhradníci** navíc obsahuje hlídky a stanoviště (B-08).
- RÁD bez funkce Vedoucího akce: tab **Nastavení** skrytý, v tabu Přihlášky maskované částky.
- ÚČE: jen tab **Přihlášky** ke čtení; platby řeší z `/oddil/platby`.

**Stavy / načítání / chyba:** neexistující `:id` → „Akci jsme nenašli“, CTA „Zpět na seznam akcí“. Ostatní vzor § 5.

**Oprávnění:** dle role a `EVENT_ASSIGNMENT` (zásada 1).

**Mobil/desktop:** taby jako scrollovatelný řádek; metriky pod taby (na mobilu 2×2).

**Notifikace:** viz jednotlivé taby.

## 5. B-04 · Tab Nastavení akce — `/oddil/akce/:id?tab=nastaveni`

**Účel a publikum:** úplná konfigurace akce — od základních údajů po ceník, číselníky, dokumenty a storno pravidla — stejnou rodinou formulářů jako systémové šablony (plocha C, C-07).

**Layout a komponenty:** sbalitelné sekce, každá s tlačítkem „Uložit“: ① Základ · ② Přihlašování a kapacita · ③ Ceník · ④ Výběrové číselníky · ⑤ Povinné dokumenty · ⑥ Storno pravidla · ⑦ Publikace.

**Obsah a pole:**

- ① Základ: název, typ (ze šablony, needitovatelný), místo konání, popis, SS (needitovatelný, generuje systém).
- ② Přihlašování a kapacita: okno od–do, kapacita + náhradníci, viditelnost (veřejná / interní / přes odkaz), sdílecí odkaz.
- ③ Ceník: tabulka typ účastníka × období platnosti; `non_DU` je základní cena a musí pokrýt celé okno ([validation.md](validation.md)).
- ④ Výběrové číselníky (`EVENT_FIELD`): u každé položky kapacita a fáze (`on_submit` / `before_event`); položka s `required_phase = before_event` musí mít nulový příplatek (guard v UI).
- ⑤ Povinné dokumenty: typ, formáty a limit velikosti + přepínač „umožnit trvalý dokument osoby“.
- ⑥ Storno pravidla: termínovaná procenta; `percent` ∈ ⟨0; 100⟩.
- ⑦ Publikace: přepínač koncept ↔ publikováno; **placenou akci nelze publikovat bez bankovního účtu**.

**Stavy:**

- **Prázdný:** nová akce z konceptu má sekce předvyplněné ze šablony; číselníky lze mít prázdné.
- **Úspěch:** po uložení snackbar „Uloženo“ + poznámka u ceníku „Změna ceny nepřecení už podané přihlášky.“ ([validation.md](validation.md)).
- **Chyba (validace):** inline pod polem; publikace bez účtu → banner v `error-container`.
- **Bez práva (VO/RÁD bez `can_edit_event`):** sekce jen ke čtení, tlačítka „Uložit“ skrytá.

**Interakce a validace:** pravidla dle [validation.md](validation.md) → Akce, Ceny a storna, Výběrové číselníky. Snížit kapacitu pod počet započítaných přihlášek nelze; snížit kapacitu položky číselníku pod počet voleb nelze.

**Oprávnění:** editace jen s `can_edit_event`.

**Mobil/desktop:** sekce jako karty; ceníková tabulka scrolluje ve vlastním kontejneru.

**Notifikace:** změny nastavení nespouští e-mail účastníkům.

## 6. B-05 · Tab Přihlášky — `/oddil/akce/:id?tab=prihlasky`

**Účel a publikum:** seznam všech přihlášek akce se stavem a stavem úhrady a rychlá cesta k jednotlivé přihlášce i k hromadným úkonům.

**Layout a komponenty:** datová tabulka s checkbox výběrem pro hromadné akce; filter chips podle stavu; vyhledávání podle jména; při výběru kontextová lišta hromadných akcí.

**Obsah a pole:** sloupce Účastník · Stav (chip) · Cena · Uhrazeno · Zbývá · Splatnost · Kontakt. **Maskování dle role:** RÁD mimo Vedoucího akce vidí Cena/Uhrazeno/Zbývá jako `———`; jako Vedoucí akce vidí předepsáno/uhrazeno/zbývá, ne slevy ani storno poplatky ([authorization.md](authorization.md)). Hromadné akce: **„Poslat připomínku platby“** (jen na přihlášky ve stavu čekání na platbu) a **„Exportovat výběr“** (CSV).

**Stavy:**

- **Prázdný:** „Tabulka přihlášek akce“ z [ux-texty-stavy.md](ux-texty-stavy.md) § 4 (ikona `inbox`, „Zatím žádné přihlášky“, CTA „Zkopírovat sdílecí odkaz“).
- **Úspěch (hromadná připomínka):** snackbar „Odesláno N připomínek platby“; do auditu záznam per přihláška.
- **Načítání:** skeleton řádků; **Chyba:** § 5.

**Interakce a validace:** klik na řádek → B-06. Připomínka platby jen pro přihlášky s nenulovou zbývající částkou a nastaveným kontaktem. Export = ploché CSV.

**Oprávnění:** čtení dle role; hromadné akce a export dle práva k přihláškám.

**Mobil/desktop:** tabulka → karty; hromadné akce jako bottom sheet.

**Notifikace:** připomínka platby → `EMAIL_PAYMENT_REMINDER` per přihláška ([notifications.md](notifications.md)).

## 7. B-06 · Detail přihlášky — `/oddil/akce/:id/prihlaska/:pid`

**Účel a publikum:** jedna přihláška z pohledu vedoucího — týž checklist bran a stavů jako tokenový rozcestník (plocha A, A-06), rozšířený o akce, které smí jen vedoucí.

**Návrhový princip — zrcadlo rozcestníku:** stavová část (hlavička, checklist bran, karty Dokumenty a Platba) je **tatáž sdílená komponenta** jako `/stav/:token` a `/muj-ucet/prihlasky/:pid` ([ux-texty-stavy.md](ux-texty-stavy.md)). Tento oddíl popisuje jen admin odchylky.

**Obsah a pole — admin odchylky:**

- **Posouzení dokumentu:** u nahraného dokumentu „Schválit“ a „Zamítnout“; zamítnutí = dialog s výběrem důvodu (_nečitelné · neúplný dokument · prošlá platnost · jiný dokument_) + volný text — po odeslání se stane komentářem viditelným rodiči. Schválení/zamítnutí spustí `evaluate()`; zamítnutí povinného dokumentu vrátí přihlášku do `PendingDocuments` i ze `Paid`.
- **Ruční úprava přihlášky (jen HVO, VO s `can_edit_registrations`):** změna typu účastníka / ceny (`can_edit_prices`, loguje se), oprava kontaktního e-mailu, změna volby číselníku. Text „Změna se zapíše do auditu.“
- **Storno vedoucím:** dialog s výpočtem poplatku k dnešku + nepovinný důvod.
- **Chybějící datum narození:** banner „Bránu zástupce nelze vyhodnotit — chybí datum narození.“ + tlačítko „Doplnit datum narození“ (jen vedoucí; edituje osobu).
- **Platba:** vedoucí vidí historii alokací; ruční párování se dělá z `/oddil/platby` (B-10) — jen odkaz „Otevřít v platbách“.

**Stavy:** všech devět stavů dědí z rozcestníku; navíc **náhradník** — místo platby sekce „Vybrat jako náhradníka“ vede na B-08.

**Interakce a validace:** posouzení dokumentu a storno jen dle role/přiřazení; RÁD dokument neposuzuje. Každá mutace → snackbar + `evaluate()` + audit.

**Oprávnění:** dle role a `EVENT_ASSIGNMENT`.

**Mobil/desktop:** jako rozcestník; admin akce v `overflow` menu na mobilu, jako tlačítka na desktopu.

**Notifikace:** zamítnutí dokumentu → `EMAIL_DOCUMENT_REJECTED`; storno → `EMAIL_REGISTRATION_CANCELED` ([notifications.md](notifications.md)).

## 8. B-07 · Tab Dokumenty — fronta posuzování — `/oddil/akce/:id?tab=dokumenty`

**Účel a publikum:** jedno místo, kde vedoucí odbaví všechny nahrané dokumenty akce — rychle a se stejným tónem zamítnutí.

**Layout a komponenty:** fronta karet per dokument (náhled, název souboru, velikost), filter chips `Čeká na posouzení` (výchozí) · `Schválené` · `Zamítnuté`. Na kartě chip stavu a akční tlačítka.

**Obsah a pole:** fronta čekajících s badge počtu; u zamítnutého viditelný důvod. Akce na kartě čekajícího: „Schválit“ · „Zamítnout“ → dialog s předvolbami důvodu + volný text. Klik na náhled → větší náhled v dialogu.

**Stavy:**

- **Prázdný:** „Fronta dokumentů“ z [ux-texty-stavy.md](ux-texty-stavy.md) § 4 (ikona `done_all`, „Vše posouzeno“).
- **Úspěch:** po posouzení karta zmizí z fronty, badge klesne, snackbar „Dokument schválen“ / „Dokument zamítnut — účastník dostane e-mail“; `evaluate()` přepočítá přihlášku.
- **Načítání:** skeleton karet; **Chyba:** § 5.

**Interakce a validace:** posuzuje jen HVO nebo VO s `can_edit_registrations`; RÁD sem nemá zápis. Zamítnutí bez důvodu nelze odeslat.

**Oprávnění:** viz výše.

**Mobil/desktop:** karty v jednom sloupci; na mobilu primární akce jako velké tlačítko, dialog důvodu jako bottom sheet.

**Notifikace:** zamítnutí → `EMAIL_DOCUMENT_REJECTED` ([notifications.md](notifications.md)).

## 9. B-08 · Tab Náhradníci a hlídky — `/oddil/akce/:id?tab=nahradnici`

**Účel a publikum:** řídit čekací listinu a nabídky uvolněných míst a u závodní akce sestavené hlídky a stanoviště.

**Layout a komponenty:** sekce „Náhradníci“ (M3 list v pořadí podání) + u závodní akce sekce „Hlídky“ a „Stanoviště“ (karty). U náhradníka chip stavu nabídky.

**Obsah a pole:** náhradníci v pořadí podání; u aktuálního běžící nabídka s termínem platnosti a akcí „Zrušit nabídku“, u dalších „Nabídnout místo“ (aktivní jen když je volné místo). Text „Pořadí je podle času podání, výběr je na vedoucím — pořadí se účastníkům neukazuje.“ Guard: současně běžících nabídek nejvýše tolik, kolik je volných míst. **Hlídky:** složení dle věkové kategorie a unikátního názvu v akci ([race-patrols.md](race-patrols.md)); vedoucí je jen ke čtení upravuje a řeší nekonzistenci. **Stanoviště:** rozhodčí; prázdný slot je platný stav.

**Stavy:**

- **Prázdný — náhradníci:** „Náhradníci akce“ z [ux-texty-stavy.md](ux-texty-stavy.md) § 4 (ikona `group_off`).
- **Prázdný — hlídky:** „Hlídky závodu“ z § 4 (ikona `flag`).
- **Úspěch:** po „Nabídnout místo“ → náhradník dostane nabídku (token), chip „Nabídka běží 48 h“, snackbar. Po vypršení se místo vrátí do fronty.
- **Načítání / Chyba:** § 5.

**Interakce a validace:** nabídku spouští jen HVO/VO s `can_edit_registrations`; přijetí/odmítnutí řeší náhradník na `/nabidka/:token` (A-07). Prázdný slot rozhodčího je platný stav, ne chyba.

**Oprávnění:** viz výše.

**Mobil/desktop:** seznamy v jednom sloupci; hlídky jako karty s kapitánem nahoře.

**Notifikace:** nabídka → e-mail náhradníkovi; při vypršení `EMAIL_SUBSTITUTE_OFFER_DECLINED` ([notifications.md](notifications.md)).

## 10. B-09 · Tab Docházka — `/oddil/akce/:id?tab=dochazka`

**Účel a publikum:** zapsat, kdo dorazil — na tábořišti, na telefonu, jednou rukou. Jediná výhradně mobilní obrazovka plochy B.

**Layout a komponenty:** seznam účastníků (nebo členů družiny) s velkými přepínači **Přítomen / Nepřítomen / (nezapsáno)**; segmented button pro filtr podle družiny; sticky souhrn „Přítomno N / M“.

**Obsah a pole:** každý řádek jméno, družina, třístavový přepínač (výchozí „nezapsáno“ — šedý). Nezapsaný ≠ nepřítomný; nejvýše jeden záznam na osobu a akci.

**Stavy:**

- **Prázdný:** „Docházka“ z [ux-texty-stavy.md](ux-texty-stavy.md) § 4 (ikona `checklist`, „Není koho zapsat“).
- **Úspěch:** změna přepínače se ukládá okamžitě (optimisticky), tichý snackbar „Uloženo“; souhrn se přepočítá.
- **Načítání / Chyba:** § 5.

**Interakce a validace:** zápis smí HVO nebo kdokoli s `can_record_attendance` (i RÁD) — samostatné oprávnění, nezávislé na platbách.

**Oprávnění:** viz výše.

**Mobil/desktop:** primárně mobil; desktop zobrazí týž seznam vlevo, vpravo souhrn.

**Notifikace:** žádné.

## 11. B-10 · Platby — párování — `/oddil/platby`

**Účel a publikum:** pracoviště účetní — srovnat příchozí platby s přihláškami, dořešit nespárované a rozhodnout přeplatky, přesně na korunu.

**Layout a komponenty:** dvoupanel (desktop): vlevo fronta transakcí, vpravo detail vybrané transakce s návrhy párování; filter chips `Nespárované` (výchozí) · `Spárované` · `Přeplatky` · `Vratky`. Tlačítka „Nahrát výpis“ a „Zapsat platbu ručně“. Sekce „Přeplatky k rozhodnutí“.

**Obsah a pole:** fronta nespárovaných s návrhy dle pravidel párování ([payment-matching.md](payment-matching.md), párování je přesné bez tolerance); přeplatky se třemi akcemi: **Vrátit** (záporná alokace `refund`) · **Převést na jinou přihlášku** téže osoby · **Ponechat jako dar**. Spárované ke čtení s metodou párování a cílovou přihláškou.

**Stavy:**

- **Prázdný — nespárované:** „Fronta transakcí (ÚČE)“ z [ux-texty-stavy.md](ux-texty-stavy.md) § 4 (ikona `account_balance`, „Žádné nespárované platby“).
- **Prázdný — přeplatky:** inline řádek „Žádný přeplatek nečeká na rozhodnutí.“
- **Úspěch:** po potvrzení párování transakce zmizí z fronty, přihláška projde `evaluate()`, snackbar „Platba spárována“, badge Platby klesne.
- **M:N:** jedna platba za víc přihlášek → návrh s víc cíli a ručním potvrzením (sekundární scénář).
- **Načítání / Chyba:** § 5.

**Interakce a validace:** párovat a řešit přeplatky smí HVO a ÚČE ([authorization.md](authorization.md) → Platby). Součet alokací transakce nesmí překročit její částku; vratka nesmí stáhnout součet přihlášky pod nulu. Ruční zápis generuje `external_id` (`manual:<uuid>`); VS/SS bez počátečních nul.

**Oprávnění:** HVO a ÚČE.

**Mobil/desktop:** dvoupanel se na mobilu hroutí — fronta jako seznam, detail transakce jako bottom sheet.

**Notifikace:** spárování platby → `EMAIL_PAYMENT_CONFIRMATION`; přeplatek → `EMAIL_PAYMENT_RECONCILIATION_ALERT` ([notifications.md](notifications.md)).

## 12. B-11 · Osoby — `/oddil/osoby`

**Účel a publikum:** evidence členů a hostů oddílu — kdo je kdo, ve které družině, v jakém stavu — a úprava jejich údajů.

**Layout a komponenty:** datová tabulka + tlačítko „Přidat osobu“; filter chips `Členové` (výchozí) · `Hosté` · `Neaktivní` · podle družiny; vyhledávání. Detail osoby jako pravý panel / bottom sheet.

**Obsah a pole:** sloupce Jméno · Typ (člen / host) · Věk · Družina · Zástupci · Stav. Detail — Základ: jméno (našeptávač whitelistu, [validation.md](validation.md)), příjmení, datum narození, pohlaví, adresa, pojišťovna, kontakt. Sekce Vazby (zástupci u dítěte), Členství (družina, stav), Chytré sloupce (dle přístupu). **Citlivá/zdravotní data** se zobrazují jen podle role — RÁD nezletilý je nevidí; maskováno s ikonou `lock` ([authorization.md](authorization.md) → Citlivá data).

**Stavy:**

- **Prázdný:** „Evidence osob“ z [ux-texty-stavy.md](ux-texty-stavy.md) § 4 (ikona `person_search`, CTA „Přidat osobu“).
- **Úspěch:** úprava → snackbar „Uloženo“ + audit. Změna stavu host → člen dialogem.
- **Bez práva:** VO/RÁD vidí evidenci ke čtení (RÁD jen svou družinu); editace jen HVO.
- **Načítání / Chyba:** § 5.

**Interakce a validace:** upravit údaje osoby a měnit stav smí jen HVO ([authorization.md](authorization.md) → Osoby). Podmíněná povinnost polí dle kontextu ([validation.md](validation.md)). Datum narození dítěte edituje jen vedoucí oddílu.

**Oprávnění:** viz výše.

**Mobil/desktop:** tabulka → karty; detail jako bottom sheet.

**Notifikace:** změna, která uvolní chybějící kontakt, může navázat na výzvu k převzetí účtu; jinak žádné.

## 13. B-12 · Družiny a závody — `/oddil/druziny`

**Účel a publikum:** členění oddílu do družin (pro docházku i chytré sloupce) a příprava závodních stanovišť.

**Layout a komponenty:** karty družin (název, vedoucí, počet členů) + tlačítko „Založit družinu“; u oddílu se zapnutým modulem závodů sekce „Stanoviště“.

**Obsah a pole:** družiny s vedoucím a členy, možnost přeřadit; funkce `leader`/`deputy` nejvýše jedna na družinu ([validation.md](validation.md)). Stanoviště (jsou-li závody): správa napříč akcemi oddílu.

**Stavy:**

- **Prázdný:** „Družiny“ z [ux-texty-stavy.md](ux-texty-stavy.md) § 4 (ikona `groups`, CTA „Založit družinu“).
- **Úspěch:** založení/přeřazení → snackbar + audit.
- **Načítání / Chyba:** § 5.

**Interakce a validace:** definovat družiny a jejich členy smí jen HVO. Osoba může být `leader`/`deputy` ve víc družinách téhož oddílu; nejvýše jeden `leader` a jeden `deputy` na družinu.

**Oprávnění:** HVO.

**Mobil/desktop:** karty v jednom sloupci; seznam členů jako bottom sheet.

**Notifikace:** žádné.

## 14. B-13 · Reporty oddílu — `/oddil/reporty`

**Účel a publikum:** oddílové reporty — členská základna, účast, platby, docházka — jako podklad pro vedení a vykazování.

**Layout a komponenty:** vlevo výběr reportů (M3 list, kód + název), vpravo plocha reportu: parametry · graf · tabulka · tlačítko „Export CSV“. Scope je vlastní oddíl ([reports.md](reports.md), [authorization.md](authorization.md) → Reporty).

**Obsah a pole:** výběr dle role (R1 Akce a docházka · R2 Členové v čase · R3 Účast na akcích · R4 Docházka schůzek · R5 Dobrovolnické hodiny · R7 Platby). ÚČE vidí jen R7; RÁD jen osoby své družiny; VO/HVO celý oddíl. Parametry: období od–do (výchozí 12 měsíců) · granularita · typ akce. Pod výsledkem metadata generování.

**Stavy:**

- **Prázdný:** „Reporty“ z [ux-texty-stavy.md](ux-texty-stavy.md) § 4 (ikona `monitoring`, „Za toto období nejsou data“). Série s nulami není prázdný stav — prázdné koše se kreslí jako nuly.
- **Úspěch (export):** snackbar „CSV exportováno“.
- **Načítání:** skeleton se zachovanou plochou grafu; **Chyba:** § 5.

**Interakce a validace:** reporty jen ke čtení; scope se odvozuje z role, ne z parametru requestu.

**Oprávnění:** dle role (viz výše).

**Mobil/desktop:** výběr reportů jako chips; grafy responzivní, tabulky ve scrollu.

**Notifikace:** žádné.

## 15. B-14 · Nastavení oddílu — `/oddil/nastaveni`

**Účel a publikum:** konfigurace oddílu jako celku — údaje, bankovní účet, lhůty, moduly, tým rolí — na jednom místě.

**Layout a komponenty:** sbalitelné sekce: ① Základ · ② Bankovní účet · ③ Lhůty a pravidla · ④ Moduly · ⑤ Tým a role · ⑥ Členské příspěvky.

**Obsah a pole:**

- ① Základ: název, město, typ, IČO (modulo 11).
- ② Bankovní účet: číslo účtu (Fio), stav synchronizace; token se nastavuje (maskovaný), párování běží i z nahraného výpisu bez API ([fio-sync.md](fio-sync.md)).
- ③ Lhůty a pravidla: schválení zástupcem, nabídka náhradníkovi, vypršení nezaplacených přihlášek.
- ④ Moduly: družiny, závody, dobrovolnické hodiny.
- ⑤ Tým a role: seznam účtů s rolemi + „Pozvat do týmu“ (pozvánka na roli VO/RÁD/ÚČE, platí 14 dní). Odebrání role uzavře otevřená přiřazení akcí a zapíše audit; posledního HVO odebrat nelze.
- ⑥ Členské příspěvky: lokální složka příspěvku, výchozí sazba pro rok.

**Stavy:**

- **Úspěch:** uložení sekce → snackbar + audit.
- **Bez práva:** jen HVO má zápis; ÚČE vidí bankovní účet ke čtení; ostatní sekce skryté.
- **Chyba (validace):** IČO modulo 11, pozvánka validní e-mail; inline pod polem.
- **Načítání / Chyba:** § 5.

**Interakce a validace:** nastavení účtu, lhůt, modulů a správu rolí smí jen HVO ([authorization.md](authorization.md)). Přijetí pozvánky existujícím účtem přidá roli k němu.

**Oprávnění:** HVO; ÚČE jen bankovní účet ke čtení.

**Mobil/desktop:** sekce jako karty; správa rolí jako tabulka → karty.

**Notifikace:** pozvánka na roli → `EMAIL_ROLE_INVITE`; odebrání role → notifikace dotčenému účtu ([notifications.md](notifications.md)).
