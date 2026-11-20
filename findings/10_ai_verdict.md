# Werdykt AI: jak wybrać język i czego unikać

> To opinia AI (Claude), które napisało wszystkie porty, oparta na pomiarach z tego katalogu i na doświadczeniu
> z pracy nad nimi. Dotyczy jednego rodzaju obciążenia (przetwarzanie tekstu, dużo krótko żyjących obiektów)
> i jednej maszyny (`../MACHINE.md`). Liczby wydajności pochodzą z plików wniosków, do których odsyła każde zdanie.

## Najpierw pytanie, nie język

Przy gotowej referencji i bramce bit w bit przepisanie biblioteki zajmuje AI godziny (`06_port_cost.md`), więc koszt
napisania przestaje być argumentem. Liczą się trzy rzeczy: jak program zachowuje się tam, gdzie będzie działał
(wolna maszyna, tłok, ciasna pamięć, szybki start), co już umie zespół i jak łatwo sprawdzić poprawność kodu
pisanego przez AI.

| sytuacja | wybór AI | dlaczego (dane) |
|---|---|---|
| zespół i system na JVM | zostać przy Javie; obraz natywny z PGO, gdy liczą się pamięć i start | Java JIT jest stabilna przez godzinę pracy, obraz natywny z PGO ma czas Javy przy ok. 0,6 jej pamięci i 0,5–0,6 czasu startu (`02_speed_memory.md`, `08_python_wasm_long_run.md`) |
| najwyższa przepustowość na własnej maszynie | Rust | jeden z czterech najszybszych (ok. 0,7 czasu Javy), najkrótszy start, mało pamięci, a kompilator odrzuca część błędów AI przed uruchomieniem (`02_speed_memory.md`, `07_compiler_vs_gate.md`) |
| ekosystem .NET | C# JIT; NativeAOT dla szybkiego startu | C# JIT jest w czołówce czasu i dobrze trzyma ogon opóźnień w strumieniu żądań (`02_speed_memory.md`, `03_contention.md`) |
| ciasna pamięć | Rust albo obraz natywny | przy limicie 200 MB Java JIT spada do 38–49 dok/s, obraz natywny Javy trzyma 73–78, Rust 146–162 (`03_contention.md`) |
| maszyna dzielona z innymi programami | Rust, C++, C# | jednorodne zestawy tych implementacji trzymały krótki ogon opóźnień, gdy pięć instancji dzieliło maszynę (`03_contention.md`) |
| wołanie z Pythona | biblioteka natywna przez interfejs C, ale z tanim formatem odpowiedzi | sam algorytm ma tempo Rustu, a koszt robi przekazywanie odpowiedzi i blokada interpretera (`08_python_wasm_long_run.md`) |

## Czego unikać

- **TypeScriptu (Node, Bun) do pracy obliczeniowej.** Był 5,7–9,1 raza wolniejszy od Javy (`02_speed_memory.md`),
  w strumieniu 60 dok/s nie nadążał (p99 5–9 s), a we wspólnym serwerze kończył się przekroczeniem limitu sterty
  (`03_contention.md`). Do tego bez osobnego sprawdzania typów błąd typu wychodzi dopiero w działaniu
  (`09_ai_work.md`).
- **Portu bez automatycznej bramki równoważności.** Najgroźniejszy błąd w C++ przeszedł kompilację, a bez bramki
  szybsza implementacja mogłaby po prostu liczyć coś innego (`01_equivalence.md`, `07_compiler_vs_gate.md`).
- **Domyślnych ustawień pamięci.** Każdy runtime wymagał świadomej decyzji: JVM limitu sterty, .NET trybu
  odśmiecania, V8 limitu sterty, Go limitu pamięci. Domyślne potrafiły przekroczyć budżet pamięci
  (`09_ai_work.md`), a TypeScript we wspólnym serwerze i tak skończył na limicie sterty (`03_contention.md`).
- **Ręcznego asemblera i GPU jako pierwszego kroku.** Asembler pokazał, czego nie zrobił kompilator, a zysk dała
  poprawka w zwykłym kodzie i wektoryzacja (`04_assembler_simd.md`). Karta graficzna przegrała z dobrym indeksem na
  CPU (`05_gpu.md`).
- **Wyboru po jednym wykresie z wolnej maszyny.** Pod tłokiem kolejność się zmienia: Go zachowuje najmniej własnej
  przepustowości, a JVM zachowuje się różnie zależnie od przebiegu (`03_contention.md`).
- **Mierzenia na maszynie, na której coś działa w tle, bez kontroli.** Różnice poniżej ok. 10% są tu remisem
  (`../METHODOLOGY.md`).

## Języki oczami AI

- **Kotlin** — najprzyjemniejszy do portu z Javy, prawie zdanie w zdanie, ten sam profiler; na JVM nieco wolniejszy
  od Javy (`02_speed_memory.md`).
- **Scala** — dobry kompilator, ale idiomatyczne kolekcje w gorącej ścieżce kosztują, a kompilacja jest wolna.
  Sensowna, gdy zespół i tak pisze w Scali.
- **Rust** — najwięcej pracy na start, bo trzeba było zbudować odtworzenie Javy, ale potem najmniej niespodzianek:
  kompilator wyłapuje błędy, które w innych językach wychodzą w testach. Pod Windows brakuje profilera.
- **C#** — napisy jak w Javie, dobre komunikaty błędów, bardzo szybki JIT. Wymaga strojenia odśmiecania pod limit
  pamięci.
- **Go** — razem z TypeScriptem najkrótszy port (`../data/port_cost.csv`), zero zależności, świetny wbudowany
  profiler. GC niegeneracyjny zmusza do układania danych
  bez wskaźników, a pod tłokiem Go traci najwięcej.
- **C++** — wydajność i narzędzia diagnostyczne (sanitizer, listing asemblera) bardzo dobre, ale kompilator przepuszcza
  błędy czasu życia obiektów. Z AI tylko z bramką i testami.
- **TypeScript** — szybki do napisania, wolny w działaniu, z limitami pamięci silnika. Dobry do warstwy wokół
  biblioteki, nie do samego liczenia.

**Wniosek AI:** nie ma jednego zwycięzcy, ale jest jedna rzecz, bez której wybór nie ma sensu: automatyczna, ścisła
definicja poprawności. Z nią AI przepisze bibliotekę na każdy z tych języków w godziny, a decyzję można oprzeć na
pomiarze w warunkach docelowych, nie na reputacji języka.
