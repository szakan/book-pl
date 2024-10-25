## Zaawansowane typy

System typów Rust ma pewne cechy, o których do tej pory wspomnieliśmy, ale których jeszcze nie omówiliśmy. Zaczniemy od omówienia nowych typów w ogólności, badając, dlaczego nowe typy są przydatne jako typy. Następnie przejdziemy do aliasów typów, cechy podobnej do nowych typów, ale o nieco innej semantyce. Omówimy również typ `!` i typy o dynamicznym rozmiarze.

### Using the Newtype Pattern for Type Safety and Abstraction

> Uwaga: Ta sekcja zakłada, że ​​przeczytałeś wcześniejszą sekcję [„Używanie
> wzorca Newtype do implementacji cech zewnętrznych w
> typach zewnętrznych.”][using-the-newtype-pattern]<!-- ignore -->

Wzorzec newtype jest również przydatny do zadań wykraczających poza te, które omówiliśmy do tej pory, w tym do statycznego wymuszania, aby wartości nigdy nie były mylone, oraz
wskazywania jednostek wartości. Przykład użycia newtypes do
wskazania jednostek widziałeś w Liście 20-15: przypomnij sobie, że struktury `Millimeters` i `Meters`
opakowały wartości `u32` w newtype. Gdybyśmy napisali funkcję z
parametrem typu `Millimeters`, nie moglibyśmy skompilować programu, który
przypadkowo próbowałby wywołać tę funkcję z wartością typu `Meters` lub
zwykłego `u32`.

Możemy również użyć wzorca newtype, aby oddzielić niektóre szczegóły implementacji
typu: nowy typ może ujawnić publiczny interfejs API, który różni się od
interfejsu API prywatnego typu wewnętrznego.

Nowe typy mogą również ukrywać wewnętrzną implementację. Na przykład moglibyśmy dostarczyć typ
`People`, aby opakować `HashMap<i32, String>`, który przechowuje identyfikator osoby
skojarzony z jej imieniem. Kod używający `People` będzie współdziałał tylko z
publicznym interfejsem API, który udostępniamy, takim jak metoda dodawania ciągu nazw do kolekcji `People`; ten kod nie musi wiedzieć, że przypisujemy identyfikator `i32` do nazw
wewnętrznie. Wzorzec newtype to lekki sposób na osiągnięcie enkapsulacji,
aby ukryć szczegóły implementacji, co omówiliśmy w sekcji [„Enkapsulacja, która
ukrywa szczegóły
implementacji”][encapsulation-that-hides-implementation-details]<!-- ignore -->
rozdziału 18.

### Creating Type Synonyms with Type Aliases

Rust umożliwia deklarowanie *aliasu typu*, aby nadać istniejącemu typowi
inną nazwę. W tym celu używamy słowa kluczowego `type`. Na przykład możemy utworzyć
alias `Kilometers` dla `i32` w następujący sposób:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:here}}
```

Teraz alias `Kilometers` jest *synonimem* dla `i32`; w przeciwieństwie do typów `Milimeters`
i `Meters`, które utworzyliśmy w Listingu 20-15, `Kilometers` nie jest oddzielnym,
nowym typem. Wartości, które mają typ `Kilometers` będą traktowane tak samo jak
wartości typu `i32`:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:there}}
```

Ponieważ `Kilometers` i `i32` są tego samego typu, możemy dodawać wartości obu
typów i możemy przekazywać wartości `Kilometers` do funkcji, które przyjmują parametry `i32`. Jednak stosując tę ​​metodę, nie otrzymujemy korzyści ze sprawdzania typów,
które otrzymujemy ze wzorca newtype omówionego wcześniej. Innymi słowy, jeśli
gdzieś pomylimy wartości `Kilometers` i `i32`, kompilator nie zgłosi nam
błędu.

Głównym przypadkiem użycia synonimów typów jest redukcja powtórzeń. Na przykład, możemy
mieć długi typ taki jak ten:

