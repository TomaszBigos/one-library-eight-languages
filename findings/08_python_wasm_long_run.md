# Python nad Rustem, WebAssembly, długie działanie, OpenJ9

Biblioteka w Ruście wołana z Pythona przez interfejs C przetwarza 341 tekstów konstytucji w tempie 0,42 (polski)
i 0,32 (malgaski) tempa samego Rustu (69 i 71 wobec 164 i 224 dok/s, jeden wątek), choć czas wewnątrz biblioteki
jest ten sam (1,37 wobec 1,59 s i 1,15 wobec 1,12 s) (`../data/python_rust.csv`). Różnica powstaje poza algorytmem:
odpowiedź przechodzi przez JSON, budowany w Ruście i rozbierany w Pythonie, więc mediana czasu dokumentu po stronie
wywołującego to 7,3 i 6,9 ms wobec 3,3 i 2,8 ms. Cztery wątki Pythona dają tylko 1,26–1,27 przepustowości jednego
(86 i 90 dok/s), a proces zajmuje najwyżej 44% maszyny, bo wątki dzielą blokadę interpretera; Rust na czterech
wątkach robi 376 i 375 dok/s. Na krótkich dokumentach strata jest mniejsza: 0,51 i 0,67 tempa Rustu na jednym wątku
(`../data/python_rust.csv`).

Ten sam port jako WebAssembly (wasmtime, bez wątków) ma 0,61 i 0,53 przepustowości Rustu natywnego, któremu też
wyłączono podział dokumentu na wątki (82 wobec 133 i 98 wobec 185 dok/s), a czas biblioteki 1,6 i 2,0 raza dłuższy
(`../data/wasm.csv`).

