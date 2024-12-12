## Praca z dowolną liczbą przyszłości

Kiedy w poprzedniej sekcji zmieniliśmy używanie dwóch przyszłości na trzy,
musieliśmy również zmienić używanie `join` na używanie `join3`. Byłoby irytujące,
gdybyśmy musieli wywoływać inną funkcję za każdym razem, gdy zmienilibyśmy liczbę przyszłości, które
chcieliśmy połączyć. Na szczęście mamy makroformę `join`, do której możemy przekazać
dowolną liczbę argumentów. Obsługuje ona również oczekiwanie na same przyszłości.
W związku z tym moglibyśmy przepisać kod z Listingu 17-13, aby użyć `join!` zamiast
`join3`, jak w Listingu 17-14:

<Listing number="17-14" caption="Using `join!` to wait for multiple futures" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-14/src/main.rs:here}}
```

</Listing>

To zdecydowanie miła poprawa w porównaniu z koniecznością zamiany `join` na
`join3` i `join4` itd.! Jednak nawet ta forma makro działa tylko wtedy, gdy
znamy liczbę przyszłości z wyprzedzeniem. W prawdziwym Ruście jednak umieszczanie
przyszłości w kolekcji, a następnie czekanie na ukończenie niektórych lub wszystkich przyszłości w tej
kolekcji, jest powszechnym wzorcem.

Aby sprawdzić wszystkie przyszłości w pewnej kolekcji, będziemy musieli przejść przez nie i
dołączyć do *wszystkich* z nich. Funkcja `trpl::join_all` akceptuje każdy typ, który
implementuje cechę `Iterator`, o której dowiedzieliśmy się w rozdziale 13, więc
wydaje się być idealnym rozwiązaniem. Spróbujmy umieścić nasze przyszłości w wektorze i
zamieńmy `join!` na `join_all`.

<Listing  number="17-15" caption="Storing anonymous futures in a vector and calling `join_all`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-15/src/main.rs:here}}
```

</Listing>

Niestety, to się nie kompiluje. Zamiast tego otrzymujemy ten błąd:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-16/
cargo build
copy just the compiler error
-->


```text
error[E0308]: mismatched types
  --> src/main.rs:43:37
   |
8  |           let tx1_fut = async move {
   |  _______________________-
9  | |             let vals = vec![
10 | |                 String::from("hi"),
11 | |                 String::from("from"),
...  |
19 | |             }
20 | |         };
   | |_________- the expected `async` block
21 |
22 |           let rx_fut = async {
   |  ______________________-
23 | |             while let Some(value) = rx.recv().await {
24 | |                 println!("received '{value}'");
25 | |             }
26 | |         };
   | |_________- the found `async` block
...
43 |           let futures = vec![tx1_fut, rx_fut, tx_fut];
   |                                       ^^^^^^ expected `async` block, found a different `async` block
   |
   = note: expected `async` block `{async block@src/main.rs:8:23: 20:10}`
              found `async` block `{async block@src/main.rs:22:22: 26:10}`
   = note: no two async blocks, even if identical, have the same type
   = help: consider pinning your async block and and casting it to a trait object
```

To może być zaskakujące. W końcu żaden z nich niczego nie zwraca, więc każdy
blok generuje `Future<Output = ()>`. Jednak `Future` jest cechą, a nie
konkretnym typem. Konkretne typy to poszczególne struktury danych generowane
przez kompilator dla bloków asynchronicznych. Nie można umieścić dwóch różnych ręcznie napisanych
struktur w `Vec`, a to samo dotyczy różnych struktur
generowanych przez kompilator.

Aby to zadziałało, musimy użyć *obiektów cech*, tak jak zrobiliśmy to w [„Zwracanie
błędów z funkcji uruchamiania”][dyn] w rozdziale 12. (Obiekty cech omówimy szczegółowo w rozdziale 18.) Używanie obiektów cech pozwala nam traktować każdą z
anonimowych przyszłości wygenerowanych przez te typy jako ten sam typ, ponieważ wszystkie one
implementują cechę `Future`.

