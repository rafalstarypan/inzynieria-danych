# Etap 7 — podsumowanie zmian i plan prezentacji

## Podsumowanie zmian

### Co dostarczono (domyka PK 10)
- **5. algorytm obfuskacji**: `MBAObfuscator` (Mixed Boolean-Arithmetic)
- **Balansowanie zbioru ML**: 1:5 → 1:1 (105 czystych vs 105 obfuskowanych)
- **Wizualizacje ML**: 4 wykresy (top-15 cech, ROC, PR, macierz pomyłek)
- **Porównanie ML vs detektor progowy** na tym samym zbiorze testowym
- **Per-variant recall** (out-of-distribution: seed=99)
- **Persystencja modelu** (`save_ml_detector` / `load_ml_detector`)
- **Integracja MLDetector w UI** — zakładka *Detektor* z 3 trybami
- **Auto-instalacja zależności** (sklearn, joblib) w `cell-install`

### Nowe komórki notebooku (etap 7)

| ID komórki | Rodzaj | Zawartość |
|---|---|---|
| `cell-md-mba` | md | opis algorytmu MBA + cytat literatury |
| `cell-mba-impl` | code | `MBAObfuscator` + `_MBATransformer` |
| `cell-mba-demo` | code | demo + smoke test semantyczny (5/5) |
| `etap6-ml-impl` | code | (przepisany) `StaticFeatureExtractor` + `MLDetector` z `class_weight='balanced'` |
| `etap6-ml-train` | code | (nowy) zbalansowany dataset + 5-fold CV + raport |
| `etap6-md-plots` | md | opis wizualizacji ML |
| `etap6-plots-ml` | code | 4 wykresy: top-15 cech, ROC, PR, macierz |
| `etap6-md-compare` | md | opis porównania detektorów |
| `etap6-compare-detectors` | code | tabela + bar chart ML vs próg |
| `etap6-md-pervariant` | md | opis per-variant recall |
| `etap6-per-variant-recall` | code | tabela + bar chart per technika |
| `etap6-md-persistence` | md | opis zapisu modelu |
| `etap6-persistence` | code | `save_ml_detector` / `load_ml_detector` + smoke test |
| `etap7-intro` | md | wprowadzenie do etapu 7 |
| `etap7-display-ui` | code | świeży UI po treningu ML |
| `etap7-summary` | md | wnioski z etapu 7 |

### Zmienione istniejące komórki

| Komórka | Zmiana |
|---|---|
| `cell-install` | dodano auto-instalację `scikit-learn` i `joblib` |
| `etap4-schema` | dodano `'mba'` do `PARAM_SCHEMA`, `OBFUSCATOR_CLASSES`, `OBFUSCATOR_LABELS` |
| `etap4-validate` | domyślny `pipeline` zawiera teraz `mba` |
| `etap4-make-ui` | zakładka *Detektor* z trybami (progowy/ML/oba) + obsługa modelu w `on_run` |
| `etap5-benchmark-core` | `BENCHMARK_PIPELINES` ma 7 wariantów (dodano `'mba'`, zaktualizowano `'full'`) |

### Algorytm MBA — szczegóły

Operuje wyłącznie na `ast.Constant(int)`:
- Pomija `bool` (uwaga: `isinstance(True, int) == True` w Pythonie)
- Pomija wartości spoza `[min_value, max_value]`
- Z prawdopodobieństwem `apply_probability` zamienia na wyrażenie postaci
  `BinOp(Constant(a), {Add|Sub|BitXor}, Constant(b))`, gdzie `b`
  policzone z formuły zachowującej wartość

Pozycja w pipeline: `string_encrypt → mba → dead_code → cff → renaming`.

### Mierzalne wyniki

**Benchmark (7 wariantów × 25 próbek = 175 wierszy):**
- 100% kompilacji
- 100% poprawności semantycznej dla próbek z kontraktem
- MBA: czas obfuskacji 1.08 ms, podbija LOC z 8.8 → 8.88 (minimalny narzut)
- Pełny pipeline: 9.96 ms, depth 15.04, x4.4 LOC

**ML detektor:**
- 210 próbek (105 clean / 105 obf, ratio 1:1)
- 5-fold CV F1 weighted: **0.857 ± 0.034**
- Test set accuracy: **0.86**
- F1 clean: **0.85**, F1 obfuscated: **0.86**
- Top cechy: `identifier_entropy` (0.125), `total_entropy` (0.100),
  `line_entropy` (0.080), `num_chars` (0.079), `string_constant_ratio` (0.071)

