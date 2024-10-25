## Budowanie jednowątkowego serwera WWW

Zaczniemy od uruchomienia jednowątkowego serwera internetowego. Zanim zaczniemy,
spójrzmy na krótki przegląd protokołów używanych do budowy serwerów internetowych. Szczegóły tych protokołów wykraczają poza zakres tej książki, ale
krótki przegląd da ci potrzebne informacje.

Dwa główne protokoły używane w serwerach internetowych to *Hypertext Transfer
Protocol* *(HTTP)* i *Transmission Control Protocol* *(TCP)*. Oba protokoły
są protokołami *request-response*, co oznacza, że ​​*klient* inicjuje żądania, a
*serwer* słucha żądań i dostarcza odpowiedź klientowi.
Treść tych żądań i odpowiedzi jest definiowana przez protokoły.

TCP to protokół niższego poziomu, który opisuje szczegóły tego, w jaki sposób informacje
przechodzą z jednego serwera do drugiego, ale nie określa, jakie to są informacje.
HTTP opiera się na TCP, definiując zawartość żądań i
odpowiedzi. Technicznie rzecz biorąc, możliwe jest używanie HTTP z innymi protokołami, ale w
większości przypadków HTTP wysyła swoje dane przez TCP. Będziemy pracować z
surowymi bajtami żądań i odpowiedzi TCP i HTTP.

### Listening to the TCP Connection

Nasz serwer WWW musi nasłuchiwać połączenia TCP, więc to jest pierwsza część, nad którą będziemy pracować. Standardowa biblioteka oferuje moduł `std::net`, który nam to umożliwia. Stwórzmy nowy projekt w zwykły sposób:

```console
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
```

Teraz wprowadź kod z Listingu 21-1 w *src/main.rs*, aby rozpocząć. Ten kod będzie
nasłuchiwał pod adresem lokalnym `127.0.0.1:7878` przychodzących strumieni TCP. Gdy
otrzyma przychodzący strumień, wyświetli `Connection Established!`.

