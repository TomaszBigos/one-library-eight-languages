# Jak AI pracowało nad portami

> Pisane z perspektywy AI (Claude), które napisało porty, narzędzia pomiarowe i bramki. Człowiek wyznaczał cel,
> podejmował decyzje (zgodność bit w bit, warunki pomiaru, zgody na instalację narzędzi) i sprawdzał wyniki. Liczby
> z zapisów sesji obejmują pracę od pierwszego portu; wcześniejsza praca nad samą Javą nie ma już zapisów.

## Skala

Osiem języków, dwie biblioteki (polska i malgaska), 18 implementacji w macierzy (JIT, obrazy natywne, z profilem
kompilacji i bez), 46 629 linii kodu bibliotek, 15 371 linii testów i 1065 funkcji testowych
(`../data/ai_scale.csv`). Każda implementacja musiała dać te same 25 850 raportów co Java. Od pierwszego portu AI
pracowało przez 28 dni, ok. 81 godzin aktywnie, w tym ok. 23 godziny na narzędzia, pomiary, dane i dokumentację, a nie
na kod bibliotek. W tym czasie napisało ok. 8,0 mln tokenów i przeczytało ok. 3,1 mld tokenów kontekstu (głównie ten
sam kontekst rozmowy czytany ponownie). Wykonało 1203 edycje plików źródłowych, 840 budowań i przebiegów testów
oraz 284 przebiegi bramki równoważności (`../data/ai_scale.csv`). Projekt ma 117 skryptów w Pythonie (bramki,
pomiary, buildery danych, eksport) i 8356 zapisanych wyników pomiarów i bramek (`../data/ai_scale.csv`).

## Jak wyglądał port

Każdy port szedł tym samym przepisem: najpierw środowisko i szkielet, potem trening detektorów dający pliki bajt
w bajt takie jak w Javie, potem tekst (pozycje znaków, normalizacja Unicode, wyrażenia regularne), biblioteka
malgaska, biblioteka polska, program pomiarowy, pomiar wyjściowy i dopiero wtedy rundy optymalizacji. Port był
skończony dopiero przy 0 różnic na bramce, nie wtedy, gdy się kompilował. Źródło portu zmieniało się w czasie: Kotlin,
Scala i Rust powstały z Javy, C# z Rusta, Go z Rusta i C#, C++ z Rusta, a TypeScript z C++ (`../data/ai_work.csv`).
Zawsze brany był port już udowodniony równy Javie i najbliższy nowemu językowi. Rust, pierwszy język spoza JVM,
zbudował to, czego potrzebowały wszystkie następne (patrz niżej). Dlatego późniejsze porty zajmowały
2,7–4,8 godziny pracy AI razem z optymalizacją, Rust 9,0 (`../data/port_cost.csv`). Prosty, „dosłowny” port był
zwykle wyraźnie wolniejszy od Javy; szybkość przychodziła dopiero z rund optymalizacji.

## Ukryta połowa pracy: odtworzyć Javę

