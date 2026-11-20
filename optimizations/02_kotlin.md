# Kotlin

> Port idiomatyczny na tej samej JVM co Java. Zmiany algorytmiczne dostał razem z Javą (`00_common.md`), w kolejności
> Java → Kotlin → Scala. Tu są zmiany, które wynikały z tego, jak Kotlin kompiluje lambdy, `inline` i kolekcje.

## Droga

Pierwsza wersja, pisana idiomatycznie (sekwencje, lambdy, `List`), miała ×1,2–1,4 czasu Javy na konstytucjach
i próbkach polskich, a próbki malgaskie przy jednym kliencie ×1,75 (CPU ×2,55). Po rundzie zmian Kotlina ×1,08–1,22.
W macierzy końcowej ×1,14 czasu Javy, szczyt RSS 363 MB, start 4,5 s (`../data/language_summary.csv`).

## Co pomogło

| zmiana | co było nie tak | technika | efekt |
|---|---|---|---|
| K1 | lokalna funkcja (`fun` wewnątrz funkcji nie może być `inline`) owijała rozszerzenie `inline`, więc powstawały lambdy `Function2` z opakowanym `Int` i obiektami `Ref` | wywołanie rozszerzenia `inline` wprost | bez lambd i opakowań w budowie indeksu (to była najdłuższa kompilacja C2, 3,3 s) |
| K2 | `merge(via, nanos, Long::plus)` na każdy token: opakowany `Long` i hasz wyliczenia, najgorętsza linia metody (5% próbek) | `LongArray` indeksowana `ordinal` | jak Java #33 |
| K3 | `indices.asSequence().filter.map.take` przy każdym słowie: opakowany indeks kategorii i trzy lambdy na kategorię; detektory jako `List`, więc `all {}` brało iterator | pętla z wyjściem po pierwszej pasującej kategorii, pula jako `Array<Array<…>>` | (z K1–K5: ×1,08–1,22 Javy) |
| K5 | szczyt RSS mg dwumodalny: 350 albo 510 MB przy tej samej żywej stercie; jedna kompilacja C2 dużej metody budującej indeks brała dużą arenę | metoda rozbita na trzy, bez zmiennych nullowalnych (`!!`) w pętlach | kompilacja maks. 8 MB, szczyt RSS 337–357 MB w 6 / 6 przebiegach (Java ok. 345) |
| homoglify | pudełkowane znaki w zamianie homoglifów | tablica `CharArray` | (`../findings/11_optimizations.md`) |
| `tally` na tablicach | mapy liczników w ścieżce dokumentu | tablice prymitywów | część rundy „ogony idiomu” |
| rundy wspólne | — | to samo co w Javie | w rundzie 8 CPU pl −34…41% |

## Co nie pomogło

| zmiana | dlaczego | pomiar |
|---|---|---|
| K6: `isSelf` bez dwóch przechwytujących lambd na wywołanie | C2 sam usuwał te alokacje (analiza ucieczki) | 6,2 / 6,8 vs 6,3 / 6,3 s, CPU bez różnicy; cofnięty |

Ta sama zmiana w Scali (S6) pomogła (`03_scala.md`): lambdy Kotlina C2 umiał zlikwidować, a domknięcia Scali
z `flatMap` i `getOrElse` na `Option` już nie.

## Przykład: czas etapu per token (K2)

Przed: `viaNanos` było mapą, a każdy token robił `viaNanos.merge(via, nanos, Long::plus)`, czyli opakowanie `Long`
i haszowanie wyliczenia.

Po: tablica prymitywów indeksowana numerem wyliczenia, ta sama kolejność pierwszego wystąpienia i te same sumy:

```kotlin
val viaNanos = LongArray(Via.entries.size)
val viaSeen = ArrayList<Via>()
val responses = occurrences.mapIndexed { i, occurrence ->
    val nOne = System.nanoTime()
    val resp = (if (documentMemo) jobAnswer[jobOf[i]] else null)
        ?: classify(occurrence.lower, chosen, shortForeigns[i], documentFirstFit)
            .also { if (documentMemo) jobAnswer[jobOf[i]] = it }
    val vo = resp.via.ordinal
    if (viaNanos[vo] == 0L && resp.via !in viaSeen) viaSeen += resp.via
    viaNanos[vo] += System.nanoTime() - nOne
    resp
}
viaSeen.forEach { via -> stageNanos["classify:$via"] = viaNanos[via.ordinal] }
```

`mapIndexed` z lambdą zostało: to `inline`, więc nie kosztuje.

## Przykład: wyszukiwanie kategorii (K3, schemat)

```kotlin
// przed: sekwencja z lambdami, opakowany indeks na każdą kategorię
indices.asSequence().filter { pool[it].all { d -> d.isSelf(q) } }.map { names[it] }.take(minFits)

// po: pętla po tablicy tablic, wyjście po minFits trafieniach
var found = 0
for (c in pool.indices) {
    val detectors = pool[c]
    var all = true
    for (d in detectors) if (!d.isSelf(q)) { all = false; break }
    if (all && ++found == minFits) break
}
```

## Techniki w skrócie

- `inline` działa tylko wtedy, gdy wywołujący też jest wpleciony; lokalna funkcja przerywa ten łańcuch i wraca
  `FunctionN` z opakowaniami.
- Sekwencje i `List<Int>` w gorącej pętli to opakowane liczby i iteratory; tablice prymitywów i zwykła pętla.
- Szczyt RSS może zależeć od jednej kompilacji C2. Diagnoza: `-XX:CompileCommand=MemStat,*.*,print` i NMT
  (kategoria Compiler); lekarstwo: podzielić metodę.
- Nie wszystko trzeba przepisywać: drobne lambdy C2 usuwa sam (K6). Każdą zmianę mierzyć, zanim zostanie.
