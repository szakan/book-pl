# Projekt końcowy: Budowa wielowątkowego serwera WWW

To była długa podróż, ale dotarliśmy do końca książki. W tym
rozdziale zbudujemy jeszcze jeden projekt, aby zademonstrować niektóre
koncepcje, które omówiliśmy w ostatnich rozdziałach, a także podsumować niektóre wcześniejsze
lekcje.

W naszym ostatnim projekcie stworzymy serwer WWW, który powie „hello” i będzie wyglądał jak
Rysunek 20-1 w przeglądarce internetowej.

![hello from rust](img/trpl20-01.png)

<span class="caption">Rysunek 20-1: Nasz ostateczny współdzielony projekt</span>

Oto nasz plan budowy serwera WWW:

1. Poznaj trochę TCP i HTTP.

2. Nasłuchuj połączeń TCP na gnieździe.

3. Przeanalizuj niewielką liczbę żądań HTTP.

4. Utwórz właściwą odpowiedź HTTP.

5. Popraw przepustowość naszego serwera za pomocą puli wątków.

Zanim zaczniemy, powinniśmy wspomnieć o jednym szczególe: metoda, której użyjemy, nie
będzie najlepszym sposobem na zbudowanie serwera WWW z Rust. Członkowie społeczności
opublikowali szereg gotowych do produkcji skrzynek dostępnych na
[crates.io](https://crates.io/), które zapewniają bardziej kompletne implementacje serwera WWW i
puli wątków niż te, które zbudujemy. Jednak naszym zamiarem w tym
rozdziale jest pomoc w nauce, a nie pójście na łatwiznę. Ponieważ Rust jest
językiem programowania systemowego, możemy wybrać poziom abstrakcji, z którym chcemy
pracować, i możemy przejść na niższy poziom niż jest to możliwe lub praktyczne w innych
językach. Dlatego napiszemy ręcznie podstawowy serwer HTTP i pulę wątków,
abyś mógł poznać ogólne idee i techniki stojące za skrzyniami, których możesz
użyć w przyszłości.