<Listing number="21-1" file-name="src/main.rs" caption="Listening for incoming streams and printing a message when we receive a stream">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-01/src/main.rs}}
```

</Listing>

Używając `TcpListener` możemy nasłuchiwać połączeń TCP pod adresem
`127.0.0.1:7878`. W adresie sekcja przed dwukropkiem to adres IP
reprezentujący Twój komputer (jest on taki sam na każdym komputerze i nie
reprezentuje konkretnie komputera autorów), a `7878` to port. Wybraliśmy ten port z dwóch powodów: HTTP nie jest normalnie akceptowany na tym porcie, więc
nasz serwer raczej nie będzie kolidował z żadnym innym serwerem internetowym, który możesz mieć
uruchomiony na swoim komputerze, a 7878 to *rust* wpisany w telefonie.

Funkcja `bind` w tym scenariuszu działa jak funkcja `new`, ponieważ
zwróci nową instancję `TcpListener`. Funkcja ta nazywa się `bind`,
ponieważ w sieciach łączenie się z portem w celu nasłuchiwania jest znane jako „wiązanie
z portem”.

Funkcja `bind` zwraca `Result<T, E>`, co wskazuje, że wiązanie może się nie powieść. Na przykład połączenie z portem 80 wymaga uprawnień administratora (użytkownicy niebędący administratorami mogą nasłuchiwać tylko na portach wyższych niż 1023), więc jeśli spróbujemy połączyć się z portem 80, nie będąc
administratorem, wiązanie nie zadziała. Wiązanie również nie zadziała, na przykład, jeśli uruchomimy dwie instancje naszego programu i będziemy mieć dwa programy nasłuchujące
tego samego portu. Ponieważ piszemy podstawowy serwer tylko w celach edukacyjnych, nie będziemy się martwić obsługą tego typu błędów; zamiast tego użyjemy `unwrap`, aby
zatrzymać program, jeśli wystąpią błędy.

Metoda `incoming` w `TcpListener` zwraca iterator, który daje nam
sekwencję strumieni (dokładniej strumieni typu `TcpStream`). Pojedynczy
*strumień* reprezentuje otwarte połączenie między klientem a serwerem.
*połączenie* to nazwa pełnego procesu żądania i odpowiedzi, w którym
klient łączy się z serwerem, serwer generuje odpowiedź, a serwer
zamyka połączenie. W związku z tym odczytamy z `TcpStream`, aby zobaczyć, co
klient wysłał, a następnie zapiszemy naszą odpowiedź do strumienia, aby odesłać dane z powrotem do
klienta. Ogólnie rzecz biorąc, ta pętla `for` przetworzy każde połączenie po kolei i
wygeneruje serię strumieni do obsłużenia.

Na razie nasza obsługa strumienia polega na wywołaniu `unwrap` w celu zakończenia
naszego programu, jeśli strumień ma jakieś błędy; jeśli nie ma żadnych błędów,
program drukuje komunikat. Dodamy więcej funkcji dla przypadku powodzenia w
następnym listingu. Powodem, dla którego możemy otrzymywać błędy z metody `incoming`,
gdy klient łączy się z serwerem, jest to, że tak naprawdę nie iterujemy
połączeń. Zamiast tego iterujemy *próby połączenia*.
Połączenie może się nie powieść z wielu powodów, z których wiele jest specyficznych dla systemu operacyjnego. Na przykład wiele systemów operacyjnych ma limit
liczby równoczesnych otwartych połączeń, które mogą obsługiwać; nowe próby połączenia
powyżej tej liczby spowodują błąd, dopóki niektóre otwarte
połączenia nie zostaną zamknięte.

Spróbujmy uruchomić ten kod! Wywołaj `cargo run` w terminalu, a następnie załaduj
*127.0.0.1:7878* w przeglądarce internetowej. Przeglądarka powinna wyświetlić komunikat o błędzie,
taki jak „Połączenie zresetowane”, ponieważ serwer obecnie nie wysyła żadnych
danych. Ale gdy spojrzysz na swój terminal, powinieneś zobaczyć kilka komunikatów,
które zostały wydrukowane, gdy przeglądarka połączyła się z serwerem!

```text
     Running `target/debug/hello`
Connection established!
Connection established!
Connection established!
```

Czasami zobaczysz wiele komunikatów wydrukowanych dla jednego żądania przeglądarki; powodem może być to, że przeglądarka wysyła żądanie strony, a także żądanie innych zasobów, takich jak ikona *favicon.ico*, która pojawia się na karcie przeglądarki.

Może się również zdarzyć, że przeglądarka próbuje połączyć się z serwerem wiele razy,
ponieważ serwer nie odpowiada żadnymi danymi. Gdy `stream` wychodzi poza zakres i zostaje porzucony na końcu pętli, połączenie jest zamykane jako
część implementacji `drop`. Przeglądarki czasami radzą sobie z zamkniętymi
połączeniami, ponawiając próbę, ponieważ problem może być tymczasowy. Ważnym
czynnikiem jest to, że udało nam się uzyskać uchwyt połączenia TCP!

Pamiętaj, aby zatrzymać program, naciskając <kbd>ctrl</kbd>-<kbd>c</kbd>, gdy
zakończysz uruchamianie określonej wersji kodu. Następnie uruchom ponownie program,
wywołując polecenie `cargo run` po wprowadzeniu każdego zestawu zmian w kodzie,
aby upewnić się, że uruchamiasz najnowszy kod.

### Reading the Request

Zaimplementujmy funkcjonalność odczytu żądania z przeglądarki! Aby
oddzielić kwestie związane z pierwszym uzyskaniem połączenia, a następnie podjęciem działań
z tym połączeniem, uruchomimy nową funkcję do przetwarzania połączeń. W
tej nowej funkcji `handle_connection` odczytamy dane ze strumienia TCP i
wydrukujemy je, abyśmy mogli zobaczyć dane wysyłane z przeglądarki. Zmień kod, aby
wyglądał jak Listing 21-2.

<Listing number="21-2" file-name="src/main.rs" caption="Reading from the `TcpStream` and printing the data">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-02/src/main.rs}}
```

</Listing>

