## Zmiana naszego jednowątkowego serwera na wielowątkowy serwer

W tej chwili serwer będzie przetwarzał każde żądanie po kolei, co oznacza, że ​​nie będzie
przetwarzał drugiego połączenia, dopóki pierwsze nie zostanie ukończone. Gdyby
serwer otrzymywał coraz więcej żądań, to szeregowe wykonywanie byłoby coraz
mniej optymalne. Jeśli serwer otrzyma żądanie, którego przetwarzanie zajmuje dużo czasu, kolejne żądania będą musiały czekać, aż długie żądanie zostanie
zakończone, nawet jeśli nowe żądania można przetworzyć szybko. Będziemy musieli to naprawić, ale najpierw przyjrzymy się problemowi w działaniu.

### Simulating a Slow Request in the Current Server Implementation

Przyjrzymy się, jak wolno przetwarzane żądanie może wpłynąć na inne żądania kierowane do
naszej obecnej implementacji serwera. W listingu 21-10 zaimplementowano obsługę żądania
do */sleep* z symulowaną wolną odpowiedzią, która spowoduje, że serwer przejdzie w stan uśpienia
na 5 sekund przed udzieleniem odpowiedzi.

<Listing number="21-10" file-name="src/main.rs" caption="Simulating a slow request by sleeping for 5 seconds">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-10/src/main.rs:here}}
```

</Listing>

Zmieniliśmy `if` na `match` teraz, gdy mamy trzy przypadki. Musimy
wyraźnie dopasować fragment `request_line` do wzorca dopasowania do
wartości literału ciągu; `match` nie wykonuje automatycznego odwoływania się i
dereferencji, jak robi to metoda równości.

Pierwsze ramię jest takie samo jak blok `if` z Listingu 21-9. Drugie ramię
dopasowuje żądanie do */sleep*. Po otrzymaniu tego żądania serwer
uśpi się na 5 sekund przed wyrenderowaniem pomyślnej strony HTML. Trzecie ramię jest takie samo jak blok `else` z Listingu 21-9.

Możesz zobaczyć, jak prymitywny jest nasz serwer: prawdziwe biblioteki obsługiwałyby
rozpoznawanie wielu żądań w znacznie mniej rozwlekły sposób!

Uruchom serwer za pomocą `cargo run`. Następnie otwórz dwa okna przeglądarki: jedno dla
*http://127.0.0.1:7878/* i drugie dla *http://127.0.0.1:7878/sleep*. Jeśli
wpiszesz adres URI */* kilka razy, jak poprzednio, zobaczysz, że reaguje szybko.
Ale jeśli wpiszesz */sleep*, a następnie załadujesz */*, zobaczysz, że */* czeka, aż
`sleep` prześpi się przez pełne 5 sekund przed załadowaniem.

Istnieje wiele technik, których moglibyśmy użyć, aby uniknąć cofania się żądań za
wolnym żądaniem; tą, którą zaimplementujemy, jest pula wątków.

### Improving Throughput with a Thread Pool

*Pula wątków* to grupa utworzonych wątków, które czekają i są gotowe do
obsługi zadania. Gdy program otrzymuje nowe zadanie, przypisuje jeden z
wątków w puli do zadania, a ten wątek przetworzy zadanie.
Pozostałe wątki w puli są dostępne do obsługi wszelkich innych zadań, które pojawią się,
podczas gdy pierwszy wątek przetwarza. Gdy pierwszy wątek zakończy
przetwarzanie swojego zadania, wraca do puli bezczynnych wątków, gotowy do obsługi
nowego zadania. Pula wątków umożliwia jednoczesne przetwarzanie połączeń,
zwiększając przepustowość serwera.

Ograniczymy liczbę wątków w puli do niewielkiej liczby, aby chronić nas
przed atakami typu Denial of Service (DoS); gdybyśmy kazali naszemu programowi tworzyć nowy wątek
dla każdego żądania, gdy ono nadejdzie, ktoś wysyłający 10 milionów żądań do naszego
serwera mógłby wywołać spustoszenie, wykorzystując wszystkie zasoby naszego serwera i zatrzymując
przetwarzanie żądań.

Zamiast generować nieograniczoną liczbę wątków, będziemy mieć stałą liczbę
wątków oczekujących w puli. Żądania, które napływają, są wysyłane do puli w celu
przetworzenia. Pula będzie utrzymywać kolejkę przychodzących żądań. Każdy z
wątków w puli wyrzuci żądanie z tej kolejki, obsłuży żądanie,
a następnie poprosi kolejkę o kolejne żądanie. Dzięki temu projektowi możemy przetwarzać do
„N” żądań jednocześnie, gdzie „N” to liczba wątków. Jeśli każdy
wątek odpowiada na długotrwałe żądanie, kolejne żądania mogą nadal
tworzyć kopię zapasową w kolejce, ale zwiększyliśmy liczbę długotrwałych żądań,
które możemy obsłużyć przed osiągnięciem tego punktu.

Ta technika jest tylko jednym z wielu sposobów na poprawę przepustowości
serwera WWW. Inne opcje, które możesz rozważyć, to *model fork/join*,
*model jednowątkowego asynchronicznego wejścia/wyjścia* lub *model wielowątkowego asynchronicznego wejścia/wyjścia*. Jeśli
jesteś zainteresowany tym tematem, możesz przeczytać więcej o innych rozwiązaniach i
spróbować je wdrożyć; w przypadku języka niskiego poziomu, takiego jak Rust, wszystkie te
opcje są możliwe.

Zanim zaczniemy wdrażać pulę wątków, omówmy, jak powinno wyglądać jej używanie. Kiedy próbujesz zaprojektować kod, napisanie interfejsu klienta
najpierw może pomóc w ukierunkowaniu projektu. Napisz API kodu tak, aby było
ustrukturyzowane w sposób, w jaki chcesz je wywołać; następnie zaimplementuj funkcjonalność
w ramach tej struktury, zamiast implementować funkcjonalność, a następnie
projektować publiczne API.

Podobnie jak w przypadku korzystania z programowania sterowanego testami w projekcie w rozdziale 12,
tutaj użyjemy programowania sterowanego kompilatorem. Napiszemy kod, który wywołuje
potrzebne nam funkcje, a następnie przyjrzymy się błędom kompilatora, aby określić,
co powinniśmy zmienić, aby kod działał. Zanim to jednak zrobimy,
przeanalizujemy technikę, której nie będziemy używać jako punktu wyjścia.

<!-- Old headings. Do not remove or links may break. -->
<a id="code-structure-if-we-could-spawn-a-thread-for-each-request"></a>

#### Spawning a Thread for Each Request

Najpierw sprawdźmy, jak mógłby wyglądać nasz kod, gdyby tworzył nowy wątek dla
każdego połączenia. Jak wspomniano wcześniej, nie jest to nasz ostateczny plan ze względu na
problemy z potencjalnym tworzeniem nieograniczonej liczby wątków, ale jest to
punkt wyjścia, aby najpierw uzyskać działający serwer wielowątkowy. Następnie dodamy
pulę wątków jako ulepszenie, a porównanie dwóch rozwiązań będzie
łatwiejsze. Listing 21-11 pokazuje zmiany, które należy wprowadzić w `main`, aby utworzyć nowy wątek,
obsługujący każdy strumień w pętli `for`.

<Listing number="21-11" file-name="src/main.rs" caption="Spawning a new thread for each stream">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-11/src/main.rs:here}}
```

