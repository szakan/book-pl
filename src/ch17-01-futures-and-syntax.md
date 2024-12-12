## Futures i Składnia asynchroniczna

Kluczowymi elementami programowania asynchronicznego w Rust są *futures* oraz słowa kluczowe Rust
`async` i `await`.

*Future* to wartość, która może nie być gotowa teraz, ale stanie się gotowa w pewnym
momencie w przyszłości. (Ta sama koncepcja pojawia się w wielu językach, czasami
pod innymi nazwami, takimi jak „task” lub „promise”). Rust zapewnia cechę `Future`
jako element konstrukcyjny, dzięki czemu różne operacje asynchroniczne mogą być implementowane
z różnymi strukturami danych, ale ze wspólnym interfejsem. W Rust mówimy, że
typy implementujące cechę `Future` są futures. Każdy typ,
który implementuje `Future`, przechowuje własne informacje o postępie,
który został
osiągnięty i co oznacza „gotowy”.

Słowo kluczowe `async` można stosować do bloków i funkcji, aby określić, że
mogą być one przerywane i wznawiane. W obrębie bloku asynchronicznego lub funkcji asynchronicznej możesz
używać słowa kluczowego `await`, aby czekać, aż przyszłość stanie się gotowa, nazywane *oczekiwaniem na
przyszłość*. Każde miejsce, w którym oczekujesz na przyszłość w bloku asynchronicznym lub funkcji, jest
miejscem, w którym blok asynchroniczny lub funkcja mogą zostać wstrzymane i wznowione. Proces
sprawdzania w przyszłości, czy jej wartość jest już dostępna, nazywa się *sondowaniem*.

Niektóre inne języki również używają słów kluczowych `async` i `await` do
programowania asynchronicznego. Jeśli znasz te języki, możesz zauważyć pewne
znaczące różnice w sposobie działania Rusta, w tym w sposobie obsługi
składni. Jest ku temu dobry powód, jak zobaczymy!

Przez większość czasu podczas pisania asynchronicznego Rusta używamy słów kluczowych `async` i `await`. Rust kompiluje je do równoważnego kodu przy użyciu cechy `Future`, podobnie jak kompiluje pętle `for` do równoważnego kodu przy użyciu cechy `Iterator`.
Ponieważ Rust zapewnia cechę `Future`, możesz ją również zaimplementować dla
własnych typów danych, gdy zajdzie taka potrzeba. Wiele funkcji, które zobaczymy
w tym rozdziale, zwraca typy z własnymi implementacjami `Future`.
Wrócimy do definicji cechy pod koniec rozdziału i zagłębimy się
w to, jak ona działa, ale to wystarczająco dużo szczegółów, abyśmy mogli iść dalej.

To wszystko może wydawać się trochę abstrakcyjne. Napiszmy nasz pierwszy program asynchroniczny: mały
web scraper. Przekażemy dwa adresy URL z wiersza poleceń, pobierzemy oba
jednocześnie i zwrócimy wynik tego, który zakończy się jako pierwszy. Ten
przykład będzie miał sporo nowej składni, ale nie martw się. Wyjaśnimy
wszystko, co musisz wiedzieć, w trakcie.
### Nasz pierwszy program asynchroniczny

Aby ten rozdział skupiał się na nauce asynchroniczności, a nie na żonglowaniu częściami
ekosystemu, stworzyliśmy skrzynię `trpl` (`trpl` to skrót od „Języka programowania Rust”). Ponownie eksportuje ona wszystkie typy, cechy i funkcje,
których będziesz potrzebować, głównie ze skrzynek [`futures`][futures-crate] i [`tokio`][tokio].

- Skrzynia `futures` jest oficjalnym domem dla eksperymentów Rust dla kodu asynchronicznego i to właśnie tam pierwotnie zaprojektowano typ `Future`.

