## Niebezpieczny Rust

Cały kod, który omówiliśmy do tej pory, miał gwarancje bezpieczeństwa pamięci Rusta
wymuszone w czasie kompilacji. Jednak Rust ma ukryty w sobie drugi język,
który nie wymusza tych gwarancji bezpieczeństwa pamięci: nazywa się *niebezpieczny Rust*
i działa tak samo jak zwykły Rust, ale daje nam dodatkowe supermoce.

Niebezpieczny Rust istnieje, ponieważ analiza statyczna jest z natury konserwatywna. Kiedy
kompilator próbuje ustalić, czy kod spełnia gwarancje,
lepiej jest dla niego odrzucić niektóre prawidłowe programy niż zaakceptować niektóre nieprawidłowe
programy. Chociaż kod *może* być w porządku, jeśli kompilator Rusta nie ma
wystarczającej ilości informacji, aby być pewnym, odrzuci kod. W takich przypadkach,
możesz użyć niebezpiecznego kodu, aby powiedzieć kompilatorowi: „Zaufaj mi, wiem, co robię”. Należy jednak pamiętać, że używasz niebezpiecznego Rusta na własne ryzyko: jeśli
używasz niebezpiecznego kodu niepoprawnie, mogą wystąpić problemy z powodu braku bezpieczeństwa pamięci, takie jak
dereferencja wskaźnika zerowego.

Innym powodem, dla którego Rust ma niebezpieczne alter ego, jest to, że podstawowy sprzęt komputerowy
jest z natury niebezpieczny. Gdyby Rust nie pozwalał na wykonywanie niebezpiecznych operacji, nie mógłbyś
wykonywać pewnych zadań. Rust musi umożliwiać programowanie systemów niskiego poziomu, takie jak bezpośrednia interakcja z systemem operacyjnym lub nawet
pisanie własnego systemu operacyjnego. Praca z programowaniem systemów niskiego poziomu jest jednym z celów tego języka. Przyjrzyjmy się, co możemy zrobić z niebezpiecznym
Rust i jak to zrobić.

### Niebezpieczne supermoce

Aby przejść do niebezpiecznego Rusta, użyj słowa kluczowego `unsafe`, a następnie rozpocznij nowy blok,
który zawiera niebezpieczny kod. Możesz wykonać pięć czynności w niebezpiecznym Ruście, których nie możesz wykonać w bezpiecznym Ruście, które nazywamy *niebezpiecznymi supermocami*. Te supermoce
obejmują możliwość:

* Dereferencji surowego wskaźnika
* Wywołania niebezpiecznej funkcji lub metody
* Dostępu do zmiennej zmiennej statycznej lub jej modyfikacji
* Implementacji niebezpiecznej cechy
* Dostępu do pól `unii`

Ważne jest, aby zrozumieć, że `unsafe` nie wyłącza sprawdzania pożyczania
ani nie wyłącza żadnych innych kontroli bezpieczeństwa Rusta: jeśli użyjesz referencji w niebezpiecznym
kodzie, nadal będzie ona sprawdzana. Słowo kluczowe `unsafe` daje dostęp tylko do
tych pięciu funkcji, które następnie nie są sprawdzane przez kompilator pod kątem bezpieczeństwa pamięci. Nadal uzyskasz pewien stopień bezpieczeństwa wewnątrz niebezpiecznego bloku.

Ponadto, `unsafe` nie oznacza, że ​​kod wewnątrz bloku jest koniecznie
niebezpieczny lub że na pewno będzie miał problemy z bezpieczeństwem pamięci: intencją jest,
aby jako programista, zapewnić, że kod wewnątrz `unsafe` bloku będzie miał
dostęp do pamięci w prawidłowy sposób.

Ludzie są omylni i błędy się zdarzają, ale wymagając, aby te pięć
niebezpiecznych operacji znajdowało się wewnątrz bloków oznaczonych adnotacją `unsafe`, będziesz wiedzieć, że
wszelkie błędy związane z bezpieczeństwem pamięci muszą znajdować się w `unsafe` bloku. Utrzymuj
`unsafe` bloki małe; będziesz wdzięczny później, gdy będziesz badać błędy pamięci.

