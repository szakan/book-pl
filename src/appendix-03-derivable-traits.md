## Dodatek C: Cechy Wyprowadzalne

W różnych miejscach książki omawialiśmy atrybut `derive`, który
możesz zastosować do definicji struktury lub wyliczenia. Atrybut `derive` generuje
kod, który zaimplementuje cechę z własną domyślną implementacją w
typie, który adnotowałeś składnią `derive`.

W tym dodatku podajemy odniesienie do wszystkich cech w standardowej
bibliotece, których możesz użyć z `derive`. Każda sekcja obejmuje:

* Jakie operatory i metody wyprowadzające tę cechę umożliwią
* Co robi implementacja cechy dostarczona przez `derive`
* Co implementacja cechy oznacza dla typu
* Warunki, w których możesz lub nie możesz zaimplementować cechy
* Przykłady operacji, które wymagają cechy

Jeśli chcesz uzyskać inne zachowanie niż to zapewniane przez atrybut `derive`,
skonsultuj [dokumentację standardowej biblioteki](../std/index.html)<!-- ignore -->
dla każdej cechy, aby uzyskać szczegółowe informacje na temat ręcznego ich implementowania.

Cechy wymienione tutaj są jedynymi zdefiniowanymi przez bibliotekę standardową, które
mogą zostać zaimplementowane w Twoich typach za pomocą `derive`. Inne cechy zdefiniowane w bibliotece standardowej nie mają rozsądnego domyślnego zachowania, więc to od Ciebie zależy,
jak je zaimplementujesz w sposób, który ma sens dla tego, co próbujesz osiągnąć.

Przykładem cechy, której nie można wyprowadzić, jest `Display`, która obsługuje
formatowanie dla użytkowników końcowych. Zawsze powinieneś rozważyć odpowiedni sposób
wyświetlania typu użytkownikowi końcowemu. Jakie części typu powinien mieć możliwość zobaczenia przez użytkownika końcowego? Jakie części uzna za istotne? Jaki format danych
będzie dla niego najbardziej istotny? Kompilator Rust nie ma takiej wiedzy, więc
nie może zapewnić Ci odpowiedniego domyślnego zachowania.

Lista cech, które można wyprowadzić, podana w tym dodatku nie jest wyczerpująca:
biblioteki mogą zaimplementować `derive` dla swoich własnych cech, dzięki czemu lista cech,
których możesz użyć `derive`, jest naprawdę otwarta. Implementacja `derive`
wymaga użycia makra proceduralnego, które jest omówione w sekcji
[„Makra”][macros]<!-- ignore --> rozdziału 19.

### `Debug` for Programmer Output

Cecha `Debug` umożliwia formatowanie debugowania w ciągach formatujących, które wskazujesz, dodając `:?` w symbolach zastępczych `{}`.

Cecha `Debug` umożliwia drukowanie wystąpień typu w celach debugowania, dzięki czemu Ty i inni programiści używający Twojego typu możecie sprawdzić wystąpienie
w określonym punkcie wykonywania programu.

Cecha `Debug` jest wymagana na przykład podczas korzystania z makra `assert_eq!`.
To makro drukuje wartości wystąpień podanych jako argumenty, jeśli potwierdzenie równości się nie powiedzie, dzięki czemu programiści mogą zobaczyć, dlaczego dwa wystąpienia nie były równe.

### `PartialEq` and `Eq` for Equality Comparisons

Cecha `PartialEq` umożliwia porównywanie wystąpień typu w celu sprawdzenia równości i umożliwia użycie operatorów `==` i `!=`.

Wyprowadzenie `PartialEq` implementuje metodę `eq`. Gdy `PartialEq` jest wyprowadzane na podstawie
struktur, dwa wystąpienia są równe tylko wtedy, gdy *wszystkie* pola są równe, a wystąpienia nie są równe, jeśli jakiekolwiek pola nie są równe. Gdy jest wyprowadzane na podstawie wyliczeń,
każdy wariant jest równy sobie i nie jest równy innym wariantom.

Cecha `PartialEq` jest wymagana na przykład przy użyciu makra
`assert_eq!`, które musi być w stanie porównać dwa wystąpienia typu
pod kątem równości.

