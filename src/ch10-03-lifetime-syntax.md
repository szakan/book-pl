## Walidacja odwołań za pomocą okresów życia

Czasy życia to kolejny rodzaj generyczności, z których już korzystaliśmy. Zamiast
upewniać się, że typ ma pożądane zachowanie, czasy życia zapewniają, że
odniesienia są prawidłowe tak długo, jak tego potrzebujemy.

Jednym szczegółem, którego nie omówiliśmy w sekcji [“References and
Borrowing”][references-and-borrowing]<!-- ignore --> w rozdziale 4, jest to, że
każde odniesienie w Rust ma *czas życia*, który jest zakresem, w którym
to odniesienie jest prawidłowe. W większości przypadków czasy życia są niejawne i wnioskowane,
podobnie jak w większości przypadków typy są wnioskowane. Musimy adnotować typy tylko wtedy,
gdy możliwe jest wiele typów. W podobny sposób musimy adnotować czasy życia,
gdy czasy życia odniesień mogą być powiązane na kilka różnych sposobów. Rust
wymaga od nas adnotacji relacji przy użyciu ogólnych parametrów czasu życia,
aby upewnić się, że rzeczywiste odniesienia używane w czasie wykonywania będą z pewnością prawidłowe.

Adnotowanie okresów życia nie jest nawet koncepcją, którą posiada większość innych języków programowania, więc będzie to dla Ciebie nieznane. Chociaż nie omówimy okresów życia w całości w tym rozdziale, omówimy typowe sposoby, w jakie możesz się zetknąć ze składnią okresu życia, abyś mógł się oswoić z tą koncepcją.

### Zapobieganie zwisającym odniesieniom za pomocą czasów życia

Głównym celem czasów życia jest zapobieganie *wiszącym odniesieniom*, które powodują, że
program odwołuje się do danych innych niż te, do których ma się odwoływać.
Rozważmy program z Listingu 10-16, który ma zakres zewnętrzny i zakres wewnętrzny.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/src/main.rs}}
```

<span class="caption">Wylistowanie 10-16: Próba użycia odniesienia, którego wartość wyszła poza zakres</span>

> Uwaga: Przykłady w listach 10-16, 10-17 i 10-23 deklarują zmienne
> bez nadawania im wartości początkowej, więc nazwa zmiennej istnieje w
> zakresie zewnętrznym. Na pierwszy rzut oka może się to wydawać sprzeczne z brakiem wartości null w Rust. Jednak jeśli spróbujemy użyć zmiennej przed nadaniem jej
> wartości, otrzymamy błąd kompilacji, który pokazuje, że Rust rzeczywiście
> nie zezwala na wartości null.

Zakres zewnętrzny deklaruje zmienną o nazwie `r` bez wartości początkowej, a zakres wewnętrzny deklaruje zmienną o nazwie `x` z wartością początkową równą 5. Wewnątrz
zakresu wewnętrznego próbujemy ustawić wartość `r` jako odniesienie do `x`. Następnie
zakres wewnętrzny się kończy i próbujemy wydrukować wartość w `r`. Ten kod nie zostanie
skompilowany, ponieważ wartość, do której odnosi się `r`, wyszła poza zakres, zanim
spróbujemy jej użyć. Oto komunikat o błędzie:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/output.txt}}
```

Zmienna `x` nie „żyje wystarczająco długo”. Powodem jest to, że `x` będzie poza
zakresem, gdy wewnętrzny zakres kończy się w wierszu 7. Ale `r` jest nadal prawidłowe dla
zewnętrznego zakresu; ponieważ jego zakres jest większy, mówimy, że „żyje dłużej”. Gdyby
Rust pozwolił na działanie tego kodu, `r` odwoływałoby się do pamięci, która została
zwolniona, gdy `x` wyszło poza zakres, a wszystko, co próbowaliśmy zrobić z `r`,
nie działałoby poprawnie. Jak więc Rust ustala, że ​​ten kod jest nieprawidłowy?
Używa sprawdzania pożyczania.

### Sprawdzanie pożyczek

