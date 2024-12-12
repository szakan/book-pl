## Używanie przekazywania komunikatów do przesyłania danych między wątkami

Coraz popularniejszym podejściem do zapewnienia bezpiecznej współbieżności jest *przekazywanie wiadomości*, w którym wątki lub aktorzy komunikują się, wysyłając sobie nawzajem wiadomości
zawierające dane. Oto pomysł w haśle z [dokumentacji języka Go](https://golang.org/doc/effective_go.html#concurrency):
„Nie komunikuj się poprzez współdzielenie pamięci; zamiast tego, współdziel pamięć poprzez komunikację”.

Aby osiągnąć współbieżność wysyłania wiadomości, standardowa biblioteka Rusta zapewnia
implementację *kanałów*. Kanał to ogólna koncepcja programowania, za pomocą której
dane są wysyłane z jednego wątku do drugiego.

Kanał w programowaniu można sobie wyobrazić jako kierunkowy kanał
wody, taki jak strumień lub rzeka. Jeśli umieścisz coś takiego jak gumowa kaczka
w rzece, będzie ona płynąć w dół rzeki do końca drogi wodnej.

Kanał składa się z dwóch połówek: nadajnika i odbiornika. Połowa nadajnika to
miejsce w górnym biegu rzeki, gdzie wrzucasz gumowe kaczki do rzeki, a
połowa odbiornika to miejsce, w którym gumowa kaczka kończy w dolnym biegu rzeki. Jedna część twojego
kodu wywołuje metody w nadajniku z danymi, które chcesz wysłać, a
inna część sprawdza, czy po stronie odbiorczej nie nadeszły wiadomości. Mówi się, że kanał jest
*zamknięty*, jeśli połowa nadajnika lub odbiornika zostanie porzucona.

Tutaj opracujemy program, który ma jeden wątek generujący wartości i
wysyłający je kanałem oraz inny wątek odbierający wartości i
wypisujący je. Będziemy wysyłać proste wartości między wątkami za pomocą kanału,
aby zilustrować tę funkcję. Gdy już zapoznasz się z tą techniką, możesz
używać kanałów dla dowolnych wątków, które muszą się ze sobą komunikować, takich jak
system czatu lub system, w którym wiele wątków wykonuje części obliczeń,
i wysyła części do jednego wątku, który agreguje wyniki.

Najpierw w Liście 16-6 utworzymy kanał, ale nic z nim nie zrobimy.
Należy pamiętać, że to się jeszcze nie skompiluje, ponieważ Rust nie potrafi określić, jakiego typu wartości chcemy przesłać przez kanał.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-06/src/main.rs}}
```

<span class="caption">Listing 16-6: Tworzenie kanału i przypisywanie dwóch
połówek do `tx` i `rx`</span>

Tworzymy nowy kanał za pomocą funkcji `mpsc::channel`; `mpsc` oznacza
*wielu producentów, jednego konsumenta*. Krótko mówiąc, sposób, w jaki standardowa biblioteka Rusta
implementuje kanały, oznacza, że ​​kanał może mieć wiele końców *wysyłających*, które
produkują wartości, ale tylko jeden koniec *odbierający*, który te wartości konsumuje. Wyobraź sobie
wiele strumieni płynących razem do jednej dużej rzeki: wszystko wysłane dowolnym
ze strumieni trafi na końcu do jednej rzeki. Na razie zaczniemy od pojedynczego
producenta, ale dodamy wielu producentów, gdy ten przykład
zacznie działać.

Funkcja `mpsc::channel` zwraca krotkę, której pierwszym elementem jest
koniec wysyłający — nadajnik — a drugim elementem jest koniec odbierający — odbiornik. Skróty `tx` i `rx` są tradycyjnie używane w wielu dziedzinach
odpowiednio dla *nadajnika* i *odbiornika*, więc nazywamy nasze zmienne w ten sposób,
aby wskazać każdy koniec. Używamy instrukcji `let` ze wzorcem, który
destrukturyzuje krotki; omówimy użycie wzorców w instrukcjach `let`
i destrukturyzację w rozdziale 19. Na razie wiedz, że używanie instrukcji `let`
w ten sposób jest wygodnym podejściem do wyodrębniania części krotki zwróconej
przez `mpsc::channel`.

Przenieśmy koniec nadawczy do utworzonego wątku i sprawmy, aby wysłał jeden
ciąg, tak aby utworzony wątek komunikował się z wątkiem głównym, jak pokazano w
Listingu 16-7. To tak, jakby wrzucić gumową kaczkę do rzeki w górę rzeki lub
wysyłać wiadomość czatu z jednego wątku do drugiego.

<Listing number="16-7" file-name="src/main.rs" caption="Moving `tx` to a spawned thread and sending “hi”">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-07/src/main.rs}}
```

