# Słownik pojęć

## Kompilacja i uruchamianie

- **JVM** — maszyna wirtualna Javy; uruchamia bajtkod Javy, Kotlina i Scali.
- **HotSpot** — standardowa JVM z OpenJDK (tu JDK 27). **OpenJ9** — inna JVM (Eclipse, tu JDK 25).
- **JIT (kompilacja w trakcie działania)** — program startuje jako bajtkod, a metody wykonywane często są kompilowane do
  kodu maszynowego na podstawie profilu zebranego w trakcie działania. Dlatego pierwsze przejścia są wolniejsze.
- **C1 / C2** — dwa kompilatory JIT w HotSpot: szybki, który zbiera profil, i optymalizujący. Zbyt duża metoda może
  nigdy nie dostać C2.
- **Rozgrzewanie** — pierwsze przejścia po starcie, w których JIT dopiero kompiluje; porównania dotyczą stanu
  ustalonego.
- **AOT (kompilacja z góry)** — cały program kompilowany do kodu maszynowego przed uruchomieniem.
- **Obraz natywny (GraalVM native-image)** — program JVM skompilowany AOT do jednego pliku wykonywalnego: szybszy
  start i mniej pamięci, bez JIT w trakcie działania. **NativeAOT** — to samo dla .NET (C#).
- **PGO (optymalizacja sterowana profilem)** — budowanie w dwóch krokach. Najpierw program z instrumentacją działa na
  reprezentatywnych danych i zapisuje, które funkcje i gałęzie są gorące. Potem kompilator buduje wersję ostateczną:
  wplata i układa kod według tego profilu. Tu używane w GraalVM, Ruście (LLVM), C++ (MSVC) i Go; trening zawsze na
  próbkach, nigdy na zestawie pomiarowym.
- **LTO (optymalizacja przy linkowaniu)** — kompilator widzi cały program naraz; w Ruście sprawdzone i odrzucone.
- **Kompilacja warstwowa (.NET)** — jak C1/C2: najpierw szybki kod, potem optymalizowany po zliczeniu wywołań.
- **x86-64-v3 / AVX2** — poziom zestawu instrukcji procesora (m.in. wektory 256-bitowe); kod budowany pod nowsze
  procesory.
- **SIMD, wektoryzacja** — jedna instrukcja wykonana na kilku danych naraz.
- **Asembler (MASM, `asm!`)** — kod maszynowy pisany ręcznie, osobno albo wewnątrz funkcji Rusta.
- **MSVC** — kompilator i linker Microsoftu: C++, linkowanie Rusta i NativeAOT pod Windows. **Toolchain** — zestaw
  kompilatora, linkera i bibliotek dla danego celu.
- **V8 / Node** — silnik JavaScriptu z Chrome i jego środowisko serwerowe; TypeScript uruchamiany wprost, z usuniętymi
  typami, które nie są sprawdzane. **Bun** — inne środowisko JS (silnik JavaScriptCore z Safari).
- **WebAssembly (wasmtime)** — przenośny kod bajtowy uruchamiany w piaskownicy; tu port Rusta, bez wątków.
- **Interfejs C (FFI), `ctypes`** — wołanie funkcji biblioteki natywnej z innego języka; Python przez standardowy
  moduł `ctypes`.

## Pamięć i odśmiecanie

- **Sterta** — pamięć na obiekty tworzone w trakcie działania. **Żywa sterta** — obiekty wciąż osiągalne.
- **GC (odśmiecanie)** — automatyczne zwalnianie nieużywanych obiektów; kosztuje czas procesora i pauzy.
- **GC generacyjny** — osobno młode i stare obiekty, więc tanio sprząta wiele krótko żyjących (JVM, .NET, V8).
  Go ma GC niegeneracyjny i nieprzesuwający: koszt rośnie z liczbą wskaźników w żywej stercie.
- **Serial GC / G1** — dwa odśmiecacze HotSpot: prosty jednowątkowy i domyślny, regionowy, wielowątkowy.
- **`-Xmx` / `-Xms`** — największy i początkowy rozmiar sterty JVM.
- **Regiony / segmenty (.NET)** — dwa sposoby organizacji sterty .NET; **LOH** — osobna sterta dużych obiektów;
  **ConserveMemory** — GC oddaje pamięć wcześniej kosztem procesora.
- **GOGC / GOMEMLIMIT** — w Go: jak często GC zbiera (przyrost sterty w procentach) i miękki limit pamięci.
- **`--max-old-space-size`** — limit starszej części sterty V8.
- **Alokator (mimalloc)** — biblioteka przydzielająca pamięć w programach bez GC (Rust, C++).
- **Arena** — jeden duży blok pamięci na wiele małych obiektów, zwalniany razem.
- **RSS** — pamięć fizyczna zajęta przez proces; **szczyt RSS** — największa wartość w trakcie pomiaru.
- **ECC** — pamięć z korekcją błędów; ta maszyna jej nie ma, stąd rzadkie przekłamania pojedynczych bitów.

## Wątki

- **Wątek** — osobny tok wykonania. Tu długi dokument dzielony jest na słowa liczone równolegle przez kilka wątków.
- **Pula wątków** — stały zbiór wątków roboczych (ForkJoin w Javie, rayon w Ruście, pula wątków .NET).
- **Wątek wywołujący pracuje** — wątek, który zlecił pracę, liczy swoją porcję zamiast czekać na pomocników.
- **Gorutyna** — lekki wątek Go, rozdzielany przez runtime na wątki systemowe.
- **Worker (JS)** — osobny izolat V8 w osobnym wątku, bez wspólnych obiektów z wątkiem głównym.
- **GIL** — globalna blokada interpretera Pythona: kod Pythona wykonuje naraz tylko jeden wątek.
- **Próg równoległości** — liczba słów, poniżej której dokument liczy jeden wątek (tu 256).
- **Konsument** — wątek programu pomiarowego, który bierze dokumenty z kolejki (1 albo 4).
- **Rdzeń / procesor logiczny** — ta maszyna ma 2 rdzenie i 4 procesory logiczne (`MACHINE.md`).
- **Przypięcie do procesorów** — proces może używać tylko wybranych procesorów (scenariusz „mała maszyna”).

## Pomiar

- **Przejście** — jedno przetworzenie całego zestawu (341 tekstów konstytucji).
- **Czas biblioteki (`library_s`)** — suma czasów samej biblioteki na dokumentach; **dok/s** — dokumenty na sekundę
  zegara całego przejścia (z serializacją i zapisem raportu).
- **`x_java`** — krotność czasu Javy JIT z tego samego przebiegu (0,7 = 30% szybciej).
- **p50 / p99 / p999** — mediana i percentyle opóźnień: 99% (99,9%) dokumentów skończyło się szybciej.
- **Pętla zamknięta / otwarta** — następny dokument dopiero po poprzednim / dokumenty przychodzą według
  harmonogramu, niezależnie od tempa programu.
- **Harmonogram Poissona** — losowe odstępy między przyjściami przy zadanym średnim tempie.
- **Coordinated omission** — błąd pomiaru, gdy opóźnienie liczy się od faktycznego startu, a nie od zaplanowanego
  przyjścia (ukrywa czas w kolejce); tu unikany.
- **Tłok** — inny proces zajmuje zadany procent procesora.
- **Seria** — pełny przebieg wszystkich pomiarów; seria 2 = powtórzenie tymi samymi plikami wykonywalnymi.
- **Kwadrat łaciński** — kolejność implementacji przesuwana co rundę, żeby pozycja w serii nie myliła się
  z implementacją. **Przeplot (ABBA)** — warianty na zmianę, w obu kolejnościach.
- **A/B tej samej binarki** — ten sam plik wykonywalny, warianty różnią się tylko ustawieniem.
- **Profiler (JFR, pprof)** — narzędzie próbkujące, gdzie program spędza czas: JFR dla JVM, pprof dla Go.
- **Zimny start** — start bez gotowych detektorów (trening od zera); start z cache — detektory wczytane z dysku.
- **Token** — jednostka tekstu, w której model AI liczy wejście i wyjście (w kodzie tego projektu ok. 2,6 znaku).
  **Okno kontekstu** — ile tokenów model może mieć naraz w jednym zapytaniu (np. 200 tys. albo 1 mln).
- **Pokolenie cache'u** — komplet plików detektorów dla jednej wersji danych; zmiana danych daje nowy klucz pliku,
  więc stare pokolenia zostają obok.

## Poprawność

- **Port** — przepisanie biblioteki na inny język tak, żeby dawała te same odpowiedzi.
- **Bramka równoważności** — porównanie każdego raportu portu z Javą, pole po polu; port jest poprawny tylko przy 0
  różnic.
- **Raport** — wynik biblioteki dla dokumentu albo pojedynczego słowa.
- **Skrót (digest)** — skrót wszystkich raportów przejścia; ten sam w każdej rundzie oznacza tę samą pracę.
- **Bit w bit / bajt w bajt** — identyczny wynik / identyczne pliki.
- **Fuzzing różnicowy** — losowe wejścia z ziarnem podane obu implementacjom i porównanie wyników.
- **Test referencyjny (złoty plik)** — zapisany wynik Javy, który port musi odtworzyć.
- **AddressSanitizer** — tryb budowania C++, który wykrywa błędy pamięci w trakcie działania.
- **Kontrola negatywna** — celowo zepsuty wariant, który narzędzie musi wykryć.

## Algorytm i tekst

- **NSA (algorytm selekcji negatywnej)** — losuje detektory, które nie pasują do żadnego słowa języka („swoich”);
  słowo pasujące do któregoś detektora uznaje się za obce.
- **Dopasowanie Hamminga / r-contiguous** — dwa sposoby porównania detektora ze słowem: liczba różnych pozycji albo
  wspólny ciąg r znaków.
- **Odległość edycyjna (Levenshtein)** — liczba wstawień, usunięć i zamian znaków; etap literówek (TYPO).
- **PoliMorf** — słownik form wyrazów języka polskiego.
- **Iniekcja** — wstawka w tekście, która próbuje przejąć sterowanie modelem językowym; biblioteka wykrywa obce
  wstawki i znane wzorce.
- **Homoglif** — znak z innego pisma, który wygląda jak łaciński (np. cyrylickie „а”).
- **UTF-16, surogat** — zapis znaku Unicode w jednej albo dwóch jednostkach 16-bitowych; surogat to połowa pary.
- **NFKC** — normalizacja Unicode sprowadzająca warianty znaków (np. ligatury) do postaci podstawowej.
- **Wyrażenie regularne** — wzorzec wyszukiwania w tekście; jego semantyka różni się między językami.
- **Cache werdyktów** — zapamiętane odpowiedzi dla słów już widzianych.
- **Pipeline** — łańcuch etapów klasyfikacji; tu dwa: polski i malgaski.