- Tokio jest obecnie najszerzej używanym asynchronicznym środowiskiem wykonawczym w Rust, szczególnie (ale
nie tylko!) w aplikacjach internetowych. Istnieją inne świetne środowiska wykonawcze,
które mogą być bardziej odpowiednie dla Twoich celów. Używamy Tokio pod maską,
ponieważ jest dobrze przetestowane i szeroko stosowane.

W niektórych przypadkach `trpl` zmienia również nazwy lub opakowuje oryginalne interfejsy API, abyśmy mogli skupić się na szczegółach dotyczących tego rozdziału. Jeśli chcesz zrozumieć, co robi skrzynia, zachęcamy do sprawdzenia [jej kodu źródłowego][crate-source].
Będziesz mógł zobaczyć, z której skrzyni pochodzi każdy reeksport, a my pozostawiliśmy
obszerne komentarze wyjaśniające, co robi skrzynia.

Utwórz nowy projekt binarny o nazwie `hello-async` i dodaj skrzynię `trpl` jako
zależność:

```console
$ cargo new hello-async
$ cd hello-async
$ cargo add trpl
```

Teraz możemy użyć różnych części dostarczonych przez `trpl` do napisania naszego pierwszego programu asynchronicznego. Zbudujemy małe narzędzie wiersza poleceń, które pobiera dwie strony internetowe,
wyciąga element `<title>` z każdej i drukuje tytuł tej, która najpierw zakończy cały proces.

Zacznijmy od napisania funkcji, która przyjmuje adres URL jednej strony jako parametr, wysyła do niego żądanie i zwraca tekst elementu tytułu:
<Listing number="17-1" file-name="src/main.rs" caption="Defining an async function to get the title element from an HTML page">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-01/src/main.rs:all}}
```

</Listing>

W Listingu 17-1 definiujemy funkcję o nazwie `page_title` i oznaczamy ją
słowem kluczowym `async`. Następnie używamy funkcji `trpl::get`, aby pobrać dowolny adres URL,
i, i oczekujemy na odpowiedź, używając słowa kluczowego `await`. Następnie
pobieramy tekst odpowiedzi, wywołując jej metodę `text` i ponownie
oczekując na nią słowem kluczowym `await`. Oba te kroki są asynchroniczne. W przypadku
`get` musimy czekać, aż serwer odeśle pierwszą część swojej
odpowiedzi, która będzie zawierać nagłówki HTTP, pliki cookie itd. Ta część
odpowiedzi może zostać dostarczona oddzielnie od treści żądania. Zwłaszcza jeśli
treść jest bardzo duża, może minąć trochę czasu, zanim wszystko dotrze. Dlatego
musimy czekać na *całość* odpowiedzi, więc metoda `text`
jest również asynchroniczna.

Musimy jawnie oczekiwać na oba te futures, ponieważ futures w Rust są
*leniwe*: nie robią nic, dopóki nie poprosisz ich o to za pomocą `await`. (W rzeczywistości,
Rust wyświetli ostrzeżenie kompilatora, jeśli nie użyjesz futures.) Powinno to
przypomnieć ci naszą dyskusję na temat iteratorów [w rozdziale 13][iterators-lazy].
Iteratory nie robią nic, dopóki nie wywołasz ich metody `next` — czy to bezpośrednio, czy
używając pętli `for` lub metod takich jak `map`, które używają `next` w tle. W przypadku
futures obowiązuje ta sama podstawowa idea: nie robią nic, dopóki nie poprosisz ich
o to jawnie. To lenistwo pozwala Rustowi unikać uruchamiania kodu asynchronicznego, dopóki nie jest
naprawdę potrzebny.

> Uwaga: To różni się od zachowania, które widzieliśmy podczas używania `thread::spawn` w
> poprzednim rozdziale, gdzie zamknięcie, które przekazaliśmy do innego wątku, zaczęło
> działać natychmiast. Jest to również inne niż podejście wielu innych języków
> do asynchroniczności! Ale jest to ważne dla Rust. Zobaczymy, dlaczego tak jest później.

Gdy mamy `response_text`, możemy go przeanalizować do instancji typu
`Html` za pomocą `Html::parse`. Zamiast surowego ciągu mamy teraz typ danych, którego możemy użyć do pracy z HTML jako bogatszą strukturą danych. W szczególności,
możemy użyć metody `select_first`, aby znaleźć pierwszą instancję danego selektora CSS. Przekazując ciąg `"title"`, otrzymamy pierwszy element `<title>`
w dokumencie, jeśli taki istnieje. Ponieważ może nie być żadnego pasującego
elementu, `select_first` zwraca `Option<ElementRef>`. Na koniec używamy metody
`Option::map`, która pozwala nam pracować z elementem w `Option`, jeśli jest on
obecny, i nic nie robić, jeśli nie jest. (Możemy również użyć wyrażenia `match`
tutaj, ale `map` jest bardziej idiomatyczne.) W treści funkcji, którą przekazujemy do
`map`, wywołujemy `inner_html` na `title_element`, aby uzyskać jego zawartość, która jest
`String`. Kiedy wszystko jest powiedziane i zrobione, mamy `Option<String>`.

Zauważ, że słowo kluczowe Rusta `await` znajduje się po wyrażeniu, na które czekasz,
a nie przed nim. To znaczy, że jest to *słowo kluczowe postfix*. Może się to różnić od tego,
do czego możesz być przyzwyczajony, jeśli używałeś async w innych językach. Rust wybrał
to, ponieważ sprawia, że ​​łańcuchy metod są o wiele przyjemniejsze w pracy. W rezultacie,
możemy zmienić treść `page_url_for`, aby połączyć wywołania funkcji `trpl::get` i `text`
z `await` między nimi, jak pokazano w Listingu 17-2:

<Listing number="17-2" file-name="src/main.rs" caption="Chaining with the `await` keyword">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-02/src/main.rs:chaining}}
```

