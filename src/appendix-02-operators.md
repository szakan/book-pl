## Dodatek B: Operatory i symbole

Ten dodatek zawiera słownik składni Rust, w tym operatory oraz inne symbole, które pojawiają się samodzielnie lub w kontekście ścieżek, generyków, ograniczeń cech, makr, atrybutów, komentarzy, krotek oraz nawiasów.

### Operatory

Tabela B-1 zawiera operatory w Rust, przykład ich użycia w kontekście, krótkie wyjaśnienie oraz informację, czy dany operator można przeciążać. Jeśli operator jest przeciążalny, w tabeli podano odpowiednią cechę, którą należy użyć do przeciążenia tego operatora.

<span class="caption">Table B-1: Operators</span>

| Operator | Przykład | Wyjaśnienie | Przeciążalny? |
|----------|---------|-------------|---------------|
| `!` | `ident!(...)`, `ident!{...}`, `ident![...]` | Rozszerzenie makra | |
| `!` | `!expr` | Bitowy lub logiczny dopełniacz | `Not` |
| `!=` | `expr != expr` | Porównanie różności | `PartialEq` |
| `%` | `expr % expr` | Aritmetyczny reszta | `Rem` |
| `%=` | `var %= expr` | Aritmetyczny reszta i przypisanie | `RemAssign` |
| `&` | `&expr`, `&mut expr` | Pożyczka | |
| `&` | `&type`, `&mut type`, `&'a type`, `&'a mut type` | Typ wskaźnika pożyczonego | |
| `&` | `expr & expr` | Bitowe AND | `BitAnd` |
| `&=` | `var &= expr` | Bitowe AND i przypisanie | `BitAndAssign` |
| `&&` | `expr && expr` | Krótkozasięgowe logiczne AND | |
| `*` | `expr * expr` | Aritmetyczne mnożenie | `Mul` |
| `*=` | `var *= expr` | Aritmetyczne mnożenie i przypisanie | `MulAssign` |
| `*` | `*expr` | Dereferencja | `Deref` |
| `*` | `*const type`, `*mut type` | Surowy wskaźnik | |
| `+` | `trait + trait`, `'a + trait` | Ograniczenie typu złożonego | |
| `+` | `expr + expr` | Aritmetyczne dodawanie | `Add` |
| `+=` | `var += expr` | Aritmetyczne dodawanie i przypisanie | `AddAssign` |
| `,` | `expr, expr` | Separator argumentów i elementów | |
| `-` | `- expr` | Aritmetyczne negowanie | `Neg` |
| `-` | `expr - expr` | Aritmetyczne odejmowanie | `Sub` |
| `-=` | `var -= expr` | Aritmetyczne odejmowanie i przypisanie | `SubAssign` |
| `->` | `fn(...) -> type`, <code>&vert;...&vert; -> type</code> | Typ zwracany funkcji i zamknięcia | |
| `.` | `expr.ident` | Dostęp do członka | |
| `..` | `..`, `expr..`, `..expr`, `expr..expr` | Literali zakresu lewostronnie wyłączającego | `PartialOrd` |
| `..=` | `..=expr`, `expr..=expr` | Literali zakresu lewostronnie włączającego | `PartialOrd` |
| `..` | `..expr` | Składnia aktualizacji literału struktury | |
| `..` | `variant(x, ..)`, `struct_type { x, .. }` | “Wzorzec i reszta” | |
| `...` | `expr...expr` | (Przestarzałe, użyj `..=` zamiast) W wzorcu: wzorzec zakresu włączającego | |
| `/` | `expr / expr` | Aritmetyczne dzielenie | `Div` |
| `/=` | `var /= expr` | Aritmetyczne dzielenie i przypisanie | `DivAssign` |
| `:` | `pat: type`, `ident: type` | Ograniczenia | |
| `:` | `ident: expr` | Inicjalizator pola struktury | |
| `:` | `'a: loop {...}` | Etykieta pętli | |
| `;` | `expr;` | Terminator instrukcji i elementu | |
| `;` | `[...; len]` | Część składni tablicy o stałej wielkości | |
| `<<` | `expr << expr` | Przesunięcie w lewo | `Shl` |
| `<<=` | `var <<= expr` | Przesunięcie w lewo i przypisanie | `ShlAssign` |
| `<` | `expr < expr` | Porównanie mniejsze niż | `PartialOrd` |
| `<=` | `expr <= expr` | Porównanie mniejsze lub równe | `PartialOrd` |
| `=` | `var = expr`, `ident = type` | Przypisanie/ekwiwalencja | |
| `==` | `expr == expr` | Porównanie równości | `PartialEq` |
| `=>` | `pat => expr` | Część składni ramienia dopasowania | |
| `>` | `expr > expr` | Porównanie większe niż | `PartialOrd` |
| `>=` | `expr >= expr` | Porównanie większe lub równe | `PartialOrd` |
| `>>` | `expr >> expr` | Przesunięcie w prawo | `Shr` |
| `>>=` | `var >>= expr` | Przesunięcie w prawo i przypisanie | `ShrAssign` |
| `@` | `ident @ pat` | Wiązanie wzorca | |
| `^` | `expr ^ expr` | Bitowe XOR | `BitXor` |
| `^=` | `var ^= expr` | Bitowe XOR i przypisanie | `BitXorAssign` |
| <code>&vert;</code> | <code>pat &vert; pat</code> | Alternatywy wzorca | |
| <code>&vert;</code> | <code>expr &vert; expr</code> | Bitowe OR | `BitOr` |
| <code>&vert;=</code> | <code>var &vert;= expr</code> | Bitowe OR i przypisanie | `BitOrAssign` |
| <code>&vert;&vert;</code> | <code>expr &vert;&vert; expr</code> | Krótkozasięgowe logiczne OR | |
| `?` | `expr?` | Propagacja błędów | |

