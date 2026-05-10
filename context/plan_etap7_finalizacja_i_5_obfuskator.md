# Plan etapu 7 — finalizacja etapu 6 + 5. obfuskator (MBA)

## Cel

Zamknąć wszystkie luki pozostawione w etapie 6 (PK 10) i dorzucić piąty
algorytm obfuskacji udokumentowany w `inz_wiedzy_raport_1.pdf`
(sekcja 2.3 — Mixed Boolean-Arithmetic).

**Rozmiar:** ~8 h (sesja A: ~5 h, sesja B: ~3 h).

## Decyzje przed startem (zatwierdzone)

- 5. algorytm: **MBA (Mixed Boolean-Arithmetic)**
- MBA **domyślnie włączony** w pipeline i w `BENCHMARK_PIPELINES['full']`
- UI z trybem detektora: `progowy / ML / oba` — **oba** jako default
- Cel ilościowy: **F1 ≥ 0.85** dla obu klas, **accuracy ≥ 0.90**

## Część A — finalizacja etapu 6

### A1. Balansowanie klas + retrening (45 min)

**Problem:** dataset etapu 6 ma 25 czystych vs 125 obfuskowanych (1:5).
Skutek: recall klasy `clean` = 0.60, F1 weighted = 0.84.

**Rozwiązanie:**
1. Dodatkowe próbki czyste z 4 seedów generatora (`42, 99, 7, 123`),
   po 25 sztuk → łącznie ~105 czystych (5 z `TEST_SAMPLES` + 100 z
   generatora).
2. Strategia obfuskacji: każdą próbkę obfuskujemy **jedną** techniką
   wybraną cyklicznie (mod liczby technik). Daje ~105 obf z równym
   rozkładem 6 technik (`renaming`, `dead_code`, `string_encrypt`, `cff`,
   `mba`, `full`).
3. `class_weight='balanced'` w `RandomForestClassifier` jako safety net.

**Cel:** F1 obu klas ≥ 0.85, accuracy ≥ 0.90.

### A2. Wizualizacje ML (60 min)

Nowa komórka `etap6-plots-ml` rysuje 4 wykresy w `subplots(2, 2)`:

- **Top-15 cech** wg `feature_importances_` (horizontal bar)
- **ROC curve** dla RF na test set (z AUC w legendzie)
- **PR curve** (Precision-Recall)
- **Heatmap macierzy pomyłek** (z annotacjami)

### A3. Porównanie ML vs detektor progowy (45 min)

Nowa komórka `etap6-compare-detectors`:

- Funkcja `compare_detectors(threshold_det, ml_det, X_test, y_test) -> dict`
- Liczy: `accuracy / precision / recall / F1` dla obu detektorów
- Tabela tekstowa + bar chart side-by-side per metryka
- Cel: pokazać, że **ML > próg o ≥ 5 pp accuracy**

### A4. Per-variant recall (30 min)

Nowa komórka `etap6-per-variant-recall`:

- Generuje świeże próbki obfuskowane (seed=99 — out-of-distribution względem
  treningu na seed=42)
- Liczy recall MLDetectora dla każdej z 6 technik osobno
- Tabela posortowana malejąco
- Pokazuje, które techniki najbardziej mylą detektor

### A5. Persystencja modelu (30 min)

Nowa komórka `etap6-persistence`:

- `save_ml_detector(detector, path)` — `joblib.dump({'model', 'scaler',
  'features', 'sklearn_version'})`
- `load_ml_detector(path)` — z warning'iem przy mismatch wersji `sklearn`
- Round-trip smoke test: zapis → wczyt → predykcja zgodna na 2 próbkach
  (czysta + obfuskowana)

### A6. Integracja MLDetector w UI (90 min)

Modyfikacja `etap4-make-ui` (zakładka *Detektor* + handler `on_run`):

**Zakładka Detektor — nowe widżety:**

- `Dropdown` „Tryb detektora": `Progowy / ML / Oba` (default: `Oba`)
- `Text` „Plik modelu ML" + button *Załaduj model*
- `HTML` ze statusem modelu

**Handler `on_run`:**

- Zawsze liczy entropie (przed/po) i pokazuje tabelę
- Dla trybu `threshold` lub `both`: pokazuje decyzję progową
- Dla trybu `ml` lub `both`: pokazuje decyzję ML + `predict_proba`
- Dla `both`: dodatkowo linia „Zgodność detektorów: TAK/NIE"
- Jeśli `ml_detector` nie jest dostępny w globalsach i nie został
  załadowany przez UI — komunikat „uruchom komórkę etap6 albo załaduj model"

### A7. Dokumenty (45 min)

- `context/etap6_podsumowanie_i_prezentacja.md` (retrofit)
- `context/etap7_podsumowanie_i_prezentacja.md` (po implementacji)
- `context/etap7_przygotowanie_do_prezentacji.md` (krótki, na zajęcia)

## Część B — nowy obfuskator: `MBAObfuscator`

### B1. Algorytm

Dla każdego `ast.Constant(int)` w drzewie:

1. Pomiń `bool` (uwaga: `isinstance(True, int) == True`)
2. Pomiń wartości spoza `[min_value, max_value]`
3. Z prawdopodobieństwem `apply_probability` zamień na wyrażenie MBA

Generowanie wyrażenia dla wartości `v`:

- Wybierz losowy operator z `{+, -, ^}`
- Wylosuj `a ∈ [-1000, 1000]`
- Policz `b` tak, by `eval(a OP b) == v`:
  - `+`: `b = v - a` → `BinOp(a, Add, b)`
  - `-`: `b = a - v` → `BinOp(a, Sub, b)`
  - `^`: `b = v ^ a` → `BinOp(a, BitXor, b)`