Kompilator Rust ma *sprawdzacz pożyczania*, który porównuje zakresy, aby ustalić,
czy wszystkie pożyczenia są prawidłowe. Listing 10-17 pokazuje ten sam kod, co Listing 10-16, ale z adnotacjami pokazującymi czasy życia zmiennych.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-17/src/main.rs}}
```

<span class="caption">Wylistowanie 10-17: Adnotacje dotyczące okresów życia `r` i
`x`, nazwanych odpowiednio  `'a` i `'b`</span>

Tutaj oznaczyliśmy czas życia `r` za pomocą `'a`, a czas życia `x`
za pomocą `'b`. Jak widać, wewnętrzny blok `'b` jest znacznie mniejszy niż zewnętrzny blok czasu życia
`'a`. W czasie kompilacji Rust porównuje rozmiar obu
czasów życia i widzi, że `r` ma czas życia `'a`, ale odwołuje się do pamięci
z czasem życia `'b`. Program jest odrzucany, ponieważ `'b` jest krótsze niż
`'a`: podmiot odniesienia nie żyje tak długo, jak odniesienie.

Listing 10-18 naprawia kod, dzięki czemu nie ma on zwisającego odniesienia i
kompiluje się bez żadnych błędów.

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-18/src/main.rs}}
```

<span class="caption">Wypis 10-18: Prawidłowe odniesienie, ponieważ dane mają dłuższy czas życia niż odniesienie</span>

Tutaj `x` ma czas życia `'b`, który w tym przypadku jest dłuższy niż `'a`. To
oznacza, że ​​`r` może odwoływać się do `x`, ponieważ Rust wie, że odwołanie w `r` będzie
zawsze prawidłowe, podczas gdy `x` jest prawidłowe.

Teraz, gdy wiesz, gdzie są czasy życia odwołań i jak Rust analizuje
czasy życia, aby zapewnić, że odwołania będą zawsze prawidłowe, przyjrzyjmy się ogólnym
czasom życia parametrów i wartościom zwracanym w kontekście funkcji.

### Ogólne czasy życia w funkcji

Napiszemy funkcję, która zwraca dłuższy z dwóch wycinków ciągu. Ta
funkcja przyjmie dwa wycinki ciągu i zwróci jeden wycinek ciągu. Po
zaimplementowaniu funkcji `longest` kod w listingu 10-19 powinien
wypisać `The longest string is abcd`.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-19/src/main.rs}}
```

<span class="caption">Wylistowanie 10-19: Funkcja `main`, która wywołuje funkcję `longest`, aby znaleźć dłuższy z dwóch fragmentów ciągu</span>

Należy zauważyć, że chcemy, aby funkcja przyjmowała wycinki ciągu, które są referencjami,
a nie ciągami, ponieważ nie chcemy, aby funkcja `najdłuższa` przejmowała
własność swoich parametrów. Zapoznaj się z sekcją [“String Slices as
Parameters”][string-slices-as-parameters]<!-- ignore --> w rozdziale 4,
aby uzyskać więcej informacji na temat tego, dlaczego parametry, których używamy w Liście 10-19, są tymi,
których chcemy.

Jeśli spróbujemy zaimplementować funkcję `najdłuższą`, jak pokazano w Liście 10-20,
nie skompiluje się.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/src/main.rs:here}}
```

<span class="caption">Wylistowanie 10-20: Implementacja funkcji  `longest` zwracająca dłuższy z dwóch fragmentów ciągu, ale jeszcze się niekompilująca</span>

Zamiast tego otrzymujemy następujący błąd dotyczący okresów życia:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/output.txt}}
```

Tekst pomocy ujawnia, że ​​typ zwracany wymaga ogólnego parametru czasu życia,
ponieważ Rust nie potrafi stwierdzić, czy zwracane odwołanie odnosi się do
`x` czy `y`. Właściwie nie wiemy tego ani jednego, ponieważ blok `if` w ciele
tej funkcji zwraca odwołanie do `x`, a blok `else` zwraca odwołanie do `y`!

Podczas definiowania tej funkcji nie znamy konkretnych wartości,
które zostaną przekazane do tej funkcji, więc nie wiemy, czy przypadek `if` czy przypadek
`else` zostanie wykonany. Nie znamy również konkretnych czasów życia
odniesień, które zostaną przekazane, więc nie możemy przyjrzeć się zakresom, jak zrobiliśmy to w
Listingach 10-17 i 10-18, aby ustalić, czy zwracane przez nas odwołanie będzie
zawsze prawidłowe. Sprawdzanie pożyczania również nie może tego ustalić, ponieważ
nie wie, jak czasy życia `x` i `y` odnoszą się do czasu życia
wartości zwracanej. Aby naprawić ten błąd, dodamy ogólne parametry czasu życia, które
definiują relację między odniesieniami, aby sprawdzanie pożyczania mogło
wykonać swoją analizę.

