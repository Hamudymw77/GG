# 12. Metodiky a životní cyklus vývoje softwaru

---

## Co je životní cyklus vývoje softwaru (SDLC)

Software Development Life Cycle (SDLC) je strukturovaný proces popisující **všechny fáze vývoje softwaru** — od nápadu až po nasazení a údržbu. Každá metodika tyto fáze organizuje jiným způsobem.

### Základní fáze SDLC

1. **Analýza požadavků** — co má systém dělat, kdo jsou uživatelé, jaká jsou omezení
2. **Návrh (Design)** — architektura, datový model, UI, technologie
3. **Implementace (Coding)** — samotné programování
4. **Testování** — ověření správnosti, výkonu, bezpečnosti
5. **Nasazení (Deployment)** — zpřístupnění uživatelům
6. **Údržba** — opravy chyb, nové funkce, aktualizace

---

## Waterfall (vodopád)

Lineární sekvenční přístup — každá fáze musí být **zcela dokončena** před začátkem další. Nejstarší metodika (60. léta).

```
Analýza
  ↓
Návrh
  ↓
Implementace
  ↓
Testování
  ↓
Nasazení
  ↓
Údržba
```

**Výhody:**
- Jasná dokumentace, dobře definované fáze
- Vhodné pro projekty s pevnými, neměnnými požadavky
- Snadná správa a sledování pokroku

**Nevýhody:**
- Velmi špatná reakce na změny — změna v pozdní fázi = drahé předělávání
- Zákazník vidí výsledek až na konci (může zklamat)
- Testování až po implementaci — chyby se odhalí pozdě

**Kdy se hodí:** vojenské systémy, vládní projekty, hardwarové projekty kde požadavky se nemění.

---

## Agile (agilní metodiky)

Agilní metodiky jsou skupinou přístupů reagujících **iterativně** na měnící se požadavky. Vznikly jako reakce na problémy Waterfallu.

### Agile Manifesto (2001) — 4 hodnoty

1. **Jedinci a interakce** nad procesy a nástroji
2. **Fungující software** nad obsáhlou dokumentací
3. **Spolupráce se zákazníkem** nad sjednáváním smluv
4. **Reakce na změnu** nad dodržováním plánu

### Scrum — nejrozšířenější agilní framework

Práce je rozdělena do **sprintů** (2-4 týdny). Na konci každého sprintu je fungující inkrement produktu.

**Role v Scrumu:**
- **Product Owner** — prioritizuje backlog, zastupuje zákazníka
- **Scrum Master** — facilitátor, odstraňuje překážky, hlídá proces
- **Development Team** — vývojáři (3-9 lidí), sami organizují práci

**Artefakty:**
- **Product Backlog** — seznam všech požadavků (user stories), prioritizovaný
- **Sprint Backlog** — co tým dělá v aktuálním sprintu
- **Inkrement** — fungující část softwaru na konci sprintu

**Události:**
- **Sprint Planning** — co budeme dělat v tomto sprintu
- **Daily Standup** — každodenní 15min: co jsem udělal, co budu dělat, co mě blokuje
- **Sprint Review** — prezentace inkrementu zákazníkovi
- **Retrospektiva** — co šlo dobře, co zlepšit

```
Product Backlog
      │
      ▼
Sprint Planning
      │
      ▼
Sprint (2-4 týdny)
  ├── Daily Standup (každý den)
  └── Development
      │
      ▼
Sprint Review + Retrospektiva
      │
      ▼ (opakuj)
Fungující inkrement
```

### Kanban

Vizualizace toku práce pomocí **Kanban tabule**. Méně strukturované než Scrum — žádné sprinty, focus na plynulý tok.

```
┌──────────┬──────────┬──────────┬──────────┐
│  Backlog │   TODO   │   WIP    │   Done   │
├──────────┼──────────┼──────────┼──────────┤
│ Feature A│ Feature C│ Feature E│ Feature G│
│ Feature B│ Feature D│          │ Feature H│
└──────────┴──────────┴──────────┴──────────┘
```

