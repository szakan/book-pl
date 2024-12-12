## Korzystanie z obiektów cech, które dopuszczają wartości różnych typów

W rozdziale 8 wspomnieliśmy, że jednym z ograniczeń wektorów jest to, że mogą one
przechowywać elementy tylko jednego typu. Stworzyliśmy obejście w Liście 8-9, gdzie
zdefiniowaliśmy enum `SpreadsheetCell`, które miało warianty do przechowywania liczb całkowitych, zmiennoprzecinkowych,
i tekstu. Oznaczało to, że mogliśmy przechowywać różne typy danych w każdej komórce i
nadal mieć wektor, który reprezentował wiersz komórek. Jest to doskonałe
rozwiązanie, gdy nasze wymienne elementy są stałym zestawem typów, które znamy,
gdy kompilujemy nasz kod.

Czasami jednak chcemy, aby nasz użytkownik biblioteki mógł rozszerzyć zestaw
typów, które są prawidłowe w określonej sytuacji. Aby pokazać, jak możemy to osiągnąć,
utworzymy przykładowe narzędzie graficznego interfejsu użytkownika (GUI), które iteruje
po liście elementów, wywołując metodę `draw` dla każdego z nich, aby narysować go na
ekranie — typowa technika dla narzędzi GUI. Utworzymy skrzynkę biblioteczną o nazwie
`gui`, która zawiera strukturę biblioteki GUI. Ta skrzynia może zawierać
pewne typy do wykorzystania przez ludzi, takie jak `Button` lub `TextField`. Ponadto,
użytkownicy `gui` będą chcieli tworzyć własne typy, które można rysować: na przykład, jeden programista może dodać `Image`, a inny może dodać
`SelectBox`.

Nie zaimplementujemy w pełni rozwiniętej biblioteki GUI dla tego przykładu, ale pokażemy,
jak poszczególne elementy będą do siebie pasować. W momencie pisania biblioteki nie możemy
znać i zdefiniować wszystkich typów, które inni programiści mogą chcieć utworzyć. Ale wiemy,
że `gui` musi śledzić wiele wartości różnych typów i
musi wywołać metodę `draw` dla każdej z tych wartości o różnym typie. Nie musi
dokładnie wiedzieć, co się stanie, gdy wywołamy metodę `draw`,
tylko, że wartość będzie miała dostępną metodę do wywołania.

Aby to zrobić w języku z dziedziczeniem, moglibyśmy zdefiniować klasę o nazwie
`Component`, która ma metodę o nazwie `draw`. Pozostałe klasy, takie jak
`Button`, `Image` i `SelectBox`, dziedziczyłyby po `Component` i w ten sposób
dziedziczyłyby metodę `draw`. Każda z nich mogłaby zastąpić metodę `draw`, aby zdefiniować
swoje niestandardowe zachowanie, ale framework mógłby traktować wszystkie typy tak, jakby
były instancjami `Component` i wywoływać na nich `draw`. Ale ponieważ Rust
nie ma dziedziczenia, potrzebujemy innego sposobu na ustrukturyzowanie biblioteki `gui`, aby
umożliwić użytkownikom rozszerzanie jej o nowe typy.

### Definiowanie cechy dla typowego zachowania

Aby zaimplementować zachowanie, jakie chcemy, aby miało `gui`, zdefiniujemy cechę o nazwie
`Draw`, która będzie miała jedną metodę o nazwie `draw`. Następnie możemy zdefiniować wektor, który
przyjmuje *obiekt cechy*. Obiekt cechy wskazuje zarówno na wystąpienie typu
implementującego naszą określoną cechę, jak i na tabelę używaną do wyszukiwania metod cech w
tym typie w czasie wykonywania. Tworzymy obiekt cechy, określając jakiś rodzaj
wskaźnika, taki jak odwołanie `&` lub inteligentny wskaźnik `Box<T>`, następnie słowo kluczowe `dyn`, a następnie określając odpowiednią cechę. (Omówimy powód, dla którego
obiekty cech muszą używać wskaźnika w rozdziale 20 w sekcji [„Typy o dynamicznym
rozmiarze i cecha o `rozmiarze`.”][dynamically-sized]<!-- ignore -->) Możemy
używać obiektów cech zamiast typu ogólnego lub konkretnego. Gdziekolwiek używamy
obiektu cechy, system typów Rust zapewni w czasie kompilacji, że każda wartość
użyta w tym kontekście zaimplementuje cechę obiektu cechy. W związku z tym
nie musimy znać wszystkich możliwych typów w czasie kompilacji.

Wspomnieliśmy, że w Rust powstrzymujemy się od nazywania struktur i wyliczeń
„obiektami”, aby odróżnić je od obiektów innych języków. W strukturze lub
wyliczeniu dane w polach struktury i zachowanie w blokach `impl` są
oddzielone, podczas gdy w innych językach dane i zachowanie połączone w jeden
koncept są często określane jako obiekt. Jednak obiekty cech *są* bardziej podobne
do obiektów w innych językach w tym sensie, że łączą dane i zachowanie.
Jednak obiekty cech różnią się od tradycyjnych obiektów tym, że nie możemy dodawać danych do
obiektu cechy. Obiekty cech nie są tak przydatne jak obiekty w innych
językach: ich konkretnym celem jest umożliwienie abstrakcji w ramach typowego
zachowania.

