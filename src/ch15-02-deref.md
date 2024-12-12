## Traktowanie inteligentnych znaczników jak zwykłych odniesień z cechą  `Deref`

Implementacja cechy `Deref` pozwala dostosować zachowanie
*operatora dereferencji* `*` (nie mylić z operatorem mnożenia lub glob). Implementując `Deref` w taki sposób, że inteligentny wskaźnik może być
traktowany jak zwykłe odniesienie, możesz pisać kod, który działa na
odniesieniach i używać tego kodu również ze wskaźnikami inteligentnymi.

Najpierw przyjrzyjmy się, jak operator dereferencji działa ze zwykłymi odniesieniami.
Następnie spróbujemy zdefiniować niestandardowy typ, który zachowuje się jak `Box<T>` i zobaczymy, dlaczego
operator dereferencji nie działa jak odniesienie w naszym nowo zdefiniowanym
typie. Przeanalizujemy, w jaki sposób implementacja cechy `Deref` umożliwia
działanie inteligentnych wskaźników w sposób podobny do odniesień. Następnie przyjrzymy się
funkcji *przymusu deref* w Rust i temu, jak pozwala nam ona pracować zarówno z odniesieniami, jak i inteligentnymi wskaźnikami.

> Uwaga: istnieje jedna duża różnica między typem `MyBox<T>`, który zamierzamy
> zbudować, a prawdziwym `Box<T>`: nasza wersja nie będzie przechowywać swoich danych na stercie.
> Skupiamy się w tym przykładzie na `Deref`, więc miejsce, w którym dane są faktycznie przechowywane,
> jest mniej ważne niż zachowanie przypominające wskaźnik.

<!-- Stary link, nie usuwaj -->
<a id="following-the-pointer-to-the-value-with-the-dereference-operator"></a>

### Podążanie za wskaźnikiem do wartości

Regularne odniesienie jest typem wskaźnika, a jednym ze sposobów myślenia o wskaźniku jest
jak strzałka do wartości przechowywanej gdzie indziej. W Listingu 15-6 tworzymy
odniesienie do wartości `i32`, a następnie używamy operatora dereferencji, aby śledzić
odniesienie do wartości:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-06/src/main.rs}}
```

<span class="caption">Listing 15-6: Używanie operatora dereferencji do śledzenia
odniesienia do wartości `i32`</span>

Zmienna `x` zawiera wartość `i32` `5`. Ustawiamy `y` jako odniesienie do
`x`. Możemy stwierdzić, że `x` jest równe `5`. Jednak jeśli chcemy dokonać
afirmacji o wartości w `y`, musimy użyć `*y`, aby śledzić odniesienie
do wartości, na którą wskazuje (stąd *dereference*), aby kompilator mógł porównać
rzeczywistą wartość. Po dereferencji `y` mamy dostęp do wartości całkowitej, na którą wskazuje
`y`, którą możemy porównać z `5`.

Gdybyśmy zamiast tego spróbowali napisać `assert_eq!(5, y);`, otrzymalibyśmy ten błąd
kompilacji:

```console
{{#include ../listings/ch15-smart-pointers/output-only-01-comparing-to-reference/output.txt}}
```

Porównywanie liczby i odwołania do liczby nie jest dozwolone, ponieważ są one
różnymi typami. Musimy użyć operatora dereferencji, aby śledzić odwołanie
do wartości, na którą wskazuje.

### Używanie `Box<T>` jako odniesienia

Możemy przepisać kod z Listingu 15-6, aby użyć `Box<T>` zamiast
referencji; operator dereferencji użyty w `Box<T>` w Listingu 15-7
działa w ten sam sposób, co operator dereferencji użyty w referencji w
Listingu 15-6:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-07/src/main.rs}}
```

<span class="caption">Listing 15-7: Używanie operatora dereferencji w
`Box<i32>`</span>

Główną różnicą między Listingiem 15-7 a Listingiem 15-6 jest to, że tutaj ustawiamy
`y` jako instancję `Box<T>` wskazującą na skopiowaną wartość `x`, a nie
odniesienie wskazujące na wartość `x`. W ostatnim stwierdzeniu możemy
używać operatora dereferencji, aby śledzić wskaźnik `Box<T>` w taki sam sposób, w jaki robiliśmy to, gdy `y` było odniesieniem. Następnie zbadamy, co jest specjalnego w `Box<T>`, co pozwala nam używać operatora dereferencji poprzez
zdefiniowanie własnego typu.

### Definiowanie własnego inteligentnego wskaźnika

Zbudujmy inteligentny wskaźnik podobny do typu `Box<T>` dostarczanego przez
bibliotekę standardową, aby zobaczyć, jak inteligentne wskaźniki zachowują się inaczej niż
domyślnie referencje. Następnie przyjrzymy się, jak dodać możliwość użycia operatora
dereferencji.

Typ `Box<T>` jest ostatecznie zdefiniowany jako struktura krotki z jednym elementem, więc
Listing 15-8 definiuje typ `MyBox<T>` w ten sam sposób. Zdefiniujemy również
`nową` funkcję, aby pasowała do `nowej` funkcji zdefiniowanej w `Box<T>`.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-08/src/main.rs:here}}
```

<span class="caption">Listing 15-8: Definiowanie typu `MyBox<T>`</span>

Definiujemy strukturę o nazwie `MyBox` i deklarujemy ogólny parametr `T`, ponieważ
chcemy, aby nasz typ zawierał wartości dowolnego typu. Typ `MyBox` jest strukturą krotki
z jednym elementem typu `T`. Funkcja `MyBox::new` przyjmuje jeden parametr
typu `T` i zwraca instancję `MyBox`, która zawiera przekazaną wartość.

Spróbujmy dodać funkcję `main` z Listingu 15-7 do Listingu 15-8 i
zmienić ją tak, aby używała typu `MyBox<T>`, który zdefiniowaliśmy, zamiast `Box<T>`.
Kod z Listingu 15-9 nie skompiluje się, ponieważ Rust nie wie, jak dereferencjonować
`MyBox`.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-09/src/main.rs:here}}
```