Wprowadzamy `std::io::prelude` i `std::io::BufReader` do zakresu, aby uzyskać dostęp
do cech i typów, które pozwalają nam czytać ze strumienia i zapisywać do niego. W pętli `for`
w funkcji `main` zamiast drukować komunikat informujący, że nawiązaliśmy
połączenie, wywołujemy teraz nową funkcję `handle_connection` i przekazujemy jej
`stream`.

W funkcji `handle_connection` tworzymy nową instancję `BufReader`, która
opakowuje odwołanie do `stream`. `BufReader` dodaje buforowanie, zarządzając wywołaniami
metod cech `std::io::Read` za nas.

Tworzymy zmienną o nazwie `http_request`, aby zbierać wiersze żądania,
które przeglądarka wysyła do naszego serwera. Wskazujemy, że chcemy zbierać te wiersze w wektorze, dodając adnotację typu `Vec<_>`.

`BufReader` implementuje cechę `std::io::BufRead`, która zapewnia metodę `lines`
. Metoda `lines` zwraca iterator `Result<String,
std::io::Error>`, dzieląc strumień danych za każdym razem, gdy widzi bajt nowej linii. Aby uzyskać każdy `String`, mapujemy i `rozpakowujemy` każdy `Result`. `Result`
może być błędem, jeśli dane nie są prawidłowym UTF-8 lub jeśli wystąpił problem
z odczytem ze strumienia. Ponownie, program produkcyjny powinien obsługiwać te błędy
bardziej elegancko, ale dla
prostoty decydujemy się zatrzymać program w przypadku błędu.

Przeglądarka sygnalizuje koniec żądania HTTP, wysyłając dwa znaki nowej linii z rzędu, więc aby uzyskać jedno żądanie ze strumienia, pobieramy lines, aż
otrzymamy wiersz, który jest pustym ciągiem. Po zebraniu linii do
wektora, drukujemy je, używając ładnego formatowania debugowania, abyśmy mogli
przyjrzeć się instrukcjom, które przeglądarka internetowa wysyła do naszego serwera.

Wypróbujmy ten kod! Uruchom program i ponownie wyślij żądanie w przeglądarce internetowej. Zauważ, że nadal otrzymamy stronę błędu w przeglądarce, ale wynik naszego
programu w terminalu będzie teraz wyglądał podobnie do tego:

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language: en-US,en;q=0.5",
    "Accept-Encoding: gzip, deflate, br",
    "DNT: 1",
    "Connection: keep-alive",
    "Upgrade-Insecure-Requests: 1",
    "Sec-Fetch-Dest: document",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-User: ?1",
    "Cache-Control: max-age=0",
]
```

W zależności od przeglądarki możesz otrzymać nieco inny wynik. Teraz, gdy
drukujemy dane żądania, możemy zobaczyć, dlaczego otrzymujemy wiele połączeń
z jednego żądania przeglądarki, patrząc na ścieżkę po `GET` w pierwszym wierszu
żądania. Jeśli wszystkie powtarzające się połączenia żądają */*, wiemy, że
przeglądarka próbuje pobrać */* wielokrotnie, ponieważ nie otrzymuje odpowiedzi
od naszego programu.

Przeanalizujmy te dane żądania, aby zrozumieć, czego przeglądarka żąda od
naszego programu.

### A Closer Look at an HTTP Request

HTTP jest protokołem tekstowym, a żądanie przyjmuje następujący format:

```text
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

Pierwszy wiersz to *wiersz żądania*, który zawiera informacje o tym, czego
klient żąda. Pierwsza część wiersza żądania wskazuje na używaną *metodę*, taką jak `GET` lub `POST`, która opisuje, w jaki sposób klient
wysyła to żądanie. Nasz klient użył żądania `GET`, co oznacza, że ​​prosi o
informacje.

