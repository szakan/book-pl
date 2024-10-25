## `RefCell<T>` i wzorzec wewnętrznej zmienności

*Zmienność wewnętrzna* to wzorzec projektowy w Rust, który pozwala na mutację
danych, nawet gdy istnieją niezmienne odwołania do tych danych; zwykle ta akcja jest niedozwolona przez reguły pożyczania. Aby zmutować dane, wzorzec używa
kodu `niebezpiecznego` wewnątrz struktury danych, aby nagiąć zwykłe reguły Rust, które rządzą
mutacją i pożyczaniem. Niebezpieczny kod wskazuje kompilatorowi, że sprawdzamy reguły ręcznie, zamiast polegać na kompilatorze, który sprawdzi je
za nas; niebezpieczny kod omówimy szerzej w rozdziale 20.

Możemy używać typów, które używają wzorca zmienności wewnętrznej tylko wtedy, gdy możemy
zapewnić, że reguły pożyczania będą przestrzegane w czasie wykonywania, nawet jeśli
kompilator nie może tego zagwarantować. Następnie zaangażowany `niebezpieczny` kod jest opakowany w
bezpieczny interfejs API, a typ zewnętrzny jest nadal niezmienny.

Przyjrzyjmy się tej koncepcji, przyglądając się typowi `RefCell<T>`, który podąża za
wzorcem zmienności wewnętrznej.

### Enforcing Borrowing Rules at Runtime with `RefCell<T>`

W przeciwieństwie do `Rc<T>`, typ `RefCell<T>` reprezentuje pojedyncze prawo własności do danych, które
posiada. Czym więc różni się `RefCell<T>` od typu takiego jak `Box<T>`?
Przypomnij sobie zasady pożyczania, których nauczyłeś się w rozdziale 4:

* W dowolnym momencie możesz mieć *albo* (ale nie obie) jedną zmienną referencję
lub dowolną liczbę niezmiennych referencji.
* Referencje muszą być zawsze prawidłowe.

W przypadku referencji i `Box<T>` niezmienniki reguł pożyczania są wymuszane w
czasie kompilacji. W przypadku `RefCell<T>` niezmienniki te są wymuszane *w czasie wykonywania*.
W przypadku referencji, jeśli złamiesz te reguły, otrzymasz błąd kompilatora. W przypadku
`RefCell<T>`, jeśli złamiesz te reguły, Twój program wpadnie w panikę i zakończy działanie.

Zalety sprawdzania reguł pożyczania w czasie kompilacji polegają na tym, że błędy
zostaną wykryte wcześniej w procesie rozwoju i nie będą miały wpływu na
wydajność w czasie wykonywania, ponieważ cała analiza jest wykonywana wcześniej. Z tych
powodów sprawdzanie reguł pożyczania w czasie kompilacji jest najlepszym wyborem w
większości przypadków, dlatego jest to domyślne ustawienie Rusta.

Zaletą sprawdzania reguł pożyczania w czasie wykonywania jest to, że
pewne scenariusze bezpieczne dla pamięci są wtedy dozwolone, podczas gdy zostałyby
odrzucone przez sprawdzanie w czasie kompilacji. Analiza statyczna, podobnie jak kompilator Rust,
jest z natury konserwatywna. Niektórych właściwości kodu nie można wykryć poprzez
analizę kodu: najsłynniejszym przykładem jest problem zatrzymania, który
wykracza poza zakres tej książki, ale jest ciekawym tematem do zbadania.

Ponieważ niektóre analizy są niemożliwe, jeśli kompilator Rust nie może być pewien, że
kod jest zgodny z regułami własności, może odrzucić poprawny program; w ten sposób jest konserwatywny. Gdyby Rust zaakceptował niepoprawny program, użytkownicy
nie mogliby zaufać gwarancjom Rust. Jednak jeśli Rust
odrzuci poprawny program, programista będzie miał niedogodności, ale nic
katastrofalnego nie może się wydarzyć. Typ `RefCell<T>` jest przydatny, gdy masz pewność, że
kod przestrzega reguł pożyczania, ale kompilator nie jest w stanie tego zrozumieć i
zagwarantować.

Podobnie jak `Rc<T>`, `RefCell<T>` jest przeznaczony wyłącznie do użytku w scenariuszach jednowątkowych
i spowoduje błąd kompilacji, jeśli spróbujesz go użyć w kontekście
wielowątkowym. Omówimy, jak uzyskać funkcjonalność `RefCell<T>` w programie wielowątkowym w rozdziale 16.

