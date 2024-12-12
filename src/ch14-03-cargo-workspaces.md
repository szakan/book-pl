## Obszary robocze Cargo

W rozdziale 12 zbudowaliśmy pakiet, który zawierał skrzynię binarną i skrzynię biblioteczną. W miarę rozwoju projektu możesz zauważyć, że skrzynia biblioteczna
staje się coraz większa i będziesz chciał podzielić pakiet na
wiele skrzyń bibliotecznych. Cargo oferuje funkcję o nazwie *workspaces*, która może
pomóc w zarządzaniu wieloma powiązanymi pakietami, które są rozwijane równolegle.

### Tworzenie obszaru roboczego

*Przestrzeń robocza* to zestaw pakietów, które współdzielą ten sam katalog *Cargo.lock* i wyjściowy. Stwórzmy projekt przy użyciu przestrzeni roboczej — użyjemy trywialnego kodu, aby
skupić się na strukturze przestrzeni roboczej. Istnieje wiele sposobów
strukturyzowania przestrzeni roboczej, więc pokażemy tylko jeden powszechny sposób. Będziemy mieć przestrzeń roboczą zawierającą plik binarny i dwie biblioteki. Plik binarny, który zapewni
główną funkcjonalność, będzie zależał od dwóch bibliotek. Jedna biblioteka zapewni
funkcję `add_one`, a druga biblioteka funkcję `add_two`.
Te trzy skrzynie będą częścią tej samej przestrzeni roboczej. Zaczniemy od utworzenia
nowego katalogu dla przestrzeni roboczej:

```console
$ mkdir add
$ cd add
```

Następnie w katalogu *add* tworzymy plik *Cargo.toml*, który
skonfiguruje cały obszar roboczy. Ten plik nie będzie miał sekcji `[package]`.
Zamiast tego rozpocznie się od sekcji `[workspace]`, która pozwoli nam dodawać
członków do obszaru roboczego, określając ścieżkę do pakietu z naszym binarnym
crate; w tym przypadku ta ścieżka to *adder*:

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-01-workspace-with-adder-crate/add/Cargo.toml}}
```

Następnie utworzymy binarną skrzynię `adder`, uruchamiając `cargo new` w katalogu
*add*:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-01-adder-crate/add
rm -rf adder
cargo new adder
copy output below
-->

```console
$ cargo new adder
     Created binary (application) `adder` package
```

W tym momencie możemy zbudować przestrzeń roboczą, uruchamiając `cargo build`. Pliki
w katalogu *add* powinny wyglądać tak:

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

Obszar roboczy ma jeden katalog *target* na najwyższym poziomie, w którym zostaną umieszczone skompilowane
artefakty; pakiet `adder` nie ma własnego
katalogu *target*. Nawet gdybyśmy uruchomili `cargo build` z katalogu
*adder*, skompilowane artefakty i tak trafiłyby do *add/target*
zamiast *add/adder/target*. Cargo strukturuje katalog *target* w
obszarze roboczym w ten sposób, ponieważ skrzynie w obszarze roboczym mają być od siebie zależne. Gdyby każda skrzynia miała własny katalog *target*, każda skrzynia musiałaby
ponownie skompilować każdą z pozostałych skrzyń w obszarze roboczym, aby umieścić artefakty
w swoim własnym katalogu *target*. Dzięki współdzieleniu jednego katalogu *target* skrzynie
mogą uniknąć niepotrzebnego przebudowywania.

### Tworzenie drugiego pakietu w obszarze roboczym

Next, let’s create another member package in the workspace and call it
`add_one`. Change the top-level *Cargo.toml* to specify the *add_one* path in
the `members` list:

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/Cargo.toml}}
```

Następnie wygeneruj nową skrzynkę biblioteczną o nazwie `add_one`:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-02-add-one/add
rm -rf add_one
cargo new add_one --lib
copy output below
-->

```console
$ cargo new add_one --lib
     Created library `add_one` package
```

Twój katalog *add* powinien teraz zawierać następujące katalogi i pliki:

```text
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

In the *add_one/src/lib.rs* file, let’s add an `add_one` function:

<span class="filename">Filename: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/add_one/src/lib.rs}}
```

Teraz możemy mieć pakiet `adder` z naszym plikiem binarnym zależnym od pakietu `add_one`, który ma naszą bibliotekę. Najpierw musimy dodać zależność ścieżki od
`add_one` do *adder/Cargo.toml*.

