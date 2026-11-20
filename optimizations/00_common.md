# Wspólne: zmiany algorytmiczne

> Zmiany, które powstały w Javie (pierwsza implementacja) i potem trafiły do każdego portu. Liczby są z Javy, chyba że
> napisano inaczej. Największą część zysku w każdym języku dały właśnie one; cechy języka decydowały o reszcie
> (`../findings/11_optimizations.md`).

## Co pomogło

| zmiana | technika | efekt |
|---|---|---|
| porównanie Hamminga bez kopii (#0) | dopełnienie znakiem `#` liczone w pętli zamiast dwóch nowych napisów na każde porównanie | razem z #1 i #2: długie dokumenty 18–24× szybciej |
| NSA liczy, zamiast zbierać (#1) | licznik dopasowań zamiast listy detektorów i wyjście na pierwszym dopasowaniu | (w #0) |
| literówki z progiem i filtrem długości (#2) | odległość edycyjna przerywana po przekroczeniu progu, słowa o złej długości pomijane bez liczenia | (w #0) |
| detektory w jednej tablicy (#3) | wszystkie detektory zbioru w jednej tablicy kodów i tablicy przesunięć zamiast napisu na detektor (75–84% żywej sterty to były napisy detektorów) | żywa sterta 116 / 140 MB (pl / mg) |
| memo werdyktu w dokumencie (#6), słowa równolegle (#9) | każde różne słowo dokumentu klasyfikowane raz, różne słowa na wszystkich rdzeniach | podstawa dalszych zmian ścieżki dokumentu |
| indeks odwrotny detektorów (#10) | zamiast porównywać słowo z 3000 detektorami: okna słowa wyszukiwane w posortowanej tablicy okien detektorów | pl −43…47%, mg −66…75% czasu |
| tryb dokumentu kończy po pierwszej pasującej kategorii (#11) | wystarczy jedna kategoria, w której słowo jest „swoje” | klasyfikacja mg −21…42% |
| cache werdyktów między dokumentami (#23) | ograniczona mapa słowo → werdykt w instancji klasyfikatora | strumień aktów prawnych −27…40% czasu serwera |
| prefiltr literałów wzorców (#24; w mg #7–#8) | wzorzec iniekcji uruchamiany tylko, gdy tekst zawiera jego obowiązkowy literał | etap wzorców pl −67…94% |
| mapa bitowa par Hamminga (#26) | dla Hamminga z progiem 4 na 6 pozycjach zapytanie = najwyżej 15 testów bitu zamiast zliczania niezgodności | pl −35…55%, mg −48…65% (długie dokumenty i klasyfikacja) |
| trening przez indeks nad zbiorem „swoich” słów (T1) | kandydat na detektor sprawdzany indeksem, nie liniowo po zbiorze | zimny start Javy ×0,38–0,49 |
| jeden indeks na pulę zbiorów (4a, `PoolIndex`) | jedno zapytanie na słowo dla wszystkich 256 / 522 zbiorów detektorów: klucz → maska bitowa zbiorów | Hamming pl −24…29%, mg −28…39% CPU; drugi matcher (r-contiguous) pl ×14, mg ×36 szybciej |
| indeks segmentów literówek (4b, Pass-Join) | słowo dzielone na k+1 segmentów; kandydat musi mieć jeden z nich na tej samej pozycji, dopiero potem odległość edycyjna | CPU pl −35…38% |
| runda 8 | sondy Pass-Join świadome wielu trafień, tablica pism BMP, szybki tokenizer, słownik obcy przez hasz z odciskiem | CPU pl −22…32%, mg −20…38% (wobec rundy 7) |

## Przykład: porównanie Hamminga bez kopii

Przed (wersja referencyjna, zachowana w testach): każde porównanie tworzyło dwa nowe napisy dopełnione do stałej
długości.

```java
private static boolean reference(HammingMatching m, int normalizedLength, String a, String b) {
    String na = m.normalize(a), nb = m.normalize(b);       // dwie kopie dopełnione znakiem '#'
    int threshold = m.effectiveThreshold(a, b), dist = 0;
    for (int i = 0; i < normalizedLength; i++) {
        if (na.charAt(i) != nb.charAt(i) && ++dist > threshold) return false;
    }
    return true;
}
```

Po: dopełnienie „wirtualne”, bez alokacji. Test porównuje obie wersje na 100 000 losowych par.

```java
public boolean matches(String a, String b) {
    int threshold = effectiveThreshold(a, b);
    int la = a.length(), lb = b.length();
    int end = Math.min(normalizedLength, Math.max(la, lb));
    int dist = 0;
    for (int i = 0; i < end; i++) {
        char ca = i < la ? a.charAt(i) : PAD;              // poza końcem słowa: znak dopełnienia
        char cb = i < lb ? b.charAt(i) : PAD;
        if (ca != cb && ++dist > threshold) return false;
    }
    return true;
}
```

## Przykład: jedno zapytanie na słowo dla całej puli

Fragment z Rusta (Java, C#, Go, C++ i TypeScript używają tego samego algorytmu). Zamiast pytać osobno każdy z 522 zbiorów
detektorów, okna słowa wyszukuje się raz, a wynik to maski bitowe zbiorów.

```rust
pub fn answer(&self, word: &JStr, non_latin: bool) -> Answer {
    let w = self.words;                        // liczba słów 64-bitowych masek
    let mut decided = [0u64; MAX_WORDS];       // maski na stosie, bez alokacji
    let mut selfs = [0u64; MAX_WORDS];
    if non_latin {                             // pismo niełacińskie: w żadnym zbiorze nie jest „swoje”
        decided[..w].fill(u64::MAX);
        self.trim(&mut decided);
        return Answer { decided, selfs };
    }
    let mut matched = [0u64; MAX_WORDS];
    let lw = word.len();
    if self.r > 0 {                            // r-contiguous: każde okno r znaków słowa
        decided[..w].copy_from_slice(&self.eligible);
        if lw >= self.r {
            for st in 0..=lw - self.r {
                self.or(&mut matched, self.find(window(&word[st..st + self.r])));
            }
        }
    } else {
        ...                                    // Hamming: pary pozycji, ta sama idea
    }
    ...
}
```

## Co się nie sprawdziło albo sprawdziło się później

- **Indeks segmentów literówek** za pierwszym razem (#27) dał wynik w szumie. Po kolejnych zmianach literówki urosły do
  ok. 52% pracy polskiego pipeline'u i ten sam pomysł (4b) dał −35% CPU. „Odrzucone pomiarem” jest względne wobec
  reszty kosztu.
- **SymSpell** dla literówek odrzucony rachunkiem, przed implementacją: 32 512 haseł dałoby ok. 3 mln wariantów
  (ok. 23 MB), a zapytanie to ok. 46 odczytów z tablicy większej niż pamięć podręczna procesora, porównywalnie
  z istniejącym indeksem.
- **Ramka z profilu, która nie była kosztem**: wyszukiwanie wpisów zbioru bez blokady (#22, potem jeszcze raz jako #35)
  usunęło 55% próbek tej ramki i nie zmieniło ani czasu, ani CPU. Po kompilacji JIT próbki z wkompilowanej pętli
  trafiały na sąsiednią ramkę (`01_java.md`).

## Techniki w skrócie

- Mniej pracy na słowo: indeks zamiast skanu, maska bitowa zamiast zliczania, wyjście przy pierwszym rozstrzygnięciu.
- Mniej pracy na dokument: każde różne słowo raz, cache między dokumentami, prefiltr przed drogim wyrażeniem regularnym.
- Każda zmiana ma wyłącznik na tym samym buildzie (np. `-Dnsa.pool-index.disabled=true`) i test równoważności ze
  starą ścieżką; bez tego nie dałoby się odróżnić zysku od szumu ani sprawdzić neutralności.