> Uwaga: W rozdziale 8 omówiliśmy inny sposób uwzględnienia wielu typów w
> `Vec`: użycie wyliczenia do reprezentowania każdego z różnych typów, które mogą
> pojawić się w wektorze. Nie możemy tego jednak zrobić tutaj. Po pierwsze, nie mamy
> możliwości nazwania różnych typów, ponieważ są anonimowe. Po drugie,
> powodem, dla którego sięgnęliśmy po wektor i `join_all`, była możliwość
> pracy z dynamiczną kolekcją przyszłości, w której nie wiemy,
> czym będą wszystkie do czasu wykonania.

Zaczynamy od umieszczenia każdej z przyszłości w `vec!` w `Box::new`, jak pokazano
w Liście 17-16.

<Listing number="17-16" caption="Trying to use `Box::new` to align the types of the futures in a `Vec`" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-16/src/main.rs:here}}
```

</Listing>

Niestety, to nadal się nie kompiluje. W rzeczywistości mamy ten sam podstawowy
błąd, co wcześniej, ale otrzymujemy go zarówno dla drugiego, jak i trzeciego wywołania `Box::new`, a także otrzymujemy nowe błędy odnoszące się do cechy `Unpin`. Wrócimy
do błędów `Unpin` za chwilę. Najpierw naprawmy błędy typu w wywołaniach
`Box::new`, jawnie adnotując typ zmiennej `futures`:

<Listing number="17-17" caption="Fixing the rest of the type mismatch errors by using an explicit type declaration" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-17/src/main.rs:here}}
```

</Listing>

Typ, który musieliśmy tutaj napisać, jest nieco skomplikowany, więc przeanalizujmy go:

* Najbardziej wewnętrznym typem jest sama przyszłość. Wyraźnie zauważamy, że wyjście
przyszłości to typ jednostki `()`, pisząc `Future<Output = ()>`.
* Następnie adnotujemy cechę za pomocą `dyn`, aby oznaczyć ją jako dynamiczną.
* Całe odniesienie do cechy jest zawarte w `Box`.
* Na koniec wyraźnie stwierdzamy, że `futures` to `Vec` zawierający te elementy.

To już zrobiło dużą różnicę. Teraz, gdy uruchamiamy kompilator, mamy tylko
błędy wspominające o `Unpin`. Chociaż jest ich trzy, zauważ, że
każdy jest bardzo podobny w swojej zawartości.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-17
cargo build
copy *only* the errors
-->

```text
error[E0277]: `{async block@src/main.rs:8:23: 20:10}` cannot be unpinned
   --> src/main.rs:46:24
    |
46  |         trpl::join_all(futures).await;
    |         -------------- ^^^^^^^ the trait `Unpin` is not implemented for `{async block@src/main.rs:8:23: 20:10}`, which is required by `Box<{async block@src/main.rs:8:23: 20:10}>: std::future::Future`
    |         |
    |         required by a bound introduced by this call
    |
    = note: consider using the `pin!` macro
            consider using `Box::pin` if you need to access the pinned value outside of the current scope
    = note: required for `Box<{async block@src/main.rs:8:23: 20:10}>` to implement `std::future::Future`
note: required by a bound in `join_all`
   --> /Users/chris/.cargo/registry/src/index.crates.io-6f17d22bba15001f/futures-util-0.3.30/src/future/join_all.rs:105:14
    |
102 | pub fn join_all<I>(iter: I) -> JoinAll<I::Item>
    |        -------- required by a bound in this function
...
105 |     I::Item: Future,
    |              ^^^^^^ required by this bound in `join_all`

error[E0277]: `{async block@src/main.rs:8:23: 20:10}` cannot be unpinned
  --> src/main.rs:46:9
   |
46 |         trpl::join_all(futures).await;
   |         ^^^^^^^^^^^^^^^^^^^^^^^ the trait `Unpin` is not implemented for `{async block@src/main.rs:8:23: 20:10}`, which is required by `Box<{async block@src/main.rs:8:23: 20:10}>: std::future::Future`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<{async block@src/main.rs:8:23: 20:10}>` to implement `std::future::Future`
