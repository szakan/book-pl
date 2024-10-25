## Porównanie wydajności: pętle kontra iteratory

Aby ustalić, czy użyć pętli, czy iteratorów, musisz wiedzieć, która implementacja jest szybsza: wersja funkcji `search` z wyraźną pętlą
`for` czy wersja z iteratorami.

Przeprowadziliśmy test porównawczy, ładując całą zawartość *The Adventures of
Sherlock Holmes* Sir Arthura Conana Doyle'a do `String` i szukając
słowa *the* w zawartości. Oto wyniki testu porównawczego dla
wersji `search` z pętlą `for` i wersji z iteratorami:

```text
test bench_search_for ... bench: 19 620 300 ns/iter (+/- 915 700)
test bench_search_iter ... bench: 19 234 900 ns/iter (+/- 657 200)
```

Wersja iteratora była nieco szybsza! Nie będziemy tutaj wyjaśniać kodu testowego,
ponieważ nie chodzi o udowodnienie, że obie wersje są równoważne,
ale o uzyskanie ogólnego obrazu tego, jak te dwie implementacje wypadają w porównaniu,
pod względem wydajności.

Aby uzyskać bardziej kompleksowy test, należy sprawdzić, używając różnych tekstów o
różnych rozmiarach jako `contents`, różnych słów i słów o różnej długości,
jako `query` i wszelkiego rodzaju innych wariantów. Chodzi o to, że:
iteratory, chociaż są abstrakcją wysokiego poziomu, są kompilowane do mniej więcej takiego samego kodu, jakbyś sam napisał kod niższego poziomu. Iteratory są jedną
z *abstrakcji bezkosztowych* Rusta, przez co rozumiemy, że użycie abstrakcji
nie nakłada dodatkowego narzutu na czas wykonania. Jest to analogiczne do tego, jak Bjarne
Stroustrup, pierwotny projektant i implementator języka C++, definiuje
*zerowe obciążenie* w „Foundations of C++” (2012):

> Ogólnie rzecz biorąc, implementacje języka C++ przestrzegają zasady zerowego obciążenia: czego
> nie używasz, nie płacisz. I dalej: czego używasz, nie mógłbyś napisać
> lepiej.

Jako inny przykład, poniższy kod pochodzi z dekodera audio. Algorytm dekodowania używa liniowej operacji matematycznej przewidywania, aby

oszacować przyszłe wartości na podstawie liniowej funkcji poprzednich próbek. Ten
kod używa łańcucha iteratorów, aby wykonać pewne obliczenia matematyczne na trzech zmiennych w zakresie:

wycinku danych „bufora”, tablicy 12 „współczynników” i wartości, o którą
przesunąć dane w „qlp_shift”. Zadeklarowaliśmy zmienne w tym przykładzie,
ale nie nadaliśmy im żadnych wartości; chociaż ten kod nie ma większego znaczenia
poza kontekstem, to nadal jest zwięzłym, realnym przykładem tego, jak Rust
tłumaczy idee wysokiego poziomu na kod niskiego poziomu.

```rust,ignore
let buffer: &mut [i32];
let coefficients: [i64; 12];
let qlp_shift: i16;

for i in 12..buffer.len() {
    let prediction = coefficients.iter()
                                 .zip(&buffer[i - 12..i])
                                 .map(|(&c, &s)| c * s as i64)
                                 .sum::<i64>() >> qlp_shift;
    let delta = buffer[i];
    buffer[i] = prediction as i32 + delta;
}
```

Aby obliczyć wartość `prediction`, ten kod przechodzi przez każdą z 12 wartości w `coefficients` i używa metody `zip`, aby sparować wartości współczynników z 12 poprzednimi wartościami w `buffer`. Następnie dla każdej pary mnożymy wartości, sumujemy wszystkie wyniki i przesuwamy bity w bitach sumy `qlp_shift` w prawo.

Obliczenia w aplikacjach, takich jak dekodery audio, często stawiają wydajność na pierwszym miejscu. Tutaj tworzymy iterator, używając dwóch adapterów, a następnie
konsumujemy wartość. Do jakiego kodu asemblera skompilowałby się ten kod Rust? Cóż,
w chwili pisania tego tekstu kompiluje się do tego samego kodu asemblera, który napisałbyś ręcznie.
Nie ma żadnej pętli odpowiadającej iteracji wartości w
`coefficients`: Rust wie, że jest 12 iteracji, więc „rozwija” pętlę. *Rozwijanie* to optymalizacja, która usuwa narzut pętli
kontrolującej kod i zamiast tego generuje powtarzalny kod dla każdej iteracji
pętli.

Wszystkie współczynniki są przechowywane w rejestrach, co oznacza, że ​​dostęp do
wartości jest bardzo szybki. Nie ma żadnych kontroli granic dostępu do tablicy w czasie wykonywania.
Wszystkie te optymalizacje, które Rust jest w stanie zastosować, sprawiają, że wynikowy kod
jest niezwykle wydajny. Teraz, gdy to wiesz, możesz używać iteratorów i zamknięć
bez obaw! Sprawiają, że kod wydaje się być wyższego poziomu, ale nie nakładają kary za wydajność w czasie wykonywania.

## Summary

Zamknięcia i iteratory to funkcje Rusta inspirowane pomysłami języka programowania funkcyjnego. Przyczyniają się do zdolności Rusta do jasnego wyrażania
pomysłów wysokiego poziomu przy wydajności niskiego poziomu. Implementacje zamknięć i
iteratorów są takie, że wydajność środowiska wykonawczego nie jest zakłócona. Jest to część
celu Rusta, jakim jest dążenie do zapewnienia abstrakcji o zerowym koszcie.

Teraz, gdy poprawiliśmy ekspresywność naszego projektu I/O, przyjrzyjmy się
kilku innym funkcjom `cargo`, które pomogą nam udostępnić projekt
światu.
