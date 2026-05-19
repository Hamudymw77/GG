# 1. Co je export a import dat

Export dat znamená uložení dat z databáze do souboru.

Import dat znamená načtení dat ze souboru do databáze.

Používá se pro zálohy, přenos dat mezi systémy nebo sdílení dat.

---

# 2. Formáty exportovaných dat

## SQL dump

Soubor obsahuje SQL příkazy (CREATE TABLE, INSERT).

Lze ho znovu spustit a obnovit databázi přesně.

Přípona: `.sql`

---

## CSV (Comma-Separated Values)

Textový soubor, každý řádek = jeden záznam, hodnoty odděleny čárkou.

```
id,jmeno,email
1,Pavel,pavel@t.cz
2,Eva,eva@t.cz
```

Výhody: jednoduchý, čitelný, Excel ho umí otevřít.

Nevýhody: žádná struktura, žádné datové typy.

---

## JSON

Datový formát používaný hlavně pro webové aplikace.

```json
[
  {"id": 1, "jmeno": "Pavel", "email": "pavel@t.cz"},
  {"id": 2, "jmeno": "Eva",   "email": "eva@t.cz"}
]
```

---

## XML

Starší formát, hierarchická struktura.

```xml
<zakaznici>
  <zakaznik><id>1</id><jmeno>Pavel</jmeno></zakaznik>
</zakaznici>
```

---

# 3. Export v MySQL

## SELECT INTO OUTFILE

Vyexportuje výsledek dotazu do souboru na serveru.

```sql
SELECT * FROM zakaznik
INTO OUTFILE '/tmp/zakaznici.csv'
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n';
```

Soubor se uloží na server, ne na klientský počítač.

---

## mysqldump (v terminálu)

Vytvoří zálohu celé databáze jako SQL soubor.

```
mysqldump -u root -p mojeDB > export.sql
```

Nebo jen konkrétní tabulku:

```
mysqldump -u root -p mojeDB zakaznik > export_zakaznik.sql
```

---

# 4. Import v MySQL

## LOAD DATA INFILE

Načte CSV soubor do tabulky.

```sql
LOAD DATA INFILE '/tmp/zakaznici.csv'
INTO TABLE zakaznik
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;  -- přeskočí hlavičku
```

---

## Spuštění SQL souboru (v terminálu)

```
mysql -u root -p mojeDB < export.sql
```

---

# 5. Import přes INSERT INTO ... SELECT

Data lze přesouvat mezi tabulkami nebo databázemi pomocí SQL.

```sql
-- Přesun dat ze staging tabulky do produkční
INSERT INTO zakaznik (jmeno, email)
SELECT jmeno, email FROM staging_zakaznik
WHERE email IS NOT NULL;
```

---

# 6. Kdy použít jaký formát

Pro zálohu databáze → SQL dump (mysqldump).

Pro sdílení dat s Excelem → CSV.

Pro webové API → JSON.

Pro starší systémy nebo dokumenty → XML.

Pro přenos dat mezi databázemi → SQL nebo CSV.

---

# 7. MySQL Workbench – export/import přes GUI

MySQL Workbench nabízí vizuální průvodce pro export a import.

Export: Server → Data Export → vybrat databázi → Export to Self-Contained File.

Import: Server → Data Import → načíst SQL soubor → Start Import.

---

# Shrnutí

Export = uložení dat z databáze do souboru.

Import = načtení dat ze souboru do databáze.

Formáty: SQL, CSV, JSON, XML.

mysqldump = záloha jako SQL soubor.

SELECT INTO OUTFILE = export do CSV na serveru.

LOAD DATA INFILE = import z CSV na serveru.

---

# Typické doplňující otázky

## Jaký je rozdíl mezi CSV a SQL exportem?

CSV obsahuje pouze data bez struktury.
SQL export obsahuje příkazy pro vytvoření tabulek i vložení dat.

---

## Co dělá mysqldump?

Vytváří SQL soubor se strukturou a daty databáze, který lze použít pro obnovu.

---

## Co dělá LOAD DATA INFILE?

Načte CSV soubor ze serveru do databázové tabulky. Je to rychlejší než INSERT.

---

## Proč exportovat do JSON?

JSON je standardní formát pro webové API a aplikace. Snadno se zpracovává v JavaScriptu.
