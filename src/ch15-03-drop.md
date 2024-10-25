## Uruchamianie kodu podczas czyszczenia z cechą `Drop`

Drugą ważną cechą wzorca inteligentnego wskaźnika jest `Drop`, która pozwala
dostosować, co się dzieje, gdy wartość ma wyjść poza zakres. Możesz
dostarczyć implementację cechy `Drop` dla dowolnego typu, a ten kod może
być używany do zwalniania zasobów, takich jak pliki lub połączenia sieciowe.

Wprowadzamy `Drop` w kontekście inteligentnych wskaźników, ponieważ
funkcjonalność cechy `Drop` jest prawie zawsze używana podczas implementowania
inteligentnego wskaźnika. Na przykład, gdy `Box<T>` zostanie usunięty, zwolniona zostanie przestrzeń na stercie, na którą wskazuje pole.

W niektórych językach, dla niektórych typów, programista musi wywołać kod w celu zwolnienia pamięci
lub zasobów za każdym razem, gdy zakończy korzystanie z instancji tych typów. Przykłady
obejmują uchwyty plików, gniazda lub blokady. Jeśli zapomną, system może
zostać przeciążony i ulec awarii. W Rust możesz określić, że konkretny fragment kodu
zostanie uruchomiony, gdy wartość wyjdzie poza zakres, a kompilator wstawi ten kod automatycznie. W rezultacie nie musisz uważać, aby
umieszczać kodu czyszczącego wszędzie w programie, w którym instancja określonego
typu została ukończona — nadal nie spowoduje to wycieku zasobów!

Kod, który ma zostać uruchomiony, gdy wartość wyjdzie poza zakres, określasz, implementując cechę
``Drop`. Cecha `Drop` wymaga zaimplementowania jednej metody o nazwie
`drop`, która przyjmuje zmienne odwołanie do `self`. Aby zobaczyć, kiedy Rust wywołuje `drop`,
zaimplementujmy teraz `drop` za pomocą instrukcji `println!`.

Listing 15-14 pokazuje strukturę `CustomSmartPointer`, której jedyną niestandardową
funkcjonalnością jest to, że wydrukuje `Dropping CustomSmartPointer!`, gdy
instancja wyjdzie poza zakres, aby pokazać, kiedy Rust uruchomi funkcję `drop`.

<Listing number="15-14" file-name="src/main.rs" caption="A `CustomSmartPointer` struct that implements the `Drop` trait where we would put our cleanup code">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-14/src/main.rs}}
```

</Listing>

Cecha `Drop` jest zawarta w preludium, więc nie musimy jej wprowadzać do
zakresu. Implementujemy cechę `Drop` w `CustomSmartPointer` i dostarczamy
implementację dla metody `drop`, która wywołuje `println!`. Ciało funkcji
`drop` to miejsce, w którym można umieścić dowolną logikę, którą chcesz uruchomić, gdy
instancja Twojego typu wyjdzie poza zakres. Drukujemy tutaj tekst, aby
wizualnie zademonstrować, kiedy Rust wywoła `drop`.

W `main` tworzymy dwie instancje `CustomSmartPointer`, a następnie drukujemy
`CustomSmartPointers created`. Na końcu `main` nasze instancje
`CustomSmartPointer` wyjdą poza zakres, a Rust wywoła kod, który umieściliśmy
w metodzie `drop`, drukując naszą ostateczną wiadomość. Należy zauważyć, że nie musieliśmy
wywoływać jawnie metody `drop`.

Po uruchomieniu tego programu zobaczymy następujący wynik:
```console
{{#include ../listings/ch15-smart-pointers/listing-15-14/output.txt}}
```

Rust automatycznie wywołał `drop` dla nas, gdy nasze instancje wyszły poza zakres,
wywołując określony przez nas kod. Zmienne są usuwane w odwrotnej kolejności
ich tworzenia, więc `d` zostało usunięte przed `c`. Celem tego przykładu jest
pokazanie wizualnego przewodnika po tym, jak działa metoda `drop`; zwykle określasz
kod czyszczący, który musi zostać uruchomiony przez Twój typ, a nie
komunikat drukowania.

### Dropping a Value Early with `std::mem::drop`

