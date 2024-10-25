## Współdzielona współbieżność stanu

Przekazywanie wiadomości to dobry sposób obsługi współbieżności, ale nie jest to jedyny
metoda. Inną metodą byłoby umożliwienie wielu wątkom dostępu do tych samych współdzielonych
danych. Rozważmy ponownie tę część sloganu z dokumentacji języka Go:
„nie komunikuj się poprzez współdzielenie pamięci”.

Jak wyglądałaby komunikacja poprzez współdzielenie pamięci? Ponadto, dlaczego
entuzjaści przekazywania wiadomości przestrzegają przed korzystaniem ze współdzielenia pamięci?

W pewnym sensie kanały w dowolnym języku programowania są podobne do pojedynczego posiadania,
ponieważ po przesłaniu wartości kanałem nie powinieneś już używać tej wartości. Współbieżność współdzielonej pamięci jest jak wielokrotne posiadanie: wiele wątków
może uzyskać dostęp do tej samej lokalizacji pamięci w tym samym czasie. Jak widziałeś w rozdziale 15,
gdzie inteligentne wskaźniki umożliwiły wielokrotne posiadanie, wielokrotne posiadanie może
dodawać złożoności, ponieważ tymi różnymi właścicielami trzeba zarządzać. System typów i reguły posiadania Rusta w dużym stopniu pomagają w prawidłowym zarządzaniu. Na przykład przyjrzyjmy się mutexom, jednemu z bardziej powszechnych prymitywów współbieżności
dla pamięci współdzielonej.

### Using Mutexes to Allow Access to Data from One Thread at a Time

*Mutex* to skrót od *wzajemnego wykluczenia*, jak w przypadku mutexu, który pozwala
tylko jednemu wątkowi na dostęp do danych w dowolnym momencie. Aby uzyskać dostęp do danych w
mutexie, wątek musi najpierw zasygnalizować, że chce uzyskać dostęp, prosząc o nabycie *blokady*
mutexu. Blokada to struktura danych, która jest częścią mutexu, która
śledzi, kto aktualnie ma wyłączny dostęp do danych. Dlatego
mutex jest opisywany jako *strzeżący* przechowywanych w nim danych za pośrednictwem systemu blokowania.

Mutexy mają opinię trudnych w użyciu, ponieważ
trzeba pamiętać o dwóch zasadach:

* Przed użyciem danych należy podjąć próbę nabywcy blokady.

Po zakończeniu pracy z danymi, których strzeże mutex, należy odblokować
dane, aby inne wątki mogły nabywca blokady.

Aby uzyskać rzeczywistą metaforę mutexu, wyobraź sobie dyskusję panelową na
konferencji z tylko jednym mikrofonem. Zanim panelista będzie mógł przemówić, musi
poprosić lub zasygnalizować, że chce użyć mikrofonu. Gdy dostaną
mikrofon, mogą mówić tak długo, jak chcą, a następnie przekazać
mikrofon następnemu paneliście, który poprosi o wypowiedź. Jeśli panelista zapomni
oddać mikrofonu po skończeniu z nim, nikt inny nie będzie mógł
mówić. Jeśli zarządzanie współdzielonym mikrofonem pójdzie źle, panel nie będzie działał
zgodnie z planem!

Zarządzanie mutexami może być niezwykle trudne do prawidłowego wykonania, dlatego tak wiele osób jest entuzjastycznie nastawionych do kanałów. Jednak dzięki systemowi typów i zasadom własności Rust nie można źle blokować i odblokowywać.
#### The API of `Mutex<T>`

Jako przykład użycia mutexa, zacznijmy od użycia mutexa w kontekście jednowątkowym, jak pokazano na Liście 16-12:

<Listing number="16-12" file-name="src/main.rs" caption="Exploring the API of `Mutex<T>` in a single-threaded context for simplicity">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-12/src/main.rs}}
```

</Listing>

Podobnie jak w przypadku wielu typów, tworzymy `Mutex<T>` przy użyciu powiązanej funkcji `new`.
Aby uzyskać dostęp do danych wewnątrz mutexu, używamy metody `lock`, aby uzyskać blokadę. To wywołanie zablokuje bieżący wątek, więc nie będzie mógł wykonać żadnej pracy, dopóki nie nadejdzie nasza kolej na blokadę.

Wywołanie `lock` nie powiedzie się, jeśli inny wątek trzymający blokadę wpadnie w panikę. W takim przypadku nikt nigdy nie będzie w stanie uzyskać blokady, więc zdecydowaliśmy się na
`odwinięcie` i spowodowanie paniki tego wątku, jeśli znajdziemy się w takiej sytuacji.

Po uzyskaniu blokady możemy traktować wartość zwracaną, nazwaną w tym przypadku `num`, jako zmienne odwołanie do danych wewnątrz. System typów zapewnia, że
uzyskamy blokadę przed użyciem wartości w `m`. Typem `m` jest
`Mutex<i32>`, a nie `i32`, więc *musimy* wywołać `lock`, aby móc użyć wartości `i32`. Nie możemy zapomnieć; system typów nie pozwoli nam uzyskać dostępu do wewnętrznego `i32`
w przeciwnym razie.

Jak możesz podejrzewać, `Mutex<T>` jest inteligentnym wskaźnikiem. Dokładniej rzecz biorąc, wywołanie
`lock` *zwraca* inteligentny wskaźnik o nazwie `MutexGuard`, opakowany w
`LockResult`, który obsłużono za pomocą wywołania `unwrap`. Inteligentny
wskaźnik `MutexGuard` implementuje `Deref`, aby wskazywał na nasze wewnętrzne dane; inteligentny
wskaźnik ma również implementację `Drop`, która automatycznie zwalnia blokadę, gdy
`MutexGuard` wychodzi poza zakres, co ma miejsce na końcu wewnętrznego zakresu. W
rezultacie nie ryzykujemy zapomnienia o zwolnieniu blokady i zablokowania mutexu przed użyciem przez inne wątki, ponieważ zwolnienie blokady następuje
automatycznie.

Po usunięciu blokady możemy wydrukować wartość mutexu i zobaczyć, że udało nam się zmienić wewnętrzny `i32` na 6.

#### Sharing a `Mutex<T>` Between Multiple Threads

Teraz spróbujmy udostępnić wartość między wieloma wątkami za pomocą `Mutex<T>`.
Uruchomimy 10 wątków i sprawimy, że każdy z nich zwiększy wartość licznika o 1, tak aby
licznik przeszedł z 0 do 10. Następny przykład w listingu 16-13 będzie zawierał
błąd kompilatora, a my wykorzystamy ten błąd, aby dowiedzieć się więcej o używaniu
`Mutex<T>` i jak Rust pomaga nam używać go poprawnie.

<Listing number="16-13" file-name="src/main.rs" caption="Ten threads each increment a counter guarded by a `Mutex<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-13/src/main.rs}}
```

</Listing>

Tworzymy zmienną `counter`, aby trzymać `i32` wewnątrz `Mutex<T>`, tak jak zrobiliśmy
w Liście 16-12. Następnie tworzymy 10 wątków, iterując po zakresie
liczb. Używamy `thread::spawn` i dajemy wszystkim wątkom to samo zamknięcie: takie,
które przenosi licznik do wątku, uzyskuje blokadę na `Mutex<T>` poprzez
wywołanie metody `lock`, a następnie dodaje 1 do wartości w mutexie. Kiedy
wątek zakończy wykonywanie swojego zamknięcia, `num` wyjdzie poza zakres i zwolni blokadę,
aby inny wątek mógł ją uzyskać.

W wątku głównym zbieramy wszystkie uchwyty połączeń. Następnie, tak jak zrobiliśmy w Liście
16-2, wywołujemy `join` na każdym uchwycie, aby upewnić się, że wszystkie wątki zakończą. W tym momencie wątek główny uzyska blokadę i wydrukuje wynik tego
programu.

Zasugerowaliśmy, że ten przykład się nie skompiluje. Teraz dowiedzmy się dlaczego!

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-13/output.txt}}
```

