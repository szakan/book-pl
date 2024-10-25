<!-- Stary nagłówek. Nie usuwaj, bo linki mogą się zepsuć. -->
<a id="closures-anonymous-functions-that-can-capture-their-environment"></a>

## Closures: Anonymous Funkcje that Capture Their Environment

Zamknięcia Rusta to anonimowe funkcje, które można zapisać w zmiennej lub przekazać jako
argumenty do innych funkcji. Można utworzyć zamknięcie w jednym miejscu, a następnie
wywołać je gdzie indziej, aby ocenić je w innym kontekście. W przeciwieństwie do
funkcji, zamknięcia mogą przechwytywać wartości z zakresu, w którym są zdefiniowane.
Pokażemy, w jaki sposób te funkcje zamknięć umożliwiają ponowne wykorzystanie kodu i dostosowywanie
zachowania.

<!-- Stare nagłówki. Nie usuwaj, bo linki mogą się zepsuć. -->
<a id="creating-an-abstraction-of-behavior-with-closures"></a>
<a id="refactoring-using-functions"></a>
<a id="refactoring-with-closures-to-store-code"></a>

### Capturing the Environment with Closures

Najpierw sprawdzimy, jak możemy użyć zamknięć, aby przechwycić wartości ze
środowiska, w którym są zdefiniowane, w celu późniejszego wykorzystania. Oto scenariusz: Co jakiś czas nasza firma produkująca koszulki rozdaje ekskluzywną koszulkę w limitowanej edycji
komuś z naszej listy mailingowej jako promocję. Osoby z listy mailingowej mogą
opcjonalnie dodać swój ulubiony kolor do swojego profilu. Jeśli osoba wybrana do
darmowej koszulki ma swój ulubiony kolor, otrzymuje koszulkę w tym kolorze. Jeśli
osoba nie określiła ulubionego koloru, otrzymuje kolor, którego firma
obecnie ma najwięcej.

Istnieje wiele sposobów na zaimplementowanie tego. W tym przykładzie użyjemy
wyliczenia o nazwie `ShirtColor`, które ma warianty `Red` i `Blue` (ograniczając
liczbę dostępnych kolorów dla uproszczenia). Reprezentujemy
inwentarz firmy za pomocą struktury `Inventory`, która ma pole o nazwie `shirts`, które
zawiera `Vec<ShirtColor>` reprezentujące kolory koszulek aktualnie dostępne w magazynie.
Metoda `giveaway` zdefiniowana w `Inventory` pobiera opcjonalną preferencję koloru koszuli zwycięzcy darmowej koszuli i zwraca kolor koszuli, który dana osoba otrzyma. Ta konfiguracja jest pokazana w Listingu 13-1:

