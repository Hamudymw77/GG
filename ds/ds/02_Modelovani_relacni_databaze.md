# 1. Co je modelování databáze

Modelování databáze znamená navrhnutí struktury databáze před tím, než ji začneme vytvářet.

Vytváříme tzv. datový model – plán toho, jaké tabulky budou existovat, jaké budou mít sloupce a jak budou propojeny.

---

# 2. Proč modelovat

Bez modelu bychom mohli:

- zapomenout na důležitá data
- špatně propojit tabulky
- mít zbytečně duplicitní data

Model pomáhá odhalit problémy dříve, než napíšeme první řádek SQL.

---

# 3. Typy modelů

## Konceptuální model

Popisuje co databáze obsahuje, bez technických detailů.

Například: zákazník dělá objednávky, objednávka obsahuje produkty.

Tvoří se jako ER diagram (Entity-Relationship).

---

## Logický model

Upřesňuje konceptuální model. Přidá atributy a typy vztahů (1:N, M:N).

Stále není závislý na konkrétní databázi.

---

## Fyzický model

Je to finální podoba databáze připravená pro konkrétní DBMS.

Obsahuje přesné datové typy, indexy, cizí klíče.

Lze ho vygenerovat jako SQL script.

---

# 4. ER diagram

ER diagram (Entity-Relationship diagram) graficky zobrazuje tabulky a jejich vztahy.

## Co je entita

Entita je věc nebo objekt, o kterém ukládáme data.

Příklady entit: Zákazník, Produkt, Objednávka.

---

## Co je atribut

Atribut je vlastnost entity.

Příklad: entita Zákazník má atributy: jmeno, email, telefon.

---

## Co je vztah

Vztah popisuje, jak jsou entity propojeny.

Příklad: Zákazník **zadává** Objednávku.

---

# 5. Typy vztahů

## 1:N (jeden k mnoha)

Jeden zákazník může mít mnoho objednávek.

Jedna objednávka patří jen jednomu zákazníkovi.

Toto je nejčastější typ vztahu.

---

## M:N (mnoho k mnoha)

Jedna objednávka může obsahovat více produktů.

Jeden produkt může být ve více objednávkách.

Tento vztah se řeší pomocnou (vazební) tabulkou.

Příklad: tabulka `objednavka_produkt` propojuje objednávky a produkty.

---

## 1:1 (jeden k jednomu)

Jeden zaměstnanec má jeden průkaz.

Používá se méně často.

---

# 6. Pomocná tabulka pro M:N vztah

Vztah M:N nelze přímo vyjádřit v relační databázi.

Vytvoříme pomocnou tabulku, která obsahuje cizí klíče obou entit.

Příklad:

Tabulka `objednavka_produkt`:

| objednavka_id | produkt_id | pocet |
|--------------|------------|-------|
| 1            | 3          | 2     |
| 1            | 7          | 1     |
| 2            | 3          | 5     |

---

# 7. Nástroje pro modelování

## Oracle Data Modeler

Grafický nástroj pro návrh databázového modelu.

Umožňuje nakreslit ER diagram a vygenerovat SQL script.

---

## Další nástroje

- MySQL Workbench (EER Modeler)
- draw.io (pro jednoduché diagramy)
- dbdiagram.io (online)

---

# Shrnutí

Modelování = navrhnutí databáze před vytvořením.

Tři typy modelů: konceptuální, logický, fyzický.

ER diagram zobrazuje entity a jejich vztahy.

Typy vztahů: 1:N (nejčastější), M:N (řeší se pomocnou tabulkou), 1:1.

---

# Typické doplňující otázky

## Co je ER diagram?

Grafické znázornění tabulek a jejich vztahů v databázi.

---

## Jak se řeší vztah M:N?

Pomocnou (vazební) tabulkou, která obsahuje cizí klíče obou tabulek.

---

## Jaký je rozdíl mezi konceptuálním a fyzickým modelem?

Konceptuální = obecný popis bez technických detailů.
Fyzický = konkrétní SQL struktura pro daný DBMS.

---

## Co je entita?

Věc nebo objekt, o kterém ukládáme data. Odpovídá tabulce v databázi.
