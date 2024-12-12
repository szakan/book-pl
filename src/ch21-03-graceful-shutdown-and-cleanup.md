## Łagodne zamykanie i czyszczenie

Kod w Listingu 21-20 odpowiada na żądania asynchronicznie poprzez
użycie puli wątków, zgodnie z naszym zamierzeniem. Otrzymujemy pewne ostrzeżenia dotyczące pól `workers`,
`id` i `thread`, których nie używamy w sposób bezpośredni, co przypomina nam,
że niczego nie czyścimy. Gdy używamy mniej eleganckiej metody
<kbd>ctrl</kbd>-<kbd>c</kbd>, aby zatrzymać wątek główny, wszystkie inne wątki
również są natychmiast zatrzymywane, nawet jeśli są w trakcie obsługi
żądania.

Następnie zaimplementujemy cechę `Drop`, aby wywołać `join` w każdym z
wątków w puli, aby mogły zakończyć żądania, nad którymi pracują, przed
zamknięciem. Następnie zaimplementujemy sposób, aby powiedzieć wątkom, że powinny przestać
akceptować nowe żądania i się wyłączyć. Aby zobaczyć ten kod w akcji, zmodyfikujemy
nasz serwer tak, aby akceptował tylko dwa żądania przed łagodnym zamknięciem
puli wątków.

### Implementacja cechy `Drop` w `ThreadPool`

Zacznijmy od zaimplementowania `Drop` w naszej puli wątków. Kiedy pula zostanie
usunięta, wszystkie nasze wątki powinny się połączyć, aby mieć pewność, że zakończą swoją pracę.
Listing 21-22 pokazuje pierwszą próbę implementacji `Drop`; ten kod nie będzie jeszcze działał.