Następna część wiersza żądania to */*, która wskazuje na *Uniform Resource
Identifier* *(URI)*, którego żąda klient: URI jest prawie, ale nie do końca,
tym samym, co *Uniform Resource Locator* *(URL)*. Różnica między URI
i URL-ami nie jest istotna dla naszych celów w tym rozdziale, ale specyfikacja HTTP
używa terminu URI, więc możemy po prostu w myślach zastąpić URL URI tutaj.

Ostatnia część to wersja HTTP używana przez klienta, a następnie wiersz żądania
kończy się sekwencją *CRLF*. (CRLF oznacza *powrót karetki* i *przesunięcie wiersza*,
które są terminami z czasów maszyn do pisania!) Sekwencję CRLF można również
zapisać jako `\r\n`, gdzie `\r` to powrót karetki, a `\n` to przesunięcie wiersza. Sekwencja
CRLF oddziela wiersz żądania od pozostałych danych żądania.
Zauważ, że gdy CRLF jest drukowany, widzimy nowy początek wiersza, a nie `\r\n`.

Patrząc na dane wiersza żądania, które otrzymaliśmy do tej pory z uruchomienia naszego programu,
widzimy, że `GET` to metoda, */* to URI żądania, a `HTTP/1.1` to
wersja.

Po wierszu żądania pozostałe wiersze, zaczynając od `Host:`, to
nagłówki. Żądania `GET` nie mają treści.

Spróbuj wysłać żądanie z innej przeglądarki lub poprosić o inny
adres, taki jak *127.0.0.1:7878/test*, aby zobaczyć, jak zmieniają się dane żądania.

Teraz, gdy wiemy, o co prosi przeglądarka, możemy wysłać jej jakieś dane!

### Writing a Response

Zamierzamy wdrożyć wysyłanie danych w odpowiedzi na żądanie klienta.
Odpowiedzi mają następujący format:

```text
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

Pierwszy wiersz to *linia statusu* zawierająca wersję HTTP użytą w
odpowiedzi, numeryczny kod statusu podsumowujący wynik żądania oraz
frazę powodu, która zawiera tekstowy opis kodu statusu. Po
sekwencji CRLF znajdują się wszelkie nagłówki, kolejna sekwencja CRLF oraz treść
odpowiedzi.

Oto przykładowa odpowiedź, która używa wersji HTTP 1.1, ma kod statusu
200, frazę powodu OK, brak nagłówków i brak treści:

```text
HTTP/1.1 200 OK\r\n\r\n
```

Kod statusu 200 to standardowa odpowiedź o powodzeniu. Tekst to maleńka
pomyślna odpowiedź HTTP. Zapiszmy to w strumieniu jako naszą odpowiedź na
pomyślne żądanie! Z funkcji `handle_connection` usuń
`println!`, która drukowała dane żądania i zastąp ją kodem z
Listingu 21-3.

<Listing number="21-3" file-name="src/main.rs" caption="Pisanie małej, pomyślnej odpowiedzi HTTP do strumienia">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-03/src/main.rs:here}}
```

</Listing>

Pierwszy nowy wiersz definiuje zmienną `response`, która przechowuje dane
komunikatu o powodzeniu. Następnie wywołujemy `as_bytes` w naszym `response`, aby przekonwertować ciąg
danych na bajty. Metoda `write_all` w `stream` przyjmuje `&[u8]` i wysyła
te bajty bezpośrednio przez połączenie. Ponieważ operacja `write_all`
może się nie powieść, używamy `unwrap` w przypadku każdego wyniku błędu, jak poprzednio. Ponownie, w prawdziwej
aplikacji dodałbyś tutaj obsługę błędów.

Dzięki tym zmianom uruchommy nasz kod i złóżmy żądanie. Nie drukujemy już żadnych danych w terminalu, więc nie zobaczymy żadnego wyjścia poza
wyjściem z Cargo. Gdy ładujesz *127.0.0.1:7878* w przeglądarce internetowej, powinieneś
otrzymać pustą stronę zamiast błędu. Właśnie ręcznie zakodowałeś otrzymanie żądania HTTP i wysłanie odpowiedzi!

### Returning Real HTML

Zaimplementujmy funkcjonalność zwracania czegoś więcej niż pustej strony. Utwórz
nowy plik *hello.html* w katalogu głównym swojego projektu, a nie w katalogu
*src*. Możesz wprowadzić dowolny kod HTML; Listing 21-4 pokazuje jedną
możliwość.

