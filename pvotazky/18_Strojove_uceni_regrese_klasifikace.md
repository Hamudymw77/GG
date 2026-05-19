# 18. Strojové učení — regrese a klasifikace

---

## Co je supervised learning

Supervised learning (učení s učitelem) je typ ML, kde model se učí z dat, která mají **správné odpovědi (labels)**. Cílem je naučit model predikovat výstup pro nová data.

```
Vstup (features) + Správná odpověď (label)
              │
              ▼
         Trénování
              │
              ▼
        Natrénovaný model
              │
              ▼
Nový vstup → Predikce
```

### Typy supervised learningu

| Typ | Výstup | Příklad |
|---|---|---|
| **Regrese** | Číslo (spojitý) | Cena domu, teplota zítřka |
| **Klasifikace** | Kategorie (diskrétní) | Spam/ne-spam, druh květiny |

---

## Regrese

### Lineární regrese

Modeluje **lineární vztah** mezi vstupními proměnnými (X) a číselným výstupem (y).

$$y = w_0 + w_1 x_1 + w_2 x_2 + ... + w_n x_n$$

Cílem je najít váhy $w_i$ minimalizující chybu (MSE).

```python
import numpy as np
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error, r2_score

# Ukázková data — cena domu podle plochy
np.random.seed(42)
X = np.random.randint(40, 200, 100).reshape(-1, 1)   # plocha m2
y = X.ravel() * 25000 + np.random.randn(100) * 100000  # cena Kc

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# Trénování
model = LinearRegression()
model.fit(X_train, y_train)

# Predikce
y_pred = model.predict(X_test)

# Metriky
mse = mean_squared_error(y_test, y_pred)
r2 = r2_score(y_test, y_pred)
print(f"RMSE: {np.sqrt(mse):,.0f} Kc")    # odmocnina MSE — ve stejných jednotkách
print(f"R2: {r2:.3f}")                     # 1.0 = perfektní, 0 = průměr

# Koeficienty
print(f"Sklon: {model.coef_[0]:,.0f} Kc/m2")
print(f"Intercept: {model.intercept_:,.0f} Kc")
```

### Regresní metriky

| Metrika | Vzorec | Co říká |
|---|---|---|
| MSE | $\frac{1}{n}\sum(y - \hat{y})^2$ | Průměr čtvercových chyb (citlivý na outliers) |
| RMSE | $\sqrt{MSE}$ | Ve stejných jednotkách jako y; snadnější interpretace |
| MAE | $\frac{1}{n}\sum|y - \hat{y}|$ | Průměr absolutních chyb; odolnější vůči outliers |
| R² | $1 - \frac{SS_{res}}{SS_{tot}}$ | Podíl rozptylu vysvětlený modelem; 1 = perfektní |

---

## Klasifikace

### Logistická regrese

Navzdory názvu je to klasifikátor. Modeluje **pravděpodobnost** příslušnosti k třídě (0 nebo 1).

$$P(y=1) = \sigma(w^T x) = \frac{1}{1 + e^{-w^T x}}$$

Sigmoid funkce $\sigma$ převede libovolné číslo na pravděpodobnost (0, 1).

```python
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report, confusion_matrix

# Iris dataset — 3 druhy kosatců
iris = load_iris()
X, y = iris.data, iris.target

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

model = LogisticRegression(max_iter=1000)
model.fit(X_train, y_train)
y_pred = model.predict(X_test)

print(f"Accuracy: {accuracy_score(y_test, y_pred):.3f}")
print(classification_report(y_test, y_pred, target_names=iris.target_names))
```

### Klasifikační metriky

```
Confusion matrix (pro binární klasifikaci):

                  Predikce
                Pos   Neg
Skutečnost  Pos  TP    FN
            Neg  FP    TN

TP = True Positive  (správně spam)
TN = True Negative  (správně ne-spam)
FP = False Positive (označil ne-spam jako spam) — "chyba 1. druhu"
FN = False Negative (propustil spam)            — "chyba 2. druhu"
```

| Metrika | Vzorec | Kdy použít |
|---|---|---|
| Accuracy | $\frac{TP+TN}{total}$ | Vyvážené třídy |
| Precision | $\frac{TP}{TP+FP}$ | Důležité neplašit zbytečně (spam filtr) |
| Recall | $\frac{TP}{TP+FN}$ | Důležité nechybět žádný (rakovina) |
| F1-score | $2 \cdot \frac{P \cdot R}{P + R}$ | Kompromis precision/recall |