### Składnia adnotacji czasu życia

Adnotacje czasu życia nie zmieniają czasu życia żadnego z odniesień. Zamiast tego
opisują relacje czasu życia wielu odniesień do siebie
bez wpływu na czasy życia. Podobnie jak funkcje mogą akceptować dowolny typ,
gdy sygnatura określa parametr typu generycznego, funkcje mogą akceptować
odniesienia o dowolnym czasie życia, określając ogólny parametr czasu życia.

Adnotacje czasu życia mają niecodzienną składnię: nazwy parametrów czasu życia muszą zaczynać się od apostrofu (`'`) i zwykle są pisane małymi literami
i bardzo krótkie, jak typy generyczne. Większość osób używa nazwy `'a` jako pierwszej
adnotacji czasu życia. Adnotacje parametrów czasu życia umieszczamy po `&`
odniesienia, używając spacji do oddzielenia adnotacji od typu odniesienia.

Oto kilka przykładów: odwołanie do `i32` bez parametru czasu życia, odwołanie do `i32`, który ma parametr czasu życia o nazwie `'a` i zmienne odwołanie do `i32`, który również ma czas życia `'a`.

```rust,ignore
&i32 // odniesienie
&'a i32 // odniesienie z jawnym czasem życia
&'a mut i32 // zmienne odniesienie z jawnym czasem życia
```

Jedna adnotacja czasu życia sama w sobie nie ma większego znaczenia, ponieważ adnotacje mają na celu poinformowanie Rusta, jak ogólne parametry czasu życia wielu odniesień odnoszą się do siebie. Przyjrzyjmy się, jak adnotacje czasu życia odnoszą się do siebie w kontekście funkcji `longest`.
### Adnotacje czasu życia w sygnaturach funkcji

Aby użyć adnotacji czasu życia w sygnaturach funkcji, musimy zadeklarować
ogólne parametry *lifetime* w nawiasach kątowych między nazwą funkcji
a listą parametrów, tak jak zrobiliśmy to z ogólnymi parametrami *type*.

Chcemy, aby sygnatura wyrażała następujące ograniczenie: zwrócone
odwołanie będzie prawidłowe tak długo, jak oba parametry będą prawidłowe. Jest to
związek między czasami życia parametrów a wartością zwracaną. Nazwiemy
czas życia `'a`, a następnie dodamy go do każdego odwołania, jak pokazano w Liście
10-21.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-21/src/main.rs:here}}
```

<span class="caption">Listing 10-21: Definicja funkcji `longest`, która określa, że ​​wszystkie odwołania w sygnaturze muszą mieć taki sam czas życia
`'a`</span>

Ten kod powinien się skompilować i wygenerować oczekiwany przez nas wynik, gdy użyjemy go z funkcją
``main` z Listingu 10-19.

Sygnatura funkcji teraz mówi Rust, że przez pewien czas istnienia `'a`, funkcja
przyjmuje dwa parametry, z których oba są wycinkami ciągu, które istnieją co najmniej tak długo, jak czas istnienia `'a`. Sygnatura funkcji mówi również Rust, że wycinek ciągu zwrócony z funkcji będzie istniał co najmniej tak długo, jak czas istnienia `'a`.
W praktyce oznacza to, że czas istnienia odwołania zwróconego przez funkcję
`longest` jest taki sam, jak krótszy z czasów istnienia wartości,
do których odwołują się argumenty funkcji. To właśnie tych relacji chcemy, aby
Rust używał podczas analizowania tego kodu.

Pamiętaj, że gdy określamy parametry czasu istnienia w tej sygnaturze funkcji,
nie zmieniamy czasów istnienia żadnych przekazanych lub zwróconych wartości. Zamiast tego,
określamy, że sprawdzanie pożyczania powinno odrzucać wszelkie wartości, które nie
spełniają tych ograniczeń. Należy zauważyć, że funkcja `najdłuższa` nie musi
wiedzieć dokładnie, jak długo `x` i `y` będą żyć, tylko że pewien zakres może być
podstawiony za `'a`, który spełni ten podpis.

