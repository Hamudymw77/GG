# 1. Co je MySQL Workbench

MySQL Workbench je bezplatný grafický nástroj od Oracle pro práci s MySQL databázemi.

Nabízí tři hlavní oblasti: SQL vývoj, datové modelování a správu serveru.

---

# 2. Komponenty MySQL Workbench

## SQL Editor

Textový editor pro psaní a spouštění SQL dotazů.

Zvýraznění syntaxe, automatické doplňování.

Výsledky se zobrazují v přehledné tabulce.

Historie dotazů.

---

## Model Editor (EER Diagram)

Grafický editor pro návrh databázového schématu.

EER = Enhanced Entity-Relationship diagram.

Lze kreslit tabulky, sloupce, vztahy a automaticky vygenerovat SQL (forward engineering).

Lze také importovat existující databázi a vygenerovat diagram (reverse engineering).

---

## Server Administration

Sekce pro správu MySQL serveru.

Obsahuje:

- User Management – správa uživatelů a práv
- Server Status – stav serveru, statistiky
- Data Import / Export – zálohy a obnovení

---

# 3. Vizuální EXPLAIN

MySQL Workbench umí zobrazit EXPLAIN plán dotazu graficky.

Ukáže, které kroky jsou pomalé (červené = problém) a kde chybí index.

---

# 4. Performance Dashboard

Zobrazuje metriky výkonu serveru v reálném čase:

- Počet dotazů za sekundu
- Využití Buffer Pool (InnoDB)
- Aktivní spojení
- Pomalé dotazy

---

# 5. Příkazy pro monitoring serveru

## SHOW STATUS

Statistiky serveru od posledního startu.

```sql
SHOW STATUS LIKE 'Questions';          -- počet provedených dotazů
SHOW STATUS LIKE 'Threads_connected';  -- aktuální počet připojení
SHOW STATUS LIKE 'Innodb_buffer_pool_reads';  -- čtení z disku (ideálně nízké)
```

---

## SHOW VARIABLES

Konfigurační proměnné serveru.

```sql
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';  -- velikost buffer pool
SHOW VARIABLES LIKE 'max_connections';           -- max počet připojení
SHOW VARIABLES LIKE 'version';                   -- verze MySQL
SHOW VARIABLES LIKE 'datadir';                   -- kde jsou uložena data
```

---

## SHOW PROCESSLIST

Aktuálně běžící dotazy a připojení.

```sql
SHOW PROCESSLIST;
```

---

## PERFORMANCE_SCHEMA

Systémová databáze pro detailní monitoring výkonu.

```sql
SELECT * FROM performance_schema.events_statements_summary_by_digest
ORDER BY sum_timer_wait DESC LIMIT 10;
```

---

# 6. Forward a Reverse Engineering

Forward Engineering: z EER diagramu vygeneruje SQL skript pro vytvoření databáze.

Reverse Engineering: z existující databáze vygeneruje EER diagram.

Database → Forward Engineer / Reverse Engineer v menu Workbench.

---

# 7. Import a export v Workbench

Data Import/Export: Server → Data Import nebo Data Export.

Nabízí:

- Export celé databáze jako SQL dump
- Export vybraných tabulek
- Import SQL souboru nebo dumpem

---

# Shrnutí

MySQL Workbench: SQL Editor + Model Editor + Server Administration.

EER Diagram: grafický návrh databáze, forward/reverse engineering.

SHOW STATUS: statistiky serveru.

SHOW VARIABLES: konfigurace serveru.

SHOW PROCESSLIST: aktuální připojení a dotazy.

PERFORMANCE_SCHEMA: detailní monitoring výkonu.

---

# Typické doplňující otázky

## Co je EER diagram?

Enhanced Entity-Relationship diagram – grafická reprezentace databázového schématu. V Workbench ho lze nakreslit a vygenerovat z něj SQL.

---

## Co je forward engineering?

Proces, při kterém se z EER diagramu (nebo modelu) automaticky vygeneruje SQL skript pro vytvoření databáze.

---

## Co zobrazuje SHOW PROCESSLIST?

Seznam aktuálních připojení a dotazů na serveru. Pomáhá identifikovat zaseknuté nebo pomalé dotazy.

---

## Co je Performance Schema?

Systémová databáze MySQL pro detailní sledování výkonu – pomalé dotazy, čekací doby, využití zdrojů.
