## Akceptowanie argumentów wiersza poleceń

Utwórzmy nowy projekt, jak zwykle, z `cargo new`. Nazwiemy nasz projekt
`minigrep`, aby odróżnić go od narzędzia `grep`, które możesz już mieć
w swoim systemie.

```console
$ cargo new minigrep
     Utworzono projekt binarny (application) `minigrep`
$ cd minigrep
```

Pierwszym zadaniem jest sprawienie, aby `minigrep` akceptował dwa argumenty wiersza poleceń:
ścieżkę pliku i ciąg do wyszukania. To znaczy, chcemy móc uruchomić nasz
program za pomocą `cargo run`, dwóch myślników wskazujących, że poniższe argumenty są
dla naszego programu, a nie dla `cargo`, ciągu do wyszukania i ścieżki do pliku do wyszukania, w następujący sposób:

```console
$ cargo run -- searchstring example-filename.txt
```

W tej chwili program wygenerowany przez `cargo new` nie może przetworzyć argumentów, które mu
podajemy. Niektóre istniejące biblioteki na [crates.io](https://crates.io/) mogą pomóc
w napisaniu programu, który akceptuje argumenty wiersza poleceń, ale ponieważ dopiero uczysz się tej koncepcji, zaimplementujmy tę możliwość sami.
### Odczytywanie wartości argumentów

Aby umożliwić `minigrep` odczytywanie wartości argumentów wiersza poleceń, które do niego przekazujemy,
będziemy potrzebować funkcji `std::env::args` dostarczonej w standardowej
bibliotece Rust. Funkcja ta zwraca iterator argumentów wiersza poleceń przekazanych do `minigrep`. Iteratory zostaną omówione w pełni w [Rozdziale 13][ch13]<!-- ignore
-->. Na razie musisz znać tylko dwa szczegóły dotyczące iteratorów: iteratory
produkują szereg wartości, a my możemy wywołać metodę `collect` na iteratorze,
aby przekształcić go w kolekcję, taką jak wektor, która zawiera wszystkie elementy,
które iterator generuje.

Kod w Liście 12-1 umożliwia programowi `minigrep` odczytanie dowolnych
argumentów wiersza poleceń przekazanych do niego, a następnie zebranie wartości w wektorze.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-01/src/main.rs}}
```

<span class="caption">Listing 12-1: Zbieranie argumentów wiersza poleceń do
wektora i drukowanie ich</span>

Najpierw wprowadzamy moduł `std::env` do zakresu za pomocą polecenia `use`, abyśmy mogli
używać jego funkcji `args`. Zauważ, że funkcja `std::env::args` jest
zagnieżdżona w dwóch poziomach modułów. Jak omówiliśmy w [Rozdziale
7][ch7-idiomatic-use]<!-- ignore -->, w przypadkach, gdy pożądana funkcja jest
zagnieżdżona w więcej niż jednym module, zdecydowaliśmy się wprowadzić moduł nadrzędny do zakresu, a nie funkcję. Dzięki temu możemy łatwo używać innych funkcji
z `std::env`. Jest to również mniej dwuznaczne niż dodanie `use std::env::args`, a
następnie wywołanie funkcji za pomocą samego `args`, ponieważ `args` można łatwo pomylić z funkcją zdefiniowaną w bieżącym module.

> ### Funkcja `args` i nieprawidłowy kod Unicode
>
> Należy zauważyć, że `std::env::args` wpadnie w panikę, jeśli którykolwiek argument będzie zawierał nieprawidłowy kod Unicode
>. Jeśli program musi zaakceptować argumenty zawierające nieprawidłowy kod Unicode
>, użyj zamiast tego `std::env::args_os`. Ta funkcja zwraca iterator,
> który generuje wartości `OsString` zamiast wartości `String`. Wybraliśmy tutaj
> użycie `std::env::args` dla uproszczenia, ponieważ wartości `OsString` różnią się
> w zależności od platformy i są bardziej skomplikowane w obsłudze niż wartości `String`.

W pierwszym wierszu `main` wywołujemy `env::args` i natychmiast używamy
`collect`, aby przekształcić iterator w wektor zawierający wszystkie wartości wygenerowane
przez iterator. Możemy użyć funkcji `collect`, aby utworzyć wiele rodzajów
kolekcji, więc jawnie adnotujemy typ `args`, aby określić, że
chcemy wektora ciągów. Chociaż bardzo rzadko musimy adnotować typy w
Rust, `collect` to jedna z funkcji, którą często trzeba adnotować, ponieważ Rust
nie jest w stanie wywnioskować, jakiego rodzaju kolekcji chcesz.

Na koniec drukujemy wektor za pomocą makra debugowania. Spróbujmy uruchomić kod
najpierw bez argumentów, a następnie z dwoma argumentami:

```console
{{#include ../listings/ch12-an-io-project/listing-12-01/output.txt}}
```

```console
{{#include ../listings/ch12-an-io-project/output-only-01-with-args/output.txt}}
```

Zauważ, że pierwszą wartością w wektorze jest `"target/debug/minigrep"`, która
jest nazwą naszego pliku binarnego. Pasuje to do zachowania listy argumentów w
C, pozwalając programom używać nazwy, pod którą zostały wywołane podczas wykonywania.
Często wygodnie jest mieć dostęp do nazwy programu, na wypadek gdybyś chciał
wyświetlić ją w wiadomościach lub zmienić zachowanie programu na podstawie tego,
jaki alias wiersza poleceń został użyty do wywołania programu. Jednak na potrzeby tego
rozdziału zignorujemy to i zapiszemy tylko dwa potrzebne nam argumenty.

### Zapisywanie wartości argumentów w zmiennych

Program jest obecnie w stanie uzyskać dostęp do wartości określonych jako argumenty wiersza poleceń. Teraz musimy zapisać wartości dwóch argumentów w zmiennych, abyśmy mogli używać tych wartości w pozostałej części programu. Robimy to w Liście
12-2.

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-02/src/main.rs}}
```

<span class="caption">Listing 12-2: Tworzenie zmiennych do przechowywania argumentu zapytania i argumentu ścieżki pliku</span>

Jak widzieliśmy, drukując wektor, nazwa programu zajmuje pierwszą
wartość w wektorze w `args[0]`, więc zaczynamy argumenty od indeksu `1`.
Pierwszy argument `minigrep` zajmuje ciąg, którego szukamy, więc umieszczamy
odniesienie do pierwszego argumentu w zmiennej `query`. Drugim argumentem
będzie ścieżka do pliku, więc umieszczamy odniesienie do drugiego argumentu w zmiennej `file_path`.

Tymczasowo drukujemy wartości tych zmiennych, aby udowodnić, że kod
działa tak, jak zamierzyliśmy. Uruchommy ten program ponownie z argumentami `test`
i `sample.txt`:

```console
{{#include ../listings/ch12-an-io-project/listing-12-02/output.txt}}
```

Świetnie, program działa! Wartości argumentów, których potrzebujemy, są
zapisywane w odpowiednich zmiennych. Później dodamy trochę obsługi błędów, aby poradzić sobie
z pewnymi potencjalnie błędnymi sytuacjami, takimi jak sytuacja, gdy użytkownik nie poda żadnych
argumentów; na razie zignorujemy tę sytuację i zamiast tego zajmiemy się dodaniem możliwości
odczytu plików.
[ch13]: ch13-00-functional-features.html
[ch7-idiomatic-use]: ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#tworzenie-idiomatycznych-Ścieżek-use
