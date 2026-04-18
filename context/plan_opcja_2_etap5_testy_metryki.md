# Opcja 2 — Etap 5: testy i metryki na rzeczywistych danych

## Cel
Dostarczyć ilościowe dowody, że obfuskatory działają poprawnie (zachowują
semantykę) i skutecznie (utrudniają analizę), a detektor je rozpoznaje.
Odpowiada PK 7 z listy („Testowanie aplikacji na bazie danych pochodzących
[z generatora oraz z zasobów]”).

Efekt końcowy: zestaw automatycznych testów + tabele i wykresy w
notatniku, gotowe do wklejenia do sprawozdania.

## Zakres

### 1. Zbiór danych testowych
Trzy źródła:
1. **`TEST_SAMPLES`** — 5 przykładów ręcznych (już w notatniku).
2. **`CodeGenerator.generate_batch(n=50, seed=42)`** — batch
   powtarzalny.
3. **Zewnętrzny zbiór „realnych” fragmentów Pythona** — np. 20 krótkich
   funkcji z dowolnego open-source'owego projektu (np. `cpython/Lib`),
   trzymanych w katalogu `context/real_samples/*.py`. Kryterium doboru:
   jedna funkcja/klasa, < 80 linii, brak importów z `typing_extensions`,
   kompiluje się standalone.

### 2. Metryki

**Poprawność (zachowanie semantyki)**
- *Testy różnicowe*: dla funkcji o ustalonym kontrakcie (`bubble_sort`,
  `binary_search`, `caesar_encrypt`) uruchamiamy wejścia losowe i
  porównujemy wyniki `f(x) == f_obf(x)`. Metryka: odsetek zgodnych.
- *Kompilacja*: `compile(source, '<str>', 'exec')` — odsetek próbek,
  które skompilowały się po każdym kroku pipeline'u.

**Utrudnienie analizy**
- Rozmiar kodu: LOC, liczba znaków, liczba węzłów AST.
- Złożoność: głębokość AST, złożoność cyklomatyczna
  (`ast` + własny visitor liczący `If`/`For`/`While`/`And`/`Or`).
- Entropia całkowita, entropia identyfikatorów, entropia linii (już są
  w `EntropyDetector`).
- Stosunek unikalnych tokenów do wszystkich.

**Wydajność**
- Czas obfuskacji per technika (`time.perf_counter`).
- Czas detekcji.

**Detekcja**
- Klasyfikacja kodu na bazie progów entropii — macierz pomyłek
  (TP/TN/FP/FN), accuracy, precision, recall, F1.
- ROC: zmienne progi entropii całkowitej → krzywa.

### 3. Moduł `Benchmark`
Nowa klasa w notatniku:
```python
@dataclass
class BenchResult:
    sample: str
    technique: str           # "none", "renaming", "deadcode", ... "full"
    loc: int
    chars: int
    ast_nodes: int
    ast_depth: int
    cyclomatic: int
    entropy_total: float
    entropy_ident: float
    entropy_line: float
    unique_token_ratio: float
    obf_time_ms: float
    detect_time_ms: float
    compiles: bool
    semantics_ok: Optional[bool]   # None jeśli brak kontraktu

class Benchmark:
    def run(self, samples: dict, pipelines: dict) -> pd.DataFrame: ...
    def differential_test(self, original, obfuscated, cases) -> bool: ...
```
`pipelines` to dict np. `{"renaming": Pipeline([Renamer()]),
"full": Pipeline([StringEnc, DeadCode, CFF, Renamer])}`.

### 4. Wizualizacje (matplotlib)

1. **Bar chart**: średnia entropia całkowita per technika (grupowane
   słupki: oryginał vs obfuskacja).
2. **Box plot**: rozkład entropii dla wygenerowanych 50 próbek — każda
   technika osobno.
3. **Scatter**: LOC przed vs po (każdy punkt = próbka, kolor = technika).
   Pokazuje koszt rozmiarowy obfuskacji.
4. **Heatmapa**: macierz pomyłek detektora na progu optymalnym.
5. **Krzywa ROC**: entropia całkowita jako cecha binarna.
6. **Tabela zbiorcza**: average po technikach — entropia, LOC, czas,
   accuracy detekcji.