Komunikat o błędzie informuje, że wartość `counter` została przeniesiona w poprzedniej
iteracji pętli. Rust informuje nas, że nie możemy przenieść własności `counter` do wielu wątków. Naprawmy błąd kompilatora za pomocą
metody multiple-ownership, którą omówiliśmy w rozdziale 15.

#### Multiple Ownership with Multiple Threads

W rozdziale 15 nadaliśmy wartości wielu właścicieli, używając inteligentnego wskaźnika
`Rc<T>`, aby utworzyć wartość zliczaną referencyjnie. Zróbmy to samo tutaj i zobaczmy,
co się stanie. Owiniemy `Mutex<T>` w `Rc<T>` w Listingu 16-14 i sklonujemy
`Rc<T>` przed przeniesieniem własności do wątku.

<Listing number="16-14" file-name="src/main.rs" caption="Attempting to use `Rc<T>` to allow multiple threads to own the `Mutex<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-14/src/main.rs}}
```

</Listing>

Once again, we compile and get... different errors! The compiler is teaching us
a lot.

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-14/output.txt}}
```

Wow, ten komunikat o błędzie jest bardzo rozwlekły! Oto najważniejsza część, na której należy się skupić:
`` `Rc<Mutex<i32>>` nie można bezpiecznie wysłać między wątkami``. Kompilator
mówi nam również, dlaczego: ``cecha `Send` nie jest zaimplementowana dla
`Rc<Mutex<i32>>```. O `Send` porozmawiamy w następnej sekcji: to jedna z cech,
która zapewnia, że ​​typy, których używamy z wątkami, są przeznaczone do
używania w sytuacjach współbieżnych.

Niestety, `Rc<T>` nie jest bezpieczny do udostępniania między wątkami. Kiedy `Rc<T>`
zarządza liczbą odwołań, dodaje ją do liczby dla każdego wywołania `clone` i
odejmuje od liczby, gdy każdy klon jest usuwany. Ale nie używa żadnych
prymitywów współbieżności, aby upewnić się, że zmiany liczby nie mogą zostać
przerwane przez inny wątek. Może to prowadzić do błędnych obliczeń — subtelnych błędów, które
mogą z kolei prowadzić do wycieków pamięci lub utraty wartości, zanim skończymy
z nią. Potrzebujemy typu dokładnie takiego jak `Rc<T>`, ale takiego, który wprowadza zmiany
w liczbie odniesień w sposób bezpieczny dla wątków.

#### Atomic Reference Counting with `Arc<T>`

Na szczęście `Arc<T>` *jest* typem podobnym do `Rc<T>`, którego można bezpiecznie używać w
sytuacjach współbieżnych. *a* oznacza *atomic*, co oznacza, że ​​jest to typ *atomically
reference counted*. Atomics to dodatkowy rodzaj prymitywu współbieżności, którego nie będziemy tutaj szczegółowo omawiać: zapoznaj się ze standardową dokumentacją biblioteki dla [`std::sync::atomic`][atomic]<!-- ignore -->, aby uzyskać więcej
szczegółów. W tym momencie musisz tylko wiedzieć, że atomics działają jak typy prymitywne, ale można je bezpiecznie udostępniać między wątkami.

Możesz się zastanawiać, dlaczego wszystkie typy prymitywne nie są atomowe i dlaczego standardowe
typy biblioteczne nie są zaimplementowane do domyślnego używania `Arc<T>`. Powodem jest to, że
bezpieczeństwo wątków wiąże się z karą za wydajność, którą chcesz ponieść tylko wtedy,
kiedy naprawdę tego potrzebujesz. Jeśli wykonujesz operacje na wartościach w ramach
pojedynczego wątku, Twój kod może działać szybciej, jeśli nie musi wymuszać
gwarancji, jakie zapewniają atomy.