</Listing>

Dzięki temu udało nam się napisać naszą pierwszą funkcję asynchroniczną! Zanim dodamy
trochę kodu w `main`, aby ją wywołać, porozmawiajmy trochę więcej o tym, co
napisaliśmy i co to oznacza.

Gdy Rust widzi blok oznaczony słowem kluczowym `async`, kompiluje go do
unikalnego, anonimowego typu danych, który implementuje cechę `Future`. Gdy Rust widzi
funkcję oznaczoną słowem kluczowym `async`, kompiluje ją do funkcji nieasynchronicznej, której
ciało jest blokiem asynchronicznym. Typ zwracany przez funkcję asynchroniczną to typ
anonimowego typu danych, który kompilator tworzy dla tego bloku asynchronicznego.

W związku z tym napisanie `async fn` jest równoważne z napisaniem funkcji, która zwraca
*future* typu zwracanego. Gdy kompilator widzi definicję funkcji, taką jak `async fn page_title` w Liście 17-1, jest to równoważne z nieasynchroniczną
funkcją zdefiniowaną w następujący sposób:

```rust
# extern crate trpl; // required for mdbook test
use std::future::Future;
use trpl::Html;

fn page_title(url: &str) -> impl Future<Output = Option<String>> + '_ {
    async move {
        let text = trpl::get(url).await.text().await;
        Html::parse(&text)
            .select_first("title")
            .map(|title| title.inner_html())
    }
}
```

Przeanalizujmy każdą część przekształconej wersji:

* Używa składni `impl Trait`, którą omówiliśmy w sekcji [„Cechy jako

Parametry”][impl-trait] w rozdziale 10.
* Zwrócona cecha to `Future`, z powiązanym typem `Output`. Zauważ,
że typem `Output` jest `Option<String>`, który jest taki sam jak

