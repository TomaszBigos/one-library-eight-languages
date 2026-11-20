# Podsumowanie i porady per język

> Najkrótsza wersja całego materiału. Liczby dla każdego języka zebrane są w `../data/language_summary.csv`
> (z `matrix.csv`, `load_curve.csv`, `scenarios.csv`, `ai_work.csv`, `code_metrics.csv`, `code_tokens.csv`,
> `port_cost.csv`). Porady to opinia AI, które napisało wszystkie porty (`10_ai_verdict.md`).

## Siedem wniosków

1. **Bez równoważności bit w bit nie ma porównania.** Osiem implementacji daje te same 25 850 raportów co Java,
   a bramka złapała błędy, których kompilator nie widział (`01_equivalence.md`, `07_compiler_vs_gate.md`).
2. **Czwórka bez wyraźnego zwycięzcy.** Rust, Go, C# i C++ potrzebują 0,72–0,75 czasu Javy JIT, a między sobą różnią się
   o mniej niż 10%, czyli remisują (`../data/language_summary.csv`, `18_port_matrix.md`).
3. **Pamięć i start to osobna oś.** Najmniej pamięci zajmują obrazy natywne JVM i Rust, a najszybciej startuje Rust:
   0,7 s wobec 3,8 s Javy JIT (`02_speed_memory.md`, `../data/language_summary.csv`).
4. **Warunki pracy zmieniają kolejność.**
   - Pod tłokiem Go zachowuje najmniej własnej przepustowości.
   - Przy limicie 200 MB Java JIT robi 42 dok/s, a Rust 152.
   - Na jednym procesorze czwórka prowadzi wyraźnie przed Javą (`03_contention.md`, `../data/language_summary.csv`).
   - Przy 120 dok/s Rust trzyma p99 41–44 ms, Java JIT 71–117 ms, a obraz natywny Javy nie nadąża (0,5–0,7 s)
     (`03_contention.md`, `../data/latency_curve.csv`).
5. **TypeScript do warstwy, nie do liczenia.** Jest ok. 6 razy wolniejszy od Javy, a w strumieniu 60 dok/s nie
   nadąża: p99 ok. 6,4 s (`../data/language_summary.csv`).
6. **AI przepisuje bibliotekę w godziny, a nie w tygodnie,** pod warunkiem że istnieje referencja i automatyczna
   bramka. Szacunek pracy ręcznej to dziesiątki dni na port (`06_port_cost.md`, `13_manual_estimate.md`).
7. **Przenosiły się algorytmy, nie sztuczki.** Najwięcej dały zmiany algorytmiczne, wdrażane we wszystkich
   językach. Cechy języka decydowały o pamięci i pułapkach: GC, JIT na małej maszynie, limity sterty
   (`11_optimizations.md`, `12_gc_threads.md`).

## Porady per język

- **Java** — punkt odniesienia: czas 1,0, szczyt pamięci ok. 340 MB, start 3,8 s.
  - *Wybierz, gdy* system już działa na JVM. Obraz natywny z PGO daje czas Javy przy ok. 0,6 jej pamięci
    (`02_speed_memory.md`).
  - *Uważaj na* domyślne ustawienia sterty i GC oraz na ciasny limit pamięci: przy 200 MB zostaje 42 dok/s.
    G1, Parallel ani ZGC nie skróciły ogona opóźnienia wobec Serial (`12_gc_threads.md`), a w obrazie natywnym
    zamiana raportu na JSON (Jackson) kosztuje tyle co sama biblioteka: w godzinie pracy przejście 4,1 s wobec 2,4 s
    JIT (`08_python_wasm_long_run.md`).
  - Najwięcej optymalizacji (70), dobry profiler (JFR).
- **Kotlin** — czas 1,14 Javy, najprzyjemniejszy port z Javy, 0 odrzuceń kompilatora na 70 budowań.
  - *Wybierz, gdy* zespół woli zwięźlejszą składnię na JVM.
  - *Uważaj na* biblioteki standardowe wciągane przez zależności frameworka.
- **Scala** — czas 1,24 Javy, najdłuższe linie i najgłębsze wcięcia w kodzie, kompilacja wolna.
  - *Wybierz, gdy* zespół i tak pisze w Scali.
  - *Uważaj na* idiomatyczne kolekcje w gorącej ścieżce.
- **Rust** — czas 0,72, szczyt pamięci ok. 240 MB, start 0,7 s. Najlepszy pod limitem pamięci (152 dok/s przy 200 MB).
  - *Wybierz, gdy* liczą się przepustowość, pamięć i start naraz.
  - *Uważaj na* koszt pierwszego portu: trzeba było odtworzyć zachowanie Javy. Brak profilera pod Windows.
- **C#** — czas 0,75, krótki ogon opóźnień w strumieniu żądań, napisy jak w Javie.
  - *Wybierz, gdy* system jest na .NET.
  - *Uważaj na* ustawienia GC pod limit pamięci i na kompilację warstwową na jednym procesorze
    (`11_optimizations.md`).
- **Go** — czas 0,72, mało pamięci, najprostszy w wyglądzie kod (pierwsze miejsce w rankingu czytelności).
  - *Wybierz, gdy* liczy się prostota wdrożenia (zero zależności) i krótki port.
  - *Uważaj na* pracę pod tłokiem (zostaje 31% własnej przepustowości) i na GC przy wielu wskaźnikach.
  - Najczęściej odrzucał kod AI przy kompilacji (15,8 na 100 budowań).
- **C++** — czas 0,75, dobre narzędzia diagnostyczne (sanitizer, listing asemblera).
  - *Wybierz, gdy* zespół ma doświadczenie w C++ i potrzebuje integracji z kodem natywnym.
  - *Uważaj na* błędy czasu życia obiektów: przechodzą kompilację, więc konieczne są bramka i testy. Także na wersję
    alokatora: z mimalloc 2.1.7 program pomiarowy tracił pamięć po każdym zakończonym wątku i padał po ok.
    1200–1350 przejściach, z 3.3.2 trzyma pamięć płasko przez godzinę (`08_python_wasm_long_run.md`).
- **TypeScript** — czas ok. 6 razy dłuższy niż Java i najmniej kodu w tokenach.
  - *Wybierz* do warstwy wokół biblioteki: interfejsu, integracji, skryptów.
  - *Uważaj na* limit sterty V8 i na typy, które nie są sprawdzane przy uruchomieniu.

## Gdzie dalej

Szczegóły i liczby: `01`–`08` (wyniki pomiarów), `09`–`13` (praca AI i jej koszt), `14` (ciekawostki), `15`–`16`
(tokeny, detektory), `17` (prompty), `18` (porty język na język), `19` (czytelność kodu), `20` (operacje tekstowe), `21` (język dokumentu). Pojęcia: `../GLOSSARY.md`.
