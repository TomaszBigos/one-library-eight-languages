# Tłok, wspólny serwer, scenariusze

Gdy inny proces zajmuje CPU (osiągnięte 58–72% maszyny), Go zachowuje 29–33% własnej przepustowości, a pozostałe
implementacje 44–61% (`../data/load_curve.csv`); najwięcej zostaje Javie JIT, ale jej proces obciążający osiągnął też
najmniej (58–60%). Przy ciasnym limicie pamięci 200 MB Java JIT spada do 38–49 dok/s (82–102 przy 300 MB), obraz
natywny Javy trzyma 73–78 dok/s, a Rust 146–162 (`../data/scenarios.csv`). Przy procesie wypychającym cache
procesora (bloki 4 MB) najszybszy jest C++ (73–89 dok/s), przed Rustem (53–86) i Javą JIT (47–58); na jednym wątku
prowadzą Rust, C++ i Go (116–165 dok/s), Java JIT ma 78–103 (`../data/scenarios.csv`). W strumieniu 60 dok/s
(otwarta pętla, opóźnienie od zaplanowanego przyjścia) p99 Rusta, C++, C# JIT, Go i Javy JIT mieści się w 26–57 ms,
obrazów natywnych JVM i C# NativeAOT w 58–126 ms, a TypeScript nie nadąża (p99 5–9 s) (`../data/scenarios.csv`).
Gdy pięć instancji dzieli maszynę po 30 dok/s każda, jednorodne zestawy Rusta, C++, C# JIT i C# NativeAOT trzymają
p99 działających instancji do 75 ms we wszystkich przebiegach (pojedyncze nieudane instancje opisuje `../data/README.md`), Go do 192 ms, Java JIT w polskim waha się między 92 a 520 ms, obraz natywny
Javy przekracza swoją przepustowość, zaniżoną serializacją raportu w programie pomiarowym (p99 0,7–8,9 s), a TypeScript w polskim kończy się przekroczeniem limitu sterty
(`../data/colocate.csv`).

Krzywa opóźnienia (otwarta pętla od 20 do 120 dok/s, 4 konsumentów, ten sam harmonogram przyjść dla wszystkich): przy 20 dok/s p99 wszystkich siedmiu implementacji mieści się w 31–54 ms w polskim i 28–46 ms w malgaskim. Rust trzyma p99 w 31–47 ms (polski) i 21–41 ms (malgaski) na całej krzywej. Przy 120 dok/s kolejność w polskim to Rust 44 ms, C++ 56, Go 61, C# NativeAOT 77, C# JIT 110, Java JIT 117, a w malgaskim Rust 41, C# JIT 50, C++ 50, Go 60, Java JIT 71, C# NativeAOT 141. Obraz natywny Javy rośnie od 40–60 dok/s (p99 85 i 144 ms przy 60/s) do 499 i 700 ms przy 120/s, bo obsługuje tylko ok. 105 ze 111 przychodzących dok/s i kolejka rośnie; to ta sama granica przepustowości, zaniżona serializacją raportu w programie pomiarowym (`../data/latency_curve.csv`, `02_speed_memory.md`).

Wniosek: pod tłokiem kolejność jest inna niż na wolnej maszynie, bo o wyniku decyduje
zachowanie środowiska (odśmiecanie, liczba wątków, pamięć), a nie sam czas biblioteki; w zestawach mieszanych z Javą
ogon zależy od przebiegu, więc podajemy zakresy, nie jedną liczbę.
