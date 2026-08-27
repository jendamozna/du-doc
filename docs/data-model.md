# Datový model (ER diagram)

Implementační detail ke specifikaci [README.md](../README.md).

> Návrh schématu odvozený ze specifikace. Spojovací (M:N) a historizační tabulky jsou uvedeny zvlášť.
>
> **Aktérské cizí klíče se v diagramu nekreslí.** Pole typu `*_by_account_id`, `actor_account_id` a `initiator_account_id` zaznamenávají, kdo operaci provedl; v databázi jsou to cizí klíče na `ACCOUNT`, ale relace se nekreslí, aby diagram nezahltila hvězda čar kolem `ACCOUNT`. Nakreslené vazby na `ACCOUNT` proto značí, že je účet **předmětem** záznamu (např. `EVENT_ASSIGNMENT`), ne jeho původcem.

```mermaid
%%{init: {
  "theme": "base",
  "layout": "elk",
  "elk": {
    "nodePlacementStrategy": "LINEAR_SEGMENTS"
  },
  "themeVariables": {
    "background": "#ffffff",
    "primaryColor": "#eff6ff",
    "primaryBorderColor": "#1e3a8a",
    "primaryTextColor": "#0f172a",
    "secondaryColor": "#f5f3ff",
    "secondaryBorderColor": "#7c3aed",
    "tertiaryColor": "#ecfeff",
    "tertiaryBorderColor": "#0f766e",
    "lineColor": "#475569"
  }
}}%%

erDiagram
    REGION ||--o{ UNIT_REGION : includes
    UNIT ||--o{ UNIT_REGION : "membership (versioned)"
    UNIT ||--o{ EVENT : organizes
    UNIT ||--o{ ACTION_TEMPLATE : defines
    UNIT ||--o{ BANK_ACCOUNT : has
    UNIT ||--o{ UNIT_PATROL : has
    UNIT ||--o{ PERSON_UNIT : tracks
    UNIT ||--o{ USER_ROLE : "scoped to"
    UNIT ||--o{ CUSTOM_FIELD : defines
    UNIT ||--o{ LOCATION : defines
    UNIT ||--o{ DU_MEMBERSHIP : "registers (evidencni oddil)"
    UNIT ||--o{ DU_FEE_BATCH : submits
    DU_FEE_BATCH ||--o{ DU_FEE_BATCH_ITEM : contains
    DU_FEE_BATCH ||--o{ DU_MEMBERSHIP : "creates on payment"
    DU_FEE_BATCH ||--o{ PAYMENT_ALLOCATION : "paid by"
    UNIT }o--o| LOCATION : "based at"
    EVENT }o--o| LOCATION : "held at"

    PERSON ||--o| ACCOUNT : has
    PERSON ||--o{ PERSON_UNIT : "tracked in"
    PERSON_UNIT ||--o{ PERSON_UNIT_HISTORY : "state changes"
    PERSON ||--o{ PARENT_CHILD : "as parent"
    PERSON ||--o{ PARENT_CHILD : "as child"
    PERSON ||--o{ UNIT_PATROL_MEMBER : is
    PERSON ||--o{ REGISTRATION : submits
    PERSON ||--o{ DU_MEMBERSHIP : "has (globalne, 1 za rok)"
    PERSON ||--o{ DU_FEE_BATCH_ITEM : "listed in"
    PERSON ||--o{ ATTENDANCE_RECORD : attends
    PERSON ||--o{ CUSTOM_FIELD_VALUE : has
    PERSON ||--o{ PERSON_COURSE : completes

    ACCOUNT ||--o{ OAUTH_IDENTITY : has
    ACCOUNT ||--o{ USER_ROLE : has

    UNIT_PATROL ||--o{ UNIT_PATROL_MEMBER : contains
    UNIT_PATROL ||--o{ CUSTOM_FIELD : scopes

    ACTION_TEMPLATE ||--o{ EVENT : "instantiated as"
    EVENT ||--o{ EVENT_ASSIGNMENT : delegates
    ACCOUNT ||--o{ EVENT_ASSIGNMENT : "assigned to (versioned)"
    EVENT ||--o{ EVENT_PRICE : has
    EVENT_PRICE ||--o{ REGISTRATION : "priced by"
    EVENT ||--o{ CANCELLATION_RULE : has
    EVENT ||--o{ EVENT_FIELD : has
    EVENT ||--o{ REGISTRATION : contains
    REGISTRATION ||--o{ REGISTRATION : "sub-registrations"
    EVENT ||--o{ EVENT_DOCUMENT : requires
    EVENT ||--o{ ATTENDANCE_RECORD : attendance
    EVENT }o--o| BANK_ACCOUNT : "linked to"
    EVENT }o--o| REGION : "region snapshot"

    EVENT_FIELD ||--o{ EVENT_FIELD_OPTION : offers
    EVENT_FIELD_OPTION ||--o{ REGISTRATION_FIELD_VALUE : "chosen by"
    REGISTRATION ||--o{ REGISTRATION_FIELD_VALUE : selects
    EVENT_DOCUMENT ||--o{ REGISTRATION_DOCUMENT : "fulfilled by"
    REGISTRATION ||--o{ REGISTRATION_DOCUMENT : uploads

    EVENT ||--o{ RACE_PATROL : "race patrols"
    REGISTRATION ||--o{ RACE_PATROL : owns
    RACE_PATROL ||--o{ RACE_PATROL_MEMBER : has
    PERSON ||--o{ RACE_PATROL_MEMBER : "in patrol"
    EVENT ||--o{ WORKSHOP_BLOCK : "has blocks"
    EVENT ||--o{ WORKSHOP : offers
    WORKSHOP }o--o| LOCATION : "held at"
    WORKSHOP_BLOCK ||--o{ WORKSHOP_OFFERING : contains
    WORKSHOP ||--o{ WORKSHOP_OFFERING : "scheduled as"
    WORKSHOP_OFFERING ||--o{ WORKSHOP_REGISTRATION : has
    REGISTRATION ||--o{ WORKSHOP_REGISTRATION : "enrolled via"
    PERSON ||--o{ WORKSHOP_REGISTRATION : attends

    REGISTRATION ||--o{ PAYMENT_ALLOCATION : "paid by"
    BANK_ACCOUNT ||--o{ BANK_TRANSACTION : records
    BANK_TRANSACTION ||--o{ PAYMENT_ALLOCATION : "split into"

    CUSTOM_FIELD ||--o{ CUSTOM_FIELD_VALUE : has
    COURSE ||--o{ PERSON_COURSE : "offered as"
    COURSE ||--o{ EVENT : "awarded by"
    EVENT ||--o{ PERSON_COURSE : "completed at"

    PERSON ||--o{ CONSENT : grants
    PERSON ||--o{ GDPR_AUDIT : "subject of"
    PERSON ||--o{ PERSON_SENSITIVE_DATA : has
    UNIT ||--o{ PERSON_SENSITIVE_DATA : owns
    EVENT ||--o{ PERSON_SENSITIVE_DATA : "context of"
    PERSON ||--o{ PARENT_INVITATION : "guardian invite"
    PERSON ||--o{ RECOMMENDATION : "as mentor"
    PERSON ||--o{ REPORT_MERGE : "candidate A"
    PERSON ||--o{ REPORT_MERGE : "candidate B"
    PERSON ||--o{ MERGE_REQUEST : "as source"
    PERSON ||--o{ MERGE_REQUEST : "as target"
    MERGE_REQUEST ||--o{ MERGE_APPROVAL : "approved by"
    MERGE_REQUEST ||--o{ MERGE_LOG : "logged as"
    UNIT ||--o{ NAME_EXCEPTION : approves
    UNIT ||--o{ AUDIT_LOG : logs
    REGISTRATION ||--o{ SUBSTITUTE_OFFER : offers
    REGISTRATION ||--o{ RECOMMENDATION : requires
    UNIT ||--o{ UNIT_MODULE : enables
    UNIT ||--o{ UNIT_SETTING : has
    UNIT ||--o{ MANDATE : has
    UNIT ||--o| UNIT_MAIL_SETTING : "mail config"
    EVENT ||--o{ EVENT_CUSTOM_FIELD : collects
    CUSTOM_FIELD ||--o{ EVENT_CUSTOM_FIELD : "included in"

    REGION {
        int id PK
        string name
        string state "active / merged / cancelled"
        date valid_from
        date valid_to "NULL = aktivni"
        int merged_into_region_id FK "nastupnicky region"
    }
    UNIT {
        int id PK
        string name
        string type "hq_ico / branch / collective"
        string ico
        bool is_hq
        int location_id FK "sidlo (volitelne)"
    }
    UNIT_REGION {
        int id PK
        int unit_id FK
        int region_id FK
        date valid_from
        date valid_to "NULL = aktualni"
    }
    PERSON {
        int id PK
        string first_name
        string last_name
        string nickname
        string title_before
        string title_after
        string gender "male / female / other"
        date birth_date "povinne u registrovaneho clena"
        string email "kontaktni e-mail (nemusi byt unikatni)"
        string address "trvale bydliste"
        string insurance_company
        int merged_into_person_id FK "NULL = platna osoba; jinak tombstone po slouceni"
    }
    PERSON_UNIT {
        int id PK
        int person_id FK
        int unit_id FK
        string membership_state "guest / registered_member"
        string record_state "active / inactive / archived"
        datetime valid_from
        datetime valid_to
    }
    ACCOUNT {
        int id PK
        int person_id FK "1:1"
        string login_email "prihlasovaci e-mail (unikatni)"
        string password_hash
    }
    OAUTH_IDENTITY {
        int id PK
        int account_id FK
        string provider "google / facebook"
        string provider_user_id
        bool email_verified
    }
    USER_ROLE {
        int id PK
        int account_id FK
        int unit_id FK "role scope"
        string role "HVO / VO / VD / RAD / ADM / UCE"
    }
    PARENT_CHILD {
        int id PK
        int parent_person_id FK
        int child_person_id FK
        string state "pending / active / cancelled / readonly_after_adulthood"
        int approved_by_account_id FK "NULL, dokud stav pending; existujici rodic nebo HVO"
        datetime valid_from
        datetime valid_to
    }
    UNIT_PATROL {
        int id PK
        int unit_id FK
        string name
    }
    UNIT_PATROL_MEMBER {
        int id PK
        int unit_patrol_id FK
        int person_id FK
        string role "leader / advisor / member"
    }
    EVENT {
        int id PK
        int unit_id FK
        int bank_account_id FK
        int region_id_snapshot FK "region pri zalozeni akce"
        int location_id FK "misto konani (volitelne)"
        int action_template_id FK "sablona (snapshot)"
        string name
        string ss "specific symbol"
        string type "club / one_off / weekend / course / certificate / recommendation / group / race / workshop"
        int course_id FK "udeluje kurz po absolvovani"
        int capacity
        int substitute_count
        string visibility "public / internal / private"
        string share_slug "neverejny sdileci odkaz"
        datetime starts_at
        datetime ends_at
        datetime registration_from
        datetime registration_to
        int payment_due_days "splatnost: pocet dni od prihlaseni"
        date payment_due_date "splatnost: pevne datum (alternativa k payment_due_days)"
        bool volunteers_enabled
        datetime volunteer_registration_from
        datetime volunteer_registration_to
        bool age_at_year_end "vek pocitan ke konci roku (jinak k datu akce)"
    }
    EVENT_PRICE {
        int id PK
        int event_id FK
        string membership_type "DU / non_DU / volunteer / leader / leader_child / sponsor"
        decimal amount
        date valid_from
        date valid_to
    }
    CANCELLATION_RULE {
        int id PK
        int event_id FK
        decimal percent
        date valid_until
    }
    EVENT_ASSIGNMENT {
        int id PK
        int event_id FK,UK "unikat: akce + ucet, jen mezi otevrenymi zaznamy"
        int account_id FK,UK
        bool can_edit_event
        bool can_edit_registrations
        bool can_edit_prices
        bool can_record_attendance
        int assigned_by_account_id FK
        datetime assigned_at
        int revoked_by_account_id FK "kdo pristup odebral; NULL = pristup trva"
        datetime revoked_at "NULL = pristup trva; zaznam se nemaze"
    }
    EVENT_FIELD {
        int id PK
        int event_id FK
        string name "nazev ciselniku"
        string comment "verejny popis pro ucastnika"
        string internal_note "neverejna poznamka pro vedouci"
        string selection_mode "exclusive / shared"
        string assigned_by "self / leader"
        int max_select "max poctu voleb (NULL = bez limitu)"
        string required_phase "on_submit / before_payment / before_event (NULL = nepovinny)"
        string condition "podminka zpusobilosti (NULL = vsichni)"
    }
    EVENT_FIELD_OPTION {
        int id PK
        int event_field_id FK
        string value
        int capacity "max ucastniku na polozku (1 = unikatni; NULL = bez limitu)"
        decimal price_modifier "priplatek k zakladni cene (muze byt zaporny)"
    }
    REGISTRATION_FIELD_VALUE {
        int id PK
        int registration_id FK
        int event_field_option_id FK
    }
    EVENT_DOCUMENT {
        int id PK
        int event_id FK
        string name
        bool required
    }
    REGISTRATION_DOCUMENT {
        int id PK
        int registration_id FK
        int event_document_id FK "ktery pozadavek plni"
        string file
        string state "pending / uploaded / approved / rejected"
        string review_note "duvod zamitnuti"
        int reviewed_by_account_id FK "kdo posoudil"
        datetime uploaded_at
        datetime reviewed_at "NULL = neposouzeno"
    }
    ACTION_TEMPLATE {
        int id PK
        int unit_id FK "NULL = systemova sablona"
        string type "club / one_off / weekend / course / certificate / recommendation / group / race / workshop"
        string name
        json config "vychozi nastaveni akce"
        bool active
    }
    RACE_PATROL {
        int id PK
        int event_id FK
        int owner_registration_id FK "prihlaska, ktera hlidku zalozila"
        string name "unikatni v ramci akce"
        string category "Stezka / Pesinka / Serpa_s_detmi / Pocestni"
    }
    RACE_PATROL_MEMBER {
        int id PK
        int race_patrol_id FK
        int person_id FK "osoba z club scope"
        string role "leader (kapitan) / member"
    }
    WORKSHOP_BLOCK {
        int id PK
        int event_id FK
        string name "casovy blok akce"
        datetime starts_at
        datetime ends_at
    }
    WORKSHOP {
        int id PK
        int event_id FK
        int location_id FK "misto konani (volitelne)"
        string type "workshop / seminar"
        string name
        string description
        string instructor "lektor"
        int min_age
        string requirements "potreby"
        int capacity "max ucastniku na beh"
    }
    WORKSHOP_OFFERING {
        int id PK
        int workshop_block_id FK "casovy blok"
        int workshop_id FK "workshop / seminar"
    }
    WORKSHOP_REGISTRATION {
        int id PK
        int workshop_offering_id FK
        int registration_id FK "prihlaska (club scope)"
        int person_id FK "ucastnik z club scope"
    }
    REGISTRATION {
        int id PK
        int event_id FK
        int person_id FK
        int parent_registration_id FK "nadrazena prihlaska (NULL = hlavni); definuje club scope"
        int price_id FK "EVENT_PRICE platna k okamziku podani; zafixovana"
        decimal base_price "snapshot zakladni ceny pri podani"
        string vs "variable symbol"
        string category "participant / volunteer / substitute"
        string state "New / PendingGuardian / PendingDocuments / PendingPayment / PartialPaid / Paid / Overpayment / Canceled / Expired"
        string guardian_email "e-mail zak. zastupce (nezletily bez rodice)"
        string guardian_approval_token
        datetime guardian_approved_at "NULL = neschvaleno"
        string token "sprava prihlasky bez uctu"
        datetime created_at "podani prihlasky; vychozi bod relativni splatnosti"
    }
    PAYMENT_ALLOCATION {
        int id PK
        int bank_transaction_id FK
        int registration_id FK "vylucne s fee_batch_id"
        int fee_batch_id FK "hromadna platba prispevku DU; vylucne s registration_id"
        decimal amount "alokovana cast platby; zaporna = vratka"
        string matched_by "auto / manual"
        string match_method "ss_vs_amount / ss_vs_partial / ss_vs_overpayment / vs_exact_name / ss_exact_name / vs_partial_name / vs_overpayment_name / manual / refund"
        datetime matched_at
        datetime confirmation_sent_at "NULL = neodeslano"
    }
    BANK_ACCOUNT {
        int id PK
        int unit_id FK
        string name
        string account_number
        string bank_code
        string provider "fio = synchronizace z API / manual = ruční evidence"
        string api_token_enc "read-only token, sifrovany; NULL = bez synchronizace"
        datetime last_sync_at "NULL = nesynchronizovano"
        string sync_state "ok / error"
    }
    BANK_TRANSACTION {
        int id PK
        int bank_account_id FK,UK "unikat: ucet + external_id"
        string external_id UK "id pohybu z banky, jinak manual:<uuid> / stmt:<otisk radku>"
        string source "import / statement_import / manual_entry"
        int entered_by_user_id FK "kdo pohyb zapsal; NULL = automaticky import"
        string ss
        string vs
        decimal amount
        string sender_name
        string sender_account
        string sender_bank_code
        string message
        string transaction_type
        date date "datum transakce dle banky"
        datetime imported_at "kdy ji stahl import nebo kdy vznikl rucni zapis"
        datetime ignored_at "ucetni oznacila jako neprirazovanou; NULL = v rade k parovani"
        datetime voided_at "stornovany rucni zapis; jen u source != import a bez alokaci"
    }
    DU_MEMBERSHIP {
        int id PK
        int person_id FK,UK "unikat: osoba + rok, globalne pres cely system"
        int unit_id FK "evidencni oddil - kdo clenstvi zalozil; neomezuje platnost"
        int year UK
        int fee_batch_id FK "davka, jejiz platba clenstvi zalozila; NULL = zalozil ADM rucne"
    }
    DU_FEE_RATE {
        int id PK
        int year UK "unikat: rok; sazbu spravuje ADM"
        decimal amount
        int set_by_account_id FK
        datetime set_at
        datetime locked_at "prvni platba na tento rok; NULL = jeste lze menit"
    }
    DU_FEE_BATCH {
        int id PK
        int unit_id FK "oddil, ktery davku podal = budouci evidencni oddil"
        int year
        string vs UK "variabilni symbol davky"
        decimal total_amount "pocet polozek x sazba; zamrzne pri uzamceni"
        string state "draft / locked / paid / canceled"
        datetime locked_at "vygenerovani QR; seznam osob se uz nemeni"
        datetime paid_at "sparovano ucetni ustredi; NULL = nezaplaceno"
        int created_by_account_id FK
        datetime created_at
    }
    DU_FEE_BATCH_ITEM {
        int id PK
        int batch_id FK,UK "unikat: davka + osoba"
        int person_id FK,UK
        datetime skipped_at "clenstvi vzniklo jinou davkou driv; NULL = zpracovano"
    }
    ATTENDANCE_RECORD {
        int id PK
        int event_id FK,UK "unikat: akce + osoba"
        int person_id FK,UK
        bool present "false = zapsan, nedorazil"
        decimal volunteer_hours "odpracovane hodiny"
    }
    CUSTOM_FIELD {
        int id PK
        int unit_id FK
        int unit_patrol_id FK "druzina (volitelne)"
        string name
        string visibility "none / view / edit (vlastnik uctu)"
        string permission "none / view / edit (radce)"
    }
    CUSTOM_FIELD_VALUE {
        int id PK
        int custom_field_id FK
        int person_id FK
        string value
    }
    COURSE {
        int id PK
        string name
        int validity_months
    }
    LOCATION {
        int id PK
        int unit_id FK "vlastnik"
        string name
        decimal lat
        decimal lng
        string address "volitelne"
    }
    PERSON_COURSE {
        int id PK
        int person_id FK
        int course_id FK
        int source_event_id FK "vzdelavaci akce (volitelne)"
        date completed_on
        date valid_to "cache: completed_on + validity_months"
        string certificate_file
    }
    CONSENT {
        int id PK
        int person_id FK
        string type "processing / photo / health / ..."
        string purpose
        datetime granted_at
        datetime revoked_at "NULL = platny"
        date retention_until
    }
    SUBSTITUTE_OFFER {
        int id PK
        int registration_id FK
        string token
        datetime offered_at
        datetime expires_at
        string state "offered / accepted / expired"
    }
    RECOMMENDATION {
        int id PK
        int registration_id FK
        int mentor_person_id FK "NULL = jen e-mail"
        string mentor_email
        string type "mentor / leader"
        string expectation
        string state "requested / confirmed / rejected"
        datetime confirmed_at
    }
    EVENT_CUSTOM_FIELD {
        int id PK
        int event_id FK
        int custom_field_id FK "chytry sloupec zarazeny do prihlasky"
        bool required "povinne pole prihlasky"
    }
    MANDATE {
        int id PK
        int unit_id FK
        string file
        date valid_from
        date valid_to
    }
    UNIT_MODULE {
        int id PK
        int unit_id FK
        string code "payment_matching / payment_confirmation / training / custom_fields / reports"
        bool active
        json config "nastaveni modulu"
        datetime activated_at
    }
    UNIT_SETTING {
        int id PK
        int unit_id FK
        string key "napr. reminder_frequency_days"
        string value
    }
    UNIT_MAIL_SETTING {
        int id PK
        int unit_id FK
        string from_email "odesilatel (volitelne)"
        string smtp_email "e-mail pro odchozi postu"
        string smtp_password_enc "sifrovane heslo (libsodium)"
    }
    GDPR_AUDIT {
        int id PK
        string action "anonymize / purge / ..."
        int person_id FK "dotcena osoba (NULL = hromadne)"
        string scope "person / unit / guests / sensitive"
        int by_account_id FK "kdo (NULL = system)"
        string detail
        datetime created_at
    }
    AUDIT_LOG {
        int id PK
        string entity_type "race_patrol / race_patrol_member / registration / registration_document / event / event_assignment / parent_child / ..."
        int entity_id "bez FK (polymorfni)"
        string action "create / update / delete / join / leave / approve / reject / cancel"
        int unit_id FK "izolace a mazani per oddil"
        int actor_account_id FK "NULL = system nebo akter bez uctu"
        string actor_email "akter bez uctu (token)"
        json detail "co se zmenilo / duvod"
        datetime created_at
    }
    PERSON_SENSITIVE_DATA {
        int id PK
        int person_id FK
        int unit_id FK "vlastnici oddil"
        int event_id FK "kontext akce (volitelne)"
        string category "health / allergy / medication / diet"
        string content
        datetime created_at
    }
    PARENT_INVITATION {
        int id PK
        int child_person_id FK
        string email
        string token
        datetime expires
        int invited_by_account_id FK
        datetime accepted_at "NULL = nevyrizena"
        datetime created_at
    }
    NAME_WHITELIST {
        int id PK
        string name "krestni jmeno"
    }
    NAME_EXCEPTION {
        int id PK
        int unit_id FK
        string name "krestni jmeno mimo whitelist"
        int approved_by_account_id FK "schvalil HVO"
        datetime created_at
    }
    MERGE_REQUEST {
        int id PK
        string kind "person / child"
        int source_person_id FK "tombstone po slouceni"
        int target_person_id FK "vysledna osoba"
        int initiator_account_id FK
        int keep_account_id FK "ktery ucet zustava"
        string state "pending / ready / rejected / completed / reverted"
        datetime created_at
        datetime expires_at "propadnuti zadosti bez odezvy"
        datetime completed_at
    }
    MERGE_APPROVAL {
        int id PK
        int merge_request_id FK
        string party "initiator / candidate / hvo / parent"
        int account_id FK
        bool approved "NULL = nerozhodnuto"
        datetime decided_at
    }
    MERGE_LOG {
        int id PK
        int merge_request_id FK
        int source_person_id FK
        int target_person_id FK
        json snapshot "pro revert"
        int merged_by_account_id FK
        datetime created_at
        datetime reverted_at "NULL = platne"
    }
    REPORT_MERGE {
        int id PK
        int person_a_id FK
        int person_b_id FK
        string reason "duvod"
        datetime created_at
    }
    PERSON_UNIT_HISTORY {
        int id PK
        int person_unit_id FK
        string from_membership
        string to_membership
        string from_record
        string to_record
        string note
        int changed_by_account_id FK
        datetime changed_at
    }
```
