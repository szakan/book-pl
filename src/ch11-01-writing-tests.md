## Jak pisać testy

Testy to funkcje Rust, które weryfikują, czy kod nietestowy działa w oczekiwany sposób. Ciała funkcji testowych zazwyczaj wykonują te trzy czynności:

1. Skonfiguruj wszelkie potrzebne dane lub stan.
2. Uruchom kod, który chcesz przetestować.
3. Potwierdź, że wyniki są zgodne z oczekiwaniami.

Przyjrzyjmy się funkcjom, które Rust udostępnia specjalnie do pisania testów, które
wykonują te czynności, w tym atrybut `test`, kilka makr i atrybut
`should_panic`.

### Anatomia funkcji testowej

Najprościej rzecz ujmując, test w Rust to funkcja, która jest adnotowana atrybutem `test`. Atrybuty to metadane dotyczące fragmentów kodu Rust; jednym z przykładów jest
atrybut `derive`, którego użyliśmy ze strukturami w rozdziale 5. Aby zmienić funkcję
w funkcję testową, dodaj `#[test]` w wierszu przed `fn`. Gdy uruchamiasz
testy za pomocą polecenia `cargo test`, Rust tworzy binarny program uruchamiający testy, który uruchamia
adnotowane funkcje i raportuje, czy każda
funkcja testowa przechodzi, czy nie przechodzi.

Za każdym razem, gdy tworzymy nowy projekt biblioteki za pomocą Cargo, automatycznie generowany jest dla nas moduł testowy z funkcją testową. Ten moduł zapewnia
szablon do pisania testów, dzięki czemu nie musisz szukać dokładnej
struktury i składni za każdym razem, gdy rozpoczynasz nowy projekt. Możesz dodać tyle
dodatkowych funkcji testowych i tyle modułów testowych, ile chcesz!

Przyjrzymy się niektórym aspektom działania testów, eksperymentując z szablonem
test zanim faktycznie przetestujemy jakikolwiek kod. Następnie napiszemy kilka testów w świecie rzeczywistym, które wywołają jakiś napisany przez nas kod i potwierdzą, że jego zachowanie jest poprawne.

Utwórzmy nowy projekt biblioteki o nazwie `adder`, który będzie dodawał dwie liczby:

```console
$ cargo new adder --lib
    Utworzono projekt biblioteki `adder`
$ cd adder
```

Zawartość pliku *src/lib.rs* w bibliotece `adder` powinna wyglądać tak, jak na Listingu 11-1.

<span class="filename">Filename: src/lib.rs</span>

<!-- manual-regeneration
cd listings/ch11-writing-automated-tests
rm -rf listing-11-01
cargo new listing-11-01 --lib --name adder
cd listing-11-01
cargo test
git co output.txt
cd ../../..
-->

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

<span class="caption">Listing 11-1: Moduł testowy i funkcja wygenerowane
automatycznie przez `cargo new`</span>

Na razie zignorujmy dwie pierwsze linijki i skupmy się na funkcji. Zwróć uwagę na adnotację
`#[test]`: ten atrybut wskazuje, że jest to funkcja testowa, więc
program uruchamiający test wie, że ma traktować tę funkcję jako test. Możemy również mieć funkcje nietestowe w module `tests`, aby pomóc w skonfigurowaniu typowych scenariuszy lub wykonywaniu typowych operacji, więc zawsze musimy wskazać, które funkcje są testami.

Przykładowa treść funkcji używa makra `assert_eq!`, aby potwierdzić, że `result`,
który zawiera wynik dodania 2 i 2, jest równy 4. To potwierdzenie służy jako
przykład formatu typowego testu. Uruchommy je, aby sprawdzić, czy ten test
zalicza się.

