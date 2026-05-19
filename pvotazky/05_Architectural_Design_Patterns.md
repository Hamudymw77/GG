# 5. Architektonické vzory (MVC, Multitier, Monolithic, P2P, Client/Server)

---

## Co jsou architektonické vzory

Architektonický vzor je **vysokoúrovňová šablona** pro organizaci celého systému — jak jsou rozděleny komponenty, jak spolu komunikují, jak se škáluje. Na rozdíl od návrhových vzorů (které řeší konkrétní problémy v kódu) architektonické vzory řeší celkovou strukturu aplikace.

Výběr správné architektury závisí na:
- Velikosti a složitosti systému
- Počtu uživatelů
- Požadavcích na škálování
- Týmu a technologiích

---

## Client/Server

**Základní architektura internetu.** Klient (browser, mobilní app) posílá požadavky, server je zpracovává a vrací odpověď.

```
Klient                    Server
┌──────┐  HTTP request   ┌──────────┐
│      │ ───────────────►│          │
│      │  HTTP response  │  Server  │──► Databáze
│      │ ◄───────────────│          │
└──────┘                 └──────────┘
```

- **Výhody:** jednoduchá správa, centralizovaná data, snadné aktualizace
- **Nevýhody:** single point of failure (server spadne → vše nefunguje), škálování

```python
# Jednoduchý HTTP server v Pythonu
from http.server import HTTPServer, BaseHTTPRequestHandler

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b"Hello from server!")

server = HTTPServer(("localhost", 8080), Handler)
server.serve_forever()
```

---

## MVC — Model View Controller

MVC rozděluje aplikaci do **tří vrstev** se striktně oddělenými zodpovědnostmi.

```
Uživatel
  │
  ▼
Controller ──► Model ──► Databáze
  │              │
  ▼              ▼
View ◄──────────────
```

- **Model** — data a business logika; neví nic o zobrazení
- **View** — zobrazení dat uživateli; šablony, HTML, GUI
- **Controller** — přijímá vstupy, koordinuje Model a View

### Proč MVC

Bez MVC je vše smíchané — logika v šabloně, databázové dotazy v HTML. MVC to odděluje, takže:
- Lze měnit View bez změny Modelu
- Lze testovat Model nezávisle
- Tým může pracovat paralelně (frontend/backend)

```python
# Příklad MVC struktury (zjednodušeno)

# MODEL — data a logika
class UzivatelModel:
    def __init__(self):
        self.uzivatele = []

    def pridat(self, jmeno, email):
        self.uzivatele.append({"jmeno": jmeno, "email": email})

    def vsichni(self):
        return self.uzivatele

# VIEW — zobrazení
class UzivatelView:
    def zobraz(self, uzivatele):
        for u in uzivatele:
            print(f"{u['jmeno']} ({u['email']})")

# CONTROLLER — koordinace
class UzivatelController:
    def __init__(self):
        self.model = UzivatelModel()
        self.view = UzivatelView()

    def pridat_uzivatele(self, jmeno, email):
        self.model.pridat(jmeno, email)

    def zobraz_vse(self):
        data = self.model.vsichni()
        self.view.zobraz(data)

ctrl = UzivatelController()
ctrl.pridat_uzivatele("Jan", "jan@test.cz")
ctrl.zobraz_vse()
```

Frameworky používající MVC: Django (Python), Ruby on Rails, Laravel (PHP), Spring MVC (Java).

---

## Multitier (vícevrstvá architektura)

Rozdělení aplikace do **fyzicky oddělených vrstev** (tiers). Nejčastější je 3-tier:

```
┌─────────────────┐
│  Presentation   │  ← Frontend (browser, mobilní app)
│     Tier        │
└────────┬────────┘
         │  HTTP/REST
┌────────▼────────┐
│   Application   │  ← Business logika (API server)
│      Tier       │
└────────┬────────┘
         │  SQL/ORM
┌────────▼────────┐
│     Data        │  ← Databáze (MySQL, PostgreSQL)
│     Tier        │
└─────────────────┘
```

- **Presentation Tier** — co vidí uživatel; HTML/JS/CSS nebo mobilní app
- **Application Tier** — zpracování požadavků, business pravidla, API
- **Data Tier** — ukládání a načítání dat; databázový server

