# Jedna biblioteka, osiem języków

Ta sama biblioteka (wykrywanie języka tekstu i prób iniekcji w tekstach polskich i malgaskich, algorytm selekcji
negatywnej) napisana w Javie, a potem przeniesiona do Kotlina, Scali, Rusta, C#, Go, C++ i TypeScriptu. Każda
implementacja daje bit w bit te same odpowiedzi co Java; różnią się czasem, pamięcią, startem i zachowaniem pod
obciążeniem. Porty, narzędzia i pomiary napisał agent AI (Claude), a autor wyznaczał cel, podejmował decyzje
i sprawdzał wyniki. To repozytorium zawiera wyniki, opisy i dane; nie zawiera kodu bibliotek.

Repozytorium: <https://github.com/TomaszBigos/one-library-eight-languages>. Strony HTML na stronie repozytorium widać
jako kod; do czytania w przeglądarce służy wersja przez GitHub Pages:
<https://tomaszbigos.github.io/one-library-eight-languages/> (albo pobrane pliki otwarte z dysku).

## Od czego zacząć

| plik | dla kogo |
|---|---|
| [`opis.html`](opis.html) | **wersja opisowa w przeglądarce**: architektura, testy, wnioski, optymalizacje per język, metodologia, opis kolumn, słownik pojęć (z diagramami) |
| [`wyniki.pdf`](wyniki.pdf) | ta sama wersja opisowa do czytania i druku, dodatkowo z wykresami i wybranymi przekrojami tabel |
| [`index_offline.html`](index_offline.html) | wszystkie tabele z filtrami i sortowaniem; pobrany plik działa otwarty wprost z dysku |
| [`charts.html`](charts.html) | wykresy liczone z tabel |
| [`findings/00_summary.md`](findings/00_summary.md) | najkrótsza wersja: siedem wniosków i porady per język |
| [`ARCHITECTURE.md`](ARCHITECTURE.md) | co robi biblioteka, jak jest zbudowana, jak powstawała, jakie decyzje ją ukształtowały |
| [`README_FOR_AI.md`](README_FOR_AI.md) | instrukcja dla agenta AI: można dać mu cały katalog i poprosić o streszczenie, porównanie albo odpowiedź na pytanie |

## Najważniejsze wyniki

- **Te same odpowiedzi w każdym języku.** Osiem implementacji daje te same 25 850 raportów co Java, a bramka
  równoważności złapała błędy, których kompilatory nie widziały (`findings/01_equivalence.md`).
- **Czwórka bez wyraźnego zwycięzcy.** Rust, Go, C# i C++ potrzebują 0,72–0,75 czasu Javy JIT i różnią się między
  sobą o mniej niż 10% (`data/language_summary.csv`).
- **Pamięć i start to osobna oś.** Najmniej pamięci zajmują obrazy natywne JVM i Rust; Rust startuje w 0,7 s wobec
  3,8 s Javy JIT (`findings/02_speed_memory.md`).
- **Warunki pracy zmieniają kolejność.** Przy limicie 200 MB Java JIT robi 42 dokumenty na sekundę, a Rust 152; na
  jednym procesorze czwórka wyraźnie wyprzedza Javę (`findings/03_contention.md`).
- **AI przepisuje bibliotekę w godziny, nie w tygodnie,** pod warunkiem że istnieje referencja i automatyczna bramka
  (`findings/06_port_cost.md`, `findings/13_manual_estimate.md`).
- **Przenosiły się algorytmy, nie sztuczki.** Najwięcej dały zmiany algorytmiczne wdrożone we wszystkich językach;
  cechy języka decydowały o pamięci i pułapkach (`optimizations/`).

## Zawartość

| katalog / plik | zawartość |
|---|---|
| `data/` | wyniki liczbowe, jedna tabela CSV na plik; opis kolumn w `data/README.md` |
| `findings/` | wnioski per temat, każdy z odwołaniem do danych |
| `optimizations/` | co pomogło, a co nie, w każdej implementacji, z krótkimi fragmentami kodu |
| `diagrams/` | diagramy architektury (SVG) |
| `slides/` | przykładowe slajdy i instrukcja zrobienia własnych |
| `ARCHITECTURE.md`, `TESTING.md` | architektura i historia; testy, bramki jakości i narzędzia pomiarowe |
| `METHODOLOGY.md`, `MACHINE.md`, `GLOSSARY.md` | jak mierzono, na jakim sprzęcie, słownik pojęć |
| `index.html` | ta sama strona co `index_offline.html`, ale czyta pliki CSV, więc wymaga serwera plików (np. GitHub Pages) |

## Zastrzeżenia

- Wszystkie pomiary pochodzą z jednej, nieizolowanej maszyny (`MACHINE.md`). Wyniki są względne: krotności wobec
  Javy przenoszą się lepiej niż czasy bezwzględne. Równoważność odpowiedzi od sprzętu nie zależy.
- Kodu bibliotek tu nie ma; fragmenty w `optimizations/` są skrócone i ilustrują technikę. Nie ma też tekstów
  korpusów użytych do pomiarów (m.in. konstytucje z Wikisource), tylko wyniki na nich.
- Opinie AI (`findings/10_ai_verdict.md`, `findings/23_text_languages.md`, `findings/24_language_design.md`, porady
  w `findings/00_summary.md`) są opiniami, nie wynikami pomiaru.

## Licencja

Tekst, dane, diagramy i fragmenty kodu w tym repozytorium: **Creative Commons Attribution 4.0 International
(CC BY 4.0)**, pełny tekst w [`LICENSE`](LICENSE). Wolno kopiować, rozpowszechniać, zmieniać i wykorzystywać, także
komercyjnie, pod warunkiem podania autora, licencji i informacji o zmianach.

Proponowane oznaczenie: *Tomasz Bigoś, „Jedna biblioteka, osiem języków”, licencja CC BY 4.0,
https://github.com/TomaszBigos/one-library-eight-languages*.
