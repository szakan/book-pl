## Definiowanie i tworzenie instancji struktur

Struktury są podobne do krotek, które omawiane były [w rozdziale 3][tuples]<!-- ignore -->. 
Podobnie do krotek, poszczególne części struktur mogą różnić się typami.
W odróżnieniu od krotek, każdy fragment danych musisz nazwać,
aby jasne było co każdy oznacza. W wyniku tego nazewnictwa struktury są bardziej
elastyczne od krotek. 
Nie musisz polegać na kolejności danych, aby dostać się do wartości danej struktury.

Aby zdefiniować strukturę posługujemy się słowem kluczowym `struct`, po którym wstawiamy nazwę struktury.
Nazwa struktury powinna odzwierciedlać znaczenie grupy danych znajdujących się w danej strukturze.
Następnie, w nawiasach klamrowych definiujemy nazwy i typy fragmentów danych, które nazywamy *atrybutami*. 
Na przykład, w listingu 5-1 widzimy strukturę, w której znajdują się przykładowe dane profilu użytkownika.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-01/src/main.rs:here}}
```

<span class="caption">Listing 5-1: `User` definicja struktury</span>

Aby wykorzystać strukturę po jej zdefiniowaniu tworzymy *instancję* tej struktury poprzez podanie konkretnych wartości dla każdego pola.
Tworzymy strukturę przez podanie jej nazwy, następnie nawiasy klamrowe zawierające pary *klucz: wartość*, gdzie klucze to nazwy pól, a wartości to dane, które chcemy umieścić w tych polach.
Nie musimy podawać atrybutów w tej samej kolejności w jakiej zostały one zdefiniowane
podczas deklaracji struktury.
Innymi słowy, definicja struktury jest ogólnym szablonem, a instancje jakby wypełniają
dany szablon jakimiś danymi tworząc wartości typu struktury.
Przykładowa deklaracja użytkownika pokazana jest w listingu 5-2.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-02/src/main.rs:here}}
```

<span class="caption">Listing 5-2: Tworzenie instancji struktury `User`</span>

Aby uzyskać dostęp dowybranej wartości ze struktury używamy kropki.
Na przykład, jeśli chcielibyśmy zdobyć tylko adres email użytkownika moglibyśmy napisać `user1.email` gdziekolwiek ta wartość byłaby nam potrzebna.
Jeśli instancja jest mutowalna możemy zmienić wartość używając kropki i przypisując do wybranego pola.
Listing 5-3 pokazuje jak zmienić pole `email` w mutowalnej instancji struktury `User`.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-03/src/main.rs:here}}
```

<span class="caption">Listing 5-3: Zmiana wartości pola `email` instancji struktury `User`.

Należy pamiętać, że cała instancja musi być mutowalna;
Rust nie pozwala nam zaznaczyć poszczególnych pól jako mutowalnych.
Jak z każdym wyrażeniem, możemy skonstruować nową instancję struktury jako
ostatnie wyrażenie w ciele funkcji, aby dana instancja została zwrócona przez funkcję.

Listing 5-4 ukazuje funkcję `build_user` zwracającą instancję struktury `User` z pewnym emailem i nazwą użytkownika.
Polu `active` przypisana jest wartość `true`, a polu `sign_in_count` przypisana jest wartość `1`.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-04/src/main.rs:here}}
```

<span class="caption">Listing 5-4: Funkcja `build_user`, która jako argument 
przyjmuje email i nazwę użytkownika, a zwraca instancję struktury `User`</span>

Nadanie parametrom funkcji tej samej nazwy co polom struktury ma sens, ale
przez to musimy powtarzać nazwy pól `email` i `username`, co jest trochę męczące.
Jeśli jakaś struktura miałaby więcej atrybutów powtarzanie każdego z nich
byłoby jeszcze bardziej męczące. Na szczęście, istnieje wygodny skrótowiec!

<!-- Old heading. Do not remove or links may break. -->
<a id="using-the-field-init-shorthand-when-variables-and-fields-have-the-same-name"></a>

### Skrócona inicjalizacja pól

