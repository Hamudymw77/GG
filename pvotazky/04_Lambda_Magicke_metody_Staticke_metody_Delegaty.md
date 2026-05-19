# 4. Lambda, magické metody, statické metody, delegáty

---

## Lambda funkce (anonymní funkce)

Lambda je **jednořádková anonymní funkce** — nemá jméno, zapisuje se inline. Hodí se tam kde potřebuješ jednoduchou funkci na jedno použití.

### Syntaxe

```python
lambda parametry: výraz
```

```python
# Normální funkce
def soucet(a, b):
    return a + b

# Ekvivalentní lambda
soucet = lambda a, b: a + b

print(soucet(3, 5))   # 8
```

### Typické použití lambdy

```python
# Řazení podle klíče
lidi = [("Jan", 25), ("Eva", 20), ("Petr", 30)]
lidi.sort(key=lambda x: x[1])   # seřadí podle věku
print(lidi)   # [('Eva', 20), ('Jan', 25), ('Petr', 30)]

# map() — aplikuj funkci na každý prvek
cisla = [1, 2, 3, 4, 5]
dvojnasobky = list(map(lambda x: x * 2, cisla))
print(dvojnasobky)   # [2, 4, 6, 8, 10]

# filter() — vyfiltruj prvky splňující podmínku
suda = list(filter(lambda x: x % 2 == 0, cisla))
print(suda)   # [2, 4]

# sorted() s klíčem
slova = ["banán", "jablko", "kiwi", "ananas"]
serazena = sorted(slova, key=lambda s: len(s))   # podle délky
print(serazena)   # ['kiwi', 'banán', 'jablko', 'ananas']
```

### Omezení lambdy

Lambda může obsahovat jen **jeden výraz**, ne příkazy (žádný `if` blok, `for`, `return`). Pro složitější logiku použij normální funkci.

---

## Magické (dunder) metody

Magické metody (dunder = double underscore) jsou speciální metody s předem daným významem. Umožňují třídám přizpůsobit chování pro vestavěné operátory a funkce Pythonu.

### Nejdůležitější magické metody

```python
class Vektor:
    def __init__(self, x, y):        # konstruktor — volán při vytvoření objektu
        self.x = x
        self.y = y

    def __str__(self):               # volán při print() nebo str()
        return f"Vektor({self.x}, {self.y})"

    def __repr__(self):              # volán v interaktivním interpretu, debugging
        return f"Vektor(x={self.x}, y={self.y})"

    def __add__(self, other):        # volán při +
        return Vektor(self.x + other.x, self.y + other.y)

    def __sub__(self, other):        # volán při -
        return Vektor(self.x - other.x, self.y - other.y)

    def __mul__(self, skalar):       # volán při *
        return Vektor(self.x * skalar, self.y * skalar)

    def __len__(self):               # volán při len()
        return 2   # 2D vektor má 2 složky

    def __eq__(self, other):         # volán při ==
        return self.x == other.x and self.y == other.y

    def __abs__(self):               # volán při abs()
        return (self.x**2 + self.y**2) ** 0.5

v1 = Vektor(1, 2)
v2 = Vektor(3, 4)

print(v1)           # Vektor(1, 2)       ← __str__
print(v1 + v2)      # Vektor(4, 6)       ← __add__
print(v1 == v2)     # False              ← __eq__
print(abs(v2))      # 5.0                ← __abs__
```

### Kontextový manažer — `__enter__` a `__exit__`

```python
class SpravceSouboru:
    def __init__(self, jmeno, mod):
        self.jmeno = jmeno
        self.mod = mod

    def __enter__(self):          # volán při vstupu do with bloku
        self.soubor = open(self.jmeno, self.mod)
        return self.soubor

    def __exit__(self, *args):    # volán při opuštění with bloku (i při výjimce)
        self.soubor.close()

with SpravceSouboru("data.txt", "w") as f:
    f.write("Hello")
# soubor je automaticky zavřen
```

### Iterovatelnost — `__iter__` a `__next__`

```python
class Rozsah:
    def __init__(self, start, konec):
        self.aktualni = start
        self.konec = konec

    def __iter__(self):           # volán při for loop
        return self

    def __next__(self):           # volán pro každý prvek
        if self.aktualni >= self.konec:
            raise StopIteration
        hodnota = self.aktualni
        self.aktualni += 1
        return hodnota

for x in Rozsah(1, 5):
    print(x)   # 1 2 3 4
```

---

## Statické metody a metody třídy

### Instanční metoda (běžná)

Přijímá `self` — má přístup k instanci i třídě.

