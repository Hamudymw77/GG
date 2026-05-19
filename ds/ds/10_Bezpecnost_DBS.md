# 1. Co je bezpečnost databáze

Bezpečnost databáze chrání data před neoprávněným přístupem, ztrátou nebo zneužitím.

Zahrnuje správu uživatelů, přístupová práva, šifrování a ochranu před útoky.

---

# 2. Správa uživatelů

## Vytvoření uživatele

```sql
CREATE USER 'aplikace'@'localhost' IDENTIFIED BY 'silne_heslo';
```

Uživatel `aplikace` se může přihlásit jen z localhostu.

---

## Přidání práv (GRANT)

```sql
GRANT SELECT, INSERT, UPDATE ON mojeDB.* TO 'aplikace'@'localhost';
```

Uživatel může číst, vkládat a aktualizovat data v databázi mojeDB.

Nemůže mazat data ani měnit strukturu databáze.

---

## Odebrání práv (REVOKE)

```sql
REVOKE INSERT ON mojeDB.* FROM 'aplikace'@'localhost';
```

---

## Zrušení uživatele

```sql
DROP USER 'aplikace'@'localhost';
```

---

# 3. Princip minimálních práv

Každý uživatel nebo aplikace by měla mít jen ta práva, která nutně potřebuje.

Například:

- čtenářský web → jen SELECT
- reportovací nástroj → jen SELECT
- aplikace → SELECT, INSERT, UPDATE, DELETE
- správce → všechna práva

---

# 4. Role (MySQL 8+)

Role je pojmenovaná sada práv, kterou lze přiřadit uživatelům.

```sql
CREATE ROLE 'ctenar';
GRANT SELECT ON mojeDB.* TO 'ctenar';

GRANT 'ctenar' TO 'uzivatel1'@'localhost';
```

---

# 5. SQL Injection

SQL Injection je útok, kdy útočník do vstupního pole zadá SQL příkaz.

Pokud aplikace přímo vkládá vstup do dotazu, útočník může:

- zobrazit data, která by neměl vidět
- smazat data
- obejít přihlášení

---

## Příklad útoku

Nebezpečný kód (PHP):

```php
$query = "SELECT * FROM uzivatel WHERE email = '" . $_POST['email'] . "'";
```

Útočník zadá: `' OR '1'='1`

Výsledný dotaz:

```sql
SELECT * FROM uzivatel WHERE email = '' OR '1'='1'
```

Vrátí všechny záznamy, protože `'1'='1'` je vždy true.

---

## Ochrana: prepared statements (parametrizované dotazy)

```php
$stmt = $pdo->prepare("SELECT * FROM uzivatel WHERE email = ?");
$stmt->execute([$_POST['email']]);
```

Hodnota z formuláře se nikdy nevloží přímo do SQL. Databáze ji zpracuje odděleně jako data, ne jako kód.

---

# 6. Šifrování hesel

Hesla nikdy neukládáme jako prostý text.

Používáme jednosměrné hashování (SHA-256, bcrypt).

```sql
-- Uložení zahashovaného hesla
INSERT INTO uzivatel (email, heslo)
VALUES ('pavel@t.cz', SHA2('mojeheslo', 256));

-- Ověření při přihlášení
SELECT id FROM uzivatel
WHERE email = 'pavel@t.cz' AND heslo = SHA2('mojeheslo', 256);
```

---

# 7. Auditní log

Auditní log zaznamenává, kdo co a kdy změnil.

Slouží pro:

- zjištění, kdo smazal data
- regulatorní požadavky (GDPR, SOX)
- detekci podezřelého chování

Implementuje se pomocí triggerů nebo specializovaných nástrojů.

---

# 8. Rozdíly MySQL vs SQL Server

| Funkce | MySQL | SQL Server |
|--------|-------|-----------|
| Uživatelé | CREATE USER | CREATE LOGIN + CREATE USER |
| Práva | GRANT / REVOKE | GRANT / DENY / REVOKE |
| Role | Ano (MySQL 8+) | Ano |
| Šifrování | AES_ENCRYPT | ENCRYPTBYKEY |
| Audit | Plugin nebo triggery | SQL Server Audit |

---

# Shrnutí

Bezpečnost = správa uživatelů + práva + ochrana před útoky.

CREATE USER + GRANT: vytvoření uživatele a přidání práv.

Princip minimálních práv: uživatel má jen co nutně potřebuje.

SQL Injection: útok přes vstupní pole – ochrana = prepared statements.

Hesla se hashují (SHA2, bcrypt), nikdy neukládáme plaintext.

Auditní log zaznamenává změny dat.

---

# Typické doplňující otázky

## Co je SQL Injection?

Útok, kdy útočník vloží SQL příkaz do vstupního pole. Ochrání ho prepared statements.

---

## Co je prepared statement?

Parametrizovaný dotaz, kde se vstup od uživatele nezpracovává jako SQL kód, ale jako data.

---

## Proč hashovat hesla?

Aby útočník, který získá přístup k databázi, nemohl přečíst hesla uživatelů.

---

## Co je princip minimálních práv?

Každý uživatel nebo aplikace má jen ta práva, která skutečně potřebuje pro svou práci.
