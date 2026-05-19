# 22. Vlákna, paralelní a asynchronní programování

---

## Co je souběžnost (concurrency)

Souběžnost = schopnost programu **zdánlivě nebo skutečně vykonávat více úkolů najednou**.

```
Concurrency (souběžnost) — správa více úkolů
    ├── Parallelismus — skutečně najednou (více CPU jader)
    └── Interleaving — střídání na jednom CPU (přepínání kontextu)
```

### Proč potřebujeme souběžnost?
- I/O operace jsou pomalé (disk, síť) — CPU čeká nečinně
- Moderní CPU mají více jader — lze je využít pro výpočty
- UI nesmí "zamrznout" při dlouhé operaci

---

## GIL — Global Interpreter Lock

CPython (standardní Python) má **GIL — globální zámek**, který zabraňuje více vláknům spouštět Python bytekód najednou.

```
Vlákno 1: --- Python kód --- [GIL] --- Python kód ---
Vlákno 2:                [čeká na GIL]   --- Python kód ---

GIL se uvolní při I/O operacích (čekání na síť, disk, sleep)!
```

**Důsledky:**
- Threading v Pythonu **nepomáhá pro CPU-bound úkoly** (výpočty)
- Threading **pomáhá pro I/O-bound úkoly** (síť, soubory) — GIL se uvolní při čekání
- Pro skutečný paralelismus použij **multiprocessing** (každý process má vlastní GIL)

---

## Threading — vlákna

```python
import threading
import time

def stahni_data(url: str, vysledky: dict) -> None:
    print(f"Stahuji {url}...")
    time.sleep(2)   # simulace síťového požadavku
    vysledky[url] = f"Data z {url}"
    print(f"Hotovo: {url}")

urls = ["api.com/users", "api.com/posts", "api.com/comments"]
vysledky = {}
vlakna = []

zacatek = time.time()

for url in urls:
    vlakno = threading.Thread(target=stahni_data, args=(url, vysledky))
    vlakno.start()
    vlakna.append(vlakno)

for vlakno in vlakna:
    vlakno.join()   # počkej až vlákno skončí

print(f"Hotovo za {time.time()-zacatek:.1f}s")   # ~2s místo 6s (paralelně)
```

### Race condition a Lock

Při sdíleném stavu mezi vlákny hrozí **race condition** — výsledek závisí na pořadí provedení.

```python
import threading

citac = 0
zamek = threading.Lock()   # zámek pro mutual exclusion

def inkrementuj(n: int) -> None:
    global citac
    for _ in range(n):
        with zamek:        # jen jedno vlákno najednou
            citac += 1     # kritická sekce — čtení + přírůstek + zápis

vlakna = [threading.Thread(target=inkrementuj, args=(10000,)) for _ in range(10)]
for v in vlakna: v.start()
for v in vlakna: v.join()

print(citac)   # 100000 (správně, bez zámku může být méně)
```

### Daemon vlákna

```python
import threading
import time

def monitor():
    while True:
        print("Monitor běží...")
        time.sleep(1)

vlakno = threading.Thread(target=monitor, daemon=True)
vlakno.start()   # skončí automaticky když hlavní vlákno skončí
```

---

## Multiprocessing — procesy

Každý process má vlastní paměť a vlastní GIL — skutečný paralelismus pro CPU-bound úkoly.

```python
import multiprocessing
import time

def faktorial(n: int) -> int:
    result = 1
    for i in range(2, n + 1):
        result *= i
    return result

if __name__ == "__main__":   # NUTNÉ pro multiprocessing na Windows!
    cisla = [50000, 60000, 70000, 80000]

    zacatek = time.time()

    with multiprocessing.Pool(processes=4) as pool:   # 4 procesy
        vysledky = pool.map(faktorial, cisla)          # každé cislo v jiném procesu

    print(f"Hotovo za {time.time()-zacatek:.2f}s")
```

### Komunikace mezi procesy (Queue)

```python
import multiprocessing

def pracovnik(fronta_vstup, fronta_vystup):
    while True:
        ukol = fronta_vstup.get()
        if ukol is None:
            break
        fronta_vystup.put(ukol ** 2)   # umocni a pošli výsledek

if __name__ == "__main__":
    vstup = multiprocessing.Queue()
    vystup = multiprocessing.Queue()

    process = multiprocessing.Process(target=pracovnik, args=(vstup, vystup))
    process.start()

    for i in range(5):
        vstup.put(i)
    vstup.put(None)   # signál pro ukončení

    process.join()

    vysledky = [vystup.get() for _ in range(5)]
    print(vysledky)   # [0, 1, 4, 9, 16]
```

