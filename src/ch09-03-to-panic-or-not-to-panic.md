## Panikować! czy nie?

Jak więc zdecydować, kiedy należy wywołać `panic!`, a kiedy zwrócić
`Result`? Gdy kod panikuje, nie ma sposobu na odzyskanie. Możesz wywołać `panic!`
w przypadku dowolnej sytuacji błędu, niezależnie od tego, czy istnieje możliwość odzyskania, czy nie, ale
wtedy podejmujesz decyzję, że sytuacja jest nieodwracalna w imieniu
kodu wywołującego. Gdy zdecydujesz się zwrócić wartość `Result`, podajesz
opcje kodowi wywołującemu. Kod wywołujący może wybrać próbę odzyskania w sposób
odpowiedni dla swojej sytuacji lub może zdecydować, że wartość `Err`
w tym przypadku jest nieodwracalna, więc może wywołać `panic!` i zmienić
błąd odzyskiwalny w nieodwracalny. Dlatego zwracanie `Result` jest
dobrym domyślnym wyborem, gdy definiujesz funkcję, która może zakończyć się niepowodzeniem.

W sytuacjach takich jak przykłady, kod prototypowy i testy, bardziej
odpowiednie jest pisanie kodu, który panikuje, zamiast zwracania `Result`. Przyjrzyjmy się
dlaczego tak jest, a następnie omówmy sytuacje, w których kompilator nie może stwierdzić, że
awaria jest niemożliwa, ale Ty jako człowiek możesz. Rozdział zakończy się
kilkoma ogólnymi wskazówkami, jak zdecydować, czy panikować w kodzie biblioteki.

### Examples, Prototype Code, and Tests

Kiedy piszesz przykład ilustrujący jakąś koncepcję, uwzględnienie
solidnego kodu obsługi błędów może sprawić, że przykład będzie mniej jasny. W przykładach
rozumie się, że wywołanie metody takiej jak `unwrap`, która może panikować, jest
zastępstwem sposobu, w jaki chcesz, aby Twoja aplikacja obsługiwała błędy, co może się
różnić w zależności od tego, co robi reszta Twojego kodu.

Podobnie, metody `unwrap` i `expect` są bardzo przydatne podczas prototypowania,
zanim będziesz gotowy zdecydować, jak obsługiwać błędy. Pozostawiają one wyraźne znaczniki w
Twoim kodzie na czas, gdy będziesz gotowy, aby uczynić swój program bardziej solidnym.

Jeśli wywołanie metody nie powiedzie się w teście, chcesz, aby cały test się nie powiódł, nawet jeśli
ta metoda nie jest testowaną funkcjonalnością. Ponieważ `panic!` to sposób, w jaki test
jest oznaczany jako niepowodzenie, wywołanie `unwrap` lub `expect` jest dokładnie tym, co powinno się
stać.

### Cases in Which You Have More Information Than the Compiler

Właściwe byłoby również wywołanie `unwrap` lub `expect`, gdy masz jakąś
inną logikę, która zapewnia, że ​​`Result` będzie miał wartość `Ok`, ale ta logika
nie jest czymś, co kompilator rozumie. Nadal będziesz mieć wartość `Result`,
którą musisz obsłużyć: każda wywoływana przez Ciebie operacja nadal ma możliwość
zakończenia się niepowodzeniem, nawet jeśli jest to logicznie niemożliwe w Twojej konkretnej sytuacji. Jeśli możesz upewnić się, ręcznie sprawdzając kod,
że nigdy nie będziesz mieć wariantu `Err`, całkowicie dopuszczalne jest wywołanie
`unwrap`, a jeszcze lepiej udokumentowanie powodu, dla którego uważasz, że nigdy nie będziesz mieć wariantu
`Err` w tekście `expect`. Oto przykład:

```rust
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-08-unwrap-that-cant-fail/src/main.rs:here}}
```

