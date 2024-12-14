## Refactoring to Improve Modularity and Obsługa błędów

Aby ulepszyć nasz program, naprawimy cztery problemy związane ze strukturą programu i sposobem, w jaki radzi sobie z potencjalnymi błędami. Po pierwsze, nasza funkcja `main`
teraz wykonuje dwa zadania: analizuje argumenty i odczytuje pliki. W miarę rozwoju naszego programu liczba oddzielnych zadań, które obsługuje funkcja `main`,
będzie się zwiększać. W miarę jak funkcja zyskuje obowiązki, trudniej jest
rozważać, testować i zmieniać bez psucia jednej z jej
części. Najlepiej jest oddzielić funkcjonalność, aby każda funkcja odpowiadała
za jedno zadanie.

Ten problem wiąże się również z drugim problemem: chociaż `query` i `file_path`
są zmiennymi konfiguracyjnymi dla naszego programu, zmienne takie jak `contents` są używane
do wykonywania logiki programu. Im dłuższy staje się `main`, tym więcej zmiennych
będziemy musieli wprowadzić do zakresu; im więcej zmiennych mamy w zakresie, tym trudniej
będziemy śledzić cel każdej z nich. Najlepiej jest zgrupować zmienne konfiguracyjne w jedną strukturę, aby ich cel był jasny.

Trzeci problem polega na tym, że użyliśmy `expect`, aby wydrukować komunikat o błędzie, gdy
odczyt pliku się nie powiedzie, ale komunikat o błędzie po prostu drukuje `Should have been
able to read the file`. Odczyt pliku może się nie powieść na kilka sposobów:
na przykład plik może być nieobecny lub możemy nie mieć uprawnień do jego otwarcia.
Teraz, niezależnie od sytuacji, wydrukowalibyśmy ten sam komunikat o błędzie dla
wszystkich, co nie dałoby użytkownikowi żadnych informacji!

Po czwarte, używamy `expect` do obsługi błędu, a jeśli użytkownik uruchomi nasz program
bez podania wystarczającej liczby argumentów, otrzyma błąd `index out of bounds`
z Rust, który nie wyjaśnia jasno problemu. Najlepiej byłoby, gdyby cały
kod obsługi błędów znajdował się w jednym miejscu, tak aby przyszli konserwatorzy mieli tylko jedno miejsce,
gdzie mogliby skonsultować kod, gdyby logika obsługi błędów wymagała zmiany. Posiadanie całego
kodu obsługi błędów w jednym miejscu zapewni również, że będziemy drukować wiadomości, które będą znaczące dla naszych użytkowników końcowych.

Rozwiążmy te cztery problemy, refaktoryzując nasz projekt.

### Separation of Concerns for Binary Projects

Problem organizacyjny przydzielania odpowiedzialności za wiele zadań do
funkcji `main` jest powszechny w wielu projektach binarnych. W rezultacie społeczność Rust opracowała wytyczne dotyczące rozdzielania osobnych zadań
programu binarnego, gdy `main` zaczyna się rozrastać. Proces ten składa się z następujących
kroków:

* Podziel program na plik *main.rs* i plik *lib.rs* i przenieś
logikę programu do *lib.rs*.
* Dopóki logika analizy wiersza poleceń jest mała, może pozostać w
*main.rs*.
* Gdy logika analizy wiersza poleceń zaczyna się komplikować, wypakuj ją
z *main.rs* i przenieś do *lib.rs*.

Obowiązki, które pozostają w funkcji `main` po tym procesie,
powinny być ograniczone do następujących:

* Wywołanie logiki analizy wiersza poleceń z wartościami argumentów
* Skonfigurowanie dowolnej innej konfiguracji
* Wywołanie funkcji `run` w *lib.rs*
* Obsługa błędu, jeśli `run` zwróci błąd

Ten wzorzec dotyczy rozdzielenia kwestii: *main.rs* obsługuje uruchamianie
programu, a *lib.rs* obsługuje całą logikę bieżącego zadania. Ponieważ
nie możesz bezpośrednio przetestować funkcji `main`, ta struktura pozwala przetestować całą
logikę programu, przenosząc ją do funkcji w *lib.rs*. Kod, który
pozostaje w *main.rs*, będzie wystarczająco mały, aby zweryfikować jego poprawność, odczytując go. Przeróbmy nasz program, postępując zgodnie z tym procesem.

