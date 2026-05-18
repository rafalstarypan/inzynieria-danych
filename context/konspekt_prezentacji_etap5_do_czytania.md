# Konspekt prezentacji — tekst do czytania

## 1. Wstęp

Dzień dobry.

Chciałbym zaprezentować kolejny etap naszego projektu pod tytułem „Obfuskator i detektor obfuskacji kodu źródłowego programów”.

Projekt dotyczy automatycznego przekształcania kodu Pythona do postaci trudniejszej do analizy oraz wykrywania, czy dany kod wygląda na obfuskowany.

W poprzednim etapie skupiliśmy się na tym, żeby obfuskator był konfigurowalny.

Dodaliśmy parametry dla wszystkich technik, zapis i odczyt konfiguracji z pliku JSON oraz interfejs użytkownika w `ipywidgets`.

W obecnym etapie zrobiliśmy krok dalej.

Nie chcemy już tylko pokazywać, że obfuskacja działa wizualnie.

Chcemy ją mierzyć.

Dlatego dodaliśmy mini-benchmark, który sprawdza poprawność działania obfuskatorów oraz porównuje ich wpływ na kod.

## 2. Przypomnienie architektury

Na początku krótko przypomnę, co już było zaimplementowane.

W notebooku mamy cztery techniki obfuskacji.

Pierwsza technika to zmiana nazw identyfikatorów, czyli `IdentifierRenamer`.

Druga technika to wstawianie martwego kodu, czyli `DeadCodeInserter`.

Trzecia technika to szyfrowanie literałów tekstowych, czyli `StringEncryptor`.

Czwarta technika to spłaszczanie przepływu sterowania, czyli `ControlFlowFlattener`.

Te techniki można uruchamiać pojedynczo albo łączyć w pipeline.

Domyślny pipeline ma kolejność: najpierw szyfrowanie stringów, potem martwy kod, potem spłaszczanie przepływu sterowania, a na końcu zmiana nazw identyfikatorów.

Zmiana nazw jest na końcu celowo.

Wcześniejsze techniki dodają nowe zmienne pomocnicze, więc chcemy, żeby one też zostały przemianowane.

## 3. Etap 4 — konfiguracja i UI

Zanim przejdę do benchmarku, pokażę krótko etap 4.

W etapie 4 dodaliśmy jedno centralne miejsce z opisem parametrów, czyli `PARAM_SCHEMA`.

Ten schemat mówi, jakie parametry ma każdy obfuskator, jakie są ich typy, wartości domyślne i dopuszczalne zakresy.

Dzięki temu ten sam opis parametrów służy do trzech rzeczy.

Po pierwsze, służy do budowania domyślnej konfiguracji.

Po drugie, służy do walidacji konfiguracji.

Po trzecie, służy do generowania widgetów w interfejsie użytkownika.

To ogranicza ryzyko, że interfejs, walidator i kod wykonujący obfuskację zaczną się od siebie różnić.

W praktyce oznacza to, że jeśli chcemy dodać nowy parametr, robimy to w jednym miejscu.

Interfejs użytkownika pozwala wybrać techniki obfuskacji, ustawić ich parametry, wygenerować próbki kodu, uruchomić pipeline i zobaczyć wynik.

Można też zapisać konfigurację do pliku JSON i później ją wczytać.

To zamyka część projektu dotyczącą konfiguracji i obsługi przez użytkownika.

## 4. Przejście do etapu 5

Problem polega na tym, że samo pokazanie obfuskowanego kodu nie wystarcza.

Kod po obfuskacji może wyglądać na bardziej skomplikowany, ale musimy sprawdzić, czy faktycznie zachowuje poprawność.

Musimy też sprawdzić, które techniki najbardziej zmieniają kod.

Dlatego w etapie 5 dodaliśmy mini-benchmark.

Benchmark automatycznie uruchamia różne warianty obfuskacji na zestawie próbek i zbiera metryki.

To pozwala porównywać techniki nie tylko opisowo, ale też liczbowo.

## 5. Dane testowe

Benchmark korzysta z dwóch źródeł danych.

Pierwszym źródłem są ręcznie przygotowane próbki w `TEST_SAMPLES`.

Są tam między innymi `bubble_sort`, `binary_search`, szyfr Cezara, prosta klasa stosu i funkcja zliczająca słowa.

