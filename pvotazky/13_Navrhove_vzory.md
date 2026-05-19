# 13. Návrhové vzory (creational, structural, behavioral)

---

## Co jsou návrhové vzory

Návrhový vzor (design pattern) je **opakovaně použitelné řešení běžného problému** v návrhu softwaru. Není to hotový kód — je to šablona/recept, který adaptujete na konkrétní situaci.

Pocházejí z knihy "Gang of Four" (GoF, 1994) — 23 klasických vzorů rozdělených do 3 kategorií.

**Proč vzory:**
- Ověřená řešení od zkušených vývojářů
- Společný slovník — vývojáři si porozumí rychleji ("použij zde Singleton")
- Lepší design — loosely coupled, snáze testovatelný kód

---

## 1. Creational (vytvářecí) vzory

Řeší **jak vytvářet objekty** — skrývají logiku vytváření, zvyšují flexibilitu.

### Singleton

Zajišťuje, že třída má **pouze jednu instanci** v celém programu. Typické pro konfiguraci, logger, databázové spojení.

```python
class Konfigurace:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._data = {}   # inicializace jen jednou
        return cls._instance   # vždy vrátí tu samou instanci

    def nastav(self, klic: str, hodnota):
        self._data[klic] = hodnota

    def ziskej(self, klic: str):
        return self._data.get(klic)

# Test — vždy stejná instance
k1 = Konfigurace()
k2 = Konfigurace()
k1.nastav("debug", True)
print(k2.ziskej("debug"))   # True — k1 a k2 jsou ten samý objekt
print(k1 is k2)              # True
```

### Factory Method

Rodičovská třída definuje rozhraní pro vytváření objektů, **potomci rozhodují co se vytvoří**. Odděluje vytváření od použití.

```python
from abc import ABC, abstractmethod

class Notifikace(ABC):
    @abstractmethod
    def posli(self, zprava: str) -> None:
        pass

class EmailNotifikace(Notifikace):
    def posli(self, zprava: str) -> None:
        print(f"Email: {zprava}")

class SMSNotifikace(Notifikace):
    def posli(self, zprava: str) -> None:
        print(f"SMS: {zprava}")

class PushNotifikace(Notifikace):
    def posli(self, zprava: str) -> None:
        print(f"Push: {zprava}")

# Factory — vytváří správný objekt podle parametru
def vytvor_notifikaci(typ: str) -> Notifikace:
    typy = {
        "email": EmailNotifikace,
        "sms": SMSNotifikace,
        "push": PushNotifikace,
    }
    if typ not in typy:
        raise ValueError(f"Neznámý typ: {typ}")
    return typy[typ]()

notif = vytvor_notifikaci("email")
notif.posli("Vaše objednávka byla odeslána")   # Email: Vaše objednávka...
```

### Builder

Odděluje **konstrukci složitého objektu** od jeho reprezentace. Staví objekt krok po kroku.

```python
class Pizza:
    def __init__(self):
        self.testo = None
        self.omacka = None
        self.suroviny = []

    def __str__(self):
        return f"Pizza: {self.testo}, {self.omacka}, {self.suroviny}"

class PizzaBuilder:
    def __init__(self):
        self._pizza = Pizza()

    def testo(self, typ: str):
        self._pizza.testo = typ
        return self   # vrátí self pro řetězení

    def omacka(self, typ: str):
        self._pizza.omacka = typ
        return self

    def suroviny(self, *suroviny: str):
        self._pizza.suroviny.extend(suroviny)
        return self

    def build(self) -> Pizza:
        return self._pizza

# Fluent interface — metodové řetězení
pizza = (PizzaBuilder()
    .testo("tenké")
    .omacka("rajčatová")
    .suroviny("sýr", "šunka", "olivy")
    .build())

print(pizza)   # Pizza: tenké, rajčatová, ['sýr', 'šunka', 'olivy']
```

---

## 2. Structural (strukturální) vzory

Řeší **jak skládat třídy a objekty** do větších celků.

### Decorator

Přidává nové chování objektu **bez změny jeho třídy**. Obaluje objekt dalšími vrstvami funkcí.

