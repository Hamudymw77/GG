# 23. Vlastnosti datových struktur

---

## Přehled vlastností

Každá datová struktura má tyto vlastnosti:
- **Seřazenost** — zachovává pořadí? je seřazená?
- **Duplicity** — povoluje stejné hodnoty víckrát?
- **Indexace** — lze přistupovat podle pozice/klíče?
- **Mutabilita** — lze měnit po vytvoření?
- **Hashování** — lze použít jako klíč ve slovníku?

---

## List `[]`

```python
seznam = [30, 10, 20, 10]

# Seřazenost — zachovává pořadí vložení
print(seznam)         # [30, 10, 20, 10]

# Duplicity — povoluje
print(len(seznam))    # 4 (10 tam je dvakrát)

# Indexace — podle pozice, O(1)
print(seznam[0])      # 30
print(seznam[-1])     # 10

# Mutabilita — lze měnit
seznam[0] = 99
seznam.append(50)

# Hashování — NELZE (je mutabilní)
# hash(seznam)   → TypeError
```

- ✅ zachovává pořadí vložení
- ✅ povoluje duplicity
- ✅ indexace podle pozice
- ✅ mutabilní
- ❌ nelze hashovat (nelze použít jako klíč v dict)

---

## Set `{}`

```python
mnozina = {30, 10, 20, 10}   # 10 se přidá jen jednou

# Seřazenost — NEZACHOVÁ pořadí
print(mnozina)         # {10, 20, 30} — pořadí není zaručeno

# Duplicity — NEPOVOLUJE
print(len(mnozina))    # 3 (ne 4)

# Indexace — NELZE
# mnozina[0]   → TypeError

# Mutabilita — lze přidávat/odebírat
mnozina.add(40)
mnozina.remove(10)

# Hashování — prvky musí být hashable (proto nelze dát list do setu)
# {[1,2]}   → TypeError
```

- ❌ nezachová pořadí
- ❌ nepovoluje duplicity (unikátní hodnoty)
- ❌ nelze indexovat
- ✅ mutabilní
- ❌ nelze hashovat, ale prvky musí být hashable

**Použití:** rychlá kontrola jestli prvek existuje — O(1), odstraňování duplicit.

---

## Dict `{klíč: hodnota}`

```python
slovnik = {"jmeno": "Jan", "vek": 25, "jmeno": "Petr"}   # duplicitní klíč → přepis

# Seřazenost — zachovává pořadí vložení (Python 3.7+)
print(list(slovnik.keys()))    # ['jmeno', 'vek']

# Duplicity — klíče UNIKÁTNÍ, hodnoty mohou být stejné
print(slovnik)                 # {'jmeno': 'Petr', 'vek': 25}

# Indexace — podle KLÍČE, O(1)
print(slovnik["jmeno"])        # 'Petr'
print(slovnik.get("email", "neznámý"))   # 'neznámý' — bezpečné

# Mutabilita — lze měnit
slovnik["email"] = "jan@ok.cz"

# Hashování — klíče musí být hashable (str, int, tuple — OK; list — ne)
```

- ✅ zachovává pořadí vložení
- ❌ klíče musí být unikátní (hodnoty mohou být stejné)
- ✅ indexace podle klíče — O(1)
- ✅ mutabilní
- ❌ klíče musí být hashable

**Použití:** mapování klíč → hodnota, konfigurace, počítání výskytů.

---

## Tuple `()`

```python
ntice = (30, 10, 20, 10)

# Seřazenost — zachovává pořadí vložení
print(ntice)          # (30, 10, 20, 10)

# Duplicity — povoluje
print(len(ntice))     # 4

# Indexace — podle pozice, O(1)
print(ntice[0])       # 30
print(ntice[1:3])     # (10, 20)

# Mutabilita — NELZE měnit
# ntice[0] = 99   → TypeError

# Hashování — LZE (je immutabilní)
print(hash(ntice))    # funguje
d = {ntice: "bod"}    # tuple jako klíč v dict
```

- ✅ zachovává pořadí vložení
- ✅ povoluje duplicity
- ✅ indexace podle pozice
- ❌ immutabilní — nelze měnit
- ✅ hashable — lze použít jako klíč v dict

**Použití:** souřadnice `(x, y)`, návrat více hodnot z funkce, klíče ve slovníku.

---

## Srovnávací tabulka

| Vlastnost | list | set | dict | tuple |
|---|---|---|---|---|
| Seřazenost | pořadí vložení | ❌ žádné | pořadí vložení | pořadí vložení |
| Duplicity | ✅ ano | ❌ ne | klíče ne | ✅ ano |
| Indexace | pozice O(1) | ❌ ne | klíč O(1) | pozice O(1) |
| Mutabilita | ✅ ano | ✅ ano | ✅ ano | ❌ ne |
| Hashable | ❌ ne | ❌ ne | ❌ ne | ✅ ano |

---

## Shrnutí

- **List** — obecný seznam, zachovává pořadí, povoluje duplicity, lze měnit
- **Set** — unikátní hodnoty, rychlé hledání O(1), bez pořadí
- **Dict** — klíč → hodnota, klíče unikátní, přístup O(1)
- **Tuple** — jako list ale neměnný, hashable → lze použít jako klíč

---

## Typické otázky u maturity

### Proč nelze list dát do setu?
List je mutabilní → nelze ho hashovat. Set vyžaduje hashable prvky.

### Jaký je rozdíl mezi set a frozenset?
`set` je mutabilní (lze přidávat). `frozenset` je immutabilní → lze hashovat a použít jako klíč v dict.

### Jak odstraním duplicity ze seznamu?
```python
seznam = [1, 2, 2, 3, 3]
bez_duplikatu = list(set(seznam))         # ztratí pořadí
bez_duplikatu = list(dict.fromkeys(seznam))  # zachová pořadí
```