<span class="filename">Filename: adder/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/adder/Cargo.toml:6:7}}
```

Cargo nie zakłada, że ​​skrzynie w przestrzeni roboczej będą od siebie zależne, więc
musimy wyraźnie określić relacje zależności.

Następnie użyjmy funkcji `add_one` (ze skrzyni `add_one`) w skrzyni
`adder`. Otwórz plik *adder/src/main.rs* i dodaj wiersz `use` na górze,
aby umieścić nową skrzynię biblioteki `add_one` w zakresie. Następnie zmień funkcję `main`,
aby wywołać funkcję `add_one`, jak w Liście 14-7.

<Listing number="14-7" file-name="adder/src/main.rs" caption="Using the `add_one` library crate from the `adder` crate">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-07/add/adder/src/main.rs}}
```

</Listing>

Zbudujmy przestrzeń roboczą, uruchamiając `cargo build` w katalogu *add* najwyższego poziomu!

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.68s
```

Aby uruchomić skrzynkę binarną z katalogu *add*, możemy określić, który pakiet w obszarze roboczym chcemy uruchomić, używając argumentu `-p` i nazwy pakietu za pomocą `cargo run`:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo run -p adder
skopiuj dane wyjściowe poniżej; skrypt aktualizujący dane wyjściowe nie obsługuje prawidłowo podkatalogów w ścieżkach
-->

```console
$ cargo run -p adder
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/adder`
Witaj, świecie! 10 plus jeden to 11!
```

Spowoduje to uruchomienie kodu w pliku *adder/src/main.rs*, który zależy od pakietu `add_one`.

#### Poleganie na pakiecie zewnętrznym w obszarze roboczym

Zauważ, że obszar roboczy ma tylko jeden plik *Cargo.lock* na najwyższym poziomie,
zamiast mieć *Cargo.lock* w katalogu każdej skrzyni. Zapewnia to, że
wszystkie skrzynie używają tej samej wersji wszystkich zależności. Jeśli dodamy pakiet `rand`
do plików *adder/Cargo.toml* i *add_one/Cargo.toml*, Cargo
rozwiąże oba z nich do jednej wersji `rand` i zapisze to w jednym
*Cargo.lock*. Sprawienie, że wszystkie skrzynie w obszarze roboczym używają tych samych zależności,
oznacza, że ​​skrzynie zawsze będą ze sobą kompatybilne. Dodajmy skrzynkę
`rand` do sekcji `[dependencies]` w pliku *add_one/Cargo.toml*, abyśmy mogli użyć skrzynki `rand` w skrzynce `add_one`:

<!-- Podczas aktualizacji używanej wersji `rand` zaktualizuj również wersję
`rand` używaną w tych plikach, aby wszystkie były zgodne:
* ch02-00-guessing-game-tutorial.md
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
-->

<span class="filename">Filename: add_one/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add/add_one/Cargo.toml:6:7}}
```

Teraz możemy dodać `use rand;` do pliku *add_one/src/lib.rs*, a zbudowanie
całego obszaru roboczego poprzez uruchomienie `cargo build` w katalogu *add* spowoduje wczytanie
i skompilowanie pakietu `rand`. Otrzymamy jedno ostrzeżenie, ponieważ nie odnosimy się do `rand`, który wprowadziliśmy do zakresu:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add
cargo build
skopiuj poniższe dane wyjściowe; skrypt aktualizujący dane wyjściowe nie obsługuje prawidłowo podkatalogów w ścieżkach
-->

```console
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
   --snip--
   Compiling rand v0.8.5
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: `rand`
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

warning: `add_one` (lib) generated 1 warning
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 10.18s
```

Najwyższego poziomu *Cargo.lock* zawiera teraz informacje o zależnościach
`add_one` od `rand`. Jednak nawet jeśli `rand` jest używany gdzieś w
obszarze roboczym, nie możemy go używać w innych skrzyniach w obszarze roboczym, chyba że dodamy
`rand` również do ich plików *Cargo.toml*. Na przykład, jeśli dodamy `use rand;`
do pliku *adder/src/main.rs* dla pakietu `adder`, otrzymamy błąd:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-03-use-rand/add
cargo build
skopiuj poniższe dane wyjściowe; skrypt aktualizujący dane wyjściowe nie obsługuje prawidłowo podkatalogów w ścieżkach
-->

```console
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import `rand`
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate `rand`
```

Aby to naprawić, edytuj plik *Cargo.toml* dla pakietu `adder` i wskaż,
że `rand` jest również jego zależnością. Zbudowanie pakietu `adder` spowoduje
dodanie `rand` do listy zależności dla `adder` w *Cargo.lock*, ale nie zostaną pobrane żadne
dodatkowe kopie `rand`. Cargo zapewni, że każda
skrzynia w każdym pakiecie w obszarze roboczym używającym pakietu `rand` będzie używać
tej samej wersji, o ile określą zgodne wersje `rand`, oszczędzając
nam miejsce i zapewniając, że skrzynie w obszarze roboczym będą ze sobą zgodne.

Jeśli skrzynie w obszarze roboczym określają niezgodne wersje tej samej zależności,
Cargo rozwiąże każdą z nich, ale nadal będzie próbował rozwiązać jak najmniej wersji.

#### Dodawanie testu do obszaru roboczego

Aby wprowadzić kolejne udoskonalenie, dodajmy test funkcji `add_one::add_one` w skrzyni `add_one`:

<span class="filename">Filename: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add/add_one/src/lib.rs}}
```

