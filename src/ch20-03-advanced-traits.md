## Zaawansowane cechy

Po raz pierwszy omówiliśmy cechy w sekcji [„Cechy: Definiowanie współdzielonego
zachowania”][traits-defining-shared-behavior]<!-- ignore --> rozdziału
10, ale nie omawialiśmy bardziej zaawansowanych szczegółów. Teraz, gdy wiesz więcej o Rust, możemy przejść do szczegółów.

### Określanie typów zastępczych w definicjach cech z typami skojarzonymi

*Typy skojarzone* łączą symbol zastępczy typu z cechą, tak że definicje
metod cech mogą używać tych typów symboli zastępczych w swoich sygnaturach.
Implementator cechy określi konkretny typ, który ma być używany zamiast
typu symbolu zastępczego dla konkretnej implementacji. W ten sposób możemy zdefiniować
cechę, która używa niektórych typów, bez konieczności dokładnego poznania, jakie to typy,
aż do momentu zaimplementowania cechy.

Większość zaawansowanych funkcji w tym rozdziale opisaliśmy jako rzadko
potrzebne. Typy skojarzone znajdują się gdzieś pośrodku: są używane rzadziej
niż funkcje wyjaśnione w pozostałej części książki, ale częściej niż wiele
innych funkcji omówionych w tym rozdziale.

Jednym z przykładów cechy z typem skojarzonym jest cecha `Iterator`, którą
dostarcza biblioteka standardowa. Typ skojarzony nazywa się `Item` i zastępuje
typ wartości, nad którymi iteruje typ implementujący cechę `Iterator`. Definicja cechy `Iterator` jest taka, jak pokazano w Liście
20-12.

<Listing number="20-12" caption="The definition of the `Iterator` trait that has an associated type `Item`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-12/src/lib.rs}}
```

</Listing>

Typ `Item` jest symbolem zastępczym, a definicja metody `next` pokazuje, że
zwróci ona wartości typu `Option<Self::Item>`. Implementatorzy cechy
`Iterator` określą konkretny typ dla `Item`, a metoda `next`
zwróci `Option` zawierającą wartość tego konkretnego typu.

Typy skojarzone mogą wydawać się koncepcją podobną do generyków, ponieważ
te ostatnie pozwalają nam zdefiniować funkcję bez określania, jakie typy może ona
obsługiwać. Aby zbadać różnicę między tymi dwoma koncepcjami, przyjrzymy się
implementacji cechy `Iterator` w typie o nazwie `Counter`, który określa, że
typem `Item` jest `u32`:

<Listing file-name="src/lib.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-22-iterator-on-counter/src/lib.rs:ch19}}
```

</Listing>

Ta składnia wydaje się porównywalna do składni generyków. Dlaczego więc nie zdefiniować po prostu cechy
`Iterator` z generykami, jak pokazano w Listingu 20-13?