<Listing number="13-1" file-name="src/main.rs" caption="Shirt company giveaway situation">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-01/src/main.rs}}
```

</Listing>

`store` zdefiniowany w `main` ma dwie niebieskie koszulki i jedną czerwoną koszulkę pozostałe do rozdania w ramach tej limitowanej promocji. Wywołujemy metodę `giveaway`
dla użytkownika preferującego czerwoną koszulkę i użytkownika bez preferencji.

Kod ten można zaimplementować na wiele sposobów, a tutaj, aby skupić się na
zamknięciach, trzymaliśmy się koncepcji, których już się nauczyłeś, z wyjątkiem treści
metody `giveaway`, która używa zamknięcia. W metodzie `giveaway` otrzymujemy
preferencję użytkownika jako parametr typu `Option<ShirtColor>` i wywołujemy metodę
`unwrap_or_else` w `user_preference`. Metoda [`unwrap_or_else` w
`Option<T>`][unwrap-or-else]<!-- ignore --> jest zdefiniowana przez bibliotekę standardową.
Przyjmuje jeden argument: zamknięcie bez żadnych argumentów, które zwraca wartość `T`
(ten sam typ przechowywany w wariancie `Some` `Option<T>`, w tym przypadku
`ShirtColor`). Jeśli `Option<T>` jest wariantem `Some`, `unwrap_or_else`
zwraca wartość z `Some`. Jeśli `Option<T>` jest wariantem `None`
, `unwrap_or_else` wywołuje zamknięcie i zwraca wartość zwróconą przez
zamknięcie.

Określamy wyrażenie zamknięcia `|| self.most_stocked()` jako argument dla
`unwrap_or_else`. Jest to zamknięcie, które samo w sobie nie przyjmuje żadnych parametrów (gdyby
zamknięcie miało parametry, pojawiłyby się one między dwoma pionowymi paskami).
Ciało zamknięcia wywołuje `self.most_stocked()`. Definiujemy zamknięcie
tutaj, a implementacja `unwrap_or_else` oceni zamknięcie
później, jeśli wynik będzie potrzebny.

Uruchomienie tego kodu drukuje:

```console
{{#include ../listings/ch13-functional-features/listing-13-01/output.txt}}
```

Ciekawym aspektem jest to, że przekazaliśmy zamknięcie, które wywołuje
`self.most_stocked()` na bieżącej instancji `Inventory`. Standardowa biblioteka
nie musiała nic wiedzieć o zdefiniowanych przez nas typach `Inventory` lub `ShirtColor`, ani o logice, której chcemy użyć w tym scenariuszu. Zamknięcie przechwytuje
niezmienną referencję do instancji `self` `Inventory` i przekazuje ją z
kodem, który określiliśmy do metody `unwrap_or_else`. Funkcje z drugiej strony
nie są w stanie przechwycić swojego otoczenia w ten sposób.

### Closure Type Inference and Annotation

Istnieje więcej różnic między funkcjami i zamknięciami. Zamknięcia nie
zwykle wymagają adnotacji typów parametrów lub wartości zwracanej,
jak robią to funkcje `fn`. Adnotacje typu są wymagane w funkcjach, ponieważ
typy są częścią jawnego interfejsu udostępnianego użytkownikom. Sztywne zdefiniowanie tego
interfejsu jest ważne, aby zapewnić, że wszyscy zgadzają się co do typów
wartości używanych i zwracanych przez funkcję. Zamknięcia z drugiej strony nie są używane
w takim udostępnionym interfejsie: są przechowywane w zmiennych i używane bez
nazywania ich i udostępniania użytkownikom naszej biblioteki.

Zamknięcia są zazwyczaj krótkie i istotne tylko w wąskim kontekście,
a nie w dowolnym dowolnym scenariuszu. W tych ograniczonych kontekstach kompilator może
wnioskować typy parametrów i typ zwracany, podobnie jak jest w stanie
wnioskować typy większości zmiennych (rzadko zdarzają się przypadki, w których kompilator
potrzebuje również adnotacji typu zamknięcia).

Podobnie jak w przypadku zmiennych, możemy dodać adnotacje typu, jeśli chcemy zwiększyć
wyraźność i przejrzystość kosztem większej rozwlekłości niż jest to absolutnie konieczne. Adnotacje typów dla zamknięcia wyglądałyby jak definicja
przedstawiona w Liście 13-2. W tym przykładzie definiujemy zamknięcie i przechowujemy je
w zmiennej, zamiast definiować zamknięcie w miejscu, w którym przekazujemy je jako
argument, jak zrobiliśmy to w Liście 13-1.

<Listing number="13-2" file-name="src/main.rs" caption="Adding optional type annotations of the parameter and return value types in the closure">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-02/src/main.rs:here}}
```

</Listing>

Po dodaniu adnotacji typu składnia zamknięć wygląda bardziej podobnie do
składni funkcji. Tutaj definiujemy funkcję, która dodaje 1 do swojego parametru i
zamknięcie, które ma takie samo zachowanie, dla porównania. Dodaliśmy kilka spacji,
aby wyrównać odpowiednie części. Ilustruje to, jak składnia zamknięć jest podobna
do składni funkcji, z wyjątkiem użycia potoków i ilości składni, która jest
opcjonalna:

```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

Pierwszy wiersz pokazuje definicję funkcji, a drugi wiersz pokazuje w pełni
adnotowaną definicję zamknięcia. W trzecim wierszu usuwamy adnotacje typu
z definicji zamknięcia. W czwartym wierszu usuwamy nawiasy, które
są opcjonalne, ponieważ ciało zamknięcia ma tylko jedno wyrażenie. Wszystkie te
są prawidłowymi definicjami, które będą generować takie samo zachowanie, gdy zostaną wywołane. Wiersze
`add_one_v3` i `add_one_v4` wymagają, aby zamknięcia zostały ocenione, aby
mogły zostać skompilowane, ponieważ typy zostaną wywnioskowane z ich użycia. Jest to
podobne do `let v = Vec::new();` wymagającego wstawienia adnotacji typu lub wartości
pewnego typu do `Vec`, aby Rust mógł wywnioskować typ.

W przypadku definicji zamknięć kompilator wywnioskuje jeden konkretny typ dla każdego z
ich parametrów i dla ich wartości zwracanej. Na przykład, Listing 13-3 pokazuje
definicję krótkiego zamknięcia, które po prostu zwraca wartość, którą otrzymuje jako
parametr. To zamknięcie nie jest zbyt przydatne, z wyjątkiem celów tego
przykładu. Należy zauważyć, że nie dodaliśmy żadnych adnotacji typu do definicji.
Ponieważ nie ma adnotacji typu, możemy wywołać zamknięcie z dowolnym typem,
co zrobiliśmy tutaj z `String` za pierwszym razem. Jeśli następnie spróbujemy wywołać
`example_closure` z liczbą całkowitą, otrzymamy błąd.

<Listing number="13-3" file-name="src/main.rs" caption="Attempting to call a closure whose types are inferred with two different types">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-03/src/main.rs:here}}
```

</Listing>

Kompilator zwraca nam następujący błąd:

```console
{{#include ../listings/ch13-functional-features/listing-13-03/output.txt}}
```

Gdy po raz pierwszy wywołujemy `example_closure` z wartością `String`, kompilator
wnioskuje, że typ `x` i typ zwracany zamknięcia to `String`. Te
typy są następnie blokowane w zamknięciu w `example_closure` i otrzymujemy błąd typu, gdy następnym razem próbujemy użyć innego typu z tym samym zamknięciem.

### Capturing References or Moving Ownership

Zamknięcia mogą przechwytywać wartości ze swojego otoczenia na trzy sposoby, które
bezpośrednio odwzorowują trzy sposoby, w jakie funkcja może przyjąć parametr: pożyczanie
niezmienne, pożyczanie zmienne i przejmowanie własności. Zamknięcie zdecyduje,
którego z nich użyć na podstawie tego, co ciało funkcji robi z
przechwyconymi wartościami.

W Liście 13-4 definiujemy zamknięcie, które przechwytuje niezmienne odwołanie do
wektora o nazwie `list`, ponieważ potrzebuje tylko niezmiennego odwołania, aby wydrukować
wartość:

<Listing number="13-4" file-name="src/main.rs" caption="Defining and calling a closure that captures an immutable reference">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-04/src/main.rs}}
```

</Listing>

Ten przykład ilustruje również, że zmienna może być powiązana z definicją zamknięcia,
i możemy później wywołać zamknięcie, używając nazwy zmiennej i nawiasów,
tak jakby nazwa zmiennej była nazwą funkcji.

Ponieważ możemy mieć wiele niezmiennych odwołań do `list` w tym samym czasie,
`list` jest nadal dostępna z kodu przed definicją zamknięcia, po
definicji zamknięcia, ale przed wywołaniem zamknięcia i po wywołaniu zamknięcia. Ten kod kompiluje się, uruchamia i drukuje:

```console
{{#include ../listings/ch13-functional-features/listing-13-04/output.txt}}
```

Następnie w Listingu 13-5 zmieniamy ciało zamknięcia, aby dodać element do wektora `list`. Zamknięcie teraz przechwytuje zmienną referencję:

<Listing number="13-5" file-name="src/main.rs" caption="Defining and calling a closure that captures a mutable reference">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-05/src/main.rs}}
```

</Listing>

Ten kod kompiluje się, uruchamia i drukuje:

```console
{{#include ../listings/ch13-functional-features/listing-13-05/output.txt}}
```

Zauważ, że nie ma już `println!` między definicją a wywołaniem
zamknięcia `borrows_mutably`: gdy `borrows_mutably` jest zdefiniowane, przechwytuje ono
zmienne odwołanie do `list`. Nie używamy zamknięcia ponownie po wywołaniu
zamknięcia, więc zmienne pożyczenie kończy się. Pomiędzy definicją zamknięcia a
wywołaniem zamknięcia niezmienne pożyczenie do print nie jest dozwolone, ponieważ żadne inne
pożyczenia nie są dozwolone, gdy istnieje zmienne pożyczenie. Spróbuj dodać tam `println!`
aby zobaczyć, jaki otrzymasz komunikat o błędzie!

Jeśli chcesz wymusić na zamknięciu przejęcie własności wartości, których używa w
środowisku, nawet jeśli ciało zamknięcia nie wymaga ściśle
własności, możesz użyć słowa kluczowego `move` przed listą parametrów.

Ta technika jest najbardziej przydatna podczas przekazywania zamknięcia do nowego wątku w celu przeniesienia
danych, tak aby były własnością nowego wątku. Omówimy wątki i dlaczego warto ich używać szczegółowo w rozdziale 16, kiedy będziemy mówić o współbieżności, ale na razie pokrótce przeanalizujmy tworzenie nowego wątku przy użyciu zamknięcia, które wymaga słowa kluczowego `move`. Listing 13-6 pokazuje zmodyfikowany Listing 13-4,
aby wydrukować wektor w nowym wątku, a nie w wątku głównym:

<Listing number="13-6" file-name="src/main.rs" caption="Using `move` to force the closure for the thread to take ownership of `list`">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-06/src/main.rs}}
```

</Listing>

Tworzymy nowy wątek, dając wątkowi zamknięcie do uruchomienia jako argument.
Treść zamknięcia drukuje listę. W Liście 13-4, zamknięcie przechwyciło tylko
`list` używając niezmiennego odwołania, ponieważ jest to najmniejszy dostęp
do `list` potrzebny do jej wydrukowania. W tym przykładzie, mimo że ciało zamknięcia
nadal potrzebuje tylko niezmiennego odwołania, musimy określić, że `list` powinno
zostać przeniesione do zamknięcia, umieszczając słowo kluczowe `move` na początku
definicji zamknięcia. Nowy wątek może zakończyć się przed zakończeniem reszty wątku głównego lub wątek główny może zakończyć się jako pierwszy. Gdyby wątek główny
zachował własność `list`, ale zakończył się przed zakończeniem nowego wątku i usunął
`list`, niezmienne odwołanie w wątku byłoby nieprawidłowe. Dlatego
kompilator wymaga, aby `list` zostało przeniesione do zamknięcia przekazanego nowemu wątkowi,
aby odwołanie było prawidłowe. Spróbuj usunąć słowo kluczowe `move` lub użyć `list`
w wątku głównym po zdefiniowaniu zamknięcia, aby zobaczyć, jakie błędy kompilatora otrzymasz!

<!-- Stare nagłówki. Nie usuwaj, bo linki mogą się zepsuć. -->
<a id="storing-closures-using-generic-parameters-and-the-fn-traits"></a>
<a id="limitations-of-the-cacher-implementation"></a>
<a id="moving-captured-values-out-of-the-closure-and-the-fn-traits"></a>

### Moving Captured Values Out of Closures and the `Fn` Traits

Gdy zamknięcie przechwyci odniesienie lub przejmie własność wartości ze
środowiska, w którym zamknięcie jest zdefiniowane (wpływając w ten sposób na to, co, jeśli cokolwiek,
zostanie przeniesione *do* zamknięcia), kod w ciele zamknięcia definiuje, co
dzieje się z odniesieniami lub wartościami, gdy zamknięcie jest później oceniane (w ten sposób
wpływając na to, co, jeśli cokolwiek, zostanie przeniesione *poza* zamknięcie). Ciało zamknięcia może
zrobić dowolną z następujących rzeczy: przenieść przechwyconą wartość poza zamknięcie, zmutować
przechwyconą wartość, nie przenosić ani nie mutować wartości lub nie przechwycić niczego ze
środowiska na początek.

Sposób, w jaki zamknięcie przechwytuje i obsługuje wartości ze środowiska, wpływa na to,
które cechy implementuje zamknięcie, a cechy to sposób, w jaki funkcje i struktury
mogą określać, jakich rodzajów zamknięć mogą używać. Zamknięcia automatycznie
implementują jedną, dwie lub wszystkie trzy z tych cech `Fn` w sposób addytywny,
w zależności od tego, jak ciało zamknięcia obsługuje wartości:

1. `FnOnce` dotyczy zamknięć, które można wywołać raz. Wszystkie zamknięcia implementują
przynajmniej tę cechę, ponieważ wszystkie zamknięcia można wywołać. Zamknięcie, które
przenosi przechwycone wartości poza swoje ciało, zaimplementuje tylko `FnOnce` i żadną
z innych cech `Fn`, ponieważ można je wywołać tylko raz.
2. `FnMut` dotyczy zamknięć, które nie przenoszą przechwyconych wartości poza swoje ciało, ale mogą mutować przechwycone wartości. Te zamknięcia można
wywołać więcej niż raz.
3. `Fn` dotyczy zamknięć, które nie przenoszą przechwyconych wartości poza swoje ciało
i które nie mutują przechwyconych wartości, a także zamknięć, które nie przechwytują
niczego ze swojego otoczenia. Te zamknięcia można wywołać więcej niż raz
bez zmiany ich środowiska, co jest ważne w przypadkach, takich jak
wielokrotne wywoływanie zamknięcia jednocześnie.

Przyjrzyjmy się definicji metody `unwrap_or_else` w `Option<T>`, której
użyliśmy w Liście 13-1:

```rust,ignore
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

