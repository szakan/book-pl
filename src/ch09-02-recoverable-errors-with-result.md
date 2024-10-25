## Odwracalne błędy z `Result`

Większość błędów nie jest na tyle poważna, aby program musiał się całkowicie zatrzymać.
Czasami, gdy funkcja zawodzi, dzieje się tak z powodu, który można łatwo zinterpretować i na który można zareagować. Na przykład, jeśli próbujesz otworzyć plik, a ta
operacja się nie powiedzie, ponieważ plik nie istnieje, możesz chcieć utworzyć plik,
zamiast kończyć proces.

Przypomnij sobie z [“Handling Potential Failure with `Result`”][handle_failure]<!--
ignore --> w rozdziale 2, że wyliczenie `Result` jest zdefiniowane jako mające dwa
warianty, `Ok` i `Err`, w następujący sposób:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` i `E` to parametry typu generycznego: omówimy je bardziej szczegółowo w rozdziale 10. Teraz musisz wiedzieć, że `T` reprezentuje
typ wartości, która zostanie zwrócona w przypadku powodzenia w wariancie `Ok`, a `E` reprezentuje typ błędu, który zostanie zwrócony w przypadku niepowodzenia w wariancie `Err`. Ponieważ `Result` ma te parametry typu generycznego, możemy użyć typu `Result` i funkcji zdefiniowanych w nim w wielu różnych sytuacjach, w których wartość powodzenia i wartość błędu, które chcemy
zwrócić, mogą się różnić.

Wywołajmy funkcję, która zwraca wartość `Result`, ponieważ funkcja może się
nie powieść. W Liście 9-3 próbujemy otworzyć plik.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-03/src/main.rs}}
```

<span class="caption">Listing 9-3: Opening a file</span>

Typem zwracanym przez `File::open` jest `Result<T, E>`. Ogólny parametr `T`
został wypełniony przez implementację `File::open` typem wartości
success, `std::fs::File`, która jest uchwytem pliku. Typem `E` używanym w
wartości błędu jest `std::io::Error`. Ten typ zwracany oznacza, że ​​wywołanie
`File::open` może się powieść i zwrócić uchwyt pliku, z którego możemy
odczytać lub do którego możemy
zapisać. Wywołanie funkcji może się również nie powieść: na przykład plik może nie
istnieć lub możemy nie mieć uprawnień dostępu do pliku. Funkcja `File::open`
musi mieć sposób, aby powiedzieć nam, czy powiodła się, czy nie, i jednocześnie
podawać nam uchwyt pliku lub informacje o błędzie. Te
informacje są dokładnie tym, co przekazuje wyliczenie `Result`.

W przypadku, gdy `File::open` się powiedzie, wartość w zmiennej
`greeting_file_result` będzie instancją `Ok`, która zawiera uchwyt pliku.
W przypadku, gdy się nie powiedzie, wartość w `greeting_file_result` będzie
instancją `Err`, która zawiera więcej informacji o rodzaju błędu, który
wystąpił.

