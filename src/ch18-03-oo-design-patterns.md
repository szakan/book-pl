## Implementacja wzorca projektowego obiektowego

*Wzorzec stanu* jest obiektowym wzorcem projektowym. Istotą
wzorca jest to, że definiujemy zestaw stanów, jakie wartość może mieć wewnętrznie.
Stany są reprezentowane przez zestaw *obiektów stanu*, a zachowanie wartości
zmienia się w zależności od jej stanu. Przeanalizujemy przykład struktury
postu na blogu, która ma pole do przechowywania swojego stanu, który będzie obiektem stanu
z zestawu „wersja robocza”, „recenzja” lub „opublikowano”.

Obiekty stanu współdzielą funkcjonalność: w Rust oczywiście używamy struktur i
cech, a nie obiektów i dziedziczenia. Każdy obiekt stanu jest odpowiedzialny
za swoje własne zachowanie i za kontrolowanie, kiedy powinien zmienić się w inny
stan. Wartość, która przechowuje obiekt stanu, nic nie wie o różnym
zachowaniu stanów ani o tym, kiedy przejść między stanami.

Zaletą korzystania ze wzorca stanu jest to, że gdy wymagania biznesowe programu się zmienią, nie będziemy musieli zmieniać kodu
wartości przechowującej stan ani kodu, który używa wartości. Będziemy musieli tylko
zaktualizować kod wewnątrz jednego z obiektów stanu, aby zmienić jego reguły lub ewentualnie
dodać więcej obiektów stanu.

Najpierw zaimplementujemy wzorzec stanu w bardziej tradycyjny,
obiektowy sposób, a następnie zastosujemy podejście, które jest nieco bardziej naturalne w
Rust. Przyjrzyjmy się stopniowej implementacji przepływu pracy wpisu na blogu przy użyciu
wzorca stanu.

Ostateczna funkcjonalność będzie wyglądać następująco:

1. Wpis na blogu zaczyna się jako pusty szkic.

2. Po zakończeniu szkicu żądana jest recenzja wpisu.

3. Po zatwierdzeniu wpisu zostaje on opublikowany.

4. Tylko opublikowane wpisy na blogu zwracają zawartość do druku, więc niezatwierdzone wpisy nie mogą zostać przypadkowo opublikowane.

Wszelkie inne zmiany podejmowane w poście nie powinny mieć żadnego efektu. Na przykład, jeśli
spróbujemy zatwierdzić szkic wpisu na blogu przed poproszeniem o recenzję, wpis
powinien pozostać nieopublikowanym szkicem.

Listing 18-11 pokazuje ten przepływ pracy w formie kodu: to przykładowe użycie
API, które zaimplementujemy w skrzyni bibliotecznej o nazwie `blog`. To się jeszcze nie skompiluje,
ponieważ nie zaimplementowaliśmy skrzyni `blog`.

