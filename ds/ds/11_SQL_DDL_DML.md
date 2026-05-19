# 1. Co je SQL

SQL (Structured Query Language) je jazyk pro práci s relačními databázemi.

Pomocí SQL vytváříme tabulky, vkládáme data, vyhledáváme, měníme a mažeme.

SQL se dělí do skupin podle toho, co dělá.

---

# 2. Skupiny SQL příkazů

## DDL – Data Definition Language

Příkazy pro práci se strukturou databáze (tabulky, sloupce).

CREATE – vytvoření tabulky nebo databáze.

ALTER – změna existující tabulky.

DROP – smazání tabulky nebo databáze.

TRUNCATE – smazání všech dat z tabulky.

---

## DML – Data Manipulation Language

Příkazy pro práci s daty.

INSERT – vložení nového záznamu.

SELECT – výběr dat.

UPDATE – změna existujících dat.

DELETE – smazání dat.

---

## DCL – Data Control Language

Příkazy pro správu přístupových práv.

GRANT – přidání práv uživateli.

REVOKE – odebrání práv.

---

## TCL – Transaction Control Language

Příkazy pro řízení transakcí.

BEGIN / START TRANSACTION – začátek transakce.

COMMIT – potvrzení transakce.

ROLLBACK – zrušení transakce.

---

# 3. DDL příkazy

## CREATE TABLE

```sql
CREATE TABLE zakaznik (
    id    INT AUTO_INCREMENT PRIMARY KEY,
    jmeno VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE
);
```

---

## ALTER TABLE

Přidání sloupce:

```sql
ALTER TABLE zakaznik ADD COLUMN telefon VARCHAR(20);
```

Změna datového typu:

```sql
ALTER TABLE zakaznik MODIFY COLUMN jmeno VARCHAR(200);
```

Přejmenování sloupce:

```sql
ALTER TABLE zakaznik RENAME COLUMN telefon TO tel;
```

Smazání sloupce:

```sql
ALTER TABLE zakaznik DROP COLUMN tel;
```

---

## DROP TABLE

```sql
DROP TABLE IF EXISTS zakaznik;
```

---

# 4. SELECT – výběr dat

Základní syntaxe:

```sql
SELECT sloupce FROM tabulka WHERE podmínka ORDER BY sloupec LIMIT n;
```

Pořadí klíčových slov ve SELECT:

SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY → LIMIT

---

## Příklady

Výběr všech zákazníků:

```sql
SELECT * FROM zakaznik;
```

Výběr konkrétních sloupců:

```sql
SELECT jmeno, email FROM zakaznik;
```

Filtrování podmínkou:

```sql
SELECT * FROM zakaznik WHERE mesto = 'Praha';
```

Seřazení:

```sql
SELECT * FROM zakaznik ORDER BY jmeno ASC;
```

---

# 5. JOIN – spojení tabulek

JOIN spojí dva dotazy do jednoho výsledku na základě podmínky.

## INNER JOIN

Vrátí jen zákazníky, kteří mají alespoň jednu objednávku.

```sql
SELECT z.jmeno, o.datum, o.celkem
FROM zakaznik z
INNER JOIN objednavka o ON o.zakaznik_id = z.id;
```

---

## LEFT JOIN

Vrátí všechny zákazníky, i ty bez objednávky.

```sql
SELECT z.jmeno, o.datum
FROM zakaznik z
LEFT JOIN objednavka o ON o.zakaznik_id = z.id;
```

Zákazník bez objednávky bude mít v sloupci `datum` hodnotu NULL.

---

## RIGHT JOIN

Vrátí všechny záznamy z pravé tabulky, i ty bez shody v levé.

---

## CROSS JOIN

Vrátí kartézský součin – každý řádek s každým. Bez podmínky.

Pozor: 10 × 10 řádků = 100 řádků výsledku.

---

# 6. GROUP BY a HAVING

GROUP BY seskupí řádky podle hodnoty sloupce.

```sql
SELECT mesto, COUNT(*) AS pocet
FROM zakaznik
GROUP BY mesto;
```

HAVING filtruje skupiny (podobně jako WHERE, ale pro skupiny):

```sql
SELECT mesto, COUNT(*) AS pocet
FROM zakaznik
GROUP BY mesto
HAVING pocet > 2;
```

WHERE filtruje řádky před seskupením, HAVING filtruje skupiny po seskupení.

---

# 7. VIEW (pohled)

VIEW je uložený SELECT dotaz. Chová se jako tabulka, ale data neukládá.

```sql
CREATE VIEW prehled_zakazniku AS
SELECT z.jmeno, COUNT(o.id) AS pocet_objednavek
FROM zakaznik z
LEFT JOIN objednavka o ON o.zakaznik_id = z.id
GROUP BY z.id;

SELECT * FROM prehled_zakazniku;
```

---

# Shrnutí

DDL: CREATE, ALTER, DROP, TRUNCATE – struktura.

DML: INSERT, SELECT, UPDATE, DELETE – data.

DCL: GRANT, REVOKE – práva.

TCL: BEGIN, COMMIT, ROLLBACK – transakce.

JOIN: propojení tabulek (INNER = shoda, LEFT = všichni z levé).

GROUP BY + HAVING: agregace dat.

VIEW: uložený dotaz jako virtuální tabulka.

---

# Typické doplňující otázky

## Jaký je rozdíl mezi WHERE a HAVING?

WHERE filtruje řádky před GROUP BY. HAVING filtruje skupiny po GROUP BY.

---

## Jaký je rozdíl mezi INNER JOIN a LEFT JOIN?

INNER JOIN: jen záznamy, které mají shodu v obou tabulkách.
LEFT JOIN: všechny záznamy z levé tabulky, u neshody NULL z pravé.

---

## Co je VIEW?

Uložený SELECT dotaz. Vypadá jako tabulka, ale data neukládá fyzicky.

---

## Jaký je rozdíl mezi DELETE a TRUNCATE?

DELETE maže konkrétní řádky (lze s WHERE), vrátí se zpět. TRUNCATE maže vše, nelze vrátit.
