## Przechowywanie kluczy z powiązanymi wartościami w mapach skrótów

Ostatnią z naszych popularnych kolekcji jest *mapa haszująca*. Typ `HashMap<K, V>`
przechowuje mapowanie kluczy typu `K` na wartości typu `V` za pomocą *funkcji haszującej*, która określa sposób umieszczania tych kluczy i wartości w pamięci.
Wiele języków programowania obsługuje ten rodzaj struktury danych, ale często
używają innej nazwy, takiej jak *hash*, *map*, *object*, *hash table*,
*dictionary* lub *associative array*, żeby wymienić tylko kilka.

Mapy skrótów są przydatne, gdy chcesz wyszukać dane nie za pomocą indeksu, jak to jest możliwe w przypadku wektorów, ale za pomocą klucza, który może być dowolnego typu. Na przykład,
w grze możesz śledzić wynik każdej drużyny na mapie skrótów, w której
każdy klucz to nazwa drużyny, a wartości to wynik każdej drużyny. Podając nazwę drużyny, możesz pobrać jej wynik.

W tej sekcji omówimy podstawowe API map skrótów, ale wiele innych smaczków
ukrywa się w funkcjach zdefiniowanych w `HashMap<K, V>` przez bibliotekę standardową.
Jak zawsze, sprawdź dokumentację biblioteki standardowej, aby uzyskać więcej informacji.

### Tworzenie nowej mapy skrótów

Jednym ze sposobów utworzenia pustej mapy haszującej jest użycie `new` i dodanie elementów za pomocą
`insert`. W Liście 8-20 śledzimy wyniki dwóch drużyn, których nazwy to *Blue* i *Yellow*. Drużyna Blue zaczyna z 10 punktami, a
Yellow zaczyna z 50.

<Listing number="8-20" caption="Creating a new hash map and inserting some keys and values">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-20/src/main.rs:here}}
```

</Listing>

Należy zauważyć, że najpierw musimy `użyć` `HashMap` z części kolekcji
standardowej biblioteki. Z naszych trzech typowych kolekcji ta jest najrzadziej
używana, więc nie jest uwzględniana w funkcjach automatycznie wprowadzanych do zakresu w preludium. Mapy skrótów mają również mniejsze wsparcie ze strony
standardowej biblioteki; nie ma wbudowanego makra do ich konstruowania, na przykład.

Podobnie jak wektory, mapy skrótów przechowują swoje dane na stercie. Ta `HashMap` ma
klucze typu `String` i wartości typu `i32`. Podobnie jak wektory, mapy skrótów są
jednorodne: wszystkie klucze muszą mieć ten sam typ, a wszystkie wartości
muszą mieć ten sam typ.

### Uzyskiwanie dostępu do wartości w mapie skrótów

Możemy uzyskać wartość z mapy skrótów, podając jej klucz metodzie `get`, jak pokazano na Liście 8-21.

<Listing number="8-21" caption="Accessing the score for the Blue team stored in the hash map">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-21/src/main.rs:here}}
```

</Listing>

Tutaj `score` będzie miało wartość powiązaną z drużyną Blue, a
wynikiem będzie `10`. Metoda `get` zwraca `Option<&V>`; jeśli nie ma wartości dla tego klucza w mapie skrótów, `get` zwróci `None`. Ten program
obsługuje `Option`, wywołując `copied`, aby uzyskać `Option<i32>` zamiast
`Option<&i32>`, a następnie `unwrap_or`, aby ustawić `score` na zero, jeśli `scores` nie ma wpisu dla klucza.

Możemy iterować po każdej parze klucz-wartość w mapie skrótów w podobny sposób, jak robimy to z wektorami, używając pętli `for`:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-03-iterate-over-hashmap/src/main.rs:here}}
```

Ten kod wydrukuje każdą parę w dowolnej kolejności:

```text
Yellow: 50
Blue: 10
```

### Mapy skrótów i własność

W przypadku typów implementujących cechę `Copy`, takich jak `i32`, wartości są kopiowane
do mapy skrótów. W przypadku posiadanych wartości, takich jak `String`, wartości zostaną przeniesione, a
mapa skrótów będzie właścicielem tych wartości, jak pokazano w Listingu 8-22.

<Listing number="8-22" caption="Showing that keys and values are owned by the hash map once they’re inserted">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-22/src/main.rs:here}}
```

</Listing>

Nie możemy używać zmiennych `field_name` i `field_value` po tym, jak zostały przeniesione do mapy skrótów za pomocą wywołania `insert`.