</Listing>

Jak dowiedziałeś się w rozdziale 16, `thread::spawn` utworzy nowy wątek, a następnie
uruchomi kod w zamknięciu w nowym wątku. Jeśli uruchomisz ten kod i załadujesz
*/sleep* w przeglądarce, a następnie */* w dwóch kolejnych kartach przeglądarki, rzeczywiście zobaczysz,
że żądania do */* nie muszą czekać na zakończenie */sleep*. Jednak, jak
wspomnieliśmy, ostatecznie przytłoczy to system, ponieważ będziesz tworzyć
nowe wątki bez żadnych ograniczeń.

<!-- Old headings. Do not remove or links may break. -->
<a id="creating-a-similar-interface-for-a-finite-number-of-threads"></a>

#### Creating a Finite Number of Threads

Chcemy, aby nasza pula wątków działała w podobny, znany sposób, więc przejście z
wątków na pulę wątków nie wymagało dużych zmian w kodzie, który używa
naszego API. Listing 21-12 pokazuje hipotetyczny interfejs dla struktury `ThreadPool`, której chcemy użyć zamiast `thread::spawn`.

<Listing number="21-12" file-name="src/main.rs" caption="Our ideal `ThreadPool` interface">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-12/src/main.rs:here}}
```

</Listing>

Używamy `ThreadPool::new`, aby utworzyć nową pulę wątków z konfigurowalną liczbą
wątków, w tym przypadku czterema. Następnie, w pętli `for`, `pool.execute` ma
podobny interfejs jak `thread::spawn`, ponieważ przyjmuje zamknięcie, które pula powinna
uruchomić dla każdego strumienia. Musimy zaimplementować `pool.execute`, aby przyjmowało zamknięcie i przekazywało je wątkowi w puli do uruchomienia. Ten kod jeszcze się nie
skompiluje, ale spróbujemy, aby kompilator mógł nas pokierować, jak to naprawić.

<!-- Stare nagłówki. Nie usuwaj, ponieważ linki mogą się zepsuć. -->
<a id="building-the-threadpool-struct-using-compiler-driven-development"></a>

#### Building `ThreadPool` Using Compiler Driven Development

Wprowadź zmiany w Listingu 21-12 do *src/main.rs*, a następnie użyjmy
błędów kompilatora z `cargo check` do sterowania naszym rozwojem. Oto pierwszy
błąd, który otrzymujemy:

```console
{{#include ../listings/ch21-web-server/listing-21-12/output.txt}}
```

Świetnie! Ten błąd mówi nam, że potrzebujemy typu lub modułu `ThreadPool`, więc
zbudujemy go teraz. Nasza implementacja `ThreadPool` będzie niezależna od rodzaju
pracy, którą wykonuje nasz serwer WWW. Zmieńmy więc skrzynię `hello` ze skrzyni
binarnej na skrzynię biblioteczną, aby przechowywać naszą implementację `ThreadPool`. Po
zmianie na skrzynię biblioteczną możemy również użyć osobnej biblioteki puli wątków
do dowolnej pracy, którą chcemy wykonać przy użyciu puli wątków, a nie tylko do obsługi
żądań internetowych.

Utwórz plik *src/lib.rs* zawierający następujące elementy, które są najprostszą
definicją struktury `ThreadPool`, jaką możemy mieć na razie:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/lib.rs}}
```

</Listing>

Then edit *main.rs* file to bring `ThreadPool` into scope from the library
crate by adding the following code to the top of *src/main.rs*:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/main.rs:here}}
```

</Listing>

Ten kod nadal nie będzie działał, ale sprawdźmy go ponownie, aby uzyskać następny błąd, którym musimy się zająć:

```console
{{#include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/output.txt}}
```

Ten błąd wskazuje, że musimy utworzyć powiązaną funkcję o nazwie
`new` dla `ThreadPool`. Wiemy również, że `new` musi mieć jeden parametr,
który może zaakceptować `4` jako argument i powinien zwrócić instancję `ThreadPool`.
Zaimplementujmy najprostszą funkcję `new`, która będzie miała te
cechy:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/src/lib.rs}}
```

</Listing>

Wybraliśmy `usize` jako typ parametru `size`, ponieważ wiemy, że
ujemna liczba wątków nie ma sensu. Wiemy również, że użyjemy tego
4 jako liczby elementów w kolekcji wątków, do czego służy typ
`usize`, jak omówiono w sekcji [„Typy całkowite”][integer-types]<!--
ignore --> rozdziału 3.

Sprawdźmy kod jeszcze raz:

```console
{{#include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/output.txt}}
```

Teraz błąd występuje, ponieważ nie mamy metody `execute` w `ThreadPool`.
Przypomnij sobie z sekcji [„Tworzenie skończonej liczby wątków”](#creating-a-finite-number-of-threads)<!-- ignore -->, że
zdecydowaliśmy, że nasza pula wątków powinna mieć interfejs podobny do `thread::spawn`.
Ponadto zaimplementujemy funkcję `execute`, aby przyjmowała ona otrzymane
zamknięcie i przekazywała je bezczynnemu wątkowi w puli do uruchomienia.

Zdefiniujemy metodę `execute` w `ThreadPool`, aby przyjmowała zamknięcie jako
parametr. Przypomnij sobie z sekcji [„Przenoszenie przechwyconych wartości poza zamknięcie i cechy
`Fn`”][fn-traits]<!-- ignore --> w rozdziale 13, że możemy przyjąć
zamknięcia jako parametry z trzema różnymi cechami: `Fn`, `FnMut` i
`FnOnce`. Musimy zdecydować, jakiego rodzaju zamknięcia tutaj użyć. Wiemy, że skończymy na zrobieniu czegoś podobnego do implementacji biblioteki standardowej `thread::spawn`, więc możemy sprawdzić, co ogranicza sygnaturę `thread::spawn`
w swoim parametrze. Dokumentacja pokazuje nam następujące:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

Parametr typu `F` jest tym, czym się tutaj interesujemy; parametr typu `T` jest powiązany z wartością zwracaną, a my się tym nie zajmujemy.
Widzimy, że `spawn` używa `FnOnce` jako cechy ograniczonej do `F`. Prawdopodobnie
to jest to, czego chcemy, ponieważ ostatecznie przekażemy argument, który otrzymamy w
`execute` do `spawn`. Możemy być pewni, że `FnOnce` jest cechą, której
chcemy użyć, ponieważ wątek do uruchomienia żądania wykona zamknięcie tego żądania tylko raz, co odpowiada `Once` w `FnOnce`.

Parametr typu `F` ma również cechę ograniczoną `Send` i czas życia ograniczony
``static`, które są przydatne w naszej sytuacji: potrzebujemy `Send`, aby przenieść zamknięcie
z jednego wątku do drugiego i `'static`, ponieważ nie wiemy, ile czasu zajmie wykonanie wątku. Utwórzmy metodę `execute` w
`ThreadPool`, która przyjmie ogólny parametr typu `F` z następującymi ograniczeniami:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-03-define-execute/src/lib.rs:here}}
```

</Listing>

Nadal używamy `()` po `FnOnce`, ponieważ `FnOnce` reprezentuje zamknięcie,
które nie przyjmuje żadnych parametrów i zwraca typ jednostki `()`. Podobnie jak w przypadku
definicji funkcji, typ zwracany może zostać pominięty w sygnaturze, ale nawet jeśli nie mamy żadnych parametrów, nadal potrzebujemy nawiasów.

Znów, jest to najprostsza implementacja metody `execute`: nie robi ona
nic, ale próbujemy tylko skompilować nasz kod. Sprawdźmy to jeszcze raz:

```console
{{#include ../listings/ch21-web-server/no-listing-03-define-execute/output.txt}}
```

Kompiluje się! Ale zauważ, że jeśli spróbujesz `cargo run` i złożysz żądanie w
przeglądarce, zobaczysz błędy w przeglądarce, które widzieliśmy na początku
rozdziału. Nasza biblioteka nie wywołuje jeszcze zamknięcia przekazanego do `execute`!

> Uwaga: Powiedzenie, które możesz usłyszeć o językach ze ścisłymi kompilatorami, takimi jak
> Haskell i Rust, brzmi: „jeśli kod się kompiluje, to działa”. Ale to powiedzenie nie jest
> uniwersalnie prawdziwe. Nasz projekt się kompiluje, ale absolutnie nic nie robi! Gdybyśmy
> budowali prawdziwy, kompletny projekt, byłby to dobry moment, aby
> zacząć pisać testy jednostkowe, aby sprawdzić, czy kod się kompiluje *i* zachowuje się tak, jak
> chcemy.

#### Validating the Number of Threads in `new`

Nie robimy nic z parametrami `new` i `execute`.
Zaimplementujmy ciała tych funkcji z zachowaniem, jakiego chcemy. Na początek
pomyślmy o `new`. Wcześniej wybraliśmy typ bez znaku dla parametru `size`,
ponieważ pula z ujemną liczbą wątków nie ma sensu.
Jednak pula z zerową liczbą wątków również nie ma sensu, a mimo to zero jest
całkowicie prawidłowym `usize`. Dodamy kod sprawdzający, czy `size` jest większe od zera, zanim
zwrócimy instancję `ThreadPool` i sprawimy, że program wpadnie w panikę, jeśli otrzyma
zero, używając makra `assert!`, jak pokazano na Liście 21-13.

<Listing number="21-13" file-name="src/lib.rs" caption="Implementing `ThreadPool::new` to panic if `size` is zero">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-13/src/lib.rs:here}}
```

</Listing>

Dodaliśmy również dokumentację dla naszego `ThreadPool` z komentarzami doc.
Zauważ, że postępowaliśmy zgodnie z dobrymi praktykami dokumentacji, dodając sekcję, która
wywołuje sytuacje, w których nasza funkcja może panikować, jak omówiono w
Rozdziale 14. Spróbuj uruchomić `cargo doc --open` i kliknąć strukturę `ThreadPool`,
aby zobaczyć, jak wygląda wygenerowana dokumentacja dla `new`!

Zamiast dodawać makro `assert!`, jak zrobiliśmy tutaj, moglibyśmy zmienić `new`
na `build` i zwrócić `Result`, tak jak zrobiliśmy to z `Config::build` w projekcie I/O
w Liście 12-9. Ale zdecydowaliśmy w tym przypadku, że próba utworzenia
puli wątków bez żadnych wątków powinna być nieodwracalnym błędem. Jeśli
czujesz się ambitny, spróbuj napisać funkcję o nazwie `build` z następującym
podpisem, aby porównać ją z funkcją `new`:
```rust,ignore
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### Creating Space to Store the Threads

Teraz, gdy mamy sposób, aby wiedzieć, że mamy prawidłową liczbę wątków do przechowywania w
puli, możemy utworzyć te wątki i przechowywać je w strukturze `ThreadPool`
przed zwróceniem struktury. Ale jak „przechowujemy” wątek? Przyjrzyjmy się jeszcze raz sygnaturze `thread::spawn`:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

Funkcja `spawn` zwraca `JoinHandle<T>`, gdzie `T` jest typem zwracanym przez
zamknięcie. Spróbujmy użyć również `JoinHandle` i zobaczmy, co się stanie. W naszym przypadku zamknięcia, które przekazujemy do puli wątków, obsłużą połączenie,
i niczego nie zwrócą, więc `T` będzie typem jednostki `()`.

Kod w listingu 21-14 zostanie skompilowany, ale nie utworzy jeszcze żadnych wątków.
Zmieniliśmy definicję `ThreadPool`, aby zawierała wektor instancji
`thread::JoinHandle<()>`, zainicjowaliśmy wektor o pojemności
`size`, skonfigurowaliśmy pętlę `for`, która uruchomi kod w celu utworzenia wątków i
zwróciliśmy instancję `ThreadPool` zawierającą je.
<Listing number="21-14" file-name="src/lib.rs" caption="Creating a vector for `ThreadPool` to hold the threads">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-14/src/lib.rs:here}}
```

</Listing>

Wprowadziliśmy `std::thread` do zakresu w skrzyni bibliotecznej, ponieważ używamy `thread::JoinHandle` jako typu elementów w wektorze w
`ThreadPool`.

Po otrzymaniu prawidłowego rozmiaru nasz `ThreadPool` tworzy nowy wektor, który może
przechowywać elementy `size`. Funkcja `with_capacity` wykonuje to samo zadanie co
`Vec::new`, ale z ważną różnicą: wstępnie przydziela miejsce w
wektorze. Ponieważ wiemy, że musimy przechowywać elementy `size` w wektorze, wykonanie
tego przydzielenia z góry jest nieco bardziej wydajne niż użycie `Vec::new`,
które zmienia rozmiar w miarę wstawiania elementów.

Gdy ponownie uruchomisz `cargo check`, powinno się to powieść.

#### A `Worker` Struct Responsible for Sending Code from the `ThreadPool` to a Thread

W pętli `for` w Listingu 21-14 pozostawiliśmy komentarz dotyczący tworzenia
wątków. Tutaj przyjrzymy się, jak faktycznie tworzymy wątki. Standardowa
biblioteka udostępnia `thread::spawn` jako sposób tworzenia wątków, a
`thread::spawn` oczekuje otrzymania pewnego kodu, który wątek powinien uruchomić, gdy tylko
wątek zostanie utworzony. Jednak w naszym przypadku chcemy utworzyć wątki i sprawić, by

czekały* na kod, który wyślemy później. Standardowa
implementacja wątków w bibliotece nie obejmuje żadnego sposobu, aby to zrobić; musimy
to zaimplementować ręcznie.

Zaimplementujemy to zachowanie, wprowadzając nową strukturę danych między
`ThreadPool` a wątkami, które będą zarządzać tym nowym zachowaniem. Nazwiemy
tę strukturę danych *Worker*, co jest powszechnym terminem w implementacjach
pulowania. Worker pobiera kod, który należy uruchomić i uruchamia kod w wątku Workera. Pomyśl o ludziach pracujących w kuchni w
restauracji: pracownicy czekają, aż przyjdą zamówienia od klientów, a następnie
są odpowiedzialni za przyjęcie tych zamówień i ich realizację.

Zamiast przechowywać wektor wystąpień `JoinHandle<()>` w puli wątków,
będziemy przechowywać wystąpienia struktury `Worker`. Każdy `Worker` będzie przechowywać pojedynczy
`JoinHandle<()>`. Następnie zaimplementujemy metodę w `Worker`, która
będzie przyjmować zamknięcie kodu do uruchomienia i wysyłać je do już działającego wątku w celu
wykonania. Nadamy również każdemu pracownikowi `id`, abyśmy mogli rozróżniać
różne pracownicy w puli podczas rejestrowania lub debugowania.

Oto nowy proces, który będzie miał miejsce, gdy utworzymy `ThreadPool`. Zaimplementujemy kod, który wysyła zamknięcie do wątku po skonfigurowaniu `Worker`
w ten sposób:

1. Zdefiniuj strukturę `Worker`, która zawiera `id` i `JoinHandle<()>`.
2. Zmień `ThreadPool`, aby zawierał wektor instancji `Worker`.
3. Zdefiniuj funkcję `Worker::new`, która przyjmuje numer `id` i zwraca
instancję `Worker`, która zawiera `id` i wątek utworzony z pustym
zamknięciem.
4. W `ThreadPool::new` użyj licznika pętli `for`, aby wygenerować `id`, utwórz
nowego `Worker` z tym `id` i zapisz workera w wektorze.

Jeśli jesteś gotowy na wyzwanie, spróbuj samodzielnie zaimplementować te zmiany, zanim
spojrzysz na kod w Liście 21-15.

Gotowy? Poniżej znajduje się Listing 21-15 przedstawiający jeden ze sposobów wprowadzenia powyższych modyfikacji.

<Listing number="21-15" file-name="src/lib.rs" caption="Modifying `ThreadPool` to hold `Worker` instances instead of holding threads directly">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-15/src/lib.rs:here}}
```

</Listing>

Zmieniliśmy nazwę pola w `ThreadPool` z `threads` na `workers`,
ponieważ teraz zawiera ono instancje `Worker` zamiast instancji `JoinHandle<()>`. Używamy licznika w pętli `for` jako argumentu dla
`Worker::new` i przechowujemy każdego nowego `Workera` w wektorze o nazwie `workers`.

Kod zewnętrzny (taki jak nasz serwer w *src/main.rs*) nie musi znać
szczegółów implementacji dotyczących użycia struktury `Worker` w `ThreadPool`,
więc czynimy strukturę `Worker` i jej funkcję `new` prywatnymi. Funkcja
`Worker::new` używa podanego przez nas `id` i przechowuje instancję `JoinHandle<()>`, która jest tworzona przez utworzenie nowego wątku przy użyciu pustego zamknięcia.

> Uwaga: Jeśli system operacyjny nie może utworzyć wątku, ponieważ nie ma
> wystarczających zasobów systemowych, `thread::spawn` wpadnie w panikę. Spowoduje to, że
> cały nasz serwer wpadnie w panikę, nawet jeśli utworzenie niektórych wątków może się
> powieść. Dla uproszczenia takie zachowanie jest w porządku, ale w produkcyjnej implementacji puli wątków prawdopodobnie będziesz chciał użyć
> [`std::thread::Builder`][builder]<!-- ignore --> i jej
> [`spawn`][builder-spawn]<!-- ignore --> metody, która zwraca `Result`.

Ten kod zostanie skompilowany i zapisze liczbę instancji `Worker`, które

określiliśmy jako argument dla `ThreadPool::new`. Ale *nadal* nie przetwarzamy
zamknięcia, które otrzymujemy w `execute`. Przyjrzyjmy się, jak to zrobić.

#### Sending Requests to Threads via Channels

Następnym problemem, z którym się zmierzymy, jest to, że zamknięcia nadane `thread::spawn` nie robią
absolutnie nic. Obecnie otrzymujemy zamknięcie, które chcemy wykonać w metodzie
`execute`. Ale musimy nadać `thread::spawn` zamknięcie do uruchomienia, gdy
tworzymy każdego `Workera` podczas tworzenia `ThreadPool`.

Chcemy, aby struktury `Worker`, które właśnie utworzyliśmy, pobierały kod do uruchomienia z
kolejki przechowywanej w `ThreadPool` i wysyłały ten kod do swojego wątku w celu uruchomienia.

Kanały, o których dowiedzieliśmy się w rozdziale 16 — prosty sposób komunikacji między
dwoma wątkami — byłyby idealne do tego przypadku użycia. Użyjemy kanału, aby funkcjonował
jako kolejka zadań, a `execute` wyśle ​​zadanie z `ThreadPool` do
instancji `Worker`, które wyślą zadanie do swojego wątku. Oto plan:

1. `ThreadPool` utworzy kanał i będzie przechowywał nadawcę.
2. Każdy `Worker` będzie przechowywał odbiorcę.
3. Utworzymy nową strukturę `Job`, która będzie przechowywała zamknięcia, które chcemy wysłać
kanałem.
4. Metoda `execute` wyśle ​​zadanie, które chce wykonać, przez
nadawcę.
5. W swoim wątku `Worker` będzie pętlą po swoim odbiorcy i wykona
zamknięcia wszystkich otrzymanych zadań.

Zacznijmy od utworzenia kanału w `ThreadPool::new` i przechowywania nadawcy
w instancji `ThreadPool`, jak pokazano w Liście 21-16. Struktura `Job`
na razie niczego nie przechowuje, ale będzie typem elementu, który wysyłamy
kanałem.
<Listing number="21-16" file-name="src/lib.rs" caption="Modifying `ThreadPool` to store the sender of a channel that transmits `Job` instances">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-16/src/lib.rs:here}}
```

</Listing>

W `ThreadPool::new` tworzymy nasz nowy kanał i pula przechowuje
nadawcę. To się pomyślnie skompiluje.

Spróbujmy przekazać odbiornik kanału do każdego pracownika, gdy pula wątków
tworzy kanał. Wiemy, że chcemy użyć odbiornika w wątku, który
pracownicy tworzą, więc odwołamy się do parametru `receiver` w zamknięciu. Kod w Listingu 21-17 jeszcze się nie skompiluje.

<Listing number="21-17" file-name="src/lib.rs" caption="Passing the receiver to the workers">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-17/src/lib.rs:here}}
```

</Listing>

Wprowadziliśmy kilka małych i prostych zmian: przekazujemy odbiornik do
`Worker::new`, a następnie używamy go wewnątrz zamknięcia.

Gdy próbujemy sprawdzić ten kod, otrzymujemy następujący błąd:

```console
{{#include ../listings/ch21-web-server/listing-21-17/output.txt}}
```

Kod próbuje przekazać `receiver` do wielu instancji `Worker`. To
nie zadziała, jak pamiętasz z rozdziału 16: implementacja kanału, którą
Rust zapewnia, to wielu *producentów*, jeden *konsumentów*. Oznacza to, że nie możemy
po prostu sklonować końca konsumującego kanału, aby naprawić ten kod. Nie
chcemy również wysyłać wiadomości wiele razy do wielu konsumentów; chcemy jednej listy
wiadomości z wieloma pracownikami, tak aby każda wiadomość została przetworzona raz.

Ponadto, usunięcie zadania z kolejki kanału wiąże się z mutacją
`receiver`, więc wątki potrzebują bezpiecznego sposobu udostępniania i modyfikowania `receiver`;
w przeciwnym razie możemy uzyskać warunki wyścigu (jak opisano w rozdziale 16).

Przypomnij sobie bezpieczne dla wątków inteligentne wskaźniki omówione w rozdziale 16: aby współdzielić
własność między wieloma wątkami i umożliwić wątkom mutację wartości,
musimy użyć `Arc<Mutex<T>>`. Typ `Arc` pozwoli wielu pracownikom posiadać
odbiornik, a `Mutex` zapewni, że tylko jeden pracownik otrzyma zadanie od
odbiornika w danym momencie. Listing 21-18 pokazuje zmiany, które musimy wprowadzić.

<Listing number="21-18" file-name="src/lib.rs" caption="Sharing the receiver among the workers using `Arc` and `Mutex`">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-18/src/lib.rs:here}}
```

</Listing>

W `ThreadPool::new` umieszczamy odbiornik w `Arc` i `Mutex`. Dla każdego
nowego pracownika klonujemy `Arc`, aby zwiększyć liczbę odwołań, dzięki czemu pracownicy mogą
dzielić własność odbiornika.

Dzięki tym zmianom kod się kompiluje! Docieramy do celu!

#### Implementing the `execute` Method

W końcu zaimplementujmy metodę `execute` w `ThreadPool`. Zmienimy również
`Job` ze struktury na alias typu dla obiektu cechy, który przechowuje typ zamknięcia,
które otrzymuje `execute`. Jak omówiono w sekcji [„Tworzenie synonimów typów
z aliasami typów”][creating-type-synonyms-with-type-aliases]<!-- ignore -->
rozdziału 20, aliasy typów pozwalają nam skracać długie typy,
aby ułatwić ich używanie. Spójrz na Listing 21-19.

<Listing number="21-19" file-name="src/lib.rs" caption="Creating a `Job` type alias for a `Box` that holds each closure and then sending the job down the channel">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-19/src/lib.rs:here}}
```

</Listing>

Po utworzeniu nowej instancji `Job` przy użyciu zamknięcia, które otrzymujemy w `execute`,
wysyłamy to zadanie do wysyłającego końca kanału. Wywołujemy `unwrap` w
`send` w przypadku, gdy wysyłanie się nie powiedzie. Może się tak zdarzyć, na przykład, jeśli
zatrzymamy wszystkie nasze wątki przed wykonywaniem, co oznacza, że ​​odbiorca przestał
odbierać nowe wiadomości. W tej chwili nie możemy zatrzymać naszych wątków przed
wykonywaniem: nasze wątki kontynuują wykonywanie tak długo, jak istnieje pula.
Powodem, dla którego używamy `unwrap` jest to, że wiemy, że przypadek awarii nie wystąpi, ale
kompilator o tym nie wie.

Ale jeszcze nie skończyliśmy! W procesie roboczym nasze zamknięcie przekazywane do
`thread::spawn` nadal *odwołuje* się tylko do odbierającego końca kanału.
Zamiast tego potrzebujemy, aby zamknięcie pętliło w nieskończoność, prosząc odbierający koniec kanału
o zadanie i uruchamiając zadanie, gdy je otrzyma. Wprowadźmy zmianę pokazaną na Liście 21-20 na `Worker::new`.

<Listing number="21-20" file-name="src/lib.rs" caption="Receiving and executing the jobs in the worker’s thread">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-20/src/lib.rs:here}}
```

