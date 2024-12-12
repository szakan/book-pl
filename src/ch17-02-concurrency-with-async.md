## Współbieżność z asynchronicznością

W tej sekcji zastosujemy asynchroniczność do niektórych z tych samych wyzwań współbieżności,
z którymi mierzyliśmy się w przypadku wątków w rozdziale 16. Ponieważ omówiliśmy już wiele
kluczowych idei, w tej sekcji skupimy się na tym, co różni
wątki od przyszłości.

W wielu przypadkach interfejsy API do pracy ze współbieżnością przy użyciu asynchroniczności są bardzo
podobne do tych do używania wątków. W innych przypadkach kończą się na ukształtowaniu
zupełnie inaczej. Nawet jeśli interfejsy API *wyglądają* podobnie między wątkami a asynchronicznością,
często mają inne zachowanie — i prawie zawsze mają inne
cechy wydajności.

### Rachunkowość

Pierwszym zadaniem, z którym zmierzyliśmy się w rozdziale 16, było zliczanie w dwóch oddzielnych wątkach.
Zróbmy to samo, używając async. Skrzynia `trpl` dostarcza funkcję `spawn_task`,
która wygląda bardzo podobnie do API `thread::spawn`, oraz funkcję `sleep`,
która jest asynchroniczną wersją API `thread::sleep`. Możemy ich użyć razem,
aby zaimplementować ten sam przykład zliczania, co w przypadku wątków, w Liście 17-6.

<Listing number="17-6" caption="Using `spawn_task` to count with two" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-06/src/main.rs:all}}
```

</Listing>

Jako punkt wyjścia ustawiamy naszą funkcję `main` za pomocą `trpl::run`, tak
żeby nasza funkcja najwyższego poziomu mogła być asynchroniczna.

> Uwaga: od tego momentu w rozdziale każdy przykład będzie zawierał ten
> dokładnie ten sam kod opakowujący z `trpl::run` w `main`, więc często będziemy go pomijać
> tak jak robimy to z `main`. Nie zapomnij uwzględnić go w swoim kodzie!

Następnie piszemy dwie pętle w tym bloku, każda z wywołaniem `trpl::sleep`,
które czeka pół sekundy (500 milisekund) przed wysłaniem następnej
wiadomości. Umieszczamy jedną pętlę w ciele `trpl::spawn_task`, a drugą w pętli `for` najwyższego poziomu. Dodajemy również `await` po wywołaniu `sleep`.

To robi coś podobnego do implementacji opartej na wątkach — w tym
fakt, że możesz zobaczyć wiadomości pojawiające się w innej kolejności w swoim własnym
terminalu, gdy go uruchomisz.

<!-- Nie wyodrębniam danych wyjściowych, ponieważ zmiany w tych danych wyjściowych nie są znaczące;
zmiany prawdopodobnie wynikają z faktu, że wątki działają inaczej, a nie ze
zmian w kompilatorze -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
```

Ta wersja zatrzymuje się, gdy tylko pętla for w ciele głównego bloku asynchronicznego
zakończy się, ponieważ zadanie utworzone przez `spawn_task` jest zamykane, gdy kończy się główna
funkcja. Jeśli chcesz działać aż do zakończenia zadania,
musisz użyć uchwytu łączenia, aby poczekać na zakończenie pierwszego zadania. W przypadku
wątków użyliśmy metody `join` do „blokowania”, dopóki wątek nie zakończy działania.
W Liście 17-7 możemy użyć `await`, aby zrobić to samo, ponieważ sam uchwyt zadania
jest przyszłością. Jego typ `Output` jest `Result`, więc również go rozwijamy
po oczekiwaniu.

