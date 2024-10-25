## Charakterystyka języków obiektowych

W społeczności programistów nie ma konsensusu co do tego, jakie cechy musi mieć
język, aby można go było uznać za obiektowy. Rust jest pod wpływem wielu
paradygmatów programowania, w tym OOP; na przykład w rozdziale 13 zbadaliśmy cechy
pochodzące z programowania funkcyjnego. Można argumentować, że języki OOP
mają pewne wspólne cechy, a mianowicie obiekty, hermetyzację i
dziedziczenie. Przyjrzyjmy się, co oznacza każda z tych cech i czy
Rust ją obsługuje.

### Objects Contain Data and Behavior

Książka *Design Patterns: Elements of Reusable Object-Oriented Software* autorstwa

Ericha Gammy, Richarda Helma, Ralpha Johnsona i Johna Vlissidesa (Addison-Wesley Professional, 1994), potocznie nazywana książką *The Gang of Four*, jest
katalogiem obiektowych wzorców projektowych. Definiuje OOP w następujący sposób:

> Programy obiektowe składają się z obiektów. *Obiekt* pakuje zarówno
> dane, jak i procedury, które operują na tych danych. Procedury są
> zazwyczaj nazywane *metodami* lub *operacjami*.

Zgodnie z tą definicją Rust jest obiektowy: struktury i wyliczenia mają dane,
a bloki `impl` dostarczają metod strukturom i wyliczeniom. Mimo że struktury i
wyliczenia z metodami nie są *nazywane* obiektami, zapewniają tę samą
funkcjonalność, zgodnie z definicją obiektów Gang of Four.

### Encapsulation that Hides Implementation Details

Innym aspektem powszechnie kojarzonym z OOP jest idea *enkapsulacji*,
co oznacza, że ​​szczegóły implementacji obiektu nie są dostępne dla
kodu używającego tego obiektu. Dlatego jedynym sposobem na interakcję z obiektem jest
przez jego publiczne API; kod używający obiektu nie powinien mieć możliwości sięgnięcia do wnętrza obiektu i bezpośredniej zmiany danych lub zachowania. Umożliwia to
programiście zmianę i refaktoryzację wnętrza obiektu bez konieczności
zmiany kodu, który używa obiektu.

Omówiliśmy, jak kontrolować enkapsulację w rozdziale 7: możemy użyć słowa kluczowego `pub`,
aby zdecydować, które moduły, typy, funkcje i metody w naszym kodzie
powinny być publiczne, a domyślnie wszystko inne jest prywatne. Na przykład,
możemy zdefiniować strukturę `AveragedCollection`, która ma pole zawierające wektor
wartości `i32`. Struktura może również mieć pole, które zawiera średnią
wartości w wektorze, co oznacza, że ​​średnia nie musi być obliczana
na żądanie, gdy ktoś jej potrzebuje. Innymi słowy, `AveragedCollection`
przechowa dla nas obliczoną średnią. W listingu 18-1 znajduje się definicja struktury
`AveragedCollection`:

<Listing number="18-1" file-name="src/lib.rs" caption="An `AveragedCollection` struct that maintains a list of integers and the average of the items in the collection">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-01/src/lib.rs}}
```

</Listing>

Struktura jest oznaczona jako `pub`, aby inny kod mógł jej używać, ale pola w strukturze pozostają prywatne. Jest to ważne w tym przypadku, ponieważ chcemy
zapewnić, że za każdym razem, gdy wartość zostanie dodana lub usunięta z listy, średnia
również zostanie zaktualizowana. Robimy to, implementując metody `add`, `remove` i `average`
w strukturze, jak pokazano w Listingu 18-2:

<Listing number="18-2" file-name="src/lib.rs" caption="Implementations of the public methods `add`, `remove`, and `average` on `AveragedCollection`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-02/src/lib.rs:here}}
```

</Listing>

Publiczne metody `add`, `remove` i `average` to jedyne sposoby dostępu
lub modyfikacji danych w instancji `AveragedCollection`. Gdy element jest dodawany
do `list` za pomocą metody `add` lub usuwany za pomocą metody `remove`,
implementacje każdego z nich wywołują prywatną metodę `update_average`, która obsługuje
również aktualizację pola `average`.

Pola `list` i `average` pozostają prywatne, więc nie ma możliwości, aby
zewnętrzny kod mógł bezpośrednio dodawać lub usuwać elementy do lub z pola `list`;
w przeciwnym razie pole `average` może zostać zdesynchronizowane, gdy `list`
się zmieni. Metoda `average` zwraca wartość w polu `average`,
umożliwiając zewnętrznemu kodowi odczytanie `average`, ale nie modyfikowanie go.

