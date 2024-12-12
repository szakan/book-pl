## Publikowanie Crate w Crates.io

Użyliśmy pakietów z [crates.io](https://crates.io/)<!-- ignore --> jako
zależności naszego projektu, ale możesz również udostępniać swój kod innym osobom,
publikując własne pakiety. Rejestr skrzynek w
[crates.io](https://crates.io/)<!-- ignore --> dystrybuuje kod źródłowy
twoich pakietów, więc głównie hostuje kod, który jest open source.

Rust i Cargo mają funkcje, które ułatwiają ludziom znalezienie i używanie opublikowanego pakietu. Omówimy niektóre z tych funkcji, a następnie wyjaśnimy,
jak opublikować pakiet.

### Tworzenie przydatnej dokumentacji Komentarze

Dokładne dokumentowanie pakietów pomoże innym użytkownikom dowiedzieć się, jak i kiedy ich
używają, więc warto poświęcić czas na pisanie dokumentacji. W rozdziale
3 omówiliśmy, jak komentować kod Rust za pomocą dwóch ukośników, `//`. Rust ma również
szczególny rodzaj komentarza do dokumentacji, znany wygodnie jako
*komentarz dokumentacji*, który generuje dokumentację HTML. HTML
wyświetla zawartość komentarzy dokumentacji dla publicznych elementów API przeznaczonych
dla programistów zainteresowanych wiedzą, jak *używa* Twojej skrzyni, w przeciwieństwie do tego, jak
Twoja skrzynia jest *implementowana*.

Komentarze dokumentacji używają trzech ukośników, `///`, zamiast dwóch i obsługują
notację Markdown do formatowania tekstu. Umieść komentarze dokumentacji tuż
przed dokumentowanym elementem. Listing 14-1 pokazuje komentarze dokumentacji
dla funkcji `add_one` w skrzyni o nazwie `my_crate`.

<Listing number="14-1" file-name="src/lib.rs" caption="A documentation comment for a function">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-01/src/lib.rs}}
```

</Listing>

Tutaj podajemy opis działania funkcji `add_one`, rozpoczynamy
sekcję nagłówkiem `Examples`, a następnie podajemy kod, który pokazuje,
jak używać funkcji `add_one`. Możemy wygenerować dokumentację HTML z
tego komentarza dokumentacji, uruchamiając `cargo doc`. To polecenie uruchamia narzędzie
`rustdoc` dystrybuowane z Rust i umieszcza wygenerowaną dokumentację HTML
w katalogu *target/doc*.

Dla wygody uruchomienie `cargo doc --open` spowoduje utworzenie kodu HTML dla
dokumentacji bieżącej skrzyni (a także dokumentacji dla wszystkich
zależności skrzyni) i otwarcie wyniku w przeglądarce internetowej. Przejdź do funkcji
`add_one`, a zobaczysz, jak renderowany jest tekst w komentarzach dokumentacji,
jak pokazano na rysunku 14-1:

<img alt="Rendered HTML documentation for the `add_one` function of `my_crate`" src="img/trpl14-01.png" class="center" />

<span class="caption">Figure 14-1: HTML documentation for the `add_one`
function</span>

#### Często używane sekcje

Użyliśmy nagłówka Markdown `# Examples` w Listingu 14-1, aby utworzyć sekcję
w HTML z tytułem „Examples”. Oto kilka innych sekcji, których twórcy
często używają w swojej dokumentacji:

* **Paniki**: Scenariusze, w których dokumentowana funkcja może
wpaść w panikę. Wywołujący funkcję, którzy nie chcą, aby ich programy wpadały w panikę, powinni
upewnić się, że nie wywołują funkcji w takich sytuacjach.

* **Błędy**: Jeśli funkcja zwraca `Result`, opisanie rodzajów
błędów, które mogą wystąpić i warunków, które mogą spowodować zwrócenie tych błędów, może być pomocne dla wywołujących, aby mogli napisać kod obsługujący
różne rodzaje błędów na różne sposoby.

