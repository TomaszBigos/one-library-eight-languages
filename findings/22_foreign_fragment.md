# Wstawka w obcym języku

> Dokument = ok. 1000 słów konstytucji języka pipeline'u (polskiej albo malgaskiej), osobno i z akapitem ok. 200 słów
> w jednym z 40 języków zbioru konstytucji (chiński i japoński: ok. 400 znaków), wstawionym po zdaniu w połowie tekstu.
> Każdy dokument 25 razy w przejściu, 5 przejść, 3 rundy w zmienianej kolejności, cache werdyktów wyłączony; czas =
> czas biblioteki na dokument (mediana powtórzeń, potem rund). 13 implementacji. Go zmierzono w osobnej sesji: jego
> zegar etapów odczytywał na Windows czas co ok. 1 ms, więc dokument krótszy niż milisekunda wychodził jako 0 albo
> ok. 1000 µs; po przejściu na licznik wysokiej rozdzielczości systemu (jak w pozostałych implementacjach) pomiar
> powtórzono razem z Javą i Rustem jako kontrolą. Ich czasy w obu sesjach różnią się o 0–6%
> (`../data/foreign_fragment.csv`).

**Pytanie: ile kosztuje akapit po turecku (albo w innym języku) w polskim tekście i czy pipeline go zauważa?**

Zauważa każdy. Wszystkie 39 wstawek w polskim pipelinie i 40 w malgaskim zmieniają raport, a żadna nie robi
z dokumentu „obcego dokumentu” (`foreign_document` = false). W polskim każda wstawka łacińska dodaje 1–28 bramkowanych
spanów iniekcji, a niełacińska span zmiany pisma (grecka 3) i zwykle jeden span iniekcji (grecka 3, ukraińska 10). Spany języka mniejszości dała tylko
niemiecka (4). W malgaskim wstawki łacińskie dodają 1–14 spanów iniekcji (polska 13), a niełacińskie tylko span zmiany
pisma, bez iniekcji (wyjątek: ukraińska, której fragment kończy się zdaniem po łacinie) (`../data/foreign_fragment.csv`).
Sygnały są te same we wszystkich 13 implementacjach: każda z 3 rund każdej implementacji dała jeden skrót raportów na
pipeline, na tekstach, których bramka równoważności wcześniej nie widziała.

Koszt jest umiarkowany. Wstawka to ok. 17% słów dokumentu, a czas rośnie średnio o 25–35% w implementacjach
kompilowanych i JVM oraz o 33–54% w TypeScript. Słowo łacińskiej wstawki kosztuje 1,3–1,9 raza tyle co słowo tekstu
pipeline'u (TypeScript do 2,4), bo słowa obce rzadziej kończą się na słowniku i przechodzą dalsze etapy klasyfikacji.
Przykłady na słowo: Java 1,49 µs tekstu polskiego wobec 2,29 µs wstawki łacińskiej (mediana języków), Rust 0,80 wobec
1,27 µs, C++ 0,84 wobec 1,42 µs, Go 1,14 wobec 1,70 µs; w malgaskim Java 0,62 wobec 1,08 µs, Rust 0,52 wobec 0,80 µs,
Go 0,50 wobec 0,81 µs
(`../data/foreign_fragment.csv`, `../charts.html`).

Dwa wyjątki:

- **Turecki w polskim pipelinie** jest najdroższą wstawką łacińską w 12 z 13 implementacji (poza Node):
  Java 5,86 µs na słowo, Rust 2,47 µs. Rośnie etap wyrażeń regularnych iniekcji (Java 0,44 → 1,19 ms, Rust 0,23 →
  0,49 ms na dokument): litery ı i İ zmieniają się inaczej przy zmianie wielkości, więc tekst z nimi idzie wolniejszą,
  dokładną ścieżką wzorców. W malgaskim pipelinie turecki się nie wyróżnia. To ten sam turecki co w
  `21_doc_language.md`, tylko w krótkim akapicie zamiast całych konstytucji.
- **Chiński i japoński** (długie tokeny bez spacji) kosztują na token 1,6–5,5 µs w implementacjach kompilowanych i JVM
  oraz 19–28 µs w TypeScript (Node i Bun) (`../data/foreign_fragment.csv`).

Kolejność implementacji jest podobna jak w macierzy: najtańsze są Rust, C++ i C# JIT, potem Go (w malgaskim razem
z nimi) i C# NativeAOT, dalej Java, Kotlin i Scala (JIT i obrazy natywne), na końcu TypeScript (`../data/foreign_fragment.csv`, `02_speed_memory.md`).

**Wniosek:** obcy akapit w tekście nie jest dla tej biblioteki kosztownym przypadkiem brzegowym. Kosztuje ok. jedną
trzecią czasu dokumentu więcej, a każda implementacja zgłasza go tak samo. Wyjątkiem jest pismo z literami
o nietypowej zmianie wielkości (turecki), które spowalnia wzorce, a nie klasyfikację słów.