Musimy dodać do kodu w Listingu 9-3, aby wykonać różne akcje w zależności
od wartości zwróconej przez `File::open`. Listing 9-4 pokazuje jeden ze sposobów obsługi
`Result` przy użyciu podstawowego narzędzia, wyrażenia `match`, które omówiliśmy w
Rozdziale 6.

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-04/src/main.rs}}
```

<span class="caption">Listing 9-4: Using a `match` expression to handle the
`Result` warianty, które mogą zostać zwrócone</span>

Należy zauważyć, że podobnie jak wyliczenie `Option`, wyliczenie `Result` i jego warianty zostały
wprowadzone do zakresu przez preludium, więc nie musimy określać `Result::`
przed wariantami `Ok` i `Err` w ramionach `match`.

Gdy wynikiem jest `Ok`, ten kod zwróci wewnętrzną wartość `file` z
wariantu `Ok`, a następnie przypiszemy tę wartość uchwytu pliku do zmiennej
`greeting_file`. Po `match` możemy użyć uchwytu pliku do odczytu lub
zapisu.

Drugie ramię `match` obsługuje przypadek, gdy otrzymujemy wartość `Err` z
`File::open`. W tym przykładzie wybraliśmy wywołanie makra `panic!`. Jeśli
w naszym bieżącym katalogu nie ma pliku o nazwie *hello.txt* i uruchomimy ten
kod, zobaczymy następujące dane wyjściowe z makra `panic!`:

```console
{{#include ../listings/ch09-error-handling/listing-09-04/output.txt}}
```

Jak zwykle, wynik ten dokładnie mówi nam, co poszło nie tak.

### Matching on Different Errors

Kod w Listingu 9-4 będzie `panic!`  niezależnie od przyczyny `File::open`.
Chcemy jednak podjąć różne działania w przypadku różnych przyczyn niepowodzenia: jeśli
`File::open` nie powiodło się, ponieważ plik nie istnieje, chcemy utworzyć plik
i zwrócić uchwyt do nowego pliku. Jeśli `File::open` nie powiodło się z jakiegokolwiek innego
powodu — na przykład dlatego, że nie mieliśmy uprawnień do otwarcia pliku — nadal chcemy, aby kod `panic!`  w taki sam sposób, jak w Listingu 9-4. W tym celu
dodajemy wewnętrzne wyrażenie `match` pokazane w Listingu 9-5.

<span class="filename">Filename: src/main.rs</span>

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-05/src/main.rs}}
```

<span class="caption">Listing 9-5: Obsługa różnych rodzajów błędów na różne sposoby</span>

Typ wartości, którą `File::open` zwraca wewnątrz wariantu `Err` to
`io::Error`, który jest strukturą dostarczaną przez bibliotekę standardową. Ta struktura
ma metodę `kind`, którą możemy wywołać, aby uzyskać wartość `io::ErrorKind`. Wyliczenie
`io::ErrorKind` jest dostarczane przez bibliotekę standardową i ma warianty
reprezentujące różne rodzaje błędów, które mogą wynikać z operacji `io`. Wariantem, którego chcemy użyć, jest `ErrorKind::NotFound`, który wskazuje,
że plik, który próbujemy otworzyć, jeszcze nie istnieje. Dlatego dopasowujemy na
`greeting_file_result`, ale mamy również wewnętrzne dopasowanie na `error.kind()`.

Warunek, który chcemy sprawdzić w wewnętrznym dopasowaniu, to czy wartość zwrócona
przez `error.kind()` jest wariantem `NotFound` wyliczenia `ErrorKind`. Jeśli tak,
próbujemy utworzyć plik za pomocą `File::create`. Jednak ponieważ `File::create`
również może się nie powieść, potrzebujemy drugiego ramienia w wewnętrznym wyrażeniu `match`. Gdy
plik nie może zostać utworzony, drukowany jest inny komunikat o błędzie. Drugie ramię
zewnętrznego `match` pozostaje takie samo, więc program panikuje przy każdym błędzie oprócz błędu braku pliku.

> ### Alternatives to Using `match` with `Result<T, E>`
>
> To sporo `match`! Wyrażenie `match` jest bardzo przydatne, ale też bardzo
> prymitywne. W rozdziale 13 dowiesz się o zamknięciach, które są używane
> z wieloma metodami zdefiniowanymi w `Result<T, E>`. Te metody mogą być
> bardziej zwięzłe niż używanie `match` podczas obsługi wartości `Result<T, E>` w kodzie.
>
> Na przykład oto inny sposób na zapisanie tej samej logiki, co pokazano w Liście
> 9-5, tym razem przy użyciu zamknięć i metody `unwrap_or_else`:
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore
> use std::fs::File;
> use std::io::ErrorKind;
>
> fn main() {
>     let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
>         if error.kind() == ErrorKind::NotFound {
>             File::create("hello.txt").unwrap_or_else(|error| {
>                 panic!("Problem creating the file: {:?}", error);
>             })
>         } else {
>             panic!("Problem opening the file: {:?}", error);
>         }
>     });
> }
> ```
>
> Chociaż ten kod zachowuje się tak samo jak Listing 9-5, nie zawiera
> żadnych wyrażeń `match` i jest czystszy w czytaniu. Wróć do tego przykładu
> po przeczytaniu rozdziału 13 i wyszukaj metodę `unwrap_or_else` w
> dokumentacji biblioteki standardowej. Wiele innych takich metod może oczyścić ogromne
> zagnieżdżone wyrażenia `match`, gdy masz do czynienia z błędami.

### Shortcuts for Panic on Error: `unwrap` and `expect`

Używanie `match` działa wystarczająco dobrze, ale może być trochę rozwlekłe i nie zawsze dobrze komunikuje intencję. Typ `Result<T, E>` ma wiele metod pomocniczych,
zdefiniowanych w celu wykonywania różnych, bardziej szczegółowych zadań. Metoda `unwrap` jest
metodą skrótu zaimplementowaną tak jak wyrażenie `match`, które napisaliśmy w
Listingu 9-4. Jeśli wartość `Result` jest wariantem `Ok`, `unwrap` zwróci
wartość wewnątrz `Ok`. Jeśli `Result` jest wariantem `Err`, `unwrap` wywoła
makro `panic!` za nas. Oto przykład `unwrap` w akcji:

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-04-unwrap/src/main.rs}}
```