```python
class Pocitadlo:
    pocet = 0   # atribut třídy (sdílený všemi instancemi)

    def __init__(self):
        Pocitadlo.pocet += 1
        self.id = Pocitadlo.pocet   # atribut instance

    def info(self):          # instanční metoda — self = konkrétní instance
        return f"Instance #{self.id}"
```

### Metoda třídy — `@classmethod`

Přijímá `cls` (třídu) místo `self`. Má přístup k třídě, ne ke konkrétní instanci. Používá se jako alternativní konstruktor nebo pro operace nad třídou.

```python
class Datum:
    def __init__(self, rok, mesic, den):
        self.rok = rok
        self.mesic = mesic
        self.den = den

    @classmethod
    def z_retezce(cls, retezec):     # alternativní konstruktor
        rok, mesic, den = map(int, retezec.split("-"))
        return cls(rok, mesic, den)  # cls = Datum

d = Datum.z_retezce("2024-06-15")
print(d.rok, d.mesic, d.den)   # 2024 6 15
```

### Statická metoda — `@staticmethod`

Nemá ani `self` ani `cls`. Patří do třídy logicky, ale nepotřebuje přístup k instanci ani třídě. Jde o helper funkci.

```python
class Matematika:
    @staticmethod
    def je_prime(n):
        if n < 2:
            return False
        for i in range(2, int(n**0.5) + 1):
            if n % i == 0:
                return False
        return True

print(Matematika.je_prime(17))   # True — volá se přes třídu
```

| | Instanční | @classmethod | @staticmethod |
|---|---|---|---|
| První parametr | `self` | `cls` | (žádný) |
| Přístup k instanci | Ano | Ne | Ne |
| Přístup ke třídě | Ano (přes `self.__class__`) | Ano | Ne |
| Typické využití | Operace s instancí | Alternativní konstruktor | Helper funkce |

---

## Delegáty (v Pythonu — callable, first-class functions)

Python nemá klíčové slovo `delegate` jako C#, ale funkce jsou **first-class objekty** — lze je předávat jako parametry, ukládat do proměnných, vracet z funkcí.

### Funkce jako parametr

```python
def proved(operace, a, b):
    return operace(a, b)   # "delegát" — zavolá předanou funkci

def scitej(a, b): return a + b
def nasob(a, b):  return a * b

print(proved(scitej, 3, 4))   # 7
print(proved(nasob, 3, 4))    # 12
print(proved(lambda a, b: a ** b, 3, 4))   # 81
```

### Funkce vracející funkci (closure)

```python
def vytvor_nasobitel(faktor):
    def nasob(x):
        return x * faktor   # faktor je "uzavřen" v closure
    return nasob

trojnasobek = vytvor_nasobitel(3)
print(trojnasobek(5))   # 15
print(trojnasobek(10))  # 30
```

### `callable()` — ověření zda je objekt volatelný

```python
class Pozdrav:
    def __call__(self, jmeno):   # __call__ = objekt lze volat jako funkci
        return f"Ahoj, {jmeno}!"

p = Pozdrav()
print(callable(p))   # True
print(p("Jan"))      # Ahoj, Jan!
```

---

## Shrnutí

- **Lambda** = anonymní jednořádková funkce; typicky pro `map`, `filter`, `sort`
- **Magické metody** = přizpůsob chování třídy pro operátory (`__add__`, `__str__`, `__len__`...)
- **`@classmethod`** = přijímá `cls`, alternativní konstruktor
- **`@staticmethod`** = helper funkce patřící logicky do třídy, bez `self`/`cls`
- **Delegáty** v Pythonu = předávání funkcí jako parametrů (first-class functions)

---

## Typické doplňující otázky

### Jaký je rozdíl mezi `__str__` a `__repr__`?
`__str__` je pro čitelný výstup pro uživatele (volán `print()`). `__repr__` je pro vývojáře a debugging — měl by být jednoznačný a ideálně reprodukovatelný.

### Kdy použít `@classmethod` vs `@staticmethod`?
`@classmethod` když potřebuješ přístup ke třídě (alternativní konstruktor, dědičnost). `@staticmethod` když funkce patří logicky do třídy ale nepotřebuje ani `self` ani `cls`.

### Co je closure?
Funkce, která "si pamatuje" proměnné z okolního scope ve kterém byla vytvořena, i po jeho ukončení.

### Proč Python nemá přetěžování funkcí (overloading) jako Java?
Python je dynamicky typovaný — funkce neví co dostane, takže přetěžování nedává smysl. Místo toho se používají výchozí parametry a `*args`/`**kwargs`.