<Listing number="17-7" caption="Using `await` with a join handle to run a task to completion" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-07/src/main.rs:handle}}
```

</Listing>

Ta zaktualizowana wersja działa, dopóki *obie* pętle się nie zakończą.

<!-- Nie wyodrębniam danych wyjściowych, ponieważ zmiany w tych danych wyjściowych nie są znaczące;
zmiany te prawdopodobnie wynikają z faktu, że wątki działają inaczej, a nie ze
zmian w kompilatorze -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

Jak dotąd wygląda na to, że asynchroniczność i wątki dają nam te same podstawowe wyniki, tylko
z inną składnią: używając `await` zamiast wywoływania `join` na uchwycie łączenia i oczekując na wywołania `sleep`.

Większą różnicą jest to, że nie musieliśmy tworzyć kolejnego wątku systemu operacyjnego, aby to zrobić. Właściwie nie musimy nawet tworzyć tutaj zadania. Ponieważ
bloki asynchroniczne kompilują się do anonimowych futures, możemy umieścić każdą pętlę w bloku asynchronicznym i pozwolić środowisku wykonawczemu wykonać je obie do końca za pomocą funkcji `trpl::join`.

W rozdziale 16 pokazaliśmy, jak używać metody `join` w typie `JoinHandle`
zwracanym po wywołaniu `std::thread::spawn`. Funkcja `trpl::join` jest
podobna, ale dla futures. Gdy podasz mu dwie przyszłości, wygeneruje jedną nową
przyszłość, której wyjście jest krotką z wyjściem każdej z przekazanych przyszłości,
gdy *oba* się zakończą. Tak więc w Listingu 17-8 używamy `trpl::join`, aby czekać,
aż zarówno `fut1`, jak i `fut2` się zakończą. *Nie* czekamy na `fut1` i `fut2`, ale
zamiast tego na nową przyszłość wygenerowaną przez `trpl::join`. Ignorujemy wyjście,
ponieważ jest to po prostu krotka z dwiema wartościami jednostek.

<Listing number="17-8" caption="Using `trpl::join` to await two anonymous futures" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-08/src/main.rs:join}}
```

</Listing>

Gdy to uruchomimy, zobaczymy, że obie przyszłości dobiegają końca:

<!-- Nie wyodrębniam danych wyjściowych, ponieważ zmiany w tych danych wyjściowych nie są znaczące;
zmiany te prawdopodobnie wynikają z faktu, że wątki działają inaczej, a nie ze
zmian w kompilatorze -->

```text
hi number 1 from the first task!
hi number 1 from the second task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

Tutaj zobaczysz dokładnie tę samą kolejność za każdym razem, co bardzo różni się od tego, co widzieliśmy w przypadku wątków. Dzieje się tak, ponieważ funkcja `trpl::join` jest *sprawiedliwa*,
co oznacza, że ​​sprawdza każdą przyszłość równie często, naprzemiennie między nimi i nigdy
nie pozwala, aby jedna wyprzedziła drugą, jeśli druga jest gotowa. W przypadku wątków system operacyjny
decyduje, który wątek sprawdzić i jak długo pozwolić mu działać. W przypadku asynchronicznego Rust środowisko wykonawcze
decyduje, które zadanie sprawdzić. (W praktyce szczegóły stają się skomplikowane,
ponieważ asynchroniczne środowisko wykonawcze może używać wątków systemu operacyjnego pod maską jako
części sposobu zarządzania współbieżnością, więc zagwarantowanie uczciwości może być bardziej pracochłonne dla środowiska wykonawczego — ale nadal jest możliwe!) Środowiska wykonawcze nie muszą gwarantować
uczciwości dla żadnej danej operacji, a często oferują różne interfejsy API, aby umożliwić
wybór, czy chcesz uczciwości, czy nie.

Wypróbuj kilka z tych różnych wariantów oczekiwania na przyszłość i zobacz, co one
robią:

* Usuń blok asynchroniczny z jednej lub obu pętli.
* Oczekuj na każdy blok asynchroniczny natychmiast po jego zdefiniowaniu.
* Owiń tylko pierwszą pętlę w blok asynchroniczny i oczekuj na wynikową przyszłość
po ciele drugiej pętli.

Aby zwiększyć wyzwanie, sprawdź, czy potrafisz ustalić, jaki będzie wynik w
każdym przypadku *przed* uruchomieniem kodu!

### Przekazywanie wiadomości

Współdzielenie danych między obiektami przyszłości również będzie znajome: ponownie użyjemy przekazywania wiadomości,
ale tym razem z asynchronicznymi wersjami typów i funkcji. Podążymy
nieco inną ścieżką niż w rozdziale 16, aby zilustrować niektóre z kluczowych
różnic między współbieżnością opartą na wątkach i na obiektach przyszłości. W Liście 17-9,
zaczniemy od pojedynczego bloku asynchronicznego — *nie* tworząc oddzielnego zadania, ponieważ
utworzyliśmy oddzielny wątek.

<Listing number="17-9" caption="Creating an async channel and assigning the two halves to `tx` and `rx`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-09/src/main.rs:channel}}
```

