# 1. Co jsou datové typy

Datový typ určuje, jaká data může sloupec obsahovat.

Správná volba datového typu zajistí datovou integritu, úsporné ukládání a dobrý výkon.

---

# 2. Numerické typy

## Celá čísla

| Typ | Rozsah | Místo |
|-----|--------|-------|
| TINYINT | -128 až 127 (nebo 0–255 UNSIGNED) | 1 byte |
| SMALLINT | -32 768 až 32 767 | 2 byty |
| MEDIUMINT | -8 388 608 až 8 388 607 | 3 byty |
| INT | -2 147 483 648 až 2 147 483 647 | 4 byty |
| BIGINT | velmi velká celá čísla | 8 bytů |

Příklad: `vek TINYINT UNSIGNED` (rozsah 0–255).

---

## Desetinná čísla

DECIMAL(p, s): přesná desetinná čísla. p = počet číslic celkem, s = počet číslic za desetinnou čárkou.

Použití: finanční data (cena, tržba).

```sql
cena DECIMAL(10, 2)  -- max 99999999.99
```

FLOAT / DOUBLE: aproximativní desetinná čísla.

Nevhodné pro finanční data kvůli zaokrouhlovacím chybám.

---

# 3. Textové typy

## CHAR(n)

Pevná délka. Vždy zabírá n znaků.

Vhodné pro data pevné délky: PSČ, kód státu.

```sql
psc CHAR(5)
```

---

## VARCHAR(n)

Proměnná délka. Zabírá jen tolik místa, kolik je potřeba.

Vhodné pro jména, emaily, adresy.

```sql
jmeno VARCHAR(100)
email VARCHAR(150)
```

---

## TEXT

Proměnná délka, až 65 535 znaků.

Nelze indexovat celý sloupec (jen prefix).

Vhodné pro delší texty (popis produktu, komentáře).

Varianty: TINYTEXT (255), TEXT (65535), MEDIUMTEXT (16MB), LONGTEXT (4GB).

---

# 4. Datumové a časové typy

| Typ | Formát | Rozsah |
|-----|--------|--------|
| DATE | YYYY-MM-DD | 1000-01-01 až 9999-12-31 |
| TIME | HH:MM:SS | -838:59:59 až 838:59:59 |
| DATETIME | YYYY-MM-DD HH:MM:SS | 1000-01-01 až 9999-12-31 |
| TIMESTAMP | YYYY-MM-DD HH:MM:SS | 1970-01-01 až 2038-01-19 |
| YEAR | YYYY | 1901 až 2155 |

TIMESTAMP: ukládá se v UTC, zobrazuje v lokálním čase. Automatická aktualizace.

DATETIME: ukládá se přesně tak, jak bylo zadáno.

---

# 5. Logický typ

MySQL nemá skutečný BOOLEAN – používá TINYINT(1).

0 = false, 1 = true.

```sql
aktivni BOOLEAN DEFAULT TRUE
```

---

# 6. Binární typy

BLOB (Binary Large Object): pro soubory, obrázky.

Varianty: TINYBLOB, BLOB, MEDIUMBLOB, LONGBLOB.

Poznámka: ukládání souborů do databáze se nedoporučuje – lepší je ukládat cestu k souboru.

---

# 7. JSON typ (MySQL 5.7+)

Nativní podpora JSON pro ukládání strukturovaných dat.

```sql
metadata JSON

-- Dotaz na JSON hodnotu
SELECT JSON_EXTRACT(metadata, '$.barva') FROM produkt;
-- nebo zkrácený zápis
SELECT metadata->>'$.barva' FROM produkt;
```

---

# 8. Rozdíly mezi databázemi

| Typ | MySQL | Oracle | SQL Server |
|-----|-------|--------|-----------|
| Řetězec | VARCHAR | VARCHAR2 | NVARCHAR |
| Celé číslo | INT | NUMBER(10) | INT |
| Desetinné | DECIMAL | NUMBER(p,s) | DECIMAL |
| Datum+čas | DATETIME | DATE (obsahuje čas) | DATETIME2 |
| Boolean | TINYINT(1) | NUMBER(1) / CHAR(1) | BIT |
| Velký text | LONGTEXT | CLOB | NVARCHAR(MAX) |
| Binární | LONGBLOB | BLOB | VARBINARY(MAX) |
| JSON | JSON | JSON (21c+) | NVARCHAR + JSON_VALUE |
| Auto-klíč | AUTO_INCREMENT | SEQUENCE | IDENTITY |

---

# 9. CAST a CONVERT

Převod datových typů:

```sql
SELECT CAST('2024-01-15' AS DATE);
SELECT CAST(cena AS CHAR);
SELECT CONVERT(cena, CHAR);
SELECT CAST(NULL AS DECIMAL(10,2));
```

---

# Shrnutí

Numerické: INT (celé), DECIMAL (přesné desetinné), FLOAT (aproximativní).

Textové: CHAR (pevná), VARCHAR (proměnná), TEXT (dlouhý text).

Datumové: DATE, TIME, DATETIME, TIMESTAMP.

Boolean: TINYINT(1) v MySQL.

JSON: nativní typ v MySQL 5.7+.

Rozdíly: VARCHAR2/Oracle, NVARCHAR/SQL Server, NUMBER/Oracle.

---

# Typické doplňující otázky

## Jaký je rozdíl mezi CHAR a VARCHAR?

CHAR má pevnou délku, vždy zabírá n znaků. VARCHAR má proměnnou délku, zabírá jen tolik, kolik je potřeba.

---

## Proč používat DECIMAL místo FLOAT pro peníze?

FLOAT je aproximativní – může způsobit zaokrouhlovací chyby. DECIMAL ukládá přesné hodnoty.

---

## Jaký je rozdíl mezi DATETIME a TIMESTAMP?

TIMESTAMP se ukládá v UTC a je omezen do roku 2038. DATETIME ukládá datum přesně jak bylo zadáno bez konverze časového pásma.

---

## Co je JSON typ v MySQL?

Nativní datový typ pro ukládání JSON objektů. Umožňuje dotazování na konkrétní klíče pomocí JSON_EXTRACT nebo -> operátoru.