```python
# Funkcionální dekorátor v Pythonu (přesná implementace vzoru)
import time
import functools

def casomeric(funkce):
    @functools.wraps(funkce)
    def obal(*args, **kwargs):
        zacatek = time.time()
        vysledek = funkce(*args, **kwargs)
        konec = time.time()
        print(f"{funkce.__name__} trvalo {konec-zacatek:.4f}s")
        return vysledek
    return obal

def logovac(funkce):
    @functools.wraps(funkce)
    def obal(*args, **kwargs):
        print(f"Volám {funkce.__name__} s {args}")
        return funkce(*args, **kwargs)
    return obal

@casomeric
@logovac
def vypocitej(n: int) -> int:
    return sum(range(n))

vypocitej(1000000)
# Volám vypocitej s (1000000,)
# vypocitej trvalo 0.0234s
```

### Adapter

Umožní spolupráci dvou **nekompatibilních rozhraní**. "Redukce" — přizpůsobí starý kód novému rozhraní.

```python
# Staré rozhraní (nelze změnit)
class StaryPlatebniSystem:
    def proved_transakci(self, castka_halere: int) -> bool:
        print(f"Stará platba: {castka_halere} haléřů")
        return True

# Nové rozhraní které očekává aplikace
class NovyPlatebniSystem:
    def zaplat(self, castka_koruny: float) -> bool:
        raise NotImplementedError

# Adapter — překládá mezi starým a novým
class PlatebniAdapter(NovyPlatebniSystem):
    def __init__(self, stary: StaryPlatebniSystem):
        self._stary = stary

    def zaplat(self, castka_koruny: float) -> bool:
        castka_halere = int(castka_koruny * 100)   # převod
        return self._stary.proved_transakci(castka_halere)

stary = StaryPlatebniSystem()
adapter = PlatebniAdapter(stary)
adapter.zaplat(199.99)   # Stará platba: 19999 haléřů
```

### Facade (fasáda)

Poskytuje **jednoduché rozhraní** k složitému podsystému. Skrývá komplexitu.

```python
# Složité podsystémy
class Video:
    def dekoduj(self, soubor): print(f"Dekóduji video: {soubor}")
    def zakoduj(self, format): print(f"Kóduji do: {format}")

class Audio:
    def normalizuj(self): print("Normalizuji audio")
    def komprimuj(self): print("Kompresuji audio")

class MetadataEditor:
    def nastav_titulek(self, t): print(f"Titulek: {t}")

# Fasáda — jednoduché rozhraní
class VideoEditor:
    def __init__(self):
        self.video = Video()
        self.audio = Audio()
        self.meta = MetadataEditor()

    def zpracuj(self, soubor: str, titulek: str, format: str):
        self.video.dekoduj(soubor)
        self.audio.normalizuj()
        self.audio.komprimuj()
        self.meta.nastav_titulek(titulek)
        self.video.zakoduj(format)
        print("Hotovo!")

editor = VideoEditor()
editor.zpracuj("film.mp4", "Můj film", "h264")   # jednoduché volání
```

---

## 3. Behavioral (behaviorální) vzory

Řeší **komunikaci a odpovědnost** mezi objekty.

### Observer (pozorovatel)

Objekt (subject) informuje všechny registrované **pozorovatele** o změně svého stavu. Základ event-driven architektury.

```python
from abc import ABC, abstractmethod
from typing import List

class Pozorovatel(ABC):
    @abstractmethod
    def aktualizuj(self, zprava: str) -> None:
        pass

class Subjekt:
    def __init__(self):
        self._pozorovatele: List[Pozorovatel] = []

    def prihlasit(self, pozorovatel: Pozorovatel) -> None:
        self._pozorovatele.append(pozorovatel)

    def odhlasit(self, pozorovatel: Pozorovatel) -> None:
        self._pozorovatele.remove(pozorovatel)

    def notifikuj(self, zprava: str) -> None:
        for p in self._pozorovatele:
            p.aktualizuj(zprava)

class SkladZbozi(Subjekt):
    def __init__(self):
        super().__init__()
        self._pocet = 0

    def naskladni(self, pocet: int) -> None:
        self._pocet += pocet
        self.notifikuj(f"Naskladněno {pocet} kusů, celkem: {self._pocet}")

class EmailUpozorneni(Pozorovatel):
    def aktualizuj(self, zprava: str) -> None:
        print(f"[EMAIL] {zprava}")

class SMSUpozorneni(Pozorovatel):
    def aktualizuj(self, zprava: str) -> None:
        print(f"[SMS] {zprava}")

sklad = SkladZbozi()
sklad.prihlasit(EmailUpozorneni())
sklad.prihlasit(SMSUpozorneni())
sklad.naskladni(100)
# [EMAIL] Naskladněno 100 kusů, celkem: 100
# [SMS] Naskladněno 100 kusů, celkem: 100
```

