# 1. Co je normalizace

Normalizace je proces, při kterém upravujeme strukturu databáze tak, aby:

- data nebyla zbytečně opakovaná
- změna dat na jednom místě stačila
- nevznikaly nekonzistentní data

Normalizace probíhá v několika krocích, kterým říkáme normální formy.

---

# 2. Proč normalizovat

Bez normalizace se stávají tyto problémy:

## Redundance

Data jsou opakovaná zbytečně.

Příklad: jméno zákazníka je v každé řádce objednávky.

---

## Anomálie při vkládání

Nemůžeme vložit produkt bez objednávky, pokud jsou v jedné tabulce.

---

## Anomálie při mazání

Smazáním objednávky ztratíme i informace o zákazníkovi.

---

## Anomálie při úpravě

Zákazník změní email a musíme ho změnit na 50 místech.

---

# 3. Nulová normální forma (0NF)

Tabulka bez jakékoliv normalizace. Data jsou zmuchlaná do jedné tabulky.

Příklad špatné tabulky:

| id | zakaznik | email | produkty | ceny |
|----|---------|-------|---------|------|
| 1  | Pavel   | p@t   | Notebook, Myš | 12990, 299 |
| 2  | Eva     | e@t   | Notebook | 12990 |

Problém: více hodnot v jednom sloupci.

---

# 4. První normální forma (1NF)

Každý sloupec musí obsahovat jednu hodnotu. Žádné opakující se skupiny.

Každý řádek musí být unikátní (musí existovat primární klíč).

Opravená tabulka:

| id | zakaznik | email | produkt  | cena  |
|----|---------|-------|---------|-------|
| 1  | Pavel   | p@t   | Notebook | 12990 |
| 2  | Pavel   | p@t   | Myš      | 299   |
| 3  | Eva     | e@t   | Notebook | 12990 |

Problém: Pavel a jeho email se opakují.

---

# 5. Druhá normální forma (2NF)

Splňuje 1NF.

Každý sloupec závisí na celém primárním klíči, ne jen na jeho části.

Platí hlavně pro tabulky se složeným primárním klíčem.

Řešení: oddělíme zákazníka do vlastní tabulky.

Tabulka zákazník:

| id | jmeno | email |
|----|-------|-------|
| 1  | Pavel | p@t   |
| 2  | Eva   | e@t   |

Tabulka objednávka:

| id | zakaznik_id | produkt  | cena  |
|----|------------|---------|-------|
| 1  | 1          | Notebook | 12990 |
| 2  | 1          | Myš      | 299   |
| 3  | 2          | Notebook | 12990 |

Problém: cena produktu se opakuje.

---

# 6. Třetí normální forma (3NF)

Splňuje 2NF.

Žádný sloupec nesmí záviset na jiném sloupci, který není primárním klíčem.

Řešení: oddělíme produkt do vlastní tabulky.

Tabulka produkt:

| id | nazev    | cena  |
|----|---------|-------|
| 1  | Notebook | 12990 |
| 2  | Myš      | 299   |

Tabulka polozka_objednavky:

| objednavka_id | produkt_id | pocet |
|--------------|------------|-------|
| 1             | 1          | 1     |
| 1             | 2          | 1     |
| 2             | 1          | 1     |

Teď je vše oddělené a žádná data se neopakují.

---

# 7. Kdy normalizovat a kdy ne

Normalizace je obecně správná praxe pro transakční databáze.

Ale v datových skladech (DWH) se někdy záměrně denormalizuje pro rychlost dotazů.

---

# Shrnutí

Normalizace odstraňuje opakující se data a anomálie.

1NF: každý sloupec = jedna hodnota, unikátní řádky.

2NF: každý sloupec závisí na celém primárním klíči.

3NF: žádná závislost mezi neklíčovými sloupci.

Výsledek: data na jednom místě, snadná údržba.

---

# Typické doplňující otázky

## Co je anomálie?

Problém při vkládání, mazání nebo úpravě dat způsobený špatnou strukturou tabulky.

---

## Jaký je rozdíl mezi 2NF a 3NF?

2NF: každý sloupec závisí na celém klíči (ne jen části).
3NF: žádný neklíčový sloupec nezávisí na jiném neklíčovém sloupci.

---

## Proč se někdy denormalizuje?

Pro výkon – v datových skladech je rychlejší číst z jedné velké tabulky než z mnoha spojených.