Teraz uruchom `cargo test` w katalogu najwyższego poziomu *add*. Uruchomienie `cargo test` w
przestrzeni roboczej o takiej strukturze jak ta spowoduje uruchomienie testów dla wszystkich skrzyń w
przestrzeni roboczej:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test
skopiuj dane wyjściowe poniżej; skrypt aktualizujący dane wyjściowe nie obsługuje podkatalogów w
ścieżkach prawidłowo
-->

```console
$ cargo test
Kompilacja add_one v0.1.0 (file:///projects/add/add_one)
Kompilacja adder v0.1.0 (file:///projects/add/adder)
Zakończono test [niezoptymalizowany + debuginfo] cele w 0,27 s
Uruchamianie unittests src/lib.rs (target/debug/deps/add_one-f0253159197f7841)

uruchamianie 1 testu
test tests::it_works ... ok

wynik testu: ok. 1 zaliczony; 0 niezaliczony; 0 zignorowany; 0 zmierzony; 0 odfiltrowany; zakończono w 0,00 s

Uruchamianie unittests src/main.rs (target/debug/deps/adder-49979ff40686fa8e)

uruchamianie 0 testów

wynik testu: ok. 0 zaliczonych; 0 niezaliczonych; 0 zignorowanych; 0 zmierzonych; 0 odfiltrowanych; ukończone w 0,00 s

Doc-tests add_one

uruchomiono 0 testów

wynik testu: ok. 0 zaliczonych; 0 niezaliczonych; 0 zignorowanych; 0 zmierzonych; 0 odfiltrowanych; ukończone w 0,00 s
```

Pierwsza sekcja wyników pokazuje, że test `it_works` w skrzyni `add_one`
zaliczony. Następna sekcja pokazuje, że w skrzyni `adder`
nie znaleziono żadnych testów, a ostatnia sekcja pokazuje, że w skrzyni `add_one` nie znaleziono żadnych testów dokumentacji.

Możemy również uruchomić testy dla jednej konkretnej skrzyni w obszarze roboczym z
katalogu najwyższego poziomu, używając flagi `-p` i określając nazwę skrzyni,
którą chcemy przetestować:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test -p add_one
skopiuj dane wyjściowe poniżej; skrypt aktualizujący dane wyjściowe nie obsługuje prawidłowo podkatalogów w ścieżkach
-->

```console
$ cargo test -p add_one
Zakończono test [niezoptymalizowany + debuginfo] cele w ciągu 0,00 s
Uruchamianie testów jednostkowych src/lib.rs (target/debug/deps/add_one-b3235fea9a156f74)

uruchamianie 1 testu
test tests::it_works ... ok

wynik testu: ok. 1 zaliczony; 0 niezaliczony; 0 zignorowany; 0 zmierzonych; 0 odfiltrowanych; ukończone w 0,00 s

Doc-tests add_one

uruchomiono 0 testów

wynik testu: ok. 0 zaliczonych; 0 nieudanych; 0 zignorowanych; 0 zmierzonych; 0 odfiltrowanych; ukończone w 0,00 s
```

To wyjście pokazuje, że `cargo test` uruchomił tylko testy dla skrzyni `add_one` i
nie uruchomił testów skrzyni `adder`.

Jeśli opublikujesz skrzynie w obszarze roboczym w [crates.io](https://crates.io/),
każda skrzynia w obszarze roboczym będzie musiała zostać opublikowana osobno. Podobnie jak `cargo
test`, możemy opublikować konkretną skrzynię w naszym obszarze roboczym, używając flagi `-p`
i określając nazwę skrzyni, którą chcemy opublikować.

Aby poćwiczyć, dodaj skrzynię `add_two` do tego obszaru roboczego w podobny sposób, jak skrzynię `add_one`!

W miarę rozwoju projektu rozważ użycie przestrzeni roboczej: łatwiej jest zrozumieć
mniejsze, pojedyncze komponenty niż jeden duży fragment kodu. Ponadto, trzymanie
skrzyń w przestrzeni roboczej może ułatwić koordynację między skrzyniami, jeśli są one
często zmieniane w tym samym czasie.
