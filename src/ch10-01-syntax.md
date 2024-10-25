## Generic Typy danych

Używamy generyków do tworzenia definicji dla elementów, takich jak sygnatury funkcji lub
struktury, których możemy następnie używać z wieloma różnymi konkretnymi typami danych. Najpierw przyjrzyjmy się, jak definiować funkcje, struktury, wyliczenia i metody przy użyciu
generyków. Następnie omówimy, jak generyki wpływają na wydajność kodu.

### In Function Definitions

Definiując funkcję, która używa generyków, umieszczamy generyki w sygnaturze funkcji, w której zwykle określamy typy danych parametrów i wartości zwracanej. Dzięki temu nasz kod staje się bardziej elastyczny i zapewnia
więcej funkcji wywołującym naszą funkcję, jednocześnie zapobiegając duplikacji kodu.

Kontynuując naszą `largest` funkcję, Listing 10-4 pokazuje dwie funkcje, które
obie znajdują największą wartość w wycinku. Następnie połączymy je w jedną
funkcję, która używa generyków.

<Listing number="10-4" file-name="src/main.rs" caption="Two functions that differ only in their names and in the types in their signatures">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-04/src/main.rs:here}}
```

</Listing>

Funkcja `largest_i32` to ta, którą wyodrębniliśmy w Liście 10-3, która znajduje
największy `i32` w wycinku. Funkcja `largest_char` znajduje największy
`char` w wycinku. Ciała funkcji mają ten sam kod, więc wyeliminujmy
duplikację, wprowadzając ogólny parametr typu w pojedynczej funkcji.

Aby sparametryzować typy w nowej pojedynczej funkcji, musimy nazwać parametr typu, tak jak robimy to w przypadku parametrów wartości funkcji. Możesz użyć
dowolnego identyfikatora jako nazwy parametru typu. Ale użyjemy `T`, ponieważ, zgodnie z
konwencją, nazwy parametrów typu w Rust są krótkie, często składają się tylko z jednej litery, a
konwencją nazewnictwa typów w Rust jest UpperCamelCase. Skrót od *type*, `T` jest
domyślnym wyborem większości programistów Rust.

Kiedy używamy parametru w ciele funkcji, musimy zadeklarować
nazwę parametru w sygnaturze, aby kompilator wiedział, co ta nazwa oznacza.
Podobnie, kiedy używamy nazwy parametru typu w sygnaturze funkcji, musimy
zadeklarować nazwę parametru typu przed jej użyciem. Aby zdefiniować generyczną
`largest` funkcję, umieszczamy deklaracje nazwy typu w nawiasach kątowych,
`<>`, między nazwą funkcji a listą parametrów, w następujący sposób:

```rust,ignore
fn largest<T>(list: &[T]) -> &T {
```

Czytamy tę definicję jako: funkcja `largest` jest generyczna w odniesieniu do pewnego typu
`T`. Ta funkcja ma jeden parametr o nazwie `list`, który jest wycinkiem wartości
typu `T`. Funkcja `largest` zwróci odwołanie do wartości
tego samego typu `T`.

Listing 10-5 pokazuje połączoną definicję funkcji `largest` używającą generycznego
typu danych w jej sygnaturze. Listing pokazuje również, jak możemy wywołać funkcję
z wycinkiem wartości `i32` lub wartości `char`. Należy zauważyć, że ten kod nie zostanie jeszcze skompilowany, ale naprawimy to później w tym rozdziale.

<Listing number="10-5" file-name="src/main.rs" caption="The `largest` function using generic type parameters; this doesn’t compile yet">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/src/main.rs}}
```

</Listing>

If we compile this code right now, we’ll get this error:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/output.txt}}
```

Tekst pomocy wspomina `std::cmp::PartialOrd`, który jest *cechą*, a o cechach
porozmawiamy w następnej sekcji. Na razie wiedz, że ten błąd
oświadcza, że ​​treść `largest` nie będzie działać dla wszystkich możliwych typów, jakimi może być `T`. Ponieważ chcemy porównywać wartości typu `T` w treści, możemy
używać tylko typów, których wartości można uporządkować. Aby umożliwić porównania, standardowa
biblioteka ma cechę `std::cmp::PartialOrd`, którą można zaimplementować w typach
(więcej informacji na temat tej cechy można znaleźć w Załączniku C). Postępując zgodnie z sugestią z tekstu pomocy, ograniczamy typy prawidłowe dla `T` tylko do tych, które implementują
`PartialOrd`, a ten przykład zostanie skompilowany, ponieważ standardowa
biblioteka implementuje `PartialOrd` zarówno w `i32`, jak i `char`.

### In Struct Definitions

Możemy również zdefiniować struktury, aby użyć parametru typu generycznego w jednym lub większej liczbie pól, używając składni `<>`. Listing 10-6 definiuje strukturę `Point<T>`, aby przechowywać wartości współrzędnych `x` i `y` dowolnego typu.

<Listing number="10-6" file-name="src/main.rs" caption="A `Point<T>` struct that holds `x` and `y` values of type `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-06/src/main.rs}}
```

</Listing>

Składnia używania typów generycznych w definicjach struktur jest podobna do tej używanej w definicjach
funkcji. Najpierw deklarujemy nazwę parametru typu w nawiasach kątowych tuż po nazwie struktury. Następnie używamy typu generycznego w definicji struktury, gdzie w przeciwnym razie określilibyśmy konkretne typy
danych.

Należy zauważyć, że ponieważ użyliśmy tylko jednego typu generycznego do zdefiniowania `Point<T>`, ta
definicja mówi, że struktura `Point<T>` jest generyczna w stosunku do pewnego typu `T`, a
pola `x` i `y` są *oba* tego samego typu, niezależnie od tego, jaki to typ. Jeśli
utworzymy wystąpienie `Point<T>`, które ma wartości różnych typów, jak w
Listingu 10-7, nasz kod się nie skompiluje.

<Listing number="10-7" file-name="src/main.rs" caption="The fields `x` and `y` must be the same type because both have the same generic data type `T`.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/src/main.rs}}
```