Jeśli wstawimy odwołania do wartości do mapy skrótów, wartości nie zostaną przeniesione
do mapy skrótów. Wartości, do których wskazują odwołania, muszą być prawidłowe
przynajmniej tak długo, jak długo mapa skrótów jest prawidłowa. Więcej na temat tych problemów omówimy w sekcji
[“Sprawdzanie referencji za pomocą
Czasów życia”][validating-references-with-lifetimes]<!-- ignore --> w
rozdziale 10.
### Aktualizowanie mapy skrótów

Chociaż liczba par klucz-wartość może się zwiększać, każdy unikatowy klucz może mieć
tylko jedną wartość skojarzoną z nim na raz (ale nie odwrotnie: na przykład zarówno drużyna Niebieskich, jak i Żółtych mogą mieć wartość `10`
przechowywaną w mapie skrótów `scores`).

Gdy chcesz zmienić dane w mapie skrótów, musisz zdecydować, jak
obsłużyć przypadek, gdy klucz ma już przypisaną wartość. Możesz zastąpić
starą wartość nową, całkowicie ignorując starą wartość. Możesz
zachować starą wartość i zignorować nową wartość, dodając nową wartość tylko wtedy, gdy
klucz *nie* ma już wartości. Możesz też połączyć starą wartość i
nową wartość. Przyjrzyjmy się, jak wykonać każdą z tych czynności!

#### Nadpisywanie wartości

Jeśli wstawimy klucz i wartość do mapy skrótów, a następnie wstawimy ten sam klucz
z inną wartością, wartość powiązana z tym kluczem zostanie zastąpiona.
Mimo że kod w Liście 8-23 wywołuje `insert` dwa razy, mapa skrótów
będzie zawierać tylko jedną parę klucz-wartość, ponieważ wstawiamy wartość dla klucza Niebieskiego zespołu za każdym razem.

<Listing number="8-23" caption="Replacing a value stored with a particular key">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-23/src/main.rs:here}}
```

</Listing>

Ten kod wydrukuje `{"Blue": 25}`. Oryginalna wartość `10` została
nadpisana.

<!-- Old headings. Do not remove or links may break. -->
<a id="only-inserting-a-value-if-the-key-has-no-value"></a>

#### Dodawanie klucza i wartości tylko wtedy, gdy klucz nie jest obecny

Często sprawdza się, czy konkretny klucz już istnieje w mapie skrótów
z wartością, a następnie podejmuje się następujące działania: jeśli klucz istnieje w
mapie skrótów, istniejąca wartość powinna pozostać taka, jaka jest; jeśli klucz
nie istnieje, wstaw go i jego wartość.

Mapy skrótów mają do tego specjalne API o nazwie `entry`, które przyjmuje klucz,
który chcesz sprawdzić, jako parametr. Wartością zwracaną przez metodę `entry` jest wyliczenie o nazwie `Entry`, które reprezentuje wartość, która może lub nie istnieć. Załóżmy, że
chcemy sprawdzić, czy klucz dla drużyny żółtej ma skojarzoną
z nim wartość. Jeśli nie, chcemy wstawić wartość `50` i to samo dla drużyny niebieskiej. Korzystając z API `entry`, kod wygląda jak Listing 8-24.

<Listing number="8-24" caption="Using the `entry` method to only insert if the key does not already have a value">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-24/src/main.rs:here}}
```

</Listing>

Metoda `or_insert` w `Entry` jest zdefiniowana tak, aby zwracać zmienne odwołanie do
wartości odpowiadającego klucza `Entry`, jeśli ten klucz istnieje, a jeśli nie,
wstawia parametr jako nową wartość dla tego klucza i zwraca zmienne odwołanie do nowej wartości. Ta technika jest o wiele bardziej przejrzysta niż
samodzielne pisanie logiki i dodatkowo lepiej współpracuje z programem sprawdzającym pożyczkę.

Uruchomienie kodu z Listingu 8-24 spowoduje wydrukowanie `{"Yellow": 50, "Blue": 10}`.
Pierwsze wywołanie `entry` wstawi klucz dla żółtego zespołu o wartości
`50`, ponieważ żółty zespół nie ma jeszcze wartości. Drugie wywołanie
`entry` nie zmieni mapy skrótów, ponieważ niebieski zespół ma już wartość
`10`.

#### Aktualizowanie wartości na podstawie starej wartości

