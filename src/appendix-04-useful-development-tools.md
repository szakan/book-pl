## Załącznik D - Przydatne narzędzia programistyczne

W tym dodatku omówimy kilka przydatnych narzędzi programistycznych, które zapewnia projekt Rust. Przyjrzymy się automatycznemu formatowaniu, szybkim sposobom stosowania poprawek ostrzeżeń, linterowi i integracji z IDE.

### Automatic Formatting with `rustfmt`

Narzędzie `rustfmt` formatuje kod zgodnie ze stylem kodu społeczności.
Wiele projektów współpracy używa `rustfmt`, aby zapobiec argumentom dotyczącym tego, którego stylu użyć podczas pisania Rust: każdy formatuje swój kod za pomocą tego narzędzia.

Aby zainstalować `rustfmt`, wprowadź następujące polecenie:

```console
$ rustup component add rustfmt
```

To polecenie daje `rustfmt` i `cargo-fmt`, podobnie jak Rust daje ci zarówno `rustc`, jak i `cargo`. Aby sformatować dowolny projekt Cargo, wprowadź następujące polecenie:

```console
$ cargo fmt
```

Uruchomienie tego polecenia zmienia format całego kodu Rust w bieżącym pakiecie. Powinno to zmienić tylko styl kodu, a nie semantykę kodu. Aby uzyskać więcej informacji na temat `rustfmt`, zobacz [jego dokumentację][rustfmt].

[rustfmt]: https://github.com/rust-lang/rustfmt

### Fix Your Code with `rustfix`

Narzędzie rustfix jest dołączone do instalacji Rust i może automatycznie naprawiać
ostrzeżenia kompilatora, które mają jasny sposób na naprawienie problemu, który prawdopodobnie
jest tym, czego chcesz. Prawdopodobnie widziałeś już ostrzeżenia kompilatora. Na przykład,
rozważ ten kod:

<span class="filename">Filename: src/main.rs</span>

```rust
fn do_something() {}

fn main() {
    for i in 0..100 {
        do_something();
    }
}
```

Tutaj wywołujemy funkcję `do_something` 100 razy, ale nigdy nie używamy zmiennej `i` w ciele pętli `for`. Rust ostrzega nas o tym:

```console
$ cargo build
   Compiling myprogram v0.1.0 (file:///projects/myprogram)
warning: unused variable: `i`
 --> src/main.rs:4:9
  |
4 |     for i in 0..100 {
  |         ^ help: consider using `_i` instead
  |
  = note: #[warn(unused_variables)] on by default

    Finished dev [unoptimized + debuginfo] target(s) in 0.50s
```

Ostrzeżenie sugeruje, że zamiast tego użyjemy `_i` jako nazwy: podkreślenie
oznacza, że ​​zamierzamy, aby ta zmienna była nieużywana. Możemy automatycznie
zastosować tę sugestię za pomocą narzędzia `rustfix`, uruchamiając polecenie `cargo
fix`:

```console
$ cargo fix
    Checking myprogram v0.1.0 (file:///projects/myprogram)
      Fixing src/main.rs (1 fix)
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s
```

Gdy ponownie przyjrzymy się *src/main.rs*, zobaczymy, że `cargo fix` zmienił kod:

<span class="filename">Filename: src/main.rs</span>

```rust
fn do_something() {}

fn main() {
    for _i in 0..100 {
        do_something();
    }
}
```

Zmienna pętli `for` nazywa się teraz `_i`, a ostrzeżenie nie pojawia się już.

Możesz również użyć polecenia `cargo fix`, aby przenieść swój kod między
różnymi edycjami Rust. Edycje są omówione w [Załączniku E][editions].
.

### More Lints with Clippy

Narzędzie Clippy to zbiór lintów do analizy kodu, dzięki czemu możesz wyłapać
typowe błędy i ulepszyć kod Rust.

Aby zainstalować Clippy, wprowadź następujące polecenie:

```console
$ rustup component add clippy
```

Aby uruchomić narzędzia Clippy’ego w dowolnym projekcie Cargo, wprowadź następujące polecenie:

```console
$ cargo clippy
```

Na przykład, powiedzmy, że piszesz program, który używa przybliżenia stałej matematycznej, takiej jak pi, tak jak ten program:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let x = 3.1415;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

Uruchomienie `cargo clippy` w tym projekcie powoduje następujący błąd:

```text
error: approximate value of `f{32, 64}::consts::PI` found
 --> src/main.rs:2:13
  |
2 |     let x = 3.1415;
  |             ^^^^^^
  |
  = note: `#[deny(clippy::approx_constant)]` on by default
  = help: consider using the constant directly
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#approx_constant
```

Ten błąd informuje, że Rust ma już zdefiniowaną bardziej precyzyjną stałą `PI` i że Twój program byłby bardziej poprawny, gdybyś użył stałej. Następnie powinieneś zmienić swój kod, aby użyć stałej `PI`. Poniższy kod nie powoduje żadnych błędów ani ostrzeżeń ze strony Clippy:
<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let x = std::f64::consts::PI;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

Więcej informacji na temat Clippy znajdziesz w [jego dokumentacji][clippy].

[clippy]: https://github.com/rust-lang/rust-clippy

### IDE Integration Using `rust-analyzer`

Aby ułatwić integrację IDE, społeczność Rust zaleca używanie
[`rust-analyzer`][rust-analyzer]<!-- ignore -->. To narzędzie to zestaw
narzędzi zorientowanych na kompilator, które obsługują [Language Server Protocol][lsp]<!--
ignore -->, czyli specyfikację dla IDE i języków programowania, aby
się ze sobą komunikować. Różni klienci mogą używać `rust-analyzer`, takiego jak
[wtyczka Rust Analyzer dla Visual Studio Code][vscode].

[lsp]: http://langserver.org/
[vscode]: https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer

Odwiedź stronę główną projektu `rust-analyzer` [rust-analyzer]<!-- ignore -->, aby uzyskać instrukcje dotyczące instalacji, a następnie zainstaluj obsługę serwera języka w swoim
konkretnym IDE. Twoje IDE zyska takie możliwości, jak autouzupełnianie, przeskakiwanie do definicji i błędy inline.

[rust-analyzer]: https://rust-analyzer.github.io
[editions]: appendix-05-editions.md
