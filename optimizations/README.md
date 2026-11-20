# Optymalizacje per język

Co przyspieszyło, a co nie, w każdej z ośmiu implementacji, z krótkimi fragmentami kodu. Po każdej zmianie bramka
równoważności musiała pokazać 0 różnic w odpowiedziach (`../findings/01_equivalence.md`), a efekt mierzono A/B na tym
samym materiale (`../METHODOLOGY.md`). Liczby pojedynczych zmian pochodzą z dzienników optymalizacji i nie są w osobnym
pliku w `../data/`; stan końcowy (krotność czasu Javy, pamięć, start) jest w `../data/language_summary.csv`.

| plik | zawartość |
|---|---|
| `00_common.md` | zmiany algorytmiczne, które dostała każda implementacja (indeksy detektorów, indeks literówek, prefiltry) |
| `01_java.md` | Java (HotSpot, obraz natywny GraalVM): GC, flagi JVM, kompilator C2 |
| `02_kotlin.md` | Kotlin: lambdy, `inline`, tablice prymitywów, jedna kompilacja C2 |
| `03_scala.md` | Scala: domknięcia, `IntRef`, kolekcje Scali |
| `04_rust.md` | Rust: alokacje, alokator, toolchain, PGO, arena |
| `05_csharp.md` | C#: GC, kompilacja warstwowa, dynamiczne PGO, NativeAOT, SIMD |
| `06_go.md` | Go: GC bez generacji, analiza ucieczki, mapy bez alokacji |
| `07_cpp.md` | C++: aliasing wskaźników, inicjalizacja statycznych, PGO MSVC |
| `08_typescript.md` | TypeScript: tablice typowane, klucze napisowe w V8, limit sterty, workery |

Jak czytać:

- **Krotność** (×0,70) = czas po zmianie podzielony przez czas przed nią; mniej znaczy szybciej. Procent przy nazwie
  etapu (np. „etap wzorców −67%”) dotyczy tylko tego etapu, nie całego dokumentu.
- **Kod** to fragmenty bibliotek skrócone do istoty techniki: pominięte linie oznacza `...`, komentarze są przetłumaczone
  i skrócone. Wersja „przed” pochodzi z implementacji referencyjnej zachowanej w testach albo z opisu w dzienniku; wtedy
  jest podpisana jako schemat.
- **Odrzucone** znaczy: zaimplementowane, zmierzone i cofnięte, bo zysk był w szumie albo poniżej progu. Ten sam pomysł
  bywał odrzucony w jednym języku i przyjęty w innym, a raz odrzucony w Javie wrócił po innych zmianach
  (`00_common.md`).
- Maszyna nie była izolowana (`../MACHINE.md`): pojedyncze zmiany poniżej ok. 10% rozstrzygano udziałem etapu w profilu
  i wyłącznikiem na tym samym buildzie, nie samym czasem.