- Dla `expression_depth > 1` rekurencyjnie rozwiń `a` lub `b` (50% szansa
  każde)

**Inwariant bezpieczeństwa:** wartość po `eval()` zawsze == oryginał,
niezależnie od głębokości.

### B2. Klasa + parametry

```python
class MBAObfuscator(BaseObfuscator):
    """Mixed Boolean-Arithmetic — literały int → wyrażenia."""
    def __init__(self, seed=None, apply_probability=0.7,
                 min_value=-1000, max_value=1000, expression_depth=1):
        ...
    def transform(self, source: str) -> str:
        ...
```

`PARAM_SCHEMA['mba']`:

| Parametr | Typ | Zakres | Default | Etykieta |
|---|---|---|---|---|
| `seed` | `int?` | — | `None` | Seed |
| `apply_probability` | `float` | 0.0–1.0 | 0.7 | Prawdopodobieństwo zamiany |
| `min_value` | `int` | -100000–0 | -1000 | Min wartość |
| `max_value` | `int` | 1–100000 | 1000 | Max wartość |
| `expression_depth` | `int` | 1–3 | 1 | Głębokość wyrażenia |

`OBFUSCATOR_CLASSES['mba'] = MBAObfuscator`
`OBFUSCATOR_LABELS['mba'] = 'MBA (literały)'`

### B3. Pozycja w pipeline

Domyślny `pipeline`:
```
['string_encrypt', 'mba', 'dead_code', 'cff', 'renaming']
```

Uzasadnienie kolejności:

- MBA musi działać na **oryginalnych** literałach (zanim dead_code
  i CFF dorzucą swoje stałe)
- `renaming` zostaje na końcu (jak dotąd) — żeby objąć też nazwy z CFF
- `string_encrypt` przed MBA — żeby MBA działał na literałach z funkcji
  deszyfrującej

### B4. Komórki

| ID | Pozycja | Rodzaj | Zawartość |
|---|---|---|---|
| `cell-md-mba` | po `cell-cff-demo` | md | opis algorytmu + cytat z literatury |
| `cell-mba-impl` | po `cell-md-mba` | code | `MBAObfuscator` + `_MBATransformer` |
| `cell-mba-demo` | po `cell-mba-impl` | code | demo + smoke test semantyczny |

Numeracja w `cell-md-pipeline` (sekcja "## 7. Pipeline") nie zmienia się
— MBA wskakuje jako algorytm 5 (`## 7. Algorytm 5 — MBA`).

### B5. Aktualizacja istniejących komórek

| Komórka | Zmiana |
|---|---|
| `etap4-schema` | dodaj `'mba'` do `PARAM_SCHEMA`, `OBFUSCATOR_CLASSES`, `OBFUSCATOR_LABELS` |
| `etap4-validate` | zmień default `pipeline` na 5-elementowy (z `mba`) |
| `etap5-benchmark-core` | dodaj `'mba': ['mba']` do `BENCHMARK_PIPELINES`, zaktualizuj `'full'` |
| `76a771c7` (etap6) | balansowanie + `class_weight='balanced'` + auto-uwzględnienie `mba` |

## Kryteria ukończenia

1. Wszystkie istniejące testy przechodzą (semantyka `bubble_sort` 5/5 dla
   pełnego pipeline z MBA).
2. `MBAObfuscator(seed=42).transform(src)` daje powtarzalny wynik.
3. `eval()` literałów MBA == oryginalne wartości (smoke test).
4. ML detector: `accuracy ≥ 0.90`, `F1 obu klas ≥ 0.85`.
5. Save → load modelu zachowuje predykcje (test na 2 próbkach).
6. Per-variant recall — żaden wariant nie ma `recall < 0.50`.
7. UI: zakładka *Detektor* wyświetla 3 tryby; tryb `Oba` pokazuje porównanie.
8. Komórka `etap5-run-benchmark` kończy się tabelką dla 7 wariantów
   (`none, renaming, dead_code, string_encrypt, cff, mba, full`).

## Ryzyka

| Ryzyko | Prawdopodobieństwo | Mitygacja |
|---|---|---|
| MBA na boolach łamie semantykę (`isinstance(True, int) == True`) | wysokie | jawne `isinstance(value, bool)` skip |
| `expression_depth=3` produkuje gigantyczny kod | średnie | clamp `max=3` w schemie + ostrzeżenie w demo |
| Class imbalance fix (upsample) wycieka do test set | średnie | upsample przed split, `train_test_split` ze stratyfikacją |
| `joblib.dump` z innej wersji `sklearn` rzuca przy load | niskie | warning, nie error |
| MBA + CFF razem produkują nielegalny AST | niskie | smoke test: `_verify` po pełnym pipeline |
| `ast.unparse(Constant(value=-3))` produkuje niejednoznaczny string | niskie | używamy tylko nieujemnych w `Constant`, ujemne przez `UnaryOp(USub)` |

## Plan dzienny

**Sesja 1 (~5 h) — etap 6 finalizacja:**

- A1 balansowanie + retrening (45 min)
- A2 wizualizacje (60 min)
- A3 porównanie detektorów (45 min)
- A4 per-variant recall (30 min)
- A5 persystencja (30 min)
- A6 UI integracja (90 min)
- bufor (30 min)

**Sesja 2 (~3 h) — MBA + dokumenty:**

- B2/B3 MBAObfuscator + transformer (60 min)
- B4 wpięcie w schema/builder/pipeline (30 min)
- B5 demo + smoke test (30 min)
- A7 dokumenty (45 min)
- bufor (15 min)
