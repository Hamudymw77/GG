# 1. Co je pokročilé modelování

Pokročilé modelování databáze řeší situace, které nejdou jednoduše vyjádřit standardními vztahy 1:N nebo M:N.

Dvě hlavní techniky: Self-Reference (rekurzivní vztah) a Arc (exkluzivní alternativní vztah).

---

# 2. Self-Reference (rekurzivní vztah)

Self-reference je vztah, kde tabulka odkazuje sama na sebe.

Typický příklad: organizační hierarchie zaměstnanců.

Každý zaměstnanec má nadřízeného – a nadřízený je také zaměstnanec.

```sql
CREATE TABLE zamestnanec (
    id         INT AUTO_INCREMENT PRIMARY KEY,
    jmeno      VARCHAR(100) NOT NULL,
    manazer_id INT,  -- NULL = vrchol hierarchie (CEO)
    FOREIGN KEY (manazer_id) REFERENCES zamestnanec(id)
);
```

Cizí klíč odkazuje na primární klíč té samé tabulky.

---

## Příklady self-reference

Organizační hierarchie (zaměstnanec → manažer).

Kategorie produktů (podkategorie → nadkategorie).

Složky souborového systému (složka → nadřazená složka).

Komentáře s odpověďmi (komentář → rodičovský komentář).

---

# 3. WITH RECURSIVE – procházení hierarchie

Pro výpis celé hierarchie (stromová struktura) nestačí jednoduchý JOIN.

Používáme rekurzivní CTE:

```sql
WITH RECURSIVE hierarchie AS (
    -- kotva: začneme od CEO (bez nadřízeného)
    SELECT id, jmeno, manazer_id, 0 AS uroven
    FROM zamestnanec WHERE manazer_id IS NULL

    UNION ALL

    -- rekurzivní krok: přidej přímé podřízené
    SELECT z.id, z.jmeno, z.manazer_id, h.uroven + 1
    FROM zamestnanec z
    JOIN hierarchie h ON h.id = z.manazer_id
)
SELECT REPEAT('  ', uroven) AS odsazeni, jmeno, uroven
FROM hierarchie ORDER BY uroven, jmeno;
```

---

# 4. Arc vztah (exkluzivní alternativa)

Arc je modelovací vzor, kde entita patří právě jednomu z více alternativních rodičů.

Příklad: Platba může být buď platba kartou NEBO platba převodem.

Nikdy obojí zároveň. Vždy právě jedno.

---

## Implementace arc v SQL

Přístup 1: cizí klíče + CHECK constraint

```sql
CREATE TABLE platba (
    id                  INT AUTO_INCREMENT PRIMARY KEY,
    castka              DECIMAL(12,2),
    karta_id            INT,
    prevod_id           INT,
    -- Právě jeden z cizích klíčů musí být vyplněn
    CHECK (
        (karta_id IS NOT NULL AND prevod_id IS NULL)
        OR
        (karta_id IS NULL AND prevod_id IS NOT NULL)
    )
);
```

Přístup 2: tabulka platba + subtypy (viz otázka 21 – Supertype/Subtype)

---

# 5. Kdy použít self-reference vs arc

Self-reference: data mají přirozenou stromovou nebo hierarchickou strukturu.

Arc: jedna tabulka musí odkazovat na právě jednu z více tabulek (exkluzivní výběr).

---

# 6. Příklady z praxe

## Self-reference

Tabulka `kategorie` – každá kategorie má nadkategorii:

```sql
CREATE TABLE kategorie (
    id              INT AUTO_INCREMENT PRIMARY KEY,
    nazev           VARCHAR(100),
    nadkategorie_id INT,
    FOREIGN KEY (nadkategorie_id) REFERENCES kategorie(id)
);
```

---

## Arc – platební metody

Tabulka `platba_karta` a `platba_prevod` – každá platba patří právě jedné:

```sql
CREATE TABLE platba_karta (
    id         INT AUTO_INCREMENT PRIMARY KEY,
    cislo_karty VARCHAR(20),
    platna_do  DATE
);

CREATE TABLE platba_prevod (
    id         INT AUTO_INCREMENT PRIMARY KEY,
    cislo_uctu VARCHAR(30),
    banka      VARCHAR(100)
);

CREATE TABLE platba (
    id                  INT AUTO_INCREMENT PRIMARY KEY,
    castka              DECIMAL(12,2),
    karta_id            INT,
    prevod_id           INT,
    CHECK (
        (karta_id IS NOT NULL AND prevod_id IS NULL) OR
        (karta_id IS NULL AND prevod_id IS NOT NULL)
    )
);
```

---

# Shrnutí

Self-reference: tabulka odkazuje sama na sebe (hierarchie).

WITH RECURSIVE: procházení hierarchické struktury.

Arc: entita patří právě jednomu ze dvou nebo více alternativních rodičů.

CHECK constraint: zajišťuje platnost arc vztahu na úrovni databáze.

---

# Typické doplňující otázky

## Co je self-reference?

Cizí klíč, který odkazuje na primární klíč té samé tabulky. Používá se pro hierarchie.

---

## Kdy použít WITH RECURSIVE?

Pro procházení stromové nebo hierarchické struktury – organizace, kategorie, složky.

---

## Co je arc vztah?

Situace, kdy tabulka musí odkazovat na právě jednu z více alternativních tabulek. Exkluzivní výběr.

---

## Jak zajistit platnost arc v databázi?

CHECK constraintem – ověří, že právě jeden cizí klíč je vyplněn a ostatní jsou NULL.