<Listing number="21-4" file-name="hello.html" caption="A sample HTML file to return in a response">

```html
{{#include ../listings/ch21-web-server/listing-21-05/hello.html}}
```

</Listing>

To jest minimalny dokument HTML5 z nagłówkiem i tekstem. Aby zwrócić to
z serwera po otrzymaniu żądania, zmodyfikujemy `handle_connection`, jak
pokazano w Liście 21-5, aby odczytać plik HTML, dodać go do odpowiedzi jako treść,
i wysłać.

<Listing number="21-5" file-name="src/main.rs" caption="Sending the contents of *hello.html* as the body of the response">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-05/src/main.rs:here}}
```

</Listing>

Dodaliśmy `fs` do polecenia `use`, aby wprowadzić moduł systemu plików biblioteki standardowej do zakresu. Kod odczytu zawartości pliku do
ciągu powinien wyglądać znajomo; użyliśmy go w rozdziale 12, gdy odczytaliśmy zawartość
pliku dla naszego projektu I/O w Liście 12-4.

Następnie używamy `format!`, aby dodać zawartość pliku jako treść odpowiedzi o powodzeniu. Aby zapewnić prawidłową odpowiedź HTTP, dodajemy nagłówek `Content-Length`,
który jest ustawiony na rozmiar treści naszej odpowiedzi, w tym przypadku rozmiar
`hello.html`.

Uruchom ten kod za pomocą `cargo run` i załaduj *127.0.0.1:7878* w przeglądarce;
powinieneś zobaczyć wyrenderowany kod HTML!

Obecnie ignorujemy dane żądania w `http_request` i po prostu
bezwarunkowo odsyłamy zawartość pliku HTML. Oznacza to, że jeśli spróbujesz
zażądać *127.0.0.1:7878/coś-innego* w swojej przeglądarce, nadal otrzymasz
tę samą odpowiedź HTML. W tej chwili nasz serwer jest bardzo ograniczony i
nie robi tego, co robi większość serwerów internetowych. Chcemy dostosować nasze odpowiedzi
w zależności od żądania i odesłać plik HTML tylko dla poprawnie sformatowanego
żądania do */*.

### Validating the Request and Selectively Responding

Teraz nasz serwer internetowy zwróci kod HTML w pliku niezależnie od tego, czego zażądał
klient. Dodajmy funkcjonalność sprawdzającą, czy przeglądarka
żąda */* przed zwróceniem pliku HTML i zwracającą błąd, jeśli
przeglądarka zażąda czegoś innego. W tym celu musimy zmodyfikować `handle_connection`,
jak pokazano na liście 21-6. Ten nowy kod sprawdza zawartość otrzymanego żądania,
na podstawie tego, jak wygląda żądanie */* i dodaje bloki `if` i
`else`, aby traktować żądania inaczej.

<Listing number="21-6" file-name="src/main.rs" caption="Handling requests to */* differently from other requests">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-06/src/main.rs:here}}
```

</Listing>

Będziemy patrzeć tylko na pierwszy wiersz żądania HTTP, więc zamiast odczytywać całe żądanie do wektora, wywołujemy `next`, aby pobrać
pierwszy element z iteratora. Pierwszy `unwrap` zajmuje się `Option` i
zatrzymuje program, jeśli iterator nie ma żadnych elementów. Drugi `unwrap` obsługuje
`Result` i ma taki sam efekt, jak `unwrap`, który znajdował się w `map` dodanym w
Listingu 21-2.

Następnie sprawdzamy `request_line`, aby zobaczyć, czy jest równy wierszowi żądania GET
do ścieżki */*. Jeśli tak, blok `if` zwraca zawartość naszego pliku
HTML.

Jeśli `request_line` *nie* jest równy żądaniu GET do ścieżki */*, oznacza to, że
otrzymaliśmy jakieś inne żądanie. Dodamy kod do bloku `else` za chwilę, aby odpowiedzieć na wszystkie inne żądania.