<span class="caption">Listing 15-9: Próba użycia `MyBox<T>` w ten sam sposób, w jaki używaliśmy odniesień i `Box<T>`</span>

Oto wynikowy błąd kompilacji:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-09/output.txt}}
```

Nasz typ `MyBox<T>` nie może zostać dereferencjonowany, ponieważ nie zaimplementowaliśmy tej możliwości w naszym typie. Aby umożliwić dereferencjonowanie za pomocą operatora `*`,
implementujemy cechę `Deref`.

### Traktowanie typu jak referencji poprzez implementację cechy `Deref`

Jak omówiono w sekcji [„Implementowanie cechy w typie”][impl-trait]<!-- ignore
--> rozdziału 10, aby zaimplementować cechę, musimy zapewnić
implementacje wymaganych metod cechy. Cecha `Deref`, dostarczona
przez bibliotekę standardową, wymaga od nas zaimplementowania jednej metody o nazwie `deref`, która
pożycza `self` i zwraca odwołanie do danych wewnętrznych. Listing 15-10
zawiera implementację `Deref` do dodania do definicji `MyBox`:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-10/src/main.rs:here}}
```

<span class="caption">Listing 15-10: Implementacja `Deref` w `MyBox<T>`</span>

Składnia `type Target = T;` definiuje typ skojarzony, którego ma używać cecha `Deref`. Typy skojarzone to nieco inny sposób deklarowania
parametru ogólnego, ale na razie nie musisz się nimi martwić; omówimy je
bardziej szczegółowo w rozdziale 19.

Wypełniamy ciało metody `deref` za pomocą `&self.0`, więc `deref` zwraca
odwołanie do wartości, do której chcemy uzyskać dostęp za pomocą operatora `*`; przypomnij sobie z sekcji
[„Używanie struktur krotek bez pól nazwanych w celu tworzenia różnych
typów”][struktury krotek]<!-- ignoruj ​​--> rozdziału 5, że `.0` uzyskuje dostęp
do pierwszej wartości w strukturze krotek. Funkcja `main` w Listingu 15-9, która
wywołuje `*` na wartości `MyBox<T>` teraz się kompiluje, a asercje przechodzą!