Ponieważ nazwy parametrów i pól struktury są takie same, w listingu 5-4 możemy użyć składni tzw. *skróconej inicjializacji pól* (ang. field init shorthand), aby zmienić funkcję `build_user`, tak aby nie zmieniać jej zachowania, ale też nie powtarzając `username` i `email`. Taki zabieg widzimy w listingu 5-5.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-05/src/main.rs:here}}
```

<span class="caption">Listing 5-5: Funkcja `build_user` używająca skróconej inicjalizacji pól `username` oraz `email`, które mają takie same nazwy jak parametry funkcji</span>

Tutaj tworzymy nową instancję struktury `User`, która posiada pole o nazwie `email`. Chcemy nadać polu `email` wartość znajdującą się w parametrze `email` funkcji `build_user`. Skoro pole `email` i parametr `email` mają takie same nazwy wystarczy, że napiszemy `email` jedynie raz zamiast musieć napisać `email: email`.

### Tworzenie instancji z innej instancji przy użyciu składni zmiany struktury

Czasem bardzo przydatnym jest stworzenie nowej struktury, która jest w zasadzie
kopią innej struktury, w której chcemy zmienić tylko niektóre pola.
Do tego celu zastosujemy *składnię zmiany struktury*.

Listing 5-6 obrazuje tworzenie instancji struktury `User` zapisanej do zmiennej `user2` bez użycia naszej nowej składni. Nadajemy nowe wartości polom `email` i `username`, ale poza tym zostawiamy te same wartości w instancji `user1`, które przypisaliśmy w listingu 5-2.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-06/src/main.rs:here}}
```

<span class="caption">Listing 5-6: Tworzenie nowej instancji struktury `User` używając niektórych wartości z instancji `user1`</span>

Przy użyciu składni zmiany struktury możemy osiągnąć ten sam efekt mniejszym nakładem kodu,
co widzimy w listingu 5-7. Składnia `..` oznacza, że pozostałym polom, którym nie oznaczyliśmy ręcznie
wartości przypisane zostaną wartości z danej, oznaczonej instancji.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-07/src/main.rs:here}}
```

<span class="caption">Listing 5-7: Użycie składni zmiany struktury w celu przypisania w `User`
nowej wartości `email`, jednocześnie używając wartości pozostałych pól z `user1`</span>

Kod przedstawiony w listingu 5-7 tworzy też instancję w zmiennej `user2`, która
zmienia wartości w polach `email` i `username`, ale pozostawia wartości
w polach `active` i `sign_in_count` ze zmienną `user1`. `..user1` musi być na końcu,
aby określić, że wszystkie pozostałe pola powinny pobierać swoje wartości z
odpowiadających im pól w `user1`, ale możemy określić wartości dla dowolnej
liczby pól w dowolnej kolejności, niezależnie od kolejności pól w
definicji struktury.

Należy zauważyć, że składnia aktualizacji struktury używa `=` jak przypisania; dzieje się tak, ponieważ
przenosi ona dane, tak jak widzieliśmy w sekcji [„Zmienne i dane wchodzące w interakcję z
Move”][move]<!-- ignore -->. W tym przykładzie nie możemy już używać
`user1` jako całości po utworzeniu `user2`, ponieważ `String` w polu
`username` `user1` został przeniesiony do `user2`. Gdybyśmy podali `user2` nowe wartości
`String` zarówno dla `email`, jak i `username`, a zatem użyli tylko wartości
`active` i `sign_in_count` z `user1`, to `user1` nadal byłby
prawidłowy po utworzeniu `user2`. Zarówno `active`, jak i `sign_in_count` to typy, które
implementują cechę `Copy`, więc zachowanie omówione w sekcji [„Stack-Only
Data: Copy”][copy]<!-- ignore --> miałoby zastosowanie.

### Wykorzystanie braku nazywania pól w struktorach krotkowych do tworzenia nowych typów

Możesz też stworzyć struktury podobne do krotek, nazywane *struktorami krotkowymi* (ang. tuple structs).
Atutem strukturr krotkowych jest przypisanie znaczenia polom bez ich nazywania, a jedynie przez przypisanie polom ich typu.
Struktury krotkowe przydatne są najbardziej, kiedy: chciałbyś użyć krotkę, chcesz nadać jej nazwę i odróżnić ją od innych poprzez posiadanie innego typu, oraz kiedy nazywanie każdego pola (jak w normalnej strukturze) byłoby zbyt rozwlekłe lub zbędne.

Aby zdefiniować strukturę krotkową, najpierw wpisz słowo kluczowe `struct`, po nim nazwę struktury, a następnie typy w krotce.
Dla przykładu, tutaj pokazane są działania na dwóch strukturach-krotkach, tj. `Color` i `Point`:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-01-tuple-structs/src/main.rs}}
```

