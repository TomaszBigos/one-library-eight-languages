# Przekłamania pamięci i inne ciekawostki

## Przekłamania pamięci (komputer bez ECC)

Rzadkie różnice skrótu wyników wyglądały jak niedeterminizm w kodzie. Za każdym razem okazywały się jednym
przestawionym bitem: w pliku zrzutu albo w pamięci procesu Pythona, który liczył skrót. Udokumentowane przypadki
zmienionego tekstu:

| gdzie | było | stało się | bajt |
|---|---|---|---|
| zrzut raportów Javy (302 MB, 25 850 linii) | przecinek `,` | litera `l` | 0x2C → 0x6C |
| raport C# | `horonantsary` | `horonantsa2y` | 0x72 → 0x32 |
| proces Pythona liczący skrót (plik na dysku poprawny) | `NEUTRAL` | `NEUTR` + znak sterujący + `L` | 0x41 → 0x01 |
| odpowiedź w bramce równoważności | pole `startIndex` | `star4Index` | 0x74 → 0x34 |
| zrzut TypeScriptu (Bun), 1 dokument z 341 | `token` | `tok%n` | 0x65 → 0x25 |
| zrzut w teście wspólnego serwera | `NEUTRAL` | znak sterujący + `EUTRAL` | 0x4E → 0x0E |

Wszystkie sześć to ten sam bit (0x40): pięć razy z 1 na 0, raz z 0 na 1. To odcisk sprzętu, nie kodu. Wielka litera,
która traci ten bit, staje się znakiem sterującym, dlatego część przypadków wyszła jako „niepoprawny znak w JSON”
przy czytaniu zrzutu. Przekłamania przychodziły też seriami: w ciągu ok. dwóch minut dwa uszkodzone zrzuty i dwa złe
skróty u dwóch różnych programów. Potem sanitizer pamięci był czysty, a 40 powtórek dało 0 różnic.

Prawdopodobnie z tej samej przyczyny:
- dwie awarie kompilatora obrazów natywnych przy dużym zużyciu pamięci („uszkodzenie” wewnętrznych struktur); po
  zmniejszeniu jego pamięci z 12 do 7 GB się nie powtórzyły;
- pojedyncze, nieodtwarzalne różnice skrótu Javy JIT (12 powtórek z 12 zgodnych);
- rzadkie awarie w trakcie odśmiecania (przyczyna niepotwierdzona, `../METHODOLOGY.md`).

Jak to obsłużono:
- każdy przebieg z innym skrótem zostawia zrzut, który przelicza się ponownie i porównuje bajt w bajt; różnica
  jednego bitu oznacza sprzęt, a bramkę się powtarza;
- nieudanej instancji nie przypisuje się implementacji bez takiego sprawdzenia;
- runda z takim przekłamaniem ma ważny czas, a w danych widać ją jako `rounds_digest_ok` mniejsze od `rounds`
  (`../data/README.md`).

Pamięci nie wymieniono. Zamiast tego zmniejszono szczytowe obciążenie maszyny.

## Inne ciekawostki

- **Port znalazł błędy w oryginale.**
  - Lista 10 najlepszych kategorii dla słowa zależała od liczby rdzeni: na jednym wątku 15 z 16 list wyszło inaczej.
  - Remis odległości marek rozstrzygał się inaczej przy każdym starcie JVM.
  - Pod tureckimi ustawieniami regionalnymi „Ignoruj” zmieniało się w „ıgnoruj” i polski tekst stawał się „obcy”.

  Wszystko to naprawiono w Javie (`09_ai_work.md`).
- **Narzędzie referencyjne samo nie wytrzymało fuzzingu.** Zrzut raportów Javy pada na samotnych surogatach
  UTF-16, więc takie teksty fuzzing podaje tylko na żądanie.
- **Jedna metoda za duża dla kompilatora.** Metoda klasyfikacji dokumentu malgaskiego w Javie (2318 bajtów bajtkodu,
  poniżej formalnego limitu dużych metod) przy zwykłym wywołaniu zostawała na poziomie C1. Optymalizujący C2
  kompilował ją tylko jako pętlę w trakcie działania. Po podziale na etapy CPU na dokument w malgaskim spadł o 45%
  przy identycznym wyniku (`11_optimizations.md`).
- **Jedna kompilacja zmieniała szczyt pamięci.** W Kotlinie szczyt pamięci w malgaskim wynosił 350 albo 510 MB przy
  tej samej żywej stercie, zależnie od tego, czy jedna duża metoda trafiła do kompilatora C2.
- **C# na jednym procesorze nie kończył kompilacji.** Kompilacja warstwowa zaczyna liczyć wywołania dopiero po 100 ms
  bez nowych kompilacji; na jednym procesorze przy ciągłej pracy ten moment się odsuwał i metody zostawały na
  najniższej warstwie. Jedno ustawienie dało 2,5 raza
  więcej (`11_optimizations.md`).
- **Pomiar zmieniał wynik.** Zegar etapów w Ruście przeliczał jednostki przy każdym odczycie i sam zabierał
  zauważalną część czasu w polskim, przez co Rust wyglądał na wolniejszy od C# na tym etapie.
- **Obraz natywny szybszy w bibliotece, wolniejszy na wyjściu.** Czas biblioteki obrazu natywnego Javy był równy JIT,
  ale zamiana raportu na JSON kosztowała go 2,2–2,4 s na przejście, a JIT 0,45–0,68 s (`../data/README.md`).
