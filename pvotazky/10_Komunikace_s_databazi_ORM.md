# 10. Komunikace s databázovým systémem — připojení, ukládání, načítání, ORM

---

## Jak program komunikuje s databází

Program se připojí k databázi přes **databázový driver** (ovladač) — knihovnu specifickou pro danou DB. Posílá SQL dotazy a dostává výsledky jako Python objekty.

```
Python aplikace
      │
      │  (SQL dotazy)
      ▼
  DB Driver (sqlite3, psycopg2, mysql-connector)
      │
      ▼
  Databázový server (SQLite, PostgreSQL, MySQL)
      │
      ▼
  Data na disku
```

---

## SQLite — přímá komunikace bez ORM

SQLite je souborová databáze vhodná pro lokální vývoj. Modul `sqlite3` je součástí standardní knihovny.

```python
import sqlite3

# Připojení — vytvoří soubor db.sqlite (nebo :memory: pro RAM)
conn = sqlite3.connect("db.sqlite")
cursor = conn.cursor()

# Vytvoření tabulky
cursor.execute("""
    CREATE TABLE IF NOT EXISTS uzivatele (
        id    INTEGER PRIMARY KEY AUTOINCREMENT,
        jmeno TEXT    NOT NULL,
        email TEXT    UNIQUE NOT NULL,
        vek   INTEGER
    )
""")

# Vložení dat — parametrizovaný dotaz (ochrana před SQL injection)
cursor.execute(
    "INSERT INTO uzivatele (jmeno, email, vek) VALUES (?, ?, ?)",
    ("Jan Novák", "jan@test.cz", 25)
)

# Vložení více záznamů najednou
data = [
    ("Eva Nová", "eva@test.cz", 30),
    ("Petr Svoboda", "petr@test.cz", 28),
]
cursor.executemany(
    "INSERT INTO uzivatele (jmeno, email, vek) VALUES (?, ?, ?)",
    data
)

conn.commit()   # DŮLEŽITÉ — uloží změny (bez commit() se ztratí)

# Načítání dat
cursor.execute("SELECT * FROM uzivatele WHERE vek > ?", (20,))
radky = cursor.fetchall()   # [(1, 'Jan Novák', 'jan@test.cz', 25), ...]
for radek in radky:
    print(radek)

# Jeden záznam
cursor.execute("SELECT * FROM uzivatele WHERE email = ?", ("jan@test.cz",))
uzivatel = cursor.fetchone()

# Pojmenované sloupce
conn.row_factory = sqlite3.Row   # přístup přes jméno sloupce
cursor = conn.cursor()
cursor.execute("SELECT * FROM uzivatele")
for row in cursor.fetchall():
    print(row["jmeno"], row["email"])

conn.close()
```

### Context manager — bezpečnější práce s DB

```python
# with automaticky commit/rollback a zavření spojení
with sqlite3.connect("db.sqlite") as conn:
    conn.execute(
        "UPDATE uzivatele SET vek = ? WHERE email = ?",
        (26, "jan@test.cz")
    )
    # commit se provede automaticky při opuštění with bloku
    # při výjimce se provede rollback
```

---

## ORM — Object-Relational Mapping

ORM je vrstva, která **mapuje databázové tabulky na Python třídy**. Nemusíš psát SQL — pracuješ s objekty.

```
Tabulka uzivatele → třída Uzivatel
Řádek tabulky     → instance třídy
Sloupec           → atribut instance
```

**Výhody ORM:**
- Píšeš Python, ne SQL
- Ochrana před SQL injection automaticky
- Přenositelné mezi různými databázemi
- Automatické migrace schématu

**Nevýhody ORM:**
- Menší kontrola nad SQL (složité dotazy)
- Může generovat neefektivní SQL
- Další závislost a učení

---

## SQLAlchemy — nejpopulárnější Python ORM

