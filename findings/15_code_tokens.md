# Ile tokenów zajmuje kod w kontekście AI

> Liczba znaków na token jest skalibrowana na zapisach sesji AI, bez wysyłania kodu na zewnątrz
> (`../data/code_tokens.csv`, metoda w `../data/README.md`). To przybliżenie z dokładnością ok. ±10%.

## Kalibracja

W 33 miejscach zapisu sesji AI wczytało jeden plik i od razu wysłało następne zapytanie. Przyrost kontekstu minus
tokeny poprzedniej odpowiedzi to liczba tokenów tego pliku. Wychodzi ok. 2,6 znaku na token w kodzie (kwartyle
2,4–2,65; 22 próbki) i ok. 1,9 w polskim tekście Markdown, bo polskie litery z ogonkami kosztują więcej tokenów.
Próbek jest za mało, żeby rozróżnić języki programowania, więc dla całego kodu przyjęto jedną wartość
(`../data/code_tokens.csv`).

## Wynik

| implementacja | biblioteka | z testami | okno 200 tys. (z testami) |
|---|---|---|---|
| TypeScript | 73 tys. | 81 tys. | 40% |
| Go | 103 tys. | 117 tys. | 59% |
| Scala | 112 tys. | 163 tys. | 82% |
| C++ | 115 tys. | 131 tys. | 65% |
| Kotlin | 121 tys. | 186 tys. | 93% |
| C# | 124 tys. | 134 tys. | 67% |
| Rust | 162 tys. | 179 tys. | 89% |
| Java | 214 tys. | 348 tys. | 174% |

(`../data/code_tokens.csv`)

Każda biblioteka oprócz Javy mieści się w oknie 200 tys. tokenów. Java nie mieści się nawet bez testów, bo to
oryginał z najdłuższymi komentarzami i największym zestawem testów. Wszystkie osiem implementacji to ok. 1,02 mln
tokenów bibliotek i 1,34 mln z testami, czyli więcej niż okno 1 mln (`../data/code_tokens.csv`). Liczba tokenów nie
idzie za liczbą linii: Go ma najwięcej linii kodu (`../data/port_cost.csv`), ale krótkich, więc w tokenach jest
drugi od końca.

Cały materiał `publish/` w chwili eksportu to ok. 51 tys. tokenów tekstu i ok. 74 tys. tokenów danych CSV, razem
ok. 125 tys. Mieści się więc w oknie 200 tys. i da się go w całości przedstawić modelowi AI
(`../data/code_tokens.csv`).

## Koszt jednego wczytania

Przy przykładowej stawce 3 $ za milion tokenów wejścia wczytanie jednej biblioteki kosztuje 0,22–0,64 $, a całego
`publish/` ok. 0,38 $. Kolumny `usd_per_load_at_1/3/15` podają to samo przy 1, 3 i 15 $ za milion
(`../data/code_tokens.csv`). To stawki przykładowe, nie cennik konkretnego modelu. Treść czytaną ponownie z pamięci
podręcznej modelu zwykle rozlicza się taniej.

Dla porównania: przez całą pracę nad portami model przeczytał ok. 3,07 mld tokenów kontekstu (`../data/ai_scale.csv`).
To tyle, co ok. 2300 wczytań wszystkich ośmiu implementacji z testami, bo przy każdej odpowiedzi model czyta od nowa
całą dotychczasową rozmowę.

**Wniosek:** jedną bibliotekę AI może mieć w kontekście w całości, ale wszystkich implementacji naraz już nie. Praca
nad portami szła więc plik po pliku, z dziennikami i bramką zamiast pamięci całego kodu.