* **Bezpieczeństwo**: Jeśli wywołanie funkcji jest `unsafe` (omawiamy niebezpieczeństwo w
rozdziale 20), powinna być sekcja wyjaśniająca, dlaczego funkcja jest niebezpieczna,
i obejmująca niezmienniki, których funkcja oczekuje od wywołujących.

Większość komentarzy do dokumentacji nie wymaga wszystkich tych sekcji, ale jest to
dobra lista kontrolna, która przypomni Ci o aspektach Twojego kodu, którymi użytkownicy będą chcieli się
zainteresować.

#### Dokumentacja Komentarze jako testy

Dodanie przykładowych bloków kodu w komentarzach do dokumentacji może pomóc zademonstrować,
jak korzystać z biblioteki, a robienie tego ma dodatkową zaletę: uruchomienie `cargo
test` spowoduje uruchomienie przykładów kodu w dokumentacji jako testów! Nic nie jest
lepsze niż dokumentacja z przykładami. Ale nic nie jest gorsze niż przykłady,
które nie działają, ponieważ kod zmienił się od czasu napisania dokumentacji. Jeśli uruchomimy `cargo test` z dokumentacją dla funkcji `add_one`
z Listingu 14-1, zobaczymy sekcję w wynikach testu taką jak ta:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-01/
cargo test
copy just the doc-tests section below
-->

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
```

Teraz, jeśli zmienimy funkcję lub przykład tak, że `assert_eq!` w
example panikuje i ponownie uruchomimy `cargo test`, zobaczymy, że testy dokumentacji wykryją, że
przykład i kod nie są ze sobą zsynchronizowane!

#### Komentowanie zawartych elementów

Styl komentarza doc `//!` dodaje dokumentację do elementu zawierającego
komentarze, a nie do elementów następujących po komentarzach. Zazwyczaj używamy
tych komentarzy doc wewnątrz pliku głównego skrzynki (zgodnie z konwencją *src/lib.rs*) lub
wewnątrz modułu, aby udokumentować skrzynkę lub moduł jako całość.

Na przykład, aby dodać dokumentację opisującą cel skrzynki `my_crate`,
która zawiera funkcję `add_one`, dodajemy komentarze do dokumentacji,
które zaczynają się od `//!` na początku pliku *src/lib.rs*, jak pokazano w Listingu
14-2:

<Listing number="14-2" file-name="src/lib.rs" caption="Documentation for the `my_crate` crate as a whole">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-02/src/lib.rs:here}}
```

</Listing>

Zauważ, że nie ma żadnego kodu po ostatnim wierszu zaczynającym się od `//!`. Ponieważ
zaczęliśmy komentarze od `//!` zamiast `///`, dokumentujemy element,
który zawiera ten komentarz, a nie element, który następuje po tym komentarzu. W
tym przypadku tym elementem jest plik *src/lib.rs*, który jest korzeniem skrzyni. Te
komentarze opisują całą skrzynię.

Gdy uruchomimy `cargo doc --open`, te komentarze zostaną wyświetlone na stronie głównej dokumentacji `my_crate` nad listą publicznych elementów w
skrzyni, jak pokazano na rysunku 14-2:

<img alt="Rendered HTML documentation with a comment for the crate as a whole" src="img/trpl14-02.png" class="center" />

<span class="caption">Rysunek 14-2: Wyrenderowana dokumentacja dla `my_crate`,
w tym komentarz opisujący skrzynkę jako całość</span>

Komentarze do dokumentacji w elementach są przydatne do opisywania skrzyń i
modułów, zwłaszcza. Używaj ich, aby wyjaśnić ogólny cel kontenera, aby
pomóc użytkownikom zrozumieć organizację skrzyni.

### Eksportowanie wygodnego publicznego interfejsu API za pomocą `pub use`