</Listing>

Tutaj najpierw wywołujemy `lock` na `receiver`, aby uzyskać mutex, a następnie wywołujemy `unwrap`, aby panikować przy jakichkolwiek błędach. Uzyskanie blokady może się nie powieść, jeśli mutex
jest w stanie *zatrucia*, co może się zdarzyć, jeśli jakiś inny wątek wpadł w panikę,
trzymając blokadę zamiast ją zwolnić. W tej sytuacji wywołanie
`unwrap`, aby ten wątek panikował, jest właściwym działaniem. Możesz
zmienić to `unwrap` na `expect` z komunikatem o błędzie, który jest dla Ciebie zrozumiały.

Jeśli uzyskamy blokadę mutexu, wywołujemy `recv`, aby odebrać `Job` z
kanału. Ostateczne `unwrap` również omija wszelkie błędy, które mogą wystąpić,
jeśli wątek trzymający nadawcę został wyłączony, podobnie jak metoda `send`
zwraca `Err`, jeśli odbiorca zostanie wyłączony.

Wywołanie `recv` blokuje, więc jeśli nie ma jeszcze zadania, bieżący wątek
będzie czekać, aż zadanie stanie się dostępne. `Mutex<T>` zapewnia, że ​​tylko jeden
wątek `Worker` na raz próbuje zażądać zadania.