Drugim źródłem danych jest `CodeGenerator`.

Generator tworzy krótkie programy Pythona, na przykład funkcje arytmetyczne, funkcje z pętlami, funkcje tekstowe i proste klasy.

W benchmarku używamy pięciu ręcznych próbek oraz dwudziestu próbek wygenerowanych automatycznie.

Łącznie daje to dwadzieścia pięć programów wejściowych.

Każdy program jest testowany w kilku wariantach.

## 6. Porównywane warianty

W benchmarku porównujemy sześć wariantów.

Pierwszy wariant to `none`, czyli kod bez obfuskacji.

Ten wariant jest potrzebny jako punkt odniesienia.

Drugi wariant to `renaming`, czyli sama zmiana nazw identyfikatorów.

Trzeci wariant to `dead_code`, czyli samo wstawianie martwego kodu.

Czwarty wariant to `string_encrypt`, czyli samo szyfrowanie literałów tekstowych.

Piąty wariant to `cff`, czyli samo spłaszczanie przepływu sterowania.

Szósty wariant to `full`, czyli pełny pipeline składający się ze wszystkich technik.

Dzięki takiemu podziałowi możemy zobaczyć, czy dana zmiana wynika z jednej konkretnej techniki, czy dopiero z ich połączenia.

## 7. Kontrakty semantyczne

Bardzo ważnym elementem benchmarku są testy semantyczne.

Sama kompilacja kodu nie wystarcza.

Kod może być poprawny składniowo, ale nadal może zwracać zły wynik.

Dlatego dla wybranych funkcji definiujemy kontrakty testowe.

Kontrakt oznacza listę wejść, dla których porównujemy wynik funkcji przed i po obfuskacji.

W obecnej wersji mamy kontrakty dla trzech przykładów.

Pierwszy przykład to `bubble_sort`.

Dla tej funkcji sprawdzamy między innymi pustą listę, listę jednoelementową, typową listę z kilkoma elementami i listę odwróconą.

Drugi przykład to `binary_search`.

Tutaj sprawdzamy przypadki, gdy element istnieje w tablicy, gdy go nie ma, oraz przypadki brzegowe, takie jak pusta lista.

Trzeci przykład to szyfr Cezara, czyli `caesar_encrypt`.

Tutaj sprawdzamy różne teksty i różne przesunięcia.

Jeśli funkcja po obfuskacji zwraca dokładnie taki sam wynik jak funkcja oryginalna, uznajemy, że kontrakt semantyczny został spełniony.

## 8. Metryki benchmarku

Benchmark zbiera kilka metryk.

Pierwsza metryka to `compiles`.

Mówi ona, czy kod po transformacji nadal kompiluje się jako poprawny kod Pythona.

Druga metryka to `semantics_ok`.

Mówi ona, czy kod po obfuskacji zachowuje wynik działania dla funkcji, które mają zdefiniowane kontrakty.

Trzecia metryka to `loc`, czyli liczba niepustych linii kodu.

Ta metryka pokazuje, jak bardzo obfuskacja zwiększa rozmiar kodu.

Czwarta metryka to `ast_depth`, czyli głębokość drzewa składniowego AST.

Ta metryka pokazuje, czy struktura kodu stała się bardziej zagnieżdżona i trudniejsza do analizy.

Piąta metryka to `entropy_total`, czyli entropia całego kodu źródłowego.

Szósta metryka to `entropy_identifiers`, czyli entropia znaków w identyfikatorach.

Te metryki są ważne dla detekcji, bo kod obfuskowany często ma bardziej losowe nazwy i mniej naturalny rozkład znaków.

Ostatnia metryka to `obf_time_ms`, czyli czas obfuskacji w milisekundach.

Dzięki niej widzimy koszt wykonania poszczególnych technik.

## 9. Uruchomienie benchmarku

Teraz przechodzę do komórki `etap5-run-benchmark`.

Ta komórka buduje zestaw dwudziestu pięciu próbek.

Następnie uruchamia wszystkie sześć wariantów pipeline'u.

Łącznie daje to sto pięćdziesiąt przypadków testowych, ponieważ mamy dwadzieścia pięć próbek i sześć wariantów.

Po uruchomieniu dostajemy tabelę podsumowania.

W tabeli wiersz odpowiada jednej technice obfuskacji.

