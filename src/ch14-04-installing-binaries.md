<!-- Old link, do not remove -->
<a id="installing-binaries-from-cratesio-with-cargo-install"></a>

## Installing Binaries with `cargo install`

Polecenie `cargo install` pozwala na lokalną instalację i używanie skrzyń binarnych. Nie ma ono na celu zastępowania pakietów systemowych; ma być wygodnym sposobem dla programistów Rust na instalowanie narzędzi, którymi inni podzielili się na
[crates.io](https://crates.io/)<!-- ignore -->. Należy pamiętać, że można instalować tylko pakiety, które mają cele binarne. *Cel binarny* to program uruchamialny,
który jest tworzony, jeśli skrzynia ma plik *src/main.rs* lub inny plik określony
jako plik binarny, w przeciwieństwie do celu biblioteki, który nie jest uruchamialny sam w sobie, ale
nadaje się do uwzględnienia w innych programach. Zazwyczaj skrzynie mają
informacje w pliku *README* o tym, czy skrzynia jest biblioteką, ma
cel binarny, czy jedno i drugie.

Wszystkie pliki binarne zainstalowane za pomocą `cargo install` są przechowywane w folderze *bin* katalogu głównego instalacji. Jeśli zainstalowałeś Rust za pomocą *rustup.rs* i nie masz żadnych
niestandardowych konfiguracji, ten katalog będzie *$HOME/.cargo/bin*. Upewnij się, że ten
katalog znajduje się w `$PATH`, aby móc uruchamiać programy zainstalowane za pomocą
`cargo install`.

Na przykład w rozdziale 12 wspomnieliśmy, że istnieje implementacja Rust
narzędzia `grep` o nazwie `ripgrep` do wyszukiwania plików. Aby zainstalować `ripgrep`,
możemy uruchomić następujące polecenie:

<!-- manual-regeneration
cargo install coś, czego nie masz, skopiuj odpowiednie dane wyjściowe poniżej
-->

```console
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v13.0.0
  Downloaded 1 crate (243.3 KB) in 0.88s
  Installing ripgrep v13.0.0
--snip--
   Compiling ripgrep v13.0.0
    Finished release [optimized + debuginfo] target(s) in 3m 10s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v13.0.0` (executable `rg`)
```

Przedostatni wiersz wyjścia pokazuje lokalizację i nazwę
zainstalowanego pliku binarnego, który w przypadku `ripgrep` to `rg`. Jeśli
katalog instalacyjny znajduje się w `$PATH`, jak wspomniano wcześniej, możesz
uruchomić `rg --help` i zacząć używać szybszego, bardziej zardzewiałego narzędzia do wyszukiwania plików!