Podczas adnotacji okresów życia w funkcjach adnotacje umieszczane są w sygnaturze funkcji,
a nie w ciele funkcji. Adnotacje okresu życia stają się częścią
kontraktu funkcji, podobnie jak typy w sygnaturze. Posiadanie
sygnatur funkcji zawierających kontrakt okresu życia oznacza, że ​​analiza wykonywana przez kompilator Rust może być prostsza. Jeśli występuje problem ze sposobem adnotacji funkcji lub sposobem jej wywołania, błędy kompilatora mogą wskazywać na część
naszego kodu i ograniczenia bardziej precyzyjnie. Jeśli zamiast tego kompilator Rust
wyciągnął więcej wniosków na temat tego, jakie miały być relacje okresów życia,
kompilator może być w stanie wskazać tylko na użycie naszego kodu wiele kroków od przyczyny problemu.

Gdy przekazujemy konkretne odniesienia do `longest`, konkretny czas życia, który jest
podstawiany za `'a`, jest częścią zakresu `'x`, która nakłada się na
zakres `'y`. Innymi słowy, ogólny czas życia `'a` otrzyma konkretny
czas życia, który jest równy mniejszemu z okresów życia `'x` i `'y`. Ponieważ
adnotowaliśmy zwrócone odniesienie tym samym parametrem czasu życia `'a`,
zwrócone odniesienie będzie również ważne przez długość mniejszego z
okresów życia `'x` i `'y`.

Przyjrzyjmy się, w jaki sposób adnotacje czasu życia ograniczają funkcję `longest`,
przekazując odniesienia, które mają różne konkretne czasy życia. Wylistowanie 10-22 jest
prostym przykładem.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-22/src/main.rs:here}}
```

<span class="caption">Listing 10-22: Używanie funkcji `longest` z
odniesieniami do wartości `String`, które mają różne konkretne czasy życia</span>

W tym przykładzie `string1` jest prawidłowy do końca zakresu zewnętrznego, `string2`
jest prawidłowy do końca zakresu wewnętrznego, a `result` odwołuje się do czegoś,
co jest prawidłowe do końca zakresu wewnętrznego. Uruchom ten kod, a zobaczysz,
że sprawdzanie pożyczania zatwierdza; skompiluje i wydrukuje `The longest string
is long string is long`.

Następnie wypróbujmy przykład, który pokazuje, że czas życia odwołania w
`result` musi być krótszym czasem życia dwóch argumentów. Przeniesiemy
deklarację zmiennej `result` poza zakres wewnętrzny, ale pozostawimy
przypisanie wartości zmiennej `result` wewnątrz zakresu za pomocą
`string2`. Następnie przeniesiemy `println!`, który używa `result` poza zakres wewnętrzny, po zakończeniu zakresu wewnętrznego. Kod w Liście 10-23
nie zostanie skompilowany.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/src/main.rs:here}}
```

<span class="caption">Listing 10-23: Próba użycia `result` po `string2`
wyszło poza zakres</span>

Próbując skompilować ten kod, otrzymujemy następujący błąd:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/output.txt}}
```

Błąd pokazuje, że aby `result` było prawidłowe dla instrukcji `println!`,
`string2` musiałoby być prawidłowe do końca zewnętrznego zakresu. Rust wie o tym,
ponieważ adnotowaliśmy czasy życia parametrów funkcji i wartości zwracanych, używając tego samego parametru czasu życia `'a`.

Jako ludzie możemy spojrzeć na ten kod i zobaczyć, że `string1` jest dłuższy niż
`string2`, a zatem `result` będzie zawierał odwołanie do `string1`.

Ponieważ `string1` nie wyszedł jeszcze poza zakres, odwołanie do `string1` będzie
nadal prawidłowe dla instrukcji `println!`. Jednak kompilator nie widzi,
że odwołanie jest prawidłowe w tym przypadku. Powiedzieliśmy Rust, że czas życia
odniesienia zwróconego przez funkcję `longest` jest taki sam, jak krótszy z
czasów życia przekazanych odniesień. Dlatego sprawdzanie pożyczania
nie zezwala na kod z Listingu 10-23 jako potencjalnie mający nieprawidłowe odniesienie.

