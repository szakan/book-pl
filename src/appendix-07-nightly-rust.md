## Załącznik G - How Rust is Made and “Nightly Rust”

Ten dodatek dotyczy tego, jak powstaje Rust i jak to wpływa na Ciebie jako programistę Rust.

### Stability Without Stagnation

Jako język Rust *bardzo* dba o stabilność kodu. Chcemy, aby
Rust był solidnym fundamentem, na którym można budować, a gdyby rzeczy
ciągle się zmieniały, byłoby to niemożliwe. Jednocześnie, jeśli nie możemy
eksperymentować z nowymi funkcjami, możemy nie odkryć ważnych wad do czasu
ich wydania, kiedy nie będziemy już mogli niczego zmienić.

Naszym rozwiązaniem tego problemu jest to, co nazywamy „stabilnością bez stagnacji”,
a naszą przewodnią zasadą jest to: nigdy nie powinieneś obawiać się aktualizacji do
nowej wersji stabilnego Rust. Każda aktualizacja powinna być bezbolesna, ale powinna również
przynosić nowe funkcje, mniej błędów i szybsze czasy kompilacji.

### Choo, Choo! Release Channels and Riding the Trains

Rozwój Rust odbywa się według *harmonogramu*. Oznacza to, że cały rozwój odbywa się
na gałęzi `master` repozytorium Rust. Wydania są zgodne z modelem pociągu wydań oprogramowania, który był używany przez Cisco IOS i inne projekty oprogramowania. Istnieją trzy *kanały wydań* dla Rust:

* Nocny
* Beta
* Stabilny

Większość programistów Rust korzysta głównie ze stabilnego kanału, ale ci, którzy chcą
wypróbować nowe eksperymentalne funkcje, mogą użyć kanału nocnego lub beta.

Oto przykład, jak działa proces rozwoju i wydania:
załóżmy, że zespół Rust pracuje nad wydaniem Rust 1.5. Wydanie to
miało miejsce w grudniu 2015 r., ale dostarczy nam realistycznych
numerów wersji. Do Rust dodawana jest nowa funkcja: nowe zatwierdzenie ląduje na gałęzi `master`. Każdej nocy produkowana jest nowa nocna wersja Rust. Każdy dzień jest
dniem wydania, a te wydania są tworzone przez naszą infrastrukturę wydań
automatycznie. Tak więc w miarę upływu czasu nasze wydania wyglądają tak, raz na noc:

```text
nightly: * - - * - - *
```

Co sześć tygodni nadszedł czas na przygotowanie nowego wydania! Gałąź `beta` repozytorium Rust odgałęzia się od gałęzi `master` używanej przez nightly. Teraz,
są dwa wydania:

```text
nightly: * - - * - - *
                     |
beta:                *
```

Większość użytkowników Rust nie korzysta aktywnie z wersji beta, ale testuje je w systemie CI, aby pomóc Rust wykryć możliwe regresje. W międzyczasie,
co noc ukazuje się kolejna wersja:

```text
nightly: * - - * - - * - - * - - *
                     |
beta:                *
```

Załóżmy, że znaleziono regresję. Dobrze, że mieliśmy trochę czasu na przetestowanie wersji beta, zanim regresja wkradła się do stabilnej wersji! Poprawka jest stosowana
do `master`, więc nightly jest naprawione, a następnie poprawka jest przenoszona wstecznie do gałęzi
`beta` i produkowana jest nowa wersja beta:

```text
nightly: * - - * - - * - - * - - * - - *
                     |
beta:                * - - - - - - - - *
```

Sześć tygodni po stworzeniu pierwszej wersji beta nadszedł czas na wersję stabilną! Gałąź
`stable` jest produkowana z gałęzi `beta`:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |
beta:                * - - - - - - - - *
                                       |
stable:                                *
```

Hurra! Rust 1.5 jest gotowy! Jednak zapomnieliśmy o jednej rzeczy: ponieważ minęło sześć
tygodni, potrzebujemy również nowej wersji beta *następnej* wersji Rust, 1.6.
Więc po `stable` rozgałęzia się od `beta`, następna wersja `beta` rozgałęzia się
ponownie od `nightly`:
```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |                         |
beta:                * - - - - - - - - *       *
                                       |