<Listing number="21-22" file-name="src/lib.rs" caption="Joining each thread when the thread pool goes out of scope">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-22/src/lib.rs:here}}
```

</Listing>

Najpierw przechodzimy przez każdy z wątków puli `workers`. Używamy `&mut`,
ponieważ `self` jest zmienną referencją, a także musimy być w stanie
zmutować `worker`. Dla każdego wątku wyświetlamy komunikat informujący, że dany
konkretny wątek jest wyłączany, a następnie wywołujemy `join` w wątku tego wątku. Jeśli wywołanie `join` się nie powiedzie, używamy `unwrap`, aby Rust wpadł w panikę i przeszedł
w nieeleganckie wyłączenie.

Oto błąd, który otrzymujemy, gdy kompilujemy ten kod:

```console
{{#include ../listings/ch21-web-server/listing-21-22/output.txt}}
```

Błąd mówi nam, że nie możemy wywołać `join`, ponieważ mamy tylko zmienną pożyczkę
każdego `worker`, a `join` przejmuje własność jego argumentu. Aby rozwiązać ten
problem, musimy przenieść wątek z instancji `Worker`, która jest właścicielem
`thread`, aby `join` mógł wykorzystać wątek. Zrobiliśmy to w Liście 17-15: jeśli
`Worker` posiada `Option<thread::JoinHandle<()>>`, możemy wywołać metodę
`take` na `Option`, aby przenieść wartość z wariantu `Some` i
pozostawić wariant `None` na jego miejscu. Innymi słowy, `Worker`, który jest uruchomiony,
będzie miał wariant `Some` w `thread`, a gdy będziemy chcieli wyczyścić
`Worker`, zastąpimy `Some` przez `None`, aby `Worker` nie miał
wątku do uruchomienia.

Wiemy więc, że chcemy zaktualizować definicję `Worker` w następujący sposób:

<Listing file-name="src/lib.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/no-listing-04-update-worker-definition/src/lib.rs:here}}
```

</Listing>

Now let’s lean on the compiler to find the other places that need to change.
Checking this code, we get two errors:

```console
{{#include ../listings/ch21-web-server/no-listing-04-update-worker-definition/output.txt}}
```

Zajmijmy się drugim błędem, który wskazuje na kod na końcu
`Worker::new`; musimy owinąć wartość `thread` w `Some`, gdy tworzymy
nowego `Worker`. Wprowadź następujące zmiany, aby naprawić ten błąd:

<Listing file-name="src/lib.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/no-listing-05-fix-worker-new/src/lib.rs:here}}
```

</Listing>

Pierwszy błąd jest w naszej implementacji `Drop`. Wspomnieliśmy wcześniej, że
zamierzaliśmy wywołać `take` na wartości `Option`, aby przenieść `thread` z `worker`.
Następujące zmiany to zrobią:

<Listing file-name="src/lib.rs">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/no-listing-06-fix-threadpool-drop/src/lib.rs:here}}
```

</Listing>

Jak omówiono w rozdziale 18, metoda `take` w `Option` usuwa wariant `Some`
i pozostawia `None` na jego miejscu. Używamy `if let`, aby zdestrukturyzować `Some` i pobrać wątek; następnie wywołujemy `join` w wątku. Jeśli wątek pracownika jest już `None`, wiemy, że wątek pracownika został już wyczyszczony, więc w takim przypadku nic się nie dzieje.

### Sygnalizowanie wątkom zaprzestania nasłuchiwania zadań

Przy wszystkich zmianach, które wprowadziliśmy, nasz kod kompiluje się bez żadnych ostrzeżeń.
Jednak zła wiadomość jest taka, że ​​ten kod nie działa jeszcze tak, jak byśmy chcieli.
Kluczem jest logika zamknięć uruchamianych przez wątki instancji `Worker`
: w tej chwili wywołujemy `join`, ale to nie zamknie wątków,
ponieważ `loop` w nieskończoność szukają zadań. Jeśli spróbujemy usunąć nasz
`ThreadPool` za pomocą naszej obecnej implementacji `drop`, wątek główny
zablokuje się w nieskończoność, czekając na zakończenie pierwszego wątku.

Aby rozwiązać ten problem, będziemy potrzebować zmiany w implementacji `drop` `ThreadPool`, a następnie zmiany w pętli `Worker`.

Najpierw zmienimy implementację `drop` `ThreadPool`, aby jawnie usunąć `sender` przed oczekiwaniem na zakończenie wątków. Listing 21-23 pokazuje
zmiany w `ThreadPool`, aby jawnie usunąć `sender`. Używamy tej samej techniki `Option`
i `take`, co w przypadku wątku, aby móc przenieść `sender` z `ThreadPool`:

<Listing number="21-23" file-name="src/lib.rs" caption="Explicitly drop `sender` before joining the worker threads">

```rust,noplayground,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-23/src/lib.rs:here}}
```

</Listing>

Usunięcie `sender` zamyka kanał, co oznacza, że ​​nie zostaną wysłane żadne wiadomości. Gdy tak się stanie, wszystkie wywołania `recv` wykonywane przez pracowników w nieskończonej pętli zwrócą błąd. W listingu 21-24 zmieniamy pętlę `Worker`, aby w takim przypadku łagodnie wyjść z pętli, co oznacza, że ​​wątki zakończą się, gdy implementacja `ThreadPool` `drop` wywoła na nich `join`.

<Listing number="21-24" file-name="src/lib.rs" caption="Explicitly break out of the loop when `recv` returns an error">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-24/src/lib.rs:here}}
```

</Listing>

Aby zobaczyć ten kod w akcji, zmodyfikujmy `main` tak, aby akceptował tylko dwa żądania przed prawidłowym wyłączeniem serwera, jak pokazano na liście 21-25.

<Listing number="21-25" file-name="src/main.rs" caption="Shut down the server after serving two requests by exiting the loop">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/listing-21-25/src/main.rs:here}}
```

</Listing>