Jeśli uruchomimy ten kod bez pliku *hello.txt*, zobaczymy komunikat o błędzie z wywołania `panic!`, które wykonuje metoda `unwrap`:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-04-unwrap
cargo run
copy and paste relevant text
-->

```text
wątek 'main' spanikował przy 'wywołaniu `Result::unwrap()` na `Err` wartość: Os {
code: 2, kind: NotFound, message: "Brak takiego pliku lub katalogu" }',
src/main.rs:4:49
```

Podobnie, metoda `expect` pozwala nam również wybrać komunikat o błędzie `panic!`.
Użycie `expect` zamiast `unwrap` i podanie dobrych komunikatów o błędach może przekazać
twoje intencje i ułatwić namierzenie źródła paniki. Składnia
`expect` wygląda następująco:

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-05-expect/src/main.rs}}
```

Używamy `expect` w ten sam sposób co `unwrap`: aby zwrócić uchwyt pliku lub wywołać
makro `panic!`. Komunikat o błędzie używany przez `expect` w wywołaniu `panic!`
będzie parametrem, który przekażemy do `expect`, a nie domyślnym komunikatem
`panic!` używanym przez `unwrap`. Oto jak to wygląda:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-05-expect
cargo run
copy and paste relevant text
-->

```text
wątek 'main' spanikował, gdy 'hello.txt powinien zostać uwzględniony w tym projekcie: Os {
kod: 2, rodzaj: NotFound, komunikat: "Brak takiego pliku lub katalogu" }',
src/main.rs:5:10
```

W kodzie o jakości produkcyjnej większość Rustaceanów wybiera `expect` zamiast
`unwrap` i podaje więcej kontekstu, dlaczego operacja ma się zawsze
udawać. W ten sposób, jeśli twoje założenia okażą się błędne, będziesz mieć więcej
informacji do wykorzystania w debugowaniu.

### Propagating Errors

Gdy implementacja funkcji wywołuje coś, co może się nie powieść, zamiast
obsługiwać błąd w samej funkcji, możesz zwrócić błąd do
kodu wywołującego, aby mógł on zdecydować, co zrobić. Jest to znane jako *rozprzestrzenianie*
błędu i daje większą kontrolę kodowi wywołującemu, gdzie może być więcej
informacji lub logiki, która dyktuje, jak błąd powinien być obsłużony, niż to, co
masz dostępne w kontekście swojego kodu.

Na przykład Listing 9-6 pokazuje funkcję, która odczytuje nazwę użytkownika z pliku. Jeśli
plik nie istnieje lub nie można go odczytać, ta funkcja zwróci te błędy
do kodu, który wywołał funkcję.

<span class="filename">Filename: src/main.rs</span>

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-06/src/main.rs:here}}
```

<span class="caption">Wylistowanie 9-6: Funkcja, która zwraca błędy do kodu wywołującego za pomocą `match`</span>

Tę funkcję można napisać w znacznie krótszy sposób, ale zaczniemy od
wykonania dużej części ręcznie, aby zbadać obsługę błędów; na końcu
pokażemy krótszą drogę. Przyjrzyjmy się typowi zwracanemu funkcji,
najpierw: `Result<String, io::Error>`. Oznacza to, że funkcja zwraca
wartość typu `Result<T, E>`, gdzie parametr generyczny `T` został
wypełniony konkretnym typem `String`, a typ generyczny `E` został
wypełniony konkretnym typem `io::Error`.