<Listing number="18-11" file-name="src/main.rs" caption="Code that demonstrates the desired behavior we want our `blog` crate to have">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:all}}
```

</Listing>

Chcemy zezwolić użytkownikowi na utworzenie nowego szkicu wpisu na blogu za pomocą `Post::new`.
Chcemy zezwolić na dodawanie tekstu do wpisu na blogu. Jeśli spróbujemy pobrać zawartość wpisu
od razu, przed zatwierdzeniem, nie powinniśmy otrzymać żadnego tekstu, ponieważ
wpis jest nadal szkicem. Dodaliśmy `assert_eq!` w kodzie w celach demonstracyjnych. Doskonałym testem jednostkowym byłoby potwierdzenie, że szkic wpisu
na blogu zwraca pusty ciąg z metody `content`, ale nie będziemy
pisać testów dla tego przykładu.

Następnie chcemy włączyć żądanie przeglądu wpisu i chcemy, aby
`content` zwracał pusty ciąg podczas oczekiwania na przegląd. Gdy wpis
otrzyma zatwierdzenie, powinien zostać opublikowany, co oznacza, że ​​tekst wpisu
zostanie zwrócony po wywołaniu `content`.

Zauważ, że jedynym typem, z którym wchodzimy w interakcję ze skrzyni, jest typ `Post`. Ten typ będzie używał wzorca stanu i będzie przechowywał wartość, która będzie
jednym z trzech obiektów stanu reprezentujących różne stany, w których może znajdować się post
—wersja robocza, oczekiwanie na recenzję lub opublikowanie. Zmiana z jednego stanu na inny
będzie zarządzana wewnętrznie w typie `Post`. Stany zmieniają się w
odpowiedzi na metody wywoływane przez użytkowników naszej biblioteki na instancji `Post`,
ale nie muszą oni bezpośrednio zarządzać zmianami stanu. Ponadto użytkownicy nie mogą
popełnić błędu w stanach, np. opublikować posta przed jego recenzją.

### Defining `Post` and Creating a New Instance in the Draft State

Zacznijmy implementację biblioteki! Wiemy, że potrzebujemy
publicznej struktury `Post`, która zawiera pewną treść, więc zaczniemy od
definicji struktury i powiązanej publicznej funkcji `new`, aby utworzyć
instancję `Post`, jak pokazano na Liście 18-12. Utworzymy również prywatną cechę
`State`, która zdefiniuje zachowanie, jakie muszą mieć wszystkie obiekty stanu dla `Post`.

Następnie `Post` będzie zawierał obiekt cechy `Box<dyn State>` wewnątrz `Option<T>`
w prywatnym polu o nazwie `state`, aby przechowywać obiekt stanu. Za chwilę zobaczysz, dlaczego
`Option<T>` jest konieczna.

<Listing number="18-12" file-name="src/lib.rs" caption="Definition of a `Post` struct and a `new` function that creates a new `Post` instance, a `State` trait, and a `Draft` struct">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-12/src/lib.rs}}
```

</Listing>

Cecha `State` definiuje zachowanie współdzielone przez różne stany postów. Obiektami stanu są `Draft`, `PendingReview` i `Published`, a wszystkie one będą implementować cechę `State`. Na razie cecha nie ma żadnych metod i
zaczniemy od zdefiniowania tylko stanu `Draft`, ponieważ to jest stan, w którym
chcemy, aby post się rozpoczął.

Gdy tworzymy nowy `Post`, ustawiamy jego pole `state` na wartość `Some`, która
zawiera `Box`. To `Box` wskazuje na nowe wystąpienie struktury `Draft`.
Gwarantuje to, że za każdym razem, gdy tworzymy nowe wystąpienie `Post`, rozpocznie się ono
jako szkic. Ponieważ pole `state` `Post` jest prywatne, nie ma możliwości
utworzenia `Post` w żadnym innym stanie! W funkcji `Post::new` ustawiamy pole
`content` na nowy, pusty `String`.

### Storing the Text of the Post Content

Widzieliśmy w Liście 18-11, że chcemy móc wywołać metodę o nazwie
`add_text` i przekazać jej `&str`, która jest następnie dodawana jako treść tekstowa
wpisu na blogu. Implementujemy to jako metodę, zamiast ujawniać pole `content`
jako `pub`, tak abyśmy później mogli zaimplementować metodę, która będzie kontrolować sposób, w jaki dane pola `content` są odczytywane. Metoda `add_text` jest dość
prosta, więc dodajmy implementację z Liście 18-13 do bloku `impl Post` :

<Listing number="18-13" file-name="src/lib.rs" caption="Implementing the `add_text` method to add text to a post’s `content`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-13/src/lib.rs:here}}
```

</Listing>

Metoda `add_text` przyjmuje zmienne odwołanie do `self`, ponieważ
zmieniamy instancję `Post`, na której wywołujemy `add_text`. Następnie wywołujemy
`push_str` na `String` w `content` i przekazujemy argument `text`, aby dodać do
zapisanego `content`. To zachowanie nie zależy od stanu, w jakim znajduje się post,
więc nie jest częścią wzorca stanu. Metoda `add_text` w ogóle nie wchodzi w interakcję
z polem `state`, ale jest częścią zachowania, które chcemy
obsługiwać.

### Ensuring the Content of a Draft Post Is Empty

Nawet po wywołaniu `add_text` i dodaniu treści do naszego wpisu, nadal
chcemy, aby metoda `content` zwracała pusty fragment ciągu, ponieważ wpis
jest nadal w stanie roboczym, jak pokazano w wierszu 7 Listingu 18-11. Na razie
zaimplementujmy metodę `content` za pomocą najprostszej rzeczy, która spełni to
wymaganie: zawsze zwracając pusty fragment ciągu. Zmienimy to później,
gdy zaimplementujemy możliwość zmiany stanu wpisu, aby można go było opublikować.
Do tej pory wpisy mogły znajdować się tylko w stanie roboczym, więc treść wpisu zawsze powinna być
pusta. Listing 18-14 pokazuje tę implementację zastępczą:

<Listing number="18-14" file-name="src/lib.rs" caption="Adding a placeholder implementation for the `content` method on `Post` that always returns an empty string slice">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-14/src/lib.rs:here}}
```