Polecenie `cargo test` uruchamia wszystkie testy w naszym projekcie, jak pokazano na Liście
11-2.

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-01/output.txt}}
```

<span class="caption">Listing 11-2: Dane wyjściowe z uruchomienia automatycznie wygenerowanego testu</span>

Cargo skompilowało i uruchomiło test. Widzimy wiersz `running 1 test`. Następny
wiersz pokazuje nazwę wygenerowanej funkcji testowej, zwanej `it_works`, i że
wynik uruchomienia tego testu to `ok`. Ogólne podsumowanie `test result: ok.`
oznacza, że ​​wszystkie testy przeszły pomyślnie, a część, która brzmi `1 przeszedł pomyślnie; 0
nie przeszedł pomyślnie`, sumuje liczbę testów, które przeszły pomyślnie lub nie przeszły pomyślnie.

Można oznaczyć test jako zignorowany, aby nie był uruchamiany w określonym
wystąpieniu; omówimy to w sekcji  [“Running 
Podzbiór testów według nazwy”][subset]<!-- ignore --> section w dalszej części tego rozdziału. Ponieważ
tutaj tego nie zrobiliśmy, podsumowanie pokazuje `0 ignored`. Możemy również przekazać
argument do polecenia `cargo test`, aby uruchamiać tylko testy, których nazwa pasuje do
ciągu; nazywa się to *filtrowaniem* i omówimy to w sekcji [„Uruchamianie
podzbioru testów według nazwy”][podzbiór]<!-- ignoruj ​​-->. Nie
filtrowaliśmy również uruchamianych testów, więc koniec podsumowania pokazuje `0 odfiltrowanych`.

Statystyka `0 zmierzonych` dotyczy testów porównawczych mierzących wydajność.
Testy porównawcze są, w chwili pisania tego tekstu, dostępne tylko w nocnym Rust. Zobacz
[the documentation about benchmark tests][bench], aby dowiedzieć się więcej.

Kolejna część wyników testu zaczynająca się od `Doc-tests adder` dotyczy
wyników wszelkich testów dokumentacji. Nie mamy jeszcze żadnych testów dokumentacji,
ale Rust może kompilować dowolne przykłady kodu, które pojawiają się w naszej dokumentacji API.
Ta funkcja pomaga zachować synchronizację dokumentacji i kodu! Omówimy, jak
pisać testy dokumentacji w sekcji [“Dokumentacja Komentarze jako
Testy”][doc-comments]<!-- ignore --> rozdziału 14. Na razie
zignorujemy wynik `Doc-tests`.

Zacznijmy dostosowywać test do naszych własnych potrzeb. Najpierw zmieńmy nazwę
funkcji `it_works` na inną, taką jak `exploration`, w następujący sposób:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/src/lib.rs}}
```

Następnie uruchom ponownie `cargo test`. Teraz wynik pokazuje `exploration` zamiast
`it_works`:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/output.txt}}
```

Teraz dodamy kolejny test, ale tym razem zrobimy test, który się nie powiedzie! Testy
nie powiodą się, gdy coś w funkcji testowej wpadnie w panikę. Każdy test jest uruchamiany w nowym
wątku, a gdy wątek główny zauważy, że wątek testowy umarł, test jest
oznaczony jako nieudany. W rozdziale 9 mówiliśmy o tym, że najprostszym sposobem na panikę
jest wywołanie makra `panic!`. Wprowadź nowy test jako funkcję o nazwie
`another`, tak aby plik *src/lib.rs* wyglądał jak na Listingu 11-3.

<span class="filename">Filename: src/lib.rs</span>

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-03/src/lib.rs:here}}
```

<span class="caption">Listing 11-3: Dodawanie drugiego testu, który się nie powiedzie, ponieważ
wywołujemy makro `panic!`</span>