Nie chciałbyś, aby rzeczywisty serwer WWW wyłączył się po obsłużeniu zaledwie dwóch
żądań. Ten kod pokazuje tylko, że łagodne wyłączenie i czyszczenie działają.

Metoda `take` jest zdefiniowana w cesze `Iterator` i ogranicza iterację
maksymalnie do dwóch pierwszych elementów. `ThreadPool` wyjdzie poza zakres na
końcu `main`, a implementacja `drop` zostanie uruchomiona.

Uruchom serwer za pomocą `cargo run` i wyślij trzy żądania. Trzecie żądanie
powinno spowodować błąd, a w terminalu powinieneś zobaczyć wynik podobny do tego:

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-25
cargo run
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
trzecie żądanie spowoduje błąd, ponieważ serwer zostanie wyłączony
skopiuj dane wyjściowe poniżej
Nie można zautomatyzować, ponieważ dane wyjściowe zależą od tworzenia żądań
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 1.0s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

Możesz zobaczyć inną kolejność wydrukowanych pracowników i wiadomości. Możemy zobaczyć,
jak działa ten kod na podstawie wiadomości: pracownicy 0 i 3 otrzymali pierwsze dwa
żądania. Serwer przestał akceptować połączenia po drugim połączeniu,
a implementacja `Drop` w `ThreadPool` zaczyna się wykonywać, zanim pracownik 3
nawet rozpocznie swoje zadanie. Usunięcie `sender` rozłącza wszystkich pracowników i
nakazuje im się wyłączyć. Każdy z pracowników drukuje wiadomość, gdy się rozłącza,
a następnie pula wątków wywołuje `join`, aby poczekać, aż każdy wątek roboczy zakończy działanie.

Zauważ jeden interesujący aspekt tego konkretnego wykonania: `ThreadPool`
usunął `sender` i zanim którykolwiek z pracowników otrzymał błąd, próbowaliśmy dołączyć
do pracownika 0. Pracownik 0 nie otrzymał jeszcze błędu od `recv`, więc wątek główny
zablokował się, czekając, aż pracownik 0 zakończy działanie. W międzyczasie pracownik 3 otrzymał
zadanie, a następnie wszystkie wątki otrzymały błąd. Gdy pracownik 0 zakończył działanie, wątek
główny czekał, aż reszta pracowników zakończy działanie. W tym momencie
wszyscy wyszli ze swoich pętli i zatrzymali się.

Gratulacje! Zakończyliśmy nasz projekt; mamy podstawowy serwer WWW, który używa
puli wątków do asynchronicznej odpowiedzi. Możemy wykonać łagodne
wyłączenie serwera, które czyści wszystkie wątki w puli.

Oto pełny kod w celach informacyjnych:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/main.rs}}
```

</Listing>

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/lib.rs}}
```

</Listing>

Tutaj moglibyśmy zrobić więcej! Jeśli chcesz dalej ulepszać ten projekt, oto
kilka pomysłów:

* Dodaj więcej dokumentacji do `ThreadPool` i jego publicznych metod.
* Dodaj testy funkcjonalności biblioteki.
* Zmień wywołania `unwrap` na bardziej niezawodną obsługę błędów.
* Użyj `ThreadPool` do wykonywania innych zadań niż obsługa żądań internetowych.
* Znajdź skrzynię puli wątków na [crates.io](https://crates.io/) i zaimplementuj
podobny serwer internetowy, używając skrzyni. Następnie porównaj jego API i
niezawodność z pulą wątków, którą zaimplementowaliśmy.

## Streszczenie

Dobra robota! Dotarłeś do końca książki! Chcemy Ci podziękować za
dołączenie do nas w tej wycieczce po Rust. Jesteś teraz gotowy, aby wdrażać własne
projekty Rust i pomagać w projektach innych osób. Pamiętaj, że istnieje
przyjazna społeczność innych Rustaceanów, którzy chętnie pomogą Ci w każdym
wyzwaniu, jakie napotkasz w swojej podróży po Rust.