#### Ekstrakcja parsera argumentów

Wyodrębnimy funkcjonalność do analizy argumentów do funkcji, którą
`main` wywoła, aby przygotować się do przeniesienia logiki analizy wiersza poleceń do
*src/lib.rs*. Listing 12-5 pokazuje nowy początek `main`, który wywołuje nową
funkcję `parse_config`, którą na razie zdefiniujemy w *src/main.rs*.

<Listing number="12-5" file-name="src/main.rs" caption="Extracting a `parse_config` function from `main`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-05/src/main.rs:here}}
```

</Listing>

Nadal zbieramy argumenty wiersza poleceń do wektora, ale zamiast
przypisywać wartość argumentu o indeksie 1 do zmiennej `query` i
wartość argumentu o indeksie 2 do zmiennej `file_path` w funkcji `main`,
przekazujemy cały wektor do funkcji `parse_config`. Funkcja
`parse_config` przechowuje następnie logikę, która określa, który argument
znajduje się w której zmiennej i przekazuje wartości z powrotem do `main`. Nadal tworzymy zmienne `query` i `file_path` w `main`, ale `main` nie ma już
odpowiedzialności za określanie, jak argumenty wiersza poleceń i zmienne
odpowiadają sobie.

Ta przeróbka może wydawać się przesadą w przypadku naszego małego programu, ale refaktoryzujemy
w małych, przyrostowych krokach. Po wprowadzeniu tej zmiany uruchom program ponownie,
aby sprawdzić, czy parsowanie argumentów nadal działa. Dobrze jest często sprawdzać postępy,
aby pomóc zidentyfikować przyczynę problemów, gdy wystąpią.
#### Grupowanie wartości konfiguracji

Możemy wykonać jeszcze jeden mały krok, aby jeszcze bardziej ulepszyć funkcję `parse_config`.
W tej chwili zwracamy krotkę, ale natychmiast rozbijamy ją
na pojedyncze części. To znak, że być może nie mamy jeszcze odpowiedniej abstrakcji.

Innym wskaźnikiem pokazującym, że jest miejsce na poprawę, jest część `config`
funkcji `parse_config`, która oznacza, że ​​dwie zwracane przez nas wartości są powiązane i
są częścią jednej wartości konfiguracji. Obecnie nie przekazujemy tego
znaczenia w strukturze danych inaczej niż poprzez grupowanie dwóch wartości w
krotce; zamiast tego umieścimy dwie wartości w jednej strukturze i nadamy każdemu z
pól struktury znaczącą nazwę. Dzięki temu przyszłym
opiekunom tego kodu łatwiej będzie zrozumieć, jak różne wartości są ze sobą powiązane i jakie jest ich przeznaczenie.

Listing 12-6 pokazuje ulepszenia funkcji `parse_config`.

<Listing number="12-6" file-name="src/main.rs" caption="Refactoring `parse_config` to return an instance of a `Config` struct">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-06/src/main.rs:here}}
```

</Listing>

Dodaliśmy strukturę o nazwie `Config` zdefiniowaną tak, aby miała pola o nazwach `query` i
`file_path`. Sygnatura `parse_config` wskazuje teraz, że zwraca wartość
`Config`. W treści `parse_config`, gdzie zwracaliśmy
wycinki stringów odwołujące się do wartości `String` w `args`, teraz definiujemy `Config` tak, aby zawierało posiadane wartości `String`. Zmienna `args` w `main` jest właścicielem
wartości argumentów i pozwala tylko funkcji `parse_config` je pożyczyć,
co oznacza, że ​​naruszylibyśmy zasady pożyczania Rust, gdyby `Config` próbował przejąć
własność wartości w `args`.