</Listing>

Tutaj używamy `trpl::channel`, asynchronicznej wersji API kanału wieloproducenta,
jednego konsumenta, którego używaliśmy z wątkami w rozdziale 16. Asynchroniczna
wersja API różni się tylko trochę od wersji opartej na wątkach:
używa zmiennego, a nie niezmiennego odbiornika `rx`, a jego metoda `recv`
produkuje przyszłość, na którą musimy czekać, zamiast generować wartość bezpośrednio. Teraz
możemy wysyłać wiadomości od nadawcy do odbiorcy. Zauważ, że nie musimy
tworzyć osobnego wątku ani nawet zadania; musimy jedynie czekać na wywołanie `rx.recv`.

Synchroniczna metoda `Receiver::recv` w `std::mpsc::channel` blokuje się, dopóki
nie otrzyma wiadomości. Metoda `trpl::Receiver::recv` tego nie robi, ponieważ
jest asynchroniczna. Zamiast blokować, przekazuje kontrolę z powrotem do środowiska wykonawczego, aż do momentu, gdy
wiadomość zostanie odebrana lub strona wysyłająca kanału zostanie zamknięta. Natomiast nie
czekamy na wywołanie `send`, ponieważ ono nie blokuje. Nie musi tego robić,
ponieważ kanał, do którego je wysyłamy, jest nieograniczony.

> Uwaga: Ponieważ cały ten kod asynchroniczny jest uruchamiany w bloku asynchronicznym w wywołaniu `trpl::run`
>, wszystko w nim może uniknąć blokowania. Jednak kod *poza* nim
> zostanie zablokowany po zwróceniu funkcji `run`. To jest cały sens funkcji
> `trpl::run`: pozwala ona *wybrać*, gdzie zablokować pewien zestaw kodu asynchronicznego,
> a tym samym gdzie przejść między kodem synchronicznym a asynchronicznym. W większości środowisk wykonawczych asynchronicznych,
> `run` jest w rzeczywistości nazywane `block_on` właśnie z tego powodu.

Zwróć uwagę na dwie rzeczy w tym przykładzie: Po pierwsze, wiadomość dotrze od razu!
Po drugie, chociaż używamy tutaj przyszłości, nie ma jeszcze współbieżności. Wszystko
na liście dzieje się po kolei, tak jak gdyby nie było żadnych przyszłości.

Zajmijmy się pierwszą częścią, wysyłając serię wiadomości i śpiąc
między nimi, jak pokazano w Liście 17-10:

<!-- We cannot test this one because it never stops! -->