</Listing>

W tym przykładzie, gdy przypisujemy wartość całkowitą `5` do `x`, informujemy
kompilator, że typ generyczny `T` będzie liczbą całkowitą dla tej instancji
`Point<T>`. Następnie, gdy określimy `4.0` dla `y`, które zdefiniowaliśmy tak, aby miało ten sam typ co `x`, otrzymamy błąd niezgodności typów, taki jak ten:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/output.txt}}
```

Aby zdefiniować strukturę `Point`, w której `x` i `y` są generyczne, ale mogą mieć
różne typy, możemy użyć wielu parametrów typu generycznego. Na przykład w
Listingu 10-8 zmieniamy definicję `Point` na generyczną dla typów `T`
i `U`, gdzie `x` jest typu `T`, a `y` jest typu `U`.

<Listing number="10-8" file-name="src/main.rs" caption="A `Point<T, U>` generic over two types so that `x` and `y` can be values of different types">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-08/src/main.rs}}
```

</Listing>

Teraz wszystkie pokazane wystąpienia `Point` są dozwolone! Możesz użyć tylu parametrów typu generycznego, ile chcesz w definicji, ale użycie więcej niż kilku sprawia, że ​​kod jest trudny do odczytania. Jeśli stwierdzasz, że potrzebujesz wielu typów generycznych w swoim kodzie, może to oznaczać, że kod wymaga restrukturyzacji na mniejsze
części.

### In Enum Definitions

Podobnie jak w przypadku struktur, możemy zdefiniować wyliczenia, aby przechowywać ogólne typy danych w ich wariantach. Przyjrzyjmy się ponownie wyliczeniu `Option<T>`, które zapewnia standardowa
biblioteka, a którego użyliśmy w rozdziale 6:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

