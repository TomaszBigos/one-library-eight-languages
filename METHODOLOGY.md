# Metodologia

> Bez dat kalendarzowych; kolejność zdarzeń opisana względnie.

## Bramka równoważności
Każdy port przetwarza ten sam zestaw (25 850 raportów: dokumenty i słowa, dwa warianty dopasowania) i porównuje
raport z Javą ściśle, bez zaokrągleń. Port jest poprawny tylko przy 0 różnic. Dodatkowo: zimny trening detektorów
bajt w bajt, fuzzing różnicowy (losowe wejścia z ziarnem), AddressSanitizer dla C++.

## Pomiar wydajności
- **Zestaw:** 341 tekstów konstytucji i ustaw zasadniczych z Wikisource (40 języków, 2,28 mln słów), znalezionych
  wyszukiwarką każdej Wikisource po słowie „konstytucja”. To nie jest po jednym tekście na kraj: ten sam kraj bywa
  w kilku wersjach historycznych, a pokrycie krajów zależy od tego, co jest na Wikisource. Same teksty nie są częścią
  tego katalogu.
- **Program pomiarowy:** biblioteka w tym samym procesie, bez sieci; każda implementacja ma własny program o tym samym
  zachowaniu (te same dokumenty w tej samej kolejności, ten sam harmonogram przyjść). Raporty z każdego przejścia idą
  do pliku, a ich skrót musi być identyczny w każdej rundzie i każdej implementacji — inaczej to nie jest porównanie
  tej samej pracy.
- **Dwie miary czasu:** czas biblioteki (suma czasów dokumentów, to płaci wywołujący) i zegar całego przejścia
  (dok/s), który obejmuje też zamianę raportu na JSON i zapis. Skala tej różnicy, duża u obrazów natywnych JVM:
  `data/README.md`.
- **Seria końcowa:** dwa pełne przebiegi tych samych plików wykonywalnych, wyniki obu podane osobno. W macierzy
  3 rundy w każdej serii, z kolejnością implementacji przesuwaną co rundę (kwadrat łaciński), bo wcześniej pozycja
  w serii myliła się z implementacją; w pozostałych pomiarach 2 rundy, liczba rund jest w każdym wierszu danych.
  Krotności zawsze wobec Javy JIT z tego samego przebiegu.
- **Warunki maszyny:** pomiary czasu przy zamkniętej przeglądarce i bez innej pracy; skrypt kończył pomiar przed
  następnym blokiem, gdy przeglądarka się uruchomiła. Pomiary orientacyjne przy otwartej przeglądarce nie weszły
  do `data/`.
- **Scenariusze:** tłok na CPU (równoległy proces o zadanym obciążeniu, faktycznie osiągnięte zapisane), strumień
  żądań (otwarta pętla: dokumenty przychodzą według harmonogramu Poissona wspólnego dla wszystkich implementacji,
  opóźnienie liczone od zaplanowanego przyjścia, więc z kolejką), głodny cache, ciasna pamięć (twardy sufit pamięci
  procesu), zimny zryw (minuta bezczynności przed strumieniem), mała maszyna (proces przypięty do części procesorów),
  wspólny serwer (kilka programów naraz, każdy z własnym strumieniem). W zamkniętej pętli jeden klient, w otwartej
  4 konsumentów na instancję.
- **Jedno ustawienie naraz:** pytania o pojedyncze ustawienie (profil kompilacji, odśmiecanie, serializacja raportu)
  rozstrzygane tym samym plikiem wykonywalnym z różnymi zmiennymi środowiska, rundy w przeplocie w obu kolejnościach.