<Listing number="17-10" caption="Sending and receiving multiple messages over the async channel and sleeping with an `await` between each message" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-10/src/main.rs:many-messages}}
```

</Listing>

Oprócz wysyłania wiadomości, musimy je również odbierać. W tym przypadku,
możemy to zrobić ręcznie, po prostu wykonując `rx.recv().await` cztery razy, ponieważ
wiemy, ile wiadomości przychodzi. W rzeczywistości, jednak,
zwykle będziemy czekać na jakąś *nieznaną* liczbę wiadomości. W takim przypadku, musimy
czekać, aż ustalimy, że nie ma więcej wiadomości.

W Listingu 16-10, użyliśmy pętli `for` do przetworzenia wszystkich elementów otrzymanych z
synchronicznego kanału. Jednak Rust nie ma jeszcze sposobu na napisanie pętli `for`
na *asynchronicznej* serii elementów. Zamiast tego, musimy użyć nowego rodzaju pętli,
której wcześniej nie widzieliśmy, pętli warunkowej `while let`. Pętla `while let`
to wersja pętli konstrukcji `if let`, którą widzieliśmy w rozdziale 6. Pętla
będzie się wykonywać tak długo, jak długo określony przez nią wzorzec
będzie pasował do wartości.

Wywołanie `rx.recv` generuje `Future`, na którą czekamy. Środowisko wykonawcze wstrzyma `Future`, dopóki nie będzie gotowe. Po nadejściu wiadomości przyszłość
zostanie rozwiązana na `Some(message)` tyle razy, ile wiadomości zostanie nadejściu. Po zamknięciu kanału,
niezależnie od tego, czy *jakieś* wiadomości nadeszły, przyszłość
zostanie rozwiązana na `None`, aby wskazać, że nie ma więcej wartości i powinniśmy
zakończyć sondowanie — to znaczy zakończyć oczekiwanie.

Pętla `while let` łączy to wszystko razem. Jeśli wynikiem wywołania
`rx.recv().await` jest `Some(message)`, uzyskujemy dostęp do wiadomości i możemy jej
użyć w ciele pętli, tak jak moglibyśmy to zrobić za pomocą `if let`. Jeśli wynikiem jest
`None`, pętla się kończy. Za każdym razem, gdy pętla się kończy, ponownie osiąga punkt oczekiwania,
więc środowisko wykonawcze ją wstrzymuje, aż do momentu nadejścia kolejnej wiadomości.

Kod teraz pomyślnie wysyła i odbiera wszystkie wiadomości. Niestety,
nadal jest kilka problemów. Po pierwsze, wiadomości nie przychodzą w
półsekundowych odstępach. Przychodzą wszystkie naraz, dwie sekundy (2000 milisekund)
po uruchomieniu programu. Po drugie, ten program nigdy się nie kończy! Zamiast tego,
czeka wiecznie na nowe wiadomości. Będziesz musiał go wyłączyć za pomocą <span
class="keystroke">ctrl-c</span>.

Zacznijmy od zrozumienia, dlaczego wszystkie wiadomości przychodzą jednocześnie po pełnym
opóźnieniu, a nie z opóźnieniami między każdą z nich. W ramach danego
bloku asynchronicznego kolejność, w jakiej słowa kluczowe `await` pojawiają się w kodzie, jest również kolejnością, w jakiej występują podczas uruchamiania programu.

W Liście 17-10 znajduje się tylko jeden blok asynchroniczny, więc wszystko w nim działa
liniowo. Nadal nie ma współbieżności. Wszystkie wywołania `tx.send` są wykonywane,
przeplatane wszystkimi wywołaniami `trpl::sleep` i ich powiązanymi punktami
oczekiwania. Dopiero wtedy pętla `while let` może przejść przez dowolny z punktów `await` w wywołaniach `recv`.

Aby uzyskać pożądane zachowanie, w którym opóźnienie uśpienia występuje między otrzymaniem
każdej wiadomości, musimy umieścić operacje `tx` i `rx` w ich własnych blokach asynchronicznych. Następnie środowisko wykonawcze może wykonać każdy z nich osobno, używając `trpl::join`,
tak jak w przykładzie zliczania. Ponownie czekamy na wynik wywołania
`trpl::join`, a nie na poszczególne futures. Gdybyśmy oczekiwali na poszczególne futures
po kolei, skończylibyśmy z powrotem w przepływie sekwencyjnym — dokładnie tego, czego
próbujemy *nie* robić.

<!-- Nie możemy tego przetestować, ponieważ to się nigdy nie kończy! -->

<Listing number="17-11" caption="Separating `send` and `recv` into their own `async` blocks and awaiting the futures for those blocks" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-11/src/main.rs:futures}}
```

</Listing>

Dzięki zaktualizowanemu kodowi w Listingu 17-11 wiadomości są drukowane w odstępach 500-milisekundowych, a nie w pośpiechu po dwóch sekundach.

Program nadal się nie kończy, ze względu na sposób, w jaki pętla `while let`
współdziała z pętlą `trpl::join`:

* Przyszłość zwrócona z `trpl::join` kończy się dopiero po zakończeniu *obu* przyszłości
przekazanych do niej.
* Przyszłość `tx` kończy się po zakończeniu uśpienia po wysłaniu ostatniej
wiadomości w `vals`.
* Przyszłość `rx` nie zakończy się, dopóki pętla `while let` się nie zakończy.
* Pętla `while let` nie zakończy się, dopóki oczekiwanie na `rx.recv` nie wygeneruje `None`.
* Oczekiwanie na `rx.recv` zwróci `None` dopiero po zamknięciu drugiego końca kanału.
* Kanał zostanie zamknięty tylko wtedy, gdy wywołamy `rx.close` lub gdy strona nadawcy,
`tx`, zostanie pominięta.
* Nie wywołujemy `rx.close` nigdzie, a `tx` nie zostanie pominięte, dopóki
najbardziej zewnętrzny blok asynchroniczny przekazany do `trpl::run` się nie zakończy.
* Blok nie może się zakończyć, ponieważ jest blokowany po zakończeniu `trpl::join`, co
przenosi nas z powrotem na górę tej listy!

