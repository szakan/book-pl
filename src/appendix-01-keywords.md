## Załącznik A: Keywords

Poniższa lista zawiera słowa kluczowe zarezerwowane do aktualnego lub przyszłego użytku w języku Rust. W związku z tym nie mogą być one używane jako identyfikatory (z wyjątkiem tzw. surowych identyfikatorów, o których porozmawiamy w sekcji “[Raw
Identifiers][raw-identifiers]<!-- ignore -->”). Identyfikatory to nazwy funkcji, zmiennych, parametrów, pól struktur, modułów, skrzynek, stałych, makr, wartości statycznych, atrybutów, typów, cech lub czasów życia.

[raw-identifiers]: #raw-identifiers

### Słowa kluczowe aktualnie w użyciu

Poniżej znajduje się lista słów kluczowych aktualnie używanych, wraz z opisem ich funkcji.

* `as` - wykonuje rzutowanie prymitywne, rozróżnia specyficzną cechę zawierającą element lub zmienia nazwę elementów w instrukcjach use
* `async` - zwraca obiekt Future zamiast blokować bieżący wątek
* `await` - wstrzymuje wykonywanie do momentu, aż wynik Future będzie gotowy
* `break` - natychmiast wychodzi z pętli
* `const` - definiuje stałe elementy lub wskaźniki surowe
* `continue` - przechodzi do następnej iteracji pętli
* `crate` - w ścieżce modułu odnosi się do głównego modułu crate
* `dyn` - dynamiczna dyspozycja do obiektu cechy
* `else` - alternatywne wykonanie w konstrukcjach if i if let
* `enum` - definiuje wyliczenie
* `extern` - łączy zewnętrzną funkcję lub zmienną
* `false` - literał logiczny false
* `fn` - definiuje funkcję lub typ wskaźnika funkcji
* `for` - iteruje po elementach iteratora, implementuje cechę lub określa wyższą rangę życia
* `if` - rozgałęzia się na podstawie wyniku wyrażenia warunkowego
* `impl` - implementuje wbudowaną funkcjonalność lub funkcjonalność cechy
* `in` - część składni pętli for
* `let` - wiąże zmienną
* `loop` - wykonuje nieskończoną pętlę
* `match` - dopasowuje wartość do wzorców
* `mod` - definiuje moduł
* `move` - powoduje, że wyrażenie zamykające przejmuje własność wszystkich przechwytywanych elementów
* `mut` - oznacza zmienność referencji, wskaźników surowych lub wiązań wzorców
* `pub` - oznacza publiczną widoczność pól struktury, bloków impl lub modułów
* `ref` - wiąże przez referencję
* `return` - zwraca z funkcji
* `Self` - alias typu dla typu, który definiujemy lub implementujemy
* `self` - obiekt metody lub bieżący moduł
* `static` - zmienna globalna lub czas życia trwający przez cały czas wykonywania programu
* `struct` - definiuje strukturę
* `super` - odnosi się do modułu nadrzędnego względem bieżącego modułu
* `trait` - definiuje cechę
* `true` - literał logiczny true
* `type` - definiuje alias typu lub typ powiązany
* `union` - definiuje unię; jest słowem kluczowym tylko w deklaracji unii
* `unsafe` - oznacza niebezpieczny kod, funkcje, cechy lub implementacje
* `use` - wprowadza symbole do zakresu
* `where` - oznacza klauzule ograniczające typ
* `while` - warunkowo wykonuje pętlę na podstawie wyniku wyrażenia

[union]: ../reference/items/unions.html

### Słowa kluczowe zarezerwowane do wykorzystania w przyszłości

Poniższe słowa kluczowe nie mają jeszcze przypisanej funkcji, ale są zarezerwowane przez Rust do potencjalnego przyszłego użytku.

* `abstract`
* `become`
* `box`
* `do`
* `final`
* `macro`
* `override`
* `priv`
* `try`
* `typeof`
* `unsized`
* `virtual`
* `yield`

### Surowe identyfikatory

Surowe identyfikatory to składnia, która pozwala używać słów kluczowych w miejscach, gdzie normalnie byłyby niedozwolone. Używasz surowego identyfikatora, poprzedzając słowo kluczowe prefiksem `r#`.

Na przykład `match` jest słowem kluczowym. Jeśli spróbujesz skompilować następującą funkcję, która używa `match` jako swojej nazwy:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
fn match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}
```

otrzymasz następujący błąd:

```text
error: expected identifier, found keyword `match`
 --> src/main.rs:4:4
  |
4 | fn match(needle: &str, haystack: &str) -> bool {
  |    ^^^^^ expected identifier, found keyword
```

Błąd pokazuje, że nie możesz używać słowa kluczowego `match` jako identyfikatora funkcji. Aby użyć `match` jako nazwy funkcji, musisz skorzystać ze składni surowego identyfikatora, jak w poniższym przykładzie:

<span class="filename">Filename: src/main.rs</span>

```rust
fn r#match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}

fn main() {
    assert!(r#match("foo", "foobar"));
}
```

Ten kod skompiluje się bez żadnych błędów. Zauważ prefiks `r#` zarówno w definicji nazwy funkcji, jak i w miejscu, gdzie funkcja jest wywoływana w `main`.

Surowe identyfikatory pozwalają na użycie dowolnego słowa jako identyfikatora, nawet jeśli to słowo jest zarezerwowanym słowem kluczowym. Daje to większą swobodę w wyborze nazw identyfikatorów oraz umożliwia integrację z programami napisanymi w językach, gdzie te słowa nie są słowami kluczowymi. Ponadto surowe identyfikatory pozwalają używać bibliotek napisanych w innej edycji Rust niż ta, której używa Twoja skrzynka. Na przykład `try` nie jest słowem kluczowym w edycji 2015, ale jest w edycji 2018. Jeśli zależysz od biblioteki napisanej w edycji 2015, która posiada funkcję `try`, będziesz musiał użyć składni surowego identyfikatora, `r#try` w tym przypadku, aby wywołać tę funkcję w kodzie napisanym w edycji 2018. Zobacz [Dodatek E][appendix-e]<!-- ignore -->, aby uzyskać więcej informacji o edycjach.

[appendix-e]: appendix-05-editions.html