Uruchom testy ponownie, używając `cargo test`. Wynik powinien wyglądać jak w Listingu
11-4, który pokazuje, że nasz test `exploration` przeszedł, a `another` nie.

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-03/output.txt}}
```

<span class="caption">Listing 11-4: Wyniki testów, gdy jeden test został zaliczony, a drugi nie</span>

Zamiast `ok`, wiersz `test tests::another` pokazuje `FAILED`. Dwie nowe sekcje pojawiają się między poszczególnymi wynikami a podsumowaniem: pierwsza
wyświetla szczegółowy powód niepowodzenia każdego testu. W tym przypadku otrzymujemy
szczegóły, że `another` się nie powiódł, ponieważ `wyskoczył z paniki przy 'Make this test fail'` w
wierszu 10 w pliku *src/lib.rs*. Następna sekcja zawiera tylko nazwy wszystkich
nieudanych testów, co jest przydatne, gdy jest wiele testów i wiele
szczegółowych wyników nieudanych testów. Możemy użyć nazwy nieudanego testu, aby uruchomić tylko ten test, aby łatwiej go debugować; więcej o sposobach uruchamiania testów powiemy w
sekcji [“Kontrolowanie sposobu uruchamiania testów”][controlling-how-tests-are-run]<!-- ignore
-->.

Wiersz podsumowania wyświetla się na końcu: ogólnie rzecz biorąc, nasz wynik testu to `FAILED`. Mieliśmy
jeden test zaliczony i jeden nieudany.

Teraz, gdy już wiesz, jak wyglądają wyniki testów w różnych scenariuszach, przyjrzyjmy się innym makrom oprócz `panic!`, które są przydatne w testach.

### Sprawdzanie wyników za pomocą makra `assert!`

Makro `assert!`, dostarczane przez bibliotekę standardową, jest przydatne, gdy
chcesz się upewnić, że jakiś warunek w teście zostanie oceniony jako `true`. Podajemy makrze
`assert!` argument, który zostanie oceniony jako wartość logiczna. Jeśli wartość jest
`true`, nic się nie dzieje i test przechodzi. Jeśli wartość jest `false`, makro
`assert!` wywołuje `panic!`, aby spowodować niepowodzenie testu. Użycie makra `assert!`
pomaga nam sprawdzić, czy nasz kod działa w sposób, w jaki zamierzamy.

W rozdziale 5, Listing 5-15, użyliśmy struktury `Rectangle` i metody `can_hold`, które są powtórzone tutaj w Listing 11-5. Umieśćmy ten kod w pliku
*src/lib.rs*, a następnie napiszmy dla niego kilka testów, używając makra `assert!`.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-05/src/lib.rs:here}}
```

<span class="caption">Listing 11-5: Korzystanie ze struktury `Rectangle` i jej
metody `can_hold` z rozdziału 5</span>

Metoda `can_hold` zwraca wartość logiczną, co oznacza, że ​​jest to idealny przypadek użycia
makra `assert!`. W Liście 11-6 piszemy test, który ćwiczy metodę
`can_hold`, tworząc instancję `Rectangle` o szerokości 8 i
wysokości 7 i potwierdzając, że może ona pomieścić inną instancję `Rectangle`, która
ma szerokość 5 i wysokość 1.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-06/src/lib.rs:here}}
```

<span class="caption">Listing 11-6: Test dla `can_hold` sprawdzający, czy
większy prostokąt może rzeczywiście pomieścić mniejszy prostokąt</span>

Zwróć uwagę, że dodaliśmy nowy wiersz wewnątrz modułu `tests`: `use super::*;`.
Moduł `tests` jest zwykłym modułem, który przestrzega zwykłych reguł widoczności,
które omówiliśmy w rozdziale 7 w sekcji  [“Ścieżki odwoływania się do elementu w module
Drzewo”][paths-for-referring-to-an-item-in-the-module-tree]<!-- ignore -->
. Ponieważ moduł `tests` jest modułem wewnętrznym, musimy przenieść testowany
kod w module zewnętrznym do zakresu modułu wewnętrznego. Używamy tutaj
globu, więc wszystko, co zdefiniujemy w module zewnętrznym, jest dostępne dla tego modułu
`tests`.

Nazwaliśmy nasz test `larger_can_hold_smaller` i utworzyliśmy dwie instancje
`Rectangle`, których potrzebowaliśmy. Następnie wywołaliśmy makro `assert!` i
przekazaliśmy mu wynik wywołania `larger.can_hold(&smaller)`. To wyrażenie powinno
zwrócić `true`, więc nasz test powinien przejść. Przekonajmy się!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-06/output.txt}}
```

