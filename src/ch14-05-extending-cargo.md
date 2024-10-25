## Rozszerzanie Cargo za pomocą niestandardowych poleceń

Cargo jest zaprojektowane tak, aby można je było rozszerzać o nowe podpolecenia bez konieczności
modyfikowania Cargo. Jeśli plik binarny w `$PATH` nazywa się `cargo-something`, można go
uruchomić tak, jakby był podpoleceniem Cargo, uruchamiając `cargo something`. Niestandardowe
polecenia, takie jak to, są również wyświetlane po uruchomieniu `cargo --list`. Możliwość
użycia `cargo install` do zainstalowania rozszerzeń, a następnie uruchomienia ich tak, jak
wbudowane narzędzia Cargo, to superwygodna zaleta projektu Cargo!

## Summary

Dzielenie się kodem z Cargo i [crates.io](https://crates.io/)<!-- ignore --> jest
częścią tego, co sprawia, że ​​ekosystem Rust jest przydatny do wielu różnych zadań. Standardowa biblioteka Rust jest
mała i stabilna, ale skrzynie są łatwe do udostępniania, używania i
ulepszania w innym czasie niż ten w języku. Nie krępuj się
dzielenia się kodem, który jest dla Ciebie przydatny w [crates.io](https://crates.io/)<!-- ignore
-->; prawdopodobnie będzie on przydatny również dla kogoś innego!
