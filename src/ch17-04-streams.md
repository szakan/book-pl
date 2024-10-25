## Strumienie

Do tej pory w tym rozdziale trzymaliśmy się głównie indywidualnych przyszłości. Jednym dużym
wyjątkiem był używany przez nas kanał asynchroniczny. Przypomnij sobie, jak używaliśmy odbiornika dla naszego
kanału asynchronicznego w [„Message Passing”][17-02-messages] wcześniej w tym rozdziale.
Asynchroniczna metoda `recv` generuje sekwencję elementów w czasie. Jest to
przykład znacznie bardziej ogólnego wzorca, często nazywanego *strumieniem*.

Sekwencja elementów to coś, co widzieliśmy wcześniej, gdy przyjrzeliśmy się cesze
`Iterator` w rozdziale 13, ale istnieją dwie różnice między iteratorami
a odbiornikiem kanału asynchronicznego. Pierwsza różnica to element czasu:
iteratory są synchroniczne, podczas gdy odbiornik kanału jest asynchroniczny.
Druga różnica to API. Pracując bezpośrednio z `Iteratorem`, nazywamy
jego synchroniczną metodą `next`. W przypadku strumienia `trpl::Receiver` w szczególności
wywołaliśmy asynchroniczną metodę `recv`, ale poza tym te API wydają się
bardzo podobne.

To podobieństwo nie jest przypadkiem. Strumień jest podobny do asynchronicznej
formy iteracji. Podczas gdy `trpl::Receiver` konkretnie czeka na odebranie
wiadomości, API strumienia ogólnego przeznaczenia jest znacznie bardziej ogólne:
udostępnia następny element w taki sam sposób, jak `Iterator`, ale asynchronicznie. Podobieństwo między iteratorami i strumieniami w Rust oznacza, że ​​możemy faktycznie utworzyć strumień
z dowolnego iteratora. Podobnie jak w przypadku iteratora, możemy pracować ze strumieniem,
wywołując jego metodę `next`, a następnie czekając na wynik, jak w Liście 17-30.

<Listing number="17-30" caption="Creating a stream from an iterator and printing its values" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-30/src/main.rs:stream}}
```

</Listing>

Zaczynamy od tablicy liczb, którą konwertujemy na iterator, a następnie wywołujemy
`map`, aby podwoić wszystkie wartości. Następnie konwertujemy iterator na strumień
używając funkcji `trpl::stream_from_iter`. Następnie przechodzimy przez elementy w strumieniu, gdy tylko się pojawią, za pomocą pętli `while let`.

Niestety, gdy próbujemy uruchomić kod, nie kompiluje się. Zamiast tego, jak
możemy zobaczyć na wyjściu, raportuje, że nie ma dostępnej metody `next`.

<!-- TODO: fix up the path here? -->
<!-- manual-regeneration
cd listings/chapter-17-async-await/listing-17-30
cargo build
copy only the error output
-->

```console
error[E0599]: no method named `next` found for struct `Iter` in the current scope
 --> src/main.rs:8:40
  |