</Listing>

Ponownie używamy `thread::spawn`, aby utworzyć nowy wątek, a następnie używamy `move`,
aby przenieść `tx` do zamknięcia, tak aby utworzony wątek był właścicielem `tx`. Utworzony
wątek musi być właścicielem nadajnika, aby móc wysyłać wiadomości przez
kanał. Nadajnik ma metodę `send`, która przyjmuje wartość, którą chcemy
wysłać. Metoda `send` zwraca typ `Result<T, E>`, więc jeśli odbiorca
został już usunięty i nie ma gdzie wysłać wartości, operacja wysyłania
zwróci błąd. W tym przykładzie wywołujemy `unwrap`, aby panikować w przypadku
błędu. Ale w prawdziwej aplikacji obsłużylibyśmy to prawidłowo: wróć do
rozdziału 9, aby przejrzeć strategie prawidłowej obsługi błędów.

W liście 16-8 pobierzemy wartość od odbiornika w wątku głównym. To
przypomina wyciągnięcie gumowej kaczki z wody na końcu rzeki lub
odebranie wiadomości na czacie.

<Listing number="16-8" file-name="src/main.rs" caption="Receiving the value “hi” in the main thread and printing it">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-08/src/main.rs}}
```

</Listing>

Odbiornik ma dwie przydatne metody: `recv` i `try_recv`. Używamy `recv`,
skrótu od *receive*, który zablokuje wykonywanie wątku głównego i zaczeka,
aż wartość zostanie wysłana kanałem. Po wysłaniu wartości `recv`
zwróci ją w `Result<T, E>`. Po zamknięciu nadajnika `recv` zwróci
błąd, aby zasygnalizować, że nie będzie już żadnych wartości.

Metoda `try_recv` nie blokuje, ale zamiast tego natychmiast zwróci `Result<T, E>`: wartość `Ok` zawierającą wiadomość, jeśli jest dostępna, i wartość `Err`,
jeśli tym razem nie ma żadnych wiadomości. Użycie `try_recv` jest przydatne, jeśli
ten wątek ma inną pracę do wykonania podczas oczekiwania na wiadomości: moglibyśmy napisać pętlę, która wywołuje `try_recv` co jakiś czas, obsługuje wiadomość, jeśli jest dostępna, a w przeciwnym razie wykonuje inną pracę przez chwilę, aż do ponownego sprawdzenia.

Użyliśmy `recv` w tym przykładzie dla uproszczenia; nie mamy żadnej innej pracy do wykonania przez wątek główny,
oprócz oczekiwania na wiadomości, więc zablokowanie wątku głównego jest właściwe.

Gdy uruchomimy kod z Listingu 16-8, zobaczymy wartość wydrukowaną z wątku głównego:

<!-- Nie wyodrębniam danych wyjściowych, ponieważ zmiany w tych danych wyjściowych nie są znaczące;
zmiany prawdopodobnie wynikają z faktu, że wątki działają inaczej, a nie ze
zmian w kompilatorze -->

```text
Got: hi
```

Perfect!

### Kanały i przeniesienie własności

Reguły własności odgrywają kluczową rolę w wysyłaniu wiadomości, ponieważ pomagają
pisać bezpieczny, współbieżny kod. Zapobieganie błędom w programowaniu współbieżnym jest
zaletą myślenia o własności w programach Rust. Przeprowadźmy
eksperyment, aby pokazać, jak kanały i własność współpracują ze sobą, aby zapobiegać
problemom: spróbujemy użyć wartości `val` w utworzonym wątku *po* wysłaniu jej kanałem. Spróbuj skompilować kod z Listingu 16-9, aby zobaczyć, dlaczego
ten kod nie jest dozwolony:

<Listing number="16-9" file-name="src/main.rs" caption="Attempting to use `val` after we’ve sent it down the channel">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-09/src/main.rs}}
```

</Listing>