Ta definicja powinna teraz nabrać dla Ciebie więcej sensu. Jak widzisz, wyliczenie
`Option<T>` jest generyczne w stosunku do typu `T` i ma dwie odmiany: `Some`, która
przechowuje jedną wartość typu `T`, i odmianę `None`, która nie przechowuje żadnej wartości.
Używając wyliczenia `Option<T>` możemy wyrazić abstrakcyjną koncepcję
wartości opcjonalnej, a ponieważ `Option<T>` jest generyczne, możemy użyć tej abstrakcji
bez względu na typ wartości opcjonalnej.

Wyliczenia mogą również używać wielu typów generycznych. Definicja wyliczenia `Result`,
którego użyliśmy w rozdziale 9, jest jednym z przykładów:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

Wyliczenie `Result` jest generyczne dla dwóch typów, `T` i `E`, i ma dwie odmiany:
`Ok`, które przechowuje wartość typu `T`, i `Err`, które przechowuje wartość typu
`E`. Ta definicja ułatwia używanie wyliczenia `Result` wszędzie tam, gdzie
mamy operację, która może się powieść (zwrócić wartość jakiegoś typu `T`) lub nie powieść
(zwrócić błąd jakiegoś typu `E`). W rzeczywistości, to właśnie tego użyliśmy do otwarcia
pliku w Liście 9-3, gdzie `T` zostało wypełnione typem `std::fs::File`, gdy
plik został pomyślnie otwarty, a `E` zostało wypełnione typem
`std::io::Error`, gdy wystąpiły problemy z otwarciem pliku.

Gdy rozpoznasz sytuacje w swoim kodzie z wieloma definicjami struktur lub wyliczeń,
które różnią się tylko typami przechowywanych przez nie wartości, możesz
uniknąć duplikacji, używając zamiast tego typów generycznych.

### In Method Definitions

We can implement methods on structs and enums (as we did in Chapter 5) and use
generic types in their definitions too. Listing 10-9 shows the `Point<T>`
struct we defined in Listing 10-6 with a method named `x` implemented on it.

<Listing number="10-9" file-name="src/main.rs" caption="Implementing a method named `x` on the `Point<T>` struct that will return a reference to the `x` field of type `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-09/src/main.rs}}
```

</Listing>

Tutaj zdefiniowaliśmy metodę o nazwie `x` w `Point<T>`, która zwraca odwołanie
do danych w polu `x`.

Należy zauważyć, że musimy zadeklarować `T` zaraz po `impl`, abyśmy mogli użyć `T` do określenia,
że implementujemy metody w typie `Point<T>`. Deklarując `T` jako
typ ogólny po `impl`, Rust może zidentyfikować, że typ w nawiasach kątowych w `Point` jest typem ogólnym, a nie typem konkretnym. Mogliśmy
wybrać inną nazwę dla tego parametru ogólnego niż parametr ogólny zadeklarowany w definicji struktury, ale używanie tej samej nazwy jest
konwencjonalne. Metody napisane w `impl`, który deklaruje typ ogólny,
zostaną zdefiniowane w dowolnym wystąpieniu typu, niezależnie od tego, jaki typ konkretny ostatecznie
zastępuje typ ogólny.

Możemy również określić ograniczenia dotyczące typów ogólnych podczas definiowania metod w typie. Moglibyśmy na przykład implementować metody tylko na instancjach `Point<f32>`,
a nie na instancjach `Point<T>` z dowolnym typem generycznym. W Listingu 10-10
używamy konkretnego typu `f32`, co oznacza, że ​​nie deklarujemy żadnych typów po `impl`.

<Listing number="10-10" file-name="src/main.rs" caption="An `impl` block that only applies to a struct with a particular concrete type for the generic type parameter `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-10/src/main.rs:here}}
```

</Listing>
Ten kod oznacza, że ​​typ `Point<f32>` będzie miał metodę `distance_from_origin`
; inne wystąpienia `Point<T>`, w których `T` nie jest typu `f32`, nie będą
mieć zdefiniowanej tej metody. Metoda mierzy, jak daleko nasz punkt jest od
punktu o współrzędnych (0,0, 0,0) i używa operacji matematycznych, które są
dostępne tylko dla typów zmiennoprzecinkowych.

Parametry typu generycznego w definicji struktury nie zawsze są takie same, jak te,
których używasz w sygnaturach metod tej samej struktury. W listingu 10-11 użyto typów generycznych `X1` i `Y1` dla struktury `Point` oraz `X2` `Y2` dla sygnatury metody `mixup`, aby przykład był bardziej przejrzysty. Metoda tworzy nową instancję `Point` z wartością `x` z `self` `Point` (typu `X1`) i wartością `y` z przekazanego `Point` (typu `Y2`).
<Listing number="10-11" file-name="src/main.rs" caption="A method that uses generic types different from its struct’s definition">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-11/src/main.rs}}
```

