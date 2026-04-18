# Opcja 3 — Etap 5: rozszerzenie detektora o uczenie maszynowe

## Cel
Uzupełnić warstwę detekcji o klasyfikator ML, zgodnie z literaturą z
`inz_wiedzy_raport_1.pdf` (Raitsis i in. 2025 — RF/GB/SVM, accuracy
97.3%; Wermke i in. 2016 — klasyfikator na cechach statycznych bytecode
z accuracy 99%). Obecny `EntropyDetector` ma tylko surowe progi — ML
daje lepszą separację i pozwala sensownie omówić *features*,
overfitting, walidację krzyżową.

Efekt końcowy: klasa `MLDetector`, macierz cech, wytrenowany Random
Forest, porównanie z detektorem progowym, wykres ważności cech.

## Zakres

### 1. Zbiór treningowy
Musimy mieć próbki obu klas:

- **Klasa 0 (czysty)** — 150 próbek: `TEST_SAMPLES` × 3 kopie +
  `CodeGenerator.generate_batch(n=135, seed=42)` + ewentualne
  próbki z `context/real_samples/`.
- **Klasa 1 (obfuskowany)** — te same 150 próbek przepuszczone przez
  różne konfiguracje pipeline'u:
  - 40× sam renaming
  - 30× sam dead code
  - 30× sam string encrypt
  - 25× sam CFF
  - 25× pełny pipeline

**Po obu stronach: 150 próbek.** Rozsądny balans.

**Podział:** stratified 70/15/15 (train/val/test). Seed stały, żeby
wyniki były powtarzalne.

### 2. Ekstrakcja cech

Klasa `FeatureExtractor` produkująca wektor `np.ndarray` długości ~20:

| # | Cecha | Źródło |
|---|---|---|
| 1 | `entropy_total` | `EntropyDetector` |
| 2 | `entropy_identifiers` | `EntropyDetector` |
| 3 | `entropy_lines_mean` | `EntropyDetector` |
| 4 | `entropy_lines_std` | policzone na linii |
| 5 | `loc` | `len(source.splitlines())` |
| 6 | `char_count` | `len(source)` |
| 7 | `avg_line_length` | — |
| 8 | `avg_identifier_length` | AST walk |
| 9 | `std_identifier_length` | — |
| 10 | `unique_token_ratio` | tokenize |
| 11 | `keyword_density` | liczba słów kluczowych / tokeny |
| 12 | `comment_density` | liczba komentarzy / LOC |
| 13 | `docstring_density` | liczba docstringów / funkcje |
| 14 | `ast_depth` | max głębokość drzewa |
| 15 | `ast_node_count` | `len(list(ast.walk(tree)))` |
| 16 | `cyclomatic_complexity` | własny visitor |
| 17 | `max_switch_size` | największe `if/elif` — wskaźnik CFF |
| 18 | `string_literal_count` | `ast.Constant(str)` |
| 19 | `mean_string_entropy` | średnia entropia stringów |
| 20 | `bytes_call_density` | wywołania `bytes(...)` / LOC — ślad XOR |

Uzasadnienie listy cech opiera się bezpośrednio na fragmentach z raportu
literatury (sekcja 3 i 5.2).

### 3. Klasa `MLDetector`

```python
class MLDetector:
    def __init__(self, model=None, extractor=None):
        self.model = model or RandomForestClassifier(
            n_estimators=200, max_depth=None, random_state=42)
        self.extractor = extractor or FeatureExtractor()
        self.feature_names: list[str] = []

    def fit(self, sources: list[str], labels: list[int]) -> dict:
        X = np.vstack([self.extractor.extract(s) for s in sources])
        self.model.fit(X, labels)
        return self._cv_report(X, labels)

    def predict(self, source: str) -> int: ...
    def predict_proba(self, source: str) -> float: ...
    def explain(self, source: str) -> dict: ...   # top-k cech + wartości
    def feature_importance(self) -> pd.DataFrame: ...
    def save(self, path): ...         # joblib
    def load(self, path): ...
```

### 4. Porównanie modeli
Wytrenować co najmniej trzy klasyfikatory i porównać:

- **Random Forest** (baseline z literatury).
- **Gradient Boosting** (`sklearn.ensemble.GradientBoostingClassifier`).
- **Logistic Regression** (po standaryzacji) — prosta baseline.

Dla każdego:
- 5-krotna walidacja krzyżowa na treningu: mean ± std z accuracy, F1.
- Wyniki na teście: accuracy, precision, recall, F1, ROC-AUC.
- Macierz pomyłek.

