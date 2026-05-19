# 3. Algoritmizace — Rekurze, Brute Force, Heuristiky, Nedeterministické algoritmy

---

## Rekurze

Rekurze je technika, kdy **funkce volá samu sebe**. Každá rekurzivní funkce musí mít:
- **Base case (základní případ)** — podmínka, kdy se přestane volat. Bez toho nastane nekonečná rekurze a `RecursionError`.
- **Rekurzivní případ** — volání sama sebe s menším/jednodušším vstupem.

### Jak to funguje — faktoriál

```
5! = 5 × 4!
4! = 4 × 3!
3! = 3 × 2!
2! = 2 × 1!
1! = 1   ← BASE CASE, stop
```

```python
def faktorial(n):
    if n == 0:                    # base case
        return 1
    return n * faktorial(n - 1)  # rekurzivní případ

print(faktorial(5))   # 120
```

### Co se děje v paměti (zásobník)

Každé volání funkce se uloží na zásobník a čeká. Jakmile se dosáhne base case, výsledky se postupně vracejí:

```
faktorial(3)
  → faktorial(2)
      → faktorial(1)
          → faktorial(0) → vrátí 1
        ← vrátí 1 × 1 = 1
    ← vrátí 2 × 1 = 2
  ← vrátí 3 × 2 = 6
```

Python má limit ~1000 volání → při překročení `RecursionError`. Proto pro velké vstupy je lepší smyčka.

### Fibonacci — rekurze vs memoizace

```python
# Rekurzivní — O(2^n), POMALÉ
def fib(n):
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)
```

Problém: počítá stejné hodnoty znovu a znovu:
```
fib(4)
├── fib(3)
│    ├── fib(2)   ← počítá se podruhé!
│    └── fib(1)
└── fib(2)        ← počítá se podruhé!
```

**Memoizace** = uložíme výsledky do cache, nepočítáme znovu:

```python
from functools import lru_cache

@lru_cache
def fib(n):
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)

print(fib(50))   # okamžitě — výsledky se ukládají
```

### Kdy rekurzi použít

- Problém má přirozenou rekurzivní strukturu — stromy, grafy, fraktály
- Kód je čitelnější než iterativní verze
- Vstup není příliš velký

---

## Brute Force (hrubá síla)

Brute Force = **vyzkoušej všechny možné kombinace** a vyber správnou. Vždy najde správné řešení, ale je pomalý pro velké vstupy.

### Příklad — najdi dvojice se součtem 10

```python
cisla = [1, 3, 5, 7, 9, 2]

for i in range(len(cisla)):
    for j in range(i + 1, len(cisla)):
        if cisla[i] + cisla[j] == 10:
            print(cisla[i], "+", cisla[j])
# 1 + 9
# 3 + 7
```

Složitost O(n²) — pro každý prvek projdeme všechny ostatní.

### Problém obchodního cestujícího (TSP)

Najdi nejkratší cestu procházející všemi městy právě jednou a vrať se na start.

Brute Force řešení = vyzkoušej všechny permutace měst:
```
4 města = 6 kombinací          → OK
10 měst = 362 880 kombinací    → pomalé
20 měst = 1 216 451 004 188 016 kombinací → nepraktické
```

Pro velké vstupy se proto používají heuristiky nebo aproximační algoritmy.

### Kdy Brute Force použít

- Vstup je malý (pár desítek prvků)
- Potřebujeme zaručeně správný výsledek
- Jako ověření pro testování jiného algoritmu

---

## Heuristiky

Heuristika = přibližný algoritmus. **Nenajde vždy optimální řešení**, ale najde dostatečně dobré řešení rychle. Používá se tam, kde přesné řešení trvá příliš dlouho.

### Greedy (chamtivý) algoritmus

Vždy vezmi **lokálně nejlepší volbu** bez zpětného pohledu. Rychlý, ale nemusí najít globální optimum.

**Příklad: vrácení mincí** — vrať 36 Kč, máš mince [25, 10, 5, 1]:

```
Greedy: vždy vezmi největší minci co se vejde
  36 - 25 = 11  → vezmu 25
  11 - 10 =  1  → vezmu 10
   1 -  1 =  0  → vezmu 1
Výsledek: [25, 10, 1] = 3 mince ✅
```

```python
def vrat_mince(castka, mince):
    mince.sort(reverse=True)   # od největší
    vysledek = []
    for m in mince:
        while castka >= m:
            vysledek.append(m)
            castka -= m
    return vysledek

print(vrat_mince(36, [25, 10, 5, 1]))   # [25, 10, 1]
```

