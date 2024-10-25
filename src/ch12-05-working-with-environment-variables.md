## Praca ze zmiennymi środowiskowymi

Ulepszymy `minigrep`, dodając dodatkową funkcję: opcję wyszukiwania bez uwzględniania wielkości liter, którą użytkownik może włączyć za pomocą zmiennej środowiskowej. Możemy uczynić tę funkcję opcją wiersza poleceń i wymagać, aby
użytkownicy wprowadzali ją za każdym razem, gdy chcą, aby została zastosowana, ale zamiast tego czyniąc ją
zmienną środowiskową, pozwalamy naszym użytkownikom ustawić zmienną środowiskową raz,
aby wszystkie ich wyszukiwania były bez uwzględniania wielkości liter w tej sesji terminala.

### Writing a Failing Test for the Case-Insensitive `search` Function

Najpierw dodamy nową funkcję `search_case_insensitive`, która zostanie wywołana, gdy
zmienna środowiskowa będzie miała wartość. Będziemy nadal postępować zgodnie z procesem TDD,
więc pierwszym krokiem jest ponowne napisanie nieudanego testu. Dodamy nowy test dla
nowej funkcji `search_case_insensitive` i zmienimy nazwę naszego starego testu z
`one_result` na `case_sensitive`, aby wyjaśnić różnice między tymi dwoma
testami, jak pokazano w Liście 12-20.

<Listing number="12-20" file-name="src/lib.rs" caption="Dodawanie nowego nieudanego testu dla funkcji nieuwzględniającej wielkości liter, którą zamierzamy dodać">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-20/src/lib.rs:here}}
```

</Listing>

Zauważ, że edytowaliśmy również `contents` starego testu. Dodaliśmy nowy wiersz
z tekstem `"Duct tape."` używając wielkiej litery *D*, który nie powinien pasować do zapytania
`"duct"`, gdy przeszukujemy w sposób uwzględniający wielkość liter. Zmiana starego testu
w ten sposób pomaga upewnić się, że przypadkowo nie zepsujemy funkcjonalności wyszukiwania uwzględniającego wielkość liter, którą już zaimplementowaliśmy. Ten test powinien teraz przejść
i powinien nadal przechodzić, gdy będziemy pracować nad wyszukiwaniem bez uwzględniania wielkości liter.

Nowy test wyszukiwania bez uwzględniania wielkości liter używa `"rUsT"` jako swojego zapytania. W
funkcji `search_case_insensitive`, którą zamierzamy dodać, zapytanie `"rUsT"`
powinno pasować do wiersza zawierającego `"Rust:"` z wielkiej litery *R* i pasować do wiersza
`"Trust me."`, mimo że oba mają inną wielkość liter niż zapytanie. To
jest nasz nieudany test i nie skompiluje się, ponieważ nie zdefiniowaliśmy jeszcze
funkcji `search_case_insensitive`. Możesz dodać szkieletową implementację, która zawsze zwraca pusty wektor, podobnie jak zrobiliśmy to
dla funkcji `search` w Liście 12-16, aby zobaczyć, jak test się kompiluje i kończy niepowodzeniem.

### Implementing the `search_case_insensitive` Function

Funkcja `search_case_insensitive`, pokazana w Liście 12-21, będzie prawie taka sama jak funkcja `search`. Jedyną różnicą jest to, że zapiszemy małą literą
`query` i każdy `line`, tak aby niezależnie od wielkości liter argumentów wejściowych,
były one takie same, gdy sprawdzimy, czy wiersz zawiera zapytanie.

<Listing number="12-21" file-name="src/lib.rs" caption="Defining the `search_case_insensitive` function to lowercase the query and the line before comparing them">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-21/src/lib.rs:here}}
```

</Listing>