Cecha `Eq` nie ma żadnych metod. Jej celem jest zasygnalizowanie, że dla każdej wartości
adnotowanego typu wartość jest równa sobie. Cecha `Eq` może być
stosowana tylko do typów, które implementują również `PartialEq`, chociaż nie wszystkie typy, które implementują `PartialEq`, mogą implementować `Eq`. Jednym z przykładów są typy liczb zmiennoprzecinkowych: implementacja liczb zmiennoprzecinkowych stwierdza, że ​​dwa
wystąpienia wartości niebędącej liczbą (`NaN`) nie są sobie równe.

Przykładem sytuacji, w której `Eq` jest wymagane, są klucze w `HashMap<K, V>`, dzięki czemu
`HashMap<K, V>` może stwierdzić, czy dwa klucze są takie same.
### `PartialOrd` and `Ord` for Ordering Comparisons

Cecha `PartialOrd` pozwala porównywać wystąpienia typu w celu sortowania. Typ implementujący `PartialOrd` może być używany z operatorami `<`, `>`,
`<=` i `>=`. Cechę `PartialOrd` można stosować tylko do typów,
które implementują również `PartialEq`.

Wyprowadzenie `PartialOrd` implementuje metodę `partial_cmp`, która zwraca
`Option<Ordering>`, która będzie równa `None`, gdy podane wartości nie spowodują
uporządkowania. Przykładem wartości, która nie powoduje uporządkowania, mimo że
większość wartości tego typu można porównywać, jest wartość zmiennoprzecinkowa niebędąca liczbą (`NaN`). Wywołanie `partial_cmp` z dowolną liczbą zmiennoprzecinkową i wartością zmiennoprzecinkową `NaN`
spowoduje zwrócenie `None`.

Gdy wywodzi się ze struktur, `PartialOrd` porównuje dwa wystąpienia, porównując
wartość w każdym polu w kolejności, w jakiej pola pojawiają się w definicji struktury. Gdy wywodzi się ze struktur, warianty wyliczenia zadeklarowane wcześniej w
definicji wyliczenia są uważane za mniejsze niż warianty wymienione później.

Cecha `PartialOrd` jest wymagana na przykład dla metody `gen_range`
ze skrzyni `rand`, która generuje losową wartość w zakresie określonym przez
wyrażenie zakresu.

Cecha `Ord` pozwala wiedzieć, że dla dowolnych dwóch wartości adnotowanego
typu będzie istniało prawidłowe uporządkowanie. Cecha `Ord` implementuje metodę `cmp`,
która zwraca `Ordering` zamiast `Option<Ordering>`, ponieważ prawidłowe
uporządkowanie zawsze będzie możliwe. Cechę `Ord` można stosować tylko do typów,
które implementują również `PartialOrd` i `Eq` (a `Eq` wymaga `PartialEq`). Gdy
jest pochodną struktur i wyliczeń, `cmp` zachowuje się tak samo, jak pochodna
implementacja dla `partial_cmp` z `PartialOrd`.

Przykładem sytuacji, gdy `Ord` jest wymagane, jest przechowywanie wartości w `BTreeSet<T>`,
strukturze danych, która przechowuje dane na podstawie kolejności sortowania wartości.
### `Clone` and `Copy` for Duplicating Values

Cecha `Klon` pozwala jawnie utworzyć głęboką kopię wartości, a
proces duplikacji może obejmować uruchomienie dowolnego kodu i skopiowanie danych ze sterty. Więcej informacji na temat `Klonu` można znaleźć w sekcji [„Sposoby interakcji zmiennych i danych:
Klon”][ways-variables-and-data-interact-clone]<!-- ignore --> w
rozdziale 4.

Wyprowadzenie `Klonu` implementuje metodę `klon`, która po zaimplementowaniu dla
całego typu wywołuje `klon` w każdej części typu. Oznacza to, że wszystkie
pola lub wartości w typie muszą również implementować `Klon`, aby wyprowadzić `Klon`.