Aby jak najbardziej odizolować niebezpieczny kod, najlepiej jest umieścić niebezpieczny kod
w bezpiecznej abstrakcji i zapewnić bezpieczne API, o czym porozmawiamy później w rozdziale, gdy będziemy badać niebezpieczne funkcje i metody. Części standardowej
biblioteki są implementowane jako bezpieczne abstrakcje w niebezpiecznym kodzie, który został
skontrolowany. Owinięcie niebezpiecznego kodu bezpieczną abstrakcją zapobiega wyciekaniu użycia `unsafe`
do wszystkich miejsc, w których Ty lub Twoi użytkownicy moglibyście chcieć użyć funkcjonalności zaimplementowanej za pomocą `unsafe` kodu, ponieważ użycie bezpiecznej
abstrakcji jest bezpieczne.

Przyjrzyjmy się po kolei każdej z pięciu niebezpiecznych supermocy. Przyjrzymy się również
niektórym abstrakcjom, które zapewniają bezpieczny interfejs dla niebezpiecznego kodu.

### Dereferencja surowego wskaźnika

W rozdziale 4, w sekcji [„Dangling References”][dangling-references]<!-- ignore
-->, wspomnieliśmy, że kompilator zapewnia, że ​​odwołania są zawsze
poprawne. Niebezpieczny Rust ma dwa nowe typy zwane *raw pointers*, które są podobne do
references. Podobnie jak w przypadku referencji, surowe wskaźniki mogą być niezmienne lub zmienne i
są zapisywane odpowiednio jako `*const T` i `*mut T`. Gwiazdka nie jest
operatorem dereferencji; jest częścią nazwy typu. W kontekście surowych
wskaźników, *immutable* oznacza, że ​​wskaźnika nie można bezpośrednio przypisać
po dereferencji.

W odróżnieniu od odniesień i inteligentnych wskaźników, surowe wskaźniki:

* Mogą ignorować reguły pożyczania, mając zarówno niezmienne, jak i
zmienne wskaźniki lub wiele zmiennych wskaźników do tej samej lokalizacji
* Nie mają gwarancji, że wskażą na prawidłową pamięć
* Mogą być nullem
* Nie implementują żadnego automatycznego czyszczenia

Rezygnując z wymuszania tych gwarancji przez Rust, możesz zrezygnować z
gwarantowanego bezpieczeństwa w zamian za większą wydajność lub możliwość
interfejsu z innym językiem lub sprzętem, gdzie gwarancje Rust nie mają zastosowania.

Listing 20-1 pokazuje, jak utworzyć niezmienny i zmienny surowy wskaźnik z
odniesień.

<Listing number="20-1" caption="Creating raw pointers from references">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-01/src/main.rs:here}}
```

</Listing>

Zauważ, że nie uwzględniamy słowa kluczowego `unsafe` w tym kodzie. Możemy tworzyć
surowe wskaźniki w bezpiecznym kodzie; nie możemy jednak dereferencjonować surowych wskaźników poza
niebezpiecznym blokiem, jak zobaczysz za chwilę.

Utworzyliśmy surowe wskaźniki, używając `as` do rzutowania niezmiennej i zmiennej
referencji na odpowiadające im typy surowych wskaźników. Ponieważ utworzyliśmy je
bezpośrednio z referencji, których poprawność jest gwarantowana, wiemy, że te konkretne surowe
wskaźniki są prawidłowe, ale nie możemy zakładać tego w odniesieniu do dowolnego surowego
wskaźnika.

Aby to zademonstrować, utworzymy następnie surowy wskaźnik, którego poprawności nie możemy być
pewni. Wypis 20-2 pokazuje, jak utworzyć surowy wskaźnik do dowolnej
lokalizacji w pamięci. Próba użycia dowolnej pamięci jest niezdefiniowana: pod tym adresem mogą znajdować się
dane lub ich nie być, kompilator może zoptymalizować kod,
tak aby nie było dostępu do pamięci, lub program może zgłosić błąd segmentacji. Zwykle nie ma dobrego powodu, aby pisać taki kod, ale jest to
możliwe.

<Listing number="20-2" caption="Creating a raw pointer to an arbitrary memory address">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-02/src/main.rs:here}}
```