---

## Asyncio — asynchronní programování

`asyncio` používá **event loop** — jeden thread, ale efektivně přepíná mezi úkoly při čekání na I/O.

### Co je coroutine

Funkce definovaná s `async def` — lze ji pozastavit (await) a obnovit.

```python
import asyncio
import time

async def stahni(url: str) -> str:
    print(f"Začínám: {url}")
    await asyncio.sleep(2)   # await = vzdej CPU jiným úkolům (ne-blokující!)
    print(f"Hotovo: {url}")
    return f"Data z {url}"

async def main():
    zacatek = time.time()

    # Spustí všechny coroutines souběžně
    vysledky = await asyncio.gather(
        stahni("api.com/users"),
        stahni("api.com/posts"),
        stahni("api.com/comments"),
    )

    print(f"Hotovo za {time.time()-zacatek:.1f}s")   # ~2s
    print(vysledky)

asyncio.run(main())
```

### Async HTTP s aiohttp

```python
import asyncio
import aiohttp   # pip install aiohttp

async def ziskej(session, url: str) -> dict:
    async with session.get(url) as odpoved:   # ne-blokující HTTP
        return await odpoved.json()

async def main():
    urls = [
        "https://jsonplaceholder.typicode.com/users/1",
        "https://jsonplaceholder.typicode.com/users/2",
        "https://jsonplaceholder.typicode.com/users/3",
    ]

    async with aiohttp.ClientSession() as session:
        ukoly = [ziskej(session, url) for url in urls]
        vysledky = await asyncio.gather(*ukoly)

    for data in vysledky:
        print(data["name"])

asyncio.run(main())
```

### Async/Await pravidla

```python
# async def = coroutine funkce
async def moje_funkce():
    pass

# await = čekání na coroutine (jen uvnitř async def)
async def main():
    vysledek = await moje_funkce()

    # asyncio.gather = souběžné spuštění více coroutines
    a, b = await asyncio.gather(funkce_a(), funkce_b())
```

---

## concurrent.futures — vysokoúrovňové API

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor
import time

def zpracuj(n: int) -> int:
    time.sleep(0.1)   # simulace I/O
    return n * n

cisla = list(range(20))

# Threading pool pro I/O-bound úkoly
with ThreadPoolExecutor(max_workers=8) as executor:
    vysledky = list(executor.map(zpracuj, cisla))
    print(vysledky)

# Process pool pro CPU-bound úkoly
with ProcessPoolExecutor(max_workers=4) as executor:
    vysledky = list(executor.map(zpracuj, cisla))
```

---

## Kdy použít co

| Situace | Řešení |
|---|---|
| Síť, soubory, DB (I/O-bound) | threading nebo asyncio |
| Výpočty (CPU-bound) | multiprocessing |
| Hodně souběžných I/O operací | asyncio (nejefektivnější) |
| Jednoduchá paralelizace | concurrent.futures |

---

## Shrnutí

- **GIL** = Python zámek; threading pomáhá I/O, ne CPU; pro CPU použij multiprocessing
- **Thread** = sdílená paměť, přepínání OS; `Lock` pro ochranu sdíleného stavu
- **Multiprocessing** = vlastní procesy + vlastní paměť; skutečný paralelismus
- **asyncio** = event loop, jeden thread; `async def`, `await`, `asyncio.gather`; nejlepší pro hodně I/O
- `concurrent.futures` = `ThreadPoolExecutor` (I/O) nebo `ProcessPoolExecutor` (CPU)

---

## Typické doplňující otázky

### Co je race condition?
Situace, kdy výsledek závisí na pořadí provedení operací více vlákny. Kritická sekce (čtení-modifikace-zápis) musí být chráněna zámkem (Lock, Mutex).

### Jaký je rozdíl mezi threading a asyncio?
Threading = OS přepíná vlákna (preemptivní). Asyncio = event loop přepíná coroutines při await (kooperativní). Asyncio je efektivnější pro tisíce souběžných I/O operací, threading je jednodušší na pochopení.

### Proč `if __name__ == "__main__"` u multiprocessing?
Na Windows multiprocessing spouští nové procesy importem skriptu. Bez této podmínky by každý nový process znovu spustil Pool a vznikl by nekonečný cyklus.

### Co je deadlock?
Situace kdy dvě vlákna čekají navzájem na sebe — vlákno A drží zámek 1 a čeká na zámek 2, vlákno B drží zámek 2 a čeká na zámek 1. Řešení: vždy zamykat zámky ve stejném pořadí.