Jeśli ta funkcja zakończy się powodzeniem bez żadnych problemów, kod, który ją
wywołuje, otrzyma wartość `Ok`, która zawiera `String`—nazwę użytkownika, którą ta
funkcja odczytała z pliku. Jeśli ta funkcja napotka jakiekolwiek problemy,
kod wywołujący otrzyma wartość `Err`, która zawiera wystąpienie `io::Error`,
które zawiera więcej informacji o tym, jakie były problemy. Wybraliśmy
`io::Error` jako typ zwracany tej funkcji, ponieważ jest to
typ wartości błędu zwracanej przez obie operacje wywoływane w
treści tej funkcji, które mogą się nie powieść: funkcja `File::open` i metoda
`read_to_string`.

Treść funkcji rozpoczyna się od wywołania funkcji `File::open`. Następnie
obsługujemy wartość `Result` za pomocą `match` podobnego do `match` w Liście 9-4.
Jeśli `File::open` się powiedzie, uchwyt pliku w zmiennej wzorca `file`
staje się wartością w zmiennej zmiennej `username_file` i funkcja
kontynuuje działanie. W przypadku `Err` zamiast wywoływać `panic!`, używamy słowa kluczowego `return`,
aby całkowicie wyjść wcześniej z funkcji i przekazać wartość błędu
z `File::open`, teraz w zmiennej wzorca `e`, z powrotem do kodu wywołującego jako
wartość błędu tej funkcji.

Tak więc jeśli mamy uchwyt pliku w `username_file`, funkcja tworzy nowy
`String` w zmiennej `username` i wywołuje metodę `read_to_string` na
uchwycie pliku w `username_file`, aby odczytać zawartość pliku do
`username`. Metoda `read_to_string` zwraca również `Result`,
ponieważ może się nie powieść, nawet jeśli `File::open` zakończyło się powodzeniem. Potrzebujemy więc kolejnego `match`, aby
obsłużyć ten `Result`: jeśli `read_to_string` się powiedzie, nasza funkcja
powiodła się i zwracamy nazwę użytkownika z pliku, który jest teraz w `username`
zawiniętego w `Ok`. Jeśli `read_to_string` się nie powiedzie, zwracamy wartość błędu w taki sam sposób, w jaki zwróciliśmy wartość błędu w `match`, który obsłużył
wartość zwracaną przez `File::open`. Nie musimy jednak wyraźnie mówić
`return`, ponieważ jest to ostatnie wyrażenie w funkcji.

Kod wywołujący ten kod obsłuży następnie pobranie wartości `Ok`,
która zawiera nazwę użytkownika lub wartości `Err`, która zawiera `io::Error`. To
od wywołującego kodu zależy, co zrobić z tymi wartościami. Jeśli wywołujący
kod otrzyma wartość `Err`, może wywołać `panic!` i spowodować awarię programu, użyć
domyślnej nazwy użytkownika lub wyszukać nazwę użytkownika w innym miejscu niż plik, na przykład. Nie mamy wystarczających informacji o tym, co wywołujący kod faktycznie
próbuje zrobić, więc propagujemy wszystkie informacje o powodzeniu lub błędzie w górę, aby
obsłużył je odpowiednio.

Ten wzór propagacji błędów jest tak powszechny w Rust, że Rust udostępnia
operator znaku zapytania `?`, aby to ułatwić.

#### A Shortcut for Propagating Errors: the `?` Operator

Listing 9-7 shows an implementation of `read_username_from_file` that has the
same functionality as in Listing 9-6, but this implementation uses the
`?` operator.

<span class="filename">Filename: src/main.rs</span>