```rust,ignore
Box<dyn Fn() + Send + 'static>
```

Pisanie tego długiego typu w sygnaturach funkcji i jako adnotacje typu w całym kodzie może być męczące i podatne na błędy. Wyobraź sobie projekt pełen kodu takiego jak w Listingu 20-24.

<Listing number="20-24" caption="Using a long type in many places">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-24/src/main.rs:here}}
```

</Listing>

Alias ​​typu sprawia, że ​​ten kod jest bardziej zarządzalny, ponieważ zmniejsza powtórzenia. W
Listingu 20-25 wprowadziliśmy alias o nazwie `Thunk` dla typu verbose i
możemy zastąpić wszystkie użycia typu krótszym aliasem `Thunk`.

<Listing number="20-25" caption="Introducing a type alias `Thunk` to reduce repetition">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-25/src/main.rs:here}}
```

</Listing>

Ten kod jest o wiele łatwiejszy do odczytania i napisania! Wybór znaczącej nazwy dla
aliasu typu może również pomóc w przekazaniu intencji (*thunk* to słowo oznaczające kod,
który ma zostać oceniony później, więc jest to odpowiednia nazwa dla zamknięcia, które
zostaje zapisane).

Aliasy typów są również powszechnie używane z typem `Result<T, E>` w celu zmniejszenia
powtarzania. Rozważ moduł `std::io` w bibliotece standardowej. Operacje
we/wy często zwracają `Result<T, E>` w celu obsługi sytuacji, gdy operacje
nie działają. Ta biblioteka ma strukturę `std::io::Error`, która reprezentuje wszystkie
możliwe błędy we/wy. Wiele funkcji w `std::io` będzie zwracać
`Result<T, E>`, gdzie `E` to `std::io::Error`, takie jak te funkcje w
cesze `Write`:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-05-write-trait/src/lib.rs}}
```

`Result<..., Error>` jest często powtarzany. W związku z tym `std::io` ma następującą deklarację typu alias:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:here}}
```

Because this declaration is in the `std::io` module, we can use the fully
qualified alias `std::io::Result<T>`; that is, a `Result<T, E>` with the `E`
filled in as `std::io::Error`. The `Write` trait function signatures end up
looking like this:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:there}}
```

Alias ​​typu pomaga na dwa sposoby: ułatwia pisanie kodu *i* daje nam spójny interfejs w całym `std::io`. Ponieważ jest to alias, jest
po prostu kolejnym `Result<T, E>`, co oznacza, że ​​możemy używać z nim dowolnych metod, które działają na
`Result<T, E>`, a także specjalnej składni, takiej jak operator `?`.

### The Never Type that Never Returns

Rust ma specjalny typ o nazwie `!`, który w żargonie teorii typów jest znany jako
*typ pusty*, ponieważ nie ma wartości. My wolimy nazywać go *typem nigdy*,
ponieważ zastępuje typ zwracany, gdy funkcja nigdy nie zwróci. Oto przykład:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-07-never-type/src/lib.rs:here}}
```

Ten kod można odczytać jako „funkcja `bar` zwraca nigdy”. Funkcje, które zwracają
nigdy, nazywane są *funkcjami rozbieżnymi*. Nie możemy tworzyć wartości typu `!`
więc `bar` nigdy nie może zwrócić.

Ale jaki pożytek z typu, dla którego nigdy nie można tworzyć wartości? Przypomnij sobie kod z
Listingu 2-5, części gry w zgadywanie liczb; odtworzyliśmy jego część
tutaj w Listingu 20-26.

<Listing number="20-26" caption="A `match` with an arm that ends in `continue`">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:ch19}}
```

</Listing>

W tamtym czasie pominęliśmy pewne szczegóły w tym kodzie. W rozdziale 6 w sekcji [„Operator przepływu sterowania
`match`”][operator przepływu sterowania-match]<!-- ignore -->
omówiliśmy, że ramiona `match` muszą zwracać ten sam typ. Tak więc na przykład poniższy kod nie działa:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-08-match-arms-different-types/src/main.rs:here}}
```