note: required by a bound in `JoinAll`
  --> /Users/chris/.cargo/registry/src/index.crates.io-6f17d22bba15001f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`

error[E0277]: `{async block@src/main.rs:8:23: 20:10}` cannot be unpinned
  --> src/main.rs:46:33
   |
46 |         trpl::join_all(futures).await;
   |                                 ^^^^^ the trait `Unpin` is not implemented for `{async block@src/main.rs:8:23: 20:10}`, which is required by `Box<{async block@src/main.rs:8:23: 20:10}>: std::future::Future`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<{async block@src/main.rs:8:23: 20:10}>` to implement `std::future::Future`
note: required by a bound in `JoinAll`
  --> /Users/chris/.cargo/registry/src/index.crates.io-6f17d22bba15001f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`

Some errors have detailed explanations: E0277, E0308.
For more information about an error, try `rustc --explain E0277`.
```

To *dużo* do strawienia, więc rozłóżmy to na czynniki pierwsze. Pierwsza część wiadomości
mówi nam, że pierwszy blok asynchroniczny (`src/main.rs:8:23: 20:10`) nie
implementuje cechy `Unpin` i sugeruje użycie `pin!` lub `Box::pin`, aby to rozwiązać. Później w rozdziale zagłębimy się w kilka szczegółów na temat `Pin` i
`Unpin`. Na razie jednak możemy po prostu postępować zgodnie z radą kompilatora, aby
wyjść z impasu! W listingu 17-18 zaczynamy od zaktualizowania adnotacji typu dla
`futures`, przy czym `Pin` otacza każde `Box`. Po drugie, używamy `Box::pin`, aby przypiąć
same futures.

<Listing number="17-18" caption="Using `Pin` and `Box::pin` to make the `Vec` type check" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-18/src/main.rs:here}}
```

</Listing>

Jeśli skompilujemy i uruchomimy to, w końcu otrzymamy wynik, na jaki liczyliśmy:

<!-- Nie wyodrębniam wyniku, ponieważ zmiany w tym wyniku nie są znaczące;
zmiany prawdopodobnie wynikają z faktu, że wątki działają inaczej, a nie ze
zmian w kompilatorze -->

```text
received 'hi'
received 'more'
received 'from'
received 'messages'
received 'the'
received 'for'
received 'future'
received 'you'
```

Uff!

Możemy tu jeszcze trochę poeksperymentować. Po pierwsze, użycie `Pin<Box<T>>`
wiąże się z niewielkim dodatkowym narzutem wynikającym z umieszczania tych przyszłości na
stosie `Box` — a robimy to tylko po to, aby typy się ze sobą zgadzały. W końcu nie
potrzebujemy *przydziału* stosu: te przyszłości są lokalne dla
tej konkretnej funkcji. Jak wspomniano powyżej, `Pin` jest typem opakowaniowym, więc możemy
skorzystać z posiadania jednego typu w `Vec` — pierwotny powód, dla którego
sięgnęliśmy po `Box` — bez wykonywania przydziału stosu. Możemy używać `Pin` bezpośrednio
z każdą przyszłością, używając makra `std::pin::pin`.

Musimy jednak nadal wyraźnie określić typ przypiętego odniesienia;
w przeciwnym razie Rust nadal nie będzie wiedział, aby interpretować je jako dynamiczne obiekty cech,
a właśnie tego potrzebujemy w `Vec`. Dlatego `przypinamy!` każdą przyszłość,
kiedy ją definiujemy, i definiujemy `futures` jako `Vec` zawierający przypięte, zmienne
odniesienia do dynamicznego typu `Future`, jak w Liście 17-19.

<Listing number="17-19" caption="Using `Pin` directly with the `pin!` macro to avoid unnecessary heap allocations" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-19/src/main.rs:here}}
```

</Listing>

Dotarliśmy tak daleko, ignorując fakt, że możemy mieć różne typy `Output`. Na przykład w Listingu 17-20 anonimowa przyszłość dla `a` implementuje
`Future<Output = u32>`, anonimowa przyszłość dla `b` implementuje `Future<Output =
&str>`, a anonimowa przyszłość dla `c` implementuje `Future<Output = bool>`.