Przez 46–104 minuty nieprzerwanej pracy (900–2000 przejść, cache werdyktów jak w produkcji) żadna z sześciu
implementacji (Java JIT, obraz natywny Javy, Rust, Go, C# JIT, C++) nie kończy wolniej, niż zaczęła: mediana ostatniej
dziesiątej części przejść to 0,978–1,013 mediany pierwszej (`../data/endurance_runs.csv`). Każde przejście obejmuje
bibliotekę i zamianę raportu każdego tekstu na JSON, tak samo we wszystkich implementacjach. W polskim przejście trwa
1,70 s (C#), 1,72 s (Go), 1,73 s (C++), 1,82 s (Rust), 2,42 s (Java JIT) i 4,12 s (obraz natywny z PGO); w malgaskim
1,39 s (Go), 1,44 s (C++), 1,49 s (C#), 1,64 s (Rust), 2,19 s (Java JIT) i 3,93 s (obraz natywny). Mediany okien
5-minutowych każdego przebiegu różnią się najwyżej o 0,1 s, poza jednym oknem Javy JIT w polskim (2,66 s wobec
2,41–2,46 s w pozostałych) (`../data/endurance.csv`). Różnice między Rustem, Go, C# i C++ obejmują też zamianę raportu
na JSON własną biblioteką każdego języka, więc nie są porównaniem samych bibliotek; to robi macierz
(`02_speed_memory.md`).

Pamięć też nie rośnie. Java JIT w polskim trzyma 469–472 MB od początku do końca, w malgaskim waha się między 284
a 433 MB bez trendu; Go, Rust i C++ wahają się najwyżej o 19 MB. Pamięć C# w malgaskim waha się między 372 a 531 MB bez
trendu, a jej szczyt (547 MB) pada już w pierwszych 5 minutach (`../data/endurance.csv`). C++ zmierzono z alokatorem
mimalloc 3.3.2: przejście 1,73 s w polskim i 1,44 s w malgaskim, pamięć procesu 341–360 i 346–357 MB
(`../data/endurance.csv`). Pierwsza próba, z mimalloc 2.1.7 użytym we wszystkich pozostałych pomiarach C++, kończyła
się błędem procesu (kod 0xC0000409) po 1215 przejściach w polskim i 1345 w malgaskim. Program pomiarowy tworzy nowy
wątek na każde przejście, a ta wersja alokatora nie odzyskiwała segmentów pamięci zakończonych wątków: pamięć
prywatna procesu rosła o ok. 13 MB na przejście. Sterta systemowa i mimalloc 3.3.2 (wersja, której używa port Rustu)
trzymają pamięć płasko. W krótkich pomiarach obie wersje alokatora dają czas w granicach 3%, szczyt pamięci
w granicach 10 MB i te same raporty, więc pozostałe liczby C++ nie wymagają przeliczenia (`../data/README.md`).

Obraz natywny Javy jest tu 1,7–1,8 raza wolniejszy od JIT, choć sama biblioteka ma w nim ten sam albo krótszy czas: różnicę robi
zamiana raportu na JSON biblioteką Jackson, która kosztuje obraz natywny 2,2–2,4 s na przejście, a JIT 0,45–0,68 s
(`../data/README.md`). Widać to w czasie CPU na przejście: 5,0–5,2 s wobec 3,1–3,5 s w polskim i 4,3–4,4 s wobec
2,6–2,9 s w malgaskim (`../data/endurance.csv`). Powtórka obrazu natywnego w polskim (ta sama binarka i ten sam
program pomiarowy, 1500 przejść, 104 minuty) dała to samo tempo: 4,13 wobec 4,12 s (`../data/endurance_runs.csv`,
`run` = 2). JIT nie musi niczego doganiać: już w pierwszych 5 minutach jest w stanie ustalonym (2,43 s w polskim,
2,19 s w malgaskim) i tak zostaje (`../data/endurance.csv`). Obraz natywny trzyma za to 0,59–0,61 szczytu pamięci
Javy (289 wobec 489 MB, 301 wobec 490 MB) (`../data/endurance_runs.csv`). W macierzy, przy wyłączonym cache i samym
czasie biblioteki, obraz natywny z PGO był nieco szybszy od JIT (0,91–0,98, `../data/matrix.csv`), więc kolejność
tych dwóch zależy od tego, co się mierzy.

Pierwsza seria tych pomiarów dla Javy JIT, obrazu natywnego i Rustu szła starszą wersją programu pomiarowego, która
zamieniała raport na JSON tylko w ostatnim przejściu, a nie w każdym. Obraz natywny miał w niej 2,0 s na przejście,
a późniejsza powtórka nową wersją 4,1 s, co wyglądało jak trzy tempa jednej binarki. Tamte przebiegi zastąpiono
powtórką obecną wersją programu (`14_curiosities.md`). W powtórce dwa przebiegi (Java JIT w malgaskim po 48 minutach,
obraz natywny w polskim po kilku minutach) padły z naruszeniem dostępu do pamięci w trakcie odśmiecania, w odstępie
7 minut, w dwóch różnych środowiskach uruchomieniowych; ponowione przeszły do końca z tymi samymi raportami.

Ta sama Java na OpenJ9 (JDK 25) dochodzi do stałego tempa po około 10–12 przejściach, HotSpot (JDK 27) po 2–3
(`../data/openj9.csv`). W stanie ustalonym (przejścia 13–24, każde razem z zamianą raportu na JSON) OpenJ9 potrzebuje
2,98 s wobec 2,28–2,29 s HotSpot w polskim (1,30 raza) i 2,88–2,93 wobec 1,84–1,85 s w malgaskim (1,57 raza), a szczyt
pamięci procesu przy tej samej stercie 384 MB wynosi 543–604 wobec 390–418 MB w polskim i 1079–1084 wobec 378–391 MB
w malgaskim (`../data/openj9_runs.csv`).

Wniosek: Python płaci za przejście przez JSON i blokadę interpretera, nie za algorytm; WebAssembly kosztuje 1,6–2,0
raza czasu biblioteki; przez godzinę pracy nic nie dryfuje ani w czasie, ani w pamięci; HotSpot jest dla tej biblioteki lepszą JVM pod
względem czasu i pamięci; obraz natywny Javy w długim działaniu jest stabilny, ale przejście razem z zamianą raportu
na JSON trwa w nim 1,7–1,8 raza dłużej niż w JIT.
