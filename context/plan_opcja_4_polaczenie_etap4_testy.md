# Opcja 4 — Połączenie: zamknięcie etapu 4 (UI + konfiguracja) + zestaw testów

## Cel
Zamknąć PK 9 (parametry + UI + I/O konfiguracji) i jednocześnie pokazać,
że **to działa** — mini-benchmark na znanych próbkach z
wizualizacjami. Mniej ambitne niż opcja 3 (ML), ale szersze niż opcja 1
(sama konfiguracja) — daje mocny materiał na prezentację: *coś do
klikania* i *coś do oglądania*.

Ta opcja to złożenie Opcji 1 + „lekkiej” wersji Opcji 2. Świadomie
odcinamy: `real_samples` z zewnątrz (zostaje generator + `TEST_SAMPLES`),
kontrakty testowe tylko dla 3 funkcji, tylko 3 wykresy zamiast 6.

## Zakres

### Część A — z Opcji 1 (pełny zakres)
1. Rozszerzenie konstruktorów wszystkich 4 obfuskatorów o parametry
   (tabela parametrów jak w `plan_opcja_1_...md` sekcja 1).
2. `PARAM_SCHEMA` jako pojedyncze źródło prawdy.
3. `build_obfuscator`, `build_pipeline`, `validate_config`.
4. Nowy, pełny `config` (obfuscators, pipeline, generator, detector).
5. `save_config` / `load_config` z walidacją.
6. UI `ipywidgets` — panel z sekcjami A–E (obfuskatory, generator,
   detektor, I/O konfiguracji, uruchomienie).

**Szczegóły w [plan_opcja_1_etap4_konfiguracja_ui.md](plan_opcja_1_etap4_konfiguracja_ui.md).**

### Część B — mini-benchmark (okrojona Opcja 2)
1. **Metryki** (bez rzeczy dodatkowych z opcji 2):
   - poprawność: `compiles`, `semantics_ok`,
   - utrudnienie: LOC, `ast_depth`, `entropy_total`,
     `entropy_identifiers`,
   - wydajność: `obf_time_ms`.
2. **Zbiór danych**:
   - 5 próbek z `TEST_SAMPLES`,
   - 20 z `CodeGenerator.generate_batch(n=20, seed=42)`,
   - **bez** `real_samples/*.py`.
3. **Kontrakty różnicowe** tylko dla 3 funkcji: `bubble_sort`,
   `binary_search`, `caesar_encrypt`. Po ~10 przypadków testowych
   każda.
4. **Klasa `Benchmark`** — okrojona wersja z opcji 2 (bez scatterów
   i ROC):
   ```python
   class Benchmark:
       def run(self, samples: dict, pipelines: dict) -> pd.DataFrame: ...
       def differential_test(self, original, obfuscated, cases) -> bool: ...
   ```
5. **Trzy wykresy**:
   - bar chart: średnia `entropy_total` per technika
     (oryginał / renaming / dead_code / string_enc / cff / full),
   - bar chart: średnie LOC per technika,
   - macierz pomyłek detektora progowego.
6. **Tabela zbiorcza** (`pd.DataFrame`) renderowana w notatniku.

### Część C — integracja UI z benchmarkiem
- Przycisk „Uruchom benchmark z obecną konfiguracją” → uruchamia
  benchmark na aktualnie wybranym pipeline i pokazuje tabelę + 1 wykres
  w `Output`.
- Demonstruje, że konfiguracja nie jest martwym plikiem, tylko steruje
  rzeczywistym pomiarem.

## Nowe/zmienione komórki notatnika

| # | Rodzaj | Zawartość | Skąd |
|---|---|---|---|
| edycja | `__init__` 4 obfuskatorów z parametrami | A |
| nowa | `PARAM_SCHEMA` + builder + walidator | A |
| nowa | `save_config` / `load_config` / `default_config` | A |
| nowa | `make_ui()` (sekcje A–E) | A |
| nowa md | instrukcja obsługi UI | A |
| nowa | visitor cyklomatyki (minimalny) | B |
| nowa | `CONTRACTS` dla 3 funkcji + `differential_test` | B |
| nowa | `Benchmark.run` | B |
| nowa | uruchomienie benchmarku → `df` → 3 wykresy | B |
| edycja UI | dodanie przycisku „Uruchom benchmark” | C |
| nowa md | dyskusja wyników (co widać w tabeli) | B |

