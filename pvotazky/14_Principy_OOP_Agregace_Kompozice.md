# 14. Principy OOP, agregace a kompozice

---

## Co je OOP

OOP (objektově orientované programování) je způsob psaní kódu, kde vše modeluješ jako **objekty**. Každý objekt reprezentuje něco z reálného světa — auto, uživatele, bankovní účet.

Před OOP se programovalo procedurálně — jen funkce a proměnné, vše v jednom. OOP přineslo **lepší organizaci, znovupoužitelnost a čistší kód**.

- **Třída** = šablona/blueprint — popisuje jak objekt vypadá a co umí
- **Objekt** = konkrétní instance třídy — vzniká z šablony, existuje v paměti
- **Atribut** = data objektu (co má)
- **Metoda** = funkce objektu (co umí)

```python
class Pes:
    def __init__(self, jmeno, vek):
        self.jmeno = jmeno   # atribut
        self.vek = vek       # atribut

    def stekej(self):        # metoda
        print(f"{self.jmeno}: Haf!")

rex = Pes("Rex", 3)   # objekt — instance třídy Pes
rex.stekej()           # Rex: Haf!
```

---

## Čtyři pilíře OOP

### 1. Zapouzdření (Encapsulation)

**Co to je:** Třída si chrání svá vnitřní data. Zvenku k nim nejde přistoupit přímo — jedině přes metody, které mohou validovat hodnoty.

**Proč je to důležité:** Zabraňuješ nevalidním stavům. Nikdo nemůže nastavit záporný věk nebo záporný zůstatek bez tvého vědomí.

**Jak v Pythonu:**
- `_nazev` — konvence "neměň zvenku" (ale jde to)
- `__nazev` — name mangling → přejmenuje se na `_Trida__nazev`, těžší přístup
- `@property` — getter/setter vypadající jako atribut

```python
class BankovniUcet:
    def __init__(self, zustatek):
        self.__zustatek = zustatek   # private

    @property
    def zustatek(self):
        return self.__zustatek       # getter — čtení

    def vloz(self, castka):
        if castka <= 0:
            raise ValueError("Musí být kladné")
        self.__zustatek += castka    # validace před změnou

    def vyber(self, castka):
        if castka > self.__zustatek:
            raise ValueError("Nedostatek prostředků")
        self.__zustatek -= castka

ucet = BankovniUcet(1000)
ucet.vloz(500)
print(ucet.zustatek)          # 1500
# ucet.__zustatek = -999      # NEFUNGUJE — to je celý smysl
```

---

### 2. Dědičnost (Inheritance)

**Co to je:** Třída (potomek) přebírá atributy a metody jiné třídy (rodiče). Vztah "IS-A" — Pes IS-A Zvíře.

**Proč je to důležité:** Sdílíš kód, nemusíš ho psát znovu. Potomek může rodičovské metody přepsat (`override`).

**Klíčové pojmy:**
- `super()` — zavolá metodu rodiče
- Python podporuje i vícenásobnou dědičnost (více rodičů najednou)

```python
class Zvire:
    def __init__(self, jmeno):
        self.jmeno = jmeno

    def dychej(self):
        print(f"{self.jmeno} dýchá")

    def mluv(self):
        print("...")

class Pes(Zvire):
    def __init__(self, jmeno, plemeno):
        super().__init__(jmeno)      # zavolá __init__ rodiče
        self.plemeno = plemeno

    def mluv(self):                  # override — přepsání rodičovské metody
        print(f"{self.jmeno}: Haf!")

rex = Pes("Rex", "Ovčák")
rex.dychej()   # zděděno od Zvire → "Rex dýchá"
rex.mluv()     # přepsáno v Pes   → "Rex: Haf!"
```

---

### 3. Polymorfismus (Polymorphism)

**Co to je:** Různé třídy reagují na stejnou zprávu (volání metody) každá po svém. Polymorfismus = "mnoho forem".

**Proč je to důležité:** Kód, který pracuje s rodičem, funguje i s potomky — nemusíš psát `if isinstance(...)` pro každý typ.

**Dva druhy v Pythonu:**
- **Duck typing** — "Když to chodí jako kachna a kváká jako kachna, je to kachna" — Python nekontroluje typ, jen jestli metoda existuje
- **Override** — potomek přepíše metodu rodiče

```python
class Pes:
    def mluv(self):
        return "Haf!"

class Kocka:
    def mluv(self):
        return "Mňau!"

class Papousek:
    def mluv(self):
        return "Ahoj!"

zvierata = [Pes(), Kocka(), Papousek()]
for z in zvierata:
    print(z.mluv())   # každý odpoví jinak, volání je identické
# Haf!
# Mňau!
# Ahoj!
```

