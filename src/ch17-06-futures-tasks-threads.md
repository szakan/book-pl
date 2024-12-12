## Przyszłości, zadania i wątki

Jak widzieliśmy w poprzednim rozdziale, wątki zapewniają jedno podejście do współbieżności.
W tym rozdziale widzieliśmy inne podejście do współbieżności, używając asynchroniczności z
futures i strumieniami. Możesz się zastanawiać, dlaczego miałbyś wybrać jedno lub drugie. Odpowiedź brzmi: to zależy! A w wielu przypadkach wybór nie dotyczy wątków
*lub* asynchroniczności, ale raczej wątków *i* asynchroniczności.

Wiele systemów operacyjnych dostarcza modele współbieżności oparte na wątkach od
dziesiątek lat, a wiele języków programowania w rezultacie je obsługuje.
Jednak nie są one pozbawione kompromisów. W wielu systemach operacyjnych
używają one sporo pamięci dla każdego wątku i wiążą się z pewnym narzutem na
uruchamianie i wyłączanie. Wątki są również opcją tylko wtedy, gdy Twój
system operacyjny i sprzęt je obsługują! W przeciwieństwie do głównych komputerów stacjonarnych i mobilnych, niektóre systemy wbudowane w ogóle nie mają systemu operacyjnego, więc nie mają też
wątków!

Model asynchroniczny zapewnia inny — i ostatecznie uzupełniający się — zestaw
kompromisów. W modelu asynchronicznym operacje współbieżne nie wymagają własnych
wątków. Zamiast tego mogą być uruchamiane w zadaniach, jak wtedy, gdy użyliśmy `trpl::spawn_task`, aby
rozpocząć pracę od funkcji synchronicznej w całej sekcji strumieni. Zadanie
jest podobne do wątku, ale zamiast być zarządzane przez system operacyjny,
jest zarządzane przez kod na poziomie biblioteki: środowisko wykonawcze.

W poprzedniej sekcji zobaczyliśmy, że możemy zbudować `Stream`, używając asynchronicznego
kanału i generując asynchroniczne zadanie, które możemy wywołać z synchronicznego kodu. Możemy
zrobić dokładnie to samo z wątkiem! W Liście 17-40 użyliśmy
`trpl::spawn_task` i `trpl::sleep`. W Liście 17-41 zastępujemy je interfejsami API `thread::spawn` i `thread::sleep` ze standardowej biblioteki w funkcji `get_intervals`.