</Listing>

Przypomnijmy, że możemy tworzyć surowe wskaźniki w bezpiecznym kodzie, ale nie możemy *dereferować*
surowych wskaźników i odczytywać danych, na które wskazują. W Liście 20-3 używamy
operatora dereferencji `*` na surowym wskaźniku, który wymaga bloku `unsafe`.

<Listing number="20-3" caption="Dereferencing raw pointers within an `unsafe` block">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-03/src/main.rs:here}}
```

</Listing>

Utworzenie wskaźnika nie powoduje żadnych szkód; tylko wtedy, gdy próbujemy uzyskać dostęp do wartości, na którą on wskazuje, możemy skończyć z nieprawidłową wartością.

Należy również zauważyć, że w Listingu 20-1 i 20-3 utworzyliśmy surowe wskaźniki `*const i32` i `*mut i32`, które oba wskazywały na tę samą lokalizację w pamięci, gdzie przechowywany jest `num`. Gdybyśmy zamiast tego próbowali utworzyć niezmienną i zmienną referencję do
`num`, kod nie zostałby skompilowany, ponieważ reguły własności Rusta nie
dopuszczają zmiennej referencji w tym samym czasie, co jakiekolwiek niezmienne referencje. Za pomocą
surowych wskaźników możemy utworzyć zmienny wskaźnik i niezmienny wskaźnik do tej samej lokalizacji i zmienić dane za pomocą zmiennego wskaźnika, co potencjalnie
może spowodować wyścig danych. Uważaj!

Przy wszystkich tych zagrożeniach, dlaczego miałbyś kiedykolwiek używać surowych wskaźników? Jednym z głównych przypadków użycia jest interfejs z kodem C, jak zobaczysz w następnej sekcji,
[„Wywoływanie niebezpiecznej funkcji lub
metody.”](#wywoływanie-niebezpiecznej-funkcji-lub-metody)<!-- ignore --> Innym przypadkiem jest
budowanie bezpiecznych abstrakcji, których sprawdzanie pożyczania nie rozumie.
Wprowadzimy niebezpieczne funkcje, a następnie przyjrzymy się przykładowi bezpiecznej
abstrakcji, która używa niebezpiecznego kodu.

### Wywoływanie niebezpiecznej funkcji lub metody

Drugim typem operacji, którą możesz wykonać w bloku unsafe, jest wywoływanie
niebezpiecznych funkcji. Niebezpieczne funkcje i metody wyglądają dokładnie tak samo jak zwykłe
funkcje i metody, ale mają dodatkowy `unsafe` przed resztą
definicji. Słowo kluczowe `unsafe` w tym kontekście wskazuje, że funkcja ma
wymagania, które musimy spełnić, gdy ją wywołujemy, ponieważ Rust nie może
zagwarantować, że spełniliśmy te wymagania. Wywołując niebezpieczną funkcję w bloku
`unsafe`, mówimy, że przeczytaliśmy dokumentację tej funkcji i
bierzemy odpowiedzialność za przestrzeganie kontraktów funkcji.

Oto niebezpieczna funkcja o nazwie `dangerous`, która nie robi niczego w swoim
body:
```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-01-unsafe-fn/src/main.rs:here}}
```

We must call the `dangerous` function within a separate `unsafe` block. If we
try to call `dangerous` without the `unsafe` block, we’ll get an error:

```console
{{#include ../listings/ch20-advanced-features/output-only-01-missing-unsafe/output.txt}}
```

With the `unsafe` block, we’re asserting to Rust that we’ve read the function’s
documentation, we understand how to use it properly, and we’ve verified that
we’re fulfilling the contract of the function.

Bodies of unsafe functions are effectively `unsafe` blocks, so to perform other
unsafe operations within an unsafe function, we don’t need to add another
`unsafe` block.

#### Tworzenie bezpiecznej abstrakcji na niebezpiecznym kodzie

Tylko dlatego, że funkcja zawiera niebezpieczny kod, nie oznacza, że ​​musimy oznaczyć
całą funkcję jako niebezpieczną. W rzeczywistości, owinięcie niebezpiecznego kodu w bezpieczną funkcję jest
typową abstrakcją. Jako przykład, przeanalizujmy funkcję `split_at_mut`
ze standardowej biblioteki, która wymaga pewnego niebezpiecznego kodu. Przeanalizujemy,
jak możemy ją zaimplementować. Ta bezpieczna metoda jest zdefiniowana na zmiennych wycinkach: bierze
jeden wycinek i tworzy dwa, dzieląc wycinek pod indeksem podanym jako
argument. Listing 20-4 pokazuje, jak używać `split_at_mut`.

<Listing number="20-4" caption="Using the safe `split_at_mut` function">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-04/src/main.rs:here}}
```

</Listing>

Nie możemy zaimplementować tej funkcji używając tylko bezpiecznego Rusta. Próba mogłaby wyglądać
jak Listing 20-5, który się nie skompiluje. Dla uproszczenia zaimplementujemy `split_at_mut` jako funkcję, a nie metodę i tylko dla wycinków
wartości `i32`, a nie dla typu generycznego `T`.

<Listing number="20-5" caption="An attempted implementation of `split_at_mut` using only safe Rust">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-05/src/main.rs:here}}
```

</Listing>

Ta funkcja najpierw pobiera całkowitą długość wycinka. Następnie potwierdza, że
indeks podany jako parametr znajduje się w wycinku, sprawdzając, czy jest
mniejszy lub równy długości. To potwierdzenie oznacza, że ​​jeśli przekażemy indeks,
który jest większy niż długość podziału wycinka, funkcja wpadnie w panikę,
zanim spróbuje użyć tego indeksu.

Następnie zwracamy dwa zmienne wycinki w krotce: jeden od początku
oryginalnego wycinka do indeksu `mid` i drugi od `mid` do końca
wycinka.

Gdy spróbujemy skompilować kod z Listingu 20-5, otrzymamy błąd.

```console
{{#include ../listings/ch20-advanced-features/listing-20-05/output.txt}}
```

Sprawdzanie pożyczania w Rust nie rozumie, że pożyczamy różne części
wycinka; wie tylko, że pożyczamy z tego samego wycinka dwa razy.
Pożyczanie różnych części wycinka jest zasadniczo w porządku, ponieważ dwa
wycinki się nie nakładają, ale Rust nie jest wystarczająco inteligentny, aby to wiedzieć. Kiedy
wiemy, że kod jest w porządku, a Rust nie, czas sięgnąć po niebezpieczny kod.

Listing 20-6 pokazuje, jak używać  `unsafe` bloku, surowego wskaźnika i kilku wywołań
niebezpiecznych funkcji, aby implementacja `split_at_mut` działała.

<Listing number="20-6" caption="Using unsafe code in the implementation of the `split_at_mut` function">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-06/src/main.rs:here}}
```

</Listing>

Przypomnij sobie z sekcji [„Typ wycinka”][the-slice-type]<!-- ignoruj ​​--> w
rozdziale 4, że wycinki są wskaźnikami do pewnych danych i długości wycinka.
Używamy metody `len`, aby uzyskać długość wycinka i metody `as_mut_ptr`,
aby uzyskać dostęp do surowego wskaźnika wycinka. W tym przypadku, ponieważ mamy
zmienny wycinek do wartości `i32`, `as_mut_ptr` zwraca surowy wskaźnik typu
`*mut i32`, który zapisaliśmy w zmiennej `ptr`.

Zachowuje się twierdzenie, że indeks `mid` znajduje się w wycinku. Następnie przechodzimy do
niebezpiecznego kodu: funkcja `slice::from_raw_parts_mut` przyjmuje surowy wskaźnik
i długość i tworzy wycinek. Używamy tej funkcji, aby utworzyć wycinek,
który zaczyna się od `ptr` i ma długość `mid` elementów. Następnie wywołujemy metodę `add`
na `ptr` z `mid` jako argumentem, aby uzyskać surowy wskaźnik, który zaczyna się od
`mid`, i tworzymy wycinek, używając tego wskaźnika i pozostałej liczby
elementów po `mid` jako długości.

Funkcja `slice::from_raw_parts_mut` jest niebezpieczna, ponieważ przyjmuje surowy
wskaźnik i musi ufać, że ten wskaźnik jest prawidłowy. Metoda `add` na surowych
wskaźnikach jest również niebezpieczna, ponieważ musi ufać, że lokalizacja przesunięcia jest również prawidłowym
wskaźnikiem. Dlatego musieliśmy umieścić blok `unsafe` wokół naszych wywołań do
`slice::from_raw_parts_mut` i `add`, abyśmy mogli je wywołać. Patrząc
na kod i dodając stwierdzenie, że `mid` musi być mniejsze lub równe
`len`, możemy stwierdzić, że wszystkie surowe wskaźniki używane w bloku `unsafe`
będą prawidłowymi wskaźnikami do danych w wycinku. To jest dopuszczalne i
odpowiednie użycie `unsafe`.

Należy zauważyć, że nie musimy oznaczać powstałej funkcji `split_at_mut` jako
`unsafe` i możemy wywołać tę funkcję z bezpiecznego Rusta. Stworzyliśmy bezpieczną
abstrakcję do niebezpiecznego kodu z implementacją funkcji, która używa
`unsafe` kodu w bezpieczny sposób, ponieważ tworzy tylko prawidłowe wskaźniki z danych, do których ta funkcja ma dostęp.

W przeciwieństwie do tego, użycie `slice::from_raw_parts_mut` w Liście 20-7 prawdopodobnie spowodowałoby awarię, gdy użyty zostanie wycinek. Ten kod zajmuje dowolną lokalizację pamięci i tworzy wycinek o długości 10 000 elementów.

<Listing number="20-7" caption="Creating a slice from an arbitrary memory location">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-07/src/main.rs:here}}
```

</Listing>

Nie jesteśmy właścicielami pamięci w tej dowolnej lokalizacji i nie ma gwarancji, że wycinek tworzony przez ten kod zawiera prawidłowe wartości `i32`. Próba użycia
`wartości` tak, jakby był to prawidłowy wycinek, skutkuje niezdefiniowanym zachowaniem.

#### Używając `extern` Funkcja do wywołania kodu zewnętrznego

Czasami kod Rust może wymagać interakcji z kodem napisanym w innym
języku. W tym celu Rust ma słowo kluczowe `extern`, które ułatwia tworzenie
i używanie *Foreign Function Interface (FFI)*. FFI to sposób, w jaki
język programowania definiuje funkcje i umożliwia innemu (obcemu)
językowi programowania wywoływanie tych funkcji.

Listing 20-8 pokazuje, jak skonfigurować integrację z funkcją `abs`
ze standardowej biblioteki C. Funkcje zadeklarowane w blokach `extern` są
zawsze niebezpieczne do wywołania z kodu Rust. Powodem jest to, że inne języki nie
egzekwowają reguł i gwarancji Rust, a Rust nie może ich sprawdzić, więc
odpowiedzialność za zapewnienie bezpieczeństwa spoczywa na programiście.

<Listing number="20-8" file-name="src/main.rs" caption="Declaring and calling an `extern` function defined in another language">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-08/src/main.rs}}
```