</Listing>

Dzięki tej dodanej metodzie `content` wszystko w Liście 18-11 aż do wiersza 7 działa zgodnie z przeznaczeniem.

### Requesting a Review of the Post Changes Its State

Next, we need to add functionality to request a review of a post, which should
change its state from `Draft` to `PendingReview`. Listing 18-15 shows this code:

<Listing number="18-15" file-name="src/lib.rs" caption="Implementing `request_review` methods on `Post` and the `State` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-15/src/lib.rs:here}}
```

</Listing>

Dajemy `Post` publiczną metodę o nazwie `request_review`, która przyjmie zmienne
odwołanie do `self`. Następnie wywołujemy wewnętrzną metodę `request_review` na
bieżącym stanie `Post`, a ta druga metoda `request_review` pobiera
bieżący stan i zwraca nowy stan.

Dodajemy metodę `request_review` do cechy `State`; wszystkie typy, które
implementują tę cechę, będą teraz musiały implementować metodę `request_review`.
Należy zauważyć, że zamiast mieć `self`, `&self` lub `&mut self` jako pierwszy
parametr metody, mamy `self: Box<Self>`. Ta składnia oznacza, że
metoda jest prawidłowa tylko wtedy, gdy jest wywoływana na `Box` zawierającym typ. Ta składnia przejmuje
własność `Box<Self>`, unieważniając stary stan, dzięki czemu wartość stanu
`Post` może przekształcić się w nowy stan.

Aby wykorzystać stary stan, metoda `request_review` musi przejąć własność
wartości stanu. Tutaj pojawia się `Option` w polu `state` `Post`
: wywołujemy metodę `take`, aby pobrać wartość `Some` z pola `state`
i pozostawić `None` w jego miejscu, ponieważ Rust nie pozwala nam mieć
niewypełnionych pól w strukturach. Pozwala nam to przenieść wartość `state` z
`Post` zamiast ją pożyczać. Następnie ustawimy wartość `state` posta na
wynik tej operacji.

Musimy ustawić `state` na `None` tymczasowo, zamiast ustawiać ją bezpośrednio
za pomocą kodu takiego jak `self.state = self.state.request_review();`, aby przejąć własność
wartości `state`. Dzięki temu `Post` nie będzie mógł użyć starej wartości `state` po tym, jak
przekształcimy ją w nowy stan.

Metoda `request_review` w `Draft` zwraca nową, pudełkową instancję nowej struktury
`PendingReview`, która reprezentuje stan, gdy post czeka na
recenzję. Struktura `PendingReview` implementuje również metodę `request_review`,
ale nie wykonuje żadnych transformacji. Zamiast tego zwraca samą siebie, ponieważ gdy
żądamy recenzji posta, który jest już w stanie `PendingReview`, powinien on pozostać
w stanie `PendingReview`.

Teraz możemy zacząć dostrzegać zalety wzorca stanu: metoda
`request_review` w `Post` jest taka sama, niezależnie od jej wartości `state`. Każdy
stan odpowiada za własne reguły.

Pozostawimy metodę `content` w `Post` bez zmian, zwracając pusty
wycinek ciągu. Teraz możemy mieć `Post` w stanie `PendingReview`, jak również w stanie
`Draft`, ale chcemy, aby to samo zachowanie miało miejsce w stanie `PendingReview`.
Listing 18-11 działa teraz do wiersza 10!

<!-- Old headings. Do not remove or links may break. -->
<a id="adding-the-approve-method-that-changes-the-behavior-of-content"></a>

### Adding `approve` to Change the Behavior of `content`

Metoda `approve` będzie podobna do metody `request_review`: ustawi `state` na wartość, którą bieżący stan określa jako odpowiednią, gdy ten stan zostanie zatwierdzony, jak pokazano na Liście 18-16:

<Listing number="18-16" file-name="src/lib.rs" caption="Implementing the `approve` method on `Post` and the `State` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-16/src/lib.rs:here}}
```

