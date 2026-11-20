# Który język pasuje do pracy z tekstem i NSA

> Opinia AI, które napisało wszystkie porty, oparta na danych z tego katalogu (tak jak `10_ai_verdict.md`). Pomiary
> nie mierzą składni wprost, tylko jej skutki: ilość i wygląd kodu, szybkość etapów tekstowych, ile trzeba było
> odtworzyć z platformy Javy, ile kosztował styl idiomatyczny i jak często kod się nie kompilował.

## Trzy zastrzeżenia

- **Java była specyfikacją.** Raport podaje pozycje w jednostkach UTF-16, a wzorce działają według semantyki `java.util.regex`
  (`09_ai_work.md`). To z góry faworyzuje języki z takim samym modelem tekstu.
- **Porty to tłumaczenie, nie niezależne napisanie.** Każdy ma architekturę Javy, a nie tę, którą podpowiedziałby
  sam język.
- **To samo AI i kolejność portów.** Późniejsze porty dostawały gotowe rozwiązania (`06_port_cost.md`), więc koszt
  i liczba błędów zależą też od tego, który port był który z kolei.

## Sześć osi

| oś | wypada najlepiej | wypada najsłabiej | źródło |
|---|---|---|---|
| ilość kodu biblioteki (linie) | TypeScript 4 086, Scala 4 227, Kotlin 4 629 | Go 8 205, Rust 7 374 | `../data/language_summary.csv` |
| ilość kodu (tokeny) | TypeScript 73 280, Go 103 419, Scala 112 273 | Java 213 900 (34% komentarzy), Rust 162 237 | `../data/language_summary.csv`, `19_readability.md` |
| wygląd kodu (ranking metryk) | Go, Java, Rust | TypeScript, Scala | `19_readability.md` |
| etapy tekstowe (× Javy) | C#, Rust, C++: tokenizacja pl 0,59 / 0,49 / 0,58, wzorce pl 0,61 / 0,60 / 0,63 | TypeScript 1,9–8,4× wolniej w każdym etapie | `20_text_ops.md` |
| zgodność modelu tekstu ze specyfikacją | Java, Kotlin, Scala: ta sama biblioteka JDK | Rust, Go, C++: inne napisy, a do tego regex, NFKC i właściwości znaków z JDK | `09_ai_work.md` |
| błędy kompilacji na 100 budowań | Kotlin 0, Java 2,4, C# 6,2 | Go 15,8, Scala 12,1 | `../data/language_summary.csv` |

C# i TypeScript stoją pośrodku czwartej osi: napisy liczą tak jak Java (UTF-16), ale wyrażenia regularne
i normalizację trzeba było odtworzyć jak w Ruście (`09_ai_work.md`). TypeScript ma 0 błędów kompilacji, bo
wykonywany jest bez kompilacji; jego błędy typów wychodziły dopiero przy uruchomieniu (`../data/ai_work.csv`).

## Idiom a szybkość

Najwięcej o składni mówi to, ile kosztował styl, który dany język uważa za naturalny:

- **Scala** w wersji idiomatycznej (`for` po zakresach, `Vector`, krotki, `Option`, łańcuchy `filter.map`) miała
  ×4,6–5,3 czasu Javy; po przepisaniu gorących miejsc na pętle `while` i tablice ×1,24–1,62
  (`../optimizations/03_scala.md`).
- **Kotlin** idiomatyczny miał ×1,2–1,4, po poprawkach ×1,08–1,22; część jego lambd kompilator JVM usuwał sam,
  domknięć Scali już nie (`../optimizations/02_kotlin.md`).
- **Rust i C++** w pierwszej wersji też były wolne (Rust ×9,5 / ×3,8 Javy), ale przez kopie słów i brak indeksów
  przeniesionych później z Javy, nie przez składnię (`../optimizations/04_rust.md`, `../optimizations/07_cpp.md`).

Wniosek: w językach najbardziej ekspresyjnych gorące pętle NSA (tablice detektorów, maski, indeksy) trzeba było
pisać wbrew ich idiomowi. Tam, gdzie prosty kod z pętlami jest też idiomatyczny (Rust, C#, C++, Go), koszt przyszedł
z innej strony: z alokacji i z modelu tekstu.

## Ocena

- **C#: najlepszy kompromis dla tego problemu.** Napisy w UTF-16 jak w specyfikacji, więc bez emulacji pozycji;
  `Span` i `stackalloc` do pracy na tekście bez alokacji; najszybszy albo w czołówce w etapach tekstowych
  (`20_text_ops.md`), ×0,75 czasu Javy całości przy średniej ilości kodu (`../data/language_summary.csv`).
- **Kotlin: najlepszy na JVM.** Zwięzły, najmniej błędów kompilacji, ×1,14 czasu Javy; koszt idiomu umiarkowany.
- **Scala: najbardziej ekspresyjna** (najmniej linii na funkcję, `19_readability.md`), ale jej idiom trzeba było
  porzucić w gorących miejscach.
- **Rust: w czwórce najszybszych** (×0,72, tyle co Go; czwórka różni się o mniej niż 10%, `00_summary.md`), za cenę
  dużej ilości kodu i emulacji UTF-16.
- **Go: najczytelniejszy według metryk**, ale najdłuższy w liniach, z największą liczbą błędów kompilacji i innym
  modelem tekstu.
- **TypeScript: najkrótszy kod i najwolniejsze etapy tekstowe**; do warstwy, nie do liczenia (`00_summary.md`).
- **Java** jest punktem odniesienia: jej biblioteka wyznaczyła specyfikację, więc pełna zgodność modelu tekstu jest
  tu z definicji.

## Czego te dane nie mówią

Nie mówią, który język ma obiektywnie najlepszą składnię. Do tego trzeba by napisać tę samą bibliotekę niezależnie
w każdym języku, zgodnie z jego naturą (np. Rust na UTF-8, bez specyfikacji opartej na Javie), i dać kod do oceny
ludziom, którzy nie wiedzą, który jest który. Mechanizmy, których w tym projekcie brakowało we wszystkich językach,
zbiera `24_language_design.md`.