<Listing number="17-20" caption="Three futures with distinct types" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-20/src/main.rs:here}}
```

</Listing>

Możemy użyć `trpl::join!`, aby na nie czekać, ponieważ pozwala to na przekazanie
wielu typów future i tworzy krotkę tych typów. *Nie możemy* użyć
`trpl::join_all`, ponieważ wymaga, aby przekazane futures all miały ten sam
typ. Pamiętaj, że ten błąd był tym, co sprawiło, że zaczęliśmy przygodę z `Pin`!

To fundamentalny kompromis: możemy albo obsługiwać dynamiczną liczbę
futures za pomocą `join_all`, o ile wszystkie mają ten sam typ, albo możemy obsługiwać
ustawioną liczbę futures za pomocą funkcji `join` lub makra `join!`,
nawet jeśli mają różne typy. Jest to jednak takie samo, jak praca z dowolnymi innymi
typami w Rust. Futures nie są niczym szczególnym, mimo że mamy pewną
ładną składnię do pracy z nimi, a to jest dobra rzecz.

### Przyszłość wyścigów

Kiedy „łączymy” futures z rodziną funkcji i makr `join`,
wymagamy, aby *wszystkie* z nich się zakończyły, zanim przejdziemy dalej. Czasami jednak
potrzebujemy tylko *pewnej* future z zestawu, aby zakończyć, zanim przejdziemy dalej — trochę podobnie do
wyścigu jednej future z drugą.

W Liście 17-21 ponownie używamy `trpl::race`, aby uruchomić dwie futures, `slow` i
`fast`, przeciwko sobie. Każda z nich drukuje komunikat, gdy zaczyna działać,
zatrzymuje się na pewien czas, wywołując i oczekując na `sleep`, a następnie drukuje
kolejny komunikat, gdy się zakończy. Następnie przekazujemy oba do `trpl::race` i czekamy, aż
jedna z nich się zakończy. (Wynik tutaj nie będzie zbyt zaskakujący: `szybki` wygrywa!)
W przeciwieństwie do sytuacji, gdy użyliśmy `race` w [Naszym pierwszym programie asynchronicznym][programie asynchronicznym], my
po prostu ignorujemy instancję `Either`, którą tutaj zwraca, ponieważ wszystkie
interesujące zachowania mają miejsce w treści bloków asynchronicznych.

<Listing number="17-21" caption="Using `race` to get the result of whichever future finishes first" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-21/src/main.rs:here}}
```

</Listing>

Zauważ, że jeśli odwrócisz kolejność argumentów na `race`, kolejność
„uruchomionych” wiadomości się zmieni, nawet jeśli `szybka` przyszłość zawsze kończy się
jako pierwsza. Dzieje się tak, ponieważ implementacja tej konkretnej funkcji `race` nie jest
sprawiedliwa. Zawsze uruchamia ona przyszłości przekazane jako argumenty w kolejności, w jakiej są
przekazywane. Inne implementacje *są* sprawiedliwe i będą losowo wybierać przyszłość,
która zostanie zbadana jako pierwsza. Niezależnie od tego, czy implementacja race, której używamy, jest
sprawiedliwa, *jedna* z przyszłości dotrze do pierwszej `await` w jej treści,
zanim inne zadanie będzie mogło zostać uruchomione.

Przypomnij sobie z [Nasz pierwszy program asynchroniczny][async-program], że w każdym punkcie oczekiwania,
Rust daje środowisku wykonawczemu szansę na wstrzymanie zadania i przełączenie się na inne, jeśli
oczekiwana przyszłość nie jest gotowa. Odwrotna sytuacja również jest prawdziwa: Rust *tylko* wstrzymuje
bloki asynchroniczne i przekazuje kontrolę z powrotem środowisku wykonawczemu w punkcie oczekiwania. Wszystko
pomiędzy punktami oczekiwania jest synchroniczne.

Oznacza to, że jeśli wykonujesz wiele prac w bloku asynchronicznym bez punktu oczekiwania,
ta przyszłość zablokuje postęp innych przyszłości. Czasami możesz usłyszeć, że jedna przyszłość *głoduje* innych przyszłości. W niektórych przypadkach
może to nie być duży problem. Jednak jeśli wykonujesz jakiś kosztowny
projekt lub długotrwałą pracę, lub jeśli masz przyszłość, która będzie wykonywać jakieś
konkretne zadanie w nieskończoność, musisz pomyśleć o tym, kiedy i gdzie
oddać kontrolę z powrotem do środowiska wykonawczego.

