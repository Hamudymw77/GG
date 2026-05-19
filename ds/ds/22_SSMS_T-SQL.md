# 1. Co je Microsoft SQL Server

Microsoft SQL Server je relační DBMS od Microsoftu.

Používá dialekt SQL nazvaný T-SQL (Transact-SQL).

Nástroj pro správu a vývoj: SSMS (SQL Server Management Studio).

---

# 2. SQL Server Management Studio (SSMS)

SSMS je bezplatná desktopová aplikace pro Windows.

Umožňuje:

- Připojení k SQL Server instancím (lokálním i vzdáleným)
- Psaní a spouštění T-SQL dotazů
- Správu databází, tabulek, indexů, uložených procedur
- Import a export dat
- Správu záloh
- Monitorování výkonu (Activity Monitor)

---

# 3. T-SQL – specifika oproti MySQL

| Funkce | MySQL | T-SQL (SQL Server) |
|--------|-------|-------------------|
| Omezení výsledku | LIMIT 10 | SELECT TOP 10 |
| Auto-increment | AUTO_INCREMENT | IDENTITY(1,1) |
| Oddělovač bloků | DELIMITER // | GO |
| Datový typ text | VARCHAR | NVARCHAR (Unicode) |
| Aktuální čas | NOW() | GETDATE() |
| NULL náhrada | IFNULL(x, y) | ISNULL(x, y) |
| Řetězcové spojení | CONCAT() | STRING_AGG(), + |
| Podmíněný výraz | IF() | CASE WHEN |
| Systémové tabulky | information_schema | sys.tables, sys.columns |

---

# 4. TOP místo LIMIT

```sql
-- MySQL
SELECT * FROM zakaznik LIMIT 10;

-- T-SQL
SELECT TOP 10 * FROM zakaznik;

-- T-SQL s procentem
SELECT TOP 10 PERCENT * FROM zakaznik;
```

---

# 5. IDENTITY místo AUTO_INCREMENT

```sql
CREATE TABLE produkt (
    id    INT IDENTITY(1,1) PRIMARY KEY,
    nazev NVARCHAR(200) NOT NULL,
    cena  DECIMAL(10,2)
);
```

IDENTITY(1,1): začíná od 1, přičítá 1.

---

# 6. GO

GO je oddělovač dávek (batch) v T-SQL.

Slouží k oddělení bloků kódu při spouštění v SSMS.

```sql
CREATE TABLE test (id INT);
GO

INSERT INTO test VALUES (1);
GO
```

---

# 7. TRY / CATCH

T-SQL má vestavěné zpracování chyb pomocí TRY/CATCH.

```sql
BEGIN TRY
    BEGIN TRANSACTION;
    UPDATE ucet SET zustatek = zustatek - 1000 WHERE id = 1;
    UPDATE ucet SET zustatek = zustatek + 1000 WHERE id = 2;
    COMMIT;
END TRY
BEGIN CATCH
    ROLLBACK;
    SELECT ERROR_MESSAGE() AS chyba;
END CATCH;
```

---

# 8. Systémový katalog v SQL Server

```sql
-- Tabulky v databázi
SELECT name, type_desc FROM sys.tables;

-- Sloupce tabulky
SELECT name, system_type_id FROM sys.columns WHERE object_id = OBJECT_ID('zakaznik');

-- Uložené procedury
SELECT name FROM sys.procedures;

-- SQL Server Agent: plánování úloh (zálohy, ETL)
-- Nastavuje se přes SSMS → SQL Server Agent
```

---

# 9. SQL Server Agent

SQL Server Agent je plánovač úloh vestavěný v SQL Serveru.

Slouží pro:

- Automatické zálohy
- Spouštění ETL procesů
- Pravidelné čištění dat
- Alerting při chybách

---

# Shrnutí

T-SQL: TOP místo LIMIT, IDENTITY místo AUTO_INCREMENT, GO jako oddělovač, NVARCHAR, GETDATE(), ISNULL().

SSMS: nástroj pro správu SQL Serveru (dotazy, zálohy, monitoring).

TRY/CATCH: zpracování chyb v T-SQL.

sys.tables / sys.columns: systémový katalog v SQL Serveru.

SQL Server Agent: plánovač úloh.

---

# Typické doplňující otázky

## Jaký je rozdíl mezi T-SQL a MySQL?

T-SQL používá TOP místo LIMIT, IDENTITY místo AUTO_INCREMENT, NVARCHAR místo VARCHAR, GO jako oddělovač, sys.* místo information_schema pro katalog.

---

## Co je SSMS?

SQL Server Management Studio – bezplatná aplikace od Microsoftu pro správu SQL Serveru.

---

## Co je SQL Server Agent?

Plánovač úloh v SQL Serveru. Spouští zálohy, ETL procesy a alerting automaticky podle rozvrhu.

---

## Co je IDENTITY v T-SQL?

Automaticky inkrementovaný sloupec. Ekvivalent AUTO_INCREMENT v MySQL. IDENTITY(1,1) = začíná od 1, přičítá 1.