Tworzymy instancję `IpAddr` poprzez parsowanie zakodowanego na stałe ciągu. Widzimy, że `127.0.0.1` jest prawidłowym adresem IP, więc dopuszczalne jest użycie `expect`
tutaj. Jednak posiadanie zakodowanego na stałe, prawidłowego ciągu nie zmienia typu zwracanego
metody `parse`: nadal otrzymujemy wartość `Result`, a kompilator
nadal będzie kazał nam obsługiwać `Result` tak, jakby wariant `Err` był możliwy,
ponieważ kompilator nie jest wystarczająco inteligentny, aby zauważyć, że ten ciąg jest zawsze
poprawnym adresem IP. Gdyby ciąg adresu IP pochodził od użytkownika, a nie był
zakodowany na stałe w programie i dlatego *miał* możliwość awarii,
zdecydowanie chcielibyśmy obsługiwać `Result` w bardziej niezawodny sposób.
Wspomnienie założenia, że ​​ten adres IP jest zakodowany na stałe, skłoni nas do
zmiany `expect` na lepszy kod obsługi błędów, jeśli w przyszłości będziemy musieli uzyskać
adres IP z innego źródła.

### Guidelines for Obsługa błędów

Wskazane jest, aby kod panikował, gdy istnieje możliwość, że kod może
znaleźć się w złym stanie. W tym kontekście *zły stan* to taki, gdy jakieś założenie,
gwarancja, kontrakt lub niezmiennik zostały złamane, na przykład gdy do kodu przekazano nieprawidłowe wartości,
sprzeczne wartości lub brakujące wartości — plus jedno lub
więcej z następujących:

* Zły stan to coś, co jest nieoczekiwane, w przeciwieństwie do czegoś,
co prawdopodobnie zdarzy się okazjonalnie, na przykład gdy użytkownik wprowadzi dane w złym
formacie.
* Twój kod po tym punkcie musi polegać na tym, że nie znajduje się w tym złym stanie,
zamiast sprawdzać problem na każdym kroku.
* Nie ma dobrego sposobu na zakodowanie tych informacji w używanych typach. Przeanalizujemy
przykład tego, co mamy na myśli w sekcji [“Encoding States and Behavior
  as Types”][encoding]<!-- ignore -->  rozdziału 18.

Jeśli ktoś wywołuje Twój kod i przekazuje wartości, które nie mają sensu, najlepiej
zwrócić błąd, jeśli to możliwe, aby użytkownik biblioteki mógł zdecydować, co
chce zrobić w tym przypadku. Jednak w przypadkach, gdy kontynuowanie może być
niebezpieczne lub szkodliwe, najlepszym wyborem może być wywołanie `panic!` i powiadomienie
osoby korzystającej z Twojej biblioteki o błędzie w jej kodzie, aby mogła go naprawić podczas
rozwoju. Podobnie `panic!` jest często odpowiednie, jeśli wywołujesz
zewnętrzny kod, który jest poza Twoją kontrolą i zwraca nieprawidłowy stan, którego
nie masz możliwości naprawić.

Jednak gdy spodziewana jest awaria, bardziej odpowiednie jest zwrócenie `Result`
niż wykonanie wywołania `panic!`. Przykłady obejmują parser otrzymujący nieprawidłowe
dane lub żądanie HTTP zwracające status wskazujący, że osiągnięto limit
szybkości. W takich przypadkach zwrócenie `Result` oznacza, że ​​awaria jest
oczekiwaną możliwością, którą wywołujący kod musi zdecydować, jak obsłużyć.

Gdy kod wykonuje operację, która może narazić użytkownika na ryzyko, jeśli zostanie
wywołana przy użyciu nieprawidłowych wartości, kod powinien najpierw sprawdzić, czy wartości są prawidłowe,
a jeśli nie są prawidłowe, wpaść w panikę. Dzieje się tak głównie ze względów bezpieczeństwa:
próba wykonania operacji na nieprawidłowych danych może narazić kod na luki w zabezpieczeniach.
To jest główny powód, dla którego biblioteka standardowa wywoła `panic!`, jeśli spróbujesz
dostępu do pamięci poza zakresem: próba dostępu do pamięci, która nie należy do
bieżącej struktury danych, jest częstym problemem bezpieczeństwa. Funkcje często mają
*kontrakty*: ich zachowanie jest gwarantowane tylko wtedy, gdy dane wejściowe spełniają określone
wymagania. Panika, gdy kontrakt jest naruszany, ma sens, ponieważ
naruszenie kontraktu zawsze wskazuje na błąd po stronie wywołującego, a nie jest to rodzaj błędu,
który kod wywołujący musi jawnie obsłużyć. W rzeczywistości nie
ma rozsądnego sposobu na odzyskanie kodu wywołującego; wywołujący *programiści* muszą
naprawić kod. Kontrakty dla funkcji, zwłaszcza gdy naruszenie spowoduje
panikę, powinny być wyjaśnione w dokumentacji API dla tej funkcji.

