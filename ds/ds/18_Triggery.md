# 1. Co je trigger

Trigger je procedura, která se automaticky spustí při určité události na tabulce.

Událost = INSERT, UPDATE nebo DELETE.

Trigger nepotřebujeme volat – spustí se sám.

---

# 2. Kombinace BEFORE / AFTER × INSERT / UPDATE / DELETE

Celkem existuje 6 kombinací:

| Čas | Událost | Použití |
|-----|---------|---------|
| BEFORE | INSERT | Validace před vložením |
| AFTER | INSERT | Aktualizace related tabulek |
| BEFORE | UPDATE | Validace, nastavení hodnot |
| AFTER | UPDATE | Audit log, přepočet |
| BEFORE | DELETE | Archivace, kontrola |
| AFTER | DELETE | Audit log, cleanup |

---

# 3. NEW a OLD

V triggeru jsou k dispozici speciální záznamy:

NEW: nové hodnoty sloupců (dostupné u INSERT a UPDATE).

OLD: původní hodnoty sloupců (dostupné u UPDATE a DELETE).

```sql
-- Trigger při UPDATE
AFTER UPDATE ON zakaznik
BEGIN
    -- OLD.email = původní email
    -- NEW.email = nový email
END;
```

---

# 4. BEFORE INSERT – validace

Spouští se před vložením záznamu.

Lze zabránit vložení pomocí SIGNAL SQLSTATE.

```sql
DELIMITER //
CREATE TRIGGER trg_kontrola_ceny
BEFORE INSERT ON produkt
FOR EACH ROW
BEGIN
    IF NEW.cena <= 0 THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Cena musí být kladná';
    END IF;
END //
DELIMITER ;
```

---

# 5. AFTER INSERT – aktualizace dat

Po vložení záznamu se automaticky provedou další akce.

```sql
DELIMITER //
CREATE TRIGGER trg_po_objednavce
AFTER INSERT ON objednavka
FOR EACH ROW
BEGIN
    UPDATE zakaznik
    SET vernostni_body = vernostni_body + FLOOR(NEW.celkem / 100)
    WHERE id = NEW.zakaznik_id;
END //
DELIMITER ;
```

---

# 6. AFTER UPDATE – auditní log

```sql
DELIMITER //
CREATE TRIGGER trg_audit_zakaznik
AFTER UPDATE ON zakaznik
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (tabulka, zaznam_id, akce, stara_hodnota, nova_hodnota, cas)
    VALUES ('zakaznik', OLD.id, 'UPDATE', OLD.email, NEW.email, NOW());
END //
DELIMITER ;
```

---

# 7. BEFORE DELETE – archivace

```sql
DELIMITER //
CREATE TRIGGER trg_archiv_objednavky
BEFORE DELETE ON objednavka
FOR EACH ROW
BEGIN
    INSERT INTO archiv_objednavek (puvodni_id, zakaznik_id, datum, celkem)
    VALUES (OLD.id, OLD.zakaznik_id, OLD.datum, OLD.celkem);
END //
DELIMITER ;
```

---

# 8. Omezení triggerů

Trigger nemůže volat jiný trigger (kaskáda) v základní konfiguraci.

Trigger nemůže použít COMMIT nebo ROLLBACK.

Trigger nesmí přímo číst nebo modifikovat tabulku, na které je definován.

Výkon: hodně triggerů = pomalejší INSERT/UPDATE/DELETE.

---

# 9. Správa triggerů

```sql
SHOW TRIGGERS FROM mojeDB;

DROP TRIGGER IF EXISTS trg_audit_zakaznik;
```

---

# Shrnutí

Trigger se spustí automaticky při INSERT, UPDATE nebo DELETE.

BEFORE: validace, úprava hodnot před provedením.

AFTER: audit log, aktualizace related tabulek.

NEW: nové hodnoty (INSERT, UPDATE). OLD: původní hodnoty (UPDATE, DELETE).

SIGNAL SQLSTATE: zabránění provedení a vyvolání chyby.

---

# Typické doplňující otázky

## Co je trigger?

Procedura, která se automaticky spustí při INSERT, UPDATE nebo DELETE na tabulce.

---

## K čemu slouží NEW a OLD?

NEW obsahuje nové hodnoty sloupců (po změně), OLD obsahuje původní hodnoty (před změnou).

---

## Jaký je rozdíl mezi BEFORE a AFTER triggerem?

BEFORE se spustí před provedením operace – lze validovat nebo upravit data. AFTER se spustí po provedení – typicky pro audit log.

---

## Co je SIGNAL SQLSTATE?

Způsob, jak z triggeru (nebo procedury) vyvolat chybu a zabránit provedení operace.
