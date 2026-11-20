# Wrażliwość na język dokumentu

> Te same przebiegi co `20_text_ops.md`, rozbite na język konstytucji (40 języków, 1–15 tekstów na język). Czas raportu
> jest w pełnych milisekundach, więc liczymy tylko języki, w których Java potrzebuje co najmniej 30 ms na przejście:
> 27 języków w polskim pipeline i 23 w malgaskim; reszta stoi w tabeli bez krotności (`reliable` = false,
> `../data/doc_language.csv`).

**Pytanie: czy turecki fragment przechodzi przez polski pipeline szybciej w Ruście, czy w C++?** Remis: 7 tureckich
konstytucji to 28 ms na przejście w Ruście i 29 ms w C++ (0,65 i 0,67 Javy), przy dokładności pomiaru 1 ms. Go ma
0,77, C# 0,70, obraz natywny Javy 0,98, TypeScript 5,56. Przez malgaski pipeline najszybszy na tureckim jest Go
(0,55), potem Rust i C# (0,61) i C++ (0,67) (`../data/doc_language.csv`).

Najszybsza implementacja dla danego języka dokumentu (`rank` = 1):

| pipeline | wygrane | uwagi |
|---|---|---|
| polski (27 języków) | Rust 17 · C# 5 · Go 3 · C++ 2 | Rust prowadzi w cyrylicy (bułgarski, rosyjski, serbski, ukraiński: 0,55–0,61; poza macedońskim, niżej), grece (0,58) i arabskim (0,64); koreański wygrywa Go (0,76) |
| malgaski (23 języki) | Go 9 · C# 8 · Rust 6 | bliżej remisu: najlepsza i druga implementacja różnią się zwykle o 0,01–0,10 Javy |

Gdzie implementacja jest wyraźnie wrażliwa na język:

- **Go na polskim tekście w polskim pipeline: 1,69 Javy** (135 wobec 80 ms), choć na prawie każdym innym języku
  Go jest szybszy od Javy. Różnicę robi etap wzorców: 66 ms wobec 33 w Javie i 16–19 w C++, C# i Ruście. Polskie
  wzorce iniekcji pracują naprawdę tylko na polskim tekście, bo w obcym filtr literałów odrzuca większość z nich
  przed wyrażeniem; na polskim tekście etap wzorców w Go trwa dwa razy dłużej niż w Javie
  (`../data/doc_language.csv`, `tokenize_ms`, `patterns_ms`).
- **C# na koreańskim w polskim pipeline: 1,66** — tokenizacja 15,7 ms wobec 8,2 w Javie i 4,8–5,3 w Go, C++
  i Ruście; na koreańskim przez malgaski C# ma 0,75. To jedyny język, na którym C# w polskim pipeline wyraźnie
  przegrywa z Javą.
- **Pojedyncze odchylenia 1,2–1,7 bez jednego winnego etapu** (Rust: macedoński w polskim 1,55, litewski 1,24,
  portugalski w malgaskim 1,67; C#: rumuński i rosyjski w malgaskim 1,60 i 1,42). W macedońskim i litewskim
  tokenizacja i wzorce Rusta są jak w Javie, więc różnica leży poza etapami tekstowymi; na portugalskim w malgaskim
  Rust ma i tokenizację, i wzorce mniej więcej dwa razy dłuższe niż Java. Bez jednego etapu, który by to tłumaczył,
  traktujemy te liczby raczej jako zakłócenie przebiegu niż cechę języka; nie powtarzaliśmy tych pomiarów.

TypeScript jest najwolniejszy w każdym języku dokumentu (2,5–11,0 Javy); najbliżej Javy na francuskim w polskim
pipeline (2,48), najdalej na tureckim w malgaskim (11,03). Kotlin i Scala zwykle stoją w 0,9–1,5 Javy bez wyraźnego
wzoru.

Wniosek: kolejność implementacji zależy od języka dokumentu mniej niż od pipeline'u. W polskim pipeline Rust jest
najszybszy na 17 z 27 języków; w malgaskim trzy implementacje dzielą prowadzenie. Jedyne duże, wyjaśnione
odchylenie to wzorce w Go na tekście w języku pipeline'u, czyli dokładnie tam, gdzie pipeline ma najwięcej pracy.
