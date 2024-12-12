## Async i Await

Wiele operacji, o których wykonanie prosimy komputer, może zająć trochę czasu. Na przykład, jeśli użyłeś edytora wideo, aby utworzyć film z rodzinnej uroczystości,
jego eksportowanie może zająć od kilku minut do kilku godzin. Podobnie, pobieranie
filmu udostępnionego przez kogoś z rodziny może zająć dużo czasu. Byłoby miło,
gdybyśmy mogli robić coś innego, podczas gdy czekamy na zakończenie tych długotrwałych
procesów.

Eksportowanie wideo będzie wykorzystywać tyle mocy procesora i procesora graficznego, ile tylko będzie możliwe. Gdybyś miał
tylko jeden rdzeń procesora, a twój system operacyjny nigdy nie wstrzymywał tego eksportu, dopóki się nie zakończy, nie mógłbyś robić niczego innego na swoim komputerze, gdy był uruchomiony.
To byłoby jednak dość frustrujące doświadczenie. Zamiast tego, system operacyjny twojego komputera
może — i robi to! — niewidocznie przerywać eksport na tyle często,
żebyś mógł wykonać inną pracę w międzyczasie.

Pobieranie pliku jest inne. Nie zajmuje dużo czasu procesora. Zamiast tego,
procesor musi czekać na dane z sieci. Chociaż możesz zacząć
odczytywać dane, gdy część z nich jest już obecna, może minąć trochę czasu, zanim reszta
się pojawi. Nawet gdy wszystkie dane są już obecne, wideo może być dość duże, więc załadowanie wszystkiego może zająć trochę czasu. Może to potrwać tylko sekundę lub dwie — ale
to bardzo długo dla nowoczesnego procesora, który może wykonywać miliardy
operacji na sekundę. Fajnie byłoby móc wykorzystać CPU do
innej pracy, czekając na zakończenie wywołania sieciowego — więc, ponownie, Twój
system operacyjny niewidocznie przerwie Twój program, więc inne rzeczy mogą się
zdarzyć, gdy operacja sieciowa jest nadal w toku.

> Uwaga: Eksport wideo to rodzaj operacji, która jest często opisywana jako
> „ograniczona przez procesor” lub „ograniczona przez obliczenia”. Jest ona ograniczona przez szybkość
> zdolności komputera do przetwarzania danych w *procesorze* lub *procesorze* i ile z tej szybkości
> może wykorzystać. Pobieranie wideo to rodzaj operacji, która jest często
> opisywana jako „związana z wejściem/wyjściem”, ponieważ jest ograniczona przez prędkość
> *wejścia i wyjścia* komputera. Może przebiegać tylko tak szybko, jak dane mogą być przesyłane przez
> sieć.

W obu tych przykładach niewidoczne przerwania systemu operacyjnego zapewniają
formę współbieżności. Ta współbieżność ma miejsce tylko na poziomie całego
programu: system operacyjny przerywa jeden program, aby umożliwić innym
programom wykonanie pracy. W wielu przypadkach, ponieważ rozumiemy nasze programy na
znacznie bardziej szczegółowym poziomie niż system operacyjny, możemy dostrzec wiele
możliwości współbieżności, których system operacyjny nie widzi.

Na przykład, jeśli tworzymy narzędzie do zarządzania pobieraniem plików, powinniśmy być w stanie napisać nasz program w taki sposób, aby rozpoczęcie jednego pobierania nie blokowało
interfejsu użytkownika, a użytkownicy powinni móc rozpocząć wiele pobrań jednocześnie. Wiele interfejsów API systemów operacyjnych do interakcji z siecią jest jednak
*blokujących*. Oznacza to, że te interfejsy API blokują postęp programu, dopóki
przetwarzane przez nie dane nie będą całkowicie gotowe.

> Uwaga: Tak właśnie działa *większość* wywołań funkcji, jeśli się nad tym zastanowić! Jednak
> zazwyczaj rezerwujemy termin „blokowanie” dla wywołań funkcji, które wchodzą w interakcję z
> plikami, siecią lub innymi zasobami na komputerze, ponieważ są to miejsca, w których
> indywidualny program skorzystałby na tym, że operacja
> *nie* blokuje.

Możemy uniknąć blokowania naszego wątku głównego, tworząc dedykowany wątek do
pobierania każdego pliku. Jednak ostatecznie odkrylibyśmy, że narzut tych
wątków stanowi problem. Byłoby również przyjemniej, gdyby wywołanie nie blokowało
od samego początku. Na koniec, ale nie mniej ważne, lepiej byłoby, gdybyśmy mogli pisać w
tym samym bezpośrednim stylu, którego używamy w kodzie blokującym. Coś podobnego do tego:

```rust,ignore,does_not_compile
let data = fetch_data_from(url).await;
println!("{data}");
```

To jest dokładnie to, co daje nam asynchroniczna abstrakcja Rusta. Zanim jednak zobaczymy, jak to
działa w praktyce, musimy zrobić krótki objazd po różnicach
między paralelizmem a współbieżnością.