Oto podsumowanie powodów, dla których warto wybrać `Box<T>`, `Rc<T>` lub `RefCell<T>`:

* `Rc<T>` umożliwia wielu właścicieli tych samych danych; `Box<T>` i `RefCell<T>`
mają pojedynczych właścicieli.
* `Box<T>` zezwala na niezmienne lub zmienne pożyczki sprawdzane w czasie kompilacji; `Rc<T>`
zezwala tylko na niezmienne pożyczki sprawdzane w czasie kompilacji; `RefCell<T>` zezwala
na niezmienne lub zmienne pożyczki sprawdzane w czasie wykonywania.
* Ponieważ `RefCell<T>` zezwala na zmienne pożyczki sprawdzane w czasie wykonywania, możesz
mutować wartość wewnątrz `RefCell<T>` nawet wtedy, gdy `RefCell<T>` jest
niezmienny.

Mutowanie wartości wewnątrz wartości niezmiennej to wzorzec *wewnętrznej zmienności*. Przyjrzyjmy się sytuacji, w której wewnętrzna zmienność jest przydatna i
przeanalizujmy, jak to jest możliwe.

### Interior Mutability: A Mutable Borrow to an Immutable Value

Konsekwencją reguł pożyczania jest to, że gdy masz niezmienną wartość, nie możesz jej pożyczyć w sposób zmienny. Na przykład ten kod nie skompiluje się:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/src/main.rs}}
```

If you tried to compile this code, you’d get the following error:

```console
{{#include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/output.txt}}
```

Istnieją jednak sytuacje, w których przydatne byłoby, aby wartość mutowała
się w swoich metodach, ale wydawała się niezmienna dla innego kodu. Kod poza metodami wartości nie byłby w stanie mutować wartości. Użycie `RefCell<T>` jest
jednym ze sposobów na uzyskanie możliwości posiadania wewnętrznej zmienności, ale `RefCell<T>`
nie omija całkowicie reguł pożyczania: sprawdzanie pożyczania w
kompilatorze zezwala na tę wewnętrzną zmienność, a reguły pożyczania są sprawdzane
w czasie wykonywania. Jeśli naruszysz reguły, otrzymasz `panik!` zamiast
błędu kompilatora.

Przeanalizujmy praktyczny przykład, w którym możemy użyć `RefCell<T>` do mutowania
niezmiennej wartości i zobaczmy, dlaczego jest to przydatne.

#### A Use Case for Interior Mutability: Mock Objects

Czasami podczas testowania programista używa typu zamiast innego typu,
aby zaobserwować określone zachowanie i upewnić się, że zostało ono poprawnie zaimplementowane.
Ten typ zastępczy nazywa się *test double*. Pomyśl o nim w sensie
„dublera kaskaderskiego” w filmach, gdzie osoba wkracza i zastępuje
aktora, aby wykonać określoną trudną scenę. Test double zastępuje inne typy,
gdy uruchamiamy testy. *Obiekty pozorowane* to określone typy test double,
które rejestrują to, co dzieje się podczas testu, dzięki czemu możesz upewnić się, że wykonano prawidłowe
działania.

Rust nie ma obiektów w takim samym sensie, w jakim inne języki mają obiekty,
a Rust nie ma wbudowanej w standardową bibliotekę funkcjonalności obiektów pozorowanych,
jak niektóre inne języki. Możesz jednak z pewnością utworzyć strukturę,
która będzie służyć tym samym celom, co obiekt pozorowany.

Oto scenariusz, który przetestujemy: utworzymy bibliotekę, która śledzi wartość
w stosunku do wartości maksymalnej i wysyła wiadomości na podstawie tego, jak blisko wartości maksymalnej jest bieżąca wartość. Ta biblioteka może być używana na przykład do śledzenia limitu
użytkownika dotyczącego liczby wywołań API, które może wykonać.

Nasza biblioteka zapewni tylko funkcjonalność śledzenia, jak blisko wartości maksymalnej jest wartość i jakie wiadomości powinny być w jakich godzinach. Aplikacje,
które korzystają z naszej biblioteki, będą musiały zapewnić mechanizm wysyłania
wiadomości: aplikacja może umieścić wiadomość w aplikacji, wysłać
e-mail, wysłać wiadomość tekstową lub coś innego. Biblioteka nie musi znać
tych szczegółów. Potrzebuje tylko czegoś, co implementuje cechę, którą zapewnimy,
zwaną `Messenger`. Wypis 15-20 pokazuje kod biblioteki:

<Listing number="15-20" file-name="src/lib.rs" caption="A library to keep track of how close a value is to a maximum value and warn when the value is at certain levels">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-20/src/lib.rs}}
```

</Listing>

Ważną częścią tego kodu jest to, że cecha `Messenger` ma jedną metodę
o nazwie `send`, która przyjmuje niezmienne odwołanie do `self` i tekst
wiadomości. Ta cecha jest interfejsem, który nasz obiekt pozorowany musi zaimplementować, aby
makieta mogła być używana w taki sam sposób, jak prawdziwy obiekt. Inną ważną częścią
jest to, że chcemy przetestować zachowanie metody `set_value` w
`LimitTracker`. Możemy zmienić to, co przekazujemy dla parametru `value`, ale
`set_value` nie zwraca niczego, na czym moglibyśmy wykonać asercje. Chcemy móc powiedzieć, że jeśli utworzymy `LimitTracker` z czymś, co implementuje
cechę `Messenger` i określoną wartość dla `max`, gdy przekażemy różne
liczby dla `value`, komunikatorowi zostanie powiedziane, aby wysłał odpowiednie wiadomości.

Potrzebujemy obiektu pozorowanego, który zamiast wysyłać wiadomość e-mail lub tekstową, gdy
wywołujemy `send`, będzie śledził tylko wiadomości, które ma wysłać. Możemy
utworzyć nową instancję obiektu pozorowanego, utworzyć `LimitTracker`, który używa obiektu pozorowanego, wywołać metodę `set_value` na `LimitTracker`, a następnie sprawdzić, czy
obiekt pozorowany ma wiadomości, których się spodziewamy. Listing 15-21 pokazuje próbę
zaimplementowania obiektu pozorowanego, aby to zrobić, ale sprawdzanie pożyczania na to nie pozwala:

<Listing number="15-21" file-name="src/lib.rs" caption="An attempt to implement a `MockMessenger` that isn’t allowed by the borrow checker">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-21/src/lib.rs:here}}
```

</Listing>

Ten kod testowy definiuje strukturę `MockMessenger`, która ma pole `sent_messages`
z `Vec` wartości `String`, aby śledzić wiadomości, które ma wysłać. Definiujemy również powiązaną funkcję `new`, aby ułatwić
tworzenie nowych wartości `MockMessenger`, które zaczynają się od pustej listy wiadomości. Następnie implementujemy cechę `Messenger` dla `MockMessenger`, abyśmy mogli przekazać
`MockMessenger` do `LimitTracker`. W definicji metody `send`,
przyjmujemy przekazaną wiadomość jako parametr i przechowujemy ją na liście `MockMessenger`
sent_messages`.