Nasza pula wątków jest teraz w stanie roboczym! Wykonaj `cargo run` i złóż kilka
żądań:

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-20
cargo run
make some requests to 127.0.0.1:7878
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field is never read: `workers`
 --> src/lib.rs:7:5
  |
7 |     workers: Vec<Worker>,
  |     ^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: field is never read: `id`
  --> src/lib.rs:48:5
   |
48 |     id: usize,
   |     ^^^^^^^^^

warning: field is never read: `thread`
  --> src/lib.rs:49:5
   |
49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

warning: `hello` (lib) generated 3 warnings
    Finished dev [unoptimized + debuginfo] target(s) in 1.40s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
```

Sukces! Mamy teraz pulę wątków, która wykonuje połączenia asynchronicznie.
Nigdy nie tworzy się więcej niż cztery wątki, więc nasz system nie zostanie
przeciążony, jeśli serwer otrzyma wiele żądań. Jeśli wyślemy żądanie do
*/sleep*, serwer będzie mógł obsłużyć inne żądania, zlecając ich uruchomienie innemu
wątkowi.

> Uwaga: Jeśli otworzysz */sleep* w wielu oknach przeglądarki jednocześnie,
> mogą one ładować się pojedynczo w 5-sekundowych odstępach. Niektóre przeglądarki internetowe wykonują
> wiele wystąpień tego samego żądania sekwencyjnie ze względu na buforowanie. To
> ograniczenie nie jest spowodowane przez nasz serwer internetowy.

Po zapoznaniu się z pętlą `while let` w rozdziałach 17 i 18, możesz
zastanawiać się, dlaczego nie napisaliśmy kodu wątku roboczego, jak pokazano na liście 21-21.
<Listing number="21-21" file-name="src/lib.rs" caption="An alternative implementation of `Worker::new` using `while let`">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-21/src/lib.rs:here}}
```