### Paralelizm i współbieżność

W poprzednim rozdziale traktowaliśmy paralelizm i współbieżność jako w większości
zamienne. Teraz musimy je dokładniej rozróżnić, ponieważ
różnice pojawią się, gdy zaczniemy pracować.

Rozważ różne sposoby, w jakie zespół może podzielić pracę nad projektem oprogramowania. Możemy
przydzielić jednej osobie wiele zadań, możemy przydzielić jedno zadanie każdemu
członkowi zespołu lub możemy zastosować mieszankę obu podejść.

Kiedy osoba pracuje nad kilkoma różnymi zadaniami, zanim którekolwiek z nich zostanie
ukończone, to jest *współbieżność*. Być może masz dwa różne projekty sprawdzone
na swoim komputerze i gdy się znudzisz lub utkniesz w jednym projekcie, przełączasz się
na drugi. Jesteś tylko jedną osobą, więc nie możesz robić postępów w obu zadaniach
w tym samym czasie — ale możesz wykonywać wiele zadań jednocześnie, robiąc postępy w wielu
zadaniach, przełączając się między nimi.

<figure>

<img alt="Concurrent work flow" src="img/trpl17-01.svg" class="center" />

<figcaption>Figure 17-1: A concurrent workflow, switching between Task A and Task B.</figcaption>

</figure>

Kiedy zgadzasz się podzielić grupę zadań pomiędzy osoby w zespole, przy czym
każda osoba bierze jedno zadanie i pracuje nad nim sama, to jest *paralelizm*. Każda
osoba w zespole może robić postępy w dokładnie tym samym czasie.

<figure>

<img alt="Concurrent work flow" src="img/trpl17-02.svg" class="center" />

<figcaption>Figure 17-2: A parallel workflow, where work happens on Task A and Task B independently.</figcaption>

</figure>

W obu tych sytuacjach może zaistnieć konieczność koordynacji różnych
zadań. Być może *myślałeś*, że zadanie, nad którym pracowała jedna osoba, było całkowicie
niezależne od pracy wszystkich innych, ale w rzeczywistości wymagało czegoś, co musi
dokończyć inna osoba w zespole. Część pracy można było wykonać równolegle, ale
część z niej była w rzeczywistości *seryjna*: mogła się wydarzyć tylko w serii, jedna rzecz po drugiej, jak na rysunku 17-3.

<figure>

<img alt="Concurrent work flow" src="img/trpl17-03.svg" class="center" />

<figcaption>Rysunek 17-3: Częściowo równoległy przepływ pracy, w którym praca nad zadaniem A i zadaniem B jest wykonywana niezależnie do momentu, aż zadanie A3 zostanie zablokowane na wynikach zadania B3.</figcaption>

</figure>

Podobnie możesz zdać sobie sprawę, że jedno z Twoich zadań zależy od innego
Twojego zadania. Teraz Twoja praca współbieżna również stała się szeregowa.

Paralelizm i współbieżność mogą się również przecinać. Jeśli dowiesz się,
że współpracownik utknął, dopóki nie skończysz jednego ze swoich zadań, prawdopodobnie
skupisz wszystkie swoje wysiłki na tym zadaniu, aby „odblokować” swojego współpracownika. Ty i Twój
współpracownik nie jesteście już w stanie pracować równolegle, a Ty również nie jesteście już w stanie
pracować współbieżnie nad swoimi własnymi zadaniami.

Ta sama podstawowa dynamika wchodzi w grę w przypadku oprogramowania i sprzętu. Na komputerze
z jednym rdzeniem procesora, procesor może wykonywać tylko jedną operację na raz, ale może
nadal pracować współbieżnie. Używając narzędzi takich jak wątki, procesy i asynchroniczność,
komputer może wstrzymać jedną aktywność i przełączyć się na inne, zanim ostatecznie
powróci do tej pierwszej aktywności. Na komputerze z wieloma rdzeniami procesora może
również wykonywać pracę równolegle. Jedno jądro może wykonywać jedną czynność, podczas gdy inne jądro
robi coś zupełnie niezwiązanego, a te czynności dzieją się w tym samym
czasie.

Pracując z asynchronicznością w Rust, zawsze mamy do czynienia ze współbieżnością.
W zależności od sprzętu, systemu operacyjnego i środowiska wykonawczego asynchronicznego, którego
używamy — więcej o środowiskach wykonawczych asynchronicznych wkrótce! — współbieżność może również wykorzystywać paralelizm
pod maską.

Teraz zagłębmy się w to, jak naprawdę działa programowanie asynchroniczne w Rust! W dalszej części
tego rozdziału:

* zobaczymy, jak używać składni `async` i `await` Rust
* zbadamy, jak używać modelu asynchronicznego do rozwiązywania niektórych z tych samych problemów, które
rozważaliśmy w rozdziale 16
* przyjrzymy się, w jaki sposób wielowątkowość i asynchroniczność zapewniają uzupełniające się rozwiązania, których
w wielu przypadkach można nawet używać razem