Najpierw zapisujemy ciąg `query` małymi literami i zapisujemy go w zmiennej o tej samej nazwie. Wywołanie `to_lowercase` w zapytaniu jest konieczne, aby niezależnie od tego, czy zapytanie użytkownika to `"rust"`, `"RUST"`, `"Rust"` czy `"rUsT"`,
traktować zapytanie tak, jakby było `"rust"` i nie zwracać uwagi na wielkość liter.
Chociaż `to_lowercase` obsłuży podstawowy Unicode, nie będzie w 100% dokładne. Gdybyśmy
pisali prawdziwą aplikację, chcielibyśmy wykonać tutaj trochę więcej pracy, ale
ta sekcja dotyczy zmiennych środowiskowych, a nie Unicode, więc tutaj
na tym poprzestaniemy.

Należy zauważyć, że `query` jest teraz `String`, a nie wycinkiem ciągu, ponieważ wywołanie
`to_lowercase` tworzy nowe dane, a nie odwołuje się do istniejących danych. Załóżmy, że
zapytanie to `"rUsT"`, na przykład: ten fragment ciągu nie zawiera małej litery
`u` ani `t`, których moglibyśmy użyć, więc musimy przydzielić nowy `String` zawierający
`"rust"`. Kiedy teraz przekazujemy `query` jako argument do metody `contains`,
musimy dodać ampersand, ponieważ sygnatura `contains` jest zdefiniowana tak, aby przyjmować
fragment ciągu.

Następnie dodajemy wywołanie `to_lowercase` w każdym `line`, aby wszystkie
znaki były pisane małymi literami. Teraz, gdy przekonwertowaliśmy `line` i `query` na małe litery,
znajdziemy dopasowania niezależnie od wielkości liter w zapytaniu.

Zobaczmy, czy ta implementacja przejdzie testy:

```console
{{#include ../listings/ch12-an-io-project/listing-12-21/output.txt}}
```

Świetnie! Przeszli. Teraz wywołajmy nową funkcję `search_case_insensitive`
z funkcji `run`. Najpierw dodamy opcję konfiguracji do struktury `Config`, aby przełączać się między wyszukiwaniem uwzględniającym wielkość liter i wyszukiwaniem bez uwzględniania wielkości liter. Dodanie
tego pola spowoduje błędy kompilatora, ponieważ nie inicjalizujemy tego pola
nigdzie jeszcze:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/lib.rs:here}}
```

Dodaliśmy pole `ignore_case`, które zawiera wartość logiczną. Następnie potrzebujemy funkcji `run`, aby sprawdzić wartość pola `ignore_case` i użyć jej do podjęcia decyzji,
czy wywołać funkcję `search` lub `search_case_insensitive`,
jak pokazano na liście 12-22. To nadal nie zostanie skompilowane.

<Listing number="12-22" file-name="src/lib.rs" caption="Calling either `search` or `search_case_insensitive` based on the value in `config.ignore_case`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/lib.rs:there}}
```

</Listing>

Na koniec musimy sprawdzić zmienną środowiskową. Funkcje do
pracy ze zmiennymi środowiskowymi znajdują się w module `env` w standardowej
bibliotece, więc przenosimy ten moduł do zakresu na górze *src/lib.rs*. Następnie
użyjemy funkcji `var` z modułu `env`, aby sprawdzić, czy jakaś wartość
została ustawiona dla zmiennej środowiskowej o nazwie `IGNORE_CASE`, jak pokazano w
Listingu 12-23.