### Strategy (strategie)

Definuje rodinu algoritmů, každý v samostatné třídě. Algoritmus lze **vyměnit za běhu**.

```python
from abc import ABC, abstractmethod

class RaziciStrategie(ABC):
    @abstractmethod
    def serad(self, data: list) -> list:
        pass

class BubbleSort(RaziciStrategie):
    def serad(self, data: list) -> list:
        arr = data.copy()
        for i in range(len(arr)):
            for j in range(len(arr) - i - 1):
                if arr[j] > arr[j+1]:
                    arr[j], arr[j+1] = arr[j+1], arr[j]
        return arr

class QuickSort(RaziciStrategie):
    def serad(self, data: list) -> list:
        if len(data) <= 1:
            return data
        pivot = data[len(data) // 2]
        return (self.serad([x for x in data if x < pivot]) +
                [x for x in data if x == pivot] +
                self.serad([x for x in data if x > pivot]))

class Sorter:
    def __init__(self, strategie: RaziciStrategie):
        self._strategie = strategie

    def nastav_strategii(self, strategie: RaziciStrategie):
        self._strategie = strategie   # výměna za běhu

    def serad(self, data: list) -> list:
        return self._strategie.serad(data)

sorter = Sorter(BubbleSort())
print(sorter.serad([3, 1, 4, 1, 5]))   # [1, 1, 3, 4, 5]

sorter.nastav_strategii(QuickSort())   # výměna strategie
print(sorter.serad([3, 1, 4, 1, 5]))   # [1, 1, 3, 4, 5]
```

---

## Přehled vzorů

| Kategorie | Vzor | K čemu |
|---|---|---|
| Creational | Singleton | Jedna instance v celém programu |
| Creational | Factory | Vytvoření objektu bez specifikace přesné třídy |
| Creational | Builder | Krok po kroku sestavení složitého objektu |
| Structural | Decorator | Přidání chování bez změny třídy |
| Structural | Adapter | Propojení nekompatibilních rozhraní |
| Structural | Facade | Jednoduché rozhraní ke složitému podsystému |
| Behavioral | Observer | Notifikace pozorovatelů při změně stavu |
| Behavioral | Strategy | Zaměnitelné algoritmy |

---

## Shrnutí

- **Creational** — jak vytvářet objekty (Singleton, Factory, Builder)
- **Structural** — jak skládat objekty (Decorator, Adapter, Facade)
- **Behavioral** — jak objekty komunikují (Observer, Strategy)
- Vzory nejsou kód k zkopírování — jsou to šablony pro myšlení

---

## Typické doplňující otázky

### Kdy použít Singleton a co je jeho nevýhoda?
Singleton pro sdílený stav (konfigurace, logger, DB spojení). Nevýhoda: globální stav ztěžuje testování a může způsobit skryté závislosti.

### Jaký je rozdíl mezi Decorator a Inheritance (dědičností)?
Dědičnost přidává chování staticky (při psaní kódu). Decorator přidává chování dynamicky za běhu, lze vrstvit více dekorátorů.

### Co je rozdíl mezi Factory a Builder?
Factory vytvoří objekt v jednom kroku. Builder staví komplexní objekt postupně (krok po kroku), vhodné pro objekty s mnoha volitelnými parametry.

### Kde se Observer vzor reálně používá?
Všude kde jsou eventy — GUI (button click → handler), MVC (Model notifikuje View), React (state changes → re-render), databázové triggery.