Typ `guess` w tym kodzie musiałby być liczbą całkowitą *i* ciągiem znaków,
a Rust wymaga, aby `guess` miało tylko jeden typ. Co więc zwraca `continue`
? Jak mogliśmy zwrócić `u32` z jednego ramienia i mieć inne ramię,
które kończy się na `continue` w Liście 20-26?

Jak zapewne zgadłeś, `continue` ma wartość `!`. Oznacza to, że kiedy Rust
oblicza typ `guess`, bierze pod uwagę oba ramiona dopasowania, pierwsze z wartością `u32`, a drugie z wartością `!`. Ponieważ `!` nigdy nie może mieć wartości, Rust decyduje, że typem `guess` jest `u32`.

Formalny sposób opisu tego zachowania polega na tym, że wyrażenia typu `!` można
wymusić na dowolnym innym typie. Możemy zakończyć to ramię `match` poleceniem
`continue`, ponieważ `continue` nie zwraca wartości; zamiast tego przenosi kontrolę
z powrotem na górę pętli, więc w przypadku `Err` nigdy nie przypisujemy wartości do
`guess`.

Typ never jest również przydatny w przypadku makra `panic!`. Przypomnij sobie funkcję `unwrap`, którą wywołujemy na wartościach `Option<T>`, aby wygenerować wartość lub panikę z
tą definicją:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-09-unwrap-definition/src/lib.rs:here}}
```

W tym kodzie dzieje się to samo, co w `match` w Listingu 20-26: Rust
widzi, że `val` ma typ `T`, a `panic!` ma typ `!`, więc wynikiem
całego wyrażenia `match` jest `T`. Ten kod działa, ponieważ `panic!`
nie generuje wartości; kończy program. W przypadku `None` nie będziemy
zwracać wartości z `unwrap`, więc ten kod jest prawidłowy.

Jednym z ostatnich wyrażeń, które ma typ `!`, jest `loop`:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-10-loop-returns-never/src/main.rs:here}}
```

Tutaj pętla nigdy się nie kończy, więc `!` jest wartością wyrażenia. Jednak nie byłoby to prawdą, gdybyśmy dołączyli `break`, ponieważ pętla zakończyłaby się,
gdyby dotarła do `break`.

### Dynamically Sized Types and the `Sized` Trait

Rust musi znać pewne szczegóły dotyczące swoich typów, takie jak ilość miejsca,
które należy przydzielić na wartość określonego typu. To sprawia, że ​​jeden z aspektów jego systemu typów
na początku jest nieco mylący: koncepcja *typów o dynamicznym rozmiarze*.
Czasami nazywane *DST* lub *typami bez rozmiaru*, typy te pozwalają nam pisać
kod przy użyciu wartości, których rozmiar możemy poznać tylko w czasie wykonywania.

