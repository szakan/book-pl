## Składnia wzorca

W tej sekcji zbieramy całą składnię obowiązującą we wzorcach i omawiamy, dlaczego i kiedy warto użyć każdej z nich.

### Matching Literals

Jak widziałeś w rozdziale 6, możesz bezpośrednio dopasowywać wzorce do literałów. Poniższy kod podaje kilka przykładów:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-01-literals/src/main.rs:here}}
```

Ten kod drukuje `jeden`, ponieważ wartość w `x` wynosi 1. Ta składnia jest przydatna,
gdy chcesz, aby Twój kod wykonał działanie, gdy otrzyma konkretną
wartość.

### Matching Named Variables

Zmienne nazwane to niepodważalne wzorce, które pasują do dowolnej wartości i używaliśmy ich
wiele razy w tej książce. Istnieje jednak komplikacja, gdy używasz
zmiennych nazwanych w wyrażeniach `match`. Ponieważ `match` rozpoczyna nowy zakres,
zmienne zadeklarowane jako część wzorca wewnątrz wyrażenia `match` będą
przyćmiewać zmienne o tej samej nazwie poza konstrukcją `match`, jak to ma miejsce
w przypadku wszystkich zmiennych. W Liście 19-11 deklarujemy zmienną o nazwie `x` z
wartością `Some(5)` i zmienną `y` z wartością `10`. Następnie tworzymy wyrażenie
`match` na wartości `x`. Spójrz na wzorce w ramionach dopasowania i
`println!` na końcu i spróbuj ustalić, co kod wydrukuje,
zanim uruchomisz ten kod lub przeczytasz dalej.

<Listing number="19-11" file-name="src/main.rs" caption="A `match` expression with an arm that introduces a shadowed variable `y`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-11/src/main.rs:here}}
```

</Listing>

Przeanalizujmy, co się dzieje, gdy uruchomione zostanie wyrażenie `match`. Wzorzec
w pierwszym ramieniu dopasowania nie pasuje do zdefiniowanej wartości `x`, więc kod
jest kontynuowany.

Wzorzec w drugim ramieniu dopasowania wprowadza nową zmienną o nazwie `y`, która
będzie pasowała do dowolnej wartości wewnątrz wartości `Some`. Ponieważ znajdujemy się w nowym zakresie wewnątrz
wyrażenia `match`, jest to nowa zmienna `y`, a nie `y` zadeklarowane na początku z wartością 10. To nowe powiązanie `y` będzie pasowało do dowolnej wartości
wewnątrz `Some`, którą mamy w `x`. Dlatego to nowe `y` wiąże się z
wartością wewnętrzną `Some` w `x`. Ta wartość to `5`, więc wyrażenie dla
tego ramienia jest wykonywane i drukuje `Matched, y = 5`.

Gdyby `x` było wartością `None` zamiast `Some(5)`, wzorce w pierwszych
dwóch ramionach nie pasowałyby, więc wartość pasowałaby do
podkreślenia. Nie wprowadziliśmy zmiennej `x` do wzorca
ramienia podkreślenia, więc `x` w wyrażeniu jest nadal zewnętrznym `x`, które nie zostało
zacienione. W tym hipotetycznym przypadku `match` wydrukowałoby `Default
case, x = None`.

Gdy wyrażenie `match` zostanie wykonane, jego zakres się kończy, podobnie jak zakres
wewnętrznego `y`. Ostatnie `println!` generuje `na końcu: x = Some(5), y = 10`.

