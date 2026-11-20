# C++

> Port jako biblioteka nagłówków (MSVC, C++23), jedna zależność: tablica haszy `ankerl::unordered_dense`. Pierwsza wersja
> powstała celowo bez optymalizacji; potem dostała listę z Javy i Rusta: dziewięć z dziesięciu zmian C1–C10 to
> optymalizacje wcześniej sprawdzone gdzie indziej. Własne były drobne, ale pouczające: aliasing i statyczne zmienne
> lokalne.

## Droga

| etap | konstytucje mg / pl, jeden wątek (Rust PGO w tym samym czasie: 1,6 / 2,5 s) |
|---|---|
| pierwsza wersja (bez indeksu puli, segmentów literówek i prefiltrów, kopia słowa na każdym kroku) | 30,6 / 54,2 s |
| po C2 | 3,4 / 6,9 s |
| po C3 | — / 4,8 s |
| po C5–C6 | 2,8 / 4,0 s |
| macierz końcowa (po C10, PGO, wymianie alokatora; `../data/language_summary.csv`) | ×0,75 czasu Javy (Rust ×0,72); szczyt RSS 295 MB, start 1,2 s |

## Co pomogło

| zmiana | technika | efekt |
|---|---|---|
| C1–C3, C6–C8 | indeks puli, wzorce pl z prefiltrem literałów, segmenty literówek, prefiltr wzorców mg, marki po paśmie długości, tablica łuków PoliMorf | patrz „Droga” |
| C4 | tablice Unicode przez wskaźnik atomowy zamiast statycznej zmiennej lokalnej funkcji: strażnik jej inicjalizacji to odczyt pamięci wątku przy KAŻDYM znaku | — |
| C5 | morfologia mg bez napisu na wariant i bez `std::function` (bufory wielokrotnego użytku) | (w „Drodze”) |
| C9 | mapy zadań i memo dokumentu z kluczami-widokami na słowa tokenów (`ViewMap`) zamiast kopii każdego różnego słowa | — |
| C10 | `distance_within` ze wskaźnikiem danych w zmiennej lokalnej i poprzednią komórką w zmiennej; znalezione w listingu asemblera | mediany 0,89 i 0,84 czasu pierwszej formy w dwóch seriach mikrobenchmarku (`../data/kernels.csv`) |
| tablica haszy | `ankerl::unordered_dense` (adresowanie otwarte) zamiast `std::unordered_map` (węzeł na wpis) | (`../findings/11_optimizations.md`) |
| PGO MSVC | `/GENPROFILE` → trening na próbkach → `/USEPROFILE` | przepustowość +3%; czas serwera mg +4% (etap wzorców) |

## Co nie pomogło albo zaszkodziło

| zmiana | dlaczego | pomiar |
|---|---|---|
| H1: jądro odległości edycyjnej w MASM | mediany 0,93 i 0,70 czasu pierwszej formy C++, w najszybszych rundach tyle co C10; osobnej funkcji asemblerowej nie da się wpleść w pętlę wywołującą | wyłączone domyślnie (`../findings/04_assembler_simd.md`) |
| mimalloc 2.1.7 | harness tworzył wątek konsumenta na przejście, a ta wersja nie odzyskiwała segmentów porzuconych przez zakończone wątki | ok. 13 MB na przejście, proces padał po ok. 35 min; wymiana na 3.3.2 |
| profiler próbkujący, pierwsza wersja | zatrzymany wątek trzymał blokadę sterty, a profiler alokował | zakleszczenie; teraz w czasie zatrzymania tylko rozwijanie stosu do bufora przydzielonego z góry |

## Przykład: kompilator nie wie, że zapis nie zmienia wskaźnika (C10)

Przed: `dp` to `std::vector`. Po każdym zapisie `dp[j] = …` MSVC od nowa czytał wskaźnik danych wektora, bo nie umiał
dowieść, że zapis do elementu nie zmienia samego obiektu wektora, a `dp[j - 1]` czytał z pamięci zamiast z rejestru.

```cpp
for (size_t i = 1; i <= m; i++) {
    size_t prev = dp[0];
    dp[0] = i;
    size_t row_min = dp[0];
    for (size_t j = 1; j <= n; j++) {
        size_t temp = dp[j];
        dp[j] = a[i - 1] == b[j - 1] ? prev : 1 + std::min(prev, std::min(dp[j], dp[j - 1]));
        prev = temp;
        row_min = std::min(row_min, dp[j]);
    }
    if (row_min > max_distance) return max_distance + 1;
}
```

Po: surowy wskaźnik w zmiennej lokalnej, lewy sąsiad w zmiennej `left`, znak słowa `a` czytany raz na wiersz.

```cpp
size_t* dp = dpv.data();
const char16_t* pa = a.data();
const char16_t* pb = b.data();
for (size_t j = 0; j <= n; j++) dp[j] = j;
for (size_t i = 1; i <= m; i++) {
    size_t prev = dp[0], left = i, row_min = i;
    dp[0] = i;
    char16_t ca = pa[i - 1];
    for (size_t j = 1; j <= n; j++) {
        size_t temp = dp[j];
        size_t best = std::min(prev, std::min(temp, left)) + 1;
        size_t v = ca == pb[j - 1] ? prev : best;
        dp[j] = v;
        left = v;
        prev = temp;
        row_min = std::min(row_min, v);
    }
    if (row_min > max_distance) return max_distance + 1;
}
return dp[n];
```

W najszybszych rundach ręczne jądro w asemblerze miało 101 ns na parę, ta wersja 102 i 117 ns (dwie serie). Asembler
posłużył głównie do diagnozy: pokazał, czego kompilator nie zrobił.

## Techniki w skrócie

- Gorąca pętla po `std::vector` z zapisami: wskaźnik `data()` do zmiennej lokalnej; wartości z poprzedniej iteracji
  w zmiennych, nie w pamięci. Sprawdzać listing (`/FAs`), nie zgadywać.
- Statyczna zmienna lokalna funkcji ma strażnika inicjalizacji przy każdym wywołaniu; w pętli po znakach to widać.
- Widoki na słowa zamiast kopii w mapach; napis źródłowy trzymać w zmiennej, bo MSVC nie wydłuża
  życia tymczasowych w pętli zakresowej.
- Standardowy kontener i standardowy alokator nie zawsze są dobrym wyborem domyślnym, ale biblioteka zewnętrzna też ma
  wersje z błędami: wyciek mimalloc 2.1.7 przy wątku na przejście.