### Symbole niebędące operatorami

Poniższa lista zawiera wszystkie symbole, które nie pełnią funkcji operatorów; to znaczy, nie zachowują się jak wywołanie funkcji lub metody.

Tabela B-2 pokazuje symbole, które występują samodzielnie i są ważne w różnych miejscach.

<span class="caption">Table B-2: Stand-Alone Syntax</span>

| Symbol | Wyjaśnienie |
|--------|-------------|
| `'ident` | Nazwana długość życia lub etykieta pętli |
| `...u8`, `...i32`, `...f64`, `...usize`, itd. | Literał numeryczny określonego typu |
| `"..."` | Literał łańcucha tekstowego |
| `r"..."`, `r#"..."#`, `r##"..."##`, itd. | Surowy literał łańcucha tekstowego, znaki ucieczki nie są przetwarzane |
| `b"..."` | Literał bajtowego łańcucha; tworzy tablicę bajtów zamiast łańcucha |
| `br"..."`, `br#"..."#`, `br##"..."##`, itd. | Surowy literał bajtowego łańcucha, połączenie surowego i bajtowego literału łańcucha |
| `'...'` | Literał znaku |
| `b'...'` | Literał bajtu ASCII |
| <code>&vert;...&vert; expr</code> | Zamknięcie |
| `!` | Zawsze pusty typ dolny dla funkcji divergujących |
| `_` | Wzorzec wiązania "ignorowanego"; używany również do poprawy czytelności literałów całkowitych |

Tabela B-3 pokazuje symbole, które występują w kontekście ścieżki przez hierarchię modułów do elementu.

<span class="caption">Table B-3: Path-Related Syntax</span>

| Symbol | Wyjaśnienie |
|--------|-------------|
| `ident::ident` | Ścieżka przestrzeni nazw |
| `::path` | Ścieżka względem zewnętrznego prelude, gdzie wszystkie inne crate są zrootowane (tj. wyraźnie absolutna ścieżka, w tym nazwa crate) |
| `self::path` | Ścieżka względem bieżącego modułu (tj. wyraźnie względna ścieżka). |
| `super::path` | Ścieżka względem rodzica bieżącego modułu |
| `type::ident`, `<type as trait>::ident` | Powiązane stałe, funkcje i typy |
| `<type>::...` | 	Powiązany element dla typu, który nie może być bezpośrednio nazwany (np., `<&T>::...`, `<[T]>::...`, itd.) |
| `trait::method(...)` | Rozróżnienie wywołania metody poprzez nazwę cechy, która ją definiuje |
| `type::method(...)` | Rozróżnienie wywołania metody poprzez nazwę typu, dla którego jest zdefiniowana |
| `<type as trait>::method(...)` | Rozróżnienie wywołania metody poprzez nazwę cechy i typu |

Tabela B-4 pokazuje symbole, które występują w kontekście używania generycznych parametrów typów.

<span class="caption">Table B-4: Generics</span>

| Symbol | Wyjaśnienie |
|--------|-------------|
| `path<...>` | Określa parametry dla generycznego typu w typie (np., `Vec<u8>`) |
| `path::<...>`, `method::<...>` | Określa parametry dla generycznego typu, funkcji lub metody w wyrażeniu; często nazywane turbofiszem (np., `"42".parse::<i32>()`) |
| `fn ident<...> ...` | Definiuje funkcję generyczną |
| `struct ident<...> ...` | Definiuje strukturę generyczną |
| `enum ident<...> ...` | Definiuje enumerację generyczną |
| `impl<...> ...` | Definiuje generyczną implementację |
| `for<...> type` | Granice długości życia wyższego rzędu |
| `type<ident=type>` | Typ generyczny, w którym jeden lub więcej powiązanych typów ma konkretne przypisania (np., `Iterator<Item=T>`) |

