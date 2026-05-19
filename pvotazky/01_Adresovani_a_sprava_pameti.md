# 1. Adresování a správa paměti

---

## Jak program pracuje s pamětí

Když spustíš program, operační systém mu přidělí část RAM. Tato paměť je rozdělena do několika oblastí, každá slouží jinému účelu.

### Oblasti paměti programu

- **Code segment (text)** — zde je uložen samotný kód programu (instrukce). Je read-only.
- **Data segment** — globální a statické proměnné.
- **Stack (zásobník)** — lokální proměnné a volání funkcí. Spravuje se automaticky, funguje jako LIFO (Last In, First Out).
- **Heap (halda)** — dynamicky alokovaná paměť. Spravuje se manuálně (C/C++) nebo automaticky garbage collectorem (Python, Java).

```
[ Code segment ]   ← instrukce programu (read-only)
[ Data segment ]   ← globální proměnné
[ Heap         ]   ← roste nahoru, dynamická alokace
      ↑
      ↓
[ Stack        ]   ← roste dolů, lokální proměnné, volání funkcí
```

---

## Stack vs Heap

### Stack

- Alokace a dealokace je automatická — při volání funkce se přidá rámec (frame), při návratu se odebere
- Velmi rychlý přístup
- Omezená velikost — přetečení = **StackOverflow** (typicky nekonečná rekurze)
- Ukládají se sem: lokální proměnné, parametry funkcí, návratové adresy

### Heap

- Dynamická alokace — paměť si "žádáš" za běhu programu
- Větší prostor než stack
- Pomalejší přístup
- V Pythonu jsou na heapu **všechny objekty**

```python
def funkce():
    x = 10          # x je na stacku (lokální proměnná)
    seznam = [1, 2] # seznam (objekt) je na heapu, x odkazuje na heap

funkce()
# po skončení funkce: x zaniká (stack se uvolní), seznam může být GC
```

---

## Reference a ukazatele

### Ukazatel (pointer) — C/C++

Ukazatel je proměnná, která **obsahuje adresu** jiné proměnné v paměti. Umožňuje přímý přístup k libovolné adrese — výkonné, ale nebezpečné.

```c
// C příklad (jen pro pochopení konceptu)
int x = 42;
int *ptr = &x;   // ptr obsahuje adresu x
*ptr = 100;      // změníme hodnotu přes adresu → x je teď 100
```

### Reference — Python

Python nepoužívá ukazatele. Každá proměnná je **reference** (odkaz) na objekt na heapu. Proměnná neobsahuje hodnotu, ale odkaz na objekt.

```python
a = [1, 2, 3]
b = a           # b odkazuje na STEJNÝ objekt jako a

b.append(4)
print(a)        # [1, 2, 3, 4] — a a b jsou stejný objekt!

# Jak udělat kopii (ne odkaz):
import copy
c = a.copy()        # mělká kopie — nový seznam, ale stejné prvky
d = copy.deepcopy(a)  # hluboká kopie — vše nové
```

### Mělká vs hluboká kopie

- **Mělká kopie** (`copy()`, `[:]`) — zkopíruje kontejner, ale vnitřní objekty sdílí
- **Hluboká kopie** (`deepcopy()`) — zkopíruje vše rekurzivně, nic nesdílí

```python
import copy

original = [[1, 2], [3, 4]]
melka = original.copy()
hluboka = copy.deepcopy(original)

original[0].append(99)
print(melka)    # [[1, 2, 99], [3, 4]] — vnitřní seznam sdílený!
print(hluboka)  # [[1, 2], [3, 4]]    — nezávislá kopie
```

---

## Garbage Collection

Garbage collector (GC) je mechanismus, který **automaticky uvolňuje paměť**, která se již nepoužívá. Programátor nemusí ručně dealokovat objekty.

### Jak Python GC funguje

Python používá dva mechanismy:

**1. Reference counting (počítání referencí)**

Každý objekt má čítač kolikrát na něj někdo odkazuje. Jakmile čítač klesne na 0, objekt je okamžitě uvolněn.

```python
import sys

a = [1, 2, 3]          # čítač = 1
b = a                  # čítač = 2
print(sys.getrefcount(a))  # 3 (a, b, a parametr getrefcount)

del b                  # čítač = 2
del a                  # čítač = 1 (stále getrefcount drží odkaz)
# po skončení → čítač = 0 → paměť uvolněna
```

**2. Cyclic GC (cyklický garbage collector)**

Reference counting nestačí na **cyklické reference** — objekty odkazující na sebe navzájem, jejichž čítač nikdy neklesne na 0.

```python
import gc

class Uzel:
    def __init__(self):
        self.odkaz = None

a = Uzel()
b = Uzel()
a.odkaz = b    # a → b
b.odkaz = a    # b → a  → cyklická reference!

del a
del b
# čítač obou je stále 1, reference counting nestačí
# Python cyclic GC toto detekuje a uvolní

gc.collect()   # manuální spuštění GC (normálně probíhá automaticky)
```

### Generace GC

Python GC třídí objekty do **3 generací**:
- **Gen 0** — nové objekty, kontrolovány nejčastěji
- **Gen 1** — přežily jednu kontrolu
- **Gen 2** — staré objekty, kontrolovány nejméně

Mladé objekty umírají rychle (krátkodobé proměnné), staré přežívají (globální objekty).

---

## Správa paměti v Pythonu — shrnutí chování

```python
# Immutable objekty (int, str, tuple) — Python je cachuje (interning)
a = 256
b = 256
print(a is b)   # True — stejný objekt v paměti (cache pro -5 až 256)

a = 1000
b = 1000
print(a is b)   # False — nové objekty (mimo cache rozsah)

# id() vrátí adresu objektu v paměti
x = "hello"
print(id(x))   # např. 140234567890
```

---

## Shrnutí

- Paměť programu je rozdělena na code, data, stack a heap
- **Stack** — automatický, rychlý, pro lokální proměnné
- **Heap** — dynamický, pro objekty; v Pythonu vše je na heapu
- Python používá **reference**, ne ukazatele — proměnná = odkaz na objekt
- **Garbage collector** automaticky uvolňuje paměť — reference counting + cyclic GC
- Mělká kopie sdílí vnitřní objekty, hluboká (`deepcopy`) je zcela nezávislá

---

## Typické doplňující otázky

### Co je stack overflow?
Přetečení zásobníku — typicky při nekonečné rekurzi. Každé volání funkce zabere místo na stacku, při přetečení program spadne.

### Proč Python nemá ukazatele?
Python je navržen pro bezpečnost a jednoduchost. Přímý přístup k paměti by umožnil nebezpečné operace. Reference jsou bezpečnější abstrakce.

### Co je rozdíl mezi `is` a `==`?
`==` porovnává hodnotu, `is` porovnává identitu (stejná adresa v paměti). `a is b` je True jen pokud jsou to doslova ten samý objekt.

### Kdy použít `deepcopy`?
Když chceš zcela nezávislou kopii objektu, který obsahuje vnořené objekty (seznam v seznamu, slovník se seznamy apod.).