**Porównanie ML vs progowy:**
| Metryka | Progowy | ML (RF) | Δ |
|---|---|---|---|
| accuracy | 0.619 | 0.857 | +0.238 |
| precision | 1.000 | 0.826 | -0.174 |
| recall | 0.238 | 0.905 | +0.667 |
| F1 | 0.385 | 0.864 | +0.479 |

Detektor progowy ma idealną precyzję, ale dramatycznie niski recall —
łapie tylko 23.8% obfuskacji. ML daje znacznie lepszy balans.

**Per-variant recall (OOD: seed=99):**
| Technika | Recall |
|---|---|
| renaming | 100.0% |
| dead_code | 100.0% |
| full | 100.0% |
| cff | 93.3% |
| mba | 83.3% |
| string_encrypt | **46.7%** |

`string_encrypt` jest znacząco trudniejszy do wykrycia — to ważne
spostrzeżenie, opisane w sekcji „Limitacje".

---

## Plan prezentacji

### Cel
Pokazać domknięcie projektu (PK 10): kompletny obfuskator z 5 algorytmami
i klasyfikator ML porównany z detektorem progowym, wszystko zintegrowane
w UI z persystencją.

### Czas: ~12 min

### Scenariusz

**1. Wstęp (1 min)**
- Przypomnienie: etap 5 — benchmark (compile + semantyka), etap 6 — ML
  detektor (StaticFeatureExtractor + RandomForest).
- Dziś etap 7: domykamy + dorzucamy 5. obfuskator.

**2. Algorytm 5 — MBA (2 min)**
- Pokaż komórkę `cell-md-mba` (definicja MBA z literatury).
- Cytat: „Xu i in. (2020) — sekcja 2.3 raportu literaturowego — MBA
  zastępuje wyrażenia arytmetyczne ich równoważnikami łączącymi
  operacje arytmetyczne i bitowe."
- Pokaż `cell-mba-demo`: literał `42` → `(518 + -476)`, `7` → `(-822 ^ -819)`.
- Komentarz: „Zachowuje semantykę — możemy łączyć z innymi technikami
  bez ryzyka wyniku innego niż oryginalny."

**3. Balansowanie zbioru (2 min)**
- Pokaż komórkę `etap6-ml-train`.
- Wyjaśnij: w etapie 6 mieliśmy 1:5, recall clean = 0.60.
- Teraz: 4 seedy generatora × 25 próbek + cykliczne obfuskowanie po
  jednej technice → 105 clean / 105 obf.
- Wynik: F1 obu klas ≥ 0.85, 5-fold CV F1 = 0.857 ± 0.034.

**4. Wizualizacje (2 min)**
- Pokaż komórkę `etap6-plots-ml` (4 wykresy):
  - Top-15 cech: `identifier_entropy` dominuje, ale 4 z top-12 to
    cechy strukturalne (AST counts, długości).
  - ROC + AUC: pokaż AUC, im wyżej tym lepiej.
  - PR curve: precision pozostaje wysoka aż do recall ~0.9.
  - Macierz pomyłek: 17 TN, 4 FP, 2 FN, 19 TP — dobry balans.

**5. Porównanie detektorów (2 min)**
- Pokaż komórkę `etap6-compare-detectors`.
- Najmocniejszy slajd: tabela ML vs progowy.
- Komentarz: „Próg ma idealną precyzję — kiedy uznaje że to obfuskacja,
  zawsze ma rację. Ale recall = 23.8% — łapie tylko co czwartą próbkę.
  ML osiąga recall 90.5% przy precyzji 82.6% — znacznie bardziej
  użyteczny detektor."

**6. Per-variant recall i ograniczenia (1.5 min)**
- Pokaż komórkę `etap6-per-variant-recall`.
- Główny insight: `string_encrypt` ma recall 46.7% — najtrudniejszy do
  wykrycia.
- Powód: zaszyfrowane stringi wyglądają jak normalne stringi.
  Cechy strukturalne AST są niewystarczające.
- Komentarz: „To naturalna motywacja do potencjalnego etapu 8 —
  cechy specyficzne dla zaszyfrowanych ciągów (entropia per-string,
  ratio printable chars)."

**7. UI z trybami detektora — live demo (2 min)**
- Pokaż komórkę `etap7-display-ui`.
- Zakładka *Detektor* — dropdown trybu (default: Oba).
- W zakładce *Dane* — Generuj próbki (seed=42), wybierz np. `gen_class_002`.
- W zakładce *Pipeline* — domyślnie wszystkie 5 obfuskatorów aktywne.
- Kliknij *Obfuskuj*.
- Pokaż wynik: tabela metryk + decyzje obu detektorów + zgodność.

