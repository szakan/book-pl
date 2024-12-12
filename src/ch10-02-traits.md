## Cechy: definiowanie współdzielonego zachowania

*Cecha* definiuje funkcjonalność, którą dany typ ma i może dzielić z
innymi typami. Możemy użyć cech, aby zdefiniować wspólne zachowanie w sposób abstrakcyjny. Możemy
używać *granic cech*, aby określić, że typ ogólny może być dowolnym typem, który ma
określone zachowanie.

> Uwaga: Cechy są podobne do funkcji często nazywanej *interfejsami* w innych
> językach, chociaż z pewnymi różnicami.

### Definiowanie cechy

Zachowanie typu składa się z metod, które możemy wywołać w tym typie. Różne
typy mają takie samo zachowanie, jeśli możemy wywołać te same metody we wszystkich tych
typach. Definicje cech to sposób grupowania sygnatur metod w celu
zdefiniowania zestawu zachowań niezbędnych do osiągnięcia pewnego celu.

Na przykład powiedzmy, że mamy wiele struktur, które przechowują różne rodzaje i
ilości tekstu: strukturę `NewsArticle`, która przechowuje artykuł w określonej lokalizacji i `Tweet`, który może mieć maksymalnie 280 znaków wraz z
metadanymi wskazującymi, czy był to nowy tweet, retweet czy odpowiedź
na innego tweeta.

Chcemy utworzyć bibliotekę agregatora mediów o nazwie `aggregator`, która może
wyświetlać podsumowania danych, które mogą być przechowywane w instancji `NewsArticle` lub `Tweet`. Aby to zrobić, potrzebujemy podsumowania z każdego typu i poprosimy o to
summary, wywołując metodę `summarize` na instancji. Listing 10-12 pokazuje
definicję publicznej cechy `Summary`, która wyraża to zachowanie.

<Listing number="10-12" file-name="src/lib.rs" caption="A `Summary` trait that consists of the behavior provided by a `summarize` method">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-12/src/lib.rs}}
```

</Listing>

Tutaj deklarujemy cechę za pomocą słowa kluczowego `trait`, a następnie nazwy cechy,
która w tym przypadku jest `Summary`. Deklarujemy również cechę jako `pub`, aby
skrzynie zależne od tej skrzyni mogły również korzystać z tej cechy, jak zobaczymy w
kilku przykładach. W nawiasach klamrowych deklarujemy sygnatury metod,
które opisują zachowania typów implementujących tę cechę, w
tym przypadku jest to `fn summary(&self) -> String`.

Po sygnaturze metody zamiast podawać implementację w nawiasach klamrowych, używamy średnika. Każdy typ implementujący tę cechę musi zapewnić
swoje własne niestandardowe zachowanie dla treści metody. Kompilator wymusi,
że każdy typ, który ma cechę `Summary`, będzie miał metodę `summarize`
zdefiniowaną dokładnie z tą sygnaturą.

Cecha może mieć wiele metod w swojej treści: sygnatury metod są wymienione
po jednej w każdym wierszu, a każdy wiersz kończy się średnikiem.

### Implementacja cechy w typie

Teraz, gdy zdefiniowaliśmy pożądane sygnatury metod cechy `Summary`,
możemy zaimplementować je w typach w naszym agregatorze mediów. Listing 10-13 pokazuje
implementację cechy `Summary` w strukturze `NewsArticle`, która używa
nagłówka, autora i lokalizacji do utworzenia wartości zwracanej
`summarize`. W przypadku struktury `Tweet` definiujemy `summarize` jako nazwę użytkownika,
po której następuje cały tekst tweeta, zakładając, że treść tweeta jest
już ograniczona do 280 znaków.

<Listing number="10-13" file-name="src/lib.rs" caption="Implementing the `Summary` trait on the `NewsArticle` and `Tweet` types">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-13/src/lib.rs:here}}
```

</Listing>