8 |         while let Some(value) = stream.next().await {
  |                                        ^^^^
  |
  = note: the full type name has been written to '/Users/chris/dev/rust-lang/book/listings/ch17-async-await/listing-17-30/target/debug/deps/async_await-bbd5bb8f6851cb5f.long-type-18426562901668632191.txt'
  = note: consider using `--verbose` to print the full type name to the console
  = help: items from traits can only be used if the trait is in scope
help: the following traits which provide `next` are implemented but not in scope; perhaps you want to import one of them
  |
1 + use futures_util::stream::stream::StreamExt;
  |
1 + use std::iter::Iterator;
  |
1 + use std::str::pattern::Searcher;
  |
1 + use trpl::StreamExt;
  |
help: there is a method `try_next` with a similar name
  |
8 |         while let Some(value) = stream.try_next().await {
  |                                        ~~~~~~~~

For more information about this error, try `rustc --explain E0599`.
```

Jak sugeruje wynik, powodem błędu kompilatora jest to, że potrzebujemy
odpowiedniej cechy w zakresie, aby móc użyć metody `next`. Biorąc pod uwagę naszą dotychczasową dyskusję, można by się rozsądnie spodziewać, że będzie to `Stream`, ale cechą, której tutaj potrzebujemy, jest w rzeczywistości `StreamExt`. `Ext` oznacza „rozszerzenie”: jest to
powszechny wzorzec w społeczności Rust służący do rozszerzania jednej cechy o inną.

Dlaczego potrzebujemy `StreamExt` zamiast `Stream` i co sama cecha `Stream` robi? Krótko mówiąc, odpowiedź jest taka, że ​​w całym ekosystemie Rust cecha
`Stream` definiuje interfejs niskiego poziomu, który skutecznie łączy cechy
`Iterator` i `Future`. Cecha `StreamExt` dostarcza zestaw interfejsów API wyższego poziomu na `Stream`, w tym metodę `next`, a także inne metody narzędziowe podobne do tych, które zapewnia cecha `Iterator`. Wrócimy
do cech `Stream` i `StreamExt` nieco bardziej szczegółowo pod koniec
rozdziału. Na razie to wystarczy, abyśmy mogli kontynuować.

Poprawką błędu kompilatora jest dodanie instrukcji `use` dla `trpl::StreamExt`,
jak w Listingu 17-31.

<Listing number="17-31" caption="Successfully using an iterator as the basis for a stream" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-31/src/main.rs:all}}
```

</Listing>

Po połączeniu wszystkich tych elementów kod działa tak, jak chcemy! Co więcej, teraz, gdy mamy `StreamExt` w zakresie, możemy używać wszystkich jego metod użytkowych,
tak jak w przypadku iteratorów. Na przykład w listingu 17-32 używamy metody
`filter`, aby odfiltrować wszystko oprócz wielokrotności trzech i pięciu.

<Listing number="17-32" caption="Filtering a `Stream` with the `StreamExt::filter` method" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-32/src/main.rs:all}}
```

</Listing>

Oczywiście, to nie jest zbyt interesujące. Moglibyśmy to zrobić za pomocą normalnych iteratorów
i bez żadnej asynchroniczności. Przyjrzyjmy się więc innym rzeczom, które możemy
zrobić, a które są unikalne dla strumieni.

### Composing Strumienie

Wiele koncepcji jest naturalnie reprezentowanych jako strumienie: elementy stają się dostępne w
kolejce lub pracują z większą ilością danych, niż mieści się w pamięci komputera, poprzez
wyciąganie ich fragmentów z systemu plików na raz lub dane docierające przez
sieć w czasie. Ponieważ strumienie są przyszłościami, możemy ich używać również z dowolnym innym
rodzajem przyszłości i możemy je łączyć w interesujący sposób. Na przykład,
możemy grupować zdarzenia, aby uniknąć wyzwalania zbyt wielu wywołań sieciowych, ustawiać limity czasu
w sekwencjach długotrwałych operacji lub ograniczać zdarzenia interfejsu użytkownika, aby
unikać wykonywania niepotrzebnej pracy.

Zacznijmy od zbudowania małego strumienia wiadomości, jako zastępstwa dla strumienia
danych, które możemy zobaczyć z WebSocket lub innego protokołu komunikacji w czasie rzeczywistym. W Liście 17-33 tworzymy funkcję `get_messages`, która zwraca
`impl Stream<Item = String>`. W celu jego implementacji tworzymy asynchroniczny
kanał, przechodzimy przez pierwsze dziesięć liter alfabetu angielskiego i wysyłamy je
przez kanał.

Używamy również nowego typu: `ReceiverStream`, który konwertuje odbiornik `rx` z
`trpl::channel` na `Stream` za pomocą metody `next`. Wracając do `main`, używamy
pętli `while let`, aby wydrukować wszystkie wiadomości ze strumienia.

<Listing number="17-33" caption="Using the `rx` receiver as a `ReceiverStream`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-33/src/main.rs:all}}
```

