## Organizacja testów

Jak wspomniano na początku rozdziału, testowanie jest złożoną dyscypliną, a
różne osoby używają różnej terminologii i organizacji. Społeczność Rust
myśli o testach w kategoriach dwóch głównych kategorii: testów jednostkowych i testów integracyjnych. *Testy jednostkowe* są małe i bardziej ukierunkowane, testują jeden moduł w izolacji
na raz i mogą testować prywatne interfejsy. *Testy integracyjne* są całkowicie
zewnętrzne w stosunku do biblioteki i wykorzystują kod w taki sam sposób, jak każdy inny zewnętrzny
kod, używając tylko publicznego interfejsu i potencjalnie wykonując wiele
modułów na test.

Pisanie obu rodzajów testów jest ważne, aby upewnić się, że elementy biblioteki
robią to, czego od nich oczekujesz, oddzielnie i razem.
### Unit Tests

Celem testów jednostkowych jest przetestowanie każdej jednostki kodu w izolacji od
reszty kodu, aby szybko ustalić, gdzie kod działa, a gdzie nie, zgodnie z
oczekiwaniami. Testy jednostkowe należy umieścić w katalogu *src* w każdym pliku z
kodem, który testują. Przyjętą konwencją jest utworzenie modułu o nazwie `tests`
w każdym pliku, aby zawierał funkcje testowe i adnotację modułu za pomocą
`cfg(test)`.

#### Testy Module i `#[cfg(test)]`

Adnotacja `#[cfg(test)]` w module tests mówi Rustowi, aby skompilował i uruchomił
kod testowy tylko wtedy, gdy uruchamiasz `cargo test`, a nie gdy uruchamiasz `cargo build`.
Oszczędza to czas kompilacji, gdy chcesz tylko skompilować bibliotekę i oszczędza miejsce
w skompilowanym artefakcie, ponieważ testy nie są uwzględniane.
Zobaczysz, że ponieważ testy integracyjne znajdują się w innym katalogu, nie potrzebują
adnotacji `#[cfg(test)]`. Jednak ponieważ testy jednostkowe znajdują się w tych samych plikach,
co kod, użyjesz `#[cfg(test)]`, aby określić, że nie powinny być
uwzględniane w skompilowanym wyniku.

Przypomnij sobie, że gdy wygenerowaliśmy nowy projekt `adder` w pierwszej sekcji
tego rozdziału, Cargo wygenerowało dla nas ten kod:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

Ten kod to automatycznie generowany moduł testowy. Atrybut `cfg`
oznacza *configuration* i informuje Rust, że następujący element powinien być uwzględniony tylko
przy określonej opcji konfiguracji. W tym przypadku opcją konfiguracji jest `test`, która jest dostarczana przez Rust do kompilowania i
uruchamiania testów. Używając atrybutu `cfg`, Cargo kompiluje nasz kod testowy tylko
jeśli aktywnie uruchamiamy testy za pomocą `cargo test`. Obejmuje to wszystkie funkcje pomocnicze,
które mogą znajdować się w tym module, oprócz funkcji
oznaczonych adnotacją `#[test]`.

#### Testing Private Funkcje

W społeczności testerów toczy się debata na temat tego, czy prywatne
funkcje powinny być testowane bezpośrednio, a inne języki utrudniają lub uniemożliwiają testowanie prywatnych funkcji. Niezależnie od tego, której ideologii testowania przestrzegasz,
zasady prywatności Rusta pozwalają na testowanie prywatnych funkcji.
Rozważ kod z Listingu 11-12 z prywatną funkcją `internal_adder`.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-12/src/lib.rs}}
```

<span class="caption">Listing 11-12: Testing a private function</span>

Zwróć uwagę, że funkcja `internal_adder` nie jest oznaczona jako `pub`. Testy to po prostu
kod Rust, a moduł `tests` to po prostu kolejny moduł. Jak omówiliśmy w
sekcji [“Paths for Referring to an Item in the Module Tree”][paths]<!-- ignore -->
, elementy w modułach potomnych mogą używać elementów w modułach przodków. W
tym teście przenosimy wszystkie elementy rodzica modułu `test` do zakresu za pomocą
`use super::*`, a następnie test może wywołać `internal_adder`. Jeśli uważasz, że
prywatne funkcje nie powinny być testowane, w Rust nie ma niczego, co by Cię do tego zmusiło.

### Integration Tests

W Rust testy integracyjne są całkowicie zewnętrzne w stosunku do biblioteki. Używają biblioteki
w taki sam sposób, w jaki używałby jej każdy inny kod, co oznacza, że ​​mogą wywoływać tylko funkcje, które są częścią publicznego API biblioteki. Ich celem jest testowanie,
czy wiele części biblioteki działa poprawnie. Jednostki kodu,
które działają poprawnie same w sobie, mogą mieć problemy po zintegrowaniu, więc pokrycie testowe zintegrowanego kodu jest również ważne. Aby utworzyć testy integracyjne, najpierw potrzebujesz katalogu *tests*.

#### The *tests* Directory

Tworzymy katalog *tests* na najwyższym poziomie katalogu naszego projektu, obok
*src*. Cargo wie, że w tym katalogu należy szukać plików testów integracyjnych. Następnie możemy utworzyć tyle plików testowych, ile chcemy, a Cargo skompiluje każdy z nich jako osobną skrzynkę.

Utwórzmy test integracyjny. Mając kod z Listingu 11-12 nadal w pliku
*src/lib.rs*, utwórz katalog *tests* i utwórz nowy plik o nazwie
*tests/integration_test.rs*. Twoja struktura katalogów powinna wyglądać następująco:

```text
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