<!-- Celowo nie użyto rustdoc_include tutaj; funkcja `main` w pliku
panicuje. Chcemy ją uwzględnić w celach eksperymentalnych dla czytelników, ale
nie chcemy jej uwzględniać w celach testowych rustdoc. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-07/src/main.rs:here}}
```

<span class="caption">Wylistowanie 9-7: Funkcja, która zwraca błędy do kodu wywołującego za pomocą operatora `?` </span>
Znak `?` umieszczony po wartości `Result` jest zdefiniowany tak, aby działał niemal tak samo, jak wyrażenia `match`, które zdefiniowaliśmy do obsługi wartości `Result` w Liście
9-6. Jeśli wartością `Result` jest `Ok`, wartość wewnątrz `Ok` zostanie
zwrócona z tego wyrażenia, a program będzie kontynuowany. Jeśli wartością
jest `Err`, `Err` zostanie zwrócone z całej funkcji, tak jakbyśmy
użyli słowa kluczowego `return`, więc wartość błędu zostanie rozpropagowana do kodu wywołującego.

Istnieje różnica między tym, co robi wyrażenie `match` z Listingu 9-6,
a tym, co robi operator `?`: wartości błędów, na których wywołano operator `?`,
przechodzą przez funkcję `from`, zdefiniowaną w cesze `From` w bibliotece
standardowej, która służy do konwersji wartości z jednego typu na inny.
Gdy operator `?` wywołuje funkcję `from`, otrzymany typ błędu jest
konwertowany na typ błędu zdefiniowany w typie zwracanym bieżącej
funkcji. Jest to przydatne, gdy funkcja zwraca jeden typ błędu, aby reprezentować
wszystkie sposoby, w jakie funkcja może zawieść, nawet jeśli części mogą zawieść z wielu różnych
powodów.

Na przykład moglibyśmy zmienić funkcję `read_username_from_file` w Listingu
9-7, aby zwracała niestandardowy typ błędu o nazwie `OurError`, który definiujemy. Jeśli również
zdefiniujemy `impl From<io::Error> dla OurError`, aby skonstruować wystąpienie
`OurError` z `io::Error`, wówczas operator `?` wywołany w ciele
`read_username_from_file` wywoła `from` i przekonwertuje typy błędów bez
konieczności dodawania żadnego dodatkowego kodu do funkcji.

W kontekście Listingu 9-7 `?` na końcu wywołania `File::open`
zwróci wartość wewnątrz `Ok` do zmiennej `username_file`. Jeśli wystąpi błąd, operator `?` zwróci wcześniej całą funkcję i przekaże
dowolną wartość `Err` do kodu wywołującego. To samo dotyczy `?` na
końcu wywołania `read_to_string`.

Operator `?` eliminuje wiele szablonów i upraszcza implementację tej funkcji. Możemy nawet skrócić ten kod jeszcze bardziej, łącząc łańcuchowo
wywołania metod bezpośrednio po `?`, jak pokazano na Listingu 9-8.

<span class="filename">Filename: src/main.rs</span>

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-08/src/main.rs:here}}
```

<span class="caption">Wylistowanie 9-8: Łańcuchowe wywołania metod po operatorze `?`</span>
Przenieśliśmy tworzenie nowego `String` w `username` na początek
funkcji; ta część się nie zmieniła. Zamiast tworzyć zmienną
`username_file`, połączyliśmy wywołanie `read_to_string` bezpośrednio z
wynikiem `File::open("hello.txt")?`. Nadal mamy `?` na końcu wywołania
`read_to_string` i nadal zwracamy wartość `Ok` zawierającą `username`,
gdy zarówno `File::open`, jak i `read_to_string` zakończą się powodzeniem, zamiast zwracać
błędy. Funkcjonalność jest taka sama jak w Listingu 9-6 i Listingu 9-7;
to po prostu inny, bardziej ergonomiczny sposób zapisu.

Listing 9-9 pokazuje sposób, aby to jeszcze skrócić, używając `fs::read_to_string`.

<span class="filename">Filename: src/main.rs</span>

