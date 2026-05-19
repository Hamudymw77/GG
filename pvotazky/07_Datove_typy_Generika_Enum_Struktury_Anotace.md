# 7. Datové typy, generika, výčtové typy, struktury, anotace, operátory

---

## Datové typy v Pythonu

Datový typ určuje **jaký druh hodnoty** proměnná uchovává a jaké operace lze provádět.

### Základní (primitivní) typy

```python
# Celá čísla — int (neomezená přesnost v Pythonu)
vek = 25
velke_cislo = 10 ** 100   # Python zvládne

# Desetinná čísla — float (IEEE 754, 64-bit)
pi = 3.14159
print(0.1 + 0.2)   # 0.30000000000000004 — floating point nepřesnost!

# Přesná desetinná čísla — Decimal (pro finance)
from decimal import Decimal
print(Decimal("0.1") + Decimal("0.2"))   # 0.3 — přesně

# Logická hodnota — bool (podtyp int v Pythonu)
je_aktivni = True
print(type(True))   # <class 'bool'>
print(True + 1)     # 2 — bool je int!

# Textový řetězec — str (immutable)
jmeno = "Jan"
print(jmeno[0])     # J — indexování
print(jmeno[::-1])  # naJ — obrácení

# Žádná hodnota
x = None            # ekvivalent null/nil
```

### Složené typy

```python
# Seznam — list (mutable, řazený, duplicity povoleny)
cisla = [1, 2, 3, 2]
cisla.append(4)      # [1, 2, 3, 2, 4]

# N-tice — tuple (immutable, řazená)
souradnice = (10, 20)
# souradnice[0] = 5   # TypeError!

# Množina — set (mutable, neřazená, bez duplicit)
unikatni = {1, 2, 3, 2, 1}
print(unikatni)   # {1, 2, 3}

# Slovník — dict (mutable, klíč→hodnota, od Python 3.7 zachovává pořadí)
osoba = {"jmeno": "Jan", "vek": 25}
print(osoba["jmeno"])   # Jan
```

### Mutabilita

Klíčový koncept — zda lze objekt po vytvoření měnit.

```python
# Immutable (neměnné) — int, float, str, tuple, bool
a = "hello"
b = a
a += " world"
print(b)   # "hello" — b se nezměnilo, vznikl nový objekt

# Mutable (měnitelné) — list, dict, set
x = [1, 2, 3]
y = x            # y odkazuje na STEJNÝ objekt
y.append(4)
print(x)         # [1, 2, 3, 4] — x se změnilo!
```

---

## Typové anotace

Python je dynamicky typovaný, ale od verze 3.5 podporuje **volitelné typové anotace**. Neprovádí kontrolu za běhu, ale pomáhají editorům a nástrojům jako mypy.

```python
# Bez anotací
def soucet(a, b):
    return a + b

# S anotacemi
def soucet(a: int, b: int) -> int:
    return a + b

def pozdrav(jmeno: str, vek: int = 0) -> str:
    return f"Ahoj {jmeno}, je ti {vek} let"

# Složitější typy
from typing import List, Dict, Optional, Tuple, Union

def zpracuj(data: List[int]) -> Dict[str, int]:
    return {"soucet": sum(data), "pocet": len(data)}

def najdi(seznam: List[str], cil: str) -> Optional[int]:
    try:
        return seznam.index(cil)
    except ValueError:
        return None   # Optional = může být None

def kombinovany(x: Union[int, str]) -> str:   # int nebo str
    return str(x)
```

---

## Generika (Generic types)

Generika umožňují psát kód, který funguje s **různými typy** aniž bys duplikoval logiku.

```python
from typing import TypeVar, Generic, List

T = TypeVar("T")   # typová proměnná — zástupce pro libovolný typ

# Generická třída — zásobník pro libovolný typ
class Zasobnik(Generic[T]):
    def __init__(self):
        self._data: List[T] = []

    def push(self, item: T) -> None:
        self._data.append(item)

    def pop(self) -> T:
        return self._data.pop()

    def je_prazdny(self) -> bool:
        return len(self._data) == 0

# Použití se specifickými typy
int_zasobnik: Zasobnik[int] = Zasobnik()
int_zasobnik.push(1)
int_zasobnik.push(2)
print(int_zasobnik.pop())   # 2

str_zasobnik: Zasobnik[str] = Zasobnik()
str_zasobnik.push("hello")
```

```python
# Generická funkce — vrátí první prvek libovolného seznamu
def prvni(seznam: List[T]) -> T:
    return seznam[0]

print(prvni([1, 2, 3]))       # 1 (int)
print(prvni(["a", "b"]))      # "a" (str)
```

---

## Výčtové typy (Enum)

