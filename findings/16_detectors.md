# Detektory w liczbach

> Policzone z plików detektorów zapisanych na dysku (aktualny zbiór: najnowszy plik każdej kategorii, dopasowania
> i ziarna) (`../data/detectors.csv`). Pojęcia: `../GLOSSARY.md` (NSA, detektor, Hamming / r-contiguous).

## Ile ich jest

Każda kategoria słów ma własny zbiór 3000 detektorów dla każdego z dwóch dopasowań (Hamming i r-contiguous).
- **Polski:** 256 kategorii i jedno ziarno, czyli 1,54 mln detektorów.
- **Malgaski:** 261 kategorii i dwa ziarna, czyli 3,13 mln detektorów (`../data/detectors.csv`).
- **Słowa „swoje”:** na kategorię przypada mediana 128 słów w polskim i 104 w malgaskim. Detektorów jest więc
  ok. 23–29 razy więcej niż słów, które muszą omijać.
- **Cache:** na dysku leży 8844 pliki i 389 MB. To kilka pokoleń, bo każda zmiana danych daje nowy klucz:
  w malgaskim 7, w polskim 3 (`../data/detectors.csv`).
- **Zimny trening:** wszystkich zbiorów trwa na CPU ok. 8,8 s w polskim i 16,4–18,7 s w malgaskim
  (`../data/gpu.csv`).
- **Równoważność:** każda z ośmiu implementacji trenuje te same detektory bajt w bajt (`01_equivalence.md`).

## Jak wyglądają

Detektory losuje model liter uczony na słowach obcych, więc wyglądają jak słowa z różnych języków naraz. Pierwsze
detektory kategorii „zwierzęta”:
- w polskim: `gwe`, `għa`, `ìṣermerisch`, `ameshegeç`, `mháigedia`, `ithecat`, `šul`;
- w malgaskim: `užiješellangsahanného`, `vszekovoitupkiya`, `pricendoutekhayulličkeción`, `aur`, `hlaje`.

Używają 296 różnych znaków w polskim i ok. 175 w malgaskim (`../data/detectors.csv`), bo listy słów obcych
obejmują kilkadziesiąt języków. Mediana długości to 12–13 znaków dla Hamminga i 8 dla r-contiguous
(`../data/detectors.csv`). Hamming przyjmuje więcej długich kandydatów: w malgaskim 11,6% jego detektorów ma ponad
30 znaków, najdłuższe ok. 480 (`../data/detector_lengths.csv`).

Litery detektorów są bliższe językom obcym niż własnemu. W słowach malgaskich „a” to 24,5% liter, a w detektorach
8,8%. W polskich słowach „w” to 4,2% liter (w detektorach 0,7%), „z” 4,2% (1,6%), „k” 5,4% (2,9%)
(`../data/detector_letters.csv`).

## Ciekawostki

- **Prawdziwe słowa obce.** Ok. 0,5% różnych detektorów Hamminga i 1,1–1,2% r-contiguous to prawdziwe słowa
  z listy słów obcych (`../data/detectors.csv`), np.:
  - niemieckie `schuldest`, niderlandzkie `trakteren`, tureckie `düşünmen`, zuluskie `nokugula`;
  - wspólne dla kilku języków słowiańskich `prednost`, włoskie `promette`, wspólne dla katalońskiego, hiszpańskiego
    i portugalskiego `preocupo`.
- **Powtórki.** Tylko 46–67% detektorów jest różnych, a 22–39% różnych detektorów występuje w dwóch albo więcej
  kategoriach (`../data/detectors.csv`). Wszystkie kategorie losują kandydatów z tym samym ziarnem, więc dostają ten
  sam ciąg kandydatów i różnią się tylko tym, które odrzuciły.
- **Słowa własnego języka.** Kilkaset detektorów w każdym zbiorze to słowa samego języka, ale z innych kategorii
  (`equal_to_self_word`). Zgadza się to z konstrukcją: detektor kategorii omija tylko słowa swojej kategorii. Dlatego
  biblioteka pyta wszystkie kategorie, a słowo z dowolnej z nich zawsze jest „swoje”.

**Wniosek:** detektory to kilka milionów pseudo-słów, które wyglądają jak języki obce, a nie jak język chroniony.
Właśnie dlatego trafiają w obce wstawki, a nie w zwykły tekst. Ponieważ wszystkie implementacje trenują je bajt
w bajt tak samo, każda różnica w czasie pochodzi z tej samej pracy.
