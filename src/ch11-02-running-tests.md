## Kontrolowanie sposobu uruchamiania testów

Podobnie jak `cargo run` kompiluje kod, a następnie uruchamia wynikowy plik binarny,
`cargo test` kompiluje kod w trybie testowym i uruchamia wynikowy plik binarny
testu. Domyślnym zachowaniem pliku binarnego wygenerowanego przez `cargo test` jest uruchamianie
wszystkich testów równolegle i przechwytywanie danych wyjściowych generowanych podczas przebiegów testów,
zapobiegając wyświetlaniu danych wyjściowych i ułatwiając odczyt
danych wyjściowych związanych z wynikami testu. Możesz jednak określić opcje wiersza poleceń,
aby zmienić to domyślne zachowanie.

Niektóre opcje wiersza poleceń przechodzą do `cargo test`, a niektóre do wynikowego pliku binarnego
testu. Aby oddzielić te dwa typy argumentów, należy wymienić argumenty,
które przechodzą do `cargo test`, a następnie separator `--`, a następnie te, które przechodzą do
pliku binarnego testu. Uruchomienie `cargo test --help` wyświetla opcje,
których można użyć z `cargo test`, a uruchomienie `cargo test -- --help` wyświetla opcje,
których można użyć po separatorze.

### Uruchamianie testów równolegle lub kolejno

Gdy uruchamiasz wiele testów, domyślnie są one uruchamiane równolegle przy użyciu wątków,
co oznacza, że ​​kończą się szybciej i otrzymujesz szybsze informacje zwrotne. Ponieważ
testy są uruchamiane w tym samym czasie, musisz upewnić się, że Twoje testy nie zależą
od siebie ani od żadnego współdzielonego stanu, w tym współdzielonego środowiska, takiego jak
bieżący katalog roboczy lub zmienne środowiskowe.

Na przykład, powiedzmy, że każdy z Twoich testów uruchamia kod, który tworzy plik na dysku
o nazwie *test-output.txt* i zapisuje pewne dane do tego pliku. Następnie każdy test odczytuje
dane z tego pliku i potwierdza, że ​​plik zawiera określoną wartość,
która jest inna w każdym teście. Ponieważ testy są uruchamiane w tym samym czasie, jeden
test może nadpisać plik w czasie między zapisaniem i
odczytaniem pliku przez inny test. Drugi test zakończy się niepowodzeniem, nie dlatego, że kod jest
niepoprawny, ale dlatego, że testy kolidowały ze sobą podczas uruchamiania
równolegle. Jednym z rozwiązań jest upewnienie się, że każdy test zapisuje do innego pliku;
innym rozwiązaniem jest uruchomienie testów pojedynczo.

Jeśli nie chcesz uruchamiać testów równolegle lub chcesz mieć bardziej szczegółową
kontrolę nad liczbą używanych wątków, możesz wysłać flagę `--test-threads`
i liczbę wątków, których chcesz użyć, do pliku binarnego testu. Spójrz na
następujący przykład:

```console
$ cargo test -- --test-threads=1
```
Ustawiliśmy liczbę wątków testowych na `1`, informując program, aby nie używał żadnego
paralelizmu. Uruchomienie testów przy użyciu jednego wątku zajmie więcej czasu niż uruchomienie ich
równolegle, ale testy nie będą ze sobą kolidować, jeśli będą współdzieliły
stan.

### Wyświetlanie wyników funkcji

Domyślnie, jeśli test przejdzie, biblioteka testów Rust przechwytuje wszystko, co jest drukowane na
standardowym wyjściu. Na przykład, jeśli wywołamy `println!` w teście i test
przejdzie, nie zobaczymy wyjścia `println!` w terminalu; zobaczymy tylko
linię, która wskazuje, że test przeszedł. Jeśli test się nie powiedzie, zobaczymy wszystko, co zostało
wydrukowane na standardowym wyjściu wraz z resztą komunikatu o błędzie.

Na przykład, Listing 11-10 zawiera głupią funkcję, która drukuje wartość swojego
parametru i zwraca 10, a także test, który przechodzi i test, który nie przechodzi.

<Listing number="11-10" file-name="src/lib.rs" caption="Tests for a function that calls `println!`">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-10/src/lib.rs}}
```

</Listing>

Gdy uruchomimy te testy za pomocą polecenia `cargo test`, zobaczymy następujący wynik:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-10/output.txt}}
```