Implementacja cechy w typie jest podobna do implementacji zwykłych metod. Różnica polega na tym, że po `impl` wstawiamy nazwę cechy, którą chcemy zaimplementować,
następnie używamy słowa kluczowego `for`, a następnie określamy nazwę typu, dla którego chcemy zaimplementować cechę. W bloku `impl` wstawiamy sygnatury metod,
które zostały zdefiniowane w definicji cechy. Zamiast dodawać średnik po każdym
sygnaturze, używamy nawiasów klamrowych i wypełniamy treść metody określonym
zachowaniem, jakie mają mieć metody cechy dla danego typu.

Teraz, gdy biblioteka zaimplementowała cechę `Summary` w `NewsArticle` i
`Tweet`, użytkownicy skrzyni mogą wywoływać metody cech w wystąpieniach
`NewsArticle` i `Tweet` w taki sam sposób, w jaki wywołujemy zwykłe metody. Jedyną
różnicą jest to, że użytkownik musi wprowadzić cechę do zakresu, a także
typy. Oto przykład, w jaki sposób skrzynia binarna mogłaby wykorzystać naszą skrzynię biblioteczną `aggregator`:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-01-calling-trait-method/src/main.rs}}
```

Ten kod drukuje `1 nowy tweet: horse_ebooks: oczywiście, jak prawdopodobnie już
wiesz, ludzie`.

Inne skrzynie, które zależą od skrzyni `aggregator`, mogą również wprowadzić cechę `Summary` do zakresu, aby zaimplementować `Summary` w swoich własnych typach. Jednym z ograniczeń, na które należy zwrócić uwagę, jest to, że możemy zaimplementować cechę w typie tylko wtedy, gdy cecha lub
typ, lub oba, są lokalne dla naszej skrzyni. Na przykład możemy zaimplementować standardowe cechy
biblioteki, takie jak `Display` w niestandardowym typie, takim jak `Tweet`, jako część funkcjonalności naszej skrzyni
`aggregator`, ponieważ typ `Tweet` jest lokalny dla naszej skrzyni
`aggregator`. Możemy również zaimplementować `Summary` w `Vec<T>` w naszej skrzyni
`aggregator`, ponieważ cecha `Summary` jest lokalna dla naszej skrzyni
`aggregator`.

Nie możemy jednak zaimplementować cech zewnętrznych w typach zewnętrznych. Na przykład nie możemy
zaimplementować cechy `Display` w `Vec<T>` w naszym `aggregator` crate, ponieważ
`Display` i `Vec<T>` są zdefiniowane w bibliotece standardowej i nie są
lokalne dla naszego `aggregator` crate. To ograniczenie jest częścią właściwości o nazwie
*coherence*, a dokładniej *orphan rule*, nazwanej tak, ponieważ
typ nadrzędny nie jest obecny. Ta reguła zapewnia, że ​​kod innych osób nie może
zniszczyć Twojego kodu i odwrotnie. Bez tej reguły dwie skrzynie mogłyby zaimplementować
tę samą cechę dla tego samego typu, a Rust nie wiedziałby, której implementacji
użyć.

### Domyślne implementacje

Czasami przydatne jest posiadanie domyślnego zachowania dla niektórych lub wszystkich metod
w cesze zamiast wymagania implementacji dla wszystkich metod w każdym typie.
Następnie, gdy implementujemy cechę w określonym typie, możemy zachować lub zastąpić domyślne zachowanie
każdej metody.

W Liście 10-14 określamy domyślny ciąg dla metody `summarize` cechy
``Summary` zamiast definiować tylko sygnaturę metody, jak zrobiliśmy w Liście 10-12.

<Listing number="10-14" file-name="src/lib.rs" caption="Defining a `Summary` trait with a default implementation of the `summarize` method">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-14/src/lib.rs:here}}
```

</Listing>

Aby użyć domyślnej implementacji do podsumowania wystąpień `NewsArticle`,
określamy pusty blok `impl` za pomocą `impl Summary for NewsArticle {}`.

Chociaż nie definiujemy już metody `summarize` w `NewsArticle`
bezpośrednio, dostarczyliśmy domyślną implementację i określiliśmy, że
`NewsArticle` implementuje cechę `Summary`. W rezultacie nadal możemy wywołać
metodę `summarize` w wystąpieniu `NewsArticle`, w następujący sposób:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-02-calling-default-impl/src/main.rs:here}}
```

