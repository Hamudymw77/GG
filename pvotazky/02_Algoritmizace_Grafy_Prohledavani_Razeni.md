# 2. Algoritmizace — Grafy, Prohledávání stavového prostoru, Řazení

---

## Co znamenají slova v názvu?

**Algoritmizace** = proces návrhu algoritmu. Algoritmus je přesný postup řešení problému — musí být konečný, jednoznačný a správný.

**Grafy** = datová struktura z uzlů a hran. Modeluje vztahy mezi věcmi (silnice, přátelství, závislosti).

**Prohledávání stavového prostoru** = hledání cesty nebo řešení v grafu. "Stavový prostor" = všechny možné stavy, kterými se lze dostat od startu k cíli. Prohledávání = procházení těchto stavů systematicky (BFS, DFS).

**Řazení** = seřazení prvků podle kritéria (vzestupně, abecedně...). Různé algoritmy mají různou rychlost.

---

## Grafy

Graf se skládá z **vrcholů (uzlů)** a **hran (spojení)**. Hrany mohou mít směr nebo váhu.

### Typy grafů

```
Neorientovaný:          Orientovaný:           Vážený:
   A --- B                 A --> B               A --5-- B
   |     |                 |                     |       |
   C --- D                 v                     3       2
                           C --> D               |       |
                                                 C --4-- D
```

- **Neorientovaný** — hrany bez směru (přátelství)
- **Orientovaný (digraf)** — hrany mají směr (jednosměrná ulice)
- **Vážený** — hrany mají hodnotu (vzdálenost, cena)
- **Acyklický** — neobsahuje cykly (stromy jsou speciální případ)

### Reprezentace v Pythonu

```python
# Seznam sousedů — nejběžnější
graf = {
    "A": ["B", "C"],
    "B": ["D"],
    "C": ["D"],
    "D": []
}
```

Matice sousednosti (pro husté grafy):
```
    A  B  C  D
A [ 0, 1, 1, 0 ]
B [ 1, 0, 0, 1 ]
C [ 1, 0, 0, 1 ]
D [ 0, 1, 1, 0 ]
```

---

## Prohledávání stavového prostoru

### BFS — Breadth-First Search (do šířky)

Prohledává vrstvu po vrstvě — nejdříve všechny sousedy, pak jejich sousedy. Využívá **frontu**. Najde **nejkratší cestu** v nevážených grafech.

```
Graf:          Pořadí BFS od A:
  A            A → B, C → D
 / \           vrstva 1: B, C
B   C          vrstva 2: D
 \ /
  D
```

```python
from collections import deque

def bfs(graf, start):
    navstivene = set([start])
    fronta = deque([start])
    while fronta:
        uzel = fronta.popleft()
        print(uzel, end=" ")
        for soused in graf[uzel]:
            if soused not in navstivene:
                navstivene.add(soused)
                fronta.append(soused)

bfs(graf, "A")   # A B C D
```

### DFS — Depth-First Search (do hloubky)

Jde co nejhlouběji jednou cestou, pak se vrátí a zkusí jinou. Využívá **zásobník** nebo rekurzi.

```
Graf:          Pořadí DFS od A:
  A            A → B → D → zpět → C → D (již navštíveno)
 / \           výstup: A B D C
B   C
 \ /
  D
```

```python
def dfs(graf, uzel, navstivene=None):
    if navstivene is None:
        navstivene = set()
    navstivene.add(uzel)
    print(uzel, end=" ")
    for soused in graf[uzel]:
        if soused not in navstivene:
            dfs(graf, soused, navstivene)

dfs(graf, "A")   # A B D C
```

### BFS vs DFS

| | BFS | DFS |
|---|---|---|
| Datová struktura | Fronta (queue) | Zásobník / rekurze |
| Nejkratší cesta | ✅ Ano (nevážený) | ❌ Ne |
| Paměť | Více (celá vrstva) | Méně |
| Použití | Nejkratší cesta, sítě | Cykly, topologické třídění |

---

## Řazení (Sorting)

Řazení = seřazení prvků seznamu. Algoritmy se liší složitostí, pamětí a stabilitou.

**Stabilní řazení** = zachovává pořadí prvků se stejnou hodnotou. Důležité při řazení podle více klíčů.

---

### 1. Bubble Sort (třídění bublinkami)

Opakovaně prochází seznam a vyměňuje sousedy ve špatném pořadí. Největší prvek "probublá" na konec.

```
[5, 3, 1, 4]
[3, 5, 1, 4]   ← vyměn 5 a 1
[3, 1, 5, 4]   ← vyměn 5 a 4
[3, 1, 4, 5]   ← 5 je na místě
... a tak dál
```

- Složitost: O(n²)
- Stabilní: Ano
- Použití: jen výuka, v praxi nepoužívat

---

### 2. Selection Sort (výběrové třídění)

Najde minimum v neseřazené části a přesune ho na začátek. Opakuje.

```
[5, 3, 1, 4]   ← najdi minimum (1), dej na pozici 0
[1, 3, 5, 4]   ← najdi minimum v [3,5,4] (3), dej na pozici 1
[1, 3, 5, 4]   ← najdi minimum v [5,4] (4), dej na pozici 2
[1, 3, 4, 5]   ← hotovo
```

- Složitost: O(n²) vždy
- Stabilní: Ne
- Použití: jen výuka

---

### 3. Insertion Sort (třídění vkládáním)

Buduje seřazenou část po jednom prvku — každý nový prvek vloží na správné místo.

```
[5, 3, 1, 4]
[3, 5, 1, 4]   ← 3 vloženo před 5
[1, 3, 5, 4]   ← 1 vloženo na začátek
[1, 3, 4, 5]   ← 4 vloženo mezi 3 a 5
```

- Složitost: O(n²) průměr, O(n) pro již seřazený vstup
- Stabilní: Ano
- Použití: malé pole, nebo téměř seřazená data

---

### 4. Merge Sort (třídění slučováním)

Rozděl a panuj. Rekurzivně rozdělí pole na půl, seřadí každou půlku, pak sloučí.

```
[38, 27, 43, 3]
  [38, 27]  [43, 3]
  [38][27]  [43][3]
  [27, 38]  [3, 43]    ← merge
  [3, 27, 38, 43]      ← merge
```

- Složitost: O(n log n) vždy
- Stabilní: Ano
- Nevýhoda: potřebuje extra paměť O(n)

---

### 5. Quick Sort (rychlé třídění)

Vybere **pivot**, rozdělí prvky na menší a větší, rekurzivně seřadí obě části.

```
[3, 6, 8, 10, 1, 2]   pivot = 6
  menší: [3, 1, 2]
  rovné: [6]
  větší: [8, 10]
→ quick_sort([3,1,2]) + [6] + quick_sort([8,10])
→ [1, 2, 3, 6, 8, 10]
```

- Složitost: O(n log n) průměr, O(n²) nejhorší (špatný pivot)
- Stabilní: Ne
- Použití: nejrychlejší v praxi pro velká data

---

### 6. Timsort (Python sort())

Hybridní algoritmus = Merge Sort + Insertion Sort. Python ho používá interně pro `sort()` a `sorted()`.

```python
seznam = [3, 1, 4, 1, 5]
seznam.sort()              # in-place, Timsort O(n log n)
serazeny = sorted(seznam)  # nový seznam
```

- Složitost: O(n log n), O(n) pro již seřazená data
- Stabilní: Ano
- Použití: vždy, když řadíš v Pythonu

---

## Srovnávací tabulka řazení

| Algoritmus | Nejlepší | Průměr | Nejhorší | Stabilní | Paměť |
|---|---|---|---|---|---|
| Bubble Sort | O(n) | O(n²) | O(n²) | ✅ Ano | O(1) |
| Selection Sort | O(n²) | O(n²) | O(n²) | ❌ Ne | O(1) |
| Insertion Sort | O(n) | O(n²) | O(n²) | ✅ Ano | O(1) |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | ✅ Ano | O(n) |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | ❌ Ne | O(log n) |
| Timsort | O(n) | O(n log n) | O(n log n) | ✅ Ano | O(n) |

---

## Shrnutí

- **Graf** = uzly + hrany; neorientovaný / orientovaný / vážený
- **BFS** = do šířky, fronta, najde nejkratší cestu
- **DFS** = do hloubky, rekurze/zásobník, cykly a průzkum
- **Bubble/Selection/Insertion Sort** = jednoduché, O(n²), jen pro malá data nebo výuku
- **Merge Sort** = vždy O(n log n), stabilní, potřebuje extra paměť
- **Quick Sort** = nejrychlejší v průměru, může být O(n²)
- **Timsort** = co Python používá, nejlepší volba v praxi

---

## Typické otázky u maturity

### Co je stavový prostor?
Všechny možné stavy problému. BFS/DFS ho prohledávají systematicky — BFS najde nejkratší cestu, DFS jde co nejhlouběji.

### Proč Quick Sort může být O(n²)?
Pokud jako pivot vždy vybereme největší nebo nejmenší prvek (například seřazené pole). Pak se nerozdělí na poloviny ale na [jeden prvek] + [zbytek].

### Co je stabilní řazení?
Zachová pořadí prvků se stejnou hodnotou. Např. řadíš lidi podle věku — dva lidi stejného věku zůstanou ve stejném pořadí jako byly.

### Kdy použít Merge Sort místo Quick Sort?
Merge Sort je vždy O(n log n) a stabilní. Quick Sort je rychlejší v průměru ale nestabilní a hrozí O(n²). Pro důležitá data (kde záleží na stabilitě) použij Merge Sort.