**Výhody:** každou vrstvu lze škálovat nezávisle, bezpečnost (DB není přímo přístupná z internetu), snadná výměna vrstvy.

---

## Monolitická architektura

Celá aplikace je **jeden celek** — jeden deployment, jeden proces. Všechny části jsou propojeny a nasazují se dohromady.

```
┌─────────────────────────────────────┐
│           MONOLIT                   │
│  ┌──────┐ ┌──────┐ ┌─────────────┐ │
│  │ User │ │Order │ │  Payment    │ │
│  │module│ │module│ │   module    │ │
│  └──────┘ └──────┘ └─────────────┘ │
└─────────────────┬───────────────────┘
                  │
             ┌────▼────┐
             │   DB    │
             └─────────┘
```

**Výhody:**
- Jednoduché na vývoj a testování na začátku
- Jednoduchý deployment
- Přímé volání funkcí mezi moduly (rychlé)

**Nevýhody:**
- Škálování = musíš škálovat celou aplikaci
- Jedna chyba může shodit vše
- S růstem se stává nepřehlednou
- Změna jedné části vyžaduje nasazení celku

**Vhodné pro:** malé projekty, startupy v raných fázích, jednoduchý systém.

---

## Microservices (mikroslužby) — opak monolitu

Každá funkce je **samostatná služba** s vlastní databází, komunikují přes API. Zmíníš jako alternativu k monolitu.

```
┌──────────┐  ┌──────────┐  ┌──────────┐
│  User    │  │  Order   │  │ Payment  │
│ Service  │  │ Service  │  │ Service  │
│  + DB    │  │  + DB    │  │  + DB    │
└────┬─────┘  └────┬─────┘  └────┬─────┘
     └──────────────┼─────────────┘
                 API Gateway
```

**Výhody:** nezávislé škálování, nezávislý deployment, odolnost (jedna služba spadne, ostatní fungují).
**Nevýhody:** složitost, síťová komunikace je pomalejší než přímé volání.

---

## P2P — Peer-to-Peer

Každý účastník je **zároveň klient i server**. Žádný centrální server. Příklady: BitTorrent, blockchain.

```
  Peer A ◄──────► Peer B
    ▲                ▲
    │                │
    ▼                ▼
  Peer C ◄──────► Peer D
```

**Výhody:** žádný single point of failure, decentralizace, škáluje samo.
**Nevýhody:** těžká správa, bezpečnost, vyhledávání zdrojů je složité.

---

## Porovnání architektur

| Architektura | Škálování | Složitost | Vhodné pro |
|---|---|---|---|
| Client/Server | Střední | Nízká | Většina webových app |
| MVC | Střední | Nízká | Webové frameworky |
| 3-Tier | Vysoké | Střední | Enterprise aplikace |
| Monolith | Nízké | Nízká na začátku | Malé projekty |
| Microservices | Velmi vysoké | Velmi vysoká | Velké systémy (Netflix, Amazon) |
| P2P | Velmi vysoké | Vysoká | Sdílení souborů, blockchain |

---

## Shrnutí

- **Client/Server** = klient posílá požadavky, server odpovídá; základ webu
- **MVC** = oddělení dat (Model), zobrazení (View) a logiky (Controller)
- **3-Tier** = fyzicky oddělené vrstvy: prezentace, aplikace, data
- **Monolith** = vše v jednom; jednoduché, ale těžko škálovatelné
- **Microservices** = každá funkce = samostatná služba; flexibilní, ale složité
- **P2P** = každý je klient i server; decentralizované

---

## Typické doplňující otázky

### Jaký je rozdíl mezi MVC a 3-tier?
MVC je softwarový vzor pro organizaci kódu v jedné aplikaci. 3-tier je fyzická architektura kde vrstvy běží na různých serverech.

### Kdy přejít z monolitu na microservices?
Když monolit roste tak, že tým nemůže pracovat bez konfliktů, nebo když různé části potřebují různé škálování. Netflix přešel z monolitu na 600+ microservices postupně.

### Co je REST API?
Architektonický styl pro komunikaci mezi klientem a serverem přes HTTP. Používá URL pro zdroje a HTTP metody (GET, POST, PUT, DELETE) pro operace.