Bez cechy `Deref` kompilator może jedynie dereferować referencje `&`.
Metoda `deref` daje kompilatorowi możliwość przyjęcia wartości dowolnego typu,
który implementuje `Deref` i wywołania metody `deref` w celu uzyskania referencji `&`, którą
wie, jak dereferować.

Gdy wpisaliśmy `*y` w Listingu 15-9, w tle Rust faktycznie uruchomił ten
kod:

```rust,ignore
*(y.deref())
```

Rust zastępuje operator `*` wywołaniem metody `deref`, a następnie
zwykłą dereferencją, dzięki czemu nie musimy zastanawiać się, czy musimy
wywołać metodę `deref`. Ta funkcja Rusta pozwala nam pisać kod, który działa
identycznie niezależnie od tego, czy mamy zwykłe odwołanie, czy typ implementujący
`Deref`.

Powodem, dla którego metoda `deref` zwraca odwołanie do wartości, a
zwykła dereferencja poza nawiasami w `*(y.deref())` jest nadal konieczna,
jest system własności. Gdyby metoda `deref` zwróciła wartość
bezpośrednio zamiast odwołania do wartości, wartość zostałaby przeniesiona poza
`self`. Nie chcemy przejmować własności wartości wewnętrznej wewnątrz `MyBox<T>` w
tym przypadku ani w większości przypadków, w których używamy operatora dereferencji.

Należy zauważyć, że operator `*` jest zastępowany wywołaniem metody `deref`, a
następnie wywołaniem operatora `*` tylko raz, za każdym razem, gdy używamy `*` w naszym kodzie.
Ponieważ podstawienie operatora `*` nie rekuruje w nieskończoność,
otrzymujemy dane typu `i32`, które pasują do `5` w `assert_eq!` w
Listingu 15-9.
### Ukryte wymuszenia deref z funkcją i metodami

*Przymus deref* konwertuje odwołanie do typu, który implementuje cechę `Deref`
na odwołanie do innego typu. Na przykład przymus deref może konwertować
`&String` na `&str`, ponieważ `String` implementuje cechę `Deref` w taki sposób, że
zwraca `&str`. Przymus deref to udogodnienie, które Rust wykonuje na argumentach
funkcji i metod, i działa tylko na typach, które implementują cechę `Deref`. Dzieje się to automatycznie, gdy przekazujemy odwołanie do wartości określonego typu
jako argument do funkcji lub metody, która nie pasuje do typu
parametru w definicji funkcji lub metody. Sekwencja wywołań metody `deref`
konwertuje typ, który podaliśmy, na typ, którego potrzebuje parametr.

Przymus deref został dodany do Rust, aby programiści piszący wywołania
funkcji i metod nie musieli dodawać tak wielu jawnych odwołań i dereferencji
za pomocą `&` i `*`. Funkcja przymusu deref pozwala nam również pisać więcej kodu, który
może działać zarówno dla odniesień, jak i inteligentnych wskaźników.

Aby zobaczyć przymus deref w akcji, użyjmy typu `MyBox<T>` zdefiniowanego w
Listingu 15-8, a także implementacji `Deref`, którą dodaliśmy w Listingu
15-10. Listing 15-11 pokazuje definicję funkcji, która ma parametr wycinka ciągu:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-11/src/main.rs:here}}
```

<span class="caption">Listing 15-11: Funkcja `hello`, która ma parametr
`name` typu `&str`</span>

Funkcję `hello` możemy wywołać z wycinkiem ciągu jako argumentem, takim jak na przykład
`hello("Rust");`. Przymus deref umożliwia wywołanie `hello`
z odwołaniem do wartości typu `MyBox<String>`, jak pokazano w Liście 15-12:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-12/src/main.rs:here}}
```

<span class="caption">Listing 15-12: Wywołanie `hello` z odwołaniem do wartości
`MyBox<String>`, co działa dzięki przymusowi deref</span>