Kolumna `samples` mówi, ile próbek zostało przetestowanych dla danego wariantu.

Kolumna `compile_rate` mówi, jaki procent wyników poprawnie się kompilował.

Kolumna `semantic_rate` mówi, jaki procent testów semantycznych zakończył się sukcesem.

Kolumna `avg_loc` pokazuje średnią liczbę linii kodu.

Kolumna `avg_ast_depth` pokazuje średnią głębokość AST.

Kolumna `avg_entropy_total` pokazuje średnią entropię całkowitą.

Kolumna `avg_entropy_identifiers` pokazuje średnią entropię identyfikatorów.

Kolumna `avg_time_ms` pokazuje średni czas obfuskacji.

Najważniejsze są dla nas dwie pierwsze jakościowe informacje.

Po pierwsze, chcemy mieć brak błędów kompilacji.

Po drugie, chcemy mieć brak błędów semantycznych dla funkcji z kontraktami.

W aktualnym uruchomieniu smoke test pokazał sto pięćdziesiąt wierszy benchmarku, zero błędów kompilacji i zero błędów semantycznych w kontraktach.

To oznacza, że dla sprawdzonych przypadków pipeline zachowuje poprawność.

## 10. Interpretacja wyników tabeli

Tabela pozwala też zobaczyć różnice między technikami.

Zmiana nazw identyfikatorów zwykle mocno wpływa na entropię identyfikatorów.

To jest oczekiwane, ponieważ naturalne nazwy są zastępowane losowymi ciągami znaków.

Martwy kod zwykle zwiększa liczbę linii kodu.

To też jest oczekiwane, ponieważ do programu dodawane są instrukcje, które nie wpływają na wynik.

Szyfrowanie stringów może zwiększać złożoność wyrażeń i wpływać na entropię całkowitą.

Spłaszczanie przepływu sterowania zwykle zwiększa liczbę linii oraz głębokość AST.

Pełny pipeline najczęściej daje największy koszt rozmiarowy, bo łączy skutki wszystkich technik.

To jest dobra ilustracja kompromisu w obfuskacji.

Kod jest trudniejszy do analizy, ale staje się większy i bardziej złożony.

## 11. Wykres entropii

Teraz przechodzę do komórki `etap5-plots`.

Pierwszy wykres pokazuje średnią entropię całkowitą dla poszczególnych technik.

Ten wykres pomaga zobaczyć, które techniki najbardziej zmieniają rozkład znaków w kodzie.

Wysoka entropia może oznaczać, że kod wygląda mniej naturalnie.

To jest użyteczne z punktu widzenia detekcji obfuskacji.

Jednocześnie sama entropia nie opisuje wszystkiego.

Niektóre techniki mogą mocno zmieniać strukturę kodu, ale nie muszą bardzo podnosić entropii.

Dlatego potrzebujemy kilku metryk, a nie tylko jednej.

## 12. Wykres LOC

Drugi wykres pokazuje średnią liczbę linii kodu.

Ten wykres pokazuje koszt rozmiarowy obfuskacji.

Jeśli technika dodaje dużo kodu pomocniczego albo martwego kodu, liczba linii rośnie.

To jest ważne, bo obfuskacja nie jest darmowa.

Zwiększa trudność analizy, ale może zwiększać rozmiar źródła i utrudniać debugowanie.

Na tym wykresie dobrze widać, które techniki są najbardziej „ciężkie” dla kodu.

## 13. Macierz pomyłek detektora

Trzeci wykres to macierz pomyłek detektora progowego.

Detektor progowy podejmuje decyzję na podstawie dwóch progów: entropii całkowitej i entropii identyfikatorów.

Jeżeli któraś z tych wartości przekroczy próg, kod zostaje oznaczony jako obfuskowany.

Macierz pomyłek pokazuje cztery typy wyników.

`TN` oznacza kod czysty poprawnie rozpoznany jako czysty.

`TP` oznacza kod obfuskowany poprawnie rozpoznany jako obfuskowany.

`FP` oznacza kod czysty błędnie oznaczony jako obfuskowany.

`FN` oznacza kod obfuskowany, którego detektor nie wykrył.

W aktualnym smoke teście detektor miał wysoką precyzję, ale niski recall.

Oznacza to, że jeśli detektor już coś oznaczy jako obfuskowane, to zwykle ma rację.

