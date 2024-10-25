## Odczyt pliku

Teraz dodamy funkcjonalność odczytu pliku określonego w argumencie `file_path`
. Najpierw potrzebujemy przykładowego pliku, aby go przetestować: użyjemy pliku z
niewielką ilością tekstu w wielu wierszach z kilkoma powtarzającymi się słowami. W listingu 12-3
znajduje się wiersz Emily Dickinson, który będzie działał dobrze! Utwórz plik o nazwie
*poem.txt* na poziomie głównym swojego projektu i wprowadź wiersz „I’m Nobody!
Who are you?”

<Numer oferty="12-3" nazwa pliku="poem.txt" caption="Wiersz Emily Dickinson stanowi dobry przypadek testowy.">

```text
{{#include ../listings/ch12-an-io-project/listing-12-03/poem.txt}}
```

</Listing>

Mając już tekst, edytuj plik *src/main.rs* i dodaj kod do odczytu pliku, jak pokazano na Liście 12-4.

<Listing number="12-4" file-name="src/main.rs" caption="Reading the contents of the file specified by the second argument">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/src/main.rs:here}}
```

</Listing>

Najpierw wprowadzamy odpowiednią część biblioteki standardowej za pomocą instrukcji `use`
: potrzebujemy `std::fs` do obsługi plików.

W `main`, nowa instrukcja `fs::read_to_string` pobiera `file_path`, otwiera
ten plik i zwraca wartość typu `std::io::Result<String>`, która zawiera
zawartość pliku.

Po tym ponownie dodajemy tymczasową instrukcję `println!`, która drukuje wartość
`contents` po odczytaniu pliku, dzięki czemu możemy sprawdzić, czy program działa
do tej pory.

Uruchommy ten kod z dowolnym ciągiem jako pierwszym argumentem wiersza poleceń (ponieważ
jeszcze nie zaimplementowaliśmy części przeszukiwania) i plikiem *poem.txt* jako
drugim argumentem:

```console
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/output.txt}}
```

Świetnie! Kod odczytał, a następnie wydrukował zawartość pliku. Ale kod
ma kilka wad. W tej chwili funkcja `main` ma wiele
obowiązków: ogólnie rzecz biorąc, funkcje są bardziej przejrzyste i łatwiejsze w utrzymaniu, jeśli
każda funkcja odpowiada tylko za jeden pomysł. Innym problemem jest to, że
nie radzimy sobie z błędami tak dobrze, jak moglibyśmy. Program jest nadal mały, więc te
wady nie stanowią dużego problemu, ale w miarę rozwoju programu trudniej będzie je
naprawić w czysty sposób. Dobrą praktyką jest rozpoczęcie refaktoryzacji na wczesnym etapie
rozwoju programu, ponieważ znacznie łatwiej jest refaktoryzować mniejsze ilości
kodu. Zrobimy to później.