</Listing>

W `main` zdefiniowaliśmy `Point`, który ma `i32` dla `x` (z wartością `5`)
i `f64` dla `y` (z wartością `10.4`). Zmienna `p2` jest strukturą `Point`,
która ma wycinek ciągu dla `x` (z wartością `"Hello"`) i `char` dla `y`
(z wartością `c`). Wywołanie `mixup` na `p1` z argumentem `p2` daje nam `p3`,
które będzie miało `i32` dla `x`, ponieważ `x` pochodzi z `p1`. Zmienna `p3`
będzie miała `char` dla `y`, ponieważ `y` pochodzi z `p2`. Wywołanie makra `println!`
wydrukuje `p3.x = 5, p3.y = c`.

Celem tego przykładu jest pokazanie sytuacji, w której niektóre parametry generyczne są deklarowane za pomocą `impl`, a niektóre za pomocą definicji
metody. Tutaj parametry generyczne `X1` i `Y1` są deklarowane po
`impl`, ponieważ są zgodne z definicją struktury. Parametry generyczne `X2`
i `Y2` są deklarowane po `fn mixup`, ponieważ są istotne tylko dla
metody.

### Performance of Code Using Generics

Możesz się zastanawiać, czy istnieje koszt wykonania podczas korzystania z parametrów typu generycznego. Dobra wiadomość jest taka, że ​​korzystanie z typów generycznych nie sprawi, że program będzie działał wolniej niż w przypadku typów konkretnych.

Rust osiąga to, wykonując monomorfizację kodu przy użyciu typów generycznych w czasie kompilacji. *Monomorfizacja* to proces przekształcania kodu generycznego w kod konkretny poprzez wypełnianie typów konkretnych, które są używane podczas kompilacji. W tym procesie kompilator wykonuje odwrotne kroki niż te, których użyliśmy do utworzenia funkcji generycznej w Liście 10-5: kompilator sprawdza wszystkie miejsca, w których wywoływany jest kod generyczny i generuje kod dla typów konkretnych, z którymi wywoływany jest kod generyczny.

Przyjrzyjmy się, jak to działa, korzystając z generycznych typów biblioteki standardowej
`Option<T>` enum:

```rust
let integer = Some(5);
let float = Some(5.0);
```

Kiedy Rust kompiluje ten kod, wykonuje monomorfizację. Podczas tego
procesu kompilator odczytuje wartości, które zostały użyte w instancjach `Option<T>`
i identyfikuje dwa rodzaje `Option<T>`: jeden to `i32`, a drugi to `f64`. W związku z tym rozszerza on ogólną definicję `Option<T>` o dwie
definicje wyspecjalizowane dla `i32` i `f64`, zastępując tym samym ogólną
definicję konkretnymi.

Zmonoforyzowana wersja kodu wygląda podobnie do poniższej (kompilator
używa innych nazw niż te, których używamy tutaj do ilustracji):

<Listing file-name="src/main.rs">

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

</Listing>

Generyczna `Option<T>` jest zastępowana przez konkretne definicje utworzone przez
kompilator. Ponieważ Rust kompiluje kod generyczny do kodu, który określa
typ w każdej instancji, nie ponosimy kosztów wykonania za używanie generyków. Gdy kod
jest uruchamiany, działa tak samo, jak gdybyśmy powielili każdą definicję ręcznie. Proces monomorfizacji sprawia, że ​​generyki Rusta są niezwykle wydajne
w czasie wykonywania.