<Listing number="12-23" file-name="src/lib.rs" caption="Checking for any value in an environment variable named `IGNORE_CASE`">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-23/src/lib.rs:here}}
```

</Listing>

Tutaj tworzymy nową zmienną, `ignore_case`. Aby ustawić jej wartość, wywołujemy funkcję
`env::var` i przekazujemy jej nazwę zmiennej środowiskowej `IGNORE_CASE`. Funkcja `env::var` zwraca `Result`, która będzie
udaną odmianą `Ok` zawierającą wartość zmiennej środowiskowej, jeśli
zmienna środowiskowa jest ustawiona na dowolną wartość. Zwróci odmianę `Err`,
jeśli zmienna środowiskowa nie jest ustawiona.

Używamy metody `is_ok` w `Result`, aby sprawdzić, czy zmienna środowiskowa
jest ustawiona, co oznacza, że ​​program powinien wykonać wyszukiwanie bez uwzględniania wielkości liter.
Jeśli zmienna środowiskowa `IGNORE_CASE` nie jest ustawiona na nic, `is_ok`
zwróci `false`, a program wykona wyszukiwanie z uwzględnieniem wielkości liter. Nie
interesuje nas *wartość* zmiennej środowiskowej, tylko to, czy jest ustawiona czy nie, więc sprawdzamy `is_ok` zamiast używać `unwrap`, `expect` lub jakiejkolwiek innej metody, którą widzieliśmy w `Result`.

Przekazujemy wartość zmiennej `ignore_case` do instancji `Config`, aby funkcja
`run` mogła odczytać tę wartość i zdecydować, czy wywołać
`search_case_insensitive` czy `search`, jak zaimplementowaliśmy w Listingu 12-22.

Spróbujmy! Najpierw uruchomimy nasz program bez ustawionej zmiennej środowiskowej i z zapytaniem `to`, które powinno pasować do każdego wiersza zawierającego
słowo *to* w całości małymi literami:

```console
{{#include ../listings/ch12-an-io-project/listing-12-23/output.txt}}
```
Wygląda na to, że to nadal działa! Teraz uruchommy program z `IGNORE_CASE` set
na `1`, ale z tym samym zapytaniem *to*:

```console
$ IGNORE_CASE=1 cargo run -- to poem.txt
```

Jeśli używasz programu PowerShell, musisz ustawić zmienną środowiskową i
uruchomić program jako osobne polecenia:

```console
PS> $Env:IGNORE_CASE=1; cargo run -- to poem.txt
```

Spowoduje to, że `IGNORE_CASE` będzie trwało do końca sesji powłoki.
Można je usunąć za pomocą polecenia cmdlet `Remove-Item`:

```console
PS> Remove-Item Env:IGNORE_CASE
```

Powinniśmy otrzymać wiersze zawierające *to*, które mogą mieć wielkie litery:
<!-- manual-regeneration
cd listings/ch12-an-io-project/listing-12-23
IGNORE_CASE=1 cargo run -- to poem.txt
can't extract because of the environment variable
-->

```console
Czy ty też jesteś nikim?
Jak ponuro być kimś!
Mówić swoje imię przez cały dzień
Do podziwiającego bagna!
```

Świetnie, mamy też wiersze zawierające *Do*! Nasz program `minigrep` może teraz wykonywać
wyszukiwanie bez uwzględniania wielkości liter kontrolowane przez zmienną środowiskową. Teraz wiesz,
jak zarządzać opcjami ustawionymi za pomocą argumentów wiersza poleceń lub zmiennych
środowiskowych.

Niektóre programy zezwalają na argumenty *i* zmienne środowiskowe dla tej samej
konfiguracji. W takich przypadkach programy decydują, że jeden lub drugi ma pierwszeństwo. W innym ćwiczeniu do samodzielnego wykonania spróbuj kontrolować uwzględnianie wielkości liter
za pomocą argumentu wiersza poleceń lub zmiennej środowiskowej. Zdecyduj,
czy argument wiersza poleceń lub zmienna środowiskowa powinny mieć pierwszeństwo,
jeśli program jest uruchamiany z jednym ustawionym na uwzględnianie wielkości liter, a drugim na ignorowanie wielkości liter.

Moduł `std::env` zawiera wiele innych przydatnych funkcji do obsługi
zmiennych środowiskowych: sprawdź jego dokumentację, aby zobaczyć, co jest dostępne.
