# Optymalizacje per język

> Liczby optymalizacji pochodzą z dzienników portów (`../data/ai_work.csv`); dzienniki zapisują rundy w różnej
> formie, więc to wartości z dokładnością do kilku pozycji. Efekty pojedynczych zmian to pomiary A/B z dzienników
> (ta sama binarka albo stara i nowa w przeplocie, `../METHODOLOGY.md`) i nie występują w osobnym pliku w `data/`.
> Każdy język osobno, z odrzuconymi pomysłami i przykładami kodu: `../optimizations/`.

## Ile ich było

| język | numeracja | wdrożone własne | odrzucone własne | wspólne JVM (wdrożone / odrzucone) | rundy własne |
|---|---|---|---|---|---|
| Java | #0–#35 | 32 | 4 | 38 / 8 | 7 |
| Kotlin | K1–K6 | 5 | 1 | 38 / 8 | 4 |
| Scala | S1–S7, s9 | 8 | 0 | 38 / 8 | 4 |
| Rust | R1–R50 | 44 | 5 | — | 8 |
| C# | C1–C16 | ok. 36 | ok. 7 | — | 13 |
| Go | G1–G19, W1 | 19 | 1 | — | 5 |
| C++ | C1–C10 | 10 | 0 | — | 2 |
| TypeScript | T1–T9 | 9 | 0 | — | 1 |

Razem ok. 160 wdrożonych optymalizacji własnych i 38 wspólnych dla trzech języków JVM, przy ok. 26 odrzuconych
pomiarem (`../data/ai_work.csv`). Wspólne rundy szły zawsze w kolejności Java → Kotlin → Scala. Późniejsze porty
dostawały gotową listę: dziewięć z dziesięciu zmian C++ i większość zmian TypeScriptu to optymalizacje wcześniej
sprawdzone w Javie albo Ruście, dlatego skończyły z najmniejszą liczbą rund. Część pomysłów szła w drugą stronę: z Go do C# i Rusta,
z C# do Rusta, z Rusta i C# z powrotem do JVM (`09_ai_work.md`). Większość optymalizacji była algorytmiczna (indeksy
detektorów, indeks literówek, prefiltry wzorców) i dawała podobny zysk w każdym języku. Niżej zmiany, które działały
tylko dzięki cechom danego języka albo jego środowiska.

## Co pomagało tylko w danym języku

- **Java (HotSpot).**
  - Serial GC zamiast domyślnego G1: długie dokumenty −14…20% czasu w polskim, −22…44% w malgaskim, przy szczycie
    pamięci do 476 MB.
  - Podział dużej metody, której kompilator C2 nie kompilował (zostawała na poziomie C1): CPU na dokument
    w malgaskim −45% przy identycznym wyniku.
  - Jawny początkowy rozmiar sterty, gdy JDK 27 zmienił domyślny: szczyt pamięci zimnego startu w polskim
    510 → 446 MB, młodych GC 166 → 39.
  - Obraz natywny GraalVM ma sens dopiero z PGO: 1,16–1,39 czasu Javy bez profilu, 0,91–0,98 z profilem
    (`02_speed_memory.md`).
- **Kotlin.**
  - Szczyt pamięci w malgaskim był dwumodalny (350 albo 510 MB przy tej samej żywej stercie), bo jedna kompilacja C2
    dużej metody brała dużą arenę; pomógł podział metody.
  - Tablica znaków zamiast pudełkowanych znaków w zamianie homoglifów.
- **Scala.**
  - `ArraySeq` i pętle zamiast `Vector` i krotek w ścieżce dokumentu: malgaski z 1,49 do 1,19 czasu Javy.
  - W drugą stronę decyzja „tylko kolekcje Scali, bez kolekcji Javy”: współbieżna mapa Scali w cache werdyktów
    kosztowała w malgaskim ok. +5% czasu i +9% CPU.
- **Rust.**
  - Toolchain MSVC, alokator mimalloc, kod pod x86-64-v3 i PGO (razem z rundą zmian kodu): polski 0,70, malgaski
    0,73 czasu sprzed tej rundy. W macierzy końcowej PGO już nic nie zmieniało (`02_speed_memory.md`).
  - Słownik słów obcych w jednej arenie zamiast osobnych napisów: pamięć procesu w polskim −65…70 MB.
  - Zegar etapów bez przeliczania jednostek przy każdym odczycie: polski 0,89 czasu.
  - Odrzucone: LTO (wynik w szumie, budowanie dwa razy dłuższe) i własna tablica haszy zamiast tej z biblioteki
    standardowej (wolniej w obu językach tekstu).
- **C#.**
  - GC na segmentach zamiast regionów: szczyt pamięci w polskim ~490–500 → 397–418 MB, w malgaskim ~570–595 →
    456–491 MB, czas w szumie.
  - NativeAOT z preferencją szybkości i kodem pod x86-64-v3: 0,91 (polski) i 0,88 (malgaski) czasu domyślnego obrazu.
  - Opóźnienie liczenia wywołań w kompilacji warstwowej ustawione na 0. Na jednym procesorze ciągła praca odsuwała
    rekompilację w nieskończoność, a metody zostawały na najniższej warstwie; po zmianie przepustowość wzrosła
    2,5 raza (polski 42 → 106 dok/s), na pełnej maszynie bez zmian.
  - Interner tokenów odrzucony: zysk 1,5–1,7% dokumentu, poniżej progu.
- **Go.**
  - Prosty port spędzał 35–40% CPU w odśmiecaniu. Detektory w jednym buforze bez wskaźników zmniejszyły żywą stertę
    z ok. 300 do 140–150 MB.
  - Wyszukiwanie w mapie bez tworzenia napisu-klucza i interner tokenów: mapa w tokenizerze malgaskim 380 → 190 ms.
  - `GOGC` 100–800 nie zmieniał nic przed tymi zmianami (problemem był koszt cyklu, nie ich liczba); po nich
    200–400 dawało 0,91–0,93 czasu na wolnej maszynie, ale nie pod tłokiem, więc zostało domyślne.
  - PGO pomogło w malgaskim (0,69–0,70 wobec 0,83 bez, `02_speed_memory.md`). Filtr Blooma odrzucony.
- **C++.**
  - Poprawka pętli, w której kompilator po każdym zapisie od nowa czytał wskaźnik danych wektora, bo nie umiał
    dowieść, że zapis go nie zmienia; wykazał to dopiero listing asemblera (`04_assembler_simd.md`).
  - PGO MSVC i tablica haszująca z adresowaniem otwartym zamiast standardowej.
- **TypeScript.**
  - Detektory w jednej tablicy `Uint16Array` zamiast milionów napisów: sterta w malgaskim 190 → 46 MB.
  - Limit sterty 256 MB: szczyt pamięci w malgaskim 1133 → 465 MB, w polskim 1054 → 451 MB, przy czasie dłuższym
    o kilka procent. Bez limitu leniwe odśmiecanie V8 trzymało ok. 1 GB.
  - Tablice typowane zamiast `Map`, bez `BigInt` i bez sklejanych kluczy napisowych w gorącej ścieżce.

**Wniosek:** największe zyski w każdym języku dały te same zmiany algorytmiczne. Cechy języka decydowały
o pamięci i o pojedynczych pułapkach: ustawieniach GC, kompilacji JIT na małej maszynie, układzie danych pod
nieprzesuwający GC Go, limicie sterty V8.