**8. Wnioski (1 min)**
- 5 algorytmów obfuskacji + 2 detektory (progowy + ML) + benchmark + UI
  + persystencja → kompletny system.
- ML > progowy o ~24 pp accuracy, F1 weighted 0.86.
- `string_encrypt` jest najtrudniejszy do wykrycia — kierunek na etap 8
  (jeśli będzie).

---

## Slajdy

| # | Tytuł | Treść |
|---|---|---|
| 1 | Etap 7 — finalizacja + MBA | PK 10; co domykamy |
| 2 | Algorytm 5 — MBA | definicja, przykład, cytat z literatury |
| 3 | Balansowanie ML | 1:5 → 1:1, 4 seedy generatora |
| 4 | Wizualizacje ML | 4 panele: cechy / ROC / PR / macierz |
| 5 | ML vs Progowy | tabela porównawcza + bar chart |
| 6 | Per-variant recall | string_encrypt jako limitacja |
| 7 | UI z trybami detektora | zrzut ekranu zakładki Detektor |
| 8 | Wnioski + plan etap 8 | co osiągnięto + co dalej |

---

## Gotowe odpowiedzi

**„Dlaczego MBA, a nie inna technika z literatury?"**
MBA jest konkretnie wymieniona w naszym raporcie literaturowym (Xu i in.
2020, sekcja 2.3). Inne kandydatki — wirtualizacja, samomodyfikujący
się kod — są zbyt zaawansowane na zakres projektu. MBA jest
ortogonalna do 4 istniejących technik (operuje na **wartościach**,
nie nazwach / strukturze / stringach).

**„Dlaczego accuracy 0.86, a nie 0.95?"**
Nasz zbiór ma tylko 210 próbek. RF z 30 cechami przy 168 próbkach
treningowych zaczyna overfitować — 5-fold CV potwierdza F1 ≈ 0.86 ± 0.03.
Większy zbiór (np. 1000+ próbek z PyPI) prawdopodobnie podniósłby
wyniki, ale to zakres etapu 8.

**„Dlaczego `string_encrypt` jest taki słaby?"**
Szyfrowanie XOR z deszyfratorem produkuje stringi wyglądające jak
normalne literały tekstowe. Cechy `total_entropy`, `string_token_ratio`
zmieniają się minimalnie, bo deszyfrator dorzuca do kodu funkcję
i wywołania, ale nie modyfikuje znacząco statystyk AST. Lepsza detekcja
wymaga cech per-string (entropia każdego literału osobno).

**„Co znaczy `class_weight='balanced'`?"**
Random Forest waży klasy odwrotnie proporcjonalnie do liczności.
W naszym przypadku zbiór jest już 1:1, więc to safety net — gdyby
balansowanie się nie powiodło, RF dalej działałby rozsądnie.

**„Czy `joblib.dump` zachowuje wszystko, co potrzebne?"**
Tak: model RF, scaler, listę nazw cech i wersję sklearn. Przy
mismatch wersji `load_ml_detector` drukuje warning, ale nie
zatrzymuje wczytywania.

**„Co dalej (etap 8)?"**
- Real-world samples (kod ze stdlib / PyPI) — pokazałby FP rate
  na realnym kodzie
- Dodatkowe cechy specyficzne dla `string_encrypt`
- Adwersarialny obfuskator celowo unikający ML detektora
- Porównanie wielu modeli (RF/GB/LR) z grid-search hiperparametrów

---

## Kryteria demonstracyjne

Przed prezentacją uruchom notebook od `cell-install` do końca:

- [ ] Komórka instalacyjna kończy się „Wszystkie zależności załadowane"
- [ ] `cell-mba-demo` kończy się „Semantyka zachowana dla x ∈ {-5, 0, 5, 11, 100}"
- [ ] `etap5-run-benchmark` daje 7 wariantów × 25 próbek, błędy = 0
- [ ] `etap6-ml-train` kończy się raportem klasyfikacji (F1 ≥ 0.85)
- [ ] `etap6-plots-ml` rysuje 4 panele
- [ ] `etap6-compare-detectors` rysuje tabelę z `Δ accuracy ≥ +0.05`
- [ ] `etap6-per-variant-recall` rysuje tabelę i wykres
- [ ] `etap6-persistence` zapisuje plik `ml_detector.joblib`
- [ ] `etap7-display-ui` renderuje panel z 4 zakładkami
