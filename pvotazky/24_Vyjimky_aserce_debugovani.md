# 24. Výjimky, aserce a debugování

---

## Co jsou výjimky

Výjimka (exception) je objekt reprezentující **chybový stav**, který narušil normální běh programu. Python je objektový — všechny výjimky jsou třídy dědící z `BaseException`.

```
BaseException
├── SystemExit         ← sys.exit()
├── KeyboardInterrupt  ← Ctrl+C
└── Exception          ← "normální" chyby
    ├── ValueError     ← neplatná hodnota
    ├── TypeError      ← špatný typ
    ├── NameError      ← neexistující proměnná
    ├── IndexError     ← index mimo rozsah
    ├── KeyError       ← klíč nenalezen v dict
    ├── FileNotFoundError ← soubor neexistuje
    ├── ZeroDivisionError ← dělení nulou
    ├── AttributeError ← neexistující atribut/metoda
    ├── ImportError    ← nelze importovat modul
    └── RuntimeError   ← obecná runtime chyba
```

---

## Try / Except / Else / Finally

```python
def nacti_soubor(cesta: str) -> str:
    try:
        # Kód který může vyvolat výjimku
        with open(cesta, "r", encoding="utf-8") as f:
            obsah = f.read()

    except FileNotFoundError:
        # Zpracování konkrétní výjimky
        print(f"Soubor '{cesta}' neexistuje")
        return ""

    except PermissionError:
        # Více except bloků — zachytí různé výjimky
        print("Nemám oprávnění číst soubor")
        return ""

    except (ValueError, TypeError) as e:
        # Zachycení více typů najednou
        print(f"Chyba hodnoty nebo typu: {e}")
        return ""

    except Exception as e:
        # Obecná zachycovací síť (použij opatrně)
        print(f"Neočekávaná chyba: {type(e).__name__}: {e}")
        raise   # re-raise — propaguj dál

    else:
        # Provede se POUZE pokud try proběhl BEZ výjimky
        print(f"Načteno {len(obsah)} znaků")
        return obsah

    finally:
        # Provede se VŽDY (výjimka i bez)
        print("Pokus o načtení dokončen")
```

### Kdy použít else

```python
# else = kod ktery patri do try ale nechces ho mit "uvnitr try"
try:
    cislo = int(input("Zadej cislo: "))
except ValueError:
    print("To není číslo")
else:
    # jistota: cislo bylo úspěšně přiřazeno
    print(f"Zadal jsi: {cislo}")
```

---

## Vlastní výjimky

```python
# Hierarchie vlastních výjimek
class AplikaceError(Exception):
    """Základní výjimka aplikace."""
    pass

class ValidationError(AplikaceError):
    """Chyba validace vstupních dat."""
    def __init__(self, pole: str, zprava: str):
        self.pole = pole
        self.zprava = zprava
        super().__init__(f"Validace '{pole}': {zprava}")

class DatabazoveError(AplikaceError):
    """Chyba při komunikaci s databází."""
    def __init__(self, dotaz: str, pricina: str):
        self.dotaz = dotaz
        super().__init__(f"DB chyba pro dotaz '{dotaz}': {pricina}")

# Použití
def validuj_email(email: str) -> str:
    if "@" not in email:
        raise ValidationError("email", f"'{email}' neobsahuje @")
    if len(email) < 5:
        raise ValidationError("email", "Email je příliš krátký")
    return email.strip()

try:
    validuj_email("nevalidni")
except ValidationError as e:
    print(f"Pole: {e.pole}")    # email
    print(f"Chyba: {e.zprava}")  # 'nevalidni' neobsahuje @
except AplikaceError as e:
    print(f"Obecná chyba aplikace: {e}")
```

---

## Context Manager a výjimky

```python
class SpravaSouboru:
    def __init__(self, cesta: str):
        self.cesta = cesta

    def __enter__(self):
        self.soubor = open(self.cesta, "w", encoding="utf-8")
        return self.soubor

    def __exit__(self, typ_vyjimky, hodnota_vyjimky, traceback):
        self.soubor.close()
        if typ_vyjimky is not None:
            print(f"Výjimka: {typ_vyjimky.__name__}: {hodnota_vyjimky}")
        return False   # False = výjimka se propaguje dál; True = potlačí se

with SpravaSouboru("test.txt") as f:
    f.write("Ahoj!")
    # i při výjimce __exit__ zaručí zavření souboru
```

---

## Aserce

`assert` je kontrolní mechanismus pro **invarianty a předpoklady**. Používá se v debug módu — lze je zakázat (`python -O skript.py`).