Tutaj wywołujemy funkcję `hello` z argumentem `&m`, który jest
odwołaniem do wartości `MyBox<String>`. Ponieważ zaimplementowaliśmy cechę `Deref`
w `MyBox<T>` w Listingu 15-10, Rust może zamienić `&MyBox<String>` na `&String`
poprzez wywołanie `deref`. Standardowa biblioteka zapewnia implementację `Deref`
w `String`, która zwraca wycinek ciągu, i jest to w dokumentacji API
dla `Deref`. Rust wywołuje `deref` ponownie, aby zamienić `&String` na `&str`, co
pasuje do definicji funkcji `hello`.

Gdyby Rust nie implementował przymusu deref, musielibyśmy napisać kod z Listingu 15-13 zamiast kodu z Listingu 15-12, aby wywołać `hello` z wartością
typu `&MyBox<String>`.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-13/src/main.rs:here}}
```

<span class="caption">Listing 15-13: Kod, który musielibyśmy napisać, gdyby Rust
nie miał przymusu deref</span>

`(*m)` dereferuje `MyBox<String>` do `String`. Następnie `&` i
`[..]` biorą wycinek ciągu `String`, który jest równy całemu ciągowi, aby
dopasować podpis `hello`. Ten kod bez przymusów deref jest trudniejszy do
odczytania, napisania i zrozumienia ze wszystkimi tymi symbolami. Przymus deref
pozwala Rustowi automatycznie obsługiwać te konwersje.

Gdy cecha `Deref` jest zdefiniowana dla zaangażowanych typów, Rust przeanalizuje
typy i użyje `Deref::deref` tyle razy, ile będzie to konieczne, aby uzyskać odwołanie,
które będzie pasowało do typu parametru. Liczba wstawień `Deref::deref` jest ustalana w czasie kompilacji, więc nie ma kary za korzystanie z wymuszenia deref!

### Jak przymus Deref oddziałuje ze zmiennością

Podobnie jak używasz cechy `Deref` do nadpisania operatora `*` w
niezmiennych referencjach, możesz użyć cechy `DerefMut` do nadpisania operatora `*`
w zmiennych referencjach.

Rust wymusza deref, gdy znajduje typy i implementacje cech w trzech
przypadkach:

* Od `&T` do `&U`, gdy `T: Deref<Target=U>`
* Od `&mut T` do `&mut U`, gdy `T: DerefMut<Target=U>`
* Od `&mut T` do `&U`, gdy `T: Deref<Target=U>`

Pierwsze dwa przypadki są takie same, z wyjątkiem tego, że drugi
implementuje zmienność. Pierwszy przypadek stwierdza, że ​​jeśli masz `&T`, a `T`
implementuje `Deref` do pewnego typu `U`, możesz uzyskać `&U` transparentnie. W
drugim przypadku stwierdza się, że takie samo wymuszanie deref ma miejsce w przypadku referencji zmiennych.

Trzeci przypadek jest trudniejszy: Rust również wymusi referencję zmienną na
niezmienną. Ale odwrotna sytuacja *nie* jest możliwa: referencje niezmienne
nigdy nie wymuszą referencji zmiennych. Ze względu na reguły pożyczania, jeśli masz referencję zmienną, ta referencja zmienna musi być jedyną referencją do tych
danych (w przeciwnym razie program nie skompiluje się). Konwersja jednej referencji zmiennej
na jedną referencję niezmienną nigdy nie złamie reguł pożyczania.
Konwersja referencji niezmiennej na referencję zmienną wymagałaby, aby
początkowa referencja niezmienna była jedyną referencją niezmienną do tych danych, ale
reguły pożyczania tego nie gwarantują. Dlatego Rust nie może założyć, że konwersja referencji niezmiennej na referencję zmienną jest
możliwa.

[impl-trait]: ch10-02-traits.html#implementing-a-trait-on-a-type
[tuple-structs]: ch05-01-defining-structs.html#wykorzystanie-braku-nazywania-pól-w-struktorach-krotkowych-do-tworzenia-nowych-typów