Aby utworzyć wyrażenie `match`, które porównuje wartości zewnętrznego `x` i
`y`, zamiast wprowadzać zacienioną zmienną, musielibyśmy zamiast tego użyć warunku
ochrony dopasowania. O obrońcach porozmawiamy później w sekcji [„Dodatkowe
warunki-warunkowe-z-obrońcami-do-meczu”](#dodatkowe-warunki-warunkowe-z-obrońcami-do-meczu)<!--
ignoruj ​​-->.

### Multiple Patterns

W wyrażeniach `match` możesz dopasować wiele wzorców, używając składni `|`,
która jest operatorem wzorca *or*. Na przykład w poniższym kodzie dopasowujemy
wartość `x` do ramion dopasowania, z których pierwsze ma opcję *or*,
co oznacza, że ​​jeśli wartość `x` pasuje do którejkolwiek z wartości w tym ramieniu, kod tego ramienia zostanie uruchomiony:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-02-multiple-patterns/src/main.rs:here}}
```

This code prints `one or two`.

### Matching Ranges of Values with `..=`

Składnia `..=` pozwala nam dopasować do zakresu wartości inkluzywnych. W
następującym kodzie, gdy wzorzec pasuje do którejkolwiek z wartości w podanym
zakresie, to ramię zostanie wykonane:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-03-ranges/src/main.rs:here}}
```

Jeśli `x` wynosi 1, 2, 3, 4 lub 5, pierwsze ramię będzie pasować. Ta składnia jest
wygodniejsza w przypadku wielu wartości dopasowania niż użycie operatora `|` do wyrażenia
tego samego pomysłu; gdybyśmy użyli `|` musielibyśmy określić `1 | 2 | 3 | 4 | 5`.
Określenie zakresu jest znacznie krótsze, zwłaszcza jeśli chcemy dopasować, powiedzmy, dowolną
liczbę od 1 do 1000!

Kompilator sprawdza, czy zakres nie jest pusty w czasie kompilacji, a ponieważ
jedynymi typami, dla których Rust może stwierdzić, czy zakres jest pusty, czy nie, są `char` i
wartości liczbowe, zakresy są dozwolone tylko z wartościami liczbowymi lub `char`.

Oto przykład użycia zakresów wartości `char`:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-04-ranges-of-char/src/main.rs:here}}
```

Rust can tell that `'c'` is within the first pattern’s range and prints `early
ASCII letter`.

### Destructuring to Break Apart Values

Możemy również użyć wzorców do destrukturyzacji struktur, wyliczeń i krotek, aby użyć
różnych części tych wartości. Przeanalizujmy każdą wartość.

#### Destructuring Structs

Wylistowanie 19-12 przedstawia strukturę `Point` z dwoma polami, `x` i `y`, które możemy rozdzielić, używając wzorca z instrukcją `let`.

<Listing number="19-12" file-name="src/main.rs" caption="Destructuring a struct’s fields into separate variables">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-12/src/main.rs}}
```

</Listing>

Ten kod tworzy zmienne `a` i `b`, które pasują do wartości pól `x`
i `y` struktury `p`. Ten przykład pokazuje, że nazwy zmiennych
we wzorcu nie muszą pasować do nazw pól struktury.
Jednakże, często dopasowuje się nazwy zmiennych do nazw pól, aby łatwiej było zapamiętać, które zmienne pochodzą z których pól. Ze względu na to
powszechne użycie i ponieważ pisanie `let Point { x: x, y: y } = p;` zawiera
wiele duplikacji, Rust ma skrót dla wzorców, które pasują do pól struktury:
wystarczy podać nazwę pola struktury, a zmienne utworzone
ze wzorca będą miały takie same nazwy. Listing 19-13 zachowuje się w ten sam
sposób, co kod w Listing 19-12, ale zmienne utworzone we wzorcu `let`
to `x` i `y` zamiast `a` i `b`.