```python
def vypocitej_prumer(cisla: list) -> float:
    assert len(cisla) > 0, "Seznam nesmí být prázdný"   # kontrola předpokladu
    return sum(cisla) / len(cisla)

# v testování
def test_secti():
    assert secti(2, 3) == 5, "2 + 3 musí být 5"
    assert secti(-1, 1) == 0, "-1 + 1 musí být 0"

# NESPRAVNE pouziti aserce (nepoužívej pro validaci vstupu od uzivatele!)
# Aserce lze vypnout python -O → bezpecnostni riziko
def spatne(castka: float):
    assert castka > 0   # SPATNE — pouzij if + ValueError misto toho

def spravne(castka: float):
    if castka <= 0:
        raise ValueError(f"Částka musí být kladná, got {castka}")
```

---

## Traceback — čtení chybových výpisů

```
Traceback (most recent call last):         ← začíná od kořene
  File "app.py", line 15, in main          ← volající funkce
    vysledek = spocitej(data)
  File "app.py", line 8, in spocitej       ← hloubší úroveň
    return soucet / pocet
ZeroDivisionError: division by zero        ← typ a zpráva výjimky
```

Čtení: **poslední řádek = typ a zpráva výjimky**. Traceback čti zdola nahoru — nejbližší příčina je dole.

```python
import traceback

try:
    1 / 0
except ZeroDivisionError:
    # Získej traceback jako string
    tb = traceback.format_exc()
    print(tb)
    # nebo ulož do logu
```

---

## Debugování — nástroje

### print debugging (základní)

```python
def vypocet(data: list) -> float:
    print(f"DEBUG: vstup = {data}, len = {len(data)}")   # rychlé ladění
    vysledek = sum(data) / len(data)
    print(f"DEBUG: vysledek = {vysledek}")
    return vysledek
```

### Logging pro debugging

```python
import logging

# Nastavení — DEBUG úroveň pro výpis ladících zpráv
logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s [%(levelname)s] %(funcName)s: %(message)s"
)

logger = logging.getLogger(__name__)

def vypocet(data: list) -> float:
    logger.debug(f"Vstup: {data}")
    if not data:
        logger.error("Prázdný seznam!")
        raise ValueError("Prázdný seznam")
    vysledek = sum(data) / len(data)
    logger.debug(f"Výsledek: {vysledek}")
    return vysledek
```

### pdb — Python Debugger

```python
import pdb

def problematicka_funkce(data):
    pdb.set_trace()   # spustí interaktivní debugger (nebo breakpoint())
    # ... kód ...

# pdb příkazy:
# n (next)     — vykonej aktuální řádek
# s (step)     — vstup do volané funkce
# c (continue) — pokračuj do dalšího breakpointu
# p proměnná   — vytiskni hodnotu
# l (list)     — zobraz kód kolem aktuálního řádku
# q (quit)     — ukonči debugger
```

### Breakpoint() (Python 3.7+)

```python
def neco_spatne(x):
    mezivysledek = x * 2
    breakpoint()   # automaticky použije pdb nebo IDE debugger
    return mezivysledek + 1
```

---

## Shrnutí

- `try/except/else/finally` — zachytávání výjimek; `else` pro kód bez výjimky; `finally` vždy
- Vlastní výjimky = třídy dědící z `Exception`; dávají smysluplnou hierarchii
- `assert` = kontrola invariantů v kódu; **ne** pro validaci vstupu uživatele
- Traceback čti **zdola nahoru** — typ výjimky je poslední řádek
- Debugging: print → logging → `breakpoint()` / pdb → IDE debugger

---

## Typické doplňující otázky

### Jaký je rozdíl mezi assert a raise?
`assert` lze zakázat (`python -O`) — je pro interní invarianty a development. `raise` nelze zakázat — je pro validaci vstupů, chybové stavy v produkčním kódu.

### Co je stack unwinding?
Při výjimce Python prochází call stack zdola nahoru, hledá except blok. Každá funkce na zásobníku dostane šanci výjimku zachytit. Pokud žádný except nenajde, program skončí s tracebackem.

### Kdy re-raise výjimku (raise bez argumentu)?
Když chceš zalogovat výjimku ale nechat ji propagovat dál. Nebo v obecném `except Exception` — nechceš výjimku pohltit, jen reagovat.

### Co dělá finally při return ve try?
`finally` se spustí VŽDY — i před return ve try nebo except. Pokud finally obsahuje vlastní return, přepíše return z try.