</Listing>

W bloku `extern "C"` wymieniamy nazwy i sygnatury zewnętrznych
funkcji z innego języka, które chcemy wywołać. Część `"C"` definiuje, którego
*interfejsu binarnego aplikacji (ABI)* używa zewnętrzna funkcja: ABI
definiuje sposób wywołania funkcji na poziomie asemblera. ABI `"C"` jest
najbardziej powszechny i ​​podąża za ABI języka programowania C.

> #### Wywoływanie funkcji Rust z innych języków
>
> Możemy również użyć `extern`, aby utworzyć interfejs, który pozwala innym językom
> wywoływać funkcje Rust. Zamiast tworzyć cały blok `extern`, dodajemy
> słowo kluczowe `extern` i określamy ABI do użycia tuż przed słowem kluczowym `fn`
> dla odpowiedniej funkcji. Musimy również dodać adnotację `#[no_mangle]`, aby
> powiedzieć kompilatorowi Rust, aby nie zmieniał nazwy tej funkcji. *Mangling* to
> sytuacja, gdy kompilator zmienia nazwę, którą nadaliśmy funkcji, na inną nazwę,
> która zawiera więcej informacji do wykorzystania przez inne części procesu kompilacji,
> ale jest mniej czytelna dla człowieka. Każdy kompilator języka programowania
> zmienia nazwy nieco inaczej, więc aby funkcja Rust mogła być nazwana przez
> inne języki, musimy wyłączyć mangling nazw kompilatora Rust.
>
> W poniższym przykładzie udostępniamy funkcję `call_from_c` z kodu C, po jej skompilowaniu do biblioteki współdzielonej i powiązaniu z C:
>
> ```rust
> #[no_mangle]
> pub extern "C" fn call_from_c() {
>     println!("Just called a Rust function from C!");
> }
> ```
>
> Takie użycie `extern` nie wymaga `unsafe`.