<Listing number="19-13" file-name="src/main.rs" caption="Destructuring struct fields using struct field shorthand">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-13/src/main.rs}}
```

</Listing>

Ten kod tworzy zmienne `x` i `y`, które pasują do pól `x` i `y`
zmiennej `p`. Rezultatem jest to, że zmienne `x` i `y` zawierają
wartości ze struktury `p`.

Możemy również destrukturyzować za pomocą wartości literowych jako części wzorca struktury
zamiast tworzyć zmienne dla wszystkich pól. Dzięki temu możemy przetestować
niektóre pola pod kątem określonych wartości, jednocześnie tworząc zmienne
destrukturyzujące pozostałe pola.

W Liście 19-14 mamy wyrażenie `match`, które rozdziela wartości `Point`
na trzy przypadki: punkty leżące bezpośrednio na osi `x` (co jest prawdą, gdy
`y = 0`), na osi `y` (`x = 0`) lub żadne z nich.
<Listing number="19-14" file-name="src/main.rs" caption="Destructuring and matching literal values in one pattern">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-14/src/main.rs:here}}
```

</Listing>

Pierwsze ramię będzie pasowało do dowolnego punktu leżącego na osi `x` poprzez określenie, że
pole `y` pasuje, jeśli jego wartość pasuje do dosłownego `0`. Wzorzec nadal
tworzy zmienną `x`, której możemy użyć w kodzie dla tego ramienia.

Podobnie drugie ramię pasuje do dowolnego punktu na osi `y` poprzez określenie, że
pole `x` pasuje, jeśli jego wartość wynosi `0` i tworzy zmienną `y` dla
wartości pola `y`. Trzecie ramię nie określa żadnych dosłownych, więc
dopasowuje dowolny inny `Punkt` i tworzy zmienne zarówno dla pól `x`, jak i `y`.

W tym przykładzie wartość `p` pasuje do drugiego ramienia na mocy `x`
zawierającego 0, więc ten kod wydrukuje `Na osi y w 7`.

Pamiętaj, że wyrażenie `match` zatrzymuje sprawdzanie ramion po znalezieniu
pierwszego pasującego wzorca, więc nawet jeśli `Point { x: 0, y: 0}` znajduje się na osi `x` i osi `y`, ten kod wydrukuje tylko `Na osi x w punkcie 0`.

#### Destructuring Enums

W tej książce destrukturyzowaliśmy wyliczenia (na przykład Listing 6-5 w rozdziale 6),
ale jeszcze nie omówiliśmy wprost, że wzorzec destrukturyzacji wyliczenia
odpowiada sposobowi, w jaki zdefiniowane są dane przechowywane w wyliczeniu. Jako
przykład, w Listing 19-15 używamy wyliczenia `Message` z Listing 6-2 i piszemy
`match` ze wzorcami, które destrukturyzują każdą wartość wewnętrzną.

<Listing number="19-15" file-name="src/main.rs" caption="Destructuring enum variants that hold different kinds of values">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-15/src/main.rs}}
```

</Listing>

Ten kod wydrukuje `Zmień kolor na czerwony 0, zielony 160 i niebieski 255`. Spróbuj
zmienić wartość `msg`, aby zobaczyć kod z innych ramion.

W przypadku wariantów wyliczeń bez żadnych danych, takich jak `Message::Quit`, nie możemy dalej destrukturyzować
wartości. Możemy dopasować tylko do wartości dosłownej `Message::Quit`,
a żadne zmienne nie znajdują się w tym wzorcu.

W przypadku wariantów wyliczeń typu struktura, takich jak `Message::Move`, możemy użyć wzorca
podobnego do wzorca, który określamy, aby dopasować struktury. Po nazwie wariantu,
umieszczamy nawiasy klamrowe, a następnie wymieniamy pola ze zmiennymi, aby rozdzielić
części do użycia w kodzie dla tego ramienia. Tutaj używamy formy skróconej, tak jak zrobiliśmy to w Liście 19-13.

W przypadku wariantów wyliczeń typu krotka, takich jak `Message::Write`, który zawiera krotkę z jednym
elementem i `Message::ChangeColor`, który zawiera krotkę z trzema elementami,
wzorzec jest podobny do wzorca, który określamy w celu dopasowania krotek. Liczba
zmiennych we wzorcu musi odpowiadać liczbie elementów w wariancie, który
dopasowujemy.

#### Destructuring Nested Structs and Enums

Do tej pory wszystkie nasze przykłady dopasowywały struktury lub wyliczenia o jeden poziom głębiej,
ale dopasowywanie może działać również na zagnieżdżonych elementach! Na przykład możemy przebudować
kod w Liście 19-15, aby obsługiwał kolory RGB i HSV w komunikacie `ChangeColor`, jak pokazano w Liście 19-16.

<Listing number="19-16" caption="Matching on nested enums">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-16/src/main.rs}}
```