Możemy ręcznie zamknąć `rx`, wywołując `rx.close` gdzieś, ale
to nie ma większego sensu. Zatrzymanie po obsłużeniu dowolnej liczby wiadomości
spowodowałoby zamknięcie programu, ale moglibyśmy przegapić wiadomości. Potrzebujemy innego sposobu,
aby upewnić się, że `tx` zostanie pominięte *przed* końcem funkcji.

Obecnie blok asynchroniczny, w którym wysyłamy wiadomości, pożycza tylko `tx`, ponieważ
wysyłanie wiadomości nie wymaga własności, ale gdybyśmy mogli przenieść `tx` do
tego bloku asynchronicznego, zostałby on usunięty po zakończeniu tego bloku. W rozdziale 13
nauczyliśmy się, jak używać słowa kluczowego `move` z zamknięciami, a w rozdziale 16 zobaczyliśmy,
że często musimy przenosić dane do zamknięć podczas pracy z wątkami.
Ta sama podstawowa dynamika dotyczy bloków asynchronicznych, więc słowo kluczowe `move` działa z
blokami asynchronicznymi tak samo, jak z zamknięciami.

W Liście 17-12 zmieniamy blok asynchroniczny do wysyłania wiadomości ze zwykłego bloku
`async` na blok `async move`. Gdy uruchamiamy *tę* wersję kodu,
wyłącza się ona prawidłowo po wysłaniu i odebraniu ostatniej wiadomości.

<Listing number="17-12" caption="A working example of sending and receiving messages between futures which correctly shuts down when complete" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-12/src/main.rs:with-move}}
```

</Listing>

Ten kanał asynchroniczny jest również kanałem wielu producentów, więc możemy wywołać `clone`
na `tx`, jeśli chcemy wysyłać wiadomości z wielu futures. W Liście 17-13,
klonujemy `tx`, tworząc `tx1` poza pierwszym blokiem asynchronicznym. Przenosimy `tx1`
do tego bloku tak samo, jak zrobiliśmy to wcześniej z `tx`. Następnie, później, przenosimy oryginalny
`tx` do *nowego* bloku asynchronicznego, gdzie wysyłamy więcej wiadomości z nieco wolniejszym
opóźnieniem. Umieściliśmy ten nowy blok asynchroniczny po bloku asynchronicznym do odbierania
wiadomości, ale równie dobrze mógłby się znaleźć przed nim. Kluczem jest kolejność, w jakiej
futures są oczekiwane, a nie kolejność, w jakiej są tworzone.

Oba bloki asynchroniczne do wysyłania wiadomości muszą być blokami `async move`, tak,
żeby zarówno `tx`, jak i `tx1` zostały usunięte, gdy te bloki się zakończą. W przeciwnym razie
wrócimy do tej samej nieskończonej pętli, w której zaczęliśmy. Na koniec przełączamy się z
`trpl::join` na `trpl::join3`, aby obsłużyć dodatkową przyszłość.
<Listing number="17-13" caption="Using multiple producers with async blocks" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-13/src/main.rs:here}}
```

</Listing>

Teraz widzimy wszystkie wiadomości z obu wysyłających futures. Ponieważ wysyłające
futures używają nieco innych opóźnień po wysłaniu, wiadomości są również
odbierane w tych różnych odstępach czasu.

<!-- Nie wyodrębniam danych wyjściowych, ponieważ zmiany w tych danych wyjściowych nie są znaczące;
zmiany prawdopodobnie wynikają z faktu, że wątki działają inaczej, a nie
zmian w kompilatorze -->

```text
received 'hi'
received 'more'
received 'from'
received 'the'
received 'messages'
received 'future'
received 'for'
received 'you'
```

To dobry początek, ale ogranicza nas do zaledwie kilku przyszłości: dwóch z
`join` lub trzech z `join3`. Zobaczmy, jak moglibyśmy pracować z większą liczbą przyszłości.

[streams]: ch17-05-streams.html