Struktura Twojego publicznego API jest ważnym czynnikiem przy publikowaniu
skrzyni. Osoby korzystające z Twojej skrzyni są mniej zaznajomione ze strukturą niż Ty i mogą mieć trudności ze znalezieniem części, których chcą użyć, jeśli Twoja skrzynia
ma dużą hierarchię modułów.

W rozdziale 7 omówiliśmy, jak upublicznić elementy za pomocą słowa kluczowego `pub` i
wprowadzić elementy do zakresu za pomocą słowa kluczowego `use`. Jednak struktura, która
ma dla Ciebie sens podczas tworzenia skrzyni, może nie być zbyt wygodna
dla Twoich użytkowników. Możesz chcieć zorganizować swoje struktury w hierarchii,
zawierającej wiele poziomów, ale wówczas osoby, które chcą użyć typu,
który zdefiniowałeś głęboko w hierarchii, mogą mieć problem ze znalezieniem, że ten typ istnieje.
Mogą też być zirytowane koniecznością wprowadzania `use`
`my_crate::some_module::another_module::UsefulType;` zamiast `use`
`my_crate::UsefulType;`.

Dobra wiadomość jest taka, że ​​jeśli struktura *nie* jest wygodna dla innych do użycia
z innej biblioteki, nie musisz zmieniać swojej wewnętrznej organizacji:
zamiast tego możesz ponownie wyeksportować elementy, aby utworzyć publiczną strukturę, która różni się
od Twojej prywatnej struktury, używając `pub use`. Ponowny eksport bierze publiczny
element w jednej lokalizacji i sprawia, że ​​jest on publiczny w innej lokalizacji, tak jakby był
zdefiniowany w innej lokalizacji.

Na przykład powiedzmy, że utworzyliśmy bibliotekę o nazwie `art` do modelowania koncepcji artystycznych.
W tej bibliotece znajdują się dwa moduły: moduł `kinds` zawierający dwa wyliczenia
o nazwach `PrimaryColor` i `SecondaryColor` oraz moduł `utils` zawierający
funkcję o nazwie `mix`, jak pokazano na Liście 14-3:

<Listing number="14-3" file-name="src/lib.rs" caption="An `art` library with items organized into `kinds` and `utils` modules">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-03/src/lib.rs:here}}
```

</Listing>

Rysunek 14-3 pokazuje, jak wyglądałaby pierwsza strona dokumentacji tej skrzyni wygenerowanej przez `cargo doc`:

<img alt="Rendered documentation for the `art` crate that lists the `kinds` and `utils` modules" src="img/trpl14-03.png" class="center" />

<span class="caption">Rysunek 14-3: Strona główna dokumentacji `art`, która zawiera listę modułów `kinds` i `utils`</span>

Należy zauważyć, że typy `PrimaryColor` i `SecondaryColor` nie są wymienione na
stronie głównej, podobnie jak funkcja `mix`. Musimy kliknąć `kinds` i `utils`, aby je
zobaczyć.

Inna skrzynia zależna od tej biblioteki wymagałaby poleceń `use`, które
przenoszą elementy z `art` do zakresu, określając strukturę modułu, która jest
obecnie zdefiniowana. Listing 14-4 pokazuje przykład skrzyni, która używa elementów
`PrimaryColor` i `mix` ze skrzyni `art`:

<Listing number="14-4" file-name="src/main.rs" caption="A crate using the `art` crate’s items with its internal structure exported">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-04/src/main.rs}}
```

</Listing>

Autor kodu w Listingu 14-4, który używa skrzyni `art`, musiał
wymyślić, że `PrimaryColor` znajduje się w module `kinds`, a `mix` w module
`utils`. Struktura modułu skrzyni `art` jest bardziej istotna dla
programistów pracujących nad skrzynią `art` niż dla tych, którzy jej używają. Wewnętrzna
struktura nie zawiera żadnych przydatnych informacji dla kogoś, kto próbuje
zrozumieć, jak używać skrzyni `art`, ale raczej powoduje zamieszanie, ponieważ
programiści, którzy jej używają, muszą dowiedzieć się, gdzie szukać, i muszą określić
nazwy modułów w poleceniach `use`.

