## Ulepszanie naszego projektu I/O

Dzięki tej nowej wiedzy o iteratorach możemy ulepszyć projekt I/O w
rozdziale 12, używając iteratorów, aby miejsca w kodzie były bardziej przejrzyste i
zwięzłe. Przyjrzyjmy się, w jaki sposób iteratory mogą ulepszyć naszą implementację funkcji
`Config::build` i funkcji `search`.

### Usuwanie `clone` za pomocą iteratora

W Liście 12-6 dodaliśmy kod, który pobierał fragment wartości `String` i tworzył
instancję struktury `Config` poprzez indeksowanie fragmentu i klonowanie
wartości, umożliwiając strukturze `Config` posiadanie tych wartości. W Liście 13-17 odtworzyliśmy implementację funkcji `Config::build`, tak jak była
w Liście 12-23:

<Listing number="13-17" file-name="src/lib.rs" caption="Reproduction of the `Config::build` function from Listing 12-23">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-23-reproduced/src/lib.rs:ch13}}
```

</Listing>

Wtedy powiedzieliśmy, żeby nie martwić się nieefektywnymi wywołaniami `clone`, ponieważ
usuniemy je w przyszłości. Cóż, ten czas nadszedł!

Potrzebowaliśmy tutaj `clone`, ponieważ mamy fragment z elementami `String` w
parametrze `args`, ale funkcja `build` nie jest właścicielem `args`. Aby zwrócić
własność instancji `Config`, musieliśmy sklonować wartości z pól `query`
i `file_path` instancji `Config`, aby instancja `Config` mogła posiadać swoje wartości.

Dzięki naszej nowej wiedzy o iteratorach możemy zmienić funkcję `build`, aby
przejęła własność iteratora jako swojego argumentu zamiast pożyczać fragment.
Zamiast kodu, który sprawdza długość fragmentu i indeksuje do określonych lokalizacji, użyjemy funkcjonalności iteratora. Wyjaśni to, co robi
funkcja `Config::build`, ponieważ iterator uzyska dostęp do wartości.

Gdy `Config::build` przejmie własność iteratora i przestanie używać operacji indeksowania, które pożyczają, możemy przenieść wartości `String` z iteratora do
`Config` zamiast wywoływać `clone` i dokonywać nowego przydziału.

#### Bezpośrednie używanie zwróconego iteratora

Otwórz plik *src/main.rs* swojego projektu I/O, który powinien wyglądać następująco:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-24-reproduced/src/main.rs:ch13}}
```

Najpierw zmienimy początek funkcji `main`, którą mieliśmy w Listingu
12-24, na kod z Listingu 13-18, który tym razem używa iteratora. To
nie skompiluje się, dopóki nie zaktualizujemy również `Config::build`.

<Listing number="13-18" file-name="src/main.rs" caption="Passing the return value of `env::args` to `Config::build`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-18/src/main.rs:here}}
```

</Listing>

Funkcja `env::args` zwraca iterator! Zamiast zbierać
wartości iteratora do wektora, a następnie przekazywać wycinek do `Config::build`, teraz
przekazujemy własność iteratora zwróconego z `env::args` bezpośrednio do
`Config::build`.

Następnie musimy zaktualizować definicję `Config::build`. W pliku *src/lib.rs*
projektu I/O zmieńmy sygnaturę `Config::build`, aby
wyglądała jak Listing 13-19. Nadal się to nie skompiluje, ponieważ musimy zaktualizować
ciało funkcji.

<Listing number="13-19" file-name="src/lib.rs" caption="Updating the signature of `Config::build` to expect an iterator">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-19/src/lib.rs:here}}
```

</Listing>

Standardowa dokumentacja biblioteki dla funkcji `env::args` pokazuje, że
typ iteratora, który zwraca, to `std::env::Args`, a ten typ implementuje
cechę `Iterator` i zwraca wartości `String`.

Zaktualizowaliśmy sygnaturę funkcji `Config::build`, więc parametr
`args` ma typ ogólny z cechami ograniczonymi przez `impl Iterator<Item = String>`
zamiast `&[String]`. To użycie składni `impl Trait`, którą omówiliśmy w sekcji
[„Cechy jako parametry”][impl-trait]<!-- ignore --> rozdziału 10
oznacza, że ​​`args` może być dowolnym typem, który implementuje cechę `Iterator` i
zwraca elementy `String`.