stable:                                *
```

Nazywa się to „modelem pociągu”, ponieważ co sześć tygodni wydanie „opuszcza stację”, ale nadal musi odbyć podróż przez kanał beta, zanim
dotrze jako wydanie stabilne.

Wydania Rust są wydawane co sześć tygodni, jak w zegarku. Jeśli znasz datę jednego wydania Rust, możesz znać datę następnego: jest sześć tygodni później. Miłym aspektem planowania wydań co sześć tygodni jest to, że następny pociąg
nadchodzi wkrótce. Jeśli jakaś funkcja przypadkowo ominie konkretne wydanie, nie musisz się
martwić: kolejne pojawi się w niedługim czasie! Pomaga to zmniejszyć presję,
aby przemycić potencjalnie niedopracowane funkcje blisko terminu wydania.

Dzięki temu procesowi zawsze możesz sprawdzić kolejną kompilację Rust i
sam sprawdzić, czy łatwo jest ją uaktualnić: jeśli wydanie beta nie działa
zgodnie z oczekiwaniami, możesz zgłosić to zespołowi i naprawić przed wydaniem następnego stabilnego wydania! Wersje beta ulegają uszkodzeniu stosunkowo rzadko, ale
`rustc` to nadal oprogramowanie i błędy istnieją.

### Maintenance time

Projekt Rust obsługuje najnowszą stabilną wersję. Gdy zostanie wydana nowa stabilna
wersja, stara wersja osiągnie koniec swojego cyklu życia (EOL). Oznacza to, że
każda wersja jest obsługiwana przez sześć tygodni.

### Unstable Features

Jest jeszcze jeden haczyk w tym modelu wydania: niestabilne funkcje. Rust używa
techniki zwanej „flagami funkcji”, aby określić, które funkcje są włączone w
danej wersji. Jeśli nowa funkcja jest w trakcie aktywnego rozwoju, trafia do
`master`, a zatem do nightly, ale za *flagą funkcji*. Jeśli Ty, jako
użytkownik, chcesz wypróbować funkcję w toku, możesz, ale musisz
używac nightly release Rust i opatrzyć swój kod źródłowy adnotacją z
odpowiednią flagą, aby się zapisać.

Jeśli używasz wersji beta lub stabilnej Rust, nie możesz używać żadnych
flag funkcji. To jest klucz, który pozwala nam na praktyczne wykorzystanie nowych funkcji,
zanim ogłosimy je stabilnymi na zawsze. Ci, którzy chcą wybrać najnowocześniejsze
rozwiązania, mogą to zrobić, a ci, którzy chcą mieć niezawodne doświadczenie, mogą pozostać przy
stabilnym i mieć pewność, że ich kod się nie zepsuje. Stabilność bez stagnacji.

Ta książka zawiera tylko informacje o stabilnych funkcjach, ponieważ funkcje w trakcie prac wciąż się zmieniają i z pewnością będą się różnić od czasu, gdy ta książka została napisana, do momentu, gdy zostaną włączone w stabilnych kompilacjach. Dokumentację funkcji dostępnych tylko w nocy można znaleźć w Internecie.

### Rustup and the Role of Rust Nightly

Rustup ułatwia przełączanie się między różnymi kanałami wydań Rust, na
globalnie lub na podstawie projektu. Domyślnie będziesz mieć zainstalowaną stabilną wersję Rust. Aby
instalować co noc, na przykład:

```console
$ rustup toolchain install nightly
```

Możesz również zobaczyć wszystkie *łańcuchy narzędzi* (wersje Rust i powiązane
komponenty), które zainstalowałeś za pomocą `rustup`. Oto przykład na komputerze z systemem Windows jednego z Twoich autorów:

```powershell
> rustup toolchain list
stable-x86_64-pc-windows-msvc (default)
beta-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc
```

Jak widać, stabilny zestaw narzędzi jest domyślny. Większość użytkowników Rusta używa stabilnego
przez większość czasu. Możesz chcieć używać stabilnego przez większość czasu, ale użyj
nightly w konkretnym projekcie, ponieważ zależy Ci na najnowocześniejszej funkcji.
Aby to zrobić, możesz użyć `rustup override` w katalogu tego projektu, aby ustawić
nightly zestaw narzędzi jako ten, którego `rustup` powinien używać, gdy jesteś w tym katalogu:

```console
$ cd ~/projects/needs-nightly
$ rustup override set nightly
```

Teraz za każdym razem, gdy wywołasz `rustc` lub `cargo` wewnątrz
*~/projects/needs-nightly*, `rustup` upewni się, że używasz nightly
Rust, a nie domyślnego stabilnego Rust. Jest to przydatne, gdy masz
wiele projektów Rust!

### The RFC Process and Teams

Jak więc dowiedzieć się o tych nowych funkcjach? Model rozwoju Rust opiera się na
procesie *Request For Komentarze (RFC)*. Jeśli chcesz ulepszyć Rust,
możesz napisać propozycję zwaną RFC.

Każdy może napisać RFC, aby ulepszyć Rust, a propozycje są sprawdzane i
omawiane przez zespół Rust, który składa się z wielu podzespołów tematycznych. Istnieje
pełna lista zespołów [na
stronie internetowej Rust](https://www.rust-lang.org/governance), która obejmuje zespoły dla
każdego obszaru projektu: projektowania języka, implementacji kompilatora,
infrastruktury, dokumentacji i innych. Właściwy zespół czyta
propozycję i komentarze, pisze własne komentarze i ostatecznie
istnieje konsensus co do zaakceptowania lub odrzucenia funkcji.

Jeśli funkcja zostanie zaakceptowana, w repozytorium Rust otwierany jest problem i
ktoś może ją zaimplementować. Osoba, która bardzo dobrze ją zaimplementuje, może nie być osobą, która pierwotnie zaproponowała tę funkcję! Gdy implementacja jest gotowa, ląduje ona na gałęzi `master` za bramką funkcji, jak omawialiśmy
w sekcji [„Niestabilne funkcje”](#unstable-features)<!-- ignore -->.

Po pewnym czasie, gdy programiści Rust, którzy używają wydań nocnych, będą mogli
wypróbować nową funkcję, członkowie zespołu omówią funkcję, sposób, w jaki działa w trybie nocnym, i zdecydują, czy powinna ona zostać wprowadzona do stabilnej wersji Rust, czy nie.
Jeśli zapadnie decyzja o kontynuowaniu, bramka funkcji jest usuwana, a
funkcja jest teraz uważana za stabilną! Wjeżdża pociągami do nowej stabilnej wersji
Rust.
