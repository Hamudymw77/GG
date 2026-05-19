# 17. Strojové učení — příprava dat, bias, korelace

---

## Co je strojové učení

Strojové učení (Machine Learning, ML) je oblast AI, kde algoritmy **učí se z dat** místo aby byly explicitně naprogramovány. Program sám nalezne vzory a pravidla v datech.

```
Tradiční programování:  Pravidla + Data → Výsledky
Strojové učení:         Data + Výsledky → Pravidla (model)
```

### Typy ML

- **Supervised learning (učení s učitelem)** — data mají správné odpovědi (labels); model se učí mapovat vstup → výstup (klasifikace, regrese)
- **Unsupervised learning** — data bez labels; model hledá vzory sám (clustering, redukce dimenzí)
- **Reinforcement learning** — agent se učí akcemi v prostředí, dostává odměny/tresty

---

## Životní cyklus ML projektu

```
1. Definice problému
2. Sběr dat
3. Příprava dat (EDA, čištění, feature engineering)
4. Výběr modelu
5. Trénování
6. Vyhodnocení
7. Nasazení
8. Monitoring
```

Příprava dat (kroky 2-3) typicky zabere **60-80 % času** celého projektu.

---

## Pandas — práce s daty

Pandas je hlavní Python knihovna pro manipulaci s tabulkovými daty.

```python
import pandas as pd
import numpy as np

# Načtení dat
df = pd.read_csv("data.csv", encoding="utf-8")

# Základní průzkum dat (EDA — Exploratory Data Analysis)
print(df.shape)          # (počet řádků, počet sloupců)
print(df.dtypes)         # datové typy sloupců
print(df.head())         # prvních 5 řádků
print(df.describe())     # statistiky: min, max, mean, std, quartily
print(df.info())         # přehled: typy, chybějící hodnoty
```

---

## Chybějící hodnoty (Missing Values)

Reálná data obsahují mezery (NaN = Not a Number). Je nutné se s nimi vypořádat.

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    "vek": [25, np.nan, 30, np.nan, 22],
    "plat": [50000, 60000, np.nan, 45000, 55000],
    "mesto": ["Praha", "Brno", None, "Praha", "Ostrava"]
})

# Detekce chybějících hodnot
print(df.isnull().sum())       # počet NaN v každém sloupci
print(df.isnull().mean())      # procento chybějících hodnot

# Strategie 1: Odstranění řádků s NaN
df_bez_nan = df.dropna()

# Strategie 2: Imputation — doplnění hodnotami
df["vek"].fillna(df["vek"].mean(), inplace=True)       # průměr
df["plat"].fillna(df["plat"].median(), inplace=True)   # medián
df["mesto"].fillna(df["mesto"].mode()[0], inplace=True)  # nejčastější hodnota

# Strategie 3: Interpolace (pro časové řady)
df["hodnota"] = df["hodnota"].interpolate(method="linear")
```

---

## Normalizace a standardizace

Algoritmy ML jsou citlivé na **měřítko** (scale) vstupních dat. Pokud jeden feature má hodnoty 0-1 a druhý 0-1000000, model bude ignorovat ten první.

### Min-Max Normalizace (škálování do [0, 1])

$$x' = \frac{x - x_{min}}{x_{max} - x_{min}}$$

```python
from sklearn.preprocessing import MinMaxScaler

scaler = MinMaxScaler()
df_scaled = scaler.fit_transform(df[["vek", "plat"]])
# Všechny hodnoty v rozsahu [0, 1]

# Vlastnoručně:
def normalizuj(serie):
    return (serie - serie.min()) / (serie.max() - serie.min())
```

### Standardizace (Z-score, mean=0, std=1)

$$x' = \frac{x - \mu}{\sigma}$$

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
df_standard = scaler.fit_transform(df[["vek", "plat"]])
# Mean ≈ 0, std ≈ 1

# Vlastnoručně:
def standardizuj(serie):
    return (serie - serie.mean()) / serie.std()
```

**Kdy co použít:**
- **MinMax** — výstup v pevném rozsahu [0,1]; citlivé na odlehlé hodnoty (outliers)
- **StandardScaler** — gaussovské rozdělení; odolnější vůči outliers; pro většinu algoritmů

---

## Odlehlé hodnoty (Outliers)

Outlier = hodnota, která se výrazně liší od ostatních. Může zkreslit model.

```python
import pandas as pd
import numpy as np

platy = pd.Series([30000, 35000, 40000, 38000, 1000000])   # 1M = outlier

# Metoda IQR (Interquartile Range)
Q1 = platy.quantile(0.25)
Q3 = platy.quantile(0.75)
IQR = Q3 - Q1

dolni = Q1 - 1.5 * IQR
horni = Q3 + 1.5 * IQR

outliers = platy[(platy < dolni) | (platy > horni)]
print(f"Outliery: {outliers.tolist()}")   # [1000000]

# Odstranění outlierů
filtrovane = platy[(platy >= dolni) & (platy <= horni)]
```

