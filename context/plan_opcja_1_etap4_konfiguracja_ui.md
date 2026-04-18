# Opcja 1 — Dokończenie etapu 4: pełna konfiguracja + UI

## Cel
Zamknąć PK 9 z listy punktów kontrolnych: „Uzupełnienie interfejsu użytkownika
o możliwość ustawiania, zapisywania i odczytywania parametrów algorytmów
oraz parametrów generatorów danych na dysku”.

Efekt końcowy: w notatniku jeden panel `ipywidgets`, w którym można wybrać
dowolną kombinację obfuskatorów z ich parametrami, uruchomić pipeline na
wpisanym kodzie, zobaczyć wynik + metryki detektora, a całą konfigurację
zapisać/wczytać z pliku JSON.

## Stan wyjściowy
Obecny `# Etap 4` w [obfuskacja_projekt_etap4.ipynb](../obfuskacja_projekt_etap4.ipynb)
ma tylko szkielet:
- `config` obsługuje wyłącznie `renaming` / `deadcode`,
- brak parametrów dla `StringEncryptor`, `ControlFlowFlattener`, pipeline,
  generatora, detektora,
- brak UI (na diagramie architektury w Cell 30 UI jest zapowiedziane,
  ale nie zaimplementowane).

## Zakres zmian

### 1. Rozszerzenie konstruktorów obfuskatorów o parametry
Dziś klasy tworzone są bez argumentów — parametry są „zaszyte” w kodzie.
Trzeba dodać konfigurowalne pola:

| Klasa | Parametr | Domyślna wartość | Znaczenie |
|---|---|---|---|
| `IdentifierRenamer` | `name_length` | 8 | długość generowanych nazw |
| `IdentifierRenamer` | `alphabet` | `"abcdefghijklmnop..."` | alfabet nazw (np. losowy / leksykalny / homoglify) |
| `IdentifierRenamer` | `seed` | `None` | determinizm |
| `DeadCodeInserter` | `insertion_probability` | 0.3 | szansa wstawienia dla danego węzła |
| `DeadCodeInserter` | `max_insertions` | 5 | limit wstawień na funkcję |
| `DeadCodeInserter` | `templates` | lista stringów | wybór rodzajów martwego kodu |
| `DeadCodeInserter` | `seed` | `None` | — |
| `StringEncryptor` | `key_length` | równa długości stringa | długość klucza XOR |
| `StringEncryptor` | `skip_docstrings` | `True` | zachowanie docstringów |
| `StringEncryptor` | `min_string_length` | 1 | nie szyfruj krótszych |
| `StringEncryptor` | `seed` | `None` | — |
| `ControlFlowFlattener` | `shuffle_blocks` | `True` | losowa kolejność `elif` |
| `ControlFlowFlattener` | `state_var_name` | `"_state"` | — |
| `ControlFlowFlattener` | `apply_to_classes` | `True` | czy wchodzić w metody klas |
| `ControlFlowFlattener` | `seed` | `None` | — |

Zmiana **dotyka istniejącego kodu w komórkach etapu 2 i 3** — trzeba
zaktualizować sygnatury `__init__` oraz ciała `_transform`/`visit_*`,
ale bez zmiany zewnętrznego zachowania dla wartości domyślnych.

### 2. Nowa, pełna struktura `config`
```json
{
  "pipeline": ["string_encrypt", "dead_code", "cff", "renaming"],
  "obfuscators": {
    "renaming":      {"name_length": 8, "alphabet": "lower", "seed": null},
    "dead_code":     {"insertion_probability": 0.3, "max_insertions": 5,
                      "templates": ["unused_var", "false_if", "empty_loop"],
                      "seed": null},
    "string_encrypt":{"key_length": null, "skip_docstrings": true,
                      "min_string_length": 1, "seed": null},
    "cff":           {"shuffle_blocks": true, "state_var_name": "_state",
                      "apply_to_classes": true, "seed": null}
  },
  "generator": {
    "num_samples": 10,
    "types": ["arithmetic", "loop", "string", "class"],
    "seed": 42
  },
  "detector": {
    "thresholds": {"total_entropy": 4.8, "identifier_entropy": 4.2}
  }
}
```

### 3. Funkcje fabrykujące i walidacja
- `build_obfuscator(name: str, params: dict) -> BaseObfuscator` — mapuje
  nazwę na klasę (`"renaming"` → `IdentifierRenamer(**params)`, itd.),
- `build_pipeline(config: dict) -> ObfuscationPipeline` — składa pipeline
  w kolejności z `config["pipeline"]`,
- `validate_config(config: dict) -> list[str]` — zwraca listę błędów
  (nieznany obfuskator, zły typ pola, brak sekcji). Wywoływane przed
  każdym `run` i przed `save_config`.

### 4. UI `ipywidgets`
Panel złożony z kilku sekcji (`VBox` + `Accordion`):

**Sekcja A — wybór i konfiguracja obfuskatorów**
- `SelectMultiple` z listą obfuskatorów, z zachowaniem kolejności
  (przyciski „↑ / ↓” do zmiany),
