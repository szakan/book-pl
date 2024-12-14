## Developing the Library’s Functionality with Test-Driven Development

Teraz, gdy wyodrębniliśmy logikę do *src/lib.rs* i pozostawiliśmy argument
zbierania i obsługi błędów w *src/main.rs*, znacznie łatwiej jest pisać testy
dla podstawowej funkcjonalności naszego kodu. Możemy wywoływać funkcje bezpośrednio z
różnymi argumentami i sprawdzać wartości zwracane bez konieczności wywoływania naszego pliku binarnego
z wiersza poleceń.

W tej sekcji dodamy logikę wyszukiwania do programu `minigrep`
używając procesu test-driven development (TDD) z następującymi krokami:

1. Napisz test, który się nie powiedzie i uruchom go, aby upewnić się, że nie powiedzie się z oczekiwanego
powodu.

2. Napisz lub zmodyfikuj tylko tyle kodu, aby nowy test przeszedł.

3. Przerób kod, który właśnie dodałeś lub zmieniłeś i upewnij się, że testy
nadal przechodzą.

4. Powtórz od kroku 1!

Chociaż jest to tylko jeden z wielu sposobów pisania oprogramowania, TDD może pomóc w kierowaniu
projektowaniem kodu. Napisanie testu przed napisaniem kodu, który sprawia, że ​​test przechodzi,
pomaga utrzymać wysoki poziom pokrycia testem w całym procesie.

Przetestujemy implementację funkcjonalności, która faktycznie wykona
wyszukiwanie ciągu zapytania w zawartości pliku i wygeneruje listę
wierszy pasujących do zapytania. Dodamy tę funkcjonalność w funkcji o nazwie
`search`.

### Writing a Failing Test

Ponieważ już ich nie potrzebujemy, usuńmy polecenia `println!` z plików
*src/lib.rs* i *src/main.rs*, których użyliśmy do sprawdzenia zachowania programu.
Następnie w *src/lib.rs* dodaj moduł `tests` z funkcją testową, tak jak zrobiliśmy to w
[Rozdział 11][ch11-anatomy]<!-- ignore -->. Funkcja testowa określa,
jakie zachowanie chcemy, aby miała funkcja `search`: przyjmie zapytanie i tekst,
który ma zostać przeszukany, i zwróci tylko wiersze z tekstu, które zawierają zapytanie. Wypis 12-15 pokazuje ten test, który jeszcze się nie skompiluje.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-15/src/lib.rs:here}}
```

<span class="caption">Listing 12-15: Tworzenie nieudanego testu dla funkcji `search`, którą chcielibyśmy mieć</span>

Ten test wyszukuje ciąg `"duct"`. Przeszukiwany tekst składa się z trzech
linii, z których tylko jedna zawiera `"duct"` (Należy zauważyć, że ukośnik odwrotny po
otwierającym podwójnym cudzysłowie mówi Rustowi, aby nie umieszczał znaku nowej linii na początku
zawartości tego literału ciągu). Twierdzimy, że wartość zwrócona przez
funkcję `search` zawiera tylko oczekiwaną przez nas linię.

Nie jesteśmy jeszcze w stanie uruchomić tego testu i obserwować, jak się nie powiedzie, ponieważ test nawet się nie
kompiluje: funkcja `search` jeszcze nie istnieje! Zgodnie z zasadami TDD dodamy tylko tyle kodu, aby test się skompilował i uruchomił,
dodając definicję funkcji `search`, która zawsze zwraca pusty
wektor, jak pokazano w Liście 12-16. Następnie test powinien się skompilować i zakończyć się niepowodzeniem,
ponieważ pusty wektor nie pasuje do wektora zawierającego linię `"safe,
fast, productivity."`

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-16/src/lib.rs:here}}
```

<span class="caption">Listing 12-16: Definiowanie wystarczającej ilości funkcji `search`, aby nasz test mógł zostać skompilowany</span>

Zauważ, że musimy zdefiniować jawny czas życia `'a` w sygnaturze
`search` i użyć tego czasu życia z argumentem `contents` i wartością
zwracaną. Przypomnijmy w [Rozdziale 10][ch10-lifetimes]<!-- ignore -->, że parametry czasu życia
określają, który czas życia argumentu jest połączony z czasem życia
wartości zwracanej. W tym przypadku wskazujemy, że zwrócony wektor powinien zawierać
wycinki ciągu, które odwołują się do wycinków argumentu `contents` (zamiast
argumentu `query`).