Listing 18-3 pokazuje, jak zdefiniować cechę o nazwie `Draw` za pomocą jednej metody o nazwie
`draw`:

<Listing number="18-3" file-name="src/lib.rs" caption="Definition of the `Draw` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-03/src/lib.rs}}
```

</Listing>

Ta składnia powinna wyglądać znajomo z naszych dyskusji na temat definiowania cech
w rozdziale 10. Następnie pojawia się nowa składnia: Listing 18-4 definiuje strukturę o nazwie
`Screen`, która zawiera wektor o nazwie `components`. Ten wektor jest typu
`Box<dyn Draw>`, który jest obiektem cechy; jest on zamiennikiem dowolnego typu wewnątrz
`Box`, który implementuje cechę `Draw`.

<Listing number="18-4" file-name="src/lib.rs" caption="Definition of the `Screen` struct with a `components` field holding a vector of trait objects that implement the `Draw` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-04/src/lib.rs:here}}
```

</Listing>

On the `Screen` struct, we’ll define a method named `run` that will call the
`draw` method on each of its `components`, as shown in Listing 18-5:

<Listing number="18-5" file-name="src/lib.rs" caption="A `run` method on `Screen` that calls the `draw` method on each component">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-05/src/lib.rs:here}}
```

</Listing>

Działa to inaczej niż definiowanie struktury, która używa parametru typu generycznego
z ograniczeniami cech. Parametr typu generycznego można zastąpić
tylko jednym typem konkretnym na raz, podczas gdy obiekty cech pozwalają na użycie wielu typów konkretnych do wypełnienia obiektu cechy w czasie wykonywania. Na przykład, moglibyśmy zdefiniować strukturę `Screen`, używając typu generycznego i ograniczenia cech,
jak w Liście 18-6:

<Listing number="18-6" file-name="src/lib.rs" caption="An alternate implementation of the `Screen` struct and its `run` method using generics and trait bounds">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-06/src/lib.rs:here}}
```

</Listing>

Ogranicza nas to do instancji `Screen`, która ma listę komponentów, wszystkie typu `Button` lub wszystkie typu `TextField`. Jeśli będziesz mieć tylko jednorodne
kolekcje, używanie generyków i granic cech jest lepsze, ponieważ
definicje zostaną zmonmorfizowane w czasie kompilacji, aby użyć konkretnych typów.

Z drugiej strony, przy metodzie używającej obiektów cech, jedna instancja `Screen`
może zawierać `Vec<T>`, która zawiera `Box<Button>`, a także
`Box<TextField>`. Przyjrzyjmy się, jak to działa, a następnie omówimy
implikacje wydajnościowe w czasie wykonywania.

### Wdrażanie cechy

Teraz dodamy kilka typów, które implementują cechę `Draw`. Dostarczymy typ
`Button`. Ponownie, implementacja biblioteki GUI wykracza poza zakres
tej książki, więc metoda `draw` nie będzie miała żadnej użytecznej implementacji w swoim
body. Aby wyobrazić sobie, jak mogłaby wyglądać implementacja, struktura `Button`
może mieć pola dla `szerokości`, `wysokości` i `etykiety`, jak pokazano w Liście 18-7:

<Listing number="18-7" file-name="src/lib.rs" caption="A `Button` struct that implements the `Draw` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-07/src/lib.rs:here}}
```

</Listing>

Pola `width`, `height`, i `label` w `Button`  będą się różnić od pól
w innych komponentach; na przykład typ `TextField` może mieć te same pola plus pole `placeholder`. Każdy z typów, które chcemy narysować
na ekranie, zaimplementuje cechę `Draw`, ale użyje innego kodu w metodzie
`draw`, aby zdefiniować sposób rysowania tego konkretnego typu, tak jak ma to miejsce w przypadku `Button`
(bez faktycznego kodu GUI, jak wspomniano). Na przykład typ `Button`
może mieć dodatkowy blok `impl` zawierający metody związane z tym, co
się dzieje, gdy użytkownik kliknie przycisk. Tego rodzaju metody nie będą miały zastosowania do
typów takich jak `TextField`.

Jeśli ktoś korzystający z naszej biblioteki zdecyduje się zaimplementować strukturę `SelectBox`, która ma pola
`width`, `height` i `options`, zaimplementuje również cechę `Draw` w typie
`SelectBox`, jak pokazano na Liście 18-8:

<Listing number="18-8" file-name="src/main.rs" caption="Another crate using `gui` and implementing the `Draw` trait on a `SelectBox` struct">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-08/src/main.rs:here}}
```

</Listing>

