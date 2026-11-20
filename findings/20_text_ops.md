# Operacje tekstowe per język programowania

> 341 konstytucji, jeden klient, cache werdyktów wyłączony, dwa przejścia (mierzone drugie), 3 rundy w przeplocie
> (mediana); ten sam skrót raportów we wszystkich 54 przebiegach (`../data/text_ops.csv`). Czas etapu = ms na całe
> przejście.

Etapy tekstowe to znacząca część dokumentu: w Javie tokenizacja i wzorce iniekcji zajmują 16 i 20% czasu dokumentu
w polskim (344 i 416 ms na przejście) oraz 20 i 15% w malgaskim (288 i 214 ms) (`../data/text_ops.csv`). Wzorce to
wyrażenia regularne o semantyce Javy. Java, Kotlin i Scala używają `java.util.regex`; Rust, C#, Go, C++ i TypeScript
korzystają z własnego odpowiednika tego podzbioru składni, napisanego przez AI, bo wbudowane silniki tych języków inaczej traktują
wielkość liter, granice słów i klasy Unicode, a bramka wymaga raportów identycznych z Javą, łącznie z pozycjami znaków
w UTF-16 (`09_ai_work.md`).

| etap | Java, ms | najszybsze (× Javy) | wolniejsze od Javy |
|---|---:|---|---|
| tokenizacja, polski | 344 | Rust 0,49 · Go 0,50 · C++ 0,58 · C# 0,59 · Java natywnie 0,70 | Scala 1,10 · TypeScript 3,51 |
| tokenizacja, malgaski | 288 | C++ 0,66 · C# 0,73 · Go 0,80 · Java natywnie 0,86 · Rust 0,90 | Kotlin 1,27 · Scala 1,67 · TypeScript 4,46 |
| wzorce, polski | 416 | Rust 0,60 · C# 0,61 · C++ 0,63 · Scala 0,85 · Go 0,90 | TypeScript 4,87 |
| wzorce, malgaski | 214 | C# 0,97 · Scala 1,00 | Go 1,05 · Kotlin 1,09 · Java natywnie 1,12 · C++ 1,22 · Rust 1,33 · TypeScript 7,49 |
| skróty (kropka po skrócie), polski | 39 | C# 0,54 · C++ 0,58 · Rust 0,66 | Scala 1,21 · Kotlin 1,52 · TypeScript 1,92 |
| znaki niewidzialne, malgaski | 15 | C# 0,40 · Go 0,53 · C++ 0,80 · Rust 0,81 | Kotlin 1,15 · Scala 1,22 · TypeScript 8,43 |

Trzy obserwacje:

- **Tokenizacja** (podział na słowa, pismo każdego znaku, małe litery) jest w Ruście, Go, C++ i C# 1,1–2,0 raza
  szybsza niż w Javie JIT; to najprostszy etap i najbardziej zależy od tego, jak tanio język przechodzi po znakach.
- **Wzorce:** na 184 wzorcach frazowych malgaskich `java.util.regex` jest równy najlepszemu z własnych silników
  (C# 0,97) i szybszy od pozostałych; na 24 wzorcach polskich własne silniki Rusta, C# i C++ są o 37–40% szybsze.
  Kolejność implementacji zależy więc od zestawu wzorców, nie tylko od języka; przyczyny tej różnicy nie badaliśmy.
- **TypeScript** jest 1,9–8,4 raza wolniejszy od Javy w każdym etapie tekstowym, najbardziej we wzorcach malgaskich
  (7,5) i w znakach niewidzialnych (8,4).

Kotlin i Scala używają tego samego `java.util.regex` i dostają we wzorcach 0,85–1,09 Javy, więc ich różnice wobec
Javy mogą pochodzić tylko z kodu wokół wyrażeń (filtr, pętla po słowach), nie z silnika (`../data/text_ops.csv`).

Wniosek: przy operacjach tekstowych liczy się nie tylko język, ale też to, czyj silnik wykonuje wyrażenia regularne.
Wbudowany silnik Javy dorównuje najlepszym własnym silnikom na długich frazach i przegrywa z nimi na krótkich
polskich wzorcach. Etapy dla każdego języka dokumentu osobno: `21_doc_language.md`.