Ten kod drukuje `Dostępny nowy artykuł! (Czytaj więcej...)`.

Utworzenie domyślnej implementacji nie wymaga od nas zmiany czegokolwiek w
implementacji `Podsumowania` w `Tweet` w Liście 10-13. Powodem jest to, że
składnia nadpisywania domyślnej implementacji jest taka sama jak składnia
implementacji metody cechy, która nie ma domyślnej implementacji.

Domyślne implementacje mogą wywoływać inne metody w tej samej cesze, nawet jeśli te inne metody nie mają domyślnej implementacji. W ten sposób cecha może
zapewnić wiele przydatnych funkcji i wymagać od implementatorów określenia tylko jej niewielkiej części. Na przykład moglibyśmy zdefiniować cechę `Summary`, aby miała metodę
`summarize_author`, której implementacja jest wymagana, a następnie zdefiniować metodę
`summarize`, która ma domyślną implementację wywołującą metodę
`summarize_author`:

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:here}}
```

Aby użyć tej wersji `Summary`, musimy zdefiniować `summarize_author` tylko wtedy, gdy implementujemy cechę w typie:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:impl}}
```

Po zdefiniowaniu `summarize_author` możemy wywołać `summarize` na wystąpieniach struktury
`Tweet`, a domyślna implementacja `summarize` wywoła definicję `summarize_author`, którą podaliśmy. Ponieważ zaimplementowaliśmy
`summarize_author`, cecha `Summary` zapewniła nam zachowanie metody
`summarize` bez konieczności pisania dodatkowego kodu. Oto, jak to wygląda:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/main.rs:here}}
```

Ten kod drukuje `1 nowy tweet: (Przeczytaj więcej od @horse_ebooks...)`.

Należy pamiętać, że nie można wywołać domyślnej implementacji z
nadrzędnej implementacji tej samej metody.

### Cechy jako parametry

Teraz, gdy wiesz, jak definiować i implementować cechy, możemy zbadać, jak używać
cech do definiowania funkcji akceptujących wiele różnych typów. Użyjemy cechy
`Summary`, którą zaimplementowaliśmy w typach `NewsArticle` i `Tweet` w
Listingu 10-13, aby zdefiniować funkcję `notify`, która wywołuje metodę `summarize`
na swoim parametrze `item`, który jest pewnego typu implementującym cechę `Summary`. Aby to zrobić, używamy składni `impl Trait`, takiej jak ta:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-04-traits-as-parameters/src/lib.rs:here}}
```

Zamiast konkretnego typu dla parametru `item`, określamy słowo kluczowe `impl`
i nazwę cechy. Ten parametr akceptuje dowolny typ, który implementuje określoną cechę. W treści `notify` możemy wywołać dowolne metody na `item`
pochodzące z cechy `Summary`, takie jak `summarize`. Możemy wywołać `notify`
i przekazać dowolne wystąpienie `NewsArticle` lub `Tweet`. Kod, który wywołuje funkcję
z dowolnym innym typem, takim jak `String` lub `i32`, nie zostanie skompilowany,
ponieważ te typy nie implementują `Summary`.

<!-- Old headings. Do not remove or links may break. -->
<a id="fixing-the-largest-function-with-trait-bounds"></a>

#### Składnia związana z cechami

Składnia `impl Trait` działa w prostych przypadkach, ale w rzeczywistości jest składnią
sugar dla dłuższej formy znanej jako *trait bound*; wygląda ona następująco:

```rust,ignore
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

Ta dłuższa forma jest równoważna przykładowi z poprzedniej sekcji, ale jest
bardziej rozwlekła. Umieszczamy granice cech z deklaracją parametru typu generycznego
po dwukropku i w nawiasach kątowych.

Składnia `impl Trait` jest wygodna i zapewnia bardziej zwięzły kod w prostych
przypadkach, podczas gdy pełniejsza składnia granic cech może wyrażać większą złożoność w innych
przypadkach. Na przykład możemy mieć dwa parametry, które implementują `Summary`. Zrobienie tego za pomocą składni `impl Trait` wygląda następująco:

```rust,ignore
pub fn notification(item1: &impl Summary, item2: &impl Summary) {
```

Użycie `impl Trait` jest właściwe, jeśli chcemy, aby ta funkcja zezwalała na posiadanie różnych typów `item1` i
`item2` (pod warunkiem, że oba typy implementują `Summary`). Jeśli jednak chcemy wymusić, aby oba parametry miały ten sam typ, musimy użyć ograniczenia cechy, takiego jak to:

```rust,ignore
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

Typ ogólny `T` określony jako typ parametrów `item1` i `item2`
ogranicza funkcję tak, że konkretny typ wartości
przekazanej jako argument dla `item1` i `item2` musi być taki sam.

#### Określanie wielu granic cech za pomocą składni `+`

Możemy również określić więcej niż jedno ograniczenie cechy. Powiedzmy, że chcemy, aby `notify` używało
formatowania wyświetlania, jak również `summarize` w `item`: określamy w definicji `notify`, że `item` musi implementować zarówno `Display`, jak i `Summary`. Możemy to zrobić,
używając składni `+`:

```rust,ignore
pub fn notify(item: &(impl Summary + Display)) {
```

Składnia `+` jest również prawidłowa w przypadku ograniczeń cech w typach generycznych:

```rust,ignore
pub fn notify<T: Summary + Display>(item: &T) {
```

With the two trait bounds specified, the body of `notify` can call `summarize`
and use `{}` to format `item`.

#### Bardziej przejrzyste granice cech z klauzulami `where`

Używanie zbyt wielu ograniczeń cech ma swoje wady. Każdy rodzaj ma własne ograniczenia cech, więc funkcje z wieloma parametrami typu rodzajowego mogą zawierać wiele informacji o ograniczeniach cech między nazwą funkcji a listą jej parametrów, co utrudnia odczytanie sygnatury funkcji. Z tego powodu Rust ma alternatywną składnię do określania ograniczeń cech wewnątrz klauzuli `where` po sygnaturze funkcji. Zamiast więc pisać to:

```rust,ignore
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

we can use a `where` clause, like this:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-07-where-clause/src/lib.rs:here}}
```

Sygnatura tej funkcji jest mniej zaśmiecona: nazwa funkcji, lista parametrów i typ zwracany znajdują się blisko siebie, podobnie jak w przypadku funkcji bez wielu ograniczeń cech.

### Zwracanie typów implementujących cechy

Możemy również użyć składni `impl Trait` w pozycji return, aby zwrócić wartość pewnego typu, która implementuje cechę, jak pokazano tutaj:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-05-returning-impl-trait/src/lib.rs:here}}
```

Używając `impl Summary` jako typu zwracanego, określamy, że funkcja
`returns_summarizable` zwraca jakiś typ, który implementuje cechę `Summary`
bez podawania nazwy konkretnego typu. W tym przypadku `returns_summarizable`
zwraca `Tweet`, ale kod wywołujący tę funkcję nie musi o tym wiedzieć.

Możliwość określenia typu zwracanego tylko przez implementowaną cechę jest
szczególnie przydatna w kontekście zamknięć i iteratorów, które omawiamy w
rozdziale 13. Zamknięcia i iteratory tworzą typy, które zna tylko kompilator lub
typy, których określenie jest bardzo długie. Składnia `impl Trait` pozwala zwięźle
określić, że funkcja zwraca jakiś typ, który implementuje cechę `Iterator`
bez konieczności wypisywania bardzo długiego typu.

Możesz jednak użyć `impl Trait` tylko wtedy, gdy zwracasz pojedynczy typ. Na przykład ten kod, który zwraca `NewsArticle` lub `Tweet` z typem zwracanym określonym jako `impl Summary`, nie zadziała:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-06-impl-trait-returns-one-type/src/lib.rs:here}}
```

