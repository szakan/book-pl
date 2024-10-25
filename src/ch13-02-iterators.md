## Przetwarzanie serii elementów za pomocą iteratorów

Wzorzec iteratora pozwala na wykonywanie pewnych zadań na sekwencji elementów po kolei. Iterator odpowiada za logikę iterowania po każdym elemencie i
określania, kiedy sekwencja się zakończyła. Kiedy używasz iteratorów, nie musisz
samodzielnie ponownie implementować tej logiki.

W Rust iteratory są *leniwe*, co oznacza, że ​​nie mają żadnego efektu, dopóki nie wywołasz
metod, które zużywają iterator, aby go wykorzystać. Na przykład kod w
Listingu 13-10 tworzy iterator po elementach w wektorze `v1`, wywołując
metodę `iter` zdefiniowaną w `Vec<T>`. Ten kod sam w sobie nie robi niczego
użytecznego.

<Listing number="13-10" file-name="src/main.rs" caption="Creating an iterator">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-10/src/main.rs:here}}
```

</Listing>

Iterator jest przechowywany w zmiennej `v1_iter`. Po utworzeniu
iteratora możemy go używać na wiele sposobów. W Liście 3-5 w rozdziale 3,
iterowaliśmy po tablicy, używając pętli `for`, aby wykonać pewien kod na każdym z jej
elementów. W tle niejawnie tworzyło to, a następnie zużywało iterator,
ale do tej pory nie wyjaśnialiśmy, jak dokładnie to działa.

W przykładzie w Liście 13-11, oddzielamy tworzenie iteratora od
używania iteratora w pętli `for`. Gdy pętla `for` jest wywoływana przy użyciu
iteratora w `v1_iter`, każdy element w iteratorze jest używany w jednej
iteracji pętli, która drukuje każdą wartość.

<Listing number="13-11" file-name="src/main.rs" caption="Using an iterator in a `for` loop">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-11/src/main.rs:here}}
```

</Listing>

W językach, w których standardowe biblioteki nie udostępniają iteratorów,
prawdopodobnie napisałbyś tę samą funkcjonalność, rozpoczynając zmienną o indeksie
0, używając tej zmiennej do indeksowania wektora w celu uzyskania wartości i
zwiększając wartość zmiennej w pętli, aż osiągnie ona całkowitą liczbę
elementów w wektorze.

Iteratory obsługują całą tę logikę za Ciebie, ograniczając powtarzalny kod,
który potencjalnie mógłbyś zepsuć. Iteratory dają Ci większą elastyczność w używaniu tej samej
logiki z wieloma różnymi rodzajami sekwencji, a nie tylko strukturami danych, do których możesz
indeksować, jak wektory. Przyjrzyjmy się, jak iteratory to robią.

### The `Iterator` Trait and the `next` Method

Wszystkie iteratory implementują cechę o nazwie `Iterator`, która jest zdefiniowana w bibliotece standardowej. Definicja cechy wygląda następująco:

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

  // metody z domyślnymi implementacjami pominięte
}
```

Zauważ, że ta definicja używa nowej składni: `type Item` i `Self::Item`,
które definiują *typ powiązany* z tą cechą. Porozmawiamy o typach powiązanych szczegółowo w rozdziale 20. Na razie wszystko, co musisz wiedzieć, to to, że
ten kod mówi, że implementacja cechy `Iterator` wymaga również zdefiniowania
typu `Item`, a ten typ `Item` jest używany w typie zwracanym przez metodę `next`
. Innymi słowy, typ `Item` będzie typem zwróconym przez
iterator.

Cecha `Iterator` wymaga od implementatorów zdefiniowania tylko jednej metody: metody
`next`, która zwraca jeden element iteratora na raz, zawinięty w
`Some`, a po zakończeniu iteracji zwraca `None`.

Możemy wywołać metodę `next` bezpośrednio w iteratorach; Wylistowanie 13-12 pokazuje,
jakie wartości są zwracane w wyniku powtarzanych wywołań `next` w iteratorze utworzonym
z wektora.

<Listing number="13-12" file-name="src/lib.rs" caption="Calling the `next` method on an iterator">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-12/src/lib.rs:here}}
```

</Listing>

Zwróć uwagę, że musieliśmy uczynić `v1_iter` zmiennym: wywołanie metody `next` na
iteratorze zmienia stan wewnętrzny, którego iterator używa do śledzenia, gdzie
się znajduje w sekwencji. Innymi słowy, ten kod *konsumuje* lub zużywa,
iterator. Każde wywołanie `next` pochłania element z iteratora. Nie musieliśmy
czynić `v1_iter` zmiennym, gdy użyliśmy pętli `for`, ponieważ pętla przejęła
własność `v1_iter` i uczyniła ją zmienną w tle.

Zauważ również, że wartości, które otrzymujemy z wywołań `next`, są niezmiennymi
odniesieniami do wartości w wektorze. Metoda `iter` generuje iterator
nad niezmiennymi odniesieniami. Jeśli chcemy utworzyć iterator, który przejmuje
własność `v1` i zwraca posiadane wartości, możemy wywołać `into_iter` zamiast
`iter`. Podobnie, jeśli chcemy iterować po zmiennych referencjach, możemy wywołać
`iter_mut` zamiast `iter`.

### Methods that Consume the Iterator

Cecha `Iterator` ma wiele różnych metod z domyślnymi
implementacjami dostarczonymi przez bibliotekę standardową; możesz dowiedzieć się o tych
metodach, przeglądając dokumentację API biblioteki standardowej dla cechy `Iterator`. Niektóre z tych metod wywołują metodę `next` w swojej definicji, dlatego też musisz zaimplementować metodę `next` podczas implementowania cechy
`Iterator`.

