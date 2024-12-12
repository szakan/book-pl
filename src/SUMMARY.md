# Język programowania Rust

[Język programowania Rust](title-page.md)
[Foreword](foreword.md)
[Introduction](ch00-00-introduction.md)

## Getting started

- [Pierwsze kroki](ch01-00-getting-started.md)
    - [Instalacja](ch01-01-installation.md)
    - [Hello, World!](ch01-02-hello-world.md)
    - [Hello, Cargo!](ch01-03-hello-cargo.md)

- [Programowanie gry polegającej na zgadywaniu](ch02-00-guessing-game-tutorial.md)

- [Typowe koncepcje programowania](ch03-00-common-programming-concepts.md)
    - [Zmienne i zmienność](ch03-01-variables-and-mutability.md)
    - [Typy danych](ch03-02-data-types.md)
    - [Funkcje](ch03-03-how-functions-work.md)
    - [Komentarze](ch03-04-comments.md)
    - [Przepływ sterowania](ch03-05-control-flow.md)

- [Zrozumienie własności](ch04-00-understanding-ownership.md)
    - [Czym jest własność??](ch04-01-what-is-ownership.md)
    - [Odwołania i pożyczanie](ch04-02-references-and-borrowing.md)
    - [Typ wycinka](ch04-03-slices.md)

- [Używanie struktur do strukturyzacji powiązanych danych](ch05-00-structs.md)
    - [Definiowanie i tworzenie instancji struktur](ch05-01-defining-structs.md)
    - [Przykładowy program wykorzystujący struktury](ch05-02-example-structs.md)
    - [Składnia metody](ch05-03-method-syntax.md)

- [Wyliczenia i dopasowywanie wzorców](ch06-00-enums.md)
    - [Definiowanie wyliczenia](ch06-01-defining-an-enum.md)
    - [The `match` Przepływ sterowania Construct](ch06-02-match.md)
    - [Concise Przepływ sterowania with `if let`](ch06-03-if-let.md)

## Basic Rust Literacy

- [Managing Growing Projects with Packages, Crates, and Modules](ch07-00-managing-growing-projects-with-packages-crates-and-modules.md)
    - [Pakiety i skrzynie](ch07-01-packages-and-crates.md)
    - [Definiowanie modułów w celu kontrolowania zakresu i prywatności](ch07-02-defining-modules-to-control-scope-and-privacy.md)
    - [Paths for Referring to an Item in the Module Tree](ch07-03-paths-for-referring-to-an-item-in-the-module-tree.md)
    - [Wprowadzanie ścieżek do zakresu za pomocą słowa kluczowego `use`](ch07-04-bringing-paths-into-scope-with-the-use-keyword.md)
    - [Rozdzielanie modułów do różnych plików](ch07-05-separating-modules-into-different-files.md)

- [Wspólne kolekcje](ch08-00-common-collections.md)
    - [Przechowywanie list wartości z wektorami](ch08-01-vectors.md)
    - [Przechowywanie zakodowanego tekstu UTF-8 z ciągami](ch08-02-strings.md)
    - [Przechowywanie kluczy z powiązanymi wartościami w mapach skrótów](ch08-03-hash-maps.md)

- [Obsługa błędów](ch09-00-error-handling.md)
    - [Nieodwracalne błędy z `panic!`](ch09-01-unrecoverable-errors-with-panic.md)
    - [Odwracalne błędy z `Result`](ch09-02-recoverable-errors-with-result.md)
    - [Panikować! czy nie?](ch09-03-to-panic-or-not-to-panic.md)

- [Generic Types, Traits, and Lifetimes](ch10-00-generics.md)
    - [Generic Typy danych](ch10-01-syntax.md)
    - [Cechy: definiowanie współdzielonego zachowania](ch10-02-traits.md)
    - [Walidacja odwołań za pomocą okresów życia](ch10-03-lifetime-syntax.md)

- [Pisanie testów automatycznych](ch11-00-testing.md)
    - [Jak pisać testy](ch11-01-writing-tests.md)
    - [Kontrolowanie sposobu uruchamiania testów](ch11-02-running-tests.md)
    - [Organizacja testów](ch11-03-test-organization.md)

- [An I/O Project: Building a Command Line Program](ch12-00-an-io-project.md)
    - [Akceptowanie argumentów wiersza poleceń](ch12-01-accepting-command-line-arguments.md)
    - [Odczyt pliku](ch12-02-reading-a-file.md)
    - [Refactoring to Improve Modularity and Obsługa błędów](ch12-03-improving-error-handling-and-modularity.md)
    - [Rozwijanie funkcjonalności biblioteki za pomocą programowania sterowanego testami](ch12-04-testing-the-librarys-functionality.md)
    - [Praca ze zmiennymi środowiskowymi](ch12-05-working-with-environment-variables.md)
    - [Zapisywanie komunikatów o błędach na standardowym wyjściu zamiast na standardowym wyjściu](ch12-06-writing-to-stderr-instead-of-stdout.md)

## Thinking in Rust