oryginalny typ zwracany z wersji `async fn` `page_title`.
* Cały kod wywoływany w treści oryginalnej funkcji jest zawinięty w blok
`async move`. Pamiętaj, że bloki są wyrażeniami. Cały ten blok jest
wyrażeniem zwróconym z funkcji.
* Ten asynchroniczny blok generuje wartość typu `Option<String>`, jak opisano

powyżej. Ta wartość pasuje do typu `Output` w typie zwracanym. Jest to
podobne do innych bloków, które widziałeś.
* Nowe ciało funkcji to blok `async move` ze względu na sposób, w jaki używa parametru
`url`. (O `async` i `async move` porozmawiamy znacznie szerzej później
w tym rozdziale.)
* Nowa wersja funkcji ma rodzaj czasu życia, którego wcześniej nie widzieliśmy
w typie wyjściowym: `'_`. Ponieważ funkcja zwraca `Future`, który odwołuje się
do referencji — w tym przypadku referencji z parametru `url` — musimy
powiedzieć Rust, że chcemy, aby ta referencja została uwzględniona. Nie musimy
tutaj nazywać czasu życia, ponieważ Rust jest wystarczająco inteligentny, aby wiedzieć, że istnieje tylko jedna
referencja, która może być zaangażowana, ale *musimy* wyraźnie określić, że
wynikowa `Future` jest ograniczona tym czasem życia.

Teraz możemy wywołać `page_title` w `main`. Na początek po prostu otrzymamy tytuł
pojedynczej strony. W Listingu 17-3 stosujemy ten sam wzór, którego użyliśmy do
pobierania argumentów wiersza poleceń w rozdziale 12. Następnie przekazujemy pierwszy adres URL
`page_title` i czekamy na wynik. Ponieważ wartość wygenerowana przez future to
`Option<String>`, używamy wyrażenia `match`, aby wydrukować różne komunikaty,
aby uwzględnić, czy strona miała `<title>`.

