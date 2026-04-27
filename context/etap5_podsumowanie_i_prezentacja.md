# Etap 5 — podsumowanie zmian i plan prezentacji

## Podsumowanie zmian

### Co dostarczono
- Mini-benchmark technik obfuskacji w notatniku `obfuskacja_projekt_etap4.ipynb`
- Kontrakty semantyczne dla wybranych funkcji testowych
- Porównanie 6 wariantów: `none`, `renaming`, `dead_code`, `string_encrypt`, `cff`, `full`
- Pomiar kompilowalności, zgodności semantycznej, LOC, głębokości AST, entropii i czasu obfuskacji
- Trzy wizualizacje: średnia entropia, średni LOC, macierz pomyłek detektora progowego

### Nowe komórki w notebooku

| ID komórki | Zawartość |
|---|---|
| `etap5-intro` | markdown — cel etapu 5 i opis mierzonych metryk |
| `etap5-benchmark-core` | `Benchmark`, `CONTRACTS`, `differential_test`, warianty pipeline'u |
| `etap5-run-benchmark` | uruchomienie benchmarku na `TEST_SAMPLES` + 20 próbkach generatora |
| `etap5-plots` | wykresy: entropia, LOC, macierz pomyłek detektora |
| `etap5-md-discussion` | wnioski i przejście do dalszych prac |

### Zakres benchmarku

Zbiór danych:
- 5 ręcznych próbek z `TEST_SAMPLES`
- 20 próbek z `CodeGenerator(seed=42).generate_batch(20)`
- razem 25 programów wejściowych

Warianty porównawcze:
- `none` — kod oryginalny
- `renaming` — tylko zmiana nazw identyfikatorów
- `dead_code` — tylko martwy kod
- `string_encrypt` — tylko szyfrowanie stringów
- `cff` — tylko spłaszczanie przepływu sterowania
- `full` — pełny pipeline: `string_encrypt -> dead_code -> cff -> renaming`

Metryki:
- `compiles` — czy kod po transformacji kompiluje się
- `semantics_ok` — czy wynik funkcji jest taki sam przed i po obfuskacji
- `loc` — liczba niepustych linii kodu
- `chars` — liczba znaków
- `ast_depth` — głębokość drzewa AST
- `entropy_total` — entropia całego źródła
- `entropy_identifiers` — entropia identyfikatorów
- `obf_time_ms` — czas obfuskacji w milisekundach
- `detected` — decyzja detektora progowego

Kontrakty semantyczne:
- `bubble_sort`
- `binary_search`
- `caesar_encrypt` w próbce `caesar_cipher`

### Co pokazują wyniki

Benchmark daje odpowiedź na trzy pytania:

1. Czy obfuskatory nie psują składni?
2. Czy pełny pipeline zachowuje semantykę dla funkcji z kontraktami?
3. Które techniki najbardziej zmieniają mierzalne cechy kodu?

Najważniejszy efekt prezentacyjny:
- etap 4 pokazywał, że obfuskator jest konfigurowalny i ma UI
- etap 5 pokazuje, że można go mierzyć liczbowo i porównywać techniki

---

## Plan prezentacji

### Cel prezentacji

Pokazać przejście od samego działania obfuskatora do mierzalnej oceny jego jakości. Projekt nie tylko generuje obfuskowany kod, ale potrafi automatycznie sprawdzić poprawność i zebrać metryki.

### Czas: ~10 min

### Scenariusz

**1. Przypomnienie etapu 4 (1 min)**
- „W etapie 4 domknęliśmy konfigurację, zapis/odczyt JSON i panel `ipywidgets`. Teraz dokładamy warstwę pomiarową.”
- Pokaż `PARAM_SCHEMA` albo UI tylko krótko, bez ponownego omawiania szczegółów.

**2. Cel etapu 5 (1 min)**
- Pokaż komórkę `etap5-intro`.
- Powiedz: „Chcemy sprawdzić, czy obfuskacja zachowuje składnię i semantykę oraz jak zmienia strukturę kodu.”
- Wymień metryki: kompilacja, semantyka, LOC, AST depth, entropia, czas.

**3. Kontrakty semantyczne (1,5 min)**
- Pokaż `CONTRACTS` w komórce `etap5-benchmark-core`.
- Wyjaśnij, że benchmark nie tylko sprawdza `compile`, ale realnie uruchamia funkcje przed i po obfuskacji.
- Przykład wypowiedzi:
  „Dla `bubble_sort` sprawdzamy puste listy, listę jednoelementową, typowe dane i listę odwróconą. Jeśli wynik po obfuskacji jest taki sam, uznajemy kontrakt za spełniony.”