Enum definuje množinu **pojmenovaných konstant**. Zabraňuje "magic strings/numbers" v kódu a usnadňuje čtení.

```python
from enum import Enum, auto

class Barva(Enum):
    CERVENA = 1
    ZELENA = 2
    MODRA = 3

class Smer(Enum):
    SEVER = auto()   # auto() přiřadí hodnotu automaticky
    JIH = auto()
    VYCHOD = auto()
    ZAPAD = auto()

# Použití
oblibena = Barva.CERVENA
print(oblibena)         # Barva.CERVENA
print(oblibena.name)    # CERVENA
print(oblibena.value)   # 1

# Porovnání
if oblibena == Barva.CERVENA:
    print("Je červená")

# Iterace přes všechny hodnoty
for barva in Barva:
    print(barva.name, barva.value)

# Praktické použití — stav objednávky
class StavObjednavky(Enum):
    NOVA = "nova"
    ZPRACOVANA = "zpracovana"
    ODESLANA = "odeslana"
    DORUCENA = "dorucena"
    ZRUSENA = "zrusena"

objednavka_stav = StavObjednavky.NOVA
# Místo: stav = "nova" — špatně, typo těžko odhalitelné
```

---

## Datové třídy (dataclass) — struktury

`@dataclass` automaticky generuje `__init__`, `__repr__`, `__eq__` a další. Slouží jako lightweight struktura pro data.

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class Bod:
    x: float
    y: float

    def vzdalenost_od_pocatku(self) -> float:
        return (self.x**2 + self.y**2) ** 0.5

b1 = Bod(3.0, 4.0)
b2 = Bod(3.0, 4.0)
print(b1)              # Bod(x=3.0, y=4.0) — automatický __repr__
print(b1 == b2)        # True — automatický __eq__
print(b1.vzdalenost_od_pocatku())  # 5.0

@dataclass
class Student:
    jmeno: str
    vek: int
    predmety: List[str] = field(default_factory=list)   # mutable default

s = Student("Jan", 20)
s.predmety.append("Matematika")
```

---

## Operátory

### Aritmetické

```python
print(10 + 3)   # 13  sčítání
print(10 - 3)   # 7   odčítání
print(10 * 3)   # 30  násobení
print(10 / 3)   # 3.333...  dělení (vždy float)
print(10 // 3)  # 3   celočíselné dělení
print(10 % 3)   # 1   zbytek po dělení
print(10 ** 3)  # 1000 umocnění
```

### Porovnávací a logické

```python
print(5 == 5)   # True   rovnost hodnot
print(5 is 5)   # True   stejný objekt (identity)
print(5 != 3)   # True   nerovnost
print([1] == [1])  # True  (hodnota stejná)
print([1] is [1])  # False (různé objekty)

# Logické
print(True and False)   # False
print(True or False)    # True
print(not True)         # False

# Zkrácené vyhodnocování (short-circuit)
x = None
print(x and x.value)   # None — x.value se ani nevyhodnotí (x je False)
```

### Bitové operátory

```python
a, b = 0b1010, 0b1100   # binárně 10 a 12
print(a & b)   # 0b1000 = 8   (AND)
print(a | b)   # 0b1110 = 14  (OR)
print(a ^ b)   # 0b0110 = 6   (XOR)
print(~a)      # -11           (NOT)
print(a << 1)  # 0b10100 = 20 (posun vlevo = násobení 2)
print(a >> 1)  # 0b0101 = 5   (posun vpravo = dělení 2)
```

---

## Shrnutí

- Python má dynamické typování — typ se určí za běhu, ne při deklaraci
- **Typové anotace** jsou volitelné, neprovádějí kontrolu za běhu, pomáhají editorům
- **Generika** umožňují typově bezpečný kód pro libovolný typ (TypeVar)
- **Enum** = pojmenované konstanty; zabraňuje magic strings/numbers
- **@dataclass** = automaticky generuje __init__, __repr__, __eq__; jako struktura pro data

---

## Typické doplňující otázky

### Jaký je rozdíl mezi `list` a `tuple`?
List je mutable (lze měnit), tuple je immutable (nelze). Tuple je rychlejší a lze jej použít jako klíč ve slovníku.

### Proč `0.1 + 0.2 != 0.3` v Pythonu?
Float je uložen v binárním formátu IEEE 754, kde 0.1 nelze přesně reprezentovat. Pro přesné výpočty (finance) použij `Decimal`.

### Co je duck typing?
Python kontroluje ne typ objektu, ale zda má požadované metody. "Pokud to chodí jako kachna a kváká jako kachna, je to kachna."

### K čemu slouží `Optional[T]`?
Říká že parametr nebo návratová hodnota může být buď typ T nebo None. Ekvivalent `Union[T, None]`.
