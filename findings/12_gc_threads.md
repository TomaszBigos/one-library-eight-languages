# Pamięć, odśmiecanie i wątki w każdym języku

> Ustawienia i ich uzasadnienia pochodzą z dzienników portów; szczyty pamięci i przepustowość — z plików w `data/`.

## Pamięć i odśmiecanie

| implementacja | zarządzanie pamięcią | użyte ustawienia | dlaczego |
|---|---|---|---|
| Java, Kotlin, Scala (HotSpot) | GC generacyjny, Serial | sterta do 384 MB, kompaktowe nagłówki obiektów, oddawanie wolnej sterty; w polskim także początkowa 384 MB | najkrótszy czas przy szczycie pamięci poniżej 500 MB |
| obrazy natywne GraalVM | Serial GC obrazu | jak na JVM | — |
| OpenJ9 | domyślny GC generacyjny | ta sama sterta 384 MB | porównanie dwóch JVM |
| C# JIT | GC .NET na segmentach, sterta dużych obiektów od 16 MB | segmenty zamiast domyślnych regionów | regiony trzymały pamięć ponad 500 MB |
| C# NativeAOT | GC .NET na regionach | duże obiekty od 1 MB, ConserveMemory=5 | mniej pamięci pod twardym limitem |
| Go | współbieżny, niegeneracyjny, nieprzesuwający | `GOGC` domyślne, dane bez wskaźników | koszt GC rośnie z liczbą wskaźników |
| Rust | bez GC (własność, zwalnianie deterministyczne) | alokator mimalloc, areny | — |
| C++ | bez GC (RAII) | statyczna biblioteka uruchomieniowa, tablice haszujące z adresowaniem otwartym, alokator mimalloc | w godzinie pracy wersja 3.3.2: 2.1.7 nie odzyskiwała pamięci wątków tworzonych na każde przejście (`08_python_wasm_long_run.md`) |
| TypeScript (Node) | GC generacyjny V8 | limit sterty 256 MB | bez limitu leniwy GC trzymał ok. 1 GB |
| TypeScript (Bun) | GC JavaScriptCore | brak limitu sterty w silniku | — |
| Python → Rust | zliczanie referencji w Pythonie, Rust bez GC | — | — |

Szczyt pamięci na 341 tekstach: obrazy natywne JVM 180–221 MB, Rust i Go 212–263 MB, C++ 281–310 MB, Java JIT
318–362 MB, Bun do 601 MB (`02_speed_memory.md`, `../data/matrix.csv`). Pod twardym limitem 200 MB Java JIT spada do
38–49 dok/s (82–102 przy 300 MB), obraz natywny Javy trzyma 73–78 dok/s, a Rust 146–162 (`03_contention.md`). OpenJ9
przy tej samej stercie ma wyższy szczyt pamięci procesu niż HotSpot (`08_python_wasm_long_run.md`). Przez godzinę pracy
pamięć żadnej z mierzonych implementacji nie rośnie (C++ dopiero po zmianie wersji alokatora) (`../data/endurance_runs.csv`); C# JIT w malgaskim waha się przy tym
między 372 a 531 MB, ze szczytem 547 MB, a Java JIT w malgaskim między 284 a 433 MB, obie bez trendu
(`08_python_wasm_long_run.md`).

Ta sama Java na czterech odśmiecaczach (zmieniona tylko flaga GC, 3 rundy w przeplocie; Shenandoah nie wchodzi
w skład użytego JDK) (`../data/gc.csv`):

| odśmiecacz | p99 przy 60 dok/s, pl / mg | dok/s w zamkniętej pętli, pl / mg | CPU na dokument, pl / mg | szczyt pamięci, pl / mg |
|---|---:|---:|---:|---:|
| Serial (produkcja) | 55 / 33 ms | 110,5 / 131,5 | 14,1 / 12,9 ms | 413 / 386 MB |
| G1 | 57 / 36 ms | 105,0 / 127,7 | 16,2 / 13,2 ms | 502 / 449 MB |
| Parallel | 73 / 51 ms | 111,0 / 139,8 | 16,2 / 11,2 ms | 453 / 398 MB |
| ZGC | 71 / 45 ms | 105,8 / 123,7 | 15,7 / 12,6 ms | 503 / 494 MB |