W teście testujemy, co się stanie, gdy `LimitTracker` otrzyma polecenie ustawienia
`value` na coś, co jest większe niż 75 procent wartości `max`. Najpierw
tworzymy nowego `MockMessenger`, który rozpocznie się od pustej listy wiadomości.
Następnie tworzymy nowego `LimitTracker` i nadajemy mu odwołanie do nowego
`MockMessenger` i wartość `max` równą 100. Wywołujemy metodę `set_value` na
`LimitTracker` z wartością 80, która stanowi więcej niż 75 procent 100. Następnie
twierdzamy, że lista wiadomości, którą śledzi `MockMessenger`,
powinna teraz zawierać jedną wiadomość.

Jednak jest jeden problem z tym testem, jak pokazano tutaj:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-21/output.txt}}
```

Nie możemy zmodyfikować `MockMessenger`, aby śledzić wiadomości, ponieważ metoda
`send` przyjmuje niezmienne odwołanie do `self`. Nie możemy również skorzystać z
sugestii z tekstu błędu, aby zamiast tego użyć `&mut self`, ponieważ wtedy
podpis `send` nie pasowałby do podpisu w definicji cechy `Messenger`
(możesz spróbować i zobaczyć, jaki komunikat o błędzie otrzymasz).

To sytuacja, w której wewnętrzna zmienność może pomóc! Przechowamy
`sent_messages` w `RefCell<T>`, a następnie metoda `send` będzie mogła
zmodyfikować `sent_messages`, aby przechowywać wiadomości, które widzieliśmy. Wylistowanie 15-22
pokazuje, jak to wygląda:

<Listing number="15-22" file-name="src/lib.rs" caption="Using `RefCell<T>` to mutate an inner value while the outer value is considered immutable">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-22/src/lib.rs:here}}
```