Istnieje wiele sposobów, w jaki moglibyśmy zarządzać danymi `String`; najłatwiejszą,
choć nieco nieefektywną, drogą jest wywołanie metody `clone` na wartościach.
Spowoduje to utworzenie pełnej kopii danych dla instancji `Config`, która
zajmuje więcej czasu i pamięci niż przechowywanie odwołania do danych ciągu.
Jednak klonowanie danych sprawia, że ​​nasz kod jest bardzo prosty, ponieważ
nie musimy zarządzać okresami istnienia odwołań; w tej sytuacji
oddanie odrobiny wydajności w celu uzyskania prostoty jest wartościowym kompromisem.

> ### Kompromisy związane z używaniem `clone`
>
> Wielu Rustaceanów ma tendencję do unikania używania `clone` w celu
> naprawienia problemów z własnością ze względu na koszt czasu wykonania. W
> [Rozdział 13][ch13]<!-- ignore --> dowiesz się, jak używać bardziej wydajnych
> metod w tego typu sytuacjach. Ale na razie możesz skopiować kilka
> ciągów, aby kontynuować robienie postępów, ponieważ te kopie wykonasz tylko
> raz, a ścieżka pliku i ciąg zapytania są bardzo małe. Lepiej mieć
> działający program, który jest trochę nieefektywny, niż próbować hiperoptymalizować kod
> przy pierwszym podejściu. W miarę zdobywania doświadczenia z Rustem, łatwiej będzie
> zacząć od najbardziej wydajnego rozwiązania, ale na razie
> całkowicie dopuszczalne jest wywołanie `clone`.

Zaktualizowaliśmy `main`, aby umieszczał wystąpienie `Config` zwrócone przez
`parse_config` w zmiennej o nazwie `config`, i zaktualizowaliśmy kod, który
wcześniej używał oddzielnych zmiennych `query` i `file_path`, więc teraz używa
zamiast tego pól w strukturze `Config`.

Teraz nasz kod wyraźniej przekazuje, że `query` i `file_path` są powiązane i
że ich celem jest skonfigurowanie sposobu działania programu. Każdy kod, który
używa tych wartości, wie, aby znaleźć je w wystąpieniu `config` w polach
nazwanych zgodnie z ich przeznaczeniem.

#### Tworzenie konstruktora dla `Config`

Do tej pory wyodrębniliśmy logikę odpowiedzialną za analizowanie argumentów wiersza poleceń
z `main` i umieściliśmy ją w funkcji `parse_config`. Dzięki temu
pomogło nam to zobaczyć, że wartości `query` i `file_path` są powiązane i że
ta relacja powinna być przekazana w naszym kodzie. Następnie dodaliśmy strukturę `Config`,
aby nazwać powiązany cel `query` i `file_path` i móc zwrócić nazwy
wartości jako nazwy pól struktury z funkcji `parse_config`.

Teraz, gdy celem funkcji `parse_config` jest utworzenie instancji `Config`, możemy zmienić `parse_config` ze zwykłej funkcji na funkcję
o nazwie `new`, która jest powiązana ze strukturą `Config`. Wprowadzenie tej zmiany
uczyni kod bardziej idiomatycznym. Możemy tworzyć instancje typów w
bibliotece standardowej, takich jak `String`, wywołując `String::new`. Podobnie, zmieniając
`parse_config` na funkcję `new` powiązaną z `Config`, będziemy mogli
tworzyć wystąpienia `Config` wywołując `Config::new`. Listing 12-7
pokazuje zmiany, które musimy wprowadzić.

