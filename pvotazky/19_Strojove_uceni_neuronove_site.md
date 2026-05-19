# 19. Strojové učení — neuronové sítě

---

## Co je neuronová síť

Neuronová síť (Neural Network, NN) je model inspirovaný biologickým mozkem — skládá se ze vzájemně propojených **neuronů** organizovaných do vrstev. Naučí se složité vzory v datech, které jsou příliš komplexní pro tradiční ML algoritmy.

Používá se pro: rozpoznávání obrazů, NLP (překlad, chatboti), generování obsahu, předpovídání časových řad.

---

## Biologická inspirace

```
Biologický neuron:          Umělý neuron:
Dendrity (vstupy)           Vstupní hodnoty x1, x2, ..., xn
Synapse (váhy)              Váhy w1, w2, ..., wn + bias b
Tělo (výpočet)              Suma: z = w1*x1 + w2*x2 + ... + b
Axon (výstup)               Aktivační funkce: a = f(z)
```

---

## Perceptron — základní neuron

```python
import numpy as np

def perceptron(vstupy: list, vahy: list, bias: float) -> float:
    z = np.dot(vstupy, vahy) + bias   # vážená suma
    return 1 if z > 0 else 0           # skoková aktivace

# Logická funkce AND
vstupy = [1, 1]
vahy = [0.5, 0.5]
bias = -0.7
print(perceptron(vstupy, vahy, bias))   # 1 (1 AND 1 = True)

vstupy = [0, 1]
print(perceptron(vstupy, vahy, bias))   # 0 (0 AND 1 = False)
```

---

## Architektura vícevrstvé sítě (MLP)

```
Vstupní vrstva    Skrytá vrstva 1   Skrytá vrstva 2   Výstupní vrstva
(Input layer)     (Hidden layer)    (Hidden layer)    (Output layer)

    x1 ──┐
         ├──► h1 ──┐
    x2 ──┤         ├──► h3 ──┐
         ├──► h2 ──┘         ├──► output
    x3 ──┘                   │
                      h4 ────┘

- Každý neuron je propojen se všemi neurony předchozí vrstvy (Dense/Fully Connected)
- Každé spojení má váhu w
- Každý neuron má bias b
```

---

## Aktivační funkce

Aktivační funkce přidávají **nelinearitu** — bez ní by celá síť byla jen lineární transformace.

```python
import numpy as np

# ReLU — Rectified Linear Unit (nejpoužívanější ve skrytých vrstvách)
def relu(z):
    return np.maximum(0, z)
# ReLU(x) = x pokud x > 0, jinak 0

# Sigmoid — pravděpodobnost (výstupní vrstva pro binární klasifikaci)
def sigmoid(z):
    return 1 / (1 + np.exp(-z))
# Sigmoid(x) = 0 az 1

# Softmax — pravděpodobnosti pro více tříd (výstupní vrstva)
def softmax(z):
    exp_z = np.exp(z - np.max(z))   # odečtení maxima pro numerickou stabilitu
    return exp_z / exp_z.sum()
# Softmax vrátí vector součtu = 1.0

# Příklady
print(relu(np.array([-2, -1, 0, 1, 2])))   # [0, 0, 0, 1, 2]
print(sigmoid(np.array([-2, 0, 2])))        # [0.12, 0.5, 0.88]
```

| Funkce | Rozsah | Kde |
|---|---|---|
| ReLU | [0, ∞) | Skryté vrstvy (nejčastější) |
| Sigmoid | (0, 1) | Binární klasifikace — výstup |
| Softmax | (0, 1), suma=1 | Multi-class klasifikace — výstup |
| Tanh | (-1, 1) | Skryté vrstvy (alternativa ReLU) |

---

## Trénování — backpropagation

Jak se síť učí:

1. **Forward pass** — data projdou sítí, dostaneme predikci
2. **Výpočet chyby (loss)** — porovnání predikce se správnou odpovědí
3. **Backward pass (backpropagation)** — gradienty chyby se šíří zpět sítí (chain rule)
4. **Aktualizace vah (Gradient Descent)** — váhy se posunou ve směru snižující chybu

$$w_{nové} = w_{staré} - \eta \cdot \frac{\partial L}{\partial w}$$

$\eta$ = learning rate (jak velký krok), $L$ = loss funkce.

```python
# Ruční gradient descent (ilustrace)
vahy = np.array([0.5, -0.3, 0.8])
learning_rate = 0.01

for epocha in range(100):
    predikce = forward_pass(X, vahy)
    loss = loss_funkce(y, predikce)
    gradient = vypocitej_gradient(X, y, vahy)
    vahy -= learning_rate * gradient   # aktualizace vah
```