Innymi słowy, informujemy Rust, że dane zwrócone przez funkcję `search`
będą istnieć tak długo, jak dane przekazane do funkcji `search` w argumencie
`contents`. To jest ważne! Dane, do których odwołuje się ** wycinek,
muszą być prawidłowe, aby odwołanie było prawidłowe; jeśli kompilator zakłada, że ​​tworzymy
wycinki ciągu `query` zamiast `contents`, niepoprawnie przeprowadzi kontrolę bezpieczeństwa.

Jeśli zapomnimy adnotacji czasu życia i spróbujemy skompilować tę funkcję,
otrzymamy następujący błąd:

```console
{{#include ../listings/ch12-an-io-project/output-only-02-missing-lifetimes/output.txt}}
```

Rust nie może wiedzieć, którego z dwóch argumentów potrzebujemy, więc musimy powiedzieć
to wprost. Ponieważ `contents` jest argumentem, który zawiera cały nasz tekst,
a my chcemy zwrócić pasujące części tego tekstu, wiemy, że `contents` jest
argumentem, który powinien być połączony z wartością zwracaną za pomocą składni czasu życia.

Inne języki programowania nie wymagają łączenia argumentów w celu zwrócenia
wartości w sygnaturze, ale ta praktyka stanie się łatwiejsza z czasem. Możesz
porównać ten przykład z sekcją [„Sprawdzanie referencji za pomocą
Czasów życia”][validating-references-with-lifetimes]<!-- ignore --> w
rozdziale 10.

Teraz uruchommy test:

```console
{{#include ../listings/ch12-an-io-project/listing-12-16/output.txt}}
```

Świetnie, test nie powiódł się, dokładnie tak jak się spodziewaliśmy. Sprawmy, żeby test przeszedł!

### Writing Code to Pass the Test

Obecnie nasz test kończy się niepowodzeniem, ponieważ zawsze zwracamy pusty wektor. Aby to naprawić i zaimplementować `search`, nasz program musi wykonać następujące kroki:

* Przejść przez każdy wiersz zawartości.
* Sprawdzić, czy wiersz zawiera nasz ciąg zapytania.
* Jeśli tak, dodać go do listy zwracanych wartości.
* Jeśli nie, nic nie robić.
* Zwrócić listę pasujących wyników.

Przeanalizujmy każdy krok, zaczynając od przejrzenia przez wiersze.

#### Iterating Through Lines with the `lines` Method

Rust ma pomocną metodę obsługi iteracji stringów wiersz po wierszu, wygodnie nazwaną `lines`, która działa tak, jak pokazano na Listingu 12-17. Należy zauważyć, że ta metoda nie zostanie jeszcze skompilowana.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-17/src/lib.rs:here}}
```

<span class="caption">Listing 12-17: Iterowanie po każdym wierszu w `contents`
</span>

Metoda `lines` zwraca iterator. Omówimy iteratory szczegółowo w
[Rozdział 13][ch13-iterators]<!-- ignore -->, ale przypomnij sobie, że widziałeś ten sposób
używania iteratora w [Listing 3-5][ch3-iter]<!-- ignore -->, gdzie użyliśmy pętli
`for` z iteratorem, aby uruchomić kod na każdym elemencie kolekcji.

#### Searching Each Line for the Query

Następnie sprawdzimy, czy bieżący wiersz zawiera nasz ciąg zapytania.
Na szczęście ciągi mają pomocną metodę o nazwie `contains`, która robi to za
nas! Dodaj wywołanie metody `contains` w funkcji `search`, jak pokazano w
Listingu 12-18. Zauważ, że to nadal się nie skompiluje.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-18/src/lib.rs:here}}
```