</Listing>

Wzorzec pierwszego ramienia w wyrażeniu `match` pasuje do wariantu wyliczenia
`Message::ChangeColor`, który zawiera wariant `Color::Rgb`; następnie
wzorzec wiąże się z trzema wewnętrznymi wartościami `i32`. Wzorzec drugiego
ramienia również pasuje do wariantu wyliczenia `Message::ChangeColor`, ale wewnętrzne wyliczenie
pasuje zamiast tego do `Color::Hsv`. Możemy określić te złożone warunki w jednym
wyrażeniu `match`, nawet jeśli zaangażowane są dwa wyliczenia.
#### Destructuring Structs and Tuples

Możemy mieszać, dopasowywać i zagnieżdżać wzorce destrukturyzacji w jeszcze bardziej złożony sposób.
Poniższy przykład pokazuje skomplikowaną destrukturyzację, w której zagnieżdżamy struktury i
krotki wewnątrz krotki i destrukturyzujemy wszystkie wartości prymitywne:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-05-destructuring-structs-and-tuples/src/main.rs:here}}
```

Ten kod pozwala nam rozbić złożone typy na ich części składowe, dzięki czemu możemy używać
wartości, którymi jesteśmy zainteresowani, oddzielnie.

Destrukturyzacja za pomocą wzorców to wygodny sposób na używanie części wartości, takich jak
wartość z każdego pola w strukturze, oddzielnie od siebie.

### Ignoring Values in a Pattern

Widziałeś, że czasami przydatne jest ignorowanie wartości we wzorcu, na przykład
w ostatnim ramieniu `match`, aby uzyskać zbiór, który tak naprawdę nic nie robi,
ale uwzględnia wszystkie pozostałe możliwe wartości. Istnieje kilka
sposobów na ignorowanie całych wartości lub części wartości we wzorcu: użycie wzorca `_`
(który widziałeś), użycie wzorca `_` w innym wzorcu,
użycie nazwy zaczynającej się od podkreślenia lub użycie `..` w celu zignorowania pozostałych
części wartości. Przyjrzyjmy się, jak i dlaczego używać każdego z tych wzorców.

#### Ignoring an Entire Value with `_`

Użyliśmy podkreślenia jako wzorca wieloznacznego, który będzie pasował do dowolnej wartości, ale
nie będzie wiązał się z wartością. Jest to szczególnie przydatne jako ostatnie ramię w wyrażeniu `match`,
ale możemy go również użyć w dowolnym wzorcu, w tym w parametrach funkcji, jak pokazano w Liście 19-17.

<Listing number="19-17" file-name="src/main.rs" caption="Using `_` in a function signature">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-17/src/main.rs}}
```

</Listing>

Ten kod całkowicie zignoruje wartość `3` przekazaną jako pierwszy argument,
i wydrukuje `Ten kod używa tylko parametru y: 4`.

W większości przypadków, gdy nie potrzebujesz już konkretnego parametru funkcji,
zmieniłbyś sygnaturę, aby nie zawierała nieużywanego parametru. Ignorowanie
parametru funkcji może być szczególnie przydatne w przypadkach, gdy na przykład
implementujesz cechę, gdy potrzebujesz pewnego sygnatury typu, ale
ciało funkcji w Twojej implementacji nie potrzebuje jednego z parametrów. W ten sposób unikniesz otrzymania ostrzeżenia kompilatora o nieużywanych parametrach funkcji, co miałoby miejsce,
gdybyś zamiast tego użył nazwy.

