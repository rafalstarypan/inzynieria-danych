# Etap 4 — podsumowanie zmian i plan prezentacji

## Podsumowanie zmian

### Co dostarczono (domyka PK 9)
- Parametryzacja wszystkich 4 obfuskatorów
- Deklaratywny schemat parametrów jako jedno źródło prawdy
- Walidacja konfiguracji + budowanie pipeline'u z configu
- Zapis i odczyt konfiguracji do/z pliku JSON
- Interaktywny panel `ipywidgets` z 4 zakładkami

### Zmiany w istniejącym kodzie

| Plik / komórka | Zmiana |
|---|---|
| [cell-renaming-impl](obfuskacja_projekt_etap4.ipynb) | `IdentifierRenamer` + `name_length`, `alphabet` (`lower/upper/mixed/hex`), `seed` |
| [cell-deadcode-impl](obfuskacja_projekt_etap4.ipynb) | `DeadCodeInserter`: rename `density` → `insertion_probability`, + `max_insertions`, `templates` |
| [cell-stringenc-impl](obfuskacja_projekt_etap4.ipynb) | `StringEncryptor` + `key_length`, `skip_docstrings`, `min_string_length` |
| [cell-cff-impl](obfuskacja_projekt_etap4.ipynb) | `ControlFlowFlattener` + `shuffle_blocks`, `state_var_name`, `apply_to_classes` |
| demo cells | aktualizacja przykładów pod nowe API |

### Nowe komórki (etap 4)

| ID komórki | Zawartość |
|---|---|
| `etap4-intro` | markdown — cele etapu + spis treści |
| `etap4-schema` | `PARAM_SCHEMA`, `OBFUSCATOR_CLASSES`, `GENERATOR_SCHEMA`, `DETECTOR_SCHEMA` |
| `etap4-validate` | `default_config()`, `validate_config()`, `_validate_value()` + sanity check |
| `etap4-builders` | `build_obfuscator()`, `build_pipeline()` |
| `etap4-configio` | `save_config()`, `load_config()` + round-trip test |
| `etap4-ui-helpers` | `_widget_for_param()`, `_read_widget()`, `_write_widget()` |
| `etap4-make-ui` | `make_ui()` — 4 zakładki + sekcja uruchomienia |
| `etap4-display-ui` | `display(make_ui())` |
| `etap4-md-usage` | instrukcja obsługi |

### Usunięto
Poprzedni szkielet etapu 4 (obsługiwał tylko `renaming` i `deadcode`, bez
walidacji, bez parametrów pozostałych algorytmów, bez UI).

### Weryfikacja
- 24/24 komórki kodu wykonują się bez błędów
- Pełny pipeline na `bubble_sort`: **5/5** przypadków testowych zachowuje
  semantykę (wejścia: `[]`, `[1]`, `[3,1,2]`, `[5,4,3,2,1]`, odwrotny 10)
- Entropia po pełnym pipeline: total **3.93 → 4.16**, identyfikatorów **3.11 → 4.25**
- Walidator wykrywa `name_length=999` (poza zakresem) i nieznany obfuskator
  `'foobar'` w pipeline
- Round-trip `save_config → load_config` zachowuje niestandardowe
  parametry (np. `alphabet='hex'`, `name_length=5`)

---

## Plan prezentacji

### Cel prezentacji
Pokazać domknięcie PK 9: projekt nie tylko obfuskuje, ale robi to
**konfigurowalnie**, z zapisem/odczytem z dysku i przez interfejs
graficzny. Jednocześnie zachowuje poprawność (semantyczną i składniową).

### Czas: ~10 min

### Scenariusz (krok po kroku)

**1. Wstęp (30 s)**
- „Na zajęciach 2 i 3 zaimplementowaliśmy 4 algorytmy obfuskacji + detektor
  entropii. W etapie 4 dołożyliśmy warstwę konfiguracji — punkt
  kontrolny 9 z listy."
- Slajd: diagram warstw (jest już w notatniku w [pxKi_1c22vT_](obfuskacja_projekt_etap4.ipynb))

**2. Architektura konfiguracji (1,5 min)**
- Pokaż komórkę `etap4-schema` — `PARAM_SCHEMA`.
- Główny punkt: *„jedno źródło prawdy"* — ten sam schemat napędza
  walidację, budowanie pipeline'u i generowanie widżetów UI. Dodanie
  nowego parametru to jedna linijka w jednym miejscu.
- Pokaż `default_config()` — deklaratywnie odtwarzany z `PARAM_SCHEMA`.

**3. Walidacja (1 min)**
- Pokaż błędną konfigurację w live demo:
  ```python
  cfg = default_config()
  cfg['obfuscators']['renaming']['name_length'] = 999
  validate_config(cfg)
  # ['obfuscators.renaming.name_length: wartość 999 > max 32']
  ```
- Druga próba — literówka w nazwie obfuskatora:
  ```python
  cfg['pipeline'] = ['foobar']
  validate_config(cfg)
  # ["pipeline: nieznany obfuskator 'foobar'"]
  ```

**4. Interfejs użytkownika — demo live (4 min)**
Uruchom komórkę `etap4-display-ui`. Scenariusz kliknięć:

1. **Zakładka Pipeline** — pokaż cztery checkboxy i akordeon z parametrami.
   Rozwiń `Renaming`, pokaż suwak `name_length` i dropdown `alphabet`.
2. **Zakładka Dane** — kliknij *Generuj próbki* (seed=42, n=10). Z listy
   wybierz `gen_class_002` (`DataProcessor`). Kod wkleja się do pola
   wejściowego.
3. **Kliknij Obfuskuj**. Pokaż:
   - wynik w polu `Kod po obfuskacji`,
   - tabelkę metryk przed/po,
   - decyzję detektora (powinien zaflagować).
4. **Zakładka Pipeline** — zmień `name_length` z 8 na 3, `alphabet`
   na `hex`. Kliknij *Obfuskuj* ponownie. Komentarz: „identyfikatory
   są teraz krótsze i heksadecymalne — entropia identyfikatorów spadła,
   co widać w metrykach".
5. **Zakładka Konfiguracja** — wpisz nazwę pliku, kliknij *Zapisz*.
   Pokaż ten plik JSON (otwórz w VS Code obok notatnika).
6. **Reset domyślny** → kliknij *Wczytaj* → UI wraca do stanu z pliku.
   Efekt: *„stan UI odtworzony z dysku"*.

**5. Poprawność (1,5 min)**
- Pokaż w konsoli (lub w slajdzie zrzut ekranu):
  ```
  bubble_sort([]) -> []  OK
  bubble_sort([1]) -> [1]  OK
  bubble_sort([3, 1, 2]) -> [1, 2, 3]  OK
  bubble_sort([5, 4, 3, 2, 1]) -> [1, 2, 3, 4, 5]  OK
  bubble_sort([10,…,1]) -> [1,…,10]  OK
  ```
- Komentarz: *„wszystkie 4 techniki złożone w pełny pipeline zachowują
  semantykę funkcji. Rozmiar wyniku rośnie ~4×, entropia identyfikatorów
  ~1.4×."*

**6. Co dalej (30 s)**
- PK 10 to „finalna wersja A, testowanie, ustalenie listy poprawek"
  — zapowiedź etapu 5: benchmark na większej liczbie próbek, wykresy,
  detektor ML (Random Forest / Gradient Boosting wg literatury z raportu 1).

---

### Slajdy (jeśli potrzebne)

| # | Tytuł | Treść |
|---|---|---|
| 1 | Etap 4 — konfiguracja i UI | PK 9; autorzy; co domykamy |
| 2 | Architektura | diagram warstw, zaznaczenie nowej warstwy *Config/UI* |
| 3 | PARAM_SCHEMA | zrzut kodu — 4 obfuskatory × parametry |
| 4 | Walidacja | przykład błędnej konfiguracji + komunikat |
| 5 | UI — zakładka Pipeline | zrzut ekranu |
| 6 | UI — zakładka Konfiguracja | zrzut ekranu + plik `config.json` |
| 7 | Wyniki | tabela metryk + wynik testów semantycznych |
| 8 | Plan etapu 5 | benchmark + ML detector |

Kluczowe liczby na slajdach:
- **4** obfuskatory × **3–4** parametry = **~15** konfigurowalnych wartości
- **5/5** testów semantycznych przechodzi
- **3.93 → 4.16** entropia całkowita, **3.11 → 4.25** entropia identyfikatorów
- **100%** kompilacji po każdym kroku pipeline'u

---

### Potencjalne pytania od prowadzącego — gotowe odpowiedzi

**„Po co tak rozbudowany schemat? Nie wystarczyło kilku `if`-ów?"**
Jedno źródło prawdy dla walidacji, buildera i UI. Dodanie nowego
parametru to zmiana w jednym miejscu, bez ryzyka rozjazdu UI ↔ kod.

**„Co jeśli ktoś ręcznie edytuje `config.json` i wpisze zły typ?"**
`load_config` waliduje po wczytaniu i rzuca `ValueError` z listą
błędów. UI przechwytuje i wyświetla w panelu *Konfiguracja*.

**„Dlaczego `renaming` musi być ostatni w pipeline?"**
Wcześniejsze kroki (string encrypt, CFF) dodają nowe identyfikatory
(`_state`, `_ret`, `_b`, `_k`). Renaming musi widzieć wszystkie, żeby
je również zamienić.

**„Czy UI działa w VS Code?"**
Tak, wymaga rozszerzenia Jupyter (standardowe). Równie dobrze działa
w klasycznym Jupyter Lab / Notebook.

**„A jak z deterministycznością?"**
Każdy obfuskator ma parametr `seed`. Ustawiony → powtarzalny wynik.
Pusty → `secrets`/`random` bez ziarna. Generator danych też ma seed.

**„Co jest planowane w etapie 5?"**
Dokumentują to pliki
[plan_opcja_2_etap5_testy_metryki.md](plan_opcja_2_etap5_testy_metryki.md)
i [plan_opcja_3_etap5_detektor_ml.md](plan_opcja_3_etap5_detektor_ml.md)
— benchmark na większej liczbie próbek + klasyfikator ML
(Random Forest / GB) zgodnie z literaturą z raportu 1.
