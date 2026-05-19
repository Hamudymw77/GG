# 15. Programovací jazyky — překlad, interpretace, typování, paradigmata

---

## Co je programovací jazyk

Programovací jazyk je formální jazyk pro zápis instrukcí, které může počítač vykonat. Různé jazyky se liší v:
- **způsobu překladu** (kompilace vs interpretace)
- **typovém systému** (statické vs dynamické typování)
- **paradigmatu** (OOP, funkcionální, imperativní...)
- **úrovni abstrakce** (nízkoúrovňové vs vysokoúrovňové)

---

## Způsob překladu

### Kompilace

Zdrojový kód je **přeložen celý najednou** do strojového kódu (nebo bajtkódu) před spuštěním. Výsledkem je spustitelný soubor.

```
Zdrojový kód (.c, .cpp, .java)
        │
        ▼
    Kompilátor
        │
        ▼
Strojový kód / bajtkód
        │
        ▼
    Spuštění
```

**Výhody:** Rychlé spuštění, chyby odhaleny před spuštěním, optimalizace.
**Příklady:** C, C++, Go, Rust, Java (do bajtkódu JVM)

### Interpretace

Zdrojový kód je **vykonáván řádek po řádku** za běhu — interpret čte instrukce a okamžitě je provádí.

```
Zdrojový kód (.py, .js, .rb)
        │
        ▼
    Interpret (za běhu)
        │
        ▼
    Vykonání
```

**Výhody:** Přenositelnost, snadnější vývoj a ladění, platformě nezávislé.
**Nevýhody:** Pomalejší, chyby se projeví až za běhu.
**Příklady:** Python, JavaScript (v prohlížeči), Ruby

### Hybridní přístup — Python

Python kombinuje oba přístupy:

```
Zdrojový kód (.py)
        │
        ▼
  Kompilátor Pythonu
        │
        ▼
  Bajtkód (.pyc)   ← optimalizovaná mezilehlá reprezentace
        │
        ▼
  CPython VM (PVM)  ← interpret bajtkódu
        │
        ▼
    Výsledek
```

### JIT (Just-In-Time) kompilace

Kombinace: interpret překládá za běhu, ale **nejčastěji používané části kompiluje** do strojového kódu za běhu (Java HotSpot, PyPy, JavaScript V8).

---

## Fáze překladu

Bez ohledu na jazyk, překlad probíhá v podobných krocích:

### 1. Lexikální analýza (Lexer/Tokenizer)

Vstupní text se rozloží na **tokeny** (základní stavební kameny):

```python
# Zdrojový kód:
x = 42 + y

# Tokeny:
[IDENTIFIER:'x'] [ASSIGN:'='] [NUMBER:42] [PLUS:'+'] [IDENTIFIER:'y']
```

### 2. Syntaktická analýza (Parser)

Tokeny se uspořádají do **AST (Abstract Syntax Tree)** — stromová reprezentace struktury programu:

```
    =
   / \
  x   +
     / \
    42   y
```

### 3. Sémantická analýza

Ověření **smysluplnosti** — typová kontrola, existence proměnných, správnost volání funkcí.

### 4. Generování kódu

Ze stromu se generuje **cílový kód** (strojový kód, bajtkód, nebo jiný jazyk).

---

## Typový systém

### Statické typování

Typy jsou **určeny při kompilaci** — musíte deklarovat typ proměnné, kompilátor ověřuje typy před spuštěním.

```java
// Java — staticky typovaný
int cislo = 42;         // musím specifikovat typ
String text = "ahoj";
cislo = "chyba";        // CHYBA při kompilaci!
```

**Výhody:** Chyby odhaleny dříve, lepší IDE podpora, výkonnější.
**Jazyky:** C, C++, Java, Go, Rust, TypeScript

### Dynamické typování

Typy jsou **určeny za běhu** — proměnná nemá pevný typ, může se měnit.

```python
# Python — dynamicky typovaný
cislo = 42        # int
cislo = "ahoj"    # ted je str — v pohodě
cislo = [1, 2, 3] # a ted list

def secti(a, b):
    return a + b   # funguje pro int i float i str

secti(1, 2)       # 3
secti("a", "b")   # "ab"
secti(1, "x")     # TypeError — ale AŽ ZA BĚHU
```

