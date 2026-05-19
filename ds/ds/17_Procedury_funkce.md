# 1. Co jsou uložené procedury a funkce

Uložená procedura je pojmenovaná skupina SQL příkazů uložená v databázi.

Zavoláme ji pomocí CALL a provede se na serveru.

Funkce se liší od procedury tím, že vždy vrací jednu hodnotu a lze ji použít přímo v SELECT.

---

# 2. Výhody procedur

Logika je na serveru, ne v aplikaci.

Snižuje počet dotazů ze sítě.

Jeden kód se volá z více míst.

Lze omezit přímý přístup k tabulkám – uživatelé volají jen procedury.

---

# 3. DELIMITER

Standardní oddělovač příkazů v MySQL je středník `;`.

Procedura uvnitř ale také obsahuje středníky.

Proto musíme dočasně změnit oddělovač.

```sql
DELIMITER //

CREATE PROCEDURE mojeProcedura()
BEGIN
    SELECT 'Ahoj';
END //

DELIMITER ;
```

---

# 4. Procedura bez parametrů

```sql
DELIMITER //
CREATE PROCEDURE vypis_zakazniky()
BEGIN
    SELECT jmeno, email FROM zakaznik ORDER BY jmeno;
END //
DELIMITER ;

CALL vypis_zakazniky();
```

---

# 5. Parametry procedury

Procedura může mít tři typy parametrů:

IN: vstupní hodnota (výchozí, stejně jako v jiných jazycích).

OUT: výstupní hodnota – procedura do ní uloží výsledek.

INOUT: vstup i výstup.

```sql
DELIMITER //
CREATE PROCEDURE soucet_trzeb(
    IN zakaznik_id INT,
    OUT celkem     DECIMAL(12,2)
)
BEGIN
    SELECT SUM(trzba) INTO celkem
    FROM objednavka
    WHERE zakaznik_id = zakaznik_id;
END //
DELIMITER ;

CALL soucet_trzeb(1, @soucet);
SELECT @soucet;
```

---

# 6. Řídicí struktury

## IF / ELSE

```sql
IF pocet > 100 THEN
    SET kategorie = 'VIP';
ELSEIF pocet > 50 THEN
    SET kategorie = 'Standard';
ELSE
    SET kategorie = 'Nový';
END IF;
```

---

## WHILE

```sql
DECLARE i INT DEFAULT 1;
WHILE i <= 10 DO
    INSERT INTO cisla (hodnota) VALUES (i);
    SET i = i + 1;
END WHILE;
```

---

# 7. Zpracování chyb (HANDLER)

```sql
DELIMITER //
CREATE PROCEDURE pridej_zakaznika(IN email VARCHAR(150))
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        SELECT 'Chyba: zákazník nebyl přidán' AS zprava;
    END;

    INSERT INTO zakaznik (email) VALUES (email);
    SELECT 'OK: zákazník přidán' AS zprava;
END //
DELIMITER ;
```

---

# 8. Funkce

Funkce vždy vrací jednu hodnotu pomocí RETURN.

Lze ji použít přímo v SELECT, WHERE nebo jako hodnotu.

```sql
DELIMITER //
CREATE FUNCTION kategorie_zakaznika(bod INT)
RETURNS VARCHAR(20) DETERMINISTIC
BEGIN
    IF bod >= 1000 THEN RETURN 'VIP';
    ELSEIF bod >= 100 THEN RETURN 'Standard';
    ELSE RETURN 'Nový';
    END IF;
END //
DELIMITER ;

SELECT jmeno, vernostni_body, kategorie_zakaznika(vernostni_body) AS kategorie
FROM zakaznik;
```

---

# 9. Správa procedur a funkcí

Zobrazení procedur:

```sql
SHOW PROCEDURE STATUS WHERE Db = 'mojeDB';
SHOW CREATE PROCEDURE nazev_procedury;
```

Smazání:

```sql
DROP PROCEDURE IF EXISTS nazev_procedury;
DROP FUNCTION IF EXISTS nazev_funkce;
```

---

# Shrnutí

Procedura: GROUP SQL příkazů spouštěná přes CALL.

Funkce: vrací jednu hodnotu, použitelná v SELECT.

Parametry: IN (vstup), OUT (výstup), INOUT (obojí).

DELIMITER: nutný pro definici procedur a funkcí.

HANDLER: zachycení chyb uvnitř procedury.

---

# Typické doplňující otázky

## Jaký je rozdíl mezi procedurou a funkcí?

Procedura se volá přes CALL, nemůže být v SELECT. Funkce vrací jednu hodnotu a lze ji použít přímo v dotazu.

---

## Proč se mění DELIMITER?

Procedura obsahuje středníky, které by MySQL interpretovalo jako konec příkazu. Změnou oddělovače tomu zabráníme.

---

## Co je HANDLER?

Mechanismus pro zachycení chyb uvnitř procedury. Jako try/catch v jiných jazycích.

---

## Co znamená DETERMINISTIC?

Funkce se stejnými vstupy vždy vrátí stejný výsledek. Opak je NOT DETERMINISTIC (výsledek závisí na čase, náhodě atd.).