Przechodzi! Dodajmy kolejny test, tym razem potwierdzający, że mniejszy
prostokąt nie może pomieścić większego prostokąta:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/src/lib.rs:here}}
```

Ponieważ prawidłowym wynikiem funkcji `can_hold` w tym przypadku jest `false`,
musimy zanegować ten wynik przed przekazaniem go do makra `assert!`. W rezultacie nasz test zostanie zaliczony, jeśli `can_hold` zwróci `false`:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/output.txt}}
```

Dwa testy, które przechodzą! Teraz zobaczmy, co się stanie z wynikami naszych testów, gdy
wprowadzimy błąd do naszego kodu. Zmienimy implementację metody `can_hold`,
zastępując znak większości znakiem mniejszości, gdy
porównuje szerokości:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/src/lib.rs:here}}
```

Uruchomienie testów teraz wygeneruje następujące wyniki:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/output.txt}}
```

Nasze testy wykryły błąd! Ponieważ `larger.width` wynosi 8, a `smaller.width` wynosi
5, porównanie szerokości w `can_hold` zwraca teraz `false`: 8 nie jest
mniejsze niż 5.

### Testowanie równości za pomocą `assert_eq!` i `assert_ne!` Makra

Powszechnym sposobem weryfikacji funkcjonalności jest testowanie równości między wynikiem
testowanego kodu a wartością, której oczekujesz od kodu. Możesz
to zrobić za pomocą makra `assert!` i przekazać mu wyrażenie za pomocą operatora `==`. Jest to jednak tak powszechny test, że biblioteka standardowa
udostępnia parę makr—`assert_eq!` i `assert_ne!`—aby przeprowadzić ten test
wygodniej. Te makra porównują dwa argumenty pod kątem równości lub
nierówności. Wydrukują również dwie wartości, jeśli asercja
się nie powiedzie, co ułatwia zobaczenie *dlaczego* test się nie powiódł; odwrotnie, makro
`assert!` wskazuje tylko, że otrzymało wartość `false` dla wyrażenia `==`, nie drukując wartości, które doprowadziły do ​​wartości `false`.

W Liście 11-7 piszemy funkcję o nazwie `add_two`, która dodaje `2` do swojego parametru, a następnie testujemy tę funkcję za pomocą makra `assert_eq!`.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-07/src/lib.rs}}
```

<span class="caption">Listing 11-7: Testowanie funkcji `add_two` przy użyciu makra
`assert_eq!`</span>

Sprawdźmy czy przejdzie!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-07/output.txt}}
```

Przekazujemy `4` jako argument do `assert_eq!`, co jest równe wynikowi
wywołania `add_two(2)`. Wiersz tego testu to `test tests::it_adds_two ...
ok`, a tekst `ok` wskazuje, że nasz test przeszedł pomyślnie!

Wprowadźmy błąd do naszego kodu, aby zobaczyć, jak wygląda `assert_eq!`, gdy
się nie powiedzie. Zmień implementację funkcji `add_two`, aby zamiast tego dodać `3`:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/src/lib.rs:here}}
```

Uruchom testy ponownie:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/output.txt}}
```

Nasz test wykrył błąd! Test `it_adds_two` nie powiódł się, a komunikat informuje
nas, że nieudane twierdzenie to `` assertion failed: `(left == right)` ``
oraz jakie są wartości `left` i `right`. Ten komunikat pomaga nam rozpocząć
debugowanie: argument `left` wynosił `4`, ale argument `right`, w którym mieliśmy
`add_two(2)`, wynosił `5`. Możesz sobie wyobrazić, że byłoby to szczególnie pomocne,
gdy mamy wiele testów.