#### Ignoring Parts of a Value with a Nested `_`

Możemy również użyć `_` wewnątrz innego wzorca, aby zignorować tylko część wartości, na przykład, gdy chcemy przetestować tylko część wartości, ale nie mamy zastosowania dla innych części w odpowiadającym kodzie, który chcemy uruchomić. Listing 19-18 pokazuje kod,
odpowiedzialny za zarządzanie wartością ustawienia. Wymagania biznesowe są takie, że
użytkownik nie powinien mieć możliwości nadpisania istniejącej personalizacji
ustawienia, ale może anulować ustawienie i nadać mu wartość, jeśli jest obecnie anulowane.

<Listing number="19-18" caption=" Using an underscore within patterns that match `Some` variants when we don’t need to use the value inside the `Some`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-18/src/main.rs:here}}
```

</Listing>

Ten kod wydrukuje `Nie można nadpisać istniejącej wartości niestandardowej`, a następnie
`setting to Some(5)`. W pierwszym ramieniu dopasowania nie musimy dopasowywać ani używać
wartości wewnątrz żadnej z odmian `Some`, ale musimy przetestować przypadek,
gdy `setting_value` i `new_setting_value` są odmianą `Some`. W takim przypadku drukujemy powód niezmieniania `setting_value` i nie jest on
zmieniany.

We wszystkich innych przypadkach (jeśli `setting_value` lub `new_setting_value` są
`Brak`) wyrażonych przez wzorzec `_` w drugim ramieniu, chcemy pozwolić, aby
`new_setting_value` stało się `setting_value`.

Możemy również użyć podkreśleń w wielu miejscach w jednym wzorcu, aby zignorować
konkretne wartości. Wypis 19-19 pokazuje przykład ignorowania drugiej i
czwartej wartości w krotce pięciu elementów.

<Listing number="19-19" caption="Ignoring multiple parts of a tuple">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-19/src/main.rs:here}}
```

</Listing>

This code will print `Some numbers: 2, 8, 32`, and the values 4 and 16 will be
ignored.

#### Ignoring an Unused Variable by Starting Its Name with `_`

Jeśli utworzysz zmienną, ale nigdzie jej nie użyjesz, Rust zazwyczaj wyświetli
ostrzeżenie, ponieważ nieużywana zmienna może być błędem. Jednak czasami
przydatna jest możliwość utworzenia zmiennej, której jeszcze nie użyjesz, na przykład gdy
tworzysz prototyp lub dopiero zaczynasz projekt. W takiej sytuacji możesz powiedzieć Rustowi,
aby nie ostrzegał Cię o nieużywanej zmiennej, rozpoczynając nazwę zmiennej
od podkreślenia. W listingu 19-20 tworzymy dwie nieużywane zmienne, ale gdy
kompilujemy ten kod, powinniśmy otrzymać ostrzeżenie tylko o jednej z nich.

<Listing number="19-20" file-name="src/main.rs" caption="Starting a variable name with an underscore to avoid getting unused variable warnings">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-20/src/main.rs}}
```

</Listing>

Tutaj otrzymujemy ostrzeżenie o nieużywaniu zmiennej `y`, ale nie otrzymujemy
ostrzeżenia o nieużywaniu `_x`.

Należy zauważyć, że istnieje subtelna różnica między użyciem tylko `_` a użyciem nazwy,
która zaczyna się od podkreślenia. Składnia `_x` nadal wiąże wartość ze zmienną,
podczas gdy `_` nie wiąże się wcale. Aby pokazać przypadek, w którym to
rozróżnienie ma znaczenie, Listing 19-21 wyświetli nam błąd.

<Listing number="19-21" caption="An unused variable starting with an underscore still binds the value, which might take ownership of the value">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-21/src/main.rs:here}}
```