Przykładem sytuacji, gdy `Klon` jest wymagany, jest wywołanie metody `to_vec` na
wycinku. Wycinek nie jest właścicielem instancji typu, które zawiera, ale wektor
zwrócony z `to_vec` będzie musiał być właścicielem swoich instancji, więc `to_vec` wywołuje
`clone` dla każdego elementu. Zatem typ przechowywany w wycinku musi implementować `Clone`.

Cecha `Copy` pozwala na duplikowanie wartości poprzez kopiowanie tylko bitów przechowywanych na
stosie; nie jest wymagany żaden dowolny kod. Więcej informacji
na temat `Copy` można znaleźć w sekcji [„Stack-Only Data:
Copy”][stack-only-data-copy]<!-- ignore --> w rozdziale 4.

Cecha `Copy` nie definiuje żadnych metod, aby zapobiec programistom
przeciążaniu tych metod i naruszaniu założenia, że ​​nie jest uruchamiany żaden dowolny kod. W ten sposób wszyscy programiści mogą założyć, że kopiowanie wartości będzie
bardzo szybkie.

Możesz wyprowadzić `Copy` dla dowolnego typu, którego wszystkie części implementują `Copy`. Typ, który
implementuje `Copy` musi również implementować `Clone`, ponieważ typ, który implementuje
`Copy` ma trywialną implementację `Clone`, która wykonuje to samo zadanie co
`Copy`.

Cecha `Copy` jest rzadko wymagana; typy, które implementują `Copy` mają
dostępne optymalizacje, co oznacza, że ​​nie musisz wywoływać `clone`, co sprawia, że
kod jest bardziej zwięzły.

Wszystko, co możliwe za pomocą `Copy`, możesz również osiągnąć za pomocą `Clone`, ale
kod może być wolniejszy lub trzeba będzie użyć `clone` w niektórych miejscach.

### `Hash` for Mapping a Value to a Value of Fixed Size

Cecha `Hash` pozwala na pobranie instancji typu o dowolnym rozmiarze i
zamapowanie tej instancji na wartość o stałym rozmiarze za pomocą funkcji skrótu. Wyprowadzenie
`Hash` implementuje metodę `hash`. Wyprowadzona implementacja metody `hash`
łączy wynik wywołania `hash` w każdej z części typu,
co oznacza, że ​​wszystkie pola lub wartości muszą również implementować `Hash`, aby wyprowadzić `Hash`.

Przykładem sytuacji, gdy `Hash` jest wymagany, jest przechowywanie kluczy w `HashMap<K, V>`
w celu wydajnego przechowywania danych.

### `Default` for Default Values

Cecha `Default` pozwala na utworzenie wartości domyślnej dla typu. Pochodna
`Default` implementuje funkcję `default`. Pochodna implementacja funkcji
`default` wywołuje funkcję `default` w każdej części typu,
co oznacza, że ​​wszystkie pola lub wartości w typie muszą również implementować `Default`, aby
pochodna była `Default`.

Funkcja `Default::default` jest powszechnie używana w połączeniu ze składnią structupdate opisaną w sekcji [„Tworzenie instancji z innych instancji ze
składnią aktualizacji struktury”][tworzenie-instancji-z-innych-instancji-ze-składnią-structupdate]<!-- ignore
--> w rozdziale 5. Możesz dostosować kilka pól struktury, a następnie
ustawić i używać wartości domyślnej dla pozostałych pól, używając
`..Default::default()`.

Cecha `Default` jest wymagana, gdy używasz metody `unwrap_or_default` na przykład w instancjach
`Option<T>`. Jeśli `Option<T>` jest `None`, metoda
`unwrap_or_default` zwróci wynik `Default::default` dla typu
`T` przechowywanego w `Option<T>`.

[creating-instances-from-other-instances-with-struct-update-syntax]:
ch05-01-defining-structs.html#tworzenie-instancji-z-innej-instancji-przy-użyciu-składni-zmiany-struktury
[stack-only-data-copy]:
ch04-01-what-is-ownership.html#dane-przechowywane-wyłącznie-na-stosie-copy-kopiowanie
[ways-variables-and-data-interact-clone]:
ch04-01-what-is-ownership.html#ways-variables-and-data-interact-clone
[macros]: ch19-06-macros.html#macros
