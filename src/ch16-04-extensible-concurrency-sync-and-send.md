## Rozszerzalna współbieżność z cechami `Sync` i `Send`

Co ciekawe, język Rust ma *bardzo* mało funkcji współbieżności. Prawie
każda funkcja współbieżności, o której mówiliśmy do tej pory w tym rozdziale, była
częścią biblioteki standardowej, a nie języka. Twoje opcje obsługi
współbieżności nie ograniczają się do języka ani biblioteki standardowej; możesz
napisać własne funkcje współbieżności lub użyć tych napisanych przez innych.

Jednak w języku osadzone są dwie koncepcje współbieżności: cechy
`std::marker` `Sync` i `Send`.

### Allowing Transference of Ownership Between Threads with `Send`

Cecha znacznika `Send` wskazuje, że własność wartości typu
implementującego `Send` może być przenoszona między wątkami. Prawie każdy typ Rust
to `Send`, ale istnieją pewne wyjątki, w tym `Rc<T>`: nie może to być
`Send`, ponieważ jeśli sklonujesz wartość `Rc<T>` i spróbujesz przenieść własność
klonu do innego wątku, oba wątki mogą zaktualizować liczbę odwołań
w tym samym czasie. Z tego powodu `Rc<T>` jest implementowany do użytku w
sytuacjach jednowątkowych, w których nie chcesz płacić kary za wydajność
bezpieczną dla wątków.

Dlatego system typów i granice cech Rust zapewniają, że nigdy nie możesz
przypadkowo wysłać wartości `Rc<T>` między wątkami w sposób niebezpieczny. Kiedy próbowaliśmy to zrobić
w Liście 16-14, otrzymaliśmy błąd `cecha Send nie jest zaimplementowana dla
Rc<Mutex<i32>>`. Gdy przeszliśmy na `Arc<T>`, czyli `Send`, kod
skompilował się.

Każdy typ złożony w całości z typów `Send` jest automatycznie oznaczany jako `Send`. Niemal wszystkie typy prymitywne to `Send`, poza wskaźnikami, które
omówimy w rozdziale 20.

### Allowing Access from Multiple Threads with `Sync`

Cecha znacznika `Sync` wskazuje, że typ implementujący
`Sync` może być bezpiecznie odwoływany z wielu wątków. Innymi słowy, każdy typ `T` jest
`Sync`, jeśli `&T` (niezmienne odniesienie do `T`) jest `Send`, co oznacza, że ​​odniesienie
może być bezpiecznie wysłane do innego wątku. Podobnie jak `Send`, typy prymitywne to
`Sync`, a typy złożone w całości z typów, które są `Sync`, są również `Sync`.

Inteligentny wskaźnik `Rc<T>` również nie jest `Sync` z tych samych powodów, dla których nie jest
`Send`. Typ `RefCell<T>` (o którym mówiliśmy w rozdziale 15) i
rodzina powiązanych typów `Cell<T>` nie są `Sync`. Implementacja sprawdzania pożyczania, którą `RefCell<T>` wykonuje w czasie wykonywania, nie jest bezpieczna dla wątków. Inteligentny
wskaźnik `Mutex<T>` to `Sync` i może być używany do współdzielenia dostępu z wieloma
wątkami, jak pokazano w sekcji [„Udostępnianie `Mutex<T>` między wieloma
wątkami”][udostępnianie-mutextu-między-wieloma-wątkami]<!-- ignoruj ​​-->.

### Ręczna implementacja `Send` i `Sync` jest niebezpieczna

Ponieważ typy składające się z cech `Send` i `Sync` są automatycznie
również `Send` i `Sync`, nie musimy implementować tych cech ręcznie. Jako
cechy znacznikowe nie mają nawet żadnych metod do implementacji. Są po prostu
przydatne do wymuszania niezmienników związanych ze współbieżnością.

Ręczna implementacja tych cech wiąże się z implementacją niebezpiecznego kodu Rust.
Omówimy używanie niebezpiecznego kodu Rust w rozdziale 20; na razie, ważną
informacją jest to, że budowanie nowych współbieżnych typów, które nie składają się z części `Send` i
`Sync` wymaga starannego przemyślenia, aby utrzymać gwarancje bezpieczeństwa. [„The
Rustonomicon”][nomicon] zawiera więcej informacji o tych gwarancjach i sposobie ich
utrzymania.

## Summary

To nie jest ostatni raz, kiedy zobaczysz współbieżność w tej książce: cały następny
rozdział skupia się na programowaniu asynchronicznym, a projekt w rozdziale 21 wykorzysta
koncepcje z tego rozdziału w bardziej realistycznej sytuacji niż mniejsze przykłady
omawiane tutaj.

Jak wspomniano wcześniej, ponieważ bardzo niewiele z tego, jak Rust obsługuje współbieżność, jest
częścią języka, wiele rozwiązań współbieżności jest implementowanych jako skrzynie.
Ewoluują one szybciej niż biblioteka standardowa, więc koniecznie poszukaj
w Internecie aktualnych, najnowocześniejszych skrzyń do użycia w
sytuacjach wielowątkowych.

Biblioteka standardowa Rust udostępnia kanały do ​​przekazywania wiadomości i inteligentne
typy wskaźników, takie jak `Mutex<T>` i `Arc<T>`, które są bezpieczne do
używania w kontekstach współbieżnych. System typów i sprawdzanie pożyczania zapewniają, że
kod wykorzystujący te rozwiązania nie skończy się wyścigami danych ani nieprawidłowymi odwołaniami.
Gdy już skompilujesz swój kod, możesz być pewien, że będzie on działał
szczęśliwie w wielu wątkach bez trudnych do wyśledzenia błędów, które są powszechne w
innych językach. Programowanie współbieżne nie jest już koncepcją, której należy się bać:
idź i spraw, aby Twoje programy były współbieżne, bez strachu!

[sharing-a-mutext-between-multiple-threads]:
ch16-03-shared-state.html#sharing-a-mutext-between-multiple-threads
[nomicon]: ../nomicon/index.html