Przypomnijmy, że `T` jest typem generycznym reprezentującym typ wartości w wariancie
`Some` `Option`. Ten typ `T` jest również typem zwracanym przez funkcję
`unwrap_or_else`: kod, który wywołuje `unwrap_or_else` na
`Option<String>`, na przykład, otrzyma `String`.

Następnie zauważ, że funkcja `unwrap_or_else` ma dodatkowy generyczny
parametr typu `F`. Typ `F` jest typem parametru o nazwie `f`, który jest
zamknięciem, które podajemy podczas wywoływania `unwrap_or_else`.

Ograniczenie cechy określone dla typu generycznego `F` to `FnOnce() -> T`, co
oznacza, że ​​`F` musi być możliwe do wywołania raz, nie przyjmować żadnych argumentów i zwracać `T`.
Użycie `FnOnce` w powiązaniu cech wyraża ograniczenie, że
`unwrap_or_else` wywoła `f` najwyżej raz. W treści
`unwrap_or_else` możemy zobaczyć, że jeśli `Option` to `Some`, `f` nie zostanie
wywołane. Jeśli `Option` to `None`, `f` zostanie wywołane raz. Ponieważ wszystkie
zamknięcia implementują `FnOnce`, `unwrap_or_else` akceptuje wszystkie trzy rodzaje
zamknięć i jest tak elastyczne, jak to tylko możliwe.