Należy zauważyć, że w niektórych językach i frameworkach testowych parametry funkcji
twierdzeń równości nazywane są `expected` i `actual`, a kolejność,
w której określamy argumenty, ma znaczenie. Jednak w Rust nazywane są `left` i
`right`, a kolejność, w której określamy oczekiwaną wartość i wartość,
którą generuje kod, nie ma znaczenia. Moglibyśmy zapisać asercję w tym teście jako
`assert_eq!(add_two(2), 4)`, co spowodowałoby taki sam komunikat o błędzie,
który wyświetla `` assertion failed: `(left == right)` ``.

Makro `assert_ne!` przejdzie, jeśli dwie podane przez nas wartości nie będą równe i
nie powiedzie się, jeśli będą równe. Ta makro jest najbardziej przydatna w przypadkach, gdy nie jesteśmy pewni,
jaka wartość *będzie*, ale wiemy, jaka wartość na pewno *nie powinna* być.
Na przykład, jeśli testujemy funkcję, która z pewnością zmieni swoje dane wejściowe
w jakiś sposób, ale sposób, w jaki dane wejściowe są zmieniane, zależy od dnia,
w którym uruchamiamy nasze testy, najlepszą rzeczą do potwierdzenia może być to, że
wyjście funkcji nie jest równe danym wejściowym.

Pod powierzchnią makra `assert_eq!` i `assert_ne!` używają odpowiednio operatorów
`==` i `!=`. Gdy asercje zawiodą, te makra drukują swoje
argumenty przy użyciu formatowania debugowania, co oznacza, że ​​porównywane wartości muszą
implementować cechy `PartialEq` i `Debug`. Wszystkie typy prymitywne i większość
standardowych typów bibliotecznych implementują te cechy. W przypadku struktur i wyliczeń, które
zdefiniujesz samodzielnie, musisz zaimplementować `PartialEq`, aby potwierdzić równość
tych typów. Musisz również zaimplementować `Debug`, aby wydrukować wartości, gdy
asercja się nie powiedzie. Ponieważ obie cechy są cechami pochodnymi, jak wspomniano w
Listingu 5-12 w rozdziale 5, zwykle jest to tak proste, jak dodanie adnotacji
`#[derive(PartialEq, Debug)]` do definicji struktury lub wyliczenia. Zobacz
Dodatek C,  [“Cechy pochodne,”][derivable-traits]<!-- ignore --> , aby uzyskać więcej
szczegółów na temat tych i innych cech pochodnych.

### Dodawanie niestandardowych komunikatów o błędach

Możesz również dodać niestandardową wiadomość, która zostanie wydrukowana z komunikatem o błędzie jako
opcjonalne argumenty dla makr `assert!`, `assert_eq!` i `assert_ne!`. Wszelkie
argumenty określone po wymaganych argumentach są przekazywane do makra
`format!` (omówionego w rozdziale 8 w sekcji [“Połączenie z operatorem `+`
lub makro `format!`”][concatenation-with-the--operator-or-the-format-macro]<!-- ignore -->
), dzięki czemu możesz przekazać ciąg formatu zawierający symbole zastępcze `{}` i
wartości, które mają znaleźć się w tych symbolach zastępczych. Niestandardowe wiadomości są przydatne do dokumentowania
znaczenia asercji; gdy test się nie powiedzie, będziesz mieć lepsze pojęcie,
na czym polega problem z kodem.

Na przykład, powiedzmy, że mamy funkcję, która wita ludzi po imieniu i chcemy sprawdzić, czy imię, które przekazujemy do funkcji, pojawi się w wynikach:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-05-greeter/src/lib.rs}}
```

Wymagania dla tego programu nie zostały jeszcze uzgodnione i jesteśmy
prawie pewni, że tekst `Hello` na początku powitania ulegnie zmianie.
Postanowiliśmy, że nie chcemy aktualizować testu, gdy wymagania się zmienią,
więc zamiast sprawdzać dokładną równość z wartością zwróconą przez funkcję
`greeting`, po prostu stwierdzimy, że dane wyjściowe zawierają tekst
parametru wejściowego.

Teraz wprowadźmy błąd do tego kodu, zmieniając `greeting` na wykluczenie
`name`, aby zobaczyć, jak wygląda domyślny błąd testu:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/src/lib.rs:here}}
```