Tabela B-5 pokazuje symbole, które występują w kontekście ograniczania generycznych parametrów typów z ograniczeniami traitów.

<span class="caption">Table B-5: Trait Bound Constraints</span>

| Symbol | Wyjaśnienie |
|--------|-------------|
| `T: U` | Parametr generyczny `T` ograniczony do typów, które implementują `U` |
| `T: 'a` | Typ generyczny `T` musi przetrwać długość życia `'a` (co oznacza, że typ nie może transytywnie zawierać żadnych referencji o krótszych długościach życia niż `'a`) |
| `T: 'static` | Typ generyczny `T` nie zawiera pożyczonych referencji, z wyjątkiem referencji `'static` |
| `'b: 'a` | Generyczna długość życia `'b` musi przetrwać długość życia `'a` |
| `T: ?Sized` | Zezwól parametrowi typu generycznego na bycie typem o dynamicznej wielkości |
| `'a + trait`, `trait + trait` | Złożone ograniczenie typu |

Tabela B-6 pokazuje symbole, które występują w kontekście wywoływania lub definiowania makr oraz określania atrybutów dla elementu..

<span class="caption">Tabela B-6: Makra i Atrybuty</span>

| Symbol | Wyjaśnienie |
|--------|-------------|
| `#[meta]` | Atrybut zewnętrzny |
| `#![meta]` | Atrybut wewnętrzny |
| `$ident` | Podstawienie makra |
| `$ident:kind` | Przechwytywanie makra |
| `$(…)…` | Powtarzanie makra |
| `ident!(...)`, `ident!{...}`, `ident![...]` | Wywołanie makra |

Tabela B-7 pokazuje symbole, które tworzą komentarze.

<span class="caption">Tabela B-7: Komentarze</span>

| Symbol | Wyjaśnienie |
|--------|-------------|
| `//` | Komentarz w linii |
| `//!` | Wewnętrzny komentarz dokumentacyjny w linii |
| `///` | Zewnętrzny komentarz dokumentacyjny w linii |
| `/*...*/` | Komentarz blokowy |
| `/*!...*/` | Wewnętrzny komentarz dokumentacyjny blokowy |
| `/**...*/` | Zewnętrzny komentarz dokumentacyjny blokowy |

Tabela B-8: Symbole w kontekście używania krotek

<span class="caption">Tabela B-8: Symbole w kontekście używania krotek</span>

| Symbol | Wyjaśnienie |
|--------|-------------|
| `()` | Pusta krotka (znana również jako typ jednostkowy), zarówno jako literał, jak i typ |
| `(expr)` | Wyrażenie w nawiasach |
| `(expr,)` | Wyrażenie krotki z jednym elementem |
| `(type,)` | 	Typ krotki z jednym elementem |
| `(expr, ...)` | Wyrażenie krotki |
| `(type, ...)` | Typ krotki |
| `expr(expr, ...)` | Wyrażenie wywołania funkcji; także używane do inicjalizacji krotek `struct` i wariantów `enum` krotek |
| `expr.0`, `expr.1`, itd. | Indeksowanie krotek |

Tabela B-9 pokazuje konteksty, w których używane są nawiasy klamrowe.

<span class="caption">Tabela B-9: Konteksty użycia nawiasów klamrowych</span>

| Context | Wyjaśnienie |
|---------|-------------|
| `{...}` | Wyrażenie blokowe |
| `Type {...}` | Literał `struct` |

Tabela B-10 pokazuje konteksty, w których używane są nawiasy kwadratowe.

<span class="caption">Tabela B-10: Konteksty użycia nawiasów kwadratowych</span>

| Context | Wyjaśnienie |
|---------|-------------|
| `[...]` | Literał tablicy |
| `[expr; len]` | Literał tablicy zawierający `len` kopii `expr` |
| `[type; len]` | Typ tablicy zawierający `len` instancji `type` |
| `expr[expr]` | Indeksowanie kolekcji. Możliwe do przeciążenia  (`Index`, `IndexMut`) |
| `expr[..]`, `expr[a..]`, `expr[..b]`, `expr[a..b]` | Indeksowanie kolekcji udające wycinanie kolekcji, używające `Range`, `RangeFrom`, `RangeTo`, lub `RangeFull`  jako “index” |