<Listing number="17-3" file-name="src/main.rs" caption="Calling the `page_title` function from `main` with a user-supplied argument">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-03/src/main.rs:main}}
```

</Listing>

Niestety, to się nie kompiluje. Jedyne miejsce, w którym możemy użyć słowa kluczowego `await`, to funkcje async lub bloki, a Rust nie pozwoli nam oznaczyć specjalnej funkcji `main` jako `async`.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-03
cargo build
copy just the compiler error
-->

```text
error[E0752]: funkcja `main` nie może być `async`
--> src/main.rs:6:1
|
6 | async fn main() {
| ^^^^^^^^^^^^^^^^^ Funkcja `main` nie może być `async`
```

Powodem, dla którego `main` nie może być oznaczone jako `async`, jest to, że kod asynchroniczny wymaga *runtime*:
skrzyni Rust, która zarządza szczegółami wykonywania kodu asynchronicznego. Funkcja `main`
programu może *inicjalizować* runtime, ale nie jest runtime
*sama*. (Więcej o tym, dlaczego tak się dzieje, dowiemy się nieco później.) Każdy program Rust,
który wykonuje kod asynchroniczny, ma co najmniej jedno miejsce, w którym konfiguruje runtime i
wykonywa futures.

Większość języków obsługujących async dołącza runtime do języka. Rust nie
działa. Zamiast tego dostępnych jest wiele różnych asynchronicznych środowisk wykonawczych, z których każde
pozwala na różne kompromisy odpowiednie do docelowego przypadku użycia. Na przykład
wysokoprzepustowy serwer WWW z wieloma rdzeniami procesora i dużą ilością pamięci RAM ma
zupełnie inne potrzeby niż mikrokontroler z jednym rdzeniem, małą ilością
pamięci RAM i bez możliwości alokacji sterty. Skrzynie, które zapewniają te środowiska wykonawcze, często dostarczają również asynchroniczne wersje typowych funkcji, takich jak
wejście/wyjście pliku lub sieci.

Tutaj i przez resztę tego rozdziału użyjemy funkcji `run`
ze skrzyni `trpl`, która przyjmuje przyszłość jako argument i uruchamia ją do
zakończenia. W tle wywołanie `run` ustawia środowisko wykonawcze, które ma zostać użyte do uruchomienia przekazanej
przyszłości. Po zakończeniu przyszłości `run` zwraca dowolną wartość wygenerowaną przez
przyszłość.

Możemy przekazać przyszłość zwróconą przez `page_title` bezpośrednio do `run`. Po
ukończeniu będziemy mogli dopasować wynikowy `Option<String>`, w sposób, w jaki próbowaliśmy to zrobić w Liście 17-3. Jednak w większości przykładów w tym rozdziale
(i większości asynchronicznego kodu w świecie rzeczywistym!), będziemy wykonywać więcej niż jedno
asynchroniczne wywołanie funkcji, więc zamiast tego przekażemy blok `async` i jawnie
oczekujemy na wynik wywołania `page_title`, jak w Liście 17-4.

<Listing number="17-4" caption="Awaiting an async block with `trpl::run`" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-04/src/main.rs:run}}
```

</Listing>

When we run this, we get the behavior we might have expected initially:

```console
{{#include ../listings/ch17-async-await/listing-17-04/output.txt}}
```

Uff: w końcu mamy działający kod asynchroniczny! Teraz się kompiluje i możemy go uruchomić. Zanim dodamy kod, aby ścigać się ze sobą na dwóch stronach, skupmy się na tym, jak działają futures.

Każdy *punkt oczekiwania* — to znaczy każde miejsce, w którym kod używa słowa kluczowego `await` — reprezentuje miejsce, w którym kontrola jest przekazywana z powrotem do środowiska wykonawczego. Aby to zadziałało, Rust musi śledzić stan zaangażowany w blok asynchroniczny, aby środowisko wykonawcze mogło rozpocząć jakąś inną pracę, a następnie wrócić, gdy będzie gotowe, aby spróbować ponownie wykonać tę. To niewidzialna maszyna stanowa,
jakbyś napisał wyliczenie w ten sposób, aby zapisać bieżący stan w każdym punkcie `await`
:

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-state-machine/src/lib.rs:enum}}
```

Ręczne pisanie kodu do przechodzenia między każdym stanem byłoby żmudne i podatne na
błędy, szczególnie gdy później dodasz więcej funkcjonalności i więcej stanów do kodu. Zamiast tego kompilator Rust automatycznie tworzy i zarządza strukturami danych maszyny stanowej
dla kodu asynchronicznego. Jeśli się zastanawiasz: tak, wszystkie
normalne reguły pożyczania i własności dotyczące struktur danych mają zastosowanie. Na szczęście,
kompilator również zajmuje się ich sprawdzaniem za nas i ma dobre komunikaty o błędach.
Przeanalizujemy kilka z nich później w tym rozdziale!

Ostatecznie coś musi wykonać tę maszynę stanową. Tym czymś jest
środowisko wykonawcze. (Dlatego czasami możesz natknąć się na odniesienia do *executorów*
podczas analizy środowisk wykonawczych: executor to część środowiska wykonawczego odpowiedzialna za
wykonywanie kodu asynchronicznego.)

Teraz możemy zrozumieć, dlaczego kompilator powstrzymał nas przed uczynieniem `main` funkcją asynchroniczną w Listingu 17-3. Gdyby `main` było funkcją asynchroniczną, coś
innego musiałoby zarządzać maszyną stanową dla dowolnej przyszłej funkcji `main` zwróconej,
ale `main` jest punktem wyjścia dla programu! Zamiast tego wywołujemy funkcję
`trpl::run` w `main`, która konfiguruje środowisko wykonawcze i uruchamia przyszłe
zwrócone przez blok `async`, dopóki nie zwróci `Ready`.

> Uwaga: niektóre środowiska wykonawcze udostępniają makra, dzięki którym *można* napisać asynchroniczną funkcję main
>. Te makra przepisują `async fn main() { ... }` na normalną `fn
> main`, która robi to samo, co zrobiliśmy ręcznie w Liście 17-5: wywołuje funkcję
>, która uruchamia przyszłość do końca w sposób, w jaki robi to `trpl::run`.

Złóżmy te elementy razem i zobaczmy, jak możemy napisać współbieżny kod, wywołując `page_title` z dwoma różnymi adresami URL przekazanymi z wiersza poleceń
i ścigając się z nimi.

<Listing number="17-5" caption="" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-05/src/main.rs:all}}
```