Aby usunąć wewnętrzną organizację z publicznego API, możemy zmodyfikować kod
skrzyni `art` w Listingu 14-3, aby dodać polecenia `pub use` w celu ponownego wyeksportowania
elementów na najwyższym poziomie, jak pokazano w Listingu 14-5:

<Listing number="14-5" file-name="src/lib.rs" caption="Adding `pub use` statements to re-export items">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-05/src/lib.rs:here}}
```

</Listing>

Dokumentacja API generowana przez `cargo doc` dla tej skrzyni będzie teraz zawierać listę
i linki do reeksportów na stronie głównej, jak pokazano na rysunku 14-4, dzięki czemu typy
`PrimaryColor` i `SecondaryColor` oraz funkcję `mix` będzie można łatwiej znaleźć.

<img alt="Rendered documentation for the `art` crate with the re-exports on the front page" src="img/trpl14-04.png" class="center" />

<span class="caption">Rysunek 14-4: Strona główna dokumentacji dla `art`, która zawiera listę reeksportów</span>

Użytkownicy skrzynki `art` mogą nadal widzieć i używać wewnętrznej struktury z Listingu
14-3, jak pokazano na Listingu 14-4, lub mogą używać wygodniejszej struktury z Listingu 14-5, jak pokazano na Listingu 14-6:

<Listing number="14-6" file-name="src/main.rs" caption="A program using the re-exported items from the `art` crate">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-06/src/main.rs:here}}
```

</Listing>

W przypadkach, gdy istnieje wiele zagnieżdżonych modułów, ponowne eksportowanie typów na najwyższym
poziomie za pomocą `pub use` może znacząco wpłynąć na doświadczenia
osób korzystających z danej skrzyni. Innym powszechnym zastosowaniem `pub use` jest ponowne eksportowanie
definicji zależności w bieżącej skrzyni, aby uczynić jej
definicje częścią publicznego API Twojej skrzyni.

Tworzenie użytecznej publicznej struktury API jest bardziej sztuką niż nauką i
możesz iterować, aby znaleźć API, które najlepiej sprawdzi się u Twoich użytkowników. Wybranie `pub
use` daje Ci elastyczność w sposobie wewnętrznej struktury skrzyni i
oddziela tę wewnętrzną strukturę od tego, co prezentujesz swoim użytkownikom. Przyjrzyj się
części kodu skrzynek, które zainstalowałeś, aby sprawdzić, czy ich wewnętrzna struktura
różni się od ich publicznego API.

### Konfigurowanie konta Crates.io