<Listing number="20-13" number="A hypothetical definition of the `Iterator` trait using generics">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-13/src/lib.rs}}
```

</Listing>

Różnica polega na tym, że podczas korzystania z typów generycznych, jak w Liście 20-13, musimy
adnotować typy w każdej implementacji; ponieważ możemy również zaimplementować
`Iterator<String> dla Counter` lub dowolny inny typ, moglibyśmy mieć wiele
implementacji `Iterator` dla `Counter`. Innymi słowy, gdy cecha ma
parametr generyczny, może być implementowana dla typu wiele razy, zmieniając
za każdym razem konkretne typy parametrów typu generycznego. Gdy używamy metody
`next` dla `Counter`, musielibyśmy podać adnotacje typu, aby
wskazać, której implementacji `Iterator` chcemy użyć.

W przypadku typów skojarzonych nie musimy adnotować typów, ponieważ nie możemy
implementować cechy w typie wiele razy. W Liście 20-12 z
definicją, która używa typów skojarzonych, możemy wybrać typ
`Iter` tylko raz, ponieważ może być tylko jeden `implement Iterator dla Counter`.
Nie musimy określać, że chcemy iteratora wartości `u32` wszędzie,
który nazywamy `next` w `Counter`.

Typy skojarzone stają się również częścią kontraktu cechy: implementatorzy
cechy muszą dostarczyć typ, który zastąpi symbol zastępczy typu skojarzonego.
Typy skojarzone często mają nazwę opisującą sposób użycia typu,
a dokumentowanie typu skojarzonego w dokumentacji API jest dobrą praktyką.

### Domyślne parametry typu generycznego i przeciążanie operatora

Gdy używamy parametrów typu generycznego, możemy określić domyślny konkretny typ dla
typu generycznego. Eliminuje to potrzebę, aby implementatorzy cechy
określali konkretny typ, jeśli domyślny typ działa. Typ domyślny określasz
podczas deklarowania typu generycznego za pomocą składni `<PlaceholderType=ConcreteType>`.

Świetnym przykładem sytuacji, w której ta technika jest przydatna, jest *przeciążanie
operatora*, w którym dostosowujesz zachowanie operatora (takiego jak `+`)
w określonych sytuacjach.

Rust nie pozwala na tworzenie własnych operatorów ani przeciążanie dowolnych
operatorów. Możesz jednak przeciążać operacje i odpowiadające im cechy wymienione
w `std::ops`, implementując cechy skojarzone z operatorem. Na
przykład w Liście 20-14 przeciążamy operator `+`, aby dodać dwie instancje `Point`
razem. Robimy to, implementując cechę `Add` w strukturze `Point`:

<Listing number="20-14" file-name="src/main.rs" caption="Implementing the `Add` trait to overload the `+` operator for `Point` instances">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-14/src/main.rs}}
```

</Listing>

Metoda `add` dodaje wartości `x` dwóch instancji `Point` i wartości `y` dwóch instancji `Point`, aby utworzyć nową `Point`. Cecha `Add` ma
skojarzony typ o nazwie `Output`, który określa typ zwracany przez metodę `add`.

Domyślny typ generyczny w tym kodzie znajduje się w cesze `Add`. Oto jego
definicja:

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

Ten kod powinien wyglądać znajomo: cecha z jedną metodą i
skojarzonym typem. Nowa część to `Rhs=Self`: ta składnia nazywa się *domyślnymi
parametrami typu*. Ogólny parametr typu `Rhs` (skrót od „right hand
side”) definiuje typ parametru `rhs` w metodzie `add`. Jeśli nie określimy konkretnego typu dla `Rhs` podczas implementacji cechy `Add`, typ
`Rhs` będzie domyślnie `Self`, który będzie typem, w którym implementujemy
`Add`.

Kiedy implementowaliśmy `Add` dla `Point`, użyliśmy domyślnego dla `Rhs`, ponieważ
chcieliśmy dodać dwie instancje `Point`. Przyjrzyjmy się przykładowi implementacji
cechy `Add`, w której chcemy dostosować typ `Rhs` zamiast używać
domyślnego.

Mamy dwie struktury, `Milimetry` i `Metry`, zawierające wartości w różnych
jednostkach. To cienkie opakowanie istniejącego typu w innej strukturze jest znane jako
*wzorzec newtype*, który opisujemy bardziej szczegółowo w sekcji [„Używanie wzorca Newtype
do implementacji cech zewnętrznych w typach zewnętrznych”][newtype]<!-- ignore
-->. Chcemy dodać wartości w milimetrach do wartości w metrach i sprawić, aby
implementacja `Add` wykonała konwersję poprawnie. Możemy zaimplementować `Add`
dla `Milimetrów` z `Metrami` jako `Rhs`, jak pokazano w Liście 20-15.

