# 1. Co je Power BI

Power BI je nástroj od Microsoftu pro vizualizaci dat a tvorbu interaktivních reportů.

Umožňuje připojit se k různým zdrojům dat (MySQL, Excel, CSV, SQL Server, cloudové služby) a vytvořit přehledné dashboardy.

---

# 2. Komponenty Power BI

## Power BI Desktop

Desktopová aplikace pro Windows.

Zde se buduje datový model, transformují data a vytvářejí vizualizace.

---

## Power BI Service

Cloudová platforma (app.powerbi.com).

Sem se nahrávají hotové reporty a sdílí s kolegy.

---

## Power BI Mobile

Mobilní aplikace pro iOS a Android.

Zobrazuje hotové dashboardy.

---

# 3. Způsoby připojení dat

## Import

Data se stáhnou do Power BI a uloží lokálně.

Nejrychlejší pro vizualizace.

Nevýhoda: data se neaktualizují automaticky (je třeba naplánovat refresh).

---

## DirectQuery

Dotazy se posílají přímo do zdrojové databáze při každém zobrazení.

Data jsou vždy aktuální.

Nevýhoda: pomalejší, závisí na výkonu databáze.

---

## Live Connection

Používá se pro Analysis Services nebo Power BI Premium.

Datový model je sdílený na serveru.

---

# 4. Datový model v Power BI

Power BI pracuje s tabulkami a vztahy mezi nimi.

Vztahy jsou definovány pomocí cizích klíčů (stejný princip jako v SQL).

Doporučený model je Star Schema (tabulka faktů + dimenze).

---

# 5. DAX – základy

DAX (Data Analysis Expressions) je jazyk pro výpočty v Power BI.

Slouží k tvorbě nových sloupců a měr (measures).

---

## Míry (Measures)

Vypočítávají se dynamicky podle filtru.

```dax
Celkova trzba = SUM(fact_prodeje[trzba])

Pocet zakazniku = DISTINCTCOUNT(fact_prodeje[zakaznik_id])

Prumerna objednavka = AVERAGE(fact_prodeje[trzba])
```

---

## Kalkulované sloupce

Přidávají nový sloupec do tabulky.

```dax
Rok mesic = YEAR(dim_cas[datum]) & "-" & FORMAT(dim_cas[datum], "MM")
```

---

## CALCULATE – modifikátor kontextu

Přepočítá míru v jiném filtrovacím kontextu.

```dax
Trzba Praha = CALCULATE(SUM(fact_prodeje[trzba]), dim_zakaznik[mesto] = "Praha")
```

---

# 6. Typy vizualizací

Sloupcový graf: srovnání hodnot.

Spojnicový graf: vývoj v čase.

Koláčový graf: podíly celku.

Karta (Card): zobrazí jedno číslo (celková tržba, počet zákazníků).

Mapa: geografická distribuce.

Tabulka / Matice: detailní přehled.

Průřez (Slicer): filtr pro interaktivní výběr (rok, region, kategorie).

---

# 7. Příprava dat v MySQL pro Power BI

Power BI se připojuje k MySQL přes ODBC konektor.

SQL dotazy pro přípravu dat:

```sql
-- Jednoduchý přehled prodejů pro Power BI
SELECT
    z.jmeno AS zakaznik,
    z.mesto,
    p.nazev AS produkt,
    p.kategorie,
    c.datum,
    c.rok,
    c.mesic,
    f.trzba,
    f.mnozstvi
FROM fact_prodeje f
JOIN dim_zakaznik z ON z.id = f.zakaznik_id
JOIN dim_produkt  p ON p.id = f.produkt_id
JOIN dim_cas      c ON c.datum_id = f.datum_id;
```

---

# Shrnutí

Power BI: nástroj pro vizualizaci a dashboardy.

Import / DirectQuery: způsoby připojení – rychlost vs aktuálnost dat.

Datový model: Star Schema – tabulka faktů + dimenze.

DAX: jazyk pro výpočty (SUM, AVERAGE, CALCULATE).

Typy vizualizací: grafy, karty, mapy, průřezy.

---

# Typické doplňující otázky

## Co je DAX?

Jazyk pro výpočty v Power BI (Data Analysis Expressions). Slouží pro tvorbu měr a kalkulovaných sloupců.

---

## Jaký je rozdíl mezi Import a DirectQuery?

Import: data jsou stažena lokálně, rychlejší. DirectQuery: každý dotaz jde přímo do databáze, data jsou aktuální.

---

## Co je míra (measure) v Power BI?

Vypočítávaná hodnota (SUM, AVERAGE…) definovaná v DAX, která reaguje na filtry v reportu.

---

## Proč Star Schema v Power BI?

Optimalizace pro analytické dotazy a přehledný datový model – fact tabulka uprostřed, dimenze kolem.