- dla każdego wybranego obfuskatora zestaw widgetów wygenerowanych
  automatycznie z deklaracji parametrów (`IntSlider`, `Checkbox`,
  `Dropdown`, `Text`).

**Sekcja B — generator danych testowych**
- `IntSlider` num_samples (1–100),
- `SelectMultiple` typów,
- `IntText` seed,
- przycisk „Generuj” → lista w dropdown do wyboru próbki,
- pole `Textarea` do ręcznego wpisania własnego kodu.

**Sekcja C — detektor**
- `FloatSlider` progów entropii,
- po uruchomieniu pokazuje tabelę (`pandas.DataFrame` w `Output`) z
  metrykami dla kodu źródłowego i kodu po obfuskacji.

**Sekcja D — I/O konfiguracji**
- `Text` z nazwą pliku (`config.json`),
- przyciski „Zapisz”, „Wczytaj”, „Reset do domyślnej”,
- `Output` z komunikatem.

**Sekcja E — uruchomienie**
- przycisk „Obfuskuj”,
- dwa pola `Textarea` side-by-side: input / output,
- pod spodem `Output` z metrykami detektora (przed/po).

### 5. Deklaratywny opis parametrów (dla auto-generowania UI)
W jednym miejscu (np. stała `PARAM_SCHEMA`) zbiorczy opis, z którego
kod generuje i widgety, i walidator, i szablon JSON:
```python
PARAM_SCHEMA = {
  "renaming": {
    "name_length":   {"type": "int",   "min": 1, "max": 32, "default": 8},
    "alphabet":      {"type": "enum",  "choices": ["lower","upper","mixed"],
                       "default": "lower"},
    "seed":          {"type": "int?",  "default": None},
  },
  ...
}
```
Jedno źródło prawdy — mniejsze ryzyko rozjazdu UI ↔ kod.

## Nowe/zmienione komórki notatnika

| # | Rodzaj | Zawartość |
|---|---|---|
| nowa md | markdown | „Etap 4 — pełna konfiguracja i UI” (zastępuje obecną krótką) |
| edycja | code | `IdentifierRenamer.__init__` + `_transform` z parametrami |
| edycja | code | `DeadCodeInserter.__init__` + `_transform` z parametrami |
| edycja | code | `StringEncryptor.__init__` + `_transform` z parametrami |
| edycja | code | `ControlFlowFlattener.__init__` + `_transform` z parametrami |
| nowa | code | `PARAM_SCHEMA` + `validate_config` + `build_obfuscator` + `build_pipeline` |
| nowa | code | `save_config`, `load_config`, `default_config` (zastąpienie obecnych) |
| nowa | code | `make_ui()` — konstrukcja panelu `ipywidgets` |
| nowa | code | wywołanie `make_ui()` — widget renderowany w notatniku |
| nowa md | markdown | instrukcja użycia UI (krok po kroku) |

## Kryteria ukończenia
1. Każdy z 4 obfuskatorów daje się skonfigurować przez UI.
2. Zmiana `seed` daje powtarzalny wynik, brak `seed` → różny za każdym razem.
3. `save_config("x.json")` + restart notatnika + `load_config("x.json")`
   odtwarza stan UI (lub: wczytana konfiguracja poprawnie inicjalizuje UI).
4. `validate_config` wykrywa co najmniej: nieznany obfuskator, zły typ,
   wartość poza zakresem `min`/`max`.
5. Pełny pipeline uruchomiony na `TEST_SAMPLES["bubble_sort"]`
   zwraca kod, który kompiluje się (`BaseObfuscator._verify`).
6. Detektor pokazuje wyższą entropię po obfuskacji niż przed.

## Ryzyka i pułapki
- **Rozjazd `ast`-owy** przy zmianie kolejności w pipeline (np. CFF po
  renamingu psuje inliner stringów). Rozwiązanie: walidator kolejności —
  `StringEncryptor` i `DeadCodeInserter` przed `CFF`, `IdentifierRenamer`
  na końcu, jak dziś — i w razie ręcznej zmiany ostrzeżenie w UI.
- **`ipywidgets` w VSCode**: wymaga rozszerzenia Jupyter; warto w README
  zostawić notkę „uruchom w Jupyter Lab / Notebook”.
- **Serializacja `None` w JSON**: trzeba zadbać, żeby `seed=null`
  zamieniał się na `None` po wczytaniu (to domyślne zachowanie `json`,
  ale warto mieć test).

## Oszacowanie czasu
- Refactor konstruktorów: ~1 h
- `PARAM_SCHEMA` + walidator + builder: ~1 h
- UI: ~2 h
- Testy manualne i integracyjne: ~1 h
- Aktualizacja dokumentacji w notatniku: ~30 min

**Razem: ~5–6 h pracy.**

## Co pokazuję na zajęciach
- Wczytanie notatnika → widok panelu.
- Zmiana parametrów (np. `name_length` z 8 na 3) → natychmiastowy wpływ
  na wynik.
- Zapis konfiguracji → pokazanie pliku JSON → wczytanie → odtworzony stan.
- Porównanie entropii przed/po.
