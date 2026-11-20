# Java

> Java była pierwsza i jest odniesieniem (krotność 1,0 w `../data/language_summary.csv`). Zmiany algorytmiczne opisuje
> `00_common.md`; tu są te, które zależały od JVM: GC, flagi, kompilator C2, profil JFR, obraz natywny GraalVM.

## Droga

Pierwsza wersja serwisu potrzebowała 70–134 s na długi dokument, a szczyt pamięci przy zimnym starcie wynosił 3–3,6 GB.
Po rundach optymalizacji długi dokument trwał 23–120 ms (ok. 700–2100× szybciej), szczyt pamięci ok. 0,4–0,46 GB.
W macierzy końcowej Java ma szczyt RSS 340 MB i start 3,8 s (`../data/language_summary.csv`).

## Co pomogło (cechy JVM)

| zmiana | technika | efekt |
|---|---|---|
| Serial GC zamiast G1 | jeden wątek GC bez pracy współbieżnej; przy celu „czas, dopóki RSS < 500 MB” | długie dokumenty pl −14…20%, mg −22…44% czasu, szczyt RSS ≤ 476 MB |
| flagi pamięci | `-Xmx384m`, kompaktowe nagłówki obiektów, sterta oddawana systemowi (`MinHeapFreeRatio` / `MaxHeapFreeRatio`) | RSS 312 / 330 MB (pl / mg), kosztem ok. +14% czasu w pl |
| `-Xms384m` po zmianie JDK | nowe JDK zmieniło domyślny rozmiar początkowy sterty; rosnąca od zera sterta dotykała więcej stron pamięci | szczyt zimnego startu pl 510 → 446 MB, młodych GC 166 → 39, czas bez zmian |
| podział dużej metody | `classifyDocument` (2318 B bajtkodu) przy zwykłym wywołaniu zostawał na poziomie C1 z profilowaniem; po podziale na etapy kompiluje go C2 | CPU na dokument mg −45%, wynik identyczny |
| wzorce bez `CASE_INSENSITIVE` (#15) | tekst raz zamieniony na małe litery, wzorzec bez flagi; osobna ścieżka dla ı, İ, ſ i znaku kelwina | etap wzorców pl −30…55% |
| tablica znak → kod (#13) | kod znaku detektora z tablicy zamiast z mapy, pamięć ostatniego zapytanego słowa | mg −19…78% |
| słowo już małymi literami (runda 11) | test „czy słowo jest już małe” przed `toLowerCase(Locale.ROOT)` i tablica BMP | CPU pl ok. −8%, mg ok. −7% (razem z resztą rundy) |
| obraz natywny GraalVM z PGO | `native-image` z profilem z treningu na próbkach | 0,91–0,98 czasu Javy JIT (bez profilu 1,16–1,39), RSS ×0,54–0,69 |

## Co nie pomogło

| zmiana | dlaczego odrzucona | pomiar |
|---|---|---|
| fastutil zamiast kolekcji JDK | mapy to tylko 2,6 / 3,7% CPU | czas w szumie, jar +24 MB |
| wyszukiwanie wpisów bez blokady (#22, #35) | artefakt profilu: próbki pętli po JIT lądują na sąsiedniej ramce | −55% próbek ramki, zero zmiany czasu i CPU |
| filtr histogramu w kodzie termometrowym (#31) | `bitCount` dwóch liczb `long` zamiast pętli po 32 bajtach; pętlę C2 najpewniej już wektoryzował, a kosztem było przejście pasma (ok. 11,5 tys. haseł na słowo spoza słownika) | w szumie |
| `Random` bez CAS | ten sam generator w zwykłym polu zamiast `AtomicLong`; CAS nie był kosztem | w szumie |
| własna pula wątków dokumentu | wspólna pula ForkJoin czyści `ThreadLocal` przy spoczynku, ale odbudowa bufora w TLAB jest tania | pl 85,3 vs 85,8 dok/s |
| interner tokenów (jak w C#) | `substring` w JVM to szybkie kopiowanie wewnętrzne; oszczędność ok. 27 próbek, koszt ok. 48 | zero netto |
| pomijanie NFKC dla znaków stabilnych | sprawdzenie w JDK jest już tanie | w szumie |
| filtr obecności klucza w indeksie literówek | kosztem jest liczba kandydatów (ok. 370 na zapytanie), nie chybienia w tablicy | w szumie (w Ruście ten sam pomysł skrócił generowanie kandydatów o ok. 24%, całość też w szumie) |
| AOT cache jako lek na rozgrzewkę | pod obciążeniem JIT i tak kompiluje te same metody | bez efektu |
| Graal JIT zamiast C2 | gorące pętle polskie (skan pasma literówek, DP Levenshteina) C2 kompiluje lepiej | pl 35–45% mniej dok/s, mg w szumie |
| Serial GC przy celu „pamięć” | ten sam GC przy innym celu: RSS +90…110 MB | odrzucony, potem przyjęty przy celu „czas” |

## Przykład: flagi produkcyjne

Plik `jvm.options` serwisu (argfile launchera `java`), ten sam w bramkach wydajności:

```
-Xmx384m
-XX:+UseSerialGC
-XX:+UseCompactObjectHeaders
-XX:MinHeapFreeRatio=5
-XX:MaxHeapFreeRatio=20
-Dnsa.verdict-cache.size=50000
```

W polskim serwisie dochodzi `-Xms384m`. Wyższy `-Xmx` łamałby sufit 500 MB RSS.

## Przykład: metoda za duża dla C2

Po podziale `classifyDocument` jest czterema wywołaniami etapów; każdy etap jest osobną, małą metodą, którą C2
kompiluje normalnie, a nie tylko przez OSR:

```java
var first = firstPass(occurrences, tokens, chosen, stageNanos);   // werdykt każdego słowa
var tally = tally(tokens, occurrences, first, stageNanos);        // liczniki języków i pism
var gates = documentGates(tokens, first, tally);                  // progi dokumentu
var signals = documentSignals(text, tokens, first, tally, gates, invisibleInWord, stageNanos);
```

Diagnoza: `-XX:+LogCompilation` pokazał, że stara metoda przy zwykłym wywołaniu zostawała na poziomie 3 (C1
z profilowaniem), a C2 dostawał ją tylko jako OSR.

## Techniki w skrócie

- GC wybierać do celu: Serial dla czasu przy małej stercie, ale zmierzyć RSS; przy zmianie JDK sprawdzić wartości
  domyślne (`InitialHeapSize`).
- Duża gorąca metoda może nie dostać C2; dzielić na etapy i sprawdzać `LogCompilation`.
- Profilu JFR nie brać dosłownie: trywialna ramka z dużym udziałem bywa sąsiadem wkompilowanej pętli. Zysk potwierdzać
  czasem i CPU, wyłącznikiem na tym samym buildzie.
- Kolekcje prymitywne i własne pule wątków nie pomogły: JVM robi tanio to, co w innych językach trzeba było robić ręcznie
  (alokacja w TLAB, kopiowanie napisów, analiza ucieczki).
