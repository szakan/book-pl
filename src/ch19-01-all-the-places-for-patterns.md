## Wszystkie miejsca. w których można używać wzorców

Wzory pojawiają się w wielu miejscach w Rust, a Ty używasz ich dużo, nie zdając sobie z tego sprawy! Ta sekcja omawia wszystkie miejsca, w których wzorce są
ważne.

### `match` Arms

Jak omówiono w rozdziale 6, używamy wzorców w ramionach wyrażeń `match`.
Formalnie wyrażenia `match` są definiowane jako słowo kluczowe `match`, wartość, do której
należy dopasować, oraz jedno lub więcej ramion dopasowania, które składają się ze wzorca i
wyrażenia, które należy uruchomić, jeśli wartość pasuje do wzorca tego ramienia, w następujący sposób:

```text
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```

Na przykład, oto wyrażenie `match` z Listingu 6-5, które pasuje do wartości
`Option<i32>` w zmiennej `x`:

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

Wzorce w tym wyrażeniu `match` to `None` i `Some(i)` po
lewej stronie każdej strzałki.

Jednym z wymogów dla wyrażeń `match` jest to, że muszą być *wyczerpujące* w tym sensie, że wszystkie możliwości wartości w wyrażeniu `match` muszą być
uwzględnione. Jednym ze sposobów upewnienia się, że uwzględniono każdą możliwość, jest posiadanie
wzorca catch-all dla ostatniego ramienia: na przykład nazwa zmiennej pasująca do dowolnej
wartości nigdy nie może się nie powieść i w ten sposób obejmuje każdy pozostały przypadek.

Konkretny wzorzec `_` będzie pasował do wszystkiego, ale nigdy nie wiąże się ze zmienną, więc jest często używany w ostatnim ramieniu dopasowania. Wzorzec `_` może być
przydatny, gdy chcesz zignorować dowolną wartość, która nie została określona, ​​na przykład. Wzorzec `_` omówimy bardziej szczegółowo w sekcji [„Ignorowanie wartości we
wzorze”][ignoring-values-in-a-pattern]<!-- ignoruj ​​--> w dalszej części tego
rozdziału.

### Wyrażenia warunkowe `if let`

W rozdziale 6 omówiliśmy, jak używać wyrażeń `if let` głównie jako krótszego sposobu
na zapisanie odpowiednika `match`, który pasuje tylko do jednego przypadku.
Opcjonalnie `if let` może mieć odpowiadające mu `else` zawierające kod do uruchomienia, jeśli
wzorzec w `if let` nie pasuje.

Listing 19-1 pokazuje, że możliwe jest również mieszanie i dopasowywanie wyrażeń `if let`, `else
if` i `else if let`. Daje nam to większą elastyczność niż wyrażenie
`match`, w którym możemy wyrazić tylko jedną wartość do porównania ze
wzorcami. Ponadto Rust nie wymaga, aby warunki w serii `if
let`, `else if`, `else if let` ramiona były ze sobą powiązane.

Kod w Listingu 19-1 określa, jaki kolor ustawić na tle na podstawie
serii sprawdzeń dla kilku warunków. W tym przykładzie utworzyliśmy zmienne z zakodowanymi na stałe wartościami, które prawdziwy program może otrzymać z danych wejściowych użytkownika.