- [Cechy języka funkcyjnego: iteratory i zamknięcia](ch13-00-functional-features.md)
    - [Closures: Anonymous Funkcje that Capture Their Environment](ch13-01-closures.md)
    - [Przetwarzanie serii elementów za pomocą iteratorów](ch13-02-iterators.md)
    - [Ulepszanie naszego projektu I/O](ch13-03-improving-our-io-project.md)
    - [Porównanie wydajności: pętle kontra iteratory](ch13-04-performance.md)

- [Więcej o Cargo i Crates.io](ch14-00-more-about-cargo.md)
    - [Dostosowywanie kompilacji za pomocą profili wydań](ch14-01-release-profiles.md)
    - [Publikowanie Crate w Crates.io](ch14-02-publishing-to-crates-io.md)
    - [Obszary robocze Cargo](ch14-03-cargo-workspaces.md)
    - [Instalowanie plików binarnych z Crates.io za pomocą `cargo install`](ch14-04-installing-binaries.md)
    - [Rozszerzanie Cargo za pomocą niestandardowych poleceń](ch14-05-extending-cargo.md)

- [Inteligentne wskaźniki](ch15-00-smart-pointers.md)
    - [Używanie `Box<T>` do wskazywania danych na stercie](ch15-01-box.md)
    - [Treating Inteligentne wskaźniki Like Regular References with the `Deref` Trait](ch15-02-deref.md)
    - [Uruchamianie kodu podczas czyszczenia z cechą `Drop`](ch15-03-drop.md)
    - [`Rc<T>`, the Reference Counted Smart Pointer](ch15-04-rc.md)
    - [`RefCell<T>` i wzorzec wewnętrznej zmienności](ch15-05-interior-mutability.md)
    - [Cykle odniesień mogą powodować wyciek pamięci](ch15-06-reference-cycles.md)

- [Bezpieczna współbieżność](ch16-00-concurrency.md)
    - [Używanie wątków do jednoczesnego uruchamiania kodu](ch16-01-threads.md)
    - [Używanie przekazywania komunikatów do przesyłania danych między wątkami](ch16-02-message-passing.md)
    - [Współdzielona współbieżność stanu](ch16-03-shared-state.md)
    - [Rozszerzalna współbieżność z cechami `Sync` i `Send`](ch16-04-extensible-concurrency-sync-and-send.md)

- [Async i Await](ch17-00-async-await.md)
    - [Futures i Składnia asynchroniczna](ch17-01-futures-and-syntax.md)
    - [Współbieżność z asynchronicznością](ch17-02-concurrency-with-async.md)
    - [Praca z dowolną liczbą przyszłości](ch17-03-more-futures.md)
    - [Strumienie](ch17-04-streams.md)
    - [Zagłębianie się w cechy asynchroniczne](ch17-05-traits-for-async.md)
    - [Przyszłości, zadania i wątki](ch17-06-futures-tasks-threads.md)

- [Funkcje programowania obiektowego Rust](ch18-00-oop.md)
    - [Charakterystyka języków obiektowych](ch18-01-what-is-oo.md)
    - [Korzystanie z obiektów cech, które dopuszczają wartości różnych typów](ch18-02-trait-objects.md)
    - [Implementacja wzorca projektowego obiektowego](ch18-03-oo-design-patterns.md)

## Advanced Topics

- [Wzory i dopasowywanie](ch19-00-patterns.md)
    - [Wszystkie miejsca. w których można używać wzorców](ch19-01-all-the-places-for-patterns.md)
    - [Obalalność: czy wzorzec może nie pasować](ch19-02-refutability.md)
    - [Składnia wzorca](ch19-03-pattern-syntax.md)

- [Zaawansowane funkcje](ch20-00-advanced-features.md)
    - [Niebezpieczny Rust](ch20-01-unsafe-rust.md)
    - [Zaawansowane cechy](ch20-03-advanced-traits.md)
    - [Zaawansowane typy](ch20-04-advanced-types.md)
    - [Zaawansowane funkcje i zamknięcia](ch20-05-advanced-functions-and-closures.md)
    - [Makra](ch20-06-macros.md)

- [Projekt końcowy: Budowa wielowątkowego serwera WWW](ch21-00-final-project-a-web-server.md)
    - [Budowanie jednowątkowego serwera WWW](ch21-01-single-threaded.md)
    - [Zmiana naszego jednowątkowego serwera na wielowątkowy serwer](ch21-02-multithreaded.md)
    - [Łagodne zamykanie i czyszczenie](ch21-03-graceful-shutdown-and-cleanup.md)

- [Załącznik](appendix-00.md)
    - [A - Słowa kluczowe](appendix-01-keywords.md)
    - [B - Operatorzy i symbole](appendix-02-operators.md)
    - [C - Cechy pochodne](appendix-03-derivable-traits.md)
    - [D - Przydatne narzędzia programistyczne](appendix-04-useful-development-tools.md)
    - [E - Wydania](appendix-05-editions.md)
    - [F - Tłumaczenia książki](appendix-06-translation.md)
    - [G - How Rust is Made and “Nightly Rust”](appendix-07-nightly-rust.md)