Przyjrzyjmy się bliżej szczegółom typu o dynamicznym rozmiarze o nazwie `str`,
którego używaliśmy w całej książce. Tak, nie `&str`, ale `str` samo w sobie,
jest DST. Nie możemy wiedzieć, jak długi jest ciąg do czasu wykonania, co oznacza,
że nie możemy utworzyć zmiennej typu `str`, ani nie możemy przyjąć argumentu typu
`str`. Rozważmy następujący kod, który nie działa:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-11-cant-create-str/src/main.rs:here}}
```

Rust musi wiedzieć, ile pamięci przydzielić dla dowolnej wartości określonego
typu, a wszystkie wartości danego typu muszą wykorzystywać tę samą ilość pamięci. Gdyby Rust
pozwolił nam napisać ten kod, te dwie wartości `str` musiałyby zajmować
tę samą ilość miejsca. Mają jednak różną długość: `s1` potrzebuje 12 bajtów pamięci, a `s2` 15. Dlatego nie można utworzyć zmiennej,
przechowującej typ o dynamicznym rozmiarze.

Co więc robimy? W tym przypadku znasz już odpowiedź: typy
`s1` i `s2` tworzymy jako `&str`, a nie `str`. Przypomnij sobie z sekcji [„String
Slices”][string-slices]<!-- ignore --> rozdziału 4, że struktura danych
slice po prostu przechowuje pozycję początkową i długość wycinka. Tak więc
chociaż `&T` jest pojedynczą wartością przechowującą adres pamięci, w której znajduje się
`T`, `&str` ma *dwie* wartości: adres `str` i jego
długość. Dzięki temu możemy poznać rozmiar wartości `&str` w czasie kompilacji: jest ona
dwukrotnie większa od długości `usize`. Oznacza to, że zawsze znamy rozmiar `&str`, niezależnie od tego, jak długi jest ciąg, do którego się odnosi. Ogólnie rzecz biorąc, w ten sposób
używane są typy o dynamicznym rozmiarze w Rust: mają one dodatkowy bit
metadanych przechowujący rozmiar dynamicznych informacji. Złota zasada
typów o dynamicznym rozmiarze polega na tym, że zawsze musimy umieszczać wartości typów o dynamicznym rozmiarze za wskaźnikiem jakiegoś rodzaju.

Możemy łączyć `str` ze wszystkimi rodzajami wskaźników: na przykład `Box<str>` lub
`Rc<str>`. Właściwie widziałeś to już wcześniej, ale z innym typem o dynamicznym rozmiarze: traits. Każda cecha jest typem o dynamicznym rozmiarze, do którego możemy się odwołać,
używając nazwy cechy. W rozdziale 18 w sekcji [„Używanie obiektów cech, które
pozwalają na wartości różnych
typów”][using-trait-objects-that-allow-for-values-of-different-types]<!--
ignore -->  wspomnieliśmy, że aby używać cech jako obiektów cech, musimy
umieścić je za wskaźnikiem, takim jak `&dyn Trait` lub `Box<dyn Trait>` (`Rc<dyn Trait>` również by zadziałało).

Aby pracować z DST, Rust udostępnia cechę `Sized`, aby określić, czy
rozmiar typu jest znany w czasie kompilacji. Ta cecha jest automatycznie implementowana
dla wszystkiego, czego rozmiar jest znany w czasie kompilacji. Ponadto Rust
niejawnie dodaje ograniczenie do `Sized` do każdej funkcji generycznej. To znaczy,
definicji funkcji generycznej takiej jak ta:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-12-generic-fn-definition/src/lib.rs}}
```

is actually treated as though we had written this:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-13-generic-implicit-sized-bound/src/lib.rs}}
```

By default, generic functions will work only on types that have a known size at
compile time. However, you can use the following special syntax to relax this
restriction:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-14-generic-maybe-sized/src/lib.rs}}
```

Cecha ograniczona do `?Sized` oznacza, że ​​„`T` może lub nie może być `Sized`”, a ta
notacja zastępuje domyślną zasadę, że typy generyczne muszą mieć znany rozmiar w
czasie kompilacji. Składnia `?Trait` z tym znaczeniem jest dostępna tylko dla
`Sized`, a nie dla żadnych innych cech.

Należy również zauważyć, że zmieniliśmy typ parametru `t` z `T` na `&T`.
Ponieważ typ może nie być `Sized`, musimy go użyć za jakimś
wskaźnikiem. W tym przypadku wybraliśmy odniesienie.

Następnie porozmawiamy o funkcjach i zamknięciach!

[encapsulation-that-hides-implementation-details]:
ch18-01-what-is-oo.html#encapsulation-that-hides-implementation-details
[string-slices]: ch04-03-slices.html#string-slices
[the-match-control-flow-operator]:
ch06-02-match.html#the-match-control-flow-operator
[using-trait-objects-that-allow-for-values-of-different-types]:
ch18-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[using-the-newtype-pattern]: ch20-03-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types