</Listing>

Dodajemy metodę `approve` do cechy `State` i dodajemy nową strukturę, która
implementuje `State`, stan `Published`.

Podobnie jak w przypadku `request_review` w `PendingReview`, jeśli wywołamy metodę
`approve` w `Draft`, nie będzie to miało żadnego efektu, ponieważ `approve`
zwróci `self`. Kiedy wywołamy `approve` w `PendingReview`, zwróci ona nową,
opakowaną instancję struktury `Published`. Struktura `Published` implementuje cechę
`State`, a zarówno dla metody `request_review`, jak i metody `approve`
zwraca samą siebie, ponieważ w tych przypadkach post powinien pozostać w stanie `Published`.

Teraz musimy zaktualizować metodę `content` w `Post`. Chcemy, aby wartość
zwrócona przez `content` zależała od bieżącego stanu `Post`, więc
będziemy mieć delegata `Post` do metody `content` zdefiniowanej na podstawie jego `state`,
jak pokazano na Liście 18-17:

<Listing number="18-17" file-name="src/lib.rs" caption="Updating the `content` method on `Post` to delegate to a `content` method on `State`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-17/src/lib.rs:here}}
```

</Listing>

Ponieważ celem jest zachowanie wszystkich tych reguł wewnątrz struktur implementujących
`State`, wywołujemy metodę `content` na wartości w `state` i przekazujemy instancję post (czyli `self`) jako argument. Następnie zwracamy wartość, która została
zwrócona z użycia metody `content` na wartości `state`.

Wywołujemy metodę `as_ref` na `Option`, ponieważ chcemy odwołania do
wartości wewnątrz `Option`, a nie własności wartości. Ponieważ `state`
jest `Option<Box<dyn State>>`, gdy wywołujemy `as_ref`, zwracany jest `Option<&Box<dyn
State>>`. Gdybyśmy nie wywołali `as_ref`, otrzymalibyśmy błąd, ponieważ
nie możemy przenieść `state` z pożyczonego `&self` parametru funkcji.

Następnie wywołujemy metodę `unwrap`, o której wiemy, że nigdy nie wpadnie w panikę, ponieważ
wiemy, że metody w `Post` zapewniają, że `state` zawsze będzie zawierało wartość `Some`, gdy te metody zostaną wykonane. To jeden z przypadków, o których mówiliśmy w
sekcji [„Przypadki, w których masz więcej informacji niż
kompilator”][more-info-than-rustc]<!-- ignore --> rozdziału 9, gdy
wiemy, że wartość `None` nigdy nie jest możliwa, nawet jeśli kompilator nie jest w stanie tego
zrozumieć.

W tym momencie, gdy wywołamy `content` w `&Box<dyn State>`, przymus deref
zadziała na `&` i `Box`, więc metoda `content` zostanie
ostatecznie wywołana w typie, który implementuje cechę `State`. Oznacza to, że
musimy dodać `treść` do definicji cechy `Stan` i to właśnie tam
umieścimy logikę dotyczącą tego, jaką treść zwrócić w zależności od tego, jaki stan mamy, jak pokazano na Liście 18-18:

<Listing number="18-18" file-name="src/lib.rs" caption="Adding the `content` method to the `State` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-18/src/lib.rs:here}}
```

</Listing>

Dodajemy domyślną implementację dla metody `content`, która zwraca pusty
wycinek ciągu. Oznacza to, że nie musimy implementować `content` w strukturach `Draft`
i `PendingReview`. Struktura `Published` zastąpi metodę `content`
i zwróci wartość w `post.content`.

Należy pamiętać, że potrzebujemy adnotacji czasu życia w tej metodzie, jak omówiliśmy w
rozdziale 10. Przyjmujemy odwołanie do `post` jako argument i zwracamy
odwołanie do części tego `post`, więc czas życia zwróconego odwołania jest
powiązany z czasem życia argumentu `post`.

I gotowe — teraz wszystko z Listingu 18-11 działa! Zaimplementowaliśmy wzorzec stanu
z regułami przepływu pracy wpisu na blogu. Logika związana z
regułami znajduje się w obiektach stanu, a nie jest rozproszona w całym `Post`.

