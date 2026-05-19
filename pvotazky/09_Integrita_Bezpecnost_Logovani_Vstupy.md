# 9. Integrita dat, bezpečnost, logování, kontrola vstupů, zpracování chyb

---

## Integrita dat

Integrita dat zajišťuje, že data jsou **správná, konzistentní a nepoškozená**. Rozlišujeme:

- **Entitní integrita** — každý záznam musí být jednoznačně identifikatelný (primární klíč)
- **Referenční integrita** — cizí klíče musí odkazovat na existující záznamy
- **Doménová integrita** — hodnoty musí být v přípustném rozsahu (validace)
- **Uživatelsky definovaná integrita** — business pravidla (věk > 0, email musí obsahovat @)

```python
from dataclasses import dataclass
from typing import Optional

@dataclass
class Uzivatel:
    jmeno: str
    email: str
    vek: int

    def __post_init__(self):
        # Validace integrity dat při vytvoření objektu
        if not self.jmeno or len(self.jmeno) < 2:
            raise ValueError("Jméno musí mít alespoň 2 znaky")
        if "@" not in self.email:
            raise ValueError("Neplatný email")
        if self.vek < 0 or self.vek > 150:
            raise ValueError("Neplatný věk")

# u = Uzivatel("J", "neplatny", -1)   # ValueError
u = Uzivatel("Jan", "jan@test.cz", 25)   # OK
```

---

## Bezpečnost

### Hašování hesel

Hesla **nikdy neukládej v plaintextu**. Použij kryptografický hash s náhodnou solí.

```python
import hashlib
import secrets

def hashuj_heslo(heslo: str) -> str:
    sul = secrets.token_hex(16)           # náhodná 16-bajtová sůl
    hashed = hashlib.sha256((sul + heslo).encode()).hexdigest()
    return f"{sul}:{hashed}"             # uložíme i sůl

def over_heslo(heslo: str, ulozeny_hash: str) -> bool:
    sul, hashed = ulozeny_hash.split(":")
    test_hash = hashlib.sha256((sul + heslo).encode()).hexdigest()
    return test_hash == hashed

ulozeny = hashuj_heslo("MojeHeslo123")
print(over_heslo("MojeHeslo123", ulozeny))   # True
print(over_heslo("SpatneHeslo", ulozeny))    # False
```

### SQL Injection — nejčastější bezpečnostní chyba

SQL injection nastane když uživatelský vstup je vložen přímo do SQL dotazu.

```python
import sqlite3

# ŠPATNĚ — SQL injection
def najdi_uzivatele_spatne(jmeno):
    conn = sqlite3.connect("db.sqlite")
    # Útočník může zadat: ' OR '1'='1 → vrátí všechny záznamy
    dotaz = f"SELECT * FROM uzivatele WHERE jmeno = '{jmeno}'"
    return conn.execute(dotaz).fetchall()

# SPRÁVNĚ — parametrizované dotazy (prepared statements)
def najdi_uzivatele_spravne(jmeno):
    conn = sqlite3.connect("db.sqlite")
    dotaz = "SELECT * FROM uzivatele WHERE jmeno = ?"
    return conn.execute(dotaz, (jmeno,)).fetchall()   # databáze ošetří vstup
```

### Sanitizace vstupu

```python
import re
import html

def sanitizuj_text(vstup: str) -> str:
    # HTML escape — zabraňuje XSS útokům
    return html.escape(vstup)

def validuj_email(email: str) -> bool:
    vzor = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(vzor, email))

def validuj_telefon(tel: str) -> bool:
    vzor = r'^\+?[\d\s\-]{9,15}$'
    return bool(re.match(vzor, tel))

print(sanitizuj_text("<script>alert('xss')</script>"))
# &lt;script&gt;alert(&#x27;xss&#x27;)&lt;/script&gt;
print(validuj_email("jan@test.cz"))   # True
print(validuj_email("neplatny"))      # False
```

---

## Zpracování chyb (výjimky)

### Hierarchie výjimek v Pythonu

```
BaseException
├── SystemExit
├── KeyboardInterrupt
└── Exception
    ├── ValueError      — neplatná hodnota
    ├── TypeError       — špatný typ
    ├── KeyError        — klíč neexistuje ve slovníku
    ├── IndexError      — index mimo rozsah
    ├── FileNotFoundError
    ├── ZeroDivisionError
    └── ...
```

### try/except/else/finally

```python
def nacti_cislo(text: str) -> int:
    try:
        return int(text)                # může vyvolat ValueError
    except ValueError:
        print(f"'{text}' není číslo")
        return 0

def zapis_soubor(jmeno: str, obsah: str) -> None:
    soubor = None
    try:
        soubor = open(jmeno, "w", encoding="utf-8")
        soubor.write(obsah)
    except PermissionError:
        print("Nemáš právo zapisovat")
    except OSError as e:
        print(f"Chyba: {e}")
    else:
        print("Zápis úspěšný")   # else se spustí jen pokud NENASTALA výjimka
    finally:
        if soubor:
            soubor.close()        # finally se spustí VŽDY (čistění zdrojů)
```