<Listing number="12-7" file-name="src/main.rs" caption="Changing `parse_config` into `Config::new`">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-07/src/main.rs:here}}
```

</Listing>

Zaktualizowaliśmy `main`, gdzie wywoływaliśmy `parse_config`, aby zamiast tego wywołać
`Config::new`. Zmieniliśmy nazwę `parse_config` na `new` i przenieśliśmy ją
w obrębie bloku `impl`, który kojarzy funkcję `new` z `Config`. Spróbuj
ponownie skompilować ten kod, aby upewnić się, że działa.

### Fixing the Obsługa błędów

Teraz zajmiemy się naprawą obsługi błędów. Przypomnijmy, że próba dostępu do
wartości w wektorze `args` o indeksie 1 lub indeksie 2 spowoduje, że program
wpadnie w panikę, jeśli wektor zawiera mniej niż trzy elementy. Spróbuj uruchomić program
bez żadnych argumentów; będzie wyglądał tak:

```console
{{#include ../listings/ch12-an-io-project/listing-12-07/output.txt}}
```

Wiersz `index out of bounds: the len is 1 but the index is 1` jest komunikatem o błędzie przeznaczonym
dla programistów. Nie pomoże on naszym użytkownikom końcowym zrozumieć, co powinni zrobić zamiast tego. Naprawmy to teraz.

#### Ulepszanie komunikatu o błędzie

W Liście 12-8 dodajemy sprawdzenie w funkcji `new`, które zweryfikuje, czy wycinek jest wystarczająco długi przed uzyskaniem dostępu do indeksu 1 i indeksu 2. Jeśli wycinek nie jest wystarczająco długi, program wpada w panikę i wyświetla lepszy komunikat o błędzie.

<Listing number="12-8" file-name="src/main.rs" caption="Adding a check for the number of arguments">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-08/src/main.rs:here}}
```

</Listing>

Ten kod jest podobny do [funkcji `Guess::new`, którą napisaliśmy w Listingu
9-13][ch9-custom-types]<!-- ignore -->, gdzie wywołaliśmy `panic!`, gdy argument
`value` był poza zakresem prawidłowych wartości. Zamiast sprawdzać
zakres wartości tutaj, sprawdzamy, czy długość `args` wynosi co najmniej
`3`, a reszta funkcji może działać przy założeniu, że ten
warunek został spełniony. Jeśli `args` ma mniej niż trzy elementy, ten warunek
będzie `true` i wywołamy makro `panic!`, aby natychmiast zakończyć program.

Dzięki tym dodatkowym kilku linijkom kodu w `new` uruchommy program bez żadnych
argumentów ponownie, aby zobaczyć, jak teraz wygląda błąd:

```console
{{#include ../listings/ch12-an-io-project/listing-12-08/output.txt}}
```

Ten wynik jest lepszy: teraz mamy rozsądny komunikat o błędzie. Jednak mamy też
zbędne informacje, których nie chcemy udostępniać naszym użytkownikom. Być może
technika, której użyliśmy w Liście 9-13, nie jest najlepsza do użycia tutaj: wywołanie
`panik!` jest bardziej odpowiednie w przypadku problemu programistycznego niż problemu użytkowania,
[jak omówiono w rozdziale 9][ch9-error-guidelines]<!-- ignore -->. Zamiast tego
użyjemy innej techniki, o której dowiedziałeś się w rozdziale 9 — [zwracającej
`Result`][ch9-result]<!-- ignore -->, która wskazuje albo powodzenie, albo błąd.

<!-- Stare nagłówki. Nie usuwaj, bo linki mogą się zepsuć. -->
<a id="returning-a-result-from-new-instead-of-calling-panic"></a>

#### Zwrócenie `Result` zamiast wywołania `panic!`

Zamiast tego możemy zwrócić wartość `Result`, która będzie zawierać instancję `Config` w przypadku
pomyślnego zakończenia i opisze problem w przypadku błędu. Zmienimy również nazwę funkcji z `new` na `build`, ponieważ wielu
programistów oczekuje, że funkcje `new` nigdy nie zawiodą. Gdy `Config::build`
komunikuje się z `main`, możemy użyć typu `Result`, aby zasygnalizować, że
wystąpił problem. Następnie możemy zmienić `main`, aby przekonwertować wariant `Err` na bardziej
praktyczny błąd dla naszych użytkowników bez otaczającego tekstu o `wątku
'main'` i `RUST_BACKTRACE`, który powoduje wywołanie `panic!`.

Listing 12-9 pokazuje zmiany, które musimy wprowadzić w wartości zwracanej
funkcji, którą teraz wywołujemy `Config::build`, oraz w ciele funkcji,
które jest potrzebne do zwrócenia `Result`. Należy pamiętać, że nie skompiluje się to, dopóki nie zaktualizujemy także `main`, co zrobimy w następnym listingu.

<Listing number="12-9" file-name="src/main.rs" caption="Returning a `Result` from `Config::build`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-09/src/main.rs:here}}
```

</Listing>

Nasza funkcja `build` zwraca `Result` z instancją `Config` w przypadku powodzenia
i literałem ciągu w przypadku błędu. Nasze wartości błędów będą zawsze literałami ciągu, które mają `'static` czas życia.

Wprowadziliśmy dwie zmiany w treści funkcji: zamiast wywoływać `panic!`
gdy użytkownik nie przekaże wystarczającej liczby argumentów, zwracamy teraz wartość `Err` i
umieściliśmy wartość zwracaną `Config` w `Ok`. Te zmiany sprawiają, że
funkcja jest zgodna z nowym podpisem typu.

Zwrócenie wartości `Err` z `Config::build` pozwala funkcji `main`
obsługiwać wartość `Result` zwróconą przez funkcję `build` i
wyjść z procesu w sposób bardziej czysty w przypadku błędu.

<!-- Stare nagłówki. Nie usuwaj, ponieważ linki mogą zostać zerwane. -->
<a id="calling-confignew-and-handling-errors"></a>

#### Wywoływanie `Config::build` i obsługa błędów

Aby obsłużyć przypadek błędu i wydrukować przyjazną dla użytkownika wiadomość, musimy zaktualizować
`main`, aby obsłużyć `Result` zwracany przez `Config::build`, jak pokazano w
Listingu 12-10. Przejmiemy również odpowiedzialność za wyjście z narzędzia wiersza poleceń
z kodem błędu innym niż zero od `panic!` i zamiast tego zaimplementujemy to
ręcznie. Kod wyjścia inny niż zero to konwencja sygnalizująca procesowi, który
wywołał nasz program, że program zakończył się ze stanem błędu.

<Listing number="12-10" file-name="src/main.rs" caption="Exiting with an error code if building a `Config` fails">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-10/src/main.rs:here}}
```

</Listing>

W tym wpisie użyliśmy metody, której jeszcze nie omówiliśmy szczegółowo:
`unwrap_or_else`, która jest zdefiniowana w `Result<T, E>` przez bibliotekę standardową.
Użycie `unwrap_or_else` pozwala nam zdefiniować pewne niestandardowe, nie-`panic!`
obsługiwanie błędów. Jeśli `Result` jest wartością `Ok`, zachowanie tej metody jest podobne
do `unwrap`: zwraca wartość wewnętrzną, którą `Ok` opakowuje. Jednak jeśli
wartość jest wartością `Err`, ta metoda wywołuje kod w *closure*, który jest
anonimową funkcją, którą definiujemy i przekazujemy jako argument do `unwrap_or_else`.
Omówimy zamknięcia bardziej szczegółowo w [Rozdział 13][ch13]<!-- ignore -->. Na
teraz musisz wiedzieć, że `unwrap_or_else` przekaże wewnętrzną wartość
`Err`, która w tym przypadku jest statycznym ciągiem `"za mało argumentów"`
który dodaliśmy w Liście 12-9, do naszego zamknięcia w argumencie `err`, który
pojawia się między pionowymi rurami. Kod w zamknięciu może następnie użyć wartości
`err` podczas uruchamiania.

Dodaliśmy nową linię `use`, aby przenieść `process` ze standardowej biblioteki do
zakresu. Kod w zamknięciu, który zostanie uruchomiony w przypadku błędu, składa się tylko z dwóch
linii: drukujemy wartość `err`, a następnie wywołujemy `process::exit`. Funkcja
`process::exit` natychmiast zatrzyma program i zwróci
liczbę, która została przekazana jako kod statusu wyjścia. Jest to podobne do obsługi opartej na
`panic!`, której użyliśmy w Liście 12-8, ale nie otrzymujemy już wszystkich
dodatkowych danych wyjściowych. Spróbujmy:

```console
{{#include ../listings/ch12-an-io-project/listing-12-10/output.txt}}
```

Świetnie! Ten wynik jest o wiele bardziej przyjazny dla naszych użytkowników.

### Wyodrębnianie logiki z `main`

Teraz, gdy zakończyliśmy refaktoryzację parsowania konfiguracji, przejdźmy do
logiki programu. Jak stwierdziliśmy w [“Separacja obaw dla projektów binarnych”](#separation-of-concerns-for-binary-projects)<!-- ignore -->,
wyodrębnimy funkcję o nazwie `run`, która będzie zawierać całą logikę aktualnie znajdującą się w funkcji
`main`, która nie jest zaangażowana w ustawianie konfiguracji ani obsługę
błędów. Kiedy skończymy, `main` będzie zwięzły i łatwy do zweryfikowania poprzez
inspekcję, a my będziemy mogli pisać testy dla całej innej logiki.

Listing 12-11 pokazuje wyodrębnioną funkcję `run`. Na razie dokonujemy
tylko niewielkiej, przyrostowej poprawy wyodrębniania funkcji. Nadal
definiujemy funkcję w *src/main.rs*.

<Listing number="12-11" file-name="src/main.rs" caption="Extracting a `run` function containing the rest of the program logic">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-11/src/main.rs:here}}
```

</Listing>

Funkcja `run` zawiera teraz całą pozostałą logikę z `main`, zaczynając
od odczytu pliku. Funkcja `run` przyjmuje instancję `Config` jako
argument.

#### Zwracanie błędów z funkcji `run`

Po oddzieleniu pozostałej logiki programu do funkcji `run` możemy
ulepszyć obsługę błędów, tak jak zrobiliśmy to z `Config::build` w Listingu 12-9.
Zamiast pozwalać programowi na panikę poprzez wywołanie `expect`, funkcja `run`
zwróci `Result<T, E>`, gdy coś pójdzie nie tak. Pozwoli nam to
na dalszą konsolidację logiki obsługi błędów w `main` w sposób
przyjazny dla użytkownika. Listing 12-12 pokazuje zmiany, które musimy wprowadzić w
sygnaturze i treści `run`.

<Listing number="12-12" file-name="src/main.rs" caption="Changing the `run` function to return `Result`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-12/src/main.rs:here}}
```

</Listing>

Wprowadziliśmy tutaj trzy znaczące zmiany. Po pierwsze, zmieniliśmy typ zwracany
funkcji `run` na `Result<(), Box<dyn Error>>`. Ta funkcja poprzednio
zwracała typ jednostki, `()`, i zachowujemy go jako wartość zwracaną w przypadku
`Ok`.

W przypadku typu błędu użyliśmy *obiektu cechy* `Box<dyn Error>` (i
wprowadziliśmy `std::error::Error` do zakresu za pomocą instrukcji `use` na górze).
Obiekty cech omówimy w [Rozdziale 18][ch18]<!-- ignore -->. Na razie po prostu
wiedz, że `Box<dyn Error>` oznacza, że ​​funkcja zwróci typ, który
implementuje cechę `Error`, ale nie musimy określać, jaki konkretny typ
będzie wartością zwracaną. Daje nam to elastyczność w zwracaniu wartości błędów, które
mogą być różnych typów w różnych przypadkach błędów. Słowo kluczowe `dyn` jest skrótem od *dynamic*.

Po drugie, usunęliśmy wywołanie `expect` na rzecz operatora `?`, o którym
mówiliśmy w [Rozdziale 9][ch9-question-mark]<!-- ignore -->. Zamiast
`panic!` w przypadku błędu, `?` zwróci wartość błędu z bieżącej funkcji,
aby wywołanie mogło zostać obsłużone.

Po trzecie, funkcja `run` zwraca teraz wartość `Ok` w przypadku powodzenia.
Zadeklarowaliśmy typ powodzenia funkcji `run` jako `()` w sygnaturze,
co oznacza, że ​​musimy opakować wartość typu jednostki w wartość `Ok`. Ta
składnia `Ok(())` może na początku wyglądać nieco dziwnie, ale użycie `()` w ten sposób jest
idiomatycznym sposobem wskazania, że ​​wywołujemy `run` tylko ze względu na jego skutki uboczne;
nie zwraca wartości, której potrzebujemy.

Po uruchomieniu tego kodu zostanie on skompilowany, ale wyświetli ostrzeżenie:

```console
{{#include ../listings/ch12-an-io-project/listing-12-12/output.txt}}
```

Rust mówi nam, że nasz kod zignorował wartość `Result`, a wartość `Result`
może wskazywać, że wystąpił błąd. Ale nie sprawdzamy, czy wystąpił błąd, a kompilator przypomina nam, że prawdopodobnie mieliśmy tu
mieć jakiś kod obsługi błędów! Naprawmy ten problem teraz.

#### Obsługa błędów zwracanych przez `run` w `main`

Sprawdzimy błędy i obsłużymy je, stosując technikę podobną do tej, której użyliśmy
z `Config::build` w Liście 12-10, ale z niewielką różnicą:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-01-handling-errors-in-main/src/main.rs:here}}
```

Używamy `if let` zamiast `unwrap_or_else`, aby sprawdzić, czy `run` zwraca wartość
`Err` i wywołać `process::exit(1)`, jeśli tak się stanie. Funkcja `run`
nie zwraca wartości, którą chcemy `odwinąć` w taki sam sposób, w jaki
`Config::build` zwraca instancję `Config`. Ponieważ `run` zwraca `()` w
przypadku sukcesu, zależy nam tylko na wykryciu błędu, więc nie potrzebujemy
`unwrap_or_else`, aby zwrócić odwiniętą wartość, która byłaby tylko `()`.

Ciała funkcji `if let` i `unwrap_or_else` są takie same w
obu przypadkach: drukujemy błąd i wychodzimy.

### Podział kodu na skrzynię biblioteczną

Nasz projekt `minigrep` jak dotąd wygląda dobrze! Teraz podzielimy plik
*src/main.rs* i umieścimy trochę kodu w pliku *src/lib.rs*. W ten sposób
możemy przetestować kod i mieć plik *src/main.rs* z mniejszą liczbą obowiązków.

Przenieśmy cały kod, który nie znajduje się w funkcji `main` z *src/main.rs* do
*src/lib.rs*:

* Definicja funkcji `run`
* Odpowiednie polecenia `use`
* Definicja `Config`
* Definicja funkcji `Config::build`

Zawartość *src/lib.rs* powinna mieć sygnatury pokazane w Liście 12-13
(pominęliśmy ciała funkcji dla zwięzłości). Należy zauważyć, że nie skompiluje się
dopóki nie zmodyfikujemy *src/main.rs* w Liście 12-14.

<Listing number="12-13" file-name="src/lib.rs" caption="Moving `Config` and `run` into *src/lib.rs*">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-13/src/lib.rs:here}}
```

</Listing>

Użyliśmy słowa kluczowego `pub` w sposób liberalny: w `Config`, w jego polach i jego metodzie
`build` oraz w funkcji `run`. Teraz mamy skrzynię biblioteczną, która ma
publiczny interfejs API, który możemy przetestować!

Teraz musimy przenieść kod, który przenieśliśmy do *src/lib.rs*, do zakresu
skrzyni binarnej w *src/main.rs*, jak pokazano na Listingu 12-14.

<Listing number="12-14" file-name="src/main.rs" caption="Using the `minigrep` library crate in *src/main.rs*">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-14/src/main.rs:here}}
```

</Listing>

Dodajemy linię `use minigrep::Config`, aby przenieść typ `Config` ze
skrzyni bibliotecznej do zakresu skrzyni binarnej i dodajemy prefiks do funkcji `run`
nazwą naszej skrzyni. Teraz wszystkie funkcjonalności powinny być połączone i powinny
działać. Uruchom program za pomocą `cargo run` i upewnij się, że wszystko działa poprawnie.

Uff! To było mnóstwo pracy, ale przygotowaliśmy się na sukces w
przyszłości. Teraz obsługa błędów jest o wiele łatwiejsza, a kod stał się
bardziej modułowy. Prawie cała nasza praca będzie odtąd wykonywana w *src/lib.rs*.

Skorzystajmy z tej nowo odkrytej modułowości, robiąc coś, co
byłoby trudne w przypadku starego kodu, ale jest łatwe w przypadku nowego kodu:
napiszemy kilka testów!

[ch13]: ch13-00-functional-features.html
[ch9-custom-types]: ch09-03-to-panic-or-not-to-panic.html#creating-custom-types-for-validation
[ch9-error-guidelines]: ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling
[ch9-result]: ch09-02-recoverable-errors-with-result.html
[ch18]: ch18-00-oop.html
[ch9-question-mark]: ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator
