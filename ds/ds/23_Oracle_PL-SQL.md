# 1. Co je Oracle SQL Developer

Oracle SQL Developer je bezplatný nástroj od Oracle Corporation.

Používá se pro správu a vývoj Oracle databází.

Podporuje psaní SQL dotazů, správu objektů, ladění PL/SQL kódu, import/export dat.

---

# 2. Oracle vs MySQL – hlavní rozdíly

| Funkce | MySQL | Oracle |
|--------|-------|--------|
| Automatický klíč | AUTO_INCREMENT | SEQUENCE + TRIGGER nebo IDENTITY (12c+) |
| Omezení výsledku | LIMIT | ROWNUM (starší) nebo FETCH FIRST n ROWS (12c+) |
| NULL náhrada | IFNULL(x, y) | NVL(x, y) |
| Podmíněná logika | IF() | DECODE() nebo CASE WHEN |
| Aktuální datum | NOW() | SYSDATE |
| Datový typ text | VARCHAR | VARCHAR2 |
| Datový typ číslo | INT, DECIMAL | NUMBER(p, s) |
| Prázdná tabulka | – | DUAL (pseudotabulka) |
| Systémové pohledy | information_schema | USER_TABLES, USER_COLUMNS, ALL_TABLES |
| Bloky kódu | – | PL/SQL (anonymní blok) |

---

# 3. DUAL – pseudotabulka

Oracle vyžaduje FROM klauzuli v každém SELECT.

DUAL je pseudotabulka pro výrazy bez skutečné tabulky.

```sql
SELECT SYSDATE FROM DUAL;
SELECT 1 + 1 AS vysledek FROM DUAL;
SELECT 'Ahoj' FROM DUAL;
```

---

# 4. SEQUENCE – automatické klíče

Oracle nemá AUTO_INCREMENT.

Místo toho se používá SEQUENCE – objekt generující pořadová čísla.

```sql
CREATE SEQUENCE seq_zakaznik
    START WITH 1
    INCREMENT BY 1
    NOCACHE;

-- Použití
INSERT INTO zakaznik (id, jmeno) VALUES (seq_zakaznik.NEXTVAL, 'Pavel');

-- Aktuální hodnota (po alespoň jednom NEXTVAL)
SELECT seq_zakaznik.CURRVAL FROM DUAL;
```

---

# 5. NVL – náhrada NULL

```sql
-- MySQL:
SELECT IFNULL(email, 'nezadán') FROM zakaznik;

-- Oracle:
SELECT NVL(email, 'nezadán') FROM zakaznik;
```

---

# 6. ROWNUM – omezení výsledku

```sql
-- MySQL:
SELECT * FROM zakaznik LIMIT 5;

-- Oracle (starší syntaxe):
SELECT * FROM zakaznik WHERE ROWNUM <= 5;

-- Oracle 12c+ (doporučeno):
SELECT * FROM zakaznik FETCH FIRST 5 ROWS ONLY;
```

---

# 7. PL/SQL – anonymní blok

PL/SQL je procedurální rozšíření Oracle SQL.

Anonymní blok se nespouští jako procedura – spustí se okamžitě.

```sql
BEGIN
    DBMS_OUTPUT.PUT_LINE('Ahoj z PL/SQL');
END;
/
```

Blok má strukturu: `DECLARE` (volitelně) → `BEGIN` → `EXCEPTION` (volitelně) → `END;`

---

## PL/SQL s proměnnými

```sql
DECLARE
    v_jmeno VARCHAR2(100);
    v_plat  NUMBER(10,2);
BEGIN
    SELECT jmeno, plat INTO v_jmeno, v_plat
    FROM zamestnanec WHERE id = 1;

    DBMS_OUTPUT.PUT_LINE('Jméno: ' || v_jmeno || ', Plat: ' || v_plat);
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Záznam nenalezen');
END;
/
```

---

# 8. Systémové pohledy Oracle

```sql
-- Tabulky aktuálního uživatele
SELECT table_name FROM USER_TABLES;

-- Všechny přístupné tabulky
SELECT owner, table_name FROM ALL_TABLES WHERE owner = 'UZIVATEL';

-- Sloupce tabulky
SELECT column_name, data_type, nullable FROM USER_TAB_COLUMNS WHERE table_name = 'ZAKAZNIK';

-- Procedury a funkce
SELECT object_name, object_type FROM USER_OBJECTS WHERE object_type IN ('PROCEDURE', 'FUNCTION');
```

---

# Shrnutí

Oracle SQL Developer: nástroj pro správu Oracle databází.

DUAL: pseudotabulka pro SELECT bez tabulky.

SEQUENCE: generátor čísel, ekvivalent AUTO_INCREMENT.

NVL: náhrada NULL (IFNULL v MySQL).

ROWNUM / FETCH FIRST: omezení počtu řádků (LIMIT v MySQL).

PL/SQL: procedurální jazyk Oracle (BEGIN...END;/).

USER_TABLES: systémové pohledy Oracle (information_schema ekvivalent).

---

# Typické doplňující otázky

## Co je DUAL?

Pseudotabulka v Oracle. Slouží pro SELECT výrazy, kde nepotřebujeme skutečnou tabulku (SELECT SYSDATE FROM DUAL).

---

## Co je SEQUENCE v Oracle?

Objekt Oracle, který generuje pořadová čísla. Ekvivalent AUTO_INCREMENT v MySQL.

---

## Co je PL/SQL?

Procedurální rozšíření Oracle SQL. Umožňuje psaní bloků kódu s proměnnými, podmínkami a výjimkami.

---

## Jaký je ekvivalent IFNULL v Oracle?

NVL(hodnota, nahradi_NULL).
