# Prompty: jak rozmawiano z AI

> Liczone z zapisów sesji od pierwszego portu; publikowane są tylko liczby, bez treści promptów
> (`../data/prompts.csv`). Prompt to wiadomość człowieka. Nie liczą się wyniki narzędzi, komunikaty systemowe ani
> automatyczne powiadomienia o zakończonych zadaniach.

## Liczby

- **Prompty:** 427, razem 5310 słów. Mediana to 9 słów, 90% promptów ma do 24 słów.
- **Krótkie i długie:** co czwarty prompt (26%) ma najwyżej 5 słów, a tylko 2% ma 50 słów albo więcej.
- **Pytania:** prawie połowa promptów (48%) zawiera pytanie (`../data/prompts.csv`).
- **Praca AI po prompcie:** mediana to 2 wywołania narzędzi. 10% promptów uruchomiło ponad 50 wywołań, a jeden 318.
  Aktywna praca AI po jednym prompcie trwała od ok. 36 sekund (mediana) do ok. 157 minut (`../data/prompts.csv`).
- **Dźwignia:** na jedno słowo promptu przypada ok. 1555 tokenów napisanych przez AI (`../data/prompts.csv`).
- **Języki w promptach:**

  | język | prompty |
  |---|---|
  | Java | 32 |
  | C# | 32 |
  | Rust | 28 |
  | Go | 20 |
  | Kotlin | 18 |
  | Scala | 18 |
  | C++ | 9 |
  | TypeScript | 7 |
  | Python | 4 |

  C# pojawia się tak często jak Java, bo miał najdłuższą serię optymalizacji i strojenia GC (`../data/prompts.csv`,
  `11_optimizations.md`).

## Co z tego wynika

Rozmowa była krótka, a praca długa. Typowy prompt to jedno zdanie: decyzja („tak, zrób”), pytanie o stan albo
nowy kierunek. Na takie zdanie AI potrafiło wykonać kilkadziesiąt kroków: zbudować, przetestować, puścić bramkę,
zmierzyć i zapisać wynik. Działało to dlatego, że kontekst nie siedział w promptach, tylko w repozytorium:
- instrukcje projektu (co gdzie leży, czym uruchamiać, jakie są pułapki);
- dzienniki portów i lista zadań;
- automatyczne bramki, które mówiły, czy wynik jest poprawny.

Prawie połowa promptów to pytania: człowiek częściej sprawdzał i decydował, niż dyktował rozwiązanie.

**Porada:** przy pracy z AI na dużym projekcie opłaca się inwestować w pliki opisujące projekt i w automatyczne
sprawdzanie, a nie w długie prompty. Krótki prompt wystarczy, jeśli AI ma skąd wziąć resztę.
