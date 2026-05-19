# 1. Co je integrita dat

Integrita dat znamená, že data v databázi jsou správná, úplná a konzistentní.

Databáze musí zabránit vložení špatných dat.

Příklad: nesmí být možné zadat objednávku pro zákazníka, který neexistuje.

---

# 2. Typy integritních omezení

## Primární klíč (PRIMARY KEY)

Zajistí, že každý řádek je unikátní.

Hodnota nesmí být NULL ani se opakovat.

Příklad: `id INT PRIMARY KEY`

---

## Cizí klíč (FOREIGN KEY)

Zajistí, že hodnota v jedné tabulce odkazuje na existující záznam v jiné tabulce.

Příklad: `zakaznik_id` v objednávce musí existovat v tabulce zákazník.

Pokud zákazník neexistuje, nelze objednávku vložit.

---

## Unikátnost (UNIQUE)

Zajistí, že hodnota se v sloupci neopakuje, ale může být NULL.

Příklad: email zákazníka musí být unikátní.

`email VARCHAR(150) UNIQUE`

---

## Nenulová hodnota (NOT NULL)

Zajistí, že sloupec musí vždy obsahovat hodnotu.

Příklad: jméno zákazníka nemůže být prázdné.

`jmeno VARCHAR(100) NOT NULL`

---

## Výchozí hodnota (DEFAULT)

Pokud není hodnota zadána, použije se výchozí.

Příklad: stav objednávky = 'nová' pokud není uvedeno jinak.

`stav VARCHAR(20) DEFAULT 'nova'`

---

## Podmíněné omezení (CHECK)

Hodnota musí splňovat zadanou podmínku.

Příklad: cena musí být větší než nula.

`cena DECIMAL(10,2) CHECK (cena > 0)`

---

# 3. Referenční integrita

Referenční integrita řeší, co se stane s objednávkami, pokud smažeme zákazníka.

## Možnosti chování (ON DELETE):

### CASCADE

Smazáním zákazníka se smažou i jeho objednávky.

`FOREIGN KEY (zakaznik_id) REFERENCES zakaznik(id) ON DELETE CASCADE`

---

### RESTRICT

Zákazník nejde smazat, pokud má objednávky.

`FOREIGN KEY (zakaznik_id) REFERENCES zakaznik(id) ON DELETE RESTRICT`

---

### SET NULL

Smazáním zákazníka se nastaví `zakaznik_id = NULL` v objednávkách.

`FOREIGN KEY (zakaznik_id) REFERENCES zakaznik(id) ON DELETE SET NULL`

---

### NO ACTION

Stejné chování jako RESTRICT – výchozí chování v MySQL.

---

# 4. Příklad: co se stane bez integritního omezení

Bez cizího klíče bychom mohli vložit objednávku pro zákazníka s id = 999, který neexistuje.

Pak bychom měli "osiřelé" záznamy, které nepatří nikomu.

S cizím klíčem databáze takový INSERT odmítne.

---

# 5. Kontrola v MySQL

MySQL kontroluje integritní omezení automaticky při každém INSERT, UPDATE a DELETE.

Pokud je omezení porušeno, databáze vrátí chybu a operaci nespustí.

---

# Shrnutí

Integrita dat zajišťuje správnost a konzistenci dat.

PRIMARY KEY: unikátní identifikátor, nesmí být NULL.

FOREIGN KEY: odkazuje na existující záznam v jiné tabulce.

UNIQUE: hodnota se neopakuje.

NOT NULL: hodnota musí být zadána.

CHECK: hodnota splňuje podmínku.

DEFAULT: výchozí hodnota pokud není zadána.

---

# Typické doplňující otázky

## Co je referenční integrita?

Zajišťuje, že cizí klíč vždy odkazuje na existující záznam. Databáze nám nedovolí vložit nebo smazat záznamy, které by porušily tuto pravidlo.

---

## Jaký je rozdíl mezi PRIMARY KEY a UNIQUE?

PRIMARY KEY: nesmí být NULL, jen jeden na tabulku.
UNIQUE: může být NULL, na tabulce může být více UNIQUE sloupců.

---

## Co dělá ON DELETE CASCADE?

Při smazání záznamu se automaticky smažou i závislé záznamy v jiných tabulkách.

---

## Co dělá ON DELETE RESTRICT?

Zabrání smazání záznamu, pokud na něj odkazují záznamy v jiné tabulce.
