# 1. Co jsou etapy vývoje databázového systému

Databázový systém se nevytváří najednou. Prochází několika fázemi od prvního nápadu až po provoz a údržbu.

Každá fáze má svůj cíl a výstupy.

---

# 2. Fáze vývoje

## 1. Analýza požadavků

V této fázi zjišťujeme, co databáze musí umět.

Mluvíme s uživateli a zákazníky. Zapisujeme jejich požadavky.

Výstup: dokument s požadavky – co se má ukládat, kdo to bude používat.

---

## 2. Konceptuální modelování

Vytvoříme ER diagram – grafické znázornění entit a jejich vztahů.

Nezabýváme se technickými detaily. Jen co a jak je propojeno.

Výstup: ER diagram.

---

## 3. Logické modelování

Z ER diagramu vytvoříme logický model.

Přidáme atributy, primární a cizí klíče, typy vztahů.

Stále není závislé na konkrétní databázi.

Výstup: logický datový model.

---

## 4. Fyzické modelování

Logický model převedeme do konkrétní databáze.

Definujeme datové typy, indexy, partitioning.

Nástroje: Oracle Data Modeler, MySQL Workbench.

Výstup: fyzický model a SQL script.

---

## 5. Implementace

Spustíme SQL script a vytvoříme databázi.

Vložíme testovací data. Propojíme s aplikací.

Výstup: funkční databáze.

---

## 6. Testování

Testujeme, zda databáze funguje správně.

Ověřujeme integritní omezení, výkon dotazů, bezpečnost.

Výstup: testovací protokol.

---

## 7. Provoz a údržba

Databáze běží v produkci.

Provádíme zálohy, sledujeme výkon, opravujeme problémy.

Přidáváme nové funkce.

---

## 8. Archivace nebo likvidace

Data, která se aktivně nepoužívají, archivujeme.

Na konci životního cyklu databázi uzavřeme nebo migrujeme do nového systému.

---

# 3. Datový slovník

Datový slovník je dokument nebo tabulka, která popisuje každý sloupec v databázi.

Co sloupec znamená, jaký má datový typ, zda je povinný.

Příklad datového slovníku:

| Tabulka | Sloupec | Typ | Povinný | Popis |
|---------|---------|-----|---------|-------|
| zakaznik | id | INT | Ano | Unikátní ID zákazníka |
| zakaznik | jmeno | VARCHAR | Ano | Celé jméno zákazníka |
| zakaznik | email | VARCHAR | Ne | E-mailová adresa |
| objednavka | datum | DATE | Ano | Datum vytvoření objednávky |

---

# 4. Migrace schématu

Když se databáze mění (přidáme sloupec, přejmenujeme tabulku), mluvíme o migraci schématu.

Migrace se dělá příkazem ALTER TABLE.

Příklad: přidáme sloupec telefon do tabulky zákazník:

```sql
ALTER TABLE zakaznik ADD COLUMN telefon VARCHAR(20);
```

---

# 5. Verzování databáze

Schéma databáze se mění v čase stejně jako kód aplikace.

Verzujeme ho podobně jako kód – každá změna má číslo verze.

Nástroje pro verzování databáze: Flyway, Liquibase.

---

# Shrnutí

Vývoj databáze prochází fázemi: analýza → konceptuální model → logický model → fyzický model → implementace → testování → provoz → archivace.

Datový slovník dokumentuje každý sloupec.

Migrace schématu = úprava existující databáze (ALTER TABLE).

Verzování databáze = sledování změn schématu v čase.

---

# Typické doplňující otázky

## Co je konceptuální model?

Grafické znázornění entit a vztahů bez technických detailů. Výstupem je ER diagram.

---

## Jaký je rozdíl mezi logickým a fyzickým modelem?

Logický = nezávislý na DBMS, popisuje co a jak.
Fyzický = konkrétní pro daný DBMS, obsahuje datové typy a indexy.

---

## Co je migrace schématu?

Změna struktury existující databáze. Provádí se příkazem ALTER TABLE.

---

## Co je datový slovník?

Dokumentace každého sloupce v databázi – typ, povinnost, popis.