<Listing number="20-15" file-name="src/lib.rs" caption="Implementing the `Add` trait on `Millimeters` to add `Millimeters` to `Meters`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-15/src/lib.rs}}
```

</Listing>

Aby dodać `Milimetry` i `Metry`, określamy `impl Add<Metry>`, aby ustawić
wartość parametru typu `Rhs` zamiast używać domyślnego `Self`.

Domyślnych parametrów typu użyjesz na dwa główne sposoby:

* Aby rozszerzyć typ bez łamania istniejącego kodu
* Aby umożliwić dostosowywanie w określonych przypadkach, których większość użytkowników nie będzie potrzebować

Standardowa cecha `Add` biblioteki jest przykładem drugiego celu:
zwykle dodajesz dwa podobne typy, ale cecha `Add` zapewnia możliwość
dostosowania poza tym. Użycie domyślnego parametru typu w definicji cechy `Add` oznacza, że ​​nie musisz określać dodatkowego parametru w większości
czasów. Innymi słowy, nie jest potrzebny jakiś szablon implementacji, co ułatwia
korzystanie z cechy.

Pierwszy cel jest podobny do drugiego, ale na odwrót: jeśli chcesz dodać
parametr typu do istniejącej cechy, możesz nadać mu wartość domyślną, aby umożliwić
rozszerzenie funkcjonalności cechy bez naruszania istniejącego
kodu implementacji.

### W pełni kwalifikowana składnia do ujednoznaczniania: wywoływanie metod o tej samej nazwie

Nic w Rust nie zabrania, aby cecha miała metodę o tej samej nazwie co
metoda innej cechy, ani Rust nie zabrania implementacji obu cech
w jednym typie. Możliwe jest również zaimplementowanie metody bezpośrednio w typie o
tej samej nazwie co metody z cech.

Podczas wywoływania metod o tej samej nazwie musisz powiedzieć Rust, której z nich
chcesz użyć. Rozważ kod w Listingu 20-16, w którym zdefiniowaliśmy dwie cechy,
`Pilot` i `Wizard`, które obie mają metodę o nazwie `fly`. Następnie implementujemy
obie cechy w typie `Human`, który ma już zaimplementowaną metodę o nazwie `fly`. Każda metoda `fly` robi coś innego.

<Listing number="20-16" file-name="src/main.rs" caption="Two traits are defined to have a ` method and are implemented on the `Human` type, and a `fly` method is implemented on `Human` directly">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-16/src/main.rs:here}}
```

</Listing>

Gdy wywołujemy `fly` na instancji `Human`, kompilator domyślnie wywołuje metodę, która jest bezpośrednio zaimplementowana w typie, jak pokazano na Liście 20-17.

<Listing number="20-17" file-name="src/main.rs" caption="Calling `fly` on an instance of `Human`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-17/src/main.rs:here}}
```

</Listing>

Uruchomienie tego kodu spowoduje wydrukowanie `*waving arms furiously*`, pokazując, że Rust
wywołał metodę `fly` zaimplementowaną w `Human` bezpośrednio.

Aby wywołać metody `fly` z cechy `Pilot` lub `Wizard`,
musimy użyć bardziej wyraźnej składni, aby określić, którą metodę `fly` mamy na myśli.
Listing 20-18 demonstruje tę składnię.