---

## Keras / TensorFlow — praktická ukázka

```python
import numpy as np
from tensorflow import keras
from tensorflow.keras import layers
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler, to_categorical

# Data
iris = load_iris()
X = iris.data
y = to_categorical(iris.target, num_classes=3)   # one-hot encoding: [0,1,0]

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Normalizace
scaler = StandardScaler()
X_train = scaler.fit_transform(X_train)
X_test = scaler.transform(X_test)

# Definice modelu
model = keras.Sequential([
    layers.Dense(16, activation="relu", input_shape=(4,)),   # 4 vstupní features
    layers.Dense(8, activation="relu"),                       # skrytá vrstva
    layers.Dense(3, activation="softmax")                     # 3 výstupní třídy
])

model.compile(
    optimizer="adam",             # varianta gradient descent
    loss="categorical_crossentropy",  # loss pro multi-class
    metrics=["accuracy"]
)

model.summary()   # přehled architektury

# Trénování
history = model.fit(
    X_train, y_train,
    epochs=50,           # kolikrát projít celá trénovací data
    batch_size=16,       # kolik vzorků najednou
    validation_split=0.2,  # část tréninku pro validaci
    verbose=0
)

# Vyhodnocení
loss, acc = model.evaluate(X_test, y_test, verbose=0)
print(f"Test accuracy: {acc:.3f}")
```

---

## Overfitting a underfitting

```
Underfitting         Dobrý model         Overfitting
    ↕                    ↕                   ↕
Model příliš         Generalizuje        Naučil se i šum
jednoduchý           dobře               v trénovacích datech

Train acc: nízká     Train acc: vysoká   Train acc: velmi vysoká
Test acc: nízká      Test acc: vysoká    Test acc: nízká
```

### Jak bojovat s overfittingem

```python
# 1. Dropout — náhodně vypíná neurony během trénování
model = keras.Sequential([
    layers.Dense(64, activation="relu"),
    layers.Dropout(0.3),   # 30% neuronů vypnuto při každém kroce
    layers.Dense(32, activation="relu"),
    layers.Dropout(0.2),
    layers.Dense(3, activation="softmax")
])

# 2. Early stopping — zastaví trénování když validace přestane klesat
early_stop = keras.callbacks.EarlyStopping(
    monitor="val_loss",
    patience=10,         # počkej 10 epoch bez zlepšení
    restore_best_weights=True
)

model.fit(X_train, y_train, epochs=200, callbacks=[early_stop], validation_split=0.2)

# 3. L2 regularizace — penalizace velkých vah
from tensorflow.keras import regularizers
layers.Dense(64, activation="relu", kernel_regularizer=regularizers.l2(0.01))
```

---

## Typy neuronových sítí

| Typ | Zkratka | Použití |
|---|---|---|
| Vícevrstvý perceptron | MLP | Tabulková data, základní klasifikace |
| Konvoluční sítě | CNN | Rozpoznávání obrazů |
| Rekurentní sítě | RNN/LSTM | Časové řady, text, sekvenční data |
| Transformery | Transformer | NLP, GPT, BERT, překlad textu |

---

## Shrnutí

- Neuron = vážená suma vstupů + bias → aktivační funkce
- Architektura: vstupní → skryté vrstvy → výstupní vrstva
- **ReLU** = skryté vrstvy; **Softmax** = multi-class výstup; **Sigmoid** = binární výstup
- Trénování: forward pass → loss → backpropagation → aktualizace vah
- **Overfitting** = memorování tréninku; řešení: Dropout, Early Stopping, regularizace
- Keras: `Sequential` → `compile` → `fit` → `evaluate`

---

## Typické doplňující otázky

### Co je backpropagation?
Algoritmus pro výpočet gradientů (derivací) loss funkce vůči všem vahám sítě. Používá chain rule (řetízkové pravidlo) a šíří chybu zpět od výstupní vrstvy k vstupní.

### Co je learning rate a proč je důležitý?
Learning rate (eta) určuje velikost kroku při aktualizaci vah. Příliš velký = přeskakujeme minimum, oscilace. Příliš malý = trénování trvá věky. Adam optimizer lr automaticky adaptuje.

### Co je batch size?
Počet vzorků zpracovaných najednou před aktualizací vah. Malý batch = časté aktualizace, více šumu. Velký batch = stabilnější gradient, ale více paměti.

### Proč ReLU místo sigmoid ve skrytých vrstvách?
Sigmoid trpí vanishing gradient problémem — gradienty se blíží nule pro velké/malé hodnoty, učení je pomalé. ReLU tento problém nemá (gradient je buď 0 nebo 1).
