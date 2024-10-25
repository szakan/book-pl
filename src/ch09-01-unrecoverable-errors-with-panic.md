## Nieodwracalne błędy z `panic!`

Czasami w kodzie dzieją się złe rzeczy i nie możesz nic z tym zrobić. W takich przypadkach Rust ma makro `panika!`. Istnieją dwa sposoby, aby wywołać panikę w praktyce: podejmując działanie, które powoduje panikę w naszym kodzie (takie jak
dostęp do tablicy po jej zakończeniu) lub jawnie wywołując makro `panika!`.
W obu przypadkach wywołujemy panikę w naszym programie. Domyślnie te paniki
wyświetlają komunikat o błędzie, rozwijają, czyszczą stos i kończą działanie. Za pomocą zmiennej
środowiskowej możesz również sprawić, aby Rust wyświetlał stos wywołań, gdy wystąpi panika, aby łatwiej było namierzyć źródło paniki.

> ### Unwinding the Stack or Aborting in Response to a Panic
>
> Domyślnie, gdy wystąpi panika, program zaczyna *odwijać*, co oznacza, że
> Rust cofa się w górę stosu i czyści dane z każdej napotkanej
> funkcji. Jednak cofanie się i czyszczenie wymaga dużo pracy. Rust,
> dlatego pozwala wybrać alternatywę natychmiastowego *przerwania*,
> co kończy program bez czyszczenia.
>
> Pamięć, której program używał, będzie musiała zostać wyczyszczona przez
> system operacyjny. Jeśli w swoim projekcie musisz sprawić, aby wynikowy plik binarny był
> jak najmniejszy, możesz przełączyć się z odwijania na przerywanie w przypadku paniki,
> dodając `panic = 'abort'` do odpowiednich sekcji `[profile]` w pliku
> *Cargo.toml*. Na przykład, jeśli chcesz przerwać w przypadku paniki w trybie zwolnienia,
> dodaj to:
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

Spróbujmy wywołać `panic!` w prostym programie:

<Listing file-name="src/main.rs">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-01-panic/src/main.rs}}
```

</Listing>

Po uruchomieniu programu zobaczysz coś takiego:

```console
{{#include ../listings/ch09-error-handling/no-listing-01-panic/output.txt}}
```

Wywołanie `panic!` powoduje komunikat o błędzie zawarty w dwóch ostatnich wierszach.
Pierwszy wiersz pokazuje nasz komunikat paniki i miejsce w naszym kodzie źródłowym, w którym
wystąpiła panika: *src/main.rs:2:5* wskazuje, że jest to drugi wiersz,
piąty znak naszego pliku *src/main.rs*.

W tym przypadku wskazany wiersz jest częścią naszego kodu i jeśli przejdziemy do tego
wiersza, zobaczymy wywołanie makra `panic!`. W innych przypadkach wywołanie `panic!` może
znajdować się w kodzie, który wywołuje nasz kod, a nazwa pliku i numer wiersza zgłoszone przez
komunikat o błędzie będą czyimś kodem, w którym wywoływane jest makro `panic!`,
a nie wierszem naszego kodu, który ostatecznie doprowadził do wywołania `panic!`.

<!-- Stary nagłówek. Nie usuwaj, bo linki mogą się zepsuć. -->
<a id="using-a-panic-backtrace"></a>

Możemy użyć backtrace funkcji, z których pochodziło wywołanie `panic!`, aby ustalić,
która część naszego kodu powoduje problem. Aby zrozumieć, jak używać
backtrace `panic!`, przyjrzyjmy się innemu przykładowi i zobaczmy, jak to wygląda, gdy
wywołanie `panic!` pochodzi z biblioteki z powodu błędu w naszym kodzie, zamiast
z naszego kodu wywołującego makro bezpośrednio. Listing 9-1 zawiera kod, który
próbuje uzyskać dostęp do indeksu w wektorze spoza zakresu prawidłowych indeksów.

<Listing number="9-1" file-name="src/main.rs" caption="Attempting to access an element beyond the end of a vector, which will cause a call to `panic!`">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-01/src/main.rs}}
```

</Listing>

Tutaj próbujemy uzyskać dostęp do setnego elementu naszego wektora (który ma
indeks 99, ponieważ indeksowanie zaczyna się od zera), ale wektor ma tylko trzy
elementy. W tej sytuacji Rust wpadnie w panikę. Użycie `[]` powinno zwrócić
element, ale jeśli przekażesz nieprawidłowy indeks, nie ma żadnego elementu, który Rust
mógłby tutaj zwrócić, który byłby poprawny.

W C próba odczytu poza końcem struktury danych jest niezdefiniowanym
zachowaniem. Możesz uzyskać wszystko, co znajduje się w lokalizacji w pamięci, która
odpowiadałaby temu elementowi w strukturze danych, nawet jeśli pamięć
nie należy do tej struktury. Nazywa się to *nadmiernym odczytem bufora* i może
prowadzić do luk w zabezpieczeniach, jeśli atakujący jest w stanie manipulować indeksem
w taki sposób, aby odczytać dane, których nie powinien mieć prawa odczytać, a które są przechowywane po
strukturze danych.

Aby chronić program przed tego typu lukami, jeśli spróbujesz odczytać
element o indeksie, który nie istnieje, Rust zatrzyma wykonywanie i odmówi
kontynuowania. Spróbujmy i zobaczmy:

```console
{{#include ../listings/ch09-error-handling/listing-09-01/output.txt}}
```

Ten błąd wskazuje na linię 4 naszego *main.rs*, gdzie próbujemy uzyskać dostęp do indeksu
`99` wektora w `v`.

Wiersz `note:` mówi nam, że możemy ustawić zmienną środowiskową `RUST_BACKTRACE`, aby uzyskać ślad wsteczny dokładnie tego, co spowodowało błąd.
*backtrace* to lista wszystkich funkcji, które zostały wywołane, aby dotrzeć do tego
punktu. Ślady wsteczne w Rust działają tak samo, jak w innych językach: kluczem do
odczytania śladu wstecznego jest rozpoczęcie od góry i czytanie, aż zobaczysz pliki, które
napisałeś. To jest miejsce, w którym powstał problem. Wiersze powyżej tego miejsca
to kod, który wywołał Twój kod; wiersze poniżej to kod, który wywołał Twój
kod. Te wiersze przed i po mogą obejmować podstawowy kod Rust, standardowy
kod biblioteki lub skrzynie, których używasz. Spróbujmy uzyskać ślad wsteczny,
ustawiając zmienną środowiskową `RUST_BACKTRACE` na dowolną wartość z wyjątkiem `0`.
Listing 9-2 pokazuje dane wyjściowe podobne do tego, które zobaczysz.

<!-- manual-regeneration
cd listings/ch09-error-handling/listing-09-01
RUST_BACKTRACE=1 cargo run
copy the backtrace output below
check the backtrace number mentioned in the text below the listing
-->

<Listing number="9-2" caption="The backtrace generated by a call to `panic!` displayed when the environment variable `RUST_BACKTRACE` is set">

```console
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at src/main.rs:4:6:
index  poza zakresem: len wynosi 3, ale indeks wynosi 99
stack backtrace:
   0: rust_begin_unwind
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/std/src/panicking.rs:645:5
   1: core::panicking::panic_fmt
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/core/src/panicking.rs:72:14
   2: core::panicking::panic_bounds_check
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/core/src/panicking.rs:208:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/core/src/slice/index.rs:255:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/core/src/slice/index.rs:18:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/alloc/src/vec/mod.rs:2770:9
   6: panic::main
             at ./src/main.rs:4:6
   7: core::ops::function::FnOnce::call_once
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/core/src/ops/function.rs:250:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

</Listing>

To dużo wyników! Dokładny wynik, który widzisz, może się różnić w zależności od
systemu operacyjnego i wersji Rust. Aby uzyskać ślady wsteczne z tymi
informacjami, muszą być włączone symbole debugowania. Symbole debugowania są włączone
domyślnie podczas korzystania z `cargo build` lub `cargo run` bez flagi `--release`,
jak tutaj.

W wynikach w Listingu 9-2, linia 6 backtrace wskazuje na linię w naszym
projekcie, która powoduje problem: linia 4 *src/main.rs*. Jeśli nie chcemy, aby
nasz program wpadł w panikę, powinniśmy rozpocząć nasze dochodzenie w miejscu wskazywanym
przez pierwszą linię wspominającą o napisanym przez nas pliku. W Listingu 9-1, gdzie
celowo napisaliśmy kod, który wywołałby panikę, sposobem na naprawienie paniki jest nie
żądanie elementu spoza zakresu indeksów wektora. Gdy kod
w przyszłości wpadnie w panikę, musisz ustalić, jakie działanie podejmuje kod,
jakie wartości powodują panikę i co kod powinien zrobić zamiast tego.

Wrócimy do `panic!` i kiedy powinniśmy, a kiedy nie powinniśmy używać `panic!` do
obsługi warunków błędu w sekcji [„Wpadać w `panic!` czy nie
`panic!`”][to-panic-or-not-to-panic]<!-- ignore --> w dalszej części tego
rozdziału. Następnie przyjrzymy się, jak odzyskać się po błędzie, używając `Wyniku`.

[to-panic-or-not-to-panic]:
ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic
