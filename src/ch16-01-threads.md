## Używanie wątków do jednoczesnego uruchamiania kodu

W większości obecnych systemów operacyjnych kod wykonywanego programu jest uruchamiany w
*procesie*, a system operacyjny będzie zarządzał wieloma procesami jednocześnie.
W ramach programu można mieć również niezależne części, które działają jednocześnie.
Funkcje, które uruchamiają te niezależne części, nazywane są *wątkami*. Na przykład serwer WWW może mieć wiele wątków, aby móc odpowiadać na
więcej niż jedno żądanie w tym samym czasie.

Podzielenie obliczeń w programie na wiele wątków w celu uruchomienia wielu
zadań w tym samym czasie może poprawić wydajność, ale również dodaje złożoności.
Ponieważ wątki mogą działać jednocześnie, nie ma nieodłącznej gwarancji co do kolejności, w jakiej części kodu w różnych wątkach będą uruchamiane. Może to prowadzić
do problemów, takich jak:

* Warunki wyścigu, w których wątki uzyskują dostęp do danych lub zasobów w
niespójnej kolejności
* Blokady, w których dwa wątki czekają na siebie, uniemożliwiając obu
wątkom kontynuowanie

Błędy, które występują tylko w określonych sytuacjach i są trudne do odtworzenia i naprawienia

Rust próbuje złagodzić negatywne skutki korzystania z wątków, ale
programowanie w kontekście wielowątkowym nadal wymaga starannego przemyślenia i
wymaga struktury kodu, która różni się od tej w programach działających w pojedynczym
wątku.

Języki programowania implementują wątki na kilka różnych sposobów, a wiele
systemów operacyjnych udostępnia API, które język może wywołać w celu tworzenia nowych wątków.
Standardowa biblioteka Rust wykorzystuje model *1:1* implementacji wątków, w którym
program wykorzystuje jeden wątek systemu operacyjnego na jeden wątek języka. Istnieją
skrzynie, które implementują inne modele wątków, które wprowadzają inne kompromisy niż
model 1:1. (System asynchroniczny Rusta, który zobaczymy w następnym rozdziale, zapewnia również inne podejście do współbieżności.)

### Creating a New Thread with `spawn`

Aby utworzyć nowy wątek, wywołujemy funkcję `thread::spawn` i przekazujemy jej zamknięcie (o zamknięciach mówiliśmy w rozdziale 13) zawierające kod, który chcemy uruchomić w nowym wątku. Przykład w listingu 16-1 drukuje tekst z wątku głównego i inny tekst z nowego wątku:

<Listing number="16-1" file-name="src/main.rs" caption="Creating a new thread to print one thing while the main thread prints something else">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-01/src/main.rs}}
```

</Listing>

Należy pamiętać, że po zakończeniu głównego wątku programu Rust wszystkie utworzone wątki
są wyłączane, niezależnie od tego, czy zakończyły działanie. Dane wyjściowe tego
programu mogą być za każdym razem nieco inne, ale będą wyglądać podobnie do
następującego:

<!-- Nie wyodrębniam danych wyjściowych, ponieważ zmiany w tych danych wyjściowych nie są znaczące;
zmiany te prawdopodobnie wynikają z faktu, że wątki działają inaczej, a nie
ze zmian w kompilatorze -->

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

Wywołania `thread::sleep` zmuszają wątek do zatrzymania wykonywania na krótki
czas, umożliwiając uruchomienie innego wątku. Wątki prawdopodobnie będą się
na zmianę wykonywać, ale nie jest to gwarantowane: zależy to od tego, jak system operacyjny
planuje wątki. W tym przebiegu wątek główny został wydrukowany jako pierwszy, mimo że
instrukcja drukowania z utworzonego wątku pojawia się jako pierwsza w kodzie. I mimo że
powiedzieliśmy utworzonemu wątkowi, aby drukował, dopóki `i` nie będzie równe 9, osiągnął on tylko 5,
zanim wątek główny został zamknięty.

Jeśli uruchomisz ten kod i zobaczysz tylko dane wyjściowe z wątku głównego lub nie zobaczysz żadnego
nakładania się, spróbuj zwiększyć liczby w zakresach, aby stworzyć więcej możliwości
dla systemu operacyjnego do przełączania się między wątkami.

### Waiting for All Threads to Finish Using `join` Handles

Kod w Listingu 16-1 nie tylko zatrzymuje utworzony wątek przedwcześnie w większości
czasów z powodu zakończenia wątku głównego, ale ponieważ nie ma gwarancji
kolejności uruchamiania wątków, nie możemy również zagwarantować, że utworzony wątek
w ogóle zostanie uruchomiony!

Możemy rozwiązać problem braku uruchomienia lub przedwczesnego zakończenia utworzonego wątku
poprzez zapisanie wartości zwracanej przez `thread::spawn` w zmiennej. Typem zwracanym przez
`thread::spawn` jest `JoinHandle`. `JoinHandle` to wartość będąca własnością, która, gdy
wywołamy na niej metodę `join`, będzie czekać na zakończenie swojego wątku. Listing 16-2
pokazuje, jak używać `JoinHandle` wątku utworzonego w Listingu 16-1 i
wywoływać `join`, aby upewnić się, że utworzony wątek zakończy się przed wyjściem z `main`:

<Listing number="16-2" file-name="src/main.rs" caption="Saving a `JoinHandle` from `thread::spawn` to guarantee the thread is run to completion">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-02/src/main.rs}}
```

</Listing>

Wywołanie `join` na uchwycie blokuje aktualnie działający wątek, dopóki
wątek reprezentowany przez uchwyt nie zakończy działania. *Zablokowanie* wątku oznacza, że
wątek nie może wykonywać pracy ani wychodzić. Ponieważ umieściliśmy wywołanie
`join` po pętli `for` wątku głównego, uruchomienie Listingu 16-2 powinno
wygenerować dane wyjściowe podobne do tego:

<!-- Nie wyodrębniam danych wyjściowych, ponieważ zmiany w tych danych wyjściowych nie są znaczące;
zmiany te prawdopodobnie wynikają z faktu, że wątki działają inaczej, a nie
ze zmian w kompilatorze -->

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

Dwa wątki nadal działają naprzemiennie, ale wątek główny czeka z powodu
wywołania `handle.join()` i nie kończy się, dopóki utworzony wątek nie zostanie ukończony.

Ale zobaczmy, co się stanie, gdy zamiast tego przeniesiemy `handle.join()` przed pętlę
`for` w `main`, w ten sposób:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/no-listing-01-join-too-early/src/main.rs}}
```

</Listing>

Główny wątek będzie czekał, aż utworzony wątek zakończy działanie, a następnie uruchomi swoją pętlę
`for`, więc dane wyjściowe nie będą już przeplatane, jak pokazano tutaj:

<!-- Nie wyodrębniam danych wyjściowych, ponieważ zmiany w tych danych wyjściowych nie są znaczące;
zmiany te prawdopodobnie wynikają z faktu, że wątki działają inaczej, a nie ze
zmian w kompilatorze -->

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

Małe szczegóły, takie jak miejsce wywołania `join`, mogą mieć wpływ na to, czy wątki będą działać w tym samym czasie.

### Using `move` Closures with Threads

Często używamy słowa kluczowego `move` z zamknięciami przekazywanymi do `thread::spawn`,
ponieważ zamknięcie przejmie wtedy własność wartości, których używa ze
środowiska, przenosząc w ten sposób własność tych wartości z jednego wątku do
drugiego. W sekcji [„Przechwytywanie odniesień lub przenoszenie własności”][capture]<!-- ignore
--> rozdziału 13 omówiliśmy `move` w kontekście zamknięć. Teraz
skupimy się bardziej na interakcji między `move` i `thread::spawn`.

Zauważ w Liście 16-1, że zamknięcie przekazywane do `thread::spawn` nie przyjmuje żadnych
argumentów: nie używamy żadnych danych z wątku głównego w kodzie utworzonego
wątku. Aby użyć danych z wątku głównego w utworzonym
wątku, zamknięcie utworzonego
wątku musi przechwycić potrzebne mu wartości. Listing 16-3 pokazuje
próbę utworzenia wektora w wątku głównym i użycia go w wątku utworzonym. Jednak to jeszcze nie zadziała, jak zaraz zobaczysz.

<Listing number="16-3" file-name="src/main.rs" caption="Attempting to use a vector created by the main thread in another thread">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-03/src/main.rs}}
```