Ponieważ przejmujemy własność `args` i będziemy mutować `args` poprzez
iterację po nim, możemy dodać słowo kluczowe `mut` do specyfikacji parametru
`args`, aby uczynić go zmiennym.

#### Używanie metod cech `Iterator` zamiast indeksowania

Następnie naprawimy treść `Config::build`. Ponieważ `args` implementuje cechę
`Iterator`, wiemy, że możemy wywołać na niej metodę `next`! Listing 13-20
aktualizuje kod z Listingu 12-23, aby użyć metody `next`:

<Listing number="13-20" file-name="src/lib.rs" caption="Changing the body of `Config::build` to use iterator methods">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-20/src/lib.rs:here}}
```

</Listing>

Pamiętaj, że pierwszą wartością w wartości zwracanej `env::args` jest nazwa
programu. Chcemy to zignorować i przejść do następnej wartości, więc najpierw wywołujemy
`next` i nic nie robimy ze zwracaną wartością. Po drugie, wywołujemy `next`, aby uzyskać
wartość, którą chcemy umieścić w polu `query` `Config`. Jeśli `next` zwróci
`Some`, używamy `match`, aby wyodrębnić wartość. Jeśli zwróci `None`, oznacza to, że
nie podano wystarczającej liczby argumentów i zwracamy wcześniej wartość `Err`. Robimy
to samo dla wartości `file_path`.

### Uczynienie kodu bardziej przejrzystym dzięki adapterom Iterator

Możemy również skorzystać z iteratorów w funkcji `search` w naszym projekcie I/O, która jest tutaj odtworzona w Liście 13-21, tak jak była w Liście 12-19:

<Listing number="13-21" file-name="src/lib.rs" caption="The implementation of the `search` function from Listing 12-19">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:ch13}}
```

</Listing>

Możemy napisać ten kod w bardziej zwięzły sposób, używając metod adaptera iteratora.
Dzięki temu unikniemy również posiadania zmiennego pośredniego wektora `results`.
Styl programowania funkcyjnego preferuje minimalizowanie ilości zmiennego stanu, aby
uczynić kod bardziej przejrzystym. Usunięcie zmiennego stanu może umożliwić przyszłe udoskonalenie,
aby wyszukiwanie odbywało się równolegle, ponieważ nie musielibyśmy zarządzać
współbieżnym dostępem do wektora `results`. Listing 13-22 pokazuje tę zmianę:

<Listing number="13-22" file-name="src/lib.rs" caption="Using iterator adapter methods in the implementation of the `search` function">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-22/src/lib.rs:here}}
```

</Listing>

Przypomnijmy, że celem funkcji `search` jest zwrócenie wszystkich wierszy w
`contents`, które zawierają `query`. Podobnie jak w przykładzie `filter` w Listingu
13-16, ten kod używa adaptera `filter`, aby zachować tylko wiersze, dla których
`line.contains(query)` zwraca `true`. Następnie zbieramy pasujące wiersze
do innego wektora za pomocą `collect`. Znacznie prościej! Możesz również wprowadzić tę samą
zmianę, aby użyć metod iteratora w funkcji `search_case_insensitive`.
### Wybór pomiędzy pętlami a iteratorami

Następnym logicznym pytaniem jest, jaki styl powinieneś wybrać w swoim kodzie i
dlaczego: oryginalną implementację z Listingu 13-21 czy wersję wykorzystującą
iteratory z Listingu 13-22. Większość programistów Rust woli używać stylu iteratora. Na początku jest trochę trudniej się z nim oswoić, ale gdy już
zapoznasz się z różnymi adapterami iteratorów i tym, co robią, iteratory mogą być łatwiejsze do
zrozumienia. Zamiast bawić się różnymi fragmentami pętli i budować
nowe wektory, kod koncentruje się na ogólnym celu pętli. To
abstrahuje część powszechnego kodu, więc łatwiej jest zobaczyć koncepcje,
które są unikalne dla tego kodu, takie jak warunek filtrowania, który każdy element w
iteratorze musi spełnić.

Ale czy te dwie implementacje są naprawdę równoważne? Intuicyjne założenie
może być takie, że pętla niższego poziomu będzie szybsza. Porozmawiajmy o
wydajności.

[impl-trait]: ch10-02-traits.html#traits-as-parameters
