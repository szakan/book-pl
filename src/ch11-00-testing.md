# Pisanie testów automatycznych

W swoim eseju z 1972 roku „The Humble Programmer” Edsger W. Dijkstra stwierdził, że
„Testowanie programów może być bardzo skutecznym sposobem na pokazanie obecności błędów, ale
jest beznadziejnie niewystarczające do pokazania ich braku”. Nie oznacza to, że
nie powinniśmy próbować testować tak dużo, jak to możliwe!

Poprawność naszych programów to stopień, w jakim nasz kod robi to, co zamierzaliśmy
robić. Rust został zaprojektowany z dużym naciskiem na poprawność
programów, ale poprawność jest złożona i niełatwa do udowodnienia. System
typów Rusta bierze na siebie ogromną część tego ciężaru, ale system typów nie jest w stanie
wyłapać wszystkiego. W związku z tym Rust obejmuje obsługę pisania zautomatyzowanych testów oprogramowania.

Powiedzmy, że piszemy funkcję `add_two`, która dodaje 2 do dowolnej liczby, która jest do niej przekazywana. Sygnatura tej funkcji akceptuje liczbę całkowitą jako parametr i zwraca
liczbę całkowitą jako wynik. Kiedy implementujemy i kompilujemy tę funkcję, Rust wykonuje całą
kontrolę typu i sprawdzanie pożyczania, których się do tej pory nauczyłeś, aby upewnić się,
że na przykład nie przekazujemy wartości `String` lub nieprawidłowego odwołania
do tej funkcji. Ale Rust *nie może* sprawdzić, czy ta funkcja zrobi dokładnie to,
co zamierzamy, czyli zwróci parametr plus 2, a nie, powiedzmy,
parametr plus 10 lub parametr minus 50! Tu właśnie wkraczają testy.

Możemy pisać testy, które na przykład potwierdzają, że kiedy przekazujemy `3` do funkcji
`add_two`, zwracana wartość to `5`. Możemy uruchamiać te testy, kiedy
wprowadzamy zmiany w naszym kodzie, aby upewnić się, że żadne istniejące poprawne zachowanie nie uległo
zmianie.

Testowanie to złożona umiejętność: chociaż nie możemy omówić w jednym rozdziale wszystkich szczegółów,
jak pisać dobre testy, w tym rozdziale omówimy mechanikę
urządzeń testujących Rusta. Porozmawiamy o adnotacjach i makrach,
które są dostępne podczas pisania testów, domyślnym zachowaniu i opcjach
dostępnych do uruchamiania testów oraz o tym, jak organizować testy w testy jednostkowe i
testy integracyjne.