Uruchomienie tego testu daje następujące wyniki:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/output.txt}}
```

Ten wynik wskazuje jedynie, że asercja się nie powiodła i w którym wierszu
asercja się znajduje. Bardziej użytecznym komunikatem o błędzie byłoby wydrukowanie wartości z funkcji
`greeting`. Dodajmy niestandardowy komunikat o błędzie złożony z ciągu formatującego z symbolem zastępczym wypełnionym rzeczywistą wartością, którą otrzymaliśmy z funkcji
`greeting`:

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/src/lib.rs:here}}
```

Teraz, gdy uruchomimy test, otrzymamy bardziej informacyjny komunikat o błędzie:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/output.txt}}
```

Możemy zobaczyć wartość, którą faktycznie otrzymaliśmy w wynikach testu, co pomoże nam debugować to, co się wydarzyło, a nie to, czego się spodziewaliśmy.

### Sprawdzanie paniki za pomocą `should_panic`

Oprócz sprawdzania wartości zwracanych, ważne jest sprawdzenie, czy nasz kod
obsługuje warunki błędów zgodnie z oczekiwaniami. Na przykład rozważ typ `Guess`,
który utworzyliśmy w rozdziale 9, Listing 9-13. Inny kod, który używa `Guess`,
zależy od gwarancji, że instancje `Guess` będą zawierać tylko wartości
od 1 do 100. Możemy napisać test, który zapewnia, że ​​próba utworzenia instancji
`Guess` z wartością spoza tego zakresu powoduje panikę.

Robimy to, dodając atrybut `should_panic` do naszej funkcji testowej. Test
zalicza się, jeśli kod wewnątrz funkcji powoduje panikę; test kończy się niepowodzeniem, jeśli kod
wewnątrz funkcji nie powoduje paniki.

Listing 11-8 pokazuje test, który sprawdza, czy warunki błędów `Guess::new`
występują wtedy, gdy się ich spodziewamy.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-08/src/lib.rs}}
```

<span class="caption">Listing 11-8: Testing that a condition will cause a
`panic!`</span>

Umieszczamy atrybut `#[should_panic]` po atrybucie `#[test]` i
przed funkcją testową, do której się on stosuje. Spójrzmy na wynik, gdy ten test
zaliczy:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-08/output.txt}}
```

Wygląda dobrze! Teraz wprowadźmy błąd do naszego kodu, usuwając warunek, że funkcja `new` wpadnie w panikę, jeśli wartość będzie większa niż 100:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/src/lib.rs:here}}
```

Gdy uruchomimy test z Listingu 11-8, zakończy się on niepowodzeniem:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/output.txt}}
```

W tym przypadku nie otrzymujemy zbyt pomocnej wiadomości, ale gdy przyjrzymy się funkcji testowej, zobaczymy, że jest ona oznaczona adnotacją `#[should_panic]`. Otrzymany błąd
oznacza, że ​​kod w funkcji testowej nie spowodował paniki.

