# Bezpieczna współbieżność

Bezpieczne i wydajne radzenie sobie z programowaniem współbieżnym to kolejny z głównych celów Rusta. *Programowanie współbieżne*, w którym różne części programu
wykonują się niezależnie, i *programowanie równoległe*, w którym różne części programu
wykonują się w tym samym czasie, stają się coraz ważniejsze, ponieważ coraz więcej
komputerów korzysta z wielu procesorów. Historycznie,
programowanie w tych kontekstach było trudne i podatne na błędy: Rust ma nadzieję
to zmienić.

Początkowo zespół Rusta uważał, że zapewnienie bezpieczeństwa pamięci i zapobieganie
problemom współbieżności to dwa oddzielne wyzwania, które należy rozwiązać różnymi
metodami. Z czasem zespół odkrył, że systemy własności i typów to
potężny zestaw narzędzi, który pomaga zarządzać bezpieczeństwem pamięci *i* problemami współbieżności! Dzięki wykorzystaniu własności i sprawdzania typów wiele błędów współbieżności
to błędy kompilacji w Ruście, a nie błędy czasu wykonania. Dlatego zamiast
musieć poświęcać dużo czasu na próby odtworzenia dokładnych okoliczności,
w których występuje błąd współbieżności w czasie wykonywania, niepoprawny kod nie będzie się
kompilował i wyświetli błąd wyjaśniający problem. W rezultacie możesz naprawić
swój kod podczas pracy nad nim, a nie potencjalnie po jego
dostarczeniu do produkcji. Ten aspekt Rust nazwaliśmy *bezstrachną*
*współbieżnością*. Bezstrachna współbieżność pozwala pisać kod, który jest wolny od
subtelnych błędów i łatwy do refaktoryzacji bez wprowadzania nowych błędów.

> Uwaga: dla uproszczenia będziemy odnosić się do wielu problemów jako
> *współbieżnych*, zamiast precyzyjniej mówić *współbieżnych i/lub
> równoległych*. Gdyby ta książka dotyczyła współbieżności i/lub paralelizmu, bylibyśmy
> bardziej konkretni. W tym rozdziale proszę w myślach zastępować *współbieżne
> i/lub równoległe* za każdym razem, gdy używamy *współbieżnych*.

Wiele języków jest dogmatycznych co do rozwiązań, które oferują do obsługi
problemów współbieżnych. Na przykład Erlang ma elegancką funkcjonalność dla
współbieżności przekazywania wiadomości, ale ma tylko niejasne sposoby na dzielenie stanu między
wątkami. Obsługa tylko podzbioru możliwych rozwiązań jest rozsądną
strategią dla języków wyższego poziomu, ponieważ język wyższego poziomu obiecuje
korzyści z oddania pewnej kontroli w celu uzyskania abstrakcji. Jednak oczekuje się, że języki niższego poziomu zapewnią rozwiązanie o najlepszej wydajności w każdej
sytuacji i będą miały mniej abstrakcji w stosunku do sprzętu. Dlatego Rust
oferuje różnorodne narzędzia do modelowania problemów w dowolny sposób, który jest odpowiedni
do Twojej sytuacji i wymagań.

Oto tematy, które omówimy w tym rozdziale:

* Jak tworzyć wątki, aby uruchamiać wiele fragmentów kodu w tym samym czasie
* *Współbieżność przekazywania wiadomości*, gdzie kanały wysyłają wiadomości między wątkami
* *Współbieżność współdzielonego stanu*, gdzie wiele wątków ma dostęp do pewnego fragmentu
danych
* Cechy `Sync` i `Send`, które rozszerzają gwarancje współbieżności Rust na
typy zdefiniowane przez użytkownika, a także typy dostarczane przez bibliotekę standardową