Innym powszechnym przypadkiem użycia map haszujących jest wyszukiwanie wartości klucza, a następnie
aktualizowanie jej na podstawie starej wartości. Na przykład, Listing 8-25 pokazuje kod, który
liczy, ile razy każde słowo pojawia się w tekście. Używamy mapy haszującej ze
słowami jako kluczami i zwiększamy wartość, aby śledzić, ile razy
widzieliśmy to słowo. Jeśli widzimy słowo po raz pierwszy, najpierw wstawimy
wartość `0`.

<Listing number="8-25" caption="Counting occurrences of words using a hash map that stores words and counts">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-25/src/main.rs:here}}
```

</Listing>

Ten kod wydrukuje `{"world": 2, "hello": 1, "wonderful": 1}`. Możesz zobaczyć
te same pary klucz-wartość wydrukowane w innej kolejności: przypomnij sobie z sekcji
[“Uzyskiwanie dostępu do wartości w mapie skrótów”][access]<!-- ignore -->, że
iteracja po mapie skrótów odbywa się w dowolnej kolejności.

Metoda `split_whitespace` zwraca iterator po podwycinkach, oddzielonych
odstępem, wartości w `text`. Metoda `or_insert` zwraca zmienną
referencję (`&mut V`) do wartości dla określonego klucza. Tutaj przechowujemy tę
zmienną referencję w zmiennej `count`, więc aby przypisać tę wartość,
musimy najpierw dereferencjonować `count` za pomocą gwiazdki (`*`). Zmienna
referencja wykracza poza zakres na końcu pętli `for`, więc wszystkie te
zmiany są bezpieczne i dozwolone przez reguły pożyczania.

### Hashing Funkcje

Domyślnie `HashMap` używa funkcji haszującej o nazwie *SipHash*, która może zapewnić
odporność na ataki typu DoS (odmowa usługi) z udziałem
tablic haszujących [^siphash]<!-- ignore -->. Nie jest to najszybszy dostępny algorytm haszujący,
ale kompromis w postaci lepszego bezpieczeństwa, który wiąże się ze spadkiem
wydajności, jest tego wart. Jeśli profilujesz swój kod i stwierdzisz, że domyślna
funkcja haszująca jest zbyt wolna do twoich celów, możesz przełączyć się na inną funkcję,
poprzez określenie innego hashera. *Hasher* to typ, który implementuje cechę
`BuildHasher`. Omówimy cechy i sposób ich implementacji w
[Chapter 10][traits]<!-- ignore -->. Nie musisz koniecznie implementować
własnego hashera od podstaw; [crates.io](https://crates.io/)<!-- ignore -->
zawiera biblioteki udostępniane przez innych użytkowników Rust, które zapewniają hashery implementujące wiele
typowych algorytmów haszujących.

[^siphash]: [https://en.wikipedia.org/wiki/SipHash](https://en.wikipedia.org/wiki/SipHash)

## Streszczenie
Wektory, ciągi znaków i mapy skrótów zapewnią dużą ilość funkcji,
niezbędnych w programach, gdy trzeba przechowywać, uzyskiwać dostęp i modyfikować dane. Oto
kilka ćwiczeń, które powinieneś być teraz w stanie rozwiązać:

1. Mając listę liczb całkowitych, użyj wektora i zwróć medianę (po posortowaniu,
wartość na środkowej pozycji) i modę (wartość, która występuje najczęściej; mapa skrótów będzie tutaj pomocna) listy.
1. Przekształć ciągi znaków na łacinę świńską. Pierwsza spółgłoska każdego słowa jest przenoszona na
koniec słowa i dodawane jest *ay*, więc *first* staje się *irst-fay*. Słowa,
które zaczynają się od samogłoski, mają zamiast tego dodane *hay* na końcu (*apple* staje się
*apple-hay*). Pamiętaj o szczegółach dotyczących kodowania UTF-8!
1. Używając mapy skrótów i wektorów, utwórz interfejs tekstowy, aby umożliwić użytkownikowi dodawanie
nazwisk pracowników do działu w firmie; na przykład „Dodaj Sally do
Inżynierii” lub „Dodaj Amira do Sprzedaży”. Następnie pozwól użytkownikowi pobrać listę wszystkich
osób w dziale lub wszystkich osób w firmie według działu, posortowaną
alfabetycznie.

Dokumentacja standardowego API biblioteki opisuje metody, które mają wektory, ciągi znaków i mapy skrótów, które będą pomocne w tych ćwiczeniach!

Wchodzimy w bardziej złożone programy, w których operacje mogą się nie powieść, więc to
doskonały moment, aby omówić obsługę błędów. Zrobimy to później!

[validating-references-with-lifetimes]:
ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[access]: #accessing-values-in-a-hash-map
[traits]: ch10-02-traits.html