> #### Dlaczego nie Enum?
>
> Być może zastanawiałeś się, dlaczego nie użyliśmy `enum` z różnymi
> możliwymi stanami postów jako wariantami. To z pewnością możliwe rozwiązanie, spróbuj
> go i porównaj wyniki końcowe, aby zobaczyć, które wolisz! Jedną z wad
> używania enum jest to, że każde miejsce, które sprawdza wartość enum, będzie potrzebowało
> wyrażenia `match` lub podobnego, aby obsłużyć każdy możliwy wariant. To może być
> bardziej powtarzalne niż to rozwiązanie z obiektem cechy.

### Trade-offs of the State Pattern

Pokazaliśmy, że Rust jest w stanie zaimplementować obiektowo zorientowany wzorzec stanu
w celu hermetyzacji różnych rodzajów zachowań, jakie post powinien mieć w
każdym stanie. Metody w `Post` nic nie wiedzą o różnych zachowaniach. W
sposób, w jaki zorganizowaliśmy kod, musimy szukać tylko w jednym miejscu, aby poznać
różne sposoby, w jakie opublikowany post może się zachowywać: implementację cechy `State`
w strukturze `Published`.

Gdybyśmy mieli stworzyć alternatywną implementację, która nie używałaby wzorca stanu, moglibyśmy zamiast tego użyć wyrażeń `match` w metodach w `Post` lub
nawet w kodzie `main`, który sprawdza stan posta i zmienia zachowanie
w tych miejscach. Oznaczałoby to, że musielibyśmy szukać w kilku miejscach,
aby zrozumieć wszystkie implikacje posta będącego w stanie opublikowanym! To
tylko by się zwiększyło, im więcej stanów dodamy: każde z tych wyrażeń `match`
potrzebowałoby innego ramienia.

W przypadku wzorca stanu metody `Post` i miejsca, w których używamy `Post` nie
potrzebują wyrażeń `match`, a aby dodać nowy stan, musielibyśmy tylko dodać
nową strukturę i zaimplementować metody cech w tej jednej strukturze.

Implementację przy użyciu wzorca stanu można łatwo rozszerzyć, aby dodać więcej
funkcjonalności. Aby zobaczyć prostotę utrzymywania kodu, który używa wzorca stanu, wypróbuj kilka z tych sugestii:

* Dodaj metodę `reject`, która zmienia stan posta z `PendingReview` z powrotem
na `Draft`.
* Wymagaj dwóch wywołań `approve`, zanim stan będzie można zmienić na `Published`.
* Zezwól użytkownikom na dodawanie treści tekstowej tylko wtedy, gdy post jest w stanie `Draft`.
Podpowiedź: niech obiekt stanu będzie odpowiedzialny za to, co może się zmienić w
treści, ale nie za modyfikowanie `Post`.

Jedną wadą wzorca stanu jest to, że ponieważ stany implementują
przejścia między stanami, niektóre stany są ze sobą sprzężone. Jeśli
dodamy inny stan między `PendingReview` i `Published`, taki jak `Scheduled`,
musielibyśmy zmienić kod w `PendingReview`, aby przejść do
`Scheduled`. Byłoby mniej pracy, gdyby `PendingReview` nie musiało się
zmieniać wraz z dodaniem nowego stanu, ale oznaczałoby to przejście
na inny wzorzec projektowy.

Inną wadą jest to, że zduplikowaliśmy część logiki. Aby wyeliminować część
duplikacji, moglibyśmy spróbować utworzyć domyślne implementacje dla metod
`request_review` i `approve` w cesze `State`, które zwracają `self`;
jednak naruszyłoby to bezpieczeństwo obiektów, ponieważ cecha nie wie, czym dokładnie będzie
konkretne `self`. Chcemy móc używać `State` jako
obiektu cech, więc potrzebujemy, aby jego metody były bezpieczne dla obiektów.

Inne duplikacje obejmują podobne implementacje metod `request_review`
i `approve` w `Post`. Obie metody delegują do implementacji
tej samej metody na wartości w polu `state` `Option` i ustawiają nową
wartość pola `state` na wynik. Gdybyśmy mieli wiele metod w `Post`,
które podążają za tym wzorcem, moglibyśmy rozważyć zdefiniowanie makra, aby wyeliminować
powtarzanie (zobacz sekcję [„Makra”][macros]<!-- ignoruj ​​--> w rozdziale 20).