### Dostęp do zmiennej statycznej lub jej modyfikacja

W tej książce nie omawialiśmy jeszcze *zmiennych globalnych*, które Rust obsługuje, ale mogą być problematyczne z powodu reguł własności Rusta. Jeśli dwa wątki
uzyskują dostęp do tej samej zmiennej zmiennej globalnej, może to spowodować wyścig danych.

W Ruście zmienne globalne nazywane są zmiennymi *statycznymi*. Listing 20-9 pokazuje
przykładową deklarację i użycie zmiennej statycznej z wycinkiem ciągu jako
wartością.

<Listing number="20-9" file-name="src/main.rs" caption="Defining and using an immutable static variable">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-09/src/main.rs}}
```

</Listing>

Zmienne statyczne są podobne do stałych, które omówiliśmy w sekcji
[„Różnice między zmiennymi a
stałymi”][differences-between-variables-and-constants]<!-- ignore -->
w rozdziale 3. Nazwy zmiennych statycznych są w `SCREAMING_SNAKE_CASE` zgodnie z
konwencją. Zmienne statyczne mogą przechowywać tylko odwołania z `'static`
czasem życia, co oznacza, że ​​kompilator Rust może ustalić czas życia i nie musimy go jawnie adnotować. Dostęp do niezmiennej zmiennej statycznej
jest bezpieczny.