Zauważ, że nigdzie w tym wyjściu nie widzimy `I got the value 4`, które jest
wyświetlane, gdy test, który się powiódł, zostanie uruchomiony. To wyjście zostało przechwycone.
Dane wyjściowe z testu, który się nie powiódł, `I got the value 8`, pojawiają się w sekcji
wyjścia podsumowania testu, która pokazuje również przyczynę niepowodzenia testu.

Jeśli chcemy zobaczyć wydrukowane wartości dla testów, które się powiodły, możemy powiedzieć Rust, aby
również wyświetlał dane wyjściowe dla testów, które się powiodły, za pomocą `--show-output`:

```console
$ cargo test -- --show-output
```

Gdy ponownie uruchomimy testy z Listingu 11-10 z flagą `--show-output`, zobaczymy następujący wynik:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-01-show-output/output.txt}}
```

### Uruchamianie podzbioru testów według nazwy

Czasami uruchomienie pełnego zestawu testów może zająć dużo czasu. Jeśli pracujesz nad
kodem w określonym obszarze, możesz chcieć uruchomić tylko testy odnoszące się do
tego kodu. Możesz wybrać, które testy uruchomić, przekazując `cargo test` nazwę
lub nazwy testów, które chcesz uruchomić jako argument.

Aby zademonstrować, jak uruchomić podzbiór testów, najpierw utworzymy trzy testy dla
naszej funkcji `add_two`, jak pokazano na Liście 11-11, i wybierzemy, które z nich uruchomić.

<Listing number="11-11" file-name="src/lib.rs" caption="Three tests with three different names">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-11/src/lib.rs}}
```

</Listing>

Jeśli uruchomimy testy bez przekazywania żadnych argumentów, jak widzieliśmy wcześniej, wszystkie testy zostaną uruchomione równolegle:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-11/output.txt}}
```

#### Uruchamianie pojedynczych testów

Możemy przekazać nazwę dowolnej funkcji testowej do `cargo test`, aby uruchomić tylko ten test:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-02-single-test/output.txt}}
```

Uruchomiono tylko test o nazwie `one_hundred`; pozostałe dwa testy nie pasowały
do tej nazwy. Wynik testu informuje nas, że mieliśmy więcej testów, które nie zostały uruchomione, poprzez
wyświetlenie na końcu `2 filtered out`.

Nie możemy w ten sposób określić nazw wielu testów; zostanie użyta tylko pierwsza wartość
podana dla `cargo test`. Istnieje jednak sposób na uruchomienie wielu testów.

#### Filtrowanie w celu uruchomienia wielu testów

Możemy określić część nazwy testu, a każdy test, którego nazwa pasuje do tej wartości,
zostanie uruchomiony. Na przykład, ponieważ nazwy dwóch naszych testów zawierają `add`, możemy
uruchomić te dwa, uruchamiając `cargo test add`:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-03-multiple-tests/output.txt}}
```

To polecenie uruchomiło wszystkie testy z `add` w nazwie i odfiltrowało test o nazwie `one_hundred`. Należy również zauważyć, że moduł, w którym pojawia się test, staje się częścią nazwy testu, więc możemy uruchomić wszystkie testy w module, filtrując
nazwę modułu.

### Ignorowanie niektórych testów, chyba że wyraźnie o nie poproszono

Czasami wykonanie kilku konkretnych testów może być bardzo czasochłonne, więc możesz chcieć je wykluczyć podczas większości przebiegów `cargo test`. Zamiast
wymieniać jako argumenty wszystkie testy, które chcesz uruchomić, możesz zamiast tego oznaczyć adnotacją testy, które wymagają dużo czasu, używając atrybutu `ignore`, aby je wykluczyć, jak pokazano
tutaj:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/src/lib.rs:here}}
```

Po `#[test]` dodajemy linię `#[ignore]` do testu, który chcemy wykluczyć.
Teraz, gdy uruchamiamy nasze testy, `it_works` działa, ale `expensive_test` nie:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/output.txt}}
```

Funkcja `expensive_test` jest wymieniona jako `ignored`. Jeśli chcemy uruchomić tylko ignorowane testy, możemy użyć `cargo test -- --ignored`:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-04-running-ignored/output.txt}}
```

Kontrolując, które testy są uruchamiane, możesz mieć pewność, że wyniki `cargo test`
zostaną zwrócone szybko. Kiedy dojdziesz do punktu, w którym ma sens sprawdzenie
wyników `ignored` testów i masz czas, aby poczekać na wyniki,
możesz zamiast tego uruchomić `cargo test -- --ignored`. Jeśli chcesz uruchomić wszystkie testy,
niezależnie od tego, czy są ignorowane, czy nie, możesz uruchomić `cargo test -- --include-ignored`.