WIP Limit (Work In Progress) — omezení počtu úkolů rozpracovaných najednou.

---

## Extrémní programování (XP)

Sada technických praktik pro vysokou kvalitu kódu:
- **Pair programming** — dva vývojáři u jednoho počítače
- **Test-Driven Development (TDD)** — nejdříve test, pak kód
- **Continuous Integration** — průběžná integrace a testování
- **Refactoring** — průběžné zlepšování kódu
- **Simple design** — nejjednodušší řešení které funguje

---

## DevOps

DevOps spojuje **vývoj (Development)** a **provoz (Operations)**. Cílem je rychlejší a spolehlivější nasazení softwaru.

```
Plan → Code → Build → Test → Release → Deploy → Operate → Monitor
  ↑__________________________________________________|
                   (smyčka zpětné vazby)
```

**Klíčové praktiky:**
- **CI/CD** (Continuous Integration / Continuous Deployment) — automatické buildy, testy a nasazení
- **Infrastructure as Code** — infrastruktura definovaná kódem (Terraform, Ansible)
- **Kontejnerizace** — Docker, Kubernetes
- **Monitoring** — sledování aplikace v produkci (Prometheus, Grafana)

---

## Porovnání metodik

| Aspekt | Waterfall | Scrum | Kanban |
|---|---|---|---|
| Iterace | Ne (sekvenční) | Sprinty (2-4 týdny) | Kontinuální |
| Změny požadavků | Špatně | Dobře | Velmi dobře |
| Dokumentace | Rozsáhlá | Minimální | Minimální |
| Vhodné pro | Pevné požadavky | Produktový vývoj | Údržba, podpora |
| Zákazník | Na začátku a konci | Každý sprint | Průběžně |

---

## Verzovací systémy — Git

Verzovací systém sleduje změny v kódu a umožňuje týmovou spolupráci.

### Základní Git workflow

```bash
# Základní příkazy (pro pochopení, ne nutně spouštět)
git init                    # inicializace repozitáře
git add .                   # přidání změn do staging area
git commit -m "zpráva"      # uložení snapshotu
git branch feature-x        # nová větev
git checkout feature-x      # přepnutí na větev
git merge feature-x         # sloučení větve
git push origin main        # nahrání na remote
git pull                    # stažení změn
```

### Gitflow — branching strategie

```
main ─────────────────────────────────── (produkce)
  │
develop ─────────────────────────────── (integrace)
  │
  ├── feature/login ─────────────────── (nová funkce)
  ├── feature/payment ────────────────
  └── hotfix/security-fix ────────────  (oprava chyby v produkci)
```

---

## Shrnutí

- **SDLC** = životní cyklus: analýza → návrh → implementace → testování → nasazení → údržba
- **Waterfall** = sekvenční, každá fáze hotová před další; špatná reakce na změny
- **Scrum** = iterativní sprinty (2-4 týdny), denní standup, Product Owner, Scrum Master
- **Kanban** = vizualizace na tabuli, WIP limity, kontinuální tok
- **DevOps** = propojení vývoje a provozu; CI/CD, automatizace, monitoring
- **Git** = verzovací systém; commit, branch, merge, push/pull

---

## Typické doplňující otázky

### Jaký je hlavní rozdíl mezi Waterfall a Agile?
Waterfall je sekvenční — nelze se vrátit. Agile je iterativní — každý sprint přináší fungující software a lze reagovat na změny požadavků.

### Co je user story?
Požadavek na funkcionalitu popsaný z pohledu uživatele: "Jako zákazník chci moci platit kartou, abych nemusel nosit hotovost." Obsahuje kritéria přijatelnosti.

### Co je CI/CD?
Continuous Integration = automatické sestavení a testování po každém commitu. Continuous Delivery/Deployment = automatické nasazení do produkce (nebo staging).

### Proč daily standup trvá max 15 minut?
Je to status meeting, ne diskuze. Tři otázky: co jsem udělal, co budu dělat, co mě blokuje. Problémy se řeší mimo standup.
