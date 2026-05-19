# 16. Soubory a serializace dat

---

## Co je serializace

Serializace je proces převodu datové struktury (objekt, slovník, seznam...) do formátu vhodného pro **uložení nebo přenos** (soubor, síť, databáze). Deserializace je opačný proces — obnova struktury ze uloženého formátu.

```
Python objekt  ──serialize──►  Uložený formát (JSON, pickle...)
Python objekt  ◄─deserialize─  Uložený formát
```

**Proč je důležitá:**
- Persistence dat mezi běhy programu
- Přenos dat po síti (API, webové služby)
- Konfigurace aplikací (JSON/YAML konfigurační soubory)
- Meziprogramová komunikace

---

## Práce se soubory — základy

### Otevření a čtení souboru

```python
# Doporučený způsob — with zaručí automatické zavření souboru
with open("soubor.txt", "r", encoding="utf-8") as f:
    obsah = f.read()       # celý soubor jako string
    print(obsah)

# Čtení po řádcích
with open("soubor.txt", "r", encoding="utf-8") as f:
    for radek in f:              # efektivní — nečte vše do paměti
        print(radek.strip())     # strip() odstraní \n na konci

# Čtení všech řádků jako list
with open("soubor.txt", "r", encoding="utf-8") as f:
    radky = f.readlines()   # ["radek1\n", "radek2\n", ...]
```

### Zápis do souboru

```python
# Zápis (přepíše existující soubor)
with open("vystup.txt", "w", encoding="utf-8") as f:
    f.write("Ahoj světe\n")
    f.write("Druhý řádek\n")

# Přidání na konec souboru
with open("log.txt", "a", encoding="utf-8") as f:
    f.write("Nový záznam\n")

# Zápis více řádků najednou
radky = ["A\n", "B\n", "C\n"]
with open("vystup.txt", "w", encoding="utf-8") as f:
    f.writelines(radky)
```

### Binární soubory

```python
# Čtení binárního souboru (obrázek, PDF...)
with open("obrazek.png", "rb") as f:   # "rb" = read binary
    data = f.read()   # typ bytes, ne str
    print(type(data))  # <class 'bytes'>

# Zápis binárních dat
with open("kopie.png", "wb") as f:   # "wb" = write binary
    f.write(data)
```

### Otevírací módy

| Mód | Popis |
|---|---|
| `"r"` | Čtení (výchozí), soubor musí existovat |
| `"w"` | Zápis, přepíše nebo vytvoří |
| `"a"` | Přidání na konec (append) |
| `"x"` | Vytvoření nového, chyba pokud existuje |
| `"b"` | Binární mód (kombinuje: `"rb"`, `"wb"`) |
| `"+"` | Čtení i zápis (`"r+"`, `"w+"`) |

---

## JSON serializace

JSON (JavaScript Object Notation) je textový formát pro výměnu dat. Lidsky čitelný, podporován všemi jazyky.

```python
import json

# Python objekty kompatibilní s JSON
data = {
    "jmeno": "Jan Novák",
    "vek": 25,
    "aktivni": True,
    "skore": 95.5,
    "tagy": ["python", "web"],
    "adresa": None   # None → null v JSON
}

# Serializace — Python dict → JSON string
json_text = json.dumps(data, ensure_ascii=False, indent=2)
print(json_text)
# {
#   "jmeno": "Jan Novák",
#   "vek": 25,
#   ...
# }

# Uložení do souboru
with open("data.json", "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False, indent=2)

# Deserializace — JSON string → Python dict
nactena_data = json.loads(json_text)
print(nactena_data["jmeno"])   # Jan Novák

# Načtení ze souboru
with open("data.json", "r", encoding="utf-8") as f:
    nactena_data = json.load(f)
```

### Mapování typů JSON ↔ Python

| JSON | Python |
|---|---|
| object `{}` | dict |
| array `[]` | list |
| string | str |
| number (int) | int |
| number (float) | float |
| true/false | True/False |
| null | None |

---

## CSV serializace

CSV (Comma-Separated Values) je tabulkový formát — každý řádek je záznam, hodnoty odděleny čárkou (nebo středníkem).

