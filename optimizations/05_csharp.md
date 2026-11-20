# C#

> Port powstał po Ruście i dostał od razu listę zmian algorytmicznych (`00_common.md`) jako pierwsze kroki: C1 to indeks
> puli, C2 indeks segmentów literówek. Dlatego jego droga zaczyna się blisko Javy, a większość własnych rund dotyczyła
> GC, kompilacji warstwowej i alokacji w ścieżce dokumentu. .NET 10, JIT i NativeAOT z jednego kodu.

## Droga

Po C1 czas pełnego zrzutu bramki spadł w mg z 38 do 15 s, w pl z 42 do 19 s (Java 17 / 19 s); po C2 pl 16 s, mg 13 s
(Java 20 / 16 s). W macierzy końcowej C# JIT ×0,75 czasu Javy, szczyt RSS 278 MB, start 1,4 s
(`../data/language_summary.csv`). NativeAOT jest ×1,2–1,4 wolniejszy od JIT.

## Co pomogło

| zmiana | technika | efekt |
|---|---|---|
| C1, C2 i dalsze indeksy | jak w Javie i Ruście | patrz „Droga” |
| C5: GC bez pracy w tle | `ConcurrentGarbageCollection=false`, odpowiednik Serial GC w Javie | CPU na dokument pl −9% (konstytucje) |
| C8: GC na segmentach | `clrgc.dll` (GC segmentowy dostarczany z runtime) zamiast regionów i próg LOH 16 MB; regiony trzymały ok. 250–300 MB zajętej pamięci przy żywym zbiorze 145–210 MB | szczyt RSS pl ok. 490–500 → 397–418 MB, mg ok. 570–595 → 456–491 MB, czas w szumie |
| próg LOH 1 MB w NativeAOT | tablice na dokument (tokeny, odpowiedzi, języki) przekraczają domyślne 85 KB, a każdy zużyty budżet LOH to był pełny GC po stercie detektorów | (w C5) |
| `ConserveMemory=5` w NativeAOT | GC oszczędzający pamięć, poziom 5 z 0–9 | szczyt RSS pl 434 → 337 MB, mg 529 → 433 MB |
| C13: NativeAOT pod szybkość | `OptimizationPreference=Speed`, `IlcInstructionSet=x86-64-v3` | ×0,91 (pl) i ×0,88 (mg) czasu domyślnego obrazu |
| C15: liczenie wywołań od razu | `TieredCompilation.CallCountingDelayMs=0`; na jednym procesorze ciągła praca odsuwała liczenie wywołań w nieskończoność i metody zostawały na warstwie 0 | jeden procesor: CPU na dokument pl 22,6 → 8,8 ms, przepustowość ×2,5 (pl 42 → 106 dok/s); pełna maszyna: czas ×1,01–1,05, przepustowość ×1,03–1,12 |
| dynamiczne PGO (domyślnie włączone) | JIT optymalizuje ścieżkę dokumentu (interfejsy, delegaty, kolekcje) według profilu z rozgrzewki | bez niego pl ×1,145, mg ×1,119 czasu; stąd przewaga JIT nad NativeAOT |
| `stackalloc` i `Span` | maski odpowiedzi indeksu na stosie, wycinki zamiast podnapisów | część rund C6–C11 (alokacje) |
| SIMD Hamminga | `Vector128<ushort>`: osiem pozycji jednym porównaniem, odległość = `PopCount` maski | 0,19–0,31 czasu pętli skalarnej w mikrobenchmarku; wyłączone domyślnie (`../findings/04_assembler_simd.md`) |

## Co nie pomogło

| zmiana | dlaczego | pomiar |
|---|---|---|
| C13 (P3): interner tokenów z własną tablicą (hasz w przejściu małych liter, ścieżka ASCII) | zysk poniżej progu | 1,5–1,7% czasu dokumentu |
| porcja pracy 4 zamiast 8 słów | — | w szumie |
| `TieredCompilation=0` jako lek na jeden procesor | pomaga (CPU pl 9,4 ms), ale wyłącza też dynamiczne PGO; `CallCountingDelayMs=0` dał więcej | 9,4 vs 8,8 ms |

## Przykład: konfiguracja runtime

Wszystko w `Directory.Build.props`, bez zmian w kodzie (wartości trafiają do `runtimeconfig.json`):

```xml
<PropertyGroup>
  <InvariantGlobalization>true</InvariantGlobalization>          <!-- jak Locale.ROOT w Javie -->
  <ConcurrentGarbageCollection>false</ConcurrentGarbageCollection>
</PropertyGroup>
<PropertyGroup Condition="'$(PublishAot)' == 'true'">
  <OptimizationPreference>Speed</OptimizationPreference>
  <IlcInstructionSet>x86-64-v3</IlcInstructionSet>
</PropertyGroup>
<ItemGroup>
  <!-- JIT: GC segmentowy i próg LOH 16 MB; NativeAOT: regiony, próg 1 MB, ConserveMemory 5 -->
  <RuntimeHostConfigurationOption Include="System.GC.Name" Value="clrgc.dll" />
  <RuntimeHostConfigurationOption Include="System.GC.LOHThreshold" Value="16777216" />
  <RuntimeHostConfigurationOption Include="System.Runtime.TieredCompilation.CallCountingDelayMs" Value="0" />
</ItemGroup>
```

## Przykład: Hamming jednym wektorem

```csharp
public static Vector128<ushort> Load(ReadOnlySpan<char> s)
{
    Span<ushort> u = stackalloc ushort[8];            // bufor na stosie
    u.Fill(MatchingStrategy.Pad);                      // dopełnienie znakiem '#'
    int n = Math.Min(8, s.Length);
    for (int i = 0; i < n; i++) u[i] = s[i];
    return Vector128.Create<ushort>(u);
}

public static bool Matches(int maxDistance, int normalizedLength,
                           Vector128<ushort> a, int la, Vector128<ushort> b, int lb)
{
    int threshold = MatchingStrategy.EffectiveThreshold(maxDistance, normalizedLength, la, lb);
    int end = Math.Min(normalizedLength, Math.Max(la, lb));
    uint diff = (~Vector128.Equals(a, b)).ExtractMostSignificantBits() & ((1u << end) - 1);
    return BitOperations.PopCount(diff) <= threshold;
}
```

C# nie ma wstawek asemblera, ale przenośne `Vector128` JIT tłumaczy na instrukcje SSE i AVX. Detektory zbioru ładuje się
raz jako tablicę wektorów.

## Techniki w skrócie

- Najwięcej dało strojenie runtime'u, nie kodu: GC segmentowy dla RSS, GC bez pracy w tle dla CPU, opóźnienie
  liczenia wywołań dla małej maszyny.
- JIT z dynamicznym PGO był szybszy od NativeAOT, bo uczy się ścieżki dokumentu na tym samym ruchu, który potem mierzy.
- Wartości domyślne GC (regiony, próg LOH 85 KB) zakładają inną aplikację niż ta: duża, stała sterta detektorów
  i krótkie, duże tablice na dokument.
- `Span`, `stackalloc` i `Vector128` dają kontrolę bliską Rustowi bez wychodzenia z bezpiecznego kodu.
