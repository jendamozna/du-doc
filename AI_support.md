# AI usecasy

Přehled míst, kde by AI dávala v systému smysl.

## Nová událost/akce

- Systém navrhne název, datum, popis nové akce podle toho co naposledy vytvářel
- Návrh storno termínů a procent podle data konání.

## Zpracování účtenek k akci

### Extrakce dat (OCR + AI)

- Z vyfocené/naskenované účtenky vytáhne: dodavatele, IČO/DIČ, datum, položky, částku, DPH, způsob platby (hotově/kartou).
- Rozpozná i papírové i PDF/e-mailové účtenky a paragony.

### Zařazení k akci a kategorizace

- Automaticky navrhne, ke které akci účtenka patří (podle data konání, vedoucího, který nahrává).
- Kategorie nákladu: materiál, jídlo, doprava, ubytování, odměny… → předvyplní řádek vyúčtování.

### Kontrola a compliance

- Upozorní na chybějící náležitosti (chybí IČO, datum, nečitelná částka) → vedoucí doplní.
- Označí účtenky mimo termín akce nebo nadlimitní položky k revizi účetní.

### Proplácení vedoucím

- Z nahraných účtenek jednoho vedoucího připraví podklad „k proplacení" (kdo, kolik, za co) → schvaluje HVO/účetní.

## Párování plateb

- Když automatika podle SS/VS selže (chybný/chybějící symbol), AI navrhne nejpravděpodobnější přihlášku podle částky, jména v poznámce transakce a času → vedoucí jen potvrdí.

## Modul vzdělávání

- Extrakce dat z nahraných certifikátů/potvrzení (OCR + AI): název kurzu, datum, platnost → předvyplnění OSOBA_KURZ.
