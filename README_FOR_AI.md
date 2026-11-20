# Instrukcja dla agenta AI

Ten katalog to kompletny materiał jednego projektu badawczego, przygotowany tak, żeby agent AI mógł go przedstawić
człowiekowi w dowolnej formie: streszczenie, porównanie, odpowiedź na pytanie, slajdy, artykuł. Człowiek może też
przejrzeć wyniki sam: `index_offline.html` (filtry, działa otwarty wprost z dysku) albo `wyniki.pdf` (do czytania
i druku).

## O czym jest projekt

Jedna biblioteka (wykrywanie języka i wstawek iniekcji w tekście, algorytm selekcji negatywnej — NSA) napisana
w Javie, a potem przeniesiona do kilku innych języków: Kotlin, Scala, Rust, C#, Go, C++, TypeScript (Node i Bun),
także jako obrazy natywne (GraalVM, NativeAOT) i WebAssembly; osobno sprawdzono wołanie najszybszej implementacji
(Rust) z Pythona przez interfejs C — tak, jak Python zwykle korzysta z bibliotek natywnych. Każdy port daje **bit w bit te same odpowiedzi** co Java — to
sprawdza bramka równoważności. Porty, narzędzia i pomiary napisało AI (Claude), a człowiek wyznaczał cel i sprawdzał
wyniki; przebieg tej pracy i werdykt AI opisują `findings/09` i `findings/10`. Potem porównano wydajność, pamięć, zachowanie pod obciążeniem i przy długim
(do godziny) działaniu, koszt wytworzenia, asembler i GPU (także trening detektorów na karcie graficznej).

## Architektura w skrócie

Pełny opis z diagramami: `ARCHITECTURE.md`. Najważniejsze, żeby wyjaśnić człowiekowi, o co chodzi:

- **Zadanie:** w tekście w oczekiwanym języku (polski albo malgaski) oznaczyć słowa i fragmenty, które do niego nie
  pasują: wstawki w innym języku, zmianę pisma, homoglify, podróbki marek, frazy iniekcji, znaki niewidzialne. Cel to
  filtr wstępny tekstów, które idą potem do modelu językowego, bez modelu językowego w czasie działania.
- **Algorytm:** NSA uczy się tylko słów „swoich” (kategorie tematyczne po ok. 150 słów) i generuje detektory poza nimi;
  słowo, do którego pasują detektory we wszystkich kategoriach, jest obce. NSA to ostatni etap. Wcześniej, w stałej
  kolejności, działają słowniki i reguły, a pole `via` w odpowiedzi mówi, który etap zdecydował.
- **Dokument:** pierwsze przejście (werdykt każdego różnego słowa) → liczniki → progi dokumentu → sygnały → raport JSON.
- **Moduły:** rdzeń bez danych, dwie biblioteki (pl, mg), dane w osobnych jarach; serwisy REST, obserwator folderu
  i strumień to tylko przykłady użycia.
- **Historia:** eksperyment na malgaskim → serwis i polski → bramki jakości → dwa programy i czysta biblioteka →
  wydajność → porty. Java jest specyfikacją: port jest poprawny tylko przy 0 różnic w raportach.
- **Kontrola:** testy jednostkowe w każdym języku (1065 funkcji testowych), bramki jakości na żywych serwisach,
  bramka równoważności z fuzzingiem, pomiary ze skrótem wyników; bramki i pomiary to skrypty Pythona (124 pliki,
  `TESTING.md`). Python jest językiem narzędzi, nie mierzoną implementacją.
- **Granica:** detektor widzi język, nie intencję (`ARCHITECTURE.md`, „Granice”).
- Wyjaśniając architekturę, pokazuj diagramy z `diagrams/` (SVG na jasnym tle) i nie dopisuj modułów, etapów ani
  decyzji, których nie ma w `ARCHITECTURE.md`.

## Mapa plików