```python
from sqlalchemy import create_engine, Column, Integer, String, ForeignKey
from sqlalchemy.orm import DeclarativeBase, Session, relationship

# Připojení k databázi
engine = create_engine("sqlite:///db.sqlite", echo=False)

# Základní třída pro všechny modely
class Base(DeclarativeBase):
    pass

# Model = mapování třídy na tabulku
class Uzivatel(Base):
    __tablename__ = "uzivatele"

    id    = Column(Integer, primary_key=True, autoincrement=True)
    jmeno = Column(String(100), nullable=False)
    email = Column(String(200), unique=True, nullable=False)
    vek   = Column(Integer)

    # Vztah 1:N — jeden uživatel má více objednávek
    objednavky = relationship("Objednavka", back_populates="uzivatel")

    def __repr__(self):
        return f"Uzivatel(id={self.id}, jmeno={self.jmeno})"

class Objednavka(Base):
    __tablename__ = "objednavky"

    id          = Column(Integer, primary_key=True)
    popis       = Column(String(200))
    uzivatel_id = Column(Integer, ForeignKey("uzivatele.id"))

    uzivatel = relationship("Uzivatel", back_populates="objednavky")

# Vytvoření tabulek
Base.metadata.create_all(engine)

# CRUD operace
with Session(engine) as session:
    # CREATE — vložení
    jan = Uzivatel(jmeno="Jan Novák", email="jan@test.cz", vek=25)
    session.add(jan)
    session.commit()

    # READ — načítání
    vsichni = session.query(Uzivatel).all()
    jan = session.query(Uzivatel).filter_by(email="jan@test.cz").first()
    mladsi = session.query(Uzivatel).filter(Uzivatel.vek < 30).all()

    # UPDATE — aktualizace
    jan.vek = 26
    session.commit()

    # DELETE — smazání
    session.delete(jan)
    session.commit()
```

---

## Transakce

Transakce zajišťuje, že skupina operací proběhne buď **celá, nebo vůbec**. Klíčové pro konzistenci dat (bankovní převod — odečti i přičti, nebo ani jedno).

```python
import sqlite3

def prevod(conn, z_uctu: int, na_ucet: int, castka: float):
    try:
        conn.execute("BEGIN")   # začátek transakce
        conn.execute(
            "UPDATE ucty SET zustatek = zustatek - ? WHERE id = ?",
            (castka, z_uctu)
        )
        conn.execute(
            "UPDATE ucty SET zustatek = zustatek + ? WHERE id = ?",
            (castka, na_ucet)
        )
        conn.execute("COMMIT")  # uloží obě změny najednou
        return True
    except Exception as e:
        conn.execute("ROLLBACK")   # zruší všechny změny
        print(f"Převod selhal: {e}")
        return False
```

---

## Vzor Repository

Repository vzor odděluje logiku přístupu k datům od business logiky.

```python
from typing import Optional, List

class UzivatelRepository:
    def __init__(self, conn: sqlite3.Connection):
        self.conn = conn

    def pridat(self, jmeno: str, email: str, vek: int) -> int:
        cursor = self.conn.execute(
            "INSERT INTO uzivatele (jmeno, email, vek) VALUES (?, ?, ?)",
            (jmeno, email, vek)
        )
        self.conn.commit()
        return cursor.lastrowid

    def najdi_podle_id(self, id: int) -> Optional[dict]:
        row = self.conn.execute(
            "SELECT * FROM uzivatele WHERE id = ?", (id,)
        ).fetchone()
        return dict(row) if row else None

    def vsichni(self) -> List[dict]:
        rows = self.conn.execute("SELECT * FROM uzivatele").fetchall()
        return [dict(r) for r in rows]

    def smazat(self, id: int) -> bool:
        self.conn.execute("DELETE FROM uzivatele WHERE id = ?", (id,))
        self.conn.commit()
        return True
```

---

## Shrnutí

- Program komunikuje s DB přes **driver** — posílá SQL, dostává výsledky
- Vždy používej **parametrizované dotazy** (?, :jmeno) — ochrana před SQL injection
- **ORM** mapuje tabulky na třídy; píšeš Python místo SQL
- **Transakce** = skupina operací proběhne celá nebo vůbec; `COMMIT` / `ROLLBACK`
- **Repository vzor** odděluje přístup k datům od zbytku aplikace

---

## Typické doplňující otázky

### Co je ORM a proč ho používat?
Object-Relational Mapping mapuje databázové tabulky na Python třídy. Píšeš Python místo SQL, máš automatickou ochranu před SQL injection, a kód je přenositelný mezi různými databázemi.

### Co je CRUD?
Create, Read, Update, Delete — čtyři základní operace s daty. V SQL: INSERT, SELECT, UPDATE, DELETE.

### Co je N+1 problém u ORM?
Při načtení n záznamů ORM může generovat n+1 dotazů (1 pro seznam + 1 pro každou relaci). Řeší se eager loading (`joinedload`, `selectinload`).

### Jaký je rozdíl mezi `fetchone()` a `fetchall()`?
`fetchone()` vrátí jeden řádek (nebo None). `fetchall()` vrátí seznam všech řádků. Pro velké datasety je lepší iterovat přes cursor než načíst vše do paměti.