Ponieważ hermetyzowaliśmy szczegóły implementacji struktury
`AveragedCollection`, możemy łatwo zmieniać aspekty, takie jak struktura danych,
w przyszłości. Na przykład moglibyśmy użyć `HashSet<i32>` zamiast
`Vec<i32>` dla pola `list`. Dopóki sygnatury publicznych metod `add`,
`remove` i `average` pozostaną takie same, kod używający
`AveragedCollection` nie musiałby się zmieniać, aby się skompilować. Gdybyśmy zamiast tego uczynili
`list` publiczną, niekoniecznie by tak było: `HashSet<i32>` i
`Vec<i32>` mają różne metody dodawania i usuwania elementów, więc zewnętrzny
kod prawdopodobnie musiałby się zmienić, gdyby modyfikował `list` bezpośrednio.

Jeśli hermetyzacja jest wymaganym aspektem języka, aby można go było uznać za obiektowy, to Rust spełnia ten wymóg. Możliwość użycia `pub` lub nie dla różnych części kodu umożliwia hermetyzację szczegółów implementacji.

### Inheritance as a Type System and as Code Sharing

*Dziedziczenie* to mechanizm, dzięki któremu obiekt może dziedziczyć elementy z
definicji innego obiektu, uzyskując w ten sposób dane i zachowanie obiektu nadrzędnego
bez konieczności ponownego ich definiowania.

Jeśli język musi mieć dziedziczenie, aby być językiem obiektowym, to
Rust nim nie jest. Nie ma sposobu na zdefiniowanie struktury, która dziedziczy pola i implementacje metod struktury nadrzędnej bez użycia makra.

Jeśli jednak jesteś przyzwyczajony do dziedziczenia w swoim zestawie narzędzi programistycznych,
możesz użyć innych rozwiązań w Rust, w zależności od powodu, dla którego sięgnąłeś po
dziedziczenie.

Wybrałbyś dziedziczenie z dwóch głównych powodów. Jednym z nich jest ponowne wykorzystanie kodu:
możesz zaimplementować określone zachowanie dla jednego typu, a dziedziczenie umożliwia ponowne wykorzystanie tej implementacji dla innego typu. Można to zrobić w ograniczonym zakresie w kodzie Rust, używając domyślnych implementacji metod cech, które widziałeś w
Listingu 10-14, gdy dodaliśmy domyślną implementację metody `summarize`
do cechy `Summary`. Każdy typ implementujący cechę `Summary` miałby
dostępną metodę `summarize` bez żadnego dalszego kodu. Jest to
podobne do klasy nadrzędnej mającej implementację metody i
dziedziczącej klasy podrzędnej mającej również implementację metody. Możemy
również zastąpić domyślną implementację metody `summarize`, gdy
implementujemy cechę `Summary`, co jest podobne do klasy podrzędnej nadpisującej
implementację metody odziedziczonej z klasy nadrzędnej.

Inny powód użycia dziedziczenia dotyczy systemu typów: umożliwienie użycia
typu podrzędnego w tych samych miejscach, co typ nadrzędny. To jest również
nazywane *polimorfizmem*, co oznacza, że ​​możesz podstawiać wiele obiektów
za siebie nawzajem w czasie wykonywania, jeśli mają one pewne wspólne cechy.

> ### Polimorfizm
>
> Dla wielu osób polimorfizm jest synonimem dziedziczenia. Ale jest to
> w rzeczywistości bardziej ogólna koncepcja, która odnosi się do kodu, który może pracować z danymi
> wielu typów. W przypadku dziedziczenia te typy są zazwyczaj podklasami.
>
> Rust zamiast tego używa generyków do abstrakcji różnych możliwych typów i
> granic cech, aby narzucić ograniczenia na to, co te typy muszą zapewnić. To jest
> czasami nazywane *ograniczonym polimorfizmem parametrycznym*.

Dziedziczenie ostatnio wyszło z łask jako rozwiązanie projektowe programowania
w wielu językach programowania, ponieważ często istnieje ryzyko współdzielenia większej ilości kodu
niż jest to konieczne. Podklasy nie zawsze powinny współdzieli wszystkie cechy swojej
klasy nadrzędnej, ale będą to robić w przypadku dziedziczenia. Może to sprawić, że projekt programu
będzie mniej elastyczny. Wprowadza również możliwość wywoływania metod w
podklasach, które nie mają sensu lub powodują błędy, ponieważ metody nie
mają zastosowania do podklasy. Ponadto niektóre języki zezwalają tylko na dziedziczenie pojedyncze (co oznacza, że ​​podklasa może dziedziczyć tylko z jednej klasy), co jeszcze bardziej ogranicza elastyczność projektu programu.

Z tych powodów Rust przyjmuje inne podejście polegające na używaniu obiektów cech
zamiast dziedziczenia. Przyjrzyjmy się, w jaki sposób obiekty cech umożliwiają polimorfizm w
Rust.