Zwracanie `NewsArticle` lub `Tweet` nie jest dozwolone ze względu na ograniczenia
dotyczące sposobu implementacji składni `impl Trait` w kompilatorze. Omówimy
jak napisać funkcję z takim zachowaniem w [„Używanie obiektów cech, które
pozwalają na wartości różnych
Types”][using-trait-objects-that-allow-for-values-of-different-types]<!--
ignore --> section of Chapter 18.

### Wykorzystanie ograniczeń cech do warunkowej implementacji metod

Używając cechy powiązanej z blokiem `impl`, który używa parametrów typu generycznego,
możemy warunkowo implementować metody dla typów, które implementują określone
cechy. Na przykład typ `Pair<T>` w Liście 10-15 zawsze implementuje funkcję
`new`, aby zwrócić nową instancję `Pair<T>` (przypomnij sobie z sekcji
[“Defining Methods”][methods]<!-- ignore --> rozdziału 5, że `Self`
jest aliasem typu dla typu bloku `impl`, który w tym przypadku jest
`Pair<T>`). Ale w następnym bloku `impl`, `Pair<T>` implementuje metodę
`cmp_display` tylko wtedy, gdy jego wewnętrzny typ `T` implementuje cechę `PartialOrd`,
która umożliwia porównanie *i* cechę `Display`, która umożliwia drukowanie.

<Listing number="10-15" file-name="src/lib.rs" caption="Conditionally implementing methods on a generic type depending on trait bounds">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-15/src/lib.rs}}
```

</Listing>

Możemy również warunkowo zaimplementować cechę dla dowolnego typu, który implementuje
inną cechę. Implementacje cechy dla dowolnego typu, który spełnia granice cechy, nazywane są *implementacjami ogólnymi* i są szeroko stosowane w standardowej bibliotece Rust. Na przykład standardowa biblioteka implementuje cechę
`ToString` dla dowolnego typu, który implementuje cechę `Display`. Blok `impl`
w standardowej bibliotece wygląda podobnie do tego kodu:

```rust,ignore
impl<T: Display> ToString for T {
    // --snip--
}
```

Ponieważ standardowa biblioteka ma tę ogólną implementację, możemy wywołać metodę
`to_string` zdefiniowaną przez cechę `ToString` na dowolnym typie, który implementuje cechę
`Display`. Na przykład możemy zamienić liczby całkowite na odpowiadające im wartości
`String` w ten sposób, ponieważ liczby całkowite implementują `Display`:

```rust
let s = 3.to_string();
```

Implementacje ogólne pojawiają się w dokumentacji cechy w sekcji
„Implementors”.

Cechy i granice cech pozwalają nam pisać kod, który używa parametrów typu generycznego, aby
zmniejszyć duplikację, ale także określić kompilatorowi, że chcemy, aby typ generyczny
miał określone zachowanie. Kompilator może następnie użyć informacji o granicach cechy,
aby sprawdzić, czy wszystkie typy konkretne używane w naszym kodzie zapewniają
poprawne zachowanie. W językach dynamicznie typowanych otrzymalibyśmy błąd w
czasie wykonania, jeśli wywołalibyśmy metodę dla typu, który jej nie zdefiniował. Ale
Rust przenosi te błędy do czasu kompilacji, więc jesteśmy zmuszeni naprawić problemy,
zanim nasz kod będzie mógł zostać uruchomiony. Ponadto nie musimy pisać kodu,
który sprawdza zachowanie w czasie wykonania, ponieważ sprawdziliśmy już w
czasie kompilacji. Robiąc to, poprawiamy wydajność bez konieczności rezygnacji z elastyczności
typów generycznych.

[using-trait-objects-that-allow-for-values-of-different-types]: ch18-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[methods]: ch05-03-method-syntax.html#defining-methods