Niestety, nie jest łatwo wyłączyć funkcję automatycznego `drop`
. Wyłączenie `drop` zwykle nie jest konieczne; cały sens cechy
`Drop` polega na tym, że jest ona obsługiwana automatycznie. Czasami jednak
możesz chcieć wyczyścić wartość wcześniej. Jednym z przykładów jest używanie inteligentnych
wskaźników, które zarządzają blokadami: możesz chcieć wymusić metodę `drop`, która
zwalnia blokadę, tak aby inny kod w tym samym zakresie mógł ją uzyskać.
Rust nie pozwala na ręczne wywołanie metody `drop` cechy `Drop`; zamiast tego
musisz wywołać funkcję `std::mem::drop` dostarczoną przez bibliotekę standardową,
jeśli chcesz wymusić usunięcie wartości przed końcem jej zakresu.

Jeśli spróbujemy ręcznie wywołać metodę `drop` cechy `Drop`, modyfikując funkcję
`main` z Listingu 15-14, jak pokazano na Listingu 15-15, otrzymamy
błąd kompilatora:

<Listing number="15-15" file-name="src/main.rs" caption="Attempting to call the `drop` method from the `Drop` trait manually to clean up early">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-15/src/main.rs:here}}
```

</Listing>

When we try to compile this code, we’ll get this error:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-15/output.txt}}
```

Ten komunikat o błędzie mówi, że nie możemy jawnie wywołać `drop`. W
komunikacie o błędzie użyto terminu *destruktor*, który jest ogólnym terminem programistycznym
na określenie funkcji, która czyści instancję. *Destruktor* jest analogiczny do
*konstruktora*, który tworzy instancję. Funkcja `drop` w Rust jest jednym
konkretnym destruktorem.

Rust nie pozwala nam jawnie wywołać `drop`, ponieważ Rust nadal
automatycznie wywołałby `drop` dla wartości na końcu `main`. Spowodowałoby to błąd
*podwójnego zwolnienia*, ponieważ Rust próbowałby wyczyścić tę samą wartość
dwa razy.

Nie możemy wyłączyć automatycznego wstawiania `drop`, gdy wartość wykracza poza
zakres, i nie możemy jawnie wywołać metody `drop`. Tak więc, jeśli musimy wymusić
wczesne wyczyszczenie wartości, używamy funkcji `std::mem::drop`.

Funkcja `std::mem::drop` różni się od metody `drop` w cesze `Drop`. Wywołujemy ją, przekazując jako argument wartość, którą chcemy wymusić, aby drop została usunięta.
Funkcja znajduje się w preludium, więc możemy zmodyfikować `main` w Liście 15-15, aby
wywołać funkcję `drop`, jak pokazano w Liście 15-16:

<Listing number="15-16" file-name="src/main.rs" caption="Calling `std::mem::drop` to explicitly drop a value before it goes out of scope">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-16/src/main.rs:here}}
```

</Listing>

Uruchomienie tego kodu spowoduje wydrukowanie następującego komunikatu:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-16/output.txt}}
```

Tekst ```Dropping CustomSmartPointer with data `some data`!``` jest drukowany
pomiędzy tekstem `CustomSmartPointer created.` i `CustomSmartPointer removed.` przed końcem tekstu main.`, pokazując, że kod metody `drop` jest wywoływany w celu
drop `c` w tym momencie.

Możesz użyć kodu określonego w implementacji cechy `Drop` na wiele sposobów, aby
uczynić czyszczenie wygodnym i bezpiecznym: na przykład możesz go użyć do utworzenia
własnego alokatora pamięci! Dzięki cesze `Drop` i systemowi własności Rust,
nie musisz pamiętać o czyszczeniu, ponieważ Rust robi to automatycznie.

Nie musisz się również martwić o problemy wynikające z przypadkowego
czyszczenia wartości, które są nadal używane: system własności, który zapewnia, że
odniesienia są zawsze prawidłowe, zapewnia również, że `drop` zostanie wywołany tylko raz, gdy
wartość nie jest już używana.

Teraz, gdy przyjrzeliśmy się `Box<T>` i niektórym cechom inteligentnych wskaźników, przyjrzyjmy się kilku innym inteligentnym wskaźnikom zdefiniowanym w standardzie
library.