---

## Korelace vs Kauzalita

### Korelace

Statistická závislost mezi dvěma proměnnými. Pearsonův korelační koeficient r ∈ [-1, 1]:
- **r = 1** — perfektní pozitivní korelace (obě rostou společně)
- **r = -1** — perfektní negativní korelace (jedna roste, druhá klesá)
- **r = 0** — žádná lineární závislost

```python
import pandas as pd

df = pd.DataFrame({
    "studijni_hodiny": [2, 5, 1, 8, 3, 9, 4],
    "znamka": [3, 1, 4, 1, 2, 1, 2]
})

# Korelační matice
korelace = df.corr()
print(korelace)
#                  studijni_hodiny  znamka
# studijni_hodiny         1.000000  -0.964
# znamka                 -0.964     1.000000

# Vizualizace (heatmap)
import matplotlib.pyplot as plt
import seaborn as sns
sns.heatmap(korelace, annot=True, cmap="coolwarm")
plt.show()
```

### Korelace ≠ Kauzalita!

**Korelace** — dvě veličiny se mění společně.
**Kauzalita** — jedna veličina způsobuje změnu druhé.

Příklady sporných korelací:
- Prodej zmrzliny koreluje s počtem utopení — oboje způsobuje **horké počasí**, ne zmrzlina utopení
- Počet pirátu klesal, CO2 rostlo — neznamená že piráti chrání klima
- Ve státech s více TV koreluje vyšší délka života — oboje souvisí s **bohatstvím**

---

## Bias (zkreslení)

Bias = systematická chyba v datech nebo modelu, která způsobuje zkreslené předpovědi.

### Typy biasu

**Selection bias** — trénovací data nejsou reprezentativní pro reálný svět.
```
Příklad: Trénuji spam filtr pouze na anglické emaily → špatně funguje pro české.
```

**Confirmation bias** — sběr dat potvrzující naše předpoklady, ignorování opačných.

**Historical bias** — data odrážejí historické nerovnosti.
```
Příklad: HR algoritmus trénovaný na historická data → diskriminuje ženy (historicky méně zastoupeny ve vedení).
```

**Label bias** — chybné nebo zaujatě přiřazené labels v trénovacích datech.

### Jak bojovat s biasem
- Různorodá trénovací data (demograficky, geograficky...)
- Pravidelný audit modelu na různých skupinách
- Fairness metriky (equal opportunity, demographic parity)

---

## Train/Test split — správné dělení dat

Model nelze hodnotit na datech, na kterých byl trénován (overfitting).

```python
from sklearn.model_selection import train_test_split
import pandas as pd

X = df.drop("cil", axis=1)   # features (vstupy)
y = df["cil"]                  # target (co chceme předpovědět)

# Rozdělení: 80% trénink, 20% test
X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,
    random_state=42   # reprodukovatelnost
)

print(f"Trénink: {len(X_train)} vzorků")
print(f"Test: {len(X_test)} vzorků")
```

### K-Fold Cross Validation

```python
from sklearn.model_selection import cross_val_score
from sklearn.linear_model import LinearRegression

model = LinearRegression()
# 5-fold CV: data rozdělena na 5 částí, model trénován 5x
skore = cross_val_score(model, X, y, cv=5, scoring="r2")
print(f"CV skóre: {skore.mean():.3f} ± {skore.std():.3f}")
```

---

## Shrnutí

- ML = algoritmy učící se z dat, ne z explicitních pravidel
- Příprava dat = 60-80 % práce: čištění, imputace NaN, normalizace, outliers
- **Normalizace** [0,1] vs **Standardizace** (mean=0, std=1)
- **Korelace** = statistická závislost; **kauzalita** = příčina-důsledek — korelace neimplikuje kauzalitu
- **Bias** = systematické zkreslení dat nebo modelu
- Vždy rozdělit data na train/test (typicky 80/20) nebo použít cross-validation

---

## Typické doplňující otázky

### Proč je normalizace dat důležitá?
Algoritmy jako KNN, SVM nebo neuronové sítě počítají vzdálenosti. Bez normalizace by feature s větším rozsahem dominoval — přestože nemusí být důležitější.

### Co je overfitting?
Model se "naučil" trénovací data příliš dobře — včetně šumu. Na trénovacích datech má výborné výsledky, ale na nových datech selhává. Opak je underfitting — model je příliš jednoduchý.

### Uveď příklad korelace, která není kauzalita.
Producenti zmrzliny mají vyšší tržby v létě, kdy je více utopení. Korelace je silná, ale příčinou je horko, ne zmrzlina.

### Co je feature engineering?
Tvorba nových vstupních proměnných (features) z existujících dat. Například z datumu extrahovat den v týdnu, měsíc, jestli je svátex. Dobrý feature engineering může výrazně zlepšit model.
