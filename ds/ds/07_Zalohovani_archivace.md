# 1. Co je záloha databáze

Záloha (backup) je kopie dat, která slouží k jejich obnově při ztrátě nebo poškození.

Zálohy se pravidelně vytvářejí a ukládají na jiné místo než databáze.

---

# 2. Typy záloh

## Plná záloha (Full backup)

Zálohuje se celá databáze.

Výhody: jednoduchá obnova, kompletní data.

Nevýhody: velká velikost, trvá déle.

---

## Přírůstková záloha (Incremental)

Zálohují se jen změny od poslední zálohy (ať už plné nebo přírůstkové).

Výhody: rychlá, malá velikost.

Nevýhody: složitější obnova (potřebuješ všechny zálohy od poslední plné).

---

## Diferenciální záloha (Differential)

Zálohují se změny od poslední plné zálohy.

Výhody: jednodušší obnova než incremental.

Nevýhody: větší než incremental.

---

## Shrnutí rozdílů

Full → všechno.

Incremental → změny od poslední zálohy.

Differential → změny od poslední plné zálohy.

---

# 3. Zálohování v MySQL

Nejčastější nástroj: `mysqldump`.

Vytvoří SQL soubor se všemi příkazy pro obnovu databáze.

Příkaz pro zálohu celé databáze:

```
mysqldump -u root -p mojeDB > zaloha.sql
```

Příkaz pro zálohu jen struktury (bez dat):

```
mysqldump -u root -p --no-data mojeDB > struktura.sql
```

Záloha více databází najednou:

```
mysqldump -u root -p --all-databases > vsechny_db.sql
```

---

# 4. Obnova ze zálohy

Obnova znamená obnovení dat ze záložního souboru.

Příkaz pro obnovu:

```
mysql -u root -p mojeDB < zaloha.sql
```

---

# 5. Binární log (Binary Log)

MySQL binární log zaznamenává každou změnu dat.

Slouží pro:

- obnovu do konkrétního okamžiku (Point-In-Time Recovery)
- replikaci mezi servery

Zapnutí binárního logu: nastavuje se v konfiguračním souboru MySQL.

Point-In-Time Recovery (PITR): nejdříve obnova z plné zálohy, pak přehrání binárního logu do požadovaného okamžiku.

---

# 6. RPO a RTO

## RPO (Recovery Point Objective)

Maximální akceptovatelná ztráta dat.

Příklad: RPO = 1 hodina znamená, že smíme ztratit max. 1 hodinu dat.

To ovlivňuje frekvenci zálohování.

---

## RTO (Recovery Time Objective)

Maximální akceptovatelná doba obnovy.

Příklad: RTO = 4 hodiny znamená, že databáze musí být obnovena do 4 hodin.

---

# 7. Rozdíl mezi zálohou a archivací

## Záloha

Slouží k obnově dat při chybě.

Krátkodobá – přepisuje se.

Aktuální data.

---

## Archivace

Dlouhodobé uložení historických dat.

Nemění se.

Slouží pro evidenci, legislativní povinnosti.

Příklad: faktury musíme archivovat 10 let.

---

# 8. Kde ukládat zálohy

Zálohy nikdy nesmí být jen na stejném serveru jako databáze.

Možnosti uložení:

- jiný disk
- síťové úložiště (NAS)
- cloud (AWS S3, Azure Blob)
- zálohovací server

---

# Shrnutí

Záloha chrání data před ztrátou.

Typy: plná (vše), přírůstková (od poslední zálohy), diferenciální (od poslední plné).

mysqldump je základní nástroj pro zálohu MySQL.

RPO = max. ztráta dat, RTO = max. doba obnovy.

Záloha = krátkodobá ochrana, archivace = dlouhodobé uchování.

---

# Typické doplňující otázky

## Jaký je rozdíl mezi incremental a differential zálohou?

Incremental: změny od poslední zálohy (jakkoli).
Differential: změny od poslední plné zálohy.

---

## Co je mysqldump?

Nástroj pro zálohu MySQL databáze do SQL souboru.

---

## Co je PITR?

Point-In-Time Recovery – obnova databáze do konkrétního okamžiku pomocí binárního logu.

---

## Jaký je rozdíl mezi zálohou a archivací?

Záloha = obnova po chybě, krátkodobá.
Archivace = dlouhodobé uchování, historická data.