</Listing>

Pole `sent_messages` jest teraz typu `RefCell<Vec<String>>` zamiast
`Vec<String>`. W funkcji `new` tworzymy nową instancję `RefCell<Vec<String>>`
wokół pustego wektora.

W celu zaimplementowania metody `send` pierwszym parametrem jest nadal
niezmienne pożyczenie `self`, które pasuje do definicji cechy. Wywołujemy
`borrow_mut` na `RefCell<Vec<String>>` w `self.sent_messages`, aby uzyskać
zmienne odwołanie do wartości wewnątrz `RefCell<Vec<String>>`, która jest
wektorem. Następnie możemy wywołać `push` na zmiennym odwołaniu do wektora, aby
śledzić wiadomości wysyłane podczas testu.

Ostatnia zmiana, którą musimy wprowadzić, dotyczy asercji: aby zobaczyć, ile elementów znajduje się
w wewnętrznym wektorze, wywołujemy `borrow` na `RefCell<Vec<String>>`, aby uzyskać
niezmienną referencję do wektora.

Teraz, gdy wiesz, jak używać `RefCell<T>`, przyjrzyjmy się bliżej, jak to działa!

#### Keeping Track of Borrows at Runtime with `RefCell<T>`

Podczas tworzenia niezmiennych i zmiennych referencji używamy odpowiednio składni `&` i `&mut`. W przypadku `RefCell<T>` używamy metod `borrow` i `borrow_mut`, które są częścią bezpiecznego API należącego do `RefCell<T>`. Metoda
`borrow` zwraca inteligentny typ wskaźnika `Ref<T>`, a `borrow_mut`
zwraca inteligentny typ wskaźnika `RefMut<T>`. Oba typy implementują `Deref`, więc
możemy traktować je jak zwykłe referencje.

`RefCell<T>` śledzi, ile inteligentnych
wskaźników `Ref<T>` i `RefMut<T>` jest aktualnie aktywnych. Za każdym razem, gdy wywołujemy `borrow`, `RefCell<T>`
zwiększa liczbę aktywnych niezmiennych pożyczeń. Gdy wartość `Ref<T>`
wychodzi poza zakres, liczba niezmiennych pożyczeń spada o jeden. Podobnie jak reguły pożyczania w czasie kompilacji, `RefCell<T>` pozwala nam mieć wiele niezmiennych pożyczeń lub jedno zmienne pożyczenie w dowolnym momencie.

Jeśli spróbujemy naruszyć te reguły, zamiast otrzymać błąd kompilatora, jak miałoby to miejsce w przypadku odwołań, implementacja `RefCell<T>` wpadnie w panikę w
czasie wykonania. Listing 15-23 pokazuje modyfikację implementacji `send` w
listingu 15-22. Celowo próbujemy utworzyć dwa aktywne, zmienne pożyczenia
dla tego samego zakresu, aby zilustrować, że `RefCell<T>` uniemożliwia nam zrobienie tego
w czasie wykonania.

<Listing number="15-23" file-name="src/lib.rs" caption="Creating two mutable references in the same scope to see that `RefCell<T>` will panic">

```rust,ignore,panics
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-23/src/lib.rs:here}}
```

</Listing>