</Listing>

When we run this code, we get exactly the results we would expect:

<!-- Nie wyodrębniam danych wyjściowych, ponieważ zmiany w tych danych wyjściowych nie są znaczące;
zmiany te prawdopodobnie wynikają z faktu, że wątki działają inaczej, a nie ze
zmian w kompilatorze -->

```text
Message: 'a'
Message: 'b'
Message: 'c'
Message: 'd'
Message: 'e'
Message: 'f'
Message: 'g'
Message: 'h'
Message: 'i'
Message: 'j'
```

Moglibyśmy to zrobić za pomocą zwykłego API `Receiver`, lub nawet zwykłego API `Iterator`. Dodajmy coś, co wymaga strumieni: dodanie limitu czasu,
który dotyczy każdego elementu w strumieniu, i opóźnienia dla elementów, które emitujemy.

W Liście 17-34 zaczynamy od dodania limitu czasu do strumienia za pomocą metody `timeout`,
która pochodzi z cechy `StreamExt`. Następnie aktualizujemy treść pętli
`while let`, ponieważ strumień zwraca teraz `Result`. Wariant `Ok`
oznacza, że ​​wiadomość dotarła w czasie; wariant `Err` oznacza, że ​​limit czasu upłynął przed dotarciem jakiejkolwiek wiadomości. `Dopasowujemy` ten wynik i albo
drukujemy wiadomość, gdy ją pomyślnie otrzymamy, albo drukujemy powiadomienie o
limitie czasu. Na koniec zauważ, że przypinamy wiadomości po zastosowaniu do nich limitu czasu,
ponieważ pomocnik limitu czasu generuje strumień, który musi zostać przypięty, aby
można go było sondować.

<Listing number="17-34" caption="Using the `StreamExt::timeout` method to set a time limit on the items in a stream" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-34/src/main.rs:timeout}}
```

</Listing>

Jednakże, ponieważ nie ma opóźnień między wiadomościami, ten limit czasu nie
zmienia zachowania programu. Dodajmy zmienne opóźnienie do wiadomości,
które wysyłamy. W `get_messages` używamy metody iteratora `enumerate` z tablicą
`messages`, abyśmy mogli uzyskać indeks każdego elementu, który wysyłamy,
razem z samym elementem. Następnie stosujemy 100-milisekundowe opóźnienie do elementów o indeksie parzystym
i 300-milisekundowe opóźnienie do elementów o indeksie nieparzystym, aby symulować różne opóźnienia,
które możemy zobaczyć w strumieniu wiadomości w świecie rzeczywistym. Ponieważ nasz limit czasu wynosi
200 milisekund, powinno to mieć wpływ na połowę wiadomości.

<Listing number="17-35" caption="Sending messages through `tx` with an async delay without making `get_messages` an async function" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-35/src/main.rs:messages}}
```

</Listing>

Aby uśpić między wiadomościami w funkcji `get_messages` bez blokowania,
musimy użyć async. Jednak nie możemy uczynić samej funkcji `get_messages` funkcją async,
ponieważ zwrócilibyśmy wtedy `Future<Output = Stream<Item = String>>`
zamiast `Stream<Item = String>>`. Wywołujący musiałby czekać
na samą funkcję `get_messages`, aby uzyskać dostęp do strumienia. Ale pamiętaj: wszystko w
danej przyszłości dzieje się liniowo; współbieżność zachodzi *pomiędzy* przyszłościami. Oczekiwanie
`get_messages` wymagałoby wysłania wszystkich wiadomości, w tym uśpienia
pomiędzy wysłaniem każdej wiadomości, przed zwróceniem strumienia odbiorcy. W rezultacie
limit czasu byłby bezużyteczny. Nie byłoby żadnych opóźnień w samym strumieniu:
wszystkie opóźnienia wystąpiłyby zanim strumień byłby dostępny.