Enter the code in Listing 11-13 into the *tests/integration_test.rs* file:

<span class="filename">Filename: tests/integration_test.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-13/tests/integration_test.rs}}
```

<span class="caption">Listing 11-13: An integration test of a function in the
`adder` crate</span>

Każdy plik w katalogu `tests` jest osobną skrzynią, więc musimy przenieść naszą
bibliotekę do zakresu każdej skrzyni testowej. Z tego powodu dodajemy `use adder` na
górę kodu, czego nie potrzebowaliśmy w testach jednostkowych.

Nie musimy adnotować żadnego kodu w *tests/integration_test.rs* za pomocą
`#[cfg(test)]`. Cargo traktuje katalog `tests` specjalnie i kompiluje pliki
w tym katalogu tylko wtedy, gdy uruchamiamy `cargo test`. Uruchom teraz `cargo test`:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-13/output.txt}}
```

Trzy sekcje wyników obejmują testy jednostkowe, test integracyjny i
testy dokumentacji. Należy pamiętać, że jeśli jakikolwiek test w sekcji się nie powiedzie, kolejne sekcje
nie zostaną uruchomione. Na przykład, jeśli test jednostkowy się nie powiedzie, nie będzie żadnego wyniku
dla testów integracyjnych i dokumentacji, ponieważ testy te zostaną uruchomione tylko wtedy, gdy wszystkie testy jednostkowe zakończą się pomyślnie.

Pierwsza sekcja testów jednostkowych jest taka sama, jak widzieliśmy: jeden wiersz
dla każdego testu jednostkowego (jeden o nazwie `internal`, który dodaliśmy w Liście 11-12), a
następnie wiersz podsumowujący dla testów jednostkowych.

Sekcja testów integracyjnych zaczyna się od wiersza `Running
tests/integration_test.rs`. Następnie znajduje się wiersz dla każdej funkcji testowej w
tym teście integracyjnym i wiersz podsumowujący dla wyników testu integracyjnego tuż przed rozpoczęciem sekcji `Doc-tests adder`.

Każdy plik testu integracyjnego ma własną sekcję, więc jeśli dodamy więcej plików do katalogu
*tests*, będzie więcej sekcji testów integracyjnych.

Nadal możemy uruchomić konkretną funkcję testu integracyjnego, określając nazwę
funkcji testowej jako argument `cargo test`. Aby uruchomić wszystkie testy w
konkretnym pliku testu integracyjnego, użyj argumentu `--test` `cargo test`
po którym następuje nazwa pliku:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-05-single-integration/output.txt}}
```

To polecenie uruchamia tylko testy z pliku *tests/integration_test.rs*.

#### Podmoduły w testach integracyjnych

W miarę dodawania kolejnych testów integracyjnych możesz chcieć utworzyć więcej plików w katalogu
*tests*, aby ułatwić ich organizację; na przykład możesz grupować funkcje testowe
według testowanej funkcjonalności. Jak wspomniano wcześniej, każdy plik
w katalogu *tests* jest kompilowany jako osobna skrzynia, co jest przydatne
do tworzenia oddzielnych zakresów, aby lepiej naśladować sposób, w jaki użytkownicy końcowi
będą korzystać z Twojej skrzyni. Oznacza to jednak, że pliki w katalogu *tests* nie
współdzielą tego samego zachowania, co pliki w *src*, jak dowiedziałeś się w rozdziale 7
dotyczącym sposobu rozdzielania kodu na moduły i pliki.