Największy koszt zgodności bit w bit nie leżał w algorytmie, tylko w zachowaniu platformy, które w Javie jest
niewidoczne. W każdym języku spoza JVM trzeba było odtworzyć:
- napisy liczone w jednostkach UTF-16, bo raport podaje pozycje tak jak Java (w Ruście, Go i C++ napisy są inne;
  C# i JavaScript, czyli też TypeScript, używają takich samych);
- właściwości znaków i normalizację NFKC z danych JDK, bo .NET, Go i biblioteki Rusta używają innej wersji Unicode;
- semantykę wyrażeń regularnych Javy: silnik napisany raz w Ruście, a potem przeniesiony do C#, Go, C++
  i TypeScriptu, bo żaden z wbudowanych silników nie daje tych samych dopasowań (inne `\w`, `\b`, surogaty, brak
  nawrotów w Go);
- generator liczb losowych Javy krok w krok (od niego zależą wytrenowane detektory) i 64-bitową arytmetykę, której
  JavaScript nie ma bez typu BigInt;
- zapis UTF-8 samotnego surogatu tak jak w Javie i kolejność iteracji wszędzie tam, gdzie wpływa na wynik.

Port działał też w drugą stronę. Przeniesienie na Kotlina i Scalę ujawniło dwie niedeterministyczne kolejności
w samej Javie, wynikające z kolejności iteracji kolekcji. Późniejsze przeglądy znalazły jeszcze remis odległości
rozstrzygany inaczej przy każdym starcie i zmianę wielkości liter zależną od ustawień regionalnych systemu (pod
tureckimi ustawieniami polski tekst stawał się „obcy”). Wszystko to naprawiono w Javie, zanim poszły następne porty.

## Jak każdy język współpracował z AI

| język | z czego | odrzucenia kompilatora / budowania | niezaliczone testy | co pomagało | co przeszkadzało |
|---|---|---|---|---|---|
| Java | — | 4 / 165 | 16 | JFR, dojrzały Maven, statyczny zakaz wywołań zależnych od ustawień regionalnych | kolejność `Set.of` zmienna między startami; duża metoda bez kompilacji C2; artefakty profilera |
| Kotlin | Java | 0 / 70 | 2 | składnia blisko Javy, ten sam profiler | starsza biblioteka standardowa wciągana przez zależności frameworka; jedna kompilacja C2 zmieniała szczyt pamięci |
| Scala | Java | 4 / 33 | 2 | kompilator łapie błędy typów | wolna kompilacja; kolekcje Scali w gorącej ścieżce kosztują; „Tests run: 0” przy zielonym budowaniu |
| Rust | Java | 16 / 215 | 7 | kompilator łapie błędy pamięci i współdzielenia; `cargo test` | brak profilera pod Windows (własne liczniki z narzutem); zmiana toolchainu na MSVC |
| C# | Rust | 9 / 146 | 0 | napisy UTF-16 jak w Javie; dobre komunikaty błędów | ustawienia odśmiecania (regiony trzymały pamięć ponad limitem); NativeAOT wymaga narzędzi MSVC |
| Go | Rust, C# | 12 / 76 | 6 | wbudowany profiler `pprof`; zero zależności; szybki build | nieużyty import to błąd (częste przy edycjach przyrostowych); GC niegeneracyjny |
| C++ | Rust | 10 / 107 | 3 | AddressSanitizer, listing asemblera, PGO | błąd czasu życia obiektu przechodzi kompilację bez ostrzeżenia; własny profiler |
| TypeScript | C++ | 0 / 28 | 4 | uruchamiany wprost, bez budowania | typy nie są sprawdzane przy uruchomieniu (błąd typu wychodzi w działaniu); limit sterty V8; brak 64-bitowych liczb całkowitych |

Liczby w tabeli to dolna granica (`../data/ai_work.csv`): wynik budowania był często skracany do kilku ostatnich
linii. Najwięcej odrzuceń na budowanie miały Go (12 / 76) i Scala (4 / 33): w Go nieużyty import albo zmienna jest
błędem kompilacji, a AI zmienia kod kawałkami. Rust (16 / 215) odrzucał mniej względnie, ale jego odrzucenia częściej
dotyczyły prawdziwego problemu, np. domknięcia trzymającego wyłączną referencję do bufora czytanego później
(`07_compiler_vs_gate.md`). Kotlin i TypeScript nie miały ani jednego odrzucenia, z różnych powodów. Kotlin był
pierwszym portem, prawie zdanie w zdanie z Javy. TypeScript nie ma etapu sprawdzania typów: kod uruchamiany
wprost w Node zgłosił błąd typu dopiero w działaniu (`runtime_fail` w `../data/ai_work.csv`). W C++ najgroźniejszy błąd
(widok na tymczasowy napis, 2658 różnych raportów) przeszedł kompilację i złapała go dopiero bramka
(`gate_runs_with_diff` w `../data/ai_work.csv`). W Scali i Kotlinie pierwsze bramki pokazały różnice w odpowiedziach
na pojedyncze słowa; w Kotlinie była to jedna różnica, czyli kolejność wynikająca z samej Javy. Bramka dla C# szła
najczęściej (80 przebiegów), bo C# miał najdłuższy ogon optymalizacji.

## Optymalizacje i powroty

Każdy język dostawał rundy optymalizacji. Rust miał 8 własnych rund (kroki R1–R50), C# 13 (C1–C16), Go 5 (G1–G19),
C++ 2 (C1–C10), TypeScript 1 (T1–T9). Java przed portami przeszła 7 dni optymalizacji (#0–#35), a potem razem
z Kotlinem i Scalą 13 wspólnych rund, każda w kolejności Java → Kotlin → Scala (`../data/ai_work.csv`). Powroty
policzone z zapisów sesji: Rust miał 6 osobnych okresów pracy (5 powrotów), Java 5, Scala i C# po 4, Kotlin, Go
i C++ po 3, a TypeScript 2 (`../data/ai_work.csv`). Powody powrotów były trzy: nowy pomiar pokazywał lukę (tłok na
procesorze, ciasna pamięć), lekcja z innego języka dawała się przenieść albo przegląd kodu wszystkich języków
znajdował ten sam koszt w kilku miejscach naraz.

Optymalizacje wędrowały między językami. Pomysł znaleziony w jednym porcie był przenoszony do pozostałych, żeby
porównanie zostało uczciwe: wątek wywołujący pracuje razem z pomocnikami (z C# do Rusta), słownik obcy w jednym
bloku pamięci (z Rusta do Go), homoglify i zwijanie wielkości liter jednym przejściem (z Go do C# i Rusta), indeksy
detektorów i segmentów literówek (z Javy do wszystkich). Dlatego C++ i TypeScript skończyły na jednej, dwóch rundach:
dostały gotową listę zmian sprawdzonych gdzie indziej. Liczba rund mówi więc o kolejności pisania i o tym, ile
wymagał dany runtime, a nie o tym, jak trudny jest język.

Część pomysłów pomiar obalił: filtr Blooma w Go (odczyt rozproszonej mapy kosztował więcej, niż oszczędzał),
interner tokenów w C# (zysk poniżej progu), kilka zmian w Ruście i Javie, które poprawiały profil, ale nie czas.
Odrzucone łatki zostały zachowane, bo „odrzucone pomiarem” jest względne: indeks segmentów literówek odrzucony na
początku dał później ok. jednej trzeciej CPU w polskim, gdy reszta kosztu zmalała.

## Z czym AI się zmagało

- **Równoważność.** Każda niezgodność z Javą była błędem, nawet jeśli dotyczyła jednej kolejności w jednym raporcie
  na 25 850. Większość czasu portu szła na szczegóły z sekcji „Ukryta połowa pracy”, nie na algorytm.
- **Uczciwy pomiar na jednej, nieizolowanej maszynie** (dwa rdzenie, `../MACHINE.md`). Szum ok. 10%, a w tle inne
  programy: pomiar czasu tylko przy zamkniętej przeglądarce. Do tego trzeba było rozstrzygać, co jest szybsze:
  - pozycja w serii myliła się z implementacją, dlatego rundy idą w kwadracie łacińskim;
  - pierwsze przejście po starcie to głównie kompilacja JIT;
  - program pomiarowy liczył też własną serializację raportu, co zaniżało dok/s obrazów natywnych JVM
    (`../data/README.md`);
  - zegar etapów w Ruście sam kosztował zauważalną część czasu w polskim, dopóki go nie poprawiono;
  - profiler potrafił przypisać próbki niewłaściwej funkcji: dwie „optymalizacje” poprawiały profil, a nie zmieniały
    czasu.
- **Ustawienia domyślne środowisk uruchomieniowych.**
  - Nowa wersja JDK zmieniła domyślny początkowy rozmiar sterty i szczyt pamięci serwisu przekroczył budżet 500 MB,
    dopóki sterty nie ustawiono jawnie.
  - .NET w trybie regionów trzymał pamięć ponad ten budżet i wrócił do segmentów.
  - V8 bez limitu sterty trzymał ok. 1 GB, a Bun limitu nie ma.
  - Go po prostym porcie spędzał dużą część CPU w odśmiecaniu, dopóki dane nie poszły do tablic bez wskaźników.
- **Sprzęt.** Pamięć bez ECC: rzadkie różnice skrótu wyników wyglądały jak niedeterminizm, a były pojedynczymi
  przestawionymi bitami (`01_equivalence.md`). Każdy taki przypadek trzeba było przeliczyć od nowa, zanim można było
  wykluczyć błąd w kodzie.
- **Narzędzia pod Windows.**
  - Powłoka Git Bash zamieniała argumenty zaczynające się od ukośnika na ścieżki (`/Fa` → `D:\Git\Fa…`, błąd
    linkera C++), zjadała ukośniki wsteczne w tekście przekazywanym do skryptów i uruchamiała interaktywnego
    Pythona, gdy wejście było puste.
  - Launcher `java` z systemowej ścieżki startował JVM jako proces potomny, więc pomiar pamięci i zatrzymanie
    trafiały w launcher, a nie w JVM.
  - Edytor kompilował klasy do katalogu budowania i psuł gotowy jar.
  - Pełny dysk udawał błąd kodu.
  - Każde nowe narzędzie (GraalVM, .NET, Go, MSVC) wymagało zgody człowieka i instalacji poza ścieżką systemową.
- **Własne błędy AI.**
  - Literówka w nazwie zestawu instrukcji procesora odrzucona przez budowanie trzy razy z rzędu.
  - Edycja źródeł w trakcie budowania.
  - Polecenia, które zawieszały powłokę.
  - Notatki rozjeżdżające się z kodem: w tym projekcie częściej myliły się notatki i narzędzia wyceniające niż sam
    kod, więc każde narzędzie wyceniające dostało wariant kontrolny, który odtwarza znany wynik.

## Jak trudne to było

Samo przepisywanie kodu okazało się łatwe: przy gotowej referencji, testach referencyjnych i bramce port to godziny,
a nie tygodnie (`06_port_cost.md`). Trudne były trzy inne rzeczy:
- zgodność semantyki platformy, której nie widać w kodzie źródłowym;
- odróżnienie prawdziwej różnicy wydajności od szumu, pozycji w serii i kosztu samego programu pomiarowego;
- dyscyplina: każda liczba sprawdzona drugim, niezależnym sposobem, a każdy port oceniany tylko po bramce.

Z perspektywy AI najtrudniejszy był Rust: pierwszy język spoza JVM, w którym trzeba było zbudować całe odtworzenie
Javy. Potem C#, ze względu na najdłuższą serię optymalizacji i strojenie odśmiecania. Najłatwiejszy był Kotlin.
TypeScript i C++ były łatwe do napisania i trudne do zaufania: błędy, które Rust czy C# odrzuciłyby przy kompilacji,
wychodziły dopiero w testach albo na bramce. Wniosek: przy pracy z AI o koszcie decyduje nie język, tylko to,
czy istnieje automatyczna, ścisła definicja poprawności. Bez bramki bit w bit żadna z tych liczb nie byłaby
porównywalna.
