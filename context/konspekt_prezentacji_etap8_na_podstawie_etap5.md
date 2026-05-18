# Konspekt prezentacji — Etap 8

## 1. Wprowadzenie

Dzień dobry.

W etapie 8 dodaliśmy do projektu adwersarialny obfuskator, czyli obfuskator, który nie tylko zaciemnia kod, ale próbuje zrobić to tak, żeby uniknąć wykrycia przez detektor ML.

Do tej pory mieliśmy klasyczne techniki obfuskacji i detektor ML, który miał rozpoznawać, czy kod wygląda na obfuskowany. W tym etapie chcieliśmy sprawdzić, czy taki detektor da się obejść, jeśli obfuskator może wygenerować kilka wariantów kodu i wybrać ten najmniej podejrzany.

## 2. Problem

Problem jest taki, że pełny pipeline obfuskacji bardzo mocno zmienia kod. Zwiększa jego rozmiar, zmienia strukturę i dodaje dużo cech, które model ML może łatwo rozpoznać jako obfuskację. Taki kod jest trudniejszy dla człowieka, ale jednocześnie bardziej charakterystyczny dla detektora.

## 3. Pomysł rozwiązania

Dlatego w etapie 8 zastosowaliśmy inne podejście. Obfuskator adwersarialny nie próbuje stworzyć najbardziej skomplikowanego kodu. Zamiast tego tworzy kilka możliwych wariantów i sprawdza, jak ocenia je detektor ML. Następnie wybiera wariant z najniższym prawdopodobieństwem obfuskacji.

To prawdopodobieństwo oznaczamy jako `p_obf`. Jeżeli `p_obf` jest powyżej około `0.5`, model traktuje kod jako obfuskowany. Jeżeli jest poniżej `0.5`, model traktuje kod jako czysty.

## 4. Porównywane warianty

W praktyce porównaliśmy trzy wersje programu `bubble_sort`.

Pierwsza wersja to kod oryginalny. Druga wersja to kod po pełnym pipeline obfuskacji. Trzecia wersja to kod po obfuskacji adwersarialnej.

## 5. Wyniki

Wyniki były następujące.

Kod oryginalny został zaklasyfikowany jako czysty. Miał `p_obf = 0.235` i rozmiar `240` znaków.

Pełny pipeline został zaklasyfikowany jako obfuskowany. Miał `p_obf = 0.995` i rozmiar `1011` znaków. To oznacza, że model był prawie pewny, że ten kod jest obfuskowany.

Wariant adwersarialny został zaklasyfikowany jako czysty. Miał `p_obf = 0.345` i rozmiar `251` znaków. Czyli mimo że kod został zmieniony, detektor ML nie uznał go za obfuskowany.

Najważniejszy wynik jest taki, że pełny pipeline został wykryty, a wariant adwersarialny nie został wykryty.

W najlepszej próbie adwersarialnej użyta została lekka transformacja `mba_low`. Wynik tej próby to `p_obf = 0.3450`, decyzja `clean` i rozmiar `251` znaków.

## 6. Interpretacja wyniku

To pokazuje, że mocniejsza obfuskacja nie zawsze jest lepsza, jeżeli celem jest uniknięcie detektora ML. W tym przypadku mała, ostrożna transformacja była skuteczniejsza niż pełny pipeline, bo nie podniosła cech statycznych kodu do poziomu typowego dla obfuskacji.

## 7. Sprawdzenie poprawności

Sprawdziliśmy też poprawność wyniku. Samo obejście detektora nie wystarcza, jeśli kod przestaje działać. Dlatego sprawdziliśmy, czy kod po obfuskacji adwersarialnej nadal się kompiluje oraz czy funkcja `bubble_sort` zwraca takie same wyniki jak przed obfuskacją.

Oba warunki zostały spełnione. Kod był poprawny składniowo i zachował semantykę funkcji.

## 8. Wnioski

Wniosek pierwszy jest taki, że detektor ML dobrze wykrywa pełny pipeline, bo pełny pipeline bardzo mocno zmienia kod.

Wniosek drugi jest taki, że obfuskacja adwersarialna może ominąć detektor, jeśli wybiera wariant o niskim `p_obf`.

Wniosek trzeci jest taki, że detektor ML trzeba testować nie tylko na standardowych technikach obfuskacji, ale też na próbkach tworzonych specjalnie przeciwko niemu.

Wniosek czwarty jest taki, że metody z etapu 5 nadal są potrzebne. Po każdej transformacji trzeba sprawdzać nie tylko decyzję detektora, ale też kompilowalność i zgodność semantyczną.

## 9. Podsumowanie

Podsumowując: w etapie 8 pokazaliśmy, że model ML może skutecznie wykrywać klasyczną obfuskację, ale może być podatny na warianty adwersarialne. Dlatego kolejnym krokiem byłoby rozszerzenie zbioru treningowego o takie próbki i sprawdzanie odporności modelu na kod tworzony specjalnie po to, żeby zmylić detektor.