<Listing number="19-1" file-name="src/main.rs" caption="Mixing `if let`, `else if`, `else if let`, and `else`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-01/src/main.rs}}
```

</Listing>

Jeśli użytkownik określi ulubiony kolor, kolor ten jest używany jako tło.
Jeśli nie określono ulubionego koloru, a dziś jest wtorek, kolor tła jest
zielony. W przeciwnym razie, jeśli użytkownik określi swój wiek jako ciąg znaków i możemy go pomyślnie przeanalizować
jako liczbę, kolor jest fioletowy lub pomarańczowy, w zależności od
wartości liczby. Jeśli żaden z tych warunków nie ma zastosowania, kolor tła jest niebieski.

Ta struktura warunkowa pozwala nam obsługiwać złożone wymagania. Dzięki
zakodowanym na stałe wartościom, które mamy tutaj, ten przykład wydrukuje `Używanie fioletu jako
koloru tła`.

Możesz zobaczyć, że `if let` może również wprowadzać zmienne zacienione w taki sam sposób, w jaki może to robić `match` arms: linia `if let Ok(age) = age` wprowadza nową
zacienioną zmienną `age`, która zawiera wartość wewnątrz wariantu `Ok`. Oznacza to, że musimy umieścić warunek `if age > 30` w tym bloku: nie możemy
połączyć tych dwóch warunków w `if let Ok(age) = age && age > 30`.
Zacieniony `age`, który chcemy porównać z 30, nie jest prawidłowy, dopóki nowy zakres nie zacznie się
od nawiasu klamrowego.

Wadą używania wyrażeń `if let` jest to, że kompilator nie sprawdza,
czy są one wyczerpujące, podczas gdy w przypadku wyrażeń `match` to robi. Gdybyśmy pominęli
ostatni blok `else` i w związku z tym nie obsłużyli niektórych przypadków, kompilator
nie powiadomiłby nas o możliwym błędzie logicznym.

### Pętle warunkowe `while let`

Podobna w konstrukcji do `if let`, pętla warunkowa `while let` pozwala pętli
`while` działać tak długo, jak długo wzorzec jest zgodny. Pierwszy raz zobaczyliśmy pętlę
`while let` w rozdziale 17, gdzie użyliśmy jej do kontynuowania pętli tak długo, jak
strumień generował nowe wartości. Podobnie, w Liście 19-2 pokazujemy pętlę `while let`, która czeka na wiadomości wysyłane między wątkami, ale w tym przypadku sprawdzając
`Result` zamiast `Option`.

<Listing number="19-2" caption="Using a `while let` loop to print values for as long as `rx.recv()` returns `Ok`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-02/src/main.rs:here}}
```

</Listing>

Ten przykład drukuje 1, 2 i 3. Kiedy zobaczyliśmy `recv` w rozdziale 16,
rozpakowaliśmy błąd bezpośrednio lub wchodziliśmy z nim w interakcję jako iteratorem za pomocą pętli `for`. Jak pokazuje Listing 19-2, możemy również użyć `while let`, ponieważ metoda
`recv` zwraca `Ok` tak długo, jak nadawca produkuje wiadomości, a następnie
produkuje `Err`, gdy strona nadawcy się rozłączy.

### `for` Loops

W pętli `for` wartość, która następuje bezpośrednio po słowie kluczowym `for` jest
wzorcem. Na przykład w `for x in y` `x` jest wzorcem. Listing 19-3
demonstruje, jak używać wzorca w pętli `for`, aby rozłożyć na części krotkę jako część pętli `for`.

<Listing number="19-3" caption="Using a pattern in a `for` loop to destructure a tuple">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-03/src/main.rs:here}}
```

</Listing>

The code in Listing 19-3 will print the following:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-03/output.txt}}
```

Dostosowujemy iterator za pomocą metody `enumerate`, tak aby generował wartość i
indeks tej wartości, umieszczony w krotce. Pierwszą wygenerowaną wartością jest
krotka `(0, 'a')`. Gdy ta wartość zostanie dopasowana do wzorca `(indeks, wartość)`,
`indeks` będzie równy `0`, a `wartość` będzie równa `'a'`, drukując pierwszy wiersz
wyjścia.

### Instrukcje `let`

Przed tym rozdziałem omawialiśmy wyraźnie używanie wzorców tylko z
`match` i `if let`, ale w rzeczywistości używaliśmy wzorców również w innych miejscach,
w tym w instrukcjach `let`. Na przykład rozważmy to proste
przypisanie zmiennej z `let`:

```rust
let x = 5;
```

Za każdym razem, gdy użyłeś instrukcji `let` w ten sposób, użyłeś wzorców,
chociaż mogłeś tego nie zauważyć! Bardziej formalnie, instrukcja `let` wygląda
tak:

```text
let PATTERN = EXPRESSION;
```