Z tego samego powodu, jeśli masz długotrwałe operacje blokujące, asynchroniczność może być
użytecznym narzędziem do zapewniania sposobów, aby różne części programu mogły się ze sobą łączyć.

Ale *jak* oddałbyś kontrolę z powrotem do środowiska wykonawczego w takich przypadkach?

### Wydajność

Symulujmy długotrwałą operację. Listing 17-22 wprowadza funkcję `slow`. Używa ona `std::thread::sleep` zamiast `trpl::sleep`, tak aby wywołanie
`slow` zablokowywało bieżący wątek na pewną liczbę milisekund. Możemy użyć
`slow`, aby zastąpić rzeczywiste operacje, które są zarówno długotrwałe, jak i
blokujące.

<Listing number="17-22" caption="Using `thread::sleep` to simulate slow operations" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-22/src/main.rs:slow}}
```

</Listing>

W Listingu 17-23 używamy `slow`, aby emulować wykonywanie tego rodzaju pracy związanej z procesorem w
parach futures. Na początek, każda futures przekazuje kontrolę z powrotem do środowiska wykonawczego
*po* wykonaniu szeregu powolnych operacji.

<Listing number="17-23" caption="Using `thread::sleep` to simulate slow operations" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-23/src/main.rs:slow-futures}}
```

</Listing>
Jeśli uruchomisz to, zobaczysz następujący wynik:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-24/
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'a' ran for 10ms
'a' ran for 20ms
'b' started.
'b' ran for 75ms
'b' ran for 10ms
'b' ran for 15ms
'b' ran for 350ms
'a' finished.
```

Podobnie jak w naszym poprzednim przykładzie, `race` nadal kończy się, gdy tylko `a` zostanie wykonane.
Nie ma jednak przeplotu między dwoma przyszłościami. Przyszłość `a` wykonuje całą
swoją pracę, dopóki nie zostanie oczekiwane wywołanie `trpl::sleep`, następnie przyszłość `b` wykonuje całą
swoją pracę, dopóki nie zostanie oczekiwane wywołanie `trpl::sleep`, a następnie przyszłość `a`
się kończy. Aby umożliwić obu przyszłościom postęp między ich wolnymi
zadaniami, potrzebujemy punktów oczekiwania, abyśmy mogli przekazać kontrolę z powrotem do środowiska wykonawczego. To
oznacza, że ​​potrzebujemy czegoś, na co możemy czekać!

Możemy już zobaczyć tego rodzaju przekazanie w Liście 17-23: gdybyśmy
usunęli `trpl::sleep` na końcu przyszłości `a`, zakończyłoby się ono
bez uruchomienia przyszłości `b` *w ogóle*. Może moglibyśmy użyć funkcji `sleep` jako punktu wyjścia?

<Listing number="17-24" caption="Using `sleep` to let operations switch off making progress" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-24/src/main.rs:here}}
```

</Listing>

W Listingu 17-24 dodajemy wywołania `trpl::sleep` z punktami oczekiwania pomiędzy każdym wywołaniem
do `slow`. Teraz praca dwóch futures jest przeplatana:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-24
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'b' started.
'b' ran for 75ms
'a' ran for 10ms
'b' ran for 10ms
'a' ran for 20ms
'b' ran for 15ms
'a' finished.
```

Future `a` nadal działa przez chwilę przed przekazaniem kontroli `b`, ponieważ
wywołuje `slow` zanim wywoła `trpl::sleep`, ale potem futures
zamieniają się miejscami za każdym razem, gdy jeden z nich osiągnie punkt oczekiwania. W tym przypadku
zrobiliśmy to po każdym wywołaniu `slow`, ale moglibyśmy rozdzielić pracę,
jakkolwiek to dla nas najbardziej sensowne.

Tak naprawdę nie chcemy tutaj *spać*: chcemy robić postępy tak szybko, jak to możliwe. Musimy po prostu przekazać kontrolę z powrotem środowisku wykonawczemu. Możemy to zrobić
bezpośrednio, używając funkcji `yield_now`. W Liście 17-25 zastępujemy wszystkie te
wywołania `sleep` funkcją `yield_now`.

<Listing number="17-25" caption="Using `yield_now` to let operations switch off making progress" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-25/src/main.rs:yields}}
```