---

### 4. Abstrakce (Abstraction)

**Co to je:** Definuješ **co** musí objekt umět, ale ne **jak** to dělá. Je to smlouva — říkáš "každý kdo dědí z této třídy MUSÍ mít tyto metody", ale implementaci nechváš na potomcích.

**Analogie:** USB konektor je abstrakce. Víš, že do USB portu můžeš zapojit myš, klávesnici nebo flash disk a bude to fungovat — ale nezajímá tě jak ty zařízení fungují uvnitř. Všechny splňují "USB rozhraní".

**Jak v Pythonu:** Pomocí `ABC` (Abstract Base Class) a `@abstractmethod`.
- Abstraktní třídu (`ABC`) **nelze přímo instancovat** — nelze udělat `Zvire()`
- Pokud potomek **neimplementuje** abstraktní metodu → Python vyhodí `TypeError` při pokusu o instanciaci
- Abstraktní třída říká: "Každý, kdo dědí ode mě, musí mít metodu `mluv()`"

```python
from abc import ABC, abstractmethod

# Abstraktní třída = šablona/smlouva
# Říká: "každé zvíře MUSÍ umět mluv() a pohyb()"
# Ale neříká jak — to záleží na konkrétním zvířeti
class Zvire(ABC):

    def __init__(self, jmeno):
        self.jmeno = jmeno          # normální atribut — tohle je ok

    @abstractmethod
    def mluv(self):                 # abstraktní metoda — MUSÍ být v potomkovi
        pass                        # pass = žádná implementace zde

    @abstractmethod
    def pohyb(self):
        pass

    def info(self):                 # normální metoda — tuto potomci DĚDÍ
        print(f"Jsem {self.jmeno}")

# Toto by vyhodilo chybu:
# z = Zvire("Neco")   # TypeError: Can't instantiate abstract class Zvire

# Potomci MUSÍ implementovat mluv() a pohyb()
class Pes(Zvire):
    def mluv(self):
        print(f"{self.jmeno}: Haf!")

    def pohyb(self):
        print(f"{self.jmeno} běží")

class Ryba(Zvire):
    def mluv(self):
        print(f"{self.jmeno}: ...")    # ryby nemluví, ale metoda existuje

    def pohyb(self):
        print(f"{self.jmeno} plave")

# Co kdybychom zapomněli implementovat pohyb()?
class SpatnyPtak(Zvire):
    def mluv(self):
        print("Vrk!")
    # pohyb() chybí!

# SpatnyPtak("Krkavec")   # TypeError! Musíš implementovat pohyb()

# Použití — pracujeme s abstrakcí Zvire, nezajímá nás konkrétní typ
zvirata = [Pes("Rex"), Ryba("Nemo")]
for z in zvirata:
    z.info()    # zděděno od Zvire — funguje pro všechny
    z.mluv()    # každý to dělá jinak
    z.pohyb()
```

**Výhoda:** Funkce `nakrm(zvire: Zvire)` funguje s libovolným zvířetem — nemusíš psát `if isinstance(zvire, Pes)`. Stačí vědět, že má metody `mluv()` a `pohyb()`.

---

## Vztahy mezi třídami

### IS-A (dědičnost)

Potomek IS-A rodič. `Pes` IS-A `Zvire`. Implementuješ dědičností.

### HAS-A — Kompozice vs Agregace

Oba vztahy říkají "objekt má jiný objekt". Liší se silou závislosti.

### Kompozice — silná závislost

Objekt **vlastní** jiný objekt a vytváří ho sám. Když zanikne rodič, zanikne i potomek. Potomek bez rodiče nedává smysl.

Příklad: `Auto` vlastní `Motor`. Motor bez auta nedává smysl, vytváří se uvnitř auta.

```python
class Motor:
    def __init__(self, vykon):
        self.vykon = vykon

    def spust(self):
        print(f"Motor {self.vykon} kW nastartoval")

class Auto:
    def __init__(self, znacka, vykon):
        self.znacka = znacka
        self.motor = Motor(vykon)   # Motor se vytváří uvnitř — kompozice

    def jed(self):
        self.motor.spust()
        print(f"{self.znacka} jede")

auto = Auto("Škoda", 85)
auto.jed()
# Motor 85 kW nastartoval
# Škoda jede
```

### Agregace — slabá závislost

Objekt **odkazuje** na jiný objekt, který existoval předtím a bude existovat i po. Lze sdílet mezi více objekty.

Příklad: `Skola` má `Studenty`. Student existuje i bez školy, může být na více školách.