Jednakże posiadanie wielu kontroli błędów we wszystkich funkcjach byłoby rozwlekłe
i denerwujące. Na szczęście możesz użyć systemu typów Rust (a zatem
kontroli typu wykonywanej przez kompilator), aby wykonać wiele kontroli za Ciebie. Jeśli Twoja
funkcja ma określony typ jako parametr, możesz kontynuować logikę swojego kodu,
wiedząc, że kompilator już upewnił się, że masz prawidłową wartość. Na
przykład, jeśli masz typ zamiast `Opcji`, Twój program oczekuje, że będzie miał *coś*, a nie *nic*. Twój kod nie musi wtedy obsługiwać
dwóch przypadków dla wariantów `Some` i `None`: będzie miał tylko jeden przypadek, aby
zdecydowanie mieć wartość. Kod próbujący przekazać nic do Twojej funkcji, nawet się nie skompiluje, więc Twoja funkcja nie musi sprawdzać tego przypadku w czasie wykonywania.
Innym przykładem jest użycie typu liczby całkowitej bez znaku, takiego jak `u32`, co zapewnia, że
parametr nigdy nie jest ujemny.
### Creating Custom Types for Validation

Rozwińmy ideę wykorzystania systemu typów Rust, aby upewnić się, że mamy prawidłową wartość
o krok dalej i przyjrzyjmy się stworzeniu niestandardowego typu do walidacji. Przypomnij sobie
grę w zgadywanie z rozdziału 2, w której nasz kod prosił użytkownika o odgadnięcie liczby
od 1 do 100. Nigdy nie sprawdzaliśmy, czy odgadnięcie użytkownika mieści się między tymi liczbami, zanim sprawdziliśmy je z naszą tajną liczbą; sprawdzaliśmy tylko, czy odgadnięcie było dodatnie. W tym przypadku konsekwencje nie były bardzo poważne: nasze
wyjście „Za wysokie” lub „Za niskie” nadal byłoby prawidłowe. Ale byłoby
przydatnym ulepszeniem pokierowanie użytkownika w stronę prawidłowych odgadnięć i uzyskanie innego
zachowania, gdy użytkownik odgadnie liczbę, która jest poza zakresem, niż gdy użytkownik wpisuje, na przykład, litery.

Jednym ze sposobów, aby to zrobić, byłoby przeanalizowanie przypuszczenia jako `i32` zamiast tylko
`u32`, aby umożliwić potencjalnie ujemne liczby, a następnie dodanie sprawdzenia, czy
liczba mieści się w zakresie, w następujący sposób:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-09-guess-out-of-range/src/main.rs:here}}
```

</Listing>

Wyrażenie `if` sprawdza, czy nasza wartość jest poza zakresem, informuje użytkownika
o problemie i wywołuje `continue`, aby rozpocząć następną iterację pętli
i poprosić o kolejne zgadywanie. Po wyrażeniu `if` możemy kontynuować
porównania między `guess` a tajną liczbą, wiedząc, że `guess` jest
między 1 a 100.

Nie jest to jednak idealne rozwiązanie: gdyby było absolutnie krytyczne, aby
program działał tylko na wartościach między 1 a 100 i miałby wiele funkcji
z tym wymaganiem, posiadanie takiego sprawdzenia w każdej funkcji byłoby
żmudne (i mogłoby mieć wpływ na wydajność).

Zamiast tego możemy utworzyć nowy typ i umieścić walidacje w funkcji, aby utworzyć
instancję typu, zamiast powtarzać walidacje wszędzie. W ten sposób funkcje mogą bezpiecznie używać nowego typu w swoich sygnaturach i
pewnie używać otrzymanych wartości. Wylistowanie 9-13 przedstawia jeden ze sposobów zdefiniowania typu
`Guess`, który utworzy wystąpienie typu `Guess` tylko wtedy, gdy funkcja `new`
otrzyma wartość z przedziału od 1 do 100.

<Listing number="9-13" caption="A `Guess` type that will only continue with values between 1 and 100">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-13/src/lib.rs}}
```

