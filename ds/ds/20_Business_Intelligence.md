# 1. Co je Business Intelligence

Business Intelligence (BI) je proces přeměny dat na informace a informací na rozhodnutí.

Zahrnuje sběr dat, jejich zpracování, analýzu a vizualizaci.

Cílem je poskytnout manažerům a analytikům přehled o stavu podniku.

---

# 2. Architektura BI systému

Zdrojové systémy (ERP, CRM, e-shop) → ETL → Datový sklad (DWH) → BI nástroje (Power BI, Tableau) → Reporting

---

# 3. ETL – základ každého BI

ETL = Extract, Transform, Load.

Extract: vytáhnutí dat ze zdrojů (databáze, soubory, API).

Transform: čištění, deduplikace, sjednocení formátů, výpočty.

Load: nahrání čistých dat do datového skladu.

---

# 4. KPI – klíčové ukazatele výkonu

KPI (Key Performance Indicator) jsou měřitelné hodnoty, které ukazují, jak se daří podniku.

Příklady KPI:

- Celková tržba
- Počet nových zákazníků
- Průměrná hodnota objednávky
- Podíl reklamací
- NPS (Net Promoter Score)

---

# 5. Balanced Scorecard

Metoda měření výkonnosti podniku ve čtyřech perspektivách:

Finanční: tržby, zisk, náklady.

Zákaznická: spokojenost, loajalita, počet zákazníků.

Interní procesy: efektivita, časy dodání, kvalita.

Učení a růst: vzdělávání zaměstnanců, inovace.

---

# 6. SCD – Slowly Changing Dimensions

Data v dimenzích se v čase mění (zákazník změní adresu, produkt změní cenu).

SCD Type 1: Přepíše starý záznam. Historii ztratíme.

SCD Type 2: Přidá nový řádek, starý označí jako neplatný. Historie se uchovává.

SCD Type 3: Přidá nový sloupec (aktuální_hodnota, predchozi_hodnota). Uchovává jen jedno předchozí stav.

---

# 7. Audit trail

Audit trail zaznamenává, kdo co a kdy změnil v databázi.

Slouží pro:

- Regulatorní požadavky (GDPR, SOX, ISO 27001)
- Forenzní analýzu
- Detekci podvodů

Implementuje se pomocí triggerů nebo speciálních auditovacích nástrojů.

---

# 8. OLAP operace

Drill down: přiblížení detailu (rok → čtvrtletí → měsíc).

Drill up (Roll up): zobrazení souhrnu (den → měsíc → rok).

Slice: filtrování jedné dimenze (jen Praha).

Dice: filtrování více dimenzí (Praha + Elektronika + Q1).

Pivot: otočení pohledu (řádky ↔ sloupce).

---

# Shrnutí

BI = přeměna dat na rozhodnutí.

ETL: Extract, Transform, Load – základ BI pipeline.

KPI: měřitelné ukazatele výkonu podniku.

SCD: Type 1 (přepsat), Type 2 (nový řádek s historií), Type 3 (nový sloupec).

Audit trail: záznam všech změn dat.

OLAP operace: drill down/up, slice, dice, pivot.

---

# Typické doplňující otázky

## Co je KPI?

Key Performance Indicator – měřitelný ukazatel výkonu, který sleduje plnění cílů podniku.

---

## Co je SCD Type 2?

Způsob uchovávání historických změn v dimenzích. Při změně se přidá nový řádek, starý se označí jako neplatný.

---

## Jaký je rozdíl mezi BI a AI?

BI analyzuje historická data a prezentuje je formou reportů a dashboardů. AI/ML predikuje budoucí vývoj a hledá vzory.

---

## Co je drill down?

OLAP operace, která přibližuje detail – například přechod z pohledu na tržby po letech na pohled po měsících.
