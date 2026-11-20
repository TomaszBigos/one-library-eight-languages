# Jaki język zaprojektowałbym po ośmiu portach

> Projekt na papierze, bez kompilatora. Opinia AI, które napisało wszystkie porty: dziesięć mechanizmów, których
> brakowało we wszystkich ośmiu językach, każdy z dowodem z tego katalogu. Ocena istniejących języków pod kątem
> pracy z tekstem: `23_text_languages.md`.

## Dziesięć mechanizmów

| # | mechanizm | co kosztowało w projekcie | źródło |
|---|---|---|---|
| 1 | **tekst z jawną jednostką**: pozycja w bajtach, w jednostkach UTF-16 i w znakach jako różne typy | porty spoza JVM musiały odtworzyć pozycje UTF-16 Javy, bo tak podaje je raport | `09_ai_work.md` |
| 2 | **wersjonowana semantyka tekstu w bibliotece standardowej**: wyrażenia regularne, NFKC, właściwości znaków, zmiana wielkości liter bez ustawień regionalnych | silnik regex napisany od nowa w Ruście i przeniesiony do czterech kolejnych języków, NFKC z danych JDK; pod tureckimi ustawieniami „Ignoruj” stawało się „ıgnoruj” | `09_ai_work.md`, `14_curiosities.md` |
| 3 | **determinizm domyślnie**: kolekcje z ustaloną kolejnością iteracji, remisy rozstrzygane jawnym kluczem | dwie kolejności zależne od iteracji kolekcji, remis inny przy każdym starcie, ranking zależny od liczby rdzeni | `09_ai_work.md`, `../ARCHITECTURE.md` |
| 4 | **płaskie tablice wartości bez nagłówków obiektów** | detektory w jednej tablicy zamiast napisu na detektor: żywa sterta Javy 116 / 140 MB, w Go ok. 300 → 140–150 MB, w TypeScripcie 190 → 46 MB | `../optimizations/00_common.md`, `06_go.md`, `08_typescript.md` |
| 5 | **maski bitowe i wektory jako zwykłe operacje** | mapa bitowa par Hamminga: −35…65% czasu; Hamming na wektorach 0,19–0,31 czasu pętli skalarnej w mikrobenchmarku, pisany osobno w każdym języku | `../optimizations/00_common.md`, `04_assembler_simd.md` |
| 6 | **wysoki poziom bez kosztu**: pętle wyższego rzędu kompilowane gwarantowanie do zwykłych pętli, bez opakowanych liczb i domknięć na stercie | Scala idiomatyczna ×4,6–5,3 czasu Javy, po przepisaniu na pętle ×1,24–1,62 | `../optimizations/03_scala.md` |
| 7 | **pamięć na czas jednego dokumentu** (arena albo region) | słownik w arenie w Ruście: szczyt pamięci pl −65…70 MB; interner w Go; `Span` i `stackalloc` w C# — trzy drogi do tego samego | `../optimizations/04_rust.md`, `06_go.md`, `05_csharp.md` |
| 8 | **budżet pamięci jako parametr programu**, nie strojenie każdego środowiska osobno | TypeScript bez limitu sterty trzymał ok. 1 GB, z limitem 451–465 MB; C# (polski) na regionach 490–500 MB, na segmentach 397–418 MB; Java po zmianie JDK 510 MB, z jawną stertą początkową 446 MB | `../optimizations/08_typescript.md`, `05_csharp.md`, `01_java.md` |
| 9 | **wbudowana kontrola „nowa ścieżka = stara”** z wyłącznikiem każdej optymalizacji | każda optymalizacja miała wyłącznik i test równoważności, pisane osobno w każdym języku; bramka 25 850 raportów, fuzzing | `../TESTING.md`, `../optimizations/README.md` |
| 10 | **tani zegar monotoniczny i liczniki etapów** w bibliotece standardowej | zegar Go na Windows z krokiem ok. 1 ms; w Ruście zegar etapów bez przeliczania jednostek dał polskiemu pipeline'owi ×0,89 czasu | `14_curiosities.md`, `../optimizations/04_rust.md` |

## Jak mogłoby to wyglądać

Pseudoskładnia, tylko ilustracja mechanizmów; żaden kompilator jej nie zna.

```
text Doc = Text<Utf16>                         // 1: jednostka pozycji w typie
                                               // 2: regex, NFKC, lower() bez locale, wersja Unicode w manifeście

packed struct Detector { code: [u8; 1..=7] }   // 4: pakowane w jedną tablicę, bez nagłówka obiektu
type Mask = bits<1024>                          // 5: maski i operacje na nich jak na liczbach

fn classify(doc: Doc) -> Report
    budget memory 400 MB                       // 8: limit programu, nie flaga środowiska
{
    arena per call {                           // 7: wszystko z tego bloku znika naraz
        let words = doc.tokens().distinct()    // 3: distinct() w kolejności pierwszego wystąpienia
        for w in words { ... }                 // 6: kompiluje się do pętli bez alokacji albo błąd kompilacji
    }
    stage "patterns" { ... }                   // 10: czas etapu w raporcie, zegar monotoniczny
}

@equivalent(to: classify_reference, inputs: fuzz(seed: 1, docs: 3000))   // 9
@switch("nsa.pool-index")                                               // 9: wyłącznik ścieżki
```

## Co już istnieje

Żaden z ośmiu języków nie łączy wszystkich dziesięciu, ale większość mechanizmów gdzieś już jest:

- **Rust**: abstrakcje bez kosztu (6), brak GC i jawne zarządzanie pamięcią (7, 8), maski i wektory (5); napisy
  UTF-8, więc punkt 1 i pozycje UTF-16 trzeba było dopisać (`../optimizations/04_rust.md`).
- **C#**: napisy UTF-16 (1), `Span`, `stackalloc` i `Vector128` (5, 7), struktury wartości (4); GC trzeba było
  stroić, żeby trzymać pamięć (8, `../optimizations/05_csharp.md`).
- **Java**: zgodność semantyki tekstu jest tu z definicji (2), bo to jej biblioteka wyznaczyła specyfikację;
  kierunek jej rozwoju (typy wartości bez nagłówków, operacje wektorowe) dotyczy punktów 4 i 5.
- **Zig** (nie mierzony w tym projekcie): jawne alokatory i obliczenia w czasie kompilacji pasują do punktów 7 i 8.

Najtrudniejsze są punkty 1–3, a one nie są sprawą składni, tylko biblioteki standardowej i jej specyfikacji.

## Wniosek

Nowy język nie jest potrzebny. Brakuje wspólnej, wersjonowanej specyfikacji tekstu (punkty 1–3) dostępnej z każdego
języka: jednego jądra (tokenizacja, pozycje UTF-16, podzbiór wyrażeń regularnych, NFKC, zmiana wielkości liter)
napisanego raz i wołanego przez interfejs C. Projekt sprawdził już taki układ: Python wołał bibliotekę Rusta przez
interfejs C i dał 0 różnic wobec Javy na 25 850 raportach bramki i 31 696 z fuzzingu w każdym pipeline
(`../data/equivalence.csv`, `08_python_wasm_long_run.md`). Pozostałe mechanizmy to wybór języka
(Rust, C#) albo dyscyplina w kodzie. Kompilator nowego języka w wersji minimalnej to miesiące pracy, a biblioteki,
narzędzia i użytkownicy, którzy decydują o jego wartości, to lata; AI przyspiesza pierwsze, drugiego nie.