</Listing>

Najpierw definiujemy strukturę o nazwie `Guess`, która ma pole o nazwie `value`, które
przechowuje `i32`. Tutaj zostanie zapisana liczba.

Następnie implementujemy powiązaną funkcję o nazwie `new` w `Guess`, która tworzy
instancje wartości `Guess`. Funkcja `new` jest zdefiniowana tak, aby miała jeden
parametr o nazwie `value` typu `i32` i zwracała `Guess`. Kod w
treści funkcji `new` testuje `value`, aby upewnić się, że jest między 1 a 100.
Jeśli `value` nie przejdzie tego testu, wykonujemy wywołanie `panic!`, które powiadomi
programistę piszącego wywołujący kod, że ma błąd, który musi
naprawić, ponieważ utworzenie `Guess` z `value` spoza tego zakresu
naruszyłoby kontrakt, na którym polega `Guess::new`. Warunki, w których
`Guess::new` może panikować, powinny zostać omówione w jego publicznej dokumentacji API; omówimy konwencje dokumentacji wskazujące na możliwość
wystąpienia `paniki!` w dokumentacji API, którą utworzysz w rozdziale 14. Jeśli
`value` przejdzie test, tworzymy nowy `Guess` z polem `value` ustawionym
na parametr `value` i zwracamy `Guess`.

Następnie implementujemy metodę o nazwie `value`, która pożycza `self`, nie ma żadnych
innych parametrów i zwraca `i32`. Tego rodzaju metoda jest czasami nazywana
*getterem*, ponieważ jej celem jest pobranie pewnych danych z jej pól i
zwrócenie ich. Ta publiczna metoda jest konieczna, ponieważ pole `value` struktury `Guess`
jest prywatne. Ważne jest, aby pole `value` było prywatne, więc kod
używający struktury `Guess` nie może ustawiać `value` bezpośrednio: kod poza
modułem *musi* użyć funkcji `Guess::new`, aby utworzyć wystąpienie
`Guess`, zapewniając w ten sposób, że nie ma możliwości, aby `Guess` miało `value`, które
nie zostało sprawdzone przez warunki w funkcji `Guess::new`.

Funkcja, która ma parametr lub zwraca tylko liczby od 1 do 100, może
następnie zadeklarować w swoim podpisie, że przyjmuje lub zwraca `Guess` zamiast
`i32` i nie musi wykonywać żadnych dodatkowych sprawdzeń w swoim ciele.
## Summary

Funkcje obsługi błędów w Rust zostały zaprojektowane, aby pomóc Ci pisać bardziej solidny kod.
Makro `panic!` sygnalizuje, że Twój program jest w stanie, którego nie może obsłużyć i
pozwala Ci powiedzieć procesowi, aby się zatrzymał, zamiast próbować kontynuować z nieprawidłowymi lub
niepoprawnymi wartościami. Wyliczenie `Result` używa systemu typów Rust, aby wskazać, że
operacje mogą się nie powieść w sposób, z którego Twój kod mógłby się odzyskać. Możesz użyć
`Result`, aby powiedzieć kodowi, który wywołuje Twój kod, że musi obsłużyć potencjalny
sukces lub niepowodzenie. Użycie `panic!` i `Result` w odpowiednich
sytuacjach sprawi, że Twój kod będzie bardziej niezawodny w obliczu nieuniknionych problemów.

Teraz, gdy poznałeś przydatne sposoby, w jakie standardowa biblioteka używa typów generycznych z wyliczeniami `Option` i `Result`, omówimy, jak działają typy generyczne i jak możesz ich
używac w swoim kodzie.

[encoding]: ch18-03-oo-design-patterns.html#encoding-states-and-behavior-as-types
