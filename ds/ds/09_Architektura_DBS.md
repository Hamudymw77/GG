# 1. Co je architektura databázového systému

Architektura databázového systému popisuje, jak je DBMS vnitřně organizován.

Jak zpracovává dotazy, jak spravuje data na disku a jak zajišťuje výkon.

---

# 2. Vrstvy DBMS

## Vrstva připojení

Přijímá dotazy od klientů (aplikace, MySQL Workbench, terminál).

Ověřuje přihlašovací údaje.

Spravuje vlákna (jedno vlákno = jedno připojení).

---

## Vrstva zpracování SQL

Parsuje SQL příkaz – zkontroluje správnost syntaxe.

Optimalizátor dotazů zvolí nejlepší plán provedení dotazu.

Například rozhodne, zda použít index nebo procházet celou tabulku.

Exekutor provede plán a vrátí výsledek.

---

## Vrstva úložiště (Storage Engine)

Stará se o fyzické uložení dat na disku.

V MySQL lze volit různé storage enginy. Nejpoužívanější je InnoDB.

---

# 3. InnoDB – výchozí storage engine

InnoDB je výchozí storage engine v MySQL od verze 5.5.

Podporuje transakce (ACID).

Podporuje cizí klíče.

Používá B-tree indexy.

---

## Klíčové komponenty InnoDB

Buffer Pool: paměť, do které se načítají data z disku. Rychlé čtení z RAM místo disku.

Redo Log: záznamy o provedených změnách. Slouží k obnově po pádu serveru.

Undo Log: záznamy o původních hodnotách. Umožňuje vrátit změny (ROLLBACK).

---

# 4. EXPLAIN – jak databáze provede dotaz

Příkaz EXPLAIN ukáže, jak MySQL provede SELECT dotaz.

```sql
EXPLAIN SELECT * FROM zakaznik WHERE email = 'pavel@t.cz';
```

Důležité sloupce výstupu:

type: způsob přístupu k datům

- ALL = prochází všechny řádky (pomalé, špatné pro velké tabulky)
- ref = používá index (rychlé)
- const = přímý přístup přes primární klíč (nejrychlejší)

rows: kolik řádků musí databáze prozkoumat.

Extra: doplňující informace (Using index, Using filesort...).

---

# 5. Systémový katalog (information_schema)

information_schema je speciální databáze, která obsahuje informace o struktuře databáze.

Lze z ní zjistit:

- jaké tabulky existují
- jaké mají sloupce
- jaké jsou indexy
- jaké jsou cizí klíče

Příklad:

```sql
SELECT TABLE_NAME, TABLE_ROWS
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'mojeDB';
```

---

# 6. Procesy a vlákna

Každé připojení k MySQL běží ve vlastním vlákně.

Příkaz SHOW PROCESSLIST zobrazí všechna aktuální připojení a co dělají.

```sql
SHOW PROCESSLIST;
```

Lze ho použít pro zjištění pomalých dotazů nebo zaseknutých spojení.

---

# 7. Storage enginy v MySQL

```sql
SHOW ENGINES;
```

Nejpoužívanější:

InnoDB: transakce, cizí klíče, výchozí.

MyISAM: starší, bez transakcí, rychlejší pro čtení.

MEMORY: data jen v paměti RAM, ztratí se po restartu.

---

# Shrnutí

DBMS má vrstvy: připojení → zpracování SQL → storage engine.

InnoDB: nejpoužívanější storage engine, podporuje transakce a cizí klíče.

EXPLAIN: ukáže plán provedení dotazu – zda se používá index.

information_schema: systémová databáze s informacemi o struktuře.

SHOW PROCESSLIST: zobrazí aktuální připojení a jejich stav.

---

# Typické doplňující otázky

## Co je storage engine?

Část DBMS zodpovědná za fyzické uložení dat na disku. MySQL podporuje více storage engineů (InnoDB, MyISAM...).

---

## Proč je InnoDB lepší než MyISAM?

InnoDB podporuje transakce, cizí klíče a MVCC (izolace transakcí). MyISAM nic z toho nepodporuje.

---

## Co ukáže EXPLAIN?

Plán provedení dotazu – zda se použije index, kolik řádků se prohledá, jaký je způsob přístupu.

---

## Co je Buffer Pool?

Oblast paměti RAM, do které InnoDB načítá stránky dat z disku. Opakované čtení stejných dat je pak z RAM, ne z disku – mnohem rychlejší.
