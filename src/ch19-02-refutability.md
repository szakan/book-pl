## Obalalność: czy wzorzec może nie pasować

Wzory występują w dwóch formach: obalalne i niepodważalne. Wzory, które będą pasować
do dowolnej możliwej przekazanej wartości, są *niepodważalne*. Przykładem może być `x` w
instrukcji `let x = 5;`, ponieważ `x` pasuje do czegokolwiek i dlatego nie może nie pasować. Wzory, które mogą nie pasować do pewnej możliwej wartości, są
*obalalne*. Przykładem może być `Some(x)` w wyrażeniu `if let Some(x) =
a_value`, ponieważ jeśli wartość w zmiennej `a_value` to `None`, a nie
`Some`, wzorzec `Some(x)` nie będzie pasował.

Parametry funkcji, instrukcje `let` i pętle `for` mogą akceptować tylko wzorce
niepodważalne, ponieważ program nie może zrobić niczego sensownego, gdy
wartości nie pasują. Wyrażenia `if let` i `while let` akceptują
wzorce obalane i niepodważalne, ale kompilator ostrzega przed
wzorcami obalanymi, ponieważ z definicji są przeznaczone do obsługi możliwych
porażek: funkcjonalność warunku polega na jego zdolności do wykonywania
inaczej w zależności od sukcesu lub porażki.

Ogólnie rzecz biorąc, nie powinieneś martwić się o rozróżnienie między wzorcami obalanymi
i niepodważalnymi; jednak musisz znać koncepcję
obalania, aby móc zareagować, gdy zobaczysz ją w komunikacie o błędzie. W
takich przypadkach będziesz musiał zmienić albo wzorzec, albo konstrukcję, z którą
używasz wzorca, w zależności od zamierzonego zachowania kodu.

Przyjrzyjmy się przykładowi tego, co się dzieje, gdy próbujemy użyć wzorca obalanego,
gdzie Rust wymaga wzorca obalanego i odwrotnie. Wypis 19-8 pokazuje
instrukcję `let`, ale dla wzorca określiliśmy `Some(x)`, wzorzec obalany. Jak można się spodziewać, ten kod nie zostanie skompilowany.

<Listing number="19-8" caption="Attempting to use a refutable pattern with `let`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-08/src/main.rs:here}}
```

</Listing>

Gdyby `some_option_value` było wartością `None`, nie pasowałoby do wzorca
`Some(x)`, co oznacza, że ​​wzorzec jest obalalny. Jednak instrukcja `let` może
tylko zaakceptować niepodważalny wzorzec, ponieważ nie ma niczego, co kod mógłby
zrobić z wartością `None`. Podczas kompilacji Rust będzie narzekał, że próbowaliśmy
użyć obalalnego wzorca, podczas gdy wymagany jest niepodważalny wzorzec:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-08/output.txt}}
```

Ponieważ nie objęliśmy (i nie mogliśmy objąć!) każdej prawidłowej wartości za pomocą
wzorca `Some(x)`, Rust słusznie generuje błąd kompilatora.

Jeśli mamy wzorzec obalalny, w którym potrzebny jest wzorzec niepodważalny, możemy
to naprawić, zmieniając kod, który używa wzorca: zamiast używać `let`, możemy
używać `if let`. Wtedy, jeśli wzorzec nie pasuje, kod po prostu pominie
kod w nawiasach klamrowych, dając mu sposób na kontynuowanie poprawnego działania. Listing
19-9 pokazuje, jak naprawić kod z Listingu 19-8.

<Listing number="19-9" caption="Using `if let` and a block with refutable patterns instead of `let`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-09/src/main.rs:here}}
```

</Listing>

Podaliśmy kodowi wyjście! Ten kod jest teraz całkowicie poprawny. Jednak,
jeśli podamy `if let` niepodważalny wzorzec (wzorzec, który zawsze będzie pasował), taki jak `x`, jak pokazano w Listingu 19-10, kompilator wyświetli
ostrzeżenie.

<Listing number="19-10" caption="Attempting to use an irrefutable pattern with `if let`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-10/src/main.rs:here}}
```

</Listing>

Rust narzeka, że ​​nie ma sensu używać `if let` z niepodważalnym
wzorcem:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-10/output.txt}}
```

Z tego powodu ramiona dopasowania muszą używać wzorców obalalnych, z wyjątkiem ostatniego
ramienia, które powinno pasować do wszystkich pozostałych wartości z niepodważalnym wzorcem. Rust
pozwala nam używać niepodważalnego wzorca w `match` z tylko jednym ramieniem, ale
ta składnia nie jest szczególnie użyteczna i można ją zastąpić prostszym
poleceniem `let`.

Teraz, gdy wiesz, gdzie używać wzorców i jaka jest różnica między wzorcami obalalnymi
i niepodważalnymi, omówmy całą składnię, której możemy użyć do tworzenia
wzorców.