<Listing number="20-18" file-name="src/main.rs" caption="Specifying which trait’s `fly` method we want to call">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-18/src/main.rs:here}}
```

</Listing>

Określenie nazwy cechy przed nazwą metody wyjaśnia Rustowi, którą implementację `fly` chcemy wywołać. Możemy również napisać
`Human::fly(&person)`, co jest równoważne z `person.fly()`, którego użyliśmy
w Liście 20-18, ale jest to trochę dłuższe, jeśli nie musimy
ujednoznaczniać.

Uruchomienie tego kodu drukuje następujące:

```console
{{#include ../listings/ch20-advanced-features/listing-20-18/output.txt}}
```

Ponieważ metoda `fly` przyjmuje parametr `self`, gdybyśmy mieli dwa *typy*, które
oba implementują jedną *cechę*, Rust mógłby ustalić, której implementacji
cechy użyć na podstawie typu `self`.

Jednak powiązane funkcje, które nie są metodami, nie mają parametru `self`. Gdy istnieje wiele typów lub cech, które definiują funkcje niebędące metodami o tej samej nazwie, Rust nie zawsze wie, jaki typ masz na myśli,
chyba że użyjesz *w pełni kwalifikowanej składni*. Na przykład w Liście 20-19
tworzymy cechę dla schroniska dla zwierząt, które chce nadać wszystkim szczeniętom imię *Spot*.
Tworzymy cechę `Animal` ze skojarzoną funkcją niebędącą metodą `baby_name`.
Cecha `Animal` jest implementowana dla struktury `Dog`, w której również
bezpośrednio dostarczamy skojarzoną funkcję niebędącą metodą `baby_name`.

<Listing number="20-19" file-name="src/main.rs" caption="A trait with an associated function and a type with an associated function of the same name that also implements the trait">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-19/src/main.rs}}
```

</Listing>

Implementujemy kod do nadawania wszystkim szczeniętom imienia Spot w powiązanej
funkcji `baby_name`, która jest zdefiniowana w `Dog`. Typ `Dog` implementuje również cechę
`Animal`, która opisuje cechy, jakie mają wszystkie zwierzęta. Szczenięta są
nazywane szczeniakami i jest to wyrażone w implementacji cechy `Animal`
w `Dog` w funkcji `baby_name` powiązanej z cechą `Animal`.

W `main` wywołujemy funkcję `Dog::baby_name`, która wywołuje powiązaną
funkcję zdefiniowaną w `Dog` bezpośrednio. Ten kod drukuje następujące:

```console
{{#include ../listings/ch20-advanced-features/listing-20-19/output.txt}}
```

To nie jest to, czego chcieliśmy. Chcemy wywołać funkcję `baby_name`, która
jest częścią cechy `Animal`, którą zaimplementowaliśmy w `Dog`, aby kod wypisał
`Mały piesek nazywany jest szczeniakiem`. Technika określania nazwy cechy, której
użyliśmy w Liście 20-18, tutaj nie pomaga; jeśli zmienimy `main` na kod w
Liście 20-20, otrzymamy błąd kompilacji.

<Listing number="20-20" file-name="src/main.rs" caption="Attempting to call the `baby_name` function from the `Animal` trait, but Rust doesn’t know which implementation to use">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-20/src/main.rs:here}}
```

</Listing>

Because `Animal::baby_name` doesn’t have a `self` parameter, and there could be
other types that implement the `Animal` trait, Rust can’t figure out which
implementation of `Animal::baby_name` we want. We’ll get this compiler error:

```console
{{#include ../listings/ch20-advanced-features/listing-20-20/output.txt}}
```

Aby rozróżnić i powiedzieć Rust, że chcemy użyć implementacji
`Animal` dla `Dog` w przeciwieństwie do implementacji `Animal` dla jakiegoś innego
typu, musimy użyć w pełni kwalifikowanej składni. Listing 20-21 pokazuje, jak
używać w pełni kwalifikowanej składni.

<Listing number="20-21" file-name="src/main.rs" caption="Using fully qualified syntax to specify that we want to call the `baby_name` function from the `Animal` trait as implemented on `Dog`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-21/src/main.rs:here}}
```

</Listing>

Dostarczamy Rustowi adnotację typu w nawiasach kątowych, która
oznacza, że ​​chcemy wywołać metodę `baby_name` z cechy `Animal`, jak
zaimplementowano w `Dog`, mówiąc, że chcemy traktować typ `Dog` jako
`Animal` dla tego wywołania funkcji. Ten kod wydrukuje teraz to, czego chcemy:

```console
{{#include ../listings/ch20-advanced-features/listing-20-21/output.txt}}
```

In general, fully qualified syntax is defined as follows:

```rust,ignore
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

W przypadku powiązanych funkcji, które nie są metodami, nie byłoby `odbiorcy`:
byłaby tylko lista innych argumentów. Możesz używać w pełni kwalifikowanej
składni wszędzie tam, gdzie wywołujesz funkcje lub metody. Możesz jednak
pominąć dowolną część tej składni, którą Rust może wywnioskować z innych informacji
w programie. Musisz użyć tej bardziej rozwlekłej składni tylko w przypadkach, gdy
istnieje wiele implementacji, które używają tej samej nazwy, a Rust potrzebuje pomocy,
aby zidentyfikować implementację, którą chcesz wywołać.

### Używanie supercech w celu wymagania funkcjonalności jednej cechy w obrębie innej cechy

Czasami możesz napisać definicję cechy, która zależy od innej cechy:
aby typ zaimplementował pierwszą cechę, chcesz wymagać, aby ten typ również
zaimplementował drugą cechę. Powinieneś to zrobić, aby Twoja definicja cechy mogła
wykorzystać powiązane elementy drugiej cechy. Cecha, na której opiera się Twoja definicja cechy, jest nazywana *supercechą* Twojej cechy.

Na przykład, powiedzmy, że chcemy utworzyć cechę `OutlinePrint` za pomocą metody
`outline_print`, która wydrukuje daną wartość sformatowaną tak, aby była
obramowana gwiazdkami. Oznacza to, że biorąc pod uwagę strukturę `Point`, która implementuje
cechę biblioteki standardowej `Display`, aby uzyskać wynik `(x, y)`, gdy wywołamy
`outline_print` na instancji `Point`, która ma `1` dla `x` i `3` dla `y`, powinna
wydrukować następujące informacje:

```text
**********
*        *
* (1, 3) *
*        *
**********
```

W implementacji metody `outline_print` chcemy użyć funkcjonalności cechy
`Display`. Dlatego musimy określić, że cecha
`OutlinePrint` będzie działać tylko dla typów, które implementują również `Display` i
zapewniają funkcjonalność, której potrzebuje `OutlinePrint`. Możemy to zrobić w
definicji cechy, określając `OutlinePrint: Display`. Ta technika jest
podobna do dodawania cechy powiązanej z cechą. Listing 20-22 pokazuje
implementację cechy `OutlinePrint`.

<Listing number="20-22" file-name="src/main.rs" caption="Implementing the `OutlinePrint` trait that requires the functionality from `Display`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-22/src/main.rs:here}}
```

</Listing>

Ponieważ określiliśmy, że `OutlinePrint` wymaga cechy `Display`,
możemy użyć funkcji `to_string`, która jest automatycznie implementowana dla każdego typu,
który implementuje `Display`. Gdybyśmy próbowali użyć `to_string` bez dodawania
dwukropka i określenia cechy `Display` po nazwie cechy, otrzymalibyśmy
błąd informujący, że nie znaleziono żadnej metody o nazwie `to_string` dla typu `&Self` w
bieżącym zakresie.

Zobaczmy, co się stanie, gdy spróbujemy zaimplementować `OutlinePrint` w typie,
który nie implementuje `Display`, takim jak struktura `Point`:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/src/main.rs:here}}
```

</Listing>

Otrzymujemy błąd informujący, że `Display` jest wymagany, ale nie zaimplementowany:

```console
{{#include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/output.txt}}
```

Aby to naprawić, implementujemy `Display` w `Point` i spełniamy ograniczenie, którego wymaga
`OutlinePrint`, w następujący sposób:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-03-impl-display-for-point/src/main.rs:here}}
```

</Listing>

Następnie zaimplementowanie cechy `OutlinePrint` w `Point` skompiluje się pomyślnie i możemy wywołać `outline_print` na instancji `Point`, aby wyświetlić ją w obrysie gwiazdek.

### Wykorzystanie wzorca Newtype do implementacji cech zewnętrznych w typach zewnętrznych

W rozdziale 10 w sekcji [„Implementowanie cechy w
typie”][implementing-a-trait-on-a-type]<!-- ignore --> wspomnieliśmy o
regule sieroty, która mówi, że możemy implementować cechę w typie tylko wtedy, gdy
cecha lub typ są lokalne dla naszej skrzyni. Można
obejść to ograniczenie, używając wzorca *newtype*, który obejmuje utworzenie
nowego typu w strukturze krotki. (Struktury krotki omówiliśmy w sekcji [„Używanie struktur krotek bez nazwanych pól do tworzenia różnych typów”][struktury-krotek]<!--
ignore --> w rozdziale 5.) Struktura krotki będzie miała jedno pole i będzie
cienką otoczką wokół typu, dla którego chcemy zaimplementować cechę. Wtedy typ
otoczki jest lokalny dla naszej skrzyni i możemy zaimplementować cechę w otoczce.
*Newtype* to termin pochodzący z języka programowania Haskell.
Nie ma żadnego spadku wydajności w czasie wykonywania przy użyciu tego wzorca, a typ
opakowania jest pomijany w czasie kompilacji.

Na przykład, powiedzmy, że chcemy zaimplementować `Display` w `Vec<T>`, czego nie możemy zrobić bezpośrednio z powodu reguły
orphan, ponieważ cecha `Display` i typ
`Vec<T>` są zdefiniowane poza naszą skrzynią. Możemy utworzyć strukturę `Wrapper`, która
przechowuje instancję `Vec<T>`; następnie możemy zaimplementować `Display` w
`Wrapper` i użyć wartości `Vec<T>`, jak pokazano w Liście 20-23.
<Listing number="20-23" file-name="src/main.rs" caption="Creating a `Wrapper` type around `Vec<String>` to implement `Display`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-23/src/main.rs}}
```

</Listing>

Implementacja `Display` używa `self.0` do dostępu do wewnętrznego `Vec<T>`,
ponieważ `Wrapper` jest strukturą krotki, a `Vec<T>` jest elementem o indeksie 0 w krotce. Wtedy możemy użyć funkcjonalności cechy `Display` w `Wrapper`.

Wadą stosowania tej techniki jest to, że `Wrapper` jest nowym typem, więc
nie ma metod wartości, którą przechowuje. Musielibyśmy zaimplementować
wszystkie metody `Vec<T>` bezpośrednio w `Wrapper`, tak aby metody
delegowały do ​​`self.0`, co pozwoliłoby nam traktować `Wrapper` dokładnie tak jak
`Vec<T>`. Gdybyśmy chcieli, aby nowy typ miał wszystkie metody, jakie ma typ wewnętrzny,
implementacja cechy `Deref` (omówionej w rozdziale 15 w sekcji [„Traktowanie inteligentnych
wskaźników jak zwykłych odniesień z cechą `Deref`”][smart-pointer-deref]<!-- ignore -->) w `Wrapper` w celu zwrócenia
typu wewnętrznego byłaby rozwiązaniem. Gdybyśmy nie chcieli, aby typ `Wrapper` miał
wszystkie metody typu wewnętrznego — na przykład, aby ograniczyć zachowanie typu `Wrapper` — musielibyśmy ręcznie zaimplementować tylko te metody, których potrzebujemy.

Ten wzorzec nowego typu jest również przydatny, nawet gdy nie są zaangażowane cechy.
Zmieńmy punkt ciężkości i przyjrzyjmy się kilku zaawansowanym sposobom interakcji z systemem typów Rust.

[newtype]: ch20-03-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types
[implementing-a-trait-on-a-type]:
ch10-02-traits.html#implementing-a-trait-on-a-type
[traits-defining-shared-behavior]:
ch10-02-traits.html#traits-defining-shared-behavior
[smart-pointer-deref]: ch15-02-deref.html#treating-smart-pointers-like-regular-references-with-the-deref-trait
[tuple-structs]: ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types
