# 21. Typy datových struktur

---

## Co jsou datové struktury

Datová struktura je způsob **organizace a uložení dat** v paměti. Různé struktury jsou lepší pro různé operace — výběr správné struktury ovlivňuje rychlost programu.

---

## Pole / List — lineární struktura

Prvky jsou uloženy **za sebou** v paměti. Python `list` je dynamické pole.

```python
cisla = [10, 20, 30, 40]

print(cisla[2])    # 30 — O(1) přístup podle indexu
cisla.append(50)   # O(1) přidání na konec
cisla.insert(1, 99)  # O(n) vložení uprostřed (musí posunout ostatní)
```

- **Výhoda:** rychlý přístup podle indexu — O(1)
- **Nevýhoda:** vložení/odebrání uprostřed — O(n)

---

## Spojový seznam (Linked List)

Každý prvek (**uzel**) obsahuje data a **odkaz na další uzel**. Prvky nejsou za sebou v paměti.

```
[10 | →] → [20 | →] → [30 | None]
```

```python
class Uzel:
    def __init__(self, data):
        self.data = data
        self.next = None   # odkaz na další uzel

# Vytvoření: 10 → 20 → 30
a = Uzel(10)
b = Uzel(20)
c = Uzel(30)
a.next = b
b.next = c

# Průchod
aktualni = a
while aktualni:
    print(aktualni.data)   # 10, 20, 30
    aktualni = aktualni.next
```

- **Výhoda:** vložení/odebrání na začátku — O(1)
- **Nevýhoda:** přístup k prvku — O(n) (musíš projít od začátku)

---

## Zásobník (Stack) — LIFO

**LIFO** = Last In, First Out — poslední přidaný = první odebraný. Jako zásobník talířů.

```
přidáš: 1, 2, 3
odebereš: 3, 2, 1
```

```python
zasobnik = []

zasobnik.append(1)   # push
zasobnik.append(2)
zasobnik.append(3)

print(zasobnik.pop())   # 3 — LIFO
print(zasobnik.pop())   # 2
print(zasobnik.pop())   # 1
```

**Použití:** zásobník volání funkcí, undo/redo v editoru, závorky matching.

---

## Fronta (Queue) — FIFO

**FIFO** = First In, First Out — první přidaný = první odebraný. Jako fronta v obchodě.

```
přidáš: 1, 2, 3
odebereš: 1, 2, 3
```

```python
from collections import deque

fronta = deque()

fronta.append(1)      # enqueue — přidej nakonec
fronta.append(2)
fronta.append(3)

print(fronta.popleft())   # 1 — FIFO
print(fronta.popleft())   # 2
```

> Proč `deque` a ne list? `list.pop(0)` je O(n) — musí posunout všechny prvky. `deque.popleft()` je O(1).

**Použití:** tiskárna (print queue), zpracování úkolů, BFS (prohledávání grafu).

---

## Binární vyhledávací strom (BST)

Stromová struktura kde každý **uzel** má max. dva potomky.

Pravidlo BST:
- levý podstrom má hodnoty **menší** než uzel
- pravý podstrom má hodnoty **větší** než uzel

```
        50
       /  \
      30   70
     / \
    20  40
```

```python
class Uzel:
    def __init__(self, hodnota):
        self.hodnota = hodnota
        self.levy = None
        self.pravy = None

koren = Uzel(50)
koren.levy = Uzel(30)
koren.pravy = Uzel(70)
koren.levy.levy = Uzel(20)
koren.levy.pravy = Uzel(40)
```

- **Hledání/vložení:** O(log n) průměrně — každým krokem zahodíme půlku
- **Výhoda oproti listu:** nemusíš procházet vše, jdeš vždy doleva nebo doprava

---

## Halda (Heap)

Stromová struktura kde **rodič je vždy menší než potomci** (min-heap). Nejmenší prvek je vždy nahoře.

```python
import heapq

halda = []
heapq.heappush(halda, 5)
heapq.heappush(halda, 1)
heapq.heappush(halda, 3)

print(heapq.heappop(halda))   # 1 — minimum vždy první
print(heapq.heappop(halda))   # 3
```

**Použití:** priority queue (úkoly s prioritou), Dijkstrův algoritmus (nejkratší cesta).

---

## Přehled složitostí

| Struktura | Přístup | Hledání | Vložení | Odebrání |
|---|---|---|---|---|
| List/Pole | O(1) | O(n) | O(n) | O(n) |
| Spojový seznam | O(n) | O(n) | O(1) | O(1) |
| Zásobník (Stack) | — | — | O(1) | O(1) |
| Fronta (Queue) | — | — | O(1) | O(1) |
| BST (průměr) | — | O(log n) | O(log n) | O(log n) |
| Halda | — | O(n) | O(log n) | O(log n) |

---

## Shrnutí

- **List** — O(1) přístup, O(n) vložení uprostřed
- **Linked List** — O(1) vložení na začátek, O(n) přístup
- **Stack** (LIFO) — push/pop; zásobník talířů, undo
- **Queue** (FIFO) — enqueue/dequeue; fronta v obchodě, tiskárna
- **BST** — O(log n) hledání; vždy jdeš doleva nebo doprava
- **Heap** — nejmenší prvek vždy nahoře; priority queue

---

## Typické otázky u maturity

### Jaký je rozdíl mezi zásobníkem a frontou?
Stack = LIFO (poslední dovnitř = první ven). Queue = FIFO (první dovnitř = první ven).

### Proč nepoužívat list jako frontu?
`list.pop(0)` je O(n) — musí posunout všechny prvky. `deque.popleft()` je O(1).

### Kdy je BST pomalý?
Když vkládáš seřazená data (1, 2, 3, 4...) — strom se stane lineárním, výška = n, hledání O(n). Říká se mu degenerovaný strom.