Subtelna różnica między stałymi a niezmiennymi zmiennymi statycznymi polega na tym, że
wartości w zmiennej statycznej mają stały adres w pamięci. Użycie wartości
zawsze umożliwi dostęp do tych samych danych. Stałe z kolei mogą
duplikować swoje dane, kiedykolwiek są używane. Inna różnica polega na tym, że zmienne
statyczne mogą być zmienne. Dostęp do zmiennych statycznych i ich modyfikacja są
*niebezpieczne*. Listing 20-10 pokazuje, jak zadeklarować, uzyskać dostęp i zmodyfikować zmienną
statyczną o nazwie `COUNTER`.

<Listing number="20-10" file-name="src/main.rs" caption="Reading from or writing to a mutable static variable is unsafe">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-10/src/main.rs}}
```

</Listing>

Podobnie jak w przypadku zwykłych zmiennych, określamy zmienność za pomocą słowa kluczowego `mut`. Każdy
kod, który odczytuje lub zapisuje z `COUNTER` musi znajdować się w bloku `unsafe`. Ten
kod kompiluje się i drukuje `COUNTER: 3` tak, jak byśmy się spodziewali, ponieważ jest
jednowątkowy. Dostęp wielu wątków do `COUNTER` prawdopodobnie doprowadziłby do
wyścigów danych.

W przypadku zmiennych danych, które są globalnie dostępne, trudno jest zapewnić, że
nie będzie żadnych wyścigów danych, dlatego Rust uważa zmienne zmienne statyczne za
niebezpieczne. Jeśli to możliwe, lepiej jest używać technik współbieżności i
bezpiecznych dla wątków inteligentnych wskaźników, które omówiliśmy w rozdziale 16, aby kompilator sprawdzał,
czy dane dostępne z różnych wątków są bezpieczne.

### Wdrażanie niebezpiecznej cechy

Możemy użyć `unsafe`, aby zaimplementować niebezpieczną cechę. Cecha jest niebezpieczna, gdy
przynajmniej jedna z jej metod ma jakiś niezmiennik, którego kompilator nie może zweryfikować.
Deklarujemy, że cecha jest `unsafe``, dodając słowo kluczowe `unsafe` przed `trait`
i oznaczając implementację cechy jako `unsafe`, jak pokazano w
Listingu 20-11.

