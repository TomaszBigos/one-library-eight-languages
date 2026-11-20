# Wnioski per temat

Każdy plik: kilka zdań wniosku, każda liczba z odwołaniem do pliku w `../data/`.

| plik | temat |
|---|---|
| `00_summary.md` | **podsumowanie**: siedem wniosków i porady per język (kiedy wybrać, na co uważać) |
| `01_equivalence.md` | osiem implementacji, te same odpowiedzi — bramki i fuzzing |
| `02_speed_memory.md` | czas i pamięć na 341 tekstach konstytucji |
| `03_contention.md` | tłok, wspólny serwer, scenariusze |
| `04_assembler_simd.md` | ręczny asembler i SIMD wobec kompilatora |
| `05_gpu.md` | GPU wobec dobrego algorytmu na CPU |
| `06_port_cost.md` | koszt wytworzenia portów (z zastrzeżeniem kolejności) |
| `07_compiler_vs_gate.md` | kto łapie błąd: kompilator czy bramka |
| `08_python_wasm_long_run.md` | Python nad Rustem, WebAssembly, godzina ciągłej pracy, OpenJ9 |
| `09_ai_work.md` | jak AI pracowało nad portami: przepis, odtwarzanie Javy, współpraca każdego języka, optymalizacje i powroty, z czym się zmagało, skala |
| `10_ai_verdict.md` | werdykt AI (opinia oparta na danych): jak wybrać język i czego unikać |
| `11_optimizations.md` | ile optymalizacji w każdym języku i które działały tylko dzięki cechom danego języka |
| `12_gc_threads.md` | pamięć, odśmiecanie (GC) i wątki w każdym języku: ustawienia, powody, pomiary |
| `13_manual_estimate.md` | szacunek: ile zająłby każdy port programiście bez AI (kod, testy, optymalizacja, uruchamianie) |
| `15_code_tokens.md` | ile tokenów zajmuje kod każdej implementacji i cały `publish/` w kontekście AI, koszt wczytania |
| `16_detectors.md` | detektory NSA w liczbach: ile, jakie długości i litery, powtórki, prawdziwe słowa obce |
| `17_prompts.md` | prompty: ile, jak długie, ile pracy AI po jednym prompcie, o jakich językach |
| `18_port_matrix.md` | port z języka na język: tabela 8 × 8, trudność, ocena, z czego portować i jak wybrać |
| `19_readability.md` | jak wygląda kod: długość linii, symbole, wcięcia, komentarze; ranking czytelności |
| `20_text_ops.md` | operacje tekstowe per język programowania: tokenizacja, wyrażenia regularne, znaki niewidzialne |
| `21_doc_language.md` | wrażliwość na język dokumentu: czy kolejność implementacji zmienia się z językiem i pismem tekstu |
| `22_foreign_fragment.md` | akapit w obcym języku w tekście pipeline'u: ile kosztuje i czy raport go zauważa (40 języków) |
| `23_text_languages.md` | **opinia AI**: który język pasuje do pracy z tekstem i NSA (sześć osi, idiom a szybkość, zastrzeżenia) |
| `24_language_design.md` | **opinia AI**: jaki język zaprojektowałby po ośmiu portach — dziesięć mechanizmów z dowodami, pseudoskładnia, wniosek |
| `14_curiosities.md` | przekłamania pamięci (sześć przypadków, ten sam bit) i inne ciekawostki z pracy nad portami |