W stwierdzeniach takich jak `let x = 5;` z nazwą zmiennej w slocie `PATTERN`,
nazwa zmiennej jest po prostu szczególnie prostą formą wzorca. Rust porównuje
wyrażenie ze wzorcem i przypisuje wszystkie znalezione nazwy. Tak więc w przykładzie
`let x = 5;`, `x` jest wzorcem oznaczającym „powiąż to, co pasuje tutaj, ze
zmienną `x`”. Ponieważ nazwa `x` jest całym wzorcem, ten wzorzec
w praktyce oznacza „powiąż wszystko ze zmienną `x`, bez względu na jej wartość”.

Aby zobaczyć aspekt dopasowania wzorca `let` jaśniej, rozważ Listing
19-4, który używa wzorca z `let` do destrukturyzacji krotki.

<Listing number="19-4" caption="Using a pattern to destructure a tuple and create three variables at once">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-04/src/main.rs:here}}
```

</Listing>

Tutaj dopasowujemy krotkę do wzorca. Rust porównuje wartość `(1, 2, 3)`
do wzorca `(x, y, z)` i widzi, że wartość pasuje do wzorca, więc Rust
wiąże `1` z `x`, `2` z `y` i `3` z `z`. Możesz myśleć o tym wzorcu krotki
jak o zagnieżdżeniu trzech indywidualnych wzorców zmiennych w nim.

Jeśli liczba elementów we wzorcu nie pasuje do liczby elementów
w krotce, ogólny typ nie będzie pasował i otrzymamy błąd kompilatora. Na przykład, Listing 19-5 pokazuje próbę destrukturyzacji krotki z trzema
elementami na dwie zmienne, co się nie powiedzie.

<Listing number="19-5" caption="Incorrectly constructing a pattern whose variables don’t match the number of elements in the tuple">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-05/src/main.rs:here}}
```

</Listing>

Próba skompilowania tego kodu powoduje następujący błąd typu:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-05/output.txt}}
```

Aby naprawić błąd, możemy zignorować jedną lub więcej wartości w krotce, używając
`_` lub `..`, jak zobaczysz w sekcji [„Ignorowanie wartości we
wzorze”][ignoring-values-in-a-pattern]<!-- ignore -->. Jeśli problem
polega na tym, że we wzorcu mamy zbyt wiele zmiennych, rozwiązaniem jest dopasowanie
typów poprzez usunięcie zmiennych, tak aby liczba zmiennych była równa liczbie
elementów w krotce.

### Function Parameters

Parametry funkcji mogą być również wzorcami. Kod w Listingu 19-6, który
deklaruje funkcję o nazwie `foo`, która przyjmuje jeden parametr o nazwie `x` typu
`i32`, powinien teraz wyglądać znajomo.

<Listing number="19-6" caption="A function signature uses patterns in the parameters">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-06/src/main.rs:here}}
```

</Listing>

Część `x` jest wzorcem! Tak jak zrobiliśmy to z `let`, mogliśmy dopasować krotkę w argumentach
funkcji do wzorca. Listing 19-7 dzieli wartości w krotce,
gdy przekazujemy ją do funkcji.

<Listing number="19-7" file-name="src/main.rs" caption="A function with parameters that destructure a tuple">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-07/src/main.rs}}
```

</Listing>

Ten kod drukuje `Bieżąca lokalizacja: (3, 5)`. Wartości `&(3, 5)` pasują do
wzorca `&(x, y)`, więc `x` jest wartością `3`, a `y` jest wartością `5`.

Możemy również używać wzorców na listach parametrów zamknięć w taki sam sposób, jak w
listach parametrów funkcji, ponieważ zamknięcia są podobne do funkcji, jak omówiono w rozdziale 13.

W tym momencie widziałeś kilka sposobów używania wzorców, ale wzorce nie
działają tak samo w każdym miejscu, w którym możemy ich użyć. W niektórych miejscach wzorce muszą być niepodważalne; w innych okolicznościach mogą być obalane. Omówimy
te dwa pojęcia dalej.

[ignoring-values-in-a-pattern]:
ch19-03-pattern-syntax.html#ignoring-values-in-a-pattern