Zamiast tego pozostawiamy `get_messages` jako zwykłą funkcję, która zwraca strumień,
i tworzymy zadanie do obsługi asynchronicznych wywołań `sleep`.

> Uwaga: wywołanie `spawn_task` w ten sposób działa, ponieważ już skonfigurowaliśmy nasze
> środowisko wykonawcze. Wywołanie tej konkretnej implementacji `spawn_task` *bez*
> wcześniejszego skonfigurowania środowiska wykonawczego spowoduje panikę. Inne implementacje wybierają
> inne kompromisy: mogą utworzyć nowe środowisko wykonawcze i w ten sposób uniknąć paniki,
> ale skończyć z odrobiną dodatkowego narzutu lub po prostu nie zapewniają samodzielnego sposobu na
> tworzenie zadań bez odniesienia do środowiska wykonawczego. Powinieneś upewnić się, że wiesz,
> jaki kompromis wybrało Twoje środowisko wykonawcze i napisać kod odpowiednio!

Teraz nasz kod ma o wiele ciekawszy wynik! Pomiędzy każdą inną parą
komunikatów widzimy zgłoszony błąd: `Problem: Elapsed(())`.

<!-- manual-regeneration
cd listings/listing-17-35
cargo run
copy only the program output, *not* the compiler output
-->

```text
Message: 'a'
Problem: Elapsed(())
Message: 'b'
Message: 'c'
Problem: Elapsed(())
Message: 'd'
Message: 'e'
Problem: Elapsed(())
Message: 'f'
Message: 'g'
Problem: Elapsed(())
Message: 'h'
Message: 'i'
Problem: Elapsed(())
Message: 'j'
```

Limit czasu nie uniemożliwia dotarcia wiadomości na końcu — nadal otrzymujemy
wszystkie oryginalne wiadomości. Dzieje się tak, ponieważ nasz kanał jest nieograniczony: może
przechowywać tyle wiadomości, ile zmieści się w pamięci. Jeśli wiadomość nie dotrze
przed upływem limitu czasu, nasz program obsługi strumienia to uwzględni, ale gdy ponownie przeszuka
strumień, wiadomość może już dotrzeć.

W razie potrzeby możesz uzyskać inne zachowanie, używając innych rodzajów kanałów lub
innych rodzajów strumieni, bardziej ogólnie. Zobaczmy jeden z nich w praktyce w naszym
ostatnim przykładzie dla tej sekcji, łącząc strumień przedziałów czasu z
tym strumieniem wiadomości.

### Merging Strumienie

Najpierw utwórzmy kolejny strumień, który będzie emitował element co milisekundę, jeśli
pozwolimy mu działać bezpośrednio. Dla uproszczenia możemy użyć funkcji `sleep`, aby wysłać
komunikat z opóźnieniem i połączyć ją z tym samym podejściem tworzenia strumienia
z kanału, którego użyliśmy w `get_messages`. Różnica polega na tym, że tym razem
odeślemy liczbę interwałów, które upłynęły, więc typem zwracanym
będzie `impl Stream<Item = u32>`, a my możemy wywołać funkcję
`get_intervals`.

W listingu 17-36 zaczynamy od zdefiniowania `count` w zadaniu. (Możemy zdefiniować
je również poza zadaniem, ale jaśniej jest ograniczyć zakres dowolnej danej
zmiennej.) Następnie tworzymy nieskończoną pętlę. Każda iteracja pętli
asynchronicznie zasypia na jedną milisekundę, zwiększa liczbę, a następnie wysyła ją
przez kanał. Ponieważ wszystko to jest zawarte w zadaniu utworzonym przez
`spawn_task`, wszystko to zostanie wyczyszczone wraz ze środowiskiem wykonawczym, łącznie z
pętlą nieskończoną.

