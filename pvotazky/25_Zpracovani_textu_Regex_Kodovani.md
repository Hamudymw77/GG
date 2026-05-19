# 25. Zpracování textu, Regex, kódování

---

## Kódování textu

Počítač ukládá jen bity a byty — kódování definuje, jak mapovat **znaky na byty**.

### ASCII

Nejstarší standard (7 bitů, 128 znaků) — anglická abeceda, číslice, speciální znaky. Nedostatečný pro jiné jazyky.

### Unicode

Standard definující číslo (code point) pro každý znak na světě. Unicode ≠ kódování — je to tabulka. Aktuálně ~150 000 znaků.

- `"A"` → U+0041
- `"č"` → U+010D
- `"🐍"` → U+1F40D

### UTF-8

Nejrozšířenější kódování Unicodu. Variabilní délka (1-4 byty):
- ASCII znaky → 1 byte (zpětně kompatibilní s ASCII)
- Latinské znaky s diakritikou → 2 byty
- Čínské/japonské znaky → 3 byty
- Emoji, historické znaky → 4 byty

```python
# str vs bytes v Pythonu
text = "Ahoj světe"   # Python str = Unicode string (code points)
print(type(text))     # <class 'str'>

# Kódování str → bytes (encode)
bajty = text.encode("utf-8")
print(bajty)          # b'Ahoj sv\xc4\x9bte'  (ě = 2 byty)
print(type(bajty))    # <class 'bytes'>
print(len(text))      # 10 znaků
print(len(bajty))     # 12 bytů (ě zabírá 2)

# Dekódování bytes → str (decode)
zpet = bajty.decode("utf-8")
print(zpet)           # Ahoj světe

# Přečtení souboru — vždy specifikuj encoding!
with open("soubor.txt", "r", encoding="utf-8") as f:
    obsah = f.read()
```

### Časté problémy s kódováním

```python
# UnicodeDecodeError — špatné kódování při čtení
try:
    with open("win_soubor.txt", "r", encoding="utf-8") as f:
        obsah = f.read()
except UnicodeDecodeError:
    # Zkus Windows-1250 (staré české soubory)
    with open("win_soubor.txt", "r", encoding="cp1250") as f:
        obsah = f.read()

# Bezpečné dekódování s errors parametrem
bajty = b"Ahoj \xff sv\xc4\x9bte"   # neplatný byte
print(bajty.decode("utf-8", errors="replace"))   # nahradí ? za neplatné
print(bajty.decode("utf-8", errors="ignore"))    # vynechá neplatné
```

---

## String metody

```python
text = "  Ahoj, Světe! Jak se Máš?  "

# Základní manipulace
print(text.strip())          # Odstraní mezery z obou stran
print(text.lstrip())         # Zleva
print(text.rstrip())         # Zprava

print(text.lower())          # ahoj, světe! jak se máš?
print(text.upper())          # AHOJ, SVĚTE! JAK SE MÁŠ?

# Hledání a testování
print("Ahoj" in text)           # True
print(text.find("Světe"))       # index začátku, -1 pokud nenalezeno
print(text.count("a"))          # počet výskytů

# Nahrazení
print(text.replace("Ahoj", "Čau"))      # Čau, Světe! Jak se Máš?

# Rozdělení a spojení
slova = text.strip().split(" ")         # Rozdělí dle mezery
print(slova)    # ["Ahoj,", "Světe!", "Jak", "se", "Máš?"]

vety = "řádek1\nřádek2\nřádek3"
radky = vety.splitlines()   # Rozdělí dle newline

# Spojení listu do stringu
print(", ".join(["A", "B", "C"]))   # A, B, C

# Testování obsahu
print("123".isdigit())   # True
print("abc".isalpha())   # True
print("   ".isspace())   # True

# Formátování
jmeno = "Jan"
vek = 25
print(f"Jmenuji se {jmeno} a je mi {vek} let")   # f-string (nejmodernější)
print("{} má {} let".format(jmeno, vek))           # .format()
print("%-10s | %5d" % (jmeno, vek))               # % operátor (starší)
```

---

## Regex — regulární výrazy

Regex (Regular Expression) je vzorec pro vyhledávání a zpracování textu. Modul `re`.

### Základní syntaxe

| Vzor | Popis | Příklad |
|---|---|---|
| `.` | Libovolný znak (kromě newline) | `a.c` → "abc", "aXc" |
| `*` | 0 nebo více opakování | `ab*` → "a", "ab", "abb" |
| `+` | 1 nebo více opakování | `ab+` → "ab", "abb" (ne "a") |
| `?` | 0 nebo 1 opakování | `ab?` → "a", "ab" |
| `{n}` | Přesně n opakování | `a{3}` → "aaa" |
| `{n,m}` | n až m opakování | `a{2,4}` → "aa", "aaa", "aaaa" |
| `[abc]` | Jeden z uvedených | `[aeiou]` → samohláska |
| `[^abc]` | Žádný z uvedených | `[^0-9]` → nečíslice |
| `[a-z]` | Rozsah znaků | `[a-zA-Z]` → písmeno |
| `^` | Začátek řetězce | `^Ahoj` |
| `$` | Konec řetězce | `konec$` |
| `\d` | Číslice `[0-9]` | `\d{3}` → "123" |
| `\D` | Nečíslice | |
| `\w` | Slovo `[a-zA-Z0-9_]` | |
| `\W` | Neslovo | |
| `\s` | Bílý znak (mezera, tab, newline) | |
| `\S` | Nebílý znak | |
| `\b` | Hranice slova | `\bcat\b` → "cat" ne "catch" |
| `\|` | Nebo | `cat\|dog` |
| `(...)` | Skupina (capture group) | `(\d+)` zachytí číslo |