### Silné vs slabé typování

- **Silné** (Python, Java) — implicitní konverze typů jsou omezené, `1 + "1"` = chyba
- **Slabé** (JavaScript, C) — agresivní implicitní konverze, `1 + "1"` = `"11"` nebo `2`

### Typové anotace v Pythonu

Python přidal anotace pro dokumentaci a statické analýzy (mypy):

```python
def pozdrav(jmeno: str, vek: int) -> str:
    return f"Ahoj {jmeno}, je ti {vek} let"

# mypy zkontroluje typy bez spuštění programu
# Ale Python samotný anotace NEVYNUCUJE za běhu
result: str = pozdrav("Jan", 25)
```

---

## Programovací paradigmata

### Imperativní (procedurální)

Popisuješ **jak** má program postupovat — krok po kroku.

```python
cisla = [5, 2, 8, 1, 9]
vysledek = []
for c in cisla:
    if c > 4:
        vysledek.append(c * 2)
# [10, 16, 18]
```

### Objektově orientované (OOP)

Program organizován do **objektů** — datové struktury s metodami.

```python
class Konto:
    def __init__(self, majitel: str):
        self._majitel = majitel
        self._zustatek = 0.0

    def vloz(self, castka: float) -> None:
        self._zustatek += castka

    def zustatek(self) -> float:
        return self._zustatek

konto = Konto("Jan")
konto.vloz(1000)
print(konto.zustatek())   # 1000.0
```

### Funkcionální

Výpočet jako **transformace dat pomocí funkcí**. Žádný měnitelný stav, žádné vedlejší efekty.

```python
from functools import reduce

cisla = [5, 2, 8, 1, 9]
vysledek = list(map(lambda x: x * 2, filter(lambda x: x > 4, cisla)))
# [10, 16, 18]

# reduce — akumulace
soucet = reduce(lambda acc, x: acc + x, cisla, 0)
# 25
```

---

## Srovnání jazyků

| Vlastnost | Python | Java | C |
|---|---|---|---|
| Typ překladu | Interpretovaný (bajtkód) | Kompilovaný (JVM bajtkód) | Kompilovaný (strojový kód) |
| Typování | Dynamické, silné | Statické, silné | Statické, slabé |
| Správa paměti | Garbage collector | Garbage collector | Manuální (malloc/free) |
| Rychlost | Pomalý | Rychlý | Nejrychlejší |
| Produktivita | Velmi vysoká | Střední | Nízká |
| Paradigma | Multi (OOP, funkc.) | Primárně OOP | Procedurální |

---

## Shrnutí

- **Kompilace** = překlad celého programu před spuštěním; rychlejší běh, statická kontrola
- **Interpretace** = řádek po řádku za běhu; pomalejší, ale flexibilní
- **Python** = interpretovaný přes bajtkód (CPython VM); dynamicky typovaný
- Fáze překladu: **Lexer → Parser (AST) → Sémantika → Kód**
- **Statické typování** (Java/C) = typy při kompilaci; **dynamické** (Python/JS) = za běhu
- Paradigmata: imperativní, OOP, funkcionální — Python podporuje všechna

---

## Typické doplňující otázky

### Jaký je rozdíl mezi kompilovaným a interpretovaným jazykem?
Kompilovaný = přeložen celý najednou před spuštěním (rychlejší runtime, chyby dřív). Interpretovaný = prováděn řádek po řádku za běhu (flexibilnější, pomalejší).

### Proč je Python pomalejší než C?
C kompiluje přímo do strojového kódu CPU. Python interpretuje bajtkód přes CPython VM, která přidává overhead navíc. Navíc Python má dynamické typování — typ každé operace musí být zjišťován za běhu.

### Co je AST?
Abstract Syntax Tree — stromová reprezentace struktury zdrojového kódu. Mezikrok při překladu; parser ze sekvence tokenů vytvoří strom odrážející syntaktickou strukturu programu.

### Kdy preferovat statické typování?
Pro velké projekty a týmy — statické typy slouží jako dokumentace a kompilátor zachytí chyby dříve než zákazník. Pro malé skripty je dynamické typování rychlejší na psaní.
