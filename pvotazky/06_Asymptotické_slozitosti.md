# 6. Asymptotické paměťové a časové složitosti

---

## Co je složitost algoritmu a proč nás zajímá

Když píšeme program, chceme vědět: **jak moc bude pomalý nebo náročný na paměť, když vstup bude velký?**

Složitost říká, jak roste čas nebo paměť **v závislosti na velikosti vstupu (n)**.

- **Časová složitost** — kolik operací algoritmus provede
- **Prostorová (paměťová) složitost** — kolik paměti algoritmus potřebuje

Příklad: máš seznam 10 čísel → hledání trvá chvíli. Máš seznam 10 000 000 čísel → záleží na algoritmu, jestli to bude vteřina nebo hodiny.

---

## Big-O notace — O(...)

Nejčastěji používaná notace. Popisuje **nejhorší případ** — jak rychle roste čas při velkém n.

Píšeme například `O(n)` — čte se "O n" nebo "big-O n".

**Důležité pravidlo:** konstanty a menší členy ignorujeme.
- `O(3n + 5)` → zapíšeme jako `O(n)`
- `O(n² + n)` → zapíšeme jako `O(n²)` (kvadrát dominuje)

---

## Přehled složitostí od nejlepší po nejhorší

```
O(1)       — konstantní      — nejrychlejší
O(log n)   — logaritmická    — velmi rychlá
O(n)       — lineární        — přijatelná
O(n log n) — lineárně-log.   — přijatelná (efektivní řazení)
O(n²)      — kvadratická     — pomalá pro velká n
O(2^n)     — exponenciální   — prakticky nepoužitelná
```

---

## O(1) — konstantní složitost

Algoritmus provede **vždy stejný počet operací**, nezávisle na velikosti vstupu.

```python
seznam = [10, 20, 30, 40, 50]

# Přístup na index — vždy 1 operace, bez ohledu na délku seznamu
print(seznam[0])   # O(1)
print(seznam[4])   # O(1)

# Jednoduchý výpočet — vždy 1 operace
def je_sude(n):
    return n % 2 == 0   # O(1)
```

**Analogie:** Otevřít slovník na stránce 42 — nezáleží jak tlustý slovník je, prostě ho otevřeš.

---

## O(n) — lineární složitost

Algoritmus projde **každý prvek jednou**. Čím větší vstup, tím proporcionálně více práce.

```python
def najdi_max(seznam):
    maximum = seznam[0]
    for prvek in seznam:   # projde všechny prvky — n operací
        if prvek > maximum:
            maximum = prvek
    return maximum

# n=10 → 10 operací
# n=1000 → 1000 operací
# n=1 000 000 → 1 000 000 operací
```

**Analogie:** Hledáš klíče v krabici — musíš prohledat každou věc jednu po druhé.

---

## O(n²) — kvadratická složitost

Obvykle vzniká **dvěma vnořenými cykly**. Pokud je vstup 10× větší, práce je 100× větší.

```python
# Vypíše všechny dvojice čísel ze seznamu
def vsechny_pary(seznam):
    for a in seznam:       # n krát
        for b in seznam:   # n krát pro každé a
            print(a, b)    # celkem n × n = n² výpisů

vsechny_pary([1, 2, 3])
# (1,1) (1,2) (1,3)
# (2,1) (2,2) (2,3)
# (3,1) (3,2) (3,3)
# → 9 výpisů pro n=3, 100 pro n=10, 1 000 000 pro n=1000
```

**Analogie:** Každý ze 100 lidí si potřese rukou s každým dalším — 100 × 100 = 10 000 potřesení.

---

## O(log n) — logaritmická složitost

Vstup se každým krokem **zmenší na polovinu**. Extrémně rychlá i pro velká n.