```python
class Student:
    def __init__(self, jmeno):
        self.jmeno = jmeno

class Skola:
    def __init__(self, nazev):
        self.nazev = nazev
        self.studenti = []

    def zapis(self, student):
        self.studenti.append(student)   # přijímá objekt zvenku — agregace

    def vypis(self):
        for s in self.studenti:
            print(f"  {s.jmeno}")

jan = Student("Jan")
petra = Student("Petra")

skola = Skola("SPŠE")
skola.zapis(jan)
skola.zapis(petra)
skola.vypis()
# Jan a Petra existují nezávisle na škole
```

| | Kompozice | Agregace |
|---|---|---|
| Závislost | Silná | Slabá |
| Kde vzniká potomek | Uvnitř rodiče | Zvenku, předán |
| Zánik rodiče | potomek zaniká | potomek přežívá |
| Sdílení | Ne | Ano (jeden objekt více místech) |
| Příklad | Auto–Motor | Škola–Student |

---

## SOLID principy

Pět pravidel pro čistý OOP design. Pomáhají psát kód, který se snadno rozšiřuje a udržuje.

### S — Single Responsibility Principle

Každá třída má **jednu zodpovědnost**. Pokud třídu musíš měnit ze dvou různých důvodů, má dvě zodpovědnosti — rozděl ji.

```python
# Špatně — třída toho dělá moc
class Uzivatel:
    def uloz_do_db(self): ...
    def posli_email(self): ...
    def vygeneruj_pdf(self): ...

# Správně — každá třída má jednu zodpovědnost
class Uzivatel:
    def __init__(self, jmeno, email):
        self.jmeno = jmeno
        self.email = email

class UzivatelRepo:
    def uloz(self, uzivatel): ...

class EmailService:
    def posli_uvitaci(self, uzivatel): ...
```

### O — Open/Closed Principle

Třída je **otevřená pro rozšíření, uzavřená pro modifikaci**. Přidáváš nové funkce novými třídami, ne úpravou starých.

```python
# Špatně — při přidání nového tvaru musím měnit funkci
def vypocti_obsah(tvar, typ):
    if typ == "kruh": ...
    elif typ == "obdelnik": ...
    # při přidání trojúhelníku musím měnit tuto funkci

# Správně — nový tvar = nová třída, funkce se nemění
def vypocti_obsah(tvar):   # funguje s jakýmkoli Tvarem
    return tvar.obsah()
```

### L — Liskov Substitution Principle

**Potomek musí fungovat všude kde fungoval rodič** — nesmí rozbít chování.

```python
# Porušení LSP — Tučňák je Pták, ale nemůže létat → rozbije kód
class Ptak:
    def let(self):
        print("Letím")

class Tucnak(Ptak):
    def let(self):
        raise Exception("Tučňák nelétá!")   # rozbije kód čekající Ptáka
```

### I — Interface Segregation Principle

**Raději více malých rozhraní než jedno velké.** Třída nemá implementovat metody které nepotřebuje.

### D — Dependency Inversion Principle

**Závisej na abstrakcích, ne na konkrétních třídách.** Místo `MySQL` třídy závisej na `Databaze` abstrakci — snáze vyměnitelné.

---

## Shrnutí

- **OOP** organizuje kód do objektů s daty a chováním
- **4 pilíře:** zapouzdření (chráníš data), dědičnost (sdílíš kód), polymorfismus (stejné volání, různé chování), abstrakce (skryješ detaily)
- **Kompozice** = objekt vlastní jiný (silná závislost, Motor v Autě)
- **Agregace** = objekt odkazuje na jiný (slabá závislost, Student ve Škole)
- **SOLID** = 5 pravidel čistého designu; nejdůležitější S (jedna zodpovědnost) a O (rozšiřuj přidáváním)

---

## Typické doplňující otázky

### Jaký je rozdíl mezi třídou a objektem?
Třída je šablona (blueprint), objekt je konkrétní instance té šablony v paměti. Z jedné třídy lze vytvořit nekonečně objektů.

### Co je `super()` a kdy se používá?
`super()` zavolá metodu rodiče. Nejčastěji v `__init__` potomka, aby se inicializovaly i atributy rodiče.

### Kdy použít kompozici a kdy agregaci?
Kompozici když potomek bez rodiče nedává smysl (Motor bez Auta). Agregaci když objekt existuje nezávisle a může být sdílen (Student ve více Školách).

### Lze v Pythonu udělat skutečně private atribut?
Ne úplně. `__nazev` způsobí name mangling (`_Trida__nazev`), ale technicky přístupné je. Python spoléhá na dohodu programátorů, ne na vynucení.

### Co je duck typing?
Pythonická forma polymorfismu — Python nekontroluje typ objektu, jen jestli má požadovanou metodu. "Pokud to chodí jako kachna a kváká jako kachna, je to kachna."
