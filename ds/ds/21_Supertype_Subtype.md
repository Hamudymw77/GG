# 1. Co jsou hierarchické struktury v relačních databázích

Některé entity v reálném světě mají hierarchickou strukturu.

Například: Osoba může být Zaměstnanec nebo Zákazník.

Vozidlo může být Osobní auto, Nákladní auto nebo Motocykl.

Relační databáze musí mít způsob, jak tuto hierarchii modelovat.

---

# 2. Generalizace a specializace

Generalizace: více konkrétních entit sdílí společné atributy → vytvoříme nadtyp.

Specializace: nadtyp se rozdělí na podtypy s vlastními atributy.

Nadtyp = Supertype.

Podtyp = Subtype.

---

# 3. Tři implementační vzory

## Vzor 1: Jedna tabulka (Single Table Inheritance)

Všechny entity v jedné tabulce.

Sloupce pro všechny podtypy, nevyplněné jsou NULL.

Přidán sloupec `typ` pro rozlišení.

```sql
CREATE TABLE osoba (
    id        INT PRIMARY KEY,
    jmeno     VARCHAR(100),
    typ       VARCHAR(20),  -- 'zamestnanec' nebo 'zakaznik'
    -- sloupce zaměstnance
    plat      DECIMAL(10,2),
    oddeleni  VARCHAR(100),
    -- sloupce zákazníka
    email     VARCHAR(150),
    vernostni_body INT
);
```

Výhody: jednoduché dotazy, jeden JOIN.

Nevýhody: mnoho NULL hodnot, nelze přidat NOT NULL na sloupce podtypu.

---

## Vzor 2: Tabulka pro každý podtyp (Concrete Table Inheritance)

Každý podtyp má vlastní úplnou tabulku.

```sql
CREATE TABLE zamestnanec (
    id       INT PRIMARY KEY,
    jmeno    VARCHAR(100),
    plat     DECIMAL(10,2),
    oddeleni VARCHAR(100)
);

CREATE TABLE zakaznik (
    id             INT PRIMARY KEY,
    jmeno          VARCHAR(100),
    email          VARCHAR(150),
    vernostni_body INT
);
```

Výhody: žádné NULL, každá tabulka obsahuje jen relevantní data.

Nevýhody: duplikace společných sloupců, obtížné dotazování přes podtypy.

---

## Vzor 3: Supertype + Subtype tabulky

Sdílené atributy jsou v nadtypové tabulce.

Specifické atributy jsou v podtypových tabulkách.

Podtyp sdílí PK s nadtypem (1:1 vztah).

```sql
CREATE TABLE osoba (
    id    INT AUTO_INCREMENT PRIMARY KEY,
    jmeno VARCHAR(100) NOT NULL,
    typ   VARCHAR(20) NOT NULL  -- diskriminátor
);

CREATE TABLE zamestnanec (
    id       INT PRIMARY KEY,
    plat     DECIMAL(10,2),
    oddeleni VARCHAR(100),
    FOREIGN KEY (id) REFERENCES osoba(id)
);

CREATE TABLE zakaznik (
    id             INT PRIMARY KEY,
    email          VARCHAR(150),
    vernostni_body INT DEFAULT 0,
    FOREIGN KEY (id) REFERENCES osoba(id)
);
```

Výhody: normalizované, žádná duplicita, bez zbytečných NULL.

Nevýhody: potřeba JOIN pro kompletní data.

---

# 4. Porovnání vzorů

| Vzor | Výhoda | Nevýhoda |
|------|--------|----------|
| Single Table | Jednoduché dotazy | Hodně NULL, nelze NOT NULL |
| Concrete Table | Čistá data | Duplicita, náročné hledání |
| Supertype/Subtype | Normalizované | Nutný JOIN |

---

# 5. UNION pro výpis z více podtypů

Vzor 2 (oddělené tabulky) neumožňuje jednoduché hledání. UNION pomůže:

```sql
SELECT id, jmeno, 'zamestnanec' AS typ FROM zamestnanec
UNION ALL
SELECT id, jmeno, 'zakaznik' AS typ FROM zakaznik;
```

---

# Shrnutí

Supertype: tabulka se sdílenými atributy.

Subtype: tabulka s atributy specifickými pro podtyp.

Tři vzory: Single Table, Concrete Table, Supertype+Subtype.

Výchozí doporučení pro produkci: Supertype+Subtype (nejlepší rovnováha).

---

# Typické doplňující otázky

## Co je supertype a subtype?

Supertype je nadřazená tabulka se sdílenými atributy. Subtype je podřazená tabulka s atributy specifickými pro daný typ.

---

## Jaký je nejlepší vzor pro supertype/subtype?

Závisí na použití. Pro normalizaci a datovou integritu je nejlepší Supertype+Subtype. Pro jednoduché dotazy Single Table.

---

## Jak se sdílí PK mezi supertype a subtype?

Subtype má stejný primární klíč jako supertype a zároveň je to cizí klíč na supertype. Vytváří 1:1 vztah.

---

## K čemu slouží diskriminátor?

Sloupec `typ` v supertype tabulce, který určuje, jakého podtypu daná entita je.