> Uwaga: Funkcje mogą również implementować wszystkie trzy cechy `Fn`. Jeśli to,
> co chcemy zrobić, nie wymaga przechwytywania wartości ze środowiska, możemy
> użyć nazwy funkcji zamiast zamknięcia, gdy potrzebujemy czegoś, co
> implementuje jedną z cech `Fn`. Na przykład, w przypadku wartości `Option<Vec<T>>`,
> moglibyśmy wywołać `unwrap_or_else(Vec::new)`, aby uzyskać nowy, pusty wektor, jeśli
> wartość wynosi `None`.

Teraz przyjrzyjmy się standardowej metodzie bibliotecznej `sort_by_key` zdefiniowanej w wycinkach,
aby zobaczyć, czym różni się ona od `unwrap_or_else` i dlaczego `sort_by_key` używa
`FnMut` zamiast `FnOnce` dla ograniczenia cechy. Zamknięcie otrzymuje jeden argument
w formie odwołania do bieżącego elementu w rozważanym wycinku,
i zwraca wartość typu `K`, którą można uporządkować. Ta funkcja jest przydatna,
gdy chcesz posortować wycinek według określonego atrybutu każdego elementu. W
Listingu 13-7 mamy listę instancji `Rectangle` i używamy `sort_by_key`,
aby uporządkować je według atrybutu `width` od najniższego do najwyższego:

<Listing number="13-7" file-name="src/main.rs" caption="Using `sort_by_key` to order rectangles by width">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-07/src/main.rs}}
```

</Listing>

This code prints:

```console
{{#include ../listings/ch13-functional-features/listing-13-07/output.txt}}
```

Powód, dla którego `sort_by_key` jest zdefiniowany tak, aby przyjmować zamknięcie `FnMut`, jest taki, że wywołuje zamknięcie
wielokrotnie: raz dla każdego elementu w wycinku. Zamknięcie `|r|
r.width` nie przechwytuje, nie mutuje ani nie przenosi niczego ze swojego otoczenia, więc
spełnia wymagania dotyczące ograniczeń cechy.

W przeciwieństwie do tego, Listing 13-8 pokazuje przykład zamknięcia, które implementuje tylko cechę `FnOnce`, ponieważ przenosi wartość poza otoczenie.
Kompilator nie pozwoli nam użyć tego zamknięcia z `sort_by_key`:

<Listing number="13-8" file-name="src/main.rs" caption="Attempting to use an `FnOnce` closure with `sort_by_key`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-08/src/main.rs}}
```

</Listing>

To wymyślony, zawiły sposób (który nie działa) na próbę zliczenia
ilości wywołań `sort_by_key` podczas sortowania `list`. Ten kod
próbuje wykonać to liczenie, wpychając `value`—`String` ze środowiska zamknięcia—do wektora `sort_operations`. Zamknięcie przechwytuje `value`
a następnie przenosi `value` z zamknięcia, przenosząc własność `value` do
wektora `sort_operations`. To zamknięcie można wywołać raz; próba wywołania
go po raz drugi nie zadziała, ponieważ `value` nie byłoby już w środowisku, aby ponownie zostać wciśnięte do `sort_operations`! Dlatego to zamknięcie
implementuje tylko `FnOnce`. Kiedy próbujemy skompilować ten kod, otrzymujemy ten błąd,
że `value` nie można przenieść z zamknięcia, ponieważ zamknięcie musi
implementować `FnMut`:

```console
{{#include ../listings/ch13-functional-features/listing-13-08/output.txt}}
```

Błąd wskazuje na linię w ciele zamknięcia, która przenosi `value` poza
środowisko. Aby to naprawić, musimy zmienić ciało zamknięcia tak, aby nie przenosiło wartości poza środowisko. Aby policzyć, ile razy zamknięcie
jest wywoływane, przechowywanie licznika w środowisku i zwiększanie jego wartości w ciele zamknięcia jest prostszym sposobem obliczenia tego. Zamknięcie
w Liście 13-9 działa z `sort_by_key`, ponieważ przechwytuje tylko zmienne
odwołanie do licznika `num_sort_operations` i dlatego może być wywoływane
więcej niż raz:

<Listing number="13-9" file-name="src/main.rs" caption="Using an `FnMut` closure with `sort_by_key` is allowed">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-09/src/main.rs}}
```

</Listing>

Cechy `Fn` są ważne podczas definiowania lub używania funkcji lub typów, które
wykorzystują zamknięcia. W następnej sekcji omówimy iteratory. Wiele
metod iteracyjnych przyjmuje argumenty zamknięcia, więc miej te szczegóły zamknięcia na uwadze,
podczas gdy będziemy kontynuować!

[unwrap-or-else]: ../std/option/enum.Option.html#method.unwrap_or_else
