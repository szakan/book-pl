## Zaawansowane funkcje i zamknięcia

W tej sekcji omówiono niektóre zaawansowane funkcje związane z funkcjami i zamknięciami, w tym wskaźniki funkcji i zwracane zamknięcia.

### Wskaźniki funkcji

Rozmawialiśmy o tym, jak przekazywać zamknięcia do funkcji; możesz również przekazywać zwykłe
funkcje do funkcji! Ta technika jest przydatna, gdy chcesz przekazać
funkcję, którą już zdefiniowałeś, zamiast definiować nowe zamknięcie. Funkcje
przymuszają do typu `fn` (z małą literą f), którego nie należy mylić z cechą zamknięcia `Fn`. Typ `fn` nazywany jest *wskaźnikiem funkcji*. Przekazywanie funkcji
ze wskaźnikami funkcji pozwoli Ci używać funkcji jako argumentów innych
funkcji.

Składnia określania, że ​​parametr jest wskaźnikiem funkcji, jest podobna do
składni zamknięć, jak pokazano w Liście 20-27, gdzie zdefiniowaliśmy funkcję
`add_one`, która dodaje jeden do swojego parametru. Funkcja `do_twice` przyjmuje dwa
parametry: wskaźnik funkcji do dowolnej funkcji, która przyjmuje parametr `i32`
i zwraca `i32`, oraz jedną wartość `i32`. Funkcja `do_twice` wywołuje
funkcję `f` dwukrotnie, przekazując jej wartość `arg`, a następnie dodaje do siebie dwa wyniki wywołania funkcji. Funkcja `main` wywołuje `do_twice` z argumentami
`add_one` i `5`.

<Listing number="20-27" file-name="src/main.rs" caption="Using the `fn` type to accept a function pointer as an argument">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-27/src/main.rs}}
```

</Listing>

Ten kod drukuje `Odpowiedź brzmi: 12`. Określamy, że parametr `f` w
`do_twice` jest `fn`, który przyjmuje jeden parametr typu `i32` i zwraca
`i32`. Następnie możemy wywołać `f` w treści `do_twice`. W `main` możemy przekazać
nazwę funkcji `add_one` jako pierwszy argument do `do_twice`.

W przeciwieństwie do zamknięć, `fn` jest typem, a nie cechą, więc określamy `fn` jako
typ parametru bezpośrednio, zamiast deklarować parametr typu ogólnego z jedną
z cech `Fn` jako ograniczeniem cechy.

Wskaźniki funkcji implementują wszystkie trzy cechy zamknięcia (`Fn`, `FnMut` i
`FnOnce`), co oznacza, że ​​zawsze można przekazać wskaźnik funkcji jako argument dla
funkcji, która oczekuje zamknięcia. Najlepiej pisać funkcje, używając
typu ogólnego i jednej z cech zamknięcia, aby funkcje mogły akceptować
funkcje lub zamknięcia.

To powiedziawszy, jednym z przykładów, w których chciałbyś akceptować tylko `fn`, a nie
zamknięcia, jest interfejs z zewnętrznym kodem, który nie ma zamknięć:
Funkcje C mogą akceptować funkcje jako argumenty, ale C nie ma zamknięć.

Jako przykład, w którym możesz użyć zamknięcia zdefiniowanego inline lub nazwanej
funkcji, przyjrzyjmy się użyciu metody `map` dostarczonej przez cechę `Iterator` w bibliotece standardowej. Aby użyć funkcji `map` do przekształcenia wektora
liczb w wektor ciągów, moglibyśmy użyć zamknięcia, takiego jak to:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-15-map-closure/src/main.rs:here}}
```

Albo moglibyśmy nazwać funkcję argumentem `map` zamiast zamknięcia,
tak:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-16-map-function/src/main.rs:here}}
```

Należy zauważyć, że musimy użyć w pełni kwalifikowanej składni, o której mówiliśmy wcześniej
w sekcji [„Zaawansowane cechy”][advanced-traits]<!-- ignore -->, ponieważ
istnieje wiele dostępnych funkcji o nazwie `to_string`. Tutaj używamy funkcji
`to_string` zdefiniowanej w cesze `ToString`, którą standardowa
biblioteka zaimplementowała dla każdego typu implementującego `Display`.

Przypomnijmy sobie z sekcji [„Wartości wyliczeniowe”][enum-values]<!-- ignore --> rozdziału
6, że nazwa każdego wariantu wyliczenia, który definiujemy, staje się również funkcją
inicjalizującą. Możemy używać tych funkcji inicjujących jako wskaźników do funkcji, które
implementują cechy zamknięcia, co oznacza, że ​​możemy określić funkcje inicjujące jako argumenty dla metod, które przyjmują zamknięcia, w następujący sposób:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-17-map-initializer/src/main.rs:here}}
```

Tutaj tworzymy instancje `Status::Value` używając każdej wartości `u32` w zakresie,
w którym `map` jest wywoływana za pomocą funkcji inicjatora `Status::Value`.
Niektórzy wolą ten styl, a inni wolą używać zamknięć. Kompilują się one do tego samego kodu, więc użyj stylu, który jest dla Ciebie bardziej zrozumiały.

### Zamknięcia powrotne

Zamknięcia są reprezentowane przez cechy, co oznacza, że ​​nie można zwracać zamknięć
bezpośrednio. W większości przypadków, gdy chcesz zwrócić cechę, możesz zamiast tego
użyć konkretnego typu, który implementuje cechę, jako wartości zwracanej
funkcji. Nie możesz jednak tego zrobić z zamknięciami, ponieważ nie mają one konkretnego typu, który jest zwracalny; nie możesz na przykład używać wskaźnika funkcji `fn` jako typu zwracanego.

Poniższy kod próbuje zwrócić zamknięcie bezpośrednio, ale się nie skompiluje:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-18-returns-closure/src/lib.rs}}
```

Błąd kompilatora wygląda następująco:

```console
{{#include ../listings/ch20-advanced-features/no-listing-18-returns-closure/output.txt}}
```

Błąd ponownie odwołuje się do cechy `Sized`! Rust nie wie, ile miejsca
będzie potrzebował do przechowywania zamknięcia. Wcześniej widzieliśmy rozwiązanie tego problemu.
Możemy użyć obiektu cechy:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-19-returns-closure-trait-object/src/lib.rs}}
```

Ten kod skompiluje się bez problemu. Więcej informacji o obiektach cech znajdziesz w
sekcji [„Używanie obiektów cech, które zezwalają na wartości różnych
typów”][using-trait-objects-that-allow-for-values-of-different-types]<!--
ignore --> w rozdziale 19.

Następnie przyjrzyjmy się makro!

[advanced-traits]:
ch20-03-advanced-traits.html#advanced-traits
[enum-values]: ch06-01-defining-an-enum.html#enum-values
[using-trait-objects-that-allow-for-values-of-different-types]:
ch18-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
