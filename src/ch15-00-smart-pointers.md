# Inteligentne wskaźniki

*Wskaźnik* to ogólna koncepcja zmiennej, która zawiera adres w
pamięci. Ten adres odnosi się do lub „wskazuje” na jakieś inne dane.
Najczęstszym
rodzajem wskaźnika w Rust jest odniesienie, o którym dowiedziałeś się w
rozdziale 4. Odniesienia są oznaczone symbolem `&` i pożyczają wartość, na którą
wskazują. Nie mają żadnych specjalnych możliwości poza odwoływaniem się do
danych i nie mają żadnego narzutu.

Z drugiej strony *Inteligentne wskaźniki* to struktury danych, które działają jak
wskaźnik, ale mają również dodatkowe metadane i możliwości. Koncepcja
inteligentnych wskaźników nie jest unikalna dla Rust: inteligentne wskaźniki powstały w C++ i istnieją
również w innych językach. Rust ma wiele inteligentnych wskaźników zdefiniowanych w
standardowej bibliotece, które zapewniają funkcjonalność wykraczającą poza tę zapewnianą przez odniesienia.
Aby zbadać ogólną koncepcję, przyjrzymy się kilku różnym przykładom
inteligentnych wskaźników, w tym typowi inteligentnego wskaźnika *zliczającego odwołania*. Ten
wskaźnik umożliwia zezwolenie danym na posiadanie wielu właścicieli poprzez śledzenie
liczby właścicieli i, gdy nie ma już właścicieli, czyszczenie danych.

Rust, ze swoją koncepcją własności i pożyczania, ma dodatkową różnicę
między odniesieniami a inteligentnymi wskaźnikami: podczas gdy odniesienia tylko pożyczają dane, w
wielu przypadkach inteligentne wskaźniki *posiadają* dane, na które wskazują.

Chociaż nie nazywaliśmy ich tak w tamtym czasie, napotkaliśmy już kilka
inteligentnych wskaźników w tej książce, w tym `String` i `Vec<T>` w rozdziale 8. Oba
te typy zaliczają się do inteligentnych wskaźników, ponieważ posiadają pewną pamięć i pozwalają
na nią manipulować. Mają również metadane i dodatkowe możliwości lub gwarancje.

Na przykład `String` przechowuje swoją pojemność jako metadane i ma dodatkową
możliwość zapewnienia, że ​​jego dane będą zawsze poprawne w formacie UTF-8.

Inteligentne wskaźniki są zwykle implementowane przy użyciu struktur. W przeciwieństwie do zwykłej
struktury, inteligentne wskaźniki implementują cechy `Deref` i `Drop`. Cecha `Deref`
pozwala instancji struktury inteligentnego wskaźnika zachowywać się jak referencja,
dzięki czemu możesz napisać kod, który będzie działał zarówno z referencjami, jak i inteligentnymi wskaźnikami.
Cecha `Drop` pozwala dostosować kod, który jest uruchamiany, gdy instancja
inteligentnego wskaźnika wykracza poza zakres. W tym rozdziale omówimy obie cechy i pokażemy, dlaczego są ważne dla inteligentnych wskaźników.

Biorąc pod uwagę, że wzorzec inteligentnego wskaźnika jest ogólnym wzorcem projektowym,
często używanym w Rust, ten rozdział nie obejmie wszystkich istniejących inteligentnych wskaźników. Wiele
bibliotek ma własne inteligentne wskaźniki, a Ty możesz nawet napisać własne. Omówimy
najczęściej spotykane inteligentne wskaźniki w bibliotece standardowej:

* `Box<T>` do przydzielania wartości na stercie
* `Rc<T>`, typ zliczający referencje, który umożliwia wielokrotne posiadanie
* `Ref<T>` i `RefMut<T>`, dostępne poprzez `RefCell<T>`, typ, który wymusza
reguły pożyczania w czasie wykonywania, a nie w czasie kompilacji

Ponadto omówimy wzorzec *wewnętrznej zmienności*, w którym niezmienny
typ udostępnia API do mutowania wartości wewnętrznej. Omówimy również
*cykle referencji*: w jaki sposób mogą one powodować wycieki pamięci i jak im zapobiegać.

Zanurzmy się!