Ale jednocześnie pomija część obfuskowanych próbek.

To jest bardzo ważny wniosek.

Prosty detektor oparty tylko na entropii nie wystarcza do wykrywania wszystkich technik.

## 14. Dlaczego detektor progowy jest ograniczony

Detektor progowy jest prosty i szybki, ale patrzy tylko na wybrane cechy.

Jeśli technika mocno zmienia nazwy identyfikatorów, entropia identyfikatorów może wzrosnąć i detektor ma szansę ją wykryć.

Ale jeśli technika głównie zmienia strukturę programu, na przykład przez spłaszczanie przepływu sterowania, sama entropia może nie wystarczyć.

Podobnie martwy kod może zwiększyć rozmiar programu, ale niekoniecznie podnieść entropię powyżej progu.

Dlatego etap 5 pokazuje ograniczenie obecnego detektora.

To ograniczenie nie jest błędem projektu.

To jest wynik eksperymentu i dobra motywacja do kolejnego etapu.

## 15. Co dalej

Naturalnym kolejnym krokiem jest rozszerzenie detektora o więcej cech statycznych.

Możemy mierzyć nie tylko entropię, ale też liczbę tokenów, średnią długość identyfikatorów, głębokość AST, złożoność cyklomatyczną, liczbę stringów i charakterystyczne wywołania takie jak `bytes`.

Na podstawie takich cech można zbudować klasyfikator ML.

Najprostszy sensowny kierunek to Random Forest albo Gradient Boosting.

Wtedy moglibyśmy porównać detektor progowy z detektorem uczonym na danych.

Oczekiwalibyśmy, że klasyfikator ML będzie lepiej wykrywał różne typy obfuskacji, ponieważ będzie korzystał z wielu cech naraz.

To jest opisane w planie dalszych prac jako etap z `FeatureExtractor` i `MLDetector`.

## 16. Podsumowanie

Podsumowując, w tym etapie dodaliśmy do projektu warstwę pomiarową.

Projekt nie tylko obfuskuje kod i pozwala go konfigurować przez UI.

Teraz potrafi też automatycznie porównać techniki obfuskacji na zestawie próbek.

Benchmark sprawdza kompilowalność, zgodność semantyczną, rozmiar kodu, głębokość AST, entropię i czas wykonania.

Wyniki pokazują, że dla sprawdzonych kontraktów obfuskacja zachowuje poprawność.

Wyniki pokazują też, że prosty detektor progowy ma ograniczenia.

To daje nam jasny kierunek rozwoju: większy benchmark, więcej cech statycznych i detektor ML.

Dziękuję.

## 17. Krótkie odpowiedzi na pytania

### Dlaczego benchmark ma tylko 25 próbek?

To jest celowo mini-benchmark do obecnego etapu projektu.

Chcieliśmy najpierw zbudować stabilny mechanizm pomiaru i pokazać go na małym, kontrolowanym zbiorze.

W kolejnym kroku można zwiększyć liczbę próbek i dodać prawdziwe fragmenty kodu z projektów open-source.

### Dlaczego tylko trzy funkcje mają testy semantyczne?

Testy semantyczne wymagają jednoznacznego API i konkretnych wejść.

Wybraliśmy funkcje, dla których łatwo porównać wynik przed i po obfuskacji.

Dla pozostałych próbek na razie mierzymy kompilowalność i cechy statyczne.

### Czy zero błędów semantycznych oznacza, że obfuskator jest w pełni poprawny?

Nie.

Oznacza to, że przeszedł obecny zestaw testów.

To jest silniejszy dowód niż sama kompilacja, ale nadal nie jest formalnym dowodem poprawności dla wszystkich możliwych programów.

### Dlaczego accuracy detektora progowego może być niskie?

Bo detektor używa tylko progów entropii.

Nie wszystkie techniki obfuskacji muszą mocno podnosić entropię.

Niektóre techniki bardziej zmieniają strukturę AST niż rozkład znaków.

Dlatego potrzebujemy detektora opartego na większej liczbie cech.

### Czy pełny pipeline zawsze jest najlepszy?

Nie zawsze.

Pełny pipeline daje silniejszą obfuskację, ale zwiększa rozmiar i złożoność kodu.

W praktyce trzeba dobrać techniki do celu: czasem wystarczy renaming, a czasem potrzebna jest mocniejsza kombinacja.