## Kryteria ukończenia
1. Wszystkie kryteria z Opcji 1 (pkt 1–6).
2. Benchmark przechodzi na 25 próbkach, `compiles=True` dla 100%,
   `semantics_ok=True` dla 100% próbek z kontraktem.
3. Entropia całkowita po pełnym pipeline'ie wyższa niż przed dla
   ≥ 95% próbek.
4. UI ma działający przycisk „Uruchom benchmark”, wynik pokazuje się
   poniżej.

## Ryzyka
- **Zakres puchnie** — łatwo z „małego benchmarku” zrobić „pełną opcję 2”.
  Trzymać się trzech wykresów i trzech kontraktów. Więcej → zostawiamy
  na kolejny etap.
- **UI + matplotlib**: wykres rysowany w `Output` widgetu może migać
  przy re-uruchomieniu; `plt.close('all')` przed `plt.show()` w
  obsłudze przycisku.
- **Kolejność refaktoringu**: najpierw konstruktory + builder + pipeline
  (bez UI), potem benchmark (używa buildera), potem UI (używa obu).
  Inaczej UI będzie zmieniane trzy razy.

## Plan dzienny (sugerowana kolejność)
Dzień 1:
- Refactor konstruktorów (1 h)
- `PARAM_SCHEMA` + builder + walidator (1 h)
- `save_config` / `load_config` (30 min)
- Prosty smoke test każdego obfuskatora z nowymi parametrami (30 min)

Dzień 2:
- `Benchmark` + cyklomatyka + kontrakty (2 h)
- Uruchomienie benchmarku + 3 wykresy (1.5 h)
- Tabela zbiorcza + dyskusja w notatniku (30 min)

Dzień 3:
- UI `ipywidgets` — sekcje A–E (3 h)
- Przycisk „Uruchom benchmark” + integracja (1 h)
- Testy manualne, polish, dokumentacja (1 h)

**Razem: ~12 h pracy rozbite na 3 sesje.**

## Co pokazuję na zajęciach
- Otwarcie notatnika → widok panelu UI.
- Konfiguracja: wybór obfuskatorów, zmiana parametrów (np. `name_length`).
- Zapis konfiguracji do `config.json`, pokazanie pliku.
- Przycisk „Obfuskuj” na wpisanym kodzie — pokazuję wejście i wyjście.
- Przycisk „Uruchom benchmark” — pokazuję tabelę wyników i bar chart:
  „widać, że CFF najbardziej podbija głębokość AST, string encrypt
  najbardziej podnosi entropię całkowitą”.
- Macierz pomyłek detektora — pokazuję które kombinacje detektor
  łapie, a które mu umykają.

## Dlaczego ta opcja jest dobra na prezentację
- **Materiał interaktywny** (UI) robi wrażenie na pokazie.
- **Materiał liczbowy** (tabela + wykresy) daje treść do sprawozdania.
- **Mniejsze ryzyko niż opcja 3** — nie wchodzimy w ML, nie trzeba
  debugować `sklearn` ani martwić się o zbiór treningowy.
- **Domyka PK 9** zgodnie z planem zajęć — zostaje przestrzeń na
  ML (opcja 3) w kolejnym etapie, jeśli będzie czas.

## Kompromisy względem Opcji 2 i 3
- Odpuszczamy `real_samples` — brak zewnętrznego benchmarku realnego
  kodu Pythona. Jeśli prowadzący zapyta „a na prawdziwym kodzie?” —
  odpowiedź: zaplanowane na kolejny etap.
- Brak ML — detekcja pozostaje progowa. Pokazujemy tabelę
  „accuracy progowego detektora = X%”, komentujemy, że dla pełnego
  pipeline'u jest wysoka, dla samego renamingu niska, i to jest
  naturalna motywacja do wprowadzenia ML później.
