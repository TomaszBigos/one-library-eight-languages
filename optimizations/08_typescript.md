# TypeScript

> Port wykonywany wprost przez Node (bez kompilacji, tylko składnia wymazywalna), bez zależności w runtime. Pierwsza
> wersja powstała w kilka godzin bez optymalizacji; potem dostała tę samą listę co C++ (T1–T9). Własne problemy
> wynikały z V8: liczby i bity 32-bitowe, koszt napisów jako kluczy, leniwy GC, izolaty bez wspólnych obiektów.

## Droga

Pierwszy pomiar: konstytucje mg 178,6 s, ponad 100× Javy (Rust PGO 1,4 s). Po T1–T9: mg 10,4 s, pl 15,8 s, czyli ok.
5–6× Javy. W macierzy końcowej ×5,97 czasu Javy, szczyt RSS 367 MB przy limicie sterty, start 2,0 s
(`../data/language_summary.csv`). Domyślnie jeden wątek.

## Co pomogło

| zmiana | technika | efekt |
|---|---|---|
| T1 | indeks puli na maskach `Uint32Array` (operacje bitowe JS są 32-bitowe); klucze liczbowe < 2^53 we własnej tablicy z adresowaniem otwartym na `Float64Array` zamiast `Map` | (z T1–T9) |
| T2 | filtry literówek (maska, bigramy, histogram; maski 64-bit jako dwie połówki) i indeks segmentów | (z T1–T9) |
| T3, T6c | wzorce pl bez flagi z prefiltrem literałów, prefiltr wzorców mg | (z T1–T9) |
| T4 | zegar etapów jako liczba `performance.now()` zamiast `process.hrtime.bigint()` (alokacja `BigInt` przy każdym odczycie, dwa odczyty na token) | (z T1–T9) |
| T5 | memo dokumentu kluczowane samym słowem z dwoma slotami zamiast sklejanych kluczy `HAMMING`, `f`, `1`, słowo | (z T1–T9) |
| T6 | odpowiedź indeksu puli w buforach wielokrotnego użytku (5 tablic typowanych na słowo), marki po paśmie długości | (z T1–T9) |
| T7 | słownik PoliMorf: tablica łuków korzenia, UTF-8 kodowany w locie bez `Buffer` na słowo | (z T1–T9) |
| T9 | detektory zbioru w jednym `Uint16Array` zamiast napisu na detektor (131 MB z 213 MB żywej sterty mg to były napisy detektorów, ok. 40 B nagłówka na napis) | sterta mg 190 → 46 MB |
| limit sterty | `--max-old-space-size=256` w `node.options`; bez niego leniwy GC V8 trzymał ok. 1 GB RSS | szczyt mg 1133 → 465 MB, pl 1054 → 451 MB, czas dłuższy o kilka procent |

## Co nie pomogło albo kosztowało

| zmiana | dlaczego | pomiar |
|---|---|---|
| T8: słowa dokumentu na workerach | izolaty JS nie współdzielą obiektów, więc każdy worker buduje własny klasyfikator; równolegle idzie tylko klasyfikacja różnych słów, a dochodzi klonowanie odpowiedzi | 3 workery: mg −11%, pl −23% czasu przy dwa razy większym RSS (ok. 2,2 GB wobec 1,1 GB bez limitu sterty); domyślnie jeden wątek |
| Bun zamiast Node | JavaScriptCore nie zna limitu sterty | szczyt ok. 1–1,2 GB; mg 11,3 s wobec 10,4 s w Node |

## Przykład: detektory w jednej tablicy (T9)

```ts
/** Detektory jednego zbioru w jednym Uint16Array: detektor i = units[start[i] .. start[i + 1]). */
export class Packed {
  units: Uint16Array;
  start: Uint32Array;
  constructor(strings: string[], maxLen: number) {
    let total = 0;
    for (const d of strings) total += Math.min(d.length, maxLen);
    this.units = new Uint16Array(total);
    this.start = new Uint32Array(strings.length + 1);
    let at = 0;
    strings.forEach((d, i) => {
      this.start[i] = at;
      const n = Math.min(d.length, maxLen);
      for (let k = 0; k < n; k++) this.units[at++] = d.charCodeAt(k);
    });
    this.start[strings.length] = at;
  }
}
```

## Przykład: zegar bez alokacji (T4)

```ts
// przed: process.hrtime.bigint() — nowy BigInt przy każdym odczycie, dwa odczyty na token
// po: liczba zmiennoprzecinkowa, rozdzielczość poniżej mikrosekundy
export const nanoNow = (): number => performance.now() * 1e6;
```

## Przykład: klucz mapy (T5)

```ts
// przed (schemat): `${kind}|${flag}|${dict}|${word}` — napis sklejany przy każdym tokenie,
// który V8 musi spłaszczyć i przehaszować przy każdym wyszukiwaniu
// po: kluczem jest samo słowo (hasz napisu V8 liczy raz), wariant to indeks slotu
const jobAnswers = new Map<string, (ClassifyResponse | null | undefined)[]>();
const slotsOf = (w: string) => {
  let a = jobAnswers.get(w);
  if (a === undefined) jobAnswers.set(w, (a = [undefined, undefined]));
  return a;
};
```

## Techniki w skrócie

- Duże dane w tablicach typowanych (`Uint16Array`, `Uint32Array`, `Float64Array`), nie w milionach napisów i obiektów:
  każdy napis V8 ma ok. 40 B nagłówka, a GC go śledzi.
- Bity w JS są 32-bitowe, a `BigInt` alokuje; maski 64-bitowe jako dwie połówki, klucze liczbowe do 2^53 w `Float64Array`.
- Napis sklejany z kilku części jest w V8 strukturą „cons” i przy wyszukiwaniu w `Map` musi zostać spłaszczony; lepiej
  kluczować gotowym napisem.
- Bez limitu sterty V8 zbiera leniwie i trzyma ok. 1 GB; limit kosztuje kilka procent czasu.
- Workery nie dzielą pamięci obiektów: zysk z wątków jest mały, a koszt pamięci liniowy.