Metody wywołujące `next` nazywane są *adapterami konsumującymi*, ponieważ ich wywołanie
zużywa iterator. Jednym z przykładów jest metoda `sum`, która przejmuje własność
iteratora i iteruje przez elementy, wielokrotnie wywołując `next`, w ten sposób
zużywając iterator. Podczas iteracji dodaje każdy element do bieżącej
sumy i zwraca sumę po zakończeniu iteracji. W wykazie 13-13 znajduje się
test ilustrujący użycie metody `sum`:

<Listing number="13-13" file-name="src/lib.rs" caption="Calling the `sum` method to get the total of all items in the iterator">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-13/src/lib.rs:here}}
```

</Listing>

Nie możemy użyć `v1_iter` po wywołaniu `sum`, ponieważ `sum` przejmuje własność iteratora, na którym go wywołaliśmy.

### Methods that Produce Other Iterators

*Adaptery iteratora* to metody zdefiniowane w cesze `Iterator`, które nie
konsumują iteratora. Zamiast tego produkują różne iteratory, zmieniając
pewien aspekt oryginalnego iteratora.

Listing 13-14 pokazuje przykład wywołania metody adaptera iteratora `map`,
która przyjmuje zamknięcie do wywołania dla każdego elementu podczas iterowania elementów.
Metoda `map` zwraca nowy iterator, który produkuje zmodyfikowane elementy. Zamknięcie tutaj tworzy nowy iterator, w którym każdy element z wektora zostanie
zwiększony o 1:

<Listing number="13-14" file-name="src/main.rs" caption="Calling the iterator adapter `map` to create a new iterator">

```rust,not_desired_behavior
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-14/src/main.rs:here}}
```

</Listing>

Jednak kod ten generuje ostrzeżenie:

```console
{{#include ../listings/ch13-functional-features/listing-13-14/output.txt}}
```

Kod w Listingu 13-14 nic nie robi; zamknięcie, które określiliśmy,
nigdy nie zostaje wywołane. Ostrzeżenie przypomina nam dlaczego: adaptery iteratorów są leniwe i
musimy tutaj skonsumować iterator.

Aby naprawić to ostrzeżenie i skonsumować iterator, użyjemy metody `collect`,
której użyliśmy w rozdziale 12 z `env::args` w Listingu 12-1. Ta metoda
konsumuje iterator i zbiera wynikowe wartości do typu danych kolekcji.

W Listingu 13-15 zbieramy wyniki iteracji po iteratorze, który
zwracany jest z wywołania `map` do wektora. Ten wektor ostatecznie
zawiera każdy element z oryginalnego wektora zwiększony o 1.

<Listing number="13-15" file-name="src/main.rs" caption="Calling the `map` method to create a new iterator and then calling the `collect` method to consume the new iterator and create a vector">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-15/src/main.rs:here}}
```

</Listing>

Ponieważ `map` przyjmuje zamknięcie, możemy określić dowolną operację, którą chcemy wykonać
na każdym elemencie. To świetny przykład tego, jak zamknięcia pozwalają dostosować pewne
zachowanie, jednocześnie ponownie wykorzystując zachowanie iteracji, które zapewnia cecha `Iterator`.

Możesz połączyć wiele wywołań adapterów iteratorów, aby wykonywać złożone działania w sposób
czytelny. Ale ponieważ wszystkie iteratory są leniwe, musisz wywołać jedną z
metod adaptera konsumującego, aby uzyskać wyniki z wywołań adapterów iteratorów.

### Using Closures that Capture Their Environment

Wiele adapterów iteratorów przyjmuje zamknięcia jako argumenty, a zazwyczaj zamknięcia, które określimy jako argumenty adapterów iteratorów, będą zamknięciami, które przechwytują ich środowisko.

W tym przykładzie użyjemy metody `filter`, która przyjmuje zamknięcie. Zamknięcie
pobiera element z iteratora i zwraca `bool`. Jeśli zamknięcie
zwróci `true`, wartość zostanie uwzględniona w iteracji wygenerowanej przez
`filter`. Jeśli zamknięcie zwróci `false`, wartość nie zostanie uwzględniona.

W Liście 13-16 używamy `filter` z zamknięciem, które przechwytuje zmienną `shoe_size`
z jej środowiska, aby iterować po kolekcji instancji struktury `Shoe`. Zwróci tylko buty o określonym rozmiarze.

<Listing number="13-16" file-name="src/lib.rs" caption="Using the `filter` method with a closure that captures `shoe_size`">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-16/src/lib.rs}}
```

</Listing>

Funkcja `shoes_in_size` przejmuje własność wektora butów i rozmiaru buta jako parametry. Zwraca wektor zawierający tylko buty o określonym
rozmiarze.

W treści `shoes_in_size` wywołujemy `into_iter`, aby utworzyć iterator,
który przejmuje własność wektora. Następnie wywołujemy `filter`, aby dostosować ten
iterator do nowego iteratora, który zawiera tylko elementy, dla których zamknięcie
zwraca `true`.

Zamknięcie przechwytuje parametr `shoe_size` ze środowiska i
porównuje wartość z rozmiarem każdego buta, zachowując tylko buty o określonym
rozmiarze. Na koniec wywołanie `collect` zbiera wartości zwrócone przez
dostosowany iterator do wektora, który jest zwracany przez funkcję.

Test pokazuje, że gdy wywołujemy `shoes_in_size`, otrzymujemy tylko buty,
które mają taki sam rozmiar jak określona przez nas wartość.
