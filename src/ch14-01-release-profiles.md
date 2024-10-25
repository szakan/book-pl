## Dostosowywanie kompilacji za pomocą profili wydań

W Rust, *profile wydania* to wstępnie zdefiniowane i konfigurowalne profile z
różnymi konfiguracjami, które pozwalają programistom mieć większą kontrolę nad
różnymi opcjami kompilacji kodu. Każdy profil jest konfigurowany niezależnie od
pozostałych.

Cargo ma dwa główne profile: profil `dev`, którego Cargo używa, gdy uruchamiasz `cargo
build`  i profil `release`, którego Cargo używa, gdy uruchamiasz `cargo build
--release`. Profil `dev` jest zdefiniowany z dobrymi domyślnymi ustawieniami dla rozwoju,
a profil `release` ma dobre domyślne ustawienia dla kompilacji wydania.

Te nazwy profili mogą być znane z wyników kompilacji:

<!-- manual-regeneration
anywhere, run:
cargo build
cargo build --release
and ensure output below is accurate
-->

```console
$ cargo build
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
$ cargo build --release
    Finished release [optimized] target(s) in 0.0s
```

`dev` i `release` to różne profile używane przez kompilator.

Cargo ma domyślne ustawienia dla każdego z profili, które mają zastosowanie, gdy nie
dodasz jawnie żadnych sekcji `[profile.*]` w pliku *Cargo.toml* projektu.
Dodając sekcje `[profile.*]` dla dowolnego profilu, który chcesz dostosować,
zastępujesz dowolny podzbiór ustawień domyślnych. Na przykład, oto domyślne
wartości dla ustawienia `opt-level` dla profili `dev` i `release`:

<span class="filename">Filename: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

Ustawienie `opt-level` kontroluje liczbę optymalizacji, które Rust zastosuje do
Twojego kodu, w zakresie od 0 do 3. Stosowanie większej liczby optymalizacji wydłuża
czas kompilacji, więc jeśli jesteś w fazie rozwoju i często kompilujesz swój kod,
będziesz potrzebował mniej optymalizacji, aby kompilować szybciej, nawet jeśli wynikowy kod
działa wolniej. Domyślny `opt-level` dla `dev` wynosi zatem `0`. Kiedy będziesz
gotowy do wydania kodu, najlepiej poświęcić więcej czasu na kompilację. Skompilujesz
w trybie wydania tylko raz, ale skompilowany program uruchomisz wiele razy,
więc tryb wydania zamienia dłuższy czas kompilacji na kod, który działa szybciej. Dlatego domyślny `opt-level` dla profilu `release` wynosi `3`.

Możesz zastąpić domyślne ustawienie, dodając inną wartość w
*Cargo.toml*. Na przykład, jeśli chcemy użyć poziomu optymalizacji 1 w
profilu rozwoju, możemy dodać te dwie linie do pliku *Cargo.toml*
naszego projektu:

<span class="filename">Filename: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 1
```

Ten kod zastępuje domyślne ustawienie `0`. Teraz, gdy uruchamiamy `cargo build`,
Cargo użyje domyślnych ustawień dla profilu `dev` plus naszego dostosowania do
`opt-level`. Ponieważ ustawiliśmy `opt-level` na `1`, Cargo zastosuje więcej
optymalizacji niż domyślne, ale nie tak wiele, jak w wersji release build.

Aby uzyskać pełną listę opcji konfiguracji i domyślnych ustawień dla każdego profilu, zobacz
[Cargo’s documentation](https://doc.rust-lang.org/cargo/reference/profiles.html).