## Pomiary szczególne
- **Działanie długotrwałe:** jeden proces na implementację (Java JIT, Java natywna z PGO, Rust, Go i C++ z PGO,
  C# JIT), ten sam zestaw w pętli 900–2000 razy (46–104 minuty), cache werdyktów jak w produkcji; każde przejście
  zamienia raport każdego tekstu na JSON (bez zapisu na dysk), tak samo we wszystkich implementacjach; zapisany czas
  każdego przejścia i co minutę
  pamięć (RSS, szczyt) oraz czas CPU procesu. Pytanie: czy coś dryfuje (czas, pamięć) i czy JIT z czasem dogania kod
  natywny.
- **Trening detektorów na GPU:** losowanie kandydatów zostaje na CPU w tej samej kolejności, karta sprawdza je
  paczkami; równoważność = pliki wytrenowanych detektorów identyczne bajt w bajt z treningiem na CPU. Czas = zimny
  start całej biblioteki (wczytanie danych + trening), CPU i GPU w przeplocie.
- **Python → Rust:** biblioteka w Ruście za interfejsem C, wołana z Pythona przez standardowy moduł `ctypes`;
  czas dokumentu = wywołanie + zamiana odpowiedzi JSON na słownik Pythona (to, co płaci wywołujący). Porównanie
  z samym Rustem zbudowanym tymi samymi flagami.
- **WebAssembly:** ten sam port Rusta w piaskownicy (wasmtime), bez wątków — wyłącznie jeden konsument; czas CPU
  procesu nie jest mierzony (piaskownica go nie udostępnia), porównywany jest czas — z Rustem natywnym, któremu
  wyłączono podział dokumentu na wątki (inaczej porównanie mierzyłoby liczbę wątków, nie piaskownicę).
- **Inna maszyna wirtualna Javy (OpenJ9):** ten sam program, inna JVM; porównanie w stanie ustalonym (kilkadziesiąt
  przejść, czas każdego zapisany), bo kompilator OpenJ9 dochodzi do pełnej szybkości wolniej niż HotSpot.
- **Praca AI:** liczona z zapisów sesji (wywołania narzędzi i ich wyniki): edycje plików według rozszerzenia,
  budowania i testy według komendy, błędy według komunikatów kompilatora i testów, przebiegi bramki według jej wyniku.
  Tokeny i czas aktywny według przypisania odpowiedzi do języka, jak w `port_cost.csv`. To przybliżenie: wynik
  skracany przy wyświetlaniu ukrywa część błędów, a najstarsze sesje (Java przed portami) nie są już dostępne.
  Rundy optymalizacji pochodzą z dzienników portów.
- **Tokeny kodu:** liczba znaków na token skalibrowana na zapisach sesji (przyrost kontekstu po wczytaniu jednego
  pliku), bez wysyłania kodu na zewnątrz; przedział z kwartyli próbek. **Detektory:** statystyki z plików cache'u,
  aktualny zbiór = najnowszy plik każdej kategorii, dopasowania i ziarna.

## Ograniczenia
- jedna maszyna (`MACHINE.md`), dwa rdzenie fizyczne, bez izolacji: różnice poniżej ~10% traktować jako remis;
- pamięć bez ECC z rzadkimi przekłamaniami pojedynczych bitów. Rozbieżności skrótu wyników były sprawdzane:
  zrzut przeliczony ponownie albo porównany bajt w bajt z poprawnym. Każda, którą dało się rozebrać, okazała się jednym
  przestawionym bitem, a w czterech dokładnie opisanych przypadkach tym samym (wartość 0x40 w bajcie znaku, zawsze z 1 na 0) — odcisk sprzętu, nie kodu.
  Czas takiej rundy zostaje w danych (przekłamanie nie zmienia pracy), a liczba rund z poprawnym skrótem jest podana
  w każdym wierszu. Kilka razy proces zatrzymała awaria w trakcie odśmiecania pamięci (przyczyna niepotwierdzona,
  zgodna z przekłamaniem) — taka runda albo instancja nie ma wyniku;
- program pomiarowy obrazów natywnych JVM zamienia raport na JSON wolniej niż JIT, więc ich dok/s są zaniżone wobec
  czasu biblioteki (`data/README.md`).
