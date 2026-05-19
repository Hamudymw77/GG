# 1. Co je transakce

Transakce je skupina SQL příkazů, které se provedou jako jeden celek.

Buď se provedou všechny, nebo žádný.

Klasický příklad: bankovní převod peněz.

```
Odečti 1000 Kč z účtu A
Přičti 1000 Kč na účet B
```

Pokud uprostřed dojde k chybě, nesmí se peníze odečíst a nepřičíst.

---

# 2. ACID vlastnosti

Transakce musí splňovat čtyři vlastnosti, označované zkratkou ACID.

## Atomicity (Atomičnost)

Transakce je nedělitelná. Buď se provede celá, nebo vůbec.

---

## Consistency (Konzistence)

Po dokončení transakce musí být databáze v konzistentním stavu. Všechna integritní omezení jsou splněna.

---

## Isolation (Izolace)

Probíhající transakce se navzájem neovlivňují. Jedna transakce nevidí rozpracované změny jiné.

---

## Durability (Trvanlivost)

Po COMMIT jsou data trvale uložena, i když dojde k výpadku proudu nebo pádu serveru.

InnoDB to zajišťuje pomocí Redo Log.

---

# 3. TCL příkazy

```sql
START TRANSACTION;   -- nebo BEGIN

-- SQL příkazy...

COMMIT;              -- potvrzení, uloží změny
-- nebo
ROLLBACK;            -- zrušení, vrátí původní stav
```

---

# 4. SAVEPOINT

SAVEPOINT umožní vrátit se na konkrétní bod v transakci, ne na začátek.

```sql
START TRANSACTION;

INSERT INTO ucet SET zůstatek = 5000 WHERE id = 1;

SAVEPOINT pred_prevodem;

UPDATE ucet SET zustatek = zustatek - 1000 WHERE id = 1;

-- Něco se pokazilo, vrátíme se na savepoint
ROLLBACK TO pred_prevodem;

COMMIT;
```

---

# 5. Autocommit

MySQL má ve výchozím nastavení autocommit = 1.

To znamená, že každý příkaz se automaticky commitne.

Při `START TRANSACTION` se autocommit dočasně vypne pro danou transakci.

```sql
SHOW VARIABLES LIKE 'autocommit';
SET autocommit = 0;  -- ruční commit je nutný
```

---

# 6. Úrovně izolace

Úroveň izolace definuje, co vidí transakce ve změnách jiných probíhajících transakcí.

MySQL má čtyři úrovně (od nejméně po nejpřísnější):

## READ UNCOMMITTED

Transakce vidí i nezcommitnuté změny jiné transakce.

Problém: Dirty Read – vidíme data, která může druhá transakce ještě odvolat.

---

## READ COMMITTED

Transakce vidí jen commitnuté změny.

Problém: Non-repeatable Read – stejný SELECT v jedné transakci může vrátit jiný výsledek, pokud jiná transakce mezitím commitne.

---

## REPEATABLE READ (výchozí v MySQL)

Transakce vždy vidí stejná data jako na začátku transakce.

Problém: Phantom Read – jiná transakce může vložit nové řádky, které se pak v podmínce vyskytnou.

---

## SERIALIZABLE

Nejpřísnější. Transakce se chovají, jako by se prováděly sériově.

Žádné problémy, ale nejpomalejší.

---

# 7. Deadlock

Deadlock nastane, když dvě transakce čekají navzájem na uvolnění zámku.

Transakce A čeká na řádek, který zamkla transakce B.

Transakce B čeká na řádek, který zamkla transakce A.

MySQL deadlock automaticky detekuje a jednu transakci vrátí (ROLLBACK).

---

# Shrnutí

Transakce = nedělitelná skupina příkazů.

ACID: Atomicity, Consistency, Isolation, Durability.

TCL: START TRANSACTION, COMMIT, ROLLBACK, SAVEPOINT.

Úrovně izolace: READ UNCOMMITTED → READ COMMITTED → REPEATABLE READ → SERIALIZABLE.

Výchozí v MySQL: REPEATABLE READ.

Deadlock: MySQL ho detekuje a automaticky řeší ROLLBACKem jedné transakce.

---

# Typické doplňující otázky

## Co je ACID?

Čtyři vlastnosti transakce: Atomicity (celá nebo nic), Consistency (konzistentní stav), Isolation (izolace od jiných transakcí), Durability (trvalost po COMMIT).

---

## Co je Dirty Read?

Situace, kdy transakce přečte data, která jiná transakce ještě nezcommitovala. Může se stát při úrovni READ UNCOMMITTED.

---

## Jaká je výchozí úroveň izolace v MySQL?

REPEATABLE READ.

---

## Co je deadlock?

Dvě transakce se navzájem blokují. MySQL to detekuje a jednu z nich odvolá.
