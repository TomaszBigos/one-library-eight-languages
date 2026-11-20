# Rust

> Pierwszy port spoza JVM. Zmiany algorytmiczne Javy (`00_common.md`) doszły jako kolejne kroki R1, R2…; tu są one tylko
> wymienione, a opisane szerzej zmiany wynikające z Rusta: alokacje, pożyczanie zamiast kopii, alokator, toolchain, PGO.
> Nazwy R1–R50 to kolejne kroki w dzienniku portu.

## Droga

| etap | konstytucje pl / mg, krotność czasu Javy |
|---|---|
| port wprost (zmiany Javy jeszcze nie przeniesione, kopia słowa na każdym kroku) | ×9,5 / ×3,8 |
| po R1–R14 (indeksy z Javy, mniej alokacji) | ×1,23 / ×1,36 |
| po R15–R24 | ×0,85 / ×0,93 |
| po R25–R41 i PGO | ×0,64 / ×0,72 |
| macierz końcowa (`../data/language_summary.csv`) | ×0,72; szczyt RSS 242 MB, start 0,7 s |

## Co pomogło

| zmiana | technika | efekt |
|---|---|---|
| R1, R2, R16, R23, R30 | indeksy z Javy: jedna odpowiedź na pulę, segmenty literówek, maski 64-bit kategorii, mapa bitowa obecności kluczy | R23: generowanie kandydatów 1214 → 922 ns na zapytanie |
| R4 | bez alokacji na token: nazwy pism jako `&'static str`, listy języków współdzielone przez `Arc` | (z R1–R14) |
| R5 | FxHash (hasz kompilatora rustc) zamiast SipHash w mapach wyszukiwania | (z R7) |
| R7 | NFKC kawałkami między granicami normalizacji; czyste kawałki kopiowane bez zmian | (z R5) |
| R13 | listy tokenów jako pożyczone wycinki `Vec<&[u16]>` zamiast kopii każdego tokenu (2,28 mln alokacji na konstytucjach) | konstytucje 2,92 / 2,53 s |
| R14 | odpowiedź indeksu puli w tablicach na stosie (do 1024 zbiorów; pule liczą 256 / 522) | (`00_common.md`) |
| R22 | forma leetspeak marki budowana tylko, gdy słowo ma cyfrę 0, 1, 3, 4, 5, 7 albo `rn` | marki mg 302 → 179 ns |
| R25 | toolchain MSVC zamiast windows-gnu, ten sam kod | sam nie przyspieszył (pl ×0,99–1,03, mg ×1,05–1,08), ale był warunkiem R27 i R29 |
| R27 | mimalloc jako alokator binarek | pl ×0,894, mg ×0,895 |
| R29 | PGO (profil z treningu na próbkach i zimnego startu) | pl ×0,953, mg ×0,932 |
| R31 | zwijanie liter wzorców z tablicy 65 536 jednostek | etap `fold` 0,116 → 0,008 s |
| R42 | słownik obcy w jednej arenie `Vec<u16>` z tablicą haszy indeksów zamiast napisu na słowo | szczyt RSS pl −65…70 MB, mg −20 MB (tyle co obraz natywny Javy) |
| R46 | test znaków niewidzialnych całego tekstu bez gałęzi (LLVM go wektoryzuje); tekst bez nich nie jest skanowany drugi raz | — |
| R50 | zegar etapów bez przeliczania jednostek przy każdym odczycie | polski ×0,89 |
| SIMD Hamminga | osiem pozycji jednym wektorem | 0,21 czasu pętli skalarnej, dopiero po `#[inline]` funkcji z innej paczki (`../findings/04_assembler_simd.md`) |

## Co nie pomogło

| zmiana | dlaczego | pomiar |
|---|---|---|
| R15: `lto = "fat"`, `codegen-units = 1` | zysk w szumie, build dwa razy dłuższy | pl 3,27 vs 3,12 s, mg 2,48 vs 2,50 s |
| R28: własna tablica haszy (odcisk + przesunięcie w `u64`, napisy w arenie) | bajty kontrolne `hashbrown` mieszczą się w L3, własna tablica nie wygrała | pl ×1,008, mg ×1,026 (wolniej w 4 / 4 rundach mg) |
| R34: mapa zadań z rozmiarem z góry | bez przehaszowania, ale wolniej | 0,098 → 0,104 s |
| R37: liczniki pism na stosie i tablica pism BMP | w szumie | mg ×1,003, pl ×0,992 |
| R40: tablice łuków słownika PoliMorf dla węzłów po pierwszym bajcie | klasyfikacja −2,4%, ale świadkowie −1…2% | pl ×0,996 |
| wstawka `asm!` | 0,86–0,87 czasu kodu z kompilatora, tylko za cenę jednej architektury | wyłączona domyślnie (`../findings/04_assembler_simd.md`) |

## Przykład: alokator

Jedna deklaracja w paczce binarek; biblioteki alokatora nie wybierają, więc decyduje aplikacja:

```rust
#[global_allocator]
static GLOBAL: mimalloc::MiMalloc = mimalloc::MiMalloc;
```

Bez cechy domyślnej (`--no-default-features`) binarka wraca do sterty systemowej, co pozwala zmierzyć A/B ten sam kod.

## Przykład: słownik w arenie (R42)

Przed: mapa słowo → języki z osobnym napisem na każde słowo. Po: wszystkie słowa
w jednym wektorze jako `[długość, jednostki…]`, a tablica haszy trzyma tylko pozycje w arenie.

```rust
pub struct ForeignWordDictionary {
    arena: Vec<u16>,                  // każde słowo jako [długość, jednostki UTF-16…], posortowane
    starts: Vec<u32>,                 // pozycja słowa w arenie, po numerze
    langs: Vec<Languages>,
    index: HashTable<(u32, u32)>,     // (pozycja w arenie, numer), haszowane po jednostkach słowa
}

fn word_at(&self, at: u32) -> &JStr {
    let at = at as usize;
    let len = self.arena[at] as usize;
    &self.arena[at + 1..at + 1 + len]
}

fn find(&self, word: &JStr) -> Option<usize> {
    self.index.find(hash_of(word), |&(at, _)| self.word_at(at) == word).map(|&(_, i)| i as usize)
}
```

Pytanie, od którego się zaczęło: dlaczego obraz natywny Javy miał mniejszy RSS niż Rust. Różnica w pl to były prawie
w całości słowniki obce.

## Techniki w skrócie

- Port wprost był wolny z dwóch powodów: brakowało indeksów Javy i wszędzie były kopie (`String` na token, klon słowa
  na każdym kroku). Indeksy przeniesiono jak w każdym porcie, a wśród zmian specyficznych dla Rusta pożyczanie
  (`&[u16]`), `Arc` dla wspólnych list i bufory na stosie dały więcej niż mikrooptymalizacje.
- Środowisko budowania to osobna runda: toolchain, alokator, `target-cpu=x86-64-v3`, PGO (po każdej zmianie kodu od nowa).
  LTO nie dało nic.
- Biblioteka standardowa (`hashbrown`) jest trudna do pobicia własną tablicą.
- Funkcja z innej paczki bez `#[inline]` nie jest wpleciona i potrafi zablokować wektoryzację.
