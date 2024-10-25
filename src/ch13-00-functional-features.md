# Cechy języka funkcyjnego: iteratory i zamknięcia

Projekt Rust został zainspirowany wieloma istniejącymi językami i
technikami, a jednym ze znaczących wpływów jest *programowanie funkcyjne*.
Programowanie w stylu funkcyjnym często obejmuje używanie funkcji jako wartości poprzez
przekazywanie ich w argumentach, zwracanie ich z innych funkcji, przypisywanie ich
do zmiennych w celu późniejszego wykonania itd.

W tym rozdziale nie będziemy dyskutować nad kwestią, czym jest lub
nie jest programowanie funkcyjne, ale zamiast tego omówimy niektóre funkcje Rust, które są podobne do funkcji w wielu językach często określanych jako funkcyjne.

Dokładniej, omówimy:

* *Zamknięcia*, konstrukcję podobną do funkcji, którą można przechowywać w zmiennej
* *Iteratory*, sposób przetwarzania serii elementów
* Jak używać zamknięć i iteratorów, aby ulepszyć projekt wejścia/wyjścia w rozdziale 12
* Wydajność zamknięć i iteratorów (Uwaga, spoiler: są szybsze, niż możesz myśleć!)

Omówiliśmy już inne funkcje Rust, takie jak dopasowywanie wzorców i
wyliczenia, na które również wpływa styl funkcjonalny. Ponieważ opanowanie
zamknięć i iteratorów jest ważną częścią pisania idiomatycznego, szybkiego kodu Rust, poświęcimy im cały ten rozdział.