Zauważ, że `black` i `origin` mają różne typy, bo są instancjami różnych struktur krotkowych.
Każda zdefiniowana struktura ma własny, niepowtarzalny typ, nawet gdy pola w dwóch strukturach mają identyczne typy.
Na przykład, funkcja przyjmująca parametr typu `Color` nie może także przyjąć argumentu typu `Point`, mimo że np. oba typy mogą składać się z trzech wartości typu `i32`.
Oprócz tego wyjątku struktury krotkowe zachowują się całkiem jak krotki: można użyć składni przypisania destrukturyzującego aby przypisać pola zmiennym, a także indeksu pola poprzedzonego `.`, aby uzyskać do niego dostęp.

### Struktura jednostkowa bez żadnych pól

Możesz także definiować struktury nie posiadające żadnych pól!
Są to tzw. *struktury jednostkowe* (ang. unit-like structs), bo zachowują się podobnie do `()`, czyli typu jednostkowego wspomnianego w rozdziale [„Typ krotki”][tuples]<!-- ignore -->.
Struktury jednostkowe mogą być przydatne, gdy chcemy zaimplementować cechę na jakimś typie, ale nie potrzebujemy przechowywać żadnych danych. Cechy omawiamy w rozdziale 10.

Oto przykład deklaracji i utworzenia instancji struktury jednostki o nazwie `AlwaysEqual`:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-04-unit-like-structs/src/main.rs}}
```

Aby zdefiniować `AlwaysEqual`, używamy słowa kluczowego `struct`, nazwy, której chcemy, a
następnie średnika. Nie ma potrzeby stosowania nawiasów klamrowych ani okrągłych! Następnie możemy uzyskać
instancję `AlwaysEqual` w zmiennej `subject` w podobny sposób: używając
nazwy, którą zdefiniowaliśmy, bez żadnych nawiasów klamrowych ani okrągłych. Wyobraź sobie, że później
zaimplementujemy zachowanie dla tego typu tak, aby każda instancja
`AlwaysEqual` była zawsze równa każdej instancji dowolnego innego typu, być może w celu
otrzymania znanego wyniku do celów testowych. Nie potrzebowalibyśmy żadnych danych, aby
zaimplementować to zachowanie! W rozdziale 10 zobaczysz, jak definiować cechy i
implementować je w dowolnym typie, w tym w strukturach typu jednostkowego.


> ### Własność danych struktury
>
> W definicji struktury `User` w listingu 5-1 użyliśmy posiadanego typu `String`a
> zamiast wycinka łańcuchowego `&str`. To świadomy wybór, gdyż chcemy, aby instancje struktury posiadały wszystkie
> swoje dane oraz żeby te dane były ważne, jeśli sama struktura jest ważna.
>
> Struktury mogą przechowywać referencje do danych należących do czegoś innego,
> ale do tego potrzebne byłyby informacje o *długości życia* zmiennych (ang. lifetime).
> Jest to funkcja Rusta, o której powiemy więcej w rozdziale 10. 
> Długość życia gwarantuje nam, że dane wskazywane przez referencję
> są ważne dopóki struktura istnieje.
> Spróbujmy przechować referencję do struktury bez podania informacji
> o długości życia tak jak tutaj, co nie zadziała:
>
> <span class="filename">Nazwa pliku: src/main.rs</span>
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore,does_not_compile
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
>
> fn main() {
>     let user1 = User {
>         active: true,
>         username: "jakisusername123",
>         email: "ktos@example.com",
>         sign_in_count: 1,
>     };
> }
> ```
>
> Kompilator da ci znać, że potrzebuje specyfikatoru długości życia:
>
> ```console
> $ cargo run
>    Compiling struktury v0.1.0 (file:///projects/struktury)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
>
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
>
> For more information about this error, try `rustc --explain E0106`.
> error: could not compile `structs` due to 2 previous errors
> ```

>
> W rozdziale 10 pokażemy jak pozbyć się tych błędów, aby przechować
> referencje do innych struktur, ale póki co pozbędziemy się ich po prostu
> używając posiadanych typów, takich jak `String` zamiast referencji typu `&str`.

<!-- manual-regeneration
for the error above
after running update-rustc.sh:
pbcopy < listings/ch05-using-structs-to-structure-related-data/no-listing-02-reference-in-struct/output.txt
paste above
add `> ` before every line -->

[tuples]: ch03-02-data-types.html#krotki
[move]: ch04-01-what-is-ownership.html#przenoszenie-zmiennych-i-danych
[copy]: ch04-01-what-is-ownership.html#dane-przechowywane-wyłącznie-na-stosie-copy-kopiowanie
