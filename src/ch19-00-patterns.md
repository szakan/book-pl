# Wzory i dopasowywanie

*Wzory* to specjalna składnia w Rust służąca do dopasowywania do struktury
typów, zarówno złożonych, jak i prostych. Używanie wzorców w połączeniu z wyrażeniami `match`
i innymi konstrukcjami daje większą kontrolę nad przepływem sterowania programu. Wzorzec składa się z pewnej kombinacji następujących elementów:

* Literały
* Zdestrukturyzowane tablice, wyliczenia, struktury lub krotki
* Zmienne
* Symbole wieloznaczne
* Symbole zastępcze

Niektóre przykładowe wzorce obejmują `x`, `(a, 3)` i `Some(Color::Red)`. W
kontekstach, w których wzorce są prawidłowe, te komponenty opisują kształt
danych. Następnie nasz program dopasowuje wartości do wzorców, aby ustalić, czy
ma on prawidłowy kształt danych, aby kontynuować uruchamianie określonego fragmentu kodu.

Aby użyć wzorca, porównujemy go z jakąś wartością. Jeśli wzorzec pasuje do
wartości, używamy części wartości w naszym kodzie. Przypomnij sobie wyrażenia `match` w
rozdziale 6, które wykorzystywały wzorce, takie jak przykład maszyny sortującej monety. Jeśli
wartość pasuje do kształtu wzorca, możemy użyć nazwanych elementów. Jeśli
nie pasuje, kod powiązany ze wzorcem nie zostanie uruchomiony.

Ten rozdział jest odniesieniem do wszystkiego, co dotyczy wzorców. Omówimy
prawidłowe miejsca użycia wzorców, różnicę między wzorcami obalalnymi i nieobalalnymi oraz różne rodzaje składni wzorców, z którymi możesz się spotkać. Do
końca rozdziału będziesz wiedział, jak używać wzorców, aby wyrazić wiele pojęć w
jasny sposób.