</Listing>

Zamknięcie używa `v`, więc przechwyci `v` i uczyni je częścią
środowiska zamknięcia. Ponieważ `thread::spawn` uruchamia to zamknięcie w nowym wątku,
powinniśmy mieć dostęp do `v` w tym nowym wątku. Ale kiedy kompilujemy ten
przykład, otrzymujemy następujący błąd:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-03/output.txt}}
```

Rust *wnioskuje*, jak przechwycić `v`, a ponieważ `println!` potrzebuje tylko odwołania
do `v`, zamknięcie próbuje pożyczyć `v`. Jest jednak pewien problem: Rust nie może
powiedzieć, jak długo będzie działał utworzony wątek, więc nie wie, czy odwołanie
do `v` zawsze będzie prawidłowe.

W liście 16-4 przedstawiono scenariusz, w którym bardziej prawdopodobne jest odniesienie do `v`, które nie będzie prawidłowe:

<Listing number="16-4" file-name="src/main.rs" caption="A thread with a closure that attempts to capture a reference to `v` from a main thread that drops `v`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-04/src/main.rs}}
```

</Listing>

Gdyby Rust pozwolił nam uruchomić ten kod, istnieje możliwość, że utworzony wątek
zostałby natychmiast umieszczony w tle bez uruchamiania się w ogóle. Utworzony
wątek ma odwołanie do `v` wewnątrz, ale wątek główny natychmiast usuwa
`v`, używając funkcji `drop`, którą omówiliśmy w rozdziale 15. Następnie, gdy utworzony
wątek zaczyna się wykonywać, `v` nie jest już prawidłowy, więc odwołanie do niego
również jest nieprawidłowe. O nie!

Aby naprawić błąd kompilatora w Liście 16-3, możemy skorzystać z porady w komunikacie o błędzie:

<!-- manual-regeneration
po automatycznej regeneracji, spójrz na listings/ch16-fearless-concurrency/listing-16-03/output.txt i skopiuj odpowiednią część
-->

```text
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
```

Dodając słowo kluczowe `move` przed zamknięciem, zmuszamy zamknięcie do przejęcia
własności wartości, których używa, zamiast pozwolić Rustowi wnioskować, że
powinien pożyczyć wartości. Modyfikacja Listingu 16-3 pokazana w Listingu
16-5 skompiluje się i uruchomi zgodnie z naszymi zamierzeniami:

<Listing number="16-5" file-name="src/main.rs" caption="Using the `move` keyword to force a closure to take ownership of the values it uses">


```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-05/src/main.rs}}
```

</Listing>

Możemy być skłonni spróbować tego samego, aby naprawić kod w Listingu 16-4, gdzie
główny wątek wywołał `drop`, używając zamknięcia `move`. Jednak ta poprawka
nie zadziała, ponieważ to, co Listing 16-4 próbuje zrobić, jest niedozwolone z
innego powodu. Gdybyśmy dodali `move` do zamknięcia, przenieślibyśmy `v` do
środowiska zamknięcia i nie moglibyśmy już wywołać `drop` w wątku głównym. Zamiast tego otrzymalibyśmy ten błąd kompilatora:

```console
{{#include ../listings/ch16-fearless-concurrency/output-only-01-move-drop/output.txt}}
```

Reguły własności Rusta uratowały nas ponownie! Otrzymaliśmy błąd z kodu w
Listingu 16-3, ponieważ Rust był konserwatywny i pożyczał tylko `v` dla
wątku, co oznaczało, że wątek główny teoretycznie mógł unieważnić odniesienie do utworzonego
wątku. Mówiąc Rustowi, aby przeniósł własność `v` do utworzonego
wątku, gwarantujemy Rustowi, że wątek główny nie będzie już używał `v`. Jeśli
zmienimy Listing 16-4 w ten sam sposób, naruszymy reguły własności,
gdy spróbujemy użyć `v` w wątku głównym. Słowo kluczowe `move` zastępuje
konserwatywne domyślne pożyczanie Rusta; nie pozwala nam naruszyć reguł własności.

Mając podstawową wiedzę na temat wątków i interfejsu API wątków, przyjrzyjmy się, co
możemy *zrobić* z wątkami.

[capture]: ch13-01-closures.html#capturing-references-or-moving-ownership
