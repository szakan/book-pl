## Załącznik E - Wydania

W rozdziale 1 zobaczyłeś, że `cargo new` dodaje trochę metadanych do pliku
*Cargo.toml* na temat wydania. Ten dodatek mówi, co to oznacza!

Język Rust i kompilator mają sześciotygodniowy cykl wydań, co oznacza, że ​​użytkownicy otrzymują
stały strumień nowych funkcji. Inne języki programowania wydają większe
zmiany rzadziej; Rust wydaje mniejsze aktualizacje częściej. Po
jakimś czasie wszystkie te drobne zmiany sumują się. Ale od wydania do wydania trudno jest
spojrzeć wstecz i powiedzieć: „Wow, między Rust 1.10 a Rust 1.31, Rust
bardzo się zmienił!”

Co dwa lub trzy lata zespół Rust produkuje nową *edycję* Rust. Każda
edycja łączy funkcje, które trafiły do ​​przejrzystego pakietu z
w pełni zaktualizowaną dokumentacją i narzędziami. Nowe edycje są dostarczane jako część zwykłego
sześciotygodniowego procesu wydawania.

Edycje służą różnym celom dla różnych osób:

* Dla aktywnych użytkowników Rust, nowa edycja łączy przyrostowe zmiany w
łatwym do zrozumienia pakiecie.
* Dla osób niebędących użytkownikami, nowa edycja sygnalizuje, że nastąpiły pewne duże postępy,
które mogą sprawić, że Rust będzie wart ponownego przyjrzenia się.
* Dla tych, którzy rozwijają Rust, nowa edycja stanowi punkt zborny dla
całego projektu.

W momencie pisania tego tekstu, dostępne są trzy edycje Rust: Rust 2015, Rust
2018 i Rust 2021. Ta książka została napisana przy użyciu idiomów edycji Rust 2021.

Klucz `edition` w *Cargo.toml* wskazuje, której edycji kompilator powinien
użyć dla Twojego kodu. Jeśli klucz nie istnieje, Rust używa `2015` jako wartości edycji ze względu na kompatybilność wsteczną.

Każdy projekt może zdecydować się na inną edycję niż domyślna edycja 2015.
Edycje mogą zawierać niekompatybilne zmiany, takie jak dodanie nowego słowa kluczowego, które

powoduje konflikt z identyfikatorami w kodzie. Jednak jeśli nie zdecydujesz się na te
zmiany, Twój kod będzie nadal kompilowany, nawet gdy uaktualnisz używaną
wersję kompilatora Rust.

Wszystkie wersje kompilatora Rust obsługują dowolną edycję, która istniała przed wydaniem tego
kompilatora i mogą łączyć ze sobą skrzynki dowolnych obsługiwanych edycji. Zmiany edycji wpływają tylko na sposób, w jaki kompilator początkowo analizuje
kod. Dlatego jeśli używasz Rust 2015 i jedna z Twoich zależności używa
Rust 2018, Twój projekt zostanie skompilowany i będzie mógł używać tej zależności.
Odwrotna sytuacja, w której Twój projekt używa Rust 2018, a zależność używa
Rust 2015, również działa.

Aby było jasne: większość funkcji będzie dostępna we wszystkich edycjach. Programiści używający
dowolnej edycji Rust będą nadal dostrzegać ulepszenia w miarę pojawiania się nowych stabilnych wersji. Jednak w niektórych przypadkach, głównie gdy dodawane są nowe słowa kluczowe, niektóre nowe
funkcje mogą być dostępne tylko w późniejszych edycjach. Będziesz musiał zmienić
edycję, jeśli chcesz skorzystać z takich funkcji.

Aby uzyskać więcej szczegółów, [*Edition
Guide*](https://doc.rust-lang.org/stable/edition-guide/) to kompletna książka
o edycjach, która wymienia różnice między edycjami i wyjaśnia,
jak automatycznie uaktualnić kod do nowej edycji za pomocą `cargo fix`.