</Listing>

Otrzymamy błąd, ponieważ wartość `s` nadal zostanie przeniesiona do `_s`,
co uniemożliwi nam ponowne użycie `s`. Jednak użycie samego podkreślenia
nigdy nie wiąże się z wartością. Listing 19-22 skompiluje się bez żadnych błędów,
ponieważ `s` nie zostanie przeniesione do `_`.

<Listing number="19-22" caption="Using an underscore does not bind the value">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-22/src/main.rs:here}}
```

</Listing>

This code works just fine because we never bind `s` to anything; it isn’t moved.

#### Ignoring Remaining Parts of a Value with `..`

W przypadku wartości, które mają wiele części, możemy użyć składni `..`, aby użyć określonych
części i zignorować resztę, unikając konieczności wymieniania podkreśleń dla każdej
ignorowanej wartości. Wzorzec `..` ignoruje wszelkie części wartości, których nie
dopasowaliśmy jawnie w pozostałej części wzorca. W Liście 19-23 mamy strukturę
`Point`, która przechowuje współrzędną w przestrzeni trójwymiarowej. W wyrażeniu
`match` chcemy operować tylko na współrzędnej `x` i zignorować
wartości w polach `y` i `z`.

<Listing number="19-23" caption="Ignoring all fields of a `Point` except for `x` by using `..`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-23/src/main.rs:here}}
```

</Listing>

Wypisujemy wartość `x`, a następnie po prostu uwzględniamy wzorzec `..`. Jest to szybsze
od konieczności wypisywania `y: _` i `z: _`, szczególnie gdy pracujemy ze
strukturami, które mają wiele pól w sytuacjach, w których tylko jedno lub dwa pola są
istotne.

Składnia `..` zostanie rozszerzona do tylu wartości, ile będzie potrzeba. Wypis 19-24
pokazuje, jak używać `..` z krotką.

<Listing number="19-24" file-name="src/main.rs" caption="Matching only the first and last values in a tuple and ignoring all other values">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-24/src/main.rs}}
```

</Listing>

W tym kodzie pierwsza i ostatnia wartość są dopasowywane do `first` i `last`.
`..` dopasuje i zignoruje wszystko w środku.

Jednak użycie `..` musi być jednoznaczne. Jeśli nie jest jasne, które wartości są
przeznaczone do dopasowania, a które powinny zostać zignorowane, Rust zwróci błąd.
Listing 19-25 pokazuje przykład użycia `..` w sposób niejednoznaczny, więc nie zostanie
skompilowany.

<Listing number="19-25" file-name="src/main.rs" caption="An attempt to use `..` in an ambiguous way">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-25/src/main.rs}}
```

</Listing>

Podczas kompilacji tego przykładu pojawia się następujący błąd:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-25/output.txt}}
```

Rust nie jest w stanie określić, ile wartości w krotce zignorować
przed dopasowaniem wartości do `second`, a następnie ile kolejnych wartości zignorować
po tym. Ten kod może oznaczać, że chcemy zignorować `2`, powiązać
`second` z `4`, a następnie zignorować `8`, `16` i `32`; lub że chcemy zignorować
`2` i `4`, powiązać `second` z `8`, a następnie zignorować `16` i `32`; i tak dalej.
Nazwa zmiennej `second` nie oznacza niczego szczególnego dla Rust, więc otrzymujemy
błąd kompilatora, ponieważ użycie `..` w dwóch takich miejscach jest niejednoznaczne.

### Extra Conditionals with Match Guards

*Match guard* to dodatkowy warunek `if`, określony po wzorcu w
ramie `match`, który musi również pasować, aby to ramię zostało wybrane. Match guards są
przydatne do wyrażania bardziej złożonych idei niż pozwala na to sam wzorzec.

Warunek może używać zmiennych utworzonych we wzorcu. Listing 19-26 pokazuje
`match`, gdzie pierwsze ramię ma wzorzec `Some(x)` i ma również match guard `if x % 2 == 0` (który będzie prawdziwy, jeśli liczba jest parzysta).

<Listing number="19-26" caption="Adding a match guard to a pattern">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-26/src/main.rs:here}}
```