### 5. Kontrakty testowe (`differential_test`)
Dla próbek z `TEST_SAMPLES` zdefiniować wejścia:

| Funkcja | Przypadki testowe |
|---|---|
| `bubble_sort` | `[]`, `[1]`, `[3,1,2]`, `[5,4,3,2,1]`, losowe 20× |
| `binary_search` | 10× różne tablice + target obecny/nieobecny |
| `caesar_encrypt` | kilka kombinacji (tekst, shift) |
| `Stack` | sekwencje `push`/`pop`/`peek`/`is_empty` |
| `count_words` | 5 różnych tekstów |

Uruchomienie: `exec(obfuscated_code, namespace)` → wywołanie funkcji
z tego `namespace` → porównanie z oryginałem.

### 6. Struktura komórek notatnika

| # | Rodzaj | Zawartość |
|---|---|---|
| md | Etap 5 — Testy i metryki | wprowadzenie, cele |
| code | klasa `Benchmark` + `BenchResult` | |
| code | visitor złożoności cyklomatycznej | |
| code | definicje `CONTRACTS` (wejścia/oczekiwane wyniki) | |
| code | `differential_test` | |
| code | załadowanie `real_samples/*.py` | |
| code | uruchomienie benchmarku → `df` | |
| md | „Wyniki — poprawność” | |
| code | tabela: odsetek `compiles`, `semantics_ok` per technika | |
| md | „Wyniki — utrudnienie analizy” | |
| code | wykres 1 (bar entropia) | |
| code | wykres 2 (box plot) | |
| code | wykres 3 (scatter LOC) | |
| md | „Wyniki — detekcja” | |
| code | macierz pomyłek + ROC | |
| md | „Dyskusja i wnioski” | |

### 7. Eksport wyników
- `df.to_csv("context/benchmark_results.csv")` — do sprawozdania.
- Wykresy zapisywane do `context/figures/*.png` (`plt.savefig`).

## Kryteria ukończenia
1. Benchmark przechodzi na min. 50 próbkach z generatora + 5 ręcznych +
   20 rzeczywistych.
2. `compiles == True` dla 100% próbek (brak regresji w obfuskatorach).
3. `semantics_ok == True` dla 100% próbek z kontraktem.
4. Entropia całkowita po pełnym pipeline'u jest **wyższa** niż przed
   dla ≥ 95% próbek (jeśli nie — raportujemy i dyskutujemy dlaczego).
5. Detektor na optymalnym progu: accuracy ≥ 0.9 na zbiorze
   oryginał/obfuskacja.
6. Wszystkie wykresy wyrenderowane i zapisane do `context/figures/`.

## Ryzyka
- **Różnice semantyczne w stringach po `StringEncryptor`** — XOR musi
  wrócić do tego samego bajta; f-stringów i konkatenacji dotykać ostrożnie.
- **`ControlFlowFlattener`** może zepsuć funkcje z `yield` lub wczesnymi
  `return` w pętlach — jeśli wychodzi, wyłączamy CFF dla tych próbek
  i raportujemy jako znane ograniczenie (nie ukrywamy).
- **Selekcja „realnych” próbek**: jeśli zawierają `import` modułów
  niedostępnych w sandboxie, wywalą się na `exec`. Trzeba je
  przefiltrować przed włączeniem do benchmarku.

## Oszacowanie czasu
- `Benchmark` + visitor cyklomatyki: ~1.5 h
- Kontrakty i `differential_test`: ~1 h
- Zbiór `real_samples`: ~1 h (selekcja + sanity check)
- Wizualizacje: ~1.5 h
- Opis wyników w notatniku: ~1 h

**Razem: ~5–6 h pracy.**

## Co pokazuję na zajęciach
- Uruchomienie całego benchmarku (może w tle, przy starcie prezentacji).
- Tabela zbiorcza: „po włączeniu CFF entropia rośnie o X%, rozmiar o Y%,
  detektor łapie Z% przypadków”.
- Krzywa ROC + wybór progu.
- Jeden wykres-podsumowanie pasujący na slajd.