Implementując wzorzec stanu dokładnie tak, jak jest zdefiniowany dla języków
zorientowanych obiektowo, nie wykorzystujemy w pełni zalet Rusta, jak moglibyśmy.
Przyjrzyjmy się niektórym zmianom, które możemy wprowadzić w skrzynce `blog`, które mogą powodować
nieprawidłowe stany i przejścia do błędów kompilacji.

#### Encoding States and Behavior as Types

We’ll show you how to rethink the state pattern to get a different set of
trade-offs. Rather than encapsulating the states and transitions completely so
outside code has no knowledge of them, we’ll encode the states into different
types. Consequently, Rust’s type checking system will prevent attempts to use
draft posts where only published posts are allowed by issuing a compiler error.

Let’s consider the first part of `main` in Listing 18-11:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:here}}
```

</Listing>

Nadal umożliwiamy tworzenie nowych postów w stanie roboczym za pomocą `Post::new`
i możliwość dodawania tekstu do zawartości posta. Ale zamiast mieć metodę
`content` w wersji roboczej posta, która zwraca pusty ciąg, sprawimy, że
wersje robocze postów w ogóle nie będą miały metody `content`. W ten sposób, jeśli spróbujemy uzyskać
zawartość wersji roboczej posta, otrzymamy błąd kompilatora informujący nas, że metoda
nie istnieje. W rezultacie niemożliwe będzie przypadkowe
wyświetlenie zawartości wersji roboczej posta w produkcji, ponieważ ten kod nawet się nie skompiluje.
Listing 18-19 pokazuje definicję struktury `Post` i struktury `DraftPost`,
a także metody każdej z nich:

<Listing number="18-19" file-name="src/lib.rs" caption="A `Post` with a `content` method and `DraftPost` without a `content` method">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-19/src/lib.rs}}
```

</Listing>

Zarówno struktura `Post`, jak i `DraftPost` mają prywatne pole `content`, które
przechowuje tekst wpisu na blogu. Struktury nie mają już pola `state`, ponieważ
przenosimy kodowanie stanu do typów struktur. Struktura `Post`
będzie reprezentować opublikowany wpis i ma metodę `content`, która
zwraca `content`.

Nadal mamy funkcję `Post::new`, ale zamiast zwracać wystąpienie
`Post`, zwraca wystąpienie `DraftPost`. Ponieważ `content` jest prywatne
i nie ma żadnych funkcji, które zwracają `Post`, nie można teraz utworzyć wystąpienia `Post`.

Struktura `DraftPost` ma metodę `add_text`, więc możemy dodawać tekst do
`content` jak poprzednio, ale pamiętaj, że `DraftPost` nie ma zdefiniowanej metody `content`! Teraz program zapewnia, że ​​wszystkie posty zaczynają się jako wersje robocze, a wersje robocze
postów nie mają swojej zawartości dostępnej do wyświetlenia. Każda próba obejścia
tych ograniczeń spowoduje błąd kompilatora.

#### Implementing Transitions as Transformations into Different Types

Jak więc uzyskać opublikowany post? Chcemy wymusić regułę, że szkic posta musi zostać sprawdzony i zatwierdzony przed opublikowaniem. Post w stanie
oczekującym na sprawdzenie nadal nie powinien wyświetlać żadnej treści. Zaimplementujmy
te ograniczenia, dodając kolejną strukturę, `PendingReviewPost`, definiując metodę
`request_review` w `DraftPost`, aby zwrócić `PendingReviewPost` i
definiując metodę `approve` w `PendingReviewPost`, aby zwrócić `Post`, jak
pokazano w Liście 18-20:

<Listing number="18-20" file-name="src/lib.rs" caption="A `PendingReviewPost` that gets created by calling `request_review` on `DraftPost` and an `approve` method that turns a `PendingReviewPost` into a published `Post`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-20/src/lib.rs:here}}
```

</Listing>

Metody `request_review` i `approve` przejmują własność `self`, w ten sposób
konsumując instancje `DraftPost` i `PendingReviewPost` i przekształcając je
odpowiednio w `PendingReviewPost` i opublikowany `Post`. W ten sposób
nie będziemy mieć żadnych zalegających instancji `DraftPost` po wywołaniu
`request_review` na nich itd. Struktura `PendingReviewPost` nie
ma zdefiniowanej metody `content`, więc próba odczytania jej zawartości
powoduje błąd kompilatora, tak jak w przypadku `DraftPost`. Ponieważ jedynym sposobem na uzyskanie
opublikowanej instancji `Post`, która ma zdefiniowaną metodę `content`, jest wywołanie
metody `approve` w `PendingReviewPost`, a jedynym sposobem na uzyskanie
`PendingReviewPost` jest wywołanie metody `request_review` w `DraftPost`,
zakodowaliśmy teraz przepływ pracy wpisu na blogu w systemie typów.

Musimy jednak również wprowadzić kilka drobnych zmian w `main`. Metody `request_review` i
`approve` zwracają nowe instancje zamiast modyfikować strukturę, do której są wywoływane,
więc musimy dodać więcej przypisań `let post =` shadowing, aby zapisać
zwrócone instancje. Nie możemy również mieć asercji dotyczących zawartości wersji roboczej i
oczekujących recenzji wpisów jako pustych ciągów, ani ich nie potrzebujemy: nie możemy już
kompilować kodu, który próbuje używać zawartości wpisów w tych stanach.
Zaktualizowany kod w `main` pokazano na Liście 18-21:

<Listing number="18-21" file-name="src/main.rs" caption="Modifications to `main` to use the new implementation of the blog post workflow">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-21/src/main.rs}}
```

</Listing>

Zmiany, które musieliśmy wprowadzić w `main`, aby ponownie przypisać `post`, oznaczają, że ta
implementacja nie do końca podąża już za obiektowym wzorcem stanu:
przekształcenia między stanami nie są już w całości
zamknięte w implementacji `Post`. Jednak nasza korzyść polega na tym, że nieprawidłowe stany są
teraz niemożliwe ze względu na system typów i sprawdzanie typów, które odbywa się w
czasie kompilacji! Zapewnia to, że pewne błędy, takie jak wyświetlanie zawartości
nieopublikowanego posta, zostaną wykryte, zanim trafią do produkcji.

Wypróbuj zadania sugerowane na początku tej sekcji w skrzynce `blog`, tak jak jest
po Listingu 18-21, aby zobaczyć, co myślisz o projekcie tej wersji
kodu. Zauważ, że niektóre zadania mogą być już ukończone w tym
projekcie.

Widzieliśmy, że chociaż Rust jest w stanie zaimplementować obiektowe
wzorce projektowe, inne wzorce, takie jak kodowanie stanu w systemie typów,
są również dostępne w Rust. Te wzorce mają różne kompromisy. Chociaż
możesz być bardzo zaznajomiony ze wzorcami obiektowymi, ponowne przemyślenie
problemu w celu wykorzystania funkcji Rust może przynieść korzyści, takie jak
zapobieganie niektórym błędom w czasie kompilacji. Wzorce obiektowe nie zawsze będą
najlepszym rozwiązaniem w Rust ze względu na pewne funkcje, takie jak własność, których
języki obiektowe nie mają.

## Summary

Niezależnie od tego, czy uważasz Rust za język obiektowy, czy nie, po
przeczytaniu tego rozdziału wiesz już, że możesz używać obiektów cech, aby uzyskać pewne
obiektowe funkcje w Rust. Dynamiczne wysyłanie może dać Twojemu kodowi pewną
elastyczność w zamian za odrobinę wydajności w czasie wykonywania. Możesz użyć tej
elastyczności, aby zaimplementować obiektowe wzorce, które mogą pomóc w
utrzymywalności Twojego kodu. Rust ma również inne funkcje, takie jak własność, których
języki obiektowe nie mają. Obiektowy wzorzec nie zawsze
będzie najlepszym sposobem na wykorzystanie mocnych stron Rust, ale jest dostępną
opcją.

Następnie przyjrzymy się wzorcom, które są kolejną funkcją Rust, która umożliwia
dużą elastyczność. Przyjrzeliśmy się im pokrótce w całej książce, ale
jeszcze nie widzieliśmy ich pełnego potencjału. Zaczynajmy!

[more-info-than-rustc]: ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler
[macros]: ch20-06-macros.html#macros