</Listing>

Ten przykład wydrukuje `Liczba 4 jest parzysta`. Gdy `num` jest porównywane ze
wzorcem w pierwszym ramieniu, pasuje, ponieważ `Some(4)` pasuje do `Some(x)`. Następnie
ochrona dopasowania sprawdza, czy reszta z dzielenia `x` przez 2 jest równa
0, a ponieważ tak jest, wybierane jest pierwsze ramię.

Gdyby `num` było `Some(5)`, ochrona dopasowania w pierwszym ramieniu
byłaby fałszywa, ponieważ reszta z dzielenia 5 przez 2 wynosi 1, co nie jest równe
0. Następnie Rust przeszedłby do drugiego ramienia, które byłoby dopasowane, ponieważ
drugie ramię nie ma ochrony dopasowania i dlatego pasuje do dowolnego wariantu `Some`.

Nie ma sposobu, aby wyrazić warunek `if x % 2 == 0` w obrębie wzorca, więc
ochrona dopasowania daje nam możliwość wyrażenia tej logiki. Wadą
tej dodatkowej ekspresywności jest to, że kompilator nie próbuje sprawdzać
wyczerpania, gdy zaangażowane są wyrażenia match guard.

W Liście 19-11 wspomnieliśmy, że możemy użyć match guardów, aby rozwiązać nasz
problem z cieniowaniem wzorca. Przypomnijmy, że utworzyliśmy nową zmienną wewnątrz
wzorca w wyrażeniu `match` zamiast używać zmiennej poza
`match`. Ta nowa zmienna oznaczała, że ​​nie mogliśmy testować wartości zmiennej
zewnętrznej. Listing 19-27 pokazuje, jak możemy użyć match guardów, aby rozwiązać ten
problem.

<Listing number="19-27" file-name="src/main.rs" caption="Using a match guard to test for equality with an outer variable">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-27/src/main.rs}}
```

</Listing>

Ten kod wydrukuje teraz `Domyślny przypadek, x = Some(5)`. Wzorzec w drugim
ramionie dopasowania nie wprowadza nowej zmiennej `y`, która przesłaniałaby zewnętrzne `y`,
co oznacza, że ​​możemy użyć zewnętrznego `y` w straży dopasowania. Zamiast określać
wzorzec jako `Some(y)`, co przesłaniałoby zewnętrzne `y`, określamy
`Some(n)`. Tworzy to nową zmienną `n`, która niczego nie przesłania, ponieważ
nie ma zmiennej `n` poza `match`.

Strona dopasowania `if n == y` nie jest wzorcem i dlatego nie wprowadza
nowych zmiennych. To `y` *jest* zewnętrznym `y`, a nie nowym zacienionym `y`, i
możemy poszukać wartości, która ma taką samą wartość jak zewnętrzne `y`, porównując
`n` z `y`.

Możesz również użyć operatora *or* `|` w straży dopasowania, aby określić wiele
wzorców; warunek match guard będzie dotyczył wszystkich wzorców. Listing
19-28 pokazuje pierwszeństwo podczas łączenia wzorca, który używa `|` z match guard. Ważną częścią tego przykładu jest to, że `if y` match guard
dotyczy `4`, `5`, *i* `6`, nawet jeśli mogłoby się wydawać, że `if y`
dotyczy tylko `6`.

<Listing number="19-28" caption="Combining multiple patterns with a match guard">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-28/src/main.rs:here}}
```

</Listing>