Zanim będziesz mógł opublikować jakiekolwiek skrzynki, musisz utworzyć konto na
[crates.io](https://crates.io/)<!-- ignore --> i uzyskać token API. Aby to zrobić,
odwiedź stronę główną na [crates.io](https://crates.io/)<!-- ignore --> i zaloguj się
za pomocą konta GitHub. (Konto GitHub jest obecnie wymagane, ale
strona może obsługiwać inne sposoby tworzenia konta w przyszłości.) Po
zalogowaniu przejdź do ustawień konta na
[https://crates.io/me/](https://crates.io/me/)<!-- ignore --> i pobierz
klucz API. Następnie uruchom polecenie `cargo login` i wklej swój klucz API, gdy pojawi się monit, w następujący sposób:

```console
$ cargo login
abcdefghijklmnopqrstuvwxyz012345
```

To polecenie poinformuje Cargo o Twoim tokenie API i zapisze go lokalnie w
*~/.cargo/credentials*. Należy pamiętać, że ten token jest *tajemnicą*: nie udostępniaj go
nikomu innemu. Jeśli udostępnisz go komukolwiek z jakiegokolwiek powodu, powinieneś
unieważnić go i wygenerować nowy token na [crates.io](https://crates.io/)<!-- ignoruj
-->.

### Dodawanie metadanych do nowej skrzynki

Załóżmy, że masz skrzynkę, którą chcesz opublikować. Przed opublikowaniem musisz
dodać metadane w sekcji `[package]` pliku *Cargo.toml* skrzynki.

Twoja skrzynka będzie potrzebowała unikalnej nazwy. Podczas pracy nad skrzynką lokalnie,
możesz nazwać ją, jak chcesz. Jednak nazwy skrzynek na
[crates.io](https://crates.io/)<!-- ignore --> są przydzielane według kolejności zgłoszeń. Gdy nazwa skrzynki jest zajęta, nikt inny nie może opublikować skrzynki
o tej nazwie. Przed próbą opublikowania skrzynki wyszukaj nazwę, której
chcesz użyć. Jeśli nazwa została użyta, musisz znaleźć inną nazwę i
edytować pole `name` w pliku *Cargo.toml* w sekcji `[package]`, aby
używać nowej nazwy do publikacji, w następujący sposób:

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
```

Nawet jeśli wybrałeś unikalną nazwę, po uruchomieniu `cargo publish` w celu opublikowania skrzyni w tym momencie, otrzymasz ostrzeżenie, a następnie błąd:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-01/
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
Aktualizowanie indeksu crates.io
ostrzeżenie: manifest nie zawiera opisu, licencji, pliku licencji, dokumentacji, strony głównej ani repozytorium.
Więcej informacji można znaleźć na stronie https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata.
--snip--
błąd: nie udało się opublikować w rejestrze na stronie https://crates.io

Spowodowane przez:
serwer zdalny odpowiedział błędem: brakujące lub puste pola metadanych: opis, licencja. Aby dowiedzieć się, jak przesłać metadane, zobacz stronę https://doc.rust-lang.org/cargo/reference/manifest.html
```

Ten błąd wynika z braku pewnych kluczowych informacji: opis i
licencja są wymagane, aby ludzie wiedzieli, co robi Twoja skrzynia i na jakich
warunkach mogą jej używać. W pliku *Cargo.toml* dodaj opis, który jest tylko
zdaniem lub dwoma, ponieważ pojawi się on wraz ze skrzynią w wynikach wyszukiwania. W polu
`license` musisz podać *wartość identyfikatora licencji*. [Linux
Foundation’s Software Package Data Exchange (SPDX)][spdx] zawiera listę identyfikatorów,
których możesz użyć dla tej wartości. Na przykład, aby określić, że licencjonowałeś swój pakiet
przy użyciu licencji MIT, dodaj identyfikator `MIT`:

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
license = "MIT"
```

Jeśli chcesz użyć licencji, która nie pojawia się w SPDX, musisz umieścić
tekst tej licencji w pliku, dołączyć plik do swojego projektu, a następnie
użyj `license-file`, aby określić nazwę tego pliku zamiast używać klucza
`license`.

Wskazówki dotyczące tego, która licencja jest odpowiednia dla Twojego projektu, wykraczają poza zakres
tej książki. Wiele osób w społeczności Rust licencjonuje swoje projekty w taki sam sposób jak Rust, używając podwójnej licencji `MIT LUB Apache-2.0`. Ta praktyka
pokazuje, że możesz również określić wiele identyfikatorów licencji oddzielonych
przez `LUB`, aby mieć wiele licencji dla swojego projektu.

Po dodaniu unikalnej nazwy, wersji, opisu i licencji plik
*Cargo.toml* dla projektu gotowego do publikacji może wyglądać następująco:

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"
description = "Zabawna gra, w której zgadujesz, jaki numer wybrał komputer."
license = "MIT LUB Apache-2.0"

[dependencies]
```

[Dokumentacja Cargo](https://doc.rust-lang.org/cargo/) opisuje inne
metadane, które możesz określić, aby zapewnić, że inni będą mogli łatwiej odkryć i używać twojej skrzyni.

### Publikowanie w Crates.io

Teraz, gdy utworzyłeś konto, zapisałeś swój token API, wybrałeś nazwę dla
swojej skrzyni i określiłeś wymagane metadane, możesz publikować!
Publikacja skrzyni przesyła określoną wersję do
[crates.io](https://crates.io/)<!-- ignore -->, aby inni mogli z niej korzystać.

Uważaj, ponieważ publikacja jest *trwała*. Wersji nigdy nie można
nadpisać, a kodu nie można usunąć. Jednym z głównych celów
[crates.io](https://crates.io/)<!-- ignore --> jest działanie jako trwałe archiwum
kodu, tak aby kompilacje wszystkich projektów, które zależą od skrzynek z
[crates.io](https://crates.io/)<!-- ignore --> nadal działały. Zezwolenie
na usuwanie wersji uniemożliwiłoby realizację tego celu. Jednak nie ma
żadnego limitu liczby wersji skrzynek, które możesz opublikować.

Uruchom ponownie polecenie `cargo publish`. Teraz powinno się udać:

<!-- manual-regeneration
go to some valid crate, publish a new version
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
   Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
   Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished dev [unoptimized + debuginfo] target(s) in 0.19s
   Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
```

Gratulacje! Udostępniłeś swój kod społeczności Rust i
każdy może łatwo dodać Twoją skrzynkę jako zależność swojego projektu.

### Publikowanie nowej wersji istniejącej skrzyni

Gdy wprowadzisz zmiany w swoim pakiecie i będziesz gotowy do wydania nowej wersji,
zmieniasz wartość `version` określoną w pliku *Cargo.toml* i
ponownie publikujesz. Użyj [zasad semantycznego wersjonowania][semver], aby zdecydować, jaki
jest właściwy numer następnej wersji na podstawie rodzaju wprowadzonych zmian.
Następnie uruchom `cargo publish`, aby przesłać nową wersję.

<!-- Stary link, nie usuwaj -->
<a id="removing-versions-from-cratesio-with-cargo-yank"></a>

### Wycofywanie wersji z Crates.io za pomocą `cargo yank`

Chociaż nie możesz usunąć poprzednich wersji skrzynki, możesz zapobiec dodawaniu ich przez
przyszłe projekty jako nowej zależności. Jest to przydatne, gdy
wersja skrzynki jest uszkodzona z jednego lub drugiego powodu. W takich sytuacjach Cargo
obsługuje *wyrywanie* wersji skrzynki.

Wyrywanie wersji zapobiega uzależnianiu nowych projektów od tej wersji, jednocześnie
umożliwiając kontynuowanie wszystkich istniejących projektów, które od niej zależą. Zasadniczo
wyrywanie oznacza, że ​​wszystkie projekty z *Cargo.lock* nie zostaną uszkodzone, a wszystkie przyszłe
wygenerowane pliki *Cargo.lock* nie będą używać wyrwanej wersji.

Aby wyrwać wersję skrzynki, w katalogu skrzynki, którą
wcześniej opublikowałeś, uruchom `cargo yank` i określ, którą wersję chcesz
wyrwać. Na przykład, jeśli opublikowaliśmy skrzynkę o nazwie `guessing_game` w wersji
1.0.1 i chcemy ją usunąć, w katalogu projektu dla `guessing_game` uruchomilibyśmy
:

<!-- manual-regeneration:
cargo yank carol-test --version 2.1.0
cargo yank carol-test --version 2.1.0 --undo
-->

```console
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game@1.0.1
```

Dodając `--undo` do polecenia, możesz również cofnąć szarpnięcie i pozwolić projektom na ponowne uruchomienie w zależności od wersji:

```console
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game@1.0.1
```

Yank *nie* usuwa żadnego kodu. Nie może na przykład usunąć przypadkowo przesłanych sekretów. Jeśli tak się stanie, musisz natychmiast zresetować te sekrety.
[spdx]: http://spdx.org/licenses/
[semver]: http://semver.org/
