# 1. Co je relační databáze

Relační databáze je způsob ukládání dat do tabulek, kde každá tabulka má řádky a sloupce.

Tabulky jsou navzájem propojené pomocí klíčů.

Příklad: tabulka zákazníků a tabulka objednávek jsou propojeny přes ID zákazníka.

---

# 2. Základní pojmy

## Tabulka (relace)

Tabulka je základní stavební prvek databáze. Každá tabulka uchovává jeden typ dat.

Příklad:

Tabulka Zákazník:

| id | jmeno | email |
|----|-------|-------|
| 1  | Pavel | p@t.cz |
| 2  | Eva   | e@t.cz |

---

## Řádek (záznam)

Jeden řádek v tabulce je jeden konkrétní záznam. Například jeden zákazník.

---

## Sloupec (atribut)

Sloupec popisuje vlastnost záznamu. Například jméno nebo email zákazníka.

---

## Primární klíč (Primary Key)

Primární klíč jednoznačně identifikuje každý řádek v tabulce.

Žádné dva řádky nemohou mít stejný primární klíč.

Příklad: sloupec `id` je primární klíč.

---

## Cizí klíč (Foreign Key)

Cizí klíč propojuje dvě tabulky.

Příklad: objednávka obsahuje `zakaznik_id`, které odkazuje na `id` v tabulce zákazník.

---

# 3. DBMS – Systém řízení báze dat

DBMS (Database Management System) je software, který spravuje databázi.

Zajišťuje ukládání, čtení, úpravu a mazání dat.

Stará se o bezpečnost a zálohování.

Příklady DBMS:

- MySQL
- PostgreSQL
- Oracle Database
- Microsoft SQL Server
- SQLite

---

## Co DBMS dělá

Přijímá SQL příkazy od aplikace nebo uživatele.

Spustí dotaz nad daty.

Vrátí výsledek.

---

# 4. Relační model – propojení tabulek

Tabulky jsou propojeny pomocí klíčů. Říkáme tomu relace.

Příklad:

Zákazník s id = 1 má tři objednávky. Každá objednávka obsahuje `zakaznik_id = 1`.

Tímto způsobem propojujeme data bez opakování.

---

## Proč je to výhodné

Data se neukládají opakovaně.

Stačí změnit jméno zákazníka na jednom místě.

Databáze zůstane konzistentní.

---

# 5. Typy relací

## Jeden k mnoha (1:N)

Jeden zákazník může mít mnoho objednávek.

Jedna objednávka patří jen jednomu zákazníkovi.

---

## Mnoho k mnoha (M:N)

Jedna objednávka může obsahovat více produktů.

Jeden produkt může být ve více objednávkách.

Tento vztah se řeší přes pomocnou (vazební) tabulku.

---

## Jeden k jednomu (1:1)

Jeden zaměstnanec má jeden průkaz.

Jeden průkaz patří jednomu zaměstnanci.

---

# 6. NoSQL vs relační databáze

## Relační databáze

Data jsou v tabulkách.

Pevná struktura (schéma).

Používá SQL.

Vhodná pro: e-shopy, bankovnictví, ERP systémy.

---

## NoSQL databáze

Data jsou v dokumentech, grafech nebo klíč-hodnota formátu.

Flexibilní struktura.

Vhodná pro: sociální sítě, big data, real-time aplikace.

Příklady: MongoDB, Redis, Cassandra.

---

## Klíčový rozdíl

Relační = strukturovaná data, pevné schéma, vztahy mezi tabulkami.

NoSQL = flexibilní, rychlé, bez pevné struktury.

---

# Shrnutí

Relační databáze ukládá data do tabulek propojených klíči.

Každá tabulka má primární klíč.

Propojení tabulek probíhá přes cizí klíče.

DBMS je software, který databázi spravuje.

Příklady DBMS: MySQL, Oracle, SQL Server.

---

# Typické doplňující otázky

## Co je primární klíč?

Unikátní identifikátor každého řádku v tabulce. Nesmí být NULL ani se opakovat.

---

## Co je cizí klíč?

Sloupec, který odkazuje na primární klíč jiné tabulky. Zajišťuje propojení tabulek.

---

## Jaký je rozdíl mezi SQL a NoSQL?

SQL = strukturované tabulky, pevné schéma, vztahy.
NoSQL = dokumenty nebo grafy, flexibilní schéma, rychlé pro velká data.

---

## Co dělá DBMS?

Spravuje databázi – ukládá data, zpracovává dotazy, zajišťuje bezpečnost a zálohy.