- **Profiler, który się myli.** Dwie zmiany w Javie zmniejszyły udział funkcji w profilu o połowę, a czas i CPU nie
  drgnęły: próbki z wkompilowanej pętli trafiały do sąsiedniej funkcji.
- **„Jedna binarka, trzy tempa”, czyli dwie wersje programu pomiarowego.** Obraz natywny Javy w godzinie pracy
  liczył przejście raz w 2,0 s, raz w 4,1 s, a w jednym przebiegu przez ok. 25 minut w 3,1–3,3 s. Wyglądało to na
  niestabilność binarki. Osiem osobnych startów nowszej binarki dało za każdym razem 4,10–4,16 s przy czasie samej
  biblioteki 1,9–2,0 s. Różnicą była zamiana raportu na JSON: nowsza wersja programu pomiarowego robi ją w każdym
  przejściu, starsza tylko w ostatnim, a przebieg z 2,0 s szedł do tego starszą binarką. Godzinę pracy zmierzono
  potem od nowa jedną wersją programu dla wszystkich implementacji (`08_python_wasm_long_run.md`).
- **Dwa padnięcia w siedem minut.** W tej powtórce Java JIT (po 48 minutach) i obraz natywny Javy (po kilku minutach)
  padły z naruszeniem dostępu do pamięci w trakcie odśmiecania: odśmiecacz trafił na „wskaźnik”, który był kawałkiem
  tekstu. Dwa różne środowiska uruchomieniowe, kod bez operacji na surowej pamięci; ponowione przebiegi przeszły do
  końca z tymi samymi raportami. Pasuje to do przestawionych bitów opisanych wyżej, ale dziennik systemu nie zapisał
  przy tym błędu sprzętowego, więc przyczyna nie jest ustalona (`08_python_wasm_long_run.md`).
- **Wyciek, którego krótki pomiar nie widzi.** Program pomiarowy w C++ tracił ok. 13 MB na każde przejście, bo alokator
  (mimalloc 2.1.7) nie odzyskiwał pamięci wątku tworzonego na przejście. Pomiary z jednym albo kilkoma przejściami
  tego nie pokazały; po ok. 1200 przejściach proces padał. Przyczynę wskazały zapis pamięci procesu z zewnątrz
  co 30 s i build ze stertą systemową, który pamięci nie tracił (`08_python_wasm_long_run.md`).
- **Serial lepszy od nowoczesnych odśmiecaczy.** Na tej samej Javie G1, Parallel i ZGC nie skróciły p99 w strumieniu
  60 dok/s (55 ms przy Serial wobec 57–73 ms w polskim), a G1 i ZGC zajęły ok. 500 MB zamiast 413
  (`12_gc_threads.md`).
- **Odrzucone, a potem przyjęte.** Indeks segmentów dla literówek był najpierw w szumie. Gdy reszta kosztu zmalała,
  dał ok. jedną trzecią CPU w polskim. Odrzucone łatki zostały zachowane.
- **Asembler pokazał, nie przyspieszył.** Ręczny kod był szybszy tylko dlatego, że kompilator C++ po każdym zapisie
  od nowa czytał wskaźnik danych; poprawka w zwykłym C++ dała to samo (`04_assembler_simd.md`).
- **Język gospodarza karty graficznej nie ma znaczenia.** Ten sam program GPU wołany z sześciu języków dał ten sam
  czas, 365–366 µs na słowo (`05_gpu.md`).
- **Zegar z krokiem milisekundy.** W Go na Windows zwykły odczyt czasu przesuwa się co ok. 1 ms, nawet przy zegarze
  systemowym podniesionym do 1 ms. Sumy czasów etapów po wielu dokumentach wychodziły poprawnie, ale dokument ze wstawką
  obcojęzyczną (ok. 0,5–1,5 ms) trwał według tego zegara 0 albo ok. 1000 µs. Po przejściu na licznik wysokiej
  rozdzielczości, z którego korzystają pozostałe implementacje, Go dostało czasy jak reszta. Odczyt licznika kosztuje ok.
  90 ns zamiast ok. 6 ns, a jest ich około dziesięciu na dokument (`22_foreign_fragment.md`).
- **OpenJ9 a HotSpot.** OpenJ9 dochodzi do stałego tempa po ok. 10–12 przejściach, HotSpot po 2–3; w malgaskim jego
  szczyt pamięci przekroczył 1 GB przy tej samej stercie 384 MB (`08_python_wasm_long_run.md`).
- **Nowa wersja JDK zmieniła wartość domyślną.** JDK 27 zaczyna od sterty 8 MB zamiast od maksymalnej; sterta
  rosnąca od zera dotyka więcej pamięci, więc szczyt serwisu polskiego przekroczył budżet 500 MB. Pomogło jedno
  ustawienie (`11_optimizations.md`).
- **Środowisko udawało błędy kodu.**
  - Pełny dysk: program kończył się bez wyjątku w logu.
  - Testy Scali zgłaszały „Tests run: 0” przy zielonym budowaniu.
  - Launcher `java` z systemowej ścieżki był osobnym procesem: pomiar pamięci mierzył kilkumegabajtowy stub,
    a „zatrzymany” serwis dalej odpowiadał na porcie.