Użytkownik naszej biblioteki może teraz napisać swoją funkcję `main`, aby utworzyć instancję `Screen`
. Do instancji `Screen` może dodać `SelectBox` i `Button`
umieszczając każdy z nich w `Box<T>`, aby stał się obiektem cechy. Następnie może wywołać metodę
`run` na instancji `Screen`, która wywoła `draw` na każdym z
komponentów. Listing 18-9 pokazuje tę implementację:

<Listing number="18-9" file-name="src/main.rs" caption="Using trait objects to store values of different types that implement the same trait">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-09/src/main.rs:here}}
```

</Listing>

Kiedy pisaliśmy bibliotekę, nie wiedzieliśmy, że ktoś może dodać typ
`SelectBox`, ale nasza implementacja `Screen` była w stanie operować na
nowym typie i rysować go, ponieważ `SelectBox` implementuje cechę `Draw`, co
oznacza, że ​​implementuje metodę `draw`.

Ta koncepcja — skupiania się tylko na komunikatach, na które odpowiada wartość,
a nie na konkretnym typie wartości — jest podobna do koncepcji *duck
typing* w językach dynamicznie typowanych: jeśli chodzi jak kaczka i kwacze
jak kaczka, to musi być kaczką! W implementacji `run` na `Screen`
w Liście 18-5, `run` nie musi wiedzieć, jaki jest konkretny typ każdego
komponentu. Nie sprawdza, czy komponent jest instancją `Button`
lub `SelectBox`, po prostu wywołuje metodę `draw` na komponencie. Określając `Box<dyn Draw>` jako typ wartości w wektorze `components`
, zdefiniowaliśmy `Screen` jako potrzebujący wartości, na których możemy wywołać metodę `draw`
.

Zaletą korzystania z obiektów cech i systemu typów Rust do pisania kodu
podobnego do kodu używającego duck typing jest to, że nigdy nie musimy sprawdzać, czy
wartość implementuje określoną metodę w czasie wykonywania lub martwić się o otrzymanie błędów,
jeśli wartość nie implementuje metody, ale i tak ją wywołamy. Rust nie skompiluje
naszego kodu, jeśli wartości nie implementują cech, których potrzebują obiekty cech.

Na przykład Listing 18-10 pokazuje, co się stanie, jeśli spróbujemy utworzyć `Screen`
z `String` jako komponentem:

<Listing number="18-10" file-name="src/main.rs" caption="Attempting to use a type that doesn’t implement the trait object’s trait">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-10/src/main.rs}}
```

</Listing>

We’ll get this error because `String` doesn’t implement the `Draw` trait:

```console
{{#include ../listings/ch18-oop/listing-18-10/output.txt}}
```

This error lets us know that either we’re passing something to `Screen` we
didn’t mean to pass and so should pass a different type or we should implement
`Draw` on `String` so that `Screen` is able to call `draw` on it.

### Obiekty cech wykonują dynamiczną dyspozytornię

Przypomnij sobie w sekcji [„Wydajność kodu przy użyciu
generyków”][performance-of-code-using-generics]<!-- ignoruj ​​--> w
Rozdziale 10 naszą dyskusję na temat procesu monomorfizacji wykonywanego przez
kompilator, gdy używamy granic cech dla typów generycznych: kompilator generuje
niegeneryczne implementacje funkcji i metod dla każdego konkretnego typu, którego
używamy zamiast parametru typu generycznego. Kod, który jest wynikiem
monomorfizacji, wykonuje *statyczną dyspozytornię*, co ma miejsce, gdy kompilator wie,
jaką metodę wywołujesz w czasie kompilacji. Jest to przeciwieństwo *dynamicznej dyspozytorni*, która ma miejsce, gdy kompilator nie może stwierdzić w czasie kompilacji, którą metodę
wywołujesz. W przypadkach dynamicznej dyspozytorni kompilator emituje kod, który w czasie wykonywania ustali,
którą metodę wywołać.

Gdy używamy obiektów cech, Rust musi używać dynamicznej dyspozytorni. Kompilator nie
zna wszystkich typów, które mogą być używane z kodem, który używa obiektów cech,
więc nie wie, która metoda zaimplementowana w którym typie ma zostać wywołana. Zamiast tego, w
czasie wykonywania, Rust używa wskaźników wewnątrz obiektu cechy, aby wiedzieć, którą metodę ma zostać wywołana. To wyszukiwanie powoduje koszt wykonania, który nie występuje w przypadku statycznego
rozsyłania. Dynamiczne rozsyłanie uniemożliwia również kompilatorowi wybór inline kodu
metody, co z kolei uniemożliwia pewne optymalizacje. Jednak uzyskaliśmy
dodatkową elastyczność w kodzie, który napisaliśmy w Liście 18-5 i mogliśmy
obsługiwać w Liście 18-9, więc jest to kompromis, który należy rozważyć.

[performance-of-code-using-generics]:
ch10-01-syntax.html#performance-of-code-using-generics
[dynamically-sized]: ch20-04-advanced-types.html#dynamically-sized-types-and-the-sized-trait
