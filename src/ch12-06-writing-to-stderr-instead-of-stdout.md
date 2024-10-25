## Zapisywanie komunikatów o błędach na standardowym wyjściu zamiast na standardowym wyjściu

W tej chwili zapisujemy wszystkie nasze dane wyjściowe do terminala za pomocą makra
`println!`. W większości terminali istnieją dwa rodzaje danych wyjściowych: *standardowe
wyjście* (`stdout`) dla ogólnych informacji i *standardowy błąd* (`stderr`) dla
komunikatów o błędach. To rozróżnienie umożliwia użytkownikom wybór skierowania
pomyślnego wyjścia programu do pliku, ale nadal drukowania komunikatów o błędach na
ekranie.

Makro `println!` może drukować tylko do standardowego wyjścia, więc musimy
użyć czegoś innego do drukowania do standardowego błędu.

### Checking Where Errors Are Written

Najpierw zauważmy, jak zawartość drukowana przez `minigrep` jest obecnie
zapisywana na standardowe wyjście, w tym wszelkie komunikaty o błędach, które chcemy zapisać
na standardowym wyjściu. Zrobimy to, przekierowując standardowy strumień wyjściowy
do pliku, celowo powodując błąd. Nie przekierujemy standardowego strumienia
błędów, więc wszelka zawartość wysyłana na standardowy błąd będzie nadal wyświetlana
na ekranie.

Oczekuje się, że programy wiersza poleceń będą wysyłać komunikaty o błędach do standardowego strumienia
błędów, abyśmy nadal mogli widzieć komunikaty o błędach na ekranie, nawet jeśli przekierujemy
standardowy strumień wyjściowy do pliku. Nasz program nie zachowuje się obecnie dobrze:
zaraz zobaczymy, że zapisuje on zamiast tego wyjście komunikatu o błędzie do pliku!

Aby zademonstrować to zachowanie, uruchomimy program z `>` i ścieżką do pliku,
*output.txt*, do którego chcemy przekierować standardowy strumień wyjściowy. Nie przekażemy żadnych argumentów, co powinno spowodować błąd:

```console
$ cargo run > output.txt
```

Składnia `>` mówi powłoce, aby zapisała zawartość standardowego wyjścia do
*output.txt* zamiast na ekran. Nie zobaczyliśmy oczekiwanego komunikatu o błędzie,
więc musiał on trafić do pliku. Oto, co zawiera *output.txt*:

```text
Problem z analizą argumentów: za mało argumentów
```

Tak, nasz komunikat o błędzie jest drukowany na standardowym wyjściu. Znacznie bardziej
przydatne jest drukowanie takich komunikatów o błędach na standardowym wyjściu, aby tylko
dane z pomyślnego uruchomienia trafiały do ​​pliku. Zmienimy to.

### Printing Errors to Standard Error

Użyjemy kodu z Listingu 12-24, aby zmienić sposób drukowania komunikatów o błędach.
Ze względu na refaktoryzację, którą wykonaliśmy wcześniej w tym rozdziale, cały kod, który
drukuje komunikaty o błędach, znajduje się w jednej funkcji, `main`. Biblioteka standardowa udostępnia
makro `eprintln!`, które drukuje do standardowego strumienia błędów, więc zmieńmy
dwa miejsca, w których wywoływaliśmy `println!`, aby drukować błędy, aby zamiast tego użyć `eprintln!`.

<Listing number="12-24" file-name="src/main.rs" caption="Writing error messages to standard error instead of standard output using `eprintln!`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-24/src/main.rs:here}}
```

</Listing>

Uruchommy teraz program ponownie w ten sam sposób, bez żadnych argumentów i
przekierowując standardowe wyjście za pomocą `>`:

```console
$ cargo run > output.txt
Problem z analizą argumentów: za mało argumentów
```

Teraz widzimy błąd na ekranie, a *output.txt* nie zawiera niczego, czego
oczekujemy od programów wiersza poleceń.

Uruchommy program ponownie z argumentami, które nie powodują błędu, ale nadal
przekierowują standardowe wyjście do pliku, w następujący sposób:

```console
$ cargo run -- to poem.txt > output.txt
```

Nie zobaczymy żadnego wyjścia w terminalu, a *output.txt* będzie zawierał nasze
wyniki:
<span class="filename">Filename: output.txt</span>

```text
Czy ty też jesteś nikim?
Jak nudno być kimś!
```

To pokazuje, że teraz używamy standardowego wyjścia do udanego wyjścia
i standardowego błędu do wyjścia błędu, jeśli jest to właściwe.


## Summary

W tym rozdziale podsumowano niektóre z głównych koncepcji, których się do tej pory nauczyłeś i
omówiono, jak wykonywać typowe operacje wejścia/wyjścia w Rust. Korzystając z argumentów wiersza poleceń, plików, zmiennych środowiskowych i makra `eprintln!` do drukowania
błędów, jesteś teraz przygotowany do pisania aplikacji wiersza poleceń. W połączeniu z
koncepcjami z poprzednich rozdziałów, Twój kod będzie dobrze zorganizowany, będzie skutecznie przechowywał dane w odpowiednich strukturach danych, będzie dobrze obsługiwał błędy i będzie dobrze
testowany.

Następnie przyjrzymy się niektórym funkcjom Rust, na które wpływ miały języki
funkcyjne: zamknięcia i iteratory.