**4. Warianty pipeline'u (1 min)**
- Pokaż `BENCHMARK_PIPELINES`.
- Podkreśl, że benchmark porównuje techniki osobno i pełny pipeline.
- Ważny komentarz:
  „Dzięki temu widzimy, czy wzrost entropii lub rozmiaru wynika z jednej techniki, czy dopiero z ich połączenia.”

**5. Uruchomienie benchmarku (2 min)**
- Uruchom komórkę `etap5-run-benchmark`.
- Pokaż tabelę `Podsumowanie benchmarku`.
- Omów kolumny:
  `compile_rate`, `semantic_rate`, `avg_loc`, `avg_ast_depth`, `avg_entropy_total`, `avg_entropy_identifiers`, `avg_time_ms`.
- Jeśli wszystko przechodzi:
  „Najważniejsze: brak błędów kompilacji i brak błędów semantycznych w kontraktach.”
- Jeśli pojawi się błąd:
  „To traktujemy jako wynik benchmarku, nie ukrywamy go. Wskazuje ograniczenie konkretnej techniki.”

**6. Wykresy (2 min)**
- Uruchom komórkę `etap5-plots`.
- Wykres entropii:
  „Pokazuje, które techniki najsilniej zwiększają losowość / złożoność tekstową kodu.”
- Wykres LOC:
  „Pokazuje koszt rozmiarowy obfuskacji.”
- Macierz pomyłek:
  „Pokazuje ograniczenia prostego detektora progowego. To naturalna motywacja do detektora ML.”

**7. Wnioski i dalsze prace (1,5 min)**
- Pokaż `etap5-md-discussion`.
- Podsumuj:
  „Mamy działający obfuskator, konfigurację, UI i pierwszą warstwę testów ilościowych. Następny krok to rozbudowanie detektora o większą liczbę cech i klasyfikator ML.”
- Odwołaj się do planu `plan_opcja_3_etap5_detektor_ml.md`.

---

## Slajdy

| # | Tytuł | Treść |
|---|---|---|
| 1 | Etap 5 — benchmark | Cel: pomiar jakości obfuskacji |
| 2 | Architektura testów | `TEST_SAMPLES`, `CodeGenerator`, warianty pipeline'u |
| 3 | Kontrakty semantyczne | `bubble_sort`, `binary_search`, `caesar_encrypt` |
| 4 | Metryki | kompilacja, semantyka, LOC, AST, entropia, czas |
| 5 | Tabela wyników | zrzut `benchmark_summary` |
| 6 | Wykresy | entropia, LOC, macierz pomyłek |
| 7 | Wnioski | prosty detektor działa częściowo; ML jako kolejny krok |

---

## Gotowe wypowiedzi

**„Dlaczego nie wystarczy sprawdzić, że kod się kompiluje?”**

Kompilacja potwierdza tylko poprawność składniową. Kod może się kompilować, ale zwracać inny wynik. Dlatego dla wybranych funkcji dodaliśmy testy różnicowe: uruchamiamy oryginał i wersję obfuskowaną na tych samych wejściach.

**„Dlaczego tylko trzy funkcje mają kontrakty?”**

To świadomy zakres mini-benchmarku. Ręczne próbki obejmują więcej kodu, ale kontrakty dobraliśmy dla funkcji z prostym, jednoznacznym API. Dla klas i generatora mierzymy kompilowalność oraz metryki statyczne.

**„Co oznacza macierz pomyłek detektora?”**

Wiersze to prawdziwa klasa: kod czysty albo obfuskowany. Kolumny to decyzja detektora. Jeśli pojawia się dużo `FN`, detektor nie wykrywa części obfuskacji. Jeśli pojawia się dużo `FP`, oznacza zwykły kod jako obfuskowany.

**„Dlaczego detektor progowy może być za słaby?”**

Bo używa tylko entropii. Niektóre techniki, np. martwy kod albo CFF, mogą mocno zmienić strukturę AST bez dużego wzrostu entropii. Dlatego kolejnym krokiem jest `FeatureExtractor` i klasyfikator ML.

**„Co będzie dalej?”**

Rozszerzenie benchmarku o więcej próbek, zapis wyników do CSV, dodanie cech statycznych i porównanie detektora progowego z modelem Random Forest / Gradient Boosting.

---

## Kryteria demonstracyjne

Przed prezentacją warto uruchomić notebook od setupu do końca etapu 5 i sprawdzić:

- komórki etapu 4 nadal działają
- `make_ui()` renderuje panel
- `etap5-run-benchmark` kończy się tabelą podsumowania
- `Błędy kompilacji` wynoszą 0 albo są świadomie omówione
- `Błędy semantyczne w kontraktach` wynoszą 0 albo są świadomie omówione
- `etap5-plots` renderuje trzy wykresy