### 5. Wizualizacje
1. **Bar chart ważności cech** (Random Forest `feature_importances_`).
2. **Porównanie modeli** — słupki accuracy / F1 / ROC-AUC.
3. **ROC curves** — 3 klasyfikatory + baseline progowy na jednej osi.
4. **Macierz pomyłek** najlepszego modelu na teście.
5. **Scatter 2D** — redukcja cech do 2D (`PCA`) z kolorem = klasa, do
   pokazania separowalności.

### 6. Porównanie z detektorem progowym
Podsumowująca tabela:

| Detektor | Accuracy | F1 | ROC-AUC | Uwagi |
|---|---|---|---|---|
| próg entropii całkowitej | ? | ? | ? | baseline |
| próg entropii identyfikatorów | ? | ? | ? | — |
| Logistic Regression | ? | ? | ? | — |
| Random Forest | ? | ? | ? | — |
| Gradient Boosting | ? | ? | ? | — |

Celujemy w pokazanie, że ML > progowy o kilkanaście pp.

### 7. Struktura komórek notatnika

| # | Rodzaj | Zawartość |
|---|---|---|
| md | Etap 5 — Detektor ML | motywacja + odniesienia do literatury |
| code | `FeatureExtractor` | |
| code | generacja zbioru `(sources, labels)` | |
| code | podział train/val/test ze seedem | |
| code | `MLDetector` | |
| code | trening + CV | |
| code | ewaluacja na teście | |
| code | porównanie z detektorem progowym | |
| code | wykresy (ważność cech, ROC, macierz) | |
| code | `save()` / `load()` wyuczonego modelu do `context/ml_detector.joblib` | |
| md | dyskusja, limitacje, plan dalszych prac | |

### 8. Odporność i uczciwość eksperymentu
- **Brak wycieku**: `IdentifierRenamer(seed=...)` i `CodeGenerator(seed=...)`
  muszą używać różnych seedów dla train i test, żeby nie trenować i
  testować na tym samym losowym rdzeniu.
- **Stratyfikacja per typ obfuskacji**: test set powinien zawierać
  wszystkie 5 wariantów (renaming, deadcode, stringenc, cff, full) —
  żebyśmy widzieli per-variant recall, nie tylko średnią.
- **Baseline**: próg entropii trenujemy na tym samym train set
  (wybieramy próg maksymalizujący accuracy), ewaluujemy na tym samym
  test set. Tylko wtedy porównanie jest fair.

## Kryteria ukończenia
1. `FeatureExtractor.extract(source)` zwraca wektor długości 20 dla
   dowolnego kompilującego się źródła.
2. Random Forest osiąga accuracy ≥ 0.90 na test set.
3. Przynajmniej jeden model ML przewyższa baseline progowy o ≥ 5 pp
   accuracy.
4. Ważność cech (top 5) ma sensowną interpretację, opisana w markdown.
5. Model można zapisać/wczytać z dysku (`joblib`).
6. Eksperyment jest w pełni powtarzalny (`seed=42` wszędzie).

## Ryzyka
- **Zbyt łatwy task** — jeśli po pełnym pipeline'ie accuracy = 1.0,
  to znaczy, że wybrane cechy trywialnie separują. Trzeba dodać do
  klasy 1 próbki „lekko obfuskowane” (np. samo renaming z
  `name_length=4`), żeby zadanie miało sens. To też jest omawiane w
  literaturze (Raitsis i in.).
- **Małe N (300 próbek)** — ryzyko overfittingu. Stąd 5-CV i raportowanie
  std, nie tylko mean.
- **Reprodukowalność** — `sklearn` bez `random_state` losuje inaczej
  za każdym razem; wszędzie ustawiamy seed.

## Oszacowanie czasu
- `FeatureExtractor` (20 cech): ~2 h
- Pipeline generacji zbioru: ~1 h
- `MLDetector` + trening + CV: ~1.5 h
- Porównania i wizualizacje: ~2 h
- Opis w notatniku z odniesieniami do literatury: ~1 h

**Razem: ~7–8 h pracy.**

## Co pokazuję na zajęciach
- Jeden slajd: „detektor progowy dawał X%, RF daje Y%”.
- Wykres ważności cech — które sygnały obfuskacji są najmocniejsze.
- Live demo: wklejamy obfuskowany kod → `MLDetector.predict_proba()`
  zwraca np. 0.97.
- Wklejamy ten sam kod przed obfuskacją → ~0.05. Kontrast czytelny.