<Listing number="17-41" caption="Using the `std::thread` APIs instead of the async `trpl` APIs for the `get_intervals` function" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-41/src/main.rs:threads}}
```

</Listing>

Jeśli to uruchomisz, wynik będzie identyczny. Zauważ, jak niewiele się tutaj zmienia
z perspektywy kodu wywołującego! Co więcej, mimo że jedna z naszych
funkcji utworzyła zadanie asynchroniczne w środowisku wykonawczym, a druga utworzyła
wątek systemu operacyjnego, różnice nie miały wpływu na strumienie wynikowe.

Pomimo podobieństw te dwa podejścia zachowują się bardzo różnie, chociaż
możemy mieć trudności ze zmierzeniem tego w tym bardzo prostym przykładzie. Moglibyśmy
utworzyć miliony zadań asynchronicznych na dowolnym nowoczesnym komputerze osobistym. Gdybyśmy próbowali zrobić to
z wątkami, dosłownie zabrakłoby nam pamięci!

Jednak istnieje powód, dla którego te interfejsy API są tak podobne. Wątki działają jako granica
dla zestawów operacji synchronicznych; współbieżność jest możliwa *pomiędzy* wątkami.
Zadania działają jako granica dla zestawów operacji *asynchronicznych*; współbieżność jest możliwa
zarówno *pomiędzy*, jak i *w obrębie* zadań, ponieważ zadanie może przełączać się między
przyszłościami w swoim ciele. Wreszcie, futures są najbardziej szczegółową jednostką
współbieżności w Rust, a każda przyszłość może reprezentować drzewo innych przyszłości. Środowisko wykonawcze — a konkretnie jego wykonawca — zarządza zadaniami, a zadania zarządzają przyszłościami. Pod tym względem zadania są podobne do lekkich wątków zarządzanych przez środowisko wykonawcze z
dodatkowymi możliwościami wynikającymi z zarządzania przez środowisko wykonawcze, a nie przez
system operacyjny.

Nie oznacza to, że zadania asynchroniczne są zawsze lepsze od wątków, ani też, że wątki są zawsze lepsze od zadań.

Współbieżność z wątkami jest w pewnym sensie prostszym modelem programowania niż
współbieżność z `async`. Może to być zaletą lub wadą. Wątki są
w pewnym sensie „odpal i zapomnij”, nie mają natywnego odpowiednika przyszłości, więc
po prostu działają do końca, bez przerwy, z wyjątkiem samego systemu operacyjnego. Oznacza to, że nie mają wbudowanego wsparcia dla *współbieżności wewnątrz zadania*,
jak futures. Wątki w Rust nie mają również mechanizmów anulowania —
tego tematu nie omawialiśmy szczegółowo w tym rozdziale, ale który jest domyślny,
ponieważ za każdym razem, gdy kończyliśmy przyszłość, jej stan był poprawnie czyszczony.

Te ograniczenia sprawiają również, że wątki są trudniejsze do komponowania niż przyszłości. Znacznie
trudniej jest na przykład używać wątków do budowania pomocników, takich jak
`timeout`, który zbudowaliśmy w [„Budowanie własnych abstrakcji asynchronicznych”][combining-futures]
lub metoda `throttle`, której użyliśmy ze strumieniami w [„Composing Strumienie”][streams].
Fakt, że przyszłości są bogatszymi strukturami danych, oznacza, że ​​można je komponować
w sposób bardziej naturalny, jak widzieliśmy.

Zadania dają *dodatkową* kontrolę nad przyszłościami, pozwalając wybrać, gdzie
i jak grupować przyszłości. Okazuje się, że wątki i zadania często
bardzo dobrze ze sobą współpracują, ponieważ zadania można (przynajmniej w niektórych środowiskach wykonawczych) przenosić
między wątkami. Nie wspominaliśmy o tym do tej pory, ale pod maską `Runtime`, którego używamy, w tym funkcje `spawn_blocking` i
`spawn_task`, jest domyślnie wielowątkowe! Wiele środowisk wykonawczych używa
podejścia zwanego *kradzieżą pracy*, aby transparentnie przenosić zadania między wątkami
na podstawie bieżącego wykorzystania wątków, w celu
poprawy ogólnej wydajności systemu. Aby to zbudować, potrzebne są
wątki *i* zadania, a zatem i przyszłe.

Jako domyślny sposób myślenia o tym, którego użyć, gdy:

- Jeśli praca jest *bardzo paralelizowalna*, taka jak przetwarzanie wielu danych, gdzie
każda część może być przetwarzana oddzielnie, wątki są lepszym wyborem.
- Jeśli praca jest *bardzo współbieżna*, taka jak obsługa wiadomości z wielu
różnych źródeł, które mogą przychodzić w różnych odstępach czasu lub z różnymi szybkościami,
asynchroniczność jest lepszym wyborem.

A jeśli potrzebujesz mieszanki paralelizmu i współbieżności, nie musisz
wybierać między wątkami a asynchronicznością. Możesz ich używać razem swobodnie, pozwalając każdemu
służyć tej części, w której jest najlepszy. Na przykład Listing 17-42 pokazuje dość
typowy przykład takiej mieszanki w rzeczywistym kodzie Rust.

<Listing number="17-42" caption="Sending messages with blocking code in a thread and awaiting the messages in an async block" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-42/src/main.rs:all}}
```

</Listing>

Zaczynamy od utworzenia kanału asynchronicznego. Następnie tworzymy wątek, który przejmuje
własność strony nadawcy kanału. W wątku wysyłamy
liczby od 1 do 10 i śpimy przez sekundę pomiędzy każdą z nich. Na koniec uruchamiamy
przyszłość utworzoną z blokiem asynchronicznym przekazanym do `trpl::run` tak jak robiliśmy to
w całym rozdziale. W tej przyszłości oczekujemy na te wiadomości, tak jak w
innych przykładach przekazywania wiadomości, które widzieliśmy.

Aby powrócić do przykładów, które otworzyliśmy w rozdziale: możesz sobie wyobrazić uruchomienie
zestawu zadań kodowania wideo przy użyciu dedykowanego wątku, ponieważ kodowanie wideo
jest ograniczone obliczeniowo, ale powiadamiając interfejs użytkownika, że ​​te operacje są wykonywane za pomocą
kanału asynchronicznego. Przykładów tego rodzaju mieszanki jest mnóstwo!

## Summary

To nie jest ostatni raz, kiedy zobaczysz współbieżność w tej książce: projekt w
Rozdziale 21 wykorzysta koncepcje z tego rozdziału w bardziej realistycznej sytuacji
niż mniejsze przykłady omówione tutaj — i porówna bardziej bezpośrednio, jak wygląda rozwiązywanie tego rodzaju problemów za pomocą wątków w porównaniu z zadaniami i przyszłościami.

Niezależnie od tego, czy chodzi o wątki, przyszłość i zadania, czy też ich kombinację, Rust daje Ci narzędzia, których potrzebujesz, aby pisać bezpieczny, szybki, współbieżny
kod — niezależnie od tego, czy chodzi o serwer WWW o wysokiej przepustowości, czy wbudowany system operacyjny.

Następnie omówimy idiomatyczne sposoby modelowania problemów i strukturyzacji rozwiązań,
gdy Twoje programy Rust staną się większe. Ponadto omówimy, w jaki sposób idiomy Rust
odnoszą się do tych, które możesz znać z programowania obiektowego.


[combining-futures]: ch17-03-more-futures.html#building-our-own-async-abstractions
[streams]: ch17-04-streams.html#composing-streams
