# 8. Dědičnost, method overriding, function overloading

---

## Dědičnost

Dědičnost je mechanismus, kdy **třída (potomek) přebírá atributy a metody jiné třídy (rodiče)**. Modeluje vztah "IS-A" — Pes IS-A Zvíře.

### Proč dědičnost

- Sdílíš kód — napíšeš jednou, použiješ mnohokrát
- Hierarchická organizace — obecné na vrcholu, specifické dole
- Polymorfismus — potomci mohou přepsat chování rodiče

### Základní dědičnost

```python
class Zvire:
    def __init__(self, jmeno: str, vek: int):
        self.jmeno = jmeno
        self.vek = vek

    def dychej(self):
        print(f"{self.jmeno} dýchá")

    def info(self):
        return f"{self.jmeno}, {self.vek} let"

class Pes(Zvire):               # Pes dědí od Zvire
    def __init__(self, jmeno, vek, plemeno):
        super().__init__(jmeno, vek)   # zavolá __init__ rodiče
        self.plemeno = plemeno          # vlastní atribut

    def stekej(self):           # vlastní metoda
        print(f"{self.jmeno}: Haf!")

rex = Pes("Rex", 3, "Ovčák")
rex.dychej()       # zděděno od Zvire — "Rex dýchá"
rex.stekej()       # vlastní metoda — "Rex: Haf!"
print(rex.info())  # zděděno — "Rex, 3 let"

# isinstance — ověří typ objektu
print(isinstance(rex, Pes))     # True
print(isinstance(rex, Zvire))   # True — potomek IS-A rodič
print(type(rex) == Zvire)       # False — přesný typ
```

### `super()`

`super()` odkazuje na **rodiče** — umožní zavolat jeho metodu.

```python
class Kocka(Zvire):
    def info(self):
        zakladni = super().info()           # zavolá Zvire.info()
        return f"{zakladni} (kočka)"

micka = Kocka("Micka", 5)
print(micka.info())   # "Micka, 5 let (kočka)"
```

---

## Vícenásobná dědičnost

Python podporuje dědičnost od více rodičů. Pořadí prohledávání určuje **MRO (Method Resolution Order)**.

```python
class Letajici:
    def pohyb(self):
        return "Letím"

class Plavajici:
    def pohyb(self):
        return "Plavu"

class Kachna(Letajici, Plavajici):   # dědí od obou
    pass

k = Kachna()
print(k.pohyb())           # "Letím" — Letajici je první v MRO
print(Kachna.__mro__)      # ukáže pořadí prohledávání
```

---

## Method Overriding (přepsání metody)

Potomek může **přepsat (override)** metodu rodiče a poskytnout vlastní implementaci. Původní metoda rodiče stále existuje a lze ji zavolat přes `super()`.

```python
class Tvar:
    def obsah(self) -> float:
        return 0.0              # výchozí implementace

    def popis(self) -> str:
        return f"Tvar s obsahem {self.obsah():.2f}"

class Kruh(Tvar):
    def __init__(self, polomer: float):
        self.polomer = polomer

    def obsah(self) -> float:   # OVERRIDE — přepisujeme rodičovskou metodu
        return 3.14159 * self.polomer ** 2

class Obdelnik(Tvar):
    def __init__(self, a: float, b: float):
        self.a, self.b = a, b

    def obsah(self) -> float:   # OVERRIDE
        return self.a * self.b

# Polymorfismus — stejné volání, různý výsledek
tvary = [Kruh(5), Obdelnik(4, 6)]
for t in tvary:
    print(t.popis())   # volá přepsanou obsah() každého potomka
# Tvar s obsahem 78.54
# Tvar s obsahem 24.00
```

### `@property` override

```python
class Zvire:
    @property
    def zvuk(self) -> str:
        return "..."

class Pes(Zvire):
    @property
    def zvuk(self) -> str:      # override property
        return "Haf"

class Kocka(Zvire):
    @property
    def zvuk(self) -> str:
        return "Mňau"
```

---

## Abstraktní třídy a metody

Abstraktní třída definuje **rozhraní** — metody které MUSÍ potomek implementovat. Nelze z ní vytvořit instanci.

```python
from abc import ABC, abstractmethod

class Platba(ABC):
    @abstractmethod
    def proved_platbu(self, castka: float) -> bool:
        pass                    # žádná implementace

    @abstractmethod
    def nazev(self) -> str:
        pass

    def potvrzeni(self, castka: float) -> str:   # normální metoda (sdílená)
        return f"Platba {castka} Kč přes {self.nazev()}"

class PlatbaKartou(Platba):
    def proved_platbu(self, castka: float) -> bool:
        print(f"Platím kartou: {castka} Kč")
        return True

    def nazev(self) -> str:
        return "platební karta"

class PlatbaHotove(Platba):
    def proved_platbu(self, castka: float) -> bool:
        print(f"Platím hotově: {castka} Kč")
        return True

    def nazev(self) -> str:
        return "hotovost"

# platba = Platba()     # TypeError — nelze instancovat abstraktní třídu
p = PlatbaKartou()
p.proved_platbu(299)
print(p.potvrzeni(299))
```

---

## Function Overloading (přetěžování funkcí)

Overloading = **více funkcí se stejným názvem, ale různými parametry**. Java a C# to podporují nativně. **Python overloading přímo nepodporuje** — každá definice funkce přepíše předchozí.

### Jak to řešit v Pythonu

```python
# 1. Výchozí parametry (default parameters)
def pozdrav(jmeno: str, titul: str = "") -> str:
    if titul:
        return f"Dobrý den, {titul} {jmeno}"
    return f"Ahoj, {jmeno}"

print(pozdrav("Jan"))             # Ahoj, Jan
print(pozdrav("Novák", "Ing."))   # Dobrý den, Ing. Novák

# 2. *args — proměnný počet argumentů
def soucet(*cisla: int) -> int:
    return sum(cisla)

print(soucet(1, 2))          # 3
print(soucet(1, 2, 3, 4))    # 10

# 3. isinstance — různé chování podle typu
def zpracuj(data):
    if isinstance(data, str):
        return data.upper()
    elif isinstance(data, list):
        return [x * 2 for x in data]
    elif isinstance(data, int):
        return data ** 2
    raise TypeError(f"Nepodporovaný typ: {type(data)}")

print(zpracuj("hello"))     # HELLO
print(zpracuj([1, 2, 3]))   # [2, 4, 6]
print(zpracuj(5))           # 25

# 4. @singledispatch — pythonický overloading
from functools import singledispatch

@singledispatch
def popis(hodnota):
    return f"Neznámý typ: {type(hodnota)}"

@popis.register(int)
def _(hodnota: int):
    return f"Celé číslo: {hodnota}"

@popis.register(str)
def _(hodnota: str):
    return f"Řetězec délky {len(hodnota)}: '{hodnota}'"

@popis.register(list)
def _(hodnota: list):
    return f"Seznam s {len(hodnota)} prvky"

print(popis(42))             # Celé číslo: 42
print(popis("hello"))        # Řetězec délky 5: 'hello'
print(popis([1, 2, 3]))      # Seznam s 3 prvky
```

---

## Přehled — Overriding vs Overloading

| | Overriding | Overloading |
|---|---|---|
| Co je | Potomek přepíše metodu rodiče | Více metod se stejným názvem, různé parametry |
| Kdy se rozhoduje | Za běhu (runtime) | Při kompilaci (compile-time) — jen Java/C# |
| Python | Ano, nativně | Ne nativně; řeší se přes default params, *args, isinstance |
| Klíčový nástroj | `super()`, `@abstractmethod` | `@singledispatch`, `*args`, `isinstance` |

---

## Shrnutí

- **Dědičnost** = potomek přebírá atributy a metody rodiče; `super()` zavolá rodiče
- **Override** = potomek přepíše metodu rodiče vlastní implementací
- **Abstraktní třída** = definuje rozhraní; potomci musí implementovat `@abstractmethod`
- **Overloading** v Pythonu neexistuje nativně; řeší se výchozími parametry, `*args` nebo `@singledispatch`

---

## Typické doplňující otázky

### Jaký je rozdíl mezi override a overload?
Override = potomek přepisuje metodu rodiče (stejné jméno, stejné parametry, jiná implementace). Overload = více metod se stejným jménem ale různými parametry — Python to přímo nepodporuje.

### K čemu slouží `super()`?
Zavolá metodu nebo konstruktor rodiče. Typicky v `__init__` potomka pro inicializaci rodičovských atributů, nebo v overridu pro kombinaci chování rodiče a vlastního.

### Co je MRO?
Method Resolution Order — pořadí ve kterém Python prohledává třídy při vícenásobné dědičnosti. Určuje která metoda se zavolá při konfliktu názvů.

### Proč nelze instancovat abstraktní třídu?
Protože abstraktní metody nemají implementaci — byl by to neúplný objekt. ABC vynucuje aby potomci všechny abstraktní metody implementovali.
