# Kto łapie błąd: kompilator czy bramka

> Przykłady z dzienników portów; bez liczb wydajności.

W porcie C++ wiszący widok na tymczasowy napis kompilował się bez ostrzeżeń, a zepsuł kody języków w 2658 raportach
dokumentów — złapała go dopiero bramka równoważności; ten sam kod w Ruście nie skompilowałby się. W hoście GPU w Ruście
kompilator odrzucił domknięcie trzymające wyłączną referencję do bufora, który potem był czytany — w C++, Javie, Go,
C# i TypeScripcie ten sam kod przeszedłby kompilację. W TypeScripcie błąd kolejności obliczeń złapał test jednostkowy,
a w C++ problem wydajności (przeładowywanie wskaźnika po każdym zapisie) pokazał dopiero listing asemblera. Wniosek:
Rust przenosi część błędów pamięci i współdzielenia z czasu działania do czasu kompilacji; w pozostałych językach tę
rolę pełnią bramki i testy — dlatego bramka równoważności była obowiązkowa dla każdego portu, niezależnie od języka.