| plik / katalog | zawartość |
|---|---|
| `METHODOLOGY.md` | jak mierzono: bramki, warunki maszyny, rundy, mediany — **przeczytaj przed cytowaniem liczb** |
| `MACHINE.md` | sprzęt i wersje narzędzi, na których powstały wyniki |
| `GLOSSARY.md` | słownik pojęć (PGO, JIT, AOT, GC, p99, bramka, NFKC…) — używaj tych znaczeń, tłumacząc wyniki |
| `findings/*.md` | krótkie wnioski per temat, każdy z odwołaniem do plików danych; **zacznij od `findings/00_summary.md`** (podsumowanie i porady per język) |
| `opis.html` | wersja opisowa dla człowieka w przeglądarce, wygenerowana z tych samych plików `.md` co PDF (bez wykresów i przekrojów tabel); kotwice rozdziałów np. `opis.html#f01`, `#a-arch`, `#k-data` — odsyłaj do nich człowieka zamiast do surowego Markdown |
| `README.md` | strona startowa repozytorium dla człowieka: co to jest, od czego zacząć, najważniejsze wyniki, licencja |
| `LICENSE` | licencja całego materiału: CC BY 4.0 (tekst prawny po angielsku) |
| `ARCHITECTURE.md` | **co robi biblioteka i jak jest zbudowana**: moduły, algorytm NSA, etapy klasyfikacji słowa i dokumentu, porty i bramki, historia wersji w Javie, decyzje projektowe, jak pracowało AI, granice metody; diagramy w `diagrams/*.svg` |
| `TESTING.md` | **co pilnowało jakości**: testy jednostkowe w każdym języku (liczby, rodzaje: złote pliki, nowa ścieżka = stara, referencje JDK, pokrycie), bramki jakości z materiałem i stanem odniesienia, równoważność i fuzzing, narzędzia pomiarowe, baza skryptów Pythona |
| `optimizations/*.md` | co pomogło, a co nie, w każdej implementacji: tabele zmian z efektem, odrzucone pomysły z przyczyną, krótkie przykłady kodu (`00_common.md` = zmiany wspólne dla wszystkich) |
| `data/*.csv` | wyniki liczbowe (jedna tabela na plik, nagłówek w pierwszym wierszu) |
| `data/README.md` | opis każdej kolumny każdego pliku CSV |
| `slides/` | przykładowe slajdy (`example_deck.html`) i instrukcja zrobienia własnych (`HOW_TO_MAKE_YOUR_OWN.md`) |
| `index.html` | interaktywna strona z tymi samymi danymi: każda tabela z filtrami (język tekstu, język programowania, implementacja, scenariusz, seria) i sortowaniem; czyta `data/*.csv`, więc wymaga serwera plików statycznych |
| `index_offline.html` | ta sama strona w jednym pliku, z danymi wbudowanymi w środek — działa otwarta wprost z dysku |
| `charts.html` | 27 wykresów (czas, pamięć, start, tłok, scenariusze, krzywa opóźnienia, wspólny serwer, godzina pracy, odśmiecacze Javy, OpenJ9, GPU, praca AI, szacunek ręczny, tokeny kodu, detektory, porty między językami, czytelność, prompty, operacje tekstowe, język dokumentu, wstawka obcojęzyczna) liczonych z `data/*.csv`; działa otwarty z dysku, podaje źródło każdego wykresu |
| `wyniki.pdf` | **wersja opisowa** do czytania i druku (treść wszystkich dokumentów `.md` oprócz spisów treści, sformatowana): architektura z diagramami, wnioski, optymalizacje per język, metodologia, maszyna, wybrane przekroje każdej tabeli (podpis podaje filtry), opis kolumn; zbudowana z tych samych plików, pełne tabele są tylko w `data/` |

## Zasady (obowiązkowe)

1. **Cytuj liczby wyłącznie z plików w `data/`, `findings/`, `optimizations/` oraz z `ARCHITECTURE.md` i `TESTING.md`.** Nie zgaduj, nie przeliczaj
   z pamięci. Podawaj plik źródłowy przy każdej liczbie.
2. **Rozróżniaj warunki pomiaru.** Wolna maszyna, tłok na CPU, ciasna pamięć, jeden klient albo strumień żądań
   dają różne liczby, a każdy plik ma własne odniesienie (`data/README.md`); porównuj tylko wiersze z tych samych
   warunków.
3. **Wyniki są względne do jednej maszyny** (`MACHINE.md`). Podawaj krotności („0,7× Javy”) zamiast obietnic
   bezwzględnych; inny sprzęt da inne liczby.