Serial daje najniższe p99 w strumieniu i najmniejszą pamięć w obu pipeline'ach; Parallel ma najwyższą
przepustowość w malgaskim (+6% wobec Serial), ale p99 o 33–55% wyższe; G1 i ZGC dochodzą w polskim do sufitu
500 MB. CPU na dokument (czas wszystkich wątków procesu) rośnie w polskim przy G1 i Parallel o ok. 2 ms wobec
Serial, a w malgaskim przy Parallel spada o 1,7 ms. Dla tego obciążenia (sterta 384 MB) żaden z kolektorów
równoległych ani współbieżnych nie skraca ogona opóźnienia wobec Serial.

Wniosek: w językach z GC pamięć jest ustawieniem, nie stałą. Każde środowisko wymagało decyzji (rodzaj GC, limit
sterty, układ danych), a ustawienia domyślne potrafiły przekroczyć budżet (`09_ai_work.md`). W Ruście i C++ tej decyzji
nie ma, jest za to odpowiedzialność za czas życia obiektów (`07_compiler_vs_gate.md`).

## Wątki

Biblioteka dzieli długi dokument na słowa i liczy je równolegle; dokument krótszy niż 256 słów liczy jeden wątek.

| implementacja | jak liczy długi dokument | uwagi |
|---|---|---|
| Java, Kotlin | wspólna pula ForkJoin JVM | własna pula dokumentów odrzucona pomiarem; w malgaskim krótkie kategorie liczone jednym wątkiem (lekcja z Rusta), w polskim nie, bo pod tłokiem jeden wątek przegrywał |
| Scala | globalny kontekst wykonania Scali | tylko kolekcje i mechanizmy Scali (decyzja) |
| Rust | rayon (polski), własna pula na zmiennej warunkowej (malgaski) | wątek wywołujący liczy swoją porcję (lekcja z C#) |
| C# | własny podział pracy na puli wątków .NET, porcje po 8 | wątek wywołujący pracuje; kompilacja warstwowa na jednym procesorze wymagała zmiany ustawienia (`11_optimizations.md`) |
| Go | gorutyny z progiem i porcjami | w profilu widać koszt planisty: budzenie gorutyn i oddawanie procesora |
| C++ | własna pula, porcje po 4 | wątek wywołujący liczy |
| TypeScript | jeden wątek | workery opcjonalne, każdy z własną kopią danych (ok. 240 MB) |
| Python → Rust | wątki Pythona | blokada interpretera (GIL) |
| WebAssembly | jeden wątek | piaskownica bez wątków |

Na jednym wątku prowadzą Rust, C++ i Go (116–165 dok/s), Java JIT ma 78–103 (`03_contention.md`). Gdy inny proces
zajmuje procesor, Go zachowuje 29–33% własnej przepustowości, pozostałe implementacje 44–61% (`03_contention.md`).
Python na czterech wątkach daje tylko 1,26–1,27 przepustowości jednego, bo rozbiór odpowiedzi trzyma blokadę
interpretera, a Rust na czterech wątkach robi 376 i 375 dok/s (`08_python_wasm_long_run.md`). Maszyna ma dwa rdzenie
i cztery procesory logiczne (`../MACHINE.md`), więc zyski z wątków są tu małe w porównaniu z większymi serwerami.

Wniosek: model wątków wpływał na wynik pod tłokiem bardziej niż na wolnej maszynie. Wątek wywołujący, który pracuje
razem z pomocnikami, i próg równoległości były lekcjami przenoszonymi między językami. Najsłabiej pod tłokiem wypadł
Go, a Python i TypeScript korzystają z wielu rdzeni tylko przez osobne procesy albo izolaty.