Testy, które używają `should_panic` mogą być niedokładne. Test `should_panic`
zaliczyłby nawet, gdyby test panikował z innego powodu niż ten, którego
oczekiwaliśmy. Aby testy `should_panic` były bardziej precyzyjne, możemy dodać opcjonalny parametr
`expected` do atrybutu `should_panic`. System testowy
upewni się, że wiadomość o błędzie zawiera podany tekst. Na przykład,
rozważ zmodyfikowany kod dla `Guess` w Liście 11-9, gdzie `new` funkcja
panikuje z różnymi wiadomościami w zależności od tego, czy wartość jest za mała, czy
za duża.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-09/src/lib.rs:here}}
```

<span class="caption">Listing 11-9: Testowanie `panic!` z komunikatem paniki
zawierającym określony podciąg</span>

Ten test zostanie zaliczony, ponieważ wartość, którą umieściliśmy w parametrze `should_panic`
`expected` atrybutu, jest podciągiem komunikatu, z którym funkcja `Guess::new`
wywołuje panikę. Mogliśmy określić cały komunikat paniki, którego
oczekujemy, w tym przypadku byłoby to `Guess value must be less than or equal to
100, got 200`. To, co zdecydujesz się określić, zależy od tego, jak bardzo komunikat paniki
jest unikalny lub dynamiczny i jak precyzyjny ma być test. W tym
przypadku podciąg komunikatu paniki wystarczy, aby upewnić się, że kod w
funkcji testowej wykona przypadek `else if value > 100`.

Aby zobaczyć, co się stanie, gdy test `should_panic` z komunikatem `expected`
się nie powiedzie, ponownie wprowadźmy błąd do naszego kodu, zamieniając ciała bloków
`if value < 1` i `else if value > 100`:

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/src/lib.rs:here}}
```

Tym razem gdy uruchomimy test `should_panic`, zakończy się on niepowodzeniem:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/output.txt}}
```

Komunikat o błędzie wskazuje, że ten test rzeczywiście wpadł w panikę, jak się spodziewaliśmy,
ale komunikat o panice nie zawierał oczekiwanego ciągu `'Wartość zgadywanki musi być
mniejsza lub równa 100'`. Komunikat o panice, który otrzymaliśmy w tym przypadku, brzmiał
`Wartość zgadywanki musi być większa lub równa 1, otrzymano 200.` Teraz możemy zacząć
rozpoznawać, gdzie jest nasz błąd!

### Używanie `Result<T, E>` w testach

Wszystkie nasze testy do tej pory panikują, gdy się nie powiodą. Możemy również pisać testy, które używają
`Result<T, E>`! Oto test z Listingu 11-1, przepisany tak, aby używał `Result<T,
E>` i zwracał `Err` zamiast panikować:

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-10-result-in-tests/src/lib.rs}}
```

Funkcja `it_works` ma teraz typ zwracany `Result<(), String>`. W
treści funkcji, zamiast wywoływać makro `assert_eq!`, zwracamy
`Ok(())`, gdy test się powiedzie, i `Err` z `String` wewnątrz, gdy test się nie powiedzie.

Pisanie testów tak, aby zwracały `Result<T, E>`, umożliwia użycie operatora znaku zapytania w treści testów, co może być wygodnym sposobem pisania
testów, które powinny się nie powieść, jeśli jakakolwiek operacja w nich zwróci wariant `Err`.

Nie można używać adnotacji `#[should_panic]` w testach, które używają `Result<T,
E>`. Aby potwierdzić, że operacja zwraca wariant `Err`, *nie* używaj operatora znaku zapytania w wartości `Result<T, E>`. Zamiast tego użyj
`assert!(value.is_err())`.

Teraz, gdy znasz już kilka sposobów pisania testów, przyjrzyjmy się temu, co się dzieje,
kiedy uruchamiamy nasze testy i poznajmy różne opcje, których możemy użyć z `cargo
test`.

[Łączenie-za-pomocą-operatora--lub-makra-format]:
ch08-02-strings.html#Łączenie-za-pomocą-operatora--lub-makra-format
[bench]: ../unstable-book/library-features/test.html
[ignoring]: ch11-02-running-tests.html#ignoring-some-tests-unless-specifically-requested
[subset]: ch11-02-running-tests.html#running-a-subset-of-tests-by-name
[controlling-how-tests-are-run]:
ch11-02-running-tests.html#controlling-how-tests-are-run
[derivable-traits]: appendix-03-derivable-traits.html
[doc-comments]: ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests
[paths-for-referring-to-an-item-in-the-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