Spróbuj zaprojektować więcej eksperymentów, które zmieniają wartości i czasy życia
odniesień przekazanych do funkcji `longest` i sposób, w jaki zwrócone odniesienie
jest używane. Przed kompilacją stwórz hipotezy na temat tego, czy Twoje eksperymenty przejdą
sprawdzanie pożyczania; a następnie sprawdź, czy masz rację!

### Myślenie w kategoriach życia

Sposób, w jaki musisz określić parametry czasu życia, zależy od tego, co robi twoja
funkcja. Na przykład, gdybyśmy zmienili implementację funkcji
`longest` tak, aby zawsze zwracała pierwszy parametr, a nie najdłuższy
wycinek ciągu, nie musielibyśmy określać czasu życia dla parametru `y`. Poniższy
kod zostanie skompilowany:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-08-only-one-reference-with-lifetime/src/main.rs:here}}
```

Określiliśmy parametr czasu życia `'a` dla parametru `x` i typu zwracanego, ale nie dla parametru `y`, ponieważ czas życia `y` nie ma żadnego związku z czasem życia `x` ani wartością zwracaną.

Podczas zwracania odwołania z funkcji parametr czasu życia dla typu zwracanego musi odpowiadać parametrowi czasu życia dla jednego z parametrów. Jeśli
zwrócone odwołanie *nie* odnosi się do jednego z parametrów, musi odnosić się
do wartości utworzonej w tej funkcji. Byłoby to jednak odwołanie wiszące, ponieważ wartość wyjdzie poza zakres na końcu funkcji.
Rozważ tę próbę implementacji  `longest` funkcji, która się nie
skompiluje:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/src/main.rs:here}}
```

Tutaj, mimo że określiliśmy parametr czasu życia `'a` dla typu zwracanego, ta implementacja nie skompiluje się, ponieważ wartość zwracana
czas życia nie jest w ogóle powiązana z czasem życia parametrów. Oto
komunikat o błędzie, który otrzymujemy:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/output.txt}}
```

Problem polega na tym, że `result` wykracza poza zakres i zostaje wyczyszczony na końcu
`najdłuższej` funkcji. Próbujemy również zwrócić odwołanie do `result`
z funkcji. Nie ma możliwości określenia parametrów czasu życia, które
zmieniłyby zwisające odwołanie, a Rust nie pozwoli nam utworzyć zwisającego
odniesienia. W tym przypadku najlepszym rozwiązaniem byłoby zwrócenie posiadanego typu danych,
zamiast odwołania, dzięki czemu funkcja wywołująca będzie odpowiedzialna za
wyczyszczenie wartości.

Ostatecznie składnia czasu życia polega na łączeniu czasów życia różnych
parametrów i wartości zwracanych przez funkcje. Po ich połączeniu Rust ma
wystarczająco dużo informacji, aby zezwolić na operacje bezpieczne dla pamięci i zabronić operacji,
które utworzyłyby zwisające wskaźniki lub w inny sposób naruszyły bezpieczeństwo pamięci.

### Adnotacje czasu życia w definicjach struktur

Do tej pory wszystkie zdefiniowane przez nas struktury były typami typu hold owned. Możemy definiować struktury, aby
przechowywać referencje, ale w takim przypadku musielibyśmy dodać adnotację czasu życia do
każdej referencji w definicji struktury. W listingu 10-24 znajduje się struktura o nazwie
`ImportantExcerpt`, która przechowuje wycinek ciągu.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-24/src/main.rs}}
```

<span class="caption">Listing 10-24: Struktura przechowująca odniesienie, wymagająca
adnotacji czasu życia</span>

Ta struktura ma pojedyncze pole `part`, które zawiera wycinek ciągu, który jest
odniesieniem. Podobnie jak w przypadku typów danych generycznych, deklarujemy nazwę generycznego
parametru lifetime w nawiasach kątowych po nazwie struktury, dzięki czemu możemy
używać parametru lifetime w treści definicji struktury. Ta
adnotacja oznacza, że ​​instancja `ImportantExcerpt` nie może przetrwać odwołania,
które zawiera w swoim polu `part`.

