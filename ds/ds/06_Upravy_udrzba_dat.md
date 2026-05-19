# 1. Co jsou úpravy a údržba dat

Úpravy dat znamenají vkládání nových dat, změnu existujících dat a mazání dat.

Údržba databáze zahrnuje také čištění dat, opravování chyb a optimalizaci.

---

# 2. Vkládání dat (INSERT)

Příkaz INSERT vloží nový řádek do tabulky.

Základní použití:

```sql
INSERT INTO zakaznik (jmeno, email) VALUES ('Pavel', 'pavel@t.cz');
```

---

## Vložení více řádků najednou

```sql
INSERT INTO zakaznik (jmeno, email) VALUES
    ('Pavel', 'pavel@t.cz'),
    ('Eva', 'eva@t.cz'),
    ('Karel', 'karel@t.cz');
```

Je to rychlejší než tři samostatné INSERT příkazy.

---

## UPSERT – vložit nebo aktualizovat

Pokud záznam existuje, aktualizuj ho. Pokud ne, vlož nový.

```sql
INSERT INTO zakaznik (id, jmeno, email) VALUES (1, 'Pavel', 'novy@t.cz')
ON DUPLICATE KEY UPDATE email = 'novy@t.cz';
```

---

# 3. Aktualizace dat (UPDATE)

Příkaz UPDATE změní hodnoty existujících řádků.

```sql
UPDATE zakaznik SET email = 'novy@t.cz' WHERE id = 1;
```

Pozor: bez WHERE se změní všechny řádky.

---

## UPDATE s podmínkou na více polí

```sql
UPDATE produkt SET cena = cena * 1.10 WHERE kategorie = 'Elektronika';
```

Zvýší cenu všech elektronických produktů o 10 %.

---

# 4. Mazání dat (DELETE)

Příkaz DELETE smaže řádky z tabulky.

```sql
DELETE FROM zakaznik WHERE id = 5;
```

Pozor: bez WHERE se smažou všechny řádky.

---

## Soft delete (měkké mazání)

Místo skutečného smazání nastavíme příznak `aktivni = FALSE`.

```sql
UPDATE zakaznik SET aktivni = FALSE WHERE id = 5;
```

Data zůstanou v databázi, ale aplikace je nebude zobrazovat.

Výhoda: data lze obnovit, sledujeme historii.

---

## TRUNCATE

Smaže všechny řádky tabulky, ale tabulku ponechá.

Rychlejší než DELETE bez WHERE, ale nelze vrátit zpět.

```sql
TRUNCATE TABLE log_udalosti;
```

---

# 5. ETL – přesun dat

ETL znamená Extract, Transform, Load.

Je to proces přesunu dat z jednoho místa na druhé.

Extract = načtení dat ze zdroje (CSV, jiná DB).

Transform = čištění, převody, mapování hodnot.

Load = vložení do cílové tabulky.

---

## Příklad: import z dočasné tabulky

```sql
-- Staging tabulka se surovými daty
INSERT INTO zakaznik (jmeno, email)
SELECT jmeno, email FROM staging_import
WHERE email IS NOT NULL;
```

---

# 6. Čištění dat

Čištění dat odstraní duplicity, opraví chyby a doplní chybějící hodnoty.

Příklady:

Odstranění mezer z jmen:

```sql
UPDATE zakaznik SET jmeno = TRIM(jmeno);
```

Mazání duplicitních e-mailů (ponechat jen první):

```sql
DELETE z1 FROM zakaznik z1
JOIN zakaznik z2 ON z1.email = z2.email AND z1.id > z2.id;
```

---

# 7. Optimalizace tabulky

Tabulka se časem fragmentuje po mnoha INSERT a DELETE operacích.

Příkaz OPTIMIZE TABLE defragmentuje tabulku.

```sql
OPTIMIZE TABLE objednavka;
```

Příkaz ANALYZE TABLE aktualizuje statistiky pro optimalizátor dotazů.

```sql
ANALYZE TABLE objednavka;
```

---

# Shrnutí

INSERT vkládá nová data.

UPDATE mění existující data – vždy přidat WHERE.

DELETE maže data – vždy přidat WHERE.

Soft delete = nastavení příznaku místo smazání.

ETL = přesun dat: Extract → Transform → Load.

OPTIMIZE TABLE defragmentuje tabulku.

---

# Typické doplňující otázky

## Jaký je rozdíl mezi DELETE a TRUNCATE?

DELETE maže řádky postupně, lze použít WHERE, lze vrátit zpět.
TRUNCATE maže všechno najednou, nelze použít WHERE, nelze vrátit zpět.

---

## Co je soft delete?

Místo smazání záznamu se nastaví příznak (například `aktivni = FALSE`). Data zůstanou v databázi.

---

## Co je UPSERT?

Kombinace INSERT a UPDATE. Pokud záznam existuje, aktualizuje se. Pokud ne, vloží se.

---

## Co je ETL?

Extract (načtení), Transform (transformace), Load (vložení). Proces přesunu dat ze zdroje do cílové databáze.
