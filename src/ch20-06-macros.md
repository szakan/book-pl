## Makra

Używaliśmy makr takich jak `println!` w całej tej książce, ale nie zbadaliśmy dokładnie, czym jest makro i jak działa. Termin *makro* odnosi się do rodziny
funkcji w Rust: *deklaratywnych* makr z `macro_rules!` i trzech rodzajów
makr *proceduralnych*:

* Niestandardowe makra `#[derive]`, które określają kod dodany z atrybutem `derive`
używanym w strukturach i wyliczeniach
* Makra podobne do atrybutów, które definiują niestandardowe atrybuty, których można używać w dowolnym elemencie
* Makra podobne do funkcji, które wyglądają jak wywołania funkcji, ale działają na tokenach
określonych jako ich argument

Omówimy każdy z nich po kolei, ale najpierw przyjrzyjmy się, dlaczego w ogóle
potrzebujemy makr, skoro już mamy funkcje.

### Różnica między makro a funkcją

Zasadniczo makra są sposobem pisania kodu, który pisze inny kod, co jest znane jako *metaprogramowanie*. W dodatku C omawiamy atrybut `derive`,
który generuje dla Ciebie implementację różnych cech.
Używaliśmy również makr `println!` i `vec!` w całej książce. Wszystkie te
makra *rozszerzają* się, aby wygenerować więcej kodu niż ten, który napisałeś ręcznie.

Metaprogramowanie jest przydatne do zmniejszania ilości kodu, który musisz napisać i
utrzymywać, co jest również jedną z ról funkcji. Jednak makra mają
pewne dodatkowe moce, których nie mają funkcje.

Sygnatura funkcji musi deklarować liczbę i typ parametrów, które
ma funkcja. Makra z drugiej strony mogą przyjmować zmienną liczbę
parametrów: możemy wywołać `println!("hello")` z jednym argumentem lub
`println!("hello {}", name)` z dwoma argumentami. Ponadto makra są rozszerzane
zanim kompilator zinterpretuje znaczenie kodu, więc makro może, na przykład, zaimplementować cechę w danym typie. Funkcja nie może, ponieważ jest
wywoływana w czasie wykonywania, a cecha musi zostać zaimplementowana w czasie kompilacji.

Wadą implementowania makra zamiast funkcji jest to, że
definicje makr są bardziej złożone niż definicje funkcji, ponieważ piszesz
kod Rust, który pisze kod Rust. Ze względu na tę pośredniość definicje makr są
ogólnie trudniejsze do odczytania, zrozumienia i utrzymania niż definicje funkcji.

Inną ważną różnicą między makrami a funkcjami jest to, że musisz
zdefiniować makra lub umieścić je w zakresie *przed* wywołaniem ich w pliku, w przeciwieństwie do funkcji, które możesz zdefiniować w dowolnym miejscu i wywołać w dowolnym miejscu.

### Deklaratywna Makra z `macro_rules!` do ogólnego metaprogramowania

Najczęściej używaną formą makr w Rust jest *makro deklaratywne*. Są one czasami nazywane „makrami na przykład”, „makrami `macro_rules!`”
lub po prostu „makrami”. W swojej istocie makra deklaratywne pozwalają napisać
coś podobnego do wyrażenia Rust `match`. Jak omówiono w rozdziale 6,
wyrażenia `match` to struktury sterujące, które przyjmują wyrażenie, porównują
wartość wynikową wyrażenia ze wzorcami, a następnie uruchamiają kod skojarzony
z pasującym wzorcem. Makra porównują również wartość ze wzorcami, które są
skojarzone z konkretnym kodem: w tej sytuacji wartością jest dosłowny
kod źródłowy Rust przekazany do makra; wzorce są porównywane ze
strukturą tego kodu źródłowego; a kod skojarzony z każdym wzorcem, gdy jest
dopasowany, zastępuje kod przekazany do makra. Wszystko to dzieje się podczas
kompilacji.

Aby zdefiniować makro, używasz konstrukcji `macro_rules!`. Przyjrzyjmy się, jak
używać `macro_rules!`, przyglądając się definicji makra `vec!`. Rozdział 8
opisał, jak możemy użyć makra `vec!`, aby utworzyć nowy wektor o określonych
wartościach. Na przykład następująca makroinstrukcja tworzy nowy wektor zawierający trzy
liczby całkowite:

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

Możemy również użyć makra `vec!`, aby utworzyć wektor dwóch liczb całkowitych lub wektor
pięciu wycinków ciągu. Nie moglibyśmy użyć funkcji, aby zrobić to samo,
ponieważ nie znalibyśmy liczby ani typu wartości z góry.

Listing 20-28 pokazuje nieco uproszczoną definicję makra `vec!`.

<Listing number="20-28" file-name="src/lib.rs" caption="A simplified version of the `vec!` macro definition">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-28/src/lib.rs}}
```

</Listing>

> Uwaga: Rzeczywista definicja makra `vec!` w bibliotece standardowej
> zawiera kod do wstępnego przydzielenia prawidłowej ilości pamięci z góry. Ten kod
> to optymalizacja, której nie uwzględniamy tutaj, aby uprościć przykład.

Adnotacja `#[macro_export]` wskazuje, że ta makro powinna być dostępna,
gdy tylko skrzynia, w której zdefiniowano makro, zostanie wprowadzona do
zakresu. Bez tej adnotacji makro nie może zostać wprowadzona do zakresu.

Następnie rozpoczynamy definicję makra od `macro_rules!` i nazwy
makra, które definiujemy *bez* wykrzyknika. Po nazwie, w tym przypadku
`vec`, następują nawiasy klamrowe oznaczające treść definicji makra.

Struktura w treści `vec!` jest podobna do struktury wyrażenia `match`. Tutaj mamy jedno ramię ze wzorem `( $( $x:expr ),* )`,
następnie `=>` i blok kodu skojarzony z tym wzorem. Jeśli
wzorzec pasuje, skojarzony blok kodu zostanie wyemitowany. Biorąc pod uwagę, że jest to
jedyny wzór w tej makrze, istnieje tylko jeden prawidłowy sposób dopasowania; każdy
inny wzór spowoduje błąd. Bardziej złożone makra będą miały więcej niż
jedno ramię.

Prawidłowa składnia wzorca w definicjach makr różni się od składni wzorca
opisanej w rozdziale 19, ponieważ wzorce makr są dopasowywane do struktury kodu Rust,
a nie do wartości. Przeanalizujmy, co oznaczają elementy wzorca w
Listingu 20-28; aby uzyskać pełną składnię wzorca makra, zobacz [Rust
Reference][ref].

Najpierw używamy zestawu nawiasów, aby objąć cały wzór. Używamy
znaku dolara (`$`), aby zadeklarować zmienną w systemie makr, która będzie zawierać
kod Rust pasujący do wzoru. Znak dolara jasno wskazuje, że jest to
makro zmienna, a nie zwykła zmienna Rust. Następnie pojawia się zestaw
nawiasów, który przechwytuje wartości pasujące do wzorca w nawiasach
do wykorzystania w kodzie zastępczym. W `$()` znajduje się `$x:expr`, który pasuje do dowolnego
wyrażenia Rust i nadaje wyrażeniu nazwę `$x`.

Przecinek następujący po `$()` wskazuje, że znak separatora literowego przecinka
może opcjonalnie pojawić się po kodzie pasującym do kodu w `$()`. `*`
określa, że ​​wzorzec pasuje do zera lub więcej elementów poprzedzających `*`.

Gdy wywołujemy tę makro z `vec![1, 2, 3];`, wzorzec `$x` pasuje trzy
razy do trzech wyrażeń `1`, `2` i `3`.

Teraz przyjrzyjmy się wzorowi w treści kodu powiązanego z tym ramieniem:
`temp_vec.push()` w `$()*` jest generowany dla każdej części, która pasuje do `$()`
we wzorcu zero lub więcej razy, w zależności od tego, ile razy wzorzec pasuje. `$x` jest zastępowane każdym dopasowanym wyrażeniem. Kiedy wywołamy to
makro za pomocą `vec![1, 2, 3];`, wygenerowany kod, który zastępuje to wywołanie makra,
będzie następujący:

```rust,ignore
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

Zdefiniowaliśmy makro, które może przyjmować dowolną liczbę argumentów dowolnego typu i może
generować kod w celu utworzenia wektora zawierającego określone elementy.

Aby dowiedzieć się więcej o tym, jak pisać makra, zapoznaj się z dokumentacją online lub
innymi zasobami, takimi jak [„Mała księga Rust Makra”][tlborm] rozpoczęty przez
Daniela Keepa i kontynuowany przez Lukasa Wirtha.
### Proceduralna Makra do generowania kodu z atrybutów

Drugą formą makr jest *makro proceduralne*, które działa bardziej jak
funkcja (i jest typem procedury). Makra proceduralne akceptują pewien kod jako
dane wejściowe, działają na tym kodzie i generują pewien kod jako dane wyjściowe, zamiast
dopasowywać go do wzorców i zastępować kod innym kodem, jak to robią
makra deklaratywne. Trzy rodzaje makr proceduralnych to niestandardowe pochodne,
podobne do atrybutów i podobne do funkcji, i wszystkie działają w podobny sposób.

Podczas tworzenia makr proceduralnych definicje muszą znajdować się we własnej skrzyni
ze specjalnym typem skrzyni. Wynika to ze złożonych przyczyn technicznych, które mamy nadzieję
wyeliminować w przyszłości. W Liście 20-29 pokazujemy, jak zdefiniować
makro proceduralne, gdzie `some_attribute` jest symbolem zastępczym do użycia określonej
odmiany makra.

<Listing number="20-29" file-name="src/lib.rs" caption="An example of defining a procedural macro">

```rust,ignore
use proc_macro;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

</Listing>

Funkcja definiująca makro proceduralne przyjmuje `TokenStream` jako dane wejściowe
i generuje `TokenStream` jako dane wyjściowe. Typ `TokenStream` jest definiowany przez
skrzynię `proc_macro`, która jest dołączona do Rust i reprezentuje sekwencję
tokenów. To jest rdzeń makra: kod źródłowy, na którym działa makro,
tworzy dane wejściowe `TokenStream`, a kod generowany przez makro jest danymi wyjściowymi `TokenStream`. Funkcja ma również dołączony atrybut,
który określa, jaki rodzaj makra proceduralnego tworzymy. Możemy mieć
wiele rodzajów makr proceduralnych w tej samej skrzyni.

Przyjrzyjmy się różnym rodzajom makr proceduralnych. Zaczniemy od
niestandardowego makra pochodnego, a następnie wyjaśnimy niewielkie różnice, które sprawiają, że
inne formy są różne.

### Jak napisać niestandardową makroinstrukcję `deriv`

Utwórzmy skrzynię o nazwie `hello_macro`, która definiuje cechę o nazwie
`HelloMacro` z jedną skojarzoną funkcją o nazwie `hello_macro`. Zamiast
musieć naszych użytkowników implementować cechę `HelloMacro` dla każdego z ich typów,
udostępnimy makro proceduralne, aby użytkownicy mogli adnotować swój typ za pomocą
`#[derive(HelloMacro)]`, aby uzyskać domyślną implementację funkcji `hello_macro`
. Domyślna implementacja wyświetli `Hello, Macro! My name is
TypeName!`, gdzie `TypeName` jest nazwą typu, w którym ta cecha
została zdefiniowana. Innymi słowy, napiszemy skrzynię, która umożliwi innemu
programiście pisanie kodu takiego jak Listing 20-30 przy użyciu naszej skrzyni.

<Listing number="20-30" file-name="src/main.rs" caption="The code a user of our crate will be able to write when using our procedural macro">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-30/src/main.rs}}
```

</Listing>

Ten kod wydrukuje `Hello, Macro! My name is Pancakes!`, gdy skończymy. Pierwszym krokiem jest utworzenie nowej skrzyni bibliotecznej, takiej jak ta:

```console
$ cargo new hello_macro --lib
```

Następnie zdefiniujemy cechę `HelloMacro` i powiązaną z nią funkcję:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-20-impl-hellomacro-for-pancakes/hello_macro/src/lib.rs}}
```