</Listing>

Ten kod kompiluje się i uruchamia, ale nie powoduje pożądanego zachowania wątków: powolne żądanie nadal spowoduje, że inne żądania będą czekać na przetworzenie. Powód jest nieco subtelny: struktura `Mutex` nie ma publicznej metody `unlock`, ponieważ własność blokady opiera się na czasie życia `MutexGuard<T>` w `LockResult<MutexGuard<T>>`, którą zwraca metoda `lock`. W czasie kompilacji program sprawdzający pożyczkę może następnie wymusić regułę, że zasób chroniony przez `Mutex` nie może być dostępny, chyba że trzymamy blokadę. Jednak ta implementacja może również spowodować, że blokada będzie trzymana dłużej, niż zamierzaliśmy, jeśli nie będziemy świadomi czasu życia
`MutexGuard<T>`.

Kod w Liście 21-20, który używa `let job =
receiver.lock().unwrap().recv().unwrap();` działa, ponieważ w przypadku `let` wszelkie
wartości tymczasowe używane w wyrażeniu po prawej stronie znaku równości
są natychmiast usuwane po zakończeniu instrukcji `let`. Jednak `while
let` (oraz `if let` i `match`) nie usuwa wartości tymczasowych do końca
skojarzonego bloku. W Liście 21-21 blokada pozostaje utrzymywana przez czas trwania wywołania `job()`, co oznacza, że ​​inni pracownicy nie mogą otrzymywać zadań.

[creating-type-synonyms-with-type-aliases]:
ch20-04-advanced-types.html#creating-type-synonyms-with-type-aliases
[integer-types]: ch03-02-data-types.html#integer-types
[fn-traits]:
ch13-01-closures.html#moving-captured-values-out-of-the-closure-and-the-fn-traits
[builder]: ../std/thread/struct.Builder.html
[builder-spawn]: ../std/thread/struct.Builder.html#method.spawn