Uruchom teraz ten kod i zażądaj *127.0.0.1:7878*; powinieneś otrzymać kod HTML w
*hello.html*. Jeśli wyślesz jakiekolwiek inne żądanie, takie jak
*127.0.0.1:7878/something-else*, otrzymasz błąd połączenia, taki jak te, które
widziałeś podczas uruchamiania kodu w Listingu 21-1 i Listingu 21-2.

Teraz dodajmy kod z Listingu 21-7 do bloku `else`, aby zwrócić odpowiedź
z kodem stanu 404, który sygnalizuje, że treść żądania
nie została znaleziona. Zwrócimy również kod HTML dla strony do wyświetlenia w przeglądarce,
wskazując odpowiedź użytkownikowi końcowemu.

<Listing number="21-7" file-name="src/main.rs" caption="Responding with status code 404 and an error page if anything other than */* was requested">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-07/src/main.rs:here}}
```

</Listing>

Tutaj nasza odpowiedź ma wiersz statusu z kodem statusu 404 i frazą powodu
`NIE ZNALEZIONO`. Treść odpowiedzi będzie w formacie HTML w pliku *404.html*.
Będziesz musiał utworzyć plik *404.html* obok *hello.html* dla strony błędu; ponownie możesz użyć dowolnego kodu HTML lub przykładowego kodu HTML w
Listingu 21-8.

<Listing number="21-8" file-name="404.html" caption="Sample content for the page to send back with any 404 response">

```html
{{#include ../listings/ch21-web-server/listing-21-07/404.html}}
```

</Listing>

Po wprowadzeniu tych zmian uruchom ponownie swój serwer. Żądanie *127.0.0.1:7878* powinno
zwrócić zawartość *hello.html*, a każde inne żądanie, takie jak
*127.0.0.1:7878/foo*, powinno zwrócić kod HTML błędu z *404.html*.

### A Touch of Refactoring

W tej chwili bloki `if` i `else` mają wiele powtórzeń: oba
odczytują pliki i zapisują ich zawartość do strumienia. Jedyne
różnice to wiersz statusu i nazwa pliku. Uczyńmy kod bardziej
zwięzłym, wyciągając te różnice do oddzielnych wierszy `if` i `else`,
które przypiszą wartości wiersza statusu i nazwy pliku do zmiennych;
możemy następnie bezwarunkowo używać tych zmiennych w kodzie do odczytu pliku
i zapisu odpowiedzi. Listing 21-9 pokazuje kod wynikowy po zastąpieniu
dużych bloków `if` i `else`.

<Listing number="21-9" file-name="src/main.rs" caption="Refactoring the `if` and `else` blocks to contain only the code that differs between the two cases">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-09/src/main.rs:here}}
```

</Listing>

Teraz bloki `if` i `else` zwracają tylko odpowiednie wartości dla
linii statusu i nazwy pliku w krotce; następnie używamy destrukturyzacji, aby przypisać te
dwie wartości do `status_line` i `filename` przy użyciu wzorca w instrukcji `let`, jak omówiono w rozdziale 19.

Wcześniej zduplikowany kod znajduje się teraz poza blokami `if` i `else` i
używa zmiennych `status_line` i `filename`. Ułatwia to
zauważenie różnicy między tymi dwoma przypadkami i oznacza, że ​​mamy tylko jedno miejsce,
aby zaktualizować kod, jeśli chcemy zmienić sposób, w jaki działa odczyt pliku i pisanie odpowiedzi. Zachowanie kodu w Liście 21-9 będzie takie samo, jak w
Liście 21-7.

Wspaniale! Teraz mamy prosty serwer WWW w około 40 wierszach kodu Rust,
który odpowiada na jedno żądanie stroną treści i odpowiada na wszystkie inne
żądania odpowiedzią 404.

Obecnie nasz serwer działa w jednym wątku, co oznacza, że ​​może obsługiwać tylko jedno
żądanie na raz. Przyjrzyjmy się, jak to może być problemem, symulując kilka
wolnych żądań. Następnie naprawimy to, aby nasz serwer mógł obsługiwać wiele żądań naraz.