Tutaj próbujemy wydrukować `val` po wysłaniu go kanałem przez `tx.send`.
Zezwalanie na to byłoby złym pomysłem: po wysłaniu wartości do innego
wątku, wątek ten mógłby ją zmodyfikować lub usunąć, zanim spróbujemy jej ponownie użyć. Potencjalnie modyfikacje innego wątku mogłyby spowodować błędy lub
nieoczekiwane wyniki z powodu niespójnych lub nieistniejących danych. Jednak Rust
wyświetla nam błąd, jeśli próbujemy skompilować kod z Listingu 16-9:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-09/output.txt}}
```

Nasz błąd współbieżności spowodował błąd kompilacji. Funkcja `send`
przejmuje własność swojego parametru, a gdy wartość jest przenoszona, odbiorca
przejmuje jej własność. Zapobiega to przypadkowemu użyciu wartości ponownie
po jej wysłaniu; system własności sprawdza, czy wszystko jest w porządku.

### Wysyłanie wielu wartości i widzenie odbiorcy oczekującego

Kod w Liście 16-8 został skompilowany i uruchomiony, ale nie pokazał nam wyraźnie, że
dwa oddzielne wątki komunikowały się ze sobą przez kanał. W Liście
16-10 wprowadziliśmy pewne modyfikacje, które udowodnią, że kod w Liście 16-8
działa jednocześnie: utworzony wątek będzie teraz wysyłał wiele wiadomości i
zatrzymywał się na sekundę między każdą wiadomością.

<Listing number="16-10" file-name="src/main.rs" caption="Sending multiple messages and pausing between each">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-10/src/main.rs}}
```

</Listing>

Tym razem utworzony wątek ma wektor ciągów, które chcemy wysłać do
wątku głównego. Przechodzimy przez nie, wysyłając każdy z osobna i pauzujemy
pomiędzy każdym, wywołując funkcję `thread::sleep` z wartością `Duration` równą
1 sekundzie.

W wątku głównym nie wywołujemy już jawnie funkcji `recv`:
zamiast tego traktujemy `rx` jako iterator. Dla każdej otrzymanej wartości ją
drukujemy. Po zamknięciu kanału iteracja zostanie zakończona.

Podczas uruchamiania kodu z Listingu 16-10 powinieneś zobaczyć następujący wynik
z 1-sekundową przerwą między każdym wierszem:

<!-- Nie wyodrębniam wyniku, ponieważ zmiany w tym wyniku nie są znaczące;
zmiany prawdopodobnie wynikają z tego, że wątki działają inaczej, a nie
ze zmian w kompilatorze -->

```text
Got: hi
Got: from
Got: the
Got: thread
```

Ponieważ nie mamy żadnego kodu, który zatrzymuje lub opóźnia pętlę `for` w wątku
głównym, możemy stwierdzić, że wątek główny czeka na otrzymanie wartości z wątku
utworzonego.

### Tworzenie wielu producentów poprzez klonowanie nadajnika

Wcześniej wspomnieliśmy, że `mpsc` to akronim od *multiple producers,
single consumer*. Wykorzystajmy `mpsc` i rozszerzmy kod z Listingu 16-10,
aby utworzyć wiele wątków, które wszystkie wysyłają wartości do tego samego odbiornika. Możemy to zrobić,
klonując nadajnik, jak pokazano na Listingu 16-11:

<Listing number="16-11" file-name="src/main.rs" caption="Sending multiple messages from multiple producers">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-11/src/main.rs:here}}
```

</Listing>

Tym razem, zanim utworzymy pierwszy utworzony wątek, wywołujemy `clone` na
nadajniku. To da nam nowy nadajnik, który możemy przekazać do pierwszego utworzonego wątku. Przekazujemy oryginalny nadajnik do drugiego utworzonego wątku.
To da nam dwa wątki, z których każdy wysyła różne wiadomości do jednego odbiorcy.

Po uruchomieniu kodu, Twoje dane wyjściowe powinny wyglądać mniej więcej tak:

<!-- Nie wyodrębniam danych wyjściowych, ponieważ zmiany w tych danych wyjściowych nie są znaczące;
zmiany te prawdopodobnie wynikają z faktu, że wątki działają inaczej, a nie ze
zmian w kompilatorze -->

```text
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```

Możesz zobaczyć wartości w innej kolejności, w zależności od systemu. To właśnie sprawia, że ​​współbieżność jest zarówno interesująca, jak i trudna. Jeśli poeksperymentujesz z
`thread::sleep`, nadając mu różne wartości w różnych wątkach, każde uruchomienie
będzie bardziej niedeterministyczne i za każdym razem stworzy inne dane wyjściowe.

Teraz, gdy przyjrzeliśmy się, jak działają kanały, przyjrzyjmy się innej metodzie
współbieżności.
