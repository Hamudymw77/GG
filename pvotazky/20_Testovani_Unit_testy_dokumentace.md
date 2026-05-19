# 20. Testování, unit testy, dokumentace

---

## Co je testování a proč je důležité

Testování ověřuje, že software **funguje správně** a splňuje požadavky. Bez testů:
- Chyby se odhalí až u zákazníka (drahé)
- Refactoring je riskantní — nevíš co rozbils
- Tým ztrácí důvěru v kód
- "Funguje to na mém počítači"

### Pyramida testování

```
         /\
        /  \
       / E2E\       ← End-to-End (málo, drahé, pomalé)
      /------\
     /Integr. \     ← Integrační (střední počet)
    /----------\
   / Unit testy \   ← Jednotkové (hodně, rychlé, levné)
  /______________\
```

---

## Unit testy

Unit test testuje **nejmenší izolovanou jednotku** kódu — typicky jednu funkci nebo metodu. Izolovaná = ostatní závislosti jsou nahrazeny (mock).

### Unittest — standardní knihovna

```python
import unittest

# Funkce k otestování
def secti(a: float, b: float) -> float:
    return a + b

def vydel(a: float, b: float) -> float:
    if b == 0:
        raise ValueError("Dělení nulou!")
    return a / b

class TestKalkulacka(unittest.TestCase):

    def test_secti_kladna_cisla(self):
        self.assertEqual(secti(2, 3), 5)

    def test_secti_zaporna_cisla(self):
        self.assertEqual(secti(-1, -1), -2)

    def test_secti_nula(self):
        self.assertEqual(secti(0, 5), 5)

    def test_vydel_normalni(self):
        self.assertAlmostEqual(vydel(10, 3), 3.333, places=2)

    def test_vydel_nulou_vyvola_vyjimku(self):
        with self.assertRaises(ValueError):
            vydel(10, 0)

    def test_vydel_nulou_spravna_zprava(self):
        with self.assertRaises(ValueError) as ctx:
            vydel(5, 0)
        self.assertIn("Dělení nulou", str(ctx.exception))

if __name__ == "__main__":
    unittest.main(verbosity=2)
```

### Assert metody

| Metoda | Co testuje |
|---|---|
| `assertEqual(a, b)` | a == b |
| `assertNotEqual(a, b)` | a != b |
| `assertTrue(x)` | bool(x) is True |
| `assertFalse(x)` | bool(x) is False |
| `assertIsNone(x)` | x is None |
| `assertIn(a, b)` | a in b |
| `assertRaises(Exc)` | volání vyvolá výjimku |
| `assertAlmostEqual(a, b, places)` | zaokrouhlení pro float |

---

## Pytest — populárnější alternativa

Pytest je čistší a výkonnější než unittest. Testy jsou normální funkce (nepotřebuje třídy).

```python
# test_kalkulacka.py
import pytest

def vydel(a, b):
    if b == 0:
        raise ValueError("Dělení nulou!")
    return a / b

# Základní test
def test_vydel_normalni():
    assert vydel(10, 2) == 5.0

# Parametrizace — spustí test pro každou sadu parametrů
@pytest.mark.parametrize("a, b, ocekavany", [
    (10, 2, 5.0),
    (9, 3, 3.0),
    (-6, 2, -3.0),
    (0, 5, 0.0),
])
def test_vydel_parametrizovany(a, b, ocekavany):
    assert vydel(a, b) == pytest.approx(ocekavany)

# Testování výjimek
def test_vydel_nulou():
    with pytest.raises(ValueError, match="Dělení nulou"):
        vydel(5, 0)

# Fixture — sdílené nastavení pro více testů
@pytest.fixture
def vzorovy_seznam():
    return [1, 2, 3, 4, 5]

def test_prvni_prvek(vzorovy_seznam):
    assert vzorovy_seznam[0] == 1

def test_delka(vzorovy_seznam):
    assert len(vzorovy_seznam) == 5
```

```bash
# Spuštění testů (v terminálu)
pytest test_kalkulacka.py -v    # verbose výstup
pytest --tb=short                # kratší traceback
pytest -k "vydel"               # jen testy obsahující "vydel" v názvu
```

---

## Mock — nahrazení závislostí

Mock nahradí závislost (databáze, API, soubor) simulovaným objektem — test běží izolovaně a rychle.

```python
from unittest.mock import Mock, patch, MagicMock

# Testování funkce závislé na externím API
import requests

def ziskej_pocasi(mesto: str) -> str:
    odpoved = requests.get(f"https://api.pocasi.cz/{mesto}")
    if odpoved.status_code == 200:
        data = odpoved.json()
        return f"{data['teplota']}°C"
    return "Chyba"

# Test s mockem — nevolá reálné API
def test_ziskej_pocasi():
    mock_response = Mock()
    mock_response.status_code = 200
    mock_response.json.return_value = {"teplota": 22}

    with patch("requests.get", return_value=mock_response):
        vysledek = ziskej_pocasi("Praha")

    assert vysledek == "22°C"

def test_ziskej_pocasi_chyba():
    mock_response = Mock()
    mock_response.status_code = 500

    with patch("requests.get", return_value=mock_response):
        vysledek = ziskej_pocasi("Praha")

    assert vysledek == "Chyba"
```

