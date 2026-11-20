# Scala

> Port idiomatyczny na tej samej JVM co Java, od pewnego momentu wyłącznie na kolekcjach Scali (decyzja projektowa,
> bez `java.util` w kodzie portu). Zmiany algorytmiczne opisuje `00_common.md`. Prawie wszystkie zmiany własne Scali
> dotyczyły tego samego: domknięć, opakowanych liczb i kolekcji ogólnych w gorącej pętli.

## Droga

Pierwsza wersja miała ×4,6–5,3 czasu Javy na konstytucjach, ale nierówno: na polskiej konstytucji przez polski pipeline
×1,31, na obcych ×5,7. Część tego wyniku była błędem pomiaru (harness trzymał raporty całego przejścia i Scala kończyła
je pełnymi GC); po poprawce harnessu pl ×5,0, mg ×2,77. Po rundzie zmian Scali ×1,24–1,62, po ogonach idiomu malgaski
×1,19. W macierzy końcowej ×1,24 czasu Javy, szczyt RSS 388 MB, start 5,4 s (`../data/language_summary.csv`).

## Co pomogło

| zmiana | co było nie tak | technika | efekt |
|---|---|---|---|
| S1 | `DictionaryLookup` (67% próbek pl): `Vector` 32 opakowanych liczb na kandydata, łańcuch iteratorów z krotką, `Ordering` z opakowaniem, `for` z modyfikowanym `var`, `foldLeft` po `Char` | pętle `while`; minimum jedną pętlą z ostrym `<` (pierwszy z równych, jak `minOption`) | S1–S5 razem: ×1,24–1,62 czasu Javy |
| S2 | indeks detektorów (ok. 14% próbek pl): `BoxesRunTime.equals2` przy `contains` na `Array[Int]`, `break` rzucany z domknięć `for`, `slice` + `exists` + `foldLeft` na każde okno | pętle `while`, lokalny `return` | (z S1) |
| S3 | czas per etap: mapa Scali z opakowanym `Long` na token | `Array[Long]` po `ordinal` | jak Java #33 |
| S4 | pula „jako tablice” była w rzeczywistości `Vector` czytanym przez ogólne `apply`, filtr kategorii przez `iterator.filter.map.take` | `Array[Array[…]]` i pętla | (z S1–S5) |
| S5 | odległość edycyjna marek: `for i <- 1 to m` z modyfikowanym `prev` | pętle `while` | (z S1–S5) |
| S6, S7 | `isSelf` (522 wywołania na obce słowo w mg): dwa domknięcia i `flatMap` / `map` / `getOrElse` na każde wywołanie; sugestia marki: `iterator.filter.map(krotka).filter.minByOption.map` po 49 markach | gałęzie i pętle `while` jak w Javie | CPU −4…9% w obu parach pomiaru |
| ogony idiomu (4d) | ścieżka dokumentu mg na `Vector` i krotkach | `ArraySeq` i pętle | mg ×1,49 → ×1,19 czasu Javy |
| rundy wspólne | — | to samo co w Javie | w rundzie 8 CPU pl −25% |

## Co nie pomogło albo kosztowało

| zmiana | skutek | pomiar |
|---|---|---|
| tylko kolekcje Scali (decyzja, nie optymalizacja) | `TrieMap` zamiast `ConcurrentHashMap` w cache werdyktów | mg ok. +5% czasu, +9% CPU; pl w szumie |
| S8: holder indeksu jako krotka `(PackedDetectors, Option[…])` zamiast rekordu | nie robione: ramka miała 8% próbek w Scali, ale w Javie w tym samym miejscu 37%, czyli artefakt profilu (wkompilowana pętla na sąsiedniej ramce), nie koszt krotki | — |

## Przykład: odległość edycyjna z progiem (S1)

Przed (schemat z dziennika): `for j <- 1 to n do` z modyfikowanymi `prev` i `rowMin`. Domknięcie pętli `for` przechwytuje
zmienne, więc kompilator zamienia każdą modyfikowaną zmienną na obiekt `IntRef` na stercie, a zakres `1 to n` jest
obiektem `Range` z wywołaniem `foreach`.

Po: pętle `while`, wszystko w zmiennych lokalnych.

```scala
def distanceWithin(a: String, b: String, maxDistance: Int, dp: Array[Int] = null): Int =
  val m = a.length
  val n = b.length
  val lenDiff = math.abs(m - n)
  if lenDiff > 20 then lenDiff
  else if lenDiff > maxDistance then maxDistance + 1
  else
    val row = if dp == null || dp.length < n + 1 then new Array[Int](n + 1) else dp
    var j = 0
    while j <= n do { row(j) = j; j += 1 }
    var i = 1
    var exceeded = false
    while i <= m && !exceeded do
      var prev = row(0)
      row(0) = i
      var rowMin = i
      val ai = a.charAt(i - 1)
      j = 1
      while j <= n do
        val temp = row(j)
        val v = if ai == b.charAt(j - 1) then prev
                else 1 + math.min(prev, math.min(row(j), row(j - 1)))
        row(j) = v
        prev = temp
        if v < rowMin then rowMin = v
        j += 1
      exceeded = rowMin > maxDistance
      i += 1
    if exceeded then maxDistance + 1 else row(n)
```

## Techniki w skrócie

- W gorącej pętli: `while` zamiast `for` po zakresie, bo modyfikowana zmienna w domknięciu staje się `IntRef`.
- `Array` zamiast `Vector` i `IndexedSeq`; uwaga na kolekcje, które tylko wyglądają na tablice (ogólne `apply`).
- Krotki, `Option` i łańcuchy `iterator.filter.map` na każde wywołanie to alokacje, których C2 nie zawsze usunie
  (inaczej niż proste lambdy Kotlina, `02_kotlin.md`).
- Czystość idiomu kosztuje i trzeba to zmierzyć: kolekcje współbieżne Scali kosztowały kilka procent.
