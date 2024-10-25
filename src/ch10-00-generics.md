# Generic Types, Traits, and Lifetimes

Każdy język programowania ma narzędzia do efektywnego radzenia sobie z duplikacją
koncepcji. W Rust jednym z takich narzędzi są *generyki*: abstrakcyjne zastępstwa dla
konkretnych typów lub innych właściwości. Możemy wyrazić zachowanie generyków lub
sposób, w jaki odnoszą się do innych generyków, nie wiedząc, co będzie na ich miejscu
podczas kompilacji i uruchamiania kodu.

Funkcje mogą przyjmować parametry pewnego typu generycznego, zamiast typu konkretnego,
takiego jak `i32` lub `String`, w taki sam sposób, w jaki przyjmują parametry o nieznanych
wartościach, aby uruchomić ten sam kod na wielu konkretnych wartościach. W rzeczywistości użyliśmy już
generyków w rozdziale 6 z `Option<T>`, w rozdziale 8 z `Vec<T>` i
`HashMap<K, V>` oraz w rozdziale 9 z `Result<T, E>`. W tym rozdziale
dowiesz się, jak definiować własne typy, funkcje i metody za pomocą generyków!

Najpierw przejrzymy, jak wyodrębnić funkcję, aby zmniejszyć duplikację kodu. Następnie
użyjemy tej samej techniki, aby utworzyć funkcję generyczną z dwóch funkcji, które
różnią się tylko typami parametrów. Wyjaśnimy również, jak używać
typów generycznych w definicjach struktur i wyliczeń.

Następnie nauczysz się, jak używać *cech*, aby definiować zachowanie w sposób generyczny. Możesz
łączyć cechy z typami generycznymi, aby ograniczyć typ generyczny do akceptowania
tylko tych typów, które mają określone zachowanie, w przeciwieństwie do dowolnego typu.

Na koniec omówimy *czasy życia*: różnorodne typy generyczne, które
podają kompilatorowi informacje o tym, jak odwołania są ze sobą powiązane. Czasy życia pozwalają
nam przekazać kompilatorowi wystarczające informacje o pożyczonych wartościach, aby mógł
zapewnić, że odwołania będą prawidłowe w większej liczbie sytuacji niż bez naszej
pomocy.

## Removing Duplication by Extracting a Function

Typy generyczne pozwalają nam zastąpić określone typy symbolem zastępczym, który reprezentuje
wiele typów, aby usunąć duplikację kodu. Zanim zagłębimy się w składnię typów generycznych,
przyjrzyjmy się najpierw, jak usunąć duplikację w sposób, który nie obejmuje
typów generycznych, poprzez wyodrębnienie funkcji, która zastępuje określone wartości
symbolem zastępczym, który reprezentuje wiele wartości. Następnie zastosujemy tę samą
technikę, aby wyodrębnić funkcję generyczną! Przyglądając się, jak rozpoznawać
zduplikowany kod, który można wyodrębnić do funkcji, zaczniesz rozpoznawać
zduplikowany kod, który może używać typów generycznych.

Zaczniemy od krótkiego programu z Listingu 10-1, który znajduje największą
liczbę na liście.

<Listing number="10-1" file-name="src/main.rs" caption="Finding the largest number in a list of numbers">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-01/src/main.rs:here}}
```

</Listing>

Przechowujemy listę liczb całkowitych w zmiennej `number_list` i umieszczamy odwołanie
do pierwszej liczby na liście w zmiennej o nazwie `largest`. Następnie iterujemy
po wszystkich liczbach na liście i jeśli bieżąca liczba jest większa niż
liczba zapisana w `largest`, zastępujemy odwołanie w tej zmiennej.
Jeśli jednak bieżąca liczba jest mniejsza lub równa największej liczbie widzianej
do tej pory, zmienna się nie zmienia, a kod przechodzi do następnej liczby
na liście. Po rozważeniu wszystkich liczb na liście `largest` powinno
odnosić się do największej liczby, która w tym przypadku wynosi 100.

Teraz mamy za zadanie znaleźć największą liczbę na dwóch różnych listach
liczb. Aby to zrobić, możemy zduplikować kod z Listingu 10-1 i użyć
tej samej logiki w dwóch różnych miejscach programu, jak pokazano na Listingu 10-2.

<Listing number="10-2" file-name="src/main.rs" caption="Code to find the largest number in *two* lists of numbers">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-02/src/main.rs}}
```

</Listing>

Chociaż ten kod działa, duplikowanie kodu jest żmudne i podatne na błędy. Musimy również
pamiętać o aktualizacji kodu w wielu miejscach, gdy chcemy go
zmienić.

Aby wyeliminować to duplikowanie, utworzymy abstrakcję, definiując
funkcję, która działa na dowolnej liście liczb całkowitych przekazanej jako parametr. To
rozwiązanie sprawia, że ​​nasz kod jest bardziej przejrzysty i pozwala nam wyrazić koncepcję znajdowania
największej liczby na liście w sposób abstrakcyjny.

W Liście 10-3 wyodrębniamy kod, który znajduje największą liczbę, do
funkcji o nazwie `largest`. Następnie wywołujemy funkcję, aby znaleźć największą liczbę
na dwóch listach z Listingu 10-2. Możemy również użyć tej funkcji na dowolnej innej
liście wartości `i32`, którą możemy mieć w przyszłości.

<Listing number="10-3" file-name="src/main.rs" caption="Abstracted code to find the largest number in two lists">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-03/src/main.rs:here}}
```

</Listing>

Funkcja  `largest` ma parametr o nazwie `list`, który reprezentuje dowolny
konkretny wycinek wartości `i32`, który możemy przekazać do funkcji. W rezultacie,
gdy wywołujemy funkcję, kod działa na określonych wartościach, które przekazujemy.

Podsumowując, oto kroki, które wykonaliśmy, aby zmienić kod z Listingu 10-2 na
Listing 10-3:

1. Zidentyfikuj zduplikowany kod.
1. Wyodrębnij zduplikowany kod do treści funkcji i określ
dane wejściowe i wartości zwracane tego kodu w sygnaturze funkcji.
1. Zaktualizuj dwa wystąpienia zduplikowanego kodu, aby zamiast tego wywołać funkcję.

Następnie zastosujemy te same kroki z typami generycznymi, aby zmniejszyć duplikację kodu. W taki sam sposób, w jaki treść funkcji może działać na abstrakcyjnej `liście` zamiast
określonych wartościach, typy generyczne umożliwiają kodowi działanie na typach abstrakcyjnych.

Na przykład powiedzmy, że mamy dwie funkcje: jedną, która znajduje największy element w
wycinku wartości `i32` i drugą, która znajduje największy element w wycinku wartości `char`. Jak możemy wyeliminować tę duplikację? Przekonajmy się!