<!-- Celowo nie użyto rustdoc_include tutaj; funkcja `main` w pliku
panicuje. Chcemy ją uwzględnić w celach eksperymentalnych dla czytelników, ale
nie chcemy jej uwzględniać w celach testowych rustdoc. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-09/src/main.rs:here}}
```

<span class="caption">Wykaz 9-9: Korzystanie `fs::read_to_string` zamiast
otwierać i następnie czytać plik</span>

Odczytanie pliku do ciągu jest dość powszechną operacją, więc standardowa
biblioteka udostępnia wygodną funkcję `fs::read_to_string`, która otwiera
plik, tworzy nowy `String`, odczytuje zawartość pliku, umieszcza zawartość
w tym `String` i zwraca go. Oczywiście, użycie `fs::read_to_string`
nie daje nam możliwości wyjaśnienia całej obsługi błędów, więc najpierw zrobiliśmy to
dłuższą drogą.

#### Where The `?` Operator Can Be Used

Operatora `?` można używać tylko w funkcjach, których typ zwracany jest zgodny
z wartością, dla której użyto `?`. Dzieje się tak, ponieważ operator `?` jest zdefiniowany
w celu wykonania wcześniejszego zwrotu wartości z funkcji, w taki sam sposób,
jak wyrażenie `match` zdefiniowane w Liście 9-6. W Liście 9-6
`match` używało wartości `Result`, a ramię wczesnego zwrotu zwróciło wartość
`Err(e)`. Typem zwracanym przez funkcję musi być `Result`, aby
była zgodna z tym `return`.

W Liście 9-10 przyjrzyjmy się błędowi, który otrzymamy, jeśli użyjemy operatora `?`
w funkcji `main` z typem zwracanym niezgodnym z typem wartości,
dla której używamy `?`:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-10/src/main.rs}}
```

<span class="caption">Wylistowanie 9-10: Próba użycia `?` w funkcji `main`, która zwraca `()`, nie zostanie skompilowana</span>

Ten kod otwiera plik, co może się nie powieść. Operator `?` następuje po wartości `Result`
zwróconej przez `File::open`, ale ta funkcja `main` ma typ zwracany
`()`, nie `Result`. Kiedy kompilujemy ten kod, otrzymujemy następujący komunikat o błędzie:
```console
{{#include ../listings/ch09-error-handling/listing-09-10/output.txt}}
```

Ten błąd wskazuje, że operatora `?` możemy używać tylko w
funkcji, która zwraca `Result`, `Option` lub inny typ implementujący
`FromResidual`.

Aby naprawić błąd, masz dwie możliwości. Jedną z nich jest zmiana typu zwracanego
funkcji, aby był zgodny z wartością, dla której używasz operatora `?`,
o ile nie ma żadnych ograniczeń, które by to uniemożliwiały. Inną techniką jest
użycie `match` lub jednej z metod `Result<T, E>` do obsługi `Result<T,
E>` w odpowiedni sposób.

Komunikat o błędzie wspominał również, że `?` można używać z wartościami `Option<T>`. Podobnie jak w przypadku używania `?` w `Result`, możesz używać `?` tylko w `Option` w
funkcji, która zwraca `Option`. Zachowanie operatora `?` wywołanego
na `Option<T>` jest podobne do jego zachowania wywołanego na `Result<T, E>`:
jeśli wartością jest `None`, `None` zostanie zwrócone wcześniej z funkcji w tym
momencie. Jeśli wartością jest `Some`, wartość wewnątrz `Some` jest
wartością wynikową wyrażenia, a funkcja jest kontynuowana. Listing 9-11 zawiera
przykład funkcji, która znajduje ostatni znak pierwszego wiersza w
danym tekście:

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-11/src/main.rs:here}}
```

<span class="caption">Listing 9-11: Using the `?` operator on an `Option<T>`
value</span>

Ta funkcja zwraca `Option<char>`, ponieważ możliwe, że jest tam
znak, ale możliwe też, że go nie ma. Ten kod przyjmuje argument wycinka ciągu
`text` i wywołuje na nim metodę `lines`, która zwraca
iterator po wierszach w ciągu. Ponieważ ta funkcja chce
sprawdzić pierwszy wiersz, wywołuje `next` na iteratorze, aby uzyskać pierwszą wartość
z iteratora. Jeśli `text` jest pustym ciągiem, to wywołanie `next`
zwróci `None`, w takim przypadku używamy `?`, aby zatrzymać się i zwrócić `None` z
`last_char_of_first_line`. Jeśli `text` nie jest pustym ciągiem, `next`
zwróci wartość `Some` zawierającą wycinek ciągu z pierwszego wiersza w `text`.

`?` wyodrębnia wycinek ciągu i możemy wywołać `chars` na tym wycinku ciągu,
aby uzyskać iterator jego znaków. Interesuje nas ostatni znak w
tym pierwszym wierszu, więc wywołujemy `last`, aby zwrócić ostatni element w iteratorze.
To jest `Option`, ponieważ możliwe jest, że pierwszy wiersz jest pustym
ciągiem, na przykład jeśli `text` zaczyna się od pustego wiersza, ale ma znaki w
innych wierszach, jak w `"\nhi"`. Jednak jeśli w pierwszym wierszu znajduje się ostatni znak, zostanie on zwrócony w wariancie `Some`. Operator `?` w środku
daje nam zwięzły sposób wyrażenia tej logiki, pozwalając nam zaimplementować
funkcję w jednym wierszu. Gdybyśmy nie mogli użyć operatora `?` w `Option`, musielibyśmy
zaimplementować tę logikę za pomocą większej liczby wywołań metod lub wyrażenia `match`.

Należy zauważyć, że można użyć operatora `?` w `Result` w funkcji, która zwraca
`Result`, i można użyć operatora `?` w `Option` w funkcji, która
zwraca `Option`, ale nie można mieszać i dopasowywać. Operator `?` nie
automatycznie przekonwertuje `Result` na `Option` lub odwrotnie; w takich przypadkach,
możesz użyć metod takich jak `ok` na `Result` lub `ok_or` na
`Option`, aby wykonać konwersję jawnie.

Do tej pory wszystkie używane przez nas funkcje `main` zwracają `()`. Funkcja `main` jest
specjalna, ponieważ jest punktem wejścia i wyjścia programów wykonywalnych, a istnieją
ograniczenia co do tego, jaki może być jej typ zwracany, aby programy zachowywały się zgodnie z
oczekiwaniami.

Na szczęście `main` może również zwrócić `Result<(), E>`. Listing 9-12 zawiera
kod z Listingu 9-10, ale zmieniliśmy typ zwracany `main` na
`Result<(), Box<dyn Error>>` i dodaliśmy wartość zwracaną `Ok(())` na końcu. Ten
kod zostanie teraz skompilowany:

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-12/src/main.rs}}
```