Funkcja `main` tworzy tutaj instancję struktury `ImportantExcerpt`,
która zawiera odwołanie do pierwszego zdania `String` należącego do
zmiennej `novel`. Dane w `novel` istnieją przed utworzeniem instancji `ImportantExcerpt`. Ponadto `novel` nie wychodzi poza zakres, dopóki
`ImportantExcerpt` nie wyjdzie poza zakres, więc odwołanie w instancji
`ImportantExcerpt` jest prawidłowe.

### Elizja na całe życie

Dowiedziałeś się, że każde odwołanie ma czas życia i że musisz określić
parametry czasu życia dla funkcji lub struktur, które używają odwołań. Jednak w
rozdziale 4 mieliśmy funkcję w Liście 4-9, pokazaną ponownie w Liście 10-25, która
kompilowała się bez adnotacji czasu życia.

<span class="filename">Filename: src/lib.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-25/src/main.rs:here}}
```

<span class="caption">Listing 10-25: Funkcja, którą zdefiniowaliśmy w Liście 4-9, która
skompilowała się bez adnotacji czasu życia, mimo że parametr i typ zwracany
są referencjami</span>

Powód, dla którego ta funkcja kompiluje się bez adnotacji czasu życia, jest historyczny:
we wczesnych wersjach (przed 1.0) Rust, ten kod nie zostałby skompilowany, ponieważ
każde odwołanie wymagało jawnego czasu życia. W tamtym czasie sygnatura
funkcji byłaby napisana w ten sposób:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

Po napisaniu dużej ilości kodu Rust, zespół Rust odkrył, że programiści Rust
wprowadzali te same adnotacje czasu życia wielokrotnie w określonych
sytuacjach. Sytuacje te były przewidywalne i podążały za kilkoma deterministycznymi
wzorcami. Programiści zaprogramowali te wzorce w kodzie kompilatora, aby
program sprawdzający pożyczki mógł wywnioskować czasy życia w tych sytuacjach i nie
potrzebował jawnych adnotacji.

Ten fragment historii Rust jest istotny, ponieważ możliwe jest, że pojawi się więcej
deterministycznych wzorców i zostanie dodanych do kompilatora. W przyszłości
może być wymaganych jeszcze mniej adnotacji czasu życia.

Wzory zaprogramowane w analizie odniesień Rust są nazywane
*regułami elizji czasu życia*. Nie są to reguły, których powinni przestrzegać programiści; są one
zbiorem szczególnych przypadków, które kompilator weźmie pod uwagę, a jeśli Twój kod
dopasowuje się do tych przypadków, nie musisz jawnie zapisywać czasów życia.

Reguły elizji nie zapewniają pełnego wnioskowania. Jeśli Rust deterministycznie
stosuje reguły, ale nadal istnieje niejasność co do czasu życia
odwołań, kompilator nie zgadnie, jaki powinien być czas życia pozostałych
odwołań. Zamiast zgadywać, kompilator wyświetli błąd,
który możesz rozwiązać, dodając adnotacje czasu życia.

Czasy życia parametrów funkcji lub metody nazywane są *czasami życia danych wejściowych*, a
czasy życia wartości zwracanych nazywane są *czasami życia danych wyjściowych*.

Kompilator używa trzech reguł, aby ustalić czas życia odwołań,
gdy nie ma jawnych adnotacji. Pierwsza reguła dotyczy
czasów życia danych wejściowych, a druga i trzecia reguła dotyczą
czasów życia danych wyjściowych. Jeśli
kompilator dojdzie do końca trzech reguł i nadal będą istniały odwołania,
dla których nie może ustalić czasu życia, kompilator zatrzyma się z błędem.
Te reguły dotyczą definicji `fn`, a także bloków `impl`.

Pierwsza zasada jest taka, że ​​kompilator przypisuje parametr czasu życia każdemu
parametrowi, który jest referencją. Innymi słowy, funkcja z jednym parametrem otrzymuje
jeden parametr czasu życia: `fn foo<'a>(x: &'a i32)`; funkcja z dwoma
parametrami otrzymuje dwa oddzielne parametry czasu życia: `fn foo<'a, 'b>(x: &'a i32,
y: &'b i32)`; itd.

Druga zasada jest taka, że ​​jeśli istnieje dokładnie jeden wejściowy parametr czasu życia, ten
czas życia jest przypisywany do wszystkich wyjściowych parametrów czasu życia: `fn foo<'a>(x: &'a i32)
-> &'a i32`.

Trzecia zasada jest taka, że ​​jeśli istnieje wiele wejściowych parametrów czasu życia, ale
jednym z nich jest `&self` lub `&mut self`, ponieważ jest to metoda, czas życia
`self` jest przypisywany do wszystkich wyjściowych parametrów czasu życia. Ta trzecia reguła sprawia, że
metody są o wiele przyjemniejsze do czytania i pisania, ponieważ potrzeba mniej symboli.

Udawajmy, że jesteśmy kompilatorem. Zastosujemy te reguły, aby ustalić
czasy życia odwołań w sygnaturze funkcji `first_word` w
Listingu 10-25. Sygnatura zaczyna się bez żadnych okresów życia związanych z
odniesieniami:

```rust,ignore
fn first_word(s: &str) -> &str {
```

Następnie kompilator stosuje pierwszą regułę, która określa, że ​​każdy parametr
ma swój własny czas życia. Nazwiemy ją `'a` jak zwykle, więc teraz sygnatura wygląda tak:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &str {
```

Druga reguła ma zastosowanie, ponieważ istnieje dokładnie jeden czas życia wejścia. Druga
reguła określa, że ​​czas życia jednego parametru wejściowego jest przypisywany
do czasu życia wyjściowego, więc sygnatura wygląda teraz tak:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

Teraz wszystkie odwołania w tej sygnaturze funkcji mają czasy życia, a
kompilator może kontynuować analizę bez konieczności adnotowania przez programistę
czasów życia w tej sygnaturze funkcji.

Przyjrzyjmy się innemu przykładowi, tym razem używając funkcji `longest`, która
nie miała parametrów czasu życia, gdy zaczęliśmy z nią pracować w Liście 10-20:

```rust,ignore
fn longest(x: &str, y: &str) -> &str {
```

Zastosujmy pierwszą regułę: każdy parametr ma swój własny czas życia. Tym razem
mamy dwa parametry zamiast jednego, więc mamy dwa okresy życia:

```rust,ignore
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

Możesz zobaczyć, że druga reguła nie ma zastosowania, ponieważ istnieje więcej niż jeden
okres życia wejściowego. Trzecia reguła również nie ma zastosowania, ponieważ `longest` jest
funkcją, a nie metodą, więc żaden z parametrów nie jest `self`. Po
przejrzeniu wszystkich trzech reguł nadal nie ustaliliśmy, jaki jest czas życia typu zwracanego. Dlatego otrzymaliśmy błąd podczas próby skompilowania kodu w
Listingu 10-20: kompilator przejrzał reguły eliminacji czasu życia, ale nadal nie mógł ustalić wszystkich okresów życia odwołań w sygnaturze.

Ponieważ trzecia reguła dotyczy wyłącznie sygnatur metod, przyjrzymy się
czasom życia w tym kontekście, aby zobaczyć, dlaczego trzecia reguła oznacza, że ​​nie musimy zbyt często adnotować czasów życia w sygnaturach metod.

### Adnotacje czasu życia w definicjach metod

Gdy implementujemy metody w strukturze z czasami życia, używamy tej samej składni, co
parametry typu ogólnego pokazane w Liście 10-11. Miejsce, w którym deklarujemy i
używamy parametrów czasu życia, zależy od tego, czy są one powiązane z polami struktury,
czy też z parametrami metody i wartościami zwracanymi.

Nazwy czasu życia pól struktury zawsze muszą być deklarowane po słowie kluczowym `impl`,
a następnie używane po nazwie struktury, ponieważ te czasy życia są częścią
typu struktury.

W sygnaturach metod wewnątrz bloku `impl` odwołania mogą być powiązane z
czasem życia odwołań w polach struktury lub mogą być niezależne. Ponadto reguły elizji czasu życia często sprawiają, że adnotacje czasu życia
nie są konieczne w sygnaturach metod. Przyjrzyjmy się kilku przykładom użycia
struktury o nazwie `ImportantExcerpt`, którą zdefiniowaliśmy w Liście 10-24.

Najpierw użyjemy metody o nazwie `level`, której jedynym parametrem jest odwołanie do
`self`, a wartością zwracaną jest `i32`, która nie jest odwołaniem do niczego:

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:1st}}
```

Deklaracja parametru czasu życia po `impl` i jego użycie po nazwie typu
są wymagane, ale nie musimy adnotować czasu życia odwołania
do `self` z powodu pierwszej reguły elizji.

Oto przykład, w którym obowiązuje trzecia reguła elizji czasu życia:

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:3rd}}
```

Istnieją dwa okresy życia danych wejściowych, więc Rust stosuje pierwszą regułę eliminacji czasu życia
i nadaje zarówno `&self`, jak i `announcement` własne okresy życia. Następnie, ponieważ
jednym z parametrów jest `&self`, typ zwracany otrzymuje okres życia `&self`,
a wszystkie okresy życia zostały uwzględnione.

### Statyczny czas życia

Jednym ze specjalnych okresów życia, który musimy omówić, jest `'static`, co oznacza, że
dotknięte odniesienie *może* żyć przez cały czas trwania programu. Wszystkie
literały łańcuchowe mają okres życia `'static`, który możemy opisać następująco:

```rust
let s: &'static str = "I have a static lifetime.";
```

Tekst tego ciągu jest przechowywany bezpośrednio w pliku binarnym programu, który
jest zawsze dostępny. Dlatego czas życia wszystkich literałów ciągu to
`'static`.

Możesz zobaczyć sugestie, aby użyć czasu życia `'static` w komunikatach o błędach. Ale
zanim określisz `'static` jako czas życia dla odniesienia, zastanów się,
czy odniesienie, które masz, faktycznie trwa przez cały czas życia twojego
programu, czy nie, i czy chcesz, aby tak było. W większości przypadków komunikat o błędzie
sugerujący czas życia `'static` wynika z próby utworzenia wiszącego
odniesienia lub niezgodności dostępnych czasów życia. W takich przypadkach rozwiązaniem
jest naprawienie tych problemów, a nie określenie czasu życia `'static`.

## Parametry typu ogólnego, granice cech i okresy życia razem

Przyjrzyjmy się pokrótce składni określania parametrów typu ogólnego, granic cech i okresów życia – wszystko w jednej funkcji!

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-11-generics-traits-and-lifetimes/src/main.rs:here}}
```

To jest `najdłuższa` funkcja z Listingu 10-21, która zwraca dłuższy z
dwóch wycinków ciągu. Ale teraz ma dodatkowy parametr o nazwie `ann` typu generycznego `T`, który może być wypełniony dowolnym typem, który implementuje cechę `Display`,
jak określono w klauzuli `where`. Ten dodatkowy parametr zostanie wydrukowany
przy użyciu `{}`, dlatego ograniczenie cechy `Display` jest konieczne. Ponieważ
czasy życia są typem generycznym, deklaracje parametru czasu życia
`'a` i parametru typu generycznego `T` znajdują się na tej samej liście wewnątrz nawiasów kątowych po nazwie funkcji.

