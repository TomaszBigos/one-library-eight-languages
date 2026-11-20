# GPU wobec dobrego algorytmu na CPU

Zintegrowana karta (OpenCL z systemowej biblioteki, bez instalacji) liczy pytanie każdego zapytania — czy któryś
z 3000 detektorów zbioru pasuje do słowa — wszystkie porównania naraz. Na siłę jest ok. 3 razy szybsza od CPU
sprawdzającego zbiór po zbiorze (366–372 wobec 1070–1120 µs na słowo), ale 1,6–1,7 razy wolniejsza od jednego
indeksu na CPU, który większości porównań w ogóle nie robi (220–226 µs), w obu seriach (`../data/gpu.csv`). Ten sam program GPU wywołany z sześciu języków (C++, Java
przez Panamę, Rust, Go, C#, Bun) daje ten sam czas (365–366 µs na słowo, 0 odpowiedzi różnych) — liczy się karta, nie język hosta. W treningu detektorów karta
sprawdza paczki kandydatów, a losowanie zostaje na CPU w niezmienionej kolejności, więc detektory wychodzą bajt w bajt
te same; brute force jest tu wolniejszy od CPU, a nawet po przeniesieniu na kartę pomysłu indeksu (posortowane
fragmenty słów i wyszukiwanie binarne) zimny start całej biblioteki na wolnej maszynie zostaje o 8% (malgaski) i 12%
(polski) dłuższy niż na samym CPU — trzy rundy w przeplocie, detektory identyczne we wszystkich
(`../data/gpu.csv`). Wniosek: przy algorytmie, który pomija większość pracy,
sprzęt liczący wszystko na siłę nie ma czego przyspieszyć — prawo Amdahla w praktyce.