**Kdy greedy selže** — pro mince [6, 4, 1] a částku 8:
```
Greedy: 6 + 1 + 1 = 3 mince
Optimum: 4 + 4   = 2 mince  ← greedy to nenašel!
```

Greedy se díval jen na aktuální krok, ne dopředu.

### A* algoritmus

Chytrý algoritmus pro hledání nejkratší cesty v grafu. Používá heuristickou funkci — odhad vzdálenosti do cíle. Kombinuje BFS (správnost) a greedy (rychlost). Používá ho GPS navigace.

```
f(n) = g(n) + h(n)
g(n) = skutečná vzdálenost od startu do n
h(n) = odhad vzdálenosti od n do cíle
```

---

## Nedeterministické algoritmy

Deterministický algoritmus = pro stejný vstup vždy stejný výstup, stejný průběh.

Nedeterministický algoritmus = **používá náhodu**. Pro stejný vstup může dát jiný výsledek. Ale výsledek je stále dostatečně správný nebo rychlý.

### Monte Carlo simulace

Používá velké množství náhodných pokusů k získání přibližného výsledku.

**Příklad: odhad čísla π** — náhodně házíme body do čtverce. Body co padnou do kruhu vs všechny body ≈ π/4:

```
+--------+
|  ****  |
| ****** |
| ****** |
|  ****  |
+--------+
body v kruhu / celkem = π/4
```

```python
import random

def odhadni_pi(pokusy):
    uvnitr = 0
    for _ in range(pokusy):
        x = random.uniform(-1, 1)
        y = random.uniform(-1, 1)
        if x**2 + y**2 <= 1:   # bod je v kruhu
            uvnitr += 1
    return 4 * uvnitr / pokusy

print(odhadni_pi(1000000))   # ~3.14159
```

Čím více pokusů, tím přesnější. Nikdy ale úplně přesný.

### Randomizovaný Quick Sort

Místo pevného pivotu vybereme pivot **náhodně**. Tím se vyhneme nejhoršímu případu O(n²), který nastane při seřazeném vstupu:

```python
import random

def quick_sort(arr):
    if len(arr) <= 1:
        return arr
    pivot = random.choice(arr)   # náhodný pivot
    mensi = [x for x in arr if x < pivot]
    rovne = [x for x in arr if x == pivot]
    vetsi = [x for x in arr if x > pivot]
    return quick_sort(mensi) + rovne + quick_sort(vetsi)
```

### P vs NP (bonus)

- **P problémy** — jdou vyřešit rychle (polynomiální čas). Př: řazení, hledání.
- **NP problémy** — řešení lze rychle **ověřit**, ale těžko **najít**. Př: TSP, lámání hesla.
- Otázka **P = NP?** je největší nevyřešený problém informatiky. Zatím se věří, že P ≠ NP.

---

## Srovnání přístupů

| Přístup | Rychlost | Přesnost | Příklad použití |
|---|---|---|---|
| Rekurze | záleží | ✅ správná | faktoriál, stromy, fibonacci |
| Brute Force | ❌ pomalý | ✅ vždy správný | malé vstupy, ověřování |
| Greedy (heuristika) | ✅ rychlý | ⚠️ přibližný | vrácení mincí, plánování trasy |
| A* | ✅ rychlý | ✅ optimální (s dobrou h) | GPS navigace |
| Monte Carlo | ✅ rychlý | ⚠️ přibližný | simulace, vědecké výpočty |
| Randomizovaný | ✅ rychlý | ✅ správný | Quick Sort s náhodným pivotem |

---

## Typické otázky u maturity

### Co musí mít každá rekurze?
Base case — podmínka kdy se přestane volat. Bez ní je nekonečná rekurze a program spadne s `RecursionError`.

### Co je memoizace?
Ukládáme výsledky volání funkce do cache. Pokud zavoláme funkci se stejným vstupem znovu, vrátí uložený výsledek. Fibonacci bez memoizace je O(2^n), s memoizací O(n).

### Kdy použít Brute Force a kdy heuristiku?
Brute Force pro malé vstupy kde potřebujeme zaručeně správný výsledek. Heuristiku pro velké vstupy kde stačí dostatečně dobré řešení rychle.

### Proč greedy nenajde vždy optimum?
Díví se jen na aktuální krok, ne dopředu. Volí lokálně nejlepší možnost, ale ta nemusí vést ke globálně nejlepšímu výsledku.