Warunek dopasowania stwierdza, że ​​ramię pasuje tylko wtedy, gdy wartość `x` jest
równa `4`, `5` lub `6` *i* jeśli `y` jest `true`. Gdy ten kod jest uruchomiony,
wzorzec pierwszego ramienia pasuje, ponieważ `x` jest `4`, ale strażnik dopasowania `if y`
jest false, więc pierwsze ramię nie jest wybierane. Kod przechodzi do drugiego ramienia,
które pasuje, a ten program drukuje `no`. Powodem jest to, że warunek `if`
dotyczy całego wzorca `4 | 5 | 6`, a nie tylko ostatniej wartości
`6`. Innymi słowy, pierwszeństwo strażnika dopasowania w odniesieniu do wzorca
zachowuje się w następujący sposób:

```text
(4 | 5 | 6) if y => ...
```

rather than this:

```text
4 | 5 | (6 if y) => ...
```

Po uruchomieniu kodu zachowanie pierwszeństwa jest oczywiste: gdyby ochrona dopasowania
została zastosowana tylko do ostatniej wartości na liście wartości określonych za pomocą operatora
`|`, ramię zostałoby dopasowane, a program wydrukowałby
`yes`.

### `@` Bindings

Operator *at* `@` pozwala nam utworzyć zmienną, która przechowuje wartość w tym samym czasie, gdy testujemy tę wartość pod kątem dopasowania wzorca. W listingu 19-29 chcemy
przetestować, czy pole `Message::Hello` `id` mieści się w zakresie `3..=7`. Chcemy również
powiązać wartość ze zmienną `id_variable`, abyśmy mogli jej użyć w kodzie powiązanym z ramieniem. Możemy nazwać tę zmienną `id`, tak samo jak pole, ale w tym przykładzie użyjemy innej nazwy.

<Listing number="19-29" caption="Using `@` to bind to a value in a pattern while also testing it">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-29/src/main.rs:here}}
```

</Listing>

Ten przykład wydrukuje `Znaleziono id w zakresie: 5`. Określając `id_variable
@` przed zakresem `3..=7`, przechwytujemy dowolną wartość pasującą do zakresu,
jednocześnie testując, czy wartość pasuje do wzorca zakresu.

W drugim ramieniu, gdzie mamy tylko zakres określony we wzorcu, kod
skojarzony z ramieniem nie ma zmiennej, która zawiera rzeczywistą wartość
pola `id`. Wartość pola `id` mogła wynosić 10, 11 lub 12, ale
kod, który pasuje do tego wzorca, nie wie, która to wartość. Kod wzorca
nie jest w stanie użyć wartości z pola `id`, ponieważ nie zapisaliśmy wartości
`id` w zmiennej.

W ostatnim ramieniu, gdzie określiliśmy zmienną bez zakresu, mamy
wartość dostępną do użycia w kodzie ramienia w zmiennej o nazwie `id`. Powód jest taki, że użyliśmy skróconej składni pola struktury. Ale nie zastosowaliśmy żadnego testu do wartości w polu `id` w tym ramieniu, jak zrobiliśmy to w przypadku
dwóch pierwszych ramion: każda wartość pasowałaby do tego wzorca.

Użycie `@` pozwala nam przetestować wartość i zapisać ją w zmiennej w ramach jednego wzorca.

## Summary

Wzory Rusta są bardzo przydatne w rozróżnianiu różnych rodzajów
danych. Gdy są używane w wyrażeniach `match`, Rust zapewnia, że ​​Twoje wzorce obejmują każdą
możliwą wartość, w przeciwnym razie Twój program nie zostanie skompilowany. Wzory w instrukcjach `let` i
parametrach funkcji sprawiają, że te konstrukcje są bardziej użyteczne, umożliwiając
destrukturyzację wartości na mniejsze części w tym samym czasie, co przypisywanie do
zmiennych. Możemy tworzyć proste lub złożone wzorce, aby dopasować je do naszych potrzeb.

Następnie, w przedostatnim rozdziale książki, przyjrzymy się niektórym zaawansowanym
aspektom różnych funkcji Rusta.
