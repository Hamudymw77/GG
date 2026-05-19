# 1. Co je index

Index je datová struktura, která urychluje vyhledávání v tabulce.

Bez indexu musí databáze projít každý řádek (full table scan).

S indexem přeskočí přímo na hledaná data.

Nevýhoda: index zabírá místo na disku a zpomaluje INSERT, UPDATE, DELETE.

---

# 2. B-Tree index

Nejpoužívanější typ indexu.

Interně je organizován jako vyvážený strom.

Vyhledávání má složitost O(log n) – logaritmická.

B-Tree indexy podporují:

- Rovnost: `WHERE id = 5`
- Rozsah: `WHERE cena BETWEEN 100 AND 500`
- Řazení: `ORDER BY jmeno`

---

# 3. Typy indexů v MySQL

## PRIMARY KEY

Primární klíč je automaticky indexovaný.

Určuje fyzické pořadí řádků na disku (clustered index v InnoDB).

---

## UNIQUE

Zajišťuje unikátnost hodnot v sloupci.

Je to zároveň index.

```sql
CREATE UNIQUE INDEX idx_email ON zakaznik(email);
```

---

## INDEX (běžný)

Urychluje vyhledávání, neunikátní.

```sql
CREATE INDEX idx_mesto ON zakaznik(mesto);
```

---

## FULLTEXT

Pro fulltextové vyhledávání v textu (slova, fráze).

```sql
CREATE FULLTEXT INDEX idx_popis ON produkt(popis);

SELECT * FROM produkt WHERE MATCH(popis) AGAINST('notebook');
```

---

## Složený index (composite)

Index na více sloupcích.

Musí dodržovat pravidlo levého prefixu: index (A, B, C) pomůže dotazu na (A), (A, B), nebo (A, B, C), ale ne na samotné (B) nebo (C).

```sql
CREATE INDEX idx_mesto_jmeno ON zakaznik(mesto, jmeno);
```

---

# 4. EXPLAIN – kdy se index použije

```sql
EXPLAIN SELECT * FROM zakaznik WHERE mesto = 'Praha';
```

Důležité sloupce:

type: ALL = bez indexu (špatně), ref = index použit, const = PK použit.

key: název použitého indexu.

rows: počet řádků, které musí databáze prohledat.

---

# 5. Kdy index pomáhá

Sloupce ve WHERE podmínce.

Sloupce v JOIN podmínce (ON z.id = o.zakaznik_id).

Sloupce v ORDER BY.

Sloupce s vysokou selektivitou (mnoho různých hodnot – email, jmeno).

---

# 6. Kdy index nepomáhá nebo škodí

Sloupce s nízkou selektivitou (pohlaví, boolean) – málokdy stojí za to.

Funkce nad sloupcem v podmínce:

```sql
-- Tento dotaz NEPOUŽIJE index na datum:
WHERE YEAR(datum) = 2024

-- Tento ANO:
WHERE datum BETWEEN '2024-01-01' AND '2024-12-31'
```

Tabulky s velmi malým počtem řádků – full scan je rychlejší.

---

# 7. Správa indexů

Zobrazení indexů:

```sql
SHOW INDEX FROM zakaznik;
```

Smazání indexu:

```sql
DROP INDEX idx_mesto ON zakaznik;
```

---

# Shrnutí

Index urychluje vyhledávání za cenu místa na disku.

B-Tree: výchozí typ, výborný pro rovnost i rozsah.

FULLTEXT: pro textové vyhledávání.

Složený index: pravidlo levého prefixu.

EXPLAIN: klíčový nástroj pro zjištění, zda dotaz index použije.

---

# Typické doplňující otázky

## Co je index a proč ho používáme?

Datová struktura, která urychluje vyhledávání. Místo procházení všech řádků databáze přeskočí přímo na data.

---

## Co je clustered index?

Index, kde jsou data fyzicky seřazena podle klíče. V InnoDB je clustered index vždy primární klíč.

---

## Co je pravidlo levého prefixu?

Složený index (A, B, C) urychluje dotazy na (A), (A, B), nebo (A, B, C), ale ne na samotné (B) nebo (C).

---

## Proč je EXPLAIN důležitý?

Ukáže, zda dotaz použije index nebo prochází celou tabulku (type: ALL). Pomůže identifikovat pomalé dotazy.
