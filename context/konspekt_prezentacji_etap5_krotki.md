# Krótki konspekt prezentacji — tekst do czytania

## 1. Wstęp

Dzień dobry.

Chciałbym krótko zaprezentować kolejny etap naszego projektu „Obfuskator i detektor obfuskacji kodu źródłowego programów”.

W poprzednim etapie dodaliśmy konfigurację obfuskatorów, zapis i odczyt konfiguracji z JSON oraz interfejs użytkownika w `ipywidgets`.

W tym etapie skupiliśmy się na tym, żeby obfuskację mierzyć liczbowo.

Dodaliśmy mini-benchmark, który sprawdza poprawność kodu po obfuskacji i porównuje wpływ poszczególnych technik.

## 2. Co już mamy

W projekcie mamy cztery techniki obfuskacji.

Pierwsza to zmiana nazw identyfikatorów, czyli `IdentifierRenamer`.

Druga to wstawianie martwego kodu, czyli `DeadCodeInserter`.

Trzecia to szyfrowanie stringów, czyli `StringEncryptor`.

Czwarta to spłaszczanie przepływu sterowania, czyli `ControlFlowFlattener`.

Techniki można uruchamiać osobno albo połączyć w pełny pipeline.

Domyślnie pipeline ma kolejność: `string_encrypt`, `dead_code`, `cff`, `renaming`.

Renaming jest na końcu, ponieważ wcześniejsze techniki dodają nowe zmienne pomocnicze i one też powinny zostać przemianowane.

## 3. Po co benchmark

Sama obfuskacja wizualnie wygląda dobrze, ale to nie wystarcza.

Musimy sprawdzić, czy kod po obfuskacji dalej się kompiluje i czy zachowuje ten sam wynik działania.

Musimy też wiedzieć, które techniki najbardziej zwiększają rozmiar kodu, głębokość AST i entropię.

Dlatego dodaliśmy sekcję `Etap 5 — mini-benchmark i metryki`.

## 4. Dane i warianty

Benchmark działa na dwudziestu pięciu próbkach.

Pięć próbek pochodzi z `TEST_SAMPLES`, a dwadzieścia jest generowanych przez `CodeGenerator`.

Dla każdej próbki testujemy sześć wariantów.

Pierwszy wariant to `none`, czyli kod bez obfuskacji.

Kolejne warianty to pojedyncze techniki: `renaming`, `dead_code`, `string_encrypt` i `cff`.

Ostatni wariant to `full`, czyli pełny pipeline ze wszystkimi technikami.

W sumie benchmark wykonuje sto pięćdziesiąt przypadków testowych.

## 5. Testy semantyczne

Najważniejsze jest to, że nie sprawdzamy tylko kompilacji.

Dla trzech funkcji mamy też testy semantyczne.

Są to `bubble_sort`, `binary_search` i `caesar_encrypt`.

Dla każdej z tych funkcji uruchamiamy oryginał i wersję po obfuskacji na tych samych wejściach.

Jeśli wyniki są identyczne, uznajemy, że semantyka została zachowana.

To daje mocniejszą weryfikację niż samo `compile`.

## 6. Metryki

Benchmark zbiera kilka metryk.

`compile_rate` mówi, czy kod po obfuskacji nadal się kompiluje.

`semantic_rate` mówi, czy funkcje z kontraktami zwracają te same wyniki.

`avg_loc` pokazuje średnią liczbę linii kodu.

`avg_ast_depth` pokazuje średnią głębokość drzewa AST.

`avg_entropy_total` pokazuje entropię całego źródła.

`avg_entropy_identifiers` pokazuje entropię identyfikatorów.

`avg_time_ms` pokazuje średni czas obfuskacji.

## 7. Wyniki

W aktualnym smoke teście benchmark utworzył sto pięćdziesiąt wierszy wyników.

Nie było błędów kompilacji.

Nie było też błędów semantycznych dla funkcji z kontraktami.

To oznacza, że w sprawdzonym zakresie obfuskatory zachowują poprawność.

Wyniki pokazują też różnice między technikami.

Renaming najbardziej wpływa na entropię identyfikatorów.

Martwy kod zwiększa liczbę linii.

CFF zwiększa złożoność struktury kodu.

Pełny pipeline daje największy łączny koszt, bo łączy efekty wszystkich technik.

## 8. Wykresy

W notebooku dodaliśmy trzy wykresy.

Pierwszy pokazuje średnią entropię całkowitą dla technik.

Drugi pokazuje średnią liczbę linii kodu.

Trzeci pokazuje macierz pomyłek detektora progowego.

Macierz pomyłek jest szczególnie ważna.

Pokazuje, że detektor oparty tylko na progach entropii ma ograniczenia.

W smoke teście miał wysoką precyzję, ale niski recall.

To znaczy, że jeśli już coś wykryje, zwykle ma rację, ale pomija część obfuskowanego kodu.

## 9. Wniosek

Najważniejszy wniosek jest taki, że projekt przeszedł od samego generowania obfuskowanego kodu do jego mierzalnej oceny.

Mamy konfigurację, UI, pipeline obfuskacji i benchmark.

Benchmark potwierdza poprawność dla sprawdzonych przypadków oraz pokazuje koszt poszczególnych technik.

Jednocześnie wyniki detektora pokazują, że prosty próg entropii nie wystarcza.

Naturalnym kolejnym krokiem jest detektor ML oparty na większej liczbie cech statycznych.

Na przykład można dodać `FeatureExtractor` i klasyfikator Random Forest albo Gradient Boosting.

Dziękuję.

## 10. Krótkie odpowiedzi

**Dlaczego tylko 25 próbek?**

To jest mini-benchmark do obecnego etapu. Najpierw chcieliśmy mieć stabilny mechanizm pomiaru, a potem można rozszerzyć zbiór danych.

**Dlaczego tylko trzy funkcje mają testy semantyczne?**

Bo test semantyczny wymaga jasnego API i konkretnych wejść. Wybraliśmy funkcje, dla których łatwo porównać wynik przed i po obfuskacji.

**Czy zero błędów oznacza pełną poprawność?**

Nie. Oznacza poprawność dla obecnego zestawu testów. To nie jest formalny dowód dla wszystkich programów.

**Dlaczego detektor ma niski recall?**

Bo używa tylko entropii. Nie każda technika obfuskacji mocno podnosi entropię, dlatego potrzebujemy więcej cech i prawdopodobnie modelu ML.
