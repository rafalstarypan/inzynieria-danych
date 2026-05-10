# Etap 7 — przygotowanie do prezentacji (cheat sheet)

> Krótki dokument do otwarcia tuż przed zajęciami. Pełny scenariusz
> w `etap7_podsumowanie_i_prezentacja.md`.

## Przed wejściem na salę

1. **Otwórz notebook** [obfuskacja_projekt.ipynb](../obfuskacja_projekt.ipynb)
   w VS Code (działa też Jupyter Lab).
2. **Sprawdź kernel** — prawy górny róg → musi być `.venv (Python 3.12)`.
3. **Restart kernel** (ikona ⟲) → **Run All** (▶▶).
4. Czekaj ~30 s — najwolniejsza komórka to `etap6-ml-train` (~10 s
   trening RF) i `etap5-run-benchmark` (~10 s).
5. **Scrolluj na sam dół** — ostatnia komórka `etap7-display-ui` ma
   pokazać panel z 4 zakładkami. Jeśli nie pokazuje — Run All jeszcze raz.

## Plan wystąpienia (12 min)

| # | Sekcja | Czas | Komórka do pokazania |
|---|---|---|---|
| 1 | Wstęp + przypomnienie etap 5/6 | 1 min | (ustnie) |
| 2 | Algorytm 5 — MBA | 2 min | `cell-md-mba`, `cell-mba-demo` |
| 3 | Balansowanie ML 1:5 → 1:1 | 2 min | `etap6-ml-train` |
| 4 | Wizualizacje ML | 2 min | `etap6-plots-ml` |
| 5 | Porównanie ML vs progowy | 2 min | `etap6-compare-detectors` |
| 6 | Per-variant recall | 1.5 min | `etap6-per-variant-recall` |
| 7 | UI live demo | 2 min | `etap7-display-ui` |
| 8 | Wnioski + plan etapu 8 | 1 min | (ustnie) |

## Skrypt do live demo (sekcja 7)

1. Przewiń do `etap7-display-ui`. Powinien być widoczny panel z czterema
   zakładkami: **Pipeline / Dane / Detektor / Konfiguracja**.
2. **Zakładka Dane**:
   - Kliknij *Generuj próbki* (seed=42, n=10).
   - Z dropdownu „Wygenerowane" wybierz `gen_class_002`.
   - Kod wpada do pola wejściowego.
3. **Zakładka Pipeline**:
   - Pokaż 5 checkboxów (renaming / dead_code / string_encrypt / cff / **mba**).
   - Wszystkie 5 zaznaczone domyślnie.
   - Rozwiń akordeon „MBA (literały arytmetyczne)" — pokaż parametry:
     `apply_probability=0.7`, `expression_depth=1`, `min/max_value=±1000`.
4. **Zakładka Detektor**:
   - Dropdown „Tryb detektora" — pokaż 3 opcje.
   - Domyślnie: **Oba**.
5. Wróć do dołu, kliknij **Obfuskuj**.
6. Pokaż wynik:
   - Pole „Kod po obfuskacji" — widoczne MBA literały i zmienione nazwy.
   - Tabela metryk (entropia przed/po).
   - Decyzja detektora **progowego**.
   - Decyzja detektora **ML** + `p_obf`.
   - Linia „Zgodność detektorów: TAK/NIE".

## Liczby do zapamiętania

- **5** algorytmów obfuskacji
- **30** cech statycznych
- **210** próbek w datasecie ML (105 clean + 105 obf, **1:1**)
- **F1 weighted (5-fold CV)** = **0.857 ± 0.034**
- **ML > Progowy** o **+23.8 pp** accuracy, **+47.9 pp** F1
- **string_encrypt** — najtrudniejszy (recall **46.7%**), reszta ≥ 83%
- **100%** kompilacji i semantyki w benchmarku (175 wierszy)

## Najczęstsze pytania → odpowiedzi (skróty)

**„Czemu MBA?"** → wymieniona w raporcie literaturowym (Xu 2020,
sekcja 2.3); ortogonalna do 4 istniejących (operuje na **wartościach**).

**„Czemu accuracy 0.86, nie 0.95?"** → mały zbiór (210 próbek), 30 cech
przy 168 train. Większy zbiór z PyPI to zakres etapu 8.

**„Czemu string_encrypt słaby?"** → zaszyfrowane stringi wyglądają jak
zwykłe literały. Lepsza detekcja wymaga cech per-string (entropia
każdego literału osobno).

**„Działa w Colab?"** → Tak, komórka `cell-install` wykrywa Colab i włącza
custom widget manager. Auto-instaluje sklearn / matplotlib / ipywidgets
jeśli brak.

**„Czemu RF, nie sieć neuronowa?"** → 210 próbek to za mało dla NN.
RF radzi sobie z małymi zbiorami i daje natywnie ważności cech.

## Backup plan (jeśli coś się wywali)

| Problem | Rozwiązanie |
|---|---|
| Kernel nie startuje | sprawdź `.venv (Python 3.12)` w prawym górnym rogu, *Select Kernel* → Python Environments |
| `ModuleNotFoundError: sklearn` | komórka `cell-install` powinna była zainstalować — uruchom ją ręcznie i potem *Run All Below* |
| UI pokazuje pusty Tab | restart kernel + Run All; w Colab odśwież stronę |
| Trening trwa za długo | bez paniki — `etap6-ml-train` to ~10 s; jeśli > 60 s, anuluj i sprawdź czy nie ma wcześniejszego błędu |
| `etap6-persistence` rzuca przy load | usuń `ml_detector.joblib` i uruchom ponownie etap6 (najpewniej zła wersja sklearn) |

## Po prezentacji

Jeśli prowadzący zapyta o dalsze plany:

- **Etap 8 — propozycje:**
  - real-world samples (stdlib / PyPI) — twardy benchmark FP
  - dodatkowe cechy per-string dla lepszej detekcji `string_encrypt`
  - adwersarialny obfuskator celowo unikający ML detektora
  - porównanie wielu modeli (RF/GB/LR) z hyperparam tuning
  - eksport raportu z UI do markdown / PDF