```python
from sklearn.metrics import confusion_matrix, classification_report

print(confusion_matrix(y_test, y_pred))
print(classification_report(y_test, y_pred))
```

---

## Algoritmy klasifikace

### K-Nearest Neighbors (KNN)

Klasifikuje nový bod podle **k nejbližších sousedů** v trénovacích datech (majorita rozhoduje).

```python
from sklearn.neighbors import KNeighborsClassifier

model = KNeighborsClassifier(n_neighbors=5)   # 5 nejbližších sousedů
model.fit(X_train, y_train)
print(f"Accuracy: {model.score(X_test, y_test):.3f}")
```

**Výhody:** Jednoduchý, žádné trénování (lazy learner), intuitivní.
**Nevýhody:** Pomalý na velkých datech, citlivý na měřítko (nutná normalizace), problémy ve vysokých dimenzích.

### Decision Tree (rozhodovací strom)

Rekurzivně dělí data na základě podmínek (if-else) pro minimalizaci nečistoty.

```python
from sklearn.tree import DecisionTreeClassifier
from sklearn import tree
import matplotlib.pyplot as plt

model = DecisionTreeClassifier(max_depth=3, random_state=42)
model.fit(X_train, y_train)

# Vizualizace stromu
plt.figure(figsize=(12, 6))
tree.plot_tree(model, feature_names=iris.feature_names, class_names=iris.target_names, filled=True)
plt.show()

print(f"Accuracy: {model.score(X_test, y_test):.3f}")
```

### Random Forest

Soubor (ensemble) mnoha rozhodovacích stromů. Každý strom trénován na náhodné podmnožině dat a featur. Výsledek = hlasování stromů.

```python
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)
print(f"Accuracy: {model.score(X_test, y_test):.3f}")

# Důležitost featur
for name, importance in zip(iris.feature_names, model.feature_importances_):
    print(f"{name}: {importance:.3f}")
```

### SVM (Support Vector Machine)

Hledá **hyperrovinu** (decision boundary) maximalizující mezeru (margin) mezi třídami.

```python
from sklearn.svm import SVC
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline

# SVM vyžaduje normalizaci!
pipeline = Pipeline([
    ("scaler", StandardScaler()),
    ("svm", SVC(kernel="rbf", C=1.0))
])

pipeline.fit(X_train, y_train)
print(f"Accuracy: {pipeline.score(X_test, y_test):.3f}")
```

---

## Srovnání algoritmů

| Algoritmus | Rychlost trénování | Přesnost | Interpretovatelnost |
|---|---|---|---|
| Lineární regrese | Rychlá | Nízká-střední | Vysoká |
| Logistická regrese | Rychlá | Střední | Vysoká |
| KNN | Žádné trénování | Střední | Střední |
| Decision Tree | Střední | Střední | Vysoká |
| Random Forest | Pomalá | Vysoká | Nízká |
| SVM | Pomalá | Vysoká | Nízká |

---

## Shrnutí

- **Regrese** = predikce čísla (MSE, RMSE, R² jako metriky)
- **Klasifikace** = predikce kategorie (accuracy, precision, recall, F1)
- **Lineární regrese** = lineární vztah; **Logistická regrese** = klasifikátor přes sigmoid
- **KNN** = lazy learner, k nejbližších sousedů
- **Decision Tree** = if-else strom; **Random Forest** = ensemble stromů
- **SVM** = maximalizuje margin mezi třídami; nutná normalizace

---

## Typické doplňující otázky

### Co je overfitting a jak mu předejít?
Overfitting = model se naučil trénovací data příliš dobře (i šum). Řešení: více dat, regularizace, dropout, pruning stromu, cross-validation.

### Kdy použít precision vs recall?
Precision = nechci falešné poplachy (spam filtr). Recall = nechci nic přehlédnout (detekce rakoviny). F1 je kompromis.

### Proč Random Forest funguje lépe než jeden strom?
Jeden strom snadno overfituje. Random Forest kombinuje mnoho stromů trénovaných na různých podmnožinách dat — průměrování snižuje variance (chybu).

### Co je feature importance?
Míra, jak moc každá vstupní proměnná přispívá k predikci. Random Forest počítá ji automaticky — umožní odstranit nepotřebné features (feature selection).
