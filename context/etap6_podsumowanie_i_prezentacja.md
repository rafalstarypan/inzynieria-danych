# Etap 6 — podsumowanie zmian i plan prezentacji

> Dokument retrofit: etap 6 nie miał własnego podsumowania w czasie
> implementacji; ten plik został spisany w trakcie etapu 7 na podstawie
> aktualnego stanu notatnika.

## Podsumowanie zmian

### Co dostarczono
- `StaticFeatureExtractor` — ekstraktor 30 cech statycznych kodu Pythona
- `MLDetector` — klasyfikator `RandomForestClassifier` z `StandardScaler`
- Automatyczne budowanie zbioru treningowego z `TEST_SAMPLES` + `CodeGenerator`
- Trening + raport klasyfikacji + macierz pomyłek + ważność cech

### Cechy ekstrahowane (30 sztuk)
**Strukturalne (AST):**
- `num_ast_nodes`, `num_functions`, `num_classes`, `num_returns`
- `num_ifs`, `num_fors`, `num_whiles`, `num_assigns`, `num_calls`
- `num_binops`, `num_unaryops`, `num_comparisons`
- `num_constants`, `num_string_constants`, `string_constant_ratio`
- `ast_depth`, `num_names`, `num_unique_names`, `unique_name_ratio`,
  `avg_name_length`

**Tekstowe / leksykalne:**
- `num_chars`, `num_lines`, `nonempty_lines`, `avg_line_len`
- `blank_line_ratio`, `non_alnum_ratio`
- `total_entropy`, `line_entropy`, `identifier_entropy`
- `comment_ratio`, `string_token_ratio`

### Komórki dostarczone w etapie 6 (po refaktoryzacji w etapie 7)

| ID | Rodzaj | Zawartość |
|---|---|---|
| `6cea42f3` | md | Etap 6 — opis: ekstrakcja cech, klasyfikator ML, integracja |
| `etap6-ml-impl` | code | `StaticFeatureExtractor` + `MLDetector` |
| `etap6-ml-train` | code | dataset zbalansowany + trening + 5-fold CV (etap 7) |

> Komórki finalizacji (`etap6-plots-ml`, `etap6-compare-detectors`,
> `etap6-per-variant-recall`, `etap6-persistence`) zostały dodane
> w etapie 7. Patrz `etap7_podsumowanie_i_prezentacja.md`.

### Pierwotne wyniki (przed balansowaniem z etapu 7)
- 25 czystych vs 125 obfuskowanych (1:5)
- accuracy ≈ 0.83
- F1 clean ≈ 0.55, F1 obfuscated ≈ 0.90
- najsilniejsze cechy: `total_entropy`, `num_chars`, `non_alnum_ratio`,
  `identifier_entropy`

### Co zostało zaczęte ale nieukończone (zamknięte w etapie 7)
1. UI integration — markdown obiecywał zakładkę *Detektor* z trybami,
   ale komórka implementacyjna była pusta
2. Wizualizacje ML
3. Persystencja modelu
4. Porównanie ML vs detektor progowy na tym samym zbiorze testowym
5. Per-variant recall

---

## Plan prezentacji

### Cel prezentacji
Pokazać przejście od detektora progowego (etap 5) do klasyfikatora
opartego na cechach statycznych (etap 6). Postawić tezę, że cechy
strukturalne kodu pozwalają na lepszą detekcję niż sama entropia.

### Czas: ~8 min

### Scenariusz

**1. Motywacja (1 min)**
- W etapie 5 mieliśmy detektor progowy. Pokazał, że entropia rośnie po
  obfuskacji, ale nie zawsze przekracza próg.
- Pytanie: czy można lepiej? Tak — dorzucamy więcej cech i klasyfikator.

**2. Ekstraktor cech (2 min)**
- Pokaż komórkę `etap6-ml-impl` (`StaticFeatureExtractor`).
- Wymień 5 grup cech: AST counts, długości / proporcje, entropie,
  identyfikatory, tokeny tekstowe.
- Komentarz: „każda kategoria łapie inną sygnaturę obfuskacji — entropia
  łapie szyfrowanie stringów, AST counts łapią dead code i CFF, długości
  identyfikatorów łapią renaming".

**3. MLDetector (2 min)**
- Pokaż klasę `MLDetector` z `RandomForestClassifier`.
- Wyjaśnij: RF dlatego, że nie wymaga założeń o liniowości i dobrze
  działa na cechach o różnych skalach.
- Pokaż automatyczne budowanie zbioru: 25 próbek × 5 wariantów obfuskacji
  + 25 czystych = 150.
- (Jeśli dyskutujesz wyniki: dodaj komentarz o etapie 7, gdzie
  przebalansowaliśmy do 1:1.)

**4. Wyniki (2 min)**
- Pokaż komórkę `etap6-ml-train` z `classification_report`.
- Najważniejsze cechy: `identifier_entropy`, `total_entropy`,
  `line_entropy`, `num_chars`, `string_constant_ratio`.
- Komentarz: „entropie dominują, ale 4/12 najlepszych cech to czysto
  strukturalne metryki AST — uzasadnia mieszane podejście."

**5. Co dalej (1 min)**
- Etap 7 dorzuca: balansowanie klas, wizualizacje, persystencję
  i integrację UI.
- Etap 7 dorzuca też **5. obfuskator MBA** (Mixed Boolean-Arithmetic).

---

## Slajdy (jeśli potrzebne)

| # | Tytuł | Treść |
|---|---|---|
| 1 | Etap 6 — Detektor ML | motywacja: entropia to mało |
| 2 | Cechy statyczne | 5 grup × przykłady |
| 3 | Architektura | source → AST + tokenize → 30 cech → RF |
| 4 | Wyniki | classification_report + top cech |
| 5 | Plan etapu 7 | balansowanie + UI + persystencja + MBA |

---

## Gotowe odpowiedzi

**„Dlaczego Random Forest, a nie sieć neuronowa?"**
Zbiór jest mały (~150 próbek). RF radzi sobie z małymi zbiorami i nie
wymaga normalizacji (mimo że i tak używamy `StandardScaler`). Sieć
neuronowa wymaga więcej danych i nie daje natywnie ważności cech.

**„Czy te cechy nie są skorelowane?"**
Tak, częściowo. Np. `num_chars` i `num_lines` są ze sobą skorelowane.
RF radzi sobie z korelacją cech (sample/feature subsampling), ale przy
modelach liniowych byłby problem. Świadomie zostawiamy redundantne
cechy, bo RF używa ich jako alternatywnych ścieżek decyzyjnych.

**„Co oznacza, że `identifier_entropy` jest najważniejszą cechą?"**
Renaming generuje losowe nazwy (np. `v_aabhxojw`), które mają wysoką
entropię. Inne techniki (dead_code, CFF) tego nie robią. Detektor ML
wykorzystuje tę cechę jako jeden z najsilniejszych sygnałów.

**„Dlaczego potem trenujecie jeszcze raz w etapie 7?"**
Pierwsze trenowanie miało imbalance 1:5, co psuło recall klasy `clean`.
W etapie 7 dorzucono 4× więcej próbek czystych z generatora i ustawiono
`class_weight='balanced'`.