Wróćmy do naszego przykładu: `Arc<T>` i `Rc<T>` mają to samo API, więc naprawiamy
nasz program, zmieniając linię `use`, wywołanie `new` i wywołanie
`clone`. Kod w Listingu 16-15 zostanie w końcu skompilowany i uruchomiony:

<Listing number="16-15" file-name="src/main.rs" caption="Using an `Arc<T>` to wrap the `Mutex<T>` to be able to share ownership across multiple threads">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-15/src/main.rs}}
```

</Listing>

Ten kod wydrukuje następujące informacje:

<!-- Nie wyodrębniam danych wyjściowych, ponieważ zmiany w tych danych wyjściowych nie są znaczące;
zmiany prawdopodobnie wynikają z faktu, że wątki działają inaczej, a nie ze
zmian w kompilatorze -->

```tekst
Wynik: 10
```

Zrobiliśmy to! Policzyliśmy od 0 do 10, co może nie wydawać się zbyt imponujące, ale
nauczyło nas to wiele o `Mutex<T>` i bezpieczeństwie wątków. Możesz również użyć
struktury tego programu do wykonywania bardziej skomplikowanych operacji niż tylko zwiększanie
licznika. Używając tej strategii, możesz podzielić obliczenie na niezależne
części, rozdzielić te części na wątki, a następnie użyć `Mutex<T>`, aby każdy
wątek zaktualizował końcowy wynik swoją częścią.

Należy pamiętać, że jeśli wykonujesz proste operacje numeryczne, istnieją typy prostsze
od `Mutex<T>` udostępniane przez moduł [`std::sync::atomic` biblioteki
standardowej][atomic]<!-- ignore -->. Typy te zapewniają bezpieczny, współbieżny,
atomowy dostęp do typów pierwotnych. W tym przykładzie zdecydowaliśmy się użyć `Mutex<T>` z typem pierwotnym, abyśmy mogli skupić się na tym, jak działa `Mutex<T>`.

### Similarities Between `RefCell<T>`/`Rc<T>` and `Mutex<T>`/`Arc<T>`

Być może zauważyłeś, że `counter` jest niezmienny, ale możemy uzyskać zmienną
referencję do wartości wewnątrz niego; oznacza to, że `Mutex<T>` zapewnia wewnętrzną
mutowalność, podobnie jak rodzina `Cell`. W ten sam sposób, w jaki użyliśmy `RefCell<T>` w
rozdziale 15, aby umożliwić nam mutację zawartości wewnątrz `Rc<T>`, używamy `Mutex<T>`,
aby mutację zawartości wewnątrz `Arc<T>`.

Innym szczegółem, na który należy zwrócić uwagę, jest to, że Rust nie może chronić Cię przed wszelkiego rodzaju błędami logicznymi,
gdy używasz `Mutex<T>`. Przypomnij sobie w rozdziale 15, że używanie `Rc<T>` wiązało się
z ryzykiem tworzenia cykli referencyjnych, w których dwie wartości `Rc<T>` odnoszą się
do siebie, powodując wycieki pamięci. Podobnie, `Mutex<T>` wiązało się z ryzykiem
tworzenia *impasów*. Występują one, gdy operacja musi zablokować dwa zasoby,
a dwa wątki uzyskały po jednym z zamków, co powoduje, że czekają
na siebie w nieskończoność. Jeśli interesują Cię blokady, spróbuj utworzyć program Rust,
który ma blokadę; następnie zbadaj strategie łagodzenia blokad dla
muteksów w dowolnym języku i spróbuj je zaimplementować w Rust.
Dokumentacja standardowego interfejsu API biblioteki dla `Mutex<T>` i `MutexGuard` oferuje
przydatne informacje.

Zakończymy ten rozdział, omawiając cechy `Send` i `Sync` oraz
sposób, w jaki możemy ich używać z typami niestandardowymi.

[atomic]: ../std/sync/atomic/index.html