### Vlastní výjimky

```python
class ValidacniChyba(Exception):
    def __init__(self, pole: str, zprava: str):
        super().__init__(f"Pole '{pole}': {zprava}")
        self.pole = pole
        self.zprava = zprava

class BankovniChyba(Exception):
    pass

class NedostatekProstredku(BankovniChyba):
    def __init__(self, pozadovano: float, dostupne: float):
        super().__init__(f"Požadováno: {pozadovano}, dostupné: {dostupne}")

def vyber(zustatek: float, castka: float) -> float:
    if castka > zustatek:
        raise NedostatekProstredku(castka, zustatek)
    return zustatek - castka

try:
    vyber(100, 500)
except NedostatekProstredku as e:
    print(f"Chyba: {e}")
```

---

## Logování

`logging` modul je správná cesta k záznamu co se v programu děje. **Nepoužívej `print()` pro ladění v produkci.**

### Úrovně logování

```
DEBUG    — detailní info pro vývoj
INFO     — standardní provozní informace
WARNING  — neočekávaná situace, ale program pokračuje
ERROR    — závažná chyba, operace selhala
CRITICAL — kritická chyba, program se možná ukončí
```

```python
import logging

# Konfigurace loggeru
logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    handlers=[
        logging.StreamHandler(),                             # výstup do konzole
        logging.FileHandler("aplikace.log", encoding="utf-8")  # výstup do souboru
    ]
)

logger = logging.getLogger(__name__)

def transfer(z_uctu: str, na_ucet: str, castka: float) -> bool:
    logger.info(f"Zahájení transferu {castka} Kč z {z_uctu} na {na_ucet}")
    try:
        if castka <= 0:
            logger.warning(f"Pokus o transfer záporné částky: {castka}")
            return False
        # ... logika transferu ...
        logger.info(f"Transfer úspěšně dokončen")
        return True
    except Exception as e:
        logger.error(f"Transfer selhal: {e}", exc_info=True)   # exc_info přidá traceback
        return False
```

### Hierarchie loggerů

```python
# Každý modul má vlastní logger
logger = logging.getLogger(__name__)   # jméno = název modulu

# Rodiče a potomci sdílejí konfiguraci
logger_db = logging.getLogger("app.databaze")
logger_api = logging.getLogger("app.api")
# Oba jsou potomci "app" loggeru — dědí jeho nastavení
```

---

## Kontrola vstupů — komplexní příklad

```python
from typing import Optional
import re

class FormularRegistrace:
    def __init__(self, jmeno: str, email: str, heslo: str, vek: int):
        chyby = self._validuj(jmeno, email, heslo, vek)
        if chyby:
            raise ValueError(f"Validace selhala: {'; '.join(chyby)}")
        self.jmeno = jmeno.strip()
        self.email = email.lower().strip()
        self.vek = vek

    def _validuj(self, jmeno, email, heslo, vek) -> list:
        chyby = []

        if not jmeno or len(jmeno.strip()) < 2:
            chyby.append("Jméno příliš krátké")

        if not re.match(r'^[^@]+@[^@]+\.[^@]+$', email):
            chyby.append("Neplatný email")

        if len(heslo) < 8:
            chyby.append("Heslo musí mít alespoň 8 znaků")
        if not any(c.isupper() for c in heslo):
            chyby.append("Heslo musí obsahovat velké písmeno")
        if not any(c.isdigit() for c in heslo):
            chyby.append("Heslo musí obsahovat číslo")

        if not (0 < vek < 150):
            chyby.append("Neplatný věk")

        return chyby
```

---

## Shrnutí

- **Integrita dat** = správnost a konzistence dat; validace při vstupu
- **SQL Injection** = nejčastější bezpečnostní chyba; řeš parametrizovanými dotazy
- **Hesla** nikdy v plaintextu — hašuj s náhodnou solí (sha256 + secrets)
- **try/except/finally** — except chytí výjimku, finally se spustí vždy (zavírání zdrojů)
- **Vlastní výjimky** — dědí od Exception, přidávají kontext
- **logging** — místo print(); úrovně DEBUG/INFO/WARNING/ERROR/CRITICAL

---

## Typické doplňující otázky

### Co je SQL injection a jak mu předejít?
Útočník vloží SQL kód do formuláře místo dat. Řešením jsou parametrizované dotazy — databáze zpracuje vstup jako data, ne jako SQL.

### Jaký je rozdíl mezi `except Exception` a `except BaseException`?
`Exception` chytí většinu programových chyb. `BaseException` chytí i `SystemExit` a `KeyboardInterrupt` — to obvykle nechceme zastavovat.

### Proč používat `finally`?
Kód v `finally` se spustí vždy — i při výjimce. Ideální pro uvolnění zdrojů (zavření souboru, databázového spojení).

### Proč nepoužívat `print()` pro logování?
`print()` neumí úrovně, formátování, výstup do souboru ani filtrování. `logging` je konfigurovatelný, dá se centrálně vypnout a přesměrovat.