</Listing>

To jest zarówno bardziej zrozumiałe w odniesieniu do faktycznego celu, jak i może być znacznie szybsze
niż użycie `sleep`, ponieważ timery takie jak ten używany przez `sleep` często mają
ograniczenia co do tego, jak bardzo mogą być szczegółowe. Wersja `sleep`, której używamy,
na przykład, zawsze będzie spała przez co najmniej milisekundę, nawet jeśli przekażemy jej
`Duration` jedną nanosekundę. Ponownie, współczesne komputery są *szybkie*: mogą zrobić
wiele w ciągu jednej milisekundy!

Możesz to zobaczyć sam, ustawiając mały test porównawczy, taki jak ten
w Liście 17-26. (To nie jest szczególnie rygorystyczny sposób przeprowadzania testów wydajności, ale wystarczy, aby pokazać różnicę tutaj). Tutaj pomijamy całe
wyświetlanie statusu, przekazujemy jednonanosekundowy `Duration` do `trpl::sleep` i pozwalamy,
aby każda przyszłość działała samodzielnie, bez przełączania się między przyszłościami. Następnie wykonujemy
1000 iteracji i sprawdzamy, ile czasu zajmie wykonanie zadania w przyszłości przy użyciu `trpl::sleep`
w porównaniu z wykonaniem zadania w przyszłości przy użyciu `trpl::yield_now`.

<Listing number="17-26" caption="Comparing the performance of `sleep` and `yield_now`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-26/src/main.rs:here}}
```

</Listing>

Wersja z `yield_now` jest *dużo* szybsza!

Oznacza to, że async może być użyteczny nawet w przypadku zadań związanych z obliczeniami, w zależności od tego,
co jeszcze robi Twój program, ponieważ zapewnia przydatne narzędzie do
strukturyzowania relacji między różnymi częściami programu. Jest to
forma *współpracującego wielozadaniowości*, w której każda przyszłość ma moc określania,
kiedy przekazuje kontrolę za pośrednictwem punktów oczekiwania. Każda przyszłość ma zatem również
odpowiedzialność za unikanie blokowania przez zbyt długi czas. W niektórych wbudowanych systemach operacyjnych opartych na Rust jest to *jedyny* rodzaj wielozadaniowości!

W rzeczywistym kodzie zazwyczaj nie będziesz naprzemiennie wywoływać funkcji z punktami oczekiwania w każdym wierszu, oczywiście. Chociaż oddawanie kontroli w ten sposób jest
stosunkowo niedrogie, nie jest darmowe! W wielu przypadkach próba podziału
zadania związanego z obliczeniami może znacznie je spowolnić, więc czasami lepiej jest
dla *ogólnej* wydajności pozwolić operacji na krótkie zablokowanie. Zawsze powinieneś
mierzyć, aby zobaczyć, jakie są faktyczne wąskie gardła wydajnościowe twojego kodu.
Podstawowa dynamika jest ważna, o której należy pamiętać, jeśli *widzisz*, że wiele pracy jest wykonywanej szeregowo, niż się spodziewałeś, że będzie wykonywana jednocześnie!

### Budowanie własnych abstrakcji asynchronicznych

Możemy również komponować razem futures, aby tworzyć nowe wzorce. Na przykład możemy
zbudować funkcję `timeout` z asynchronicznych bloków konstrukcyjnych, które już mamy. Kiedy
skończymy, wynikiem będzie kolejny blok konstrukcyjny, którego moglibyśmy użyć do zbudowania
jeszcze dalszych asynchronicznych abstrakcji.

Listing 17-27 pokazuje, jak można by oczekiwać, że ten `timeout` będzie działał z powolną
future.

<Listing number="17-27" caption="Using our imagined `timeout` to run a slow operation with a time limit" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-27/src/main.rs:here}}
```

</Listing>

Zaimplementujmy to! Na początek pomyślmy o API dla `timeout`:

* Musi to być sama funkcja asynchroniczna, abyśmy mogli na nią czekać.
* Jej pierwszym parametrem powinien być future do uruchomienia. Możemy uczynić ją generyczną, aby umożliwić
jej działanie z dowolną future.
* Jej drugim parametrem będzie maksymalny czas oczekiwania. Jeśli użyjemy `Duration`,
ułatwi to przekazanie do `trpl::sleep`.
* Powinna zwrócić `Result`. Jeśli future zakończy się pomyślnie,
`Result` będzie `Ok` z wartością wygenerowaną przez future. Jeśli timeout
upłynie jako pierwszy, `Result` będzie `Err` z czasem, na który timeout
czekał.

Listing 17-28 pokazuje tę deklarację.

<!-- Nie jest to testowane, ponieważ celowo się nie kompiluje. -->

<Listing number="17-28" caption="Defining the signature of `timeout`" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-28/src/main.rs:declaration}}
```

</Listing>

To spełnia nasze cele dla typów. Teraz pomyślmy o *zachowaniu*, którego
potrzebujemy: chcemy ścigać się z przekazaną przyszłością z czasem trwania. Możemy użyć
`trpl::sleep`, aby utworzyć przyszłość timera z czasu trwania i użyć `trpl::race`, aby
uruchomić ten timer z przyszłością przekazaną przez wywołującego.

Wiemy również, że `race` nie jest sprawiedliwe i sprawdza argumenty w kolejności, w jakiej są
przekazywane. Dlatego najpierw przekazujemy `future_to_try` do `race`, aby miał szansę
się ukończyć, nawet jeśli `max_time` jest bardzo krótkim czasem trwania. Jeśli `future_to_try`
zakończy się jako pierwsze, `race` zwróci `Left` z wyjściem z `future`. Jeśli
`timer` zakończy się jako pierwsze, `race` zwróci `Right` z wyjściem timera
`()`.

W Listingu 17-29 dopasowujemy wynik oczekiwania na `trpl::race`. Jeśli
`future_to_try` zakończyło się powodzeniem i otrzymamy `Left(output)`, zwracamy `Ok(output)`.
Jeśli zamiast tego upłynął czas uśpienia i otrzymamy `Right(())`, ignorujemy `()`
za pomocą `_` i zwracamy `Err(max_time)`.

<Listing number="17-29" caption="Defining `timeout` with `race` and `sleep`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-29/src/main.rs:implementation}}
```

</Listing>

Dzięki temu mamy działający `timeout`, zbudowany z dwóch innych async helperów. Jeśli
uruchomimy nasz kod, wydrukuje on tryb awarii po timeout:

```text
Failed after 2 seconds
```

Ponieważ futures komponują się z innymi futures, możesz budować naprawdę potężne narzędzia,
używając mniejszych asynchronicznych bloków konstrukcyjnych. Na przykład możesz użyć tego samego
podejścia, aby połączyć timeouty z ponownymi próbami, a następnie użyć ich z rzeczami,
takimi jak wywołania sieciowe — jeden z przykładów z początku rozdziału!

W praktyce zazwyczaj będziesz pracować bezpośrednio z `async` i `await`, a
pośrednio z funkcjami i makrami, takimi jak `join`, `join_all`, `race` itd. Będziesz musiał tylko od czasu do czasu sięgnąć po `pin`, aby użyć ich z tymi
interfejsami API.

Widzieliśmy już wiele sposobów pracy z wieloma futures w tym samym
czasie. Następnie przyjrzymy się, jak możemy pracować z wieloma przyszłościami w
sekwencji w czasie, z *strumieniami*. Oto kilka rzeczy, które możesz najpierw rozważyć:

* Użyliśmy `Vec` z `join_all`, aby poczekać, aż wszystkie przyszłości w pewnej grupie
się zakończą. Jak można użyć `Vec` do przetworzenia grupy przyszłości w
sekwencji? Jakie są kompromisy takiego postępowania?

* Przyjrzyj się typowi `futures::stream::FuturesUnorder` ze skrzyni `futures`. Czym jego użycie różniłoby się od użycia `Vec`? (Nie przejmuj się
tym, że pochodzi z części `stream` skrzyni; działa on dobrze
z dowolną kolekcją przyszłości.)


[collections]: ch08-01-vectors.html#using-an-enum-to-store-multiple-types
[dyn]: ch12-03-improving-error-handling-and-modularity.html
[async-program]: ch17-01-futures-and-syntax.html#our-first-async-program