<span class="caption">Listing 12-18: Dodawanie funkcjonalności sprawdzającej, czy
linia zawiera ciąg w `query`</span>

W tej chwili budujemy funkcjonalność. Aby ją skompilować, musimy
zwrócić wartość z treści, tak jak wskazaliśmy w sygnaturze funkcji.

#### Storing Matching Lines

Aby zakończyć tę funkcję, potrzebujemy sposobu na przechowywanie pasujących linii, które chcemy
zwrócić. W tym celu możemy utworzyć zmienny wektor przed pętlą `for` i
wywołać metodę `push`, aby zapisać `line` w wektorze. Po pętli `for`,
zwracamy wektor, jak pokazano w Liście 12-19.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:here}}
```

<span class="caption">Listing 12-19: Przechowywanie pasujących wierszy, abyśmy mogli je
zwrócić</span>

Teraz funkcja `search` powinna zwrócić tylko wiersze zawierające `query`,
a nasz test powinien przejść. Uruchommy test:

```console
{{#include ../listings/ch12-an-io-project/listing-12-19/output.txt}}
```

Nasz test przeszedł, więc wiemy, że działa!

W tym momencie moglibyśmy rozważyć możliwości refaktoryzacji
implementacji funkcji wyszukiwania, jednocześnie zachowując przechodzenie testów, aby
zachować tę samą funkcjonalność. Kod w funkcji wyszukiwania nie jest zły,
ale nie wykorzystuje niektórych przydatnych funkcji iteratorów.
Wrócimy do tego przykładu w [Rozdziale 13][ch13-iterators]<!-- ignore -->, gdzie
szczegółowo przyjrzymy się iteratorom i przyjrzymy się, jak je ulepszyć.

#### Using the `search` Function in the `run` Function

Teraz, gdy funkcja `search` działa i została przetestowana, musimy wywołać `search`
z naszej funkcji `run`. Musimy przekazać wartość `config.query` i
`zawartość`, którą `run` odczytuje z pliku, do funkcji `search`. Następnie `run`
wydrukuje każdy wiersz zwrócony przez `search`:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/src/lib.rs:here}}
```

Nadal używamy pętli `for`, aby zwrócić każdy wiersz z `search` i wydrukować go.

Teraz cały program powinien działać! Wypróbujmy to, najpierw ze słowem, które
powinno zwrócić dokładnie jeden wiersz z wiersza Emily Dickinson, “frog”:

```console
{{#include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/output.txt}}
```

Świetnie! Teraz spróbujmy słowa, które będzie pasowało do wielu linii, np. „body”:

```console
{{#include ../listings/ch12-an-io-project/output-only-03-multiple-matches/output.txt}}
```

I na koniec upewnijmy się, że nie otrzymamy żadnych wersów, gdy będziemy szukać słowa, którego nie ma nigdzie w wierszu, takiego jak „monomorfizacja”:

```console
{{#include ../listings/ch12-an-io-project/output-only-04-no-matches/output.txt}}
```

Doskonale! Zbudowaliśmy własną mini wersję klasycznego narzędzia i nauczyliśmy się wiele
o tym, jak strukturyzować aplikacje. Dowiedzieliśmy się również trochę o wejściu i wyjściu plików, czasach życia, testowaniu i analizie wiersza poleceń.

Aby zakończyć ten projekt, krótko pokażemy, jak pracować ze
zmiennymi środowiskowymi i jak drukować na standardowym błędzie, co jest
przydatne podczas pisania programów wiersza poleceń.

[validating-references-with-lifetimes]:
ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[ch11-anatomy]: ch11-01-writing-tests.html#the-anatomy-of-a-test-function
[ch10-lifetimes]: ch10-03-lifetime-syntax.html
[ch3-iter]: ch03-05-control-flow.html#przechodzenie-po-kolekcji-za-pomocą-for
[ch13-iterators]: ch13-02-iterators.html