4. **Równoważność nie zależy od sprzętu** — „8 implementacji, te same odpowiedzi” to wynik twardy, wydajność nie.
5. **Koszt wytworzenia portów zależy od kolejności ich pisania** (`findings/` o koszcie) — nie wyciągaj z czasu pracy
   wniosku „język X jest tańszy”.
6. Materiał nie zawiera kodu źródłowego bibliotek. Jedyny kod to krótkie, skrócone fragmenty w `optimizations/`,
   ilustrujące technikę optymalizacji; nie przedstawiaj ich jako kompletnej implementacji.
7. **`findings/10_ai_verdict.md`, `findings/23_text_languages.md` i `findings/24_language_design.md` to opinie AI,
   które napisało porty**, oparta na danych z tego katalogu — przy
   cytowaniu przedstawiaj ją jako opinię, nie jako wynik pomiaru.
8. **Licencja CC BY 4.0:** materiał wolno wykorzystywać, także komercyjnie. Przytaczając go albo robiąc z niego
   coś nowego (artykuł, slajdy, streszczenie do publikacji), podaj autora (Tomasz Bigoś), tytuł „Jedna biblioteka,
   osiem języków”, licencję CC BY 4.0, adres https://github.com/TomaszBigos/one-library-eight-languages i zaznacz,
   że materiał został zmieniony lub skrócony.

## Przykładowe pytania, na które materiał odpowiada

- Który język był najszybszy, a który zużywał najmniej pamięci — i na jakich warunkach?
- Czy ręczny asembler przyspieszył kod? Dlaczego zysk zniknął po poprawce w C++?
- Ile daje GPU wobec dobrego algorytmu na CPU?
- Jak zachowują się języki, gdy kilka programów dzieli jedną maszynę?
- Ile kosztuje wołanie biblioteki w Ruście z Pythona i czy Python wtedy korzysta z wielu rdzeni?
- Czy po godzinie działania JIT Javy dogania obraz natywny albo Rusta?
- Jak wyglądała praca AI nad portem, z czym się zmagało i ile razy wracało do każdego języka?
- Który język najlepiej współpracował z AI (kompilator, testy, profiler) i jaki język wybrać według AI?
- Ile było optymalizacji w każdym języku i które działały tylko w danym języku?
- Jak każdy język zarządza pamięcią i wątkami i co to zmieniało w wynikach?
- Ile trwałby każdy port bez AI? (szacunek z jawnych założeń, `findings/13`; przedstawiaj jako szacunek)
- Jak wyglądały przekłamania pamięci i jakie były inne zaskoczenia po drodze? (`findings/14`)
- Ile tokenów zajmuje kod każdej implementacji i czy cały materiał zmieści się w kontekście? (`findings/15`)
- Jak wyglądają same detektory NSA i ile ich jest? (`findings/16`)
- Jak wyglądała rozmowa z AI — ile promptów, jak krótkich, ile pracy po jednym? (`findings/17`)
- Czy warto portować z języka X na Y i z czego portować? (`findings/18`, `data/port_matrix.csv`)
- Który kod jest najczytelniejszy według metryk? (`findings/19`; przedstawiaj jako przybliżenie)
- Który język najlepiej pasuje do pracy z tekstem i do NSA? (`findings/23`; przedstawiaj jako opinię)
- Jaki język warto by zaprojektować po takim porównaniu? (`findings/24`; opinia, projekt na papierze)
- Co robi biblioteka, z czego się składa i jak działa NSA? (`ARCHITECTURE.md`, diagramy `diagrams/01`–`04`)
- Jakie testy pilnowały jakości i jakimi narzędziami mierzono? (`TESTING.md`, diagram `diagrams/07_testing.svg`)
- Jak powstawała wersja w Javie i dlaczego tak? (`ARCHITECTURE.md`, „Jak powstawała wersja w Javie”, „Decyzje projektowe”)
- Co przyspieszyło implementację w języku X, a co nie dało nic? (`optimizations/0N_<język>.md`, wspólne zmiany w `optimizations/00_common.md`)
- Ile kosztuje akapit w obcym języku w tekście i czy biblioteka go zauważa? (`findings/22`)
