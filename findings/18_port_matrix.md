# Port z języka na język: czy warto i z czego portować

> Tabela 56 par źródło → cel w `../data/port_matrix.csv`, wykres w `../charts.html`. Krotności czasu, pamięci
> i startu liczone z `../data/matrix.csv` (najlepsza wersja każdego języka, średnia geometryczna obu języków tekstu
> i obu serii). Trudność i ocena to reguły ułożone przez AI na podstawie ośmiu wykonanych portów — opinia
> z jawną regułą, nie pomiar.

## Jak czytać tabelę

- `speed_ratio`, `memory_ratio`, `start_ratio` — cel wobec źródła (0,72 = cel potrzebuje 72% czasu źródła).
- `difficulty` — w zależności od tego, czy cel jest na JVM i czy źródło spoza JVM:
  - **łatwy:** cel na JVM, bo dostaje zachowanie Javy za darmo;
  - **średni:** cel spoza JVM, a źródło spoza JVM już ma odtworzenie Javy (wyrażenia regularne, Unicode,
    normalizacja, generator liczb losowych, napisy UTF-16) do przeniesienia;
  - **trudny:** cel spoza JVM, a odtworzenie trzeba napisać od zera. O jeden stopień trudniej także przy przejściu
    z języka z GC do języka bez GC.
- `needs` — co dokładnie trzeba zrobić. `done_here` i `ai_hours_here` — czy ta para była wykonana w tym projekcie
  i ile godzin AI zajęła.
- `verdict`:
  - **warto: szybciej** — cel ma najwyżej 0,8 czasu źródła;
  - **warto: pamięć lub start** — najwyżej 0,75 pamięci albo 0,5 czasu startu;
  - **nie warto: wolniej** — co najmniej 1,25 czasu przy podobnej albo większej pamięci;
  - **tylko dla ekosystemu** — reszta, czyli bez zysku technicznego.
- `best_source` — z czego portować na dany cel: najłatwiej, przy tym samym modelu pamięci, z portu wcześniej
  sprawdzonego.

## Co wychodzi

- **Z JVM na Rust, C#, Go albo C++:** 0,58–0,75 czasu źródła, więc warto dla szybkości. To jednak port trudny:
  trzeba odtworzyć zachowanie Javy (`../data/port_matrix.csv`).
- **Wewnątrz czwórki Rust, C#, Go, C++:** krotności 0,95–1,05, czyli remis. Port między nimi ma sens tylko dla
  ekosystemu: zespołu, wdrożenia, bibliotek.
- **Z czwórki na JVM:** 1,33–1,73 czasu, więc dla szybkości nie warto (`../data/port_matrix.csv`).
- **Na TypeScript:** 4,8–8,3 razy dłużej, więc nie warto; z TypeScriptu na cokolwiek innego warto zawsze.
- **Wewnątrz JVM:** Java jest najszybsza (Kotlin → Java 0,88, Scala → Java 0,81), ale różnica nie przekracza progu,
  więc to wybór ekosystemu.
- **Najlepsze źródła według reguły:**
  - Kotlin i Scala z Javy;
  - Rust z C++ i C++ z Rusta (oba bez GC);
  - C# z Go i Go z C# (oba z GC, w obu jest już gotowe odtworzenie Javy).

  W tym projekcie poszło podobnie: C++ powstał z Rusta, Go z Rusta i C#, a TypeScript z C++. Rust był pierwszym
  portem spoza JVM, więc musiał powstać z Javy.

## Jak wybrać

1. **Najpierw cel, nie język:** czas, pamięć, start, praca pod tłokiem czy ekosystem (`10_ai_verdict.md`).
2. **Sprawdź parę w tabeli.** Jeśli ocena to „tylko dla ekosystemu”, port ma sens wyłącznie z powodów
   organizacyjnych.
3. **Portuj z implementacji, która jest najbliżej celu:** ma już odtworzoną semantykę referencji i ten sam model
   pamięci. Wtedy przepisujesz kod, a nie projektujesz go od nowa.
4. **Przed portem przygotuj bramkę bit w bit i testy referencyjne.** Bez nich nie wiadomo, czy szybszy port liczy
   to samo (`01_equivalence.md`).
5. **Zmierz w warunkach docelowych.** Kolejność pod tłokiem i w ciasnej pamięci jest inna niż na wolnej maszynie
   (`03_contention.md`).