---

## TDD — Test-Driven Development

TDD je přístup, kde **testy se píší před kódem**.

```
1. Napiš test (ČERVENÝ — selže, kód neexistuje)
2. Napiš minimální kód aby test prošel (ZELENÝ)
3. Refaktoruj kód (REFACTOR — testy stále zelené)
4. Opakuj
```

```python
# Krok 1: Napiš test (selže — funkce neexistuje)
def test_validace_email():
    assert validuj_email("test@example.com") == True
    assert validuj_email("nevalidni") == False
    assert validuj_email("") == False

# Krok 2: Minimální implementace
import re

def validuj_email(email: str) -> bool:
    if not email:
        return False
    return bool(re.match(r"^[^@]+@[^@]+\.[^@]+$", email))

# Krok 3: Test projde — případně refaktoruj
```

---

## Integrační a E2E testy

### Integrační testy

Testují spolupráci více komponent dohromady.

```python
# Integrační test — testuje skutečnou databázi
import sqlite3
import pytest

def vytvor_db():
    conn = sqlite3.connect(":memory:")   # in-memory DB pro testy
    conn.execute("CREATE TABLE uzivatele (id INTEGER PRIMARY KEY, jmeno TEXT)")
    return conn

def vloz_uzivatele(conn, jmeno: str) -> int:
    cursor = conn.execute("INSERT INTO uzivatele (jmeno) VALUES (?)", (jmeno,))
    conn.commit()
    return cursor.lastrowid

def ziskej_uzivatele(conn, user_id: int):
    row = conn.execute("SELECT * FROM uzivatele WHERE id = ?", (user_id,)).fetchone()
    return row

def test_vloz_a_ziskej_uzivatele():
    conn = vytvor_db()
    user_id = vloz_uzivatele(conn, "Jan")
    uzivatel = ziskej_uzivatele(conn, user_id)

    assert uzivatel is not None
    assert uzivatel[1] == "Jan"   # jmeno
```

---

## Dokumentace

### Docstring

```python
def vydel(a: float, b: float) -> float:
    """
    Vydeli dve cisla.

    Args:
        a: Delenec (cislo ktery delime).
        b: Delitel (nesmi byt nula).

    Returns:
        Vysledek deleni jako float.

    Raises:
        ValueError: Pokud je b == 0.

    Examples:
        >>> vydel(10, 2)
        5.0
        >>> vydel(-6, 3)
        -2.0
    """
    if b == 0:
        raise ValueError("Deleni nulou!")
    return a / b

# Doctest — testy v docstringu (spustitelne)
import doctest
doctest.testmod()   # najde a spusti >>> priklady
```

### Typy docstringů

```python
# Google style
def funkce(param: int) -> str:
    """Kratky popis.

    Args:
        param: Popis parametru.

    Returns:
        Popis navratove hodnoty.
    """

# NumPy style (populárni v data science)
def funkce(param: int) -> str:
    """
    Kratky popis.

    Parameters
    ----------
    param : int
        Popis parametru.

    Returns
    -------
    str
        Popis navratove hodnoty.
    """
```

---

## Shrnutí

- **Unit test** = test jedné funkce v izolaci; rychlé, levné, hodně jich
- **unittest** = standardní knihovna, třídy + assert metody
- **pytest** = čistší syntax, fixtures, parametrizace, populárnější
- **Mock** = náhrada závislostí (API, DB) pro izolaci testu
- **TDD** = nejdřív test (červený) → implementace (zelený) → refactor
- **Docstring** = dokumentace přímo v kódu; doctest umí spustit příklady

---

## Typické doplňující otázky

### Proč isolovat unit testy od databáze a API?
Aby testy byly rychlé (ms ne sekundy), deterministické (nezávisí na stavu externího systému) a spustitelné offline. Reálné závislosti testuj v integračních testech.

### Co je code coverage?
Procento řádků kódu pokrytých testy. `pytest --cov` měří coverage. 100% coverage neznamená že kód funguje správně — ale pomáhá najít netestované větve.

### Jaký je rozdíl mezi pytest fixture a setUp/tearDown?
`setUp`/`tearDown` se spouští před/po každém testu ve třídě (unittest). Pytest fixtures jsou flexibilnější — lze je sdílet mezi soubory, nastavit scope (function/class/module/session).

### Co je regression test?
Test přidaný po opravě chyby. Zajišťuje, že ta konkrétní chyba se nevrátí. Každá chyba by měla vést k novému testu.
