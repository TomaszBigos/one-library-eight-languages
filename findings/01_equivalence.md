# Osiem implementacji, te same odpowiedzi

Każda implementacja (Java, Kotlin, Scala, Rust, C#, Go, C++, TypeScript, a do tego obrazy natywne i WebAssembly) musi
dawać na każdym wejściu dokładnie te same raporty co Java — bez zaokrągleń, pole po polu. Bramka porównuje 25 850
raportów na język (dokumenty i pojedyncze słowa), a detektory z zimnego treningu bajt w bajt; port jest poprawny tylko
przy 0 różnic (`../data/equivalence.csv`). Druga siatka to fuzzing różnicowy: losowe teksty z ziarnem (mieszane pisma,
homoglify, znaki niewidzialne, bardzo długie słowa, wstawki iniekcji) — w ostatniej rundzie (3000 tekstów)
każda implementacja 0 różnic na 31 696 raportach na język (`../data/equivalence.csv`). Bramka znalazła błędy, których kompilatory nie widziały (zob. `07_compiler_vs_gate.md`), a kilka razy
także błędy sprzętu: losowe, za każdym razem inne rozjazdy skrótu wyników przy poprawnym pliku na dysku okazały się
pojedynczymi przekłamanymi bitami w pamięci (komputer bez ECC). Wniosek: przy porównaniu wydajności języków równoważność
wyników nie jest dodatkiem — bez niej szybsza implementacja może po prostu robić coś innego.