<Listing number="20-11" caption="Defining and implementing an unsafe trait">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-11/src/main.rs}}
```

</Listing>

Używając `unsafe impl`, obiecujemy, że będziemy przestrzegać niezmienników, których
kompilator nie może zweryfikować.

Jako przykład przypomnijmy sobie cechy znaczników `Sync` i `Send`, które omówiliśmy w
[„Rozszerzalna współbieżność z cechami `Sync` i `Send`
”][extensible-concurrency-with-the-sync-and-send-traits]<!-- ignoruj ​​-->
rozdziału 16: kompilator implementuje te cechy automatycznie,
jeśli nasze typy składają się wyłącznie z typów `Send` i `Sync`. Jeśli zaimplementujemy
typ, który zawiera typ inny niż `Send` lub `Sync`, taki jak surowe wskaźniki,
i chcemy oznaczyć ten typ jako `Send` lub `Sync`, musimy użyć `unsafe`. Rust
nie może zweryfikować, czy nasz typ przestrzega gwarancji, że można go bezpiecznie wysłać
przez wątki lub uzyskać do niego dostęp z wielu wątków; Dlatego musimy wykonać te sprawdzenia ręcznie i oznaczyć je jako `unsafe`.

### Dostęp do pól unii

Ostatnią akcją, która działa tylko z `unsafe`, jest dostęp do pól
*union*. `Union` jest podobny do `struct`, ale tylko jedno zadeklarowane pole jest
używane w danym wystąpieniu na raz. Unie są używane głównie do
interfejsu z uniami w kodzie C. Dostęp do pól unii jest niebezpieczny, ponieważ Rust
nie może zagwarantować typu danych aktualnie przechowywanych w wystąpieniu unii. Możesz dowiedzieć się więcej o uniach w [Rust Reference][reference].

### Kiedy używać niebezpiecznego kodu

Użycie `unsafe` do wykonania jednej z pięciu czynności (supermocy) omówionych powyżej
nie jest złe, ani nawet źle widziane. Jednak trudniej jest uzyskać poprawny kod `unsafe`,
ponieważ kompilator nie może pomóc w utrzymaniu bezpieczeństwa pamięci. Kiedy masz
powód, aby użyć kodu `unsafe`, możesz to zrobić, a posiadanie wyraźnej adnotacji `unsafe`
ułatwia śledzenie źródła problemów, gdy wystąpią.

[dangling-references]:
ch04-02-references-and-borrowing.html#dangling-references
[differences-between-variables-and-constants]:
ch03-01-variables-and-mutability.html#constants
[extensible-concurrency-with-the-sync-and-send-traits]:
ch16-04-extensible-concurrency-sync-and-send.html#extensible-concurrency-with-the-sync-and-send-traits
[the-slice-type]: ch04-03-slices.html#the-slice-type
[reference]: ../reference/items/unions.html