---

## Modul re — funkce

```python
import re

text = "Jan Novak (jan.novak@email.cz) narozen 1990-05-15"

# re.search() — najde PRVNÍ výskyt kdekoli v textu
shoda = re.search(r"\d{4}-\d{2}-\d{2}", text)   # datum
if shoda:
    print(shoda.group())    # 1990-05-15
    print(shoda.start())    # index začátku
    print(shoda.end())      # index konce

# re.match() — hledá jen na ZAČÁTKU textu
m = re.match(r"Jan", text)   # Jan je na začátku → najde
m2 = re.match(r"Novak", text)  # Novak není na začátku → None

# re.findall() — vrátí VŠECHNY výskyty jako list
cisla = re.findall(r"\d+", text)
print(cisla)   # ["1990", "05", "15"]

# re.finditer() — vrátí iterator Match objektů
for m in re.finditer(r"\d+", text):
    print(f"Číslo: {m.group()} na pozici {m.start()}")

# re.sub() — nahrazení
ocenzurovano = re.sub(r"\d{4}-\d{2}-\d{2}", "DATUM", text)
print(ocenzurovano)   # Jan Novak (jan.novak@email.cz) narozen DATUM

# re.split() — rozdělení dle vzoru
casti = re.split(r"[\s,;]+", "ahoj,  světe; jak se  máš")
print(casti)   # ["ahoj", "světe", "jak", "se", "máš"]
```

### Skupiny (Groups)

```python
# Capture groups — závorky zachycují část vzoru
email_vzor = r"([\w\.-]+)@([\w\.-]+)\.(\w+)"
text = "Kontakt: jan@email.cz nebo petr@work.com"

for shoda in re.finditer(email_vzor, text):
    print(f"Celý: {shoda.group(0)}")   # celá shoda
    print(f"Uživatel: {shoda.group(1)}")  # první skupina
    print(f"Doména: {shoda.group(2)}.{shoda.group(3)}")
```

### Kompilace vzoru

```python
# Zkompiluj vzor jednou, použij mnohokrát
vzor = re.compile(r"\b\d{3}-\d{3}-\d{4}\b", re.IGNORECASE)

telefony = ["123-456-7890", "ahoj", "987-654-3210"]
for t in telefony:
    if vzor.search(t):
        print(f"Telefonní číslo: {t}")
```

---

## Praktické regex příklady

```python
import re

# Validace emailu
def validuj_email(email: str) -> bool:
    vzor = r"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"
    return bool(re.match(vzor, email))

print(validuj_email("jan@email.cz"))      # True
print(validuj_email("nespravny@"))         # False

# Extrakce čísel z textu
def extrahuj_cisla(text: str) -> list:
    return [float(x) for x in re.findall(r"\d+\.?\d*", text)]

print(extrahuj_cisla("Cena: 199.90 Kc, pocet: 3"))   # [199.9, 3.0]

# Sanitizace HTML (odstranění tagů)
def odstan_html(html: str) -> str:
    return re.sub(r"<[^>]+>", "", html)

print(odstan_html("<p>Ahoj <b>světe</b>!</p>"))   # Ahoj světe!

# Datum z různých formátů
def normalizuj_datum(text: str) -> str:
    vzor = r"(\d{1,2})[./-](\d{1,2})[./-](\d{2,4})"
    shoda = re.search(vzor, text)
    if shoda:
        den, mesic, rok = shoda.groups()
        return f"{rok}-{mesic.zfill(2)}-{den.zfill(2)}"
    return text

print(normalizuj_datum("15.5.2024"))     # 2024-05-15
print(normalizuj_datum("5/15/2024"))     # 2024-15-05
```

---

## Shrnutí

- **Kódování**: `str` = Unicode (code points); `bytes` = byty; `encode()` → bytes; `decode()` → str
- **UTF-8** = variabilní délka; ASCII kompatibilní; nejrozšířenější
- **String metody**: `strip/split/join/replace/find/format` — základ zpracování textu
- **Regex**: vzorce pro hledání vzorů; `.`, `*`, `+`, `?`, `[]`, `\d`, `\w`, `^$`
- **re modulu**: `search()` (kdekoli), `match()` (začátek), `findall()` (vše), `sub()` (nahrazení)
- **Skupiny**: závorky `()` zachycují části shody → `group(1)`, `group(2)`...

---

## Typické doplňující otázky

### Jaký je rozdíl mezi re.match() a re.search()?
`match()` hledá pouze na začátku řetězce. `search()` prohledává celý řetězec a vrátí první shodu. Pro hledání kdekoliv v textu použij `search()`.

### Co je greedy vs non-greedy matching?
`.*` je greedy — zachytí co nejvíce znaků. `.*?` je non-greedy (lazy) — zachytí co nejméně. Příklad: `<.+>` na `<a>text</a>` zachytí celé; `<.+?>` zachytí jen `<a>`.

### Proč `r"..."` (raw string) pro regex?
Raw string neinterpretuje escape sekvence (`\n`, `\t`...). Bez r: `"\\d"` musíš psát dvojité lomítko. S r: `r"\d"` — přehlednější pro regex.

### Jak funguje UTF-8 kódování diakritiky?
Znaky nad ASCII (128+) jsou kódovány 2-4 byty. Například `č` (U+010D) → `0xC4 0x8D` (2 byty). Proto `len("č") = 1` (znak), ale `len("č".encode("utf-8")) = 2` (byty).