<Listing number="17-36" caption="Creating a stream with a counter that will be emitted once every millisecond" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-36/src/main.rs:intervals}}
```

</Listing>

Ten rodzaj nieskończonej pętli, która kończy się dopiero, gdy cały czas wykonywania zostanie
zerwany, jest dość powszechny w asynchronicznym Rust: wiele programów musi działać
bez końca. W przypadku asynchronii nie blokuje to niczego innego, o ile
w każdej iteracji pętli jest przynajmniej jeden punkt oczekiwania.

Wróciwszy do bloku asynchronicznego naszej głównej funkcji, zaczynamy od wywołania `get_intervals`.
Następnie łączymy strumienie `messages` i `intervals` metodą `merge`,
która łączy wiele strumieni w jeden strumień, który generuje elementy z dowolnego
strumienia źródłowego, gdy tylko elementy są dostępne, bez narzucania żadnej
szczególnej kolejności. Na koniec wykonujemy pętlę po tym połączonym strumieniu zamiast po
`messages` (Listing 17-37).

<Listing number="17-37" caption="Attempting to merge streams of messages and intervals" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-37/src/main.rs:main}}
```

</Listing>

W tym momencie ani `messages`, ani `intervals` nie muszą być przypięte ani zmienne,
ponieważ oba zostaną połączone w pojedynczy `scalony` strumień. Jednak to
wywołanie `merge` się nie kompiluje! (Podobnie jak wywołanie `next` w pętli `while`let`, ale wrócimy do tego po naprawieniu tego.) Te dwa strumienie
mają różne typy. Strumień `messages` ma typ `Timeout<impl
Stream<Item = String>>`, gdzie `Timeout` jest typem, który implementuje `Stream`
dla wywołania `timeout`. Tymczasem strumień `intervals` ma typ `impl
Stream<Item = u32>`. Aby połączyć te dwa strumienie, musimy przekształcić jeden z nich,
aby pasował do drugiego.

W Listingu 17-38 przerabiamy strumień `intervals`, ponieważ `messages`
jest już w podstawowym formacie, jakiego chcemy i musi obsługiwać błędy przekroczenia limitu czasu. Po pierwsze,
możemy użyć metody pomocniczej `map`, aby przekształcić `intervals` w ciąg.
Po drugie, musimy dopasować `Timeout` z `messages`. Ponieważ
tak naprawdę *nie chcemy* limitu czasu dla `intervals`, możemy po prostu utworzyć limit czasu,
który jest dłuższy niż inne używane przez nas okresy. Tutaj tworzymy
10-sekundowy limit czasu za pomocą `Duration::from_secs(10)`. Na koniec musimy uczynić
`stream` zmiennym, aby wywołania `next` pętli `while let` mogły iterować
się po strumieniu i przypiąć je, aby było to bezpieczne.

<!-- Nie możemy tego bezpośrednio przetestować, ponieważ nigdy się nie zatrzymuje. -->

<Listing number="17-38" caption="Aligning the types of the the `intervals` stream with the type of the `messages` stream" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-38/src/main.rs:main}}
```

</Listing>

To doprowadza nas *prawie* tam, gdzie musimy być. Wszystko sprawdza typy. Jeśli to uruchomisz, pojawią się dwa problemy. Po pierwsze, nigdy się nie zatrzyma! Będziesz
musiał to zatrzymać za pomocą <span class="keystroke">ctrl-c</span>. Po drugie,
komunikaty z alfabetu angielskiego zostaną pogrzebane pośród wszystkich
komunikatów licznika interwałów:

<!-- Nie wyodrębniam danych wyjściowych, ponieważ zmiany w tych danych wyjściowych nie są znaczące;
zmiany prawdopodobnie wynikają z tego, że zadania działają inaczej, a nie
ze zmian w kompilatorze -->

```text
--snip--
Interval: 38
Interval: 39
Interval: 40
Message: 'a'
Interval: 41
Interval: 42
Interval: 43
--snip--
```

Listing 17-39 pokazuje jeden ze sposobów rozwiązania tych dwóch ostatnich problemów. Najpierw używamy metody
`throttle` w strumieniu `intervals`, aby nie przytłoczyć strumienia
`messages`. Throttling to sposób ograniczania częstotliwości, z jaką funkcja
będzie wywoływana — lub, w tym przypadku, częstotliwości odpytywania strumienia. Raz na
sto milisekund powinno wystarczyć, ponieważ jest to w tym samym zakresie, co częstotliwość
przychodzenia naszych wiadomości.

Aby ograniczyć liczbę elementów, które zaakceptujemy ze strumienia, możemy użyć metody `take`. Stosujemy ją do *scalonego* strumienia, ponieważ chcemy ograniczyć ostateczny
wynik, a nie tylko jeden lub drugi strumień.

<Listing number="17-39" caption="Using `throttle` and `take` to manage the merged streams" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-39/src/main.rs:throttle}}
```

</Listing>

Teraz, gdy uruchamiamy program, zatrzymuje się on po pobraniu dwudziestu elementów ze
strumienia, a interwały nie przytłaczają wiadomości. Nie otrzymujemy również
`Interval: 100` lub `Interval: 200` itd., ale zamiast tego otrzymujemy `Interval: 1`,
`Interval: 2` itd. — mimo że mamy strumień źródłowy, który *może*
generować zdarzenie co milisekundę. Dzieje się tak, ponieważ wywołanie `throttle`
generuje nowy strumień, opakowując oryginalny strumień, tak że oryginalny
strumień jest sondowany tylko z szybkością przepustnicy, a nie z jego własną „natywną” szybkością. Nie mamy mnóstwa nieobsłużonych wiadomości interwałowych, które wybieramy
ignorując. Zamiast tego nigdy nie generujemy tych wiadomości interwałowych!
To jest wrodzona „leniwość” przyszłości Rusta w działaniu, pozwalająca nam
wybrać nasze cechy wydajnościowe.

<!-- manual-regeneration
cd listings/listing-17-39
cargo run
copy and paste only the program output
-->

```text
Interval: 1
Message: 'a'
Interval: 2
Interval: 3
Problem: Elapsed(())
Interval: 4
Message: 'b'
Interval: 5
Message: 'c'
Interval: 6
Interval: 7
Problem: Elapsed(())
Interval: 8
Message: 'd'
Interval: 9
Message: 'e'
Interval: 10
Interval: 11
Problem: Elapsed(())
Interval: 12
```

Jest jeszcze jedna rzecz, z którą musimy sobie poradzić: błędy! W przypadku obu tych
strumieni opartych na kanałach wywołania `send` mogą się nie powieść, gdy druga strona
kanału zostanie zamknięta — i to jest tylko kwestia tego, w jaki sposób środowisko wykonawcze wykonuje futures,
które tworzą strumień. Do tej pory ignorowaliśmy to, wywołując `unwrap`,
ale w dobrze zachowującej się aplikacji powinniśmy jawnie obsłużyć błąd, przynajmniej
kończąc pętlę, abyśmy nie próbowali wysyłać więcej wiadomości! Wypis 17-40 pokazuje
prostą strategię błędów: wydrukuj problem, a następnie `przerwij` pętle. Jak
zwykle, prawidłowy sposób obsługi błędu wysyłania wiadomości będzie się różnić — po prostu upewnij się,
że masz strategię.

<Listing number="17-40" caption="Handling errors and shutting down the loops">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-40/src/main.rs:errors}}
```

</Listing>

Teraz, gdy widzieliśmy już sporo asynchroniczności w praktyce, cofnijmy się o krok i
przyjrzyjmy się bliżej szczegółom tego, jak `Future`, `Stream` i inne kluczowe cechy,
które Rust wykorzystuje, aby asynchroniczność działała.

[17-02-messages]: ch17-02-concurrency-with-async.html#message-passing