## Streszczenie

W tym rozdziale omówiliśmy wiele! Teraz, gdy wiesz już o parametrach
typu generycznego, cechach i granicach cech oraz ogólnych parametrach czasu życia, jesteś
gotowy do pisania kodu bez powtórzeń, który działa w wielu różnych sytuacjach.
Parametry typu generycznego pozwalają na zastosowanie kodu do różnych typów. Cechy i granice cech zapewniają, że nawet jeśli typy są generyczne, będą miały zachowanie, którego potrzebuje kod. Nauczyłeś się, jak używać adnotacji czasu życia, aby zapewnić, że
ten elastyczny kod nie będzie miał żadnych zwisających odniesień. A cała ta
analiza odbywa się w czasie kompilacji, co nie wpływa na wydajność środowiska wykonawczego!

Wierzcie lub nie, jest o wiele więcej do nauczenia się na tematy, które omówiliśmy
w tym rozdziale: Rozdział 17 omawia obiekty cech, które są innym sposobem na użycie
cech. Istnieją również bardziej złożone scenariusze obejmujące adnotacje czasu życia,
które będą potrzebne tylko w bardzo zaawansowanych scenariuszach; w tym celu należy przeczytać
[Rust Reference][reference]. Ale później nauczysz się, jak pisać testy w Rust, aby mieć pewność, że Twój kod działa tak, jak powinien.

[references-and-borrowing]:
ch04-02-references-and-borrowing.html#referencje-i-pożyczanie
[string-slices-as-parameters]:
ch04-03-slices.html#wycinki-Łańcuchów-jako-parametry
[reference]: ../reference/index.html
