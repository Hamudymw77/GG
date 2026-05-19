# 11. Komunikace v síti — tvorba síťových aplikací, Berkeley socket

---

## Síťová komunikace — základy

Aby dva programy (na stejném nebo různém počítači) mohly komunikovat, potřebují:
- **IP adresu** — identifikuje počítač v síti
- **Port** — identifikuje konkrétní aplikaci/službu na počítači
- **Protokol** — pravidla komunikace (TCP, UDP, HTTP...)

```
Klient (192.168.1.10)           Server (192.168.1.20)
         │                              │
         │  TCP spojení na port 8080    │
         │ ─────────────────────────► │
         │  data (request)             │
         │ ─────────────────────────► │
         │  data (response)            │
         │ ◄───────────────────────── │
```

### TCP vs UDP

| | TCP | UDP |
|---|---|---|
| Spojení | Orientované na spojení (handshake) | Bez spojení |
| Spolehlivost | Garantuje doručení, pořadí | Bez záruky |
| Rychlost | Pomalejší | Rychlejší |
| Použití | HTTP, email, databáze | Streaming, DNS, VoIP |

---

## Berkeley Socket API

Socket je **abstrakce síťového spojení** — lze ho číst a zapisovat jako soubor. Berkeley socket je standardní API pocházející z BSD Unixu, dnes dostupné ve všech OS a programovacích jazycích.

### Typy socketů

- `SOCK_STREAM` — TCP, spolehlivý proud bytů
- `SOCK_DGRAM` — UDP, datagrams bez záruky

### Životní cyklus socketu

```
SERVER:                          KLIENT:
socket()                         socket()
bind() — přiřadí adresu/port
listen() — čeká na spojení
accept() ◄─────────────────── connect()
recv()/send() ◄──────────────► send()/recv()
close()                          close()
```

---

## TCP Server a Klient

### Jednoduchý echo server

```python
# server.py
import socket

HOST = "localhost"
PORT = 8080

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server:
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server.bind((HOST, PORT))      # přiřaď adresu a port
    server.listen(5)               # max 5 čekajících spojení
    print(f"Server běží na {HOST}:{PORT}")

    while True:
        klient_socket, adresa = server.accept()   # čeká na připojení
        print(f"Připojen: {adresa}")

        with klient_socket:
            while True:
                data = klient_socket.recv(1024)   # přijmi max 1024 bytů
                if not data:
                    break
                print(f"Přijato: {data.decode()}")
                klient_socket.sendall(data)        # pošli zpět (echo)
```

```python
# klient.py
import socket

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.connect(("localhost", 8080))    # připoj se k serveru
    s.sendall(b"Ahoj servere!")       # pošli data (bytes!)
    odpoved = s.recv(1024)
    print(f"Server odpověděl: {odpoved.decode()}")
```

### Vícevláknový server (obsluhuje více klientů)

```python
import socket
import threading

def obsluha_klienta(klient_socket, adresa):
    print(f"Nový klient: {adresa}")
    try:
        while True:
            data = klient_socket.recv(1024)
            if not data:
                break
            klient_socket.sendall(data.upper())   # echo velkými písmeny
    finally:
        klient_socket.close()
        print(f"Klient odpojen: {adresa}")

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server:
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server.bind(("localhost", 8080))
    server.listen()

    while True:
        klient, adresa = server.accept()
        vlakno = threading.Thread(
            target=obsluha_klienta,
            args=(klient, adresa)
        )
        vlakno.daemon = True   # vlákno skončí se serverem
        vlakno.start()
```

---

## UDP Socket

UDP je rychlejší, ale nezaručuje doručení ani pořadí. Vhodné pro real-time aplikace (hry, VoIP).

```python
# UDP server
import socket

with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as server:
    server.bind(("localhost", 9090))
    print("UDP server běží")

    while True:
        data, adresa = server.recvfrom(1024)   # přijmi datagram + adresu odesílatele
        print(f"Od {adresa}: {data.decode()}")
        server.sendto(b"Prijato", adresa)       # odpověz na adresu odesílatele

# UDP klient
with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as klient:
    klient.sendto(b"Ahoj UDP!", ("localhost", 9090))
    odpoved, _ = klient.recvfrom(1024)
    print(odpoved.decode())
```

---

## HTTP komunikace — requests knihovna

Pro HTTP API nepoužíváš raw sockety, ale knihovnu `requests`.

```python
import requests

# GET — načtení dat
odpoved = requests.get("https://jsonplaceholder.typicode.com/users/1")
print(odpoved.status_code)   # 200
print(odpoved.json())         # data jako Python dict

# POST — odeslání dat
nova_data = {"title": "Test", "body": "Obsah", "userId": 1}
odpoved = requests.post(
    "https://jsonplaceholder.typicode.com/posts",
    json=nova_data
)
print(odpoved.status_code)    # 201 Created

# Timeout a error handling
try:
    r = requests.get("https://api.example.com/data", timeout=5)
    r.raise_for_status()   # vyvolá výjimku pro 4xx, 5xx
    data = r.json()
except requests.Timeout:
    print("Požadavek vypršel")
except requests.HTTPError as e:
    print(f"HTTP chyba: {e}")
except requests.ConnectionError:
    print("Nelze se připojit")
```

---

## Jednoduchý HTTP server

```python
from http.server import HTTPServer, BaseHTTPRequestHandler
import json

class ApiHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == "/status":
            self.send_response(200)
            self.send_header("Content-Type", "application/json")
            self.end_headers()
            odpoved = json.dumps({"stav": "OK", "verze": "1.0"})
            self.wfile.write(odpoved.encode())
        else:
            self.send_response(404)
            self.end_headers()

    def log_message(self, format, *args):
        pass   # potlačení výchozího logování

server = HTTPServer(("localhost", 8080), ApiHandler)
print("HTTP server na portu 8080")
server.serve_forever()
```

---

## Shrnutí

- **Socket** = abstrakce síťového spojení; čteš a zapisuješ jako soubor
- **TCP** = spolehlivý (garantuje doručení), pomalejší; pro HTTP, email, DB
- **UDP** = rychlý, bez záruky; pro streaming, DNS, hry
- Životní cyklus serveru: `socket → bind → listen → accept → recv/send → close`
- Pro více klientů najednou: vlákna nebo `asyncio`
- Pro HTTP API: knihovna `requests` místo raw socketů

---

## Typické doplňující otázky

### Jaký je rozdíl mezi TCP a UDP?
TCP garantuje doručení, správné pořadí a kontrolu chyb (handshake, ACK). UDP je bez záruky — rychlejší, ale pakety mohou přijít pozdě, mimo pořadí nebo vůbec.

### Co je port a k čemu slouží?
Port je číslo (0-65535) identifikující konkrétní aplikaci/službu na počítači. Webserver typicky běží na 80 (HTTP) nebo 443 (HTTPS), SSH na 22, MySQL na 3306.

### Co znamená `bind()` u serveru?
Server si "zarezervuje" konkrétní adresu a port. Operační systém pak přesměruje příchozí spojení na tento socket.

### Proč je `setsockopt(SO_REUSEADDR, 1)` důležité?
Umožní okamžité znovu použití portu po restartu serveru. Bez toho musíš čekat minutu než OS uvolní port ze stavu TIME_WAIT.