```python
import csv

# Zápis CSV
studenti = [
    {"jmeno": "Anna", "znamka": 1, "predmet": "Python"},
    {"jmeno": "Petr", "znamka": 2, "predmet": "Java"},
    {"jmeno": "Jana", "znamka": 1, "predmet": "Python"},
]

with open("studenti.csv", "w", newline="", encoding="utf-8") as f:
    # DictWriter — píše slovníky, záhlaví z fieldnames
    writer = csv.DictWriter(f, fieldnames=["jmeno", "znamka", "predmet"])
    writer.writeheader()   # zapíše záhlaví
    writer.writerows(studenti)

# Čtení CSV
with open("studenti.csv", "r", encoding="utf-8") as f:
    reader = csv.DictReader(f)   # každý řádek jako dict
    for radek in reader:
        print(f"{radek['jmeno']}: {radek['znamka']}")
```

---

## Pickle serializace

Pickle je Pythonovský binární formát pro serializaci **libovolných Python objektů** (třídy, funkce, lambda...).

```python
import pickle

class Uzivatel:
    def __init__(self, jmeno: str, vek: int):
        self.jmeno = jmeno
        self.vek = vek

    def __repr__(self):
        return f"Uzivatel({self.jmeno}, {self.vek})"

uzivatel = Uzivatel("Jana", 30)

# Uložení do souboru (binární)
with open("uzivatel.pkl", "wb") as f:
    pickle.dump(uzivatel, f)

# Načtení — vrátí Python objekt
with open("uzivatel.pkl", "rb") as f:
    nacteny = pickle.load(f)
    print(nacteny)         # Uzivatel(Jana, 30)
    print(nacteny.jmeno)   # Jana
```

**POZOR:** Nikdy nenačítej pickle z nedůvěryhodného zdroje — pickle může spustit libovolný kód při deserializaci (bezpečnostní riziko).

---

## Pathlib — moderní práce s cestami

```python
from pathlib import Path

# Vytvoření cesty
p = Path("data") / "soubory" / "vstup.txt"   # data/soubory/vstup.txt

print(p.name)      # vstup.txt
print(p.stem)      # vstup
print(p.suffix)    # .txt
print(p.parent)    # data/soubory

# Práce se soubory
if p.exists():
    obsah = p.read_text(encoding="utf-8")   # zkratka pro open+read
    print(obsah)

p.write_text("Nový obsah", encoding="utf-8")   # zkratka pro open+write

# Procházení adresáře
for soubor in Path("data").glob("*.json"):   # všechny JSON soubory
    print(soubor)

for soubor in Path("data").rglob("*.py"):   # rekurzivně
    print(soubor)

# Vytvoření adresářů
Path("nova_slozka/podslozka").mkdir(parents=True, exist_ok=True)
```

---

## Shrnutí

- `open()` + `with` = bezpečná práce se soubory; vždy specifikovat encoding="utf-8"
- **JSON** = lidsky čitelný formát; `json.dumps/loads` (string), `json.dump/load` (soubor)
- **CSV** = tabulkový formát; `csv.DictReader/DictWriter` pro práci se slovníky
- **Pickle** = binární Python-specifický formát; ukládá libovolné objekty, ale bezpečnostní riziko z cizích zdrojů
- **pathlib** = moderní OOP práce s cestami místo `os.path`

---

## Typické doplňující otázky

### Proč preferovat JSON před pickle?
JSON je čitelný lidmi i jinými jazyky, přenositelný a bezpečný. Pickle funguje pouze v Pythonu, je binární a nebezpečný z cizích zdrojů.

### Co dělá `with open(...) as f`?
Context manager — zaručuje zavření souboru i při výjimce. Bez `with` by soubor mohl zůstat otevřený při pádu programu → únik zdroje (resource leak).

### Jaký je rozdíl mezi `json.dump` a `json.dumps`?
`json.dump(data, f)` — zapíše přímo do souboru. `json.dumps(data)` — vrátí string (s = string). Analogicky `json.load(f)` vs `json.loads(text)`.

### Proč `newline=""` při zápisu CSV?
Na Windows Python ve výchozím stavu převádí `\n` na `\r\n`, ale csv modul přidává vlastní konce řádků. Dvojí konverze by vedla ke `\r\r\n`. `newline=""` to vypne.
