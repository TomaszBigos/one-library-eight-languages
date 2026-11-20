# Jak zrobić własną prezentację z tych danych

1. Daj agentowi AI cały katalog (albo link do repozytorium) i poproś, żeby najpierw przeczytał `README_FOR_AI.md`.
2. Powiedz, dla kogo jest prezentacja i ile ma mieć slajdów. Przykładowy prompt:

   > Przeczytaj README_FOR_AI.md i METHODOLOGY.md. Zrób 8 slajdów dla programistów Javy: który język wygrał
   > wydajnością i pamięcią, czego nauczył asembler, czy warto GPU. Każda liczba z pliku w data/ z podaniem źródła.
   > Zaznacz warunki pomiaru. Na końcu slajd „ograniczenia”.

3. Sprawdź liczby na slajdach z plikami w `data/` — agent ma je cytować, nie wymyślać.
4. `example_deck.html` w tym katalogu to przykładowa odpowiedź na prompt z punktu 2: 8 slajdów, otwórz w
   przeglądarce (druk do PDF = jeden slajd na stronę). Każda liczba jest policzona z plików `data/`, a każdy slajd
   podaje plik i kolumnę źródła oraz warunki pomiaru — tak powinien wyglądać wynik agenta. Treść i styl możesz
   zmienić dowolnie.