```python
# Binární vyhledávání — hledáme v seřazeném seznamu
# Vždy vezmeme prostřední prvek a zahodíme polovinu

def binarni_hledani(arr, cil):
    levo, pravo = 0, len(arr) - 1
    while levo <= pravo:
        stred = (levo + pravo) // 2
        if arr[stred] == cil:
            return stred          # našli jsme
        elif arr[stred] < cil:
            levo = stred + 1      # zahodíme levou půlku
        else:
            pravo = stred - 1     # zahodíme pravou půlku
    return -1

# n=1 000 000 → stačí jen ~20 kroků!
# Proč? Protože 2^20 = 1 048 576 > 1 000 000
```

**Analogie:** Hledáš slovo ve slovníku — otevřeš uprostřed, víš jestli je před nebo za, zahodíš polovinu. Opakuješ.

---

## O(n log n) — lineárně logaritmická složitost

Typická pro efektivní řadicí algoritmy (Merge Sort, Quick Sort). Lepší než O(n²), ale horší než O(n).

```python
# Python sort() má O(n log n) — Timsort
seznam = [5, 2, 8, 1, 9, 3]
seznam.sort()   # O(n log n)
print(seznam)   # [1, 2, 3, 5, 8, 9]
```

---

## O(2^n) — exponenciální složitost

Každý krok **zdvojnásobí** počet operací. Rychle se stane nepoužitelnou.

```python
# Rekurzivní Fibonacci bez optimalizace
def fib(n):
    if n <= 1:
        return n
    return fib(n-1) + fib(n-2)   # každé volání se větví na dvě

# fib(10)  → cca 177 volání
# fib(30)  → cca 2 700 000 volání
# fib(50)  → cca 2 000 000 000 000 000 volání — prakticky nikdy nedoběhne
```

---

## Jak poznat složitost kódu — pravidla

```python
# Žádný cyklus → O(1)
x = 5 + 3

# Jeden cyklus přes n prvků → O(n)
for i in range(n):
    print(i)

# Dva vnořené cykly přes n → O(n²)
for i in range(n):
    for j in range(n):
        print(i, j)

# Cyklus kde i se násobí nebo dělí → O(log n)
i = 1
while i < n:
    i = i * 2   # 1, 2, 4, 8, 16... → log n kroků
```

---

## Prostorová (paměťová) složitost

Kolik **extra paměti** algoritmus potřebuje (mimo vstupní data).

```python
# O(1) prostorová — žádná extra paměť, jen pár proměnných
def soucet(seznam):
    total = 0          # jedna proměnná, nezáleží na délce seznamu
    for x in seznam:
        total += x
    return total

# O(n) prostorová — vytváříme nový seznam stejné délky jako vstup
def zdvoj(seznam):
    return [x * 2 for x in seznam]   # nový seznam délky n
```

---

## Praktické srovnání pro n = 1 000 000

| Složitost | Přibližný počet operací | Použitelné? |
|---|---|---|
| O(1) | 1 | Ano ✅ |
| O(log n) | ~20 | Ano ✅ |
| O(n) | 1 000 000 | Ano ✅ |
| O(n log n) | ~20 000 000 | Ano ✅ |
| O(n²) | 1 000 000 000 000 | Ne ❌ |
| O(2^n) | astronomické číslo | Absolutně ne ❌ |

---

## Shrnutí

- Složitost říká **jak rychle roste čas nebo paměť** s velikostí vstupu n
- Používáme **Big-O notaci** — zajímá nás nejhorší případ
- Konstanty ignorujeme: O(3n) = O(n)
- Bereme největší člen: O(n² + n) = O(n²)
- Nejlepší jsou O(1) a O(log n), nejhorší O(2^n)
- Dva vnořené cykly = O(n²), jeden cyklus = O(n), přímý přístup = O(1)

---

## Typické otázky u maturity

### Jaká je složitost přístupu do Python slovníku (dict)?
O(1) průměrně — dict je hash tabulka, klíč se přepočítá na index. Nemusí procházet vše.

### Jaká je složitost `in` pro seznam vs množinu?
- Seznam (`list`): O(n) — projde každý prvek
- Množina (`set`): O(1) průměrně — hash tabulka

### Proč preferovat O(n log n) nad O(n²) pro řazení?
Pro n = 10 000: O(n log n) ≈ 130 000 operací, O(n²) = 100 000 000 operací — rozdíl ~770×.