<span class="caption">Wylistowanie 9-12: Zmiana `main` na `Result<(), E>`
umożliwia użycie operatora `?` na wartościach `Result`</span>

Typ `Box<dyn Error>` to *obiekt cechy*, o którym porozmawiamy w sekcji
[„Używanie obiektów cech, które zezwalają na wartości różnych
typów”][obiekty-cech]<!-- ignoruj ​​--> w rozdziale 17. Na razie możesz
czytać `Box<dyn Error>` jako „dowolny rodzaj błędu”. Używanie `?` w wartości `Result`
w funkcji `main` z typem błędu `Box<dyn Error>` jest dozwolone,
ponieważ pozwala na wcześniejsze zwrócenie dowolnej wartości `Err`. Mimo że ciało
tej funkcji `main` będzie zawsze zwracać tylko błędy typu `std::io::Error`, przez
określenie `Box<dyn Error>` ten podpis będzie nadal poprawny, nawet jeśli
do ciała `main` zostanie dodany więcej kodu, który zwraca inne błędy.

Gdy funkcja `main` zwróci `Result<(), E>`, plik wykonywalny
zakończy działanie z wartością `0`, jeśli `main` zwróci `Ok(())` i zakończy działanie z wartością
niezerową, jeśli `main` zwróci wartość `Err`. Pliki wykonywalne napisane w C zwracają
liczby całkowite po zakończeniu: programy, które kończą się pomyślnie, zwracają liczbę całkowitą
`0`, a programy, które kończą się błędnie, zwracają liczbę całkowitą inną niż `0`. Rust również
zwraca liczby całkowite z plików wykonywalnych, aby były zgodne z tą konwencją.

Funkcja `main` może zwracać dowolne typy, które implementują [the
`std::process::Termination` trait][termination]<!-- ignore -->, która zawiera
funkcję `report` zwracającą `ExitCode`. Zapoznaj się ze standardową dokumentacją biblioteki, aby uzyskać więcej informacji na temat implementacji cechy `Termination` dla
własnych typów.

Teraz, gdy omówiliśmy szczegóły wywołania `panic!` lub zwrócenia `Result`,
wróćmy do tematu, jak zdecydować, co jest właściwe do użycia w jakich
przypadkach.

[handle_failure]: ch02-00-guessing-game-tutorial.html#obsługa-potencjalnych-błędów-z-użyciem-result
[trait-objects]: ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[termination]: ../std/process/trait.Termination.html
