# 1. Co jsou vnořené dotazy

Vnořený dotaz (subquery) je SELECT dotaz uvnitř jiného SQL příkazu.

Výsledek vnitřního dotazu se použije ve vnějším dotazu jako hodnota nebo seznam.

---

# 2. Kde lze použít vnořený dotaz

Ve WHERE podmínce.

Ve FROM části (jako dočasná tabulka).

Ve SELECT části (jako sloupec).

Spolu s EXISTS.

---

# 3. Vnořený dotaz ve WHERE

Jednoduchý příklad: najdi zákazníky, kteří mají průměrnou hodnotu objednávky vyšší než celkový průměr.

```sql
SELECT jmeno, email
FROM zakaznik
WHERE id IN (
    SELECT zakaznik_id
    FROM objednavka
    WHERE celkem > (SELECT AVG(celkem) FROM objednavka)
);
```

Databáze nejdříve spočítá průměr, pak najde objednávky nad průměrem, pak vrátí zákazníky.

---

## IN vs = ve WHERE

IN: vrací seznam hodnot.

```sql
WHERE id IN (SELECT zakaznik_id FROM objednavka WHERE celkem > 10000)
```

=: vrací jednu hodnotu (subquery musí vrátit přesně 1 řádek).

```sql
WHERE celkem = (SELECT MAX(celkem) FROM objednavka)
```

---

# 4. EXISTS

EXISTS kontroluje, zda vnitřní dotaz vrátí alespoň jeden řádek.

Nevrátí data, jen TRUE nebo FALSE.

Příklad: zákazníci, kteří mají alespoň jednu objednávku:

```sql
SELECT jmeno FROM zakaznik z
WHERE EXISTS (
    SELECT 1 FROM objednavka o WHERE o.zakaznik_id = z.id
);
```

NOT EXISTS: zákazníci bez jediné objednávky:

```sql
SELECT jmeno FROM zakaznik z
WHERE NOT EXISTS (
    SELECT 1 FROM objednavka o WHERE o.zakaznik_id = z.id
);
```

---

# 5. Vnořený dotaz ve FROM (derived table)

Vnitřní dotaz vytvoří dočasnou tabulku, ze které pak čteme.

Příklad: průměrná tržba zákazníka, kteří mají více než 2 objednávky:

```sql
SELECT AVG(trzba) AS prumer_aktivnich
FROM (
    SELECT zakaznik_id, SUM(celkem) AS trzba
    FROM objednavka
    GROUP BY zakaznik_id
    HAVING COUNT(*) > 2
) AS skupinova_tabulka;
```

Dočasná tabulka musí mít alias (zde `skupinova_tabulka`).

---

# 6. CTE – Common Table Expression (WITH)

CTE je pojmenovaný dočasný výsledek dotazu, definovaný na začátku.

Přehlednější než vnořený dotaz ve FROM.

```sql
WITH trzby_zakazniku AS (
    SELECT zakaznik_id, SUM(celkem) AS trzba
    FROM objednavka
    GROUP BY zakaznik_id
)
SELECT z.jmeno, t.trzba
FROM zakaznik z
JOIN trzby_zakazniku t ON t.zakaznik_id = z.id
WHERE t.trzba > 10000;
```

---

# 7. Rekurzivní CTE (WITH RECURSIVE)

Rekurzivní CTE dokáže procházet hierarchické struktury, jako je strom zaměstnanců.

Skládá se ze dvou částí:

Kotva (anchor): výchozí bod (kořen hierarchie).

Rekurzivní část: přidává podřízené záznamy.

```sql
WITH RECURSIVE podrizeni AS (
    SELECT id, jmeno, manazer_id, 0 AS uroven
    FROM zamestnanec WHERE manazer_id IS NULL   -- začínáme od šéfa

    UNION ALL

    SELECT z.id, z.jmeno, z.manazer_id, p.uroven + 1
    FROM zamestnanec z
    JOIN podrizeni p ON p.id = z.manazer_id
)
SELECT * FROM podrizeni ORDER BY uroven;
```

---

# Shrnutí

Vnořený dotaz = SELECT uvnitř jiného příkazu.

Použití: ve WHERE (IN, =), ve FROM (dočasná tabulka), ve SELECT.

EXISTS: zkontroluje zda vnořený dotaz vrátil výsledek.

CTE (WITH): pojmenovaný dočasný výsledek – přehlednější než vnořený dotaz ve FROM.

WITH RECURSIVE: procházení hierarchií (strom zaměstnanců, kategorie).

---

# Typické doplňující otázky

## Jaký je rozdíl mezi IN a EXISTS?

IN: porovná hodnoty ze seznamu. Vhodné pro menší seznamy.
EXISTS: zkontroluje existenci řádku. Rychlejší pro velké tabulky.

---

## Co je CTE?

Common Table Expression – pojmenovaný dočasný výsledek dotazu definovaný na začátku pomocí WITH.

---

## Kdy použít WITH RECURSIVE?

Při práci s hierarchickými daty – strom zaměstnanců, kategorie, složky souborového systému.

---

## Co je korelovaný poddotaz?

Poddotaz, který odkazuje na sloupec vnějšího dotazu. Spouští se pro každý řádek vnějšího dotazu.