Tworzymy zmienną `one_borrow` dla inteligentnego wskaźnika `RefMut<T>` zwróconego
z `borrow_mut`. Następnie tworzymy inną zmienną loan w ten sam sposób w
zmiennej `two_borrow`. Tworzy to dwie zmienne referencje w tym samym zakresie,
co nie jest dozwolone. Gdy uruchomimy testy dla naszej biblioteki, kod w Listingu
15-23 skompiluje się bez żadnych błędów, ale test się nie powiedzie:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-23/output.txt}}
```

Zauważ, że kod spanikował z komunikatem `already loaned:
BorrowMutError`. W ten sposób `RefCell<T>` radzi sobie z naruszeniami reguł pożyczania
w czasie wykonywania.

Wybór wychwytywania błędów pożyczania w czasie wykonywania, a nie w czasie kompilacji, jak zrobiliśmy tutaj, oznacza, że ​​potencjalnie znajdziesz błędy w swoim kodzie później
w procesie rozwoju: prawdopodobnie dopiero po wdrożeniu kodu do
środowiska produkcyjnego. Ponadto Twój kod poniósłby niewielką karę wydajności w czasie wykonywania z powodu śledzenia pożyczeń w czasie wykonywania, a nie w czasie kompilacji.
Jednak użycie `RefCell<T>` umożliwia napisanie obiektu pozorowanego, który może
modyfikować się, aby śledzić komunikaty, które widział, gdy go używasz
w kontekście, w którym dozwolone są tylko niezmienne wartości. Możesz użyć `RefCell<T>`
pomimo jego kompromisów, aby uzyskać więcej funkcjonalności niż zapewniają zwykłe odwołania.

### Having Multiple Owners of Mutable Data by Combining `Rc<T>` and `RefCell<T>`

Powszechnym sposobem użycia `RefCell<T>` jest połączenie z `Rc<T>`. Przypomnijmy, że
`Rc<T>` pozwala mieć wielu właścicieli pewnych danych, ale daje tylko niezmienny
dostęp do tych danych. Jeśli masz `Rc<T>`, który przechowuje `RefCell<T>`, możesz
pobrać wartość, która może mieć wielu właścicieli *i* którą możesz mutować!

Na przykład przypomnij sobie przykład listy wad z Listingu 15-18, gdzie użyliśmy
`Rc<T>`, aby umożliwić wielu listom współdzielenie własności innej listy. Ponieważ
`Rc<T>` przechowuje tylko niezmienne wartości, nie możemy zmienić żadnej z wartości na liście,
gdy ją utworzymy. Dodajmy `RefCell<T>`, aby uzyskać możliwość
zmiany wartości na listach. Wylistowanie 15-24 pokazuje, że używając
`RefCell<T>` w definicji `Cons`, możemy zmodyfikować wartość przechowywaną na wszystkich
listach:

<Listing number="15-24" file-name="src/main.rs" caption="Using `Rc<RefCell<i32>>` to create a `List` that we can mutate">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-24/src/main.rs}}
```

</Listing>

Tworzymy wartość, która jest instancją `Rc<RefCell<i32>>` i przechowujemy ją w zmiennej o nazwie `value`, abyśmy mogli uzyskać do niej bezpośredni dostęp później. Następnie tworzymy
`List` w `a` z wariantem `Cons`, który zawiera `value`. Musimy sklonować
`value`, aby zarówno `a`, jak i `value` miały własność wewnętrznej wartości `5`, zamiast
przenosić własność z `value` na `a` lub pożyczać `a` od
`value`.

Owijamy listę `a` w `Rc<T>`, więc gdy tworzymy listy `b` i `c`, obie
mogą odnosić się do `a`, co zrobiliśmy w Liście 15-18.

Po utworzeniu list w `a`, `b` i `c` chcemy dodać 10 do
value w `value`. Robimy to, wywołując `borrow_mut` na `value`, co wykorzystuje
funkcję automatycznego dereferencjonowania, którą omówiliśmy w rozdziale 5 (zobacz sekcję
[„Gdzie jest operator `->`?”][wheres-the---operator]<!-- ignoruj ​​-->), aby
dereferencjonować `Rc<T>` do wewnętrznej wartości `RefCell<T>`. Metoda `borrow_mut`
zwraca inteligentny wskaźnik `RefMut<T>`, a my używamy na nim operatora dereferencyjnego
i zmieniamy wewnętrzną wartość.

Kiedy drukujemy `a`, `b` i `c`, możemy zobaczyć, że wszystkie mają zmodyfikowaną
wartość 15, a nie 5:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-24/output.txt}}
```

Ta technika jest całkiem niezła! Używając `RefCell<T>`, mamy zewnętrznie
niezmienną wartość `List`. Ale możemy użyć metod w `RefCell<T>`, które zapewniają
dostęp do jego wewnętrznej zmienności, dzięki czemu możemy modyfikować nasze dane, gdy zajdzie taka potrzeba.
Sprawdzanie reguł pożyczania w czasie wykonywania chroni nas przed wyścigami danych i
czasem warto poświęcić trochę szybkości na tę elastyczność w naszych strukturach danych. Należy zauważyć, że `RefCell<T>` nie działa w przypadku kodu wielowątkowego!
`Mutex<T>` jest wersją `RefCell<T>` bezpieczną dla wątków i omówimy
`Mutex<T>` w rozdziale 16.

[wheres-the---operator]: ch05-03-method-syntax.html#wheres-the---operator
