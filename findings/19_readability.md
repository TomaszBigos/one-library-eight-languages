# Jak wygląda kod: metryki i ranking czytelności

> Liczone z kodu bibliotek każdej implementacji, bez testów (`../data/code_metrics.csv`). To metryki wyglądu
> kodu, nie ocena jego jakości. Wszystkie porty pisało to samo AI według tej samej struktury, więc różnice pochodzą
> głównie z języka i jego zwyczajów (formatowanie, klasy, operatory).

## Miary

| język | długość linii | linie > 100 znaków | symbole | wcięcie | komentarze | długość nazw | linie na funkcję |
|---|---|---|---|---|---|---|---|
| Go | 24,6 | 2,5% | 21,5% | 1,60 | 5,6% | 4,8 | 18,0 |
| Java | 36,9 | 2,3% | 17,9% | 2,42 | 34,2% | 6,5 | 13,9 |
| Rust | 30,4 | 2,4% | 24,1% | 2,09 | 8,6% | 5,5 | 15,8 |
| C# | 33,1 | 3,8% | 19,3% | 2,31 | 11,2% | 5,7 | 18,3 |
| C++ | 36,8 | 4,7% | 22,1% | 2,03 | 5,5% | 5,3 | 17,9 |
| Kotlin | 39,4 | 4,8% | 16,6% | 2,53 | 18,6% | 6,3 | 13,9 |
| TypeScript | 37,3 | 6,5% | 23,7% | 1,92 | 4,5% | 4,8 | 12,6 |
| Scala | 45,0 | 8,3% | 15,7% | 2,72 | 18,0% | 6,1 | 10,5 |

(`../data/code_metrics.csv`; długość linii i nazw w znakach, symbole = znaki niealfanumeryczne wśród niebiałych,
wcięcie = średni poziom, liczba funkcji z prostych wzorców — przybliżenie.)

## Ranking

Średnia z rang czterech miar: krótsze linie, mniej symboli, płytsze wcięcia, mniej linii ponad 100 znaków.

| miejsce | język |
|---|---|
| 1 | Go |
| 2 | Java |
| 3 | Rust |
| 4 | C# |
| 5 | C++ |
| 6 | Kotlin |
| 7 | TypeScript |
| 8 | Scala |

(`../data/code_metrics.csv`)

## Co z tego wynika

- **Go** ma najkrótsze linie i najpłytsze wcięcia: formatowanie jest wymuszone, nie ma klas, a funkcje leżą na
  najwyższym poziomie. Ten sam Go najczęściej odrzucał kod AI przy kompilacji (`09_ai_work.md`): prosty wygląd nie
  oznacza łatwego pisania.
- **Rust** ma najwięcej symboli (24,1%): referencje, ścieżki `::`, operator `?`, typy generyczne. Kod jest zwarty,
  ale gęsty.
- **Scala** ma najmniej symboli, ale najdłuższe linie i najgłębsze wcięcia, bo idiomem są długie łańcuchy wywołań.
  Kotlin wypada podobnie, łagodniej.
- **Java** ma najwięcej komentarzy (34%). To oryginał, w którym zostały opisy decyzji i pomiarów; porty dostały
  ich mniej.
- **TypeScript** ma najmniej komentarzy, krótkie funkcje (krótsze ma tylko Scala) i najwięcej długich linii poza
  Scalą.

**Opinia AI:** najłatwiej czytało mi się Kotlin i C#, bo są bliskie Javie i używają jawnych typów. Rust był czytelny dzięki
jawności (własność, błędy w typie wyniku), mimo dużej gęstości symboli. Najtrudniejsza była Scala, bo długie łańcuchy
kolekcji trzeba rozwinąć w głowie, żeby zobaczyć koszt. Ranking z metryk i ta opinia się różnią — metryki mierzą
wygląd, a czytelność zależy też od tego, ile trzeba wiedzieć o języku.