</Listing>

W Listingu 17-5 zaczynamy od wywołania `page_title` dla każdego z podanych przez użytkownika
adresów URL. Zapisujemy futures wygenerowane przez wywołanie `page_title` jako `title_fut_1` i
`title_fut_2`. Pamiętaj, że one jeszcze nic nie robią, ponieważ futures są leniwe,
a my jeszcze na nie nie czekaliśmy. Następnie przekazujemy futures do `trpl::race`,
który zwraca wartość wskazującą, która z przekazanych mu futures kończy się
pierwsza.

> Uwaga: pod maską `race` jest zbudowany na bardziej ogólnej funkcji, `select`,
> którą częściej spotkasz w rzeczywistym kodzie Rust. Funkcja `select`
> może zrobić wiele rzeczy, których funkcja `trpl::race` nie potrafi,
> ale ma też pewną dodatkową złożoność, którą możemy teraz pominąć.

Każda futures może legalnie „wygrać”, więc nie ma sensu zwracać
`Result`. Zamiast tego `race` zwraca typ, którego wcześniej nie widzieliśmy,
`trpl::Either`. Typ `Either` jest nieco podobny do `Result`, ponieważ
ma dwa przypadki. Jednak w przeciwieństwie do `Result`, w `Either` nie ma wbudowanego pojęcia sukcesu lub
porażki. Zamiast tego używa `Left` i `Right`, aby wskazać
„jedno lub drugie”.

```rust
enum Either<A, B> {
    Left(A),
    Right(B),
}
```

Funkcja `race` zwraca `Left`, jeśli pierwszy argument zakończy się jako pierwszy, z
danym wynikiem w przyszłości, i `Right` z wynikiem drugiego przyszłego argumentu, jeśli
*ten* zakończy się jako pierwszy. Jest to zgodne z kolejnością, w jakiej argumenty pojawiają się podczas
wywoływania funkcji: pierwszy argument znajduje się po lewej stronie drugiego argumentu.

Aktualizujemy również `page_title`, aby zwracał ten sam adres URL, który został przekazany. W ten sposób, jeśli
strona, która zwraca pierwszy, nie ma `<title>`, który możemy rozwiązać, możemy
nadal wydrukować sensowną wiadomość. Mając dostęp do tych informacji, kończymy,
aktualizując nasze wyjście `println!`, aby wskazać zarówno, który adres URL zakończył się jako pierwszy, jak i
jaki był `<title>` dla strony internetowej pod tym adresem URL, jeśli taki był.

Zbudowałeś teraz mały, działający skrobak sieciowy! Wybierz kilka adresów URL i uruchom
narzędzie wiersza poleceń. Możesz odkryć, że niektóre witryny są niezawodnie szybsze niż
inne, podczas gdy w innych przypadkach, która strona „wygrywa”, różni się od uruchomienia do uruchomienia. Co
ważniejsze, poznałeś podstawy pracy z futures, więc teraz możemy
zgłębić jeszcze bardziej rzeczy, które możemy zrobić za pomocą asynchroniczności.

[impl-trait]: ch10-02-traits.html#traits-as-parameters
[iterators-lazy]: ch13-02-iterators.html
<!-- TODO: map source link version to version of Rust? -->
[crate-source]: https://github.com/rust-lang/book/tree/main/packages/trpl
[futures-crate]: https://crates.io/crates/futures
[tokio]: https://tokio.rs