</Listing>

Mamy cechę i jej funkcję. W tym momencie nasz użytkownik skrzynki mógłby zaimplementować cechę, aby osiągnąć pożądaną funkcjonalność, tak jak tutaj:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-20-impl-hellomacro-for-pancakes/pancakes/src/main.rs}}
```

Musieliby jednak napisać blok implementacji dla każdego typu, którego
chcieliby użyć z `hello_macro`; chcemy oszczędzić im konieczności wykonywania tej
pracy.

Ponadto nie możemy jeszcze zapewnić funkcji `hello_macro` z domyślną
implementacją, która wydrukuje nazwę typu, w którym zaimplementowano cechę: Rust nie ma możliwości odbicia, więc nie może wyszukać nazwy typu w czasie wykonywania. Potrzebujemy makra do generowania kodu w czasie kompilacji.

Następnym krokiem jest zdefiniowanie makra proceduralnego. W momencie pisania tego tekstu
makra proceduralne muszą znajdować się we własnej skrzyni. Ostatecznie to ograniczenie
może zostać zniesione. Konwencja dotycząca strukturyzacji skrzynek i skrzynek makr jest następująca: dla skrzyni o nazwie `foo` niestandardowa skrzynia makr proceduralnych derive jest nazywana `foo_derive`. Utwórzmy nową skrzynkę o nazwie `hello_macro_derive` wewnątrz naszego projektu `hello_macro`:

```console
$ cargo new hello_macro_derive --lib
```

Nasze dwie skrzynie są ściśle powiązane, więc tworzymy procedurę makro w katalogu naszej skrzyni `hello_macro`. Jeśli zmienimy definicję
cechy w `hello_macro`, będziemy musieli zmienić implementację
proceduralnej makra w `hello_macro_derive`. Dwie skrzynie będą musiały
zostać opublikowane oddzielnie, a programiści używający tych skrzyń będą musieli dodać
obie jako zależności i włączyć je do zakresu. Zamiast tego moglibyśmy sprawić, aby skrzynia
`hello_macro` używała `hello_macro_derive` jako zależności i ponownie wyeksportowała
kod procedury makro. Jednak sposób, w jaki ustrukturyzowaliśmy projekt, umożliwia programistom używanie `hello_macro`, nawet jeśli nie chcą funkcjonalności
`derive`.

Musimy zadeklarować skrzynię `hello_macro_derive` jako procedurę makro.
Będziemy również potrzebować funkcjonalności ze skrzynek `syn` i `quote`, jak zobaczysz za chwilę, więc musimy dodać je jako zależności. Dodaj poniższe do pliku
*Cargo.toml* dla `hello_macro_derive`:

<Listing file-name="hello_macro_derive/Cargo.toml">

```toml
{{#include ../listings/ch20-advanced-features/listing-20-31/hello_macro/hello_macro_derive/Cargo.toml:6:12}}
```

</Listing>

Aby rozpocząć definiowanie makra proceduralnego, umieść kod z Listingu 20-31 w pliku
*src/lib.rs* dla pakietu `hello_macro_derive`. Należy zauważyć, że ten kod
nie skompiluje się, dopóki nie dodamy definicji dla funkcji `impl_hello_macro`.

<Listing number="20-31" file-name="hello_macro_derive/src/lib.rs" caption="Code that most procedural macro crates will require in order to process Rust code">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-31/hello_macro/hello_macro_derive/src/lib.rs}}
```

</Listing>

Zauważ, że podzieliliśmy kod na funkcję `hello_macro_derive`, która
odpowiada za parsowanie `TokenStream`, i funkcję `impl_hello_macro`, która
odpowiada za transformację drzewa składni: dzięki temu
pisanie proceduralnej makra jest wygodniejsze. Kod w zewnętrznej funkcji
(w tym przypadku `hello_macro_derive`) będzie taki sam dla niemal każdej
proceduralnej skrzyni makra, którą zobaczysz lub utworzysz. Kod, który określisz w treści
wewnętrznej funkcji (w tym przypadku `impl_hello_macro`), będzie różny
w zależności od celu Twojej proceduralnej makra.

Wprowadziliśmy trzy nowe skrzynie: `proc_macro`, [`syn`] i [`quote`]. Skrzynia
`proc_macro` jest dostarczana z Rustem, więc nie musieliśmy jej dodawać do
zależności w *Cargo.toml*. Skrzynia `proc_macro` to API kompilatora, które
pozwala nam odczytywać i manipulować kodem Rust z naszego kodu.

Skrzynia `syn` analizuje kod Rust z ciągu do struktury danych, na której
możemy wykonywać operacje. Skrzynia `quote` zamienia struktury danych `syn` z powrotem
na kod Rust. Te skrzynie znacznie ułatwiają analizowanie dowolnego rodzaju kodu Rust,
który możemy chcieć obsłużyć: napisanie pełnego parsera dla kodu Rust nie jest prostym
zadaniem.

Funkcja `hello_macro_derive` zostanie wywołana, gdy użytkownik naszej biblioteki
określi `#[derive(HelloMacro)]` w typie. Jest to możliwe, ponieważ
oznaczyliśmy tutaj funkcję `hello_macro_derive` adnotacją `proc_macro_derive` i
określiliśmy nazwę `HelloMacro`, która odpowiada naszej nazwie cechy; jest to konwencja, której
przestrzega większość makr proceduralnych.

Funkcja `hello_macro_derive` najpierw konwertuje `input` z
`TokenStream` na strukturę danych, którą możemy następnie interpretować i wykonywać na niej operacje. W tym miejscu wkracza `syn`. Funkcja `parse` w
`syn` przyjmuje `TokenStream` i zwraca strukturę `DeriveInput` reprezentującą
przetworzony kod Rust. Listing 20-32 pokazuje odpowiednie części struktury `DeriveInput`, które otrzymujemy z parsowania ciągu `struct Pancakes;`:

<Listing number="20-32" caption="Instancja `DeriveInput`, którą otrzymujemy podczas parsowania kodu zawierającego atrybut makra z oferty 20-30">

```rust,ignore
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

</Listing>

Pola tej struktury pokazują, że przeanalizowany przez nas kod Rust jest strukturą jednostkową
o `ident` (identyfikator, oznaczający nazwę) `Pancakes`. W tej strukturze jest więcej
pól opisujących wszelkiego rodzaju kod Rust; sprawdź [`syn`
dokumentację `DeriveInput`][syn-docs], aby uzyskać więcej informacji.

Wkrótce zdefiniujemy funkcję `impl_hello_macro`, w której zbudujemy
nowy kod Rust, który chcemy uwzględnić. Ale zanim to zrobimy, zauważ, że wyjście
naszego makra derive to również `TokenStream`. Zwrócony `TokenStream` jest
dodawany do kodu, który piszą nasi użytkownicy skrzyni, więc gdy skompilują swoją skrzynię,
otrzymają dodatkową funkcjonalność, którą zapewniamy w zmodyfikowanym
`TokenStream`.

Być może zauważyłeś, że wywołujemy `unwrap`, aby spowodować panikę funkcji
`hello_macro_derive`, jeśli wywołanie funkcji `syn::parse`
tutaj się nie powiedzie. Konieczne jest, aby nasza makro proceduralne panikowała w przypadku błędów, ponieważ
funkcje `proc_macro_derive` muszą zwracać `TokenStream`, a nie `Result`, aby
być zgodne z proceduralnym API makr. Uprościliśmy ten przykład, używając
`unwrap`; w kodzie produkcyjnym powinieneś podawać bardziej szczegółowe komunikaty o błędach,
o tym, co poszło nie tak, używając `panic!` lub `expect`.

Teraz, gdy mamy kod do przekształcenia adnotowanego kodu Rust z `TokenStream`
w instancję `DeriveInput`, wygenerujmy kod, który implementuje cechę
`HelloMacro` w adnotowanym typie, jak pokazano w Liście 20-33.

<Listing number="20-33" file-name="hello_macro_derive/src/lib.rs" caption="Implementing the `HelloMacro` trait using the parsed Rust code">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-33/hello_macro/hello_macro_derive/src/lib.rs:here}}
```

</Listing>

Otrzymujemy instancję struktury `Ident` zawierającą nazwę (identyfikator) typu adnotowanego przy użyciu `ast.ident`. Struktura w Liście 20-32 pokazuje, że gdy
uruchomimy funkcję `impl_hello_macro` w kodzie w Liście 20-30,
`ident`, który otrzymamy, będzie miał pole `ident` o wartości `"Pancakes"`. Tak więc,
zmienna `name` w Liście 20-33 będzie zawierać instancję struktury `Ident`,
która po wydrukowaniu będzie ciągiem `"Pancakes"`, nazwą struktury w
Liście 20-30.

Makro `quote!` pozwala nam zdefiniować kod Rust, który chcemy zwrócić.
Kompilator oczekuje czegoś innego niż bezpośredni wynik wykonania makra `quote!`, więc musimy przekonwertować go na `TokenStream`. Robimy to,
wywołując metodę `into`, która zużywa tę pośrednią reprezentację i
zwraca wartość wymaganego typu `TokenStream`.

Makro `quote!` zapewnia również kilka bardzo fajnych mechanizmów szablonowania: możemy
wpisać `#name`, a `quote!` zastąpi je wartością w zmiennej
`name`. Możesz nawet wykonać pewne powtórzenia podobne do sposobu działania zwykłych makr.
Sprawdź [dokumentację skrzynki `quote`][quote-docs], aby uzyskać dokładne wprowadzenie.

Chcemy, aby nasze makro proceduralne wygenerowało implementację naszej cechy `HelloMacro`
dla typu adnotowanego przez użytkownika, którą możemy uzyskać, używając `#name`. Implementacja cechy ma jedną funkcję `hello_macro`, której ciało zawiera
funkcjonalność, którą chcemy zapewnić: wyświetlanie `Hello, Macro! My name is`, a następnie
nazwy adnotowanego typu.

Makro `stringify!` użyte tutaj jest wbudowane w Rust. Przyjmuje wyrażenie Rust, takie jak `1 + 2`, i w czasie kompilacji zamienia je na
dosłowny ciąg, taki jak `"1 + 2"`. Jest to inne niż `format!` lub
`println!`, makra, które oceniają wyrażenie, a następnie zamieniają wynik na
`String`. Istnieje możliwość, że dane wejściowe `#name` mogą być
dosłownym wyrażeniem, więc używamy `stringify!`. Użycie `stringify!` również
zapisuje alokację poprzez konwersję `#name` na dosłowny ciąg w czasie kompilacji.

W tym momencie `cargo build` powinno zakończyć się pomyślnie zarówno w `hello_macro`
jak i `hello_macro_derive`. Podłączmy te skrzynie do kodu w Listingu
20-30, aby zobaczyć makro proceduralne w akcji! Utwórz nowy projekt binarny w
katalogu *projects*, używając `cargo new pancakes`. Musimy dodać
`hello_macro` i `hello_macro_derive` jako zależności w pliku Cargo.toml* `pancakes`
crate. Jeśli publikujesz swoje wersje `hello_macro` i
`hello_macro_derive` w [crates.io](https://crates.io/), będą to zwykłe
zależności; jeśli nie, możesz je określić jako zależności `path` w następujący sposób:

```toml
{{#include ../listings/ch20-advanced-features/no-listing-21-pancakes/pancakes/Cargo.toml:7:9}}
```

Umieść kod z Listingu 20-30 w *src/main.rs* i uruchom `cargo run`:
powinien on wydrukować `Hello, Macro! Nazywam się Pancakes!` Implementacja cechy
`HelloMacro` z makra proceduralnego została uwzględniona bez konieczności implementacji jej przez skrzynkę
`pancakes`; `#[derive(HelloMacro)]` dodał implementację cechy.

Następnie przyjrzyjmy się, w jaki sposób inne rodzaje makr proceduralnych różnią się od niestandardowych
makr pochodnych.

### Makra podobne do atrybutów

Makra podobne do atrybutów są podobne do niestandardowych makr pochodnych, ale zamiast
generować kod dla atrybutu `derive`, pozwalają na tworzenie nowych
atrybutów. Są również bardziej elastyczne: `derive` działa tylko dla struktur i
wyliczeń; atrybuty można stosować również do innych elementów, takich jak funkcje.
Oto przykład użycia makra podobnego do atrybutu: powiedzmy, że masz atrybut
o nazwie `route`, który adnotuje funkcje podczas korzystania z frameworka aplikacji internetowej:

```rust,ignore
#[route(GET, "/")]
fn index() {
```

Ten atrybut `#[route]` byłby zdefiniowany przez framework jako makro proceduralne. Sygnatura funkcji definicji makra wyglądałaby następująco:

```rust,ignore
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

Tutaj mamy dwa parametry typu `TokenStream`. Pierwszy jest dla
zawartości atrybutu: część `GET, "/"`. Drugi jest treścią
elementu, do którego atrybut jest dołączony: w tym przypadku `fn index() {}` i resztą
treści funkcji.

Poza tym makra podobne do atrybutów działają tak samo jak niestandardowe makra pochodne: tworzysz skrzynkę z typem skrzynki `proc-macro` i implementujesz
funkcję, która generuje żądany kod!

### Makra podobne do funkcji

Makra podobne do funkcji definiują makra, które wyglądają jak wywołania funkcji. Podobnie jak makra
`macro_rules!`, są bardziej elastyczne niż funkcje; na przykład
mogą przyjmować nieznaną liczbę argumentów. Jednak makra `macro_rules!` można
zdefiniować tylko przy użyciu składni podobnej do dopasowania, którą omówiliśmy w sekcji
[„Makra deklaratywne z `macro_rules!` dla ogólnego
metaprogramowania”][decl]<!-- ignore --> wcześniej. Makra podobne do funkcji przyjmują parametr
`TokenStream`, a ich definicja manipuluje tym `TokenStream`
przy użyciu kodu Rust, tak jak robią to dwa inne typy makr proceduralnych. Przykładem
makra podobnego do funkcji jest makro `sql!`, które można wywołać w następujący sposób:

```rust,ignore
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

Ta makroinstrukcja analizowałaby polecenie SQL w środku i sprawdzała, czy jest ono poprawne składniowo, co jest znacznie bardziej złożonym przetwarzaniem niż to, co może zrobić makroinstrukcja
`macro_rules!`. Makro `sql!` byłoby zdefiniowane w następujący sposób:

```rust,ignore
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

Ta definicja jest podobna do podpisu niestandardowego makra derive: otrzymujemy
tokeny znajdujące się w nawiasach i zwracamy kod, który chcieliśmy
wygenerować.

## Streszczenie

Uff! Teraz masz w swoim zestawie narzędzi kilka funkcji Rust, których prawdopodobnie nie będziesz używać
często, ale będziesz wiedział, że są dostępne w bardzo szczególnych okolicznościach.
Wprowadziliśmy kilka złożonych tematów, aby gdy natkniesz się na nie w
sugestiach komunikatów o błędach lub w kodzie innych osób, będziesz w stanie
rozpoznać te koncepcje i składnię. Użyj tego rozdziału jako odniesienia, które poprowadzi Cię
do rozwiązań.

Następnie wdrożymy w życie wszystko, co omówiliśmy w książce
i wykonamy jeszcze jeden projekt!

[ref]: ../reference/macros-by-example.html
[tlborm]: https://veykril.github.io/tlborm/
[`syn`]: https://crates.io/crates/syn
[`quote`]: https://crates.io/crates/quote
[syn-docs]: https://docs.rs/syn/2.0/syn/struct.DeriveInput.html
[quote-docs]: https://docs.rs/quote
[decl]: #declarative-macros-with-macro_rules-for-general-metaprogramming