Różne zachowanie plików katalogu *tests* jest najbardziej zauważalne, gdy
masz zestaw funkcji pomocniczych do użycia w wielu plikach testów integracyjnych i
próbujesz wykonać kroki z sekcji [„Rozdzielanie modułów do różnych plików”][separating-modules-into-files]<!-- ignore --> rozdziału 7, aby
wyodrębnić je do wspólnego modułu. Na przykład, jeśli utworzymy *tests/common.rs*
i umieścimy w nim funkcję o nazwie `setup`, możemy dodać do `setup` kod, który
chcemy wywołać z wielu funkcji testowych w wielu plikach testowych:

<span class="filename">Filename: tests/common.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/tests/common.rs}}
```

Gdy ponownie uruchomimy testy, zobaczymy nową sekcję w wynikach testu dla pliku
*common.rs*, mimo że plik ten nie zawiera żadnych funkcji testowych ani nie
wywołaliśmy nigdzie funkcji `setup`:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/output.txt}}
```

Wyświetlanie `common` w wynikach testów z wyświetlonym `runing 0 tests` nie jest tym, czego chcieliśmy. Chcieliśmy po prostu udostępnić kod innym plikom testów integracyjnych.

Aby uniknąć wyświetlania `common` w wynikach testów, zamiast tworzyć
*tests/common.rs*, utworzymy *tests/common/mod.rs*. Katalog projektu
wygląda teraz tak:

```text
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs
```

To jest starsza konwencja nazewnictwa, którą Rust również rozumie, o której wspomnieliśmy w sekcji [„Alternatywne ścieżki plików”][alt-paths]<!-- ignore -->
Rozdziału 7. Nadanie plikowi takiej nazwy mówi Rustowi, aby nie traktował modułu `common`
jako pliku testu integracyjnego. Gdy przeniesiemy kod funkcji `setup` do
*tests/common/mod.rs* i usuniemy plik *tests/common.rs*, sekcja w
wyniku testu nie będzie już wyświetlana. Pliki w podkatalogach katalogu *tests*
nie są kompilowane jako osobne skrzynki ani nie mają sekcji w
wyniku testu.

Po utworzeniu *tests/common/mod.rs* możemy go użyć z dowolnego
pliku testu integracyjnego jako modułu. Oto przykład wywołania funkcji `setup`
z testu `it_adds_two` w *tests/integration_test.rs*:

<span class="filename">Filename: tests/integration_test.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-13-fix-shared-test-code-problem/tests/integration_test.rs}}
```

Należy zauważyć, że deklaracja `mod common;` jest taka sama jak deklaracja modułu, którą zademonstrowaliśmy w Liście 7-21. Następnie w funkcji testowej możemy wywołać funkcję
`common::setup()`.

#### Testy integracyjne dla skrzyń binarnych

Jeśli nasz projekt jest binarną skrzynią, która zawiera tylko plik *src/main.rs* i
nie ma pliku *src/lib.rs*, nie możemy tworzyć testów integracyjnych w katalogu
*tests* i przenosić funkcji zdefiniowanych w pliku *src/main.rs* do
zakresu za pomocą polecenia `use`. Tylko skrzynie biblioteczne udostępniają funkcje, których mogą używać inne skrzynie; skrzynie binarne są przeznaczone do samodzielnego uruchamiania.

To jeden z powodów, dla których projekty Rust, które dostarczają pliki binarne, mają
prosty plik *src/main.rs*, który wywołuje logikę znajdującą się w pliku
*src/lib.rs*. Używając tej struktury, testy integracyjne *mogą* testować
skrzynię biblioteczną za pomocą polecenia `use`, aby udostępnić ważne funkcje.
Jeśli ważne funkcje działają, niewielka ilość kodu w pliku
*src/main.rs* również będzie działać, a ta niewielka ilość kodu nie musi być
testowana.

## Streszczenie

Funkcje testowania języka Rust zapewniają sposób określania sposobu działania kodu, aby
zapewnić, że będzie on działał zgodnie z oczekiwaniami, nawet gdy wprowadzasz zmiany. Testy jednostkowe
ćwiczą różne części biblioteki oddzielnie i mogą testować szczegóły prywatnej implementacji. Testy integracyjne sprawdzają, czy wiele części biblioteki
działa poprawnie i wykorzystują publiczne API biblioteki do testowania kodu
w taki sam sposób, w jaki będzie go używał kod zewnętrzny. Chociaż system typów i
zasady własności języka Rust pomagają zapobiegać niektórym rodzajom błędów, testy są nadal ważne, aby
zmniejszyć liczbę błędów logicznych związanych z oczekiwanym zachowaniem kodu.

Połączmy wiedzę, której nauczyłeś się w tym rozdziale i w poprzednich
rozdziałach, aby pracować nad projektem!

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
[separating-modules-into-files]:
ch07-05-separating-modules-into-different-files.html
[alt-paths]: ch07-05-separating-modules-into-different-files.html#alternatywne-Ścieżki-do-plików
