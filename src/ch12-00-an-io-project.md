# Projekt I/O: Tworzenie programu wiersza poleceń

Ten rozdział to podsumowanie wielu umiejętności, których się do tej pory nauczyłeś, i
eksploracja kilku dodatkowych standardowych funkcji biblioteki. Zbudujemy narzędzie wiersza poleceń, które
współdziała z plikiem i wejściem/wyjściem wiersza poleceń, aby przećwiczyć niektóre
koncepcje Rust, które już masz w zanadrzu.

Szybkość, bezpieczeństwo, pojedyncze wyjście binarne i obsługa wielu platform sprawiają, że Rust
jest idealnym językiem do tworzenia narzędzi wiersza poleceń, więc w naszym projekcie
utworzymy własną wersję klasycznego narzędzia wyszukiwania wiersza poleceń `grep`
(**globalnie przeszukujemy **regularne **wyrażenie **e** i **p**rint). W
najprostszym przypadku użycia `grep` przeszukuje określony plik pod kątem określonego ciągu. Aby
to zrobić, `grep` przyjmuje jako argumenty ścieżkę pliku i ciąg. Następnie odczytuje
plik, znajduje w nim wiersze zawierające argument ciągu i drukuje
te wiersze.

Po drodze pokażemy, jak sprawić, by nasze narzędzie wiersza poleceń korzystało z funkcji terminala, z których korzysta wiele innych narzędzi wiersza poleceń. Odczytamy wartość zmiennej środowiskowej, aby umożliwić użytkownikowi skonfigurowanie zachowania naszego narzędzia.
Będziemy również drukować komunikaty o błędach do standardowego strumienia konsoli błędów (`stderr`)
zamiast do standardowego wyjścia (`stdout`), tak aby na przykład użytkownik mógł
przekierować pomyślne wyjście do pliku, jednocześnie widząc komunikaty o błędach na ekranie.

Jeden z członków społeczności Rust, Andrew Gallant, stworzył już w pełni
funkcjonalną, bardzo szybką wersję `grep`, zwaną `ripgrep`. Dla porównania, nasza
wersja będzie dość prosta, ale ten rozdział da ci pewną
wiedzę podstawową, której potrzebujesz, aby zrozumieć rzeczywisty projekt, taki jak
`ripgrep`.

Nasz projekt `grep` połączy szereg koncepcji, których nauczyłeś się do tej pory:

* Organizacja kodu ([Rozdział 7][ch7]<!-- ignore -->)
* Używanie wektorów i ciągów znaków ([Rozdział 8][ch8]<!-- ignore -->)
* Obsługa błędów ([Rozdział 9][ch9]<!-- ignore -->)
* Używanie cech i okresów życia, gdy jest to właściwe ([Rozdział 10][ch10]<!-- ignore -->)
* Pisanie testów ([Rozdział 11][ch11]<!-- ignore -->)

Krótko przedstawimy również zamknięcia, iteratory i obiekty cech, które
[Rozdział 13][ch13]<!-- ignore --> i [Rozdział 18][ch18]<!-- ignore --> zostaną
szczegółowo omówione.

[ch7]: ch07-00-managing-growing-projects-with-packages-crates-and-modules.html
[ch8]: ch08-00-common-collections.html
[ch9]: ch09-00-error-handling.html
[ch10]: ch10-00-generics.html
[ch11]: ch11-00-testing.html
[ch13]: ch13-00-functional-features.html
[ch18]: ch18-00-oop.html
