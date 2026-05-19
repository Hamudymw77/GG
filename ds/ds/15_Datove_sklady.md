# 1. Co je datový sklad

Datový sklad (Data Warehouse, DWH) je centrální úložiště dat určené pro analytické dotazy.

Liší se od provozní (transakční) databáze.

---

# 2. Rozdíl OLTP vs OLAP

## OLTP – Online Transaction Processing

Provozní databáze.

Rychlé transakce (INSERT, UPDATE, DELETE).

Malá množství dat najednou.

Příklady: e-shop, bankovní systém, ERP.

---

## OLAP – Online Analytical Processing

Analytická databáze / datový sklad.

Pomalé, složité dotazy přes velká množství historických dat.

Čtení, málo zápisů.

Příklady: reporty, dashboardy, BI nástroje.

---

# 3. Architektura DWH

Data z provozních systémů → ETL → Datový sklad → Reporting / BI nástroje.

ETL = Extract, Transform, Load.

Extract: vytáhnutí dat ze zdrojových systémů.

Transform: čištění, sjednocení formátů, výpočty.

Load: nahrání do datového skladu.

---

# 4. Star Schema (hvězdné schéma)

Nejpoužívanější model pro datové sklady.

Skládá se z:

Tabulky faktů (Fact Table): obsahuje měřitelné hodnoty (tržby, množství, počty).

Dimenzí (Dimension Tables): popisné atributy (zákazník, produkt, čas, region).

Fact table je ve středu, dimenze jsou "paprsky" hvězdy.

```
dim_zakaznik ---|
dim_produkt  ---|--- fact_prodeje
dim_cas      ---|
dim_region   ---|
```

---

# 5. Snowflake Schema

Varianta Star Schema, kde jsou dimenze normalizovány (rozloženy do více tabulek).

Méně dat je uloženo opakovaně, ale dotazy jsou složitější.

---

# 6. Dimenze a Fakta

## Tabulka faktů

Obsahuje cizí klíče do dimenzí a měřitelné hodnoty.

```sql
CREATE TABLE fact_prodeje (
    datum_id     INT,
    zakaznik_id  INT,
    produkt_id   INT,
    mnozstvi     INT,
    trzba        DECIMAL(12,2)
);
```

---

## Tabulka dimenzí

Popisuje entity.

```sql
CREATE TABLE dim_zakaznik (
    id        INT PRIMARY KEY,
    jmeno     VARCHAR(100),
    email     VARCHAR(150),
    mesto     VARCHAR(100)
);

CREATE TABLE dim_cas (
    datum_id  INT PRIMARY KEY,
    datum     DATE,
    den       INT,
    mesic     INT,
    rok       INT,
    ctvrtleti INT
);
```

---

# 7. Analytické dotazy a GROUP BY ROLLUP

GROUP BY WITH ROLLUP generuje mezisoučty a celkový součet.

```sql
SELECT rok, mesic, SUM(trzba)
FROM fact_prodeje
JOIN dim_cas ON dim_cas.datum_id = fact_prodeje.datum_id
GROUP BY rok, mesic WITH ROLLUP;
```

Vrátí tržby po měsících, po letech a celkový součet.

---

# 8. SCD – Slowly Changing Dimensions

Dimenze se v čase mění (zákazník změní adresu, produkt změní cenu).

SCD Type 1: přepíše starou hodnotu – historii ztratíme.

SCD Type 2: přidá nový řádek s novou hodnotou, starý označí jako neplatný – uchovává historii.

```sql
-- SCD Type 2: zákazník se přestěhoval
UPDATE dim_zakaznik SET platny_do = '2024-12-31', aktivni = FALSE WHERE id = 1;

INSERT INTO dim_zakaznik (id_zakaznika, jmeno, email, mesto, platny_od, platny_do, aktivni)
VALUES (1, 'Pavel Novák', 'pavel@t.cz', 'Brno', '2025-01-01', NULL, TRUE);
```

---

# Shrnutí

DWH = centrální analytické úložiště dat.

OLTP vs OLAP: transakční vs analytická databáze.

ETL: Extract, Transform, Load – plnění datového skladu.

Star Schema: fact tabulka + dimenze.

GROUP BY WITH ROLLUP: mezisoučty a celkový součet.

SCD Type 2: historické sledování změn v dimenzích.

---

# Typické doplňující otázky

## Jaký je rozdíl mezi OLTP a OLAP?

OLTP je provozní databáze pro rychlé transakce. OLAP je analytická databáze pro komplexní dotazy nad historickými daty.

---

## Co je ETL?

Proces naplnění datového skladu: Extract (vytáhnutí dat ze zdrojů), Transform (čištění a transformace), Load (nahrání do DWH).

---

## Co je Star Schema?

Datový model pro DWH. Centrální tabulka faktů obsahuje měřitelné hodnoty a je spojena s dimenzemi (zákazník, produkt, čas).

---

## Co je SCD Type 2?

Způsob uchovávání historických změn v dimenzích. Při změně se přidá nový řádek, starý se označí jako neplatný.
